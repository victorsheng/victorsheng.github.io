---
title: 有限状态机的应用
abbrlink: 1280851884
date: 2018-04-21 15:17:19
tags:
categories:
---
# 应用

## 前端:
https://taobaofed.org/blog/2012/10/11/fsm-based-component-design-and-implementation/ 

![upload successful](/images/pasted-160.png)

![upload successful](/images/pasted-161.png)

![upload successful](/images/pasted-162.png)

流程图:
![upload successful](/images/pasted-159.png)

## 游戏

![upload successful](/images/pasted-163.png)

## 转门

以一个转门为例，这种专门在一些会展、博物馆、公园的门口很常见，顾客可以投币或者刷卡刷票进入，我们下面以投币(Coin)统称这个触发事件。如果你不投币，闸门是锁着的，你推不动它的转臂，而且投一次币只能进去一个人，过去之后闸门又是锁着的

![upload successful](/images/pasted-164.png)

它有两个状态：Locked、Unlocked。有两个输入（input）会影响它的状态，投币(coin)和推动转臂（push）。

- 在Locked状态， push没有作用。不管比push多少次闸门的状态还是lock
- 在Locked状态，投币会让闸门开锁，闸门可以让一个人通过
- 在Unlocked状态，投币不起作用，闸门还是开着
- 在Unlocked状态，如果有人push通过，人通过后闸门会由Unlocked状态转变成Locked状态。
这是一个简单的闸门的状态转换，却是一个很好的理解状态的典型例子。


