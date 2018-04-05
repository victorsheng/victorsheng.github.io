title: review-java-thread-pool
tags:
  - 多线程
  - 线程池
categories:
  - java
date: 2018-01-31 11:14:41
---
# 线程池
## 背景
- 如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，
- 频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。
- 请求频繁，但是连接上以后读/写很少量的数据就断开连接。考虑到服务的并发问题，如果每个请求来到以后服务都为它启动一个线程，那么这对服务的资源可能会造成很大的浪费。
<!-- more -->

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
corePoolSize（线程池的基本大小）：当提交一个任务时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，直到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreTreads()方法，线程池就会提前创建并启动所有基本线程。

maximumPoolSize（线程最大数量）：线程池允许创建的最大线程数。如果队列已满，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。但如果使用了无界的任务队列，该参数没有效果。

keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。如果任务很多，且每个任务执行时间较短，可调大该值。

TimeUnit（线程活动保持时间的单位）：keepAliveTime的时间度量单位。可选天、小时、分钟、毫秒、微妙、纳秒。

BlockingQueue<Runnable>（任务队列）：用于保存等待执行的任务的阻塞嘟列，可以选择以下几个阻塞队列

**ArrayBlockingQueue**：基于数组结构的有界阻塞队列

**LinkedBlockingQueue**：基于链表机构的阻塞队列，吞吐量通常高于ArrayBlockingQueue，静态工厂方法Executors.newFixedThreadPool()使用该队列。

**SynchronousQueue**：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除，否则插入操作一直处于阻塞状态，吞吐量通常高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool()使用该队列。

**PriorityBlockingQueue**：具有优先级的无限阻塞队列。

ThreadFactory：创建线程的工厂。

RejectedExecutionHandler：饱和策略，即队列和线程池都满了，对于新提交的任务无法执行，这时采取的处理新来的任务的方法，有4种策略可选（也可以自定义策略---实现RejectedExecutionHandler接口，如记录日志或持久化不能处理的任务）：

**CallerRunsPolicy**：使用调用者所在的线程来运行任务。

**AbortPolicy**：直接抛出RejectedExecutionException异常。（默认策略）

**DiscardPolicy**：对新任务直接丢弃，不做任何事情

**DiscardOldestPolicy**：丢掉队列里最近（the oldest unhandled）的一个任务，并执行当前新任务。



## 线程池的关闭
- ThreadPoolExecutor 提供了两个方法，用于线程池的关闭，分别是 shutdown() 和 shutdownNow()。

- shutdown()：不会立即的终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务。
- shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务。

## 核心流程


![upload successful](/images/pasted-79.png)

## 线程池规则
线程池的线程执行规则跟任务队列有很大的关系。

- 下面都假设任务队列没有大小限制：

	+ 如果线程数量<=核心线程数量，那么直接启动一个核心线程来执行任务，不会放入队列中。
	+ 如果线程数量>核心线程数，但<=最大线程数，并且任务队列是LinkedBlockingDeque的时候，超过核心线程数量的任务会放在任务队列中排队。
	+ 如果线程数量>核心线程数，但<=最大线程数，并且任务队列是SynchronousQueue的时候，线程池会创建新线程执行任务，这些任务也不会被放在任务队列中。这些线程属于非核心线程，在任务完成后，闲置时间达到了超时时间就会被清除。
	+ 如果线程数量>核心线程数，并且>最大线程数，当任务队列是LinkedBlockingDeque，会将超过核心线程的任务放在任务队列中排队。也就是当任务队列是LinkedBlockingDeque并且没有大小限制时，线程池的最大线程数设置是无效的，他的线程数最多不会超过核心线程数。
	+ 如果线程数量>核心线程数，并且>最大线程数，当任务队列是SynchronousQueue的时候，会因为线程池拒绝添加任务而抛出异常。

- 任务队列大小有限时

	+ 当LinkedBlockingDeque塞满时，新增的任务会直接创建新线程来执行，当创建的线程数量超过最大线程数量时会抛异常。
	+ SynchronousQueue没有数量限制。因为他根本不保持这些任务，而是直接交给线程池去执行。当任务数量超过最大线程数时会直接抛异常。

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


## 实际案例1(优化分页查询)
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

