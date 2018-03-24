---
title: bugfix-dubbo-channel-close
date: 2018-02-06 10:09:16
tags:
categories:
---


```
com.alibaba.dubbo.rpc.RpcException: Failed to invoke the method findByIds in the service com.ejlerp.baseinfo.api.SkuService. Tried 3 times of the providers [10.27.215.53:20880, 10.26.109.140:20881] (2/3) from the registry 10.25.242.182:2181 on the consumer 118.178.19.115 using the dubbo version 2.8.4. Last error is: Failed to invoke remote method: findByIds, provider: dubbo://10.27.215.53:20880/com.ejlerp.baseinfo.api.SkuService?application=ejlerp-pms-provider&check=false&dubbo=2.8.4&generic=false&interface=com.ejlerp.baseinfo.api.SkuService&methods=generateSkuNo,findSimpleSkuLikeSkuNo,batchFindByBarcode,findAll,saveAvoidDuplicate,translateSku,calculateWeight,queryAndTranslateSku,findByBarcode,updateColumn,queryByNo,findSimpleSkuLikeSellerOuterNoIsEnabled,getSkuSpec,findSkuLikeProductNo,updateNotEmptyById,findSimpleSkuLikeProductName,findSkuLikeProductName,isCombinedSku,findSimpleSkuByCategoryNo,findSimpleSKUBySKUNo,findSimpleSKULikeShortCutKey,findAllSimpleSku,findAllSimpleSkuIsEnabled,batchUpdateNotEmptyById,findSimpleSkuLikeSizeType,updateSku,batchInsert,save,findAllSkus,batchReplace,findSimpleSkuLikeSellerOuterNo,batchDelete,findSkubyColorType,findById,findCombinedSku,findPage,findSkuIdByCondition,deleteSkus,findSimpleSkuLikeProductNoIsEnabled,findSimpleSkuByProductIdAndColor,batchInsertAvoidDuplicate,getSkuInfoByCondition,updateOneColumnById,replace,addSku,findPageV2,findSkubySizeType,findSimpleSkuByIds,findSkuInfoForMatch,isHugeSku,findByProductIds,findSimpleSkuLikeProductShortcutKeyIsEnabled,queryByBarcode,findSimpleSkuLikeSpec,batchGenerateSkus,findSimpleSkuByProductIds,findByIds,findSkuLikeProductShortcutKey,findLikeSkuNo,findSimpleSkuLikeProductNameIsEnabled,findBySellerOuterNo,setCombinedSku,checkUniqueColumn,findByNo,hardDelete,findSimpleSkuLikeNoOrBarcode,getAllProduct,findSkuIdsByProductIds,batchUpdate,findOne,findByNos,findSimpleSkuLikeProductShortcutKey,findSkuNotInIds,update,batchHardDelete,delete,findByCondition,findDetailsBySkuId,findSkuByNoOrBarCode,findSimpleSkuByProductIdAndSize,setNeedSpec,findByProductId,findTranslateByIds,findSimpleSkuLikeSkuNoIsEnabled,findSimpleSKUByBarCode,updateSKUsCostPrice,queryLikeNo,deleteSku,findSimpleSkuLikeColorType,findSimpleSkuLikeProductNo,queryById&organization=ejlerp&owner=victor&pid=3801&revision=0.1&side=consumer&timeout=60000&timestamp=1517844500677&version=0.1, cause: message can not send, because channel is closed . url:dubbo://10.27.215.53:20880/com.ejlerp.baseinfo.api.DictService?application=ejlerp-pms-provider&check=false&codec=dubbo&dubbo=2.8.4&generic=false&heartbeat=60000&interface=com.ejlerp.baseinfo.api.DictService&methods=findCustomerDicts,findDictsByType,findCustomerDictsByType,deleteCustomerDict,updateDict,findCustomerDict,findCustomerDictByKey,findAllDicts,findMaxCodeByCustomerType,addCustomerDict,findCustomerDictMapByTypes,findDictByKey,findCustomerDictsMapByType,findDicts,deleteDict,updateCustomerDict,findDictsByTypes,findCustomerDictsByTypeAndKeys,findMaxCodeByType,findCustomerDictByType,findCustomerDictByKeys,findDict,findDictsMapByType,addDict,findDictMapByTypes,findCustomerDictByValue,findDictByValue,findCustomerDictLikeValue&organization=ejlerp&owner=victor&pid=3801&revision=0.1&side=consumer&timeout=8000&timestamp=1517844501227&version=0.1
	at com.alibaba.dubbo.rpc.cluster.support.FailoverClusterInvoker.doInvoke(FailoverClusterInvoker.java:108)
	at com.alibaba.dubbo.rpc.cluster.support.AbstractClusterInvoker.invoke(AbstractClusterInvoker.java:227)
	at com.alibaba.dubbo.rpc.cluster.support.wrapper.MockClusterInvoker.invoke(MockClusterInvoker.java:72)
	at com.alibaba.dubbo.rpc.proxy.InvokerInvocationHandler.invoke(InvokerInvocationHandler.java:52)
	at com.alibaba.dubbo.common.bytecode.proxy6.findByIds(proxy6.java)
	at com.ejlerp.pms.provider.service.agent.baseinfo.BaseInfoAgentService.findSkuByIds(BaseInfoAgentService.java:92)
	at com.ejlerp.pms.provider.service.StockOutReportServiceImpl.pageExecute(StockOutReportServiceImpl.java:114)
	at com.ejlerp.pms.provider.service.StockOutReportServiceImpl.refreshAction(StockOutReportServiceImpl.java:99)
	at com.alibaba.dubbo.common.bytecode.Wrapper51.invokeMethod(Wrapper51.java)
	at com.alibaba.dubbo.rpc.proxy.javassist.JavassistProxyFactory$1.doInvoke(JavassistProxyFactory.java:46)
	at com.alibaba.dubbo.rpc.proxy.AbstractProxyInvoker.invoke(AbstractProxyInvoker.java:72)
	at com.alibaba.dubbo.rpc.protocol.InvokerWrapper.invoke(InvokerWrapper.java:53)
	at com.alibaba.dubbo.rpc.filter.ExceptionFilter.invoke(ExceptionFilter.java:64)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.monitor.support.MonitorFilter.invoke(MonitorFilter.java:75)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.TimeoutFilter.invoke(TimeoutFilter.java:42)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.protocol.dubbo.filter.TraceFilter.invoke(TraceFilter.java:78)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.ContextFilter.invoke(ContextFilter.java:70)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.GenericFilter.invoke(GenericFilter.java:132)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.ClassLoaderFilter.invoke(ClassLoaderFilter.java:38)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.EchoFilter.invoke(EchoFilter.java:38)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol$1.reply(DubboProtocol.java:113)
	at com.alibaba.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.handleRequest(HeaderExchangeHandler.java:84)
	at com.alibaba.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.received(HeaderExchangeHandler.java:170)
	at com.alibaba.dubbo.remoting.transport.DecodeHandler.received(DecodeHandler.java:52)
	at com.alibaba.dubbo.remoting.transport.dispatcher.ChannelEventRunnable.run(ChannelEventRunnable.java:82)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
Caused by: com.alibaba.dubbo.remoting.RemotingException: message can not send, because channel is closed . url:dubbo://10.27.215.53:20880/com.ejlerp.baseinfo.api.DictService?application=ejlerp-pms-provider&check=false&codec=dubbo&dubbo=2.8.4&generic=false&heartbeat=60000&interface=com.ejlerp.baseinfo.api.DictService&methods=findCustomerDicts,findDictsByType,findCustomerDictsByType,deleteCustomerDict,updateDict,findCustomerDict,findCustomerDictByKey,findAllDicts,findMaxCodeByCustomerType,addCustomerDict,findCustomerDictMapByTypes,findDictByKey,findCustomerDictsMapByType,findDicts,deleteDict,updateCustomerDict,findDictsByTypes,findCustomerDictsByTypeAndKeys,findMaxCodeByType,findCustomerDictByType,findCustomerDictByKeys,findDict,findDictsMapByType,addDict,findDictMapByTypes,findCustomerDictByValue,findDictByValue,findCustomerDictLikeValue&organization=ejlerp&owner=victor&pid=3801&revision=0.1&side=consumer&timeout=8000&timestamp=1517844501227&version=0.1
	at com.alibaba.dubbo.remoting.transport.AbstractClient.send(AbstractClient.java:268)
	at com.alibaba.dubbo.remoting.transport.AbstractPeer.send(AbstractPeer.java:51)
	at com.alibaba.dubbo.remoting.exchange.support.header.HeaderExchangeChannel.request(HeaderExchangeChannel.java:112)
	at com.alibaba.dubbo.remoting.exchange.support.header.HeaderExchangeClient.request(HeaderExchangeClient.java:91)
	at com.alibaba.dubbo.rpc.protocol.dubbo.ReferenceCountExchangeClient.request(ReferenceCountExchangeClient.java:81)
	at com.alibaba.dubbo.rpc.protocol.dubbo.DubboInvoker.doInvoke(DubboInvoker.java:96)
	at com.alibaba.dubbo.rpc.protocol.AbstractInvoker.invoke(AbstractInvoker.java:144)
	at com.alibaba.dubbo.monitor.support.MonitorFilter.invoke(MonitorFilter.java:75)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.protocol.dubbo.filter.FutureFilter.invoke(FutureFilter.java:53)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.ConsumerContextFilter.invoke(ConsumerContextFilter.java:48)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.listener.ListenerInvokerWrapper.invoke(ListenerInvokerWrapper.java:74)
	at com.alibaba.dubbo.rpc.protocol.InvokerWrapper.invoke(InvokerWrapper.java:53)
	at com.alibaba.dubbo.rpc.cluster.support.FailoverClusterInvoker.doInvoke(FailoverClusterInvoker.java:77)
	... 35 common frames omitted
```


