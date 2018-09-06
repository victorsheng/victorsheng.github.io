title: geektime-java-core-36-spring-bean
date: 2018-08-26 14:30:13
tags:
categories:
---
# 谈谈 Spring Bean 的生命周期和作用域？

Spring Bean 生命周期比较复杂，可以分为创建和销毁两个过程。

* 实例化 Bean 对象。
* 设置 Bean 属性。
* 如果我们通过各种 Aware 接口声明了依赖关系，则会注入 Bean 对容器基础设施层面的依赖。具体包括 BeanNameAware、BeanFactoryAware 和 ApplicationContextAware，分别会注入 Bean ID、Bean Factory 或者 ApplicationContext。
* 调用 BeanPostProcessor 的前置初始化方法 postProcessBeforeInitialization。
* 如果实现了 InitializingBean 接口，则会调用 afterPropertiesSet 方法。
* 调用 Bean 自身定义的 init 方法。
* 调用 BeanPostProcessor 的后置初始化方法 postProcessAfterInitialization。
* 创建过程完毕。


Spring Bean 的销毁过程会依次调用 DisposableBean 的 destroy 方法和 Bean 自身定制的 destroy 方法。

## Spring Bean 有五个作用域
* Singleton，这是 Spring 的默认作用域，也就是为每个 IOC 容器创建唯一的一个 Bean 实例。
* Prototype，针对每个 getBean 请求，容器都会单独创建一个 Bean 实例。
如果是 Web 容器，则支持另外三种作用域：
* Request，为每个 HTTP 请求创建单独的 Bean 实例。
* Session，很显然 Bean 实例的作用域是 Session 范围。
* GlobalSession，用于 Portlet 容器，因为每个 Portlet 有单独的 Session，GlobalSession 提供一个全局性的 HTTP Session。

**Bean 的生命周期是完全被容器所管理的，从属性设置到各种依赖关系，都是容器负责注入，并进行各个阶段其他事宜的处理，Spring 容器为应用开发者定义了清晰的生命周期沟通界面。**

# Spring 的基础机制。
至少你需要理解下面两个基本方面。
* 控制反转（Inversion of Control），或者也叫依赖注入（Dependency Injection），广泛应用于 Spring 框架之中，可以有效地改善了模块之间的紧耦合问题。
从 Bean 创建过程可以看到，它的依赖关系都是由容器负责注入，具体实现方式包括带参数的构造函数、setter 方法或者AutoWired方式实现。
* AOP，我们已经在前面接触过这种切面编程机制，Spring 框架中的事务、安全、日志等功能都依赖于 AOP 技术，下面我会进一步介绍。


# Spring 框架的涵盖范围。
我前面谈到的 Spring，其实是狭义的Spring Framework，其内部包含了依赖注入、事件机制等核心模块，也包括事务、O/R Mapping 等功能组成的数据访问模块，以及 Spring MVC 等 Web 框架和其他基础组件。
广义上的 Spring 已经成为了一个庞大的生态系统，例如：
* Spring Boot，通过整合通用实践，更加自动、智能的依赖管理等，Spring Boot 提供了各种典型应用领域的快速开发基础，所以它是以应用为中心的一个框架集合。
* Spring Cloud，可以看作是在 Spring Boot 基础上发展出的更加高层次的框架，它提供了构建分布式系统的通用模式，包含服务发现和服务注册、分布式配置管理、负载均衡、分布式诊断等各种子系统，可以简化微服务系统的构建。
* 当然，还有针对特定领域的 Spring Security、Spring Data 等。
上面的介绍比较笼统，针对这么多内容，如果将目标定得太过宽泛，可能就迷失在 Spring 生态之中，我建议还是深入你当前使用的模块，如 Spring MVC。并且，从整体上把握主要前沿框架（如 Spring Cloud）的应用范围和内部设计，至少要了解主要组件和具体用途，毕竟如何构建微服务等，已经逐渐成为 Java 应用开发面试的热点之一。

# Spring AOP 自身设计的一些细节

