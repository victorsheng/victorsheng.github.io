title: beancopy源码学习
tags: '-cglib'
abbrlink: 3190299189
date: 2018-09-30 10:09:22
categories:
---

# 使用方法
BeanCopier是用于在两个bean之间进行属性拷贝的。
eanCopier支持两种方式，一种是不使用Converter的方式，仅对两个bean间属性名和类型完全相同的变量进行拷贝。另一种则引入Converter，可以对某些特定属性值进行特殊操作。

# 测试用例

```
  /**
   * 相同class内copy
   */
  @Test
  public void testSimple() {
    // 动态生成用于复制的类,false为不使用Converter类
    BeanCopier copier = BeanCopier.create(MA.class, MA.class, false);

    MA source = new MA();
    source.setIp(123);
    MA target = new MA();

    // 执行source到target的属性复制
    copier.copy(source, target, null);

    assertTrue(target.getIp() == 123);
  }
```

# 生成的类
类1
```
public class Object$$BeanCopierByCGLIB$$adb7e88b extends BeanCopier {
  public Object$$BeanCopierByCGLIB$$adb7e88b() {
  }

  public void copy(Object var1, Object var2, Converter var3) {
    ((MB)var2).setIp(((MA)var1).getIp());
  }
}


```
类2 **extends KeyFactory implements BeanCopierKey**
```
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.cglib.beans;

import org.springframework.cglib.beans.BeanCopier.BeanCopierKey;
import org.springframework.cglib.core.KeyFactory;

public class BeanCopier$BeanCopierKey$$KeyFactoryByCGLIB$$7db7c9a extends KeyFactory implements BeanCopierKey {
  private final String FIELD_0;
  private final String FIELD_1;
  private final boolean FIELD_2;

  public BeanCopier$BeanCopierKey$$KeyFactoryByCGLIB$$7db7c9a() {
  }

  public Object newInstance(String var1, String var2, boolean var3) {
    return new BeanCopier$BeanCopierKey$$KeyFactoryByCGLIB$$7db7c9a(var1, var2, var3);
  }

  public BeanCopier$BeanCopierKey$$KeyFactoryByCGLIB$$7db7c9a(String var1, String var2, boolean var3) {
    this.FIELD_0 = var1;
    this.FIELD_1 = var2;
    this.FIELD_2 = var3;
  }

  public int hashCode() {
    return ((95401 * 54189869 + (this.FIELD_0 != null ? this.FIELD_0.hashCode() : 0)) * 54189869 + (this.FIELD_1 != null ? this.FIELD_1.hashCode() : 0)) * 54189869 + (this.FIELD_2 ^ 1);
  }

  public boolean equals(Object var1) {
    if (var1 instanceof BeanCopier$BeanCopierKey$$KeyFactoryByCGLIB$$7db7c9a) {
      String var10000 = this.FIELD_0;
      String var10001 = ((BeanCopier$BeanCopierKey$$KeyFactoryByCGLIB$$7db7c9a)var1).FIELD_0;
      String var10002 = this.FIELD_0;
      if (((BeanCopier$BeanCopierKey$$KeyFactoryByCGLIB$$7db7c9a)var1).FIELD_0 == null) {
        if (var10002 != null) {
          return false;
        }
      } else if (var10002 == null || !var10000.equals(var10001)) {
        return false;
      }

      var10000 = this.FIELD_1;
      var10001 = ((BeanCopier$BeanCopierKey$$KeyFactoryByCGLIB$$7db7c9a)var1).FIELD_1;
      var10002 = this.FIELD_1;
      if (((BeanCopier$BeanCopierKey$$KeyFactoryByCGLIB$$7db7c9a)var1).FIELD_1 == null) {
        if (var10002 != null) {
          return false;
        }
      } else if (var10002 == null || !var10000.equals(var10001)) {
        return false;
      }

      if (this.FIELD_2 == ((BeanCopier$BeanCopierKey$$KeyFactoryByCGLIB$$7db7c9a)var1).FIELD_2) {
        return true;
      }
    }

    return false;
  }

  public String toString() {
    StringBuffer var10000 = new StringBuffer();
    var10000 = (this.FIELD_0 != null ? var10000.append(this.FIELD_0.toString()) : var10000.append("null")).append(", ");
    return (this.FIELD_1 != null ? var10000.append(this.FIELD_1.toString()) : var10000.append("null")).append(", ").append(this.FIELD_2).toString();
  }
}

```

