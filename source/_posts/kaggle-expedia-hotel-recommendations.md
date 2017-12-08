---
title: kaggle-expedia-hotel-recommendations
date: 2017-12-05 16:31:06
tags:
categories:
---

# 题目地址
https://www.kaggle.com/c/expedia-hotel-recommendations#description

# 学习地址
https://www.dataquest.io/blog/kaggle-tutorial/

# 学习心得
- 数据探索
``` python
import pandas as pd

destinations = pd.read_csv("destinations.csv")
test = pd.read_csv("test.csv")
train = pd.read_csv("train.csv")
```
    + 数据量
    + 数据分布
- 目标变量
    + 目标变量是什么
    + 探索目标变量
    + 探索用户id 
- 采样统计
    + 添加时间和日期
    + 挑选1000名用户
    + 区分测试集和验证集
    + 移除无关数据(click 事件)
- 一个简单的算法
    + 生成预测
    + 评估效果
    + 查找相关因素
- 更近一步
    + 生成特征
        * 从destinations生成特征
        * 生成其他特征
    + 机器学习
        * 随机森林
        * 二分分类
- 在目的地下的最受欢迎的酒店选择
    + 给予目的地,进行预测
    + 评估错误
- 更好的结果
    + 根据用户
    + 合并预测
- 生成提交文件
- 总结
    + 下一步


