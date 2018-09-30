---
title: java-genercity
abbrlink: 269373254
date: 2018-04-26 18:28:01
tags:
categories:
---
# 泛型
Java 在 1.5 引入了泛型机制，泛型本质是参数化类型，也就是说变量的类型是一个参数，在使用时再指定为具体类型。泛型可以用于类、接口、方法，通过使用泛型可以使代码更简单、安全。然而 Java 中的泛型使用了类型擦除，所以只是伪泛型。这篇文章对泛型的使用以及存在的问题做个总结，主要参考自 《Java 编程思想》。

# 问题
```
public class Holder1 {
    private Object a;

    public Holder1(Object a) {
        this.a = a;
    }

    public void set(Object a) {
        this.a = a;
    }
    public Object get(){
        return a;
    }

    public static void main(String[] args) {
        Holder1 holder1 = new Holder1("not Generic");
        String s = (String) holder1.get();
        holder1.set(1);
        Integer x = (Integer) holder1.get();
    }
```



在 Holder1 中，有一个用 Object 引用的变量。因为任何类型都可以向上转型为 Object，所以这个 Holder 可以接受任何类型。在取出的时候 Holder 只知道它保存的是一个 Object 对象，所以要强制转换为对应的类型。在 main 方法中， holder1 先是保存了一个字符串，也就是 String 对象，接着又变为保存一个 Integer 对象(参数 1 会自动装箱)。从 Holder 中取出变量时强制转换已经比较麻烦，这里还要记住不同的类型，要是转错了就会出现运行时异常。


```
public class Holder2<T> {

    private T a;
    public Holder2(T a) {
        this.a = a;
    }

    public T get() {
        return a;
    }

    public void set(T a) {
        this.a = a;
    }

    public static void main(String[] args) {
        Holder2<String> holder2 = new Holder2<>("Generic");
        String s = holder2.get();

        holder2.set("test");
        holder2.set(1);//无法编译   参数 1 不是 String 类型

    }
```
在 Holder2 中， 变量 a 是一个参数化类型 T，T 只是一个标识，用其它字母也是可以的。创建 Holder2 对象的时候，在尖括号中传入了参数 T 的类型，那么在这个对象中，所有出现 T 的地方相当于都用 String 替换了。现在的 get 的取出来的不是 Object ，而是 String 对象，因此不需要类型转换。另外，当调用 set 时，只能传入 String 类型，否则编译无法通过。这就保证了 holder2 中的类型安全，避免由于不小心传入错误的类型。

通过上面的例子可以看出泛使得代码更简便、安全。引入泛型之后，Java 库的一些类，比如常用的容器类也被改写为支持泛型，我们使用的时候都会传入参数类型，如：ArrayList<Integer> list = ArrayList<>();。

# 类型擦除
Java 的泛型使用了类型擦除机制，这个引来了很大的争议，以至于 Java 的泛型功能受到限制，只能说是”伪泛型“。什么叫类型擦除呢？简单的说就是，类型参数只存在于编译期，在运行时，Java 的虚拟机 ( JVM ) 并不知道泛型的存在。



# 型参数“<T>”(声明一个泛型类或泛型方法)

## 声明泛型类的类型参数
```
class Box<T>{
    private T item1;
    private T item2;   
}


class Box<T>{
    private T item1;
    private T item2;
}
```

## 声明泛型方法
```
  public <T> void print(T t) {
    System.out.println(t);
  }
  
```

# 通配符“<?>”(使用泛型类或泛型方法)

## 上边界限定通配符


使用 List<? extends C> list 这种形式，表示 list 可以引用一个 ArrayList ( 或者其它 List 的 子类 ) 的对象，这个对象包含的元素类型是 C 的子类型 ( 包含 C 本身）的一种。


## 下边界限定通配符

使用 List<? super C> list 这种形式，表示 list 可以引用一个 ArrayList ( 或者其它 List 的 子类 ) 的对象，这个对象包含的元素就类型是 C 的超类型 ( 包含 C 本身 ) 的一种。

## 无边界通配符


# 其他
- “?”不能添加元素
- “? extends T”也不能添加元素
- “? super T”能添加元素
