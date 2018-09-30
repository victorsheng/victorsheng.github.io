---
title: cglib应用之spring-aop
tags:
  - aop
  - cglib
abbrlink: 201383825
date: 2018-07-19 20:08:14
categories:
---
# 应用
- spring
- mybatis
- hibernate
- dubbo(jdk,Javassist)
- mockito
- guice

# spring aop介绍
Spring Aop采用的动态织入
动态织入又分静动两种，静则指织入过程只在第一次调用时执行；动则指根据代码动态运行的中间状态来决定如何操作，每次调用Target的时候都执行。 **???**

# spring ProxyFactoryBean

```
public static void main(String[] args) throws Exception {

    //1.针对不同的时期类型，提供不同的Advice
    Advice beforeAdvice = new TicketServiceBeforeAdvice();
    Advice afterReturningAdvice = new TicketServiceAfterReturningAdvice();
    Advice aroundAdvice = new TicketServiceAroundAdvice();
    Advice throwsAdvice = new TicketServiceThrowsAdvice();

    RailwayStation railwayStation = new RailwayStation();

    //2.创建ProxyFactoryBean,用以创建指定对象的Proxy对象
    ProxyFactoryBean proxyFactoryBean = new ProxyFactoryBean();
    //3.设置Proxy的接口
    proxyFactoryBean.setInterfaces(TicketService.class);
    //4. 设置RealSubject
    proxyFactoryBean.setTarget(railwayStation);
    //5.使用JDK基于接口实现机制的动态代理生成Proxy代理对象，如果想使用CGLIB，需要将这个flag设置成true
    proxyFactoryBean.setProxyTargetClass(true);

    //6. 添加不同的Advice

    proxyFactoryBean.addAdvice(afterReturningAdvice);
    proxyFactoryBean.addAdvice(aroundAdvice);
    proxyFactoryBean.addAdvice(throwsAdvice);
    proxyFactoryBean.addAdvice(beforeAdvice);
    proxyFactoryBean.setProxyTargetClass(false);
    //7通过ProxyFactoryBean生成Proxy对象
    TicketService ticketService = (TicketService) proxyFactoryBean.getObject();
    ticketService.sellTicket();

  }
```

## 工厂类初始化:由DefaultAopProxyFactory实例化AopProxy
```
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {

    /*将AdvisedSupport传入AopProxy*/
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

### AopProxy共同的初始化参数:AdvisedSupport(配置类)
AdvisedSupport有两个重要的内容：TargetSource和List<Advisor> advisors和List<Class<?>> interfaces。 
- TargetSource是目标类型和目标对象的包裹
- List<Class<?>> interfaces：包含了目标类型实现的接口，jdk proxy所需的接口
- List<Advisor> advisors：这里包含了我们在xml文件中配置的所有信息。这一部分是每个AdvisedSupport所共享的信息，而前面两个是每个AdvisedSupport所独有的信息。 

`public class AdvisedSupport extends ProxyConfig implements Advised`
- 接口Advised：主要包含TargetSource和List<Advisor> advisors和List<Class<?>> interface 

AdvisedSupport ，对于它的List<Advisor> advisors则分别对应xml中的配置： 

```
<aop:before pointcut-ref="businessService1" method="doBefore" />  
<aop:after pointcut-ref="businessService2" method="doAfter"/>  
<aop:around pointcut-ref="businessService2" method="doAround"/>
<aop:after-throwing pointcut-ref="businessService1" method="doThrowing" throwing="ex"/>
```

- ProxyConfig：则是对要产生的代理对象的一些配置，如下： 
```
	private boolean proxyTargetClass = false;

	private boolean optimize = false;

	boolean opaque = false;

	boolean exposeProxy = false;

	private boolean frozen = false;
