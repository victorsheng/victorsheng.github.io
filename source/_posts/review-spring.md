title: review_spring
tags:
  - spring
categories:
  - spring
date: 2018-01-19 23:20:53
---
# Spring的优点

- 轻量级：相较于EJB容器，Spring采用的IoC容器非常的轻量级，基础版本的Spring框架大约只有2MB。Spring可以让开发者们仅仅使用POJO(Plain Old Java Object，相对于EJB)就能够开发出企业级的应用。这样做的好处是，你不需要使用臃肿庞大的 EJB容器(应用服务器)，你只需要轻量的servlet容器(如Tomcat)。尤其在一些开发当中，很稀缺内存和CPU资源时，采用Spring比EJB无论是开发还是部署应用都更节约资源。
- 控制反转(IOC)：Spring使用控制反转技术实现了松耦合。依赖被注入到对象，而不是创建或寻找依赖对象。
- 面向切面编程(AOP)： Spring支持面向切面编程，同时把应用的业务逻辑与系统的服务分离开来。
- MVC框架：Spring MVC是一个非常好的MVC框架，可以替换其他web框架诸如Struts。
- 集成性：Spring非常容易和其他的流行框架一起集成开发，这些框架包括：ORM框架，logging框架，JEE, Quartz，以及Struts等表现层框架。
- 事务管理：Spring强大的事务管理功能，能够处理本地事务(一个数据库)或是全局事务(多个数据，采用JTA)。
- 模块分离：Spring框架是由模块构成的。虽然已经有太多的包和类了，但它们都按照模块分好类了，你只需要考虑你会用到的模块，而不用理其他的模块。
- 异常处理：由于Java的JDBC，Hibernate等API中有很多方法抛出的是checked exception，而很多开发者并不能很好的处理异常。Spring提供了统一的API将这些checked exception的异常转换成Spring的unchecked exception。
- 单元测试：Spring写出来的代码非常容易做单元测试，可以采用依赖注射(Dependency Injection)将测试的数据注射到程序中。


# Spring的IOC(将一个类实例化成对象的技术)
IOC技术第一个解释叫做控制反转(IOC)，它还有个解释就是依赖注入(DI)，这两个名字很难从字面理解，但是当你理解它的原理后就会发现它们的描述是何等准确。IOC技术的本质就是构建对象的技术换句话说就是将一个类实例化成对象的技术。

耦合具有两面性。一方面，紧密耦合的代码难以测试，难以复用，难以理解，并且表现出“打地鼠”式的bug特性（修复一个bug，导致出现一个新的或者甚至更多的bug）。另一方面，一定程度的耦合又是必须的，完全没有耦合的代码什么也做不了。为了完成有实际意义的工作，不同的类必须以适当的方式进行交互。总而言之，耦合是必须的，但应当小心谨慎的管理它。通过控制反转或者依赖注入，对象的依赖关系将由负责协调系统中各个对象的第三方组件在创建对象时设定。对象无需自行创建或管理它们的依赖关系，依赖关系将被自动注入到需要它们的对象中去。

我们通过实际生活中的一个例子来解释一下IOC：

例如我们有个roo对象作用是完成打猎的操作，那么打猎这个对象内部包含两个辅助对象：人和枪，只有人和枪赋予了打猎这个对象，那么打猎对象才能完成打猎的操作，但是构建一个人和枪的对象并不是看起来那么简单，这里以枪为例，要创造一把枪我们需要金属，需要机床，需要子弹，而机床和子弹又是两个新对象，这些对象一个个相互嵌套相互关联，大伙试想下如果我们在java代码里构建一个枪的对象那是何其的复杂，假如我们要构造的不是简单的枪对象而是更加复杂的航空母舰，那么构造这个对象的成本之高是让人难以想象的，怎么来消除这种对象相互嵌套相互依赖的关系了？

