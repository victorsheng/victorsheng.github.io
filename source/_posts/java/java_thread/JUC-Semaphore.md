---
title: AQS-Semaphore
date: 2018-04-02 23:02:38
tags:
categories:
---
# 简介
Semaphore 可以很轻松完成信号量控制，Semaphore可以控制某个资源可被同时访问的个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。

Semaphore 有两个构造函数，参数为许可的个数 permits 和是否公平竞争 fair。通过 acquire 方法能够获得的许可个数为 permits，如果超过了这个个数，就需要等待。当一个线程 release 释放了一个许可后，fair 决定了正在等待的线程该由谁获取许可，如果是公平竞争则等待时间最长的线程获取，如果是非公平竞争则随机选择一个线程获取许可。不传 fair 的构造函数默认采用非公开竞争。

```
Semaphore(int permits)
Semaphore(int permits, boolean fair)
```

# 使用案例
```
public class SemaphoreTest {

    public static void main(String[] args) {

        ExecutorService service = Executors.newCachedThreadPool();//使用并发库，创建缓存的线程池
        final Semaphore sp = new Semaphore(3);//创建一个Semaphore信号量，并设置最大并发数为3

        //availablePermits() //用来获取当前可用的访问次数
        System.out.println("初始化：当前有" + (3 - sp.availablePermits() + "个并发"));

        //创建10个任务，上面的缓存线程池就会创建10个对应的线程去执行
        for (int index = 0; index < 10; index++) {
            final int NO = index;  //记录第几个任务
            Runnable run = new Runnable() {  //具体任务
                public void run() {
                    try {
                        sp.acquire();  // 获取许可
                        System.out.println(Thread.currentThread().getName()
                                + "获取许可" + "("+NO+")，" + "剩余：" + sp.availablePermits());
                        Thread.sleep(1000);
                        // 访问完后记得释放 ，否则在控制台只能打印3条记录，之后线程一直阻塞
                        sp.release();  //释放许可
                        System.out.println(Thread.currentThread().getName()
                                + "释放许可" + "("+NO+")，" + "剩余：" + sp.availablePermits());
                    } catch (InterruptedException e) {
                    }
                }
            };
            service.execute(run);  //执行任务
        }
        service.shutdown(); //关闭线程池
    }
}
```

# 源码解析

Semaphore 通过 AQS中的 state 来进行控制 permit 的获取控制, 其实它就是一个限制数量的 ReadLock
