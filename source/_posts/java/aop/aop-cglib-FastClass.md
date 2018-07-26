title: cglib-fastClass
date: 2018-07-19 20:13:43
tags:
	- aop 
	- cglib
categories:
---

https://www.programcreek.com/java-api-examples/index.php?api=org.springframework.cglib.reflect.FastClass
# 使用fastclass 进行"反射"

```
		Class<?> serviceClass = serviceBean.getClass();
		String methodName = request.getMethodName();
		Class<?>[] parameterTypes = request.getParameterTypes();
		Object[] parameters = request.getParameters();
        //获取FastClass
		FastClass serviceFastClass = FastClass.create(serviceClass);
        //获取FastMethod
		FastMethod serviceFastMethod = serviceFastClass.getMethod(methodName, parameterTypes);
        //调用FastMethod
		Object result = serviceFastMethod.invoke(serviceBean, parameters);

```


除了演示的FastMethod之外，FastClass还可以创建FastConstructors但不会创建快速字段。但FastClass如何比正常反射更快？ Java反射由JNI执行，其中方法调用由某些C代码执行。另一方面，FastClass创建了一些字节代码，可以直接从JVM中调用该方法。但是，较新版本的HotSpot JVM（可能还有许多其他现代JVM）知道一个名为**通胀(inflation)**的概念，当反射方法执行得足够频繁时，JVM会将反射方法调用转换为FastClass的本机版本(C代码执行))。您甚至可以通过将sun.reflect.inflationThreshold属性设置为较低的值来控制此行为（至少在HotSpot JVM上）。 （默认值为15.）此属性确定在多少反射调用之后，JNI调用应由字节代码检测版本替换。因此，我建议不要在现代JVM上使用FastClass，但它可以微调旧Java虚拟机的性能。
