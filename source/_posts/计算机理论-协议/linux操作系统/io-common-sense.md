---
title: io常识
abbrlink: 3268588280
date: 2018-10-15 09:23:24
tags:
categories:
---

# 名词
- SSD（Solid State Disk,固态硬盘）
- HD（Hard Disk,机械硬盘 ）


https://help.aliyun.com/document_detail/25382.html?spm=5176.ecsbuyv3.storage.2.822136753kuhG1
# IOPS
IOPS是Input/Output Operations per Second，即每秒能处理的I/O个数，用于表示块存储处理读写（输出/输入）的能力。如果要部署事务密集型应用，典型场景比如数据库类业务应用，需要关注IOPS性能。

最普遍的IOPS性能指标是顺序操作和随机操作，如下表所示。

IOPS性能指标 | 描述
---------|---
总 IOPS | 每秒执行的I/O操作总次数。
随机读IOPS | 每秒执行的随机读I/O操作的平均次数 ,对硬盘存储位置的不连续访问。
随机写IOPS | 每秒执行的随机写I/O操作的平均次数 , 对硬盘存储位置的不连续访问。
顺序读IOPS | 每秒执行的顺序读I/O操作的平均次数 , 对硬盘存储位置的连续访问。
顺序写IOPS | 每秒执行的顺序写I/O操作的平均次数 , 对硬盘存储位置的连续访问。

# 吞吐量
吞吐量是指单位时间内可以成功传输的数据数量。

如果要部署大量顺序读写的应用，典型场景比如Hadoop离线计算型业务，需要关注吞吐量。

# 访问时延
访问时延是指块存储处理一个I/O需要的时间。

如果您的应用对时延比较敏感，比如数据库（过高的时延会导致应用性能下降或报错），建议您使用ESSD云盘、SSD云盘、SSD共享块存储或本地SSD盘类产品。

如果您的应用更偏重存储吞吐能力，对时延相对不太敏感，比如Hadoop离线计算等吞吐密集型应用，建议您使用本地HDD盘类产品，如d1或d1ne大数据型实例。

# 阿里云产品比较
参数 | ESSD云盘 | SSD云盘 | 高效云盘 | 普通云盘
---|--------|-------|------|-----
单盘最大容量 | 32768 GiB | 32768 GiB | 32768 GiB | 2000 GiB
最大IOPS | 1000000 | 25000* | 5000 | 数百
最大吞吐量 | 4000 MBps | 300 MBps* | 140 MBps | 30−40 MBps
单盘性能计算公式** | IOPS = min{1800 + 50 * 容量, 1000000} | IOPS = min{1800 + 30 * 容量, 25000} | IOPS = min{1800 + 8 * 容量, 5000} | 无
吞吐量= min{120 + 0.5 * 容量, 4000} MBps | 吞吐量= min{120 + 0.5 * 容量, 300} MBps | 吞吐量= min{100+ 0.15 * 容量, 140} MBps | 无
数据可靠性 | 99.9999999% | 99.9999999% | 99.9999999% | 99.9999999%
API名称 | cloud_essd | cloud_ssd | cloud_efficiency | cloud
典型应用场景 | OLTP数据库：如MySQL、PostgreSQL、Oracle、SQL Server等关系型数据库    NoSQL数据库：如MongoDB、HBase、Cassandra等非关系型数据库    ElasticSearch分布式日志：ELK（Elasticsearch、Logstash和Kibana）日志分析等 | PostgreSQL、MySQL、Oracle、SQL Server等中大型关系数据库应用    对数据可靠性要求高的中大型开发测试环境 | MySQL、SQL Server、PostgreSQL等中小型关系数据库应用    对数据可靠性要求高、中度性能要求的中大型开发测试应用 | 数据不被经常访问或者低I/O负载的应用场景（如果应用需要更高的I/O性能，建议使用SSD云盘）    需要低成本并且有随机读写I/O的应用环境



SSD云盘的性能因数据块大小而异，数据块越小，吞吐量越小，IOPS越高