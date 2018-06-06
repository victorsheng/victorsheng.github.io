title: dubbo线程池相关源码解析
tags:
  - dubbo
  - 线程池
  - AbortPolicy
  - jstack
categories:
  - dubbo
date: 2018-02-02 10:55:00
---
# dubbo几种线程池选择
http://dubbo.io/books/dubbo-user-book/demos/thread-model.html
ThreadPool

- fixed 固定大小线程池，启动时建立线程，不关闭，一直持有。(缺省)
- cached 缓存线程池，空闲一分钟自动删除，需要时重建。
- limited 可伸缩线程池，但池中的线程数只会增长不会收缩。只增长不收缩的目的是为了避免收缩时突然来了大流量引起的性能问题。



读取几个参数:
- threads:服务线程池大小(固定大小)
- queues:线程池队列大小，当线程池满时，排队等待执行的队列大小，建议不要设置，当线程程池时应立即失败，重试其它服务提供机器，而不是排队，除非有特殊需求

```
public class FixedThreadPool implements ThreadPool {

    public Executor getExecutor(URL url) {
        String name = url.getParameter(Constants.THREAD_NAME_KEY, Constants.DEFAULT_THREAD_NAME);
        int threads = url.getParameter(Constants.THREADS_KEY, Constants.DEFAULT_THREADS);
        int queues = url.getParameter(Constants.QUEUES_KEY, Constants.DEFAULT_QUEUES);
        return new ThreadPoolExecutor(threads, threads, 0, TimeUnit.MILLISECONDS,
                queues == 0 ? new SynchronousQueue<Runnable>() :
                        (queues < 0 ? new LinkedBlockingQueue<Runnable>()
                                : new LinkedBlockingQueue<Runnable>(queues)),
                new NamedThreadFactory(name, true), new AbortPolicyWithReport(name, url));
    }

}
```

- iothreads:IO线程池，接收网络读写中断，以及序列化和反序列化，不处理业务，业务线程池参见threads配置，此线程池和CPU相关，不建议配置。	
dubbo 中使用该参数初始化netty
` public static final int DEFAULT_IO_THREADS = Math.min(Runtime.getRuntime().availableProcessors() + 1, 32);`

```
   protected void doOpen() throws Throwable {
        NettyHelper.setNettyLoggerFactory();

        bootstrap = new ServerBootstrap();

        bossGroup = new NioEventLoopGroup(1, new DefaultThreadFactory("NettyServerBoss", true));
        workerGroup = new NioEventLoopGroup(getUrl().getPositiveParameter(Constants.IO_THREADS_KEY, Constants.DEFAULT_IO_THREADS),
                new DefaultThreadFactory("NettyServerWorker", true));

        final NettyServerHandler nettyServerHandler = new NettyServerHandler(getUrl(), this);
        channels = nettyServerHandler.getChannels();

        bootstrap.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .childOption(ChannelOption.TCP_NODELAY, Boolean.TRUE)
                .childOption(ChannelOption.SO_REUSEADDR, Boolean.TRUE)
                .childOption(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyServer.this);
                        ch.pipeline()//.addLast("logging",new LoggingHandler(LogLevel.INFO))//for debug
                                .addLast("decoder", adapter.getDecoder())
                                .addLast("encoder", adapter.getEncoder())
                                .addLast("handler", nettyServerHandler);
                    }
                });
        // bind
        ChannelFuture channelFuture = bootstrap.bind(getBindAddress());
        channelFuture.syncUninterruptibly();
        channel = channelFuture.channel();

    }
```

# dubbo的线程池拒绝策略
AbortPolicyWithReport:
- dump线程信息
- 打印error日志

