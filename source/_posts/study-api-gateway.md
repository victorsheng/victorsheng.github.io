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

## 阿里云apigateway
https://www.aliyun.com/product/apigateway

![upload successful](/images/pasted-20.png)
#### API 生命周期管理

- 支持包括 API 发布、API 测试、API 下线等生命周期管理功能。
- 支持 API 日常管理、API 版本管理、API 快速回滚等维护功能。
#### 全面的安全防护

- 支持多种认证方式，支持 HMAC (SHA-1，SHA-256) 算法签名。
- 支持 HTTPS 协议，支持 SSL 加密。
- 防攻击、防注入、请求防重放、请求防篡改。
#### 灵活的权限控制

- 用户以 APP 作为请求 API 的身份，网关支持针对 APP 的权限控制。
- 只有已经获得授权的 APP 才能请求相应的 API。
- API 提供者可以主动授权某个 APP 调用某个 API 的权限。
- API 若上架到 API 市场，则购买者可以将已购买的 API 授权给自己的 APP。
#### 精准的流量控制

- 流量控制可以用于管控 API的被访问频率、APP的请求频率、用户的请求频率。
- 流量控制的时间单位可以是分钟、小时、天。
- 同时支持流控例外，允许设置特殊的 APP 或者用户。
#### 请求校验

- 支持参数类型、参数值（范围、枚举、正则、Json Schema）校验，无效校验直接会被 API 网关拒绝，减少无效请求对后端造成的资源浪费，大大降低后端服务的处理成本。
#### 数据转换

- 通过配置映射规则，实现前、后端数据翻译。
- 支持前端请求的数据转换。
- 支持返回结果的数据转换。
#### 监控报警

- 提供可视化的API实时监控，包括：调用量、流量大小、响应时间、错误率，在陆续增加维度。
- 支持历史情况查询，以便统筹分析。
- 可配置预警方式（短信、Email），订阅预警信息，以便实时掌握API运行情况。
#### 自动工具

- 自动生成 API 文档，可供在线查看。
- API 网关提供多种语言 SDK 的示例。降低 API 的运维成本。
- 提供可视化的界面调试工具，快速测试，快速上线。
#### API 市场

- 可将 API 上架到 API 市场，供更多开发者采购和使用。

# 参考资料
http://www.infoq.com/cn/news/2016/07/API-background-architecture-floo
http://www.infoq.com/cn/articles/construct-micro-service-using-api-gateway