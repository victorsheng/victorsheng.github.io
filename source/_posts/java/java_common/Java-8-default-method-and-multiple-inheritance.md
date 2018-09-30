---
title: Java-8-default-method-and-multiple-inheritance
abbrlink: 360731080
date: 2018-07-03 16:06:42
tags:
categories:
---
# java8 多继承
在java 接口中出现了default方法后,如果一个类同时实现了2个接口,而两个接口都有相同的default方法,该如何处理呢?

## 多个父接口
```
+---------------+         +------------+
|  Interface A  |         |Interface B |
+-----------^---+         +---^--------+
            |                 |         
            |                 |         
            |                 |         
            +-+------------+--+         
              | Interface C|            
              +------------+
```
A,B拥有相同签名的默认方法default String say(String name), 如果接口C没有override这个方法， 则编译出错。
`interface C inherits unrelated defaults for say(String) from types A and B`
### 解决
```
interface C extends A,B{
	default String say(String name) {
		return "greet " + name;
	}
}
```




## 接口多层继承

```
+---------------+ 
|  Interface A1 | 
+--------+------+ 
         |        
         |        
         |        
+--------+------+ 
|  Interface A2 | 
+-------+-------+ 
        |         
        |         
        |         
+-------+--------+
|   Interface C  |
+----------------+

```

## 多层多继承
```
+---------------+                          
|  Interface A1 |                          
+--------+------+                          
         |                                 
         |                                 
         |                                 
+--------+------+         +---------------+
|  Interface A2 |         |  Interface B  |
+-------+-------+         +---------+-----+
        |       +---------+---------^      
        |       |                          
        |       |                          
+-------+-------++                         
|   Interface C  |                         
+----------------+
```

## 类和接口混杂

```
+-------------+       +-----------+
| Interface A |       |  Class B  |
+-----------+-+       +-----+-----+
            ^-+    +--+-----^      
              |    |               
          +---+----+-+             
          |  Class C |             
          +----------+
```

# 小结
- 类优先于接口。 如果一个子类继承的父类和接口有相同的方法实现。 那么子类继承父类的方法
- 子类型中的方法优先于父类型中的方法。
- 如果以上条件都不满足， 则必须显示覆盖/实现其方法，或者声明成abstract。


# 参考
http://colobu.com/2014/11/04/Java-8-default-method-and-multiple-inheritance/

