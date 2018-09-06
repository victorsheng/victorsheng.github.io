---
title: study-dump-map
date: 2018-09-03 18:23:44
tags:
categories:
---
# 官方文档
http://wiki.eclipse.org/MemoryAnalyzer

# Histogram直方图
Histogram：列出内存中的对象，对象的个数以及大小

## Shallow heap
Shallow size就是对象本身占用内存的大小，不包含其引用的对象。

* 常规对象（非数组）的Shallow size有其成员变量的数量和类型决定。
* 数组的shallow size有数组元素的类型（对象类型、基本类型）和数组长度决定

因为不像c++的对象本身可以存放大量内存，java的对象成员都是些引用。真正的内存都在堆上，看起来是一堆原生的byte[], char[], int[]，所以我们如果只看对象本身的内存，那么数量都很小。所以我们看到Histogram图是以Shallow size进行排序的，排在第一位第二位的是byte，char 。


它按类名将所有的实例对象列出来，可以点击表头进行排序,在表的第一行可以输入正则表达式来匹配结果 :
在某一项上右键打开菜单选择 list objects ->with incoming refs 将列出该类的实例：


## Retained Heap
Retained Heap的概念，它表示如果一个对象被释放掉，那会因为该对象的释放而减少引用进而被释放的所有的对象（包括被递归释放的）所占用的heap大小。于是，如果一个对象的某个成员new了一大块int数组，那这个int数组也可以计算到这个对象中。

**相对于shallow heap，Retained heap可以更精确的反映一个对象实际占用的大小（因为如果该对象释放，retained heap都可以被释放）。**


- outgoing references ：表示该对象的出节点（被该对象引用的对象）。
- incoming references ：表示该对象的入节点（引用到该对象的对象）。

## 快速找出某个实例没被释放的原因
可以右健 Path to GC Roots-->exclue all phantom/weak/soft etc. reference
从表中可以看出 PreferenceManager -> … ->HomePage这条线路就引用着这个 HomePage实例。用这个方法可以快速找到某个对象的 GC Root,一个存在 GC Root的对象是不会被 GC回收掉的.

# Histogram 对比


# Dominator Tree
Dominator Tree：列出最大的对象以及其依赖存活的Object （大小是以Retained Heap为标准排序的）
Dominator Tree：对象之间dominator关系树。如果从GC Root到达Y的的所有path都经过X，那么我们称X dominates Y，或者X是Y的Dominator Dominator Tree由系统中复杂的对象图计算而来。从MAT的dominator tree中可以看到占用内存最大的对象以及每个对象的dominator。
我们也可以右键选择Immediate Dominator”来查看某个对象的dominator。

## Path to GC Roots

查看一个对象到RC Roots的引用链
通常在排查内存泄漏的时候，我们会选择exclude all phantom/weak/soft etc.references,
意思是查看排除虚引用/弱引用/软引用等的引用链，因为被虚引用/弱引用/软引用的对象可以直接被GC给回收，我们要看的就是某个对象否还存在Strong 引用链（在导出HeapDump之前要手动出发GC来保证），如果有，则说明存在内存泄漏，然后再去排查具体引用。


# 查找导致内存泄漏的类
## 方式一：

1.查找目标类
如果在开发过程中，你的目标很明确，比如就是查找自己负责的Activity，那么通过包名或者Class筛选，OQL搜索都可以快速定位到
OQL：
点击OQL图标,在窗口输入select * from instanceof android.app.Activity 并按Ctrl + F5或者!按钮执行
2.Paths to GC Roots：exclude all phantom/weak/soft etc.references
查看一个对象到RC Roots是否存在引用链。要将虚引用/弱引用/软引用等排除，因为被虚引用/弱引用/软引用的对象可以直接被GC给回收.

3.分析具体的引用为何没有被释放，并进行修复

## 小技巧：

- 当目的不明确时，可以直接定位到RetainedHeap最大的Object，Select incoming references ，查看引用链，定位到可疑的对象然后Path to GC Roots进行引用链分析
- 如果大对象筛选看不出区别，可以试试按照class分组，再寻找可疑对象进行GC引用链分析
- 直接按照包名直接查看GC引用链，可以一次性筛选多个类，但是如下图所示，选项是 Merge Shortest Path to GCRoots，这个选项具体不是很明白，不过也能筛选出存在GC引用链的类，这种方式的准确性还待验证。


# 集合使用率分析
集合在开发中会经常使用到，如何选择合适的数据结构的集合，初始容量是多少（太小，可能导致频繁扩容），太大，又会开销跟多内存。当这些问题不是很明确时或者想查看集合的使用情况时，可以通过MAT来进行分析。

1. 筛选目标对象
2. Show Retained Set（查找当X被回收时那些将被GC回收的对象集合）
3. 筛选指定的Object（Hash Map，ArrayList）并按照大小进行分组
4. 查看指定类的Immediate dominators
5. Collections fill ratio这种方式只能查看那些具有预分配内存能力的集合，比如HashMap，ArrayList。计算方式：”size / capacity”


# Hash相关性能分析
检测每一个HashMap或者HashTable实例并按照碰撞率排序
碰撞率 = 碰撞的实体/Hash表中所有实体

1. 通过HashEntries查看key value


# 通过OQL查询empty并且未修改过的集合：
select * from java.util.ArrayList where size=0 and modCount=0
类似的
select * from java.util.HashMap where size=0 and modCount=0
select * from java.util.Hashtable where count=0 and modCount=0
1. 通过OQL查询empty并且未修改过的集合
2. Immediate dominators(查看引用者)
3. 计算空集合的Retained Size值，查看浪费了多少内存
4. 




# 参考
http://www.lightskystreet.com/2015/09/01/mat_usage/
http://www.androidperformance.com/2015/04/11/AndroidMemory-Usage-Of-MAT/