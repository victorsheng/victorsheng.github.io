title: websockt学习
abbrlink: 2779190175
date: 2018-10-10 09:42:10
tags:
categories:
---
# 概念
WebSocket是一种在单个TCP连接上进行全双工通信的协议。WebSocket通信协议于2011年被IETF定为标准RFC 6455，并由RFC7936补充规范。WebSocket API也被W3C定为标准。

WebSocket使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在WebSocket API中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

# 对比1:对比websockets和http请求
https://stackoverflow.com/questions/14703627/websockets-protocol-vs-http

HTTP是一个请求-响应协议。客户机(浏览器)想要什么，服务器就给它什么。这是。如果数据客户机想要的数据很大，服务器可能会发送流数据来消除不必要的缓冲区问题。这里的主要需求或问题是如何从客户端发出请求以及如何响应他们请求的资源(hybertext)。这就是HTTP的优势所在。

**In HTTP, only client request. Server only responds.**

WebSocket不是只有客户端可以请求的请求-响应协议。它是一个套接字(非常类似于TCP套接字)。意味着一旦连接打开，任何一方都可以发送数据，直到关闭TCP连接。它就像一个普通的插座。与TCP套接字唯一的区别是websocket可以在web上使用。在web中，对于普通的套接字有很多限制。大多数防火墙会阻止HTTP使用的其他端口80和433。代理和中介也会有问题。所以为了使协议更容易部署到现有的基础设施websocket使用HTTP握手来升级。这意味着当第一次连接要打开时，客户端发送HTTP请求告诉服务器“那不是HTTP请求，请升级到websocket协议”。

一旦服务器理解了请求并升级到websocket协议，HTTP协议就不再适用了。

所以我的答案是没有一个比另一个更好。它们是完全不同的。

为什么要实现它而不是更新http协议?
我们可以把所有东西都命名为HTTP。但是我们呢?如果它们是两个不同的东西，我更喜欢两个不同的名字。


# 对比2:拉与推之间的性能比较,http相当于拉取,空闲时浪费;websock相当于推,高效
HTTP Polling, HTTP Long Polling & WebSockets
https://www.devdummy.com/2017/12/http-polling-http-long-polling.html

# 对比3:
## 1)为什么WebSockets协议更好?客户端发送消息节省http头信息

WebSockets更适合于涉及低延迟通信的情况，特别是对于客户端到服务器消息的低延迟。对于服务器到客户端数据，使用长期连接和分块传输可以获得相当低的延迟。但是，这无助于客户机到服务器的延迟，因为需要为每个客户机到服务器的消息建立新的连接。

对于实际的HTTP浏览器连接来说，48字节的HTTP握手是不现实的，因为在实际的HTTP浏览器连接中，作为请求(双向)的一部分(包括许多头文件和cookie数据)发送的数据通常有几千字节。下面是一个使用Chrome的请求/响应示例:

示例请求(包括cookie数据2800字节，没有cookie数据490字节):

HTTP和WebSockets都有同等大小的初始连接握手，但是对于WebSocket连接，初始握手只执行一次，然后小消息只有6字节的开销(头消息2字节，掩码值4字节)。延迟开销与头的大小无关，而与解析/句柄/存储这些头的逻辑有关。此外，TCP连接设置延迟可能比每个请求的大小或处理时间更大。

## 2)为什么要实现它而不是更新HTTP协议?

目前正在努力重新设计HTTP协议，以获得更好的性能和更低的延迟，例如SPDY、HTTP 2.0和QUIC。这将改善正常HTTP请求的情况，但WebSockets和/或WebRTC DataChannel对客户机到服务器数据传输的延迟可能仍低于HTTP协议(或者它将以一种非常类似于WebSockets的模式使用)。

更新:

下面是一个思考web协议的框架:

