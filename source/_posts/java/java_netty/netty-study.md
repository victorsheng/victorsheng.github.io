---
title: netty-study
abbrlink: 958475537
date: 2018-04-17 17:57:57
tags:
categories:
---
# why

- Netty 是一个提供 asynchronous event-driven （异步事件驱动）的网络应用框架，是一个用以快速开发高性能、可扩展协议的服务器和客户端。
- 换句话说，Netty 是一个 NIO 客户端服务器框架，使用它可以快速简单地开发网络应用程序，比如服务器和客户端的协议。Netty 大大简化了网络程序的开发过程比如 TCP 和 UDP 的 socket 服务的开发。
- 现在很多互联网上面的项目比如Dubbo、Hadoop系列，MQ等都在使用netty了
- 

## 原生jdk nio
（1）NIO的类库和API繁杂，使用麻烦，你需要熟练掌握Selector、ServerSocketChannel、SocketChannel、ByteBuffer等；

（2）需要具备其他的额外技能做铺垫，例如熟悉Java多线程编程。这是因为NIO编程涉及到Reactor模式，你必须对多线程和网路编程非常熟悉，才能编写出高质量的NIO程序；

（3）可靠性能力补齐，工作量和难度都非常大。例如客户端面临断连重连、网络闪断、半包读写、失败缓存、网络拥塞和异常码流的处理等问题，NIO编程的特点是功能开发相对容易，但是可靠性能力补齐的工作量和难度都非常大；

（4）JDK NIO的BUG，例如臭名昭著的epoll bug，它会导致Selector空轮询，最终导致CPU 100%。官方声称在JDK1.6版本的update18修复了该问题，但是直到JDK1.7版本该问题仍旧存在，只不过该BUG发生概率降低了一些而已，它并没有被根本解决。该BUG以及与该BUG相关的问题单可以参见以下链接内容：

◎ http://bugs.java.com/bugdatabase/view_bug.do?bug_id=6403933

◎ http://bugs.java.com/bugdatabase/view_bug.do?bug_id=2147719


# 整体架构预览

