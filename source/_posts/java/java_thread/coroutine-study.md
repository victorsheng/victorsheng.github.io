title: Java里的协程
date: 2018-06-05 19:58:32
tags:
categories:
---
# 什么是协程(coroutine)
这东西其实有很多名词，比如有的人喜欢称为纤程(Fiber)，或者绿色线程(GreenThread)。其实最直观的解释可以定义为线程的线程。有点拗口，但本质上就是这样。

Java的JDK里有封装很好的ThreadPool，可以用来管理大量的线程生命周期，但是本质上还是不能很好的解决线程数量的问题，以及线程空转占用CPU资源的问题。

先阶段行业里的比较流行的解决方案之一就是单线程加上异步回调。其代表派是node.js以及Java里的新秀Vert.x。他们的核心思想是一样的，遇到需要进行I/O操作的地方，就直接让出CPU资源，然后注册一个回调函数，其他逻辑则继续往下走，I/O结束后带着结果向事件队列里插入执行结果，然后由事件调度器调度回调函数，传入结果。这时候执行的地方可能就不是你原来的代码区块了，具体表现在代码层面上，你会发现你的局部变量全部丢失，毕竟相关的栈已经被覆盖了，所以为了保存之前的栈上数据，你要么选择带着一起放入回调函数里，要么就不停的嵌套，从而引起反人类的Callback hell.

因此相关的Promise，CompletableFuture等技术都是为解决相关的问题而产生的。但是本质上还是不能解决业务逻辑的割裂。

# quasar


http://docs.paralleluniverse.co/quasar/
Quasar目前是由一家商业公司Parallel Universe控制着，且有自己的一套体系，包括Quasar-actor，Quasar-galaxy等各个模块，但是Quasar-core是开源的，此外Quasar自己也通过Fiber封装了很多的第三方库，目前全都在comsat这个项目里。随便找一个项目看看，你会发现其实通过Quasar的Fiber去封装第三方的同步库还是很简单的。

```
/**
 * http://colobu.com/2016/08/01/talk-about-quasar-again/
 */
public class Helloworld {
    @Suspendable
    static void m1() throws InterruptedException, SuspendExecution {
        String m = "m1";
        //System.out.println("m1 begin");
        m = m2();
        //System.out.println("m1 end");
        //System.out.println(m);
    }
    static String m2() throws SuspendExecution, InterruptedException {
        String m = m3();
        Strand.sleep(1000);
        return m;
    }
    //or define in META-INF/suspendables
    @Suspendable
    static String m3() {
        List l = Stream.of(1,2,3).filter(i -> i%2 == 0).collect(Collectors.toList());
        return l.toString();
    }
    static public void main(String[] args) throws ExecutionException, InterruptedException {
        int count = 10000;
        testThreadpool(count);
        testFiber(count);
    }
    static void testThreadpool(int count) throws InterruptedException {
        final CountDownLatch latch = new CountDownLatch(count);
        ExecutorService es = Executors.newFixedThreadPool(200);
        LongAdder latency = new LongAdder();
        long t = System.currentTimeMillis();
        for (int i =0; i< count; i++) {
            es.submit(() -> {
                long start = System.currentTimeMillis();
                try {
                    m1();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (SuspendExecution suspendExecution) {
                    suspendExecution.printStackTrace();
                }
                start = System.currentTimeMillis() - start;
                latency.add(start);
                latch.countDown();
            });
        }
        latch.await();
        t = System.currentTimeMillis() - t;
        long l = latency.longValue() / count;
        System.out.println("thread pool took: " + t + ", latency: " + l + " ms");
        es.shutdownNow();
    }
    static void testFiber(int count) throws InterruptedException {
        final CountDownLatch latch = new CountDownLatch(count);
        LongAdder latency = new LongAdder();
        long t = System.currentTimeMillis();
        for (int i =0; i< count; i++) {
            new Fiber<Void>("Caller", new SuspendableRunnable() {
                @Override
                public void run() throws SuspendExecution, InterruptedException {
                    long start = System.currentTimeMillis();
                    m1();
                    start = System.currentTimeMillis() - start;
                    latency.add(start);
                    latch.countDown();
                }
            }).start();
        }
        latch.await();
        t = System.currentTimeMillis() - t;
        long l = latency.longValue() / count;
        System.out.println("fiber took: " + t  + ", latency: " + l + " ms");
    }
}
```
如果使用线程，执行完1万个操作需要50秒，平均延迟为1秒左右(我们故意让延迟至少1秒)，线程池数量为200。(其实总时间50秒可以计算出来)
但是如果使用纤程，执行完1万个操作仅需要1.158秒，平均延迟时间为1秒，线程数量为CPU core数(缺省使用ForkJoinPool)。