- TCP:低级，双向，全双工，保证顺序传输层。没有浏览器支持(除了插件/Flash)。
- HTTP 1.0: TCP上分层的请求-响应传输协议。客户机发出一个完整的请求，服务器发出一个完整的响应，然后连接被关闭。请求方法(GET、POST、HEAD)对于服务器上的资源具有特定的事务含义。
- HTTP 1.1:维护HTTP 1.0的请求-响应性质，但允许连接为多个完整请求/完整响应(每个请求一个响应)保持打开状态。请求和响应中仍然有完整的标头，但是连接被重用而不是关闭。HTTP 1.1还添加了一些额外的请求方法(选项、PUT、DELETE、TRACE、CONNECT)，这些方法也具有特定的事务含义。然而，正如在HTTP 2.0草案的介绍中提到的，HTTP 1.1流水线并没有被广泛部署，因此这极大地限制了HTTP 1.1用于解决浏览器和服务器之间的延迟的效用。
- 长轮询:类似于HTTP(1.0或1.1)的“黑客”，服务器不立即响应(或仅部分响应报头)客户机请求。在服务器响应之后，客户机立即发送一个新请求(如果通过HTTP 1.1使用相同的连接)。
- HTTP流:允许服务器向单个客户机请求发送多个响应的各种技术(多部分/块响应)。W3C将其标准化为使用文本/事件流MIME类型的服务器发送事件。浏览器API(非常类似于WebSocket API)被称为EventSource API。
- Comet/服务器推送:这是一个综合术语，包括长轮询和HTTP流。Comet库通常支持多种技术来尝试和最大化跨浏览器和跨服务器的支持。
- WebSockets:构建在TCP上的传输层，使用HTTP友好的升级握手。与TCP(流传输)不同，websocket是基于消息的传输:消息在连线上被分隔，并在交付给应用程序之前被完整地重新组装。WebSocket连接是双向的、全双工的、长寿命的。在初始握手请求/响应之后，没有事务语义，每个消息开销非常少。客户端和服务器可以在任何时候发送消息，并且必须异步处理消息接收。
- SPDY:一个谷歌提议使用更高效的有线协议扩展HTTP，但维护所有HTTP语义(请求/响应、cookie、编码)。SPDY引入了一种新的框架格式(带有长度前缀的框架)，并指定了一种将HTTP请求/响应对分层到新的框架层的方法。可以压缩标头，并在连接建立后发送新的标头。在浏览器和服务器上有SPDY的实际实现。
- HTTP 2.0:具有与SPDY相似的目标:减少HTTP延迟和开销，同时保留HTTP语义。当前的草案来自SPDY，定义了一个升级握手和数据帧，与WebSocket握手和帧的标准非常相似。另一个HTTP 2.0草案提议(httpbis-speed-mobility)实际上使用了WebSocket作为传输层，并将SPDY复用和HTTP映射添加为WebSocket扩展(WebSocket扩展是在握手过程中协商的)。
- WebRTC:允许浏览器之间点对点连接的建议。由于底层传输是SDP/datagram，而不是TCP，这可能会降低平均延迟和最大延迟通信。这允许包/消息的无序交付，从而避免了TCP的延迟问题，因为丢失的包延迟了所有后续包的交付(以保证有序交付)。
- QUIC:是一种实验协议，旨在减少web延迟超过TCP。从表面上看，QUIC非常类似于在UDP上实现的TCP+TLS+SPDY。QUIC提供了等效于HTTP/2的多路复用和流控制，等效于TLS的安全性，以及等效于TCP的连接语义、可靠性和拥塞控制。因为TCP是在操作系统内核和中间件中实现的，所以对TCP进行重大更改几乎是不可能的。然而，由于QUIC是构建在UDP之上的，所以它没有这样的限制。QUIC是为HTTP/2语义而设计和优化的。


# 阮一峰的博客
## 背景
初次接触 WebSocket 的人，都会问同样的问题：我们已经有了 HTTP 协议，为什么还需要另一个协议？它能带来什么好处？

答案很简单，因为 HTTP 协议有一个缺陷：通信只能由客户端发起。

举例来说，我们想了解今天的天气，只能是客户端向服务器发出请求，服务器返回查询结果。HTTP 协议做不到服务器主动向客户端推送信息。

