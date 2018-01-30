title: java-study-queue
tags:
  - 数据结构
categories:
  - java
date: 2018-01-29 23:47:00
---
# BlockingQueue
- 阻塞队列
- 先进先出
- 有边界:当队列满时,插入的元素的线程会等待队列可有
- 等待队列为非空
- 生产者消费者
< !-- more -->

## 五种类
- ArrayBlockingQueue
- LinkedBlockingQueue
- PriorityBlockingQueue
- DelayQueue
- SynchronousQueue:无缓冲的队列,每一个put操作必须等待一个take操作完成,放和取得操作交替完成,否则不能继续添加元素
-