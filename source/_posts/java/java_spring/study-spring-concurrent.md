---
title: study-spring-concurrent
author: Victor
abbrlink: 3317845941
date: 2018-03-19 23:35:45
tags:
categories:
---
# Spring并发访问的线程安全性问题

- 只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域。
那么对于有状态的bean呢？
Spring对一些（如RequestContextHolder、TransactionSynchronizationManager、LocaleContextHolder等）中非线程安全状态的bean采用ThreadLocal进行处理，让它们也成为线程安全的状态，因此有状态的Bean就可以在多线程中共享了。
- 如果用有状态的bean，也可以使用用prototype模式，每次在注入的时候就重新创建一个bean，在多线程中互不影响。

## RequestContextHolder
我们可以知道HttpServletRequest是在执行doService方法之前，也就是具体的业务逻辑前进行设置的，然后在执行完业务逻辑或者抛出异常时重置RequestContextHolder移除当前的HttpServletRequest。