## 实际案例2(dts抓单)
```
executor = new ThreadPoolExecutor(config.getMaxActive()/2+1, config.getMaxActive(), config.getTimeout4Borrow(), TimeUnit.SECONDS,
				new ArrayBlockingQueue<>(config.getMaxActive()), this);

	@Override
	public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
		logger.error("wait too long to borrow worker:  " + taskOwner);
		try {
			getMQService().push(getConfig().getAlertMq(), taskOwner);
		} catch (Exception e) {
			logger.error("mq service error: ", e);
		}
		logger.warn("worker {} need sleep {} millis to back to working   ", taskOwner, getConfig().getTimeout4Borrow());
		try {
			Thread.sleep(getConfig().getTimeout4Borrow() * 5);
		} catch (Exception e) {
			logger.error("thread error :", e);
		}
	}

```

## CompletionService接口
- 根据上面的介绍我们知道，现在在Java中使用多线程通常不会再使用Thread对象了。而是会用到java.util.concurrent包下的ExecutorService来初始化一个线程池供我们使用。使用ExecutorService类的时候，我们常维护一个list保存submit的callable task所返回的Future对象。然后在主线程中遍历这个list并调用Future的get()方法取到Task的返回值。

- 其实除了使用ExecutorService外，还可通过CompletionService包装ExecutorService，然后调用其take()方法去取Future对象。

- CompletionService和ExecutorService的主要的区别在于submit的task不一定是按照加入自己维护的list顺序完成的。

- ExecutorService中从list中遍历的每个Future对象并不一定处于完成状态，这时调用get()方法就会被阻塞住，如果系统是设计成每个线程完成后就能根据其结果继续做后面的事，这样对于处于list后面的但是先完成的线程就会增加了额外的等待时间。

- 而CompletionService的实现是维护一个保存Future对象的BlockingQueue。只有当这个Future对象状态是结束的时候，才会加入到这个Queue中，take()方法其实就是Producer-Consumer中的Consumer。它会从Queue中取出Future对象，如果Queue是空的，就会阻塞在那里，直到有完成的Future对象加入到Queue中。所以，先完成的必定先被取出。这样就减少了不必要的等待时间。

## 合理的配置线程池
要想合理的配置线程池，就必须首先分析任务特性，可以从以下几个角度来进行分析：

任务的性质：CPU密集型任务，IO密集型任务和混合型任务。
任务的优先级：高，中和低。
任务的执行时间：长，中和短。
任务的依赖性：是否依赖其他系统资源，如数据库连接。
任务性质不同的任务可以用不同规模的线程池分开处理。CPU密集型任务配置尽可能小的线程，如配置Ncpu+1个线程的线程池。IO密集型任务则由于线程并不是一直在执行任务，则配置尽可能多的线程，如2*Ncpu。混合型的任务，如果可以拆分，则将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐率要高于串行执行的吞吐率，如果这两个任务执行时间相差太大，则没必要进行分解。我们可以通过Runtime.getRuntime().availableProcessors()方法获得当前设备的CPU个数。

优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先得到执行，需要注意的是如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。

执行时间不同的任务可以交给不同规模的线程池来处理，或者也可以使用优先级队列，让执行时间短的任务先执行。

依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，如果等待的时间越长CPU空闲时间就越长，那么线程数应该设置越大，这样才能更好的利用CPU。

建议使用有界队列，有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点，比如几千。有一次我们组使用的后台任务线程池的队列和线程池全满了，不断的抛出抛弃任务的异常，通过排查发现是数据库出现了问题，导致执行SQL变得非常缓慢，因为后台任务线程池里的任务全是需要向数据库查询和插入数据的，所以导致线程池里的工作线程全部阻塞住，任务积压在线程池里。如果当时我们设置成无界队列，线程池的队列就会越来越多，有可能会撑满内存，导致整个系统不可用，而不只是后台任务出现问题。当然我们的系统所有的任务是用的单独的服务器部署的，而我们使用不同规模的线程池跑不同类型的任务，但是出现这样问题时也会影响到其他任务。

## 线程池的监控

通过线程池提供的参数进行监控。线程池里有一些属性在监控线程池的时候可以使用

taskCount：线程池需要执行的任务数量。
completedTaskCount：线程池在运行过程中已完成的任务数量。小于或等于taskCount。
largestPoolSize：线程池曾经创建过的最大线程数量。通过这个数据可以知道线程池是否满过。如等于线程池的最大大小，则表示线程池曾经满了。
getPoolSize:线程池的线程数量。如果线程池不销毁的话，池里的线程不会自动销毁，所以这个大小只增不+ getActiveCount：获取活动的线程数。
通过扩展线程池进行监控。通过继承线程池并重写线程池的beforeExecute，afterExecute和terminated方法，我们可以在任务执行前，执行后和线程池关闭前干一些事情。如监控任务的平均执行时间，最大执行时间和最小执行时间等。这几个方法在线程池里是空方法。如：

