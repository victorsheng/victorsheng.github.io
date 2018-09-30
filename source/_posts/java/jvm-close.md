---
title: jvm优雅停机
tags:
  - java
  - jvm
abbrlink: 1024016852
date: 2018-07-07 21:18:52
categories:
---

Java有两种Thread：“守护线程Daemon”与“用户线程User”。
用户线程：Java虚拟机在它所有非守护线程已经离开后自动离开。
守护线程：守护线程则是用来服务用户线程的，如果没有其他用户线程在运行，那么就没有可服务对象，也就没有理由继续下去。

1.将应用自己创建的线程、timer、scheduler这类的资源设为守护线程（daemon）。setDaemon(boolean on)方法可以方便的设置线程的Daemon模式，true为Daemon模式，false为User模式。 
2.自己管理非守护线程的生命周期，当容器停止时手工释放资源。比如你可以在 Servlet 或 ServletContextListener 的 init 方法中初始化资源，在 destroy 方法中释放资源。


# 只有用户进程全部关闭后,才会执行shutdownhook(错误)
http://810364804.iteye.com/blog/2119572

# 区别(是在正常退出的情境下,not 被kill )
守护线程与普通用户线程的区别是：java程序会在所有用户线程都执行完了才结束退出，即使主线程执行完了只要还有用户线程执行程序就在运行。但是如果其他用户线程全部执行完了守护线程如果没执行完的话它会自动被jvm终止，然后结束程序。


## 主线程退出不会影响其他线程,只要用户线程还在执行
```
package com.example.demo.exist;

public class ParentTest {

  public static void main(String[] args) {
    System.out.println("parent thread begin ");

    ChildThread t1 = new ChildThread("thread1");
    ChildThread t2 = new ChildThread("thread2");
    t1.start();
    t2.start();

    System.out.println("parent thread over ");
  }
}

class ChildThread extends Thread {

  private String name = null;

  public ChildThread(String name) {
    this.name = name;
  }

  @Override
  public void run() {
    while (true) {
      System.out.println(this.name + "--child thead begin");

      try {
        Thread.sleep(500);
      } catch (InterruptedException e) {
        System.out.println(e);
      }

      System.out.println(this.name + "--child thead over");
    }
  }
}

```

# 谁来处理关闭信号呢??