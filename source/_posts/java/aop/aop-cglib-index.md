---
title: cglib类库说明
tags:
  - aop
  - cglib
abbrlink: 3756615804
date: 2018-07-18 20:13:37
categories:
---
CGLIB(Code Generation Library)是一个强大的、高性能的代码生成库。它被广泛使用在基于代理的AOP框架（例如Spring AOP和dynaop）提供方法拦截。Hibernate作为最流行的ORM工具也同样使用CGLIB库来代理单端关联（集合懒加载除外，它使用另外一种机制）。EasyMock和jMock作为流行的Java测试库，它们提供Mock对象的方式来支持测试，都使用了CGLIB来对没有接口的类进行代理。
<!-- more -->

# cglib 功能列表
- net.sf.cglib.core：底层字节码操作类；大部分与ASP相关。
- net.sf.cglib.transform：编译期、运行期的class文件转换类。
- net.sf.cglib.proxy：代理创建类、方法拦截类。
- net.sf.cglib.reflect：更快的反射类、C#风格的代理类。
- net.sf.cglib.util：集合排序工具类
- net.sf.cglib.beans：JavaBean相关的工具类


# 生成代理类Enhancer

## 返回相同的结果
```
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(PersonService.class);
enhancer.setCallback((FixedValue) () -> "Hello Tom!");
PersonService proxy = (PersonService) enhancer.create();
 
String res = proxy.sayHello(null);
 
assertEquals("Hello Tom!", res);
```

在上面的示例中，增强器将返回SampleClass的已检测子类的实例，其中所有方法调用都返回由上面的匿名FixedValue实现生成的固定值。该对象由Enhancer create（Object ...）创建，其中该方法接受任意数量的参数，这些参数用于选择被代理类的任何构造函数。 （尽管构造函数只是Java字节代码级别的方法，但Enhancer类不能检测构造函数。它也不能检测静态类或final类。）如果你只想创建一个类，但没有实例，那么Enhancer #creinClass将创建一个可用于动态创建实例的类实例。被代理类的所有构造函数都将在此动态生成的类中作为委托构造函数使用。

请注意，在上面的示例中将委托任何方法调用，也调用java.lang.Object中定义的方法。因此，对proxy.toString（）的调用也将返回“Hello cglib！”。相反，对proxy.hashCode（）的调用会导致ClassCastException，因为FixedValue拦截器总是返回一个String，即使Object＃hashCode签名需要一个原始整数。

可以做出的另一个观察是final方法没有被截获。这种方法的一个例子是Object＃getClass，它将在调用时返回类似“SampleClass $$ EnhancerByCGLIB $$ e277c63c”的内容。此类名由cglib随机生成，以避免命名冲突。在程序代码中使用显式类型时，请注意增强实例的不同类。但是，由cglib生成的类将与被代理类位于同一个包中（因此可以覆盖package-private方法）。与final方法类似，子类化方法使得无法增强final类。因此，Hibernate框架无法持久化final类。

## 根据方法签名返回结果
```
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(PersonService.class);
enhancer.setCallback((MethodInterceptor) (obj, method, args, proxy) -> {
    if (method.getDeclaringClass() != Object.class && method.getReturnType() == String.class) {
        return "Hello Tom!";
    } else {
        return proxy.invokeSuper(obj, args);
    }
});
 
PersonService proxy = (PersonService) enhancer.create();
 
assertEquals("Hello Tom!", proxy.sayHello(null));
int lengthOfName = proxy.lengthOfName("Mary");
  
assertEquals(4, lengthOfName);
```
## MethodInerceptor
- MethodInterceptor允许完全控制截获的方法，并提供一些实用程序来调用处于原始状态的被代理类的方法。但是为什么人们还是想要使用其他方法呢？因为其他方法更有效，并且cglib经常用于边缘案例框架中，其中效率起着重要作用。 MethodInterceptor的创建和链接需要生成不同类型的字节代码以及创建InvocationHandler不需要的某些运行时对象。因此，还有其他类可以与Enhancer一起使用：
- LazyLoader：尽管LazyLoader的唯一方法与FixedValue具有相同的方法签名，但LazyLoader与FixedValue拦截器根本不同。实际上，LazyLoader应该返回被代理类的子类的实例。仅当在增强对象上调用方法然后存储该方法以供将来调用生成的代理时，才会请求此实例。如果你的对象在创建过程中很昂贵而不知道是否会使用该对象，这是有道理的。请注意，必须为代理对象和延迟加载的对象调用被代理类的某些构造函数。因此，请确保有另一个廉价（可能受保护）的构造函数可用或使用代理的接口类型。您可以通过向Enhancer #create（Object ...）提供参数来选择所调用的构造。
- Dispatcher：Dispatcher与LazyLoader类似，但会在每次方法调用时调用，而不存储加载的对象。这允许更改类的实现而不更改对它的引用。同样，请注意必须为代理和生成的对象调用某些构造函数。
- ProxyRefDispatcher：此类携带对其签名中调用的代理对象的引用。这允许例如将方法调用委托给该代理的另一个方法。请注意，如果从ProxyRefDispatcher＃loadObject（Object）中调用相同的方法，这很容易导致无限循环，并且总是会导致无限循环。
- NoOp：NoOp类并不是它的名字所暗示的。相反，它将每个方法调用委托给被代理类的方法实现。

