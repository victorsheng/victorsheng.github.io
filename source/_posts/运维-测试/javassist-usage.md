---
title: javassist在easy-mapper和dubbo项目中的应用
abbrlink: 3210990680
date: 2018-05-03 20:38:58
tags:
categories:
---
# 简介
Javassist是一个开源的分析、编辑和创建Java字节码的类库。是由东京工业大学的数学和计算机科学系的 Shigeru Chiba （千叶 滋）所创建的。它已加入了开放源代码JBoss 应用服务器项目,通过使用Javassist对字节码操作为JBoss实现动态"AOP"框架。

Javassist的官方网站：http://jboss-javassist.github.io/javassist/
通过javasssit，我们可以：
- 动态创建新类或新接口的二进制字节码
- 动态扩展现有类或接口的二进制字节码(AOP)

# 入门程序
## 生成新的class

```
package com.example.demo.javassist;

import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtConstructor;
import javassist.CtField;
import javassist.CtMethod;
import javassist.CtNewMethod;
import javassist.Modifier;

/**
 * package com.tianshouzhi;
 *
 * public class User {
 *     private String name;
 *
 *     public User(String name) {
 *         this.name = name;
 *     }
 *
 *     public User() {
 *     }
 *
 *     public String getName() {
 *         return name;
 *     }
 *
 *     public void setName(String name) {
 *         this.name = name;
 *     }
 *
 *     @Override
 *     public String toString() {
 *         return "name="+name;
 *     }
 * }
 */
public class UserGenerator {
  public static void main(String[] args) throws Exception {
    ClassPool classPool = ClassPool.getDefault();
    //定义User类
    CtClass ctClassUser = classPool.makeClass("com.tianshouzhi.User");

    //定义name字段
    CtClass fieldType = classPool.get("java.lang.String");//字段类型
    String name = "name";//字段名称
    CtField ctFieldName=new CtField(fieldType, name,ctClassUser);
    ctFieldName.setModifiers(Modifier.PRIVATE);//设置访问修饰符
    ctClassUser.addField(ctFieldName, CtField.Initializer.constant("javasssit"));//添加name字段，赋值为javassist

    //定义构造方法
    CtClass[] parameters = new CtClass[]{classPool.get("java.lang.String")};//构造方法参数
    CtConstructor constructor=new CtConstructor(parameters,ctClassUser);
    String body = "{this.name=$1;}";//方法体 $1表示的第一个参数
    constructor.setBody(body);
    ctClassUser.addConstructor(constructor);

    //setName getName方法
    ctClassUser.addMethod(CtNewMethod.setter("setName",ctFieldName));
    ctClassUser.addMethod(CtNewMethod.getter("getName",ctFieldName));

    //toString方法
    CtClass returnType = classPool.get("java.lang.String");
    String methodName = "toString";
    CtMethod toStringMethod=new CtMethod(returnType, methodName, null,ctClassUser);
    toStringMethod.setModifiers(Modifier.PUBLIC);
    String methodBody = "{return \"name=\"+$0.name;}";//$0表示的是this
    toStringMethod.setBody(methodBody);
    ctClassUser.addMethod(toStringMethod);

    //代表class文件的CtClass创建完成，现在将其转换成class对象
    Class clazz = ctClassUser.toClass();
    Constructor cons = clazz.getConstructor(String.class);
    Object user = cons.newInstance("wangxiaoxiao");
    Method toString = clazz.getMethod("toString");
    System.out.println(toString.invoke(user));

    ctClassUser.writeFile(".");//在当前目录下，生成com/tianshouzhi/User.class文件
  }
}

```

## 修改class

```
package com.example.demo.javassist;

public class Looper {

  public void loop(){
    try {
      System.out.println("Looper.loop() invoked");
      Thread.sleep(1000L);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }


}

```
现在我们想统计Looper的loop方法的耗时时间。最简单的思路是使用javassist修改loop方法的源码：

在最前加入：long start=System.currentTimeMillis();

在最后插入：System.out.println("耗时:"+(System.currentTimeMillis()-start)+"ms");

