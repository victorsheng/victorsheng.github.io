title: study-java-thread-local
date: 2018-01-24 23:21:43
tags:
categories:
---
# SoftReference、Weak Reference和PhantomRefrence

## 强引用
默认
```
Object o=new Object();   
Object o1=o; 
o=null; 
o1=null;   
```
如果显式地设置o和o1为null，或超出范围，则gc认为该对象不存在引用，这时就可以收集它了。可以收集并不等于就一会被收集，什么时候收集这要取决于gc的算法，这要就带来很多不确定性。

## 其他
```
String abc=new String("abc");  //1   
SoftReference<String> abcSoftRef=new SoftReference<String>(abc);  //2   
WeakReference<String> abcWeakRef = new WeakReference<String>(abc); //3   
abc=null; //4   
abcSoftRef.clear();//5

```

上面的代码中：
    第一行在heap对中创建内容为“abc”的对象，并建立abc到该对象的强引用,该对象是强可及的。
    第二行和第三行分别建立对heap中对象的软引用和弱引用，此时heap中的对象仍是强可及的。
    第四行之后heap中对象不再是强可及的，变成软可及的。同样第五行执行之后变成弱可及的。
    
## 软引用
被 Soft Reference 指到的对象，即使没有任何 Direct Reference，也不会被清除。一直要到 JVM 内存不足且 没有 Direct Reference 时才会清除

SoftReference 是用来设计 object-cache 之用的。如此一来 SoftReference 不但可以把对象 cache 起来，也不会造成内存不足的错误 （OutOfMemoryError）。我觉得 Soft Reference 也适合拿来实作 pooling 的技巧。

-----
    如果一个对象只具有软引用，那就类似于可有可物的生活用品。如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。
    
   软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，JAVA虚拟机就会把这个软引用加入到与之关联的引用队列中。
   
软引用可以加速JVM对垃圾内存的回收速度 , 维护系统的运行安全 , 防止产生内存溢出的问题

## 弱引用
可以用来观察是否被gc掉

gc收集弱可及对象的执行过程和软可及一样，只是gc不会根据内存情况来决定是不是收集该对象。

如果你希望能随时取得某对象的信息，但又不想影响此对象的垃圾收集，那么你应该用 Weak Reference 来记住此对象，而不是用一般的 reference。

-------

如果一个对象只具有弱引用，那就类似于可有可物的生活用品。弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。<b>在垃圾回收器线程扫描它 所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。</b>不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。 
  
弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。
    


## 虚引用
 "虚引用"顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收。
    虚引用主要用来跟踪对象被垃圾回收的活动。虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列（ReferenceQueue）联合使用。当垃 圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是 否已经加入了虚引用，来了解

    被引用的对象是否将要被垃圾回收。程序如果发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

# ThreadLocal为什么使用WeakReference

![upload successful](/images/pasted-44.png)

在上文中我们发现了ThreadLocalMap的key是一个弱引用，那么为什么使用弱引用呢？使用强引用key与弱引用key的差别如下：

- 强引用key：ThreadLocal被设置为null，由于ThreadLocalMap持有ThreadLocal的强引用，如果不手动删除，那么ThreadLocal将不会回收，产生内存泄漏。
- 弱引用key：ThreadLocal被设置为null，由于ThreadLocalMap持有ThreadLocal的弱引用，即便不手动删除，ThreadLocal仍会被回收，ThreadLocalMap在之后调用set()、getEntry()和remove()函数时会清除所有key为null的Entry。

但要注意的是，ThreadLocalMap仅仅含有这些被动措施来补救内存泄漏问题。如果你在之后没有调用ThreadLocalMap的set()、getEntry()和remove()函数的话，那么仍然会存在内存泄漏问题。
在使用线程池的情况下，如果不及时进行清理，内存泄漏问题事小，甚至还会产生程序逻辑上的问题。所以，为了安全地使用ThreadLocal，必须要像每次使用完锁就解锁一样，在每次使用完ThreadLocal后都要调用remove()来清理无用的Entry。