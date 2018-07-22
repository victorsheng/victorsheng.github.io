title: springmvc源码分析
date: 2018-07-19 20:15:06
tags:
	- 源码
	- springmvc
categories:
---
# 调用顺序
errorPageFilter->hiddenHttpMethodFilter->httpPutFormContentFilter->requestContextFilter->HttpSignatureFilter->Jetty_WebSocketUpgradeFilter->dispatcherServlet@7ef5559e==org.springframework.web.servlet.DispatcherServlet,jsp=null,order=-1,inst=true

# 调用链
```
list:39, DataSetManController (com.talkingdata.opal.dataset.controller)
invoke0:-1, NativeMethodAccessorImpl (sun.reflect)
invoke:62, NativeMethodAccessorImpl (sun.reflect)
invoke:43, DelegatingMethodAccessorImpl (sun.reflect)
invoke:498, Method (java.lang.reflect)


doInvoke:209, InvocableHandlerMethod (org.springframework.web.method.support)
invokeForRequest:136, InvocableHandlerMethod (org.springframework.web.method.support)
invokeAndHandle:102, ServletInvocableHandlerMethod (org.springframework.web.servlet.mvc.method.annotation)


invokeHandlerMethod:877, RequestMappingHandlerAdapter (org.springframework.web.servlet.mvc.method.annotation)
handleInternal:783, RequestMappingHandlerAdapter (org.springframework.web.servlet.mvc.method.annotation)
handle:87, AbstractHandlerMethodAdapter (org.springframework.web.servlet.mvc.method)


doDispatch:991, DispatcherServlet (org.springframework.web.servlet)
doService:925, DispatcherServlet (org.springframework.web.servlet)
processRequest:974, FrameworkServlet (org.springframework.web.servlet)
doGet:866, FrameworkServlet (org.springframework.web.servlet)
service:687, HttpServlet (javax.servlet.http)
service:851, FrameworkServlet (org.springframework.web.servlet)
service:790, HttpServlet (javax.servlet.http)
handle:864, ServletHolder (org.eclipse.jetty.servlet)
doFilter:1655, ServletHandler$CachedChain (org.eclipse.jetty.servlet)

```


# RequestMappingHadlerAdpater

# HandlerMethod中干活的处理器


## InvocableHandlerMethod
```
	public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

		Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
		if (logger.isTraceEnabled()) {
			logger.trace("Invoking '" + ClassUtils.getQualifiedMethodName(getMethod(), getBeanType()) +
					"' with arguments " + Arrays.toString(args));
		}
		Object returnValue = doInvoke(args);
		if (logger.isTraceEnabled()) {
			logger.trace("Method [" + ClassUtils.getQualifiedMethodName(getMethod(), getBeanType()) +
					"] returned [" + returnValue + "]");
		}
		return returnValue;
	}
```




# HandlerMethodArgumentResolver接口
MapMethodProcessor (org.springframework.web.method.annotation)
ErrorsMethodArgumentResolver (org.springframework.web.method.annotation)
PathVariableMapMethodArgumentResolver (org.springframework.web.servlet.mvc.method.annotation)
AbstractNamedValueMethodArgumentResolver (org.springframework.web.method.annotation)
RequestHeaderMapMethodArgumentResolver (org.springframework.web.method.annotation)解析使用@RequestHeader注解的参数,不包括map类型
ModelMethodProcessor (org.springframework.web.method.annotation)
ModelAttributeMethodProcessor (org.springframework.web.method.annotation)
ServletResponseMethodArgumentResolver (org.springframework.web.servlet.mvc.method.annotation)
SessionStatusMethodArgumentResolver (org.springframework.web.method.annotation)

RequestParamMapMethodArgumentResolver (org.springframework.web.method.annotation)解析从request流中获取值的参数,如@RequestParam,MultipartFile,Part,默认使用时解析基本参数.

AbstractMessageConverterMethodArgumentResolver (org.springframework.web.servlet.mvc.method.annotation)
AbstractWebArgumentResolverAdapter (org.springframework.web.method.annotation)
UriComponentsBuilderMethodArgumentResolver (org.springframework.web.servlet.mvc.method.annotation)
HandlerMethodArgumentResolverComposite (org.springframework.web.method.support)
ServletRequestMethodArgumentResolver (org.springframework.web.servlet.mvc.method.annotation)
RedirectAttributesMethodArgumentResolver (org.springframework.web.servlet.mvc.method.annotation)
MatrixVariableMapMethodArgumentResolver (org.springframework.web.servlet.mvc.method.annotation)

org.springframework.web.method.support.HandlerMethodArgumentResolverComposite 组装list类型的HandlerMethodArgumentResolve


# HandlerMethodReturnValueHandler接口
MapMethodProcessor (org.springframework.web.method.annotation)
ViewNameMethodReturnValueHandler (org.springframework.web.servlet.mvc.method.annotation)
ViewMethodReturnValueHandler (org.springframework.web.servlet.mvc.method.annotation)
StreamingResponseBodyReturnValueHandler (org.springframework.web.servlet.mvc.method.annotation)
HandlerMethodReturnValueHandlerComposite (org.springframework.web.method.support)
DeferredResultMethodReturnValueHandler (org.springframework.web.servlet.mvc.method.annotation)
HttpHeadersReturnValueHandler (org.springframework.web.servlet.mvc.method.annotation)
CallableMethodReturnValueHandler (org.springframework.web.servlet.mvc.method.annotation)
ModelMethodProcessor (org.springframework.web.method.annotation)
ModelAttributeMethodProcessor (org.springframework.web.method.annotation)
ResponseBodyEmitterReturnValueHandler (org.springframework.web.servlet.mvc.method.annotation)
ModelAndViewMethodReturnValueHandler (org.springframework.web.servlet.mvc.method.annotation)
ModelAndViewResolverMethodReturnValueHandler (org.springframework.web.servlet.mvc.method.annotation)
AbstractMessageConverterMethodProcessor (org.springframework.web.servlet.mvc.method.annotation)
AsyncHandlerMethodReturnValueHandler (org.springframework.web.method.support)
AsyncTaskMethodReturnValueHandler (org.springframework.web.servlet.mvc.method.annotation)

org.springframework.web.method.support.HandlerMethodReturnValueHandlerComposite组装list类型的HandlerMethodReturnValueHandler
