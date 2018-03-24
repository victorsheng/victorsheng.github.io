title: bugfix类转换异常
date: 2018-03-13 15:47:48
tags:
categories:
---
# 异常信息
```
2018-03-13 15:36:51.409 [DubboServerHandler-10.47.124.90:20888-thread-100] ERROR com.alibaba.dubbo.rpc.filter.ExceptionFilter.error -  [DUBBO] Got unchecked and undeclared exception which called by 10.47.124.90. service: com.ejlerp.pms.api.OutOfStockOrderService, method: generateOOSPurchaseOrder, exception: java.lang.ClassCastException: com.alibaba.fastjson.JSONObject cannot be cast to com.ejlerp.pms.domain.PmsPurchaseOrderDetail, dubbo version: 2.8.4, current host: 10.47.124.90
java.lang.ClassCastException: com.alibaba.fastjson.JSONObject cannot be cast to com.ejlerp.pms.domain.PmsPurchaseOrderDetail
        at com.ejlerp.pms.provider.service.DailyPurchaseServiceImpl.writeBackDetailInfoToDailyDetails(DailyPurchaseServiceImpl.java:502)
        at com.ejlerp.pms.provider.service.DailyPurchaseServiceImpl.generatePurchaseOrderAndDetails(DailyPurchaseServiceImpl.java:121)
        at com.ejlerp.pms.provider.service.DailyPurchaseServiceImpl$$FastClassBySpringCGLIB$$3b1a314.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
:
eateNum=null,planCreateDate=null,version=null,cumulativeDefectNum=0,creator=null,createdAt=null,lastUpdater=null,lastUpdated=null,tenantId=null,isUsable=null]]
2018-03-13 15:36:51.395 [DubboServerHandler-10.47.124.90:20888-thread-100] DEBUG com.ejlerp.dal.framework.service.advice.AvoidRepeatInvokeAdvice.aroundAdvice - 新建并发锁[msi_lock_pms-1035108456]
2018-03-13 15:36:51.399 [DubboServerHandler-10.47.124.90:20888-thread-100] WARN  com.ejlerp.dal.framework.service.advice.AvoidRepeatInvokeAdvice.aroundAdvice - 存在重复锁[msi_fp_pms-1035108456]
2018-03-13 15:36:51.401 [DubboServerHandler-10.47.124.90:20888-thread-100] DEBUG com.ejlerp.dal.framework.service.advice.AvoidRepeatInvokeAdvice.aroundAdvice - 释放并发锁[msi_lock_pms-1035108456]
2018-03-13 15:36:51.404 [DubboServerHandler-10.47.124.90:20888-thread-100] DEBUG com.ejlerp.dal.framework.service.advice.AvoidRepeatInvokeAdvice.aroundAdvice - 释放并发锁[msi_lock_pms1289336139]
2018-03-13 15:36:51.409 [DubboServerHandler-10.47.124.90:20888-thread-100] ERROR com.alibaba.dubbo.rpc.filter.ExceptionFilter.error -  [DUBBO] Got unchecked and undeclared exception which called by 10.47.124.90. service: com.ejlerp.pms.api.OutOfStockOrderService, method: generateOOSPurchaseOrder, exception: java.lang.ClassCastException: com.alibaba.fastjson.JSONObject cannot be cast to com.ejlerp.pms.domain.PmsPurchaseOrderDetail, dubbo version: 2.8.4, current host: 10.47.124.90
java.lang.ClassCastException: com.alibaba.fastjson.JSONObject cannot be cast to com.ejlerp.pms.domain.PmsPurchaseOrderDetail
        at com.ejlerp.pms.provider.service.DailyPurchaseServiceImpl.writeBackDetailInfoToDailyDetails(DailyPurchaseServiceImpl.java:502)
        at com.ejlerp.pms.provider.service.DailyPurchaseServiceImpl.generatePurchaseOrderAndDetails(DailyPurchaseServiceImpl.java:121)
        at com.ejlerp.pms.provider.service.DailyPurchaseServiceImpl$$FastClassBySpringCGLIB$$3b1a314.invoke(<generated>)
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
        at sun.reflect.GeneratedMethodAccessor270.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethodWithGivenArgs(AbstractAspectJAdvice.java:629)
        at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethod(AbstractAspectJAdvice.java:618)
        at org.springframework.aop.aspectj.AspectJAroundAdvice.invoke(AspectJAroundAdvice.java:70)
:
eateNum=null,planCreateDate=null,version=null,cumulativeDefectNum=0,creator=null,createdAt=null,lastUpdater=null,lastUpdated=null,tenantId=null,isUsable=null]]
2018-03-13 15:36:51.395 [DubboServerHandler-10.47.124.90:20888-thread-100] DEBUG com.ejlerp.dal.framework.service.advice.AvoidRepeatInvokeAdvice.aroundAdvice - 新建并发锁[msi_lock_pms-1035108456]
2018-03-13 15:36:51.399 [DubboServerHandler-10.47.124.90:20888-thread-100] WARN  com.ejlerp.dal.framework.service.advice.AvoidRepeatInvokeAdvice.aroundAdvice - 存在重复锁[msi_fp_pms-1035108456]
2018-03-13 15:36:51.401 [DubboServerHandler-10.47.124.90:20888-thread-100] DEBUG com.ejlerp.dal.framework.service.advice.AvoidRepeatInvokeAdvice.aroundAdvice - 释放并发锁[msi_lock_pms-1035108456]
2018-03-13 15:36:51.404 [DubboServerHandler-10.47.124.90:20888-thread-100] DEBUG com.ejlerp.dal.framework.service.advice.AvoidRepeatInvokeAdvice.aroundAdvice - 释放并发锁[msi_lock_pms1289336139]
2018-03-13 15:36:51.409 [DubboServerHandler-10.47.124.90:20888-thread-100] ERROR com.alibaba.dubbo.rpc.filter.ExceptionFilter.error -  [DUBBO] Got unchecked and undeclared exception which called by 10.47.124.90. service: com.ejlerp.pms.api.OutOfStockOrderService, method: generateOOSPurchaseOrder, exception: java.lang.ClassCastException: com.alibaba.fastjson.JSONObject cannot be cast to com.ejlerp.pms.domain.PmsPurchaseOrderDetail, dubbo version: 2.8.4, current host: 10.47.124.90
java.lang.ClassCastException: com.alibaba.fastjson.JSONObject cannot be cast to com.ejlerp.pms.domain.PmsPurchaseOrderDetail
        at com.ejlerp.pms.provider.service.DailyPurchaseServiceImpl.writeBackDetailInfoToDailyDetails(DailyPurchaseServiceImpl.java:502)
        at com.ejlerp.pms.provider.service.DailyPurchaseServiceImpl.generatePurchaseOrderAndDetails(DailyPurchaseServiceImpl.java:121)
        at com.ejlerp.pms.provider.service.DailyPurchaseServiceImpl$$FastClassBySpringCGLIB$$3b1a314.invoke(<generated>)
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
        at sun.reflect.GeneratedMethodAccessor270.invoke(Unknown Source)
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
        at com.ejlerp.pms.provider.service.DailyPurchaseServiceImpl$$EnhancerBySpringCGLIB$$41b3ba70.generatePurchaseOrderAndDetails(<generated>)
        at sun.reflect.GeneratedMethodAccessor283.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:333)
        at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:207)
        at com.sun.proxy.$Proxy77.generatePurchaseOrderAndDetails(Unknown Source)
        at com.ejlerp.pms.provider.service.oms.OutOfStockOrderServiceImpl$ToPurchaseOrder.invoke(OutOfStockOrderServiceImpl.java:931)
        at com.ejlerp.pms.provider.service.oms.OutOfStockOrderServiceImpl.getToPurchaseOrderResult(OutOfStockOrderServiceImpl.java:250)
        at com.ejlerp.pms.provider.service.oms.OutOfStockOrderServiceImpl.generateOOSPurchaseOrder(OutOfStockOrderServiceImpl.java:215)
        at com.ejlerp.pms.provider.service.oms.OutOfStockOrderServiceImpl.generateOOSPurchaseOrder(OutOfStockOrderServiceImpl.java:125)
        at com.ejlerp.pms.provider.service.oms.OutOfStockOrderServiceImpl$$FastClassBySpringCGLIB$$5a5ccc6a.invoke(<generated>)
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
        at com.ejlerp.pms.provider.service.oms.OutOfStockOrderServiceImpl$$EnhancerBySpringCGLIB$$c7b6bc32.generateOOSPurchaseOrder(<generated>)
        at com.alibaba.dubbo.common.bytecode.Wrapper56.invokeMethod(Wrapper56.java)
        at com.alibaba.dubbo.rpc.proxy.javassist.JavassistProxyFactory$1.doInvoke(JavassistProxyFactory.java:46)
        at com.alibaba.dubbo.rpc.proxy.AbstractProxyInvoker.invoke(AbstractProxyInvoker.java:72)
        at com.alibaba.dubbo.rpc.protocol.InvokerWrapper.invoke(InvokerWrapper.java:53)
        at com.alibaba.dubbo.rpc.filter.ExceptionFilter.invoke(ExceptionFilter.java:64)
        at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
        at com.alibaba.dubbo.monitor.support.MonitorFilter.invoke(MonitorFilter.java:75)
        at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
        at com.alibaba.dubbo.rpc.filter.TimeoutFilter.invoke(TimeoutFilter.java:42)
        at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
        at com.alibaba.dubbo.rpc.protocol.dubbo.filter.TraceFilter.invoke(TraceFilter.java:78)
        at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
        at com.alibaba.dubbo.rpc.filter.ContextFilter.invoke(ContextFilter.java:70)
        at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
        at com.alibaba.dubbo.rpc.filter.GenericFilter.invoke(GenericFilter.java:132)
        at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
        at com.alibaba.dubbo.rpc.filter.ClassLoaderFilter.invoke(ClassLoaderFilter.java:38)
        at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
        at com.alibaba.dubbo.rpc.filter.EchoFilter.invoke(EchoFilter.java:38)
        at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
        at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol$1.reply(DubboProtocol.java:113)
        at com.alibaba.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.handleRequest(HeaderExchangeHandler.java:84)
        at com.alibaba.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.received(HeaderExchangeHandler.java:170)
        at com.alibaba.dubbo.remoting.transport.DecodeHandler.received(DecodeHandler.java:52)
        at com.alibaba.dubbo.remoting.transport.dispatcher.ChannelEventRunnable.run(ChannelEventRunnable.java:82)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)
```

