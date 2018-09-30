---
title: 极客时间reactor-preactor学习笔记
abbrlink: 3218294535
date: 2018-06-21 10:29:59
tags:
categories:
---

# 单服务器高性能架构模式：Reactor 和 Proactor

## Reactor
- 当一个连接一个进程时，进程可以采用“read -> 业务处理 -> write”的处理流程，如果当前连接没有数据可以读，则进程就阻塞在 read 操作上。
- 如果一个进程处理多个连接，进程阻塞在某个连接的 read 操作上，此时即使其他连接有数据可读，进程也无法去处理，很显然这样是无法做到高性能的。
- 解决这个问题的最简单的方式是将 read 操作改为非阻塞，然后进程不断地轮询多个连接。这种方式能够解决阻塞的问题，但解决的方式并不优雅。首先，轮询是要消耗 CPU 的；其次，如果一个进程处理几千上万的连接，则轮询的效率是很低的。
- 为了能够更好地解决上述问题，只有当连接上有数据的时候进程才去处理，这就是 I/O 多路复用技术的来源。
- I/O 多路复用技术归纳起来有两个关键实现点：
	- 当多条连接共用一个阻塞对象后，进程只需要在一个阻塞对象上等待，而无须再轮询所有连接，常见的实现方式有 select、epoll、kqueue 等。
	- 当某条连接有新的数据可以处理时，操作系统会通知进程，进程从阻塞状态返回，开始进行业务处理。

Reactor 模式也叫 Dispatcher 模式（在很多开源的系统里面会看到这个名称的类，其实就是实现 Reactor 模式的），更加贴近模式本身的含义，即 I/O 多路复用统一监听事件，收到事件后分配（Dispatch）给某个进程。

Reactor 模式的核心组成部分包括 Reactor 和处理资源池（进程池或线程池），其中 Reactor 负责监听和分配事件，处理资源池负责处理事件。初看 Reactor 的实现是比较简单的，但实际上结合不同的业务场景，Reactor 模式的具体实现方案灵活多变，主要体现在：

- Reactor 的数量可以变化：可以是一个 Reactor，也可以是多个 Reactor。
- 资源池的数量可以变化：以进程为例，可以是单个进程，也可以是多个进程（线程类似）。

### 单Reactor单进程

![upload successful](/images/pasted-182.png)

单 Reactor 单进程的方案在实践中应用场景不多，只适用于业务处理非常快速的场景，目前比较著名的开源软件中使用单 Reactor 单进程的是 Redis。
需要注意的是，C 语言编写系统的一般使用单 Reactor 单进程，因为没有必要在进程中再创建线程；而 Java 语言编写的一般使用单 Reactor 单线程，因为 Java 虚拟机是一个进程，虚拟机中有很多线程，业务线程只是其中的一个线程而已。

### 单 Reactor 多线程
![upload successful](/images/pasted-181.png)
介绍:
- 主线程中，Reactor 对象通过 select 监控连接事件，收到事件后通过 dispatch 进行分发。
- 如果是连接建立的事件，则由 Acceptor 处理，Acceptor 通过 accept 接受连接，并创建一个 Handler 来处理连接后续的各种事件。
- 如果不是连接建立事件，则 Reactor 会调用连接对应的 Handler（第 2 步中创建的 Handler）来进行响应。
- Handler 只负责响应事件，不进行业务处理；Handler 通过 read 读取到数据后，会发给 Processor 进行业务处理。
- Processor 会在独立的子线程中完成真正的业务处理，然后将响应结果发给主进程的 Handler 处理；Handler 收到响应后通过 send 将响应结果返回给 client。

单 Reator 多线程方案能够充分利用多核多 CPU 的处理能力，但同时也存在下面的问题：

