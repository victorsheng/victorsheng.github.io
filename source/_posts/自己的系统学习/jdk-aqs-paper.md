---
title: juc的核心框架:AbstractQueuedSynchronizer
abbrlink: 481458989
date: 2018-09-30 13:08:08
tags:
categories:
---
# 参考文章
原文:
http://gee.cs.oswego.edu/dl/papers/aqs.pdf
翻译:
http://ifeve.com/aqs-1/
http://ifeve.com/aqs-2/
http://ifeve.com/aqs-3/

pdf:
http://www.sti.uniurb.it/events/sfm15mp/slides/lea.2.pdf


# 功能需求
## acquire+release
同步器一般包含两种方法，一种是acquire，另一种是release。
- acquire操作阻塞调用的线程，直到或除非同步状态允许其继续执行。
- release操作则是通过某种方式改变同步状态，使得一或多个被acquire阻塞的线程继续执行。

j.u.c包中并没有对同步器的API做一个统一的定义。
acquire和release操作的名字和形式会各有不同。例如：
- Lock.lock，
- Semaphore.acquire，
- CountDownLatch.await和
- FutureTask.get，
在这个框架里，这些方法都是acquire操作。但是，J.U.C为支持一系列常见的使用选项，在类间都有个一致约定。在有意义的情况下，每一个同步器都支持下面的操作：
- 阻塞和非阻塞（例如tryLock）同步。**trylock可以理解为非阻塞同步**
- 可选的超时设置，让调用者可以放弃等待
- 通过中断实现的任务取消，通常是分为两个版本，一个acquire可取消，而另一个不可以。???

## 是否独占
同步器的实现根据其状态是否独占而有所不同。
- 独占状态的同步器，在同一时间只有一个线程可以通过阻塞点，
- 共享状态的同步器可以同时有多个线程在执行。


## codtion
j.u.c包里还定义了Condition接口，用于支持管程形式的await/signal操作，这些操作与独占模式的Lock类有关，且Condition的实现天生就和与其关联的Lock类紧密相关。

# 性能需求

主要的性能目标是可伸缩性，即在大部分情况下，即使，或特别在同步器有竞争的情况下，稳定地保证其效率。在理想的情况下，不管有多少线程正试图通过同步点，通过同步点的开销都应该是个常量。在某一线程被允许通过同步点但还没有通过的情况下，使其耗费的总时间最少，这是主要目标之一。然而，这也必须考虑平衡各种资源，包括总CPU时间的需求，内存负载以及线程调度的开销。例如：获取自旋锁通常比阻塞锁所需的时间更短，但是通常也会浪费CPU时钟周期，并且造成内存竞争，所以使用的并不频繁。

## 关注公平还是性能
实现同步器的这些目标包含了两种不同的使用类型。大部分应用程序是最大化其总的吞吐量，容错性，并且最好保证尽量减少饥饿的情况。然而，对于那些控制资源分配的程序来说，更重要是去维持多线程读取的公平性，可以接受较差的总吞吐量。没有任何框架可以代表用户去决定应该选择哪一个方式，因此，应该提供不同的公平策略。

至少需要提供一种方式来确定有多少线程被阻塞了。

# 设计与实现

## 伪代码
获取许可
```
while (synchronization state does not allow acquire) {
	enqueue current thread if not already queued;
	possibly block current thread;
}
dequeue current thread if it was queued;
```
释放许可
```
update synchronization state;
if (state may permit a blocked thread to acquire)
	unblock one or more queued threads;
```



需要下面三个基本组件的相互协作：

- 同步状态的原子性管理；
- 线程的阻塞与解除阻塞；
- 队列的管理；


## 同步状态
创建一个框架分别实现这三个组件是有可能的。但是，这会让整个框架既难用又没效率。例如：存储在队列节点的信息必须与解除阻塞所需要的信息一致，而暴露出的方法的签名必须依赖于同步状态的特性。

同步器框架的核心决策是为这三个组件选择一个具体实现，同时在使用方式上又有大量选项可用。这里有意地限制了其适用范围，但是提供了足够的效率，使得实际上没有理由在合适的情况下不用这个框架而去重新建造一个。

基于AQS的具体实现类必须根据暴露出的状态相关的方法定义tryAcquire和tryRelease方法，以控制acquire和release操作。当同步状态满足时，tryAcquire方法必须返回true，而当新的同步状态允许后续acquire时，tryRelease方法也必须返回true。这些方法都接受一个int类型的参数用于传递想要的状态。例如：可重入锁中，当某个线程从条件等待中返回，然后重新获取锁时，为了重新建立循环计数的场景。很多同步器并不需要这样一个参数，因此忽略它即可。???

## 线程阻塞与解除

在JSR166之前，阻塞线程和解除线程阻塞都是基于Java内置管程，没有其它非基于Java内置管程的API可以用来创建同步器。唯一可以选择的是Thread.suspend和Thread.resume，但是它们都有无法解决的竞态问题，所以也没法用

j.u.c包有一个LockSuport类，这个类中包含了解决这个问题的方法。方法LockSupport.park阻塞当前线程除非/直到有个LockSupport.unpark方法被调用（unpark方法被提前调用也是可以的）。unpark的调用是没有被计数的，因此在一个park调用前多次调用unpark方法只会解除一个park操作。另外，它们作用于每个线程而不是每个同步器。一个线程在一个新的同步器上调用park操作可能会立即返回，因为在此之前可能有“剩余的”unpark操作。但是，在缺少一个unpark操作时，下一次调用park就会阻塞。虽然可以显式地消除这个状态（译者注：就是多余的unpark调用），但并不值得这样做。在需要的时候多次调用park会更高效。

> 问题: Thread.suspend和Thread.resume,Object,wait notify,LockSupport 三者之间的联系与区别

**park方法同样支持可选的相对或绝对的超时设置，以及与JVM的Thread.interrupt结合 —— 可通过中断来unpark一个线程。**

## 队列
整个框架的关键就是如何管理被阻塞的线程的队列，该队列是严格的FIFO队列，因此，框架不支持基于优先级的同步。

同步队列的最佳选择是自身没有使用底层锁来构造的非阻塞数据结构，目前，业界对此很少有争议。而其中主要有两个选择：一个是Mellor-Crummey和Scott锁（MCS锁）[9]的变体，另一个是Craig，Landin和Hagersten锁（CLH锁）[5][8][10]的变体。一直以来，CLH锁仅被用于自旋锁。但是，在这个框架中，CLH锁显然比MCS锁更合适。因为CLH锁可以更容易地去实现“取消（cancellation）”和“超时”功能，因此我们选择了CLH锁作为实现的基础。但是最终的设计已经与原来的CLH锁有较大的出入，因此下文将对此做出解释。

CLH队列实际上并不那么像队列，因为它的入队和出队操作都与它的用途（即用作锁）紧密相关。它是一个链表队列，通过两个字段head和tail来存取，这两个字段是可原子更新的，两者在初始化时都指向了一个空节点。

//TODO



## 条件队列
AQS框架提供了一个ConditionObject类，给维护独占同步的类以及实现Lock接口的类使用。一个锁对象可以关联任意数目的条件对象，可以提供典型的管程风格的await、signal和signalAll操作，包括带有超时的，以及一些检测、监控的方法。




