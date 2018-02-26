title: source-dubbo-common
date: 2018-02-03 21:36:26
tags:
categories:
---
# ExtensionLoader源码
扩展点是Dubbo的核心，而扩展点的核心则是ExtensionLoader，这个类有点类似ClassLoader，但是ExtensionLoader是加载Dubbo的扩展点的。下面列出ExtensionLoader几个重要的属性结构。


```
public class ExtensionLoader<T> {
private static final ConcurrentMap<Class<?>, ExtensionLoader<?>> EXTENSION_LOADERS = new ConcurrentHashMap<Class<?>, ExtensionLoader<?>>();

private static final ConcurrentMap<Class<?>, Object> EXTENSION_INSTANCES = new ConcurrentHashMap<Class<?>, Object>();

private final Class<?> type;

private final ExtensionFactory objectFactory;

private final Holder<Map<String, Class<?>>> cachedClasses = new Holder<Map<String,Class<?>>>();
private final Holder<Object> cachedAdaptiveInstance = new Holder<Object>();
}
```

- 可以看到EXTENSION_LOADERS属性是一个static final的，那么说明应该是一个常量，这个就是用来装载dubbo的所有扩展点的ExtensionLoader
- 在Dubbo中，每种类型的扩展点都会有一个与其对应的ExtensionLoader，类似jvm中每个Class都会有一个ClassLoader,每个ExtensionLoader会包含多个该扩展点的实现，类似一个ClassLoader可以加载多个具体的类，但是不同的ExtensionLoader之间是隔离的，这点也和ClassLoader类似。那么理解dubbo的ExtensionLoader可以拿ClassLoader来进行类比，这样会加快自己对它的理解。
- 另一个常量属性是EXTENSION_INSTANCES，他是一个具体扩展类的实体，用于缓存，防止由于扩展点比较重，导致会浪费没必要的资源，所以在实现扩展点的时候，一定要确保扩展点可单例化，否则可能会出现问题。
- 另一个重要的属性是type，这里的type一般是接口，用于制定扩展点的类型，因为dubbo的扩展点申明是SPI的方式，所以某一个类型扩展点，就需要申明一个扩展点接口。

```
private ExtensionLoader(Class<?> type) {
    this.type = type;
    objectFactory = (type == ExtensionFactory.class ? null : ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension());
}
```

- 在ExtensionLoader.getExtensionLoader(ExtensionFactory.class)之后，不是直接返回某个扩展点，而是调用getAdaptiveExtension来获取一个扩展的适配器，这是为什么呢？因为一个扩展点有多个具体扩展的实现，那么直接通过ExtensionLoader直接返回一个扩展是不可靠的，需要一个适配器来根据实际情况返回具体的扩展实现。所以这里就有了cachedAdaptiveInstance属性的存在，dubbo里面的每个扩展的ExtensionLoader都有一个cachedAdaptiveInstance，这个属性的类型必须实现ExtensionLoader.type接口，这就是设计模式中的适配器模式。比如ExtensionFactory扩展点就有AdaptiveExtensionFactory适配器。扩展点的适配器可以是自己通过@Adaptive，也可以不提供实现，由dubbo通过动态生成Adaptive来提供一个适配器类。此处需要注意：Adaptive也是扩展点的某个实现，下面例举出ExtensionFactory扩展点的适配器
- cachedClasses,这个就是存储当前ExtensionLoader有哪些扩展点实现，从而可以实例化出某个具体的扩展点实体，cachedClasses声明为Holder<Map<String, Class<?>>>类型，其实可以理解为是Map<String, Class<?>>类型，Map的key是在type.getName文件中的=之前的内容，value这是这个扩展点实现的类对象了。

## 小结
- 1、一个扩展点类型一定是一个接口 
- 2、一个扩展点一定对应一个ExtensionLoader 
- 3、一个ExtensionLoader一定有一个Adapter 
- 4、一个扩展点可以有多个实现，并且都是用一个ExtensionLoader进行加载 
- 5、一个ExtensionLoader（除去ExtensionFactory扩展）都要有一个ExtensionFactory
