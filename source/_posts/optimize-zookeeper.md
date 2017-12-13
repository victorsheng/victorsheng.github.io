---
title: optimize_zookeeper
date: 2017-12-12 14:51:30
tags:
categories:
---
# 背景
156机器是本地开发环境上的机器
跑了如下服务:
mysql
zookeeper
dubbokeeper

# 现象
从ssh登陆服务器就比较卡
top命令后,负载一直很高
<img src="http://pic.victor123.cn/17-12-12/5122868.jpg">

# 排查
iotop命令看到zookeeper的io比较高
http://pic.victor123.cn/17-12-12/59575921.jpg

# 解决
- https://www.cnblogs.com/jxwch/p/6526271.html
- 打开这两个参数
    - autopurge.snapRetainCount这个参数指定了需要保留的文件数目，默认保留3个；
    - autopurge.purgeInterval这个参数指定了清理频率，单位是小时，需要填写一个1或者更大的数据，默认0表示不开启自动清理功能。
- 清空历史数据
<img src="http://pic.victor123.cn/17-12-12/1580197.jpg">

# 效果
<img src="http://pic.victor123.cn/17-12-12/29291132.jpg">

# 相关知识
http://www.linuxidc.com/Linux/2016-03/129509.htm
dataDir用于存储Log（事务日志）与Snapshot（快照）数据