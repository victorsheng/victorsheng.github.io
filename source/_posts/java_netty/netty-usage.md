---
title: netty应用
date: 2018-04-17 23:29:08
tags:
categories:
---
# navi-pbrpc
教程：
http://neoremind.com/2016/02/%E5%9F%BA%E4%BA%8Eprotobuf%E7%9A%84%E5%88%86%E5%B8%83%E5%BC%8F%E9%AB%98%E6%80%A7%E8%83%BDrpc%E6%A1%86%E6%9E%B6-navi-pbrpc/
github地址：
https://github.com/neoremind/navi-pbrpc

## 客户端

handler链条的顺序：
- 序列化
- 反序列化
- 客户端处理

```
SimplePbrpcClient(PbrpcClientConfiguration pbrpcClientConfiguration, boolean isShortLiveConn,
            String ip, int port, int connTimeout, int readTimeout) {
        if (pbrpcClientConfiguration != null) {
            this.pbrpcClientConfiguration = pbrpcClientConfiguration;
        }
        this.ip = ip;
        this.port = port;
        this.connTimeout = connTimeout;
        this.readTimeout = readTimeout;
        this.isShortAliveConn = isShortLiveConn;
        bootstrap = new Bootstrap();
        bootstrap.channel(NioSocketChannel.class);
        bootstrap.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, this.connTimeout);
        bootstrap.option(ChannelOption.SO_KEEPALIVE, this.pbrpcClientConfiguration.isSoKeepalive());
        bootstrap.option(ChannelOption.SO_REUSEADDR, this.pbrpcClientConfiguration.isSoReuseaddr());
        bootstrap.option(ChannelOption.TCP_NODELAY, this.pbrpcClientConfiguration.isTcpNodelay());
        bootstrap.option(ChannelOption.SO_RCVBUF, this.pbrpcClientConfiguration.getSoRcvbuf());
        bootstrap.option(ChannelOption.SO_SNDBUF, this.pbrpcClientConfiguration.getSoSndbuf());

        ChannelInitializer<SocketChannel> initializer = new ChannelInitializer<SocketChannel>() {
            @Override
            protected void initChannel(SocketChannel ch) throws Exception {
                ch.pipeline().addLast(new PbrpcMessageSerializer());
                ch.pipeline().addLast(new PbrpcMessageDeserializer());
                ch.pipeline().addLast(new PbrpcClientHandler());
            }
        };
        eventLoopGroup = new NioEventLoopGroup();
        bootstrap.group(eventLoopGroup).handler(initializer);

        startTimeoutEvictor();
    }
```


### 序列化handler
拼装协议

```
    @Override
    protected void encode(ChannelHandlerContext ctx, PbrpcMsg pbrpcMsg, List<Object> out)
            throws Exception {
        byte[] bodyBytes = ByteUtil.getNonEmptyBytes(pbrpcMsg.getData());
        NsHead nsHead = contructNsHead(pbrpcMsg.getServiceId(), pbrpcMsg.getErrorCode(),
                pbrpcMsg.getLogId(), pbrpcMsg.getData(), pbrpcMsg.getProvider());
        byte[] nsHeadBytes = nsHead.toBytes();
        ByteBuf encoded = Unpooled.copiedBuffer(nsHeadBytes, bodyBytes);
        out.add(encoded);
        // LOG.info("Send total byte size=" + (nsHeadBytes.length + bodyBytes.length) + ", body size="
        // + bodyBytes.length);
    }
```


### 反序列化handler
- 处理半包问题
- 反序列化


```
@Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        // 解决半包问题，此时Nshead还没有接收全，channel中留存的字节流不做处理
        if (in.readableBytes() < NsHead.NSHEAD_LEN) {
            return;
        }

        in.markReaderIndex();

        byte[] bytes = new byte[NsHead.NSHEAD_LEN];
        in.readBytes(bytes, 0, NsHead.NSHEAD_LEN);

        NsHead nsHead = new NsHead();
        nsHead.wrap(bytes);

        // 解决半包问题，此时body还没有接收全，channel中留存的字节流不做处理，重置readerIndex
        if (in.readableBytes() < (int) nsHead.getBodyLen()) {
            in.resetReaderIndex();
            return;
        }

        // 此时接受到了足够的一个包，开始处理
        in.markReaderIndex();

        byte[] totalBytes = new byte[(int) nsHead.getBodyLen()];
        in.readBytes(totalBytes, 0, (int) nsHead.getBodyLen());

        PbrpcMsg decoded = PbrpcMsg.of(nsHead).setData(totalBytes);
        ContextHolder.putContext("_logid", nsHead.getLogId()); // TODO

        if (decoded != null) {
            out.add(decoded);
        }
        // LOG.info("Deser data " + nsHead.getLogId() + " is" + decoded + " and using "
        // + (System.nanoTime() - start) / 1000 + "us");
    }
```

