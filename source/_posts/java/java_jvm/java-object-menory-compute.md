title: java对象字节码占用计算
date: 2018-03-27 22:22:30
tags:
categories:
---
# 对象内存结构

![upload successful](/images/pasted-104.png)
各种类型分别占多少个字节(bytes)：

计算方式：

① 对象头（Header）
② 实例数据（Instance Data）
③ 对齐填充 （Padding）

## 对象头（Header）
HotSpot 虚拟机的对象头包括两部分信息：Mark Word 和 类型指针；如果是数组对象的话，还有第三部分（option）信息：数组长度
### Mark Word
这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit 和64bit。Mark Word用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等。
对象头信息是与对象定义的数据无关的额外存储成本，考虑到虚拟机的空间效率，Mark Word被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的信息，它会根据对象的状态复用自己的存储空间。

### 类型指针（Class Pointer）
是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

### 数组长度（Length）[option]
如果对象时一个Java数组，那在对象头中还必须有一块用于记录数组长度的数据。因为虚拟机可以通过普通Java对象的元数据信息确定Java对象的大小，但是从数组的元数据中无法确定数组的大小。



### 对象头占用空间
#### 普通对象占用内存情况：

| | 32 位系统 | 64 位系统\(+UseCompressedOops\) | 64 位系统\(-UseCompressedOops\) |
| :--- | :--- | :--- | :--- |
| Mark Word | 4 bytes | 8 bytes | 8 bytes |
| Class Pointer | 4 bytes | 4 bytes | 8 bytes |
| 对象头 | 8 bytes | 12 bytes | 16 bytes |

#### 数组对象占用内存情况：

| | 32 位系统 | 64 位系统\(+UseCompressedOops\) | 64 位系统\(-UseCompressedOops\) |
| :--- | :--- | :--- | :--- |
| Mark Word | 4 bytes | 8 bytes | 8 bytes |
| Class Pointer | 4 bytes | 4 bytes | 8 bytes |
| Length | 4 bytes | 4 bytes | 4 bytes |
| 对象头 | 12 bytes | 16 bytes | 24 bytes??(有的博客说是20) |

多4字节的Length占用

## 实例数据（Instance Data）
实例数据部分是对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容，无论是从父类继承下来的，还是在子类中定义的，都需要记录起来。

- HotSpot虚拟机默认的分配策略为longs/doubles、ints、shorts/chars、bytes/booleans、oops（Ordinary Object Pointers）的顺序
- 从分配策略中可以看出，相同宽度的字段总是被分配到一起。
- 在满足这个前提条件的情况下，在父类中定义的变量会出现在子类之前。
- 如果CompactFields参数值为true（默认为true），那子类之中较窄的变量也可能会插入到父类变量的空隙之中。

### 基本数据类型占用
|Type | 32 位系统 | 64 位系统\(+UseCompressedOops\) | 64 位系统\(-UseCompressedOops\) |32位栈内存|64位栈内存|
| :--- | :--- | :--- | :--- | :--- | :--- |
| double | 8 bytes | 8 bytes |  | 8 |8|
| long | 8 bytes | 8 bytes |  | 8 |8|
| float | 4 bytes | 4 bytes |  | 4 |8|
| int | 4 bytes | 4 bytes |  | 4| 8 |
| char | 2 bytes | 2 bytes |  | 4| 8 |
| short | 2 bytes | 2 bytes |  | 4| 8 |
| byte | 1 bytes | 1 bytes |  | 4| 8 |
| boolean | 1 bytes | 1 bytes |  |4| 8 |
| oops\(ordinary object pointers\)引用类型 | 4 bytes | 4 bytes | 8 bytes |

### 对象引用（reference）类型
在64位机器上，关闭指针压缩时占用8bytes， 开启时占用4bytes。

## 对齐填充 （Padding）

对齐填充并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说就是对象的大小必须是8字节的整数倍。
例如，一个包含两个属性的对象：int和byte，并不是占用17bytes(12+4+1)，而是占用24bytes（对17bytes进行8字节对齐）

