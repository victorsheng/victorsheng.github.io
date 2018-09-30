---
title: java-unsafe
abbrlink: 23483
date: 2018-09-27 21:25:47
tags:
categories:
---
# 1. 获取 Unsafe 实例
```
public class Main {
    public static void main(String[] args) {
        Unsafe.getUnsafe();
    }
}
```
报错
```
Exception in thread "main" java.lang.SecurityException: Unsafe
  at sun.misc.Unsafe.getUnsafe(Unsafe.java:90)
  at com.bendcap.java.jvm.unsafe.Main.main(Main.java:13)

```

这是因为 Unsafe 类主要是 JDK 内部使用，并不提供给普通用户调用，也就是其名字所暗示的那样，这些操作不安全。

但是我们仍让可以通过反射获取到实例：
```
public static Unsafe createUnsafe() {
        try {
            Class<?> unsafeClass = Class.forName("sun.misc.Unsafe");
            Field field = unsafeClass.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            Unsafe unsafe = (Unsafe) field.get(null);
            return unsafe;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }


```

# 2. 不调用构造方法方法创建对象

那么我们可以通过 Unsafe 在内存中直接创建 Person 的实例:
```
public Person instancePerson() {
        Person person = null;
        try {
            person = (Person) createUnsafe().allocateInstance(Person.class);
        } catch (InstantiationException e) {
            e.printStackTrace();
        }
        person.setName("Unsafer");
        return person;
    }
```
Unsafe 只会分配 Person 对应的内存空间，而不触发构造函数。 

举一个生产实例。Gson 中反序列化的时候，ReflectiveTypeAdapterFactory 类负责通过反射设置字段值，其中在或许反序列化对应的 Class 实例的时候就用到了 Unsafe, 关键代码摘抄如下:

https://github.com/sharipol/GSON/blob/4a57ba6afddb75c2941100727d60e7e236d1e95a/gson/src/main/java/com/google/gson/internal/UnsafeAllocator.java

```
public abstract class UnsafeAllocator {
  public abstract <T> T newInstance(Class<T> c) throws Exception;

  public static UnsafeAllocator create() {
    // try JVM
    // public class Unsafe {
    //   public Object allocateInstance(Class<?> type);
    // }
    try {
      Class<?> unsafeClass = Class.forName("sun.misc.Unsafe");
      Field f = unsafeClass.getDeclaredField("theUnsafe");
      f.setAccessible(true);
      final Object unsafe = f.get(null);
      final Method allocateInstance = unsafeClass.getMethod("allocateInstance", Class.class);
      return new UnsafeAllocator() {
        @Override
        @SuppressWarnings("unchecked")
        public <T> T newInstance(Class<T> c) throws Exception {
          assertInstantiable(c);
          return (T) allocateInstance.invoke(unsafe, c);
        }
      };
    } catch (Exception ignored) {
    }
    ...
    // give up
    return new UnsafeAllocator() {
      @Override
      public <T> T newInstance(Class<T> c) {
        throw new UnsupportedOperationException("Cannot allocate " + c);
      }
    };
  }
  }

```

# 3.改变私有字段
假设我们有如下类：
```
public class SecretHolder {
    private int SECRET_VALUE = 0;

    public boolean secretValueDisclosed() {
        return SECRET_VALUE == 1;
    }
}
```

下面我们通过 Unsafe 来改变私有属性 SECRET_VALUE 的值。
```
 SecretHolder secretHolder = new SecretHolder();
        Field field = null;
        try {
            field = secretHolder.getClass().getDeclaredField("SECRET_VALUE");
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
        Unsafe unsafe = createUnsafe();
        unsafe.putInt(secretHolder, unsafe.objectFieldOffset(field), 1);
        return secretHolder.secretValueDisclosed();
```
我们通过 unsafe.putInt 直接改变了 SecretHolder 的私有属性的值。一旦我们通过反射获得了类的私有属性字段，我们就可以直接操作它的值。


