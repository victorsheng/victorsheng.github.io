---
title: btrace-study
tags:
categories:
---
# 概念:

- Probe Point: 
    - “location” or “event” at which a set of tracing statements are executed. Probe point is “place” or “event” of interest where we want to execute some tracing statements.（探测点，就是我们想要执行一些追踪语句的地方或事件）

- Trace Actions or Actions: 
    - Trace statements that are executed whenever a probe “fires”.（当探测触发时执行追踪语句）

- Action Methods: 
    - BTrace trace statements that are executed when a probe fires are defined inside a static method a class. Such methods are called “action” methods.（当在类的静态方法中定义了探测触发时执行的BTrace跟踪语句。这种方法被称为“操作”方法。）

<!-- more -->


# 学习的博客:
http://mgoann.iteye.com/blog/1409685
http://calvin1978.blogcn.com/articles/btrace1.html 
http://codepub.cn/2017/09/22/btrace-uses-tutorials/


# byteman
局部变量
http://codepub.cn/2017/09/22/byteman-uses-tutorials/


# github地址:
https://github.com/btraceio/btrace

# 可运行包 下载地址
https://github.com/btraceio/btrace/releases/tag/v1.3.10