### 客户端handler

```
public class PbrpcClientHandler extends SimpleChannelInboundHandler<PbrpcMsg> {

    private static final Logger LOG = LoggerFactory.getLogger(PbrpcMessageDeserializer.class);

    /**
     * 可配置，默认使用protobuf来做body的序列化
     */
    private Codec codec = new ProtobufCodec();

    /**
     * @see io.netty.channel.SimpleChannelInboundHandler#channelRead0(io.netty.channel.ChannelHandlerContext,
     *      java.lang.Object)
     */
    @SuppressWarnings("unchecked")
    @Override
    public void channelRead0(ChannelHandlerContext ctx, PbrpcMsg pbrpcMsg) throws Exception {
        Preconditions
                .checkArgument(pbrpcMsg != null, "Pbrpc msg is null which should never happen");
        try {
            // LOG.info("Got msg from server:" + pbrpcMsg);
            int logId = (int) pbrpcMsg.getLogId();
            CallbackContext context = CallbackPool.getContext(logId);
            if (context == null) {
                LOG.warn("Receive msg from server but no context found, logId=" + logId);
                return;
            }
            Callback<GeneratedMessage> cb = (Callback<GeneratedMessage>) context.getCallback();
            if (pbrpcMsg.getErrorCode() != null) {
                cb.handleError(ExceptionUtil.buildFromErrorCode(pbrpcMsg.getErrorCode()));
            } else {
                GeneratedMessage res = (GeneratedMessage) codec.decode(
                        CallbackPool.getResClass(logId), pbrpcMsg.getData());
                cb.handleResult(res);
            }

            // 短连接则关闭channel
            if (context.isShortAliveConn()) {
                Channel channel = context.getChannel();
                if (channel != null) {
                    LOG.info("Close " + channel + ", logId=" + logId);
                    channel.close();
                }
            }
            // LOG.info("Decoding and invoking callback " + pbrpcMsg.getLogId() + " total "
            // + (System.currentTimeMillis() - context.getStarttime())
            // + "ms, transport using " + (start - context.getStarttime()) + "ms");
        } finally {
            CallbackPool.remove((int) pbrpcMsg.getLogId());
            ContextHolder.clean();
            // ctx.fireChannelReadComplete();
        }
    }

    /**
     * @see io.netty.channel.ChannelInboundHandlerAdapter#channelReadComplete(io.netty.channel.ChannelHandlerContext)
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
        // LOG.debug("Client channelReadComplete>>>>>");
        // ctx.flush();
    }

    /**
     * @see io.netty.channel.ChannelInboundHandlerAdapter#channelActive(io.netty.channel.ChannelHandlerContext)
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        // LOG.debug("Client channelActive>>>>>");
    }

    /**
     * @see io.netty.channel.ChannelInboundHandlerAdapter#exceptionCaught(io.netty.channel.ChannelHandlerContext,
     *      java.lang.Throwable)
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        LOG.error(cause.getMessage(), cause);
        ctx.close(); // FIXME
        // ctx.fireChannelRead();
    }

}
```

## 服务端

顺序：
- 空闲处理
- 反序列化
- 客户端处理
- 序列化



