title: JDK- MashMap 学习
tags:
  - 数据结构
  - java
  - HashMap
categories:
  - java
date: 2018-01-05 11:14:00
---
# hashmap数据结构

![upload successful](/images/pasted-11.png)
```
//链表
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
    }

//数组
transient Node<K,V>[] table;
```
- HashMap是一个用于存储Key-Value键值对的集合，每一个键值对也叫做Entry。这些个键值对（Entry）分散存储在一个数组当中，这个数组就是HashMap的主干。
- HashMap 使用后台数组（backing array）作为桶，并使用链表（linked list）存储键／值对。
- 通过hash的方法，通过put和get存储和获取对象。存储对象时，我们将K/V传给put方法时，它调用hashCode计算hash从而得到bucket位置，进一步存储，HashMap会根据当前bucket的占用情况自动调整容量(超过Load Factor则resize为原来的2倍)。
- 获取对象时，我们将K传给get，它调用hashCode计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。
- 如果发生碰撞的时候，Hashmap通过链表将产生碰撞冲突的元素组织起来，在Java 8中，如果一个bucket中碰撞冲突的元素超过某个限制(默认是8)，则使用红黑树来替换链表，从而提高速度。 
- 基于Map接口实现、允许null键/值、非同步、不保证有序(比如插入的顺序)、也不保证序不随时间变化。HashMap存储着Entry(hash, key, value, next)对象。 
- 当key==null时，存在table[0]即第一个桶中，hash值为0。HashMap对key==null的键值对会做单独处理
- Capacity的默认值为16： 
	+ static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
- 负载因子的默认值为0.75： 
	+ static final float DEFAULT_LOAD_FACTOR = 0.75f; 
- 简单的说，Capacity就是bucket的大小，Load factor就是bucket填满程度的最大比例。如果对迭代性能要求很高的话不要把Capacity设置过大，也不要把load factor设置过小。当bucket中的entries的数目大于capacity*load factor时就需要调整bucket的大小为当前的2倍。 
- 可以设置初始容量Capacity，但是在HashMap处理过程中，是会把Capacity扩充成2的倍数
- HashMap中有一个成员变量modCount，这个用来实现“fast-fail”机制（也就是快速失败）。所谓快速失败就是在并发集合中，其进行迭代操作时，若有其他线程对其结构性的修改，这是迭代器会立马感知到，并且立刻抛出ConcurrentModificationException异常，而不是等待迭代完成之后才告诉你已经出错。

# get方法
- 取key的hashCode值
- 高位运算
- 取模运算
## jdk1.7的hash方法
```
final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
```
## jdk1.8的hash方法
```
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
- 混合原始哈希码的高位和低位，以此来加大低位的随机性
- 而且混合后的低位掺杂了高位的部分特征，这样高位的信息也被变相保留下来

![upload successful](/images/pasted-10.png)
图片出处:https://www.zhihu.com/question/20733617


## indexFor方法(根据上一步hash结果,计算数组的下表)

```
static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        return h & (length-1);
    }
```

##  getNode/getEntry--在链表中确定元素
```
    final Entry<K,V> getEntry(Object key) {
        if (size == 0) {
            return null;
        }

        int hash = (key == null) ? 0 : hash(key);
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }
```





# hashmap的初始长度
- 16,长度必须是2的幂
- 之所以16是为了服务于哈希函数
- index =  HashCode（Key） &  （Length - 1） 
- 与运算，101110001110101110 1001 & 1111 = 1001，十进制是9，所以 index=9。
- Hash算法最终得到的index结果，完全取决于Key的Hashcode值的最后几位
- 效果上等同于取模,而且大大提高了性能

# put方法
- 先插找
	+ 如果已存在,则替换
	+ 如果不存在,插入
    	* 检查是否需要rehash

## rehash
用一个新的数组代替已有的容量小的数组，就像我们用一个小桶装水，如果想装更多的水，就得换大水桶。

- 默认当Entry数量达到桶数量的75%时，哈希冲突已比较严重，就会成倍扩容桶数组，并重新分配所有原来的Entry。
- hashmap在扩容时候的步骤之一
- 衡量HashMap是否进行Resize的条件如下：
- HashMap.Size   >=  Capacity * LoadFactor
- 具体两个步骤
    + 1.扩容:创建一个新的Entry空数组，长度是原数组的2倍。
    + 2.ReHash:遍历原Entry数组，把所有的Entry重新Hash到新数组
    
### 1.7源码
```
    void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
    
    
   void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
