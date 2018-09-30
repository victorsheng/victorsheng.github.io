---
title: geektime-jvm-08-drynicinvoke
abbrlink: 760260758
date: 2018-08-11 15:03:12
tags:
categories:
---

# 回顾
具体来说，Java 字节码中与调用相关的指令共有五种。
1. invokestatic：用于调用静态方法。
2. invokespecial：用于调用私有实例方法、构造器，以及使用 super 关键字调用父类的实例方法或构造器，和所实现接口的默认方法。
3. invokevirtual：用于调用非私有实例方法。
4. invokeinterface：用于调用接口方法。
5. invokedynamic：用于调用动态方法。


# 引入
可以看到，在这四种调用指令中，Java 虚拟机明确要求方法调用需要提供目标方法的类名。在这种体系下，我们有两个解决方案。
- 一是调用其中一种类型的赛跑方法，比如说马类的赛跑方法。对于非马的类型，则给它套一层马甲，当成马来赛跑。
- 另外一种解决方式，是通过反射机制，来查找并且调用各个类型中的赛跑方法，以此模拟真正的赛跑。


显然，比起直接调用，这两种方法都相当复杂，执行效率也可想而知。
为了解决这个问题，Java 7 引入了一条新的指令 invokedynamic。该指令的调用机制抽象出调用点这一个概念，并允许应用程序将调用点链接至任意符合条件的方法上。

```
public static void startRace(java.lang.Object)
       0: aload_0                // 加载一个任意对象
       1: invokedynamic race     // 调用赛跑方法
```


作为 invokedynamic 的准备工作，Java 7 引入了更加底层、更加灵活的方法抽象 ：方法句柄（MethodHandle）。

# 方法句柄的概念

方法句柄是一个强类型的，能够被直接执行的引用 [2]。该引用可以指向常规的静态方法或者实例方法，也可以指向构造器或者字段。当指向字段时，方法句柄实则指向包含字段访问字节码的虚构方法，语义上等价于目标字段的 getter 或者 setter 方法。
这里需要注意的是，它并不会直接指向目标字段所在类中的 getter/setter，毕竟你无法保证已有的 getter/setter 方法就是在访问目标字段。
方法句柄的类型（MethodType）是由所指向方法的参数类型以及返回类型组成的。它是用来确认方法句柄是否适配的唯一关键。当使用方法句柄时，我们其实并不关心方法句柄所指向方法的类名或者方法名。
打个比方，如果兔子的“赛跑”方法和“睡觉”方法的参数类型以及返回类型一致，那么对于兔子递过来的一个方法句柄，我们并不知道会是哪一个方法

## api
**方法句柄的创建是通过 MethodHandles.Lookup 类来完成的**。它提供了多个 API，既可以使用反射 API 中的 Method 来查找，也可以根据类、方法名以及方法句柄类型来查找。

当使用后者这种查找方式时，用户需要区分具体的调用类型，
- 比如说对于用 invokestatic 调用的静态方法，我们需要使用 Lookup.findStatic 方法；
- 对于用 invokevirutal 调用的实例方法，以及用 invokeinterface 调用的接口方法，我们需要使用 findVirtual 方法；
- 对于用 invokespecial 调用的实例方法，我们则需要使用 findSpecial 方法。

**调用方法句柄，和原本对应的调用指令是一致的。也就是说，对于原本用 invokevirtual 调用的方法句柄，它也会采用动态绑定；而对于原本用 invkespecial 调用的方法句柄，它会采用静态绑定。**

// 获取方法句柄的不同方式
## 方式1
MethodHandles.Lookup l = Foo.lookup(); // 具备 Foo 类的访问权限
Method m = Foo.class.getDeclaredMethod("bar", Object.class);
MethodHandle mh0 = l.unreflect(m);


## 方式2
MethodType t = MethodType.methodType(void.class, Object.class);
MethodHandle mh1 = l.findStatic(Foo.class, "bar", t);


