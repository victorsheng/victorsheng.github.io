title: java-finalize
date: 2018-06-22 17:49:39
tags:
categories:
---
# 内部过程
如果类重写了finalize方法，且方法体不为空，则调用register_finalizer函数，继续看register_finalizer函数的实现
其中Universe::finalizer_register_method()缓存的是jdk中java.lang.ref.Finalizer类的register方法
在jvm中通过JavaCalls::call触发register方法，将新建的对象O封装成一个Finalizer对象，并通过add方法添加到Finalizer链表头。

- 对象O和Finalizer类的静态变量unfinalized有联系，在发生GC时，会被判定为活跃对象，因此不会被回收

## FinalizerThread线程
- 在Finalizer类的静态代码块中会创建一个FinalizerThread类型的守护线程，但是这个线程的优先级比较低，意味着在cpu吃紧的时候可能会抢占不到资源执行。
- FinalizerThread线程负责从ReferenceQueue队列中获取Finalizer对象，如果队列中没有元素，则通过wait方法将该线程挂起，等待被唤醒
- 如果返回了Finalizer对象，执行对象的runFinalizer()方法，其实可以发现：在runFinalizer()方法中主动捕获了异常，即使在执行finalize方法抛出异常时，也没有关系。

## ReferenceHandler线程
既然FinalizerThread线程是从ReferenceQueue队列中获取Finalizer对象，那么Finalizer对象是在什么情况下才会被插入到ReferenceQueue队列中？

- Finalizer的祖父类Reference中定义了ReferenceHandler线程
- 当pending被设置时，会调用ReferenceQueue的enqueue方法把Finalizer对象插入到ReferenceQueue队列中，接着通过notifyAll方法唤醒FinalizerThread线程执行后续逻辑
- 在GC过程的引用处理阶段，通过oopDesc::atomic_exchange_oop方法把发现的引用列表设置在pending字段所在的地址

## 重写Finalize方法,可能导致的内存泄漏(不能及时的回收掉,依赖于FinalizerThread线程的处理速度)


平常使用的Socket通信，SocksSocketImpl的父类重写了finalize方法
这么做主要是为了确保在用户忘记手动关闭socket连接的情况下，在该对象被回收时能够自动关闭socket来释放一些资源，但是在开发过程中，真的忘记手动调用了close方法，那么这些socket对象可能会因为FinalizeThread线程迟迟没有执行到这些对象的finalize方法，而导致一直占用某些资源，造成内存泄露。


## 没有执行GC也就是无法执行到finalize方法了。



# 对象销毁过程状态图

![upload successful](/images/pasted-184.png)

# 参考
https://www.jianshu.com/p/9d2788fffd5f