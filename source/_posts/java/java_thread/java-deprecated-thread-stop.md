title: 为什么废弃thread类的stop方法及java中的中断概念
tags:
  - 多线程
date: 2018-03-28 14:27:14
categories:
---
# 对比测试
```
package com.example.demo.thread;

public class ThreadStopTest {


    public static void main(String[] args) {

        try {
            System.out.println("try");
            Thread thread = new MyThread();
            thread.start();
            Thread.sleep(700L);
            thread.stop();
//             thread.interrupt();
        } catch (Exception e) {
            System.out.println("exception");
        } finally {
            System.out.println("finally");
        }
    }
}

class MyThread extends Thread {

    public void run() {
        try {
            System.out.println("run");
            Thread.sleep(1000L);
            throw new Exception();
        } catch (Exception e) {
            System.out.println("exception ");
        }
    }
}

```

console输出的结果是：
try
run
finally

可以看到，stop终结一个线程，并且释放监控线程的所有资源。对于主线程来说，并不能再跟踪线程的运行状况，当线程出现异常也不能被捕获。而其他线程并不知道被stop的线程出现了异常。这样导致状态不一致的情况产生。

注释掉stop 一行，换用interrupt,进行测试。输出结果是：
try
finally
run
exception

catch部分无法处理ThreadDeath异常

# 探究
##stop方法内部实现:

```
stop0(new ThreadDeath());

public class ThreadDeath extends Error {
    private static final long serialVersionUID = -4417128565033088268L;
}
```
 ThreadDeath 悄无声息的杀死及其他线程。因此，用户得不到程序可能会崩溃的警告。崩溃会在真正破坏发生后的任意时刻显现，甚至在数小时或数天之后。


## 为何不正确
首先应该明确一点：一个线程不应该由其他线程来强制中断或停止，而是应该由线程自己自行停止。所以Thread.stop(),（同样的原因还有Thread.suspend(), Thread.resume()）都已经被废弃了。

### 一、资源回收问题
线程A调用线程B的stop方法去停止线程B，调用这个方法的时候线程A其实并不知道线程B执行的具体情况，这种突然地停止会导致线程B的一些清理工作无法完成。

比如说假如线程B正在执行数据库操作，线程A突然就把线程B给stop掉了，那么线程B中的数据库链接该怎么关闭呢？线程B中方法才执行到一半，那些局部变量要怎么释放呢？

### 二、数据不同步问题
执行stop方法后线程B会马上释放锁，这有可能会引发数据不同步问题。
#### 数据不一致
```
假如一个线程正在执行：synchronized void { x = 3; y = 4;}　由于方法是同步的，多个线程访问时总能保证x,y被同时赋值，而如果一个线程正在执行到x = 3;时，被调用了 stop()方法，即使在同步块中，它也干脆地stop了，这样就产生了不完整的残废数据。而多线程编程中最最基础的条件要保证数据的完整性，所以请忘记线程的stop方法，以后我们再也不要说“停止线程”了。而且如果对象处于一种不连贯状态，那么其他线程能在那种状态下检查和修改它们。结果 很难检查出真正的问题所在。

```

#### 锁
```
package test;

public class testLock {

	public static void main(String[] args) {

		final Object lock = new Object();

		Thread thread1 = new Thread() {

			public void run() {

				try {

					synchronized (lock) {

						System.out.println("thread1 acquire lock");

						System.out.println("thread1 release lock");
					}

				} catch (ThreadDeath e) {
					// TODO: handle exception
					e.printStackTrace();
				}
			}
		};

		Thread thread2 = new Thread() {

			public void run() {

				synchronized (lock) {

					System.out.println("thread2 acquire lock");

					System.out.println("thread2 release lock");
				}
			}
		};

		thread1.start();

		thread1.stop();

		thread2.start();
	}
}
```
如果不使用stop方法，结果为：
```
thread1 acquire lock
thread1 release lock
thread2 acquire lock
thread2 release lock
```