可以看到，通过使用纤程，尽受限于系统的业务逻辑，我们没有办法提升业务的处理时间， 但是我们确可以极大的提高系统的吞吐率，如上面的简单的例子将10000个操作的处理时间从50秒提高到1秒，非凡的成就。

如果我们将方法m2中的Strand.sleep(1000);注释掉，这样这个例子中就没有什么阻塞了，我们看看在这种纯计算的情况下两者的表现：

可以看到，纤程非但没有提升性能，反而会带来性能的下降。对于这种纯计算没有阻塞的case,Quasar并不适合。

## Suspendable方法
Fiber中的run方法，如SuspendableRunnable 和 SuspendableCallable声明了SuspendExecution异常。这并不是一个真的异常，而是fiber内部工作的机制。任何运行在fiber中的可能阻塞的方法，如果声明了这个异常，就被叫做 suspendable 方法。 如果你的方法调用了一个suspendable方法，那么你的方法也是suspendable方法，所以也需要声明抛出SuspendExecution异常。

有时候不能在某个方法上声明抛出SuspendExecution异常，比如你实现某个接口，你不能更改接口的方法声明，你不得不使用其它的方法来指定suspendable方法。方法之一就是使用@Suspendable注解，在你需要指定的suspendable方法上加上这个注解就可以告诉Quasar这个方法是suspendable方法。

另一个情况就是对于第三的库，你不可能更改它们的代码，如果想指定这些库的某些方法是suspendable方法,比如java.net.URL.openStream()Ljava/io/InputStream;, 就需要另外一种解决办法，也就是在META-INF/suspendables和META-INF/suspendable-supers定义。
文件中每个方法占一行，具体(concrete)的suspendable方法应该写在META-INF/suspendables中，non-suspendable方法，但是有suspendable override的类、接口写在META-INF/suspendable-supers中(可以是具体类单不能是final, 接口和抽象类也可以)。
每一行应该是方法的签名的全称“full.class.name.methodName” 以及*通配符。
使用｀SuspendablesScanner｀可以自动增加你的方法到这些文件中，待会介绍它。

java.lang包下的方法不能标记为suspendable，其它的JDK方法则可以显示地在文件META-INF/suspendables和META-INF/suspendable-supers中标记为suspendable，并且设置环境变量co.paralleluniverse.fibers.allowJdkInstrumentation为true,但是很少这样使用。

还有一些特殊的情况也会被认为是suspendable的。

反射调用总是被看作是suspendable的。

Java 8 lambda也总是被看作suspendable的。

构造函数／类初始化器不能被标记为suspendable。

缺省情况下synchronized和blocking thread 调用不能运行在Fiber中。这是因为它们会阻塞Fiber使用的线程，导致系统处理变慢，但是如果你非要在Fiber中使用它们，可以可以将allowMonitors和allowBlocking传给instrumentation Ant task，或者将b、m传给Quasar Java agent。

