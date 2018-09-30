---
title: dubbo-概述
tags:
  - dubbo
categories:
  - dubbo
abbrlink: 2822083703
date: 2018-02-01 21:53:00
---
# 各层说明
<img src="http://dubbo.io/books/dubbo-dev-book/sources/images/dubbo-framework.jpg"/>

- config 配置层：对外配置接口，以 ServiceConfig, ReferenceConfig 为中心，可以直接初始化配置类，也可以通过 spring 解析配置生成配置类
- proxy 服务代理层：服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton, 以 ServiceProxy 为中心，扩展接口为 ProxyFactory
- registry 注册中心层：封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为 RegistryFactory, Registry, RegistryService
- cluster 路由层：封装多个提供者的路由及负载均衡，并桥接注册中心，以 Invoker 为中心，扩展接口为 Cluster, Directory, Router, LoadBalance
- monitor 监控层：RPC 调用次数和调用时间监控，以 Statistics 为中心，扩展接口为 MonitorFactory, Monitor, MonitorService
- protocol 远程调用层：封将 RPC 调用，以 Invocation, Result 为中心，扩展接口为 Protocol, Invoker, Exporter
- exchange 信息交换层：封装请求响应模式，同步转异步，以 Request, Response 为中心，扩展接口为 Exchanger, ExchangeChannel, ExchangeClient, ExchangeServer
- transport 网络传输层：抽象 mina 和 netty 为统一接口，以 Message 为中心，扩展接口为 Channel, Transporter, Client, Server, Codec
- serialize 数据序列化层：可复用的一些工具，扩展接口为 Serialization, ObjectInput, ObjectOutput, ThreadPool

接口verion1
![upload successful](/images/pasted-50.png)

接口verion2
![upload successful](/images/pasted-51.png)

# 基类模板方法-->链式过滤器

```
public abstract AbstractInvoker implements Invoker {  

    public Result invoke(Invocation inv) throws RpcException {  
        // 伪代码  
        active ++;  
        if (active > max)  
            wait();  

        doInvoke(inv);  

        active --;  
        notify();  
    }  

    protected abstract Result doInvoke(Invocation inv) throws RpcException  

}
```


```
public abstract LimitFilter implements Filter {  

    public Result invoke(Invoker chain, Invocation inv) throws RpcException {  
         // 伪代码  
        active ++;  
        if (active > max)  
            wait();  

        chain.invoke(inv);  

        active --;  
        notify();  
    }  

}
```


Invoker, Exporter, InvocationHandler, FilterChain 其实都是 invoke 行为的不同阶段，完全可以抽象掉，统一为 Invoker，减少概念。