---
title: study-locksupport
date: 2018-04-02 18:39:55
tags:
  - 多线程
categories:
---
# 介绍
这个类比较简单，是一个静态类，不需要实例化直接使用，底层是通过java未开源的Unsafe直接调用底层操作系统来完成对线程的阻塞。

# 方法
park函数是将当前调用Thread阻塞，而unpark函数则是将指定线程Thread唤醒。
## park()

## unpark(Thread thread)

## parkNanos(long nanos)
阻塞当前线程，最长不超过nanos纳秒，返回条件在park()的基础上增加了超时返回。



## parkUntil(long deadline)
阻塞当前线程，知道deadline时间（deadline - 毫秒数）。

## park(Object blocker)
记录当前线程等待的对象（阻塞对象）；

阻塞当前线程；

当前线程等待对象置为null。

为什么在java6要在入参引入blocker呢？blocker的作用到底是什么？


从线程dump结果可以看出：
有blocker的可以传递给开发人员更多的现场信息，可以查看到当前线程的阻塞对象，方便定位问题。所以java6新增加带blocker入参的系列park方法，替代原有的park方法。




# 对比
与Object类的wait/notify机制相比，park/unpark有两个优点：

- 以thread为操作对象更符合阻塞线程的直观定义
- 操作更精准，可以准确地唤醒某一个线程（notify随机唤醒一个线程，notifyAll唤醒所有等待的线程），增加了灵活性。


（2）关于“许可”

在上面的文字中，我使用了阻塞和唤醒，是为了和wait/notify做对比。

其实park/unpark的设计原理核心是“许可”：park是等待一个许可，unpark是为某线程提供一个许可。
如果某线程A调用park，那么除非另外一个线程调用unpark(A)给A一个许可，否则线程A将阻塞在park操作上。

有一点比较难理解的，是unpark操作可以再park操作之前。
也就是说，先提供许可。当某线程调用park时，已经有许可了，它就消费这个许可，然后可以继续运行。这其实是必须的。考虑最简单的生产者(Producer)消费者(Consumer)模型：Consumer需要消费一个资源，于是调用park操作等待；Producer则生产资源，然后调用unpark给予Consumer使用的许可。非常有可能的一种情况是，Producer先生产，这时候Consumer可能还没有构造好（比如线程还没启动，或者还没切换到该线程）。那么等Consumer准备好要消费时，显然这时候资源已经生产好了，可以直接用，那么park操作当然可以直接运行下去。如果没有这个语义，那将非常难以操作。

但是这个“许可”是不能叠加的，“许可”是一次性的。
比如线程B连续调用了三次unpark函数，当线程A调用park函数就使用掉这个“许可”，如果线程A再次调用park，则进入等待状态。


# 测试
```
public static void main(String[] args) {

        final Thread mainThread = Thread.currentThread();

        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("3秒后解锁主线程");
                try {
                    Thread.sleep(3000);
                    LockSupport.unpark(mainThread);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        thread.start();
        LockSupport.park();

        System.out.println("Demo.main()");
    }
```

# 底层实现原理
在Linux系统下，是用的Posix线程库pthread中的mutex（互斥量），condition（条件变量）来实现的。
mutex和condition保护了一个_counter的变量，当park时，这个变量被设置为0，当unpark时，这个变量被设置为1。