## SingleThreadExecutor

## CachedThreadPool
可缓存线程池：

- 线程数无限制
- 有空闲线程则复用空闲线程，若无空闲线程则新建线程
- 一定程序减少频繁创建/销毁线程，减少系统开销

①使用无容量队列SynchronousQueue，但maxmumPoolSize无界。如果提交任务的速度大于线程处理任务的速度，将会不断创建新线程，极端情况会因为创建过多线程而耗尽CPU资源。
②keepAliveTime为60s，空闲线程超过该时间将会终止。
③执行完任务的某线程会执行SynchronousQueue.poll()从队列中取任务，这个取的动作会持续60s，如果在60s内有新的任务，则执行新的任务，没有任务则终止线程。因此长时间保持空闲的CachedThreadPool不会占用任何资源。
④当有任务提交时，a.如果当前线程池为空或者已创建的线程都正在处理任务，则CachedThreadPool会创建新线程来执行该任务。b.如果当前线程池有空闲的线程（正在执行阻塞方法SynchronousQueue.poll()），则将任务交给该等待任务的空闲线程来执行。

<b>CachedThreadPool适用于执行很多的短期异步任务的小程序或者是负载较轻的服务器。</b>
```
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
}
```

## FixedThreadPool

定长线程池：

- 可控制线程最大并发数（同时执行的线程数）
- 超出的线程会在队列中等待
```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

# 排队策略
## 直接提交

工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集时出现锁。直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

## 无界队列

使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize 的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web 页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

## 有界队列

当使用有限的 maximumPoolSizes 时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O 边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU 使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。  


# 拒绝策略:
RejectedExecutionHandler接口提供了对于拒绝任务的处理的自定方法的机会。在ThreadPoolExecutor中已经默认包含了4中策略

## CallerRunsPolicy：

- 线程调用运行该任务的 execute 本身。此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。
- 这个策略显然不想放弃执行任务。但是由于池中已经没有任何资源了，那么就直接使用调用该execute的线程本身来执行。

## AbortPolicy：

- 处理程序遭到拒绝将抛出运行时 RejectedExecutionException
- 这种策略直接抛出异常，丢弃任务。

## DiscardPolicy：

- 不能执行的任务将被删除
- 这种策略和AbortPolicy几乎一样，也是丢弃任务，只不过他不抛出异常。

##  DiscardOldestPolicy：

- 如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重试执行程序（如果再次失败，则重复此过程）
- 该策略就稍微复杂一些，在pool没有关闭的前提下首先丢掉缓存在队列中的最早的任务，然后重新尝试运行该任务。这个策略需要适当小心。
- 设想:如果其他线程都还在运行，那么新来任务踢掉旧任务，缓存在queue中，再来一个任务又会踢掉queue中最老任务。


# 源码解析
构造方法
```
public ThreadPoolExecutor(int corePoolSize,  
                           int maximumPoolSize,  
                           long keepAliveTime,  
                           TimeUnit unit,  
                           BlockingQueue<Runnable> workQueue,  
                           ThreadFactory threadFactory,  
                           RejectedExecutionHandler handler) {  
     if (corePoolSize < 0 ||  
         maximumPoolSize <= 0 ||  
         maximumPoolSize < corePoolSize ||  
         keepAliveTime < 0)  
         throw new IllegalArgumentException();  
     if (workQueue == null || threadFactory == null || handler == null)  
         throw new NullPointerException();  
     this.corePoolSize = corePoolSize;  
     this.maximumPoolSize = maximumPoolSize;  
     this.workQueue = workQueue;  
     this.keepAliveTime = unit.toNanos(keepAliveTime);  
     this.threadFactory = threadFactory;  
     this.handler = handler;  
 }  
```

ctl是ThreadPoolExecutor的一个重要属性，它记录着ThreadPoolExecutor的线程数量和线程状态。

`private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));`

Integer有32位，其中前三位用于记录线程状态，后29位用于记录线程的数量。那线程状态有哪些呢？
```
//线程数量占用的位数
private static final int COUNT_BITS = Integer.SIZE - 3;

