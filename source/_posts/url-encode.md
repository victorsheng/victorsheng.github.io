---
title: url-encode
date: 2018-12-26 11:49:01
tags:
categories:
---
```
com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "agreed" (class com.talkingdata.opal.atm.agent.token.model.ProviderTokenResponse), not marked as ignorable (5 known properties: "token", "contacts", "email", "company", "tel"])
 at [Source: (byte[])"{xxxxxxx}"; line: 1, column: 16] (through reference chain: com.talkingdata.opal.atm.agent.token.model.ProviderTokenResponse["agreed"])
	at com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException.from(UnrecognizedPropertyException.java:60) ~[jackson-databind-2.9.7.jar:2.9.7]
	at com.fasterxml.jackson.databind.DeserializationContext.handleUnknownProperty(DeserializationContext.java:823) ~[jackson-databind-2.9.7.jar:2.9.7]
	at com.fasterxml.jackson.databind.deser.std.StdDeserializer.handleUnknownProperty(StdDeserializer.java:1153) ~[jackson-databind-2.9.7.jar:2.9.7]
	at com.fasterxml.jackson.databind.deser.BeanDeserializerBase.handleUnknownProperty(BeanDeserializerBase.java:1589) ~[jackson-databind-2.9.7.jar:2.9.7]
	at com.fasterxml.jackson.databind.deser.BeanDeserializerBase.handleUnknownVanilla(BeanDeserializerBase.java:1567) ~[jackson-databind-2.9.7.jar:2.9.7]
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.vanillaDeserialize(BeanDeserializer.java:294) ~[jackson-databind-2.9.7.jar:2.9.7]
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:151) ~[jackson-databind-2.9.7.jar:2.9.7]
	at com.fasterxml.jackson.databind.ObjectMapper._readMapAndClose(ObjectMapper.java:4013) ~[jackson-databind-2.9.7.jar:2.9.7]
	at com.fasterxml.jackson.databind.ObjectMapper.readValue(ObjectMapper.java:3091) ~[jackson-databind-2.9.7.jar:2.9.7]
```

**将前台传递的JSON 串转成Java实体对象时，出现了Unrecognized field, not marked as ignorable 错误。该错误的意思是说，不能够识别的字段没有标示为可忽略。出现该问题的原因就是JSON中包含了目标Java对象没有的属性。**

解决方法有如下几种：
- 格式化输入内容，保证传入的JSON串不包含目标对象的没有的属性。
- @JsonIgnoreProperties(ignoreUnknown = true) 在目标对象的类级别上加上该注解，并配置ignoreUnknown = true，则Jackson在反序列化的时候，会忽略该目标对象不存在的属性。
- 全局DeserializationFeature配置 
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES,false);配置该objectMapper在反序列化时，忽略目标对象没有的属性。凡是使用该objectMapper反序列化时，都会拥有该特性。


# 参考
https://blog.csdn.net/gavincook/article/details/46574661