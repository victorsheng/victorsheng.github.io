title: CountDownLatch学习
tags:
  - 多线程
date: 2018-04-02 11:02:32
categories:
---
# 简介
CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。

![upload successful](/images/pasted-115.png)


# 一个countdownlatch的使用案例
## 题目
n个赛车，让他们都在起跑线上就绪后，同时出发,并监控结束比赛的结束


## 答案
```
public class CountDownTest {
    private static class ProcessJob implements Runnable {


        private CountDownLatch startLatch;
        private CountDownLatch stopLatch;

        public ProcessJob(CountDownLatch startLatch, CountDownLatch stopLatch) {
            this.startLatch = startLatch;
            this.stopLatch = stopLatch;
        }

        @Override
        public void run() {
            try {
                startLatch.await();
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("Some Job");
            stopLatch.countDown();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        int jobSize = 20;
        CountDownLatch startLatch = new CountDownLatch(1);
        CountDownLatch stopLatch = new CountDownLatch(jobSize);
        for (int i = 0; i < jobSize; i++) {
            executorService.submit(new ProcessJob(startLatch, stopLatch));
        }
        long start = System.currentTimeMillis();
        startLatch.countDown();
        // 等待所有线程执行完任务
        stopLatch.await();
        long end = System.currentTimeMillis();
        System.out.println("Total cost time: " + (end - start));
        executorService.shutdown();
        executorService.awaitTermination(1, TimeUnit.MINUTES);
    }
}
```


# 核心代码
```
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }

        protected int tryAcquireShared(int acquires) {
       		 // 只有当当前state为0即count为0的时候能够返回
            return (getState() == 0) ? 1 : -1;
        }

        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                // 成功CAS将count变为0的线程负责唤醒等待线程
                    return nextc == 0;
            }
        }
    }
```


# 重写的aqs接口
## tryAcquireShared
```
protected int tryAcquireShared(int arg)
试图在共享模式下获取对象状态。此方法应该查询是否允许它在共享模式下获取对象状态，如果允许，则获取它。
此方法总是由执行 acquire 线程来调用。如果此方法报告失败，则 acquire 方法可以将线程加入队列（如果还没有将它加入队列），直到获得其他某个线程释放了该线程的信号。

默认实现将抛出 UnsupportedOperationException。

参数：
arg - acquire 参数。该值总是传递给 acquire 方法的那个值，或者是因某个条件等待而保存在条目上的值。该值是不间断的，并且可以表示任何内容。
返回：
在失败时返回负值；如果共享模式下的获取成功但其后续共享模式下的获取不能成功，则返回 0；如果共享模式下的获取成功并且其后续共享模式下的获取可能够成功，则返回正值，在这种情况下，后续等待线程必须检查可用性。（对三种返回值的支持使得此方法可以在只是有时候以独占方式获取对象的上下文中使用。）在成功的时候，此对象已被获取。
抛出：
IllegalMonitorStateException - 如果正在进行的获取操作将在非法状态下放置此同步器。必须以一致的方式抛出此异常，以便同步正确运行。
UnsupportedOperationException - 如果不支持共享模式
```

## tryReleaseShared

```

protected boolean tryReleaseShared(int arg)
试图设置状态来反映共享模式下的一个释放。
此方法总是由正在执行释放的线程调用。

默认实现将抛出 UnsupportedOperationException。

参数：
arg - release 参数。该值总是传递给 release 方法的那个值，或者是因某个条件等待而保存在条目上的当前状态值。该值是不间断的，并且可以表示任何内容。
返回：
如果此对象现在处于完全释放状态，从而使正在等待的线程都可以试图获得此对象，则返回 true；否则返回 false
抛出：
IllegalMonitorStateException - 如果正在进行的释放操作将在非法状态下放置此同步器。必须以一致的方式抛出此异常，以便同步正确运行
UnsupportedOperationException - 如果不支持共享模式
```


## 外部调用
await()调用acquireShared的不可中断的方法acquireSharedInterruptibly，acquireSharedInterruptibly又会调用到上面Sync实现的tryAcquire方法判断是否能够返回还是继续等待。

countDown调用sync的releaseShared方法，releaseShared方法又会调用上面Sync实现的tryReleaseShared方法根据返回结果判定是否唤醒线程


# 参考文档
https://liuzhengyang.github.io/2017/05/22/countdownlatch/
