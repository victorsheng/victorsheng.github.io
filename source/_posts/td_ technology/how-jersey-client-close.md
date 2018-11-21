---
title: how-jersey-client-close
abbrlink: 435546917
date: 2018-11-20 16:54:37
tags:
categories:
---
# 结论
如果不read the entity，则需要通过调用Response＃close（）手动关闭响应。其余,则会自动关闭


# 参考
https://stackoverflow.com/questions/35265534/closing-connection-in-get-request-using-jersey-client-2-22-1

# 源码
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
        if (shouldClose) {
          ReaderWriter.safelyClose(this.entityContent);
        }

      }

      return var9;
    }
  }
```