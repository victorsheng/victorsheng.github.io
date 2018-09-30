---
title: 'springaop的底层实现:jdkproxy-cglib源码对比'
tags:
  - aop
  - jdk proxy
  - cglib
abbrlink: 4030607687
date: 2018-07-13 10:02:10
categories:
---
# spring aop的底层

每次运行时动态的增强，生成AOP代理对象，区别在于生成AOP代理对象的时机不同，相对来说AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。

```
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {

	@Override
	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
		if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
			Class<?> targetClass = config.getTargetClass();
			if (targetClass == null) {
				throw new AopConfigException("TargetSource cannot determine target class: " +
						"Either an interface or a target is required for proxy creation.");
			}
			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
				return new JdkDynamicAopProxy(config);
			}
			return new ObjenesisCglibAopProxy(config);
		}
		else {
			return new JdkDynamicAopProxy(config);
		}
	}
```



# 使用AspectJ的编译时增强实现AOP

# spring aop
Spring AOP中的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理。JDK动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。JDK动态代理的核心是InvocationHandler接口和Proxy类。

如果目标类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。


# jdk proxy
## 添加增强代码的接口
```
public class CustomInvocationHandler implements InvocationHandler {
    private Object target;

    public CustomInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before invocation");
        Object retVal = method.invoke(target, args);
        System.out.println("After invocation");
        return retVal;
    }
}
```

## 客户端
```
  @Test
  public void test() {
    System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", true);

    CustomInvocationHandler handler = new CustomInvocationHandler(new HelloWorldImpl());
    HelloWorld proxy = (HelloWorld) Proxy.newProxyInstance(
        ProxyTest.class.getClassLoader(),
        new Class[]{HelloWorld.class},
        handler);
    proxy.sayHello("Mikan");
    proxy.sayHello2("Mikan");
  }
```

## 生成的代理类

```
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

import com.example.demo.proxy.HelloWorld;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy11 extends Proxy implements HelloWorld {
  private static Method m1;
  private static Method m3;
  private static Method m4;
  private static Method m2;
  private static Method m0;

  public $Proxy11(InvocationHandler var1) throws  {
    super(var1);
  }

  public final boolean equals(Object var1) throws  {
    try {
      return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
    } catch (RuntimeException | Error var3) {
      throw var3;
    } catch (Throwable var4) {
      throw new UndeclaredThrowableException(var4);
    }
  }

  public final void sayHello(String var1) throws  {
    try {
      super.h.invoke(this, m3, new Object[]{var1});
    } catch (RuntimeException | Error var3) {
      throw var3;
    } catch (Throwable var4) {
      throw new UndeclaredThrowableException(var4);
    }
  }

  public final void sayHello2(String var1) throws  {
    try {
      super.h.invoke(this, m4, new Object[]{var1});
    } catch (RuntimeException | Error var3) {
      throw var3;
    } catch (Throwable var4) {
      throw new UndeclaredThrowableException(var4);
    }
  }

  public final String toString() throws  {
    try {
      return (String)super.h.invoke(this, m2, (Object[])null);
    } catch (RuntimeException | Error var2) {
      throw var2;
    } catch (Throwable var3) {
      throw new UndeclaredThrowableException(var3);
    }
  }

  public final int hashCode() throws  {
    try {
      return (Integer)super.h.invoke(this, m0, (Object[])null);
    } catch (RuntimeException | Error var2) {
      throw var2;
    } catch (Throwable var3) {
      throw new UndeclaredThrowableException(var3);
    }
  }

  static {
    try {
      m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
      m3 = Class.forName("com.example.demo.proxy.HelloWorld").getMethod("sayHello", Class.forName("java.lang.String"));
      m4 = Class.forName("com.example.demo.proxy.HelloWorld").getMethod("sayHello2", Class.forName("java.lang.String"));
      m2 = Class.forName("java.lang.Object").getMethod("toString");
      m0 = Class.forName("java.lang.Object").getMethod("hashCode");
    } catch (NoSuchMethodException var2) {
      throw new NoSuchMethodError(var2.getMessage());
    } catch (ClassNotFoundException var3) {
      throw new NoClassDefFoundError(var3.getMessage());
    }
  }
}

```

生成的代理类是实现代理接口,继承Proxy类的
`public final class $Proxy11 extends Proxy implements HelloWorld`

代理类中有各个方法的静态成员变量,包括了接口的方法+toString+hashCode+equals等方法,
生成的代码中 将上下文传给代理类来做实际的调用,这样就可以在实际调用前后添加额外的逻辑了
```
  public final void sayHello(String var1) throws  {
    try {
      super.h.invoke(this, m3, new Object[]{var1});
    } catch (RuntimeException | Error var3) {
      throw var3;
    } catch (Throwable var4) {
      throw new UndeclaredThrowableException(var4);
    }
  }
```
其中`super.h` super为父类:java.lang.reflect.Proxy
h为成员变量: `protected InvocationHandler h;`
至此,将程序的访问权限交给了代理的接口


