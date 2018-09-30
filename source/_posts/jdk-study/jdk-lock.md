---
title: jdk-lock
abbrlink: 7747
date: 2018-09-29 17:36:12
tags:
categories:
---
# 简介
在jdk1.5之后，并发包中新增了Lock接口(以及相关实现类)用来实现锁功能，Lock接口提供了与synchronized关键字类似的同步功能，但需要在使用时手动获取锁和释放锁。虽然Lock接口没有synchronized关键字自动获取和释放锁那么便捷，但Lock接口却具有了锁的可操作性，可中断获取以及超时获取锁等多种非常实用的同步特性，除此之外Lock接口还有两个非常强大的实现类重入锁和读写锁

# 方法

![upload successful](/images/pasted-277.png)

## boolean lock()
 获取锁,调用该方法当前线程将会获取锁，当锁获取后，该方法将返回。

```
try{
//可能会出现线程安全的操作
}finally{
//一定在finally中释放锁
//也不能把获取锁在try中进行，因为有可能在获取锁的时候抛出异常
  lock.ublock();
}
```

## void lockInterruptibly()  throws InterruptedException


## boolean tryLock()
```
/**
     * Acquires the lock only if it is free at the time of invocation.
     *
     * <p>Acquires the lock if it is available and returns immediately
     * with the value {@code true}.
     * If the lock is not available then this method will return
     * immediately with the value {@code false}.
     *
     * <p>A typical usage idiom for this method would be:
     *  <pre> {@code
     * Lock lock = ...;
     * if (lock.tryLock()) {
     *   try {
     *     // 受保护操作
     *   } finally {
     *     lock.unlock();
     *   }
     * } else {
     *   // 执行替代行动
     * }}</pre>
     *
     * This usage ensures that the lock is unlocked if it was acquired, and
     * doesn't try to unlock if the lock was not acquired.
     *
     * @return {@code true} if the lock was acquired and
     *         {@code false} otherwise
     */
    boolean tryLock();
```

## boolean tryLock(long time,TimeUnit unit) throws InterruptedException 
超时获取锁，以下情况会返回：时间内获取到了锁，时间内被中断，时间到了没有获取到锁。

## Condition newCondition()
返回绑定到此 Lock 实例的新 Condition 实例。
**在等待条件前，锁必须由当前线程保持。调用 Condition.await() 将在等待前以原子方式释放锁，并在等待返回前重新获取锁。**

