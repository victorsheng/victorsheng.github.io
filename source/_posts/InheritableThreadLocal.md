---
title: InheritableThreadLocal
date: 2018-08-17 11:38:48
tags:
categories:
---
# 介绍
inherit 英文解释: **继承；遗传而得**

InheritableThreadLocal
- 可以在子线程和父线程之间共享实例，也同样是为了减少参数的传递
- 会复制父类的InheritableThreadLocal,但是不会影响父类的InheritableThreadLocal


# 测试用例
```
public static void main(String[] args) {
    final InheritableThreadLocal<Span> inheritableThreadLocal = new InheritableThreadLocal<Span>();
    inheritableThreadLocal.set(new Span("xiexiexie"));
    //输出 xiexiexie
    Object o = inheritableThreadLocal.get();
    System.out.println(o);
    Runnable runnable = new Runnable() {
      @Override
      public void run() {
        System.out.println("========");
        System.out.println(inheritableThreadLocal.get());
        inheritableThreadLocal.set(new Span("zhangzhangzhang"));
        System.out.println(inheritableThreadLocal.get());
      }
    };
    ExecutorService executorService = Executors.newFixedThreadPool(1);
    try {
      executorService.submit(runnable);
      TimeUnit.SECONDS.sleep(1);
      executorService.submit(runnable);
      TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    System.out.println("========");
    System.out.println(inheritableThreadLocal.get());

  }

  static class Span {

    public String name;
    public int age;

    public Span(String name) {
      this.name = name;
    }


    @Override
    public String toString() {
      final StringBuilder sb = new StringBuilder("Span{");
      sb.append("name='").append(name).append('\'');
      sb.append(", age=").append(age);
      sb.append('}');
      return sb.toString();
    }
  }

````
输出结果
```
Span{name='xiexiexie', age=0}
========
Span{name='xiexiexie', age=0}
Span{name='zhangzhangzhang', age=0}
========
Span{name='zhangzhangzhang', age=0}
Span{name='zhangzhangzhang', age=0}
========
Span{name='xiexiexie', age=0}

```


# Thread的初始化过程

```
private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        this.name = name;

        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();
        if (g == null) {
            /* Determine if it's an applet or not */

            /* If there is a security manager, ask the security manager
               what to do. */
            if (security != null) {
                g = security.getThreadGroup();
            }

            /* If the security doesn't have a strong opinion of the matter
               use the parent thread group. */
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }

        /* checkAccess regardless of whether or not threadgroup is
           explicitly passed in. */
        g.checkAccess();

        /*
         * Do we have the required permissions?
         */
        if (security != null) {
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }

        g.addUnstarted();

        this.group = g;
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext =
                acc != null ? acc : AccessController.getContext();
        this.target = target;
        setPriority(priority);
        //创建新的ThreadLocalMap,复制inheritableThreadLocals的内容到ThreadLocalMap里面
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;

        /* Set thread ID */
        tid = nextThreadID();
    }
```

# 父线程,通过包装Runnable和Callable接口
```
/**
   * If task run in {@link Executor} thread pool, use this method for propagating security context
   * 
   * @param task
   * @return
   */
  public static Runnable decorateRunnable(Runnable task) {
    Optional<OPALProcessContext> context = get();
    return () -> {
      try {
        context.ifPresent(OPALProcessContext::set);
        task.run();
      } finally {
        clear();
      }
    };
  }

  /**
   * If task run in {@link Executor} thread pool, use this method for propagating security context
   * 
   * @param task
   * @return
   */
  public static <V> Callable<V> decorateCallable(Callable<V> task) {
    Optional<OPALProcessContext> context = get();
    return () -> {
      try {
        context.ifPresent(OPALProcessContext::set);
        return task.call();
      } finally {
        clear();
      }
    };
  }
```