```
public class AbortPolicyWithReport extends ThreadPoolExecutor.AbortPolicy {

    protected static final Logger logger = LoggerFactory.getLogger(AbortPolicyWithReport.class);

    private final String threadName;

    private final URL url;

    private static volatile long lastPrintTime = 0;

    private static Semaphore guard = new Semaphore(1);

    public AbortPolicyWithReport(String threadName, URL url) {
        this.threadName = threadName;
        this.url = url;
    }

    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        String msg = String.format("Thread pool is EXHAUSTED!" +
                        " Thread Name: %s, Pool Size: %d (active: %d, core: %d, max: %d, largest: %d), Task: %d (completed: %d)," +
                        " Executor status:(isShutdown:%s, isTerminated:%s, isTerminating:%s), in %s://%s:%d!",
                threadName, e.getPoolSize(), e.getActiveCount(), e.getCorePoolSize(), e.getMaximumPoolSize(), e.getLargestPoolSize(),
                e.getTaskCount(), e.getCompletedTaskCount(), e.isShutdown(), e.isTerminated(), e.isTerminating(),
                url.getProtocol(), url.getIp(), url.getPort());
        logger.warn(msg);
        dumpJStack();
        throw new RejectedExecutionException(msg);
    }

    private void dumpJStack() {
        long now = System.currentTimeMillis();

        //dump every 10 minutes
        if (now - lastPrintTime < 10 * 60 * 1000) {
            return;
        }

        if (!guard.tryAcquire()) {
            return;
        }

        Executors.newSingleThreadExecutor().execute(new Runnable() {
            @Override
            public void run() {
                String dumpPath = url.getParameter(Constants.DUMP_DIRECTORY, System.getProperty("user.home"));

                SimpleDateFormat sdf;

                String OS = System.getProperty("os.name").toLowerCase();

                // window system don't support ":" in file name
                if(OS.contains("win")){
                    sdf = new SimpleDateFormat("yyyy-MM-dd_HH-mm-ss");
                }else {
                    sdf = new SimpleDateFormat("yyyy-MM-dd_HH:mm:ss");
                }

                String dateStr = sdf.format(new Date());
                FileOutputStream jstackStream = null;
                try {
                    jstackStream = new FileOutputStream(new File(dumpPath, "Dubbo_JStack.log" + "." + dateStr));
                    JVMUtil.jstack(jstackStream);
                } catch (Throwable t) {
                    logger.error("dump jstack error", t);
                } finally {
                    guard.release();
                    if (jstackStream != null) {
                        try {
                            jstackStream.flush();
                            jstackStream.close();
                        } catch (IOException e) {
                        }
                    }
                }

                lastPrintTime = System.currentTimeMillis();
            }
        });

    }

}
```


# 模拟dubbo线程池满时的情景
## consumer程序:

``` java
public static void main(String[] args) {
        //Prevent to get IPV6 address,this way only work in debug mode
        //But you can pass use -Djava.net.preferIPv4Stack=true,then it work well whether in debug mode or not
        System.setProperty("java.net.preferIPv4Stack", "true");
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"META-INF/spring/dubbo-demo-consumer.xml"});
        context.start();
        DemoService demoService = (DemoService) context.getBean("demoService"); // get remote service proxy

        ExecutorService executorService = Executors.newFixedThreadPool(1000);
        MyTask myTask = new MyTask(demoService);
        for (int i = 0; i < 1000; i++) {
            executorService.submit(myTask);
        }

    }

    public static class MyTask implements Runnable {


        private DemoService demoService;

        public MyTask(DemoService demoService) {
            this.demoService = demoService;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName());
            String hello = demoService.sayHello("world"); // call remote method
            System.out.println(hello); // get result
        }
    }

```

## provider程序:
``` java
public class DemoServiceImpl implements DemoService {

    private static final Logger logger = LoggerFactory.getLogger(DemoService.class);

    public String sayHello(String name) {
        try {
            System.out.println(Thread.currentThread().getName());
            Thread.sleep(10000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("[" + new SimpleDateFormat("HH:mm:ss").format(new Date()) + "] Hello " + name + ", request from consumer: " + RpcContext.getContext().getRemoteAddress());
        return "Hello " + name + ", response form provider: " + RpcContext.getContext().getLocalAddress();
    }

}
```

## consumer日志:
```
[03/02/18 01:41:43:043 CST] DubboClientHandler-192.168.0.100:20881-thread-39  WARN support.DefaultFuture:  [DUBBO] The timeout response finally returned at 2018-02-03 13:41:43.668, response Response [id=1562, version=null, status=100, event=false, error=Server side(192.168.0.100,20881) threadpool is exhausted ,detail msg:Thread pool is EXHAUSTED! Thread Name: DubboServerHandler-192.168.0.100:20881, Pool Size: 200 (active: 200, core: 200, max: 200, largest: 200), Task: 201 (completed: 1), Executor status:(isShutdown:false, isTerminated:false, isTerminating:false), in dubbo://192.168.0.100:20881!, result=null], channel: /192.168.0.100:62100 -> /192.168.0.100:20881, dubbo version: 2.0.0, current host: 192.168.0.100
[03/02/18 01:41:43:043 CST] DubboClientHandler-192.168.0.100:20881-thread-30  WARN support.DefaultFuture:  [DUBBO] The timeout response finally returned at 2018-02-03 13:41:43.668, response Response [id=1561, version=null, status=100, event=false, error=Server side(192.168.0.100,20881) threadpool is exhausted ,detail msg:Thread pool is EXHAUSTED! Thread Name: DubboServerHandler-192.168.0.100:20881, Pool Size: 200 (active: 200, core: 200, max: 200, largest: 200), Task: 201 (completed: 1), Executor status:(isShutdown:false, isTerminated:false, isTerminating:false), in dubbo://192.168.0.100:20881!, result=null], channel: /192.168.0.100:62100 -> /192.168.0.100:20881, dubbo version: 2.0.0, current host: 192.168.0.100
[03/02/18 01:41:43:043 CST] DubboClientHandler-192.168.0.100:20881-thread-59  WARN support.DefaultFuture:  [DUBBO] The timeout response finally returned at 2018-02-03 13:41:43.668, response Response [id=1560, version=null, status=100, event=false, error=Server side(192.168.0.100,20881) threadpool is exhausted ,detail msg:Thread pool is EXHAUSTED! Thread Name: DubboServerHandler-192.168.0.100:20881, Pool Size: 200 (active: 200, core: 200, max: 200, largest: 200), Task: 201 (completed: 1), Executor status:(isShutdown:false, isTerminated:false, isTerminating:false), in dubbo://192.168.0.100:20881!, result=null], channel: /192.168.0.100:62100 -> /192.168.0.100:20881, dubbo version: 2.0.0, current host: 192.168.0.100
[03/02/18 01:41:43:043 CST] DubboClientHandler-192.168.0.100:20881-thread-57  WARN support.DefaultFuture:  [DUBBO] The timeout response finally returned at 2018-02-03 13:41:43.668, response Response [id=1559, version=null, status=100, event=false, error=Server side(192.168.0.100,20881) threadpool is exhausted ,detail msg:Thread pool is EXHAUSTED! Thread Name: DubboServerHandler-192.168.0.100:20881, Pool Size: 200 (active: 200, core: 200, max: 200, largest: 200), Task: 201 (completed: 1), Executor status:(isShutdown:false, isTerminated:false, isTerminating:false), in dubbo://192.168.0.100:20881!, result=null], channel: /192.168.0.100:62100 -> /192.168.0.100:20881, dubbo version: 2.0.0, current host: 192.168.0.100

```


