---
title: unittest-powermock
date: 2018-08-10 19:37:41
tags:
categories:
---
# 介绍
EasyMock 以及 Mockito 都因为可以极大地简化单元测试的书写过程而被许多人应用在自己的工作中，但是这 2 种 Mock 工具都不可以实现对静态函数、构造函数、私有函数、Final 函数以及系统函数的模拟，但是这些方法往往是我们在大型系统中需要的功能。PowerMock 是在 EasyMock 以及 Mockito 基础上的扩展，通过定制类加载器等技术，PowerMock 实现了之前提到的所有模拟功能，使其成为大型系统上单元测试中的必备工具。

# 初次使用

## maven引用
```
 <!-- https://mvnrepository.com/artifact/org.mockito/mockito-all -->
    <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>mockito-all</artifactId>
      <version>1.10.19</version>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.powermock/powermock-module-junit4 -->
    <dependency>
      <groupId>org.powermock</groupId>
      <artifactId>powermock-module-junit4</artifactId>
      <version>1.6.6</version>
      <scope>test</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.powermock/powermock-api-mockito -->
    <dependency>
      <groupId>org.powermock</groupId>
      <artifactId>powermock-api-mockito</artifactId>
      <version>1.6.6</version>
      <scope>test</scope>
    </dependency>
```

## 模拟 Static 方法

## 模拟构造函数

## 模拟私有以及 Final 方法


## Mock(全是是假的) vs. Spy(默认是真的)
最后，我们来讨论一下在Mokita中Mock和Spy有什么区别。 当Mockito创造一个mock对象时，它是从类的类型中创建，而不是从类的实例中创建。也就是说mock对象是一个简单的只带有骨架的空壳的对象实例，同时我们可以跟踪它的所有交互方法。 不同的是，spy对象是对一个对象实例进行的封装，它仍然能够像正常对象实例一样有这正常的行为，唯一不同的是这些行为交互是可以跟踪和配置的。

# 参考
https://www.ibm.com/developerworks/cn/java/j-lo-powermock/index.html