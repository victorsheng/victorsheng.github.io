---
title: Field injection is not recommended
date: 2018-10-23 15:21:18
tags:
    - idea
categories:
---
# Field injection is not recommended
idea出现了如下警告
`Always use constructor based dependency injection in your beans. Always use assertions for mandatory dependencies`
一直是这样使用的,该如何修改呢??
```
  @Autowired
  private AService aService;
```
https://stackoverflow.com/questions/43174074/clean-code-where-should-autowired-be-applied

stackoverflow上提供了三种方法来避免idea的警告

# 警告的原因
Constructor based injection (is still auto wiring BTW) is better as it is explicit and follows regular OO rules. Whereas field injection does not. 
基于构造函数的注入更好，因为它是明确的并遵循常规的OO规则。而现场注入没有

## 总结:

https://www.baeldung.com/constructor-injection-in-spring
个人认为此种方法最为简介,声明@Component注解,同时构造函数,无需@Autowired

```
@Component
public class Car {
 
    public Car(Engine engine, Transmission transmission) {
        this.engine = engine;
        this.transmission = transmission;
    }
}
```