# 从BeanCopier#create开始
```
abstract public class BeanCopier
{
    public static BeanCopier create(Class source, Class target, boolean useConverter) {
        Generator gen = new Generator();
        gen.setSource(source);
        gen.setTarget(target);
        gen.setUseConverter(useConverter);

        // 调用类创建方法
        return gen.create();
    }

    public static class Generator extends AbstractClassGenerator {

        public BeanCopier create() {
            // 1.通过KEY_FACTORY创建key实例
            Object key = KEY_FACTORY.newInstance(source.getName(), target.getName(), useConverter);
            // 2.调用AbstractClassGenerator#create创建copy类
            return (BeanCopier)super.create(key);
        }
        ...
    }
}

```


# private static final BeanCopierKey KEY_FACTORY =(BeanCopierKey)KeyFactory.create(BeanCopierKey.class);
KeyFactory#create
-> KeyFactory$Generator#create
-> AbstractClassGenerator#create

# AbstractClassGenerator#create方法流程
(见上一篇enhancer的文章)

# KeyFactory#generateClass方法流程(类2的创建)
```
public void generateClass(ClassVisitor v) {
            ClassEmitter ce = new ClassEmitter(v);

            // 对定义key工厂类结构的接口进行判断，判断该接口是否只有newInstance一个方法，newInstance的返回值是否为Object
            Method newInstance = ReflectUtils.findNewInstance(keyInterface);
            if (!newInstance.getReturnType().equals(Object.class)) {
                throw new IllegalArgumentException("newInstance method must return Object");
            }

            // 获取newInstance的入参类型，此处使用ASM的Type来定义
            Type[] parameterTypes = TypeUtils.getTypes(newInstance.getParameterTypes());
            // 创建class
            ce.begin_class(Constants.V1_2,
                           Constants.ACC_PUBLIC,
                           getClassName(),
                           KEY_FACTORY,
                           new Type[]{ Type.getType(keyInterface) },
                           Constants.SOURCE_FILE);
            //生成默认构造函数
            EmitUtils.null_constructor(ce);
            //生成newInstance 工厂方法
            EmitUtils.factory_method(ce, ReflectUtils.getSignature(newInstance));

            //生成有参构造方法
            int seed = 0;
            CodeEmitter e = ce.begin_method(Constants.ACC_PUBLIC,
                                            TypeUtils.parseConstructor(parameterTypes),
                                            null);
            e.load_this();
            e.super_invoke_constructor();
            e.load_this();
            for (int i = 0; i < parameterTypes.length; i++) {
                seed += parameterTypes[i].hashCode();

                //为每一个入参生成一个相同类型的类字段
                ce.declare_field(Constants.ACC_PRIVATE | Constants.ACC_FINAL,
                                 getFieldName(i),
                                 parameterTypes[i],
                                 null);
                e.dup();
                e.load_arg(i);
                e.putfield(getFieldName(i));
            }
            e.return_value();
            e.end_method();

            //生成hashCode函数
            e = ce.begin_method(Constants.ACC_PUBLIC, HASH_CODE, null);
            int hc = (constant != 0) ? constant : PRIMES[(int)(Math.abs(seed) % PRIMES.length)];
            int hm = (multiplier != 0) ? multiplier : PRIMES[(int)(Math.abs(seed * 13) % PRIMES.length)];
            e.push(hc);
            for (int i = 0; i < parameterTypes.length; i++) {
                e.load_this();
                e.getfield(getFieldName(i));
                EmitUtils.hash_code(e, parameterTypes[i], hm, customizer);
            }
            e.return_value();
            e.end_method();

            //生成equals函数，在equals函数中对每个入参都进行判断
            e = ce.begin_method(Constants.ACC_PUBLIC, EQUALS, null);
            Label fail = e.make_label();
            e.load_arg(0);
            e.instance_of_this();
            e.if_jump(e.EQ, fail);
            for (int i = 0; i < parameterTypes.length; i++) {
                e.load_this();
                e.getfield(getFieldName(i));
                e.load_arg(0);
                e.checkcast_this();
                e.getfield(getFieldName(i));
                EmitUtils.not_equals(e, parameterTypes[i], fail, customizer);
            }
            e.push(1);
            e.return_value();
            e.mark(fail);
            e.push(0);
            e.return_value();
            e.end_method();

            // toString
            e = ce.begin_method(Constants.ACC_PUBLIC, TO_STRING, null);
            e.new_instance(Constants.TYPE_STRING_BUFFER);
            e.dup();
            e.invoke_constructor(Constants.TYPE_STRING_BUFFER);
            for (int i = 0; i < parameterTypes.length; i++) {
                if (i > 0) {
                    e.push(", ");
                    e.invoke_virtual(Constants.TYPE_STRING_BUFFER, APPEND_STRING);
                }
                e.load_this();
                e.getfield(getFieldName(i));
                EmitUtils.append_string(e, parameterTypes[i], EmitUtils.DEFAULT_DELIMITERS, customizer);
            }
            e.invoke_virtual(Constants.TYPE_STRING_BUFFER, TO_STRING);
            e.return_value();
            e.end_method();

            ce.end_class();
        }
        
```
# BeanCopier#generateClass方法流程
```
public void generateClass(ClassVisitor v) {
            Type sourceType = Type.getType(source);
            Type targetType = Type.getType(target);
            ClassEmitter ce = new ClassEmitter(v);

            // 创建class
            ce.begin_class(Constants.V1_2,
                           Constants.ACC_PUBLIC,
                            // 类名,在AbstractClassGenerator#getClassName中生成
                           getClassName(),
                           BEAN_COPIER,
                           null,
                           Constants.SOURCE_FILE);

            // 生成默认构造函数
            EmitUtils.null_constructor(ce);

            // [BEGIN COPY METHOD]生成copy方法
            CodeEmitter e = ce.begin_method(Constants.ACC_PUBLIC, COPY, null);
            PropertyDescriptor[] getters = ReflectUtils.getBeanGetters(source);
            PropertyDescriptor[] setters = ReflectUtils.getBeanSetters(target);

            // names可根据属性名查找对应的getter方法
            Map names = new HashMap();
            for (int i = 0; i < getters.length; i++) {
                names.put(getters[i].getName(), getters[i]);
            }

            Local targetLocal = e.make_local();
            Local sourceLocal = e.make_local();
            if (useConverter) {
                e.load_arg(1);
                e.checkcast(targetType);
                e.store_local(targetLocal);
                e.load_arg(0);                
                e.checkcast(sourceType);
                e.store_local(sourceLocal);
            } else {
                e.load_arg(1);
                e.checkcast(targetType);
                e.load_arg(0);
                e.checkcast(sourceType);
            }
            // 为每个属性生成赋值语句
            for (int i = 0; i < setters.length; i++) {
                PropertyDescriptor setter = setters[i];
                PropertyDescriptor getter = (PropertyDescriptor)names.get(setter.getName());

                if (getter != null) {
                    MethodInfo read = ReflectUtils.getMethodInfo(getter.getReadMethod());
                    MethodInfo write = ReflectUtils.getMethodInfo(setter.getWriteMethod());
                    if (useConverter) {
                        Type setterType = write.getSignature().getArgumentTypes()[0];
                        e.load_local(targetLocal);
                        e.load_arg(2);
                        e.load_local(sourceLocal);
                        e.invoke(read);
                        e.box(read.getSignature().getReturnType());
                        EmitUtils.load_class(e, setterType);
                        e.push(write.getSignature().getName());
                        e.invoke_interface(CONVERTER, CONVERT);
                        e.unbox_or_zero(setterType);
                        e.invoke(write);
                        // 如果property类型相同
                    } else if (compatible(getter, setter)) {
                        e.dup2();
                        e.invoke(read);
                        e.invoke(write);
                    }
                }
            }
            e.return_value();
            e.end_method();
            //[END COPY METHOD]

            ce.end_class();
        }


```
剩下的事情，只需要调用copier.copy(source, target, null)，就可以完成source bean到target bean的属性拷贝了。


# 参考
https://www.jianshu.com/p/f8b892e08d26

https://blog.csdn.net/chinrui/article/details/79905074