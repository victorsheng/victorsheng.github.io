---
title: bugfix-easymapper-class-not-found
abbrlink: 1638819899
date: 2018-10-10 19:23:07
tags:
categories:
---
```
2018-10-10-19:22:32.032 ERROR [qtp1744347043-8]-com.baidu.unbiz.easymapper.codegen.bytecode.JavassistByteCodeManipulator:120>>An exception occurred while compiling the following method:

 	public void map(java.lang.Object a, java.lang.Object b) {

com.demo.dataset.model.request.AlgoCertificateRequestParam source = ((com.demo.dataset.model.request.AlgoCertificateRequestParam)a);
com.demo.dataset.model.param.AlgoCertificateParam destination = ((com.demo.dataset.model.param.AlgoCertificateParam)b);
if ( !(((java.lang.String)source.getAlgId()) == null)){
destination.setAlgId(((java.lang.String)source.getAlgId()));
}
if ( !(((java.lang.String)source.getAlgSha256()) == null)){
destination.setAlgSha256(((java.lang.String)source.getAlgSha256()));
}
if ( !(((java.lang.Integer)source.getExeType()) == null)){
destination.setExeType(((java.lang.Integer)source.getExeType()));
}
if ( !(((java.lang.String)source.getLang()) == null)){
destination.setLang(((java.lang.String)source.getLang()));
}
if ( !(((java.lang.String)source.getLangVersion()) == null)){
destination.setLangVersion(((java.lang.String)source.getLangVersion()));
}
if ( !(((java.lang.String)source.getTitle()) == null)){
destination.setTitle(((java.lang.String)source.getTitle()));
}
		if(customMapping != null) {
			 customMapping.map(source, destination);
		}
	}

 for com.baidu.unbiz.generated.EasyMapper_AlgoCertificateRequestParam_TO_AlgoCertificateParam_Mapper3227511583980531$3

javassist.CannotCompileException: [source error] no such class: com.demo.dataset.model.request.AlgoCertificateRequestParam
	at javassist.CtNewMethod.make(CtNewMethod.java:79)
	at javassist.CtNewMethod.make(CtNewMethod.java:45)
>>>>	at com.baidu.unbiz.easymapper.codegen.bytecode.JavassistByteCodeManipulator.compileClass(JavassistByteCodeManipulator.java:118)
	at com.baidu.unbiz.easymapper.codegen.MappingCodeGenerator.build(MappingCodeGenerator.java:69)
	at com.baidu.unbiz.easymapper.CopyByRefMapper$2.compute(CopyByRefMapper.java:165)
	at com.baidu.unbiz.easymapper.CopyByRefMapper$2.compute(CopyByRefMapper.java:162)
	at com.baidu.unbiz.easymapper.util.Memoizer$1.call(Memoizer.java:74)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at com.baidu.unbiz.easymapper.util.Memoizer.compute(Memoizer.java:80)
	at com.baidu.unbiz.easymapper.CopyByRefMapper.map(CopyByRefMapper.java:160)
	at com.baidu.unbiz.easymapper.CopyByRefMapper.map(CopyByRefMapper.java:116)
	at com.demo.dataset.copy.CopyServiceImpl.copy(CopyServiceImpl.java:136)
	at com.demo.dataset.controller.DataSetController.receiveAlgorithmCertificate(DataSetController.java:87)
	at com.demo.dataset.controller.DataSetController$$FastClassBySpringCGLIB$$7f78dfff.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:747)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
	at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:101)
	at com.demo.util.security.signature.advice.SignAdvice.aroundAdvice(SignAdvice.java:40)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethodWithGivenArgs(AbstractAspectJAdvice.java:644)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethod(AbstractAspectJAdvice.java:633)
	at org.springframework.aop.aspectj.AspectJAroundAdvice.invoke(AspectJAroundAdvice.java:70)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:174)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:185)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:689)
	at com.demo.dataset.controller.DataSetController$$EnhancerBySpringCGLIB$$38a305b3.receiveAlgorithmCertificate(<generated>)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:209)
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:136)
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:102)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:877)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:783)
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:991)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:925)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:974)
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:877)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:707)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:851)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:790)
	at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:864)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1655)
	at org.eclipse.jetty.websocket.server.WebSocketUpgradeFilter.doFilter(WebSocketUpgradeFilter.java:215)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:99)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.web.filter.HttpPutFormContentFilter.doFilterInternal(HttpPutFormContentFilter.java:109)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:81)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.boot.web.servlet.support.ErrorPageFilter.doFilter(ErrorPageFilter.java:117)
	at org.springframework.boot.web.servlet.support.ErrorPageFilter.access$000(ErrorPageFilter.java:61)
	at org.springframework.boot.web.servlet.support.ErrorPageFilter$1.doFilterInternal(ErrorPageFilter.java:92)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.springframework.boot.web.servlet.support.ErrorPageFilter.doFilter(ErrorPageFilter.java:110)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:200)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:533)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:146)
	at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:548)
	at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:132)
	at org.eclipse.jetty.server.handler.ScopedHandler.nextHandle(ScopedHandler.java:257)
	at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:1595)
	at org.eclipse.jetty.server.handler.ScopedHandler.nextHandle(ScopedHandler.java:255)
	at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1253)
	at org.eclipse.jetty.server.handler.ScopedHandler.nextScope(ScopedHandler.java:203)
	at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:473)
	at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:1564)
	at org.eclipse.jetty.server.handler.ScopedHandler.nextScope(ScopedHandler.java:201)
	at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1155)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:144)
	at org.eclipse.jetty.server.handler.ContextHandlerCollection.handle(ContextHandlerCollection.java:219)
	at org.eclipse.jetty.server.handler.HandlerCollection.handle(HandlerCollection.java:126)
	at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:132)
	at org.eclipse.jetty.server.Server.handle(Server.java:531)
	at org.eclipse.jetty.server.HttpChannel.handle(HttpChannel.java:352)
	at org.eclipse.jetty.server.HttpConnection.onFillable(HttpConnection.java:260)
	at org.eclipse.jetty.io.AbstractConnection$ReadCallback.succeeded(AbstractConnection.java:281)
	at org.eclipse.jetty.io.FillInterest.fillable(FillInterest.java:102)
	at org.eclipse.jetty.io.ChannelEndPoint$2.run(ChannelEndPoint.java:118)
	at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.doProduce(EatWhatYouKill.java:319)
	at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.tryProduce(EatWhatYouKill.java:175)
	at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.run(EatWhatYouKill.java:133)
	at org.eclipse.jetty.util.thread.ReservedThreadExecutor$ReservedThread.run(ReservedThreadExecutor.java:366)
	at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:754)
	at org.eclipse.jetty.util.thread.QueuedThreadPool$2.run(QueuedThreadPool.java:672)
	at java.lang.Thread.run(Thread.java:745)
Caused by: javassist.compiler.CompileError: no such class: com.demo.dataset.model.request.AlgoCertificateRequestParam
	at javassist.compiler.MemberResolver.lookupClass(MemberResolver.java:400)
	at javassist.compiler.MemberResolver.lookupClassByJvmName(MemberResolver.java:319)
	at javassist.compiler.MemberResolver.resolveJvmClassName(MemberResolver.java:512)
	at javassist.compiler.MemberCodeGen.resolveClassName(MemberCodeGen.java:1145)
	at javassist.compiler.CodeGen.atDeclarator(CodeGen.java:712)
	at javassist.compiler.ast.Declarator.accept(Declarator.java:100)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:351)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:351)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
	at javassist.compiler.CodeGen.atMethodBody(CodeGen.java:292)
	at javassist.compiler.CodeGen.atMethodDecl(CodeGen.java:274)
	at javassist.compiler.ast.MethodDecl.accept(MethodDecl.java:44)
	at javassist.compiler.Javac.compileMethod(Javac.java:169)
	at javassist.compiler.Javac.compile(Javac.java:95)
	at javassist.CtNewMethod.make(CtNewMethod.java:74)
	... 99 common frames omitted
2018-10-10-19:22:32.033 ERROR [qtp1744347043-8]-com.baidu.unbiz.easymapper.codegen.MappingCodeGenerator:79>>Generating mapping code with error: Failed to compile class com.baidu.unbiz.generated.EasyMapper_AlgoCertificateRequestParam_TO_AlgoCertificateParam_Mapper3227511583980531$3, reason is [source error] no such class: com.demo.dataset.model.request.AlgoCertificateRequestParam
com.baidu.unbiz.easymapper.exception.MappingCodeGenerationException: Failed to compile class com.baidu.unbiz.generated.EasyMapper_AlgoCertificateRequestParam_TO_AlgoCertificateParam_Mapper3227511583980531$3, reason is [source error] no such class: com.demo.dataset.model.request.AlgoCertificateRequestParam
	at com.baidu.unbiz.easymapper.codegen.bytecode.JavassistByteCodeManipulator.compileClass(JavassistByteCodeManipulator.java:132)
	at com.baidu.unbiz.easymapper.codegen.MappingCodeGenerator.build(MappingCodeGenerator.java:69)
	at com.baidu.unbiz.easymapper.CopyByRefMapper$2.compute(CopyByRefMapper.java:165)
	at com.baidu.unbiz.easymapper.CopyByRefMapper$2.compute(CopyByRefMapper.java:162)
	at com.baidu.unbiz.easymapper.util.Memoizer$1.call(Memoizer.java:74)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at com.baidu.unbiz.easymapper.util.Memoizer.compute(Memoizer.java:80)
	at com.baidu.unbiz.easymapper.CopyByRefMapper.map(CopyByRefMapper.java:160)
	at com.baidu.unbiz.easymapper.CopyByRefMapper.map(CopyByRefMapper.java:116)
	at com.demo.dataset.copy.CopyServiceImpl.copy(CopyServiceImpl.java:136)
	at com.demo.dataset.controller.DataSetController.receiveAlgorithmCertificate(DataSetController.java:87)
	at com.demo.dataset.controller.DataSetController$$FastClassBySpringCGLIB$$7f78dfff.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:747)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
	at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:101)
	at com.demo.util.security.signature.advice.SignAdvice.aroundAdvice(SignAdvice.java:40)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethodWithGivenArgs(AbstractAspectJAdvice.java:644)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethod(AbstractAspectJAdvice.java:633)
	at org.springframework.aop.aspectj.AspectJAroundAdvice.invoke(AspectJAroundAdvice.java:70)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:174)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:185)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:689)
	at com.demo.dataset.controller.DataSetController$$EnhancerBySpringCGLIB$$38a305b3.receiveAlgorithmCertificate(<generated>)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:209)
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:136)
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:102)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:877)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:783)
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:991)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:925)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:974)
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:877)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:707)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:851)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:790)
	at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:864)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1655)
	at org.eclipse.jetty.websocket.server.WebSocketUpgradeFilter.doFilter(WebSocketUpgradeFilter.java:215)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:99)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.web.filter.HttpPutFormContentFilter.doFilterInternal(HttpPutFormContentFilter.java:109)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:81)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.boot.web.servlet.support.ErrorPageFilter.doFilter(ErrorPageFilter.java:117)
	at org.springframework.boot.web.servlet.support.ErrorPageFilter.access$000(ErrorPageFilter.java:61)
	at org.springframework.boot.web.servlet.support.ErrorPageFilter$1.doFilterInternal(ErrorPageFilter.java:92)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.springframework.boot.web.servlet.support.ErrorPageFilter.doFilter(ErrorPageFilter.java:110)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:200)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:533)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:146)
	at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:548)
	at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:132)
	at org.eclipse.jetty.server.handler.ScopedHandler.nextHandle(ScopedHandler.java:257)
	at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:1595)
	at org.eclipse.jetty.server.handler.ScopedHandler.nextHandle(ScopedHandler.java:255)
	at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1253)
	at org.eclipse.jetty.server.handler.ScopedHandler.nextScope(ScopedHandler.java:203)
	at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:473)
	at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:1564)
	at org.eclipse.jetty.server.handler.ScopedHandler.nextScope(ScopedHandler.java:201)
	at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1155)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:144)
	at org.eclipse.jetty.server.handler.ContextHandlerCollection.handle(ContextHandlerCollection.java:219)
	at org.eclipse.jetty.server.handler.HandlerCollection.handle(HandlerCollection.java:126)
	at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:132)
	at org.eclipse.jetty.server.Server.handle(Server.java:531)
	at org.eclipse.jetty.server.HttpChannel.handle(HttpChannel.java:352)
	at org.eclipse.jetty.server.HttpConnection.onFillable(HttpConnection.java:260)
	at org.eclipse.jetty.io.AbstractConnection$ReadCallback.succeeded(AbstractConnection.java:281)
	at org.eclipse.jetty.io.FillInterest.fillable(FillInterest.java:102)
	at org.eclipse.jetty.io.ChannelEndPoint$2.run(ChannelEndPoint.java:118)
	at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.doProduce(EatWhatYouKill.java:319)
	at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.tryProduce(EatWhatYouKill.java:175)
	at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.run(EatWhatYouKill.java:133)
	at org.eclipse.jetty.util.thread.ReservedThreadExecutor$ReservedThread.run(ReservedThreadExecutor.java:366)
	at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:754)
	at org.eclipse.jetty.util.thread.QueuedThreadPool$2.run(QueuedThreadPool.java:672)
	at java.lang.Thread.run(Thread.java:745)
Caused by: javassist.CannotCompileException: [source error] no such class: com.demo.dataset.model.request.AlgoCertificateRequestParam
	at javassist.CtNewMethod.make(CtNewMethod.java:79)
	at javassist.CtNewMethod.make(CtNewMethod.java:45)
	at com.baidu.unbiz.easymapper.codegen.bytecode.JavassistByteCodeManipulator.compileClass(JavassistByteCodeManipulator.java:118)
	... 97 common frames omitted
Caused by: javassist.compiler.CompileError: no such class: com.demo.dataset.model.request.AlgoCertificateRequestParam
	at javassist.compiler.MemberResolver.lookupClass(MemberResolver.java:400)
	at javassist.compiler.MemberResolver.lookupClassByJvmName(MemberResolver.java:319)
	at javassist.compiler.MemberResolver.resolveJvmClassName(MemberResolver.java:512)
	at javassist.compiler.MemberCodeGen.resolveClassName(MemberCodeGen.java:1145)
	at javassist.compiler.CodeGen.atDeclarator(CodeGen.java:712)
	at javassist.compiler.ast.Declarator.accept(Declarator.java:100)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:351)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:351)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
	at javassist.compiler.CodeGen.atMethodBody(CodeGen.java:292)
	at javassist.compiler.CodeGen.atMethodDecl(CodeGen.java:274)
	at javassist.compiler.ast.MethodDecl.accept(MethodDecl.java:44)
	at javassist.compiler.Javac.compileMethod(Javac.java:169)
	at javassist.compiler.Javac.compile(Javac.java:95)
	at javassist.CtNewMethod.make(CtNewMethod.java:74)
	... 99 common frames omitted
2018-10-10-19:22:32.035 ERROR [qtp1744347043-8]-com.demo.dataset.handler.CommonExceptionHandler:137>>通用异常
com.baidu.unbiz.easymapper.exception.MappingException: Generating mapping code failed for ClassMap([A]:AlgoCertificateRequestParam, [B]:AlgoCertificateParam), this should not happen, probably the framework could not handle mapping correctly based on your bean
	at com.baidu.unbiz.easymapper.CopyByRefMapper.map(CopyByRefMapper.java:170)
	at com.baidu.unbiz.easymapper.CopyByRefMapper.map(CopyByRefMapper.java:116)
	at com.demo.dataset.copy.CopyServiceImpl.copy(CopyServiceImpl.java:136)
	at com.demo.dataset.controller.DataSetController.receiveAlgorithmCertificate(DataSetController.java:87)
	at com.demo.dataset.controller.DataSetController$$FastClassBySpringCGLIB$$7f78dfff.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:747)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
	at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:101)
	at com.demo.util.security.signature.advice.SignAdvice.aroundAdvice(SignAdvice.java:40)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethodWithGivenArgs(AbstractAspectJAdvice.java:644)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethod(AbstractAspectJAdvice.java:633)
	at org.springframework.aop.aspectj.AspectJAroundAdvice.invoke(AspectJAroundAdvice.java:70)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:174)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:185)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:689)
	at com.demo.dataset.controller.DataSetController$$EnhancerBySpringCGLIB$$38a305b3.receiveAlgorithmCertificate(<generated>)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:209)
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:136)
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:102)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:877)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:783)
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:991)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:925)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:974)
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:877)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:707)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:851)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:790)
	at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:864)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1655)
	at org.eclipse.jetty.websocket.server.WebSocketUpgradeFilter.doFilter(WebSocketUpgradeFilter.java:215)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:99)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.web.filter.HttpPutFormContentFilter.doFilterInternal(HttpPutFormContentFilter.java:109)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:81)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.boot.web.servlet.support.ErrorPageFilter.doFilter(ErrorPageFilter.java:117)
	at org.springframework.boot.web.servlet.support.ErrorPageFilter.access$000(ErrorPageFilter.java:61)
	at org.springframework.boot.web.servlet.support.ErrorPageFilter$1.doFilterInternal(ErrorPageFilter.java:92)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.springframework.boot.web.servlet.support.ErrorPageFilter.doFilter(ErrorPageFilter.java:110)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:200)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1642)
	at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:533)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:146)
	at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:548)
	at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:132)
	at org.eclipse.jetty.server.handler.ScopedHandler.nextHandle(ScopedHandler.java:257)
	at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:1595)
	at org.eclipse.jetty.server.handler.ScopedHandler.nextHandle(ScopedHandler.java:255)
	at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1253)
	at org.eclipse.jetty.server.handler.ScopedHandler.nextScope(ScopedHandler.java:203)
	at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:473)
	at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:1564)
	at org.eclipse.jetty.server.handler.ScopedHandler.nextScope(ScopedHandler.java:201)
	at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1155)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:144)
	at org.eclipse.jetty.server.handler.ContextHandlerCollection.handle(ContextHandlerCollection.java:219)
	at org.eclipse.jetty.server.handler.HandlerCollection.handle(HandlerCollection.java:126)
	at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:132)
	at org.eclipse.jetty.server.Server.handle(Server.java:531)
	at org.eclipse.jetty.server.HttpChannel.handle(HttpChannel.java:352)
	at org.eclipse.jetty.server.HttpConnection.onFillable(HttpConnection.java:260)
	at org.eclipse.jetty.io.AbstractConnection$ReadCallback.succeeded(AbstractConnection.java:281)
	at org.eclipse.jetty.io.FillInterest.fillable(FillInterest.java:102)
	at org.eclipse.jetty.io.ChannelEndPoint$2.run(ChannelEndPoint.java:118)
	at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.doProduce(EatWhatYouKill.java:319)
	at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.tryProduce(EatWhatYouKill.java:175)
	at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.run(EatWhatYouKill.java:133)
	at org.eclipse.jetty.util.thread.ReservedThreadExecutor$ReservedThread.run(ReservedThreadExecutor.java:366)
	at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:754)
	at org.eclipse.jetty.util.thread.QueuedThreadPool$2.run(QueuedThreadPool.java:672)
	at java.lang.Thread.run(Thread.java:745)
2018-10-10-19:22:32.037 INFO [qtp1744347043-8]-com.demo.util.log.GenericLogger:26>>{"path":"/xx/v1/58f64de19b534df3b9564c2a3a69ed49/algorithms","method":"POST","uid":"td","trackId":"b1918814908142f1a4e9e908c8199952","ts":1539170552001,"status":500,"dur":35}

```

问题:
重启后该就不报该错误了
原因不清楚

解决方案:
使用cgblib的beancopier替换掉了easyMapper