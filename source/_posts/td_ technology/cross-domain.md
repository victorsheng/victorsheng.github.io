---
title: cross-domain
abbrlink: 4048241359
date: 2018-11-06 10:30:14
tags:
categories:
---
# server端使用COR跨域共享

CORS是一种网络浏览器的技术规范，它为Web服务器定义了一种方式，允许网页从不同的域访问其资源。而这种访问是被同源策略所禁止的。CORS系统定义了一种浏览器和服务器交互的方式来确定是否允许跨域请求。 

通过阅读我们知道，当我们进行跨越请求的时候，因为同源策略的限制，如果访问跨域请求时，跨源资源共享(CORS)机制为web服务器跨域访问控制提供了安全的跨域数据传输。

使用CORS的方式非常简单，但是需要同时对前端和服务器端做相应处理。

1、  前端

客户端使用XmlHttpRequest发起Ajax请求，当前绝大部分浏览器已经支持CORS方式，且主流浏览器均提供了对跨域资源共享的支持。

2、  服务器端

如果服务器端未做任何配置，则前端发起Ajax请求后，会得到CORS Access Deny，即跨域访问被拒绝。


# GET方法的请求过程
直接访问

# 非GET方法的请求过程
1. option方法的请求(浏览器自动添加)
2. 非GET方法的原始请求


请求头:
```
Access-Control-Request-Headers: content-type
Access-Control-Request-Method: POST
Origin: http://debug.dmk.tendcloud.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36
```

响应头:
```
Access-Control-Allow-Headers: content-type
Access-Control-Allow-Methods: POST
Access-Control-Allow-Origin: *
Access-Control-Max-Age: 1800
Allow: GET, HEAD, POST, PUT, DELETE, TRACE, OPTIONS, PATCH
Server: Jetty(9.4.12.v20180830)
Transfer-Encoding: chunked
Vary: Origin, Access-Control-Request-Method, Access-Control-Request-Headers
```


## OPTIONS方法的请求
OPTIONS请求方法的主要用途有两个：
1、获取服务器支持的HTTP请求方法；
2、用来检查服务器的性能。例如：AJAX进行跨域请求时的预检，需要向另外一个域名的资源发送一个HTTP OPTIONS请求头，用以判断实际发送的请求是否安全。
这是浏览器给我们加上的，后端并没有做任何操作。


# 参考
https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS