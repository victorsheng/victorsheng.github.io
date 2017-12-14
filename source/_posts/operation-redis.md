title: 线上redis崩溃记录
tags:
  - redis
  - breakdown
categories: []
date: 2017-12-14 09:39:00
---
# 错误日志
grep com.alibaba.dubbo.rpc.filter.ExceptionFilter.error
```
9:02
org.springframework.data.redis.RedisConnectionFailureException: Unexpected end of stream.; nested exception is redis.clients.jedis.exceptions.JedisConnectionException: Unexpected en
d of stream.好多条

org.springframework.dao.InvalidDataAccessApiUsageException: LOADING Redis is loading the dataset in memory; nested exception is redis.clients.jedis.exceptions.JedisDataException: LOADING Redis is loading the dataset in memory 好多条


org.springframework.data.redis.RedisConnectionFailureException: java.net.SocketTimeoutException: Read timed out; nested exception is redis.clients.jedis.exceptions.JedisConnectionException: java.net.SocketTimeoutException: Read timed out


org.springframework.data.redis.RedisConnectionFailureException: java.net.SocketException: Connection reset; nested exception is redis.clients.jedis.exceptions.JedisConnectionException: java.net.SocketException: Connection reset



org.springframework.data.redis.RedisConnectionFailureException: Cannot get Jedis connection; nested exception is redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool很多


9:21开始
org.springframework.dao.InvalidDataAccessApiUsageException: ERR READONLY You can't write against a read only instance; nested exception is redis.clients.jedis.exceptions.JedisDataException: ERR READONLY You can't write against a read only instance很多
```

# 期间代码回滚到了上一版本,过了一段时间系统正常

# 最后发现是 业务层面循环调用了redis操作,一次请求会有1k次的redis,导致redis崩掉,而且我们使用的阿里云的主从版云redis
- 几次循环调用,inFlow的变化
- <img src="http://pic.victor123.cn/17-12-14/44581326.jpg">
- 具体得失hset方法的调用
- <img src="http://pic.victor123.cn/17-12-14/50782333.jpg">

# 反思
- 对于监控系统,应该有各个节点的平时统计数据,包括,调用次数+相应时长
- 通过环比和同比数据的对比,可以快速的发现问题,解决问题,而不是当系统垮掉时,才有所反应

# BTW
## nest exception学习
http://www.iteye.com/problems/87876

- 即调用顺序是 action--->service--->dao---->hibernate--->jdbc 
- 抛出异常顺序是 jdbc---->hibernate---->dao--抛出-->service--继续抛出-->action 
- 异常其实是栈调用的快照 

- 1、最下层的异常是出错的原因，上边的异常是对下边的封装，目的是一致性 和 更可读；（即下边异常是引起上边异常的原因，每一个Exception都有一个cause，如hibernate异常的cause就是jdbc的异常） 
- 2、对于每一段异常，方法调用顺序是从下往上
- 3、想知道是由于前面创建的错误导致后边的异常，还是后边的异常导致前面的创建错误，后边导致前面，，，前面对后面的进行了封装，，目的是提供一致的异常（并且把原始错误显示出来 就是 nested exception 后边部分） 

## NestedRuntimeException 例子
我们最熟悉的就是 DataAccessException
```
package org.springframework.dao;

import org.springframework.core.NestedRuntimeException;

public abstract class DataAccessException extends NestedRuntimeException {
    public DataAccessException(String msg) {
        super(msg);
    }

    public DataAccessException(String msg, Throwable cause) {
        super(msg, cause);
    }
}

```
