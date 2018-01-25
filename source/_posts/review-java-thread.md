title: java-线程复习
date: 2018-01-13 10:13:27
tags:
categories:
---
# 线程
创建线程有两种方式：
一、继承 Thread 类，扩展线程。
二、实现 Runnable 接口。

Callable接口
Future接口
FutureTask实现了 Future 接口
FutureTask 的好处是 FutureTask 是为了弥补 Thread 的不足而设计的，它可以让程序员准确地知道线程什么时候执行完成并获得到线程执行完成后返回的结果。FutureTask 是一种可以取消的异步的计算任务，它的计算是通过 Callable 实现的，它等价于可以携带结果的 Runnable，并且有三个状态：等待、运行和完成。完成包括所有计算以任意的方式结束，包括正常结束、取消和异常。


# 多线程
多线程的概念很好理解就是多条线程同时存在，但要用好多线程确不容易，涉及到多线程间通信，多线程共用一个资源等诸多问题。
## synchronized
解决多个线程之间访问资源的同步性
- 对于同步方法，锁是当前实例对象。
- 对于静态同步方法，锁是当前对象的Class对象。
- 对于同步方法块，锁是Synchonized括号里配置的对象。
## volatile
解决变量在多个线程之间的可见性
## wait()、notify()、notifyAll()
wait():我的部分已经做完了,等别人让我做的时候再做,释放锁
notify():等我做完了,释放锁后,让其他一个人做
wait() 与 Thread.sleep(long time) 的区别:sleep傻等,sleep不释放锁


## ThreadLocal 变量
线程隔离
InheritableTreadLocal让子线程获取父线程中取得值

## join() 方法
等待线程对象销毁
等待子进程执行完,我再执行
Thread.yield() 方法



# ReentrantLock
## lock()
## unlock()

## condition

### 等待/通知
- awiate() 相当于Object类中的wait()方法
- await(long time,TImeUnit unit) 相当于Object类中的wait()方法
- signal() 相当于Object类中的notify()方法
- sigalAll() 相当于Object类中的notifyAll()方法

### 通知部分线程
Lock lock=new Reentrantock();
Condition conditionA=lock.newCondition();
Condition conditionB=lock.newCondition();

### 公平锁,非公平锁
- 获取锁的顺序是按照线程加锁的顺序来分配的,及先来先得的FIFO先进先出顺序

### getHoldCount(),getQueueLength(),getWaiteQueueLength()
- getHoldCount()查询当前线程保持此锁定的个数,调用lock()方法的次数
- getQueueLength()返回正在等待次锁定的线程估计数
- getWaiteQueueLength(Condition condition)返回等待与此锁定翔安的给定条件Condition的线程估计数


# 线程状态切换

![upload successful](/images/pasted-42.png)
- 监视器代表synchronized
- lock代表锁


# 线程池
## 背景
- 如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，
- 频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。
- 请求频繁，但是连接上以后读/写很少量的数据就断开连接。考虑到服务的并发问题，如果每个请求来到以后服务都为它启动一个线程，那么这对服务的资源可能会造成很大的浪费。


## 优点

- 第一：降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- 第二：提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。
- 第三：提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

## 核心接口ExecutorService,Executor

![upload successful](/images/pasted-19.png)


## 类
- ExecutorService：真正的线程池接口。
- ScheduledExecutorService:能和Timer/TimerTask类似，解决那些需要任务重复执行的问题。
- ThreadPoolExecutor:ExecutorService的默认实现。
- ScheduledThreadPoolExecutor:继承ThreadPoolExecutor的- - ScheduledExecutorService接口实现，周期性任务调度的类实现。
- Executors类里面提供了一些静态工厂，生成一些常用的线程池。


## Executors 提供四种线程池

- 1）newCachedThreadPool 是一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。对于执行很多短期异步任务的程序而言，这些线程池通常可提高程序性能。调用 execute() 将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。因此，长时间保持空闲的线程池不会使用任何资源。注意，可以使用 ThreadPoolExecutor 构造方法创建具有类似属性但细节不同（例如超时参数）的线程池。

- 2）newSingleThreadExecutor 创建是一个单线程池，也就是该线程池只有一个线程在工作，所有的任务是串行执行的，如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它，此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

- 3）newFixedThreadPool 创建固定大小的线程池，每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小，线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。