此时，最后两个拦截器可能对你没有意义。 当你总是将方法调用委托给增强类时，为什么你甚至想要增强一个类呢？ 而你是对的。 这些拦截器应该只与CallbackFilter一起使用，如下面的代码片段所示：
```
@Test
public void testCallbackFilter() throws Exception {
  Enhancer enhancer = new Enhancer();
  CallbackHelper callbackHelper = new CallbackHelper(SampleClass.class, new Class[0]) {
    @Override
    protected Object getCallback(Method method) {
      if(method.getDeclaringClass() != Object.class && method.getReturnType() == String.class) {
        return new FixedValue() {
          @Override
          public Object loadObject() throws Exception {
            return "Hello cglib!";
          };
        }
      } else {
        return NoOp.INSTANCE; // A singleton provided by NoOp.
      }
    }
  };
  enhancer.setSuperclass(MyClass.class);
  enhancer.setCallbackFilter(callbackHelper);
  enhancer.setCallbacks(callbackHelper.getCallbacks());
  SampleClass proxy = (SampleClass) enhancer.create();
  assertEquals("Hello cglib!", proxy.test(null));
  assertNotEquals("Hello cglib!", proxy.toString());
  proxy.hashCode(); // Does not throw an exception or result in an endless loop.
}
```
Enhancer实例在其Enhancer＃setCallbackFilter（CallbackFilter）方法中接受CallbackFilter，它希望增强类的方法映射到Callback实例数组的数组索引。 当在创建的代理上调用方法时，Enhancer将选择相应的拦截器并在相应的Callback上调度被调用的方法（这是迄今为止引入的所有拦截器的标记接口）。 为了使这个API不那么笨拙，cglib提供了一个CallbackHelper，它代表一个CallbackFilter，可以为你创建一个Callbacks数组。 上面的增强对象在功能上等同于MethodInterceptor示例中的对象，但它允许您编写专门的拦截器，同时保持这些拦截器的调度逻辑分离。

## 如何工作
当Enhancer创建一个类时，它将为每个拦截器创建一个私有静态字段，该拦截器在创建后被注册为增强类的Callback。这也意味着使用cglib创建的类定义在创建后无法重用，因为回调的注册不会成为生成的类的初始化阶段的一部分，而是在类已经由JVM初始化之后由cglib手动准备。这也意味着使用cglib创建的类在初始化后在技术上尚未就绪，并且例如无法通过线路发送，因为目标机器中加载的类不存在回调。

根据已注册的拦截器，cglib可能会注册其他字段，例如MethodInterceptor，其中两个私有静态字段（一个持有反射方法，另一个持有MethodProxy）按照在增强类或任何中截获的方法进行注册。它的子类。请注意，MethodProxy正在过度使用FastClass，这会触发创建其他类，并在下面进一步详细介绍。

出于所有这些原因，使用Enhancer时要小心。并且始终以防御方式注册回调类型，因为MethodInterceptor将例如触发创建其他类并在增强类中注册其他字段。这是特别危险的，因为回调变量也存储在增强器的缓存中：这意味着回调实例不是垃圾收集的（除非所有实例和增强器的ClassLoader都是，这是不寻常的）。当使用匿名类来静默地携带对其外部类的引用时，这尤其危险。回想一下上面的例子：

```
@Test
public void testFixedValue() throws Exception {
  Enhancer enhancer = new Enhancer();
  enhancer.setSuperclass(SampleClass.class);
  enhancer.setCallback(new FixedValue() {
    @Override
    public Object loadObject() throws Exception {
      return "Hello cglib!";
    }
  });
  SampleClass proxy = (SampleClass) enhancer.create();
  assertEquals("Hello cglib!", proxy.test(null));
}
```