Spring提供了一种方式，这种方式就是Spring提供一个容器，我们在xml文件里定义各个对象的依赖关系，由容器完成对象的构建，当我们Java代码里需要使用某个实例的时候就可以从容器里获取，那么对象的构建操作就被Spring容器接管，所以它被称为控制反转。

<font color=#0099ff size=5 face="黑体">控制反转的意思就是本来属于java程序里构建对象的功能交由容器接管，依赖注入就是当程序要使用某个对象时候，容器会把它注入到程序里。</font>在Java开发里我们想使用某个类提供的功能，有两种方式：一种就是构造一个新的类，新的类继承该类，另一种方式则是将某个类定义在新类里，那么两个类之间就建立一种关联关系，Spring的IOC容器就是实现了这种关联关系。

通过上面这段内容，相信大家应该会对IOC有个比较清晰的了解了。关于Spring中IOC部分是如何实现以及使用，就不进行讨论了，本文的目的也是能从整体上把握一下。但可以稍微提一点，<font color=#0099ff size=5 face="黑体">Spring的IOC功能其实就是依赖于Java的反射机制</font>。直白点说，当你在xml文件中配置好了bean（bean需要提供全类名）之后，Spring通过自己的类对xml文件进行解析，然后利用反射机制将对象创建出来，然后放到自己的数据结构中，比如Map。然后键就是bean中的id属性的值，值就是创建的对象。

# Spring的AOP

在设计模式里有一种代理模式，代理模式将继承模式和关联模式结合在一起使用，代理模式就是继承模式和关联模式的综合体，不过这个综合体的作用倒不是解决对象注入的问题，而是为具体操作对象找到一个保姆或者是秘书，这就和小说里的二号首长一样，这个二号首长对外代表了具体的实例对象，实例对象的入口和出口都是通过这个二号首长，具体的实例对象是一号首长，一号首长是要干大事的，所以一些事务性，重复性的工作例如泡茶，安排车子，这样的工作是不用劳烦一号首长的大驾，而是二号首长帮忙解决的，这就是AOP的思想。AOP解决程序开发里事务性，和核心业务无关的问题，但这些问题对于业务场景的实现是很有必要的，在实际开发里AOP也是节省代码的一种方式。

AOP将应用系统分为两部分，核心业务逻辑（Core business concerns）及横向的通用逻辑，也就是所谓的方面Crosscutting enterprise concerns，例如，所有大中型应用都要涉及到的持久化管理（Persistent）、事务管理（Transaction Management）、安全管理（Security）、日志管理（Logging）和调试管理（Debugging）等。

实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。


## AOP术语

- Join Point: Spring AOP中，join point就是一个方法。（通俗来讲就是起作用的那个方法）。

- Pointcut: 用来指定join point（通俗来讲就是描述的一组符合某个条件的join point）。通常使用pointcut表达式来限定joint point，Spring默认使用AspectJ pointcut expression language。

- Advice: 在join point上特定的时刻执行的操作，Advice有几种不同类型，下文将会讨论（通俗地来讲就是起作用的内容和时间点）。

- Introduction：给对象增加方法或者属性。

- Target object: Advice起作用的那个对象。

- AOP proxy: 为实现AOP所生成的代理。在Spring中有两种方式生成代理:JDK代理和CGLIB代理。

- Aspect: 组合了Pointcut与Advice，在Spring中有时候也称为Advisor。某些资料说Advisor是一种特殊的Aspect，其区别是Advisor只能包含一对pointcut和advice，但是aspect可以包含多对。AOP中的aspect可以类比于OOP中的class。

- Weaving：将Advice织入join point的这个过程。


## 静态代理
静态代理是编译阶段生成AOP代理类，也就是说生成的字节码就织入了增强后的AOP对象；动态代理则不会修改字节码，而是在内存中临时生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。
实现静态代理常用的方法是通过AspectJ，AspectJ 是一个基于 Java 语言的 AOP 框架，提供了强大的 AOP 功能，主要包含两个部分，第一个部分定义了如何表达、定义 AOP 编程中的语法规范；另一个部分是工具部分，包括编译器、调试工具等。
Aspectj实现了独有的编译器，在编译时生成代理类文件


