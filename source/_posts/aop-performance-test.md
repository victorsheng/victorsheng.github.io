---
title: aop性能测试
date: 2018-07-29 15:10:46
tags:
categories:
---
# 候选方案
动态代理工具比较成熟的产品有： 
- JDK自带的，
- ASM，
- CGLIB(基于ASM包装)，
- JAVAASSIST， 
使用的版本分别为： 
JDK-1.6.0_18-b07, ASM-3.3, CGLIB-2.2, JAVAASSIST-3.11.0.GA 


# 评论
cglib
Dispatcher
LazyLoader

导出字节码javap -c 类名

# 参考文章
http://javatar.iteye.com/blog/814426

https://github.com/neoremind/dynamic-proxy