FixedValue的匿名子类很难从增强的SampleClass中引用，因此匿名的FixedValue实例或持有@Test方法的类都不会被垃圾收集。这可能会在您的应用程序中导致令人讨厌的内存泄漏。因此，不要将非静态内部类与cglib一起使用。 （我只在博客文章中使用它们来保持示例简短。）

最后，你永远不应该拦截Object＃finalize（）。由于cglib的子类化方法，拦截最终化是通过覆盖它来实现的，这通常是一个坏主意。拦截finalize的增强实例将被垃圾收集器区别对待，并且还会导致这些对象在JVM的终结队列中排队。此外，如果您（意外地）在截获的finalize调用中创建了对增强类的硬引用，则您已经有效地创建了一个不可收集的实例。这通常不是你想要的。请注意，最终方法永远不会被cglib截获。因此，Object＃wait，Object＃notify和Object＃notifyAll不会产生同样的问题。但请注意，可以拦截Object＃clone，这可能是您不想做的事情。

#  BeanGenerator
它允许我们动态创建bean并使用setter和getter方法添加字段。代码生成工具可以使用它来生成简单的POJO对象：

```
BeanGenerator beanGenerator = new BeanGenerator();
 
beanGenerator.addProperty("name", String.class);
Object myBean = beanGenerator.create();
Method setter = myBean.getClass().getMethod("setName", String.class);
setter.invoke(myBean, "some string value set by a cglib");
 
Method getter = myBean.getClass().getMethod("getName");
assertEquals("some string value set by a cglib", getter.invoke(myBean));
```

# Mixin (合体)
mixin是一种允许将多个对象组合成一个的构造。 我们可以包含几个类的行为，并将该行为公开为单个类或接口。 cglib Mixins允许将多个对象组合成一个对象。 但是，为了这样做，mixin中包含的所有对象必须由接口支持。

```
public interface Interface1 {
    String first();
}
 
public interface Interface2 {
    String second();
}
 
public class Class1 implements Interface1 {
    @Override
    public String first() {
        return "first behaviour";
    }
}
 
public class Class2 implements Interface2 {
    @Override
    public String second() {
        return "second behaviour";
    }
}

	
public interface MixinInterface extends Interface1, Interface2 { }

```
应用

```
Mixin mixin = Mixin.create(
  new Class[]{ Interface1.class, Interface2.class, MixinInterface.class },
  new Object[]{ new Class1(), new Class2() }
);
MixinInterface mixinDelegate = (MixinInterface) mixin;
 
assertEquals("first behaviour", mixinDelegate.first());
assertEquals("second behaviour", mixinDelegate.second());
```


# Immutable bean
cglib的ImmutableBean允许您创建类似于例如Collections＃immutableSet的不变性包装器。 IllegalStateException将阻止底层bean的所有更改（但是，不是由Java API推荐的UnsupportedOperationException）。
```
public class SampleBean {
  private String value;
  public String getValue() {
    return value;
  }
  public void setValue(String value) {
    this.value = value;
  }
}


@Test(expected = IllegalStateException.class)
public void testImmutableBean() throws Exception {
  SampleBean bean = new SampleBean();
  bean.setValue("Hello world!");
  SampleBean immutableBean = (SampleBean) ImmutableBean.create(bean);
  assertEquals("Hello world!", immutableBean.getValue());
  bean.setValue("Hello world, again!");
  assertEquals("Hello world, again!", immutableBean.getValue());
  immutableBean.setValue("Hello cglib!"); // Causes exception.
}


```

# Bean copier

```
public class OtherSampleBean {
  private String value;
  public String getValue() {
    return value;
  }
  public void setValue(String value) {
    this.value = value;
  }
}

@Test
public void testBeanCopier() throws Exception {
  BeanCopier copier = BeanCopier.create(SampleBean.class, OtherSampleBean.class, false);
  SampleBean bean = new SampleBean();
  myBean.setValue("Hello cglib!");
  OtherSampleBean otherBean = new OtherSampleBean();
  copier.copy(bean, otherBean, null);
  assertEquals("Hello cglib!", otherBean.getValue());  
}
```

