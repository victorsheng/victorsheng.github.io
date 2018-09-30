---
title: BI-olap技术选型
tags:
  - BI
  - olap
categories:
  - BI
abbrlink: 4162788070
date: 2018-01-05 19:31:54
---
# 备选列表:
- 阿里云-分析性数据库
- GreePlum
- Presto
- Kylin
<!--more-->

# 阿里云-分析性数据库

![upload successful](/images/pasted-18.png)


# GreePlum

![upload successful](/images/pasted-9.png)
- Greenplum采用Postgresl作为底层引擎，良好的兼容了Postgresql的功能
- Greenplum的艺术，一切皆并行（Parallel Everything）
- Greenplum建立在Share-nothing无共享架构上，让每一颗CPU和每一块磁盘IO都运转起来，无共享架构将这种并行处理发挥到极致。相比一些其它传统数据仓库的Sharedisk架构，后者最大瓶颈就是在IO吞吐上，在大规模数据处理时，IO无法及时feed数据给到CPU，CPU资源处于wait 空转状态，无法充分利用系统资源，导致SQL效率低下
- 得益于Postgresql的良好扩展性（这里是extension，不是scalability），Greenplum 可以采用各种开发语言来扩展用户自定义函数（UDF）

## Greenplum MPP 与 Hadoop
### 相同点
- 分布式存储数据在多个节点服务器上
- 采用分布式并行计算框架
- 支持横向扩展来提高整体的计算能力和存储容量
- 都支持X86开放集群架构

### 差异点
- MPP按照关系数据库行列表方式存储数据（有模式），Hadoop按照文件切片方式分布式存储（无模式）
- 两者采用的数据分布机制不同，MPP采用Hash分布，计算节点和存储紧密耦合，数据分布粒度在记录级的更小粒度（一般在1k以下）；Hadoop FS按照文件切块后随机分配，节点和数据无耦合，数据分布粒度在文件块级（缺省64MB）。
- MPP采用SQL并行查询计划，Hadoop采用Mapreduce框架

# Presto
- Facebook贡献的开源MPP OLAP引擎。
- 这是一个红酒的名字，因为开发组所有的人都喜欢喝这个牌子的红酒，所以把它命名为这个名字。作为MPP引擎，它的处理方式是把所有的数据Scan出来，通过Hash的方法把数据变成更小的块，让不同的节点并发，处理完结果后快速地返回给用户。我们看到它的逻辑架构也是这样，发起一个SQL，然后找这些数据在哪些HDFS节点上，然后分配后做具体的处理，最后再把数据返回。 

# Kylin
- Kylin是由eBay开源的一个引擎，Kylin把数据读出来做计算，结算的结果会被存在HBase里，通过HBase做Ad-hoc的功能。HBase的好处是有索引的，所以做Ad-hoc的性能非常好。

![upload successful](/images/pasted-8.png)