title: java_thread_timed_wait
abbrlink: 343265683
date: 2018-11-02 09:09:08
tags:
categories:
---




![upload successful](/images/pasted-285.png)


# RUNNABLE

处于 runnable 状态下的线程正在 Java 虚拟机中执行，但它可能正在等待来自于操作系统的其它资源，比如处理器。

进行传统上的 IO 操作时，口语上我们也会说“阻塞”，但这个“阻塞”与线程的 BLOCKED 状态是两码事！
处于RUNNABLE


# BLOCKED
- 一个处于 blocked 状态的线程正在等待一个监视器锁以进入一个同步的块或方法。
一个处于 blocked 状态的线程正在等待一个监视器锁，在其调用 Object.wait 方法之后，以再次进入一个同步的块或方法。

如果有线程长时间处于 BLOCKED 状态，要考虑是否发生了死锁（deadlock）的状况。




# WAITING

一个线程进入 WAITING 状态是因为调用了以下方法：

- 不带时限的 Object.wait 方法
- 不带时限的 Thread.join 方法
- LockSupport.park
然后会等其它线程执行一个特别的动作，比如：

一个调用了某个对象的 Object.wait 方法的线程会等待另一个线程调用此对象的 Object.notify() 或 Object.notifyAll()。
一个调用了 Thread.join 方法的线程会等待指定的线程结束。
 

# TIMED_WAITING

带指定的等待时间的等待线程所处的状态。一个线程处于这一状态是因为用一个指定的正的等待时间（为参数）调用了以下方法中的其一：

- Thread.sleep
- 带时限（timeout）的 Object.wait
- 带时限（timeout）的 Thread.join
- LockSupport.parkNanos
- LockSupport.parkUntil

这样，在通知失效的情况下，她还是有机会自我唤醒的



![upload successful](/images/pasted-284.png)