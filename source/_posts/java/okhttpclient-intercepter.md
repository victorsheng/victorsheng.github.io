title: okhttpclient拦截器
author: Victor
date: 2018-06-26 10:48:03
tags:
    - httpclient
categories:
---
![upload successful](/images/pasted-193.png)

# 应用拦截器

- 不需要关心像重定向和重试这样的中间响应。
- 总是调用一次，即使HTTP响应从缓存中获取服务。
- 监视应用原始意图。不关心OkHttp注入的像If-None-Match头。
- 允许短路并不调用Chain.proceed()。
- 允许重试并执行多个Chain.proceed()调用。
# 网络拦截器

- 可以操作像重定向和重试这样的中间响应。
- 对于短路网络的缓存响应不会调用。
- 监视即将要通过网络传输的数据。
- 访问运输请求的Connection。