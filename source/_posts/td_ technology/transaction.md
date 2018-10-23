---
title: transaction
date: 2018-10-22 11:02:34
tags:
categories:
---
# 什么是事务
在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务。

对于Innodb只存在是否手动提交的设置,不存在没有事务,"没有事务"一般是自动提交

在 MySQL 命令行的默认设置下，事务都是自动提交的，即执行 SQL 语句后就会马上执行 COMMIT 操作。因此要显式地开启一个事务务须使用命令 BEGIN 或 START TRANSACTION，或者执行命令 SET AUTOCOMMIT=0，用来禁止使用当前会话的自动提交。

## 事务控制语句：
BEGIN或START TRANSACTION；显式地开启一个事务；

COMMIT；也可以使用COMMIT WORK，不过二者是等价的。COMMIT会提交事务，并使已对数据库进行的所有修改成为永久性的；

ROLLBACK；有可以使用ROLLBACK WORK，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；

SAVEPOINT identifier；SAVEPOINT允许在事务中创建一个保存点，一个事务中可以有多个SAVEPOINT；

RELEASE SAVEPOINT identifier；删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；

ROLLBACK TO identifier；把事务回滚到标记点；

SET TRANSACTION；用来设置事务的隔离级别。InnoDB存储引擎提供事务的隔离级别有READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ和SERIALIZABLE。

## MYSQL 事务处理主要有两种方法：
1、用 BEGIN, ROLLBACK, COMMIT来实现

BEGIN 开始一个事务
ROLLBACK 事务回滚
COMMIT 事务确认
2、直接用 SET 来改变 MySQL 的自动提交模式:

SET AUTOCOMMIT=0 禁止自动提交
SET AUTOCOMMIT=1 开启自动提交

# @Transactional readonly?

`START TRANSACTION READ ONLY`

InnoDB可以避免与已知为只读的事务设置事务ID（TRX_ID字段）相关的开销。 只有可能执行写操作或锁定读取的事务（例如SELECT ... FOR UPDATE）才需要事务ID。 消除不必要的事务ID会减少每次查询或数据更改语句构造读取视图时所咨询的内部数据结构的大小。



https://dev.mysql.com/doc/refman/5.7/en/innodb-performance-ro-txn.html

# web开启事务管理


# 参考
http://www.runoob.com/mysql/mysql-transaction.html