将原有的loop方法名改为loop$impl，然后再定义一个loop方法，新的loop方法内部会调用loop$impl，在调用之前和调用之后分别加入上述的代码片段。
```
package com.example.demo.javassist;

import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtMethod;
import javassist.CtNewMethod;

public class JavassisTiming {
  public static void main(String[] args) throws Exception{
    //需要修改的已有的类名和方法名
    String className="com.example.demo.javassist.Looper";
    String methodName="loop";

    //修改为原有类的方法名为loop$impl
    CtClass clazz = ClassPool.getDefault().get(className);
    CtMethod method = clazz.getDeclaredMethod(methodName);
    String newname = methodName + "$impl";
    method.setName(newname);

    //使用原始方法名loop，定义一个新方法，在这个方法内部调用loop$impl
    CtMethod newMethod = CtNewMethod.make("public void "+methodName+"(){" +
            "long start=System.currentTimeMillis();" +
            ""+newname+"();" +//调用loop$impl
            "System.out.println(\"耗时:\"+(System.currentTimeMillis()-start)+\"ms\");" +
            "}"
        , clazz);
    clazz.addMethod(newMethod);

    //调用修改的Looper类的loop方法
    Looper looper = (Looper) clazz.toClass().newInstance();
    looper.loop();
  }
}

```

## aop

```
package com.example.demo.javassist;

import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtMethod;
import javassist.CtNewMethod;

public class JavassisTiming {
  public static void main(String[] args) throws Exception{
    //需要修改的已有的类名和方法名
    String className="com.example.demo.javassist.Looper";
    String methodName="loop";

    //修改为原有类的方法名为loop$impl
    CtClass clazz = ClassPool.getDefault().get(className);
    CtMethod method = clazz.getDeclaredMethod(methodName);
    String newname = methodName + "$impl";
    method.setName(newname);

    //使用原始方法名loop，定义一个新方法，在这个方法内部调用loop$impl
    CtMethod newMethod = CtNewMethod.make("public void "+methodName+"(){" +
            "long start=System.currentTimeMillis();" +
            ""+newname+"();" +//调用loop$impl
            "System.out.println(\"耗时:\"+(System.currentTimeMillis()-start)+\"ms\");" +
            "}"
        , clazz);
    clazz.addMethod(newMethod);

    //调用修改的Looper类的loop方法
    Looper looper = (Looper) clazz.toClass().newInstance();
    looper.loop();
  }
}

```


# Javassist在easy-mapper中的应用

使用Javassist字节码增强技术，在运行时动态生成mapping过程的源代码

类似的组件:
- Apache PropertyUtils
- Apache BeanUtils
- Cglib BeanCopier
- Spring BeanUtils
- Dozer

教程:
http://neoremind.com/2016/08/easy-mapper-%E4%B8%80%E4%B8%AA%E7%81%B5%E6%B4%BB%E5%8F%AF%E6%89%A9%E5%B1%95%E7%9A%84%E9%AB%98%E6%80%A7%E8%83%BDbean-mapping%E7%B1%BB%E5%BA%93/

github地址:
https://github.com/neoremind/easy-mapper

## 生成的java文件

```
package com.baidu.unbiz.generated

public class EasyMapper_AlgorithmBaseEntity_TO_FullAlgorithmEntity_Mapper23487587931824$1 extends GeneratedMapperBase {
	public void map(java.lang.Object a, java.lang.Object b) {

com.talkingdata.opal.algorithm.entity.AlgorithmBaseEntity source = ((com.talkingdata.opal.algorithm.entity.AlgorithmBaseEntity)a);
com.talkingdata.opal.algorithm.entity.request.FullAlgorithmEntity destination = ((com.talkingdata.opal.algorithm.entity.request.FullAlgorithmEntity)b);
if ( !(((java.lang.String)source.getDataset()) == null)){ 
destination.setDataset(((java.lang.String)source.getDataset())); 
}
if ( !(((java.lang.String)source.getId()) == null)){ 
destination.setId(((java.lang.String)source.getId())); 
}
if ( !(((java.lang.String)source.getIssId()) == null)){ 
destination.setIssId(((java.lang.String)source.getIssId())); 
}
if ( !(((java.lang.String)source.getLang()) == null)){ 
destination.setLang(((java.lang.String)source.getLang())); 
}
if ( !(((java.lang.String)source.getLangVersion()) == null)){ 
destination.setLangVersion(((java.lang.String)source.getLangVersion())); 
}
if ( !(((java.lang.Integer)source.getStatus()) == null)){ 
destination.setStatus(((java.lang.Integer)source.getStatus())); 
}
if ( !(((java.lang.String)source.getTitle()) == null)){ 
destination.setTitle(((java.lang.String)source.getTitle())); 
}
		if(customMapping != null) { 
			 customMapping.map(source, destination);
		}
	}

}
```

