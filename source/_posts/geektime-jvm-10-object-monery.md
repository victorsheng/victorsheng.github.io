---
title: geektime-jvm-10-object-monery
date: 2018-08-16 18:36:25
tags:
categories:
---
# Java中创建对象的方式

1. new -通过调用构造器来初始化实例字段
2. 反射-通过调用构造器来初始化实例字段
3. bject.clone-通过直接复制已有的数据，来初始化新建对象的实例字段
4. 反序列化-通过直接复制已有的数据，来初始化新建对象的实例字段
5. Unsafe.allocateInstance-没有初始化对象的实例字段

# new的字节码
以 new 语句为例，它编译而成的字节码将包含用来请求内存的 new 指令，以及用来调用构造器的 invokespecial 指令。
```
// Foo foo = new Foo(); 编译而成的字节码
  0 new Foo
  3 dup
  4 invokespecial Foo()
  7 astore_1
```
