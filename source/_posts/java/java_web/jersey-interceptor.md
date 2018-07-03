title: jersey-interceptor
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