```
其中proxyTargetClass：表示是否强制使用CGLIB进行代理对象的创建 
exposeProxy ：表示是否暴漏代理对象，实现线程内共享，这里又是使用ThreadLocal模式。 
他们分别对应xml配置中的 
```
<aop:config expose-proxy="false" proxy-target-class="false"> 
</aop:config>
```


## JdkDynamicAopProxy
final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable
### 实现InvocationHandler接口
```
@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		MethodInvocation invocation;
		Object oldProxy = null;
		boolean setProxyContext = false;

		TargetSource targetSource = this.advised.targetSource;
		Class<?> targetClass = null;
		Object target = null;

		try {
			if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
				// The target does not implement the equals(Object) method itself.
				return equals(args[0]);
			}
			else if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
				// The target does not implement the hashCode() method itself.
				return hashCode();
			}
			else if (method.getDeclaringClass() == DecoratingProxy.class) {
				// There is only getDecoratedClass() declared -> dispatch to proxy config.
				return AopProxyUtils.ultimateTargetClass(this.advised);
			}
			else if (!this.advised.opaque && method.getDeclaringClass().isInterface() &&
					method.getDeclaringClass().isAssignableFrom(Advised.class)) {
				// Service invocations on ProxyConfig with the proxy config...
				return AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);
			}

			Object retVal;
			//对象内部的自我调用,无法实施切面的增强
			if (this.advised.exposeProxy) {
				// Make invocation available if necessary.
				oldProxy = AopContext.setCurrentProxy(proxy);
				setProxyContext = true;
			}

			// May be null. Get as late as possible to minimize the time we "own" the target,
			// in case it comes from a pool.
			target = targetSource.getTarget();
			if (target != null) {
				targetClass = target.getClass();
			}

			// Get the interception chain for this method.
			List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

			// Check whether we have any advice. If we don't, we can fallback on direct
			// reflective invocation of the target, and avoid creating a MethodInvocation.
			if (chain.isEmpty()) {
				// We can skip creating a MethodInvocation: just invoke the target directly
				// Note that the final invoker must be an InvokerInterceptor so we know it does
				// nothing but a reflective operation on the target, and no hot swapping or fancy proxying.
				Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
				retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
			}
			else {
				// 创建拦截器链
				invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
				//调用
				retVal = invocation.proceed();
			}

			// Massage return value if necessary.
			Class<?> returnType = method.getReturnType();
			if (retVal != null && retVal == target &&
					returnType != Object.class && returnType.isInstance(proxy) &&
					!RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
				// Special case: it returned "this" and the return type of the method
				// is type-compatible. Note that we can't help if the target sets
				// a reference to itself in another returned object.
				retVal = proxy;
			}
			else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
				throw new AopInvocationException(
						"Null return value from advice does not match primitive return type for: " + method);
			}
			return retVal;
		}
		finally {
			if (target != null && !targetSource.isStatic()) {
				// Must have come from TargetSource.
				targetSource.releaseTarget(target);
			}
			if (setProxyContext) {
				// Restore old proxy.
				AopContext.setCurrentProxy(oldProxy);
			}
		}
	}
```

### org.springframework.aop.framework.ReflectiveMethodInvocation
内部递归调用调用,遍历拦截链

```
@Override
	public Object proceed() throws Throwable {
		//	We start with an index of -1 and increment early.
		if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
			return invokeJoinpoint();
		}

		Object interceptorOrInterceptionAdvice =
				this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
		//动态匹配
		if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
			// Evaluate dynamic method matcher here: static part will already have
			// been evaluated and found to match.
			InterceptorAndDynamicMethodMatcher dm =
					(InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
			if (dm.methodMatcher.matches(this.method, this.targetClass, this.arguments)) {
				return dm.interceptor.invoke(this);
			}
			else {
				// Dynamic matching failed.
				//递归调用
				return proceed();
			}
		}
		//静态匹配
		else {
			// It's an interceptor, so we just invoke it: The pointcut will have
			// been evaluated statically before this object was constructed.
			return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
		}
	}org.springframework.aop.framework.ReflectiveMethodInvocation
