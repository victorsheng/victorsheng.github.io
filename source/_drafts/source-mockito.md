---
title: source-mockito
abbrlink: 3404213338
tags:
categories:
---


# 源码分析

## Mock
Mockito类中有几个mock函数的重载，最终都会调到mock(Class<T> classToMock, MockSettings mockSettings)，接受一个class类型与一个mockSettings，class就是我们需要mock对象的类型，而mockSettings则记录着此次mock的一些信息。mock的行为实则转交个MockitoCore
在MockitoCore中一是做了一下初始化工作，二是继续转交给MockUtil处理

```
 MockSettingsImpl impl = MockSettingsImpl.class.cast(settings);
 MockCreationSettings<T> creationSettings = impl.confirm(typeToMock);
 T mock = mockUtil.createMock(creationSettings);
 mockingProgress.mockingStarted(mock, typeToMock);
```

在MockUtil中则作了两件非常关键的事情:
一是创建了MockHandler对象，MockHandler是一个接口，通过追查createMockHandler函数我们可以清楚，MockHandler对象的实例是InvocationNotifierHandler类型，但它只是负责对外的包装，内部实际起作用的是MockHandlerImpl这个家伙，也是上文类图右下角的那位，这个类可谓承载了Mockito的主要逻辑，我们后面会详细的来说明。这里要先有个印象。

接着往下看，会调用mockMaker来创建最终的实例，这个MockHandler也是一个接口，其实现为ByteBuddyMockMaker，我们追进去看一下createMock函数（省略异常处理代码）

注意MockBytecodeGenerator也是UML中标红的一个类，这里的代码比较多，但做的事情比较单一，就是要动态生成一个代理类，这个代理类继承了目标类，然后实现了MockAccess接口。

很早的时候Java编译器已经用Java语言重写，能够自己编译自己，而从Java 1.6开始，编译器接口正式放到JDK的公开API，大家感兴趣可以查看JavaCompiler这个类。

而Mockito则使用的是ByteBuddy这个框架，它并不需要编译器的帮助，而是直接生成class，然后使用ClassLoader来进行加载，感兴趣的可以深入研究，其地址为:https://github.com/raphw/byte-buddy,

我们可以知道，这个也许就是前文所说实现MockAccess接口所传进来的，而这个handler，则是我们前文所说的重点: MockHandlerImpl!(准确的说是InvocationNotifierHandler，只不过其内部将主要逻辑都转交给了MockHandlerImpl， 所以下文都已MockHandlerImpl为主)


### 实例化
```
Instantiator instantiator = Plugins.getInstantiatorProvider().getInstantiator(settings);
T mockInstance = null;
mockInstance = instantiator.newInstance(mockedProxyType);

```

这个Instantiator也是一个接口，它有两个实现，一是ObjenesisInstantiator,另外一个是ConstructorInstantiator，默认情况下，都是在使用 ObjenesisInstantiator， 所以我们这里只查看ObjenesisInstantiator，并且ObjenesisInstantiator也是上述UML图中的一个标红的类，因为它承载生成对象的工作。我么看其newInstance方法:

这里调用了objenesis.newInstance，那这个objenesis是何许人也呢？这又是一个相当牛逼的框架，可以根据不同的平台选择不同的方法来new对象。总之一句话，你只要输入一个class进去，它就会输出其一个实例对象。感兴趣的可以查看其Github:https://github.com/easymock/objenesis。




# 参考
官方文档: http://static.javadoc.io/org.mockito/mockito-core/2.9.0/org/mockito/Mockito.html
https://blog.saymagic.cn/2016/09/17/understand-mockito.html