title: optimize_dubbuo_reference
tags:
  - dubbo
  - reference
  - xml
categories: []
date: 2017-12-14 17:04:00
---
# dubbo服务引用
默认情况下可以通过@Reference和xml注解来引入
例如
``` java
 @Reference(version = "0.1", timeout = 8000)
    private TenantService tenantService;

    @Reference(version = "0.1", timeout = 8000)
    private ProductService productService;

    @Reference(version = "0.1", timeout = 8000)
    private DictService dictService;

    @Reference(version = "0.1", timeout = 8000)
    private CategoryService categoryService;
```
或者
``` xml
    <dubbo:reference id="remoteBaseCustomPropsService" interface="com.ejlerp.baseinfo.api.BaseCustomPropsService"
                     version="0.1" timeout="60000"/>
    <dubbo:reference id="remoteSkuStepPriceService" interface="com.ejlerp.baseinfo.api.SkuStepPriceService"
                     version="0.1" timeout="60000"/>
    <dubbo:reference id="remoteSkuUnitService" interface="com.ejlerp.baseinfo.api.SkuUnitService"
                     version="0.1" timeout="60000"/>
```
# 出现的问题
生产环境上调用微服务超时,结果发现即使通过
``` java
service:
    @Reference(version = "0.1", timeout = 8000)
    private ProductService productService;
```
的方式设置了超时时间,但生产环境的错误,还是设置的1000ms的超时时间,
# 问题原因
在其他的service中同样是引用了这个注解
``` java
其他service:
    @Reference(version = "0.1")
    private ProductService productService;
```
Reference注解 包含了初始化和注入dubbo的bean,两种功能
其实是声明了一个bean,但是到底是用哪个timeout,是有spring初始化bean的顺序所决定的,很可能出现设置了timeout但是仍然没有效果的情况

# 解决办法
- 在2017-12-12后的,所有微服务,禁止使用Reference注解,通过autowirdj进行钟乳
- 统一使用xml的方式,进行声明dubbo的bean

# 优点
- 统一Reference方式对其他微服务的引用,放置不一致的配置出现
- xml的方式,方便设置方法级别的参数

```  xml
    <dubbo:reference id="remoteArrivalStockOutReportService" interface="com.ejlerp.pms.api.StockOutReportService"
                     version="0.1" timeout="60000">
        <dubbo:method name="refreshAction" timeout="9000000" retry="0"/>
    </dubbo:reference>
```

# 修改步骤如下
- 去除掉DubboConf.java中大部分代码
- 添加spring-dubbo,xml
- 删除旧的@Reference,同时添加@Autowird
    - 替换默认的@Reference无超时时间的,基本为dao中id 和 sn generater
    - 将web中最全的dubbo.conf复制到为服务中
    - 本项目为pms,删除掉所有pms相关的xml注入
    - 逐行全文搜索,删掉没用的service
<img src="http://pic.victor123.cn/17-12-14/76912018.jpg">
<img src="http://pic.victor123.cn/17-12-14/61534900.jpg">
    - 再次遍历dubb.conf的所有service,修改成项目中原来的超时时间,并去掉个性化设置的@Reference
- 编译,解决编译问题,补充一些@Autowird的包
- 尝试启动,补充一些egenie-web中没有声明,但是微服务中用到的service
    - 例如:com.ejlerp.messages.api.QueueService,com.ejlerp.cache.api.SortedSetCacher,com.ejlerp.cache.api.IDGenerator,com.ejlerp.cache.api.IDGenerator等
- 启动成功 
<img src="http://pic.victor123.cn/17-12-14/72995353.jpg">



- 具体改动,见git:
- http://git.ejlerp.com/egenie/ejlerp-pms/commit/4d431233af740e0f1c85c8733afcaadfb0638a18

# BTW
- http://dubbo.io/books/dubbo-user-book/demos/fault-tolerent-strategy.html
- 在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。
<img src="http://dubbo.io/books/dubbo-user-book/sources/images/cluster.jpg" />

- Failover Cluster

失败自动切换，当出现失败，重试其它服务器 1。通常用于读操作，但重试会带来更长延迟。可通过 retries="2" 来设置重试次数(不含第一次)。
Failfast Cluster

快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

- Failsafe Cluster

失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

- Failback Cluster

失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。

- Forking Cluster

并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 forks="2" 来设置最大并行数。

- Broadcast Cluster

广播调用所有提供者，逐个调用，任意一台报错则报错 2。通常用于通知所有提供者更新缓存或日志等本地资源信息。

开发人员可以根据自己的实际业务,设置失败时的处理方式