# 4. 抛出异常而不会触发 CE(Checked Exception)

通过 unsafe.throwExceptio 创建的异常不会被编译器检查，方法的调用者也不需要处理异常。

```
 public void throwException() {
        Unsafe unsafe = createUnsafe();
        unsafe.throwException(new IOException());
    }
```

# 5. 堆外内存

Java 中对象分配一般是在 Heap 中进行的（例外是 TLAB等），当应用内存不足的时候，可以通过触发 GC 进行垃圾回收，但是如果有大量对象存活到永久代，并且仍然引用可达，那么我们就需要堆外内存（Off-Heap Memory）来缓解频繁 GC 造成的压力。
Unsafe.allocateMemory 给了我们在直接内存中分配对象的能力，这块内存是非堆内存，因此，不会受到 GC 的频繁分析和干扰。
虽然这样可以缓解大量对象占用内存对 GC 和 JVM 造成的压力，这也就需要我们手动管理内存，因此，在合适的事后我们需要手动调用 freeMemory 来释放内存。
举例，我们在内存中分配字节数组：

```
public class OffHeapArray {
    private final static int BYTE = 1;
    private long size;
    private long address;
    private Unsafe unsafe;

    public OffHeapArray(long size, Unsafe unsafe) {
        this.size = size;
        this.unsafe = unsafe;
        address = unsafe.allocateMemory(size * BYTE);
    }

    public void set(long i, byte value) {
        unsafe.putByte(address + i * BYTE, value);
    }

    public int get(long idx) {
        return unsafe.getByte(address + idx * BYTE);
    }

    public long size() {
        return size;
    }

    public void freeMemory() {
        unsafe.freeMemory(address);
    }
}

```

# 6. CAS（CompareAndSwap）
java.concurrent 包中提供了大量并发相关的操作，例如 AtomicInteger 就用了 Unsafe.compareAndSwap 操作来实现 lock-free 的操作，保证更好的性能。

```
public class CASCounter {
    private Unsafe unsafe;
    private volatile long counter = 0;
    private long offset;

    public CASCounter(Unsafe unsafe) {
        this.unsafe = unsafe;
        try {
            offset = unsafe.objectFieldOffset(CASCounter.class.getDeclaredField("counter"));
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
    }

    public void increment() {
        long before = counter;
        while (!unsafe.compareAndSwapLong(this, offset, before, before + 1)) {
            before = counter;
        }
    }


    public long getCounter() {
        return counter;
    }
}

```
注： counter 需要声明为 volatile 来保证对所有线程可见（参考 Java 内存模型以及指令集重排）
这里的关键是 increment 方法，我们在 while 循环里不断尝试调用 compareAndSwapLong，检查在我们方法内部累加的同事，counter 的值有没有被其他线程改变。如有没有，就提交更改，如果不一致，那么继续尝试提交更改。


# 7. Park/Unpark
Park/Unpark 主要是被 JVM 用来做线程的上下文切换。当线程需要等待某个条件的时候，JVM 会调用 park 来阻塞该线程。
这和 Object.await 非常类似，但是 park 是操作系统调用，因此，在某些操作系统架构上，这会带来更好的性能。
当线程阻塞后，需要再次执行， JVM 会调用 unpark 方法使得该线程变得活跃。

# 结论
俗话说，面试造核弹，工作拧螺丝，虽然 Unsafe 看起来不会被用到，但是能帮助我们更好的理解 JVM 以及 JDK 中 lock-free 的实现。还有一点就是 Off-Heap Memory, 如果做服务端开发中确实遇到了大内存对象并且常驻内存的情况，堆外分配不失为一个好的策略来减轻 GC 以及 GC 带来的系统负担（可参见 R 大在阿里 JVM 中所做的一些优化），与之对应的就是 TLAB(thread local allocation buffer)，后面有机会再整理。