## 切面编程
切面编程落实到软件工程其实是为了更好地模块化，而不仅仅是为了减少重复代码。通过 AOP 等机制，我们可以把横跨多个不同模块的代码抽离出来，让模块本身变得更加内聚，进而业务开发者可以更加专注于业务逻辑本身。从迭代能力上来看，我们可以通过切面的方式进行修改或者新增功能，这种能力不管是在问题诊断还是产品能力扩展中，都非常有用。
在之前的分析中，我们已经分析了 AOP Proxy 的实现原理，简单回顾一下，它底层是基于 JDK 动态代理或者 cglib 字节码操纵等技术，运行时动态生成被调用类型的子类等，并实例化代理对象，实际的方法调用会被代理给相应的代理对象。但是，这并没有解释具体在 AOP 设计层面，什么是切面，如何定义切入点和切面行为呢？

Spring AOP 引入了其他几个关键概念：
* Aspect，通常叫作方面，它是跨不同 Java 类层面的横切性逻辑。在实现形式上，既可以是 XML 文件中配置的普通类，也可以在类代码中用“@Aspect”注解去声明。在运行时，Spring 框架会创建类似Advisor来指代它，其内部会包括切入的时机（Pointcut）和切入的动作（Advice）。
* Join Point，它是 Aspect 可以切入的特定点，在 Spring 里面只有方法可以作为 Join Point。
* Advice，它定义了切面中能够采取的动作。如果你去看 Spring 源码，就会发现 Advice、Join Point 并没有定义在 Spring 自己的命名空间里，这是因为他们是源自AOP 联盟，可以看作是 Java 工程师在 AOP 层面沟通的通用规范。

```
@Aspect
public class SignAdvice {

  @Pointcut("@annotation(Sign)")
  private void aspectjMethod() {}


  @Around(value = "aspectjMethod()")
  public Object aroundAdvice(ProceedingJoinPoint point) throws Throwable {
    Object proceed = point.proceed(args);
    return proceed;
  }



}



```

- BeforeAdvice
- AfterAdvice
- Around


![upload successful](/images/pasted-232.png)



![upload successful](/images/pasted-233.png)

* Join Point 仅仅是可利用的机会。
* Pointcut 是解决了切面编程中的 Where 问题，让程序可以知道哪些机会点可以应用某个切面动作。
* 而 Advice 则是明确了切面编程中的 What，也就是做什么；同时通过指定 Before、After 或者 Around，定义了 When，也就是什么时候做。


# 讨论
Spring容器初始化开始:
1.[BeanFactoryPostProcessor]接口实现类的构造器2.[BeanFactoryPostProcessor]的postProcessorBeanFactory方法
3.[BeanPostProcessor]接口实现类的构造器
4.[InstantiationAwareBeanPostProcessorAdapter]构造器
5.[InstantiationAwareBeanPostProcessorAdapter]的postProcessBeforeInstantiation方法(从这里开始初始化bean)
6.[Bean]的构造器
7.[InstantiationAwareBeanPostProcessorAdapter]的postProcessAfterInstantiation
8.[InstantiationAwareBeanPostProcessorAdapter]的postProcessPropertyValues方法
9.[Bean]属性注入，setter方
方法
10.[Bean]如果实现了各种XXXaware接口，依次调用各个setXXX(如BeanNameAware.setBeanName(),BeanFactoryAware.setBeanFactory())
11.[BeanPostProcessor]的postProcessBeforeInitialization方法
12.[InstantiationAwareBeanPostProcessorAdapter]的postProcessBeforeInitialization方法
13.[Bean]自定义的init-method
14.[Bean]如果实现了InitializingBean接口，此时会调用它的afterPropertiesSet方法
15.[BeanPostProcessor]的postProcessAfterInitialization方法(此时bean初始化完成)
16.[InstantiationAwareBeanPostProcessorAdapter]的postProcessInitialization方法(到这里容器初始化完成)
17.业务逻辑bean的使用

