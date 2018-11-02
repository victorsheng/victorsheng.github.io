---
title: Java Agent与字节码注入
abbrlink: 2860365721
date: 2018-10-25 19:11:09
tags:
categories:
---
# premain机制
为了能够以 Java agent 的方式运行该premain方法，我们需要将其打包成 jar 包，并在其中的 MANIFEST.MF 配置文件中，指定所谓的Premain-class。具体的命令如下所示：


- 命令行中指定 Java agent
- 我们还可以通过 Attach API 远程加载

# agentmain机制
使用 Attach API 远程加载的 Java agent 不会再先于main方法执行，这取决于另一虚拟机调用 Attach API 的时机。并且，它运行的也不再是premain方法，而是名为agentmain的方法。

相应的，我们需要更新 jar 包中的 manifest 文件，使其包含Agent-Class的配置，例如Agent-Class: org.example.MyAgent。

Java 虚拟机并不限制 Java agent 的数量。你可以在 java 命令后附上多个-javaagent参数，或者远程 attach 多个 Java agent，Java 虚拟机会按照定义顺序，或者 attach 的顺序逐个执行这些 Java agent。

在premain方法或者agentmain方法中打印一些字符串并不出奇，我们完全可以将其中的逻辑并入main方法，或者其他监听端口的线程中。除此之外，Java agent 还提供了一套 instrumentation 机制，允许应用程序拦截类加载事件，并且更改该类的字节码。

# 字节码注入(instrumentation机制)

我们先来看一个例子。在上面这段代码中，premain方法多出了一个Instrumentation类型的参数，我们可以通过它来注册类加载事件的拦截器。该拦截器需要实现ClassFileTransformer接口，并重写其中的transform方法。

## transform

- transform方法将接收一个 byte 数组类型的参数，它代表的是正在被加载的类的字节码。在上面这段代码中，我将打印该数组的前四个字节，也就是 Java class 文件的魔数（magic number）0xCAFEBABE。
- transform方法将返回一个 byte 数组，代表更新过后的类的字节码。当方法返回之后，Java 虚拟机会使用所返回的 byte 数组，来完成接下来的类加载工作。不过，如果transform方法返回 null 或者抛出异常，那么 Java 虚拟机将使用原来的 byte 数组完成类加载工作。

**基于这一类加载事件的拦截功能，我们可以实现字节码注入（bytecode instrumentation），往正在被加载的类中插入额外的字节码。**


Java agent 还提供了另外两个功能redefine和retransform。这两个功能针对的是已加载的类，并要求用户传入所要redefine或者retransform的类实例。

## redefine
redefine指的是舍弃原本的字节码，并替换成由用户提供的 byte 数组。该功能比较危险，一般用于修复出错了的字节码。

## retransform
retransform则将针对所传入的类，重新调用所有已注册的ClassFileTransformer的transform方法。它的应用场景主要有如下两个。

- 第一，在执行premain或者agentmain方法前，Java 虚拟机早已加载了不少类，而这些类的加载事件并没有被拦截，因此也没有被注入。使用retransform功能可以注入这些已加载但未注入的类。
- 第二，在定义了多个 Java agent，多个注入的情况下，我们可能需要移除其中的部分注入。当调用Instrumentation.removeTransformer去除某个注入类后，我们可以调用retransform功能，重新从原始 byte 数组开始进行注入。

Java agent 的这些功能都是通过 JVMTI agent，也就是 C agent 来实现的。JVMTI 是一个事件驱动的工具实现接口，通常，我们会在 C agent 加载后的入口方法Agent_OnLoad处注册各个事件的钩子（hook）方法。当 Java 虚拟机触发了这些事件时，便会调用对应的钩子方法。


# 基于字节码注入的 profiler
我们可以利用字节码注入来实现代码覆盖工具（例如JaCoCo），或者各式各样的 profiler。

举个例子，上面这段代码便是一个运行时类。该类维护了一个HashMap，用来统计每个类所新建实例的数目。当程序退出时，我们将逐个打印出每个类的名字，以及其新建实例的数目。

## 死循环 

举个例子，假设我们在PrintStream.println方法入口处注入System.out.println("blahblah")，由于out是PrintStream的实例，因此当执行注入代码时，我们又会调用PrintStream.println方法，从而造成死循环。
解决这一问题的关键在于设置一个线程私有的标识位，用以区分应用代码的上下文以及注入代码的上下文。当即将执行注入代码时，我们将根据标识位判断是否已经位于注入代码的上下文之中。如果不是，则设置标识位并正常执行注入代码；如果是，则直接返回，不再执行注入代码。

## 命名空间

字节码注入的另一个技术难点则是命名空间。举个例子，不少应用程序都依赖于字节码工程库 ASM。当我们的注入逻辑依赖于 ASM 时，便有可能出现注入使用最新版本的 ASM，而应用程序使用较低版本的 ASM 的问题。
JDK 本身也使用了 ASM 库，如用来生成 Lambda 表达式的适配器类。JDK 的做法是重命名整个 ASM 库，为所有类的包名添加jdk.internal前缀。我们显然不好直接更改 ASM 的包名，因此需要借助自定义类加载器来隔离命名空间。

## 观察者效应

除了上述技术难点之外，基于字节码注入的工具还有另一个问题，那便是观察者效应（observer effect）对所收集的数据造成的影响。
举个利用字节码注入收集每个方法的运行时间的例子。假设某个方法调用了另一个方法，而这两个方法都被注入了，那么统计被调用者运行时间的注入代码所耗费的时间，将不可避免地被计入至调用者方法的运行时间之中。
再举一个统计新建对象数目的例子。我们知道，即时编译器中的逃逸分析可能会优化掉新建对象操作，但它不会消除相应的统计操作，比如上述例子中对fireAllocationEvent方法的调用。在这种情况下，我们将统计没有实际发生的新建对象操作。
另一种情况则是，我们所注入的对fireAllocationEvent方法的调用，将影响到方法内联的决策。如果该新建对象的构造器调用恰好因此没有被内联，从而造成对象逃逸。在这种情况下，原本能够被逃逸分析优化掉的新建对象操作将无法优化，我们也将统计到原本不会发生的新建对象操作。
总而言之，当使用字节码注入开发 profiler 时，需要辩证地看待所收集的数据。它仅能表示在被注入的情况下程序的执行状态，而非没有注入情况下的程序执行状态。

# 面向方面编程
说到字节码注入，就不得不提面向方面编程（Aspect-Oriented Programming，AOP）。面向方面编程的核心理念是定义切入点（pointcut）以及通知（advice）。程序控制流中所有匹配该切入点的连接点（joinpoint）都将执行这段通知代码。
举个例子，我们定义一个指代所有方法入口的切入点，并指定在该切入点执行的“打印该方法的名字”这一通知。那么每个具体的方法入口便是一个连接点。
面向方面编程的其中一种实现方式便是字节码注入，比如AspectJ。

# 总结与实现
今天我介绍了 Java agent 以及字节码注入。

我们可以通过 Java agent 的类加载拦截功能，修改某个类所对应的 byte 数组，并利用这个修改过后的 byte 数组完成接下来的类加载。
基于字节码注入的 profiler，可以统计程序运行过程中某些行为的出现次数。如果需要收集 Java 核心类库的数据，那么我们需要小心避免无限递归调用。另外，我们还需通过自定义类加载器来解决命名空间的问题。
由于字节码注入会产生观察者效应，因此基于该技术的 profiler 所收集到的数据并不能反映程序的真实运行状态。它所反映的是程序在被注入的情况下的执行状态。