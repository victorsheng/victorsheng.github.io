---
title: post-multipart-form
date: 2019-01-07 17:09:46
tags:
categories:
---


## 服务端处理代码:
```
        for (Map.Entry<String, List<FormDataBodyPart>> entry : formItemMap.entrySet()) {
            for (FormDataBodyPart formDataBodyPart : entry.getValue()) {
                if (StringUtils.isNotEmpty(formDataBodyPart.getContentDisposition().getFileName())) {
                    //按文件处理
                    parseFormFileField(taskId, formDataBodyPart);
                } else {
                    //按字段处理
                    insertInputParameter(taskId, entry.getKey(), formDataBodyPart.getValue());
                }
            }
        }
```

## 从postman发出的正常请求
```
POST /demo/file?= HTTP/1.1
Host: localhost:8084
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
X-OPAL-token: eyJpc3MiOiJ0ZCIsImF1ZCI6IjExIiwiaWF0IjoxNTM5OTMwNDc0LCJleHAiOjYwMDAwMDE1Mzk5MzA0MTQsImFsZyI6IlJTMjU2In0.RstRzYRJDneQ88BmFlgDD8l3m4Z9CM9nmqUaKWLXrRlYzuGdig5HjqXYDVMB1AWRBO-PNBhIaF6qHnBP_ms6jFGzS5iy1c7Kws-IJiAvB0OuWu7az0bJpIbcRM90CNydXPRGkra0MWoti8-sLWywFHmxkNhAEQZDrGOwZkIL_IXnUoUiDKyO21djjucqRNS9EU0Xp0gdJ1ItbRlDqXqlRhdx2sm05AZUJJ_ozWb9br_pLeOG098lNbl3magzZoFsojZE-UW_-6Bw8Sl7jw2kO5ciPwJ5xEZjbQ8Kntu_o-9aveeD2p3pApQuBXhYjcefV9T6KSuuzHsFLMmpDfyPxA
cache-control: no-cache
Postman-Token: 0a1c5b63-7545-4f5d-a3f9-71f54583f584

Content-Disposition: form-data; name="bdId"

3fecc43cd35e4a128992ecf28e6d91a7

Content-Disposition: form-data; name="file"; filename="/LICENSE.txt


------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

## 错误代码发出的的请求
```
1 > POST /demo/file
1 > Content-Type: multipart/form-data
--Boundary_1_861038724_1546851949618
Content-Type: text/plain
Content-Disposition: form-data; name="Params"

{"name":"zuoye"}
--Boundary_1_861038724_1546851949618
Content-Type: text/plain
Content-Disposition: form-data; name="file"

blabla
```

**在我的请求中缺少了filename字段和Content-Type字段,so服务端报错**

# jersery客户端一般的文件上传示例
https://howtodoinjava.com/jersey/jersey-file-upload-example/
```
public static void main(String[] args) throws IOException
{
    final Client client = ClientBuilder.newBuilder().register(MultiPartFeature.class).build();
 
    final FileDataBodyPart filePart = new FileDataBodyPart("file", new File("C:/temp/sample.pdf"));
    FormDataMultiPart formDataMultiPart = new FormDataMultiPart();
    final FormDataMultiPart multipart = (FormDataMultiPart) formDataMultiPart.field("foo", "bar").bodyPart(filePart);
      
    final WebTarget target = client.target("http://localhost:8080/JerseyDemos/rest/upload/pdf");
    final Response response = target.request().post(Entity.entity(multipart, multipart.getMediaType()));
     
    //Use response object to verify upload success
     
    formDataMultiPart.close();
    multipart.close();
}
```
或者

```
      FormDataBodyPart file = new FormDataBodyPart(
          FormDataContentDisposition.name("file").fileName("param.txt").build(), download,
          MediaType.TEXT_PLAIN_TYPE);
      multipart.bodyPart(file);
```
**字段使用field方法,文件使用bodyPart方法**

# 这种判断依据是否合理:
**我认为是合理的,在没有指定哪个key对应的是filed还是file情况下,根据filename判断是合理的,当然Content-Type也可以**
从firfox发出的请求
```

POST / HTTP/1.1
[[ Less interesting headers ... ]]
Content-Type: multipart/form-data; boundary=---------------------------735323031399963166993862150
Content-Length: 834

-----------------------------735323031399963166993862150
Content-Disposition: form-data; name="text1"

text default
-----------------------------735323031399963166993862150
Content-Disposition: form-data; name="text2"

aωb
-----------------------------735323031399963166993862150
Content-Disposition: form-data; name="file1"; filename="a.txt"
Content-Type: text/plain

Content of a.txt.

-----------------------------735323031399963166993862150
Content-Disposition: form-data; name="file2"; filename="a.html"
Content-Type: text/html

<!DOCTYPE html><title>Content of a.html.</title>

-----------------------------735323031399963166993862150
Content-Disposition: form-data; name="file3"; filename="binary"
Content-Type: application/octet-stream

aωb
-----------------------------735323031399963166993862150--
```

# 补充
**Multipart的编码可以使多种类型的,对于各个部分的可以独自进行声明,对于field,是使用URLEncoding进行编码的**