```
public PbrpcServer(PbrpcServerConfiguration pbrpcServerConfiguration, int port) {
        // use default conf otherwise use specified one
        if (pbrpcServerConfiguration == null) {
            pbrpcServerConfiguration = this.pbrpcServerConfiguration;
        }
        this.port = port;

        bootstrap = new ServerBootstrap();
        bossGroup = new NioEventLoopGroup();
        workerGroup = new NioEventLoopGroup();

        bootstrap.channel(NioServerSocketChannel.class);
        bootstrap.option(ChannelOption.SO_BACKLOG, pbrpcServerConfiguration.getSoBacklog());

        bootstrap.childOption(ChannelOption.SO_KEEPALIVE, pbrpcServerConfiguration.isSoKeepalive());
        bootstrap.childOption(ChannelOption.TCP_NODELAY, pbrpcServerConfiguration.isTcpNodelay());
        bootstrap.childOption(ChannelOption.SO_REUSEADDR, true);
        bootstrap.childOption(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT);
        bootstrap.childOption(ChannelOption.SO_LINGER, pbrpcServerConfiguration.getSoLinger());
        bootstrap.childOption(ChannelOption.SO_RCVBUF, pbrpcServerConfiguration.getSoRcvbuf());
        bootstrap.childOption(ChannelOption.SO_SNDBUF, pbrpcServerConfiguration.getSoSndbuf());

        ChannelInitializer<SocketChannel> initializer = new ChannelInitializer<SocketChannel>() {
            @Override
            protected void initChannel(SocketChannel ch) throws Exception {
                ch.pipeline().addLast(
                        "idlestate",
                        new IdleStateHandler(IDLE_CHANNEL_TIMEOUT, IDLE_CHANNEL_TIMEOUT,
                                IDLE_CHANNEL_TIMEOUT));
                ch.pipeline().addLast("idle", new RpcServerChannelIdleHandler());
                ch.pipeline().addLast("deser", new PbrpcMessageDeserializer());
                ch.pipeline().addLast("coreHandler", new PbrpcServerHandler(serviceLocator));
                ch.pipeline().addLast("ser", new PbrpcMessageSerializer());
            }
        };
        bootstrap.group(bossGroup, workerGroup).childHandler(initializer);

        LOG.info("Pbrpc server init done");
    }
```

### ide handler

处理一些空闲的连接闭关它们，防止占用服务端资源

```
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof IdleStateEvent) {
            IdleStateEvent e = (IdleStateEvent) evt;
            if (e.state() == IdleState.WRITER_IDLE) {
                LOG.warn("Write idle on channel:" + ctx.channel() + " is timeout");
            } else if (e.state() == IdleState.READER_IDLE) {
                LOG.warn("Read idle on channel:" + ctx.channel() + " is timeout on "
                        + ctx.channel().remoteAddress() + ", so close it");
                // ctx.fireExceptionCaught(ReadTimeoutException.INSTANCE);
                ctx.close();
            }
        }
    }
```

### 服务端handler

