title: 智能采购实现逻辑梳理
tags:
  - ''
  - 采购
categories:
  - 业务梳理
date: 2017-11-27 19:05:00
---
# 智能采购实现逻辑梳理
## 销量部分计算(一天执行一次)
- 刷新1天,7天,30天,自定义天销量方法
- 排除掉不采购仓库的销量
- 排除掉备货仓的销量
- 分批进行持久化
- 触发一次采购部分计算

## 采购部分计算(定时半个小时执行一次)
- 获取界面上手动覆盖的参数
- 如果有批量修改勾选的ids则至处理这部分智能采购记录,否则查询全部智能采购记录
    - 如果修改了上限和下限库存,则县持久化
- 填充库存
    + 本仓库库存+本仓库的备货仓的库粗查询
- 计算存货天数
- 取采购周期,供货商
```
         * 根据excel公式,计算备货的字段
         * 日均销量=7天销量/7
         * 最低库存=日均销量*备货天数
         * 上限库存=日均销量*(备货天数+采购周期)
         * 存货预警库存=在途库存-最低库存 P.S.小于0的需要采购
         * 建议采购数量=上限采购数量-在途库存
         */ 
```
- 查询参数
- 强制覆盖采购周期和存货天数
- 计算上限和下限库存
    - Double minStock = avgSaleVolume * inventoryDays;
    - Double maxStock = avgSaleVolume * (inventoryDays + procurementCycle);
- 强制覆盖上限和下限库存
    + 如果开启智能计算跳过此步骤
    + 查询SaleStockSetting表中记录的上限和线下库存,别设置isManual=true
- 计算预警库存和建议采购量
    + Integer saleStock = v.getSaleStock();
        Integer onWayStock = v.getOnWayStock();
        int stock = saleStock + onWayStock;
        double warningStock = stock - minStock;
    + 如果预警库存<0,double originAdviceNum = maxStock - stock,并向上取整
    + 如果预警库存>=0,不进行建议
- 强制覆盖最总结果
- 对verifyNum进行赋值
- 强制覆盖verifyNum
- 持久化更新