```
newTable[i]的引用赋给了e.next，也就是使用了单链表的头插入方式，同一位置上新元素总会被放在链表的头部位置

![upload successful](/images/pasted-37.png)


在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”，可以看看下图为16扩充为32的resize示意图：
![upload successful](/images/pasted-38.png)

### 其他博客中resize的错误表述:
```
对于原bullet中的链表中的数据在扩容之后肯定还在一个链表中，因为hash值是一样的
```
此描述是错误的,在一个链表中,不一定代表hash是一样的,只是代表hash&(length-1)计算后的结果是一样的


## 并发问题
- ReHash在并发的情况下可能会形成链表环。
- 让下一次循环出现死循环

# ConcurrentHashMap
- 避免hashmap的方法有:hashtable,Collections.sysnchronizedMap
- 但是上面的都有性能问题,导致阻塞
- Sement
- <img src="http://pic.victor123.cn/18-1-4/28447320.jpg">
- 可以说，ConcurrentHashMap是一个二级哈希表。在一个总的哈希表下面，有若干个子哈希表。这样的二级结构，和数据库的水平拆分有些相似。
- 不同Segment的写入是可以并发执行的。
- 同一Segment的一写一读
- 同一Segment的并发写入
```
Get方法：

1.为输入的Key做Hash运算，得到hash值。

2.通过hash值，定位到对应的Segment对象

3.再次通过hash值，定位到Segment当中数组的具体位置。


Put方法：

1.为输入的Key做Hash运算，得到hash值。

2.通过hash值，定位到对应的Segment对象

3.获取可重入锁

4.再次通过hash值，定位到Segment当中数组的具体位置。

5.插入或覆盖HashEntry对象。

6.释放锁。
```

- 如果在统计Segment元素数量的过程中，已统计过的Segment瞬间插入新的元素，这时候该怎么办呢？
```
ConcurrentHashMap的Size方法是一个嵌套循环，大体逻辑如下：

1.遍历所有的Segment。

2.把Segment的元素数量累加起来。

3.把Segment的修改次数累加起来。

4.判断所有Segment的总修改次数是否大于上一次的总修改次数。如果大于，说明统计过程中有修改，重新统计，尝试次数+1；如果不是。说明没有修改，统计结束。

5.如果尝试次数超过阈值，则对每一个Segment加锁，再重新统计。

6.再次判断所有Segment的总修改次数是否大于上一次的总修改次数。由于已经加锁，次数一定和上次相等。

