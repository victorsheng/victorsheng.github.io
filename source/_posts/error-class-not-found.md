---
title: bugfix-class-not-found
abbrlink: 3659882520
date: 2018-10-11 18:37:21
tags:
categories:
---
```
2018-10-11-18:36:00.877 ERROR [qtp1744347043-8]-com.demo.book.provider.GeneralExceptionHandler:25>>Request 480549ae1f074a2090059558f5abe86d process fail.
java.lang.NoClassDefFoundError: com/demo/opal/book/service/impl/bookServiceImpl$1
        at com.demo.book.service.impl.bookServiceImpl.updateApproveStatus(bookServiceImpl.java:77)
        at com.demo.book.service.bookService.bookVettedResult(bookService.java:46)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:497)
        at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:333)
        at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:207)
        at com.sun.proxy.$Proxy32.bookVettedResult(Unknown Source)
        at com.demo.book.controller.bookResource.applyNotice(bookResource.java:72)
        at com.demo.book.controller.bookResource_$$_jvstc45_0._d1applyNotice(bookResource_$$_jvstc45_0.java)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:497)
        at org.glassfish.hk2.utilities.reflection.ReflectionHelper.invoke(ReflectionHelper.java:1287)
        at org.jvnet.hk2.internal.MethodInterceptorHandler$MethodInvocationImpl.proceed(MethodInterceptorHandler.java:188)
        at com.demo.util.security.signature.interceptor.SignInterceptor.invoke(SignInterceptor.java:30)
        at org.jvnet.hk2.internal.MethodInterceptorHandler.invoke(MethodInterceptorHandler.java:121)
        at com.demo.book.controller.bookResource_$$_jvstc45_0.applyNotice(bookResource_$$_jvstc45_0.java)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:497)
        at org.glassfish.jersey.server.model.internal.ResourceMethodInvocationHandlerFactory.lambda$static$0(ResourceMethodInvocationHandlerFactory.java:76)
        at org.glassfish.jersey.server.model.internal.AbstractJavaResourceMethodDispatcher$1.run(AbstractJavaResourceMethodDispatcher.java:148)
        at org.glassfish.jersey.server.model.internal.AbstractJavaResourceMethodDispatcher.invoke(AbstractJavaResourceMethodDispatcher.java:191)
        at org.glassfish.jersey.server.model.internal.JavaResourceMethodDispatcherProvider$ResponseOutInvoker.doDispatch(JavaResourceMethodDispatcherProvider.java:200)
        at org.glassfish.jersey.server.model.internal.AbstractJavaResourceMethodDispatcher.dispatch(AbstractJavaResourceMethodDispatcher.java:103)
        at org.glassfish.jersey.server.model.ResourceMethodInvoker.invoke(ResourceMethodInvoker.java:493)
        at org.glassfish.jersey.server.model.ResourceMethodInvoker.apply(ResourceMethodInvoker.java:415)
        at org.glassfish.jersey.server.model.ResourceMethodInvoker.apply(ResourceMethodInvoker.java:104)
        at org.glassfish.jersey.server.ServerRuntime$1.run(ServerRuntime.java:277)
        at org.glassfish.jersey.internal.Errors$1.call(Errors.java:272)
        at org.glassfish.jersey.internal.Errors$1.call(Errors.java:268)
        at org.glassfish.jersey.internal.Errors.process(Errors.java:316)
        at org.glassfish.jersey.internal.Errors.process(Errors.java:298)
        at org.glassfish.jersey.internal.Errors.process(Errors.java:268)
        at org.glassfish.jersey.process.internal.RequestScope.runInScope(RequestScope.java:289)
        at org.glassfish.jersey.server.ServerRuntime.process(ServerRuntime.java:256)
        at org.glassfish.jersey.server.ApplicationHandler.handle(ApplicationHandler.java:703)
        at org.glassfish.jersey.servlet.WebComponent.serviceImpl(WebComponent.java:416)
        at org.glassfish.jersey.servlet.WebComponent.service(WebComponent.java:370)
        at org.glassfish.jersey.servlet.ServletContainer.service(ServletContainer.java:389)
        at org.glassfish.jersey.servlet.ServletContainer.service(ServletContainer.java:342)
        at org.glassfish.jersey.servlet.ServletContainer.service(ServletContainer.java:229)
        at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:864)
        at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1655)
```

https://stackoverflow.com/questions/17973970/how-to-solve-java-lang-noclassdeffounderror