# vert.x
https://vertx.io/
https://github.com/vert-x3
Vert.x诞生于2011年，当时叫node.x，不过后来因为某些原因改名位Vert.x。经过三年多的发展，现在已经到了3.2版本，社区也越来越活跃，在最新的官网Vertx.io上，作者用一句话介绍了它，JVM上的Reative开发套件。Vert.x目前是见过最功能最强大，第三方库依赖最少的Java框架，它只依赖Netty4以及Jacskon，另外如果你需要建立分布式的Vert.x则再依赖HazelCast这个分布式框架，注意Vert.x3必须基于Java8。由于基于JVM，所以Vert.x可以用其他语言来实现你的业务。默认官方维护的语言是Groovy，JavaScript以及 JRuby。

Vert.x是一个异步无阻塞的网络框架，其参照物是node.js。基本上node.js能干的事情，Vert.x都能干。Vert.x利用Netty4的EventLoop来做单线程的事件循环，所以跑在Vert.x上的业务不能做CPU密集型的运算，这样会导致整个线程被阻塞。


Vert.x的执行单元叫verticle。即程序的入口，每个语言可能实现的方式不一样，比如Java需要继承一个AbstractVerticle抽象类，而javascript则直接require(“vertx”)就可以了。

verticle分两种，一种是基于EventLoop的适合I/O密集型的，还有一种是适合CPU密集型的worker verticle。而verticle之间相互通信只能通过Eventbus，可以支持point to point 的通信，也可以支持publish & subscribe通信方式。

- 所有业务逻辑其实都会跑在Netty里的EventLoop上，而EventLoop通过循环事件队列来执行所有的业务逻辑，这样可以把一些I/O操作频繁的事件及时从CPU上剥离开来，最后通过注册一个回调Handler来处理所有的事件回调。
- worker verticle。主要是用来处理同步处理的。比如第三方框架没有异步接口，最典型就是JDBC。所以可以通过worker verticle来退化到传统的基于多线程模型的实现。这也是匹配一些原项目的手段。

http://img.ptcms.csdn.net/article/201512/21/5677b09250397.jpg


# 小结
如果你希望你的代码能够跑在Fiber里面，需要一个很大的前提条件，那就是你所有的库，必须是异步无阻塞的，也就说必须类似于node.js上的库，所有的逻辑都是异步回调，而自Java里基本上所有的库都是同步阻塞的，很少见到异步无阻塞的。而且得益于J2EE，以及Java上的三大框架(SSH)洗脑，大部分Java程序员都已经习惯了基于线程，线性的完成一个业务逻辑，很难让他们接受一种将逻辑割裂的异步编程模型。

- 线程:Fiber
- 网络I/O问题:Netty
- 文件I/O:JDK7提供的NIO2就可以满足，通过AsynchronousFileChannel即可。
- 剩下的就是如何将他们封装成更友好的API了。目前能达到生产级别的这种异步工具库，JVM上只有Vert.x3，封装了Netty4，封装了AsynchronousFileChannel，而且Vert.x官方也出了一个相对应的封装了Quasar的库vertx-sync。


异步无阻塞的编码方式其实有很多种实现，比如node.js的提倡的Promise，对应到Java8的就是CompletableFuture。

事件响应式也算是一个比较流行的做法，比如ReactiveX系列，RxJava，Rxjs，RxSwift，等。我个人觉得RxJava是一个非常好的函数式响应实现(JDK9会有对应的JDK实现)，但是我们不能要求所有的程序员一眼就提炼出业务里的functor，monad(这些能力需要长期浸淫在函数式编程思想里)，反而RxJava特别适合用在前端与用户交互的部分，因为用户的点击滑动行为是一个个真实的事件流，这也是为什么RxJava在Android端非常火的原因，而后端基本上都是通过Rest请求过来，每一个请求其实已经限定了业务范围，不会再有复杂的事件逻辑，所以基本上RxJava在Vert.x这端只是做了一堆的flatmap，再加上微服务化，所有的业务逻辑都已经做了最小的边界，所以顺序的同步的编码方式更适合写业务逻辑的后端程序员。



# 参考
https://segmentfault.com/a/1190000005342905
http://colobu.com/2016/07/14/Java-Fiber-Quasar/
http://colobu.com/2016/08/01/talk-about-quasar-again/