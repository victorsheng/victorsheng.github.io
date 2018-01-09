title: java hashmap 学习
tags:
  - 数据结构
categories: []
date: 2018-01-05 11:14:00
---
# hashmap内部原理学习

![upload successful](/images/pasted-11.png)
- HashMap是一个用于存储Key-Value键值对的集合，每一个键值对也叫做Entry。这些个键值对（Entry）分散存储在一个数组当中，这个数组就是HashMap的主干。
- HashMap 使用后台数组（backing array）作为桶，并使用链表（linked list）存储键／值对。
- 通过hash的方法，通过put和get存储和获取对象。存储对象时，我们将K/V传给put方法时，它调用hashCode计算hash从而得到bucket位置，进一步存储，HashMap会根据当前bucket的占用情况自动调整容量(超过Load Factor则resize为原来的2倍)。获取对象时，我们将K传给get，它调用hashCode计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。如果发生碰撞的时候，Hashmap通过链表将产生碰撞冲突的元素组织起来，在Java 8中，如果一个bucket中碰撞冲突的元素超过某个限制(默认是8)，则使用红黑树来替换链表，从而提高速度。 
- 基于Map接口实现、允许null键/值、非同步、不保证有序(比如插入的顺序)、也不保证序不随时间变化。HashMap存储着Entry(hash, key, value, next)对象。 
- 当key==null时，存在table[0]即第一个桶中，hash值为0。HashMap对key==null的键值对会做单独处理
- Capacity的默认值为16： 
-   static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
- 负载因子的默认值为0.75： 
-   static final float DEFAULT_LOAD_FACTOR = 0.75f; 
- 简单的说，Capacity就是bucket的大小，Load factor就是bucket填满程度的最大比例。如果对迭代性能要求很高的话不要把Capacity设置过大，也不要把load factor设置过小。当bucket中的entries的数目大于capacity*load factor时就需要调整bucket的大小为当前的2倍。 
- 可以设置初始容量Capacity，但是在HashMap处理过程中，是会把Capacity扩充成2的倍数
- HashMap中有一个成员变量modCount，这个用来实现“fast-fail”机制（也就是快速失败）。所谓快速失败就是在并发集合中，其进行迭代操作时，若有其他线程对其结构性的修改，这是迭代器会立马感知到，并且立刻抛出ConcurrentModificationException异常，而不是等待迭代完成之后才告诉你已经出错。

## hash方法
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
## indexFor方法

```
static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        return h & (length-1);
    }
```
- 如图是jdk1.8中的h>>>16版本
- 混合原始哈希码的高位和低位，以此来加大低位的随机性
- 而且混合后的低位掺杂了高位的部分特征，这样高位的信息也被变相保留下来

![upload successful](/images/pasted-10.png)
https://www.zhihu.com/question/20733617


## put方法
- 先判断table(存放bullet的数组，初始类定义：transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;)是否为空，如果为空则扩充table，其中包括确保table的大小为2的整数倍。
- 如果key值为null，则特殊处理，调用putForNullKey(V value)，hash值为0，存入table中，返回。
- 如果key值不为null，则计算key的hash值；
- 然后计算key在table中的索引index;
- 遍历table[index]的链表，如果发现链表中有bullet中的键的hash值与key相等，并且调用equals()方法也返回true，则替换旧值（oldValue），保证key的唯一性；
- 如果没有，在插入之前先判断table中阈值的大小，如果table中的bullet个数size超过阈值（threshold）则扩容（resize）两倍；注意扩容的顺序，扩容之前old1->old2->old3，扩容之后old3->old2->old1，扩展之前和扩容之后的table的index不一定相同，但是对于原bullet中的链表中的数据在扩容之后肯定还在一个链表中，因为hash值是一样的。
- 插入新的bullet。注意插入的顺序，原先table[index]的链表比如 old1->old2->old3，插入新值之后为new1->old1->old2->old3.

```
public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
```
```
void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }
```
## 位运算符
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

## get方法
- 判断key值是否为null，如果是则特殊处理（在table[0]的链表中寻找）
- 否则计算hash值，进而获得table中的index值
- 在table[index]的链表中寻找，根据hash值和equals()方法获得相应的value。

```
public V get(Object key) {
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }
```
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

# rehash
- 默认当Entry数量达到桶数量的75%时，哈希冲突已比较严重，就会成倍扩容桶数组，并重新分配所有原来的Entry。
- hashmap在扩容时候的步骤之一
- 衡量HashMap是否进行Resize的条件如下：
- HashMap.Size   >=  Capacity * LoadFactor
- 具体两个步骤
    + 1.扩容:创建一个新的Entry空数组，长度是原数组的2倍。
    + 2.ReHash:遍历原Entry数组，把所有的Entry重新Hash到新数组

# 小结
- 声明HashMap时最好使用带initialCapacity的构造函数，传入数据的最大size，可以避免内部数组resize；
- 性能要求高的地方，可以将loadFactor设置的小于默认值0.75，使hash值更分散，产生hash碰撞的几率就更小，但需要的内存也更多,用空间换取时间；


# 并发问题
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


# 优秀博客推荐
http://blog.csdn.net/u013256816/article/details/50912762