## 权限
方法句柄同样也有权限问题。但它与反射 API 不同，其权限检查是在句柄的创建阶段完成的。在实际调用过程中，Java 虚拟机并不会检查方法句柄的权限。如果该句柄被多次调用的话，那么与反射调用相比，它将省下重复权限检查的开销。
需要注意的是，方法句柄的访问权限不取决于方法句柄的创建位置，而是取决于 Lookup 对象的创建位置。
举个例子，对于一个私有字段，如果 Lookup 对象是在私有字段所在类中获取的，那么这个 Lookup 对象便拥有对该私有字段的访问权限，即使是在所在类的外边，也能够通过该 Lookup 对象创建该私有字段的 getter 或者 setter。
由于方法句柄没有运行时权限检查，因此，应用程序需要负责方法句柄的管理。一旦它发布了某些指向私有方法的方法句柄，那么这些私有方法便被暴露出去了。

# 方法句柄的操作
方法句柄的调用可分为两种
## 一是需要严格匹配参数类型的 invokeExact。
它有多严格呢？假设一个方法句柄将接收一个 Object 类型的参数，如果你直接传入 String 作为实际参数，那么方法句柄的调用会在运行时抛出方法类型不匹配的异常。正确的调用方式是将该 String 显式转化为 Object 类型。

## 如果你需要自动适配参数类型，那么你可以选取方法句柄的第二种调用方式 invoke。
它同样是一个签名多态性的方法。invoke 会调用 MethodHandle.asType 方法，生成一个适配器方法句柄，对传入的参数进行适配，再调用原方法句柄。调用原方法句柄的返回值同样也会先进行适配，然后再返回给调用者。


?????
方法句柄还支持增删改参数的操作，这些操作都是通过生成另一个方法句柄来实现的。这其中，改操作就是刚刚介绍的 MethodHandle.asType 方法。删操作指的是将传入的部分参数就地抛弃，再调用另一个方法句柄。它对应的 API 是 MethodHandles.dropArguments 方法。

?????
增操作则非常有意思。它会往传入的参数中插入额外的参数，再调用另一个方法句柄，它对应的 API 是 MethodHandle.bindTo 方法。Java 8 中捕获类型的 Lambda 表达式便是用这种操作来实现的，下一篇我会详细进行解释。
增操作还可以用来实现方法的柯里化 [3]。举个例子，有一个指向 f(x, y) 的方法句柄，我们可以通过将 x 绑定为 4，生成另一个方法句柄 g(y) = f(4, y)。在执行过程中，每当调用 g(y) 的方法句柄，它会在参数列表最前面插入一个 4，再调用指向 f(x, y) 的方法句柄。

# 方法句柄的实现

```
import java.lang.invoke.*;

public class Foo {
  public static void bar(Object o) {
    new Exception().printStackTrace();
  }

  public static void main(String[] args) throws Throwable {
    MethodHandles.Lookup l = MethodHandles.lookup();
    MethodType t = MethodType.methodType(void.class, Object.class);
    MethodHandle mh = l.findStatic(Foo.class, "bar", t);
    mh.invokeExact(new Object());
  }
}

```



打印堆栈信息后，invokeExact 的目标方法竟然就是方法句柄指向的方法。

先别高兴太早。我刚刚提到过，invokeExact 会对参数的类型进行校验，并在不匹配的情况下抛出异常。如果它直接调用了方法句柄所指向的方法，那么这部分参数类型校验的逻辑将无处安放。因此，唯一的可能便是 Java 虚拟机隐藏了部分栈信息。
当我们启用了 -XX:+ShowHiddenFrames 这个参数来打印被 Java 虚拟机隐藏了的栈信息时，你会发现 main 方法和目标方法中间隔着两个貌似是生成的方法。


??????
中间的没看懂
??????

可以看到，方法句柄的调用和反射调用一样，都是间接调用。因此，它也会面临无法内联的问题。不过，与反射调用不同的是，方法句柄的内联瓶颈在于即时编译器能否将该方法句柄识别为常量。具体内容我会在下一篇中进行详细的解释。


# 总结

今天我介绍了 invokedynamic 底层机制的基石：方法句柄。
方法句柄是一个强类型的、能够被直接执行的引用。它仅关心所指向方法的参数类型以及返回类型，而不关心方法所在的类以及方法名。方法句柄的权限检查发生在创建过程中，相较于反射调用节省了调用时反复权限检查的开销。
方法句柄可以通过 invokeExact 以及 invoke 来调用。其中，invokeExact 要求传入的参数和所指向方法的描述符严格匹配。方法句柄还支持增删改参数的操作，这些操作是通过生成另一个充当适配器的方法句柄来实现的。
方法句柄的调用和反射调用一样，都是间接调用，同样会面临无法内联的问题。