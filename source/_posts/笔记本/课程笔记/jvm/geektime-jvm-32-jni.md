---
title: JNI的运行机制
abbrlink: 440930844
date: 2018-10-11 22:09:22
tags:
categories:
---
我们往往会牺牲可移植性，在 Java 代码中调用 C/C++ 代码（下面简述为 C 代码），并在其中实现所需功能。这种跨语言的调用，便需要借助 Java 虚拟机的 Java Native Interface（JNI）机制。

关于 JNI 的例子，你应该特别熟悉 Java 中标记为native的、没有方法体的方法（下面统称为 native 方法）。当在 Java 代码中调用这些 native 方法时，Java 虚拟机将通过 JNI，调用至对应的 C 函数（下面将 native 方法对应的 C 实现统称为 C 函数）中。

举个例子，Object.hashCode方法便是一个 native 方法。它对应的 C 函数将计算对象的哈希值，并缓存在对象头、栈上锁记录（轻型锁）或对象监视锁（重型锁所使用的 monitor）中，以确保该值在对象的生命周期之内不会变更。

# native 方法的链接
在调用 native 方法前，Java 虚拟机需要将该 native 方法链接至对应的 C 函数上。
链接方式主要有两种。
## 第一种是让 Java 虚拟机自动查找符合默认命名规范的 C 函数，并且链接起来。
事实上，我们并不需要记住所谓的命名规范，而是采用javac -h命令，便可以根据 Java 程序中的 native 方法声明，自动生成包含符合命名规范的 C 函数的头文件。

## 第二种链接方式则是在 C 代码中主动链接。
这种链接方式对 C 函数名没有要求。通常我们会使用一个名为registerNatives的 native 方法，并按照第一种链接方式定义所能自动链接的 C 函数。在该 C 函数中，我们将手动链接该类的其他 native 方法。

# JNI 的 API
在 C 代码中，我们也可以使用 Java 的语言特性，如 instanceof 测试等。这些功能都是通过特殊的 JNI 函数（JNI Functions）来实现的。
Java 虚拟机会将所有 JNI 函数的函数指针聚合到一个名为JNIEnv的数据结构之中。
这是一个线程私有的数据结构。Java 虚拟机会为每个线程创建一个JNIEnv，并规定 C 代码不能将当前线程的JNIEnv共享给其他线程，否则 JNI 函数的正确性将无法保证。
这么设计的原因主要有两个。一是给 JNI 函数提供一个单独命名空间。二是允许 Java 虚拟机通过更改函数指针替换 JNI 函数的具体实现，例如从附带参数类型检测的慢速版本，切换至不做参数类型检测的快速版本。
在 HotSpot 虚拟机中，JNIEnv被内嵌至 Java 线程的数据结构之中。部分虚拟机代码甚至会从JNIEnv的地址倒推出 Java 线程的地址。因此，如果在其他线程中使用当前线程的JNIEnv，会使这部分代码错误识别当前线程。

JNI 会将 Java 层面的基本类型以及引用类型映射为另一套可供 C 代码使用的数据结构。

# 局部引用与全局引用
在 C 代码中，我们可以访问所传入的引用类型参数，也可以通过 JNI 函数创建新的 Java 对象。
这些 Java 对象显然也会受到垃圾回收器的影响。因此，Java 虚拟机需要一种机制，来告知垃圾回收算法，不要回收这些 C 代码中可能引用到的 Java 对象。
这种机制便是 JNI 的局部引用（Local Reference）和全局引用（Global Reference）。垃圾回收算法会将被这两种引用指向的对象标记为不可回收。
事实上，无论是传入的引用类型参数，还是通过 JNI 函数（除NewGlobalRef及NewWeakGlobalRef之外）返回的引用类型对象，都属于局部引用。
不过，一旦从 C 函数中返回至 Java 方法之中，那么局部引用将失效。也就是说，垃圾回收器在标记垃圾时不再考虑这些局部引用。
这就意味着，我们不能缓存局部引用，以供另一 C 线程或下一次 native 方法调用时使用。


对于这种应用场景，我们需要借助 JNI 函数NewGlobalRef，将该局部引用转换为全局引用，以确保其指向的 Java 对象不会被垃圾回收。
相应的，我们还可以通过 JNI 函数DeleteGlobalRef来消除全局引用，以便回收被全局引用指向的 Java 对象。
此外，当 C 函数运行时间极其长时，我们也应该考虑通过 JNI 函数DeleteLocalRef，消除不再使用的局部引用，以便回收被引用的 Java 对象。
另一方面，由于垃圾回收器可能会移动对象在内存中的位置，因此 Java 虚拟机需要另一种机制，来保证局部引用或者全局引用将正确地指向移动过后的对象。
HotSpot 虚拟机是通过句柄（handle）来完成上述需求的。这里句柄指的是内存中 Java 对象的指针的指针。当发生垃圾回收时，如果 Java 对象被移动了，那么句柄指向的指针值也将发生变动，但句柄本身保持不变。
实际上，无论是局部引用还是全局引用，都是句柄。其中，局部引用所对应的句柄有两种存储方式，一是在本地方法栈帧中，主要用于存放 C 函数所接收的来自 Java 层面的引用类型参数；另一种则是线程私有的句柄块，主要用于存放 C 函数运行过程中创建的局部引用。
当从 C 函数返回至 Java 方法时，本地方法栈帧中的句柄将会被自动清除。而线程私有句柄块则需要由 Java 虚拟机显式清理。
进入 C 函数时对引用类型参数的句柄化，和调整参数位置（C 调用和 Java 调用传参的方式不一样），以及从 C 函数返回时清理线程私有句柄块，共同造就了 JNI 调用的额外性能开销（具体可参考该 stackoverflow 上的回答）。

# 调用jni方法时,内部执行情况
- Create a stack frame.
- Move arguments to proper register or stack locations according to ABI.
- Wrap object references to JNI handles.
- Obtain JNIEnv* and jclass for static methods and pass them as additional arguments.
- Check if should call method_entry trace function.
- Lock an object monitor if the method is synchronized.
- Check if the native function is linked already. Function lookup and linking is performed lazily.
- Switch thread from in_java to in_native state.
- Call the native function
- Check if safepoint is needed.
- Return thread to in_java state.
- Unlock monitor if locked.
- Notify method_exit.
- Unwrap object result and reset JNI handles block.
- Handle JNI exceptions.
- Remove the stack frame.
https://stackoverflow.com/questions/24746776/what-does-a-jvm-have-to-do-when-calling-a-native-method/24747484#24747484


# 总结与实践
今天我介绍了 JNI 的运行机制。
Java 中的 native 方法的链接方式主要有两种。一是按照 JNI 的默认规范命名所要链接的 C 函数，并依赖于 Java 虚拟机自动链接。另一种则是在 C 代码中主动链接。
JNI 提供了一系列 API 来允许 C 代码使用 Java 语言特性。这些 API 不仅使用了特殊的数据结构来表示 Java 类，还拥有特殊的异常处理模式。
JNI 中的引用可分为局部引用和全局引用。这两者都可以阻止垃圾回收器回收被引用的 Java 对象。不同的是，局部引用在 native 方法调用返回之后便会失效。传入参数以及大部分 JNI API 函数的返回值都属于局部引用。