---
title: 极客时间-学习笔记-人工智能基础课-线性回归
tags:
  - 人工智能
  - 线性回归
categories:
  - 人工智能
abbrlink: 1075302233
date: 2018-01-03 07:13:39
---
# 概念
- 线性回归假设输出变量是若干输入变量的线性组合,并根据这一关系求解线性组合中的最优系数.

# 误差
- 在线性回归中,误差误差是以军方误差来定义的

# 求解
- 当线性回归的模型为二维平面上的直线时,军方误差就是预测输出和真实输出之间的欧几里得距离
- 求解方法是:最小二乘法

# 防止过拟合
- 存在多个最优解,意味着存在拟合,要解决过拟合问题,常见的做法是正则化,即添加额外的惩罚项,可分为两种:灵回过和LASSO回归
- 岭回归又被称作"参数衰减"
- LASSO回归,全程是"最小绝对缩减和选择算子"
- 与岭回归相比,LASSO回归的特点在于稀疏性的引入,时间花复杂问题的常用方法,在数据压缩,信号处理等其他领域中已有广泛的应用
