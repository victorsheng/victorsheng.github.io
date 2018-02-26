title: alibaba-code-guide
date: 2018-02-03 21:03:21
tags:
categories:
---
卫语句、策略模式、状态模式
不允许直接拿 HashMap 与 Hashtable 作为查询结果集的输出。
Query:数据查询对象，各层接收上层的查询请求。注意超过 2 个参数的查询封装，禁止使用 Map 类来传输

领域模型命名规约
1) 数据对象:xxxDO，xxx即为数据表名。
2) 数据传输对象:xxxDTO，xxx为业务领域相关的名称。 
3) 展示对象:xxxVO，xxx一般为网页名称。
4) POJO是DO/DTO/BO/VO的统称，禁止命名成xxxPOJO。