dubbo提供了自动重连的机制

```
 /**
     * init reconnect thread
     */
    private synchronized void initConnectStatusCheckCommand() {
        //reconnect=false to close reconnect
        int reconnect = getReconnectParam(getUrl());
        if (reconnect > 0 && (reconnectExecutorFuture == null || reconnectExecutorFuture.isCancelled())) {
            Runnable connectStatusCheckCommand = new Runnable() {
                public void run() {
                    try {
                        if (!isConnected()) {
                            connect();
                        } else {
                            lastConnectedTime = System.currentTimeMillis();
                        }
                    } catch (Throwable t) {
                        String errorMsg = "client reconnect to " + getUrl().getAddress() + " find error . url: " + getUrl();
                        // wait registry sync provider list
                        if (System.currentTimeMillis() - lastConnectedTime > shutdown_timeout) {
                            if (!reconnect_error_log_flag.get()) {
                                reconnect_error_log_flag.set(true);
                                logger.error(errorMsg, t);
                                return;
                            }
                        }
                        if (reconnect_count.getAndIncrement() % reconnect_warning_period == 0) {
                            logger.warn(errorMsg, t);
                        }
                    }
                }
            };
            reconnectExecutorFuture = reconnectExecutorService.scheduleWithFixedDelay(connectStatusCheckCommand, reconnect, reconnect, TimeUnit.MILLISECONDS);
        }
    }
```

