源码篇

JUC，是**java.util.concurrent**⼯具包的简称，JDK 1.5开始出现，可以用于处理线程并发，包含atomic包、locks包和直接包下的其他工具类。

![1627786850701](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1627786850701.png)

- **并发基础**：Thread、Unsafe、Object、LockSupport、AQS、FutureTask、ThreadLocal、Executors。
- **典型实现**：ReentrantLock、ReentrantReadWriteLock、CountDownLatch、CyclicBarrier、Semaphore、AtomicInteger、ThreadPoolExecutor、ScheduledThreadPoolExecutor。

### 并发基础

#### Executors

##### 特点

- Executors，主要有此包中定义的 {@link Executor}、{@link ExecutorService}、{@link ScheduledExecutorService}、{@link ThreadFactory} 和 {@link Callable} 类的**工厂和实用方法**。
- Executors支持以下类型的方法：
  - 创建和返回一个 {@link ExecutorService} 的方法设置了常用的配置设置。
  - 使用常用配置设置创建和返回 {@link ScheduledExecutorService} 的方法。
  - 创建和返回“包装的”ExecutorService 的方法，通过使特定于实现的方法不可访问来**禁用重新配置**。
  - 创建并返回 {@link ThreadFactory} 的方法，该 {@link ThreadFactory} 将新创建的线程设置为已知状态。
  - 从其他类似闭包的形式创建和返回 {@link Callable} 的方法，因此它们可以用于需要 {@code Callable} 的执行方法中。

##### 构造方法

```java
public class Executors {
    // 无法实例化。
    private Executors() {}
}
```

##### 线程池便捷构造方法

###### newCachedThreadPool

```java
// 创建一个无核心线程、无界最大线程、无容量同步队列、默认线程工厂的线程池, 该实例强转后可以再次修改线程池参数
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}

// 创建一个无核心线程、无界最大线程、无容量同步队列、指定线程工厂的线程池, 该实例强转后可以再次修改线程池参数
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>(),
                                  threadFactory);
}
```

###### newFixedThreadPool

```java
// 创建一个固定线程数量、无界任务队列、默认线程工厂的线程池, 该实例强转后可以再次修改线程池参数
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

// 创建一个固定线程数量、无界任务队列、指定线程工厂的线程池, 该实例强转后可以再次修改线程池参数
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(),
                                  threadFactory);
}

// 创建一个单线程、无界任务队列、指定线程工厂的线程池, 由于该实例被包装类包装过, 所以该实例强转后不可以再次修改线程池参数
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>(),
                                threadFactory));
}

// 创建一个固定线程数量、无界任务队列、指定线程工厂的线程池, 该实例强转后可以再次修改线程池参数
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(),
                                  threadFactory);
}
```

###### newSingleThreadExecutor

```java
// 创建一个单线程、无界任务队列、默认线程工厂的线程池, 由于该实例被包装类包装过, 所以该实例强转后不可以再次修改线程池参数
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

###### newWorkStealingPool

```java
// 使用所有 {@link Runtime#availableProcessors 可用处理器} 作为其目标并行级别来创建窃取工作的线程池。
public static ExecutorService newWorkStealingPool() {
    return new ForkJoinPool
        (Runtime.getRuntime().availableProcessors(),
         ForkJoinPool.defaultForkJoinWorkerThreadFactory,
         null, true);
}

// 创建一个线程池，维护足够的线程以支持给定的并行度级别，并且可以使用多个队列来减少争用。
public static ExecutorService newWorkStealingPool(int parallelism) {
    return new ForkJoinPool
        (parallelism,
         ForkJoinPool.defaultForkJoinWorkerThreadFactory,
         null, true);
}
```

###### newSingleThreadScheduledExecutor

```java
// 创建一个单线程执行器，它可以安排命令在给定的延迟后运行, 或定期执行, 由于该实例被包装类包装过, 所以该实例强转后不可以再次修改线程池参数
public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
    return new DelegatedScheduledExecutorService
        (new ScheduledThreadPoolExecutor(1));
}

// 使用指定的线程工厂创建一个单线程执行器，它可以安排命令在给定的延迟后运行, 或定期执行, 由于该实例被包装类包装过, 所以该实例强转后不可以再次修改线程池参数
public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory) {
    return new DelegatedScheduledExecutorService
        (new ScheduledThreadPoolExecutor(1, threadFactory));
}
```

###### newScheduledThreadPool

```java
// 指定核心线程数创建一个线程池，可以安排命令在给定延迟后运行，或定期执行，该实例强转后可以再次修改线程池参数
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