7.释放锁，统计结束。
```
为了尽量不锁住所有Segment，首先乐观地假设Size过程中不会有修改。当尝试一定次数，才无奈转为悲观锁，锁住所有Segment保证强一致性。


# jdk 1.8hashmap
- HashMap采用的是数组+链表+红黑树的形式。
- 什么时候链表转化为红黑树？当数组大小已经超过64并且链表中的元素个数超过默认设定（8个）时，将链表转化为红黑树


# jdk 1.8ConcurrentHashMap


# 与hashTable的区别
- 继承的父类不同:Hashtable继承自Dictionary类，而HashMap继承自AbstractMap类，
- 线程安全性不同:Hashtable 中的方法是Synchronize的
- 是否提供contains方法:HashMap把Hashtable的contains方法去掉了
- key和value是否允许null值:Hashtable中，key和value都不允许出现null值。HashMap中没有该键，也可能使该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断。
- 两个遍历方式的内部实现上不同:Hashtable还使用了Enumeration的方式 。
- hash值不同:哈希值的使用不同，HashTable直接使用对象的hashCode。而HashMap重新计算hash值。
- 内部实现使用的数组初始化和扩容方式不同:Hashtable和HashMap它们两个内部实现方式的数组的初始大小和扩容的方式。HashTable中hash数组默认大小是11，增加的方式是 old*2+1。HashMap中hash数组的默认大小是16，而且一定是2的指数。

# 数据结构
要知道hashmap是什么，首先要搞清楚它的数据结构，在java编程语言中，最基本的结构就是两种，一个是数组，另外一个是模拟指针（引用），所有的数据结构都可以用这两个基本结构来构造的，hashmap也不例外。Hashmap实际上是一个数组和链表的结合体（在数据结构中，一般称之为“链表散列“）

# 哈希冲突

## 概念
- hashcode通过hash算法得到有限的地址区间
- 哈希冲突：由于哈希算法被计算的数据是无限的，而计算后的结果范围有限，因此总会存在不同的数据经过计算后得到的值相同，这就是哈希冲突。

## 解决哈希冲突的方法
- 开放定址法
	+ 开放定址法需要的表长度要大于等于所需要存放的元素。
- 线行探查法
	+ 它从发生冲突的单元起，依次判断下一个单元是否为空，当达到最后一个单元时，再从表首依次判断。直到碰到空闲的单元或者探查完全部单元为止。
- 平方探查法
- 双散列函数探查法
- 链地址法（拉链法）
	+ 注：在java中，链接地址法也是HashMap解决哈希冲突的方法之一，jdk1.7完全采用单链表来存储同义词，jdk1.8则采用了一种混合模式，对于链表长度大于8的，会转换为红黑树存储。
- 再哈希法
	+ 就是同时构造多个不同的哈希函数,发生冲突时，再用H2 = RH2(key) 进行计算，直到冲突不再产生，这种方法不易产生聚集，但是增加了计算时间。
- 建立公共溢出区
	+ 将哈希表分为公共表和溢出表，当溢出发生时，将所有溢出数据统一放到溢出区。
    
# 1.8源码
hash算法
```
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

```

![upload successful](/images/pasted-35.png)
混合高位和地位,以此来加大地位的随机性

- 数组的位置:    (n - 1) & hash
- 确定数组后,链表中确定相同的方法: 
	+ e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))
    + always check first node
    + 在一个数组中,不代表hash是一样的,so必须判断
    
- 数组的长度:因为16是2的整数次幂的原因,n-1 二进制中都是x个1,这样,更能减少key之间的碰撞


# 位运算符(知识补充)
左移( << )、右移( >> ) 、无符号右移( >>> )
位与、位或、位异或、位非

```
5<<2
0000 0000 0000 0000 0000 0000 0000 0101           然后左移2位后，低位补0：
0000 0000 0000 0000 0000 0000 0001 0100           换算成10进制为20

5>>2
还是先将5转为2进制表示形式：
0000 0000 0000 0000 0000 0000 0000 0101 然后右移2位，高位补0：
0000 0000 0000 0000 0000 0000 0000 0001

-5>>>3
-5换算成二进制： 1111 1111 1111 1111 1111 1111 1111 1011
-5无符号右移3位后的结果 536870911 换算成二进制： 
0001 1111 1111 1111 1111 1111 1111 1111   // (用0进行补位)
------------
位与：第一个操作数的的第n位于第二个操作数的第n位如果都是1，那么结果的第n为也为1，否则为0
5 & 3
5转换为二进制 ：0000 0000 0000 0000 0000 0000 0000 0101
3转换为二进制 ：0000 0000 0000 0000 0000 0000 0000 0011
1转换为二进制	：0000 0000 0000 0000 0000 0000 0000 0001

位异或：第一个操作数的的第n位于第二个操作数的第n位 相反，那么结果的第n为也为1，否则为0
5 ^ 3
5转换为二进制：0000 0000 0000 0000 0000 0000 0000 0101
3转换为二进制：0000 0000 0000 0000 0000 0000 0000 0011
6转换为二进制：0000 0000 0000 0000 0000 0000 0000 0110

```



# 参考
http://blog.csdn.net/u013256816/article/details/50912762
https://tech.meituan.com/java-hashmap.html
https://bestswifter.com/hashtable/


# 更新日志
- 2018.01.22:终于理解这段的代码:e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))