```

## CglibAopProxy
class CglibAopProxy implements AopProxy, Serializable
### 调用Enhancer

```
@Override
	public Object getProxy(ClassLoader classLoader) {
		if (logger.isDebugEnabled()) {
			logger.debug("Creating CGLIB proxy: target source is " + this.advised.getTargetSource());
		}

		try {
			Class<?> rootClass = this.advised.getTargetClass();
			Assert.state(rootClass != null, "Target class must be available for creating a CGLIB proxy");

			Class<?> proxySuperClass = rootClass;
			if (ClassUtils.isCglibProxyClass(rootClass)) {
				proxySuperClass = rootClass.getSuperclass();
				Class<?>[] additionalInterfaces = rootClass.getInterfaces();
				for (Class<?> additionalInterface : additionalInterfaces) {
					this.advised.addInterface(additionalInterface);
				}
			}

			// Validate the class, writing log messages as necessary.
			validateClassIfNecessary(proxySuperClass, classLoader);

			// Configure CGLIB Enhancer...
			Enhancer enhancer = createEnhancer();
			if (classLoader != null) {
				enhancer.setClassLoader(classLoader);
				if (classLoader instanceof SmartClassLoader &&
						((SmartClassLoader) classLoader).isClassReloadable(proxySuperClass)) {
					enhancer.setUseCache(false);
				}
			}
			enhancer.setSuperclass(proxySuperClass);
			enhancer.setInterfaces(AopProxyUtils.completeProxiedInterfaces(this.advised));
			enhancer.setNamingPolicy(SpringNamingPolicy.INSTANCE);
			enhancer.setStrategy(new ClassLoaderAwareUndeclaredThrowableStrategy(classLoader));
			//设置拦截器
			Callback[] callbacks = getCallbacks(rootClass);
			Class<?>[] types = new Class<?>[callbacks.length];
			for (int x = 0; x < types.length; x++) {
				types[x] = callbacks[x].getClass();
			}
			// fixedInterceptorMap only populated at this point, after getCallbacks call above
			enhancer.setCallbackFilter(new ProxyCallbackFilter(
					this.advised.getConfigurationOnlyCopy(), this.fixedInterceptorMap, this.fixedInterceptorOffset));
			enhancer.setCallbackTypes(types);

			// Generate the proxy class and create a proxy instance.
			return createProxyClassAndInstance(enhancer, callbacks);
		}
		catch (CodeGenerationException ex) {
			throw new AopConfigException("Could not generate CGLIB subclass of class [" +
					this.advised.getTargetClass() + "]: " +
					"Common causes of this problem include using a final class or a non-visible class",
					ex);
		}
		catch (IllegalArgumentException ex) {
			throw new AopConfigException("Could not generate CGLIB subclass of class [" +
					this.advised.getTargetClass() + "]: " +
					"Common causes of this problem include using a final class or a non-visible class",
					ex);
		}
		catch (Throwable ex) {
			// TargetSource.getTarget() failed
			throw new AopConfigException("Unexpected AOP exception", ex);
		}
	}
```

### 组装拦截器
TODO 关于静态方法等的拦截 逻辑没有看懂
```
private Callback[] getCallbacks(Class<?> rootClass) throws Exception {
		// Parameters used for optimization choices...
		boolean exposeProxy = this.advised.isExposeProxy();
		boolean isFrozen = this.advised.isFrozen();
		boolean isStatic = this.advised.getTargetSource().isStatic();

		// Choose an "aop" interceptor (used for AOP calls).
		//动态拦截链
		Callback aopInterceptor = new DynamicAdvisedInterceptor(this.advised);

		// Choose a "straight to target" interceptor. (used for calls that are
		// unadvised but can return this). May be required to expose the proxy.
		//直接调用链
		Callback targetInterceptor;
		if (exposeProxy) {
			targetInterceptor = isStatic ?
					new StaticUnadvisedExposedInterceptor(this.advised.getTargetSource().getTarget()) :
					new DynamicUnadvisedExposedInterceptor(this.advised.getTargetSource());
		}
		else {
			targetInterceptor = isStatic ?
					new StaticUnadvisedInterceptor(this.advised.getTargetSource().getTarget()) :
					new DynamicUnadvisedInterceptor(this.advised.getTargetSource());
		}

		// Choose a "direct to target" dispatcher (used for
		// unadvised calls to static targets that cannot return this).
		Callback targetDispatcher = isStatic ?
				new StaticDispatcher(this.advised.getTargetSource().getTarget()) : new SerializableNoOp();
		//动态拦截链
		Callback[] mainCallbacks = new Callback[] {
				aopInterceptor,  // for normal advice
				targetInterceptor,  // invoke target without considering advice, if optimized
				new SerializableNoOp(),  // no override for methods mapped to this
				targetDispatcher, this.advisedDispatcher,
				new EqualsInterceptor(this.advised),
				new HashCodeInterceptor(this.advised)
		};

		Callback[] callbacks;

		// If the target is a static one and the advice chain is frozen,
		// then we can make some optimizations by sending the AOP calls
		// direct to the target using the fixed chain for that method.
		if (isStatic && isFrozen) {
			Method[] methods = rootClass.getMethods();
			Callback[] fixedCallbacks = new Callback[methods.length];
			this.fixedInterceptorMap = new HashMap<String, Integer>(methods.length);

			// TODO: small memory optimization here (can skip creation for methods with no advice)
			for (int x = 0; x < methods.length; x++) {
				List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(methods[x], rootClass);
				fixedCallbacks[x] = new FixedChainStaticTargetInterceptor(
						chain, this.advised.getTargetSource().getTarget(), this.advised.getTargetClass());
				this.fixedInterceptorMap.put(methods[x].toString(), x);
			}

			// Now copy both the callbacks from mainCallbacks
			// and fixedCallbacks into the callbacks array.
			callbacks = new Callback[mainCallbacks.length + fixedCallbacks.length];
			System.arraycopy(mainCallbacks, 0, callbacks, 0, mainCallbacks.length);
			System.arraycopy(fixedCallbacks, 0, callbacks, mainCallbacks.length, fixedCallbacks.length);
			this.fixedInterceptorOffset = mainCallbacks.length;
		}
		else {
			callbacks = mainCallbacks;
		}
		return callbacks;
	}
