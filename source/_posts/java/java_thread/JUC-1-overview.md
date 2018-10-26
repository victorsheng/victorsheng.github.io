---
title: juc概览
tags:
  - 多线程
abbrlink: 2891225637
date: 2018-03-30 10:14:33
categories:
---


# J.U.C：

## 什么是JSR(Java Specification Requests)：
JSR，全称 Java Specification Requests， 即Java规范提案， 主要是用于向JCP(Java Community Process)提出新增标准化技术规范的正式请求。每次JAVA版本更新都会有对应的JSR更新，比如在Java 8版本中，其新特性Lambda表达式对应的是JSR 335，新的日期和时间API对应的是JSR 310。

## 什么是JSR 166：
当然，本文的关注点仅仅是JSR 166，它是一个关于Java并发编程的规范提案，在JDK中，该规范由java.util.concurrent包实现，是在JDK 5.0的时候被引入的；

- JDK6引入Deques、Navigable collections，对应的是JSR 166x，
- JDK7引入fork-join框架，用于并行执行任务，对应的是JSR 166y。

## 什么是J.U.C：
即java.util.concurrent的缩写，该包参考自EDU.oswego.cs.dl.util.concurrent，是JSR 166标准规范的一个实现；

java发布的J2SE-1.5介绍了java.util.concurrent包，是一个通过JCP(Java Community Process)和JSR创建的一个支持中间并发类的集合。这些组件都是同步组件，由抽象数据类型（ADT）类来支持内部同步状态（比如：表示一个锁是locked还是unlocked状态），并更新及监控该状态，如果其他线程修改了状态并且状态允许，则根据该状态，至少有一个方法调用线程阻塞。例如：各种形式的互斥锁、读写锁、信号量、barriers、futures、事件指示器及队列。


- 同步器(Synchronizers)
	+ 闭锁 CountDownLatch
	+ 栅栏 CyclicBarrier
	+ 信号量 Semaphore
	+ 交换器 Exchanger
  + Phaser
- 显示锁
	+ ReentrantLock
	+ ReentrantReadWriteLock
	+ StampedLock
- 原子变量类(Atomic Variables)
- 并发集合(Concurrent Collections)
- Executor框架（线程池、 Callable 、Future）
- Fork/Join并行计算框架
- BlockingQueue（阻塞队列）
- TimeUnit枚举

![upload successful](/images/pasted-113.png)

# 高级并发对象
1. Lock objects support locking idioms that simplify many concurrent applications.
2. Executors define a high-level API for launching and managing threads. Executor implementations provided by java.util.concurrent provide thread 3. pool management suitable for large-scale applications.
3. Concurrent collections make it easier to manage large collections of data, and can greatly reduce the need for synchronization.
4. Atomic variables have features that minimize synchronization and help avoid memory consistency errors.

## 1.锁对象
Lock (java.util.concurrent.locks)
ReadLockView in StampedLock (java.util.concurrent.locks)
ReentrantLock (java.util.concurrent.locks)
Segment in ConcurrentReferenceHashMap (org.springframework.util)
Segment in ConcurrentHashMap (java.util.concurrent)
WriteLock in ReentrantReadWriteLock (java.util.concurrent.locks)
WriteLockView in StampedLock (java.util.concurrent.locks)
ReadLock in ReentrantReadWriteLock (java.util.concurrent.locks)

## 2.Executors

Executor Interfaces define the three executor object types.
Thread Pools are the most common kind of executor implementation.
Fork/Join is a framework (new in JDK 7) for taking advantage of multiple processors.

### Executor

#### ExecutorService

#### ScheduledExecutorService

### Fork/Join

## 3.并发集合
BlockingQueue defines a first-in-first-out data structure that blocks or times out when you attempt to add to a full queue, or retrieve from an empty queue.
ConcurrentMap is a subinterface of java.util.Map that defines useful atomic operations. These operations remove or replace a key-value pair only if the key is present, or add a key-value pair only if the key is absent. Making these operations atomic helps avoid synchronization. The standard general-purpose implementation of ConcurrentMap is ConcurrentHashMap, which is a concurrent analog of HashMap.
ConcurrentNavigableMap is a subinterface of ConcurrentMap that supports approximate matches. The standard general-purpose implementation of ConcurrentNavigableMap is ConcurrentSkipListMap, which is a concurrent analog of TreeMap.

所有这些集合通过定义将对象添加到集合的操作与访问或删除该对象的后续操作之间的先发生关系来帮助避免内存一致性错误。

### BlockingQueue接口
BlockingQueue (java.util.concurrent)
ArrayBlockingQueue (java.util.concurrent)
DelayedWorkQueue in ScheduledThreadPoolExecutor (java.util.concurrent)
SynchronousQueue (java.util.concurrent)
BlockingDeque (java.util.concurrent)
LinkedBlockingDeque (java.util.concurrent)
DelayQueue (java.util.concurrent)
TransferQueue (java.util.concurrent)
LinkedTransferQueue (java.util.concurrent)
LinkedBlockingQueue (java.util.concurrent)
PriorityBlockingQueue (java.util.concurrent)

### ConcurrentMap接口

ConcurrentMap (java.util.concurrent)
ConcurrentReferenceHashMap (org.springframework.util)
ConcurrentNavigableMap (java.util.concurrent)
ConcurrentSkipListMap (java.util.concurrent)
SubMap in ConcurrentSkipListMap (java.util.concurrent)
ConcurrentHashMap (java.util.concurrent)


## 4.原子变量


# 思维导图
http://naotu.baidu.com/file/e6b41e23a4ec47e38b751e8dd976776c

# 参考

http://www.cnblogs.com/chenpi/p/5614290.html
http://ifeve.com/doug-lea/
http://ifeve.com/j-u-c-framework/

https://docs.oracle.com/javase/tutorial/essential/concurrency/highlevel.html
中文博客:
http://www.blogjava.net/xylz/archive/2010/07/08/325587.html

