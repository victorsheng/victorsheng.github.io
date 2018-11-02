---
title: jstack-steps
abbrlink: 4096627249
date: 2018-11-02 09:36:32
tags:
categories:
---
# 线上服务CPU100%问题快速定位
步骤一、找到最耗CPU的进程
工具：top
方法：
* 执行top -c，显示进程运行信息列表
* 键入P (大写p)，进程按照CPU使用率排序




步骤二：找到最耗CPU的线程
工具：top
方法：
* top -Hp 10765，显示一个进程的线程运行信息列表
* 键入P (大写p)，线程按照CPU使用率排序


步骤三：将线程PID转化为16进制
工具：printf
方法：printf “%x\n” 10804

步骤四：查看堆栈，找到线程在干嘛
工具：pstack/jstack/grep
方法：jstack 10765 | grep ‘0x2a34’ -C5 --color


# top相关

-p<进程号>：指定进程；
-c：显示完整的治命令；


##  top交互命令


M：根据驻留内存大小进行排序；
P：根据CPU使用百分比大小进行排序；
T：根据时间/累计时间进行排序；
w：将当前设置写入~/.toprc文件中。

## 各进程（任务）的状态监控，项目列信息说明如下：

PID — 进程id

USER — 进程所有者

PR — 进程优先级

NI — nice值。负值表示高优先级，正值表示低优先级

VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES

RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA

SHR — 共享内存大小，单位kb

S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程

%CPU — 上次更新到现在的CPU时间占用百分比

%MEM — 进程使用的物理内存百分比

TIME+ — 进程使用的CPU时间总计，单位1/100秒

COMMAND — 进程名称（命令名/命令行）

## 例子
ps ef|grep 64439
```
hadoop    64439      1  0 Oct31 ?        00:03:39

```

top -p 64439
```
[172.20.33.8:hadoop@sz-pg-dm-dev-001:/home/hadoop]$ top -p 64439
top - 09:43:34 up 320 days, 20:02,  2 users,  load average: 0.49, 0.64, 0.65
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.4 us,  0.4 sy,  0.0 ni, 98.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 13174801+total,   537404 free, 11365985+used, 17550760 buff/cache
KiB Swap: 15625212 total,  3265832 free, 12359380 used.  8042036 avail Mem

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 64439 hadoop    20   0 46.178g 2.169g   9924 S   0.0  1.7   3:39.25 java
```

# 查询线程启动时间
ps -p 64439 -o lstart