---
title: geektime-jvm-10-object-monery
abbrlink: 2827992282
date: 2018-08-16 18:36:25
tags:
categories:
---
# Java中创建对象的方式

1. new -通过调用构造器来初始化实例字段
2. 反射-通过调用构造器来初始化实例字段
3. bject.clone-通过直接复制已有的数据，来初始化新建对象的实例字段
4. 反序列化-通过直接复制已有的数据，来初始化新建对象的实例字段
5. Unsafe.allocateInstance-没有初始化对象的实例字段

# 构造方法
以 new 语句为例，它编译而成的字节码将包含用来请求内存的 new 指令，以及用来调用构造器的 invokespecial 指令。
```
// Foo foo = new Foo(); 编译而成的字节码
  0 new Foo
  3 dup
  4 invokespecial Foo()
  7 astore_1
```

子类的构造器需要调用父类的构造器。如果父类存在无参数构造器的话，该调用可以是隐式的，也就是说 Java 编译器会自动添加对父类构造器的调用。但是，如果父类没有无参数构造器，那么子类的构造器则需要显式地调用父类带参数的构造器。


总而言之，当我们调用一个构造器时，它将优先调用父类的构造器，直至 Object 类。这些构造器的调用者皆为同一对象，也就是通过 new 指令新建而来的对象。

通过 new 指令新建出来的对象，它的内存其实涵盖了所有父类中的实例字段。也就是说，虽然子类无法访问父类的私有实例字段，或者子类的实例字段隐藏了父类的同名实例字段，但是子类的实例还是会为这些父类实例字段分配内存的。


# 压缩指针

对象头:
- 标记字段:存储 Java 虚拟机有关该对象的运行数据，如哈希码、GC 信息以及锁信息
- 类型指针:指向该对象的类

在 64 位的 Java 虚拟机中，对象头的标记字段占 64 位，而类型指针又占了 64 位。也就是说，每一个 Java 对象在内存中的额外开销就是 16 个字节。以 Integer 类为例，它仅有一个 int 类型的私有字段，占 4 个字节。因此，每一个 Integer 对象的额外内存开销至少是 400%。这也是为什么 Java 要引入基本类型的原因之一。


为了尽量较少对象的内存使用量，64 位 Java 虚拟机引入了压缩指针 [1] 的概念（对应虚拟机选项 -XX:+UseCompressedOops，默认开启），将堆中原本 64 位的 Java 对象指针压缩成 32 位的。

这样一来，对象头中的类型指针也会被压缩成 32 位，使得对象头的大小从 16 字节降至 12 字节。当然，压缩指针不仅可以作用于对象头的类型指针，还可以作用于引用类型的字段，以及引用类型数组。

## 原理
那么压缩指针是什么原理呢？
打个比方，路上停着的全是房车，而且每辆房车恰好占据两个停车位。现在，我们按照顺序给它们编号。也就是说，停在 0 号和 1 号停车位上的叫 0 号车，停在 2 号和 3 号停车位上的叫 1 号车，依次类推。
原本的内存寻址用的是车位号。比如说我有一个值为 6 的指针，代表第 6 个车位，那么沿着这个指针可以找到 3 号车。现在我们规定指针里存的值是车号，比如 3 指代 3 号车。当需要查找 3 号车时，我便可以将该指针的值乘以 2，再沿着 6 号车位找到 3 号车。
这样一来，32 位压缩指针最多可以标记 2 的 32 次方辆车，对应着 2 的 33 次方个车位。当然，房车也有大小之分。大房车占据的车位可能是三个甚至是更多。不过这并不会影响我们的寻址算法：**我们只需跳过部分车号，便可以保持原本车号 *2 的寻址系统。**

## 内存补齐
上述模型有一个前提，你应该已经想到了，就是每辆车都从偶数号车位停起。这个概念我们称之为内存对齐（对应虚拟机选项 -XX:ObjectAlignmentInBytes，默认值为 8）。

默认情况下，Java 虚拟机堆中对象的起始地址需要对齐至 8 的倍数。如果一个对象用不到 8N 个字节，那么空白的那部分空间就浪费掉了。这些浪费掉的空间我们称之为对象间的填充（padding）。

## 寻址空间
在默认情况下，Java 虚拟机中的 32 位压缩指针可以寻址到 2 的 35 次方个字节，也就是 32GB 的地址空间（超过 32GB 则会关闭压缩指针）。
在对压缩指针解引用时，我们需要将其左移 3 位，再加上一个固定偏移量，便可以得到能够寻址 32GB 地址空间的伪 64 位指针了。

此外，我们可以通过配置刚刚提到的内存对齐选项（-XX:ObjectAlignmentInBytes）来进一步提升寻址范围。但是，这同时也可能增加对象间填充，导致压缩指针没有达到原本节省空间的效果。

## 内存对齐(无法关闭)
当然，就算是关闭了压缩指针，Java 虚拟机还是会进行内存对齐。此外，内存对齐不仅存在于对象与对象之间，也存在于对象中的字段之间。比如说，Java 虚拟机要求 long 字段、double 字段，以及非压缩指针状态下的引用字段地址为 8 的倍数。

字段内存对齐的其中一个原因，是让字段只出现在同一 CPU 的缓存行中。如果字段不是对齐的，那么就有可能出现跨缓存行的字段。也就是说，该字段的读取可能需要替换两个缓存行，而该字段的存储也会同时污染两个缓存行。这两种情况对程序的执行效率而言都是不利的。

# 字段重排列



# 总结
- 常见的 new 语句会被编译为 new 指令，以及对构造器的调用
- 每个类的构造器皆会直接或者间接调用父类的构造器，并且在同一个实例中初始化相应的字段。
- Java 虚拟机引入了压缩指针的概念，将原本的 64 位指针压缩成 32 位。压缩指针要求 Java 虚拟机堆中对象的起始地址要对齐至 8 的倍数。Java 虚拟机还会对每个类的字段进行重排列，使得字段也能够内存对齐。