---
title: error-web-zookeeper
date: 2017-12-28 14:46:11
tags:
    - zookeeper
categories:
    - zookeeper
---
```
2017-12-26 19:42:40.907 WARN [localhost-startStop-1-SendThread(10.26.235.193:2181)][ClientCnxn.java:1162] - Session 0x15e05c1c95a0370 for server 10.26.235.193/10.26.235.193:2181, unexpected error, closing socket connection and attempting reconnect
java.lang.NoClassDefFoundError: org/apache/zookeeper/proto/SetWatches
        at org.apache.zookeeper.ClientCnxn$SendThread.primeConnection(ClientCnxn.java:926) ~[zookeeper-3.4.8.jar:3.4.8--1]
        at org.apache.zookeeper.ClientCnxnSocketNIO.doTransport(ClientCnxnSocketNIO.java:363) ~[zookeeper-3.4.8.jar:3.4.8--1]
        at org.apache.zookeeper.ClientCnxn$SendThread.run(ClientCnxn.java:1141) [zookeeper-3.4.8.jar:3.4.8--1]
```

# 问题
- 问题1:一直包上面的错误
- 问题2:26号把25号的日志给覆盖了,同时15号的错误日志也写到了9号的文件中