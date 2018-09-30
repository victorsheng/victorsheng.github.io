---
title: cglib源码学习之Enhancer
abbrlink: 2185134716
date: 2018-07-25 20:35:01
tags:
categories:
---
# Enhancer设计图
## 源码时序图
![upload successful](/images/pasted-219.png)
注解版cglib:
https://github.com/victorsheng/cglib.git

## 类图

![upload successful](/images/pasted-220.png)

# 关键类1:AbstractClassGenerator
cglib基本所有的核心类都继承了AbstractClassGenerator
- 作为cglib代码中 最核心的调度者 ，封装了类创建的主要流程，并支持一些缓存操作、命名策略（NamingPolicy）、代码生成策略（GeneratorStrategy）等。
- 其中，protected Object create(Object key) 作为模版方法，定义了类的生成过程。同时将变化点进行封装，供继承类自主实现。

![upload successful](/images/pasted-216.png)

## GeneratorStrategy（接口） & DefaultGeneratorStrategy（默认实现类）
控制ClassGenerator生成class的字节码。
为继承类预留了两个抽象方法，可以对生成的字节码进行操作。

## NamingPolicy（接口） & DefaultNamingPolicy（默认实现类
用于控制生成类的命名规则。
一般的命名规则：
被代理类名 + $$ + CGLIB核心处理类 + "ByCGLIB" + $$ + key的hashCode


# 关键类2:KeyFactory
类库中重要的唯一标识生成器，用于cglib做cache时做map key，比较底层的基础类。


# aop如何生成com.example.demo.proxy.HelloWorldImpl2$$EnhancerByCGLIB$$6b589cfb

最外层
`HelloWorldImpl2 proxyImpo = (HelloWorldImpl2) enhancer.create();`

Enhancer
```
  public Object create() {
    this.classOnly = false;
    this.argumentTypes = null;
    return this.createHelper();
  }
  
    private Object createHelper() {
    this.preValidate();
    Object key = KEY_FACTORY.newInstance(this.superclass != null ? this.superclass.getName() : null, ReflectUtils.getNames(this.interfaces), this.filter == ALL_ZERO ? null : new WeakCacheKey(this.filter), this.callbackTypes, this.useFactory, this.interceptDuringConstruction, this.serialVersionUID);
    this.currentKey = key;
    Object result = super.create(key);
    return result;
  }
```
其中 super.create(key); 调用父类org.springframework.cglib.core.AbstractClassGenerator的create方法

```
protected Object create(Object key) {
    try {
      ClassLoader loader = this.getClassLoader();
      Map<ClassLoader, AbstractClassGenerator.ClassLoaderData> cache = CACHE;
      AbstractClassGenerator.ClassLoaderData data = (AbstractClassGenerator.ClassLoaderData)cache.get(loader);
      if (data == null) {
        Class var5 = AbstractClassGenerator.class;
        synchronized(AbstractClassGenerator.class) {
          cache = CACHE;
          data = (AbstractClassGenerator.ClassLoaderData)cache.get(loader);
          if (data == null) {
            Map<ClassLoader, AbstractClassGenerator.ClassLoaderData> newCache = new WeakHashMap(cache);
            data = new AbstractClassGenerator.ClassLoaderData(loader);
            newCache.put(loader, data);
            CACHE = newCache;
          }
        }
      }

      this.key = key;
      Object obj = data.get(this, this.getUseCache());
      return obj instanceof Class ? this.firstInstance((Class)obj) : this.nextInstance(obj);
    } catch (RuntimeException var9) {
      throw var9;
    } catch (Error var10) {
      throw var10;
    } catch (Exception var11) {
      throw new CodeGenerationException(var11);
    }
  }
```

其中Object obj = data.get(this, this.getUseCache());

```
public Object get(AbstractClassGenerator gen, boolean useCache) {
      if (!useCache) {
        return gen.generate(this);
      } else {
        Object cachedValue = this.generatedClasses.get(gen);
        return gen.unwrapCachedValue(cachedValue);
      }
    }
```

