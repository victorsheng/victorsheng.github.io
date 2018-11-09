---
title: mysql-login-fail
abbrlink: 493871707
date: 2018-11-07 15:45:44
tags:
categories:
---

# 问题
```

ERROR 2003 (HY000): Can't connect to MySQL server on '172.23.7.0' (60)
```

The error (2003) Can't connect to MySQL server on 'server' (10061) indicates that the network connection has been refused. You should check that there is a MySQL server running, that it has network connections enabled, and that the network port you specified is the one configured on the server.


# 运维解释

23段设备有公网访问权限，流量会走NAT的口，会被NAT成一个公网地址，源地址变了会出错， 我们的VPN的地址段在20段的这个区域，受到影响，手动指定路由后解决问题