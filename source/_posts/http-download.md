---
title: http-download
abbrlink: 2868668734
date: 2018-11-28 11:21:56
tags:
categories:
---

# Content-Disposition header的作用
## 响应
在常规的HTTP应答中，Content-Disposition 消息头指示回复的内容该以何种形式展示，是以内联的形式（即网页或者页面的一部分），还是以附件的形式下载并保存到本地。
```
Content-Disposition: inline
Content-Disposition: attachment
Content-Disposition: attachment; filename="filename.jpg"
```

## 请求

在multipart/form-data类型的应答消息体中， Content-Disposition 消息头可以被用在multipart消息体的子部分中，用来给出其对应字段的相关信息。各个子部分由在Content-Type 中定义的分隔符分隔。用在消息体自身则无实际意义。
```
Content-Disposition: form-data
Content-Disposition: form-data; name="fieldName"
Content-Disposition: form-data; name="fieldName"; filename="filename.jpg"
```


由于HTTP协议规定，通信内容使用US ASCII编码，就是只能使用英文字符集。若要使用其他字符集，必须根据RFC3986使用百分号将字符串编码。

```
Content-Disposition: attachment; filename=filename.ext
Content-Disposition: attachment; filename*=charset'lang'encoded-filename.ext
```

## 示例
### (响应)保存为附件
```
200 OK
Content-Type: text/html; charset=utf-8
Content-Disposition: attachment; filename="cool.html"
Content-Length: 22

<HTML>Save me!</HTML>
```
### (请求)文件上传
```
POST /test.html HTTP/1.1
Host: example.org
Content-Type: multipart/form-data;boundary="boundary"

--boundary
Content-Disposition: form-data; name="field1"

value1
--boundary
Content-Disposition: form-data; name="field2"; filename="example.txt"

value2
--boundary--
```



# Content-Type
浏览器对已知类型的文件（如jpg、pdf、txt等）直接在浏览器内打开，我们通过设置http头中的 Content-Type 来改变浏览器认知的文件类型。 这里把Content-Type 设置为octet-stream,也就是二进制文件流。这样浏览器就会直接打开文件，而不是在浏览器内打开。

Content-Type: application/octet-stream

```
Content-Type: text/html; charset=utf-8
Content-Type: multipart/form-data; boundary=something
```


# 参考
https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition
https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type