//最大线程数
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl

//状态值就是只关心前三位的值，所以把后29位清0
private static int runStateOf(int c)     { return c & ~CAPACITY; }

//线程数量就是只关心后29位的值，所以把前3位清0
private static int workerCountOf(int c)  { return c & CAPACITY; }

//两个数相或
private static int ctlOf(int rs, int wc) { return rs | wc; }
```


通常你得到线程池后，会调用其中的：submit方法或execute方法去操作；其实你会发现，submit方法最终会调用execute方法来进行操作，只是他提供了一个Future来托管返回值的处理而已，当你调用需要有返回值的信息时，你用它来处理是比较好的；这个Future会包装对Callable信息，并定义一个Sync对象（），当你发生读取返回值的操作的时候，会通过Sync对象进入锁，直到有返回值的数据通知，具体细节先不要看太多，继续向下：

```
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}
```

```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```

我们从上面的execute代码中可以看到，当提交一个任务时，当前线程数小于corePoolSize核心线程数的时候，就新添加一个线程，即addWorker(command, true)
```
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        final ReentrantLock mainLock = this.mainLock;
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int c = ctl.get();
                int rs = runStateOf(c);

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```
addWorker有2种情况，一种就是线程数量不足核心线程数，另一种就是核心线程数已满同时任务队列已满但是线程数不足最大线程数。上述boolean core就是用来区分上述2种情况的。

addWorker首先就要对线程数量自增，即ctl的后29为进行自增。这里就涉及到多线程问题，为了解决多线程问题就采用了for循环加CAS来解决，为什么没有直接用AtomicInteger的incrementAndGet？

incrementAndGet也是内部for循环加CAS，它是要确保一定要自增成功的，而这里我们不一定要自增成功，还要判断当前线程的数量合不合法。 即如果core=true,则当前线程数量不能超过incrementAndGet，如果core=false，则当前线程数量不能超过maximumPoolSize。

如果当前线程数自增成功，下面就需要创建出Worker线程，存放到workers中，如下所述
```
private final HashSet<Worker> workers = new HashSet<Worker>();

