---
title: executorservice-submit-vs-execute
abbrlink: 2771641018
date: 2018-09-05 11:09:51
tags:
    - 线程池
    - jdk
categories:
---

# execute
```
"query-asyn-type-task"@10,978 in group "main": RUNNING
checkSuccessFul:85, OkHttpClientImpl {com.talkingdata.opal.util.http.client.Impl}
sendRequest:63, OkHttpClientImpl {com.talkingdata.opal.util.http.client.Impl}
sendRequest:112, SignHttpClientWrapper {com.talkingdata.opal.util.security.signature.httpclient}
findById:61, RemoteAlgorithmServiceImpl {com.talkingdata.opal.dataset.remote.impl}
pullAndSave:328, DataSetServiceImpl {com.talkingdata.opal.dataset.service.impl}
run:195, DataSetServiceImpl$PullTask {com.talkingdata.opal.dataset.service.impl}
runWorker:1149, ThreadPoolExecutor {java.util.concurrent}
run:624, ThreadPoolExecutor$Worker {java.util.concurrent}
run:748, Thread {java.lang}

```


```
/**
     * Executes the given task sometime in the future.  The task
     * may execute in a new thread or in an existing pooled thread.
     *
     * If the task cannot be submitted for execution, either because this
     * executor has been shutdown or because its capacity has been reached,
     * the task is handled by the current {@code RejectedExecutionHandler}.
     *
     * @param command the task to execute
     * @throws RejectedExecutionException at discretion of
     *         {@code RejectedExecutionHandler}, if the task
     *         cannot be accepted for execution
     * @throws NullPointerException if {@code command} is null
     */
    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```


```
"query-asyn-type-task"@10,875 in group "main": RUNNING
checkSuccessFul:85, OkHttpClientImpl {com.talkingdata.opal.util.http.client.Impl}
sendRequest:63, OkHttpClientImpl {com.talkingdata.opal.util.http.client.Impl}
sendRequest:112, SignHttpClientWrapper {com.talkingdata.opal.util.security.signature.httpclient}
findById:61, RemoteAlgorithmServiceImpl {com.talkingdata.opal.dataset.remote.impl}
pullAndSave:328, DataSetServiceImpl {com.talkingdata.opal.dataset.service.impl}
run:195, DataSetServiceImpl$PullTask {com.talkingdata.opal.dataset.service.impl}
call:511, Executors$RunnableAdapter {java.util.concurrent}
run$$$capture:266, FutureTask {java.util.concurrent}
run:-1, FutureTask {java.util.concurrent}
runWorker:1149, ThreadPoolExecutor {java.util.concurrent}
run:624, ThreadPoolExecutor$Worker {java.util.concurrent}
run:748, Thread {java.lang}


```
# submit
AbstractExecutorService
```
    /**
     * @throws RejectedExecutionException {@inheritDoc}
     * @throws NullPointerException       {@inheritDoc}
     */
    public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }

    /**
     * @throws RejectedExecutionException {@inheritDoc}
     * @throws NullPointerException       {@inheritDoc}
     */
    public <T> Future<T> submit(Runnable task, T result) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task, result);
        execute(ftask);
        return ftask;
    }
    /**
     * Returns a {@code RunnableFuture} for the given runnable and default
     * value.
     *
     * @param runnable the runnable task being wrapped
     * @param value the default value for the returned future
     * @param <T> the type of the given value
     * @return a {@code RunnableFuture} which, when run, will run the
     * underlying runnable and which, as a {@code Future}, will yield
     * the given value as its result and provide for cancellation of
     * the underlying task
     * @since 1.6
     */
    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }

    /**
     * Returns a {@code RunnableFuture} for the given callable task.
     *
     * @param callable the callable task being wrapped
     * @param <T> the type of the callable's result
     * @return a {@code RunnableFuture} which, when run, will call the
     * underlying callable and which, as a {@code Future}, will yield
     * the callable's result as its result and provide for
     * cancellation of the underlying task
     * @since 1.6
     */
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new FutureTask<T>(callable);
    }
```


# 总结，
从上面的源码以及讲解可以总结execute()和submit()方法的区别：

　　1. 接收的参数不一样;(在ExecutorService接口中，一共有以上三个sumbit()方法，入参可以为Callable<T>，也可以为Runnable，而且方法有返回值Future<T>；)

　　2. submit()有返回值，而execute()没有;

　　　　例如，有个validation的task，希望该task执行完后告诉我它的执行结果，是成功还是失败，然后继续下面的操作。

　　3. submit()可以进行Exception处理;

　　　　例如，如果task里会抛出checked或者unchecked exception，而你又希望外面的调用者能够感知这些exception并做出及时的处理，那么就需要用到submit，通过对Future.get()进行抛出异常的捕获，然后对其进行处理。


```
"query-asyn-type-task"@10,957 in group "main": RUNNING
lambda$null$0:85, DataSetServiceImpl {com.talkingdata.opal.dataset.service.impl}
uncaughtException:-1, 1958052599 {com.talkingdata.opal.dataset.service.impl.DataSetServiceImpl$$Lambda$378}
dispatchUncaughtException:1959, Thread {java.lang}

```
# 测试两个方法的区别
- submit会将异常存到Future中
- 而execute就会直接抛出异常
```
package com.example.demo.threadpool;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import org.junit.Test;

public class ExceptionHanleTest {

  private ExecutorService executorService = Executors.newFixedThreadPool(3);

  @Test
  public void test1() {
    for (int i = 0; i < 100; i++) {
      executorService.submit(new ExceptionTask());
    }
  }

  @Test
  public void test2() throws ExecutionException, InterruptedException {
    for (int i = 0; i < 100; i++) {
      Future<?> submit = executorService.submit(new ExceptionTask());
      submit.get();
    }
  }

  @Test
  public void test3() throws ExecutionException, InterruptedException {
    for (int i = 0; i < 100; i++) {
      executorService.execute(new ExceptionTask());
    }
  }


  class ExceptionTask implements Runnable {

    @Override
    public void run() {
      throw new RuntimeException();
    }
  }
}

```

# 异常处理
```
public final class ExtendedExecutor extends ThreadPoolExecutor {

    // ...

    protected void afterExecute(Runnable r, Throwable t) {
        super.afterExecute(r, t);
        if (t == null && r instanceof Future<?>) {
            try {
                Future<?> future = (Future<?>) r;
                if (future.isDone()) {
                    future.get();
                }
            } catch (CancellationException ce) {
                t = ce;
            } catch (ExecutionException ee) {
                t = ee.getCause();
            } catch (InterruptedException ie) {
                Thread.currentThread().interrupt();
            }
        }
        if (t != null) {
            System.out.println(t);
        }
    }
}
```


# CompletableFuture