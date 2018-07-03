title: okhttpclient-close
date: 2018-06-25 10:03:40
tags:
categories:
---
# 原文
https://github.com/square/okhttp/issues/3372
https://github.com/square/okhttp/issues/3160


作者的主张:
```
我们希望鼓励用户在客户端之间共享连接池和调度程序。但如果你这样做，你不能把所有东西都关闭。

创建一个新的OkHttpClient是相对昂贵的，我不愿意轻易地创建和销毁客户端，因为它可能会鼓励低效的用例。 您可以自由地为每个测试创建和销毁客户端，但是我希望您编写代码以进行设置和拆卸，以便您可以承担该性能成本的责任。

性能成本基本上是为连接池创建ExecutorService的成本，另一个是为Dispatcher创建的成本。我在自己的服务器应用程序中发现，创建和销毁线程池的成本在运行测试的总体成本中相当可观。


对于大多数效率来说，您的流程中将有一个ConnectionPool和一个Dispatcher。 这些可以被许多OkHttpClient实例共享。
如果关闭分派器，那么使用它的OkHttpClient实例都不会起作用。 类似的关闭ConnectionPool。
一个比喻：想象一下，当你使用Firefox时，拔掉你的住宅调制解调器和WiFi路由器。 当你需要它们时很容易将它们插回去。 如果你不打算再次使用Firefox，那么知道该关闭什么是方便的。
但我仍然认为，Firefox在退出时会关闭这些设备，或者甚至提供一个选项来关闭这些设备。
```


# 自动关闭的方式


## 连接级别(java.net.Socket)的资源关闭
如果保持空闲状态，保持的线程和连接将自动释放

最多可容纳5个闲置连接，闲置5分钟后将被驱逐。
默认设置:
```
  public ConnectionPool() {
    this(5, 5, TimeUnit.MINUTES);
  }

  public ConnectionPool(int maxIdleConnections, long keepAliveDuration, TimeUnit timeUnit) {
    this.maxIdleConnections = maxIdleConnections;
    this.keepAliveDurationNs = timeUnit.toNanos(keepAliveDuration);

    // Put a floor on the keep alive duration, otherwise cleanup will spin loop.
    if (keepAliveDuration <= 0) {
      throw new IllegalArgumentException("keepAliveDuration <= 0: " + keepAliveDuration);
    }
  }
```

ConnectionPool内部维护了一个清理线程
```
  private final Runnable cleanupRunnable = new Runnable() {
    @Override public void run() {
      while (true) {
        long waitNanos = cleanup(System.nanoTime());
        if (waitNanos == -1) return;
        if (waitNanos > 0) {
          long waitMillis = waitNanos / 1000000L;
          waitNanos -= (waitMillis * 1000000L);
          synchronized (ConnectionPool.this) {
            try {
              ConnectionPool.this.wait(waitMillis, (int) waitNanos);
            } catch (InterruptedException ignored) {
            }
          }
        }
      }
    }
  };
```

实际清理:
- 如果空闲醉酒的socket它超出了保持活动时间限制或空闲连接数限制，则驱逐空闲时间最长的连接,并返回0,再次检查
- 如果有空闲状态的,返回下次检查的时间:keepAliveDurationNs - longestIdleDurationNs;
- 如果没有空闲状态的,返回下次检查的时间:keepAliveDurationNs
```
long cleanup(long now) {
    int inUseConnectionCount = 0;
    int idleConnectionCount = 0;
    RealConnection longestIdleConnection = null;
    long longestIdleDurationNs = Long.MIN_VALUE;

    // Find either a connection to evict, or the time that the next eviction is due.
    synchronized (this) {
      for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {
        RealConnection connection = i.next();

        // If the connection is in use, keep searching.
        if (pruneAndGetAllocationCount(connection, now) > 0) {
          inUseConnectionCount++;
          continue;
        }

        idleConnectionCount++;

        // If the connection is ready to be evicted, we're done.
        long idleDurationNs = now - connection.idleAtNanos;
        if (idleDurationNs > longestIdleDurationNs) {
          longestIdleDurationNs = idleDurationNs;
          longestIdleConnection = connection;
        }
      }

      if (longestIdleDurationNs >= this.keepAliveDurationNs
          || idleConnectionCount > this.maxIdleConnections) {
        // We've found a connection to evict. Remove it from the list, then close it below (outside
        // of the synchronized block).
        connections.remove(longestIdleConnection);
      } else if (idleConnectionCount > 0) {
        // A connection will be ready to evict soon.
        return keepAliveDurationNs - longestIdleDurationNs;
      } else if (inUseConnectionCount > 0) {
        // All connections are in use. It'll be at least the keep alive duration 'til we run again.
        return keepAliveDurationNs;
      } else {
        // No connections, idle or in use.
        cleanupRunning = false;
        return -1;
      }
    }

    closeQuietly(longestIdleConnection.socket());

    // Cleanup again immediately.
    return 0;
  }
```


## 输入/输出流级别的资源关闭
```
response.close();
response.body().close();
```
然而不需要,因为已经自动调用了

当我们第一次调用 response.body().string() 时，OkHttp 将响应体的缓冲资源返回的同时，调用 closeQuietly() 方法默默释放了资源。
```
public final String string() throws IOException {
  return new String(bytes(), charset().name());
}

public final byte[] bytes() throws IOException {
  //...
  BufferedSource source = source();
  byte[] bytes;
  try {
    bytes = source.readByteArray();
  } finally {
    Util.closeQuietly(source);
  }
  //...
  return bytes;
}

public static void closeQuietly(Closeable closeable) {
  if (closeable != null) {
    try {
      closeable.close();
    } catch (RuntimeException rethrown) {
      throw rethrown;
    } catch (Exception ignored) {
    }
  }
}

//持有的 Source 对象
public final Source source;

@Override
public void close() throws IOException {
  if (closed) return;
  closed = true;
  source.close();
  buffer.clear();
}


```
### 会触发的方法

```
Response.close()
Response.body().close()
Response.body().source().close()
Response.body().charStream().close()
Response.body().byteString().close()
Response.body().bytes()
Response.body().string()
```

### 小结
https://github.com/square/okhttp/issues/1240#issuecomment-329197762

在实际开发中，响应主体 RessponseBody 持有的资源可能会很大，所以 OkHttp 并不会将其直接保存到内存中，只是持有数据流连接。只有当我们需要时，才会从服务器获取数据并返回。同时，考虑到应用重复读取数据的可能性很小，所以将其设计为一次性流(one-shot)，读取后即 '关闭并释放资源'。

# 参考
https://juejin.im/post/5a524eef518825732c536025