# 排查步骤
- 1.排查最近的代码改动,没有相关的修改
- 2.在代码中加入debug信息,逐行查找变化
- 3.排查发现可能是aop的代码导致
```
@AvoidRepeatInvoke(prefix = "pms")
```

- 4.原先的逻辑
```
            if (kvCacher.exist(footprint)) {
                //判断redis是否存在调用过的足迹,如有,直接返回上一次的结果
                LOGGER.warn("存在重复锁[{}]", footprint);
                return JSON.parseObject(kvCacher.get(fpVal), retClass);

```

- 5.fastjson中对于带泛型的反持久化应该用type的方式
https://github.com/alibaba/fastjson/wiki/TypeReference

- 6.试验

```
public class Reflect {

    public List<A> hello() {
        return null;
    }


    public static void main(String[] args) throws NoSuchMethodException, NoSuchFieldException, IllegalAccessException {
        Class<Reflect> reflectClass = Reflect.class;
        Method hello = reflectClass.getMethod("hello", null);
        Type genericReturnType = hello.getGenericReturnType();
        System.out.println(genericReturnType);
        ArrayList<A> list = new ArrayList<>();
        list.add(new A("AA1"));
        list.add(new A("AA2"));
        list.add(new A("AA3"));
        list.add(new A("AA4"));
        String json = JSON.toJSONString(list);
        //传统方式
//        List<A> as = JSON.parseObject(json, new TypeReference<List<A>>() {}.getType());
		//反射方式
        List<A> as = JSON.parseObject(json, new TypeReference4Reflect(genericReturnType).getType());

        System.out.println(as);
    }


    public static class A implements Serializable {
        String name;

        public A(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            final StringBuffer sb = new StringBuffer("A{");
            sb.append("name='").append(name).append('\'');
            sb.append('}');
            return sb.toString();
        }
    }

    static public class TypeReference4Reflect<T> extends TypeReference<T> {
        public TypeReference4Reflect(T t) throws IllegalArgumentException, IllegalAccessException, SecurityException, NoSuchFieldException {
            Class<?> cla = TypeReference.class;
            Field field = cla.getDeclaredField("type");
            field.setAccessible(true);
            field.set(this, t);
        }
    }
}

```

- 7.改造

参考:http://www.iteye.com/problems/80586

```
                return JSON.parseObject(cacheResult, new TypeReference4Reflect(genericReturnType).getType());


    static public class TypeReference4Reflect<T> extends TypeReference<T> {
        public TypeReference4Reflect(T t) throws IllegalArgumentException, IllegalAccessException, SecurityException, NoSuchFieldException {
            Class<?> cla = TypeReference.class;
            Field field = cla.getDeclaredField("type");
            field.setAccessible(true);
            field.set(this, t);
        }
    }

```

# 反思
- 日志中可能会有细节,要注意观察
- 排查步骤,要全面,逐步深入,debug信息表示调用行数