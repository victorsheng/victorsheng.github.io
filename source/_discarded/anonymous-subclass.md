---
title: anonymous-subclass
abbrlink: 32121057
tags:
categories:
---

# 内存泄漏
使用静态内部类/匿名类，不要使用非静态内部类/匿名类.非静态内部类/匿名类会隐式的持有外部类的引用，外部类就有可能发生泄漏。而静态内部类/匿名类不会隐式的持有外部类引用，外部类会以正常的方式回收，如果你想在静态内部类/匿名类中使用外部类的属性或方法时，可以显示的持有一个弱引用。

# Lambda表达式与匿名内部类主要存在如下区别与相同点
区别
- 匿名内部类可以为任意接口创建实例——不管接口包含多少个抽象方法，只要匿名内部类实现所有的抽象方法即可；但Lambda表达式只能为函数式接口创建实例。
- 匿名内部类可以为抽象类甚至普通类创建实例；但ambda表达式只能为函数式接口创建实例。
- 匿名内部类实现的抽象方法的方法体允许调用接口中定义的默认方法；但Lambda表达式的代码块不允许调用接口中定义的默认方法。
联系
- Lambda表达式与匿名内部类一样，都可以直接访问“effectively final”的局部变量，以及外部类的成员变量（包括实例变量和类变量）。
- Lambda表达式创建的对象与匿名内部类生成的对象一样，都可以直接调用从接口中集成的默认方法。



# 测试
```
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class Anony {

  public void publicMethod() {
    Integer localVariable1 = 10;
    Integer localVariable2 = 10;
    Integer localVariable3 = 10;

    //继承HashMap
    Map<String, Integer> map = new HashMap<String, Integer>() {
      {
        put("a", localVariable1);
        put("b", localVariable2);
        put("c", localVariable3);
      }
    };

    Thread t = new Thread(new Runnable() {
      public void run() {
        System.out.println(localVariable1);
      }
    });

    Thread t2 = new Thread(() -> {
      System.out.println(localVariable1);
    });

    List<String> list = Arrays.asList("A", "B", "C");
    Collections.sort(list, new Comparator<String>() {
      public int compare(String p1, String p2) {
        return p1.compareTo(p2);
      }
    });

    List<String> list2 = Arrays.asList("A", "B", "C");
    Collections.sort(list, (p1, p2) -> {
      return p1.compareTo(p2);
    });
  }
}




```
## 匿名类反编译结果
```
import java.util.HashMap;

class Anony$1 extends HashMap<String, Integer> {
  Anony$1(Anony var1, Integer var2, Integer var3, Integer var4) {
    this.this$0 = var1;
    this.val$localVariable1 = var2;
    this.val$localVariable2 = var3;
    this.val$localVariable3 = var4;
    this.put("a", this.val$localVariable1);
    this.put("b", this.val$localVariable2);
    this.put("c", this.val$localVariable3);
  }
}


class Anony$2 implements Runnable {
  Anony$2(Anony var1, Integer var2) {
    this.this$0 = var1;
    this.val$localVariable1 = var2;
  }

  public void run() {
    System.out.println(this.val$localVariable1);
  }
}


import java.util.Comparator;

class Anony$3 implements Comparator<String> {
  Anony$3(Anony var1) {
    this.this$0 = var1;
  }

  public int compare(String var1, String var2) {
    return var1.compareTo(var2);
  }
}

```