```
### 实现methodInterceptor接口
```
private static class DynamicAdvisedInterceptor implements MethodInterceptor, Serializable {

		private final AdvisedSupport advised;

		public DynamicAdvisedInterceptor(AdvisedSupport advised) {
			this.advised = advised;
		}

		@Override
		public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
			Object oldProxy = null;
			boolean setProxyContext = false;
			Class<?> targetClass = null;
			Object target = null;
			try {
				if (this.advised.exposeProxy) {
					// Make invocation available if necessary.
					oldProxy = AopContext.setCurrentProxy(proxy);
					setProxyContext = true;
				}
				// May be null. Get as late as possible to minimize the time we
				// "own" the target, in case it comes from a pool...
				target = getTarget();
				if (target != null) {
					targetClass = target.getClass();
				}
				List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
				Object retVal;
				// Check whether we only have one InvokerInterceptor: that is,
				// no real advice, but just reflective invocation of the target.
				if (chain.isEmpty() && Modifier.isPublic(method.getModifiers())) {
					// We can skip creating a MethodInvocation: just invoke the target directly.
					// Note that the final invoker must be an InvokerInterceptor, so we know
					// it does nothing but a reflective operation on the target, and no hot
					// swapping or fancy proxying.
					//直接调用原方法
					Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
					retVal = methodProxy.invoke(target, argsToUse);
				}
				else {
					// We need to create a method invocation...
					//进入调用链
					retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
				}
				retVal = processReturnType(proxy, target, method, retVal);
				return retVal;
			}
			finally {
				if (target != null) {
					releaseTarget(target);
				}
				if (setProxyContext) {
					// Restore old proxy.
					AopContext.setCurrentProxy(oldProxy);
				}
			}
		}
}
```
### CglibMethodInvocation 进行实际链式调用,继承ReflectiveMethodInvocation

```
private static class CglibMethodInvocation extends ReflectiveMethodInvocation {

		private final MethodProxy methodProxy;

		private final boolean publicMethod;

		public CglibMethodInvocation(Object proxy, Object target, Method method, Object[] arguments,
				Class<?> targetClass, List<Object> interceptorsAndDynamicMethodMatchers, MethodProxy methodProxy) {

			super(proxy, target, method, arguments, targetClass, interceptorsAndDynamicMethodMatchers);
			this.methodProxy = methodProxy;
			this.publicMethod = Modifier.isPublic(method.getModifiers());
		}

		/**
		 * Gives a marginal performance improvement versus using reflection to
		 * invoke the target when invoking public methods.
		 */
		@Override
		protected Object invokeJoinpoint() throws Throwable {
			if (this.publicMethod) {
				return this.methodProxy.invoke(this.target, this.arguments);
			}
			else {
				return super.invokeJoinpoint();
			}
		}
	}
```




## 调用过程
当我们调用 ticketService.sellTicket()时，Spring会把这个方法调用转换成一个MethodInvocation对象，然后结合上述的我们添加的各种Advice,组成一个ReflectiveMethodInvocation

```
public class ReflectiveMethodInvocation implements ProxyMethodInvocation, Cloneable {
 
	protected final Object proxy;
 
	protected final Object target;
 
	protected final Method method;
 
	protected Object[] arguments;
 
	private final Class<?> targetClass;
 
	/**
	 * Lazily initialized map of user-specific attributes for this invocation.
	 */
	private Map<String, Object> userAttributes;
 
