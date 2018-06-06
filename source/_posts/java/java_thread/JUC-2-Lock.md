title: juc的Lock接口
tags:
  - 多线程
date: 2018-04-03 09:03:00
categories:
---
# Lock与synchronized比较
Lock有着和synchronized一样的语意，但是比synchronized多了一些功能，单单就从Lock接口定义来看，比synchronized多出来的功能有：

- 可中断的获取锁，就是获取锁的线程可以响应中断。
- 可以尝试获取锁，也就是非阻塞获取锁，一个线程可以尝试去获取锁，获取成功就持有锁并返回true，否则返回false。
- 带超时的尝试获取锁，就是在尝试获取锁的时候，会有超时时间，当到达了指定的时间后，还未获取到锁，就返回false。

# 方法
![upload successful](/images/pasted-116.png)

```
public interface Lock {

    //获取锁，获取到锁后返回
    //注意一定要记得释放锁
    void lock();

    //可中断的获取锁
    //获取锁的时候，如果线程正在等待获取锁，则该线程能响应中断
    void lockInterruptibly() throws InterruptedException;

    //尝试获取锁，当线程获取锁的时候，获取成功与否都会立即返回
    //不会一直等着去获取锁
    boolean tryLock();

    //带有超时时间的尝试获取锁
    //在一定的时间内获取到锁会返回true
    //在这段时间内被中断了，会返回
    //在这段时间内，没有获取到锁，会返回false
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    //释放锁
    void unlock();

    //获取一个Condition对象。
    Condition newCondition();
}
```

## lockInterruptibly
如果当前线程获得该锁，则将锁保持计数设置为 1。
   如果当前线程：
1）在进入此方法时已经设置了该线程的中断状态；或者
2）在等待获取锁的同时被中断。

   则抛出 InterruptedException，并且清除当前线程的已中断状态。


## tryLock
如果该锁没有被另一个线程保持，并且立即返回 true 值，则将锁的保持计数设置为 1。
```
即使已将此锁设置为使用公平排序策略，但是调用 tryLock() 仍将 立即获取锁（如果有可用的），
而不管其他线程当前是否正在等待该锁。在某些情况下，此“闯入”行为可能很有用，即使它会打破公
平性也如此。如果希望遵守此锁的公平设置，则使用 tryLock(0, TimeUnit.SECONDS)
```

## 总结
lock ： 在锁上等待，直到获取锁；

tryLock：立即返回，获得锁返回true,没获得锁返回false；

tryInterruptibly：在锁上等待，直到获取锁，但是会响应中断，这个方法优先考虑响应中断，而不是响应锁的普通获取或重入获取。 

# Lock接口的实现

Lock接口主要的实现是
- ReentrantLock重入锁
- ConcurrentHashMap中的Segment继承了ReentrantLock
- ReentrantReadWriteLock中的WriteLock和ReadLock也实现了Lock接口

![upload successful](/images/pasted-117.png)