## Spring的AOP代理对象的生成

spring支持AspectJ风格的AOP还是动态的，标注中用到的JoinPoint等类都来自aspectj包

Spring提供了两种方式来生成代理对象: JDKProxy和Cglib，具体使用哪种方式生成由AopProxyFactory根据AdvisedSupport对象的配置来决定。默认的策略是如果目标类是接口，则使用JDK动态代理技术，否则使用Cglib来生成代理。


```
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class SimpleAspect {
	@Pointcut("execution(* cn.outofmemory.spring_aop_aspect.*Service*.*(..))")
	public void pointCut() {
	}

	@After("pointCut()")
	public void after(JoinPoint joinPoint) {
		System.out.println("after aspect executed");
	}

	@Before("pointCut()")
	public void before(JoinPoint joinPoint) {
		//如果需要这里可以取出参数进行处理
		//Object[] args = joinPoint.getArgs();
		System.out.println("before aspect executing");
	}

	@AfterReturning(pointcut = "pointCut()", returning = "returnVal")
	public void afterReturning(JoinPoint joinPoint, Object returnVal) {
		System.out.println("afterReturning executed, return result is "
				+ returnVal);
	}

	@Around("pointCut()")
	public void around(ProceedingJoinPoint pjp) throws Throwable {
		System.out.println("around start..");
		try {
			pjp.proceed();
		} catch (Throwable ex) {
			System.out.println("error in around");
			throw ex;
		}
		System.out.println("around end");
	}

	@AfterThrowing(pointcut = "pointCut()", throwing = "error")
	public void afterThrowing(JoinPoint jp, Throwable error) {
		System.out.println("error:" + error);
	}
}
```


## jdkProxy

JDK动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。

核心是InvocationHandler接口和Proxy类。
- java.lang.reflect.Proxy，这是Java动态代理机制生成的所有动态代理类的父类，它提供静态方法来为一组接口动态地生成代理类及其对象。
- java.lang.reflect.InvocationHandler，这是调用处理器接口，它自定义了一个invoke方法，用于实现代理逻辑、转发请求给对原始类对象。

继承了Proxy类，实现了代理的接口，由于java不能多继承，这里已经继承了Proxy类了，不能再继承其他的类，所以JDK的动态代理不支持对实现类的代理，只支持接口的代理

### 实现

```
private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>>
    {
        // prefix for all proxy class names
        private static final String proxyClassNamePrefix = "$Proxy";

        // next number to use for generation of unique proxy class names
        private static final AtomicLong nextUniqueNumber = new AtomicLong();

        @Override
        public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

            Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
            for (Class<?> intf : interfaces) {
                /*
                 * Verify that the class loader resolves the name of this
                 * interface to the same Class object.
                 */
                Class<?> interfaceClass = null;
                try {
                    interfaceClass = Class.forName(intf.getName(), false, loader);
                } catch (ClassNotFoundException e) {
                }
                if (interfaceClass != intf) {
                    throw new IllegalArgumentException(
                        intf + " is not visible from class loader");
                }
                /*
                 * Verify that the Class object actually represents an
                 * interface.
                 */
                if (!interfaceClass.isInterface()) {
                    throw new IllegalArgumentException(
                        interfaceClass.getName() + " is not an interface");
                }
                /*
                 * Verify that this interface is not a duplicate.
                 */
                if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                    throw new IllegalArgumentException(
                        "repeated interface: " + interfaceClass.getName());
                }
            }

            String proxyPkg = null;     // package to define proxy class in
            int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

            /*
             * Record the package of a non-public proxy interface so that the
             * proxy class will be defined in the same package.  Verify that
             * all non-public proxy interfaces are in the same package.
             */
            for (Class<?> intf : interfaces) {
                int flags = intf.getModifiers();
                if (!Modifier.isPublic(flags)) {
                    accessFlags = Modifier.FINAL;
                    String name = intf.getName();
                    int n = name.lastIndexOf('.');
                    String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                    if (proxyPkg == null) {
                        proxyPkg = pkg;
                    } else if (!pkg.equals(proxyPkg)) {
                        throw new IllegalArgumentException(
                            "non-public interfaces from different packages");
                    }
                }
            }

            if (proxyPkg == null) {
                // if no non-public proxy interfaces, use com.sun.proxy package
                proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
            }

            /*
             * Choose a name for the proxy class to generate.
             */
            long num = nextUniqueNumber.getAndIncrement();
            String proxyName = proxyPkg + proxyClassNamePrefix + num;

            /*
             * Generate the specified proxy class.
             */
            byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces, accessFlags);
            try {
                return defineClass0(loader, proxyName,
                                    proxyClassFile, 0, proxyClassFile.length);
            } catch (ClassFormatError e) {
                /*
                 * A ClassFormatError here means that (barring bugs in the
                 * proxy class generation code) there was some other
                 * invalid aspect of the arguments supplied to the proxy
                 * class creation (such as virtual machine limitations
                 * exceeded).
                 */
                throw new IllegalArgumentException(e.toString());
            }
        }
    }
```