- 4）newScheduledThreadPool 创建一个大小无限的线程池，此线程池支持定时以及周期性执行任务的需求。

## 程池相关参数
- 1）corePoolSize：线程池的核心线程数，一般情况下不管有没有任务都会一直在线程池中一直存活，只有在 ThreadPoolExecutor 中的方法 allowCoreThreadTimeOut(boolean value) 设置为 true 时，闲置的核心线程会存在超时机制，如果在指定时间没有新任务来时，核心线程也会被终止，而这个时间间隔由第3个属性 keepAliveTime 指定。

- 2）maximumPoolSize：线程池所能容纳的最大线程数，当活动的线程数达到这个值后，后续的新任务将会被<b>阻塞</b>。

- 3）keepAliveTime：控制线程闲置时的超时时长，超过则终止该线程。一般情况下用于非核心线程，只有在 ThreadPoolExecutor 中的方法 allowCoreThreadTimeOut(boolean value) 设置为 true时，也作用于核心线程。

- 4）unit：用于指定 keepAliveTime 参数的时间单位，TimeUnit 是个 enum 枚举类型，常用的有：TimeUnit.HOURS(小时)、TimeUnit.MINUTES(分钟)、TimeUnit.SECONDS(秒) 和 TimeUnit.MILLISECONDS(毫秒)等。

- 5）workQueue：线程池的任务队列，通过线程池的 execute(Runnable command) 方法会将任务 Runnable 存储在队列中。

- 6）threadFactory：线程工厂，它是一个接口，用来为线程池创建新线程的。

## 线程池的关闭
- ThreadPoolExecutor 提供了两个方法，用于线程池的关闭，分别是 shutdown() 和 shutdownNow()。

- shutdown()：不会立即的终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务。
- shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务。

## 使用方法
```
 void execute(Runnable task) 
 
 
 Future<?> submit(Runnable task)
 
 
 <T> Future<T> submit(Callable<T> task)
 
 
<T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
 全部执行,其中一个执行结束,或异常,则取消其他Callable的运行
 
 
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
 方法 invokeAll() 会调用存在于参数集合中的所有 Callable 对象，并且返回壹個包含 Future 对象的集合
```

- future.isDone() //return true,false 无阻塞
- future.get() // return 返回值，阻塞直到该线程运行结束


## 实际案例
```
        ExecutorService slaver = Executors.newSingleThreadExecutor();
        FutureTask<List<Map<String, Object>>> lastFuture = new FutureTask<>(countCallable);
        slaver.execute(lastFuture);
        slaver.shutdown();

        List<Map<String, Object>> stockFirst = jdbcTemplate.queryForList(contentSql);
        List<Map<String, Object>> stockLast = null;
        try {
            stockLast = lastFuture.get();
        } catch (Exception e) {
            logger.error("错误", e);
        } finally {
            lastFuture.cancel(true);
        }
```

## CompletionService接口
- 根据上面的介绍我们知道，现在在Java中使用多线程通常不会再使用Thread对象了。而是会用到java.util.concurrent包下的ExecutorService来初始化一个线程池供我们使用。使用ExecutorService类的时候，我们常维护一个list保存submit的callable task所返回的Future对象。然后在主线程中遍历这个list并调用Future的get()方法取到Task的返回值。

- 其实除了使用ExecutorService外，还可通过CompletionService包装ExecutorService，然后调用其take()方法去取Future对象。

- CompletionService和ExecutorService的主要的区别在于submit的task不一定是按照加入自己维护的list顺序完成的。

- ExecutorService中从list中遍历的每个Future对象并不一定处于完成状态，这时调用get()方法就会被阻塞住，如果系统是设计成每个线程完成后就能根据其结果继续做后面的事，这样对于处于list后面的但是先完成的线程就会增加了额外的等待时间。

- 而CompletionService的实现是维护一个保存Future对象的BlockingQueue。只有当这个Future对象状态是结束的时候，才会加入到这个Queue中，take()方法其实就是Producer-Consumer中的Consumer。它会从Queue中取出Future对象，如果Queue是空的，就会阻塞在那里，直到有完成的Future对象加入到Queue中。所以，先完成的必定先被取出。这样就减少了不必要的等待时间。

# AtomicInteger类

# ReentrantReadWriteLock