日志

```
2018-02-06 08:10:57.798 [DubboClientReconnectTimer-thread-1] ERROR com.alibaba.dubbo.remoting.transport.AbstractClient.error -  [DUBBO] client reconnect to 10.27.215.53:20880 find e
rror . url: dubbo://10.27.215.53:20880/com.ejlerp.baseinfo.api.DictService?application=ejlerp-pms-provider&check=false&codec=dubbo&dubbo=2.8.4&generic=false&heartbeat=60000&interfac
e=com.ejlerp.baseinfo.api.DictService&methods=findCustomerDicts,findDictsByType,findCustomerDictsByType,deleteCustomerDict,updateDict,findCustomerDict,findCustomerDictByKey,findAllD
icts,findMaxCodeByCustomerType,addCustomerDict,findCustomerDictMapByTypes,findDictByKey,findCustomerDictsMapByType,findDicts,deleteDict,updateCustomerDict,findDictsByTypes,findCusto
merDictsByTypeAndKeys,findMaxCodeByType,findCustomerDictByType,findCustomerDictByKeys,findDict,findDictsMapByType,addDict,findDictMapByTypes,findCustomerDictByValue,findDictByValue,
findCustomerDictLikeValue&organization=ejlerp&owner=victor&pid=3801&revision=0.1&side=consumer&timeout=8000&timestamp=1517844501227&version=0.1, dubbo version: 2.8.4, current host:
118.178.19.115
com.alibaba.dubbo.remoting.RemotingException: client(url: dubbo://10.27.215.53:20880/com.ejlerp.baseinfo.api.DictService?application=ejlerp-pms-provider&check=false&codec=dubbo&dubb
o=2.8.4&generic=false&heartbeat=60000&interface=com.ejlerp.baseinfo.api.DictService&methods=findCustomerDicts,findDictsByType,findCustomerDictsByType,deleteCustomerDict,updateDict,f
indCustomerDict,findCustomerDictByKey,findAllDicts,findMaxCodeByCustomerType,addCustomerDict,findCustomerDictMapByTypes,findDictByKey,findCustomerDictsMapByType,findDicts,deleteDict
,updateCustomerDict,findDictsByTypes,findCustomerDictsByTypeAndKeys,findMaxCodeByType,findCustomerDictByType,findCustomerDictByKeys,findDict,findDictsMapByType,addDict,findDictMapBy
Types,findCustomerDictByValue,findDictByValue,findCustomerDictLikeValue&organization=ejlerp&owner=victor&pid=3801&revision=0.1&side=consumer&timeout=8000&timestamp=1517844501227&ver
sion=0.1) failed to connect to server /10.27.215.53:20880 client-side timeout 3000ms (elapsed: 3001ms) from netty client 118.178.19.115 using dubbo version 2.8.4
        at com.alibaba.dubbo.remoting.transport.netty.NettyClient.doConnect(NettyClient.java:147)
        at com.alibaba.dubbo.remoting.transport.AbstractClient.connect(AbstractClient.java:280)
        at com.alibaba.dubbo.remoting.transport.AbstractClient$1.run(AbstractClient.java:145)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:308)
        at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:180)
        at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:294)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)
```


