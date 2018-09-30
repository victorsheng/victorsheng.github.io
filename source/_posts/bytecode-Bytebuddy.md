---
title: bytecode-Bytebuddy
abbrlink: 1212406173
date: 2018-08-14 19:57:22
tags:
categories:
---
# 为什么需要在运行时生成代码？
Java 语言带有一套比较严格的类型系统。Java 要求所有变量和对象都有一个确定的类型，并且任何向不兼容类型赋值都会造成一个错误。这些错误通常都会被编译器检查出来，极少情况下会被 Java 运行时检查到，然后抛一个非法类型的错误。如此严格的类型在大多数情况下是比较令人满意的，比如在编写业务应用时。通常，可以以任何模型元素表示其自己的类型这种明确的方式来描述业务域。通过这种方式，我们可以用 Java 构建具有非常强可读性和稳定性的应用，应用中的错误也非常贴近源码。除此之外，Java 严格的类型系统造就 Java 在企业编程中的普及。

Java 强加一些限制，在其他领域限制了语言应用范围。 比如，当写一个通用的库时，这个库将被其他 Java 应用使用，我们通常不能引用任何在用户应用中定义的类型，因为当这个库被编译时，我们还不知道这些类型。为了调用用户为知代码的方法或者访问其属性，Java 类库提供了一套反射 API。使用这套反射 API，我们就可以反省为知类型，进而调用方法或者访问属性。不幸的是，这套反射 API 的用法有两个明显的缺点：

https://docs.oracle.com/javase/tutorial/reflect/index.html

## 性能开销
由于反射涉及动态解析的类型，因此无法执行某些Java虚拟机优化。 因此，反射操作的性能低于非反射操作，并且应避免在性能敏感应用程序中频繁调用的代码段中。
## 安全限制
Reflection需要运行时权限，在安全管理器下运行时可能不存在。 对于必须在受限安全上下文中运行的代码，例如在Applet中，这是一个重要的考虑因素。
## 内部接触
由于反射允许代码执行在非反射代码中非法的操作，例如访问私有字段和方法，因此使用反射可能导致意外的副作用，这可能导致代码功能失常并可能破坏可移植性。 反射代码打破了抽象，因此可能会通过升级平台来改变行为。

## 小结:
- 性能
- 就没有编译时类型检查这些方法参数是否与方法的反射调用相匹配。方法调用依然会校验，只是被推迟到了运行时。这样做，我们就错失了 Java 编程语言的一大特性。

# 字节生成工具
## Java Proxy
Java 类库自带了一个代理工具，它允许为实现了一系列接口的类创建代理。这个内置的代理供应商非常方便，但局限性也特别明显。 上面提到的安全框架就不能用这样的方式来实现的，因为我们想扩展是类而不是扩展接口。

## cglib
代码生成库（注：这里指 cglib）诞生于 Java 初期，但不幸的是没有跟上 Java 平台的发展。然而，cglib 仍然是一个相当强大的库，但其积极的开发却变得相当模糊。鉴于此，其许多用户已经离开了 cglib。

## Javassist
该库附带一个编译器，它使用包含 Java 源代码的字符串，这些字符串在应用程序的运行时被转换为 Java 字节码。这是非常有前途的，本质上是一个好主意，因为 Java 源代码显然是描述 Java 类的好方法。但是，Javassist 编译器在功能上比不了 javac 编译器，并且在动态组合字符串以实现比较复杂的逻辑时容易出错。此外，Javassist 还提供了一个类似于 Java 类库中的代理工具，但允许扩展类，并不限于接口。然而，Javassist 的代理工具的范围在其 API 和功能上仍然受到限制。


## 性能对比

![upload successful](/images/pasted-228.png)

与静态编译器类似，代码生成库在生成快速代码和快速生成代码之间面临着折衷。当在这些冲突的目标之间进行选择时，Byte Buddy 的主要侧重点在于以最少的运行时生成代码。通常，类型创建或操作不是任何程序中的常见步骤，并不会对任何长期运行的应用程序产生重大影响；特别是因为类加载或类构建（class instrumentation）是运行此类代码时最耗时且不可避免的步骤。




# mockito从cglib替换成Bytebuddy
- ByteBuddy offers a more modern take on byte code manipulation than CGLIB
- ByteBuddy offers more features than CGLIB
- ByteBuddy is actively developed
- ByteBuddy may be the trigger to support final class support
https://github.com/mockito/mockito/pull/171


# 参考
https://notes.diguage.com/byte-buddy-tutorial/