## 调用javassist的代码:
```
public Class<?> compileClass(SourceCodeContext sourceCode) throws MappingCodeGenerationException {
        StringBuilder className = new StringBuilder(sourceCode.getClassName());
        //二进制class
        CtClass byteCodeClass = classPool.makeClass(className.toString());

        Class<?> compiledClass;
        try {
            writeSourceFile(sourceCode);
            CtClass abstractMapperClass = classPool.get(sourceCode.getSuperClass().getCanonicalName());
            //父类
            byteCodeClass.setSuperclass(abstractMapperClass);

            for (String fieldDef : sourceCode.getFields()) {
                try {
                    //字段
                    byteCodeClass.addField(CtField.make(fieldDef, byteCodeClass));
                } catch (CannotCompileException e) {
                    LOGGER.error(
                            "An exception occurred while compiling: " + fieldDef + " for " + sourceCode.getClassName(),
                            e);
                    throw e;
                }
            }

            for (String methodDef : sourceCode.getMethods()) {
                try {
                    //方法
                    byteCodeClass.addMethod(CtNewMethod.make(methodDef, byteCodeClass));
                } catch (CannotCompileException e) {
                    LOGGER.error(
                            "An exception occurred while compiling the following method:\n\n " + methodDef + "\n\n for "
                                    + sourceCode.getClassName() + "\n", e);
                    throw e;
                }
            }

            compiledClass = byteCodeClass.toClass(Thread.currentThread().getContextClassLoader(),
                    this.getClass().getProtectionDomain());

            writeClassFile(byteCodeClass);
        } catch (CannotCompileException e) {
            throw new MappingCodeGenerationException(
                    String.format("Failed to compile class %s, reason is %s", className, e.getMessage()), e);
        } catch (NotFoundException e) {
            throw new MappingCodeGenerationException(
                    String.format("Failed to compile class %s, reason is %s", className, e.getMessage()), e);
        }
        return compiledClass;
    }
```
# Javassist在dubbo中的应用

## 生成代理
```
"main@1" prio=5 tid=0x1 nid=NA runnable
  java.lang.Thread.State: RUNNABLE
	  at com.alibaba.dubbo.common.bytecode.ClassGenerator.toClass(ClassGenerator.java:260)
	  at com.alibaba.dubbo.common.bytecode.ClassGenerator.toClass(ClassGenerator.java:256)
	  at com.alibaba.dubbo.common.bytecode.Wrapper.makeWrapper(Wrapper.java:237)
	  at com.alibaba.dubbo.common.bytecode.Wrapper.getWrapper(Wrapper.java:99)
	  at com.alibaba.dubbo.config.ServiceConfig.doExportUrlsFor1Protocol(ServiceConfig.java:444)
	  at com.alibaba.dubbo.config.ServiceConfig.doExportUrls(ServiceConfig.java:357)
	  at com.alibaba.dubbo.config.ServiceConfig.doExport(ServiceConfig.java:316)
	  - locked <0x7c0> (a com.alibaba.dubbo.config.spring.ServiceBean)
	  at com.alibaba.dubbo.config.ServiceConfig.export(ServiceConfig.java:215)
	  at com.alibaba.dubbo.config.spring.ServiceBean.onApplicationEvent(ServiceBean.java:121)
	  at com.alibaba.dubbo.config.spring.ServiceBean.onApplicationEvent(ServiceBean.java:50)
	  at org.springframework.context.event.SimpleApplicationEventMulticaster.invokeListener(SimpleApplicationEventMulticaster.java:167)
	  at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:139)
	  at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:393)
	  at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:347)
	  at org.springframework.context.support.AbstractApplicationContext.finishRefresh(AbstractApplicationContext.java:883)
	  at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:546)
	  - locked <0x859> (a java.lang.Object)
	  at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:139)
	  at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:93)
	  at com.alibaba.dubbo.demo.provider.Provider.main(Provider.java:27)
```

