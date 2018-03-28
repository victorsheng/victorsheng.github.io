title: java对象字节码占用计算
date: 2018-03-27 22:22:30
tags:
categories:
---
# 参考
https://www.jianshu.com/p/91e398d5d17c
https://www.jianshu.com/p/12a3c97dc2b7


# 对象内存结构

![upload successful](/images/pasted-104.png)
各种类型分别占多少个字节(bytes)：

计算方式：

① 对象头（Header）
② 实例数据（Instance Data）
③ 对齐填充 （Padding）

## 对象头（Header）
HotSpot 虚拟机的对象头包括两部分信息：Mark Word 和 类型指针；如果是数组对象的话，还有第三部分（option）信息：数组长度
## Mark Word
这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit 和64bit。Mark Word用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等。
对象头信息是与对象定义的数据无关的额外存储成本，考虑到虚拟机的空间效率，Mark Word被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的信息，它会根据对象的状态复用自己的存储空间。

### 类型指针（Class Pointer）
是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

### 数组长度（Length）[option]
如果对象时一个Java数组，那在对象头中还必须有一块用于记录数组长度的数据。因为虚拟机可以通过普通Java对象的元数据信息确定Java对象的大小，但是从数组的元数据中无法确定数组的大小。

## 实例数据（Instance Data）
实例数据部分是对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容，无论是从父类继承下来的，还是在子类中定义的，都需要记录起来。

HotSpot虚拟机默认的分配策略为longs/doubles、ints、shorts/chars、bytes/booleans、oops（Ordinary Object Pointers），从分配策略中可以看出，相同宽度的字段总是被分配到一起。在满足这个前提条件的情况下，在父类中定义的变量会出现在子类之前。如果CompactFields参数值为true（默认为true），那子类之中较窄的变量也可能会插入到父类变量的空隙之中。

## 对齐填充 （Padding）

对齐填充并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说就是对象的大小必须是8字节的整数倍。

## 占用空间
普通对象占用内存情况：

| | 32 位系统 | 64 位系统\(+UseCompressedOops\) | 64 位系统\(-UseCompressedOops\) |
| :--- | :--- | :--- | :--- |
| Mark Word | 4 bytes | 8 bytes | 8 bytes |
| Class Pointer | 4 bytes | 4 bytes | 8 bytes |
| 对象头 | 8 bytes | 12 bytes | 16 bytes |

数组对象占用内存情况：

| | 32 位系统 | 64 位系统\(+UseCompressedOops\) | 64 位系统\(-UseCompressedOops\) |
| :--- | :--- | :--- | :--- |
| Mark Word | 4 bytes | 8 bytes | 8 bytes |
| Class Pointer | 4 bytes | 4 bytes | 8 bytes |
| Length | 4 bytes | 4 bytes | 4 bytes |
| 对象头 | 12 bytes | 16 bytes | 20 bytes |


# 基本类型



|Type | 32 位系统 | 64 位系统\(+UseCompressedOops\) | 64 位系统\(-UseCompressedOops\) |
| :--- | :--- | :--- | :--- |
| double | 8 bytes | 8 bytes | 8 bytes |
| long | 8 bytes | 8 bytes | 8 bytes |
| float | 4 bytes | 4 bytes | 4 bytes |
| int | 4 bytes | 4 bytes | 4 bytes |
| char | 2 bytes | 2 bytes | 2 bytes |
| short | 2 bytes | 2 bytes | 2 bytes |
| byte | 1 bytes | 1 bytes | 1 bytes |
| boolean | 1 bytes | 1 bytes | 1 bytes |
| oops\(ordinary object pointers\) | 4 bytes | 4 bytes | 8 bytes |


注意：JVM默认是开启压缩参数的-XX:+UseCompressedOops

计算对象本身占用大小和对象总空间占用大小的区别：

1.本身占用大小，对象中除了基本类型之外，其他类型都按照引用来计算，不要计算引用中对象的大小。

2.总空间占用大小，要计算对象中每一个对象的大小，引用中的对象也要计算，再累加获得总空间。

# 包装类型
| | +useCompressedOops | -useCompressedOops |
| :--- | :--- | :--- |
| Byte, Boolean | 16 bytes | 24 bytes |
| Short, Character | 16 bytes | 24 bytes |
| Integer, Float | 16 bytes | 24 bytes |
| Long, Double | 24 bytes | 24 bytes |