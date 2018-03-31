title: 赛跑比赛实现
date: 2018-03-27 22:02:00
tags:
categories:
---
# 题目
给你n个赛车，让他们都在起跑线上就绪后，同时出发，让你用Java多线程的技术把这种情况写出来。


# 答案
```
public class Car implements Runnable {
    private int carNum;
    private CompetionLatch latch = null;
    public Car(int carNum, CompetionLatch latch) {
        this.carNum = carNum;
        this.latch = latch;
    }
    @Override
    public void run() {
        this.latch.countDown();
        this.latch.await();
        startCar();
    }
    private void startCar() {
        System.out.println("Car num " + this.carNum + " start to run.");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Car num " + this.carNum + " get to the finish line.");
    }
}

    public static void main(String[] args) {
        // 构造同步量
        final CompetionLatch latch = new CompetionLatch(CarCompetion.CAR_NUM);
        final ExecutorService carPool = 
            Executors.newFixedThreadPool(CarCompetion.CAR_NUM);
        for (int i = 0; i < CarCompetion.CAR_NUM; i++) {
            carPool.execute(new Car(i, latch));
        }
    }
}

```