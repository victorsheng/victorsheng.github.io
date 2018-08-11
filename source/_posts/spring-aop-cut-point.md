title: spring框架中的aop应用
date: 2018-08-06 15:08:01
tags:
categories:
---
https://docs.spring.io/spring/docs/2.5.x/reference/aop.html


https://github.com/lfvepclr/spring-5-framework-doc/blob/master/37-Spring_AOP_Usage.md


# 应用
- 事务
- 异步任务
- 缓存
- factory
- content

# 事务

AnnotationTransactionAspect的集成关系:
```
TransactionAspectSupport (org.springframework.transaction.interceptor)
AbstractTransactionAspect (org.springframework.transaction.aspectj)
AnnotationTransactionAspect (org.springframework.transaction.aspectj)
```
AnnotationTransactionAspect定义了pointcut
```
public aspect AnnotationTransactionAspect extends AbstractTransactionAspect {

	public AnnotationTransactionAspect() {
		super(new AnnotationTransactionAttributeSource(false));
	}

	/**
	 * Matches the execution of any public method in a type with the Transactional
	 * annotation, or any subtype of a type with the Transactional annotation.
	 */
	private pointcut executionOfAnyPublicMethodInAtTransactionalType() :
		execution(public * ((@Transactional *)+).*(..)) && within(@Transactional *);

	/**
	 * Matches the execution of any method with the Transactional annotation.
	 */
	private pointcut executionOfTransactionalMethod() :
		execution(@Transactional * *(..));

	/**
	 * Definition of pointcut from super aspect - matched join points
	 * will have Spring transaction management applied.
	 */
	protected pointcut transactionalMethodExecution(Object txObject) :
		(executionOfAnyPublicMethodInAtTransactionalType() || executionOfTransactionalMethod() ) && this(txObject);

}

```
AbstractTransactionAspect定义了around执行的内部逻辑,其中invokeWithinTransaction方法为再上一级父类TransactionAspectSupport的方法
```
	@SuppressAjWarnings("adviceDidNotMatch")
	Object around(final Object txObject): transactionalMethodExecution(txObject) {
		MethodSignature methodSignature = (MethodSignature) thisJoinPoint.getSignature();
		// Adapt to TransactionAspectSupport's invokeWithinTransaction...
		try {
			return invokeWithinTransaction(methodSignature.getMethod(), txObject.getClass(), new InvocationCallback() {
				public Object proceedWithInvocation() throws Throwable {
					return proceed(txObject);
				}
			});
		}
		catch (RuntimeException | Error ex) {
			throw ex;
		}
		catch (Throwable thr) {
			Rethrower.rethrow(thr);
			throw new IllegalStateException("Should never get here", thr);
		}
	}
```

TransactionAspectSupport.invokeWithinTransaction的代码就比较清晰了，获取TransactionManager并调用它的execute方法，传入我们的callback，在callback中执行我们的事务方法。determineTransactionManager方法用于获取TransactionManagaer，当我们没有从TransactionManagementConfigurer中配置目标TransactionManager，通过其它方式确定一个TransactionManager，比如什么都没有配置的时候，默认从上下文（BeanFactory）中获取类型为PlatformTransactionManager的Bean作为默认的TransactionManager

TransactionAttributeSource
![upload successful](/images/pasted-226.png)

第一步：使用TransactionAttributeSource根据method和targetClass获取事务属性，上文已说过。 

第二步：获取合适的事务处理器 

首先如果该拦截器本身的事务拦截器不为空则直接使用，若为空则根据事务配置属性中的qualifier属性来匹配，如果没有再根据事务拦截器的transactionManagerBeanName来匹配，最后根据PlatformTransactionManager类型来匹配。 

第三步：joinpointIdentification则为类名和方法名的结合，主要用与log信息。 

第四步：根据事务管理器和事务属性创建事务。 


