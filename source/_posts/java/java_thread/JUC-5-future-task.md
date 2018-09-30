---
title: future-task
abbrlink: 3822853633
date: 2018-04-04 08:53:18
tags:
categories:
---
```
public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
```
从上面看到就是对Callable封装成一个新的任务，即FutureTask，调用Executor的原始接口execute方法来执行FutureTask，并且返回给用户FutureTask对象，用于追踪任务的状态和数据，下面就需要我们来详细看看FutureTask如何对任务进行封装的

# FutureTask
## FutureTask的属性和构造函数
```
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;

/** The underlying callable; nulled out after running */
private Callable<V> callable;
/** The result to return or exception to throw from get() */
private Object outcome; // non-volatile, protected by state reads/writes
/** The thread running the callable; CASed during run() */
private volatile Thread runner;
/** Treiber stack of waiting threads */
private volatile WaitNode waiters;

public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```
- 有一个状态变量state,
- 一个Callable callable即原始任务，
- Object outcome存放原始任务的输出结果或者异常，
- Thread runner运行该任务的线程，
- WaitNode waiters等待获取任务结果的等待者

## 3.2 FutureTask的get方法实现
使用FutureTask阻塞式等待任务执行结果，一种是永远阻塞另一种就是阻塞一定时间否则报超时异常，如下2个方法
```
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}

public V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException {
    if (unit == null)
        throw new NullPointerException();
    int s = state;
    if (s <= COMPLETING &&
        (s = awaitDone(true, unit.toNanos(timeout))) <= COMPLETING)
        throw new TimeoutException();
    return report(s);
}
```
阻塞式等待的核心逻辑就在上述awaitDone方法中，来详细看看
```
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {
        if (Thread.interrupted()) {
            removeWaiter(q);
            throw new InterruptedException();
        }

        int s = state;
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;
            return s;
        }
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
        else if (q == null)
            q = new WaitNode();
        else if (!queued)
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);
        else if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
            LockSupport.parkNanos(this, nanos);
        }
        else
            LockSupport.park(this);
    }
}
```
可以看到有一个for循环不断处理着各种情况：

- 1 从最开始的WaitNode q = null，构建了一个WaitNode，即代表着当前线程作为一个等待者，WaitNode就是一个简单的链表，如下

```
static final class WaitNode {
    volatile Thread thread;
    volatile WaitNode next;
    WaitNode() { thread = Thread.currentThread(); }
}
```
- 2 构建好WaitNode之后就要将该WaitNode放入链表中，这时候就会涉及多线程问题，使用UNSAFE的CAS来解决，这种方式也是AtomicLong等众多原子类的底层实现方式

- 3 成功放入WaitNode链表之后，采用LockSupport的park阻塞当前线程，要么只阻塞一定时间要么一直阻塞，直到被LockSupport的unpark唤醒。LockSupport在锁的底层实现AQS中也非常常见，使用了LockSupport就可以不用在for循环里不断判断当前任务状态而浪费CPU，只需要当前任务完成之后，使用LockSupport对等待线程进行unpark，就可以使等待的线程退出等待继续往下执行

- 4 如果LockSupport阻塞时间到了，还未收到unpark，则需要从等待者链表中删除当前线程代表的等待者

## FutureTask的任务执行过程
```
public void run() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```
1 一旦FutureTask任务开始执行了，就需要将当前执行线程设置到FutureTask的volatile Thread runner属性中

2 执行原始任务Callable的call方法，可能成功也可能失败也可能被中断被取消

文档中有如下状态的迁移过程：

```
Possible state transitions:
 * NEW -> COMPLETING -> NORMAL
 * NEW -> COMPLETING -> EXCEPTIONAL
 * NEW -> CANCELLED
 * NEW -> INTERRUPTING -> INTERRUPTED
```

成功和失败方法
```
protected void set(V v) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = v;
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
        finishCompletion();
    }
}

protected void setException(Throwable t) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = t;
        UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL); // final state
        finishCompletion();
    }
}
```
都是首先将状态变成COMPLETING正在结束中，然后设置outcome，成功则设置正常的返回值，失败则设置成异常，然后根据划定最终的状态结果，成功就是NORMAL，失败就是EXCEPTIONAL，最后呢调用finishCompletion，去unpark之前说的WaitNode中对应的线程们

```
private void finishCompletion() {
    // assert state > COMPLETING;
    for (WaitNode q; (q = waiters) != null;) {
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
            for (;;) {
                Thread t = q.thread;
                if (t != null) {
                    q.thread = null;
                    LockSupport.unpark(t);
                }
                WaitNode next = q.next;
                if (next == null)
                    break;
                q.next = null; // unlink to help gc
                q = next;
            }
            break;
        }
    }

    done();

    callable = null;        // to reduce footprint
}
```
这里就是遍历WaitNode链表，对每一个WaitNode对应的线程依次进行LockSupport.unpark(t)，使其结束阻塞。WaitNode通知完毕后，调用一个done方法，目前该方法是空的实现，所以你如果想在任务完成后执行一些动作的时候就可以重写该方法


## FutureTask任务的取消
```
public boolean cancel(boolean mayInterruptIfRunning) {
    if (state != NEW)
        return false;
    if (mayInterruptIfRunning) {
        if (!UNSAFE.compareAndSwapInt(this, stateOffset, NEW, INTERRUPTING))
            return false;
        Thread t = runner;
        if (t != null)
            t.interrupt();
        UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED); // final state
    }
    else if (!UNSAFE.compareAndSwapInt(this, stateOffset, NEW, CANCELLED))
        return false;
    finishCompletion();
    return true;
}
```

取消任务，有2种情况，一种该任务正在运行，一种就是非运行状态，所以需要用户给出明示是否中断正在运行的任务，即需要一个参数mayInterruptIfRunning

中断任务就是通过中断运行该任务的线程，即直接调用该线程的interrupt()方法


# 结束语

FutureTask大部分就简单分析完了，其他的自己去看下就行了。至此我们了解了一个任务被提交经过了封装，变成了一个新的任务FutureTask,同时FutureTask也明确了该任务的整个执行过程，只留出核心execute(futureTask)方法需要被子类来实现


# 参考
https://my.oschina.net/pingpangkuangmo/blog/666762