Bean的销毁过程:
1.[DisposableBean]的destory方法
2.[Bean]自定义的destory-method方法

说明:如果有多个bean需要初始化，会循环执行5--15。


概括来说主要有四个阶段：实例化，初始化，使用，销毁。

# 自己补充
BeanFactory和ApplicationContext是Spring中两种很重要的容器，
## 前者提供了最基本的依赖注入的支持
## 后者在继承前者的基础上进行了功能的拓展，增加了事件传播，资源访问，国际化的支持等功能。同时两者的生命周期也稍微有些不同。
* MessageSource, 提供国际化的消息访问  
* 资源访问，如URL和文件  
* 事件传播  
* 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层  


1.利用MessageSource进行国际化  
   BeanFactory是不支持国际化功能的，因为BeanFactory没有扩展Spring中MessageResource接口。相反，由于ApplicationContext扩展了MessageResource接口，因而具有消息处理的能力(i18N)，具体spring如何使用国际化，以后章节会详细描述。  

2.强大的事件机制(Event)  
   基本上牵涉到事件(Event)方面的设计，就离不开观察者模式。不明白观察者模式的朋友，最好上网了解下。因为，这种模式在java开发中是比较常用的，又是比较重要的。  
ApplicationContext的事件机制主要通过ApplicationEvent和ApplicationListener这两个接口来提供的，和java swing中的事件机制一样。即当ApplicationContext中发布一个事件的时，所有扩展了ApplicationListener的Bean都将会接受到这个事件，并进行相应的处理。  

3.底层资源的访问  
    ApplicationContext扩展了ResourceLoader(资源加载器)接口，从而可以用来加载多个Resource，而BeanFactory是没有扩展ResourceLoader  

4.对Web应用的支持  
   与BeanFactory通常以编程的方式被创建不同的是，ApplicationContext能以声明的方式创建，如使用ContextLoader。当然你也可以使用ApplicationContext的实现之一来以编程的方式创建ApplicationContext实例 。  

5.其它区别  
  1).BeanFactroy采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化，这样，我们就不能发现一些存在的Spring的配置问题。而ApplicationContext则相反，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误。  

  2).BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册


# FactoryBean
我们在配置spring的时候有两种bean可以配置，

一种是普通bean：配置的class就是生成的bean的真正对象。

一种是工厂Bean，即FactoryBean：配置class不是生成bean的真正对象。

他们都被spring容器管理，工厂Bean其返回的对象不是指定类的一个实例，是该FactoryBean的getObject方法所返回的对象。

应用：它们在很多应用如(Spring的AOP、ORM、事务管理)及与其它第三框架(ehCache)集成时都有体现，

## FactoryBean接口
```
package org.springframework.beans.factory;

public interface FactoryBean<T> {

    T getObject() throws Exception;
    
    Class<?> getObjectType();
 
    boolean isSingleton();

}
```
其中是java泛型机制实现，即T表示要生成的对象。也可以不使用，推荐使用。
getObject()方法返回欲生成对象。getObjectType()欲返回对象类型,isSingleton()获得对象的机制(singleton|prototype|request|session).

## 例子:mybatis和spring集成的SqlSessionFactoryBean
```
  <!-- MyBatis -->
  <bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
    <property name="dataSource" ref="opalAPDataSource"/>
    <property name="mapperLocations"
      value="classpath*:mappers/*Mapper.xml"/>
  </bean>
```
容器通过getBean方法返回的不是SqlSessionFactory本身，而是FactoryBean实现类中getObject()方法返回的对象。 

## 例子:Spring集成 ehcache的EhCacheManagerFactoryBean
```
<bean id="cacheManager" class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">    
    <property name="configLocation">    
        <value>classpath:ehcache.xml</value>    
    </property>    
</bean>
<bean id="levelOneCache" class="org.springframework.cache.ehcache.EhCacheFactoryBean">    
    <property name="cacheManager">    
        <ref local="cacheManager" />    
    </property>    
    <property name="cacheName">    
        <value>levelOneCache</value>    
    </property>    
</bean>
```