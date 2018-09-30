---
title: jvm-jstack
tags:
  - jvm
  - jstack
categories:
  - jvm
abbrlink: 1838615533
date: 2018-02-02 23:16:05
---
# jstack
jstack命令的语法格式： jstack  <pid>。可以用jps查看java进程id。这里要注意的是：

1. 不同的 JAVA虚机的线程 DUMP的创建方法和文件格式是不一样的，不同的 JVM版本， dump信息也有差别。

2. 在实际运行中，往往一次 dump的信息，还不足以确认问题。建议产生三次 dump信息，如果每次 dump都指向同一个问题，我们才确定问题的典型性。

# jstack Dump 日志文件中的线程状态
## dump 文件里，值得关注的线程状态有
- 死锁， Deadlock（重点关注） 
- 执行中，Runnable   
- 等待资源， Waiting on condition（重点关注） 
- 等待获取监视器， Waiting on monitor entry（重点关注）
- 暂停，Suspended
- 对象等待中，Object.wait() 或 TIMED_WAITING
- 阻塞， Blocked（重点关注）  
- 停止，Parked

	

## Dump文件中的线程状态含义及注意事项
	

- Deadlock：死锁线程，一般指多个线程调用间，进入相互资源占用，导致一直等待无法释放的情况。
- Runnable：一般指该线程正在执行状态中，该线程占用了资源，正在处理某个请求，有可能正在传递SQL到数据库执行，有可能在对某个文件操作，有可能进行数据类型等转换。
- Waiting on condition：该状态出现在线程等待某个条件的发生。具体是什么原因，可以结合 stacktrace来分析。最常见的情况是线程在等待网络的读写，比如当网络数据没有准备好读时，线程处于这种等待状态，而一旦有数据准备好读之后，线程会重新激活，读取并处理数据。在 Java引入 NewIO之前，对于每个网络连接，都有一个对应的线程来处理网络的读写操作，即使没有可读写的数据，线程仍然阻塞在读写操作上，这样有可能造成资源浪费，而且给操作系统的线程调度也带来压力。在 NewIO里采用了新的机制，编写的服务器程序的性能和可扩展性都得到提高。

 如果发现有大量的线程都在处在 Wait on condition，从线程 stack看， 正等待网络读写，这可能是一个网络瓶颈的征兆。因为网络阻塞导致线程无法执行。一种情况是网络非常忙，几 乎消耗了所有的带宽，仍然有大量数据等待网络读 写；另一种情况也可能是网络空闲，但由于路由等问题，导致包无法正常的到达。所以要结合系统的一些性能观察工具来综合分析，比如 netstat统计单位时间的发送包的数目，如果很明显超过了所在网络带宽的限制 ; 观察 cpu的利用率，如果系统态的 CPU时间，相对于用户态的 CPU时间比例较高；如果程序运行在 Solaris 10平台上，可以用 dtrace工具看系统调用的情况，如果观察到 read/write的系统调用的次数或者运行时间遥遥领先；这些都指向由于网络带宽所限导致的网络瓶颈。另外一种出现 Wait on condition的常见情况是该线程在 sleep，等待 sleep的时间到了时候，将被唤醒。
- locked：线程阻塞，是指当前线程执行过程中，所需要的资源长时间等待却一直未能获取到，被容器的线程管理器标识为阻塞状态，可以理解为等待资源超时的线程。
- Waiting for monitor entry 和 in Object.wait()：Monitor是 Java中用以实现线程之间的互斥与协作的主要手段，它可以看成是对象或者 Class的锁。每一个对象都有，也仅有一个 monitor。


# 参数


# 使用案例
## 案例1

1.使用jps命令，获取需要调优的进程id为46924. 
2.使用top -Hp 46924命令获得最耗费资源的线程号(pid), TIME列就是各个Java线程耗费的CPU时间,这里我们选58767线程作为例子. 
3.使用printf “%x\n”，获得十六进制值。
4.使用jstack命令，它用来输出进程46924的堆栈信息，然后根据线程ID(58767)的十六进制值grep，如下：
这里就知道了最耗费时间的类是com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread-#2，Tomcat的线程池，方法是Object.wait()。