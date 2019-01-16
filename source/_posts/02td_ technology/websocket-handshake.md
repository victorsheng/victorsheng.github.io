title: websocket-handshake
abbrlink: 4027927501
date: 2018-11-15 15:37:50
tags:
categories:
---
# 1、发送握手请求
此时的连接状态是CONNECTING，客户端需要提供host、port、resource-name和一个是否是安全连接的标记，也就是一个WebSocket URI。

客户端发送的一个到服务器端握手请求如下：

 

        GET /chat HTTP/1.1
        Host: server.example.com
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
        Origin: http://example.com
        Sec-WebSocket-Protocol: chat, superchat
        Sec-WebSocket-Version: 13

这个升级的HTTP请求头中的字段顺序是可以随便的。与普通HTTP请求相比多了一些字段。

- Sec-WebSocket-Protocol：字段表示客户端可以接受的子协议类型，也就是在Websocket协议上的应用层协议类型。上面可以看到客户端支持chat和superchat两个应用层协议，当服务器接受到这个字段后要从中选出一个协议返回给客户端。
- Upgrade：告诉服务器这个HTTP连接是升级的Websocket连接。
- Connection：告知服务器当前请求连接是升级的。
- Origin：该字段是用来防止客户端浏览器使用脚本进行未授权的跨源攻击，这个字段在WebSocket协议中非常重要。服务器要根据这个字段判断是否接受客户端的Socket连接。可以返回一个HTTP错误状态码来拒绝连接。
- Sec-WebSocket-Key：为了表示服务器同意和客户端进行Socket连接，服务器端需要使用客户端发送的这个Key进行校验，然后返回一个校验过的字符串给客户端，客户端验证通过后才能正式建立Socket连接。服务器验证方法是：首先进行 Key + 全局唯一标示符（GUID）“258EAFA5-E914-47DA-95CA-C5AB0DC85B11”连接起来，然后将连接起来的字符串使用SHA-1哈希加密，再进行base64加密，将得到的字符串返回给客户端作为握手依据。其中GUID是一个对于不识别WebSocket的网络端点不可能使用的字符串。


发送请求的要求：
- 请求的WebSocket URI必须要是定义的有效的URI。
- 如果客户端已经有一个WebSocket连接到远程服务器端，不论是否是同一个服务器，客户端必须要等待上一个连接关闭后才能发送新的连接请求，也就是同一客户端一次只能存在一个WebSocket连接。如果想同一个服务器有多个连接，客户端必须要串行化进行。如果客户端检测到多个到不同服务器的连接，应该限制一个最大连接数，在web浏览器中应该设定最多可以打开的标签页的数目。这样可以防止到远程服务器的DDOS攻击，但这是对到多个服务器的连接，如果是到同一个服务器连接，并没有数目限制。
- 如果使用了代理服务器，那么客户端建立连接的时候需要告知代理服务器向目标服务器打开TCP连接。
- 如果连接没有打开，一定是某一方出现错误，此时客户端必须要关闭再次连接的尝试。
- 连接建立后，握手必须要是一个有效的HTTP请求
- 请求的方式必须是GET，HTTP协议的版本至少是1.1
- Upgrade字段必须包含而且必须是"websocket"，Connection字段必须内容必须是“Upgrade”
- Sec-Websocket-Version必须，而且必须是13


```
GET ws://172.22.9.1:8301/provier-websocket/389/qjm0qj13/websocket HTTP/1.1
Host: 172.22.9.1:8301
Connection: Upgrade
Pragma: no-cache
Cache-Control: no-cache
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36
Upgrade: websocket
Origin: http://localhost:9318
Sec-WebSocket-Version: 13
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7
Sec-WebSocket-Key: UvyGH6jvbDySVThkGrtHUQ==
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```


# 2、返回握手应答
服务器返回正确的相应头后，客户端验证后将建立连接，此时状态为OPEN。服务器响应头如下：

```
        HTTP/1.1 101 Switching Protocols
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
        Sec-WebSocket-Protocol: chat
```

响应头握手过程中是服务器返回的是否同意握手的依据。
- 首行返回的是HTTP/1.1协议版本和状态码101，表示变换协议（Switching Protocol）
- Upgrade 和 Connection：这两个字段是服务器返回的告知客户端同意使用升级并使用websocket协议，用来完善HTTP升级响应
- Sec-WebSocket-Accept：服务器端将加密处理后的握手Key通过这个字段返回给客户端表示服务器同意握手建立连接。
- Sec-Websocket-Procotol：服务器选择的一个应用层协议。
上述响应头字段被客户端浏览器解析，如果验证到Sec-WebSocket-Accept字段的信息符合要求就会建立连接，同时就可以发送WebSocket的数据帧了。如果该字段不符合要求或者为空或者HTTP状态码不为101，就不会建立连接。


服务器端响应步骤：
- 解析握手请求头：获取握手依据Key并进行处理，检测HTTP的GET请求和版本是否准确，Host字段是否有权限，Upgrade字段中websocket是一个与大小写无关的ASCII字符串，Connection字段是一个大小写无关的"Upgrade"ASCII字符串，Websocket协议版本必须为13，其他的关于Origin、Protocol和Extensions可选。
- 发送握手响应头：检测是否是wss协议连接，如果是就是用TLS握手连接，否则就是普通连接。服务器可以添加额外的验证信息到客户端进行验证。当进行一系列验证之后，服务器必须返回一个有效的HTTP响应头。响应头中每一行一个字段，结束必须为“\r\n”，使用的ABNF语法。

```
HTTP/1.1 101 Switching Protocols
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers
Access-Control-Allow-Origin: http://localhost:9318
Access-Control-Allow-Credentials: true
Sec-WebSocket-Extensions: permessage-deflate
Connection: Upgrade
Sec-WebSocket-Accept: bNFDWb1Xo6rILPXSpwTjabFSYBM=
Server: Jetty(9.4.9.v20180320)
Upgrade: WebSocket
```


# nginx 配置
```
# 在http上下文中增加如下配置，确保Nginx能处理正常http请求。
http{
  map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
  }

  upstream websocket {
    #ip_hash;
    server localhost:8010;  
    server localhost:8011;
  }






# 以下配置是在server上下文中添加，location指用于websocket连接的path。

  server {
    listen       80;
    server_name localhost;
    access_log /var/log/nginx/yourdomain.log;

    location / {
      proxy_pass http://websocket;
      proxy_read_timeout 300s;

      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
		}
	}
}

```

# 参考
https://www.cnblogs.com/oshyn/p/3574497.html
https://www.hi-linux.com/posts/42176.html