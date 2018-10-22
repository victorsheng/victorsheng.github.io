---
title: network-model
abbrlink: 1600481103
date: 2018-10-12 18:32:22
tags:
categories:
---
#OSI Model from wiki
https://en.wikipedia.org/wiki/OSI_model

	Layer 1: Physical Layer
	Layer 2: Data Link Layer
	Layer 3: Network Layer
	Layer 4: Transport Layer
	Layer 5: Session Layer
	Layer 6: Presentation Layer
	Layer 7: Application Layer

![https://s3.amazonaws.com/edu.cohesive.net/OSI-layer-model.gif](https://s3.amazonaws.com/edu.cohesive.net/OSI-layer-model.gif)

# 网络分层
作者：车小胖
链接：https://www.zhihu.com/question/58798786/answer/159305039
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

以TCP/IP协议五层模型的计算机网络率先出现，硬件接口实现“物理层”、“数据链路层”，操作系统内核里的TCP/IP协议栈实现“网络层”、“传输层”，所有依赖于TCP/IP协议栈的应用程序实现广泛意义上的“应用层”，这个广泛意义的“应用层”既实现了会话ID、心跳keepalive，又实现了诸如文字、图片、音频、视频、文件的不同表示。后来才有以TCP/IP为现实素材的OSI七层参考模型，希望将“会话层”、“表示层”从广泛意义上的应用层里独立出来，这样可以让应用程序瘦身，核心目标是：让千千万万不同应用程序共享“会话层”、“表示层”软件代码。很遗憾的是，迄今为止这个美好的愿望依然没有实现，究其根本原因是，不同的应用程序有大同小异的会话、表示需求，这些代码不完全能够抽象到独立的会话层、表示层，或者说，现有的应用层已经比较完美实现了会话层、表示层，对于七层模型需求没有动力。安全层但是，有心栽花花不成，无心插柳柳成荫，一个提供安全加密服务层出现了，很多人都使用过，只是一直没有人想去划分层次结构，它的名字叫SSL/TLS，有了它的加入，我们可以将TCP/IP五层结构看成六层：应用层安全层（TLS）TCP/UDPIP层数据链路层物理层有了安全层提供的服务，位于应用层的HTTP/SMTP/FTP，都可以在其名字后加一个S（Security），比如HTTPS，其实这个世界压根不存在HTTPS协议，只有HTTP协议，加上S的后缀只是告诉大家HTTP使用的是六层结构，有了SSL/TLS的安全保护。