	/**
	 * List of MethodInterceptor and InterceptorAndDynamicMethodMatcher
	 * that need dynamic checks.
	 */
	protected final List<?> interceptorsAndDynamicMethodMatchers;
 
	/**
	 * Index from 0 of the current interceptor we're invoking.
	 * -1 until we invoke: then the current interceptor.
	 */
	private int currentInterceptorIndex = -1;
 
 
	/**
	 * Construct a new ReflectiveMethodInvocation with the given arguments.
	 * @param proxy the proxy object that the invocation was made on
	 * @param target the target object to invoke
	 * @param method the method to invoke
	 * @param arguments the arguments to invoke the method with
	 * @param targetClass the target class, for MethodMatcher invocations
	 * @param interceptorsAndDynamicMethodMatchers interceptors that should be applied,
	 * along with any InterceptorAndDynamicMethodMatchers that need evaluation at runtime.
	 * MethodMatchers included in this struct must already have been found to have matched
	 * as far as was possibly statically. Passing an array might be about 10% faster,
	 * but would complicate the code. And it would work only for static pointcuts.
	 */
	protected ReflectiveMethodInvocation(
			Object proxy, Object target, Method method, Object[] arguments,
			Class<?> targetClass, List<Object> interceptorsAndDynamicMethodMatchers) {
 
		this.proxy = proxy;//proxy对象
		this.target = target;//真实的realSubject对象
		this.targetClass = targetClass;//被代理的类类型
		this.method = BridgeMethodResolver.findBridgedMethod(method);//方法引用
		this.arguments = AopProxyUtils.adaptArgumentsIfNecessary(method, arguments);//调用参数
		this.interceptorsAndDynamicMethodMatchers = interceptorsAndDynamicMethodMatchers;//Advice拦截器链
	}
 
 
	@Override
	public final Object getProxy() {
		return this.proxy;
	}
 
	@Override
	public final Object getThis() {
		return this.target;
	}
 
	@Override
	public final AccessibleObject getStaticPart() {
		return this.method;
	}
 
	/**
	 * Return the method invoked on the proxied interface.
	 * May or may not correspond with a method invoked on an underlying
	 * implementation of that interface.
	 */
	@Override
	public final Method getMethod() {
		return this.method;
	}
 
	@Override
	public final Object[] getArguments() {
		return (this.arguments != null ? this.arguments : new Object[0]);
	}
 
	@Override
	public void setArguments(Object... arguments) {
		this.arguments = arguments;
	}
 
 
	@Override
	public Object proceed() throws Throwable {
		//	没有拦截器，则直接调用Joinpoint上的method，即直接调用MethodInvocation We start with an index of -1 and increment early.
		if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
			return invokeJoinpoint();
		}
                // 取得第拦截器链上第N个拦截器 
		Object interceptorOrInterceptionAdvice =
				this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
		//PointcutInterceptor会走这个逻辑
                if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
			// Evaluate dynamic method matcher here: static part will already have
			// been evaluated and found to match.
			InterceptorAndDynamicMethodMatcher dm =
					(InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
			//当前拦截器是符合拦截规则，每个拦截器可以定义是否特定的类和方法名是否符合拦截规则
                        //实际上PointCut定义的方法签名最后会转换成这个MethodMatcher，并置于拦截器中
                        if (dm.methodMatcher.matches(this.method, this.targetClass, this.arguments)) {
			     //符合拦截规则，调用拦截器invoke()	
                             return dm.interceptor.invoke(this);
			}
			else {
				// Dynamic matching failed.
				// Skip this interceptor and invoke the next in the chain.
                                // 当前方法不需要拦截器操作，则直接往前推进
                                return proceed();
			}
		}
		else {
			// It's an interceptor, so we just invoke it: The pointcut will have
			// been evaluated statically before this object was constructed.
                        //直接调用拦截器，
                        return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
		}
	}
 
	/**
	 * Invoke the joinpoint using reflection.
	 * Subclasses can override this to use custom invocation.
	 * @return the return value of the joinpoint
	 * @throws Throwable if invoking the joinpoint resulted in an exception
	 */
	protected Object invokeJoinpoint() throws Throwable {
		return AopUtils.invokeJoinpointUsingReflection(this.target, this.method, this.arguments);
	}

```

## PointCut与Advice的结合------Adivce的条件执行
```
public interface MethodMatcher {
 
