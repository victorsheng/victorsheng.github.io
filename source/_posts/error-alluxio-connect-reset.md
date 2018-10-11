---
title: bugfix-alluxio-connect-reset
abbrlink: 1715881126
date: 2018-10-11 19:41:25
tags:
categories:
---
```
2018-10-11-19:31:54.011 ERROR [query-asyn-type-task]-alluxio.AbstractThriftClient:154>>java.net.SocketException: Connection reset
org.apache.thrift.transport.TTransportException: java.net.SocketException: Connection reset
	at org.apache.thrift.transport.TIOStreamTransport.read(TIOStreamTransport.java:129)
	at org.apache.thrift.transport.TTransport.readAll(TTransport.java:86)
	at org.apache.thrift.transport.TSaslTransport.readLength(TSaslTransport.java:376)
	at org.apache.thrift.transport.TSaslTransport.readFrame(TSaslTransport.java:453)
	at org.apache.thrift.transport.TSaslTransport.read(TSaslTransport.java:435)
	at org.apache.thrift.transport.TSaslClientTransport.read(TSaslClientTransport.java:37)
	at org.apache.thrift.transport.TTransport.readAll(TTransport.java:86)
	at org.apache.thrift.protocol.TBinaryProtocol.readAll(TBinaryProtocol.java:429)
	at org.apache.thrift.protocol.TBinaryProtocol.readI32(TBinaryProtocol.java:318)
	at org.apache.thrift.protocol.TBinaryProtocol.readMessageBegin(TBinaryProtocol.java:219)
	at org.apache.thrift.protocol.TProtocolDecorator.readMessageBegin(TProtocolDecorator.java:135)
	at org.apache.thrift.TServiceClient.receiveBase(TServiceClient.java:77)
	at alluxio.thrift.BlockWorkerClientService$Client.recv_cacheBlock(BlockWorkerClientService.java:234)
	at alluxio.thrift.BlockWorkerClientService$Client.cacheBlock(BlockWorkerClientService.java:220)
	at alluxio.client.block.RetryHandlingBlockWorkerClient$4.call(RetryHandlingBlockWorkerClient.java:181)
	at alluxio.client.block.RetryHandlingBlockWorkerClient$4.call(RetryHandlingBlockWorkerClient.java:177)
	at alluxio.AbstractThriftClient.retryRPC(AbstractThriftClient.java:146)
	at alluxio.client.block.RetryHandlingBlockWorkerClient.cacheBlock(RetryHandlingBlockWorkerClient.java:177)
	at alluxio.client.block.RemoteBlockOutStream.close(RemoteBlockOutStream.java:97)
	at alluxio.client.file.FileOutStream.close(FileOutStream.java:216)
	at com.demo.book.file.impl.FileManagementImpl.upload(FileManagementImpl.java:78)
	at com.demo.book.service.impl.bookServiceImpl.saveToFileSystemAndUpdateUri(bookServiceImpl.java:364)
	at com.demo.book.service.impl.bookServiceImpl.pullAndSave(bookServiceImpl.java:350)
	at com.demo.book.service.impl.bookServiceImpl$PullTask.run(bookServiceImpl.java:208)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run$$$capture(FutureTask.java:266)
	at java.util.concurrent.FutureTask.run(FutureTask.java)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.net.SocketException: Connection reset
	at java.net.SocketInputStream.read(SocketInputStream.java:210)
	at java.net.SocketInputStream.read(SocketInputStream.java:141)
	at java.io.BufferedInputStream.fill(BufferedInputStream.java:246)
	at java.io.BufferedInputStream.read1(BufferedInputStream.java:286)
	at java.io.BufferedInputStream.read(BufferedInputStream.java:345)
	at org.apache.thrift.transport.TIOStreamTransport.read(TIOStreamTransport.java:127)
	... 31 common frames omitted
```


```
    try (FileOutStream fos = fs.createFile(path, fileOptions);) {
      int copy = IOUtils.copy(is, fos);
      if (copy == 0) {
        throw new BadRequestException("the upload file should have content");
      }
      //fos.flush();
      fos.close();
```
注释掉fos.flush(); 不报错误了