## 代理工厂
### 接口
```
@SPI("javassist")
public interface ProxyFactory {

    /**
     * create proxy.
     *
     * @param invoker
     * @return proxy
     */
    @Adaptive({Constants.PROXY_KEY})
    <T> T getProxy(Invoker<T> invoker) throws RpcException;

    /**
     * create invoker.
     *
     * @param <T>
     * @param proxy
     * @param type
     * @param url
     * @return invoker
     */
    @Adaptive({Constants.PROXY_KEY})
    <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) throws RpcException;

}
```

### 实现
```
public class JavassistProxyFactory extends AbstractProxyFactory {

    @SuppressWarnings("unchecked")
    public <T> T getProxy(Invoker<T> invoker, Class<?>[] interfaces) {
        return (T) Proxy.getProxy(interfaces).newInstance(new InvokerInvocationHandler(invoker));
    }

    public <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) {
        // TODO Wrapper cannot handle this scenario correctly: the classname contains '$'
        final Wrapper wrapper = Wrapper.getWrapper(proxy.getClass().getName().indexOf('$') < 0 ? proxy.getClass() : type);
        return new AbstractProxyInvoker<T>(proxy, type, url) {
            @Override
            protected Object doInvoke(T proxy, String methodName,
                                      Class<?>[] parameterTypes,
                                      Object[] arguments) throws Throwable {
                return wrapper.invokeMethod(proxy, methodName, parameterTypes, arguments);
            }
        };
    }

}
```
## 实际生成的代码:
修改源码,在生成字节码文件的时候,也输出至当前目录
```
            Class aClass = mCtc.toClass(loader, pd);
            try {
                mCtc.writeFile(".");//在当前目录下
            } catch (IOException e) {
                e.printStackTrace();
            }
            return aClass;
```
Warpper0.class文件内容如下:
```
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package com.alibaba.dubbo.common.bytecode;

import com.alibaba.dubbo.common.bytecode.ClassGenerator.DC;
import com.alibaba.dubbo.demo.DemoService;
import java.lang.reflect.InvocationTargetException;
import java.util.Map;

public class Wrapper0 extends Wrapper implements DC {
  public static String[] pns;
  public static Map pts;
  public static String[] mns;
  public static String[] dmns;
  public static Class[] mts0;

  public String[] getPropertyNames() {
    return pns;
  }

  public boolean hasProperty(String var1) {
    return pts.containsKey(var1);
  }

  public Class getPropertyType(String var1) {
    return (Class)pts.get(var1);
  }

  public String[] getMethodNames() {
    return mns;
  }

  public String[] getDeclaredMethodNames() {
    return dmns;
  }

  public void setPropertyValue(Object var1, String var2, Object var3) {
    try {
      DemoService var4 = (DemoService)var1;
    } catch (Throwable var6) {
      throw new IllegalArgumentException(var6);
    }

    throw new NoSuchPropertyException("Not found property \"" + var2 + "\" filed or setter method in class com.alibaba.dubbo.demo.DemoService.");
  }

  public Object getPropertyValue(Object var1, String var2) {
    try {
      DemoService var3 = (DemoService)var1;
    } catch (Throwable var5) {
      throw new IllegalArgumentException(var5);
    }

    throw new NoSuchPropertyException("Not found property \"" + var2 + "\" filed or setter method in class com.alibaba.dubbo.demo.DemoService.");
  }

  public Object invokeMethod(Object var1, String var2, Class[] var3, Object[] var4) throws InvocationTargetException {
    DemoService var5;
    try {
      var5 = (DemoService)var1;
    } catch (Throwable var8) {
      throw new IllegalArgumentException(var8);
    }

    try {
      if ("sayHello".equals(var2) && var3.length == 1) {
        return var5.sayHello((String)var4[0]);
      }
    } catch (Throwable var9) {
      throw new InvocationTargetException(var9);
    }

    throw new NoSuchMethodException("Not found method \"" + var2 + "\" in class com.alibaba.dubbo.demo.DemoService.");
  }

  public Wrapper0() {
  }
}

```

# 参考文章

http://www.tianshouzhi.com/api/tutorials/bytecode/354