- 多线程数据共享和访问比较复杂。例如，子线程完成业务处理后，要把结果传递给主线程的 Reactor 进行发送，这里涉及共享数据的互斥和保护机制。以 Java 的 NIO 为例，Selector 是线程安全的，但是通过 Selector.selectKeys() 返回的键的集合是非线程安全的，对 selected keys 的处理必须单线程处理或者采取同步措施进行保护。
- Reactor 承担所有事件的监听和响应，只在主线程中运行，瞬间高并发时会成为性能瓶颈。
    
### 多 Reactor 多进程 / 线程

![upload successful](/images/pasted-183.png)
目前著名的开源系统 Nginx 采用的是多 Reactor 多进程，
采用多 Reactor 多线程的实现有 Memcache 和 Netty。

我多说一句，Nginx 采用的是多 Reactor 多进程的模式，但方案与标准的多 Reactor 多进程有差异。具体差异表现为主进程中仅仅创建了监听端口，并没有创建 mainReactor 来“accept”连接，而是由子进程的 Reactor 来“accept”连接，通过锁来控制一次只有一个子进程进行“accept”，子进程“accept”新连接后就放到自己的 Reactor 进行处理，不会再分配给其他子进程，更多细节请查阅相关资料或阅读 Nginx 源码。


# 补充:select、epoll、kqueue
select和iocp分别对应第3种与第5种模型，那么epoll与kqueue呢？其实也于select属于同一种模型，只是更高级一些，可以看作有了第4种模型的某些特性，如callback机制。

那么，为什么epoll,kqueue比select高级？ 

答案是，他们无轮询。因为他们用callback取代了。想想看，当套接字比较多的时候，每次select()都要通过遍历FD_SETSIZE个Socket来完成调度,不管哪个Socket是活跃的,都遍历一遍。这会浪费很多CPU时间。如果能给套接字注册某个回调函数，当他们活跃时，自动完成相关操作，那就避免了轮询，这正是epoll与kqueue做的。


（1）select，poll实现需要自己不断轮询所有fd集合，直到设备就绪，期间可能要睡眠和唤醒多次交替。而epoll其实也需要调用epoll_wait不断轮询就绪链表，期间也可能多次睡眠和唤醒交替，但是它是设备就绪时，调用回调函数，把就绪fd放入就绪链表中，并唤醒在epoll_wait中进入睡眠的进程。虽然都要睡眠和交替，但是select和poll在“醒着”的时候要遍历整个fd集合，而epoll在“醒着”的时候只要判断一下就绪链表是否为空就行了，这节省了大量的CPU时间。这就是回调机制带来的性能提升。

（2）select，poll每次调用都要把fd集合从用户态往内核态拷贝一次，并且要把current往设备等待队列中挂一次，而epoll只要一次拷贝，而且把current往等待队列上挂也只挂一次（在epoll_wait的开始，注意这里的等待队列并不是设备等待队列，只是一个epoll内部定义的等待队列）。这也能节省不少的开销。


# 补充:Reactor与Proactor
Reactor与Proactor能不能这样打个比方：
1、假如我们去饭店点餐，饭店人很多，如果我们付了钱后站在收银台等着饭端上来我们才离开，这就成了同步阻塞了。
2、如果我们付了钱后给你一个号就可以离开，饭好了老板会叫号，你过来取。这就是Reactor模型。
3、如果我们付了钱后给我一个号就可以坐到坐位上该干啥干啥，饭好了老板会把饭端上来送给你。这就是Proactor模型了。


# 补充:IO操作分两个阶段
 1、等待数据准备好(读到内核缓存) 2、将数据从内核读到用户空间(进程空间) 
 一般来说1花费的时间远远大于2。
- 1上阻塞2上也阻塞的是同步阻塞IO 
- 1上非阻塞2阻塞的是同步非阻塞IO，这讲说的Reactor就是这种模型 
- 1上非阻塞2上非阻塞是异步非阻塞IO，这讲说的Proactor模型就是这种模型

![upload successful](/images/pasted-179.png)

![upload successful](/images/pasted-180.png)