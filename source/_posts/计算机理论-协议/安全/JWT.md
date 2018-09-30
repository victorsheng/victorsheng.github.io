---
title: JWT
tags:
  - 安全
  - 认证
categories:
  - 安全
abbrlink: 464555898
date: 2018-06-05 19:50:13
---
JSON Web Token（JWT）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。

- 简洁(Compact): 可以通过URL，POST参数或者在HTTP header发送，因为数据量小，传输速度也很快

- 自包含(Self-contained)：负载中包含了所有用户所需要的信息，避免了多次查询数据库



# JWT的组成
一个JWT实际上就是一个字符串，它由三部分组成，头部、载荷与签名。

## 1.头部（Header）
JWT还需要一个头部，头部用于描述关于该JWT的最基本的信息，例如其类型以及签名所用的算法等。这也可以被表示成一个JSON对象。

```
{
  "typ": "JWT",
  "alg": "HS256"
}
```
在这里，我们说明了这是一个JWT，并且我们所用的签名算法（后面会提到）是HS256算法。

对它也要进行Base64编码，之后的字符串就成了JWT的Header（头部）。

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```

## 2.载荷（Payload）
我们先将上面的添加好友的操作描述成一个JSON对象。其中添加了一些其他的信息，帮助今后收到这个JWT的服务器理解这个JWT。
```
{
    "iss": "John Wu JWT",
    "iat": 1441593502,
    "exp": 1441594722,
    "aud": "www.example.com",
    "sub": "jrocket@example.com",
    "from_user": "B",
    "target_user": "A"
}
```

这里面的前五个字段都是由JWT的标准所定义的。

- iss: 该JWT的签发者
- sub: 该JWT所面向的用户
- aud: 接收该JWT的一方
- exp(expires): 什么时候过期，这里是一个Unix时间戳
- iat(issued at): 在什么时候签发的
这些定义都可以在标准中找到。

将上面的JSON对象进行**base64编码**可以得到下面的字符串。这个字符串我们将它称作JWT的Payload（载荷）。



## 3.签名
将上面拼接完的字符串用HS256算法进行加密。在加密的时候，我们还需要提供一个密钥（secret）。如果我们用mystar作为密钥的话，那么就可以得到我们加密后的内容

最后将这一部分签名也拼接在被签名的字符串后面，我们就得到了完整的JWT


### 签名的目的
服务器应用在接受到JWT后，会首先对头部和载荷的内容用同一算法再次签名。那么服务器应用是怎么知道我们用的是哪一种算法呢？别忘了，我们在JWT的头部中已经用alg字段指明了我们的加密算法了。

如果服务器应用对头部和载荷再次以同样方法签名之后发现，自己计算出来的签名和接受到的签名不一样，那么就说明这个Token的内容被别人动过的，我们应该拒绝这个Token，返回一个HTTP 401 Unauthorized响应。


## 信息会暴露？
是的。

所以，在JWT中，不应该在载荷里面加入任何敏感的数据。在上面的例子中，我们传输的是用户的User ID。这个值实际上不是什么敏感内容，一般情况下被知道也是安全的。

但是像密码这样的内容就不能被放在JWT中了。如果将用户的密码放在了JWT中，那么怀有恶意的第三方通过Base64解码就能很快地知道你的密码了。

# jwt校验
## 规范:
https://tools.ietf.org/html/rfc7523#section-2.1

## mitreid中对jwt格式的id_token的校验逻辑
https://github.com/mitreid-connect/simple-web-app

- 从seesion中取issuer
- 访问issuer上的jwk接口
- 通过公钥校验jwt
- 校验jwt的issuer字段
- 校验jwt的issuer字段和从seesion中取issuer字段
- 校验jwt的issue时间是否为空
- 校验jwt的issue时间是否在当前时间+窗口时间之后
- 校验jwt的Audience字段是否为空
- 校验jwt的Audience字段是否包含clientId
- 校验jwt的nonce字段是否为空
- 校验jwt的nonce字段是否与seesion中的nonce相同


# jwt校验公钥获取方式
- 本地获取
- jwk接口获取


# 参考

http://blog.leapoahead.com/2015/09/07/user-authentication-with-jwt/