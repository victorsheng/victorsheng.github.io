---
title: 线上服务无法打印异常
tags:
  - bugfix
  - jvm
  - OmitStackTraceInFastThrow
categories:
  - jvm
abbrlink: 3834343607
date: 2018-02-05 15:53:05
---
# 线上异常无堆栈信息


```
2018-02-04 15:27:54.153 [DubboServerHandler-10.26.235.193:20884-thread-100] ERROR com.alibaba.dubbo.rpc.filter.ExceptionFilter.error -  [DUBBO] Got unch
ecked and undeclared exception which called by 10.26.235.193. service: com.ejlerp.pms.api.ArrivalRecordService, method: purchaseConfirm, exception: java
.lang.NullPointerException: null, dubbo version: 2.8.4, current host: 10.26.235.193
java.lang.NullPointerException: null

```
- 无任何其他异常信息

- 查看dubbo源码:
ExceptionFilter

```
     logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost()
                            + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                            + ", exception: " + exception.getClass().getName() + ": " + exception.getMessage(), exception);

```


# 问题排查
- 对比每天的日志:发现升级后就出现了问题
- 对比其他机器是否有该问题:都有
- 对比生产/测试环境是否有该问题:发现hz6有问题


```
2018-02-02 14:12:51.230 [DubboServerHandler-10.47.124.90:20886-thread-86] ERROR com.alibaba.dubbo.rpc.filter.ExceptionFilter.error -  [DUBBO] Got unchecked and undeclared exception which called by 10.47.124.90. service: com.ejlerp.pms.api.ArrivalRecordService, method: purchaseConfirm, exception: java.lang.NullPointerException: null, dubbo version: 2.8.4, current host: 10.47.124.90
java.lang.NullPointerException: null
        at com.ejlerp.dal.framework.dao.impl.BaseDaoImpl.batchUpdate(BaseDaoImpl.java:766)
        at com.ejlerp.dal.framework.dao.impl.BaseDaoImpl$$FastClassBySpringCGLIB$$26be7043.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:738)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
        at org.springframework.dao.support.PersistenceExceptionTranslationInterceptor.invoke(PersistenceExceptionTranslationInterceptor.java:136)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:673)
        at com.ejlerp.pms.provider.dao.jdbc.PurchaseOrderDetailDaoImpl$$EnhancerBySpringCGLIB$$4f4f9329.batchUpdate(<generated>)
        at com.ejlerp.pms.provider.service.PurchaseOrderDetailServiceImpl.batchUpdate(PurchaseOrderDetailServiceImpl.java:963)
        at com.ejlerp.pms.provider.service.PurchaseOrderDetailServiceImpl$$FastClassBySpringCGLIB$$e0903e90.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:738)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
        at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:99)
        at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:282)
        at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:96)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:673)
        at com.ejlerp.pms.provider.service.PurchaseOrderDetailServiceImpl$$EnhancerBySpringCGLIB$$a4c2c1fc.batchUpdate(<generated>)
        at com.ejlerp.pms.provider.service.DailyPurchaseDetailServiceImpl.handleUniqueCodeArrive(DailyPurchaseDetailServiceImpl.java:830)
        at com.ejlerp.pms.provider.service.DailyPurchaseDetailServiceImpl.updatePurchaseState(DailyPurchaseDetailServiceImpl.java:773)
        at com.ejlerp.pms.provider.service.DailyPurchaseDetailServiceImpl$$FastClassBySpringCGLIB$$d0f852c3.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:738)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
        at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:99)
        at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:282)
        at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:96)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:673)
        at com.ejlerp.pms.provider.service.DailyPurchaseDetailServiceImpl$$EnhancerBySpringCGLIB$$5a3b675.updatePurchaseState(<generated>)
        at com.ejlerp.pms.provider.service.ArrivalRecordServiceImpl.purchaseConfirm(ArrivalRecordServiceImpl.java:166)
        at com.ejlerp.pms.provider.service.ArrivalRecordServiceImpl$$FastClassBySpringCGLIB$$eab61da4.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:738)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
        at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:99)
        at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:282)
        at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:96)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor.invoke(MethodBeforeAdviceInterceptor.java:52)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:168)
        at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:97)
        at com.ejlerp.dal.framework.service.advice.AvoidRepeatInvokeAdvice.aroundAdvice(AvoidRepeatInvokeAdvice.java:131)
        at sun.reflect.GeneratedMethodAccessor266.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethodWithGivenArgs(AbstractAspectJAdvice.java:629)
        at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethod(AbstractAspectJAdvice.java:618)
        at org.springframework.aop.aspectj.AspectJAroundAdvice.invoke(AspectJAroundAdvice.java:70)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:168)
        at org.springframework.aop.aspectj.AspectJAfterAdvice.invoke(AspectJAfterAdvice.java:47)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:168)
        at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:673)
        at com.ejlerp.pms.provider.service.ArrivalRecordServiceImpl$$EnhancerBySpringCGLIB$$c67e6bb8.purchaseConfirm(<generated>)
        at com.alibaba.dubbo.common.bytecode.Wrapper64.invokeMethod(Wrapper64.java)
        at com.alibaba.dubbo.rpc.proxy.javassist.JavassistProxyFactory$1.doInvoke(JavassistProxyFactory.java:46)
        at com.alibaba.dubbo.rpc.proxy.AbstractProxyInvoker.invoke(AbstractProxyInvoker.java:72)
```

# 问题
## 问题1(业务错误)
java.lang.NullPointerException: null
        at com.ejlerp.dal.framework.dao.impl.BaseDaoImpl.batchUpdate(BaseDaoImpl.java:766)


## 问题2 jvm jit编译优化
由于服务一直没有重启过,在导致过多次数的异常后,jvm对异常进行了优化

# OmitStackTraceInFastThrow参数
在生产环境JRE 运行在server 模式下， 从日志上看大量的NullPointException日志打印时，没有堆栈信息输出。查了一下，JIT编译会对某些异常如果大量的抛出时，会进行优化，删除堆栈信息。

这是HotSpot VM专门针对异常做的一个优化，称为fast throw，当一些异常在代码里某个特定位置被抛出很多次的话，HotSpot Server Compiler（C2）会用fast throw来优化这个抛出异常的地方，直接抛出一个事先分配好的、类型匹配的对象，这个对象的message和stack trace都被清空。

可以明确：抛出这个异常非常快，不用额外分配内存，也不用爬栈。

- -XX:-OmitStackTraceInFastThrow 关闭异常堆栈优化
- -Xint 以解释模式执行


加号则表示启用
减号代表关闭

# 启动脚本里没有添加-server参数
-server，在64位linux中，你想设成-client都不行的，所以写了也是白写。

## 在排查问题时,发现的另一个错误用法
启动脚本
错误:

```
>/dev/null 2> 1 &
```

正确

```
> /dev/null 2>&1 &
```

http://www.cnblogs.com/caolisong/archive/2007/04/25/726896.html
