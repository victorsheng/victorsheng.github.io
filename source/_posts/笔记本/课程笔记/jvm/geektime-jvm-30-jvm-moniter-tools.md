---
title: geektime-jvm-30-jvm-moniter-tools
abbrlink: 268552597
date: 2018-10-09 19:11:16
tags:
categories:
---
# jps
https://docs.oracle.com/en/java/javase/11/tools/jps.html#GUID-6EB65B96-F9DD-4356-B825-6146E9EEC81E
打印所有正在运行的进程的相关信息。

在默认情况下，jps的输出信息包括 Java 进程的进程 ID 以及主类名。我们还可以通过追加参数，来打印额外的信息。例如，
- -l将打印模块名以及包名；
- -v将打印传递给 Java 虚拟机的参数（如-XX:+UnlockExperimentalVMOptions -XX:+UseZGC）；
- -m将打印传递给主类的参数。

# jstat
https://docs.oracle.com/en/java/javase/11/tools/jstat.html#GUID-5F72A7F9-5D5A-4486-8201-E1D1BA8ACCB5

可用来打印目标 Java 进程的性能数据

在这些子命令中，-class将打印类加载相关的数据，-compiler和-printcompilation将打印即时编译相关的数据。剩下的都是以-gc为前缀的子命令，它们将打印垃圾回收相关的数据。

当监控本地环境的 Java 进程时，VMID 可以简单理解为 PID。如果需要监控远程环境的 Java 进程，你可以参考 jstat 的帮助文档。

在-gc子命令的输出中，前四列分别为两个 Survivor 区的容量（Capacity）和已使用量（Utility）。我们可以看到，这两个 Survivor 区的容量相等，而且始终有一个 Survivor 区的内存使用量为 0。

```
# Usage: jstat -outputOptions [-t] [-hlines] VMID [interval [count]]
$ jstat -gc 22126 1s 4
S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT    CGC    CGCT     GCT   
17472,0 17472,0  0,0    0,0   139904,0 47146,4   349568,0   21321,0   30020,0 28001,8 4864,0 4673,4     22    0,080   3      0,270   0      0,000    0,350
17472,0 17472,0 420,6   0,0   139904,0 11178,4   349568,0   21321,0   30020,0 28090,1 4864,0 4674,2     28    0,084   3      0,270   0      0,000    0,354
17472,0 17472,0  0,0   403,9  139904,0 139538,4  349568,0   21323,4   30020,0 28137,2 4864,0 4674,2     34    0,088   4      0,359   0      0,000    0,446
17472,0 17472,0  0,0    0,0   139904,0   0,0     349568,0   21326,1   30020,0 28093,6 4864,0 4673,4     38    0,091   5  
```

## G1GC (将所有 Survivor 内存区域的总容量以及已使用量存放至 S1C 和 S1U 中，而 S0C 和 S0U 则被设置为 0)
**S0C和S0U始终为 0，而且另一个 Survivor 区的容量（S1C）可能会下降至 0。**
这是因为，当使用 G1 GC 时，Java 虚拟机不再设置 Eden 区、Survivor 区，老年代区的内存边界，而是将堆划分为若干个等长内存区域。

换句话说，逻辑上我们只有一个 Survivor 区。当需要迁移 Survivor 区中的数据时（即 Copying GC），我们只需另外申请一个或多个内存区域，作为新的 Survivor 区。
**因此，Java 虚拟机决定在使用 G1 GC 时，将所有 Survivor 内存区域的总容量以及已使用量存放至 S1C 和 S1U 中，而 S0C 和 S0U 则被设置为 0。**


当发生垃圾回收时，Java 虚拟机可能出现 Survivor 内存区域内的对象全被回收或晋升的现象。   在这种情况下，Java 虚拟机会将这块内存区域回收，并标记为可分配的状态。这样子做的结果是，堆中可能完全没有 Survivor 内存区域，因而相应的 S1C 和 S1U 将会是 0。

## 打印目标 Java 进程的启动时间
jstat还有一个非常有用的参数-t，它将在每行数据之前打印目标 Java 进程的启动时间。例如，在下面这个示例中，第一列代表该 Java 进程已经启动了 10.7 秒。

