title: geektime-jvm-06a-tools
date: 2018-08-07 08:14:06
tags:
categories:
---
# 1.javap：查阅 Java 字节码
$ javap -p -v Foo
这里面我用到了两个选项。第一个选项是 -p。默认情况下 javap 会打印所有非私有的字段和方法，当加了 -p 选项后，它还将打印私有的字段和方法。第二个选项是 -v。它尽可能地打印所有信息。如果你只需要查阅方法对应的字节码，那么可以用 -c 选项来替换 -v。$ javap -p -v Foo

javap 的 -v 选项的输出分为几大块。
## 1. 基本信息，涵盖了原 class 文件的相关信息。
class 文件的版本号（minor version: 0，major version: 54），该类的访问权限（flags: (0x0021) ACC_PUBLIC, ACC_SUPER），该类（this_class: #7）以及父类（super_class: #8）的名字，所实现接口（interfaces: 0）、字段（fields: 4）、方法（methods: 2）以及属性（attributes: 1）的数目。
这里属性指的是 class 文件所携带的辅助信息，比如该 class 文件的源文件的名称。这类信息通常被用于 Java 虚拟机的验证和运行，以及 Java 程序的调试，一般无须深入了解。

```
Classfile ../Foo.class
  Last modified ..; size 541 bytes
  MD5 checksum 3828cdfbba56fea1da6c8d94fd13b20d
  Compiled from "Foo.java"
public class Foo
  minor version: 0
  major version: 54
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #7                          // Foo
  super_class: #8                         // java/lang/Object
  interfaces: 0, fields: 4, methods: 2, attributes: 1
```

class 文件的版本号指的是编译生成该 class 文件时所用的 JRE 版本。由较新的 JRE 版本中的 javac 编译而成的 class 文件，不能在旧版本的 JRE 上跑，否则，会出现如下异常信息。（Java 8 对应的版本号为 52，Java 10 对应的版本号为 54。）

## 2. 常量池，用来存放各种常量以及符号引用。
常量池中的每一项都有一个对应的索引（如 #1），并且可能引用其他的常量池项（#1 = Methodref #8.#23）。
```
Constant pool:
   #1 = Methodref          #8.#23         // java/lang/Object."<init>":()V
... 
   #8 = Class              #30            // java/lang/Object
...
  #14 = Utf8               <init>
  #15 = Utf8               ()V
...
  #23 = NameAndType        #14:#15        // "<init>":()V
...
  #30 = Utf8               java/lang/Object
```


![upload successful](/images/pasted-225.png)

## 3. 字段区域，用来列举该类中的各个字段。
这里最主要的信息便是该字段的类型（descriptor: I）以及访问权限（flags: (0x0002) ACC_PRIVATE）。对于声明为 final 的静态字段而言，如果它是基本类型或者字符串类型，那么字段区域还将包括它的常量值。
另外，Java 虚拟机同样使用了“描述符”（descriptor）来描述字段的类型。具体的对照如下表所示。其中比较特殊的，我已经高亮显示。