```
2018-02-06 02:34:26.962 [DubboClientReconnectTimer-thread-1] WARN  com.alibaba.dubbo.remoting.transport.AbstractClient.warn -  [DUBBO] client reconnect to 10.25.242.182:20885 find e
rror . url: dubbo://10.25.242.182:20885/com.ejlerp.baseinfo.api.VendorService?application=ejlerp-pms-provider&check=false&codec=dubbo&dubbo=2.8.4&generic=false&heartbeat=60000&inter
face=com.ejlerp.baseinfo.api.VendorService&methods=findLikeVendorNo,findByVendorTenantId,updateOneColumnById,replace,findLikeVendorShortcutKey,findPageV2,findAll,saveAvoidDuplicate,
findAndSaveByVendorName,findVendorIdAndNameMap,findMyVendorOL,enableVendor,findVendorNoAndIdMap,addVendor,updateColumn,findByIds,relateVendorOL,deleteVendor,checkUniqueColumn,findBy
No,findByVendorName,updateNotEmptyById,hardDelete,batchUpdateNotEmptyById,batchUpdate,batchInsert,findOne,save,batchHardDelete,update,findByPuchaserId,batchReplace,delete,updateVend
or,batchDelete,vendorOLSync,findById,findTmpByVendorTenantIds,findByMobile,findPage,findLikeVendorName,disableVendor,batchInsertAvoidDuplicate,findByCategoryNo,findByVendorNames&org
anization=ejlerp&owner=victor&pid=3801&revision=0.1&side=consumer&timeout=8000&timestamp=1517844500217&version=0.1, dubbo version: 2.8.4, current host: 118.178.19.115
com.alibaba.dubbo.remoting.RemotingException: client(url: dubbo://10.25.242.182:20885/com.ejlerp.baseinfo.api.VendorService?application=ejlerp-pms-provider&check=false&codec=dubbo&d
ubbo=2.8.4&generic=false&heartbeat=60000&interface=com.ejlerp.baseinfo.api.VendorService&methods=findLikeVendorNo,findByVendorTenantId,updateOneColumnById,replace,findLikeVendorShor
tcutKey,findPageV2,findAll,saveAvoidDuplicate,findAndSaveByVendorName,findVendorIdAndNameMap,findMyVendorOL,enableVendor,findVendorNoAndIdMap,addVendor,updateColumn,findByIds,relate
VendorOL,deleteVendor,checkUniqueColumn,findByNo,findByVendorName,updateNotEmptyById,hardDelete,batchUpdateNotEmptyById,batchUpdate,batchInsert,findOne,save,batchHardDelete,update,f
indByPuchaserId,batchReplace,delete,updateVendor,batchDelete,vendorOLSync,findById,findTmpByVendorTenantIds,findByMobile,findPage,findLikeVendorName,disableVendor,batchInsertAvoidDu
plicate,findByCategoryNo,findByVendorNames&organization=ejlerp&owner=victor&pid=3801&revision=0.1&side=consumer&timeout=8000&timestamp=1517844500217&version=0.1) failed to connect t
o server /10.25.242.182:20885 client-side timeout 3000ms (elapsed: 3000ms) from netty client 118.178.19.115 using dubbo version 2.8.4
        at com.alibaba.dubbo.remoting.transport.netty.NettyClient.doConnect(NettyClient.java:147)
        at com.alibaba.dubbo.remoting.transport.AbstractClient.connect(AbstractClient.java:280)
        at com.alibaba.dubbo.remoting.transport.AbstractClient$1.run(AbstractClient.java:145)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:308)
        at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:180)
        at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:294)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)
```
<img src="http://pic.victor123.cn/18-2-6/69248542.jpg" />


```
java -Xmx150M -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintHeapAtGC -Xloggc:./pms-gc.log -jar ejlerp-pms-provider-0.2-SNAPSHOT.jar --spring.profiles.active=prod --regCenter.serverList=10.25.242.182:2181 --server.port=9087 --druid.url=jdbc:mysql://rm-bp1ov5155kw37fa31i.mysql.rds.aliyuncs.com:3306/egenie_kn --druid.username=egeniekn --druid.password=ejl@2302 --spring.datasource.url=jdbc:mysql://rm-bp1ov5155kw37fa31i.mysql.rds.aliyuncs.com:3306/job_hz1 --dubbo.registry.address=zookeeper://10.25.242.182:2181 --dubbo.protocol.host=10.25.242.182 --dubbo.monitor=false --provider.appKey=pmsP-hz1-s1 --goods.rest.host=http://www.runscm.com --pmsTradeIsOpen=0 > /dev/null 2>&1 &
```