```
我们知道HashSet的内部实现就是通过HashMap来实现的，HashMap是线程不安全的，所以在对workers操作的时候必须要要进行加锁，这就用到了ThreadPoolExecutor的mainLock了

一旦worker新增成功就直接启动该Worker内部的线程。一旦worker新增失败则调用addWorkerFailed处理失败逻辑
- 创建出Worker对象时，内部分配的线程Thread为空，这个线程的创造是由线程工厂ThreadFactory负责的创造的，可能为null
- 在获取mainLock之后，发现当前线程池已被标记成大于SHUTDOWN状态、或者是SHUTDOWN但是firstTask不为null。当线程状态大于SHUTDOWN，当然addWorker要失败。后者怎么解释呢？这里就要详细解释下SHUTDOWN状态
SHUTDOWN即不再接收新的task，但是可以继续处理队列中的task。当线程数小于核心线程数的时候，提交的task作为新创建的Worker的firstTask，即firstTask不为null。当线程数大于核心线程数后，此时addWorker中创建的Worker的firstTask就是null,它只负责从队列中取出任务。所以firstTask不为null的时候就表明是新提交的任务，SHUTDOWN状态下是不允许新提交任务的，所以这种情况也要失败
```
private void addWorkerFailed(Worker w) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        if (w != null)
            workers.remove(w);
        decrementWorkerCount();
        tryTerminate();
    } finally {
        mainLock.unlock();
    }
}
```


Worker解析

```
private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable
    {
        /**
         * This class will never be serialized, but we provide a
         * serialVersionUID to suppress a javac warning.
         */
        private static final long serialVersionUID = 6138294804551838833L;

        /** Thread this worker is running in.  Null if factory fails. */
        final Thread thread;
        /** Initial task to run.  Possibly null. */
        Runnable firstTask;
        /** Per-thread task counter */
        volatile long completedTasks;

```
Worker是ThreadPoolExecutor的内部类
Worker继承了AQS即AbstractQueuedSynchronizer，用于实现锁的机制，这里先抛出一个问题：为什么要继承AQS
实现了Runnable接口，则该Worker就可以作为Thread的参数创建出Thread，线程的运行即运行该Worker的run方法，该方法中会不断的从队列中取出任务并执行
我们来详细看下Worker的run过程
```
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```
- getTask逻辑，我们从上面看到while循环中一旦getTask为null就直接退出while循环了，即Worker走向结束了，所以空闲的时候会阻塞在getTask中，一直等到获取到task或者超时。
- 为什么每次执行task都要获取锁
- worker的退出，就不再详细说明了，各位可自行研究

getTask逻辑
```
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        boolean timed;      // Are workers subject to culling?

        for (;;) {
            int wc = workerCountOf(c);
            timed = allowCoreThreadTimeOut || wc > corePoolSize;

            if (wc <= maximumPoolSize && ! (timedOut && timed))
                break;
            if (compareAndDecrementWorkerCount(c))
                return null;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }

        try {
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```
从workQueue队列中获取task有2种方法，一种就是阻塞式获取直到有task任务，另一种就是阻塞一定时间，超时则就直接返回null了。此时返回null意味着Worker就要走向结束了。

当allowCoreThreadTimeOut=true即核心线程也开始timeout计时，或者wc > corePoolSize即当前线程数超过了核心线程数也要开启计时，获取task就采用阻塞一定时间获取，一旦超时即该Worker在keepAliveTime时间内都没获取到task即处于空闲状态，这时候就返回null，即意味着该Worker就走向结束了

其他情况就是不用进行线程空闲计时，即可以一直阻塞直到有task来。

接下来一个重点问题就是每次执行task的时候为什么要先获取锁？

首先该Worker的run方法只可能被一个线程来运行，即该Worker的run方法不可能出现多线程同时运行的情况。那就是Worker有一些资源是多个线程共享的，是什么呢？我们先来看看Worker继承AQS即AbstractQueuedSynchronizer的实现情况
```
protected boolean tryAcquire(int unused) {
    if (compareAndSetState(0, 1)) {
        setExclusiveOwnerThread(Thread.currentThread());
        return true;
    }
    return false;
}

protected boolean tryRelease(int unused) {
    setExclusiveOwnerThread(null);
    setState(0);
    return true;
}

public void lock()        { acquire(1); }
public boolean tryLock()  { return tryAcquire(1); }
public void unlock()      { release(1); }
public boolean isLocked() { return isHeldExclusively(); }
```
这里就是一个简单的独占锁，但是重点是不可重入的，重入即当前线程获取锁了，还可以再次获取锁，来简单对比下重入锁的实现
```
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

也就是说Worker本身就是一个简单的独占锁，并且是不可重入的。这个锁的引入到底是为了什么呢？为什么需要在每个task执行前都要获取这个锁呢？这个当时也没太理解，但是要想找出这个原因，可以有如下思路来找：

就看看哪些地方在调用Worker获取锁的方法，获取锁的方法有lock、tryLock

最终发现interruptIdleWorkers会调用tryLock方法，如下：
```
private void interruptIdleWorkers(boolean onlyOne) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers) {
            Thread t = w.thread;
            if (!t.isInterrupted() && w.tryLock()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                } finally {
                    w.unlock();
                }
            }
            if (onlyOne)
                break;
        }
    } finally {
        mainLock.unlock();
    }
}
```

该方法就是用于中断那些空闲的Worker，怎么判断一个Worker是否空闲呢？这里就是使用w.tryLock()是否能获取锁来表示一个Worker是否空闲。当Worker在处理任务的时候即不空闲都会获取lock，所以这里就是依据Worker的锁是否被占用了来判定一个Worker是否空闲。

如当重新设置一个ThreadPoolExecutor的核心线程数的时候，如果当前线程数大于了新设置的核心线程数，就需要中断那些空闲的线程
```
public void setCorePoolSize(int corePoolSize) {
    if (corePoolSize < 0)
        throw new IllegalArgumentException();
    int delta = corePoolSize - this.corePoolSize;
    this.corePoolSize = corePoolSize;
    if (workerCountOf(ctl.get()) > corePoolSize)
        interruptIdleWorkers();
    else if (delta > 0) {
        // We don't really know how many new threads are "needed".
        // As a heuristic, prestart enough new workers (up to new
        // core size) to handle the current number of tasks in
        // queue, but stop if queue becomes empty while doing so.
        int k = Math.min(delta, workQueue.size());
        while (k-- > 0 && addWorker(null, true)) {
            if (workQueue.isEmpty())
                break;
        }
    }
}
```


# 参考
https://my.oschina.net/pingpangkuangmo/blog/668520
