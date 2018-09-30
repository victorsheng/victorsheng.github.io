---
title: jdk-LockSupport-ThreadSleep
abbrlink: 40517
date: 2018-09-28 22:27:39
tags:
categories:
---
# LockSupport

park方法和unpark(Thread thread)都是成对出现的，同时unpark必须要在park执行之后执行，当然并不是说没有不调用unpark线程就会一直阻塞，park有一个方法，它带了时间戳（parkNanos(long nanos)：为了线程调度禁用当前线程，最多等待指定的等待时间，除非许可可用）。

park()方法的源码如下：

```
    public static void park() {
        UNSAFE.park(false, 0L);
    }
```
unpark(Thread thread)方法源码如下：
```
    public static void unpark(Thread thread) {
        if (thread != null)
            UNSAFE.unpark(thread);
    }

```
从上面可以看出，其内部的实现都是通过UNSAFE（sun.misc.Unsafe UNSAFE）来实现的