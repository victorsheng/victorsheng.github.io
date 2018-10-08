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

# 参考

http://www.cnblogs.com/chenpi/p/5614290.html
http://ifeve.com/doug-lea/
http://ifeve.com/j-u-c-framework/