	/**
	 * 提供方法签名和所属的Class类型，判断是否支持 
         * Perform static checking whether the given method matches. If this
	 * returns {@code false} or if the {@link #isRuntime()} method
	 * returns {@code false}, no runtime check (i.e. no.
	 * {@link #matches(java.lang.reflect.Method, Class, Object[])} call) will be made.
	 * @param method the candidate method
	 * @param targetClass the target class (may be {@code null}, in which case
	 * the candidate class must be taken to be the method's declaring class)
	 * @return whether or not this method matches statically
	 */
	boolean matches(Method method, Class<?> targetClass);
 
	/**
	 * Is this MethodMatcher dynamic, that is, must a final call be made on the
	 * {@link #matches(java.lang.reflect.Method, Class, Object[])} method at
	 * runtime even if the 2-arg matches method returns {@code true}?
	 * <p>Can be invoked when an AOP proxy is created, and need not be invoked
	 * again before each method invocation,
	 * @return whether or not a runtime match via the 3-arg
	 * {@link #matches(java.lang.reflect.Method, Class, Object[])} method
	 * is required if static matching passed
	 */
	boolean isRuntime();
 
	/**
	 * Check whether there a runtime (dynamic) match for this method,
	 * which must have matched statically.
	 * <p>This method is invoked only if the 2-arg matches method returns
	 * {@code true} for the given method and target class, and if the
	 * {@link #isRuntime()} method returns {@code true}. Invoked
	 * immediately before potential running of the advice, after any
	 * advice earlier in the advice chain has run.
	 * @param method the candidate method
	 * @param targetClass the target class (may be {@code null}, in which case
	 * the candidate class must be taken to be the method's declaring class)
	 * @param args arguments to the method
	 * @return whether there's a runtime match
	 * @see MethodMatcher#matches(Method, Class)
	 */
	boolean matches(Method method, Class<?> targetClass, Object... args);
 
 
	/**
	 * Canonical instance that matches all methods.
	 */
	MethodMatcher TRUE = TrueMethodMatcher.INSTANCE;
 
}

```

调用

```
class InterceptorAndDynamicMethodMatcher {


	final MethodInterceptor interceptor;
 
	final MethodMatcher methodMatcher;
 
	public InterceptorAndDynamicMethodMatcher(MethodInterceptor interceptor, MethodMatcher methodMatcher) {
		this.interceptor = interceptor;
		this.methodMatcher = methodMatcher;
	}

	@Override
	public Object proceed() throws Throwable {
		//	We start with an index of -1 and increment early.
		if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
			return invokeJoinpoint();
		}
 
		Object interceptorOrInterceptionAdvice =
				this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
		//起到一定的过滤作用，如果不匹配，则直接skip
                if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
			// Evaluate dynamic method matcher here: static part will already have
			// been evaluated and found to match.
			InterceptorAndDynamicMethodMatcher dm =
					(InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
			//满足匹配规则，则拦截器Advice生效
                        if (dm.methodMatcher.matches(this.method, this.targetClass, this.arguments)) {
				return dm.interceptor.invoke(this);
			}
			else {
				// Dynamic matching failed.
				// Skip this interceptor and invoke the next in the chain.
                                //拦截器尚未生效，直接skip
                                return proceed();
			}
		}
		else {
			// It's an interceptor, so we just invoke it: The pointcut will have
			// been evaluated statically before this object was constructed.
			return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
		}
	}
}
```

# 接口设计
## Advice接口设计 
Advice:AOP联盟定义的通知接口，即拦截到相应的方法后执行的动作。 

## MethodInterceptor接口设计 
再来看下MethodInterceptor这一重要接口，所有的advice都要最终转化成MethodInterceptor，它的invoke接口方法包含了拦截器要执行的内容及执行的顺序。 

## Advisor和Pointcut接口设计 
Advisor接口本身只含有Advice,而PointcutAdvisor则是将Pointcut和Advice组合了起来

Pointcut有两个属性，ClassFilter、MethodMatcher，一个用来过滤类，一个用来过滤方法。 
然后再看下Pointcut的实现类ComposablePointcut: 


```
public class ComposablePointcut implements Pointcut, Serializable {

	private ClassFilter classFilter;

	private MethodMatcher methodMatcher;
}
```


# 参考
https://my.oschina.net/pingpangkuangmo/blog/376305