其中Object cachedValue = this.generatedClasses.get(gen);
org.springframework.cglib.core.internal.LoadingCache:
```
  public V get(K key) {
    KK cacheKey = this.keyMapper.apply(key);
    Object v = this.map.get(cacheKey);
    return v != null && !(v instanceof FutureTask) ? v : this.createEntry(key, cacheKey, v);
  }
```
LoadingCache通过FutureTask获取

Enhancer类
```
  protected Class generate(ClassLoaderData data) {
    this.validate();
    if (this.superclass != null) {
      this.setNamePrefix(this.superclass.getName());
    } else if (this.interfaces != null) {
      this.setNamePrefix(this.interfaces[ReflectUtils.findPackageProtected(this.interfaces)].getName());
    }

    return super.generate(data);
  }
```
调用父类AbstractClassGenerator的protected Class generate(AbstractClassGenerator.ClassLoaderData data)方法

```
 protected Class generate(AbstractClassGenerator.ClassLoaderData data) {
    Object save = CURRENT.get();
    CURRENT.set(this);

    Class var8;
    try {
      ClassLoader classLoader = data.getClassLoader();
      if (classLoader == null) {
        throw new IllegalStateException("ClassLoader is null while trying to define class " + this.getClassName() + ". It seems that the loader has been expired from a weak reference somehow. Please file an issue at cglib's issue tracker.");
      }

      String className;
      synchronized(classLoader) {
        className = this.generateClassName(data.getUniqueNamePredicate());
        data.reserveName(className);
        this.setClassName(className);
      }

      Class gen;
      if (this.attemptLoad) {
        try {
          gen = classLoader.loadClass(this.getClassName());
          Class var25 = gen;
          return var25;
        } catch (ClassNotFoundException var20) {
          ;
        }
      }

      byte[] b = this.strategy.generate(this);
      className = ClassNameReader.getClassName(new ClassReader(b));
      ProtectionDomain protectionDomain = this.getProtectionDomain();
      synchronized(classLoader) {
        if (protectionDomain == null) {
          gen = ReflectUtils.defineClass(className, b, classLoader);
        } else {
          gen = ReflectUtils.defineClass(className, b, classLoader, protectionDomain);
        }
      }

      var8 = gen;
    } catch (RuntimeException var21) {
      throw var21;
    } catch (Error var22) {
      throw var22;
    } catch (Exception var23) {
      throw new CodeGenerationException(var23);
    } finally {
      CURRENT.set(save);
    }

    return var8;
  }
```
DefaultGeneratorStrategy的generate方法
调用了
AbstractClassGenerator类的public void generateClass(ClassVisitor v) throws Exception方法
```
public void generateClass(ClassVisitor v) throws Exception {
    Class sc = this.superclass == null ? Object.class : this.superclass;
    if (TypeUtils.isFinal(sc.getModifiers())) {
      throw new IllegalArgumentException("Cannot subclass final class " + sc.getName());
    } else {
      List constructors = new ArrayList(Arrays.asList(sc.getDeclaredConstructors()));
      this.filterConstructors(sc, constructors);
      List actualMethods = new ArrayList();
      List interfaceMethods = new ArrayList();
      final Set forcePublic = new HashSet();
      getMethods(sc, this.interfaces, actualMethods, interfaceMethods, forcePublic);
      List methods = CollectionUtils.transform(actualMethods, new Transformer() {
        public Object transform(Object value) {
          Method method = (Method)value;
          int modifiers = 16 | method.getModifiers() & -1025 & -257 & -33;
          if (forcePublic.contains(MethodWrapper.create(method))) {
            modifiers = modifiers & -5 | 1;
          }

          return ReflectUtils.getMethodInfo(method, modifiers);
        }
      });
      ClassEmitter e = new ClassEmitter(v);
      if (this.currentData == null) {
        e.begin_class(46, 1, this.getClassName(), Type.getType(sc), this.useFactory ? TypeUtils.add(TypeUtils.getTypes(this.interfaces), FACTORY) : TypeUtils.getTypes(this.interfaces), "<generated>");
      } else {
        e.begin_class(46, 1, this.getClassName(), (Type)null, new Type[]{FACTORY}, "<generated>");
      }

      List constructorInfo = CollectionUtils.transform(constructors, MethodInfoTransformer.getInstance());
      e.declare_field(2, "CGLIB$BOUND", Type.BOOLEAN_TYPE, (Object)null);
      e.declare_field(9, "CGLIB$FACTORY_DATA", OBJECT_TYPE, (Object)null);
      if (!this.interceptDuringConstruction) {
        e.declare_field(2, "CGLIB$CONSTRUCTED", Type.BOOLEAN_TYPE, (Object)null);
      }

      e.declare_field(26, "CGLIB$THREAD_CALLBACKS", THREAD_LOCAL, (Object)null);
      e.declare_field(26, "CGLIB$STATIC_CALLBACKS", CALLBACK_ARRAY, (Object)null);
      if (this.serialVersionUID != null) {
        e.declare_field(26, "serialVersionUID", Type.LONG_TYPE, this.serialVersionUID);
      }

      for(int i = 0; i < this.callbackTypes.length; ++i) {
        e.declare_field(2, getCallbackField(i), this.callbackTypes[i], (Object)null);
      }

      e.declare_field(10, "CGLIB$CALLBACK_FILTER", OBJECT_TYPE, (Object)null);
      if (this.currentData == null) {
        this.emitMethods(e, methods, actualMethods);
        this.emitConstructors(e, constructorInfo);
      } else {
        this.emitDefaultConstructor(e);
      }

      this.emitSetThreadCallbacks(e);
      this.emitSetStaticCallbacks(e);
      this.emitBindCallbacks(e);
      if (this.useFactory || this.currentData != null) {
        int[] keys = this.getCallbackKeys();
        this.emitNewInstanceCallbacks(e);
        this.emitNewInstanceCallback(e);
        this.emitNewInstanceMultiarg(e, constructorInfo);
        this.emitGetCallback(e, keys);
        this.emitSetCallback(e, keys);
        this.emitGetCallbacks(e);
        this.emitSetCallbacks(e);
      }

      e.end_class();
    }
  }
```