## 4. 方法区域，用来列举该类中的各个方法。
除了方法描述符以及访问权限之外，每个方法还包括最为重要的代码区域（Code:)。
```
public void test();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
...
        10: goto          35
...
        34: athrow
        35: aload_0
...
        40: return
      Exception table:
         from    to  target type
             0     5    13   Class java/lang/Exception
             0     5    27   any
            13    19    27   any
      LineNumberTable:
        line 9: 0
...
        line 16: 40
      StackMapTable: number_of_entries = 3
        frame_type = 77 /* same_locals_1_stack_item */
          stack = [ class java/lang/Exception ]
...
```
代码区域一开始会声明该方法中的操作数栈（stack=2）和局部变量数目（locals=3）的最大值，以及该方法接收参数的个数（args_size=1）。注意这里局部变量指的是字节码中的局部变量，而非 Java 程序中的局部变量。
接下来则是该方法的字节码。每条字节码均标注了对应的偏移量（bytecode index，BCI），这是用来定位字节码的。比如说偏移量为 10 的跳转字节码 10: goto 35，将跳转至偏移量为 35 的字节码 35: aload_0。
紧跟着的异常表（Exception table:）也会使用偏移量来定位每个异常处理器所监控的范围（由 from 到 to 的代码区域），以及异常处理器的起始位置（target）。除此之外，它还会声明所捕获的异常类型（type）。其中，any 指代任意异常类型。
再接下来的行数表（LineNumberTable:）则是 Java 源程序到字节码偏移量的映射。如果你在编译时使用了 -g 参数（javac -g Foo.java），那么这里还将出现局部变量表（LocalVariableTable:），展示 Java 程序中每个局部变量的名字、类型以及作用域。
行数表和局部变量表均属于调试信息。Java 虚拟机并不要求 class 文件必备这些信息。

# 2.OpenJDK 项目 Code Tools：实用小工具集
OpenJDK 的 Code Tools 项目 [2] 包含了好几个实用的小工具。
在第一篇的实践环节中，我们使用了其中的字节码汇编器反汇编器 ASMTools[3]，当前 6.0 版本的下载地址位于 [4]。ASMTools 的反汇编以及汇编操作所对应的命令分别为：
```
$ java -cp /path/to/asmtools.jar org.openjdk.asmtools.jdis.Main Foo.class > Foo.jasm
```
和
```
$ java -cp /path/to/asmtools.jar org.openjdk.asmtools.jasm.Main Foo.jasm
```

该反汇编器的输出格式和 javap 的不尽相同。一般我只使用它来进行一些简单的字节码修改，以此生成无法直接由 Java 编译器生成的类，它在 HotSpot 虚拟机自身的测试中比较常见。
在第一篇的实践环节中，我们需要将整数 2 赋值到一个声明为 boolean 类型的局部变量中。我采取的做法是将编译生成的 class 文件反汇编至一个文本文件中，然后找到 boolean flag = true 对应的字节码序列，也就是下面的两个。

```
iconst_1;
istore_1;
```

将这里的 iconst_1 改为 iconst_2[5]，保存后再汇编至 class 文件即可完成第一篇实践环节的需求。
除此之外，你还可以利用这一套工具来验证我之前文章中的一些结论。比如我说过 class 文件允许出现参数类型相同、而返回类型不同的方法，并且，在作为库文件时 Java 编译器将使用先定义的那一个，来决定具体的返回类型。
具体的验证方法便是在反汇编之后，利用文本编辑工具复制某一方法，并且更改该方法的描述符，保存后再汇编至 class 文件。
Code Tools 项目还包含另一个实用的小工具 JOL[6]，当前 0.9 版本的下载地址位于 [7]。JOL 可用于查阅 Java 虚拟机中对象的内存分布，具体可通过如下两条指令来实现。

```
$ java -jar /path/to/jol-cli-0.9-full.jar internals java.util.HashMap
$ java -jar /path/to/jol-cli-0.9-full.jar estimates java.util.HashMap
```



# 3.ASM：Java 字节码框架
ASM[8] 是一个字节码分析及修改框架。它被广泛应用于许多项目之中，例如 Groovy、Kotlin 的编译器，代码覆盖测试工具 Cobertura、JaCoCo，以及各式各样通过字节码注入实现的程序行为监控工具。甚至是 Java 8 中 Lambda 表达式的适配器类，也是借助 ASM 来动态生成的。
ASM 既可以生成新的 class 文件，也可以修改已有的 class 文件。前者相对比较简单一些。ASM 甚至还提供了一个辅助类 ASMifier，它将接收一个 class 文件并且输出一段生成该 class 文件原始字节数组的代码。如果你想快速上手 ASM 的话，那么你可以借助 ASMifier 生成的代码来探索各个 API 的用法。