## 注意：

JVM默认是开启压缩参数的-XX:+UseCompressedOops
计算对象本身占用大小和对象总空间占用大小的区别：
1. 本身占用大小，对象中除了基本类型之外，其他类型都按照引用来计算，不要计算引用中对象的大小。
2. 总空间占用大小，要计算对象中每一个对象的大小，引用中的对象也要计算，再累加获得总空间。

# 包装类型
包装类（Boolean/Byte/Short/Character/Integer/Long/Double/Float）占用内存的大小等于对象头大小加上底层基础数据类型的大小。

| | +useCompressedOops | -useCompressedOops |
| :--- | :--- | :--- |
| Byte, Boolean | 16 bytes(12头+1实例+3对齐) | 24 bytes(16头+1实例+7对齐) |
| Short, Character | 16 bytes(12头+2实例+2对齐) | 24 bytes(16头+2实例+6对齐) |
| Integer, Float | 16 bytes(12头+4实例+0对齐) | 24 bytes(16头+4实例+4对齐) |
| Long, Double | 24 bytes(12头+8实例+0对齐) | 24 bytes(16头+8实例+0对齐) |

# 数组
64位机器上，数组对象的对象头占用24 bytes，启用压缩后占用16字节。比普通对象占用内存多是因为需要额外的空间存储数组的长度。基础数据类型数组占用的空间包括数组对象头以及基础数据类型数据占用的内存空间。由于对象数组中存放的是对象的引用，所以对象数组本身的大小=数组对象头+length * 引用指针大小，总大小为对象数组本身大小+存放的数据的大小之和。
## int[10]:   
开启压缩：16 + 10 * 4 = 56 bytes；
关闭压缩：24 + 10 * 4 = 64bytes。
## new Integer[3]:
 关闭压缩：
    Integer数组本身：24(header) + 3 * 8(Integer reference) = 48 bytes;
    总共：48 + 3 * 24(Integer) = 120 bytes。

 开启压缩：
    Integer数组本身：16(header) + 3 * 4(Integer reference) = 28(padding) -> 32 (bytes)
    总共：32 + 3 * 16(Integer) = 80 (bytes)

# String
```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
}
```


在关闭指针压缩时，一个String本身需要 16(Header) + 8(char[] reference) + 4(int) = 32 bytes。
56 + length * 2 bytes (8字节对齐)。
- 一个空字符串("")的大小应为：56 + 0 * 2 bytes = 56 bytes。
- 字符串"abc"的大小应为：56 + 3 * 2 = 62(8字节对齐)->64 (bytes)
- 字符串"abcde"的大小应为：56 + 5 * 2 = 66->72 (bytes)
- 字符串"abcde"在开启指针压缩时的大小为：
    String本身：12(Header) + 4(char[] reference) + 4(int hash) = 20(padding) -> 24 (bytes);   
   存储数据：16(char[] header) + 5*2 = 26(padding) -> 32 (bytes)         
   总共：24 + 32 = 56 (bytes)  


# 对象计算
## 案例1
```
class TestObject {
    private int i;
    private double d;
    private char[] c;

    public TestObject() {
        this.i = 1;
        this.d = 1.0;
        this.c = new char[]{'a', 'b', 'c'};
    }
}
```

TestObject对象有四个属性，分别为int, double, Byte, char[]类型。在打开指针压缩(-XX:+UseCompressedOops)的情况下，在64位机器上，TestObject占用的内存大小应为：
本身占用大小:12(Header) + 4(int) + 8(double) + 4(reference) = 28 (bytes)，加上8字节对齐，最终的大小应为32 bytes。


## class内的成员变量:一个int字段
对象头(12) + 实例数据(4) + 对齐填充(0) = 16

## 两个long字段
对象占用内存大小：对象头(12) + 实例数据(16) + 对齐填充(4) = 32

