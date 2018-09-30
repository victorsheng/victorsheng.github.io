---
title: jdk-conditon
abbrlink: 17553
date: 2018-09-29 21:59:53
tags:
categories:
---
作为一个示例，假定有一个绑定的缓冲区，它支持 put 和 take 方法。如果试图在空的缓冲区上执行 take 操作，则在某一个项变得可用之前，线程将一直阻塞；如果试图在满的缓冲区上执行 put 操作，则在有空间变得可用之前，线程将一直阻塞。我们喜欢在单独的等待 set 中保存 put 线程和 take 线程，这样就可以在缓冲区中的项或空间变得可用时利用最佳规划，一次只通知一个线程。可以使用两个 Condition 实例来做到这一点。
```
class BoundedBuffer {
   final Lock lock = new ReentrantLock();
   final Condition notFull  = lock.newCondition(); 
   final Condition notEmpty = lock.newCondition(); 

   final Object[] items = new Object[100];
   int putptr, takeptr, count;

   public void put(Object x) throws InterruptedException {
     lock.lock();
     try {
       while (count == items.length) 
         notFull.await();
       items[putptr] = x; 
       if (++putptr == items.length) putptr = 0;
       ++count;
       notEmpty.signal();
     } finally {
       lock.unlock();
     }
   }

   public Object take() throws InterruptedException {
     lock.lock();
     try {
       while (count == 0) 
         notEmpty.await();
       Object x = items[takeptr]; 
       if (++takeptr == items.length) takeptr = 0;
       --count;
       notFull.signal();
       return x;
     } finally {
       lock.unlock();
     }
   } 
 }
 
```

（ ArrayBlockingQueue 类提供了这项功能，因此没有理由去实现这个示例类。）

# 方法
方法签名 | 介绍
------- | -------
 void wait() | 造成当前线程在接到信号或被中断之前一直处于等待状态。
 boolean await(long time, TimeUnit unit) | 造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。
 long awaitNanos(long nanosTimeout) | 造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。
 void awaitUninterruptibly() | 造成当前线程在接到信号之前一直处于等待状态。
 boolean awaitUntil(Date deadline) | 造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。
 void signal() | 唤醒一个等待线程。
 void signalAll() | 唤醒所有等待线程。
