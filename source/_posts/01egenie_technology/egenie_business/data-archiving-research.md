---
title: data-Archiving-research
tags:
  - MySQL
categories: []
abbrlink: 3291804496
date: 2018-01-25 10:42:00
---

# 归档目的
- 主要处于性能考虑,减少查找的范围


# 归档需求
- 同时要求系统中可以查询到之前的数据,不能归档到非结构化系统中
- 要求条件需要多表进行关联,并添加自定义条件,保证业务
- 要求主子表归档具有一致性,主子表不能分离
- 已经归档过一次
- 稳定性
- 可追溯
- 数据一致性
- 定时自动执行
- 跨数据库,最好支持到不同的数据源

< !--more-->


# 归档流程梳理
- 复制指定条件的旧数据
- 删除旧数据(是否需要人工介入?)后续自动
- 读取归档数据


# 归档方案思路:
## 1.建立归档表_archive
这种方式是通过建立一个以_archive结尾的归档表来实施的。如果使用这种方式，那么一般需要在业务层进行查询的分离改造，比如基于我们的特定归档规则 ，对业务端核心代码改造或者使用proxy方案等来决定是使用主表还是归档表。同时我们还需要一个数据归档过程，当数据过时或者变成冷数据时，将该数据从主表迁移到归档表中。

在业务层查询时，我们可以通过时间字段来进行查询判断，例如将90天之前的数据在归档表中查询，否则就在主表查询。另一种方式可以通过增加status列去判断查询主表还是归档表，如果是inactive则查询归档表，否则就在主表查询。

## mysql分区

为了让优化器能将查询发送到正确的分区键，在创建分区表的时候，我们需要将分区键添加到主键里，并且理想情况下，该分区键能被包含在所有select/update/delete等语句的where条件里面，否则的话，你的查询将会按照顺序查找表对应的每个分区，这种情况下查询性能就没有那么好了。

### 根据日期进行分区
这种方式是通过对表进行分区来实现，虽然这是一种不同的物理数据模型，但是确实有助于将表的数据进行拆分到不同的物理磁盘，并且不需要代码的任何改造。作为DBA，一般对表进行分区是比较常用的方式，我们可以通过日期字段很容易确定哪些是冷数据，并根据日期将不同日期的时间分配到不同的分区中，在查询的时候，我们可以通过日期来从分区中快速定位到对应的数据，同时建立分区表也比较利于DBA对大表进行管理操作。

我们根据数据行的创建时间，按年将数据放入到不同的分区，这里需要注意的是在2020年以后，我们还需要在表里添加新的年份，当然我们可以提前加更多的分区或者部署脚本来自动化创建新的分区。

### 通过状态进行分区
我们也可以通过status状态列来进行分区，这种情况下，通常状态列会包含active/inactive两种状态，然后通过update进行状态列的更新（使用replace或者insert+delete也是可以的），将数据放入到正确的分区当中。请看下面的示例

## 通过ID进行分区


# 具体方案
## 复制表并且按照条件插入数据（此种方法除了主键索引不包括其他索引）-->手动sql
http://blog.csdn.net/bluestarf/article/details/49641047

```
CREATE TABLE lime_survey_549656_20151001 as select * from lime_survey_549656  where submitdate < "2015-10-01 00:00:00";  
   
ALTER TABLE lime_survey_549656_20151001 change id id int primary key auto_increment;  

```

## 创建一张空表，结构和索引和原表一样 -->手动sql
```
create table lime_survey_549656_20151001 like lime_survey_549656;   
INSERT INTO lime_survey_549656_20151001 select * from lime_survey_549656  where submitdate < "2015-10-01 00:00:00";  
```


## 存储过程来归档

### 问题

无法跨数据库备份


## 分区实现
### 问题
-  表中有全文索引,无法分区
- 分区的话,需要修改业务sql,改动较大-->待确定


##  pt-archvier(mysql percona工具集)
- 基础教程

https://yq.aliyun.com/articles/277145

- 是否可归档多表

https://www.percona.com/forums/questions-discussions/percona-toolkit/32449-pt-archiver-multiple-dependent-tables

- 可以多表条件,但是无法如果中途失败,无法继续归档

https://stackoverflow.com/questions/47203831/turn-mysql-query-into-percona-pt-archiver-string


##  mysql_archiver (python实现+pt-archvier)
https://github.com/dbarun/mysql_archiver
- MySQL_archiver基本上实现了数据归档的自动运转，统一的归档任务调度管理、自动监控和预警、自动生成报表。在一定程度上节约了生产力，提高了运维效率。
- 要知道每个归档任务成功与否、跑了多长时间、归档了多少数据
##   JavaDataArchiver (java实现)
https://github.com/Sunshow/JavaDataArchiver

### 问题
下载源码后,发现未实现

## 阿里云maxcompute

### 优点
相对稳定,成本低,多数据源,可视化直接操作,无需开发,只需要配置
### 问题
- 多表之间的id无法直接传递,需要使用join 同时过滤主表和字表的条件,例如sale_order和sale_order_detail(严重) 0226
- delete的条件传递 in (ids)
- delete的效率问题(性能) 0226

- 是否需要跨数据库备份


## 阿里云maxcompute+归档字段
- 先多表关联,将要归档的记录设置上需归档的状态
- 各表根据归档状态,单标归档
- 删除原表

### 问题
数据量太大,可能update会锁表

## 阿里云maxcompute+归档记录
- 先多表关联,将要归档的记录设置上需归档的状态
- 各表根据归档状态,单标归档
- 删除原表


### 问题
待试验

## 外部程序实现


# 结论
- 使用: mysql_archiver
- !!!!!!后续开发读取归档数据,同时不能影响生产库的性能(最好先定下来,否则目标 会不是mysql)
- target 也需要分区

# 目标库选择
- nogsql
- mysql my
- maxcompute

# 废弃分表策略,对于v_sale_order和v_sale_order_detail

# 结论
- 采用阿里云数据集成,跑归档任务
- 在数据集成任务结束后的回调函数,可以配置删除归档数据的任务
- 暂时目标库,还是mysql,后续的查询采用ElasticSearch进行查询,(阿里云提供1个月的免费试用,待研究确认后执行)