# Bulk bean
BulkBean获取一个getter名称数组，一个setter名称数组和一个属性类型数组作为其构造函数参数。 然后，可以通过BulkBean＃getPropertyBalues（Object）将生成的检测类提取为数组。 类似地，可以通过BulkBean＃setPropertyBalues（Object，Object []）设置bean的属性。

```
@Test
public void testBulkBean() throws Exception {
  BulkBean bulkBean = BulkBean.create(SampleBean.class, 
      new String[]{"getValue"}, 
      new String[]{"setValue"}, 
      new Class[]{String.class});
  SampleBean bean = new SampleBean();
  bean.setValue("Hello world!");
  assertEquals(1, bulkBean.getPropertyValues(bean).length);
  assertEquals("Hello world!", bulkBean.getPropertyValues(bean)[0]);
  bulkBean.setPropertyValues(bean, new Object[] {"Hello cglib!"});
  assertEquals("Hello cglib!", bean.getValue());
}

```

# bean map
```
@Test
public void testBeanGenerator() throws Exception {
  SampleBean bean = new SampleBean();
  BeanMap map = BeanMap.create(bean);
  bean.setValue("Hello cglib!");
  assertEquals("Hello cglib", map.get("value"));
}
```

# Key factory 
KeyFactory工厂允许动态创建由多个值组成的键，这些值可用于例如Map实现。 为此，KeyFactory需要一些接口来定义应该在这样的密钥中使用的值。 此接口必须包含名为newInstance的单个方法，该方法返回Object。 例如：

```
public interface SampleKeyFactory {
  Object newInstance(String first, int second);
}

@Test
public void testKeyFactory() throws Exception {
  SampleKeyFactory keyFactory = (SampleKeyFactory) KeyFactory.create(Key.class);
  Object key = keyFactory.newInstance("foo", 42);
  Map<Object, String> map = new HashMap<Object, String>();
  map.put(key, "Hello cglib!");
  assertEquals("Hello cglib!", map.get(keyFactory.newInstance("foo", 42)));
}
```
KeyFactory将确保正确实现Object＃equals（Object）和Object＃hashCode方法，以便生成的关键对象可以在Map或Set中使用。 KeyFactory也在cglib库中内部使用了很多。

# String switcher
```
@Test
public void testStringSwitcher() throws Exception {
  String[] strings = new String[]{"one", "two"};
  int[] values = new int[]{10, 20};
  StringSwitcher stringSwitcher = StringSwitcher.create(strings, values, true);
  assertEquals(10, stringSwitcher.intValue("one"));
  assertEquals(20, stringSwitcher.intValue("two"));
  assertEquals(-1, stringSwitcher.intValue("three"));
}

```
StringSwitcher允许在字符串上模拟一个switch命令，例如自Java 7以来可以使用内置的Java switch语句。如果在Java 6或更低版本中使用StringSwitcher确实为你的代码增加了一个好处，那么我仍然会怀疑 不建议使用它。

# Interface maker
```
@Test
public void testInterfaceMaker() throws Exception {
  Signature signature = new Signature("foo", Type.DOUBLE_TYPE, new Type[]{Type.INT_TYPE});
  InterfaceMaker interfaceMaker = new InterfaceMaker();
  interfaceMaker.add(signature, new Type[0]);
  Class iface = interfaceMaker.create();
  assertEquals(1, iface.getMethods().length);
  assertEquals("foo", iface.getMethods()[0].getName());
  assertEquals(double.class, iface.getMethods()[0].getReturnType());
}

```
除了任何其他类的cglib的公共API之外，接口制造商还依赖于ASM类型。 在正在运行的应用程序中创建接口几乎没有意义，因为接口仅表示编译器可用于检查类型的类型。 但是，当您生成将在以后的开发中使用的代码时，它可能是有意义的。


# Method delegate
```

public class DelegateTest {

  @Test
  public void testMethodDelegate() throws Exception {
    SampleBean bean = new SampleBean();
    bean.setValue("Hello cglib!");
    BeanDelegate delegate = (BeanDelegate) MethodDelegate.create(
        bean, "getValue", BeanDelegate.class);
    assertEquals("Hello cglib!", delegate.getValueFromDelegate());
  }
}

```
但是有一些事情需要注意：
- 工厂方法MethodDelegate #create只接受一个方法名称作为其第二个参数。这是MethodDelegate将为您代理的方法。
- 必须有一个没有为对象定义参数的方法，该方法作为第一个参数提供给工厂方法。因此，MethodDelegate没有那么强大。
- 第三个参数必须是只有一个参数的接口。 MethodDelegate实现了这个接口，可以强制转换为它。调用该方法时，它将在作为第一个参数的对象上调用代理方法。
此外，考虑这些缺点：
- cglib为每个代理创建一个新类。最终，这会浪费你的永久代堆空间
- 您不能代理带参数的方法。
- 如果您的接口接受参数，则方法委托将在没有抛出异常的情况下无法工作（返回值将始终为null）。如果你的接口需要另一种返回类型（即使它更通用），你将得到一个IllegalArgumentException。

