---
title: geektime-jvm-09-invoke-dynamic
date: 2018-08-14 22:12:54
tags:
categories:
---
我们并没有讲解 invokedynamic，而是深入地探讨了它所依赖的方法句柄。

# invokedynamic 指令
体来说，它将调用点（CallSite）抽象成一个 Java 类，并且将原本由 Java 虚拟机控制的方法调用以及方法链接暴露给了应用程序。在运行过程中，每一条 invokedynamic 指令将捆绑一个调用点，并且会调用该调用点所链接的方法句柄。



在字节码中，启动方法是用方法句柄来指定的。这个方法句柄指向一个返回类型为调用点的静态方法。该方法必须接收三个固定的参数，分别为一个 Lookup 类实例，一个用来指代目标方法名字的字符串，以及该调用点能够链接的方法句柄的类型。

由于 Java 暂不支持直接生成 invokedynamic 指令 [1]，所以接下来我会借助之前介绍过的字节码工具 ASM 来实现这一目的。......


# Java 8 的 Lambda 表达式

在 Java 8 中，Lambda 表达式也是借助 invokedynamic 来实现的。

具体来说，Java 编译器利用 invokedynamic 指令来生成实现了函数式接口的适配器。这里的函数式接口指的是仅包括一个非 default 接口方法的接口，一般通过 @FunctionalInterface 注解。不过就算是没有使用该注解，Java 编译器也会将符合条件的接口辨认为函数式接口。

```
int x = ..
IntStream.of(1, 2, 3).map(i -> i * 2).map(i -> i * x);
```

举个例子，上面这段代码会对 IntStream 中的元素进行两次映射。我们知道，映射方法 map 所接收的参数是 IntUnaryOperator（这是一个函数式接口）。也就是说，在运行过程中我们需要将 i->i2 和 i->ix 这两个 Lambda 表达式转化成 IntUnaryOperator 的实例。这个转化过程便是由 invokedynamic 来实现的。

在编译过程中，Java 编译器会对 Lambda 表达式进行解语法糖（desugar），生成一个方法来保存 Lambda 表达式的内容。该方法的参数列表不仅包含原本 Lambda 表达式的参数，还包含它所捕获的变量。(注：方法引用，如 Horse::race，则不会生成生成额外的方法。)
在上面那个例子中，第一个 Lambda 表达式没有捕获其他变量，而第二个 Lambda 表达式（也就是 i->i*x）则会捕获局部变量 x。这两个 Lambda 表达式对应的方法如下所示。可以看到，所捕获的变量同样也会作为参数传入生成的方法之中。

```
public static void main(java.lang.String[]);
    Code:
       0: bipush        10
       2: istore_1
       3: iconst_3
       4: newarray       int
       6: dup
       7: iconst_0
       8: iconst_1
       9: iastore
      10: dup
      11: iconst_1
      12: iconst_2
      13: iastore
      14: dup
      15: iconst_2
      16: iconst_3
      17: iastore
      18: invokestatic  #2                  // InterfaceMethod java/util/stream/IntStream.of:([I)Ljava/util/stream/IntStream;
      21: invokedynamic #3,  0              // InvokeDynamic #0:applyAsInt:()Ljava/util/function/IntUnaryOperator;
      26: invokeinterface #4,  2            // InterfaceMethod java/util/stream/IntStream.map:(Ljava/util/function/IntUnaryOperator;)Ljava/util/stream/IntStream;
      31: iload_1
      32: invokedynamic #5,  0              // InvokeDynamic #1:applyAsInt:(I)Ljava/util/function/IntUnaryOperator;
      37: invokeinterface #4,  2            // InterfaceMethod java/util/stream/IntStream.map:(Ljava/util/function/IntUnaryOperator;)Ljava/util/stream/IntStream;
      42: astore_2
      43: return


// i -> i * 2
  private static int lambda$0(int);
    Code:
       0: iload_0
       1: iconst_2
       2: imul
       3: ireturn

  // i -> i * x
  private static int lambda$1(int, int);
    Code:
       0: iload_1
       1: iload_0
       2: imul
       3: ireturn
```

第一次执行 invokedynamic 指令时，它所对应的启动方法会通过 ASM 来生成一个适配器类。这个适配器类实现了对应的函数式接口，在我们的例子中，也就是 IntUnaryOperator。启动方法的返回值是一个 ConstantCallSite，其链接对象为一个返回适配器类实例的方法句柄。
根据 Lambda 表达式是否捕获其他变量，启动方法生成的适配器类以及所链接的方法句柄皆不同。
- 如果该 Lambda 表达式没有捕获其他变量，那么可以认为它是上下文无关的。因此，启动方法将新建一个适配器类的实例，并且生成一个特殊的方法句柄，始终返回该实例。
- 如果该 Lambda 表达式捕获了其他变量，那么每次执行该 invokedynamic 指令，我们都要更新这些捕获了的变量，以防止它们发生了变化。



# Lambda 以及方法句柄的性能分析


# 总结与实践 
invokedymaic 指令抽象出调用点的概念，并且将调用该调用点所链接的方法句柄。在第一次执行 invokedynamic 指令时，Java 虚拟机将执行它所对应的启动方法，生成并且绑定一个调用点。之后如果再次执行该指令，Java 虚拟机则直接调用已经绑定了的调用点所链接的方法。
Lambda 表达式到函数式接口的转换是通过 invokedynamic 指令来实现的。该 invokedynamic 指令对应的启动方法将通过 ASM 生成一个适配器类。
- 对于没有捕获其他变量的 Lambda 表达式，该 invokedynamic 指令始终返回同一个适配器类的实例。
- 对于捕获了其他变量的 Lambda 表达式，每次执行 invokedynamic 指令将新建一个适配器类实例。
不管是捕获型的还是未捕获型的 Lambda 表达式，它们的性能上限皆可以达到直接调用的性能。其中，捕获型 Lambda 表达式借助了即时编译器中的逃逸分析，来避免实际的新建适配器类实例的操作。