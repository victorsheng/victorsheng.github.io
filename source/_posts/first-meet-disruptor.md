---
title: first-meet-Disruptor
abbrlink: 2068507538
date: 2018-11-23 17:11:19
tags:
categories:
---
# Disruptor描述
Disruptor是一个开源的并发框架，使用事件源驱动方式，能够在无锁的情况下实现网络的Queue并发操作

# Disruptor为什么那么快

Disruptor不用锁，取而代之的是在需要确保操作线程安全的地方，我们使用CAS（Compare And Swap/Set）操作，它是一个CPU级别的指令（CPU去更新一个值，如果想改得值不再是原来的值，操作就失败，因为有其他操作先改变了这个值）,CAS操作比锁消耗的资源少的多，因为它们之间在CPU上操作
神奇的缓存行填充，通过增加补全来确保Ring Buffer的序列号不会和其他东西同时存在于一个缓存行中，因此没有伪共享和非预期的竞争
使用内存屏障，它是CPU指令，它允许你对数据什么时候对其他进程可见性作出假设，在java里，你使用volatile关键字来实现内存屏障，使用volatile意味着你不用被迫选择加锁，并且还能让你获得性能上的提升，但是，你需要对你的设计进行一些更细致的思考，特别是你对volatile字段的使用有多频繁，以及对它们的读写有多频繁。

# Disruptor工作原理

RingBuffer到底是什么？
1.它是一个首尾相接的环，你可以把它用做在不同上下文间传递数据的buffer，它拥有一个序号，这个序号指向数组中下一个可用的元素，随着你不停的填充这个buffer，这个序号会一直增长。
2.环形buffer是没有尾指针，我们只维护了一个指向下一个可用位置的序号，而且与常用的队列的区别是，我们不用删除buffer中的数据，也就是说这些数据一直存放在buffer中，直到新的数据覆盖它们。
3.ringbuffer采用的数据结构是数组，所以要比链表快

# 问题
会不会很多没有消费,直接把老的覆盖掉了??


# 参考
http://ifeve.com/dissecting-disruptor-whats-so-special/
http://ifeve.com/locks-are-bad/
http://ifeve.com/disruptor-cacheline-padding/
http://ifeve.com/disruptor-memory-barrier/