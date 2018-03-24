---
title: bugfix-duplicate-key
date: 2018-03-19 12:26:01
tags:
categories:
---

一次bugfix的经历过
# 步骤
## 异常信息

```
java.lang.IllegalStateException: Duplicate key 


 at java.util.stream.Collectors.lambda$throwingMerger$0(Collectors.java:133) ~[?:1.8.0_92]
        at java.util.HashMap.merge(HashMap.java:1253) ~[?:1.8.0_92]
        at java.util.stream.Collectors.lambda$toMap$58(Collectors.java:1320) ~[?:1.8.0_92]
        at java.util.stream.ReduceOps$3ReducingSink.accept(ReduceOps.java:169) ~[?:1.8.0_92]
        at java.util.ArrayList$ArrayListSpliterator.forEachRemaining(ArrayList.java:1374) ~[?:1.8.0_92]
        at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:481) ~[?:1.8.0_92]
        at java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:471) ~[?:1.8.0_92]
        at java.util.stream.ReduceOps$ReduceOp.evaluateSequential(ReduceOps.java:708) ~[?:1.8.0_92]
        at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234) ~[?:1.8.0_92]
        at java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:499) ~[?:1.8.0_92]
        at 
```

## 报错代码

```
Map<Long, PmsDailyPurchase> result = all.stream().collect(Collectors.toMap(PmsDailyPurchase::getSaleOrderId, Function.identity()));
```

对于ToMap方法没有做兼容,优点是方便问题的排查
逻辑上讲应该是不会出现Duplicate key 的

## 主要逻辑
先查询,如果不存在就插入

## 问题总结
- dubbo默认容错机制:dubbo默认的2次重复调用(加起来3次)
- provider提供的接口没有幂等性
- 注解配置问题:项目中AvoidRepeatInvoke注解会对重复的调用进行处理
	+ 未调用过:真正执行,保存调用结果
	+ 调用过:直接返回调用结果
	+ 调用的凭证是60s超时的,对于响应大于60s的会自动过期,但实际上还在执行,**导致问题的发生**
- 大事务:最近上线读写分离,在整个大方的外部添加了事务注解,导致无法自动提交,当并发时,可能查不到其他事务,已经insert但是没有提交的内容(此处也有幻读的风险,但是使用阿里rds测试后,没有发生)


	

# 反思
- 修改项目级别的默认容错机制, retries=0
- 超时时间设定
	+ 统计监控微服务的方法级别超时时间,修改方法级别
- 批量接口,限制最大数量,否则调用时间会太长
- 事务的范围,需要仔细考虑,不能有过大的事务
