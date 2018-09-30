---
title: class上找不到的注解不会抛出ClassNotFoundException
abbrlink: 1148835200
date: 2018-07-13 10:45:54
tags:
categories:
---
# 解答
https://stackoverflow.com/questions/3567413/why-doesnt-a-missing-annotation-cause-a-classnotfoundexception-at-runtime


在早期的JSR-175（注释）公共草案中，讨论了编译器和运行时是否应该忽略未知注释，以便在注释的使用和声明之间提供更松散的耦合。一个具体示例是在EJB上使用应用程序服务器特定的注释来控制部署配置。如果应该在不同的应用程序服务器上部署相同的bean，那么如果运行时只是忽略未知的注释而不是引发NoClassDefFoundError，那将会很方便。

即使措辞有点模糊，我也假设您在JLS 13.5.7中指定了您所看到的行为：“...删除注释对Java编程语言中程序的二进制表示的正确链接没有影响“。我将此解释为好像删除了注释（在运行时不可用），程序仍应链接并运行，这意味着在通过反射访问时，将忽略未知注释。

# 如何会触发呢??

当程序试图使用class类中的forname方法、classloader类中的findsystemclass方法，classloader类中loadclass方法通过字符串名的形式加载此类时，会抛出该异常


Thrown when an application tries to load in a class through its string name using:

- The forName method in class Class.
- The findSystemClass method in class ClassLoader.
- The loadClass method in class ClassLoader.
but no definition for the class with the specified name could be found.


```
  @Test
  public void noclass() throws ClassNotFoundException {
    Class<?> aClass = Class.forName("www.qqq");
  }

  @Test
  public void noClass() {
    DispatcherServlet dispatcherServlet = new DispatcherServlet();
  }
```
其中,springmvc的scopew为provided,不会编译到target中
```
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.3.13.RELEASE</version>
      <scope>provided</scope>
    </dependency>
```

# ClassNotFoundException和NoClassDefFoundError的区别


## ClassNotFoundException:
- Class loader fails to **verify** a class byte code we mention in Link phase of class loading subsystem we get ClassNotFoundException.
- explicit loading明显的加载

## NoClassDefFoundError:
- Class loader fails to **resolving** references of a class in Link phase of class loading subsystem we get NoClassDefFoundError.
- NoClassDefFoundError is an Error derived from LinkageError class, which is used to indicate error cases, where a class has a dependency on some other class and that class has incompatibly changed after the compilation.
- implicit loading含蓄的加载

![upload successful](/images/pasted-213.png)

### 试验

NoClassDefFoundError is an error that occurs when a particular class is present at compile time, but was missing at run time.

```
    class A
    {
      // some code
    }
    public class B
    {
        public static void main(String[] args)
        {
            A a = new A();
        }
    }

```
When you compile the above program, two .class files will be generated. One is A.class and another one is B.class. If you remove the A.class file and run the B.class file, Java Runtime System will throw NoClassDefFoundError like below:
```
    Exception in thread "main" java.lang.NoClassDefFoundError: A
    at MainClass.main(MainClass.java:10)
    Caused by: java.lang.ClassNotFoundException: A
    at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:331)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
```