title: bugfix-jackson-version-conflict
abbrlink: 1624053647
date: 2018-10-09 16:57:25
tags:
categories:
---
# 异常  
```
2018-10-09 16:56:47.972 ERROR 7499 --- [nboundChannel-7] .WebSocketAnnotationMethodMessageHandler : Error while processing message GenericMessage [payload=byte[13], headers={simpMessageType=MESSAGE, stompCommand=SEND, nativeHeaders={destination=[/app/hello], content-length=[13]}, simpSessionAttributes={}, simpHeartbeat=[J@55dbac0a, lookupDestination=/hello, simpSessionId=xpe4qksn, simpDestination=/app/hello}]

java.lang.NoSuchMethodError: com.fasterxml.jackson.core.JsonGenerator.writeStartObject(Ljava/lang/Object;)V
	at com.fasterxml.jackson.databind.ser.BeanSerializer.serialize(BeanSerializer.java:151) ~[jackson-databind-2.9.5.jar:2.9.5]
	at com.fasterxml.jackson.databind.ser.DefaultSerializerProvider._serialize(DefaultSerializerProvider.java:480) ~[jackson-databind-2.9.5.jar:2.9.5]
	at com.fasterxml.jackson.databind.ser.DefaultSerializerProvider.serializeValue(DefaultSerializerProvider.java:319) ~[jackson-databind-2.9.5.jar:2.9.5]
	at com.fasterxml.jackson.databind.ObjectMapper.writeValue(ObjectMapper.java:2643) ~[jackson-databind-2.9.5.jar:2.9.5]
	at org.springframework.messaging.converter.MappingJackson2MessageConverter.convertToInternal(MappingJackson2MessageConverter.java:268) ~[spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.converter.AbstractMessageConverter.toMessage(AbstractMessageConverter.java:201) ~[spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.converter.CompositeMessageConverter.toMessage(CompositeMessageConverter.java:96) ~[spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.core.AbstractMessageSendingTemplate.doConvert(AbstractMessageSendingTemplate.java:180) ~[spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.core.AbstractMessageSendingTemplate.convertAndSend(AbstractMessageSendingTemplate.java:149) ~[spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.core.AbstractMessageSendingTemplate.convertAndSend(AbstractMessageSendingTemplate.java:128) ~[spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.simp.annotation.support.SendToMethodReturnValueHandler.handleReturnValue(SendToMethodReturnValueHandler.java:188) ~[spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.handler.invocation.HandlerMethodReturnValueHandlerComposite.handleReturnValue(HandlerMethodReturnValueHandlerComposite.java:107) ~[spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.handler.invocation.AbstractMethodMessageHandler.handleMatch(AbstractMethodMessageHandler.java:527) [spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.simp.annotation.support.SimpAnnotationMethodMessageHandler.handleMatch(SimpAnnotationMethodMessageHandler.java:495) [spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.simp.annotation.support.SimpAnnotationMethodMessageHandler.handleMatch(SimpAnnotationMethodMessageHandler.java:88) [spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.handler.invocation.AbstractMethodMessageHandler.handleMessageInternal(AbstractMethodMessageHandler.java:473) [spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.handler.invocation.AbstractMethodMessageHandler.handleMessage(AbstractMethodMessageHandler.java:409) [spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at org.springframework.messaging.support.ExecutorSubscribableChannel$SendTask.run(ExecutorSubscribableChannel.java:138) [spring-messaging-5.0.5.RELEASE.jar:5.0.5.RELEASE]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_151]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_151]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_151]


```

# 搜索
https://stackoverflow.com/questions/40051253/java-lang-nosuchmethoderror-using-jackson-core-streaming-api

可能是版本依赖关系

# 搜索jackson依赖

![upload successful](/images/pasted-279.png)


![upload successful](/images/pasted-280.png)

# 解决方案1:
```
    <dependency>
      <groupId>com.demo</groupId>
      <artifactId>util</artifactId>
      <version>1.0</version>
      <exclusions>
        <exclusion>
          <artifactId>jackson-annotations</artifactId>
          <groupId>com.fasterxml.jackson.core</groupId>
        </exclusion>
      </exclusions>
    </dependency>
```

无效

# 解决方案2:
```
    <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core -->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.9.7</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.9.7</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-annotations -->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
      <version>2.9.7</version>
    </dependency>
```


# 针对NoSuchMethodError的通用处理方式

## different version of the class that is missing a method [stackoverflow]
https://stackoverflow.com/questions/35186/how-do-i-fix-a-nosuchmethoderror


Without any more information it is difficult to pinpoint the problem, but the root cause is that you most likely have compiled a class against a different version of the class that is missing a method, than the one you are using when running it.

Look at the stack trace ... If the exception appears when calling a method on an object in a library, you are most likely using separate versions of the library when compiling and running. Make sure you have the right version both places.

If the exception appears when calling a method on objects instantiated by classes you made, then your build process seems to be faulty. Make sure the class files that you are actually running are updated when you compile.

## 三种情景

http://timen-zbt.iteye.com/blog/1871152

1. 在引包是引用了不匹配的包版本
2. 开发环境和运行环境的不一致
3. 以上两点都齐全，并且确实有对应的方法存在，依然报java.lang.NoSuchMethodError错误