---------------------------------------------------------------------------------------------------------------------------------------------------


# cglib

## 底层如何生成class

## 生个生成的class内容
### HelloWorldImpl$$EnhancerByCGLIB$$e5f0da81$$FastClassByCGLIB$$b4ca737c.class 代理类的FastClass

### HelloWorldImpl$$FastClassByCGLIB$$659f663a.class 被代理类的FastClass

### HelloWorldImpl$$EnhancerByCGLIB$$e5f0da81.class 代理类,继承了目标类
```
    private static final Method CGLIB$sayHello2$0$Method;
    private static final MethodProxy CGLIB$sayHello2$0$Proxy;

    static{
    //原始方法
    CGLIB$sayHello2$0$Method = var10000[0];
    //代理方法
    CGLIB$sayHello2$0$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/String;)V", "sayHello2", "CGLIB$sayHello2$0");
    
    }
    //被代理的方法 
    //methodProxy.invokeSuper会调用
    final void CGLIB$sayHello2$0(String var1) {
    super.sayHello2(var1);
  }
    //代理方法
    //methodProxy.invoke会调用
    public final void sayHello2(String var1) {
    //MethodInterceptor 自己自定义的拦截的实现
    MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
    if (this.CGLIB$CALLBACK_0 == null) {
      CGLIB$BIND_CALLBACKS(this);
      var10000 = this.CGLIB$CALLBACK_0;
    }

    if (var10000 != null) {
      //调用拦截器
      var10000.intercept(this, CGLIB$sayHello2$0$Method, new Object[]{var1}, CGLIB$sayHello2$0$Proxy);
    } else {
      super.sayHello2(var1);
    }
  }

   public static MethodProxy CGLIB$findMethodProxy(Signature var0) {
    String var10000 = var0.toString();
    switch(var10000.hashCode()) {
    case -760609420:
      if (var10000.equals("sayHello2(Ljava/lang/String;)V")) {
        return CGLIB$sayHello2$0$Proxy;
      }
      break;
    }}
```


## 代理方法

```
  public final void sayHello2(String var1) {
    MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
    if (this.CGLIB$CALLBACK_0 == null) {
      CGLIB$BIND_CALLBACKS(this);
      var10000 = this.CGLIB$CALLBACK_0;
    }

    if (var10000 != null) {
      var10000.intercept(this, CGLIB$sayHello2$0$Method, new Object[]{var1}, CGLIB$sayHello2$0$Proxy);
    } else {
      super.sayHello2(var1);
    }
  }
```


## MethodProxy创建(对象中包含代理方法和被代理方法的信息)
```
    Class var0 = Class.forName("com.example.demo.proxy.HelloWorldImpl2$$EnhancerByCGLIB$$6b589cfb");//代理类
    Class var1;//被代理类

    CGLIB$sayHello2$0$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/String;)V", "sayHello2", "CGLIB$sayHello2$0");
```

```
public class MethodProxy {
    private Signature sig1;
    private Signature sig2;
    private MethodProxy.CreateInfo createInfo;
    private final Object initLock = new Object();
    private volatile MethodProxy.FastClassInfo fastClassInfo;
    //c1:被代理对象Class
    //c2:代理对象Class
    //desc：入参类型
    //name1:被代理方法名
    //name2:代理方法名
    public static MethodProxy create(Class c1, Class c2, String desc, String name1, String name2) {
        MethodProxy proxy = new MethodProxy();
        proxy.sig1 = new Signature(name1, desc);//被代理方法签名
        proxy.sig2 = new Signature(name2, desc);//代理方法签名
        //class的关联
        proxy.createInfo = new MethodProxy.CreateInfo(c1, c2);
        return proxy;
    }

private static class CreateInfo {
    Class c1;
    Class c2;
    NamingPolicy namingPolicy;
    GeneratorStrategy strategy;
    boolean attemptLoad;

    public CreateInfo(Class c1, Class c2) {
        this.c1 = c1;
        this.c2 = c2;
        AbstractClassGenerator fromEnhancer = AbstractClassGenerator.getCurrent();
        if(fromEnhancer != null) {
            this.namingPolicy = fromEnhancer.getNamingPolicy();
            this.strategy = fromEnhancer.getStrategy();
            this.attemptLoad = fromEnhancer.getAttemptLoad();
        }

    }
}
  //MethodProxy invoke/invokeSuper都调用了init();
  //如果this.fastClassInfo不存在则进行初始化
  private void init() {
    if (this.fastClassInfo == null) {
      Object var1 = this.initLock;
      synchronized(this.initLock) {
        if (this.fastClassInfo == null) {
          MethodProxy.CreateInfo ci = this.createInfo;
          MethodProxy.FastClassInfo fci = new MethodProxy.FastClassInfo();
          //ci.c1:被代理对象Class
          fci.f1 = helper(ci, ci.c1);
          //ci.c2:代理对象Class
          fci.f2 = helper(ci, ci.c2);
          //被代理方法签名
          fci.i1 = fci.f1.getIndex(this.sig1);
          //代理方法签名
          fci.i2 = fci.f2.getIndex(this.sig2);
          this.fastClassInfo = fci;
          this.createInfo = null;
        }
      }
    }

  }
```
## 控制权交至增强方法中,可以调用MethodProxy的方法



