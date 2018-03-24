title: source-dubbo-filter
date: 2018-02-03 15:45:49
tags:
categories:
---
# dubbo中提供的Filter实现
```
echo=com.alibaba.dubbo.rpc.filter.EchoFilter
generic=com.alibaba.dubbo.rpc.filter.GenericFilter
genericimpl=com.alibaba.dubbo.rpc.filter.GenericImplFilter
token=com.alibaba.dubbo.rpc.filter.TokenFilter
accesslog=com.alibaba.dubbo.rpc.filter.AccessLogFilter
activelimit=com.alibaba.dubbo.rpc.filter.ActiveLimitFilter
classloader=com.alibaba.dubbo.rpc.filter.ClassLoaderFilter
context=com.alibaba.dubbo.rpc.filter.ContextFilter
consumercontext=com.alibaba.dubbo.rpc.filter.ConsumerContextFilter
exception=com.alibaba.dubbo.rpc.filter.ExceptionFilter
executelimit=com.alibaba.dubbo.rpc.filter.ExecuteLimitFilter
deprecated=com.alibaba.dubbo.rpc.filter.DeprecatedFilter
compatible=com.alibaba.dubbo.rpc.filter.CompatibleFilter
timeout=com.alibaba.dubbo.rpc.filter.TimeoutFilter
```

# 顺序
服务提供方的过滤器被调用顺序：
EchoFilter->ClassLoaderFilter->GenericFilter->ContextFilter->(这4个是在代码中指定的)
ExceptionFilter-> TimeoutFilter ->MonitorFilter-> TraceFilter
服务消费方的过滤器顺序：
ConsumerContextFilter->FutureFilter->MonitorFilter
负责加载过滤器的类
ProtocolFilterWrapper

# 构建Filter链

```
//provider
public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
        if (Constants.REGISTRY_PROTOCOL.equals(invoker.getUrl().getProtocol())) {
            return protocol.export(invoker);
        }
        return protocol.export(buildInvokerChain(invoker, Constants.SERVICE_FILTER_KEY, Constants.PROVIDER));
    }
//consumer
public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
        if (Constants.REGISTRY_PROTOCOL.equals(url.getProtocol())) {
            return protocol.refer(type, url);
        }
        return buildInvokerChain(protocol.refer(type, url), Constants.REFERENCE_FILTER_KEY, Constants.CONSUMER);
    }

private static <T> Invoker<T> buildInvokerChain(final Invoker<T> invoker, String key, String group) {
        Invoker<T> last = invoker;
        List<Filter> filters = ExtensionLoader.getExtensionLoader(Filter.class).getActivateExtension(invoker.getUrl(), key, group);
        if (filters.size() > 0) {
            for (int i = filters.size() - 1; i >= 0; i--) {
                final Filter filter = filters.get(i);
                final Invoker<T> next = last;
                last = new Invoker<T>() {

                    public Class<T> getInterface() {
                        return invoker.getInterface();
                    }

                    public URL getUrl() {
                        return invoker.getUrl();
                    }

                    public boolean isAvailable() {
                        return invoker.isAvailable();
                    }

                    public Result invoke(Invocation invocation) throws RpcException {
                        return filter.invoke(next, invocation);
                    }

                    public void destroy() {
                        invoker.destroy();
                    }

                    @Override
                    public String toString() {
                        return invoker.toString();
                    }
                };
            }
        }
        return last;
    }
```


# ExceptionFilter
某个系统调用dubbo请求，provider端（服务提供方）抛出了自定义的业务异常，但consumer端（服务消费方）拿到的并不是自定义的业务异常。
这是为什么呢？

```
@Activate(group = Constants.PROVIDER)  
public class ExceptionFilter implements Filter {  
  
    private final Logger logger;  
      
    public ExceptionFilter() {  
        this(LoggerFactory.getLogger(ExceptionFilter.class));  
    }  
      
    public ExceptionFilter(Logger logger) {  
        this.logger = logger;  
    }  
      
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {  
        try {  
            Result result = invoker.invoke(invocation);  
            if (result.hasException() && GenericService.class != invoker.getInterface()) {  
                try {  
                    Throwable exception = result.getException();  
  
                    // 如果是checked异常，直接抛出  
                    if (! (exception instanceof RuntimeException) && (exception instanceof Exception)) {  
                        return result;  
                    }  
                    // 在方法签名上有声明，直接抛出  
                    try {  
                        Method method = invoker.getInterface().getMethod(invocation.getMethodName(), invocation.getParameterTypes());  
                        Class<?>[] exceptionClassses = method.getExceptionTypes();  
                        for (Class<?> exceptionClass : exceptionClassses) {  
                            if (exception.getClass().equals(exceptionClass)) {  
                                return result;  
                            }  
                        }  
                    } catch (NoSuchMethodException e) {  
                        return result;  
                    }  
  
                    // 未在方法签名上定义的异常，在服务器端打印ERROR日志  
                    logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost()  
                            + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()  
                            + ", exception: " + exception.getClass().getName() + ": " + exception.getMessage(), exception);  
  
                    // 异常类和接口类在同一jar包里，直接抛出  
                    String serviceFile = ReflectUtils.getCodeBase(invoker.getInterface());  
                    String exceptionFile = ReflectUtils.getCodeBase(exception.getClass());  
                    if (serviceFile == null || exceptionFile == null || serviceFile.equals(exceptionFile)){  
                        return result;  
                    }  
                    // 是JDK自带的异常，直接抛出  
                    String className = exception.getClass().getName();  
                    if (className.startsWith("java.") || className.startsWith("javax.")) {  
                        return result;  
                    }  
                    // 是Dubbo本身的异常，直接抛出  
                    if (exception instanceof RpcException) {  
                        return result;  
                    }  
  
                    // 否则，包装成RuntimeException抛给客户端  
                    return new RpcResult(new RuntimeException(StringUtils.toString(exception)));  
                } catch (Throwable e) {  
                    logger.warn("Fail to ExceptionFilter when called by " + RpcContext.getContext().getRemoteHost()  
                            + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()  
                            + ", exception: " + e.getClass().getName() + ": " + e.getMessage(), e);  
                    return result;  
                }  
            }  
            return result;  
        } catch (RuntimeException e) {  
            logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost()  
                    + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()  
                    + ", exception: " + e.getClass().getName() + ": " + e.getMessage(), e);  
            throw e;  
        }  
    }  
  
}  
```
http://blog.csdn.net/mj158518/article/details/51228649