这说明thread1和thread2都在正确地征用同一个锁对象。先是thread1获得了lock，然后释放lock。然后thread2获得了lock，然后释放lock。但是如果使用了stop方法，那么结果为：

```
thread2 acquire lock
thread2 release lock
java.lang.ThreadDeath
	at java.lang.Thread.stop(Thread.java:850)
	at test.testLock.main(testLock.java:44)
```
可以看到thread1抛出了java.lang.ThreadDeath错误。同时thread的lock被破坏掉了，只有thread2获得了lock。因为锁被破坏，所以当前线程可能不安全。

# 线程中断
在Java中没有办法立即停止一条线程，然而停止线程却显得尤为重要，如取消一个耗时操作。因此，Java提供了一种用于停止线程的机制——中断。
- 中断只是一种协作机制，Java没有给中断增加任何语法，中断的过程完全需要程序员自己实现。若要中断一个线程，你需要手动调用该线程的interrupted方法，该方法也仅仅是将线程对象的中断标识设成true；接着你需要自己写代码不断地检测当前线程的标识位；如果为true，表示别的线程要求这条线程中断，此时究竟该做什么需要你自己写代码实现。
- 每个线程对象中都有一个标识，用于表示线程是否被中断；该标识位为true表示中断，为false表示未中断；
- 通过调用线程对象的interrupt方法将该线程的标识位设为true；可以在别的线程中调用，也可以在自己的线程中调用。

## 相关方法

- public void interrupt() 将调用者线程的中断状态设为true。
- public boolean isInterrupted() 判断调用者线程的中断状态。
- public static boolean interrupted 只能通过Thread.interrupted()调用。 它会做两步操作：返回当前线程的中断状态；将当前线程的中断状态设为false；

## 正确用法
### 中断处理
#### 处理时机

考虑点:
- 数据一致性
- 效率
- 及时性

#### 处理方式

##### 1、 中断状态的管理

一般说来，当可能阻塞的方法声明中有抛出InterruptedException则暗示该方法是可中断的，如BlockingQueue#put、BlockingQueue#take、Object#wait、Thread#sleep等，如果程序捕获到这些可中断的阻塞方法抛出的InterruptedException或检测到中断后，这些中断信息该如何处理？一般有以下两个通用原则：

入口:
- try catch 阻塞方法声明的 InterruptedException异常
- 手动判断Thread.currentThread.isInterrupted()

出口:
- 如果遇到的是可中断的阻塞方法抛出InterruptedException，可以继续向方法调用栈的上层抛出该异常InterruptedException
- 通过Thread.currentThread.interrupt()来重新设置中断状态。如果是检测并清除了中断状态，亦是如此。

##### 2、 中断的响应
程序里发现中断后该怎么响应？这就得视实际情况而定了。有些程序可能一检测到中断就立马将线程终止，有些可能是退出当前执行的任务，继续执行下一个任务……作为一种协作机制，这要与中断方协商好，当调用interrupt会发生些什么都是事先知道的，如做一些事务回滚操作，一些清理工作，一些补偿操作等。

**若不确定调用某个线程的interrupt后该线程会做出什么样的响应，那就不应当中断该线程。**

### 触发中断

## 上层调用中断的方法

- FutureTask中的cancel方法:如果传入的参数为true，它将会在正在运行异步任务的线程上调用interrupt方法，如果正在执行的异步任务中的代码没有对中断做出响应，那么cancel方法中的参数将不会起到什么效果
- ThreadPoolExecutor中的shutdownNow方法会遍历线程池中的工作线程并调用线程的interrupt方法来中断线程，所以如果工作线程中正在执行的任务没有对中断做出响应，任务将一直执行直到正常结束。


# 参考
http://www.infoq.com/cn/articles/java-interrupt-mechanism#theCommentsSection

 http://ibruce.info/2013/12/19/how-to-stop-a-java-thread/
 https://blog.csdn.net/wei_ya_wen/article/details/8626880
 http://www.xie4ever.com/2017/03/03/java-%E7%A6%81%E6%AD%A2%E4%BD%BF%E7%94%A8thread-stop%E6%96%B9%E6%B3%95/