```
public class PbrpcServerHandler extends SimpleChannelInboundHandler<PbrpcMsg> {

    private static final Logger LOG = LoggerFactory.getLogger(PbrpcMessageDeserializer.class);

    /**
     * 可配置，默认使用protobuf来做body的序列化
     */
    private Codec codec = new ProtobufCodec();

    /**
     * 可配置，服务定位器
     */
    private ServiceLocator<Integer> serviceLocator;

    /**
     * Creates a new instance of PbrpcServerHandler.
     * 
     * @param serviceLocator
     */
    @SuppressWarnings({ "rawtypes", "unchecked" })
    public PbrpcServerHandler(ServiceLocator serviceLocator) {
        this.serviceLocator = (ServiceLocator<Integer>) serviceLocator;
    }

    /**
     * @see io.netty.channel.SimpleChannelInboundHandler#channelRead0(io.netty.channel.ChannelHandlerContext,
     *      java.lang.Object)
     */
    @Override
    public void channelRead0(ChannelHandlerContext ctx, PbrpcMsg pbrpcMsg) throws Exception {
        Preconditions
                .checkArgument(pbrpcMsg != null, "Pbrpc msg is null which should never happen");
        try {
            if (pbrpcMsg.getErrorCode() != null) {
                ctx.channel().writeAndFlush(PbrpcMsg.copyLiteOf(pbrpcMsg)); // ONLY COMMUNICATIN ERROR HERE FIXME
                return;
            }
            int key = pbrpcMsg.getServiceId();
            ServiceDescriptor<Integer> servDesc = serviceLocator.getServiceDescriptor(key);
            if (servDesc == null) {
                throw new ServiceNotFoundException(" serviceId=" + pbrpcMsg.getServiceId());
            }

            GeneratedMessage arg = (GeneratedMessage) codec.decode(servDesc.getArgumentClass(),
                    pbrpcMsg.getData());
            GeneratedMessage ret = (GeneratedMessage) servDesc.getMethod().invoke(
                    servDesc.getTarget(), arg);

            PbrpcMsg retMsg = new PbrpcMsg();
            retMsg.setLogId(pbrpcMsg.getLogId());
            retMsg.setServiceId(key);
            if (ret != null && ret instanceof GeneratedMessage) {
                byte[] response = ((GeneratedMessage) ret).toByteArray();
                retMsg.setData(response);
            }
            // LOG.debug("Service biz logic will return:" + ret);
            ctx.channel().writeAndFlush(retMsg);
            // LOG.info(servDesc + " exec using " + (System.currentTimeMillis() - start) + "ms");
        } catch (ServiceNotFoundException e) {
            LOG.error(ErrorCode.SERVICE_NOT_FOUND.getMessage() + e.getMessage(), e);
            ctx.channel().writeAndFlush(
                    PbrpcMsg.copyLiteOf(pbrpcMsg).setErrorCode(ErrorCode.SERVICE_NOT_FOUND));
        } catch (CodecException e) {
            LOG.error(ErrorCode.PROTOBUF_CODEC_ERROR.getMessage() + e.getMessage(), e);
            ctx.channel().writeAndFlush(
                    PbrpcMsg.copyLiteOf(pbrpcMsg).setErrorCode(ErrorCode.PROTOBUF_CODEC_ERROR));
        } catch (InvocationTargetException e) {
            LOG.error(ErrorCode.INVOCATION_TARGET_EXCEPTION.getMessage() + e.getMessage(), e);
            ctx.channel().writeAndFlush(
                    PbrpcMsg.copyLiteOf(pbrpcMsg).setErrorCode(
                            ErrorCode.INVOCATION_TARGET_EXCEPTION));
        } catch (Exception e) {
            LOG.error(ErrorCode.UNEXPECTED_ERROR.getMessage() + e.getMessage(), e);
            ctx.channel().writeAndFlush(
                    PbrpcMsg.copyLiteOf(pbrpcMsg).setErrorCode(ErrorCode.UNEXPECTED_ERROR));
        } finally {
            ContextHolder.clean();
        }
    }

    /**
     * @see io.netty.channel.ChannelInboundHandlerAdapter#channelReadComplete(io.netty.channel.ChannelHandlerContext)
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
    }

    /**
     * @see io.netty.channel.ChannelInboundHandlerAdapter#channelActive(io.netty.channel.ChannelHandlerContext)
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
    }

    /**
     * @see io.netty.channel.ChannelInboundHandlerAdapter#exceptionCaught(io.netty.channel.ChannelHandlerContext,
     *      java.lang.Throwable)
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        LOG.error(cause.getCause().getMessage(), cause.getCause());

        long logId = (Long) (ContextHolder.getContext("_logid"));
        PbrpcMsg retMsg = new PbrpcMsg();
        retMsg.setLogId(logId);
        retMsg.setErrorCode(ErrorCode.COMMUNICATION_ERROR);
        // ctx.channel().writeAndFlush(retMsg);
        ctx.fireChannelRead(retMsg); // FIXME 对于通信异常的这样处理是否OK？
    }

}
```


# gateway项目

```
  @Override
  protected void initChannel(SocketChannel ch) throws Exception {
    
    
    CorsConfig corsConfig = CorsConfigBuilder.forOrigins(parseCorsAllowHostList())
        .allowedRequestHeaders("X-Access-Token", "content-type")
        .allowedRequestMethods(HttpMethod.GET, HttpMethod.POST, HttpMethod.PUT).build();

    ch.pipeline().addLast("codec", new HttpServerCodec());
    ch.pipeline().addLast("aggegator", new HttpObjectAggregator(maxContentLength));
    ch.pipeline().addLast("cors", new CorsHandler(corsConfig));
    ch.pipeline().addLast("idleStateHandler", new IdleStateHandler(0, 0, 300, TimeUnit.SECONDS));
    //zipkin相关的
    if (ZipkinHelper.httpTracing().isPresent())
      ch.pipeline().addLast("tracing", new RequestTracingHandler());
    //业务相关的handler
    for (ChannelHandler handler : handlers) {
      ch.pipeline().addLast(handler.getClass().getSimpleName(), handler); // Use this if need
    }
```



# 聊天室
https://waylau.com/netty-chat/

# 聊天室websocket版本