在gen = ReflectUtils.defineClass(className, b, classLoader);调用前打断点,动态代理实现类即将从数组转为class的时候,观察调用情况
```
generate:332, AbstractClassGenerator (org.springframework.cglib.core)
generate:492, Enhancer (org.springframework.cglib.proxy)
apply:93, AbstractClassGenerator$ClassLoaderData$3 (org.springframework.cglib.core)
apply:91, AbstractClassGenerator$ClassLoaderData$3 (org.springframework.cglib.core)
call:54, LoadingCache$2 (org.springframework.cglib.core.internal)
run$$$capture:266, FutureTask (java.util.concurrent)
run:-1, FutureTask (java.util.concurrent)
 - Async stacktrace
<init>:132, FutureTask (java.util.concurrent)
createEntry:52, LoadingCache (org.springframework.cglib.core.internal)
get:34, LoadingCache (org.springframework.cglib.core.internal)
get:116, AbstractClassGenerator$ClassLoaderData (org.springframework.cglib.core)
create:291, AbstractClassGenerator (org.springframework.cglib.core)
createHelper:480, Enhancer (org.springframework.cglib.proxy)
create:305, Enhancer (org.springframework.cglib.proxy)
main:24, DoCGLib2 (com.example.demo.proxy.cglib.enhancer)
```



# 参考
https://www.cnblogs.com/cruze/p/3843996.html
https://www.jianshu.com/p/f8b892e08d26