# 异步任务
针对@Async注解的使用
```
public aspect AnnotationAsyncExecutionAspect extends AbstractAsyncExecutionAspect {

	private pointcut asyncMarkedMethod() : execution(@Async (void || Future+) *(..));

	private pointcut asyncTypeMarkedMethod() : execution((void || Future+) (@Async *).*(..));

	public pointcut asyncMethod() : asyncMarkedMethod() || asyncTypeMarkedMethod();


	/**
	 * This implementation inspects the given method and its declaring class for the
	 * {@code @Async} annotation, returning the qualifier value expressed by {@link Async#value()}.
	 * If {@code @Async} is specified at both the method and class level, the method's
	 * {@code #value} takes precedence (even if empty string, indicating that the default
	 * executor should be used preferentially).
	 * @return the qualifier if specified, otherwise empty string indicating that the
	 * {@linkplain #setExecutor default executor} should be used
	 * @see #determineAsyncExecutor(Method)
	 */
	@Override
	protected String getExecutorQualifier(Method method) {
		// Maintainer's note: changes made here should also be made in
		// AnnotationAsyncExecutionInterceptor#getExecutorQualifier
		Async async = AnnotatedElementUtils.findMergedAnnotation(method, Async.class);
		if (async == null) {
			async = AnnotatedElementUtils.findMergedAnnotation(method.getDeclaringClass(), Async.class);
		}
		return (async != null ? async.value() : null);
	}


	declare error:
		execution(@Async !(void || Future+) *(..)):
		"Only methods that return void or Future may have an @Async annotation";

	declare warning:
		execution(!(void || Future+) (@Async *).*(..)):
		"Methods in a class marked with @Async that do not return void or Future will be routed synchronously";

}
```

其中父类逻辑:
```
public abstract aspect AbstractAsyncExecutionAspect extends AsyncExecutionAspectSupport {

	/**
	 * Create an {@code AnnotationAsyncExecutionAspect} with a {@code null}
	 * default executor, which should instead be set via {@code #aspectOf} and
	 * {@link #setExecutor}. The same applies for {@link #setExceptionHandler}.
	 */
	public AbstractAsyncExecutionAspect() {
		super(null);
	}


	/**
	 * Apply around advice to methods matching the {@link #asyncMethod()} pointcut,
	 * submit the actual calling of the method to the correct task executor and return
	 * immediately to the caller.
	 * @return {@link Future} if the original method returns {@code Future};
	 * {@code null} otherwise
	 */
	@SuppressAjWarnings("adviceDidNotMatch")
	Object around() : asyncMethod() {
		final MethodSignature methodSignature = (MethodSignature) thisJoinPointStaticPart.getSignature();

		AsyncTaskExecutor executor = determineAsyncExecutor(methodSignature.getMethod());
		if (executor == null) {
			return proceed();
		}

		Callable<Object> task = new Callable<Object>() {
			public Object call() throws Exception {
				try {
					Object result = proceed();
					if (result instanceof Future) {
						return ((Future<?>) result).get();
					}
				}
				catch (Throwable ex) {
					handleError(ex, methodSignature.getMethod(), thisJoinPoint.getArgs());
				}
				return null;
			}};

		return doSubmit(task, executor, methodSignature.getReturnType());
	}

	/**
	 * Return the set of joinpoints at which async advice should be applied.
	 */
	public abstract pointcut asyncMethod();

}

```
两个逻辑:
- 获取线程池:从ConcurrentHashMap中获取当前方法的线程池

```
protected AsyncTaskExecutor determineAsyncExecutor(Method method) {
		AsyncTaskExecutor executor = this.executors.get(method);
		if (executor == null) {
			Executor targetExecutor;
			String qualifier = getExecutorQualifier(method);
			if (StringUtils.hasLength(qualifier)) {
				targetExecutor = findQualifiedExecutor(this.beanFactory, qualifier);
			}
			else {
				targetExecutor = this.defaultExecutor;
				if (targetExecutor == null) {
					synchronized (this.executors) {
						if (this.defaultExecutor == null) {
							this.defaultExecutor = getDefaultExecutor(this.beanFactory);
						}
						targetExecutor = this.defaultExecutor;
					}
				}
			}
			if (targetExecutor == null) {
				return null;
			}
			executor = (targetExecutor instanceof AsyncListenableTaskExecutor ?
					(AsyncListenableTaskExecutor) targetExecutor : new TaskExecutorAdapter(targetExecutor));
			this.executors.put(method, executor);
		}
		return executor;
	}


```
- 提交任务:根据返回值的类型 提交执行任务
```
	@Nullable
	protected Object doSubmit(Callable<Object> task, AsyncTaskExecutor executor, Class<?> returnType) {
		if (CompletableFuture.class.isAssignableFrom(returnType)) {
			return CompletableFuture.supplyAsync(() -> {
				try {
					return task.call();
				}
				catch (Throwable ex) {
					throw new CompletionException(ex);
				}
			}, executor);
		}
		else if (ListenableFuture.class.isAssignableFrom(returnType)) {
			return ((AsyncListenableTaskExecutor) executor).submitListenable(task);
		}
		else if (Future.class.isAssignableFrom(returnType)) {
			return executor.submit(task);
		}
		else {
			executor.submit(task);
			return null;
		}
	}

```