// 指定核心线程数和线程工厂创建一个线程池，可以安排命令在给定延迟后运行，或定期执行，该实例强转后可以再次修改线程池参数
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory) {
    return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
}
```

###### unconfigurableExecutorService

```java
// 包装指定线程池, 以屏蔽ThreadPoolExecutor的参数设置方法
public static ExecutorService unconfigurableExecutorService(ExecutorService executor) {
    if (executor == null)
        throw new NullPointerException();
    return new DelegatedExecutorService(executor);
}
```

###### unconfigurableScheduledExecutorService

```java
// 包装指定线程池, 以屏蔽ScheduledExecutorService的参数设置方法
public static ScheduledExecutorService unconfigurableScheduledExecutorService(ScheduledExecutorService executor) {
    if (executor == null)
        throw new NullPointerException();
    return new DelegatedScheduledExecutorService(executor);
}
```

##### 线程池包装类

###### DelegatedExecutorService

```java
// 仅公开 ExecutorService 实现的 ExecutorService 方法的包装类
static class DelegatedExecutorService extends AbstractExecutorService {
    
    private final ExecutorService e;

    DelegatedExecutorService(ExecutorService executor) { e = executor; }

    public void execute(Runnable command) { e.execute(command); }
    
    public void shutdown() { e.shutdown(); }
    
    public List<Runnable> shutdownNow() { return e.shutdownNow(); }
    
    public boolean isShutdown() { return e.isShutdown(); }
    
    public boolean isTerminated() { return e.isTerminated(); }
    
    public boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException {
        return e.awaitTermination(timeout, unit);
    }
    public Future<?> submit(Runnable task) {
        return e.submit(task);
    }
    
    public <T> Future<T> submit(Callable<T> task) {
        return e.submit(task);
    }
    
    public <T> Future<T> submit(Runnable task, T result) {
        return e.submit(task, result);
    }
    
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException {
        return e.invokeAll(tasks);
    }
    
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                         long timeout, TimeUnit unit)
        throws InterruptedException {
        return e.invokeAll(tasks, timeout, unit);
    }
    
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException {
        return e.invokeAny(tasks);
    }
    
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                           long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {
        return e.invokeAny(tasks, timeout, unit);
    }
}
```

###### FinalizableDelegatedExecutorService

```java
// 仅公开 ExecutorService 实现的 ExecutorService 方法的包装类, 再加上finalize析构函数
static class FinalizableDelegatedExecutorService extends DelegatedExecutorService {

    FinalizableDelegatedExecutorService(ExecutorService executor) {
        super(executor);
    }

    protected void finalize() {
        super.shutdown();
    }
}
```

###### DelegatedScheduledExecutorService

```java
// 仅公开 ScheduledExecutorService 实现的 ScheduledExecutorService 方法的包装类
static class DelegatedScheduledExecutorService extends DelegatedExecutorService implements ScheduledExecutorService {

    private final ScheduledExecutorService e;

    DelegatedScheduledExecutorService(ScheduledExecutorService executor) {
        super(executor);
        e = executor;
    }

    public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit) {
        return e.schedule(command, delay, unit);
    }

    public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit) {
        return e.schedule(callable, delay, unit);
    }

    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit) {
        return e.scheduleAtFixedRate(command, initialDelay, period, unit);
    }

    public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit) {
        return e.scheduleWithFixedDelay(command, initialDelay, delay, unit);
    }
}
```

##### 获取线程工厂方法

###### defaultThreadFactory

```java
// 返回用于创建新线程的默认线程工厂
public static ThreadFactory defaultThreadFactory() {
    return new DefaultThreadFactory();
}
```

###### privilegedThreadFactory

```java
// 返回一个线程工厂，用于创建与当前线程具有相同权限的新线程
public static ThreadFactory privilegedThreadFactory() {
    return new PrivilegedThreadFactory();
}
```

##### 线程工厂实现类

###### DefaultThreadFactory

```java
// 默认线程工厂
static class DefaultThreadFactory implements ThreadFactory {

    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    // pool-[?: poolNumber]-thread-[?: threadNumber]
    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
                              Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
                      poolNumber.getAndIncrement() +
                     "-thread-";
    }

    // 默认创建优先级为5的非守护线程, 线程名称为pool-[?: poolNumber]-thread-[?: threadNumber]
    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                              namePrefix + threadNumber.getAndIncrement(),
                              0);

        // 默认非守护线程
        if (t.isDaemon())
            t.setDaemon(false);

        // 默认优先级为5
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

###### PrivilegedThreadFactory

```java
// 能够继承权限的线程工厂, 用于捕获访问、控制上下文和类加载器
static class PrivilegedThreadFactory extends DefaultThreadFactory {

    private final AccessControlContext acc;
    private final ClassLoader ccl;

    PrivilegedThreadFactory() {
        super();
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            // 从这个类调用 getContextClassLoader 永远不会触发安全检查，但我们会检查我们的调用者是否有这个权限。
            sm.checkPermission(SecurityConstants.GET_CLASSLOADER_PERMISSION);

            // Fail fast
            sm.checkPermission(new RuntimePermission("setContextClassLoader"));
        }
        this.acc = AccessController.getContext();
        this.ccl = Thread.currentThread().getContextClassLoader();
    }

    public Thread newThread(final Runnable r) {
        return super.newThread(new Runnable() {
            public void run() {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        Thread.currentThread().setContextClassLoader(ccl);
                        r.run();
                        return null;
                    }
                }, acc);
            }
        });
    }
}
```

