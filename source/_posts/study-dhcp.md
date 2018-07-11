title: study-dhcp
date: 2018-06-07 17:31:21
tags:
    - 网络协议
categories:
---
# 背景
最近总是遇到公司WiFi ip地址冲突的问题,于是百度了下

# 概念
DHCP协议全称为“动态主机设置协议”（Dynamic Host Configuration Protocol）。通常来说，普通电脑中都内置有DHCP客户端模块。电脑接上网络后，DHCP客户端发现新连通的网络，会在该网络上找DHCP服务器。DHCP服务器将给电脑提供合理的网络配置，并把设置信息传回本机。所谓的DHCP服务器，其实就是一些运行有DHCP服务器端软件的特殊电脑。他们像等候在网络上的服务员，为新来的顾客排忧解难。本机和DHCP服务器之间的通信，都是通过DHCP协议进行的。

动态管理100个ip地址,并设置有效期,如果过期没有续租,则收回ip

# 通信过程
DHCP协议的底层是UDP协议。我们知道，网络上的点对点沟通需要有IP地址。但新接入网络的客户机正是想通过DHCP通信来获得IP地址。这简直成了“鸡生蛋、蛋生鸡”的死胡同。幸好，除了点对点通信，UDP协议还允许广播通信。把UDP数据包发送到网络的广播地址，网络上的每个设备都能收到。因此，DHCP通信主要靠这种广播的形式进行。
DHCP通信分为四步：

- Discovery：客户机发广播，搜寻DHCP服务器。
- Offer：DHCP服务器发出邀请，提供一个可用的IP地址。
- Request：客户机正式请求使用该IP地址。
- Acknowledge：DHCP服务器确认，并提供其他配置参数。


![upload successful](/images/pasted-174.png)


# 参考
https://wizardforcel.gitbooks.io/protocol-forest-vamei/content/content/16.html