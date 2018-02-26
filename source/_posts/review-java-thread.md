title: java-线程复习
tags:
  - java
  - 多线程
categories:
  - java
date: 2018-01-13 10:13:27
---
# 线程
创建线程有两种方式：
一、继承 Thread 类，扩展线程。
二、实现 Runnable 接口。

Callable接口
Future接口
FutureTask实现了 Future 接口
FutureTask 的好处是 FutureTask 是为了弥补 Thread 的不足而设计的，它可以让程序员准确地知道线程什么时候执行完成并获得到线程执行完成后返回的结果。FutureTask 是一种可以取消的异步的计算任务，它的计算是通过 Callable 实现的，它等价于可以携带结果的 Runnable，并且有三个状态：等待、运行和完成。完成包括所有计算以任意的方式结束，包括正常结束、取消和异常。


# 多线程
多线程的概念很好理解就是多条线程同时存在，但要用好多线程确不容易，涉及到多线程间通信，多线程共用一个资源等诸多问题。
## synchronized

解决多个线程之间访问资源的同步性

- 对于同步方法，锁是当前实例对象。
- 对于静态同步方法，锁是当前对象的Class对象。
- 对于同步方法块，锁是Synchonized括号里配置的对象。

锁提供了两种主要特性：互斥（mutual exclusion） 和可见性（visibility）。互斥即一次只允许一个线程持有某个特定的锁，因此可使用该特性实现对共享数据的协调访问协议，这样，一次就只有一个线程能够使用该共享数据。可见性要更加复杂一些，它必须确保释放锁之前对共享数据做出的更改对于随后获得该锁的另一个线程是可见的 —— 如果没有同步机制提供的这种可见性保证，线程看到的共享变量可能是修改前的值或不一致的值，这将引发许多严重问题。

## volatile

解决变量在多个线程之间的可见性

出于简易性或可伸缩性的考虑，您可能倾向于使用 volatile 变量而不是锁。当使用 volatile 变量而非锁时，某些习惯用法（idiom）更加易于编码和阅读。此外，volatile 变量不会像锁那样造成线程阻塞，因此也很少造成可伸缩性问题。在某些情况下，如果读操作远远大于写操作，volatile 变量还可以提供优于锁的性能优势。

### 正确使用 volatile 变量的条件

您只能在有限的一些情形下使用 volatile 变量替代锁。要使 volatile 变量提供理想的线程安全，必须同时满足下面两个条件：

对变量的写操作不依赖于当前值。
该变量没有包含在具有其他变量的不变式中。
实际上，这些条件表明，可以被写入 volatile 变量的这些有效值独立于任何程序的状态，包括变量的当前状态。

第一个条件的限制使 volatile 变量不能用作线程安全计数器。虽然增量操作（x++）看上去类似一个单独操作，实际上它是一个由读取－修改－写入操作序列组成的组合操作，必须以原子方式执行，而 volatile 不能提供必须的原子特性。实现正确的操作需要使 x 的值在操作期间保持不变，而 volatile 变量无法实现这点。（然而，如果将值调整为只从单个线程写入，那么可以忽略第一个条件。）

大多数编程情形都会与这两个条件的其中之一冲突，使得 volatile 变量不能像 synchronized 那样普遍适用于实现线程安全。清单 1 显示了一个非线程安全的数值范围类。它包含了一个不变式 —— 下界总是小于或等于上界。

### 小结

与锁相比，Volatile 变量是一种非常简单但同时又非常脆弱的同步机制，它在某些情况下将提供优于锁的性能和伸缩性。如果严格遵循 volatile 的使用条件 —— 即变量真正独立于其他变量和自己以前的值 —— 在某些情况下可以使用 volatile 代替 synchronized 来简化代码。然而，使用 volatile 的代码往往比使用锁的代码更加容易出错。本文介绍的模式涵盖了可以使用 volatile 代替 synchronized 的最常见的一些用例。遵循这些模式（注意使用时不要超过各自的限制）可以帮助您安全地实现大多数用例，使用 volatile 变量获得更佳性能。



## wait()、notify()、notifyAll()
wait():我的部分已经做完了,等别人让我做的时候再做,释放锁
notify():等我做完了,释放锁后,让其他一个人做
wait() 与 Thread.sleep(long time) 的区别:sleep傻等,sleep不释放锁


## ThreadLocal 变量
线程隔离
InheritableTreadLocal让子线程获取父线程中取得值

## join() 方法
等待线程对象销毁
等待子进程执行完,我再执行
Thread.yield() 方法



# ReentrantLock
## lock()
## unlock()

## condition

### 等待/通知
- awiate() 相当于Object类中的wait()方法
- await(long time,TImeUnit unit) 相当于Object类中的wait()方法
- signal() 相当于Object类中的notify()方法
- sigalAll() 相当于Object类中的notifyAll()方法

### 通知部分线程
Lock lock=new Reentrantock();
Condition conditionA=lock.newCondition();
Condition conditionB=lock.newCondition();

### 公平锁,非公平锁
- 获取锁的顺序是按照线程加锁的顺序来分配的,及先来先得的FIFO先进先出顺序

### getHoldCount(),getQueueLength(),getWaiteQueueLength()
- getHoldCount()查询当前线程保持此锁定的个数,调用lock()方法的次数
- getQueueLength()返回正在等待次锁定的线程估计数
- getWaiteQueueLength(Condition condition)返回等待与此锁定翔安的给定条件Condition的线程估计数


# 线程状态切换

![upload successful](/images/pasted-42.png)
- 监视器代表synchronized
- lock代表锁



# AtomicInteger类

# ReentrantReadWriteLock


# 参考
https://www.ibm.com/developerworks/cn/java/j-jtp06197.html