title: how-ok-httpclient-release-tcp-connection
date: 2018-07-06 06:55:04
tags:
    - httpclient
categories:
---
# httpclient未close时,进行关闭
测试了main方法结束退出,和被kill -9退出的两种方式,都是发送rst

![upload successful](/images/pasted-208.png)

# okhttpclient异常关闭
![](/images/pasted-207.png)

# 问题不知道在哪进行设置的