---
title: study-java-fire-picture
date: 2018-08-08 10:01:42
tags:
categories:
---

# 

http://colobu.com/2016/08/10/Java-Flame-Graphs/
https://www.jianshu.com/p/3d67d4eaf649



# async-profiler
# 介绍ppt
https://assets.ctfassets.net/oxjq45e8ilak/4mfbX5FJuw0A8M00UK4uKa/ce60f2cab12408e01ce927e90ebb2f7a/Andrey_Pangin__Vadim_Tsesko._The_Art_of_JVM_Profiling.pdf

## 安装
```
git clone https://github.com/jvm-profiling-tools/async-profiler
git clone https://github.com/brendangregg/FlameGraph
```

## 支持平台
Linux / x64 / x86 / ARM / AArch64
macOS / x64

## 初步使用(profiler.sh启动方式)
```
# 简单
./profiler.sh start 62676
./profiler.sh stop 62676
# 进阶
./profiler.sh -d 10 -o collapsed -f /tmp/collapsed.txt pid
```

这个命令的意思是说，采集数据的时间为10秒，并且将数据按照collapsed规范进行dump，并且dump到/tmp/collapsed.txt这个文件，过了10秒之后，工具会自动停止，并且将cpu的profile数据dump到指定的路径（按照指定的规范），可以到/tmp/collapsed.txt查看具体的文件内容，但是很大程度上是看不懂的，所以需要使用FlameGraph工具来进行加工一下，可以使用下面的命令来生成火焰图：

```
/Users/victor/code/researchProjects/FlameGraph/flamegraph.pl --colors=java /tmp/collapsed.txt > flamegraph.svg
```

## Agent方式启动(可以随jvm启动而启动)

```
$ java -agentpath:/path/to/libasyncProfiler.so=start,svg,file=profile.svg ...
```


# jvmti
jvmti全称为java virtual machine tool interface，是jvm提供的一套用于和jvm交互的api，相当于操作系统中的系统调用的概念，只是系统调用面向的是os内核，而jvmti面向的是jvm内核，jvmti使得我们有能力增强jvm，可以让运行中的jvm运行我们的代码，或者向运行中的jvm要求产生一些profile数据，来分析系统的运行状况，总之，jvmti使得开发人员可以直接与jvm交互，并且有能力做一些较为偏向jvm底层的事情。

async-profiler工具也是基于jvmti来开发的，使用的是Attech技术，也就是async-profiler会向运行中的jvm发送数据（命令）来进行一些定制操作，而jvm启动的时候会随身启动用于监控项jvm发送的信号，并且做出一些相应，并且jvm会在合适的时候开启用于服务java agent的Attach Listener线程，async-profiler就是一个java agent，它会向运行中的jvm发送一个信号，使得jvm有能力对async-profiler发送的command进行一些相应，这其中细节较多，目前不做过多分析，大概了解jvm和async-profiler之间确实存在交互就可以了，而async-profiler是作为一个java agent，并且作为一个client而存在的，而jvm中会有专门的线程来负责和java agent交互（服务）



# 参考
https://www.jianshu.com/p/9364028cca4e
https://www.jianshu.com/p/19c2f211173b