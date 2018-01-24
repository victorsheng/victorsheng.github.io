title: study_equals_and_hashCode
date: 2018-01-22 12:36:24
tags:
categories:
---
https://www.ibm.com/developerworks/cn/java/j-jtp05273/index.html

# 为什么 Override equals()和hashCode()?
- 如果 Integer 不 Override equals() 和 hashCode() 情况又将如何?如果我们从未在 HashMap 或其它基于散列的集合中使用 Integer 作为关键字的话，什么也不会发生。但是，如果我们在 HashMap中 使用这类 Integer 对象作为关键字，我们将不能够可靠地检索相关的值，除非我们在 get() 调用中使用与 put() 调用中极其类似的 Integer 实例。这要求确保在我们的整个程序中，只能使用对应于特定整数值的 Integer 对象的一个实例。不用说，这种方法极不方便而且错误频频。

- Object 的interface contract要求如果根据 equals() 两个对象是相等的，那么它们必须有相同的 hashCode() 值。当其识别能力整个包含在 equals() 中时，为什么我们的根对象类需要 hashCode() ？ 
- 如果重写了equals方法，则一定要重写hashCode方法。
- hashCode() 方法纯粹用于提高效率。Java平台设计人员预计到了典型Java应用程序中基于散列的集合类（Collection Class)的重要性--如 Hashtable 、 HashMap 和 HashSet ，并且使用 equals() 与许多对象进行比较在计算方面非常昂贵。使所有Java对象都能够支持 hashCode() 并结合使用基于散列的集合，可以实现有效的存储和检索。


# hashCode方法
- 如果重写了equals方法，则一定要重写hashCode方法。

- 重写hashCode方法的原则如下：

	+ 在程序执行期间，只要equals方法的比较操作用到的信息没有被修改，那么对这同一个对象调用多次，hashCode方法必须始终如一地返回同一个整数
	+ 如果两个对象通过equals方法比较得到的结果是相等的，那么对这两个对象进行hashCode得到的值应该相同
	+ 两个不同的对象，hashCode的结果可能是相同的，这就是哈希表中的冲突。为了保证哈希表的效率，哈希算法应尽可能的避免冲突
- 关于相应的哈希算法，一个简单的算法如下:

	+ 永远不要让哈希算法返回一个常值，这时哈希表将退化成链表，查找时间复杂度也从 O(1)O(1) 退化到 O(N)O(N)
	+ 如果参数是boolean型，计算(f ? 1 : 0)
	+ 如果参数是byte, char, short或者int型，计算(int) f
	+ 如果参数是long型，计算(int) (f ^ (f >>> 32))
	+ 如果参数是float型，计算Float.floatToIntBits(f)
	+ 如果参数是double型，计算Double.doubleToLongBits(f)得到long类型的值，再根据公式计算出相应的hash值
	+ 如果参数是Object型，那么应计算其有用的成员变量的hash值，并按照下面的公式计算最终的hash值
	+ 如果参数是个数组，那么把数组中的每个值都当做单独的值，分别按照上面的方法单独计算hash值，最后按照下面的公式计算最终的hash值
	+ 组合公式：result = 31 * result + c
    
## String类的hashCode方法如下（JDK 1.8）：
```
public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;
            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

## 举个自定义类的hashCode例子：
```
class Duck {
    private int id;
    private String name;
    private double weight;
    private float height;
    private String note;
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Duck duck = (Duck) o;
        if (id != duck.id) return false;
        if (Double.compare(duck.weight, weight) != 0) return false;
        if (Float.compare(duck.height, height) != 0) return false;
        if (name != null ? !name.equals(duck.name) : duck.name != null) return false;
        return !(note != null ? !note.equals(duck.note) : duck.note != null);
    }
    @Override
    public int hashCode() {
        int result;
        long temp;
        result = id;
        result = 31 * result + (name != null ? name.hashCode() : 0);
        temp = Double.doubleToLongBits(weight);
        result = 31 * result + (int) (temp ^ (temp >>> 32));
        result = 31 * result + (height != +0.0f ? Float.floatToIntBits(height) : 0);
        result = 31 * result + (note != null ? note.hashCode() : 0);
        return result;
    }
}
```

# BTW 回顾一下 hashmap内部的实现
e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))