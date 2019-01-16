---
title: mysql锁超时处理
tags:
  - mysql
abbrlink: 1001227538
date: 2018-03-10 12:12:00
categories:
---

# 问题
生成采购单逻辑,一直报错,锁等待超时
手动删除这个记录也是报错,应该是锁了pms_daily_group的id为101958的记录

# 解决
```
select l.* from ( select 'Blocker' role, p.id, p.user, left(p.host, locate(':', p.host) - 1) host, tx.trx_id, tx.trx_state, tx.trx_started, timestampdiff(second, tx.trx_started, now()) duration, lo.lock_mode, lo.lock_type, lo.lock_table, lo.lock_index, tx.trx_query, lw.requesting_thd_id Blockee_id, lw.requesting_trx_id Blockee_trx from information_schema.innodb_trx tx, information_schema.innodb_lock_waits lw, information_schema.innodb_locks lo, information_schema.processlist p where lw.blocking_trx_id = tx.trx_id and p.id = tx.trx_mysql_thread_id and lo.lock_id = lw.blocking_lock_id union select 'Blockee' role, p.id, p.user, left(p.host, locate(':', p.host) - 1) host, tx.trx_id, tx.trx_state, tx.trx_started, timestampdiff(second, tx.trx_started, now()) duration, lo.lock_mode, lo.lock_type, lo.lock_table, lo.lock_index, tx.trx_query, null, null from information_schema.innodb_trx tx, information_schema.innodb_lock_waits lw, information_schema.innodb_locks lo, information_schema.processlist p where lw.requesting_trx_id = tx.trx_id and p.id = tx.trx_mysql_thread_id and lo.lock_id = lw.requested_lock_id) l order by role desc, trx_state desc;

kill 184631781
```
发现了未提交的事务

<table border="1" style="border-collapse:collapse">
<tr><th>role</th><th>id</th><th>user</th><th>host</th><th>trx_id</th><th>trx_state</th><th>trx_started</th><th>duration</th><th>lock_mode</th><th>lock_type</th><th>lock_table</th><th>lock_index</th><th>trx_query</th><th>Blockee_id</th><th>Blockee_trx</th></tr>
<tr><td>Blocker</td><td>184631781</td><td>egeniekn</td><td>10.26.109.140</td><td>4994407160</td><td>RUNNING</td><td>2018-03-10 10:39:22</td><td>4996</td><td>X</td><td>RECORD</td><td>`egenie_kn`.`pms_daily_group`</td><td>PRIMARY</td><td>NULL</td><td>184600424</td><td>4996523486</td></tr>
<tr><td>Blockee</td><td>184600424</td><td>egeniekn</td><td>10.25.241.203</td><td>4996523486</td><td>LOCK WAIT</td><td>2018-03-10 12:02:32</td><td>6</td><td>X</td><td>RECORD</td><td>`egenie_kn`.`pms_daily_group`</td><td>PRIMARY</td><td>/* ApplicationName=DataGrip 2017.3 */ UPDATE `egenie_kn`.`pms_daily_group` t SET t.`is_usable` = 0 WHERE t.`pms_daily_group_id` = 101958</td><td>NULL</td><td>NULL</td></tr></table>