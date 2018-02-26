---
title: manual-sql-separate-purchase-group
date: 2018-02-26 16:34:41
tags:
categories:
---

```
-- 查询出问题的采购单
SELECT *
FROM pms_purchase_order
WHERE pms_purchase_order_no IN ('CG2018-02100061021', 'CG2018-02100061023');

-- 订单为主表,先将要处理的订单,抽出到一个表中
CREATE TABLE `tmp_0226_mlj` AS
  SELECT DISTINCT
    sale_order_id,
    "XXXX" AS group_no
  FROM pms_daily_purchase_detail
  WHERE pms_purchase_order_id IN (216606, 216608) AND single = 0;

-- 由于create table的语法限制,如果 group_no 为null,无法进行update,so 先放一个长的字符串XXXX进去
DESC tmp_0226_mlj;


-- 手动更新组号
UPDATE tmp_0226_mlj
SET group_no = 'C'
WHERE group_no = "XXXX"
LIMIT 200;
UPDATE tmp_0226_mlj
SET group_no = 'D'
WHERE group_no = "XXXX"
LIMIT 200;
UPDATE tmp_0226_mlj
SET group_no = 'E'
WHERE group_no = "XXXX"
LIMIT 200;
UPDATE tmp_0226_mlj
SET group_no = 'F'
WHERE group_no = "XXXX"
LIMIT 200;
UPDATE tmp_0226_mlj
SET group_no = 'G'
WHERE group_no = "XXXX"
LIMIT 200;
UPDATE tmp_0226_mlj
SET group_no = 'H'
WHERE group_no = "XXXX"
LIMIT 200;
UPDATE tmp_0226_mlj
SET group_no = 'M'
WHERE group_no = "XXXX"
LIMIT 200;
UPDATE tmp_0226_mlj
SET group_no = 'N'
WHERE group_no = "XXXX"
LIMIT 200;


-- 预览手动分配的组号
SELECT
  count(1),
  group_no
FROM tmp_0226_mlj
GROUP BY group_no;


SELECT * from pms_daily_purchase_detail WHERE pms_purchase_order_id IN (216606, 216608) AND single = 0;


-- 更新采购单明细
UPDATE pms_daily_purchase_detail, tmp_0226_mlj
SET pms_daily_purchase_detail.group_no = tmp_0226_mlj.group_no
WHERE tmp_0226_mlj.sale_order_id = pms_daily_purchase_detail.sale_order_id
      AND pms_purchase_order_id IN (216606, 216608) AND single = 0;


-- 更新订单
UPDATE sale_order, tmp_0226_mlj
SET sale_order.group_no = tmp_0226_mlj.group_no
WHERE tmp_0226_mlj.sale_order_id = sale_order.sale_order_id;


-- 更新发货单
UPDATE wms_order, tmp_0226_mlj
SET wms_order.group_no = tmp_0226_mlj.group_no
WHERE tmp_0226_mlj.sale_order_id = wms_order.sale_order_id;
```