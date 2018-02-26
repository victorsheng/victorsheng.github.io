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

maximumPoolSize（线程最大数量）：线程池允许创建的最大线程数。如果队列已满，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。但如果使用了无解的任务队列，该参数没有效果。

keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。如果任务很多，且每个任务执行时间较短，可调大该值。

TimeUnit（线程活动保持时间的单位）：keepAliveTime的时间度量单位。可选天、小时、分钟、毫秒、微妙、纳秒。

BlockingQueue<Runnable>（任务队列）：用于保存等待执行的任务的阻塞嘟列，可以选择以下几个阻塞队列

ArrayBlockingQueue：基于数组结构的有界阻塞队列

LinkedBlockingQueue：基于链表机构的阻塞队列，吞吐量通常高于ArrayBlockingQueue，静态工厂方法Executors.newFixedThreadPool()使用该队列。

SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除，否则插入操作一直处于阻塞状态，吞吐量通常高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool()使用该队列。

PriorityBlockingQueue：具有优先级的无限阻塞队列。

ThreadFactory：创建线程的工厂。

RejectedExecutionHandler：饱和策略，即队列和线程池都满了，对于新提交的任务无法执行，这时采取的处理新来的任务的方法，有4种策略可选（也可以自定义策略---实现RejectedExecutionHandler接口，如记录日志或持久化不能处理的任务）：

CallerRunsPolicy：使用调用者所在的线程来运行任务。

AbortPolicy：直接抛出RejectedExecutionException异常。（默认策略）

DiscardPolicy：对新任务直接丢弃，不做任何事情

DiscardOldestPolicy：丢掉队列里最近（the oldest unhandled）的一个任务，并执行当前新任务。



## 线程池的关闭
- ThreadPoolExecutor 提供了两个方法，用于线程池的关闭，分别是 shutdown() 和 shutdownNow()。

- shutdown()：不会立即的终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务。
- shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务。

## 核心流程

![upload successful](/images/pasted-47.png)

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

## SingleThreadExecutor

## CachedThreadPool
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