## 反编译结果
javap -c -v src/main/java/Anony
```
警告: 二进制文件src/main/java/Anony包含Anony
Classfile /Users/victor/code/vicProjects/demo/project-old/src/main/java/Anony.class
  Last modified Aug 2, 2018; size 1972 bytes
  MD5 checksum 7855f9ea4db8c67faba60d2cd1d0b5d6
  Compiled from "Anony.java"
public class Anony
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
    #1 = Methodref          #23.#36       // java/lang/Object."<init>":()V
    #2 = Methodref          #37.#38       // java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
    #3 = Class              #39           // Anony$1
    #4 = Methodref          #3.#40        // Anony$1."<init>":(LAnony;Ljava/lang/Integer;Ljava/lang/Integer;Ljava/lang/Integer;)V
    #5 = Class              #41           // java/lang/Thread
    #6 = Class              #42           // Anony$2
    #7 = Methodref          #6.#43        // Anony$2."<init>":(LAnony;Ljava/lang/Integer;)V
    #8 = Methodref          #5.#44        // java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
    #9 = InvokeDynamic      #0:#49        // #0:run:(Ljava/lang/Integer;)Ljava/lang/Runnable;
   #10 = Class              #50           // java/lang/String
   #11 = String             #51           // A
   #12 = String             #52           // B
   #13 = String             #53           // C
   #14 = Methodref          #54.#55       // java/util/Arrays.asList:([Ljava/lang/Object;)Ljava/util/List;
   #15 = Class              #56           // Anony$3
   #16 = Methodref          #15.#57       // Anony$3."<init>":(LAnony;)V
   #17 = Methodref          #58.#59       // java/util/Collections.sort:(Ljava/util/List;Ljava/util/Comparator;)V
   #18 = InvokeDynamic      #1:#63        // #1:compare:()Ljava/util/Comparator;
   #19 = Methodref          #10.#64       // java/lang/String.compareTo:(Ljava/lang/String;)I
   #20 = Fieldref           #65.#66       // java/lang/System.out:Ljava/io/PrintStream;
   #21 = Methodref          #67.#68       // java/io/PrintStream.println:(Ljava/lang/Object;)V
   #22 = Class              #69           // Anony
   #23 = Class              #70           // java/lang/Object
   #24 = Utf8               InnerClasses
   #25 = Utf8               <init>
   #26 = Utf8               ()V
   #27 = Utf8               Code
   #28 = Utf8               LineNumberTable
   #29 = Utf8               publicMethod
   #30 = Utf8               lambda$publicMethod$1
   #31 = Utf8               (Ljava/lang/String;Ljava/lang/String;)I
   #32 = Utf8               lambda$publicMethod$0
   #33 = Utf8               (Ljava/lang/Integer;)V
   #34 = Utf8               SourceFile
   #35 = Utf8               Anony.java
   #36 = NameAndType        #25:#26       // "<init>":()V
   #37 = Class              #71           // java/lang/Integer
   #38 = NameAndType        #72:#73       // valueOf:(I)Ljava/lang/Integer;
   #39 = Utf8               Anony$1
   #40 = NameAndType        #25:#74       // "<init>":(LAnony;Ljava/lang/Integer;Ljava/lang/Integer;Ljava/lang/Integer;)V
   #41 = Utf8               java/lang/Thread
   #42 = Utf8               Anony$2
   #43 = NameAndType        #25:#75       // "<init>":(LAnony;Ljava/lang/Integer;)V
   #44 = NameAndType        #25:#76       // "<init>":(Ljava/lang/Runnable;)V
   #45 = Utf8               BootstrapMethods
   #46 = MethodHandle       #6:#77        // invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
   #47 = MethodType         #26           //  ()V
   #48 = MethodHandle       #6:#78        // invokestatic Anony.lambda$publicMethod$0:(Ljava/lang/Integer;)V
   #49 = NameAndType        #79:#80       // run:(Ljava/lang/Integer;)Ljava/lang/Runnable;
   #50 = Utf8               java/lang/String
   #51 = Utf8               A
   #52 = Utf8               B
   #53 = Utf8               C
   #54 = Class              #81           // java/util/Arrays
   #55 = NameAndType        #82:#83       // asList:([Ljava/lang/Object;)Ljava/util/List;
   #56 = Utf8               Anony$3
   #57 = NameAndType        #25:#84       // "<init>":(LAnony;)V
   #58 = Class              #85           // java/util/Collections
   #59 = NameAndType        #86:#87       // sort:(Ljava/util/List;Ljava/util/Comparator;)V
   #60 = MethodType         #88           //  (Ljava/lang/Object;Ljava/lang/Object;)I
   #61 = MethodHandle       #6:#89        // invokestatic Anony.lambda$publicMethod$1:(Ljava/lang/String;Ljava/lang/String;)I
   #62 = MethodType         #31           //  (Ljava/lang/String;Ljava/lang/String;)I
   #63 = NameAndType        #90:#91       // compare:()Ljava/util/Comparator;
   #64 = NameAndType        #92:#93       // compareTo:(Ljava/lang/String;)I
   #65 = Class              #94           // java/lang/System
   #66 = NameAndType        #95:#96       // out:Ljava/io/PrintStream;
   #67 = Class              #97           // java/io/PrintStream
   #68 = NameAndType        #98:#99       // println:(Ljava/lang/Object;)V
   #69 = Utf8               Anony
   #70 = Utf8               java/lang/Object
   #71 = Utf8               java/lang/Integer
   #72 = Utf8               valueOf
   #73 = Utf8               (I)Ljava/lang/Integer;
   #74 = Utf8               (LAnony;Ljava/lang/Integer;Ljava/lang/Integer;Ljava/lang/Integer;)V
   #75 = Utf8               (LAnony;Ljava/lang/Integer;)V
   #76 = Utf8               (Ljava/lang/Runnable;)V
   #77 = Methodref          #100.#101     // java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
   #78 = Methodref          #22.#102      // Anony.lambda$publicMethod$0:(Ljava/lang/Integer;)V
   #79 = Utf8               run
   #80 = Utf8               (Ljava/lang/Integer;)Ljava/lang/Runnable;
   #81 = Utf8               java/util/Arrays
   #82 = Utf8               asList
   #83 = Utf8               ([Ljava/lang/Object;)Ljava/util/List;
   #84 = Utf8               (LAnony;)V
   #85 = Utf8               java/util/Collections
   #86 = Utf8               sort
   #87 = Utf8               (Ljava/util/List;Ljava/util/Comparator;)V
   #88 = Utf8               (Ljava/lang/Object;Ljava/lang/Object;)I
   #89 = Methodref          #22.#103      // Anony.lambda$publicMethod$1:(Ljava/lang/String;Ljava/lang/String;)I
   #90 = Utf8               compare
   #91 = Utf8               ()Ljava/util/Comparator;
   #92 = Utf8               compareTo
   #93 = Utf8               (Ljava/lang/String;)I
   #94 = Utf8               java/lang/System
   #95 = Utf8               out
   #96 = Utf8               Ljava/io/PrintStream;
   #97 = Utf8               java/io/PrintStream
   #98 = Utf8               println
   #99 = Utf8               (Ljava/lang/Object;)V
  #100 = Class              #104          // java/lang/invoke/LambdaMetafactory
  #101 = NameAndType        #105:#108     // metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
  #102 = NameAndType        #32:#33       // lambda$publicMethod$0:(Ljava/lang/Integer;)V
  #103 = NameAndType        #30:#31       // lambda$publicMethod$1:(Ljava/lang/String;Ljava/lang/String;)I
  #104 = Utf8               java/lang/invoke/LambdaMetafactory
  #105 = Utf8               metafactory
  #106 = Class              #110          // java/lang/invoke/MethodHandles$Lookup
  #107 = Utf8               Lookup
  #108 = Utf8               (Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
  #109 = Class              #111          // java/lang/invoke/MethodHandles
  #110 = Utf8               java/lang/invoke/MethodHandles$Lookup
  #111 = Utf8               java/lang/invoke/MethodHandles
{
  public Anony();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 8: 0

  public void publicMethod();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=6, locals=9, args_size=1
         0: bipush        10
         2: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
         5: astore_1
         6: bipush        10
         8: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        11: astore_2
        12: bipush        10
        14: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        17: astore_3
        18: new           #3                  // class Anony$1
        21: dup
        22: aload_0
        23: aload_1
        24: aload_2
        25: aload_3
        26: invokespecial #4                  // Method Anony$1."<init>":(LAnony;Ljava/lang/Integer;Ljava/lang/Integer;Ljava/lang/Integer;)V
        29: astore        4
        31: new           #5                  // class java/lang/Thread
        34: dup
        35: new           #6                  // class Anony$2
        38: dup
        39: aload_0
        40: aload_1
        41: invokespecial #7                  // Method Anony$2."<init>":(LAnony;Ljava/lang/Integer;)V
        44: invokespecial #8                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
        47: astore        5
        49: new           #5                  // class java/lang/Thread
        52: dup
        53: aload_1
        54: invokedynamic #9,  0              // InvokeDynamic #0:run:(Ljava/lang/Integer;)Ljava/lang/Runnable;
        59: invokespecial #8                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
        62: astore        6
        64: iconst_3
        65: anewarray     #10                 // class java/lang/String
        68: dup
        69: iconst_0
        70: ldc           #11                 // String A
        72: aastore
        73: dup
        74: iconst_1
        75: ldc           #12                 // String B
        77: aastore
        78: dup
        79: iconst_2
        80: ldc           #13                 // String C
        82: aastore
        83: invokestatic  #14                 // Method java/util/Arrays.asList:([Ljava/lang/Object;)Ljava/util/List;
        86: astore        7
        88: aload         7
        90: new           #15                 // class Anony$3
        93: dup
        94: aload_0
        95: invokespecial #16                 // Method Anony$3."<init>":(LAnony;)V
        98: invokestatic  #17                 // Method java/util/Collections.sort:(Ljava/util/List;Ljava/util/Comparator;)V
       101: iconst_3
       102: anewarray     #10                 // class java/lang/String
       105: dup
       106: iconst_0
       107: ldc           #11                 // String A
       109: aastore
       110: dup
       111: iconst_1
       112: ldc           #12                 // String B
       114: aastore
       115: dup
       116: iconst_2
       117: ldc           #13                 // String C
       119: aastore
       120: invokestatic  #14                 // Method java/util/Arrays.asList:([Ljava/lang/Object;)Ljava/util/List;
       123: astore        8
       125: aload         7
       127: invokedynamic #18,  0             // InvokeDynamic #1:compare:()Ljava/util/Comparator;
       132: invokestatic  #17                 // Method java/util/Collections.sort:(Ljava/util/List;Ljava/util/Comparator;)V
       135: return
      LineNumberTable:
        line 11: 0
        line 12: 6
        line 13: 12
        line 16: 18
        line 24: 31
        line 30: 49
        line 34: 64
        line 35: 88
        line 41: 101
        line 42: 125
        line 45: 135
}
SourceFile: "Anony.java"
InnerClasses:
     #15; //class Anony$3
     #6; //class Anony$2
     #3; //class Anony$1
     public static final #107= #106 of #109; //Lookup=class java/lang/invoke/MethodHandles$Lookup of class java/lang/invoke/MethodHandles
BootstrapMethods:
  0: #46 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #47 ()V
      #48 invokestatic Anony.lambda$publicMethod$0:(Ljava/lang/Integer;)V
      #47 ()V
  1: #46 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #60 (Ljava/lang/Object;Ljava/lang/Object;)I
      #61 invokestatic Anony.lambda$publicMethod$1:(Ljava/lang/String;Ljava/lang/String;)I
      #62 (Ljava/lang/String;Ljava/lang/String;)I
```

其中:
- 匿名内部类作为InnerClasses
- lambda表达式作为BootstrapMethods



https://blog.csdn.net/u013096088/article/details/70475981

