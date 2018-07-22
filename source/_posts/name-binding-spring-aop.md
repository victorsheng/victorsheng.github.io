---
title: name-binding-spring-aop
date: 2018-07-13 10:02:10
tags:
  - aop
  - jdk proxy
  - cglib
categories:
---
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

# cglib 功能列表
- net.sf.cglib.core：底层字节码操作类；大部分与ASP相关。
- net.sf.cglib.transform：编译期、运行期的class文件转换类。
- net.sf.cglib.proxy：代理创建类、方法拦截类。
- net.sf.cglib.reflect：更快的反射类、C#风格的代理类。
- net.sf.cglib.util：集合排序工具类
- net.sf.cglib.beans：JavaBean相关的工具类

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

# cglib

## 底层如何生成class

## 生个生成的class内容
### HelloWorldImpl$$EnhancerByCGLIB$$e5f0da81$$FastClassByCGLIB$$b4ca737c.class 代理类的FastClass

### HelloWorldImpl$$EnhancerByCGLIB$$e5f0da81.class 代理类,继承了目标类
```
    private static final Method CGLIB$sayHello2$0$Method;
    private static final MethodProxy CGLIB$sayHello2$0$Proxy;

    static{
    CGLIB$sayHello2$0$Method = var10000[0];
    CGLIB$sayHello2$0$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/String;)V", "sayHello2", "CGLIB$sayHello2$0");
    
    }
    //被代理的方法
    final void CGLIB$sayHello2$0(String var1) {
    super.sayHello2(var1);
  }
    //代理方法
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
### HelloWorldImpl$$FastClassByCGLIB$$659f663a.class 被代理类的FastClass

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


## MethodInvoke创建

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
```

## MethodInvoke调用

```
  public Object invokeSuper(Object obj, Object[] args) throws Throwable {
    try {
      this.init();
      MethodProxy.FastClassInfo fci = this.fastClassInfo;
      //代理类FastClass.invoke(代理类的方法签名,obj,args)
      return fci.f2.invoke(fci.i2, obj, args);
    } catch (InvocationTargetException var4) {
      throw var4.getTargetException();
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

# 总结
最后我们总结一下JDK动态代理和Gglib动态代理的区别：
1.JDK动态代理是实现了被代理对象的接口，Cglib是继承了被代理对象。
2.JDK和Cglib都是在运行期生成字节码，JDK是直接写Class字节码，Cglib使用ASM框架写Class字节码，Cglib代理实现更复杂，生成代理类比JDK效率低。
3.JDK调用代理方法，是通过反射机制调用，Cglib是通过FastClass机制直接调用方法，Cglib执行效率更高。




# 参考
https://www.cnblogs.com/monkey0307/p/8328821.html



http://shift-alt-ctrl.iteye.com/blog/1894295