# 小结2
获取到Unsafe实例之后，我们就可以为所欲为了。Unsafe类提供了以下这些功能：

一、内存管理。包括分配内存、释放内存等。

该部分包括了allocateMemory（分配内存）、reallocateMemory（重新分配内存）、copyMemory（拷贝内存）、freeMemory（释放内存 ）、getAddress（获取内存地址）、addressSize、pageSize、getInt（获取内存地址指向的整数）、getIntVolatile（获取内存地址指向的整数，并支持volatile语义）、putInt（将整数写入指定内存地址）、putIntVolatile（将整数写入指定内存地址，并支持volatile语义）、putOrderedInt（将整数写入指定内存地址、有序或者有延迟的方法）等方法。getXXX和putXXX包含了各种基本类型的操作。

利用copyMemory方法，我们可以实现一个通用的对象拷贝方法，无需再对每一个对象都实现clone方法，当然这通用的方法只能做到对象浅拷贝。

二、非常规的对象实例化。

allocateInstance()方法提供了另一种创建实例的途径。通常我们可以用new或者反射来实例化对象，使用allocateInstance()方法可以直接生成对象实例，且无需调用构造方法和其它初始化方法。

这在对象反序列化的时候会很有用，能够重建和设置final字段，而不需要调用构造方法。

三、操作类、对象、变量。

这部分包括了staticFieldOffset（静态域偏移）、defineClass（定义类）、defineAnonymousClass（定义匿名类）、ensureClassInitialized（确保类初始化）、objectFieldOffset（对象域偏移）等方法。

通过这些方法我们可以获取对象的指针，通过对指针进行偏移，我们不仅可以直接修改指针指向的数据（即使它们是私有的），甚至可以找到JVM已经认定为垃圾、可以进行回收的对象。

四、数组操作。

这部分包括了arrayBaseOffset（获取数组第一个元素的偏移地址）、arrayIndexScale（获取数组中元素的增量地址）等方法。arrayBaseOffset与arrayIndexScale配合起来使用，就可以定位数组中每个元素在内存中的位置。

由于Java的数组最大值为Integer.MAX_VALUE，使用Unsafe类的内存分配方法可以实现超大数组。实际上这样的数据就可以认为是C数组，因此需要注意在合适的时间释放内存。

五、多线程同步。包括锁机制、CAS操作等。

这部分包括了monitorEnter、tryMonitorEnter、monitorExit、compareAndSwapInt、compareAndSwap等方法。

其中monitorEnter、tryMonitorEnter、monitorExit已经被标记为deprecated，不建议使用。

Unsafe类的CAS操作可能是用的最多的，它为Java的锁机制提供了一种新的解决办法，比如AtomicInteger等类都是通过该方法来实现的。compareAndSwap方法是原子的，可以避免繁重的锁机制，提高代码效率。这是一种乐观锁，通常认为在大部分情况下不出现竞态条件，如果操作失败，会不断重试直到成功。

六、挂起与恢复。

这部分包括了park、unpark等方法。

将一个线程进行挂起是通过park方法实现的，调用 park后，线程将一直阻塞直到超时或者中断等条件出现。unpark可以终止一个挂起的线程，使其恢复正常。整个并发框架中对线程的挂起操作被封装在 LockSupport类中，LockSupport类中有各种版本pack方法，但最终都调用了Unsafe.park()方法。

七、内存屏障。

这部分包括了loadFence、storeFence、fullFence等方法。这是在Java 8新引入的，用于定义内存屏障，避免代码重排序。

loadFence() 表示该方法之前的所有load操作在内存屏障之前完成。同理storeFence()表示该方法之前的所有store操作在内存屏障之前完成。fullFence()表示该方法之前的所有load、store操作在内存屏障之前完成。

# 参考
https://juejin.im/post/5b5c726c6fb9a04fdb16c951
https://www.cnblogs.com/pkufork/p/java_unsafe.html