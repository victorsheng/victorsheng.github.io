title: java-study-BlockingQueue
tags:
  - 数据结构
categories:
  - java
date: 2018-01-29 23:47:00
---
# BlockingQueue
- 阻塞队列
- 先进先出
- 有边界:当队列满时,插入的元素的线程会等待队列可有
- 等待队列为非空
- 生产者消费者
< !-- more -->

阻塞队列的主要功能并不是在于提升程序高并发时队列的性能，而是在于简化多线程间的数据共享（用于多线程间的数据传递）。

# 五种类
## ArrayBlockingQueue

* 有界。ArrayBlockingQueue是基于数组实现的有界阻塞队列；其能容纳的元素数量固定，一旦创建，就不能再增加其容量；
* FIFO。队列的获取操作作用于队列头部，添加操作作用于队列尾部，满足FIFO特性；
* 自动阻塞唤醒。试图向已满队列放入元素将导致操作阻塞，试图从空队列中获取元素同样将导致操作阻塞等待。

## LinkedBlockingQueue

* 有界，也可认为是无界。LinkedBlockingQueue是基于链表实现的阻塞队列；如果在构造函数不指定容量，则默认为一个类似无限大小的容量，大小为Integer.MAX_VALUE，在此种情况下，如果生产者速度远远大于消费者速度，由于容量很大，那么系统内存有可能会被消耗殆尽；如果在构造函数指定了容量大小，则队列容纳的元素数量固定。
* FIFO。队列的获取操作作用于队列头部，添加操作作用于队列尾部，满足FIFO特性
* 自动阻塞唤醒。put操作试图向已满队列放入元素将导致操作阻塞，直到其他线程从队列中获取元素，队列出现空闲后再唤醒操作；take操作试图从空队列中获取元素同样将导致操作阻塞等待，直到其他线程从队列中添加元素，队列中存在元素后再唤醒操作。
* 锁分离机制。对添加数据的操作与获取数据的操作分别采用了独立的、不同的锁来控制数据同步（ArrayBlockingQueue采用了同一把锁），即在高并发的情况下生产者和消费者可以并行的操作队列中的数据，进而提高整个队列的并发性能。

## PriorityBlockingQueue

PriorityBlockingQueue又称为优先级阻塞队列。其特性如下：
* 无界。由于容量不存在限制(最大为Integer.MAX_VALUE - 8，可基本认为是无限制)，队列就不会阻塞生产者，因为只要能生产数据，就可以把数据放入队列中；如果生产者速度远远大于消费者速度，由于容量无限，那么系统内存有可能会被消耗殆尽。
* 元素必须实现Comparable接口。在实现的CompareTo方法中，如果返回值为负数，则表明当前对象this的优先级越高；返回值为正数，则表明当前对象this的优先级越低。
* 实现了有序列表。优先级的判断通过构造函数传入的Compator对象实现队列元素的自定义排序；如果Compator为空，则按对象的比较方法进行排序。队列中元素的优先级依次降低，优先级最高的排在队首。默认情况下元素采取自然顺序排序，也可以通过比较器comparator来指定元素的排序规则。
* 仅阻塞消费者。当队列中无数据时，会阻塞消费者。

## DelayQueue

DelayQueue又称为延时队列。一个使用优先级队列（PriorityQueue）实现的无界阻塞队列。其特性如下：
* 队列容量无界；不能存放null元素。
* 元素必须同时实现Delayed接口与Comparable接口。元素所属类中必须实现public int compareTo(To)和long getDelay(TimeUnit unit)方法。
其中，Delayed接口继承了Comparable接口，因此必须实现compareTo方法；在public int compareTo(To)方法中，如果当前对象的延迟值小于参数对象的值，将返回一个小于0的值；如果当前对象的延迟值大于参数对象的值，将返回一个大于0的值；如果两者的延迟值相等则返回0。该队列的头部是延迟期满后保存时间最长的 Delayed 元素（也就是保证先过期的元素排在头部）。如果延迟都还没有期满，则队列头部不会被返回，即队列的 poll 方法将返回null。
* long getDelay(TimeUnit unit)方法中，是返回到激活日期的剩余时间；当返回值为0或者负数时，表面该元素已经可以被获取了；单位由单位参数指定。TimeUnit类是一个由下列常量组成的枚举类型：DAYS、HOURS、MICROSECONDS、MILLISECONDS、MINUTES、NANOSECONDS和SECONDES。
* 只有在延迟期满时才能从队列中提取元素；Delayed接口使得对象成为了延迟对象，它使得队列中的元素对象具有了延迟期。由于DelayQueue是一个没有大小限制的队列，因此往队列中插入数据的操作永远不会被阻塞，而只有获取数据的操作才会被阻塞。
适用场景：
* 缓存系统的设计：可以用DelayQueue保存缓存元素的有效期，使用一个线程循环查询DelayQueue，一旦能从队列中获取元素时候，表示缓存有效期到了。
* 定时任务调度：使用DelayQueue保存当天将会执行的任务和执行时间。一旦从队列中获取到任务就开始执行，比如TimerQueue就是用DelayQueue实现的。

## SynchronousQueue

SynchronousQueue又称为同步队列。一个不存储元素的阻塞队列。其特性如下：
* 是一个无缓冲的等待队列。队列本身不存储任何元素，每一个put操作必须等待一个take操作，否则不能继续添加元素。
* 支持公平与非公平两种操作模式。如果采用公平模式：SynchronousQueue会采用公平锁，并配合一个FIFO队列来阻塞多余的生产者和消费者，体现了“先来先服务，后来则排队”的公平策略；如果采用非公平模式（SynchronousQueue默认选项），则配合一个LIFO队列来管理多余的生产者和消费者，体现了“无视当前排队，即来即抢占服务”的不公平策略，但是，在该种模式下，如果生产者和消费者的处理速度有差距，可能有某些生产者或者消费者永远都得不到处理（因为一直被抢占，一直被欺负）。
* 与其他阻塞队列不同，其他阻塞队列维护的是一组元素，SynchronousQueue则是维护了两组线程（一组生产者线程，一组消费者线程），实现两者之间的数据传递。举个列子，有A、B两位同事，A需要将一些资料交给B。如果是A把文件直接交到B手中，那么就是SynchronousQueue实现模式；如果A通过发邮件的形式把资料给B，那么就是其他阻塞队列的实现模式。
* SyncchronousQueue吞吐量高于LinkedBlockingQueue和ArrayBlockingQueue。

适用场景：
创建线程池。

## LinkedTransferQueue

LinkedTransferQueue又称为链表传输队列。一个由链表结构组成的无界阻塞队列。其特性如下：
* 一个无界队列。
* 具备FIFO特性。
* 特别的阻塞：LinkedTransferQueue实现了TransferQueue接口，TransferQueue又继承了BlockingQueue；这两个接口的区别在于：BlockingQueue：当生产者向满队列添加元素时则会被阻塞；当消费者从空队列中获取元素时则会被阻塞；TransferQueue：相比于BlockingQueue，生产者会一直阻塞直到所添加到队列的元素被某一个消费者所消费（不只是把元素添加到队列中），其接口中的transfer方法实现了该功能。顾名思义，就是发生在元素从一个线程transfer到另一个线程的过程中，它有效地实现了元素在线程之间的传递（以建立Java内存模型中的happens-before关系的方式）。

适用场景：
生产者-消费者高并发的业务场景。