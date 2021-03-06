---
title: jersey-interceptor
abbrlink: 4167561562
date: 2018-07-03 17:40:34
tags:
categories:
---
# jersey中的interceptor
https://www.igorkromin.net/index.php/2017/10/19/jersey-jax-rs-filters-and-interceptors-execution-order-for-a-simple-get-request/

https://www.igorkromin.net/fp-content/images/programming/jersey/jerseyflow_get1.png
![upload successful](/images/pasted-204.png)
图中显示了从request到response ,jersey开放的可以进行拦截的地方


https://stackoverflow.com/questions/35727497/is-there-an-interceptor-in-jersey-similar-to-that-of-springs-handlerinterceptor

ReaderInterceptor - This provides just Annotations of the Class. No access to request/response.
ContainerRequestFilter - Provides Request Object but no Annotations/Response.
ContainerResponseFilter - Gives Request & Response. But after the web-service/Method is invoked.

# 小结
ContainerRequestFilter 接口中的参数中获取的request,只能得到request相关的http请求例如方法,header等
对于handler级别的注解,希望在jersey框架中得到ResourceInfo界别的信息,获取method,获取method上面的注解,以判断是否需要进行业务处理

# 解决方案:

## @Context
使用@Context可以使用ServletConfig，ServletContext，HttpServletRequest和HttpServletResponse。

## 使用ContainerRequestFilter+@Context来解决

```

@Provider
@RequestSignVerify
public class SignResourceFilter implements ContainerResponseFilter {

  @Context
  private ResourceInfo resourceInfo;

  @Context
  private HttpServletRequest servletRequest;

  @Autowired
  private HttpSignature signatureVerifier;


  public void setSignatureVerifier(HttpSignature signatureVerifier) {
    this.signatureVerifier = signatureVerifier;
  }

  @Override
  public void filter(ContainerRequestContext requestContext,
      ContainerResponseContext responseContext) throws IOException {

    Method resourceMethod = resourceInfo.getResourceMethod();
    if (null != resourceMethod) {
      RequestSignVerify signVerify = resourceMethod.getAnnotation(RequestSignVerify.class);
      if (signVerify != null) {
        HttpRequest httpRequest = ServletSignedRequest.from(servletRequest);
        try {
          if (this.signatureVerifier.verify(httpRequest)) {

          } else {
            responseContext.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
          }
        } catch (GeneralSecurityException e) {
          e.printStackTrace();
          throw new IOException(e);
        }
      }
    }

  }


}

```

# Filter and interceptor执行顺序

https://jersey.github.io/documentation/latest/filters-and-interceptors.html

- Client request invoked: The POST request with attached entity is built on the client and invoked.
- ClientRequestFilters: client request filters are executed on the client and they manipulate the request headers.
- Client WriterInterceptor: As the request contains an entity, writer interceptor registered on the client is executed before a MessageBodyWriter is executed. It wraps the entity output stream with the GZipOutputStream.
- Client MessageBodyWriter: message body writer is executed on the client which serializes the entity into the new GZipOutput stream. This stream zips the data and sends it to the "wire".
- Server: server receives a request. Data of entity is compressed which means that pure read from the entity input stream would return compressed data.
- Server pre-matching ContainerRequestFilters: ContainerRequestFilters are executed that can manipulate resource method matching process.
- Server: matching: resource method matching is done.
- Server: post-matching ContainerRequestFilters: ContainerRequestFilters post matching filters are executed. This include execution of all global filters (without name binding) and filters name-bound to the matched method.
- Server ReaderInterceptor: reader interceptors are executed on the server. The GZIPReaderInterceptor wraps the input stream (the stream from the "wire") into the GZipInputStream and set it to context.
- Server MessageBodyReader: server message body reader is executed and it deserializes the entity from new GZipInputStream (get from the context). This means the reader will read unzipped data and not the compressed data from the "wire".
- Server resource method is executed: the deserialized entity object is passed to the matched resource method as a parameter. The method returns this entity as a response entity.
- Server ContainerResponseFilters are executed: response filters are executed on the server and they manipulate the response headers. This include all global bound filters (without name binding) and all filters name-bound to the resource method.
- Server WriterInterceptor: is executed on the server. It wraps the original output stream with a new GZIPOuptutStream. The original stream is the stream that "goes to the wire" (output stream for response from the underlying server container).
- Server MessageBodyWriter: message body writer is executed on the server which serializes the entity into the GZIPOutputStream. This stream compresses the data and writes it to the original stream which sends this compressed data back to the client.
- Client receives the response: the response contains compressed entity data.
- Client ClientResponseFilters: client response filters are executed and they manipulate the response headers.
- Client response is returned: the javax.ws.rs.core.Response is returned from the request invocation.
- Client code calls response.readEntity(): read entity is executed on the client to extract the entity from the response.
- Client ReaderInterceptor: the client reader interceptor is executed when readEntity(Class) is called. The interceptor wraps the entity input stream with GZIPInputStream. This will decompress the data from the original input stream.
- Client MessageBodyReaders: client message body reader is invoked which reads decompressed data from GZIPInputStream and deserializes the entity.
- Client: The entity is returned from the readEntity().

# 类型
## 全局过滤器
## Name binding

过滤器和拦截器可以是名称绑定的。 名称绑定是一种概念，它允许向JAX-RS运行时说明仅针对特定资源方法执行特定过滤器或拦截器。 当过滤器或拦截器仅限于特定的资源方法时，我们说它是名称绑定的。 没有这种限制的过滤器和拦截器称为全局。

可以使用@NameBinding批注将过滤器或拦截器分配给资源方法。 注释用作应用于提供者和资源方法的其他用户实现的注释的元注释。

**效率高,需提前绑定**

```
...
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.util.zip.GZIPInputStream;
 
import javax.ws.rs.GET;
import javax.ws.rs.NameBinding;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
...
 
 
// @Compress annotation is the name binding annotation
@NameBinding
@Retention(RetentionPolicy.RUNTIME)
public @interface Compress {}
 
 
@Path("helloworld")
public class HelloWorldResource {
 
    @GET
    @Produces("text/plain")
    public String getHello() {
        return "Hello World!";
    }
 
    @GET
    @Path("too-much-data")
    @Compress
    public String getVeryLongString() {
        String str = ... // very long string
        return str;
    }
}
 
// interceptor will be executed only when resource methods
// annotated with @Compress annotation will be executed
@Compress
public class GZIPWriterInterceptor implements WriterInterceptor {
    @Override
    public void aroundWriteTo(WriterInterceptorContext context)
                    throws IOException, WebApplicationException {
        final OutputStream outputStream = context.getOutputStream();
        context.setOutputStream(new GZIPOutputStream(outputStream));
        context.proceed();
    }
}
```

## Dynamic binding

动态绑定是一种如何以动态方式将过滤器和拦截器分配给资源方法的方法。 前一章中的名称绑定使用静态方法，对绑定的更改需要更改源代码和重新编译。 使用动态绑定，您可以实现在应用程序初始化时定义绑定的代码。 以下示例显示了如何实现动态绑定。

**效率低,实时绑定,更灵活**

```
public class CompressionDynamicBinding implements DynamicFeature {
 
    @Override
    public void configure(ResourceInfo resourceInfo, FeatureContext context) {
        if (HelloWorldResource.class.equals(resourceInfo.getResourceClass())
                && resourceInfo.getResourceMethod()
                    .getName().contains("VeryLongString")) {
            context.register(GZIPWriterInterceptor.class);
        }
    }
}
```