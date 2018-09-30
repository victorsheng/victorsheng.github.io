---
title: 学习pandas api时发现了一个奇怪的现象
tags:
  - pandas
  - 机器学习
  - jupyter
categories:
  - 机器学习
abbrlink: 1551802988
date: 2017-12-10 16:15:00
---
# 学习pandas api时发现了一个奇怪的现象
<img src="http://pic.victor123.cn/17-12-10/542730.jpg">
- 通过代码
- dates = pd.date_range('20130101', periods=6)
- 定义了函数,
- 下一行无法打印出来,
- 可是下面仍然能够使用这个datas的变脸,很是神奇,百思不得其解

# 结论
- 怀疑是应该是jupyter notebook的问题,
- 直接打印出pd.date_range('20130101', periods=6)的结果都是正确的
<img src="http://pic.victor123.cn/17-12-10/27955116.jpg">