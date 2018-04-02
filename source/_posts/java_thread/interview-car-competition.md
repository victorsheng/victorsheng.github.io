title: 赛跑比赛实现
date: 2018-03-27 22:02:00
tags:
  - 多线程
categories:
---
# 题目
给你n个赛车，让他们都在起跑线上就绪后，同时出发，让你用Java多线程的技术把这种情况写出来。


# 答案
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
