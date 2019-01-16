---
title: jersey-rest-client是否需要手动关闭
abbrlink: 435546917
date: 2018-11-20 16:54:37
tags:
categories:
---
# 背景
应用系统本身对资源的的不恰当使用、配置，引起内存使用持续增加，最终导致 JVM Heap Memory 被耗尽
因此,在使用任何新技术的时候,对重量级的资源释放的问题,要时刻谨记


# 结论
如果不read the entity，则需要通过调用Response＃close（）手动关闭响应。其余,则会自动关闭


# 参考
https://stackoverflow.com/questions/35265534/closing-connection-in-get-request-using-jersey-client-2-22-1

# 源码分析

## 调用栈
```
readEntity:849, InboundMessageContext (org.glassfish.jersey.message.internal)
readEntity:808, InboundMessageContext (org.glassfish.jersey.message.internal)
readEntity:321, ClientResponse (org.glassfish.jersey.client)
translate:870, JerseyInvocation (org.glassfish.jersey.client)
lambda$invoke$1:767, JerseyInvocation (org.glassfish.jersey.client)
call:-1, 604409455 (org.glassfish.jersey.client.JerseyInvocation$$Lambda$229)
process:316, Errors (org.glassfish.jersey.internal)
process:298, Errors (org.glassfish.jersey.internal)
process:229, Errors (org.glassfish.jersey.internal)
runInScope:414, RequestScope (org.glassfish.jersey.process.internal)
invoke:765, JerseyInvocation (org.glassfish.jersey.client)
method:428, JerseyInvocation$Builder (org.glassfish.jersey.client)
get:324, JerseyInvocation$Builder (org.glassfish.jersey.client)
service层
```

## 自动关闭资源
org.glassfish.jersey.message.internal.InboundMessageContext
```
public <T> T readEntity(Class<T> rawType, Type type, Annotation[] annotations, PropertiesDelegate propertiesDelegate) {
    boolean buffered = this.entityContent.isBuffered();
    if (buffered) {
      this.entityContent.reset();
    }

    this.entityContent.ensureNotClosed();
    if (this.workers == null) {
      return null;
    } else {
      MediaType mediaType = this.getMediaType();
      mediaType = mediaType == null ? MediaType.APPLICATION_OCTET_STREAM_TYPE : mediaType;
      boolean shouldClose = !buffered;

      Object var9;
      try {
        T t = this.workers.readFrom(rawType, type, annotations, mediaType, this.headers, propertiesDelegate, this.entityContent.getWrappedStream(), (Iterable)(this.entityContent.hasContent() ? this.getReaderInterceptors() : Collections.emptyList()), this.translateNce);
        shouldClose = shouldClose && !(t instanceof Closeable) && !(t instanceof Source);
        var9 = t;
      } catch (IOException var13) {
        throw new ProcessingException(LocalizationMessages.ERROR_READING_ENTITY_FROM_INPUT_STREAM(), var13);
      } finally {
        //如果符合条件,则自动关闭
        if (shouldClose) {
          ReaderWriter.safelyClose(this.entityContent);
        }

      }

      return var9;
    }
  }
```

# 小结
jesery作为开发框架,方便了开发人员的日常开发,一般情况都会进行readEntity这样的操作,无需进行手动关闭;
但作为开发人员,也需要明白底层原理,而不仅是简单的使用,这样才会有进一步提高