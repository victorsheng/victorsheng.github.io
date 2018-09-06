title: geektime-jvm-17-jit-2
date: 2018-08-29 20:31:45
tags:
categories:
---
# Profiling

分层编译中的 0 层、2 层和 3 层都会进行 profiling，收集能够反映程序执行状态的数据。其中，最为基础的便是方法的调用次数以及循环回边的执行次数。它们被用于触发即时编译。

此外，0 层和 3 层还会收集用于 4 层 C2 编译的数据，比如说分支跳转字节码的分支 profile（branch profile），包括跳转次数和不跳转次数，以及非私有实例方法调用指令、强制类型转换 checkcast 指令、类型测试 instanceof 指令，和引用类型的数组存储 aastore 指令的类型 profile（receiver type profile）。
- 分支profile
- 类型profile

分支 profile 和类型 profile 的收集将给应用程序带来不少的性能开销。据统计，正是因为这部分额外的 profiling，使得 3 层 C1 代码的性能比 2 层 C1 代码的低 30%。

那么这些耗费巨大代价收集而来的 profile 具体有什么作用呢？
答案是，C2 可以根据收集得到的数据进行猜测，假设接下来的执行同样会按照所收集的 profile 进行，**从而作出比较激进的优化**。


# 基于分支 profile 的优化

```
public static int foo(boolean f, int in) {
  int v;
  if (f) {
    v = in;
  } else {
    v = (int) Math.sin(in);
  }

  if (v == in) {
    return 0;
  } else {
    return (int) Math.cos(v);
  }
}
// 编译而成的字节码：
public static int foo(boolean, int);
  Code:
     0: iload_0
     1: ifeq          9
     4: iload_1
     5: istore_2
     6: goto          16
     9: iload_1
    10: i2d
    11: invokestatic  java/lang/Math.sin:(D)D
    14: d2i
    15: istore_2
    16: iload_2
    17: iload_1
    18: if_icmpne     23
    21: iconst_0
    22: ireturn
    23: iload_2
    24: i2d
    25: invokestatic java/lang/Math.cos:(D)D
    28: d2i
    29: ireturn
```


![upload successful](/images/pasted-234.png)



![upload successful](/images/pasted-235.png)
在第一个条件跳转之后，C2 代码将直接返回 0。


总结一下，根据条件跳转指令的分支 profile，即时编译器可以将从未执行过的分支剪掉，以避免编译这些很有可能不会用到的代码，从而节省编译时间以及部署代码所要消耗的内存空间。此外，“剪枝”将精简程序的数据流，从而触发更多的优化。
在现实中，分支 profile 出现仅跳转或者仅不跳转的情况并不多见。当然，即时编译器对分支 profile 的利用也不仅限于“剪枝”。它还会根据分支 profile，计算每一条程序执行路径的概率，以便某些编译器优化优先处理概率较高的路径。

# 基于类型 profile 的优化

另外一个例子则是关于 instanceof 以及方法调用的类型 profile。下面这段代码将测试所传入的对象是否为 Exception 的实例，如果是，则返回它的系统哈希值；如果不是，则返回它的哈希值。

```
public static int hash(Object in) {
  if (in instanceof Exception) {
    return System.identityHashCode(in);
  } else {
    return in.hashCode();
  }
}
// 编译而成的字节码：
public static int hash(java.lang.Object);
  Code:
     0: aload_0
     1: instanceof java/lang/Exception
     4: ifeq          12
     7: aload_0
     8: invokestatic java/lang/System.identityHashCode:(Ljava/lang/Object;)I
    11: ireturn
    12: aload_0
    13: invokevirtual java/lang/Object.hashCode:()I
    16: ireturn
```


![upload successful](/images/pasted-236.png)


在 Java 虚拟机中，instanceof 测试并不简单。如果 instanceof 的目标类型是 final 类型，那么 Java 虚拟机仅需比较测试对象的动态类型是否为该 final 类型。
在讲解对象的内存分布那一篇中，我曾经提到过，对象头存有该对象的动态类型。因此，获取对象的动态类型仅为单一的内存读指令。
如果目标类型不是 final 类型，比如说我们例子中的 Exception，那么 Java 虚拟机需要从测试对象的动态类型开始，依次测试该类，该类的父类、祖先类，该类所直接实现或者间接实现的接口是否与目标类型一致。
不过，在我们的例子中，instanceof 指令的类型 profile 仅包含 Integer。根据这个信息，即时编译器可以假设，在接下来的执行过程中，所输入的 Object 对象仍为 Integer 实例。