这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。我们只能使用"轮询"：每隔一段时候，就发出一个询问，了解服务器有没有新的信息。最典型的场景就是聊天室。

轮询的效率低，非常浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）。因此，工程师们一直在思考，有没有更好的方法。WebSocket 就是这样发明的。

## 简介
WebSocket 协议在2008年诞生，2011年成为国际标准。所有浏览器都已经支持了。

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。

其他特点包括：

（1）建立在 TCP 协议之上，服务器端的实现比较容易。

（2）与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

（3）数据格式比较轻量，性能开销小，通信高效。

（4）可以发送文本，也可以发送二进制数据。

（5）没有同源限制，客户端可以与任意服务器通信。

（6）协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。

# 代码
## 客户端引用的类库
```
  <script src="/webjars/jquery/jquery.min.js"></script>
  <script src="/webjars/sockjs-client/sockjs.min.js"></script>
  <script src="/webjars/stomp-websocket/stomp.min.js"></script>
  <script src="/app.js"></script>
```
其中,WebJars are client-side web libraries (e.g. jQuery & Bootstrap) packaged into JAR (Java Archive) files.


## 客户端
```
function connect() {
  var socket = new SockJS('/gs-guide-websocket');
  stompClient = Stomp.over(socket);
  stompClient.connect({}, function (frame) {
    setConnected(true);
    console.log('Connected: ' + frame);
    stompClient.subscribe('/topic/logs', function (greeting) {
      showGreeting(JSON.parse(greeting.body).content);
    });
  });
}

function showGreeting(message) {
  $("#greetings").append("<tr><td>" + message + "</td></tr>");
}
```


## 服务端


## 比较
WebSockets

它是允许客户端和服务器之间进行同步双向通信的规范。 虽然类似于TCP套接字，但它是一个协议，作为升级的HTTP连接运行，在双方之间交换可变长度的帧，而不是流。

STOMP

它定义了客户端和服务器与消息传递语义进行通信的协议。 它没有定义任何实现细节，而是解决了易于实现的消息传递集成的有线协议。 它在WebSockets协议之上提供了更高的语义，并定义了一些映射到WebSockets框架的帧类型。 其中一些是......

- connect
- subscribe
- unsubscribe
- send (messages sent to the server)
- message (for messages send from the server) BEGIN, COMMIT, ROLLBACK (transaction management)


# spring集成websocket
https://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/websocket.html



# stomp协议(Simple Text Oriented Messaging Protocol)

![upload successful](/images/pasted-281.png)
规范:
http://stomp.github.io/stomp-specification-1.2.html
实现组件:
http://stomp.github.io/implementations.html

http://jmesnil.net/stomp-websocket/doc/

# stomp协议语法
## 整体框架
```
COMMAND
header1:value1
header2:value2

Body^@
```

## value编码
UTF-8

## Body
Only the SEND, MESSAGE, and ERROR frames MAY have a body. All other frames MUST NOT have a body.
## Standard Headers
### Header content-length
### Header content-type
例如:application/xml;charset=utf-8
### Header receipt
## 重复的头信息
## 大小限制


## 简历连接
CONNECT-->CONNECTED
### 协议协商
### 心跳
- 第一个数字表示帧的发送者可以做什么（外出的心跳）：
    - 心跳header必须包含两个用逗号分隔的正整数。
    - 0表示无法发送心跳,否则它是心跳之间可以保证的最小毫秒数
- 第二个数字表示帧的发送者想要获得的内容（传入的心跳）：
    - 0意味着它不想接受心跳 
    - 否则它是心跳之间所需的毫秒数

**心跳头是可选的。丢失的心跳标题必须与“心跳：0,0”标题一样对待，即：当事人不能发送并且不想接收心跳。**

例如:
```
CONNECT
heart-beat:<cx>,<cy>

CONNECTED:
heart-beat:<sx>,<sy>
```

如果<cx>为0（客户端无法发送心跳）或<sy>为0（服务器不想接收心跳）那么将没有,否则，每MAX（<cx>，<sy>）毫秒都会有心跳

