---
title: servlet-new-version
abbrlink: 739616101
date: 2018-11-20 17:45:15
tags:
categories:
---
# 介绍
Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。

http://www.runoob.com/wp-content/uploads/2014/07/servlet-arch.jpg

# 3.0版本
https://www.ibm.com/developerworks/cn/java/j-lo-servlet30/index.html

Servlet 3.0 作为 Java EE 6 规范体系中一员，随着 Java EE 6 规范一起发布。该版本在前一版本（Servlet 2.5）的基础上提供了若干新特性用于简化 Web 应用的开发和部署。其中有几项特性的引入让开发者感到非常兴奋，同时也获得了 Java 社区的一片赞誉之声：

- 异步处理支持：有了该特性，Servlet 线程不再需要一直阻塞，直到业务处理完毕才能再输出响应，最后才结束该 Servlet 线程。在接收到请求之后，Servlet 线程可以将耗时的操作委派给另一个线程来完成，自己在不生成响应的情况下返回至容器。针对业务处理较耗时的情况，这将大大减少服务器资源的占用，并且提高并发处理速度。
- 新增的注解支持：该版本新增了若干注解，用于简化 Servlet、过滤器（Filter）和监听器（Listener）的声明，这使得 web.xml 部署描述文件从该版本开始不再是必选的了。
- 可插性支持：熟悉 Struts2 的开发者一定会对其通过插件的方式与包括 Spring 在内的各种常用框架的整合特性记忆犹新。将相应的插件封装成 JAR 包并放在类路径下，Struts2 运行时便能自动加载这些插件。现在 Servlet 3.0 提供了类似的特性，开发者可以通过插件的方式很方便的扩充已有 Web 应用的功能，而不需要修改原有的应用。



# 4.0版本
https://www.ibm.com/developerworks/cn/java/j-javaee8-servlet4/index.html

Servlet 4.0 是 API 的最新版本，也是 Java EE 8 规范的核心更新。正如您将在本教程中了解到的，Servlet 4.0 已为 HTTP/2 准备就绪，完全包含服务器推送，同时还将其扩展到基于 servlet 的技术

Servlet 4.0 的主要新功能为服务器推送和全新 API，该 API 可在运行时发现 servlet 的 URL 映射。

服务器推送是最直观的 HTTP/2 强化功能，通过 PushBuilder 接口在 servlet 中公开。服务器推送功能还在 JavaServer Faces API 中实现，并在 RenderResponsePhase 生命周期内调用，以便 JSF 页面可以利用其增强性能。


# 补充知识:jsp与jsf
- JSP是服务器端展现动态网页内容的一种技术
- JSF是一个基于Java的web应用框架，用来简化基于web的用户接口开发集成。它是一种标准化的展现技术。JSF说明中定义一系列标准的UI组件，并且为开发提供了API。