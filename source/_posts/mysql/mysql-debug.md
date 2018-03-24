title: mysql-debug
tags:
  - mysql
  - like
categories: 
    - mysql
date: 2018-01-24 15:42:00
---
# 问题sql
``` sql
  SELECT
  *
FROM v_sale_order
WHERE 1 = 1 AND v_sale_order.is_usable = 1 AND v_sale_order.tenant_id = 100046 AND 1 = 1 AND archive_state IN (0, 1) AND
#       v_sale_order.buyer_nick LIKE '肆无忌惮演江山%'
     v_sale_order.buyer_nick LIKE 't_150175568%'
ORDER BY v_sale_order.last_update_time DESC;
```

# 如果查询内容带下划线

<table border="1" style="border-collapse:collapse">
<tr><th>id</th><th>select_type</th><th>table</th><th>type</th><th>possible_keys</th><th>key</th><th>key_len</th><th>ref</th><th>rows</th><th>Extra</th></tr>
<tr><td>1</td><td>SIMPLE</td><td>sos</td><td>ref</td><td>idx_sale_order_id,IDX_ARCHIVE_SPLIT,oms_normal,oms_check,oms_suspend,oms_normal_v2,oms_check_v2,oms_suspend_v2,IX_SaleOrderStatus_SyncPlaystate,idx_pay_time,IDX_original_order_id</td><td>IDX_original_order_id</td><td>6</td><td>const,const</td><td>837216</td><td>Using where; Using filesort</td></tr>
<tr><td>1</td><td>SIMPLE</td><td>soc</td><td>ref</td><td>idx_sale_order_id</td><td>idx_sale_order_id</td><td>9</td><td>egenie_kn.sos.sale_order_id</td><td>1</td><td>NULL</td></tr>
<tr><td>1</td><td>SIMPLE</td><td>sow</td><td>ref</td><td>idx_sale_order_id</td><td>idx_sale_order_id</td><td>9</td><td>egenie_kn.sos.sale_order_id</td><td>1</td><td>NULL</td></tr>
<tr><td>1</td><td>SIMPLE</td><td>so</td><td>eq_ref</td><td>PRIMARY</td><td>PRIMARY</td><td>8</td><td>egenie_kn.sos.sale_order_id</td><td>1</td><td>NULL</td></tr>
<tr><td>1</td><td>SIMPLE</td><td>sof</td><td>ref</td><td>idx_sale_order_id</td><td>idx_sale_order_id</td><td>9</td><td>egenie_kn.sos.sale_order_id</td><td>1</td><td>NULL</td></tr>
<tr><td>1</td><td>SIMPLE</td><td>sor</td><td>ref</td><td>idx_sale_order_id,idx_buyer_nick</td><td>idx_sale_order_id</td><td>9</td><td>egenie_kn.sos.sale_order_id</td><td>1</td><td>Using where</td></tr>
<tr><td>1</td><td>SIMPLE</td><td>soi</td><td>ref</td><td>idx_sale_order_id</td><td>idx_sale_order_id</td><td>9</td><td>egenie_kn.sos.sale_order_id</td><td>1</td><td>NULL</td></tr></table>




# 如果查询内容不带下划线


<table border="1" style="border-collapse:collapse">
<tr><th>id</th><th>select_type</th><th>table</th><th>type</th><th>possible_keys</th><th>key</th><th>key_len</th><th>ref</th><th>rows</th><th>Extra</th></tr>
<tr><td>1</td><td>SIMPLE</td><td>sor</td><td>range</td><td>idx_sale_order_id,idx_buyer_nick</td><td>idx_buyer_nick</td><td>303</td><td>NULL</td><td>7</td><td>Using index condition; Using where; Using temporary; Using filesort</td></tr>
<tr><td>1</td><td>SIMPLE</td><td>sos</td><td>eq_ref</td><td>idx_sale_order_id,IDX_ARCHIVE_SPLIT,oms_normal,oms_check,oms_suspend,oms_normal_v2,oms_check_v2,oms_suspend_v2,IX_SaleOrderStatus_SyncPlaystate,idx_pay_time,IDX_original_order_id</td><td>idx_sale_order_id</td><td>8</td><td>egenie_kn.sor.sale_order_id</td><td>1</td><td>Using where</td></tr>
<tr><td>1</td><td>SIMPLE</td><td>soc</td><td>ref</td><td>idx_sale_order_id</td><td>idx_sale_order_id</td><td>9</td><td>egenie_kn.sor.sale_order_id</td><td>1</td><td>NULL</td></tr>
<tr><td>1</td><td>SIMPLE</td><td>sow</td><td>ref</td><td>idx_sale_order_id</td><td>idx_sale_order_id</td><td>9</td><td>egenie_kn.sor.sale_order_id</td><td>1</td><td>NULL</td></tr>
<tr><td>1</td><td>SIMPLE</td><td>so</td><td>eq_ref</td><td>PRIMARY</td><td>PRIMARY</td><td>8</td><td>egenie_kn.sor.sale_order_id</td><td>1</td><td>NULL</td></tr>
<tr><td>1</td><td>SIMPLE</td><td>sof</td><td>ref</td><td>idx_sale_order_id</td><td>idx_sale_order_id</td><td>9</td><td>egenie_kn.sor.sale_order_id</td><td>1</td><td>NULL</td></tr>
<tr><td>1</td><td>SIMPLE</td><td>soi</td><td>ref</td><td>idx_sale_order_id</td><td>idx_sale_order_id</td><td>9</td><td>egenie_kn.sor.sale_order_id</td><td>1</td><td>NULL</td></tr></table>



# 原因
- %代表任意多个字符
- _代表一个字符
- 如果想搜索  _  就要用到转义符 “\” 

# 解决
``` java
value = value.replaceAll("\\_", "\\\\_");
```