## MethodProxy的invokeSuper和invoke方法

通过断点观察到实际是通过子类集成的方式调用的原本方法(实际执行的是代理类的方法)
![upload successful](/images/pasted-215.png)

```
  //在此次动态代理中有执行过
  public Object invokeSuper(Object obj, Object[] args) throws Throwable {
    try {
      this.init();
      MethodProxy.FastClassInfo fci = this.fastClassInfo;
      //代理类FastClass.invoke(代理类的方法签名,obj,args)
      //代理方法
      return fci.f2.invoke(fci.i2, obj, args);
    } catch (InvocationTargetException var4) {
      throw var4.getTargetException();
    }
  }

   //在此次动态代理中没有执行过
  public Object invoke(Object obj, Object[] args) throws Throwable {
    try {
      this.init();
      MethodProxy.FastClassInfo fci = this.fastClassInfo;
      return fci.f1.invoke(fci.i1, obj, args);
    } catch (InvocationTargetException var4) {
      throw var4.getTargetException();
    } catch (IllegalArgumentException var5) {
      if (this.fastClassInfo.i1 < 0) {
        throw new IllegalArgumentException("Protected method: " + this.sig1);
      } else {
        throw var5;
      }
    }
  }

private static class FastClassInfo {
    FastClass f1;//被代理类FastClass
    FastClass f2;//代理类FastClass
    int i1; //被代理类的方法签名(index)
    int i2;//代理类的方法签名

    private FastClassInfo() {
    }
}
```

## fastClass接口(只负责方法路由,无成员变量,线程安全)

```
     /**
     * Return the index of the matching method. The index may be used
     * later to invoke the method with less overhead. If more than one
     * method matches (i.e. they differ by return type only), one is
     * chosen arbitrarily.
     * @see #invoke(int, Object, Object[])
     * @param name the method name
     * @param parameterTypes the parameter array
     * @return the index, or <code>-1</code> if none is found.
     */
    abstract public int getIndex(String name, Class[] parameterTypes);

    /**
     * Return the index of the matching constructor. The index may be used
     * later to create a new instance with less overhead.
     * @see #newInstance(int, Object[])
     * @param parameterTypes the parameter array
     * @return the constructor index, or <code>-1</code> if none is found.
     */
    abstract public int getIndex(Class[] parameterTypes);

    /**
     * Invoke the method with the specified index.
     * @see getIndex(name, Class[])
     * @param index the method index
     * @param obj the object the underlying method is invoked from
     * @param args the arguments used for the method call
     * @throws java.lang.reflect.InvocationTargetException if the underlying method throws an exception
     */
    abstract public Object invoke(int index, Object obj, Object[] args) throws InvocationTargetException;

    /**
     * Create a new instance using the specified constructor index and arguments.
     * @see getIndex(Class[])
     * @param index the constructor index
     * @param args the arguments passed to the constructor
     * @throws java.lang.reflect.InvocationTargetException if the constructor throws an exception
     */
    abstract public Object newInstance(int index, Object[] args) throws InvocationTargetException;

    abstract public int getIndex(Signature sig);

    /**
     * Returns the maximum method index for this class.
     */
    abstract public int getMaxIndex();
```

## 通过计算getIndex+index switch的机制来进行方法调用(提高性能)

```
  public Object invoke(int var1, Object var2, Object[] var3) throws InvocationTargetException {
    HelloWorldImpl2 var10000 = (HelloWorldImpl2)var2;
    int var10001 = var1;

    try {
      switch(var10001) {
      case 0:
        return var10000.sayHello((String)var3[0]);
      case 1:
        return new Boolean(var10000.equals(var3[0]));
      case 2:
        return var10000.toString();
      case 3:
        return new Integer(var10000.hashCode());
      }
    } catch (Throwable var4) {
      throw new InvocationTargetException(var4);
    }

    throw new IllegalArgumentException("Cannot find matching method/constructor");
  }

```



# 总结
最后我们总结一下JDK动态代理和Gglib动态代理的区别：
1.JDK动态代理是实现了被代理对象的接口，Cglib是继承了被代理对象。
2.JDK和Cglib都是在运行期生成字节码，JDK是直接写Class字节码，Cglib使用ASM框架写Class字节码，Cglib代理实现更复杂，生成代理类比JDK效率低。
3.JDK调用代理方法，是通过**反射机制**调用，Cglib是通过FastClass机制直接调用方法，Cglib执行效率更高。




# 参考
https://www.cnblogs.com/monkey0307/p/8328821.html

http://shift-alt-ctrl.iteye.com/blog/1894295