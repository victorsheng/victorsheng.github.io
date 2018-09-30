---
title: java_random
abbrlink: 4525
date: 2018-09-27 19:28:03
tags:
categories:
---
```
public Random() {
    // 默认构造方法传入的seed值, 
    // 这个值由种子算法得到的一个唯一值与纳秒值通过位运算得到
    // 尽可能做到了和另一个构造方法的seed值的不同
    this(seedUniquifier() ^ System.nanoTime());
}


//构造方法
public Random(long seed) {
    if (getClass() == Random.class)
        this.seed = new AtomicLong(initialScramble(seed));
    else {
        // subclass might have overriden setSeed
        this.seed = new AtomicLong();
        setSeed(seed);
    }
}


//方法名翻译:初始化攀登
private static long initialScramble(long seed) {
    return (seed ^ multiplier) & mask;
}


//种子唯一标示
private static long seedUniquifier() {
    // L'Ecuyer, "Tables of Linear Congruential Generators of
    // Different Sizes and Good Lattice Structure", 1999
    for (;;) {
        long current = seedUniquifier.get();
        long next = current * 181783497276652981L;
        if (seedUniquifier.compareAndSet(current, next))
            return next;
    }
}
//静态字段,全局的
private static final AtomicLong seedUniquifier = new AtomicLong(8682522807148012L);
```

# 获取随机数nextInt()
```
    protected int next(int bits) {
        long oldseed, nextseed;
        AtomicLong seed = this.seed;
        do {
            oldseed = seed.get();
            // 根据一定的规则生成nextseed
            nextseed = (oldseed * multiplier + addend) & mask;
            // 更新oldseed的值，通过cas保证线程安全
        } while (!seed.compareAndSet(oldseed, nextseed));
        return (int)(nextseed >>> (48 - bits));
    }

```

这里会根据 seed 当前的值，通过一定的规则((oldseed * multiplier + addend) & mask; 即伪随机算法)算出下一个 seed，然后进行 CAS，如果 CAS 失败则继续循环上面的操作。最后根据我们需要的 bit 位数来进行返回。核心便是 CAS 算法。


# 获取随机数 nextInt(int bound) 

```
    public int nextInt(int bound) {
        if (bound <= 0)
            throw new IllegalArgumentException(BadBound);

        int r = next(31);
        int m = bound - 1;
        if ((bound & m) == 0)  // i.e., bound is a power of 2
            r = (int)((bound * (long)r) >> 31);
        else {
            for (int u = r;
                 u - (r = u % bound) + m < 0;
                 u = next(31))
                ;
        }
        return r;
    }
```
这个流程比 nextInt() 多了几步，具体步骤如下:
1. 首先获取 31 位的随机数，注意这里是 31 位，和上面 32 位不同，因为在 nextInt() 方法中可以获取到随机数可能是负数，而 nextInt(int bound) 规定只能获取到 [0,bound) 之前的随机数，也就意味着必须是正数，预留一位符号位，所以只获取了31位。(不要想着使用取绝对值这样操作，会导致性能下降)
2. 然后进行取 bound 操作。
3. 如果 bound 是2的幂次方，可以直接将第一步获取的值乘以 bound 然后右移31位，解释一下:如果 bound 是4，那么乘以4其实就是左移2位，其实就是变成了33位，再右移31位的话，就又会变成2位，最后，2位 int 的范围其实就是 [0,4) 了。
4. 如果不是 2 的幂，通过模运算进行处理。

# netxInt()
```
    public int nextInt() {
        return next(32);
    }
```

# 并发瓶颈

一般而言，CAS 相比加锁有一定的优势，但并不一定意味着高效，并发很大的情况下回造成CPU飙高。一个立刻被想到的解决方案是每次使用 Random 时都去 new 一个新的线程私有化的 Random 对象，或者使用 ThreadLocal 来维护线程私有化对象，但除此之外还存在更高效的方案，下面便来介绍本文的主角 ThreadLocalRandom。

 **为什么会cpu飙高?**
 - 静态的(初始化一个新的random实例)
 - 实例的(多线程只是用一个random实例)

# ThreadLocalRandom
在 JDK1.7 之后提供了新的类 ThreadLocalRandom 用来在并发场景下代替 Random。使用方法比较简单:
```
public class RandomTest {
    public static void main(String[] args) {
        System.out.println(ThreadLocalRandom.current().nextInt());
        System.out.println(ThreadLocalRandom.current().nextInt());
        System.out.println(ThreadLocalRandom.current().nextInt(10));
    }
}
```
在 current 方法中有:
```
public static ThreadLocalRandom current() {
    if (UNSAFE.getInt(Thread.currentThread(), PROBE) == 0)
        // 判断当前线程是否初始化, 如果没有则初始化
        localInit();
    return instance;
}

static final void localInit() {
    int p = probeGenerator.addAndGet(PROBE_INCREMENT);
    int probe = (p == 0) ? 1 : p; // skip 0
    long seed = mix64(seeder.getAndAdd(SEEDER_INCREMENT));
    Thread t = Thread.currentThread();
    //使用它来控制随机数种子。
    UNSAFE.putLong(t, SEED, seed);
    //使用它来控制初始化。
    UNSAFE.putInt(t, PROBE, probe);
}
```

可以看见如果没有初始化会对其进行初始化，而这里我们的 seed 不再是一个全局变量，而是每个线程私有的，在我们的Thread中有三个变量: 
* threadLocalRandomSeed：ThreadLocalRandom 使用它来控制随机数种子。
* threadLocalRandomProbe：ThreadLocalRandom 使用它来控制初始化。
* threadLocalRandomSecondarySeed：二级种子。


## nextInt() 
```
public int nextInt() {
    return mix32(nextSeed());
}

final long nextSeed() {
    Thread t; long r; // read and update per-thread seed
    UNSAFE.putLong(t = Thread.currentThread(), SEED,
                   r = UNSAFE.getLong(t, SEED) + GAMMA);
    return r;
}
```

可以看见由于我们每个线程各自都维护了种子，这个时候并不需要 CAS，直接进行 put，在这里利用线程之间隔离，减少了并发冲突；相比较 ThreadLocal<Random>，ThreadLocalRandom 不仅仅减少了对象维护的成本，其内部实现也更轻量级。所以 ThreadLocalRandom 性能很高。

# 总结
- Random为了保证线程安全,使用了CAS
- ThreadLocalRandom为了保证线程安全和性能,放弃了CAS,使用了UNSAFE类

# 注意
确保不同线程获取不同的 seed，最简单的方式便是每次调用都是使用 current()：

# 参考
http://mp.weixin.qq.com/s?__biz=MzU5ODUwNzY1Nw==&mid=2247484002&idx=1&sn=a16bdc8346d51faef2eb47aa270d894c&chksm=fe426b84c935e292b390bb68db126affc545df3ad362db358712ad7c891ef46549b6178edfa7&mpshare=1&scene=1&srcid=0927hMyH24CvLUFauV49pzIc#rd