##### 获取Callable实现类方法

###### RunnableAdapter实现类

```java
// 实现Callable接口, 运行给定任务, 并返回给定结果的可调用对象
static final class RunnableAdapter<T> implements Callable<T> {

    final Runnable task;
    final T result;

    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }

    public T call() {
        task.run();
        return result;
    }
}
```

###### callable(Runable, T)

```java
// 返回一个Callable对象, 该对象会在任务task运行完毕后返回, 包含任务task和指定的结果result
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}
```

###### callable(Runable)

```java
// 返回一个Callable对象, 该对象会在任务task运行完毕后返回, 包含任务task和null结果result
public static Callable<Object> callable(Runnable task) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<Object>(task, null);
}
```

###### callable(PrivilegedAction)

```java
// 返回一个Callable对象, 该对象会在action运行完毕后返回
public static Callable<Object> callable(final PrivilegedAction<?> action) {
    if (action == null)
        throw new NullPointerException();
    return new Callable<Object>() {
        public Object call() { return action.run(); }};
}
```

###### callable(PrivilegedExceptionAction)

```java
// 返回一个Callable对象, 该对象会在action运行完毕后返回
public static Callable<Object> callable(final PrivilegedExceptionAction<?> action) {
    if (action == null)
        throw new NullPointerException();
    return new Callable<Object>() {
        public Object call() throws Exception { return action.run(); }};
}
```

###### PrivilegedCallable实现类

```java
// 实现Callable接口, 能够在既定的访问控制设置下运行的可调用对象
static final class PrivilegedCallable<T> implements Callable<T> {

    private final Callable<T> task;
    private final AccessControlContext acc;

    PrivilegedCallable(Callable<T> task) {
        this.task = task;
        this.acc = AccessController.getContext();
    }

    public T call() throws Exception {
        try {
            return AccessController.doPrivileged(
                new PrivilegedExceptionAction<T>() {
                    public T run() throws Exception {
                        return task.call();
                    }
                }, acc);
        } catch (PrivilegedActionException e) {
            throw e.getException();
        }
    }
}
```

###### privilegedCallable(Callable)

```java
// 返回一个Callable对象, 该对象将在调用时在当前访问控制上下文下执行给定的 {@code callable}
public static <T> Callable<T> privilegedCallable(Callable<T> callable) {
    if (callable == null)
        throw new NullPointerException();
    return new PrivilegedCallable<T>(callable);
}
```

###### PrivilegedCallableUsingCurrentClassLoader实现类

```java
// 实现Callable接口, 能够在已建立的访问控制设置和当前ClassLoader下运行的可调用对象
static final class PrivilegedCallableUsingCurrentClassLoader<T> implements Callable<T> {

    private final Callable<T> task;
    private final AccessControlContext acc;
    private final ClassLoader ccl;

    PrivilegedCallableUsingCurrentClassLoader(Callable<T> task) {
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            // 从这个类调用 getContextClassLoader 永远不会触发安全检查，但我们会检查我们的调用者是否有这个权限。
            sm.checkPermission(SecurityConstants.GET_CLASSLOADER_PERMISSION);

            // 不管 setContextClassLoader 是否有必要，如果权限不可用，我们会很快失败。
            sm.checkPermission(new RuntimePermission("setContextClassLoader"));
        }
        
        this.task = task;
        this.acc = AccessController.getContext();
        this.ccl = Thread.currentThread().getContextClassLoader();
    }

    public T call() throws Exception {
        try {
            return AccessController.doPrivileged(
                new PrivilegedExceptionAction<T>() {
                    public T run() throws Exception {
                        Thread t = Thread.currentThread();
                        ClassLoader cl = t.getContextClassLoader();
                        if (ccl == cl) {
                            return task.call();
                        } else {
                            t.setContextClassLoader(ccl);
                            try {
                                return task.call();
                            } finally {
                                t.setContextClassLoader(cl);
                            }
                        }
                    }
                }, acc);
        } catch (PrivilegedActionException e) {
            throw e.getException();
        }
    }
}
```

###### privilegedCallableUsingCurrentClassLoader(Callable)

```java
// // 返回一个Callable对象, 该对象将在调用时在当前访问控制上下文下执行给定的 {@code callable}，并将当前上下文类加载器作为上下文类加载器
public static <T> Callable<T> privilegedCallableUsingCurrentClassLoader(Callable<T> callable) {
    if (callable == null)
        throw new NullPointerException();
    return new PrivilegedCallableUsingCurrentClassLoader<T>(callable);
}
```

