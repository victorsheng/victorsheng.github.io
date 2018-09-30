---
title: java-nio-study
abbrlink: 19816756
date: 2018-06-05 19:50:59
tags:
categories:
---
# Java NIO 由以下几个核心部分组成：

- Channels
- Buffers
- Selectors

## JAVA NIO中的一些主要Channel的实现：

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel
正如你所看到的，这些通道涵盖了UDP 和 TCP 网络IO，以及文件IO。


## buffers
ByteBuffer
CharBuffer
DoubleBuffer
FloatBuffer
IntBuffer
LongBuffer
ShortBuffer


## Selector

Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。

这是在一个单线程中使用一个Selector处理3个Channel的图示：

# buffers
## Buffer的基本用法
使用Buffer读写数据一般遵循以下四个步骤：

- 写入数据到Buffer
- 调用flip()方法(从写模式切换到读模式)
- 从Buffer中读取数据
- 调用clear()方法或者compact()方法
当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过flip()方法将Buffer从写模式切换到读模式。在读模式下，可以读取之前写入到buffer的所有数据。

一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用clear()或compact()方法。clear()方法会清空整个缓冲区。compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入
的数据将放到缓冲区未读数据的后面。

## Buffer的capacity,position和limit
缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

为了理解Buffer的工作原理，需要熟悉它的三个属性：

- capacity
- position
    - 当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.
    - 当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0. 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。
- limit
    - 在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。
    - 当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。

## Buffer的分配
要想获得一个Buffer对象首先要进行分配。 每一个Buffer类都有一个allocate方法。下面是一个分配48字节capacity的ByteBuffer的例子。


` ByteBuffer buf = ByteBuffer.allocate(48);`

## 向Buffer中写数据

写数据到Buffer有两种方式：

- 从Channel写到Buffer。
- 通过Buffer的put()方法写到Buffer里。

从Channel写到Buffer的例子

`int bytesRead = inChannel.read(buf); //read into buffer.`
通过put方法写Buffer的例子：

`buf.put(127);`

## 从Buffer中读取数据
从Buffer中读取数据有两种方式：

- 从Buffer读取数据到Channel。
- 使用get()方法从Buffer中读取数据。
从Buffer读取数据到Channel的例子：

```
//read from buffer into channel.
int bytesWritten = inChannel.write(buf);
```

使用get()方法从Buffer中读取数据的例子

` byte aByte = buf.get();` 