![components](https://waylau.gitbooks.io/netty-4-user-guide/images/components.png)
- 灵拷贝
- 统一的异步io api,方便更改
- 基于拦截链模式的事件模型
- 适用快速开发的高级组件
    + Codec 框架
    + SSL / TLS 支持
    + HTTP 实现
    + WebSockets 实现
    + Google Protocol Buffer 整合

缓冲（buffer），通道（channel），事件模型（event model）

## 内部组件:

![upload successful](/images/pasted-154.png)

NioEventLoop 是 Netty 的 Reactor 线程，其角色：
1. Boss Group：作为服务端 Acceptor 线程，用于 accept 客户端链接，并转发给 WorkerGroup 中的线程。
2. Worker Group：作为 IO 线程，负责 IO 的读写，从 SocketChannel 中读取报文或向 SocketChannel 写入报文。
3. Task Queue／Delay Task Queue：作为定时任务线程，执行定时任务，例如链路空闲检测和发送心跳消息等。

## 内部流程

![upload successful](/images/pasted-155.png)

其中步骤一至步骤九是 Netty 服务端的创建时序，步骤十至步骤十三是 TCP 网关容器创建的时序。
* 步骤一：创建 ServerBootstrap 实例，ServerBootstrap 是 Netty 服务端的启动辅助类。
* 步骤二：设置并绑定 Reactor 线程池，EventLoopGroup 是 Netty 的 Reactor 线程池，EventLoop 负责所有注册到本线程的 Channel。
* 步骤三：设置并绑定服务器 Channel，Netty Server 需要创建 NioServerSocketChannel 对象。
* 步骤四：TCP 链接建立时创建 ChannelPipeline，ChannelPipeline 本质上是一个负责和执行 ChannelHandler 的职责链。
* 步骤五：添加并设置 ChannelHandler，ChannelHandler 串行的加入 ChannelPipeline 中。
* 步骤六：绑定监听端口并启动服务端，将 NioServerSocketChannel 注册到 Selector 上。
* 步骤七：Selector 轮训，由 EventLoop 负责调度和执行 Selector 轮询操作。
* 步骤八：执行网络请求事件通知，轮询准备就绪的 Channel，由 EventLoop 执行 ChannelPipeline。
* 步骤九：执行 Netty 系统和业务 ChannelHandler，依次调度并执行 ChannelPipeline 的 ChannelHandler。
* 步骤十：通过 Proxy 代理调用后端服务，ChannelRead 事件后，通过发射调度后端 Service。
* 步骤十一：创建 Session，Session 与 Connection 是相互依赖关系。
* 步骤十二：创建 Connection，Connection 保存 ChannelHandlerContext。
* 步骤十三：添加 SessionListener，SessionListener 监听 SessionCreate 和 SessionDestory 等事件。


# reactor


![upload successful](/images/pasted-156.png)
如果你对 Java 网络 IO 这个话题感兴趣的话，肯定看过 Doug Lea 的《Scalable IO in Java》，在这个 PPT 里详细介绍了如何使用 Java NIO 的技术来实现 Douglas C. Schmidt 发表的 Reactor 论文里所描述的 IO 模型。针对这个高效的通信模型，Netty 做了非常友好的支持：

* Reactor模型
    * 我们只需要在初始化 ServerBootstrap 时，提供两个不同的 EventLoopGroup 实例，就实现了 Reactor 的主从模型。我们通常把处理建连事件的线程，叫做 BossGroup，对应 ServerBootstrap 构造方法里的 parentGroup 参数，即我们常说的 Acceptor 线程；处理已创建好的 channel  相关连 IO 事件的线程，叫做 WorkerGroup，对应 ServerBootstrap 构造方法里的 childGroup 参数，即我们常说的 IO 线程。
    * 最佳实践：通常 bossGroup 只需要设置为 1 即可，因为 ServerSocketChannel 在初始化阶段，只会注册到某一个 eventLoop 上，而这个 eventLoop 只会有一个线程在运行，所以没有必要设置为多线程（什么时候需要多线程呢，可以参考 Norman Maurer 在 StackOverflow 上的这个回答）；而 IO 线程，为了充分利用 CPU，同时考虑减少线上下文切换的开销，通常设置为 CPU 核数的两倍，这也是 Netty 提供的默认值。

* 串行化设计理念
    * Netty 从 4.x 的版本之后，所推崇的设计理念是串行化处理一个 Channel 所对应的所有 IO 事件和异步任务，单线程处理来规避并发问题。Netty 里的 Channel 在创建后，会通过 EventLoopGroup 注册到某一个 EventLoop 上，之后该 Channel 所有读写事件，以及经由 ChannelPipeline 里各个 Handler 的处理，都是在这一个线程里。一个 Channel 只会注册到一个 EventLoop 上，而一个 EventLoop 可以注册多个 Channel 。所以我们在使用时，也需要尽可能避免使用带锁的实现，能无锁化就无锁。
    * 最佳实践：Channel 的实现是线程安全的，因此我们通常在运行时，会保存一个 Channel 的引用，同时为了保持 Netty 的无锁化理念，也应该尽可能避免使用带锁的实现，尤其是在 Handler 里的处理逻辑。举个例子：这里会有一个比较特殊的容易死锁的场景，比如在业务线程提交异步任务前需要先抢占某个锁，Handler 里某个异步任务的处理也需要获取同一把锁。如果某一个时刻业务线程先拿到锁 lock1，同时 Handler 里由于事件机制触发了一个异步任务 A，并在业务线程提交异步任务之前，提交到了 EventLoop 的队列里。之后，业务线程提交任务 B，等待 B 执行完成后才能释放锁 lock1；而任务 A 在队列里排在 B 之前，先被执行，执行过程需要获取锁 lock1 才能完成。这样死锁就发生了，与常见的资源竞争不同，而是任务执行权导致的死锁。要规避这类问题，最好的办法就是不要加锁；如果实在需要用锁，需要格外注意 Netty 的线程模型与任务处理机制。

* 业务处理
    * IO 密集型的轻计算业务：此时线程的上下文切换消耗，会比 IO 线程的占用消耗更为突出，所以我们通常会建议在 IO 线程来处理请求；
    * CPU 密集型的计算业务：比如需要做远程调用，操作 DB 的业务，此时 IO 线程的占用远远超过线程上下文切换的消耗，所以我们就会建议在单独的业务线程池里来处理请求，以此来释放 IO 线程的占用。该模式，也是我们蚂蚁微服务，消息通信等最常使用的模型。该模式在后面的 RPC 协议实现举例部分会详细介绍。
    * 如文章开头所描述的场景，我们需要合理设计，来将硬件的 IO 能力，CPU 计算能力与内存结合起来，发挥最佳的效果。针对不同的业务类型，我们会选择不同的处理方式
    * 最佳实践：“Never block the event loop, reduce context-swtiching”，引自Netty committer Norman Maurer，另外阿里 HSF 的作者毕玄也有类似的总结。

* 其他实践建议
    * 最小化线程池，能复用 EventLoopGroup 的地方尽量复用。比如蚂蚁因为历史原因，有过两版 RPC 协议，在两个协议升级过渡期间，我们会复用 Acceptor 线程与 IO 线程在同一个端口处理不同协议的请求；除此，针对多应用合并部署的场景，我们也会复用 IO 线程防止一个进程开过多的 IO 线程。
    * 对于无状态的 ChannelHandler ，设置成共享模式。比如我们的事件处理器，RPC 处理器都可以设置为共享，减少不同的 Channel 对应的 ChannelPipeline 里生成的对象个数。
    * 正确使用 ChannelHandlerContext 的 ctx.write() 与 ctx.channel().write() 方法。前者是从当前 Handler 的下一个 Handler 开始处理，而后者会从 tail 开始处理。大多情况下使用 ctx.write() 即可。
    * 在使用 Channel 写数据之前，建议使用 isWritable() 方法来判断一下当前 ChannelOutboundBuffer 里的写缓存水位，防止 OOM 发生。不过实践下来，正常的通信过程不太会 OOM，但当网络环境不好，同时传输报文很大时，确实会出现限流的情况。




# demo案例

## discard服务器

```
public class DiscardServerHandler extends ChannelInboundHandlerAdapter { // (1)

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) { // (2)
        // 以静默方式丢弃接收的数据
        ((ByteBuf) msg).release(); // (3)
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // (4)
        // 出现异常时关闭连接。
        cause.printStackTrace();
        ctx.close();
    }
}
```

```
public class DiscardServer {

    private int port;

    public DiscardServer(int port) {
        this.port = port;
    }

    public void run() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(); // (1)
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap(); // (2)
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class) // (3)
             .childHandler(new ChannelInitializer<SocketChannel>() { // (4)
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ch.pipeline().addLast(new DiscardServerHandler());
                 }
             })
             .option(ChannelOption.SO_BACKLOG, 128)          // (5)
             .childOption(ChannelOption.SO_KEEPALIVE, true); // (6)

            // Bind and start to accept incoming connections.
            ChannelFuture f = b.bind(port).sync(); // (7)

            // Wait until the server socket is closed.
            // In this example, this does not happen, but you can do that to gracefully
            // shut down your server.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        int port;
        if (args.length > 0) {
            port = Integer.parseInt(args[0]);
        } else {
            port = 8080;
        }
        new DiscardServer(port).run();
    }
}
```

## echo服务

```
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ctx.write(msg); // (1)
        ctx.flush(); // (2)
    }
```

## time服务器

https://waylau.gitbooks.io/netty-4-user-guide/Getting%20Started/Writing%20a%20Time%20Server.html


## 处理一个基于流的传输
基于流的传输比如 TCP/IP, 接收到数据是存在 socket 接收的 buffer 中。不幸的是，基于流的传输并不是一个数据包队列，而是一个字节队列。意味着，即使你发送了2个独立的数据包，操作系统也不会作为2个消息处理而仅仅是作为一连串的字节而言。因此这是不能保证你远程写入的数据就会准确地读取。举个例子，让我们假设操作系统的 TCP/TP 协议栈已经接收了3个数据包：


由于基于流传输的协议的这种普通的性质，在你的应用程序里读取数据的时候会有很高的可能性被分成下面的片段
因此，一个接收方不管他是客户端还是服务端，都应该把接收到的数据整理成一个或者多个更有意思并且能够让程序的业务逻辑更好理解的数据。在上面的例子中，接收到的数据应该被构造成下面的格式


# 参考
http://www.infoq.com/cn/articles/practice-of-java-nio-communication-framework
https://waylau.gitbooks.io/netty-4-user-guide
https://github.com/netty/netty/tree/4.0/example/src/main/java/io/netty/example