```
apply:639, Proxy$ProxyClassFactory {java.lang.reflect}
apply:557, Proxy$ProxyClassFactory {java.lang.reflect}
get:230, WeakCache$Factory {java.lang.reflect}
get:127, WeakCache {java.lang.reflect}
getProxyClass0:419, Proxy {java.lang.reflect}
newProxyInstance:719, Proxy {java.lang.reflect}
main:11, ProxyTest {com.example.demo.proxy.jdk}
```

获取的对象是$Proxy对象,里面持有了真正的原始对象

## cglib
核心是MethodInterceptor接口和Enhancer类
对于final方法，无法进行代理(不报错,正常执行原方法)


```

public class CglibProxy implements MethodInterceptor {
    private Enhancer enhancer = new Enhancer();
    public Object getProxy(Class clazz){
        //设置需要创建子类的类
        enhancer.setSuperclass(clazz);
        enhancer.setCallback(this);
        //通过字节码技术动态创建子类实例
        return enhancer.create();
    }
    //实现MethodInterceptor接口方法
    public Object intercept(Object obj, Method method, Object[] args,
                            MethodProxy proxy) throws Throwable {
        System.out.println("前置代理");
        //通过代理类调用父类中的方法
        Object result = proxy.invokeSuper(obj, args);
        System.out.println("后置代理");
        return result;
    }
}


public class DoCGLib {
    public static void main(String[] args) {
    CglibProxy proxy = new CglibProxy();
    //通过生成子类的方式创建代理类
    HelloWorldImpl proxyImp = (HelloWorldImpl)proxy.getProxy(HelloWorldImpl.class);
    proxyImp.sayHello("wing");
}
}

```

# spring生命周期

