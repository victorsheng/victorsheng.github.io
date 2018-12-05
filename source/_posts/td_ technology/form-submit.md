---
title: form提交的几种方式
abbrlink: 1177069295
date: 2018-11-20 11:08:24
tags:
categories:
---
# 背景
![](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/requestBuilderForm.png)
一直使用postman作为restful接口的调试工具,但是针对post方法的几种类型,始终不明白其含义,今天彻底了解了下

# form提交的来源
**html页面上的form表单**
```
<form action="/handling-page" method="post">
  <div>
    <label for="name">用户名：</label>
    <input type="text" id="name" name="user_name" />
  </div>
  <div>
    <label for="passwd">密码：</label>
    <input type="password" id="passwd" name="user_passwd" />
  </div>
  <div>
    <input type="submit" id="submit" name="submit_button" value="提交" />
  </div>
</form>
```

## GET
提交的数据格式跟<form>元素的method属性有关。该属性指定了提交数据的 HTTP 方法。如果是 GET 方法，所有键值对会以 URL 的查询字符串形式，提交到服务器，比如/handling-page?user_name=张三&user_passwd=123&submit_button=提交。下面就是 GET 请求的 HTTP 头信息。

```
GET /handling-page?user_name=张三&user_passwd=123&submit_button=提交
Host: example.com
```

## POST
如果是 POST 方法，所有键值对会连接成一行，作为 HTTP 请求的数据体发送到服务器，比如user_name=张三&user_passwd=123&submit_button=提交。下面就是 POST 请求的头信息。

```
POST /handling-page HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 74

user_name=张三&user_passwd=123&submit_button=提交
```
# FormData(XMLHttpRequest.sendf方法)
表单数据以键值对的形式向服务器发送，这个过程是浏览器自动完成的。但是有时候，我们希望通过脚本完成过程，构造和编辑表单键值对，然后通过XMLHttpRequest.send()方法发送。浏览器原生提供了 FormData 对象来完成这项工作。

```
//获取表单对象
var myForm = document.getElementById('myForm');
var formData = new FormData(myForm);

//提交表单
var xhr = new XMLHttpRequest();
xhr.open('POST', '/handler/new', true);
xhr.onload = function () {
  if (xhr.status !== 200) {
    console.log('An error occurred!');
  }
};

xhr.send(formData);

```

## enctype 属性
表单能够用四种编码，向服务器发送数据。编码格式由表单的enctype属性决定。

### 1.脚本提交的post方法:GET 方法
如果表单使用GET方法发送数据，enctype属性无效。
数据将以 URL 的查询字符串发出。
```
?foo=bar&baz=The%20first%20line.%0AThe%20second%20line.
```
### 2.脚本提交的post方法:application/x-www-form-urlencoded
**如果表单用POST方法发送数据，并省略enctype属性，那么数据以application/x-www-form-urlencoded格式发送（默认值）。**

发送的 HTTP 请求如下。
```
Content-Type: application/x-www-form-urlencoded

foo=bar&baz=The+first+line.%0D%0AThe+second+line.%0D%0A
```
上面代码中，数据体里面的%0D%0A代表换行符（\r\n）。

![](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/requestBuilderUrlEncoded.png)

### 3.text/plain
**如果表单使用POST方法发送数据，enctype属性为text/plain，那么数据将以纯文本格式发送。**

```
Content-Type: text/plain

foo=bar
baz=The first line.
The second line.
```
对应postman的raw,可以自定义格式
![](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/58960775.png)

### 4.multipart/form-data

**如果表单使用POST方法，enctype属性为multipart/form-data，那么数据将以混合的格式发送。**

```
Content-Type: multipart/form-data; boundary=---------------------------314911788813839

-----------------------------314911788813839
Content-Disposition: form-data; name="foo"

bar
-----------------------------314911788813839
Content-Disposition: form-data; name="baz"

The first line.
The second line.

-----------------------------314911788813839--

```
这种格式也是文件上传的格式。

postman中对应的form-data
![](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/requestBuilderForm.png)


# 参考
https://wangdoc.com/javascript/bom/form.html
https://www.getpostman.com/docs/v6/postman/sending_api_requests/requests