我们可以比较 Java 进程的启动时间以及总 GC 时间（GCT 列），或者两次测量的间隔时间以及总 GC 时间的增量，来得出 GC 时间占运行时间的比例。
如果该比例超过 20%，则说明目前堆的压力较大；如果该比例超过 90%，则说明堆里几乎没有可用空间，随时都可能抛出 OOM 异常。

# jmap
在这种情况下，我们便可以请jmap命令（帮助文档）出马，分析 Java 虚拟机堆中的对象。
https://docs.oracle.com/en/java/javase/11/tools/jmap.html

jmap同样包括多条子命令。
1. -clstats，该子命令将打印被加载类的信息。
2. -finalizerinfo，该子命令将打印所有待 finalize 的对象。
3. -histo，该子命令将统计各个类的实例数目以及占用内存，并按照内存使用量从多至少的顺序排列。此外，-histo:live只统计堆中的存活对象。
4. -dump，该子命令将导出 Java 虚拟机堆的快照。同样，-dump:live只保存堆中的存活对象。
我们通常会利用jmap -dump:live,format=b,file=filename.bin命令，将堆中所有存活对象导出至一个文件之中。



由于jmap将访问堆中的所有对象，为了保证在此过程中不被应用线程干扰，jmap需要借助 **安全点机制** ，让所有线程停留在不改变堆中数据的状态。
也就是说，由jmap导出的堆快照必定是安全点位置的。这可能导致基于该堆快照的分析结果存在偏差。举个例子，假设在编译生成的机器码中，某些对象的生命周期在两个安全点之间，那么:live选项将无法探知到这些对象。

另外，如果某个线程长时间无法跑到安全点，jmap将一直等下去。上一小节的jstat则不同。这是因为垃圾回收器会主动将jstat所需要的摘要数据保存至固定位置之中，而jstat只需直接读取即可。

jmap（以及接下来的jinfo、jstack和jcmd）依赖于 Java 虚拟机的Attach API，因此只能监控本地 Java 进程。

# jinfo
jinfo命令（帮助文档）可用来查看目标 Java 进程的参数，如传递给 Java 虚拟机的-X（即输出中的 jvm_args）、-XX参数（即输出中的 VM Flags），以及可在 Java 层面通过System.getProperty获取的-D参数（即输出中的 System Properties）。

https://docs.oracle.com/en/java/javase/11/tools/jinfo.html#GUID-69246B58-28C4-477D-B375-278F5F9830A5


# jstack
jstack命令（帮助文档）可以用来打印目标 Java 进程中各个线程的栈轨迹，以及这些线程所持有的锁。
jstack的其中一个应用场景便是死锁检测。这里我用jstack获取一个已经死锁了的 Java 程序的栈信息。
https://docs.oracle.com/en/java/javase/11/tools/jstack.html

我们可以看到，jstack不仅会打印线程的栈轨迹、线程状态（BLOCKED）、持有的锁（locked …）以及正在请求的锁（waiting to lock …），而且还会分析出具体的死锁。

# jcmd
https://docs.oracle.com/en/java/javase/11/tools/jcmd.html
来替代前面除了jstat之外的所有命令。具体的替换规则你可以参考下表

# 总结与实践
今天我介绍了 JDK 中用于监控及诊断的命令行工具。我们再来回顾一下。
1. jps将打印所有正在运行的 Java 进程。
2. jstat允许用户查看目标 Java 进程的类加载、即时编译以及垃圾回收相关的信息。它常用于检测垃圾回收问题以及内存泄漏问题。
3. jmap允许用户统计目标 Java 进程的堆中存放的 Java 对象，并将它们导出成二进制文件。
4. jinfo将打印目标 Java 进程的配置参数，并能够改动其中 manageabe 的参数。
5. jstack将打印目标 Java 进程中各个线程的栈轨迹、线程状态、锁状况等信息。它还将自动检测死锁。
6. jcmd则是一把瑞士军刀，可以用来实现前面除了jstat之外所有命令的功能。