关于心跳本身，通过网络连接接收的任何新数据都表明远程端是活的。在给定的方向上，如果每隔<n>毫秒需要心跳：
- 发送方必须至少每<n>毫秒通过网络连接发送新数据
- 如果发送方没有要发送的真实STOMP帧，它必须发送一个行尾（EOL）
- 如果，在一个至少<n>毫秒的时间窗口内，接收器没有收到任何新数据，它可能会认为连接已死
- 由于时序不准确，接收器应该容忍并考虑误差范围


## 客户端帧


## 服务端帧


## Augmented BNF
```
NULL                = <US-ASCII null (octet 0)>
LF                  = <US-ASCII line feed (aka newline) (octet 10)>
CR                  = <US-ASCII carriage return (octet 13)>
EOL                 = [CR] LF 
OCTET               = <any 8-bit sequence of data>

frame-stream        = 1*frame

frame               = command EOL
                      *( header EOL )
                      EOL
                      *OCTET
                      NULL
                      *( EOL )

command             = client-command | server-command

client-command      = "SEND"
                      | "SUBSCRIBE"
                      | "UNSUBSCRIBE"
                      | "BEGIN"
                      | "COMMIT"
                      | "ABORT"
                      | "ACK"
                      | "NACK"
                      | "DISCONNECT"
                      | "CONNECT"
                      | "STOMP"

server-command      = "CONNECTED"
                      | "MESSAGE"
                      | "RECEIPT"
                      | "ERROR"

header              = header-name ":" header-value
header-name         = 1*<any OCTET except CR or LF or ":">
header-value        = *<any OCTET except CR or LF or ":">
```


# stomp例子
## 请求连接:
```
CONNECT↵
accept-version:1.1,1.0↵
heart-beat:10000,10000↵
↵
```
## 响应连接:
```
CONNECTED↵
version:1.1↵
heart-beat:0,0↵
↵
```

## 请求订阅:
```
SUBSCRIBE↵
id:sub-0↵
destination:/topic/logs↵
↵
```

## 服务端推送:
```
MESSAGE↵
destination:/topic/logs↵
content-type:application/json;charset=UTF-8↵
subscription:sub-0↵
message-id:050vphrg-1↵
content-length:592↵
↵
{"content":"[2018-10-10T17:27:06.587]"}

```


# spring集成stomp-websocket的例子
https://www.devglan.com/spring-boot/spring-boot-websocket-integration-example

https://spring.io/guides/gs/messaging-stomp-websocket/

