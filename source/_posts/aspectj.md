---
title: aspectj
date: 2018-07-09 11:26:23
tags:
    - aop
categories:
---
# full AspectJ

http://www.baeldung.com/aspectj


https://www.eclipse.org/aspectj/

# spring 2.0 aop 
https://docs.spring.io/spring/docs/2.5.x/reference/aop.html


# Spring AOP or full AspectJ如何选择?
使用最简单的方法。 Spring AOP比使用完整的AspectJ更简单，因为不需要将AspectJ编译器/ weaver引入开发和构建过程。
- 如果您只需要建议在Spring bean上执行操作，那么Spring AOP是正确的选择。
- 如果您需要建议**不是由Spring容器管理的领域对象（通常是domain object）**，那么您将需要使用AspectJ。
- 如果您希望建议除简单方法执行之外的连接点（例如，字段获取(get field)或设置连接点等），您还需要使用AspectJ。

# 代理机制
Spring AOP使用JDK动态代理或CGLIB为给定目标对象创建代理。 （只要需要选择，JDK动态代理就是首选）。
如果要代理的目标对象实现至少一个接口，则将使用JDK动态代理。目标类型实现的所有接口都将被代理。如果目标对象未实现任何接口，则将创建CGLIB代理。
如果要强制使用CGLIB代理（例如，代理为目标对象定义的每个方法，而不仅仅是那些由其接口实现的方法），您可以这样做。但是，cglib 有这些问题需要考虑：
- 无法advice final方法，因为它们无法覆盖。
- 您将在类路径上需要CGLIB 2二进制文件，而JDK可以使用动态代理。 Spring会在需要CGLIB时自动发出警告，并且在类路径中找不到CGLIB库类。
- 代理对象的构造函数将被调用两次。这是CGLIB代理模型的自然结果，其中为每个代理对象生成子类。对于每个代理实例，创建两个对象：实际代理对象和实现建议的子类实例。使用JDK代理时不会出现此行为。通常，两次调用代理类型的构造函数不是问题，因为通常只有赋值发生，并且构造函数中没有实现真正的逻辑。

## 理解aop代理机制
```
public class SimplePojo implements Pojo {

   public void foo() {
      // this next method invocation is a direct call on the 'this' reference
      this.bar();
   }
   
   public void bar() {
      // some logic...
   }
}
```

![upload successful](/images/pasted-210.png)

```
public class Main {

   public static void main(String[] args) {
   
      Pojo pojo = new SimplePojo();
      
      // this is a direct method call on the 'pojo' reference
      pojo.foo();
   }
}
```

![upload successful](/images/pasted-211.png)


```
public class Main {

   public static void main(String[] args) {
   
      ProxyFactory factory = new ProxyFactory(new SimplePojo());
      factory.addInterface(Pojo.class);
      factory.addAdvice(new RetryAdvice());

      Pojo pojo = (Pojo) factory.getProxy();
      
      // this is a method call on the proxy!
      pojo.foo();
   }
}
```

这里要理解的关键是Main类的main（..）中的客户端代码具有对代理的引用。这意味着对该对象引用的方法调用将是代理上的调用，因此代理将能够委托给与该特定方法调用相关的所有拦截器（通知）。但是，一旦调用最终到达目标对象，在这种情况下，SimplePojo引用，它可能对其自身进行的任何方法调用，例如this.bar（）或this.foo（），将被调用这个引用，而不是代理。这具有重要意义。这意味着自我调用不会导致与方法调用相关的建议有机会执行。

好的，那要做些什么呢？最好的方法（这里松散地使用最好的术语）是重构代码，以便不会发生自我调用。当然，这确实需要你做一些工作，但这是最好的，最少侵入性的方法。接下来的方法绝对是可怕的，而且我几乎要谨慎地指出它，因为它太可怕了。你可以（窒息！）通过这样做完全将你的类中的逻辑绑定到Spring AOP

**tips: this.的内部调用无法 进行代理**


### org.springframework.aop.framework.ProxyConfig#setProxyTargetClass
- 设置是否直接代理目标类，而不是仅代理特定的接口。 默认为“false”。
- 如果该目标类是接口，则将为给定接口创建JDK代理。 如果该目标类是任何其他类，则将为给定类创建CGLIB代理。
- 将其设置为“true”以强制代理TargetSource的公开目标类。 

注意：根据具体代理工厂的配置，如果未指定任何接口（并且未激活任何接口自动检测），则还将应用proxy-target-class行为。*so个人认为一般不需要设置为true*

## 编程方式使用@AspectJ 代理
```
// create a factory that can generate a proxy for the given target object
AspectJProxyFactory factory = new AspectJProxyFactory(targetObject); 

// add an aspect, the class must be an @AspectJ aspect
// you can call this as many times as you need with different aspects
factory.addAspect(SecurityManager.class);

// you can also add existing aspect instances, the type of the object supplied must be an @AspectJ aspect
factory.addAspect(usageTracker);	

// now get the proxy object...
MyInterfaceType proxy = factory.getProxy();
```