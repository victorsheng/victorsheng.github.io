---
title: content-type
date: 2018-05-17 10:31:00
tags:
categories:
---

# Media Type/Content Type/MIME Type
Media Type/Content Type/MIME Type 这三者都是指同一个东西，即互联网媒体类型，用来标识互联网中传输的内容类型。浏览器会识别各种Content Type，显示不同的内容。如音频、图片、视频、文档。 
一个媒体配型至少包括两部分：一个类型，和一个子类型，还可以包括一个或多个可选的参数。 比如说 
text/html; charset=UTF-8 中text是类型；html是子类型；charset=UTF-8是可选参数。 

# Accept与Content-Type
1.Accept属于请求头， Content-Type属于实体头。 
Http报头分为通用报头，请求报头，响应报头和实体报头。 
请求方的http报头结构：通用报头|请求报头|实体报头 
响应方的http报头结构：通用报头|响应报头|实体报头

2.Accept代表发送端（客户端）希望接受的数据类型。 
比如：Accept：text/xml; 
代表客户端希望接受的数据类型是xml类型

Content-Type代表发送端（客户端|服务器）发送的实体数据的数据类型。 
比如：Content-Type：text/html; 
代表发送端发送的数据格式是html。

二者合起来， 
Accept:text/xml； 
Content-Type:text/html 
即代表希望接受的数据类型是xml格式，本次请求发送的数据的数据格式是html。

# 参考
https://zh.wikipedia.org/wiki/%E4%BA%92%E8%81%94%E7%BD%91%E5%AA%92%E4%BD%93%E7%B1%BB%E5%9E%8B