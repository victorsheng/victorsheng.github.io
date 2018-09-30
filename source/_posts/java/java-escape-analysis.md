---
title: 逃逸分析
abbrlink: 3212954465
date: 2018-08-10 09:57:33
tags:
categories:
---
# 逃逸

逃逸分析的基本行为就是分析对象动态作用域：当一个对象在方法中被定义后，它可能被外部方法所引用，例如作为调用参数传递到其他地方中，称为方法逃逸。
```
public static StringBuffer craeteStringBuffer(String s1, String s2) {
        StringBuffer sb = new StringBuffer();
        sb.append(s1);
        sb.append(s2);
        return sb;
    }
```

StringBuffer sb是一个方法内部变量，上述代码中直接将sb返回，这样这个StringBuffer有可能被其他方法所改变，这样它的作用域就不只是在方法内部，虽然它是一个局部变量，称其逃逸到了方法外部。

甚至还有可能被外部线程访问到，譬如赋值给类变量或可以在其他线程中访问的实例变量，称为线程逃逸。

上述代码如果想要StringBuffer sb不逃出方法，可以这样写：

```
public static String createStringBuffer(String s1, String s2) {
        StringBuffer sb = new StringBuffer();
        sb.append(s1);
        sb.append(s2);
        return sb.toString();
    }
```
不直接返回 StringBuffer，那么StringBuffer将不会逃逸出方法。

如果能证明一个对象不会逃逸到方法或线程外，则可能为这个变量进行一些高效的优化。


# 参考资料
http://www.importnew.com/23150.html