- 1.Spring对Bean进行实例化（相当于程序中的new Xx()）
- 2.Spring将值和Bean的引用注入进Bean对应的属性中
- 3.如果Bean实现了BeanNameAware接口，Spring将Bean的ID传递给setBeanName()方法（实现BeanNameAware清主要是为了通过Bean的引用来获得Bean的ID，一般业务中是很少有用到Bean的ID的）
- 4.如果Bean实现了BeanFactoryAware接口，Spring将调用setBeanDactory(BeanFactory bf)方法并把BeanFactory容器实例作为参数传入。（实现BeanFactoryAware 主要目的是为了获取Spring容器，如Bean通过Spring容器发布事件等）
- 5.如果Bean实现了ApplicationContextAwaer接口，Spring容器将调用setApplicationContext(ApplicationContext ctx)方法，把y应用上下文作为参数传入.(作用与BeanFactory类似都是为了获取Spring容器，不同的是Spring容器在调用setApplicationContext方法时会把它自己作为setApplicationContext 的参数传入，而Spring容器在调用setBeanDactory前需要程序员自己指定（注入）setBeanDactory里的参数BeanFactory )
- 6.如果Bean实现了BeanPostProcess接口，Spring将调用它们的postProcessBeforeInitialization（预初始化）方法（作用是在Bean实例创建成功后对进行增强处理，如对Bean进行修改，增加某个功能）
- 7.如果Bean实现了InitializingBean接口，Spring将调用它们的afterPropertiesSet方法，作用与在配置文件中对Bean使用init-method声明初始化的作用一样，都是在Bean的全部属性设置成功后执行的初始化方法。
- 8.如果Bean实现了BeanPostProcess接口，Spring将调用它们的postProcessAfterInitialization（后初始化）方法（作用与6的一样，只不过6是在Bean初始化前执行的，而这个是在Bean初始化后执行的，时机不同 )
- 9.经过以上的工作后，Bean将一直驻留在应用上下文中给应用使用，直到应用上下文被销毁
- 10.如果Bean实现了DispostbleBean接口，Spring将调用它的destory方法，作用与在配置文件中对Bean使用destory-method属性的作用一样，都是在Bean实例销毁前执行的方法。


-----
Bean的完整生命周期经历了各种方法调用，这些方法可以划分为以下几类：

1、Bean自身的方法　　：　　这个包括了Bean本身调用的方法和通过配置文件中<bean>的init-method和destroy-method指定的方法

2、Bean级生命周期接口方法　　：　　这个包括了BeanNameAware、BeanFactoryAware、InitializingBean和DiposableBean这些接口的方法

3、容器级生命周期接口方法　　：　　这个包括了InstantiationAwareBeanPostProcessor 和 BeanPostProcessor 这两个接口实现，一般称它们的实现类为“后处理器”。

4、工厂后处理器接口方法　　：　　这个包括了AspectJWeavingEnabler, ConfigurationClassPostProcessor, CustomAutowireConfigurer等等非常有用的工厂后处理器　　接口的方法。工厂后处理器也是容器级的。在应用上下文装配配置文件之后立即调用。


# 由AbstractBeanFactory控制
## Aware接口

```
	private void invokeAwareMethods(final String beanName, final Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof BeanNameAware) {
				((BeanNameAware) bean).setBeanName(beanName);
			}
			if (bean instanceof BeanClassLoaderAware) {
				((BeanClassLoaderAware) bean).setBeanClassLoader(getBeanClassLoader());
			}
			if (bean instanceof BeanFactoryAware) {
				((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
			}
		}
	}
```

## BeanPostProcessor接口的postProcessBeforeInitialization方法


## InitializingBean接口


## BeanPostProcessor接口的postProcessAfterInitialization方法

## DisposableBean接口


![upload successful](/images/pasted-45.png)

![upload successful](/images/pasted-46.png)

https://www.cnblogs.com/zrtqsk/p/3735273.html


-----
# springbean是否线程安全

我们交由Spring管理的大多数对象其实都是一些无状态的对象，这种不会因为多线程而导致状态被破坏的对象很适合Spring的默认scope，每个单例的无状态对象都是线程安全的（也可以说只要是无状态的对象，不管单例多例都是线程安全的，不过单例毕竟节省了不断创建对象与GC的开销）。

无状态的对象即是自身没有状态的对象，自然也就不会因为多个线程的交替调度而破坏自身状态导致线程安全问题。无状态对象包括我们经常使用的DO、DTO、VO这些只作为数据的实体模型的贫血对象，还有Service、DAO和Controller，这些对象并没有自己的状态，它们只是用来执行某些操作的。例如，每个DAO提供的函数都只是对数据库的CRUD，而且每个数据库Connection都作为函数的局部变量（局部变量是在用户栈中的，而且用户栈本身就是线程私有的内存区域，所以不存在线程安全问题），用完即关（或交还给连接池）。

![upload successful](/images/pasted-36.png)