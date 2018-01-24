title: springboot配置FastJsonHttpMessageConverter(原理篇)
tags:
  - springboot
  - http
  - springMVC
categories: []
date: 2018-01-22 17:18:00
---
# springboot如何配置DispatcherServlet?
org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration中完成的

# 在springboot项目中,进行springmvc配置的新的方式
```
@Configuration
public class SpringMvcConfigure extends WebMvcConfigurerAdapter {

    @Bean
    public InternalResourceViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        // viewResolver.setViewClass(JstlView.class); // 这个属性通常并不需要手动配置，高版本的Spring会自动检测
        return viewResolver;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new Interceptor1()).addPathPatterns("/**");
        registry.addInterceptor(new Interceptor2()).addPathPatterns("/users").addPathPatterns("/users/**");
        super.addInterceptors(registry);
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        // addResourceHandler指的是访问路径，addResourceLocations指的是文件放置的目录  
        registry.addResourceHandler("/**").addResourceLocations("classpath:/res/");
    }
}
```


# HTTP序列化/反序列化
@RestController中有@ResponseBody，可以帮我们把User序列化到resp.body中。@RequestBody可以帮我们把req.body的内容转化为User对象。如果是开发Web应用，一般这两个注解对应的就是Json序列化和反序列化的操作。这里实际上已经体现了Http序列化/反序列化这个过程，只不过和普通的对象序列化有些不一样，Http序列化/反序列化的层次更高，属于一种Object2Object之间的转换。

有过Netty使用经验的对这个应该比较了解，Netty中的Decoder和Encoder就有两种基本层次，层次低的一种是Byte <---> Message，二进制与程序内部消息对象之间的转换，就是常见的序列化/反序列化；另外一种是 Message <---> Message，程序内部对象之间的转换，比较高层次的序列化/反序列化。

Http协议的处理过程，TCP字节流 <---> HttpRequest/HttpResponse <---> 内部对象，就涉及这两种序列化。在springmvc中第一步已经由Servlet容器（tomcat等等）帮我们处理了，第二步则主要由框架帮我们处理。上面所说的Http序列化/反序列化就是指的这第二个步骤，它是controller层框架的核心功能之一，有了这个功能，就能大大减少代码量，让controller的逻辑更简洁清晰，就像上面示意的代码那样，方法中只有一行代码。

spirngmvc进行第二步操作，也就是Http序列化和反序列化的核心是HttpMessageConverter。用过老版本springmvc的可能有些印象，那时候需要在xml配置文件中注入MappingJackson2HttpMessageConverter这个类型的bean，告诉springmvc我们需要进行Json格式的转换，它就是HttpMessageConverter的一种实现。
![upload successful](/images/pasted-39.png)
## 几种实现
- json:gson,fastjson,jackson
- Protobuf
- java序列化

# HttpMessageConverter优先级
另外有很重要的一点需要说明一下，springmvc可以同时配置多个Converter，根据一定的规则（主要是Content-Type、Accept、controller方法的consumes/produces、Converter.mediaType以及Converter的排列顺序这四个属性）来选择到底是使用哪一个，这使得springmvc能够一个接口支持多种报文格式。

# 默认的converters
```
ByteArrayHttpMessageConverter – converts byte arrays
StringHttpMessageConverter – converts Strings
ResourceHttpMessageConverter – converts org.springframework.core.io.Resource for any type of octet stream
SourceHttpMessageConverter – converts javax.xml.transform.Source
FormHttpMessageConverter – converts form data to/from a MultiValueMap<String, String>.
Jaxb2RootElementHttpMessageConverter – converts Java objects to/from XML (added only if JAXB2 is present on the classpath)
MappingJackson2HttpMessageConverter – converts JSON (added only if Jackson 2 is present on the classpath)
MappingJacksonHttpMessageConverter – converts JSON (added only if Jackson is present on the classpath)
AtomFeedHttpMessageConverter – converts Atom feeds (added only if Rome is present on the classpath)
RssChannelHttpMessageConverter – converts RSS feeds (added only if Rome is present on the classpath)
```
![upload successful](/images/pasted-40.png)

# 源码中的调用栈
```
com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter.write(FastJsonHttpMessageConverter.java:185)
	  at org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodProcessor.writeWithMessageConverters(AbstractMessageConverterMethodProcessor.java:231)
	  at org.springframework.web.servlet.mvc.method.annotation.HttpEntityMethodProcessor.handleReturnValue(HttpEntityMethodProcessor.java:203)
	  at org.springframework.web.method.support.HandlerMethodReturnValueHandlerComposite.handleReturnValue(HandlerMethodReturnValueHandlerComposite.java:81)
	  at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:113)
	  at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:827)
	  at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:738)
	  at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:85)
	  at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:967)
	  at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:901)
	  at 
```
# 常见的contentType

- 1.text/html
- 2.text/plain
- 3.text/css
- 4.text/javascript
- 5.application/x-www-form-urlencoded
- 6.multipart/form-data
- 7.application/json
- 8.application/xml