这里请注意，padding的填充不是在最后面的，即，不是在实例数据分配完后填充了4个字
节。而是在对象头分配完后填充了4个字节。这从属性'a'字段的偏移量为16，也能够说明填充的部分是对象头后的4个字节空间。

这是为什么了？
是这样的，在64位系统中，CPU一次读操作可读取64bit（8 bytes）的数据。如果，你在对象头分配后就进行属性 long a字段的分配，也就是说从偏移量为12的地方分配8个字节，这将导致读取属性long a时需要执行两次读数据操作。因为第一次读取到的数据中前4字节是对象头的内存，后4字节是属性long a的高4位（Java 是大端模式），低4位的数据则需要通过第二次读取操作获得。

## class内的成员变量:一个int字段+一个long字段
对象占用内存大小：对象头(12) + 实例数据(12) + 对齐填充(0) = 24

在前面的理论中，我们说过基本变量类型在内存中的存放顺序是从大到小的（顺序：longs/doubles、ints、shorts/chars、bytes/booleans）。所以，按理来说，属性int b应该被分配到了属性long a的后面。但是，从属性位置偏移量的结果来看，我们却发现属性int b被分配到了属性long a的前面，这是为什么了？是这样的，因为JVM启用了'CompactFields'选项，该选项运行分配的非静态（non-static）字段被插入到前面字段的空隙中，以提供内存的利用率。从前面的实例中，我们已经知道，对象头占用了12个字节，并且再次之后分配的long类型字段不会紧跟在对象头后面分配，而是在新一个8字节偏移量位置处开始分配，因此对象头和属性long a直接存在了4字节的空隙，而这个4字节空隙的大小符合（即，大小足以用于）属性int b的内存分配。所以，属性int b就被插入到了对象头与属性long a之间了。

## ## class内的成员变量:一个int字段+一个long字段+string字段
实例数据：long (8 bytes) + int (4 bytes) + oops (4 bytes)
对象占用内存大小：对象头(12) + 实例数据(16) + 对齐填充(4) = 32

# 参数


-XX:+UseCompressedOops（JDK 8下默认为启用）
## CompactFields
Allocate nonstatic fields in gaps between previous fields
分配一个非static的字段在前面字段缝隙中。这么做也是为了提高内存的利用率。
-XX:+CompactFields（JDK 8下默认为启用）

## FieldsAllocationStyle
-XX:FieldsAllocationStyle=1 （JDK 8下默认值为‘1’）
0 - type based with oops first, 1 - with oops last, 2 - oops in super and sub classes are together
实例对象中有效信息的存储顺序：
0：先放入oops（普通对象引用指针），然后在放入基本变量类型（顺序：longs/doubles、ints、shorts/chars、bytes/booleans）
1：先放入基本变量类型（顺序：longs/doubles、ints、shorts/chars、bytes/booleans），然后放入oops（普通对象引用指针）
2：oops和基本变量类型交叉存储

# 工具下载

https://www.javamex.com/classmexer/

-javaagent:/Users/victor/jar/classmexer-0_03/classmexer.jar



# Shallow Size和 Retained Size的区别
## Shallow Size自身占用
对象自身占用的内存大小，不包括它引用的对象。
针对非数组类型的对象，它的大小就是对象与它所有的成员变量大小的总和。当然这里面还会包括一些java语言特性的数据存储单元。
针对数组类型的对象，它的大小是数组元素对象的大小总和。

## Retained Size"总占用"
Retained Size=当前对象大小+当前对象可直接或间接引用到的对象的大小总和。(间接引用的含义：A->B->C, C就是间接引用)
换句话说，Retained Size就是当前对象被GC后，从Heap上总共能释放掉的内存。
不过，释放的时候还要排除被GC Roots直接或间接引用的对象。他们暂时不会被被当做Garbage。

参考https://blog.csdn.net/e5945/article/details/7708253

# 参考资料
https://www.jianshu.com/p/91e398d5d17c
https://www.jianshu.com/p/12a3c97dc2b7
https://segmentfault.com/a/1190000006933272