title: api网关学习
tags:
  - spring cloud
categories: []
date: 2017-12-18 19:42:00
---
# 什么是api网关
```
王延炯：API Gateway（API GW / API 网关），顾名思义，是出现在系统边界上的一个面向API的、串行集中式的强管控服务，这里的边界是企业IT系统的边界。

在微服务概念的流行之前，API GW的实体就已经诞生了，这时的主要应用场景是OpenAPI，也就是开放平台，面向的是企业外部合作伙伴，对于这个应用场景，相信接触的人会比较多。当在微服务概念流行起来之后，API网关似乎成了在上层应用层集成的标配组件。
```

# 为什么需要api网关
- 负载均衡
- 减少客户端与服务端的直接调用
- 容错
- 服务发现与注册
- 统一认证

# 选型
## spring cloud zuul
https://github.com/Netflix/zuul

## nginx kong
https://github.com/Kong/kong

# 参考资料
http://www.infoq.com/cn/news/2016/07/API-background-architecture-floo
http://www.infoq.com/cn/articles/construct-micro-service-using-api-gateway