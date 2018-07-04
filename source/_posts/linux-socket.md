title: linux-socket挥手
date: 2018-06-27 05:58:36
tags:
categories:
---
# Socket是什么

socket起源于Unix，而Unix/Linux基本哲学之一就是“一切皆文件”，都可以用“打开open –> 读写write/read –> 关闭close”模式来操作。Socket就是该模式的一个实现，        socket即是一种特殊的文件，一些socket函数就是对其进行的操作（读/写IO、打开、关闭）.     说白了Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

*注意：其实socket也没有层的概念，它只是一个facade设计模式的应用，让编程变的更简单。是一个软件抽象层。在网络编程中，我们大量用的都是通过socket实现的。*



# 四次挥手流程
连接正常关闭时双方会发送FIN，经历4次挥手过程

java中，调用Socket#close()可以关闭Socket，该方法类似Unix网络编程中的close方法，将Socket的 读写 都关闭，已经排队等待发送的数据会被尝试发送，最后（默认）发送FIN。考虑一个典型的网络事务，A向B发送数据，A发送完毕后close()，FIN发送出去；B一直read直到返回了-1，也通过close()发送FIN，4次挥手，连接关闭，一切都很和谐。


## 服务端发起流程

### 1.服务端发起关闭

![upload successful](/images/pasted-194.png)

### 2.中间状态
客户端socket为close_waite状态
服务端socket释放

![upload successful](/images/pasted-196.png)

### 3.客户端关闭

![upload successful](/images/pasted-195.png)

## 客户端发起流程

### 1.客户端发起关闭

![upload successful](/images/pasted-197.png)
### 2.中间状态
服务端socket为close_waite状态
客户端socket释放

![upload successful](/images/pasted-198.png)

### 3.服务端关闭

![upload successful](/images/pasted-199.png)

# RST关闭流程(一次通讯)
通过RST包异常退出，此时会丢弃缓冲区内的数据，也不会对RST响应ACK。

- Socket#setSoLinger(true,0)，则close时会发送RST
- 如果主动关闭方缓冲区还有数据没有被应用层消费掉，close会发送RST并忽略这些数据
- A向B发送数据，B已经通过close()方法关闭了Socket，虽然TCP规定半关闭状态下B仍然可以接收数据，但close动作关闭了该socket上的任何数据操作，如果此时A继续write，B将返回RST，A的该次write无法立即通知应用层（因为write仅把数据写入发送缓冲区），只会把状态保存在tcp协议栈内，下次write时才会抛出SocketException。

无论是客户端还是服务端的socket均可以设置soLinger参数
```
socket.setSoLinger(true, 0);
```

## 服务端发起
### 1.服务端发起关闭
![upload successful](/images/pasted-200.png)

### 2. 客户端状态

![upload successful](/images/pasted-201.png)

## 客户端发起
### 客户端发起关闭

![upload successful](/images/pasted-202.png)

### 2. 服务端状态

![upload successful](/images/pasted-203.png)

# 对已关闭socket读写会产生的异常
## 主动关闭方
close()后，无论是发送FIN/RST关闭的，之后再读写均会抛java.net.SocketException:socket is closed.

## 被动关闭方
### 被FIN关闭
写（即向”已被对方关闭的Socket”写） 
如上所说，第一次write得到RST响应但不抛异常，第二次write抛异常，ubuntu下是broken pipe (断开的管道)，win7下是Software caused connection abort: socket write error

读 – 始终返回 -1

### 被RST关闭
读写都会抛出异常：connection reset (by peer)

### 重点在于：
connection reset：另一端用RST主动关闭连接

broken pipe / Software caused connection abort: socket write error ： 对方已调用Socket#close()关闭连接，己方还在写数据

java中网络编程时很大一部分代码在做各种fail时的处理，了解各种异常发生时背后的逻辑才能正确地处理之。以上列举的只是连接关闭的异常，还有其他各种异常没有提及，以后有机会再补上。

# 小结
之所以由本篇文章中,是因为在工作中用到了okhttpClient,其理念就是尽可能复用之前socket连接,并通过后台线程回收空闲的socket连接,然而对于客户端意外关闭,似乎没有处理,经过试验发现,其socket的关闭方式正是rst方式.

## 问题
通过socket代码打断点,未发现okhttpClient 进行设置了setSoLinger


# 参考
http://novoland.github.io/%E7%BD%91%E7%BB%9C/2014/07/26/RST%E5%8F%8Ajava%20socket%E5%85%B3%E9%97%AD%E5%90%8E%E8%AF%BB%E5%86%99%E7%9A%84%E5%90%84%E7%A7%8D%E5%BC%82%E5%B8%B8.html