因此，生成的代码将测试所输入的对象的动态类型是否为 Integer。如果是的话，则继续执行接下来的代码。（该优化源自 Graal，采用 C2 可能无法复现。）

然后，即时编译器会采用和第一个例子中一致的针对分支 profile 的优化，以及对方法调用的条件去虚化内联。
我会在接下来的篇章中详细介绍内联，这里先说结果：生成的代码将测试所输入的对象动态类型是否为 Integer。如果是的话，则执行 Integer.hashCode() 方法的实质内容，也就是返回该 Integer 实例的 value 字段。


![upload successful](/images/pasted-237.png)

简化为:
![upload successful](/images/pasted-238.png)

和基于分支 profile 的优化一样，基于类型 profile 的优化同样也是作出假设，从而精简控制流以及数据流。这两者的核心都是假设。
对于分支 profile，即时编译器假设的是仅执行某一分支；对于类型 profile，即时编译器假设的是对象的动态类型仅为类型 profile 中的那几个。
那么，当假设失败的情况下，程序将何去何从？我们继续往下看。

# 去优化

Java 虚拟机给出的解决方案便是去优化，即从执行即时编译生成的机器码切换回解释执行。

在生成的机器码中，即时编译器将在假设失败的位置上插入一个陷阱（trap）。该陷阱实际上是一条 call 指令，调用至 Java 虚拟机里专门负责去优化的方法。与普通的 call 指令不一样的是，去优化方法将更改栈上的返回地址，并不再返回即时编译器生成的机器码中。

**在上面的程序控制流图中，我画了很多红色方框的问号。这些问号便代表着一个个的陷阱。一旦踏入这些陷阱，便将发生去优化，并切换至解释执行。**

去优化的过程相当复杂。由于即时编译器采用了许多优化方式，其生成的代码和原本的字节码的差异非常之大。
在去优化的过程中，需要将当前机器码的执行状态转换至某一字节码之前的执行状态，并从该字节码开始执行。这便要求即时编译器在编译过程中记录好这两种执行状态的映射。

## 举例
举例来说，经过逃逸分析之后，机器码可能并没有实际分配对象，而是在各个寄存器中存储该对象的各个字段（标量替换，具体我会在之后的篇章中进行介绍）。在去优化过程中，Java 虚拟机需要还原出这个对象，以便解释执行时能够使用该对象。
当根据映射关系创建好对应的解释执行栈桢后，Java 虚拟机便会采用 OSR 技术，动态替换栈上的内容，并在目标字节码处开始解释执行。
此外，在调用 Java 虚拟机的去优化方法时，即时编译器生成的机器码可以根据产生去优化的原因来决定是否保留这一份机器码，以及何时重新编译对应的 Java 方法。

Action_None，表示保留这一份机器码，在下一次调用该方法时重新进入这一份机器码。
如果去优化的原因与静态分析的结果有关，例如类层次分析，那么生成的机器码可以在调用去优化方法时传入 Action_Recompile，表示不保留这一份机器码，但是可以不经过重新 profile，直接重新编译。
如果去优化的原因与基于 profile 的激进优化有关，那么生成的机器码需要在调用去优化方法时传入 Action_Reinterpret，表示不保留这一份机器码，而且需要重新收集程序的 profile。
这是因为基于 profile 的优化失败的时候，往往代表这程序的执行状态发生改变，因此需要更正已收集的 profile，以更好地反映新的程序执行状态。

# 总结与实践
今天我介绍了 Java 虚拟机
- profiling 
- 基于所收集的数据的优化
- 去优化


通常情况下，解释执行过程中仅收集方法的调用次数以及循环回边的执行次数。

当方法被 3 层 C1 所编译时，生成的 C1 代码将收集条件跳转指令的分支 profile，以及类型相关指令的类型 profile。在部分极端情况下，Java 虚拟机也会在解释执行过程中收集这些 profile。

基于分支 profile 的优化以及基于类型 profile 的优化都将对程序今后的执行作出假设。这些假设将精简所要编译的代码的控制流以及数据流。在假设失败的情况下，Java 虚拟机将采取去优化，退回至解释执行并重新收集相关的 profile。

