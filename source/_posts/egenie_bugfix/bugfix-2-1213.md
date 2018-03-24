title: bugfix总结-12.13
tags:
  - bugfix
  - redis
categories:
    - redis
date: 2017-12-14 08:00:00
---
# bugfix1-redis相关
- 快速定位问题
```
2017-12-13 18:33:39.688 DEBUG [http-bio-6000-exec-17][JdbcTemplate.java:875] - SQL update affected 1 rows
2017-12-13 18:33:40.590 ERROR [http-bio-6000-exec-17][UniExceptionHandler.java:57] - 捕捉到技术异常: URI: /shop/insert  最大内存: 1012m  已分配内存: 1012m  已分配内存中的剩余空间: 532m  最大可用内存: 532m
java.lang.RuntimeException: org.springframework.data.redis.RedisSystemException: Unknown redis exception; nested exception is java.lang.NullPointerException
org.springframework.data.redis.RedisSystemException: Unknown redis exception; nested exception is java.lang.NullPointerException
        at org.springframework.data.redis.FallbackExceptionTranslationStrategy.getFallback(FallbackExceptionTranslationStrategy.java:48)
        at org.springframework.data.redis.FallbackExceptionTranslationStrategy.translate(FallbackExceptionTranslationStrategy.java:38)
        at org.springframework.data.redis.connection.jedis.JedisConnection.convertJedisAccessException(JedisConnection.java:242)
        at org.springframework.data.redis.connection.jedis.JedisConnection.set(JedisConnection.java:1236)
        at org.springframework.data.redis.connection.DefaultStringRedisConnection.set(DefaultStringRedisConnection.java:744)
        at org.springframework.data.redis.core.DefaultValueOperations$10.inRedis(DefaultValueOperations.java:172)
        at org.springframework.data.redis.core.AbstractOperations$ValueDeserializingRedisCallback.doInRedis(AbstractOperations.java:57)
        at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:207)
        at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:169)
        at org.springframework.data.redis.core.AbstractOperations.execute(AbstractOperations.java:91)
        at org.springframework.data.redis.core.DefaultValueOperations.set(DefaultValueOperations.java:169)
        at com.ejlerp.cache.redis.KVCacherImpl.set(KVCacherImpl.java:34)
        at com.alibaba.dubbo.common.bytecode.Wrapper7.invokeMethod(Wrapper7.java)
        at com.alibaba.dubbo.rpc.proxy.javassist.JavassistProxyFactory$1.doInvoke(JavassistProxyFactory.java:46)
        at com.alibaba.dubbo.rpc.proxy.AbstractProxyInvoker.invoke(AbstractProxyInvoker.java:72)
        at com.alibaba.dubbo.rpc.protocol.InvokerWrapper.invoke(InvokerWrapper.java:53)
```
- 首先这个是一个没有见过的异常
- 解决途径:
    + 搜索自己公司包开头的代码,找到调用地方
    + 下载源码,底层依赖可能有真的有
    + 在本次bugfix中,由于是微服务调用,真正看出问题是在,controller层才看出的问题
```
    @ApiOperation(value = "新增")
    @RequestMapping(value = "/insert", method = RequestMethod.POST)
    public JsonResult insert(@RequestBody String body) {
        Long id = IdGenerator.generate(getEntityName(), fetchTenantId()).getIdValue();
        CommonVo vo = CommonVo.of(getEntityName(), WebHelper.parseJsonBody(body, false, false));
        vo.put("shop_id", id);
        getService().insert(fetchTenantId(), fetchUserId(), vo);
        kvCacher.set("logistic_node_" + id.toString(), vo.get("logistic_node"));
        return new JsonResult(JsonResult.SUCCESSFUL, null);
    }

```
- vo可能别没有获取到
- 反思:at com.ejlerp.cache.redis.KVCacherImpl.set(KVCacherImpl.java:34),这个一句其实只有两个参数,一个是传入的key,一个是value,一定是其中一个入参发生了问题,才会导致底层出现问题的
- 当然,也不排除是redis服务可能出现了问题,但是当时已经重启了reids,错误依旧存在
- 最后,bug定位到了是前端一个select标签,由于数据库中的dict没有初始化,导致这个select标签的初始值没有初始化完成,当然表单提交时,便不会提交这个字段,

# 小结
- controller层的vo依然不要使用map,否则排查起来很费劲

# bugfix2 分组问题
- 这个bugfix,其实是这样的,条件写反导致的
```
         Map<Long, VSaleOrder> orderMap = omsAgentService.findByIds(callerInfo, arriveMap.keySet());
-        Set<Long> legalIds = orderMap.values().stream().filter(Objects::isNull).filter(v -> {
+        Set<Long> legalIds = orderMap.values().stream().filter(Objects::nonNull).filter(v -> {
             Integer courierPrintMarkState = v.getCourierPrintMarkState();

```
- 问题是,历史数据的修复,其实应该在线上发现问题时,作为一套完整的方案,进行提交的,而不是部署到生产环境后,又发现历史数据有问题,在半夜升级后改数据,风险真是很大
- 对于一些合并操作,虽然业务上提出了操作,其实从逻辑上讲,应该出现拆分的反响逻辑预支对应

# 问题3 jvm参数设置