# Multicast delegate
```
public interface DelegatationProvider {
  void setValue(String value);
}
 
public class SimpleMulticastBean implements DelegatationProvider {
  private String value;
  public String getValue() {
    return value;
  }
  public void setValue(String value) {
    this.value = value;
  }
}

@Test
public void testMulticastDelegate() throws Exception {
  MulticastDelegate multicastDelegate = MulticastDelegate.create(
      DelegatationProvider.class);
  SimpleMulticastBean first = new SimpleMulticastBean();
  SimpleMulticastBean second = new SimpleMulticastBean();
  multicastDelegate = multicastDelegate.add(first);
  multicastDelegate = multicastDelegate.add(second);
 
  DelegatationProvider provider = (DelegatationProvider)multicastDelegate;
  provider.setValue("Hello world!");
 
  assertEquals("Hello world!", first.getValue());
  assertEquals("Hello world!", second.getValue());
}

```

同样，有一些缺点：
对象需要实现单方法接口。 这对于第三方库来说很糟糕，当你使用CGlib做一些神奇的事情时，这种魔法会暴露在普通代码中，这很麻烦。 此外，您可以轻松地实现自己的委托（虽然没有字节代码，但我怀疑您在手动委托方面赢了很多）。
当您的代理人返回一个值时，您将只收到您添加的最后一个代表的值。 所有其他返回值都将丢失（但在某个时候由多播委托检索）。

# Constructor delegate

```

public interface SampleBeanConstructorDelegate {
  Object newInstance();
}
 
@Test
public void testConstructorDelegate() throws Exception {
  SampleBeanConstructorDelegate constructorDelegate = (SampleBeanConstructorDelegate) ConstructorDelegate.create(
    SampleBean.class, SampleBeanConstructorDelegate.class);
  SampleBean bean = (SampleBean) constructorDelegate.newInstance();
  assertTrue(SampleBean.class.isAssignableFrom(bean.getClass()));
}

```
# Parallel sorter
在排序数组数组时，ParallelSorter声称是Java标准库的数组排序器的更快替代品
```
@Test
public void testParallelSorter() throws Exception {
  Integer[][] value = {
    {4, 3, 9, 0},
    {2, 1, 6, 0}
  };
  ParallelSorter.create(value).mergeSort(0);
  for(Integer[] row : value) {
    int former = -1;
    for(int val : row) {
      assertTrue(former < val);
      former = val;
    }
  }
}
```

# Fast class and fast members
通过包装Java类并为反射API提供类似的方法，FastClass承诺比Java反射API更快地调用方法
```
@Test
public void testFastClass() throws Exception {
  FastClass fastClass = FastClass.create(SampleBean.class);
  FastMethod fastMethod = fastClass.getMethod(SampleBean.class.getMethod("getValue"));
  MyBean myBean = new MyBean();
  myBean.setValue("Hello cglib!");
  assertTrue("Hello cglib!", fastMethod.invoke(myBean, new Object[0]));
}
```

除了演示的FastMethod之外，FastClass还可以创建FastConstructors但不会创建快速字段。但FastClass如何比正常反射更快？ Java反射由JNI执行，其中方法调用由某些C代码执行。另一方面，FastClass创建了一些字节代码，可以直接从JVM中调用该方法。但是，较新版本的HotSpot JVM（可能还有许多其他现代JVM）知道一个名为**通胀(inflation)**的概念，当反射方法执行得足够频繁时，JVM会将反射方法调用转换为FastClass的本机版本(C代码执行))。您甚至可以通过将sun.reflect.inflationThreshold属性设置为较低的值来控制此行为（至少在HotSpot JVM上）。 （默认值为15.）此属性确定在多少反射调用之后，JNI调用应由字节代码检测版本替换。因此，我建议不要在现代JVM上使用FastClass，但它可以微调旧Java虚拟机的性能。


# 参考
http://www.baeldung.com/cglib

http://mydailyjava.blogspot.com/2013/11/cglib-missing-manual.html