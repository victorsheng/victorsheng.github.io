---
title: netty-study
date: 2018-04-17 17:57:57
tags:
categories:
---
# why
现在很多互联网上面的项目比如Dubbo、Hadoop系列，MQ等都在使用netty了

## 原生jdk nio
（1）NIO的类库和API繁杂，使用麻烦，你需要熟练掌握Selector、ServerSocketChannel、SocketChannel、ByteBuffer等；

（2）需要具备其他的额外技能做铺垫，例如熟悉Java多线程编程。这是因为NIO编程涉及到Reactor模式，你必须对多线程和网路编程非常熟悉，才能编写出高质量的NIO程序；

（3）可靠性能力补齐，工作量和难度都非常大。例如客户端面临断连重连、网络闪断、半包读写、失败缓存、网络拥塞和异常码流的处理等问题，NIO编程的特点是功能开发相对容易，但是可靠性能力补齐的工作量和难度都非常大；

（4）JDK NIO的BUG，例如臭名昭著的epoll bug，它会导致Selector空轮询，最终导致CPU 100%。官方声称在JDK1.6版本的update18修复了该问题，但是直到JDK1.7版本该问题仍旧存在，只不过该BUG发生概率降低了一些而已，它并没有被根本解决。该BUG以及与该BUG相关的问题单可以参见以下链接内容：

◎ http://bugs.java.com/bugdatabase/view_bug.do?bug_id=6403933

◎ http://bugs.java.com/bugdatabase/view_bug.do?bug_id=2147719




# 参考
http://www.infoq.com/cn/articles/practice-of-java-nio-communication-framework