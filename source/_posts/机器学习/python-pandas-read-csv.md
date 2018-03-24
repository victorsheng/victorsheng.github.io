---
title: pandas中read_csv方法的学习
date: 2017-12-05 23:24:55
tags:
categories:
    - 机器学习
---
#read_csv内容过大的处理方式:
```
1.
train = pd.read_csv("/Users/victor/code/kaggle/Expedia/input/train.csv",nrows=3000)

2.
chunksize=100
```


# 总api
http://pandas.pydata.org/pandas-docs/version/0.15