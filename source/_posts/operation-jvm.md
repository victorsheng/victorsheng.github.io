title: jvm崩溃
tags:
  - jvm
  - OOM
categories: []
date: 2017-12-14 14:14:00
---
# 日常日志及错误
```
2017-12-13 18:26:00.001 [pool-27-thread-1] INFO  com.ejlerp.pms.provider.mqCrond.MqPmsTradeOrderCrond.sendMsgToInter - == 处理采购推送交易库定时任务 start ==
2017-12-13 18:26:05.711 [pool-27-thread-1] ERROR org.springframework.scheduling.support.TaskUtils$LoggingErrorHandler.handleError - Unexpected error occurred in scheduled task.
java.lang.OutOfMemoryError: Java heap space
```

同时jvm由于重启过,gc日志也刷新了

# 每个jvm不仅用自己个性化的参数调优,还应该有共同参数,例如:打印gc日志,dump内存等参数

# TODO 待总结
