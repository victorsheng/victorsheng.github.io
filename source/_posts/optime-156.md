title: '156服务器优化2:定位io高的原因mysql'
tags:
  - io高
  - 优化
  - mysql
categories: []
date: 2017-12-13 13:39:00
---
#重新定位问题
-%wa指CPU等待磁盘写入完成的时间

首先看下%wa的解释：Percentage of time that the CPU or CPUs were idle during which the system had an outstanding disk I/O request.

#iostat

#pidstat 2 10 -d
发现mysql的读写量非常高
<img src="http://pic.victor123.cn/17-12-13/27569242.jpg">