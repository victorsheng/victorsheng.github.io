---
title: catch异常但是仍然回滚
date: 2018-03-21 16:28:49
tags:
categories:
---
# 上代码
## 方式1:

```

public void insert() {
    long insuranceId = IdGenerator.generateId("insurance");
    String sql = String.format("insert into a (id) values(%s)", insuranceId);
    int update = jdbcTemplate.update(sql);
  try {
      hrow new RuntimeException();
    } catch (RuntimeException e) {
        e.printStackTrace();
    }
}

```
## 方式2:

```
@Service
@Transactional(readOnly = false)
public class ExService2 {

    @Autowired
    JdbcTemplate jdbcTemplate;

    public void ex() {
        throw new RuntimeException();
    }
}


```

```
public void insert() {
    long insuranceId = IdGenerator.generateId("insurance");
    String sql = String.format("insert into a (id) values(%s)", insuranceId);
    int update = jdbcTemplate.update(sql);
try {
        exService2.ex();
//            throw new RuntimeException();
    } catch (RuntimeException e) {
        e.printStackTrace();
    }
}
```

# 测试结果
看似同样的逻辑,但是对于事务的回滚影响却截然不同

方式1:未回滚+一个异常日志
方式2:回滚:两个异常日志+`Transaction rolled back because it has been marked as rollback-only`


```
20861 [http-nio-9991-exec-1] ERROR o.a.c.c.C.[.[.[.[dispatcherServlet] - Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only] with root cause
org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only
	at org.springframework.transaction.support.AbstractPlatformTransactionManager.commit(AbstractPlatformTransactionManager.java:724)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.commitTransactionAfterReturning(TransactionAspectSupport.java:485)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:291)
	at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:96)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:655)
	at cn.egenie.admin.domain.controller.ExService$$EnhancerBySpringCGLIB$$c69282d3.insert(<generated>)
	at cn.egenie.admin.domain.controller.ExceptionController.info(ExceptionController.java:25)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:221)
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:136)
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:110)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:832)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:743)
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:85)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:961)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:895)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:967)
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:858)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:622)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:843)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:729)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:292)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:207)
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:240)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:207)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:121)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:240)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:207)
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:212)
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:106)
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502)
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:141)
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:88)
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:522)
	at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1095)
	at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:672)
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1502)
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1458)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.lang.Thread.run(Thread.java:748)
```

# 原理

 spring 具备多种事务传播机制，最常用的是REQUIRED，即如果不存在事务，则新建一个事务；如果存在事务，则加入现存的事务中。
` public void A() {
    querySomething(...)；
    try {
      B()
    } catch () {
    }
    saveSomethinf()；
}

public void B() {
    throw Exception()
}`
此时B会和A存在一个事务中。如果B抛出异常没有捕获，即使在A中捕获并处理，仍会发生异常：Transaction rolled back because it has been marked as rollback-only 因为spring会在A捕获异常之前提前捕获到异常，并将当前事务设置为rollback-only，而A觉得对异常进行了捕获，它仍然继续commit，当TransactionManager发现状态为设置为rollback-only时， 则会抛出UnexpectedRollbackException 相关代码在AbstractPlatformTransactonManager.java中：

`public final void commit(TransactionStatus status) throws TransactionException {
		if (status.isCompleted()) {
			throw new IllegalTransactionStateException(
					"Transaction is already completed - do not call commit or rollback more than once per transaction");
		}

		DefaultTransactionStatus defStatus = (DefaultTransactionStatus) status;
		if (defStatus.isLocalRollbackOnly()) {
			if (defStatus.isDebug()) {
				logger.debug("Transactional code has requested rollback");
			}
			processRollback(defStatus);
			return;
		}
		if (!shouldCommitOnGlobalRollbackOnly() && defStatus.isGlobalRollbackOnly()) {
			if (defStatus.isDebug()) {
				logger.debug("Global transaction is marked as rollback-only but transactional code requested commit");
			}
			processRollback(defStatus);
			// Throw UnexpectedRollbackException only at outermost transaction boundary
			// or if explicitly asked to.
			if (status.isNewTransaction() || isFailEarlyOnGlobalRollbackOnly()) {
				throw new UnexpectedRollbackException(
						"Transaction rolled back because it has been marked as rollback-only");
			}
			return;
		}

		processCommit(defStatus);
	}`