## provider日志:
```
DubboServerHandler-192.168.0.100:20880-thread-197
DubboServerHandler-192.168.0.100:20880-thread-198
DubboServerHandler-192.168.0.100:20880-thread-199
DubboServerHandler-192.168.0.100:20880-thread-200
DubboServerHandler-192.168.0.100:20880-thread-1
[02/02/18 10:53:44:044 CST] New I/O worker #1  WARN support.AbortPolicyWithReport:  [DUBBO] Thread pool is EXHAUSTED! Thread Name: DubboServerHandler-192.168.0.100:20880, Pool Size: 200 (active: 200, core: 200, max: 200, largest: 200), Task: 201 (completed: 1), Executor status:(isShutdown:false, isTerminated:false, isTerminating:false), in dubbo://192.168.0.100:20880!, dubbo version: 2.0.0, current host: 192.168.0.100
[02/02/18 10:53:45:045 CST] New I/O worker #1  WARN support.AbortPolicyWithReport:  [DUBBO] Thread pool is EXHAUSTED! Thread Name: DubboServerHandler-192.168.0.100:20880, Pool Size: 200 (active: 200, core: 200, max: 200, largest: 200), Task: 201 (completed: 1), Executor status:(isShutdown:false, isTerminated:false, isTerminating:false), in dubbo://192.168.0.100:20880!, dubbo version: 2.0.0, current host: 192.168.0.100
[02/02/18 10:53:45:045 CST] New I/O worker #1  WARN support.AbortPolicyWithReport:  [DUBBO] Thread pool is EXHAUSTED! Thread Name: DubboServerHandler-192.168.0.100:20880, Pool Size: 200 (active: 200, core: 200, max: 200, largest: 200), Task: 201 (completed: 1), Executor status:(isShutdown:false, isTerminated:false, isTerminating:false), in dubbo://192.168.0.100:20880!, dubbo version: 2.0.0, current host: 192.168.0.100
[02/02/18 10:53:45:045 CST] New I/O worker #1  WARN support.AbortPolicyWithReport:  [DUBBO] Thread pool is EXHAUSTED! Thread Name: DubboServerHandler-192.168.0.100:20880, Pool Size: 200 (active: 200, core: 200, max: 200, largest: 200), Task: 201 (completed: 1), Executor status:(isShutdown:false, isTerminated:false, isTerminating:false), in dubbo://192.168.0.100:20880!, dubbo version: 2.0.0, current host: 192.168.0.100
[02/02/18 10:53:45:045 CST] New I/O worker #1  WARN support.AbortPolicyWithReport:  [DUBBO] Thread pool is EXHAUSTED! Thread Name: DubboServerHandler-192.168.0.100:20880, Pool Size: 200 (active: 200, core: 200, max: 200, largest: 200), Task: 201 (completed: 1), Executor status:(isShutdown:false, isTerminated:false, isTerminating:false), in dubbo://192.168.0.100:20880!, dubbo version: 2.0.0, current host: 192.168.0.100
[0
```

## dump的文件
```
-rw-r--r--    1 victor  staff  599770  2  2 13:41 Dubbo_JStack.log.2018-02-02_13:41:34
```


## 其他结论1:
奇怪的现象provider启动一次,两次启动consumer,发现第二次consumer不会报错线程池满的异常
## 其他结论2:
只有超时时间到后,才会报线程池满的异常


