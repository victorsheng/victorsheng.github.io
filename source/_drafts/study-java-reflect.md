---
title: study-java-reflect
abbrlink: 481594819
tags:
categories:
---
http://blog.csdn.net/javazejian/article/details/70768369
# 属性

getField获得类中指定的public属性；
getDeclaredField返回指定类中指定的属性（任何可见性）

# 方法

- 继承的method无法通过子类的getDeclaredMethods方法获取到
- 父类获取的实例或者静态method,调用子类示例的method.invke可以调用到父类的或者子类重写的方法
- 静态和示例方法即使重名也不会相冲突
