---
title: 并发工具之CompletionService
author: Victor
abbrlink: 4177496835
date: 2018-04-16 22:23:47
tags:
categories:
---
# 介绍
当向Executor提交多个任务并且希望获得它们在完成之后的结果，如果用FutureTask，可以循环获取task，并调用get方法去获取task执行结果，但是如果task还未完成，获取结果的线程将阻塞直到task完成，由于不知道哪个task优先执行完毕，使用这种方式效率不会很高。在jdk5时候提出接口CompletionService，它整合了Executor和BlockingQueue的功能，可以更加方便在多个任务执行时获取到任务执行结果。

# 案例

```
public class CompletionServiceTest {

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService executorService = Executors.newFixedThreadPool(100);
        ExecutorCompletionService<Integer> completionService = new ExecutorCompletionService<Integer>(executorService);
        for (int i = 0; i < 10; i++) {
            completionService.submit(new Callable<Integer>() {
                @Override
                public Integer call() throws Exception {
                    TimeUnit.SECONDS.sleep(5);
                    int i1 = new Random().nextInt();
                    System.out.println(i1);
                    return i1;
                }
            });
        }
        int sum = 0;
        for (int i = 0; i < 10; i++) {
            Future<Integer> poll = completionService.take();
            Integer integer = poll.get();
            sum += integer;
        }
        System.out.println(sum);

    }
}
```


# 源码分析

![upload successful](/images/pasted-153.png)
ExecutorCompletionService有三个成员变量：

- executor：执行task的线程池，创建CompletionService必须指定；

- aes：主要用于创建待执行task；

- completionQueue：存储已完成状态的task，默认是基于链表结构的阻塞队列LinkedBlockingQueue。


从submit方法的源码可以看出，在提交到线程池的时候需要将FutureTask封装成QueueingFuture
