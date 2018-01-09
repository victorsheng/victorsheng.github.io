title: java集合并发修改
tags:
  - java
categories: []
date: 2018-01-05 11:21:00
---
# 并发修改
当一个或多个线程正在遍历一个集合Collection，此时另一个线程修改了这个集合的内容（添加，删除或者修改）.这就是并发修改

# 快速失败（fail—fast）
- "快速失败”，它是Java集合的一种错误检测机制。当多个线程对集合进行结构上的改变的操作时，有可能会产生fail-fast机制。记住是有可能，而不是一定。例如：假设存在两个线程（线程1、线程2），线程1通过Iterator在遍历集合A中的元素，在某个时候线程2修改了集合A的结构（是结构上面的修改，而不是简单的修改集合元素的内容），那么这个时候程序就会抛出ConcurrentModificationException 异常，从而产生fail-fast机制。

# 何时触发快速失败
- 无论add、remove、clear方法只要是涉及了改变ArrayList元素的个数的方法都会导致modCount的改变。所以我们这里可以初步判断由于expectedModCount得值与modCount的改变不同步，导致两者之间不等从而产生fail-fast机制。

# 安全失败（fail—safe）
- 采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。
- p.s.迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的

# 快速失败和安全失败的比较

|               | Fail Fast Iterator           |    Fail Safe Iterator  |
| ------------- |:-------------:| -----:|
| Throw ConcurrentModification Exception      | Yes | No |
| Clone object      | No      |   Yes |
| Memory Overhead         | No      |   Yes |
| Examples | HashMap,Vector,ArrayList,HashSet      |    CopyOnWriteArrayList,ConcurrentHashMap |

- 产生fail-fast的是在java.util包中的collection实现类；产生fail-safe的是在java.util.concurrent包中的collection实现类