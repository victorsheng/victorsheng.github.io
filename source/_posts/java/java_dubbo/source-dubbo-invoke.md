---
title: source_dubbo_invoke
tags:
  - dubbo
categories:
  - dubbo
abbrlink: 3443009135
date: 2018-02-01 19:41:00
---



# 消费方:
```
"main@1" prio=5 tid=0x1 nid=NA runnable
  java.lang.Thread.State: RUNNABLE
	  at com.alibaba.dubbo.rpc.protocol.dubbo.DubboInvoker.doInvoke(DubboInvoker.java:69)
	  at com.alibaba.dubbo.rpc.protocol.AbstractInvoker.invoke(AbstractInvoker.java:142)
	  at com.alibaba.dubbo.rpc.listener.ListenerInvokerWrapper.invoke(ListenerInvokerWrapper.java:73)
	  at com.alibaba.dubbo.monitor.support.MonitorFilter.invoke(MonitorFilter.java:74)
	  at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:68)
	  at com.alibaba.dubbo.rpc.protocol.dubbo.filter.FutureFilter.invoke(FutureFilter.java:53)
	  at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:68)
	  at com.alibaba.dubbo.rpc.filter.ConsumerContextFilter.invoke(ConsumerContextFilter.java:47)
	  at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:68)
	  at com.alibaba.dubbo.rpc.protocol.InvokerWrapper.invoke(InvokerWrapper.java:52)
	  at com.alibaba.dubbo.rpc.cluster.support.FailoverClusterInvoker.doInvoke(FailoverClusterInvoker.java:77)
	  at com.alibaba.dubbo.rpc.cluster.support.AbstractClusterInvoker.invoke(AbstractClusterInvoker.java:232)
	  at com.alibaba.dubbo.rpc.cluster.support.wrapper.MockClusterInvoker.invoke(MockClusterInvoker.java:70)
	  at com.alibaba.dubbo.rpc.proxy.InvokerInvocationHandler.invoke(InvokerInvocationHandler.java:51)
	  at com.alibaba.dubbo.common.bytecode.proxy0.sayHello(proxy0.java:-1)
	  at com.alibaba.dubbo.demo.consumer.Consumer.main(Consumer.java:35)
```

# 入口:InvokerInvocationHandler

# DubboInvoker

```
protected Result doInvoke(final Invocation invocation) throws Throwable {
        RpcInvocation inv = (RpcInvocation) invocation;
        final String methodName = RpcUtils.getMethodName(invocation);
        inv.setAttachment(Constants.PATH_KEY, getUrl().getPath());
        inv.setAttachment(Constants.VERSION_KEY, version);

        ExchangeClient currentClient;
        if (clients.length == 1) {
            currentClient = clients[0];
        } else {
            currentClient = clients[index.getAndIncrement() % clients.length];
        }
        try {
            boolean isAsync = RpcUtils.isAsync(getUrl(), invocation);
            boolean isOneway = RpcUtils.isOneway(getUrl(), invocation);
            int timeout = getUrl().getMethodParameter(methodName, Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
            if (isOneway) {
                boolean isSent = getUrl().getMethodParameter(methodName, Constants.SENT_KEY, false);
                currentClient.send(inv, isSent);
                RpcContext.getContext().setFuture(null);
                return new RpcResult();
            } else if (isAsync) {
                ResponseFuture future = currentClient.request(inv, timeout);
                RpcContext.getContext().setFuture(new FutureAdapter<Object>(future));
                return new RpcResult();
            } else {
                RpcContext.getContext().setFuture(null);
                return (Result) currentClient.request(inv, timeout).get();
            }
        } catch (TimeoutException e) {
            throw new RpcException(RpcException.TIMEOUT_EXCEPTION, "Invoke remote method timeout. method: " + invocation.getMethodName() + ", provider: " + getUrl() + ", cause: " + e.getMessage(), e);
        } catch (RemotingException e) {
            throw new RpcException(RpcException.NETWORK_EXCEPTION, "Failed to invoke remote method: " + invocation.getMethodName() + ", provider: " + getUrl() + ", cause: " + e.getMessage(), e);
        }
    }
```


# 提供方

```
"DubboServerHandler-192.168.1.4:20880-thread-10@3010" daemon prio=5 tid=0x26 nid=NA runnable
  java.lang.Thread.State: RUNNABLE
	  at com.alibaba.dubbo.demo.provider.DemoServiceImpl.sayHello(DemoServiceImpl.java:28)
	  at com.alibaba.dubbo.common.bytecode.Wrapper1.invokeMethod(Wrapper1.java:-1)
	  at com.alibaba.dubbo.rpc.proxy.javassist.JavassistProxyFactory$1.doInvoke(JavassistProxyFactory.java:45)
	  at com.alibaba.dubbo.rpc.proxy.AbstractProxyInvoker.invoke(AbstractProxyInvoker.java:71)
	  at com.alibaba.dubbo.config.invoker.DelegateProviderMetaDataInvoker.invoke(DelegateProviderMetaDataInvoker.java:48)
	  at com.alibaba.dubbo.rpc.protocol.InvokerWrapper.invoke(InvokerWrapper.java:52)
	  at com.alibaba.dubbo.rpc.filter.ExceptionFilter.invoke(ExceptionFilter.java:61)
	  at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:68)
	  at com.alibaba.dubbo.monitor.support.MonitorFilter.invoke(MonitorFilter.java:74)
	  at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:68)
	  at com.alibaba.dubbo.rpc.filter.TimeoutFilter.invoke(TimeoutFilter.java:41)
	  at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:68)
	  at com.alibaba.dubbo.rpc.protocol.dubbo.filter.TraceFilter.invoke(TraceFilter.java:77)
	  at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:68)
	  at com.alibaba.dubbo.rpc.filter.ContextFilter.invoke(ContextFilter.java:71)
	  at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:68)
	  at com.alibaba.dubbo.rpc.filter.GenericFilter.invoke(GenericFilter.java:131)
	  at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:68)
	  at com.alibaba.dubbo.rpc.filter.ClassLoaderFilter.invoke(ClassLoaderFilter.java:37)
	  at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:68)
	  at com.alibaba.dubbo.rpc.filter.EchoFilter.invoke(EchoFilter.java:37)
	  at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:68)
	  at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol$1.reply(DubboProtocol.java:99)
	  at com.alibaba.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.handleRequest(HeaderExchangeHandler.java:96)
	  at com.alibaba.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.received(HeaderExchangeHandler.java:168)
	  at com.alibaba.dubbo.remoting.transport.DecodeHandler.received(DecodeHandler.java:50)
	  at com.alibaba.dubbo.remoting.transport.dispatcher.ChannelEventRunnable.run(ChannelEventRunnable.java:79)
	  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	  at java.lang.Thread.run(Thread.java:748)


```

## ChannelEventRunnable