# wireshark分析websocket连接建立
## 第一次请求
请求
```
Hypertext Transfer Protocol
    GET /gs-guide-websocket/info?t=1539154442931 HTTP/1.1\r\n
    Host: localhost:8084\r\n
    Connection: keep-alive\r\n
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36\r\n
    Accept: */*\r\n
    Referer: http://localhost:8084/\r\n
    Accept-Encoding: gzip, deflate, br\r\n
    Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7\r\n
    Cookie: Hm_lvt_2342180055c7117a644c0e88269f1028=1538139605; scroll-cookie=0|/; Hm_lpvt_2342180055c7117a644c0e88269f1028=1538278742; Idea-eed2f053=fc396b83-b3d3-44fa-b97b-68144954c541\r\n
    \r\n
    [Full request URI: http://localhost:8084/gs-guide-websocket/info?t=1539154442931]
    [HTTP request 4/4]
    [Prev request in frame: 1052]
    [Response in frame: 1172]



```
响应
```
Hypertext Transfer Protocol
    HTTP/1.1 200 OK\r\n
    Date: Wed, 10 Oct 2018 06:54:02 GMT\r\n
    Cache-Control: no-store, no-cache, must-revalidate, max-age=0\r\n
    Content-Type: application/json;charset=utf-8\r\n
    Transfer-Encoding: chunked\r\n
    Server: Jetty(9.4.9.v20180320)\r\n
    \r\n
    [HTTP response 4/4]
    [Time since request: 0.001637000 seconds]
    [Prev request in frame: 1052]
    [Prev response in frame: 1054]
    [Request in frame: 1168]
    HTTP chunked response
    File Data: 77 bytes

```
## 第二次请求
请求
```
Hypertext Transfer Protocol
    GET /gs-guide-websocket/227/ldhduxxs/websocket HTTP/1.1\r\n
    Host: localhost:8084\r\n
    Connection: Upgrade\r\n
    Pragma: no-cache\r\n
    Cache-Control: no-cache\r\n
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36\r\n
    Upgrade: websocket\r\n
    Origin: http://localhost:8084\r\n
    Sec-WebSocket-Version: 13\r\n
    Accept-Encoding: gzip, deflate, br\r\n
    Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7\r\n
    Cookie: Hm_lvt_2342180055c7117a644c0e88269f1028=1538139605; scroll-cookie=0|/; Hm_lpvt_2342180055c7117a644c0e88269f1028=1538278742; Idea-eed2f053=fc396b83-b3d3-44fa-b97b-68144954c541\r\n
    Sec-WebSocket-Key: dO3WGUyUTNXQgBYRKu91wg==\r\n
    Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits\r\n
    \r\n
    [Full request URI: http://localhost:8084/gs-guide-websocket/227/ldhduxxs/websocket]
    [HTTP request 1/1]
    [Response in frame: 1180]

```
响应
```
Hypertext Transfer Protocol
    HTTP/1.1 101 Switching Protocols\r\n
    Date: Wed, 10 Oct 2018 06:54:02 GMT\r\n
    Sec-WebSocket-Extensions: permessage-deflate\r\n
    Connection: Upgrade\r\n
    Sec-WebSocket-Accept: LoLq2R5kSIlbKSZl99M0i5JoeGE=\r\n
    Server: Jetty(9.4.9.v20180320)\r\n
    Upgrade: WebSocket\r\n
    \r\n
    [HTTP response 1/1]
    [Time since request: 0.002971000 seconds]
    [Request in frame: 1178]

```

## websocket协议
```
Hypertext Transfer Protocol
    HTTP/1.1 101 Switching Protocols\r\n
    Date: Wed, 10 Oct 2018 06:54:02 GMT\r\n
    Sec-WebSocket-Extensions: permessage-deflate\r\n
    Connection: Upgrade\r\n
    Sec-WebSocket-Accept: LoLq2R5kSIlbKSZl99M0i5JoeGE=\r\n
    Server: Jetty(9.4.9.v20180320)\r\n
    Upgrade: WebSocket\r\n
    \r\n
    [HTTP response 1/1]
    [Time since request: 0.002971000 seconds]
    [Request in frame: 1178]

```

```
WebSocket
    1... .... = Fin: True
    .100 .... = Reserved: 0x4
    .... 0001 = Opcode: Text (1)
    1... .... = Mask: True
    .011 1110 = Payload length: 62
    Masking-Key: 40602aeb
    Masked payload
    Payload
```

# websocket与socket的关系
HTTP、WebSocket 等应用层协议，都是基于 TCP 协议来传输数据的。我们可以把这些高级协议理解成对 TCP 的封装。
对于 WebSocket 来说，它必须依赖 HTTP 协议进行一次握手 ，握手成功后，数据就直接从 TCP 通道传输，与 HTTP 无关了。

## scoket
Socket 其实并不是一个协议。它工作在 OSI 模型会话层（第5层），是为了方便大家直接使用更底层协议（一般是 TCP 或 UDP ）而存在的一个抽象层。
Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

## 小结
感觉是因为Socket实现的是TCP，而TCP是面向stream传输的，直接用Socket容易出问题，比如粘包问题。WebSocket是一个应用层协议，更抽象，不需要考虑这些问题。所以当然可以直接用Socket实现长连接，不过仍然需要实现自己的上层协议来处理分包，比较麻烦，不如直接用WebSocket。自己的客户端使用自己的私有协议还行，但浏览器的话更需要一个统一的应用层协议，所以WebSocket就出现
所以，从使用上来说，WebSocket 更易用，而 Socket 更灵活。

# 参考
https://github.com/onlyliuxin/coding2017/issues/497
https://blog.zengrong.net/post/2199.html