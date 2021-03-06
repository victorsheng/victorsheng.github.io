---
title: JUC-Double-check
abbrlink: 3473552964
date: 2018-04-05 21:22:46
tags:
categories:
---

双重检查锁定在延迟初始化的单例模式中见得比较多（单例模式实现方式很多，这里为说明双重检查锁定问题，只选取这一种方式），先来看一个版本：
## 版本1
 
```
public class Singleton {

    private static Singleton instance = null;

    private Singleton(){}

   

    public static Singleton  getInstance() {

       if(instance == null) {

           instance = new Singleton();

       }

       return instance;

    }

}
```

上面是最原始的模式，一眼就可以看出，在多线程环境下，可能会产生多个Singleton实例，于是有了其同步的版本：
## 版本2

```
public class Singleton {

    private static Singleton instance = null;

    private Singleton(){}

   

    public synchronized static Singleton getInstance() {

       if(instance == null) {

           instance = new Singleton();

       }

       return instance;

    }

}
```

在这个版本中，每次调用getInstance都需要取得Singleton.class上的锁，然而该锁只是在开始构建Singleton 对象的时候才是必要的，后续的多线程访问，效率会降低，于是有了接下来的版本：

## 版本3

```
public class Singleton {

    private static Singleton instance = null;

    private Singleton(){}

   

    public static Singleton getInstance() {

       if(instance == null) {

           synchronized(Singleton.class) {

              if(instance == null) {

                  instance = new Singleton();

              }

           }

       }

       return instance;

    }

}
```

很好的想法！不幸的是，该方案也未能解决问题之根本：

 

原因在于：初始化Singleton  和 将对象地址写到instance字段 的顺序是不确定的。在某个线程new Singleton()时，在构造方法被调用之前，就为该对象分配了内存空间并将对象的字段设置为默认值。此时就可以将分配的内存地址赋值给instance字段了，然而该对象可能还没有初始化；此时若另外一个线程来调用getInstance，取到的就是状态不正确的对象。

 

鉴于以上原因，有人可能提出下列解决方案：

## 版本4

```
public class Singleton {

    private static Singleton instance = null;

    private Singleton(){}

   

    public static Singleton getInstance() {

       if(instance == null) {

           Singleton temp;

           synchronized(Singleton.class) {

              temp = instance;

              if(temp == null) {

                  synchronized(Singleton.class) {

                     temp = new Singleton();

                  }

                  instance = temp;

              }

           }

       }

       return instance;

    }

}
```

该方案将Singleton对象的构造置于最里面的同步块，这种思想是在退出该同步块时设置一个内存屏障，以阻止初始化Singleton  和 将对象地址写到instance字段 的重新排序。

 

不幸的是，这种想法也是错误的，同步的规则不是这样的。退出监视器（退出同步）的规则是：所以在退出监视器前面的动作都必须在释放监视器之前完成。然而，并没有规定说退出监视器之后的动作不能放到退出监视器之前完成。*也就是说同步块里的代码必须在退出同步时完成，而同步块后面的代码则可以被编译器或运行时环境移到同步块中执行。*

 

编译器可以合法的，也是合理的，将instance = temp移动到最里层的同步块内，这样就出现了上个版本同样的问题。

 

在JDK1.5及其后续版本中，扩充了volatile语义，系统将不允许对 写入一个volatile变量的操作与其之前的任何读写操作 重新排序，也不允许将 读取一个volatile变量的操作与其之后的任何读写操作 重新排序。

 

在jdk1.5及其后的版本中，可以将instance 设置成volatile以让双重检查锁定生效，如下：

## 版本5(volatile)

```
public class Singleton {

    private static volatile Singleton instance = null;

    private Singleton(){}

   

    public static Singleton getInstance() {

       if(instance == null) {

           synchronized(Singleton.class) {

              if(instance == null) {

                  instance = new Singleton();

              }

           }

       }

       return instance;

    }

}
```