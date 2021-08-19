## 源码篇

JUC，是**java.util.concurrent**⼯具包的简称，JDK 1.5开始出现，可以用于处理线程并发，包含atomic包、locks包和直接包下的其他工具类。

![1627786850701](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1627786850701.png)

- **并发基础**：Thread、Unsafe、Object、LockSupport、AQS。
- **典型实现**：ReentrantLock、ReentrantReadWriteLock、CountDownLatch、CyclicBarrier、Semaphore、Exchanger、Atomic*、ThreadPoolExecutor、Executors、FutureTask、CompletableFuture、RecursiveTask。

### 并发基础

#### Object

##### 特点

- 类 {@code Object} 是类层次结构的根， 每个类都会以 {@code Object} 作为超类，所有对象（包括数组）都实现了这个类的方法。

```java
public class Object {...}
```

##### 获取Class对象方法

```java
// 返回此对象的运行时Class对象
public final native Class<?> getClass();
```

##### 获取对象哈希码方法

```java
// 返回对象的哈希码值, 主要是为了有利于散列表, 例如 {@link java.util.HashMap} 提供的散列表。
public native int hashCode();
```

##### 对象等值判断方法

- ##### **自反性**：对于任何非空引用值 {@code x}，{@code x.equals(x)} 应该返回 {@code true}。

- **对称性**：对于任何非空引用值 {@code x} 和 {@code y}，{@code x.equals(y)} 返回 {@code true}时，  {@code y.equals (x)} 也应该返回 {@code true}。

- **可传递性**：对于任何非空引用值 {@code x}、{@code y} 和 {@code z}，如果 {@code x.equals(y)} 返回 {@code true} 和{@Code y.equals(z)} 返回 {@code true}，那么 {@code x.equals(z)} 应该返回 {@code true}。

- **一致性**：

  - 对于任何非空引用值 {@code x} 和 {@code y}，如果未修改对象的 {@code equals} 比较中使用的信息，那么多次调用 {@code x.equals(y)} 始终返回 {@code true} 或始终返回 {@code false }。
  - 对于任何非空引用值 {@code x}，如果未修改对象的 {@code equals} 比较中使用的信息，那么{@code x.equals(null)} 应返回 {@code false}。

```java
// 判断某个对象是否“等于”这个对象, 默认实现比较的是引用是否相等 => 覆盖该方法时, 通常需要维护这个约定: 相等的对象必须具有相同的哈希码
public boolean equals(Object obj) {
    return (this == obj);
}
```

##### 获取对象副本方法

- {@code Object}类的方法{@code clone}执行特定的克隆操作，如果类没有实现接口{@code Cloneable}，则抛出一个{@code CloneNotSupportedException}。
- **所有数组**都被认为实现了接口{@code Cloneable}，而类 {@code Object}本身并没有实现接口 {@code Cloneable}，因此在类为{@code Object}的对象上调用{@code clone}方法将导致在运行时抛出异常。
- 此方法会创建此类的新实例，并使用此对象的相应字段的内容来初始化其所有字段，**就像通过赋值一样**，即其字段的内容本身不会被克隆。 因此，此方法执行此对象的“**浅拷贝**”，而不是“深拷贝”操作。

```java
// 创建并返回该对象的副本, 执行的是“浅拷贝”, 而不是“深拷贝”, 即x.clone() != x, 但x.clone().getClass() == x.getClass(), 以及x.clone().equals(x), 因为x与副本不是同一个对象, 但对象实例字段是同一个对象
protected native Object clone() throws CloneNotSupportedException;
```

##### 获取易于阅读对象的字符串方法

```java
// 返回易于人们阅读的该对象的字符串
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

##### 管程相关方法

```java
// 唤醒该对象监视器上任意一个等待的线程, 只能由作为此对象监视器的所有者线程调用(通过synchronized获取): 一次只有一个线程可以拥有一个对象的监视器, 但监视器上可以有多个等待线程(通过await方法成为等待线程)
public final native void notify();

// 唤醒该对象监视器上所有等待的线程, 只能由作为此对象监视器的所有者线程调用(通过synchronized获取): 一次只有一个线程可以拥有一个对象的监视器, 但监视器上可以有多个等待线程(通过await方法成为等待线程)
public final native void notifyAll();

// 当前线程阻塞等待该对象调用notify、notifyAll、中断或者指定时间过去(为0时需要一直等待), 只能由作为此对象监视器的所有者线程调用(通过synchronized获取): 一次只有一个线程可以拥有一个对象的监视器, 但监视器上可以有多个等待线程(通过await方法成为等待线程)
public final native void wait(long timeout) throws InterruptedException;

// 当前线程阻塞等待该对象调用notify、notifyAll、中断或者指定时间过去(timeout毫秒加上nanos纳秒), 只能由作为此对象监视器的所有者线程调用(通过synchronized获取): 一次只有一个线程可以拥有一个对象的监视器, 但监视器上可以有多个等待线程(通过await方法成为等待线程)
public final void wait(long timeout, int nanos) throws InterruptedException {
    if (timeout < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
            "nanosecond timeout value out of range");
    }

    if (nanos > 0) {
        timeout++;
    }

    wait(timeout);
}

// 当前线程阻塞等待该对象调用notify、notifyAll或者中断, 只能由作为此对象监视器的所有者线程调用(通过synchronized获取): 一次只有一个线程可以拥有一个对象的监视器, 但监视器上可以有多个等待线程(通过await方法成为等待线程)
public final void wait() throws InterruptedException {
    wait(0);
}
```

##### 对象析构方法

```java
// 当垃圾收集器确定不再有对该对象的引用时，由垃圾收集器在对象上调用, 其中子类可以覆盖 {@code finalize} 方法, 来处理系统资源或执行其他清理操作。
protected void finalize() throws Throwable { }
```

#### Thread

##### Runable

- Runable接口旨在，通过通用协议规范一个，**可在线程活动时执行代码的对象**。
- 实现Runnable接口的任务必须实现run（），且该方法可以被Thread线程执行。
- 由于Thread类实现了Runable接口，因此可以通过实例化Thread实例，并将其自身作为任务传入并运行；用户也可以实现Runnable接口，使得在不继承Thread的情况下运行任务。
- 在大多数情况下，如果只打算覆盖Runable#run（）方法，而不打算覆盖其他Thread方法时，则应该实现Runnable接口。除非打算修改或增强Thread类的基本行为，否则Thread类不应被子类化。

```java
@FunctionalInterface
public interface Runnable {
    
    // 当传入了实现Runnable接口的任务对象的线程启动时，其Thread#run（）会调用该Runable#run（）
    public abstract void run();
}
```

![1627987676498](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1627987676498.png)

##### 特点

- Thread，是**JVM程序中的执行线程**，在JVM中允许同时运行多个执行线程。
- 每个线程都有一个优先级。具有较高优先级的线程，会优先于具有较低优先级的线程执行。当在某个线程中运行的代码创建一个新的Thread对象时，**新线程的优先级最初设置为等于创建线程的优先级**。
- 每个线程可能也可能不标记为守护进程，当在某个线程中运行的代码创建一个新的Thread对象时，当且仅当**创建线程是守护进程时，新线程才是守护线程**。
- 每个线程都有一个用于识别的名称，多个线程可能具有相同的名称。而如果在创建线程时未指定名称，则会为其生成一个新名称。
- 当JVM启动时，通常会有一个非守护线程（通常是调用类的main方法）。该主线程会继续执行，直到发生已调用的**main方法退出**或者**非守护线程全部死亡**。
- Java 8源码备注中，创建执行线程的方法只有两种，一种是**继承Thread类**，重写Thread#run方法；另一种是**实现Runable接口**的run方法。

##### 构造方法

1. **空参构造函数**：先获取当前Thread线程的引用，然后初始化线程，默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等，然后设置指定的运行任务和生成线程ID，以及线程名称"Thread-n"。
2. **指定运行任务的构造函数(如果为null则执行Thread#run)**：先获取当前Thread线程的引用，然后初始化线程，默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等，然后设置指定的运行任务和生成线程ID，以及线程名称"Thread-n"。
3. **指定线程组和运行任务(如果为null则执行Thread#run)的构造函数**：先获取当前Thread线程的引用，然后初始化线程，默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等，然后设置指定的运行任务和生成线程ID，以及线程名称"Thread-n"。
4. **指定线程名称的构造函数**：先获取当前Thread线程的引用，然后初始化线程，默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等，然后设置指定的运行任务和生成线程ID，以及线程名称"Thread-n"。
5. **指定线程组和线程名称的构造函数**：先获取当前Thread线程的引用，然后初始化线程，默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等，然后设置指定的运行任务和生成线程ID，以及线程名称"Thread-n"。
6. **指定运行任务(如果为null则执行Thread#run)和线程名的构造函数**：先获取当前Thread线程的引用，然后初始化线程，默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等，然后设置指定的运行任务和生成线程ID，以及线程名称"Thread-n"。
7. **指定线程组、运行任务(如果为null则执行Thread#run)和线程名的构造函数**：先获取当前Thread线程的引用，然后初始化线程，默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等，然后设置指定的运行任务和生成线程ID，以及线程名称"Thread-n"。
8. **指定线程组、运行任务(如果为null则执行Thread#run)和线程名以及堆栈大小(会有平台相关性)的构造函数**：先获取当前Thread线程的引用，然后初始化线程，默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等，然后设置指定的运行任务和生成线程ID，以及线程名称"Thread-n"。

```java
public class Thread implements Runnable {
    private volatile char  name[];
    private int            priority;
    private boolean     daemon = false;
    private Runnable target;
    private ThreadGroup group;
    private ClassLoader contextClassLoader;
    ThreadLocal.ThreadLocalMap threadLocals = null;
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
    private long tid;
    private volatile int threadStatus = 0;// 用于组合VM类常量, 以得出线程的State状态
    
    // 线程最低、默认、最高优先级
    public final static int MIN_PRIORITY = 1;
    public final static int NORM_PRIORITY = 5;
    public final static int MAX_PRIORITY = 10;
    
    // 空参构造函数, 先获取当前Thread线程的引用, 然后初始化线程, 默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等, 然后设置指定的运行任务和生成线程ID, 以及线程名称"Thread-n"
    public Thread() {
        init(null, null, "Thread-" + nextThreadNum(), 0);
    }
    
    // 指定运行任务的构造函数(如果为null则执行Thread#run), 先获取当前Thread线程的引用, 然后初始化线程, 默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等, 然后设置指定的运行任务和生成线程ID, 以及线程名称"Thread-n"
    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }
    
    // 指定线程组和运行任务(如果为null则执行Thread#run)的构造函数, 先获取当前Thread线程的引用, 然后初始化线程, 默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等, 然后设置指定的运行任务和生成线程ID, 以及线程名称"Thread-n"
    public Thread(ThreadGroup group, Runnable target) {
        init(group, target, "Thread-" + nextThreadNum(), 0);
    }
    
    // 指定线程名称的构造函数, 先获取当前Thread线程的引用, 然后初始化线程, 默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等, 然后设置指定的运行任务和生成线程ID, 以及线程名称"Thread-n"
    public Thread(String name) {
        init(null, null, name, 0);
    }

    // 指定线程组和线程名称的构造函数, 先获取当前Thread线程的引用, 然后初始化线程, 默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等, 然后设置指定的运行任务和生成线程ID, 以及线程名称"Thread-n"
    public Thread(ThreadGroup group, String name) {
        init(group, null, name, 0);
    }

    // 指定运行任务(如果为null则执行Thread#run)和线程名的构造函数, 先获取当前Thread线程的引用, 然后初始化线程, 默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等, 然后设置指定的运行任务和生成线程ID, 以及线程名称"Thread-n"
    public Thread(Runnable target, String name) {
        init(null, target, name, 0);
    }
    
    // 指定线程组、运行任务(如果为null则执行Thread#run)和线程名的构造函数, 先获取当前Thread线程的引用, 然后初始化线程, 默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等, 然后设置指定的运行任务和生成线程ID, 以及线程名称"Thread-n"
    public Thread(ThreadGroup group, Runnable target, String name) {
        init(group, target, name, 0);
    }
    
    // 指定线程组、运行任务(如果为null则执行Thread#run)和线程名以及堆栈大小(会有平台相关性)的构造函数, 先获取当前Thread线程的引用, 然后初始化线程, 默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等, 然后设置指定的运行任务和生成线程ID, 以及线程名称"Thread-n"
    public Thread(ThreadGroup group, Runnable target, String name, long stackSize) {
        init(group, target, name, stackSize);
    }
    
    // 先获取当前Thread线程的引用, 然后初始化线程, 默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等, 然后设置指定的运行任务和生成线程ID
    private void init(ThreadGroup g, Runnable target, String name, long stackSize) {
        init(g, target, name, stackSize, null);
    }
    
    // 先获取当前Thread线程的引用, 然后初始化线程, 默认使用当前线程的线程组、守护线程标记、优先级、线程上下文类加载器、权限上下文等, 然后设置指定的运行任务和生成线程ID
    private void init(ThreadGroup g, Runnable target, String name, long stackSize, AccessControlContext acc) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        // 根据传入的name赋值给线程名称数组
        this.name = name.toCharArray();

        // 获取当前线程实例, 作为新初始化线程的父线程
        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();

        // 如果没有指定线程组, 则使用当前线程的线程组, 作为新初始化线程的线程组
        if (g == null) {
            // 确定它是否是小程序，如果有安全经理，请询问安全经理要做什么。
            if (security != null) {
                g = security.getThreadGroup();
            }

            // 如果安全性对此事没有强烈意见，请使用父线程组。
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }

        // checkAccess无论是否显式传入线程组。
        // 确定当前运行的线程是否有权限修改这个线程组
        g.checkAccess();

        // 我们是否拥有所需的权限？
        if (security != null) {
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }

        // 增加线程组中未启动线程的计数, 未启动的线程不会被添加到线程组中, 但必须对其进行计数, 以便其中包含未启动线程的守护线程组不会被销毁
        g.addUnstarted();

        // 设置线程组、守护线程标记(取当前线程的)、优先级(取当前线程的)、线程上下文类加载器、权限上下文、运行的目标任务、InheritableThreadLocal值、请求的堆栈大小、以及生成线程ID
        this.group = g;
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext = acc != null ? acc : AccessController.getContext();
        this.target = target;
        setPriority(priority);
        if (parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        
        // 存储指定的堆栈大小以防 VM 关心
        this.stackSize = stackSize;

        // 设置线程 ID
        tid = nextThreadID();
    }
}

// 返回对当前正在执行的线程对象的引用
public static native Thread currentThread();

// 如果实例线程为守护线程，则为true; 如果实例线程为用户线程, 则为false
public final boolean isDaemon() {
    return daemon;
}

// 返回实例线程的优先级
public final int getPriority() {
    return priority;
}

// 获取实例线程的上下文类加载器, 以供实例线程中运行的代码在加载类和资源时使用
public ClassLoader getContextClassLoader() {
    if (contextClassLoader == null)
        return null;
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        ClassLoader.checkClassLoaderPermission(contextClassLoader,
                                               Reflection.getCallerClass());
    }
    return contextClassLoader;
}

// 用于生成线程ID
private static long threadSeqNumber;
private static synchronized long nextThreadID() {
    return ++threadSeqNumber;
}
```

##### 生命周期方法

###### 启动

```java
// 开始执行该Thread执行线程, JVM调用其run方法, 而多次启动一个Thread执行线程是不合法的
public synchronized void start() {
    // 工具的Java线程状态，0表示线程“尚未启动”, 因此启动过的线程是不能被再次启动的
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    // 将指定线程添加到线程组中
    group.add(this);

    // 调用native方法, 启动该Thread执行线程
    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                // 通知组线程 {@code t} 尝试启动失败, 线程组的状态被回滚，就好像从未发生过启动线程的尝试一样
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            // 没做什么。如果start0抛出一个Throwable那么它将被传递到调用堆栈
        }
    }
}

// native方法, 启动该Thread执行线程
private native void start0();
```

###### 运行

```java
// 如果运行的目标任务不为null, 则调用Runnable#run方法; 如果子类去重写这个方法, 则是通过多态调用来执行了(不过Thread类本身也是个Runnable实现)
@Override
public void run() {
    // 如果运行的目标任务不为null, 则调用Runnable#run方法
    if (target != null) {
        target.run();
    }
}
```

###### 让步CPU

```java
// 向调度程序提示, 当前线程愿意放弃其当前对处理器的使用, 而调度程序可以随意忽略此提示
public static native void yield();
```

###### 休眠

```java
// 使当前正在执行的线程休眠（暂时停止执行）指定的毫秒数(timeout毫秒加上nanos纳秒)，且当前线程不会失去任何监视器的所有权
public static void sleep(long millis, int nanos) throws InterruptedException {
    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
            "nanosecond timeout value out of range");
    }

    if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
        millis++;
    }

    sleep(millis);
}

// 使当前正在执行的线程休眠（暂时停止执行）指定的毫秒数，且当前线程不会失去任何监视器的所有权
public static native void sleep(long millis) throws InterruptedException;
```

###### 等待实例线程执行完毕

- **join（）**：当前线程**永远等待**实例线程死亡，实质上是调用当前Thread实例的wait方法进行阻塞等待，因此建议应用程序不要在{@code Thread}实例上使用{@code wait}、{@code notify}或{@code notifyAll}。如果Thread实例被销毁，则其等待集也会被销毁，从而使得当前线程得以释放，唤醒该线程，又由于Thread实例已死亡，则会退出循环，停止等待。
- **join（long，int）**：当前线程**等待(millis毫秒+nanos纳秒)**让实例线程死亡，实质上是调用当前Thread实例的wait方法进行阻塞等待，因此建议应用程序不要在{@code Thread}实例上使用{@code wait}、{@code notify}或{@code notifyAll}。如果Thread实例被销毁，则其等待集也会被销毁，从而使得当前线程得以释放， 唤醒该线程，又由于Thread实例已死亡，则会退出循环，停止等待。
- **join（long）**：当前线程**等待millis毫秒**让实例线程死亡，为0则意味着永远等待，实质上是调用当前Thread实例的wait方法进行阻塞等待，因此建议应用程序不要在{@code Thread}实例上使用{@code wait}、{@code notify}或{@code notifyAll}。如果Thread实例被销毁，则其等待集也会被销毁，从而使得当前线程得以释放，唤醒该线程，又由于Thread实例已死亡，则会退出循环，停止等待。

```java
// 当前线程永远等待实例线程死亡, 实质上是调用当前Thread实例的wait方法进行阻塞等待, 因此建议应用程序不要在{@code Thread}实例上使用{@code wait}、{@code notify}或{@code notifyAll}。如果Thread实例被销毁, 则其等待集也会被销毁, 从而使得当前线程得以释放, 唤醒该线程, 又由于Thread实例已死亡, 则会退出循环, 停止等待
public final void join() throws InterruptedException {
    join(0);
}

// 当前线程等待(millis毫秒+nanos纳秒)让实例线程死亡, 实质上是调用当前Thread实例的wait方法进行阻塞等待, 因此建议应用程序不要在{@code Thread}实例上使用{@code wait}、{@code notify}或{@code notifyAll}。如果Thread实例被销毁, 则其等待集也会被销毁, 从而使得当前线程得以释放, 唤醒该线程, 又由于Thread实例已死亡, 则会退出循环, 停止等待
public final synchronized void join(long millis, int nanos) throws InterruptedException {
    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }
    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
            "nanosecond timeout value out of range");
    }
    if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
        millis++;
    }

    // 当前线程等待millis毫秒让实例线程死亡, 为0则意味着永远等待, 实质上是调用当前Thread实例的wait方法进行阻塞等待, 因此建议应用程序不要在{@code Thread}实例上使用{@code wait}、{@code notify}或{@code notifyAll}。
    // 如果Thread实例被销毁, 则其等待集也会被销毁, 从而使得当前线程得以释放, 唤醒该线程, 又由于Thread实例已死亡, 则会退出循环, 停止等待
    join(millis);
}

// 当前线程等待millis毫秒让实例线程死亡, 为0则意味着永远等待, 实质上是调用当前Thread实例的wait方法进行阻塞等待, 因此建议应用程序不要在{@code Thread}实例上使用{@code wait}、{@code notify}或{@code notifyAll}。如果Thread实例被销毁, 则其等待集也会被销毁, 从而使得当前线程得以释放, 唤醒该线程, 又由于Thread实例已死亡, 则会退出循环, 停止等待
public final synchronized void join(long millis) throws InterruptedException {
    // 获取系统当前毫秒数
    long base = System.currentTimeMillis();
    long now = 0;
    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    // 如果指定的millis为0, 说明需要永久阻塞等待, 直到该Thread实例对象调用notify、notifyAll、中断或者指定时间过去(为0时需要一直等待)
    if (millis == 0) {
        // 判断该实例线程是否还存活
        while (isAlive()) {
            // 当前线程阻塞等待该Thread实例对象调用notify、notifyAll、中断或者指定时间过去(为0时需要一直等待), 只能由作为此对象监视器的所有者线程调用(通过synchronized获取): 一次只有一个线程可以拥有一个对象的监视器, 但监视器上可以有多个等待线程(通过await方法成为等待线程)
            // 如果Thread实例被销毁, 则其等待集也会被销毁, 从而使得当前线程得以释放, 唤醒该线程, 又由于Thread实例已死亡, 则会退出循环, 停止等待
            wait(0);
        }
    }
    // 如果指定的millis大于0, 说明需要定时等待, 直到该Thread实例对象调用notify、notifyAll、中断或者指定时间过去(为0时需要一直等待)
    else {
        // 判断该实例线程是否还存活
        while (isAlive()) {
            // 过期时长 = 指定的时长 - 现在时长(默认为0)
            long delay = millis - now;

            // 如果过期时长 < 0, 则退出循环
            if (delay <= 0) {
                break;
            }

            // 当前线程阻塞等待该Thread实例对象调用notify、notifyAll、中断或者指定时间过去(为0时需要一直等待), 只能由作为此对象监视器的所有者线程调用(通过synchronized获取): 一次只有一个线程可以拥有一个对象的监视器, 但监视器上可以有多个等待线程(通过await方法成为等待线程)
            // 如果Thread实例被销毁, 则其等待集也会被销毁, 从而使得当前线程得以释放, 唤醒该线程, 又由于Thread实例已死亡, 则会退出循环, 停止等待
            wait(delay);

            // 重新计算现在时长 = 系统最新的毫秒数 - 过去系统的毫秒数
            now = System.currentTimeMillis() - base;
        }
    }
}
```

###### 中断方法

```java
// 中断该线程, 如果从Thread其他实例方法调用该方法, 则会清除中断状态, 然后会收到一个{@link InterruptedException}
public void interrupt() {
    if (this != Thread.currentThread())
        checkAccess();

    synchronized (blockerLock) {
        Interruptible b = blocker;
        if (b != null) {
            interrupt0();           // Just to set the interrupt flag
            b.interrupt(this);
            return;
        }
    }
    interrupt0();
}

// 一些私有native帮助方法
private native void interrupt0();
```

###### 退出

**系统调用。**

```java
// 系统调用, 用于线程实际退出之前做最后的清理工作
private void exit() {
    // 如果线程组不为空, 则通知组线程{@code t}尝试启动失败, 并清空线程组
    if (group != null) {
        group.threadTerminated(this);
        group = null;
    }

    // 积极清除所有引用字段：参见错误 4006245
    target = null;

    // 释放其中一些资源
    threadLocals = null;
    inheritableThreadLocals = null;
    inheritedAccessControlContext = null;
    blocker = null;
    uncaughtExceptionHandler = null;
}
```

###### 强制停止

- **stop（）**：强制使实例线程停止执行，被迫停止它任何正在执行的操作，并将新创建的ThreadDeath Error对象作为Throwable抛出，**已被废弃**，因为会出现数据不同步，所以可以使用同步变量作为停止标记来实现线程停止(Thread#isInterrupted()本质上就是个中断标记)。
- **stop（Throwable）**：强制线程停止并将给定的 {@code Throwable} 作为异常抛出，**已被废弃**，因为可能会生成目标线程未准备好处理的异常。

```java
// 强制使实例线程停止执行, 被迫停止它任何正在执行的操作, 并将新创建的ThreadDeath Error对象作为Throwable抛出, 已被废弃, 因为会出现数据不同步, 所以可以使用同步变量作为停止标记来实现线程停止(Thread#isInterrupted()本质上就是个中断标记)
@Deprecated
public final void stop() {
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        checkAccess();
        if (this != Thread.currentThread()) {
            security.checkPermission(SecurityConstants.STOP_THREAD_PERMISSION);
        }
    }

    // 零状态值对应于“NEW”，它不能更改为not-NEW，因为我们持有锁。
    if (threadStatus != 0) {
        // 恢复挂起的线程, 已被弃用，因为容易导致死锁: 当该线程持有锁被挂起(保持等待), 而另一个线程需要请求该锁才能恢复该线程, 则发生死锁
        resume(); // Wake up thread if it was suspended; no-op otherwise // 如果线程被挂起，则唤醒线程；否则无操作
    }

    // VM 可以处理所有线程状态
    stop0(new ThreadDeath());
}

// 一些私有native帮助方法
private native void stop0(Object o);

// 强制线程停止并将给定的 {@code Throwable} 作为异常抛出, 已被废弃, 因为可能会生成目标线程未准备好处理的异常
@Deprecated
public final synchronized void stop(Throwable obj) {
    throw new UnsupportedOperationException();
}
```

###### 销毁

**已被废弃。**

```java
// 用于在不进行任何清理的情况下销毁实例线程, 该方法未做未实现, 已被废弃, 因为和suspend一样容易导致死锁, 会在线程资源被销毁后仍然持有锁
@Deprecated
public void destroy() {
    throw new NoSuchMethodError();
}
```

###### 暂停

**已被废弃。**

```java
// 暂停该线程, 已被弃用，因为容易导致死锁: 当该线程持有锁被挂起(保持等待), 而另一个线程需要请求该锁才能恢复该线程, 则发生死锁
@Deprecated
public final void suspend() {
    checkAccess();
    suspend0();
}

// 一些私有native帮助方法
private native void suspend0();
```

###### 恢复

**已被废弃。**

```java
// 恢复挂起的线程, 已被弃用，因为容易导致死锁: 当该线程持有锁被挂起(保持等待), 而另一个线程需要请求该锁才能恢复该线程, 则发生死锁
@Deprecated
public final void resume() {
    checkAccess();
    resume0();
}

// 一些私有native帮助方法
private native void resume0();
```

##### 监控方法

###### 判断是否中断

```java
// 测试当前线程是否被中断, 该方法会清除线程的中断状态
public static boolean interrupted() {
    return currentThread().isInterrupted(true);
}

// 测试当前线程是否被中断, 该方法不会清除线程的中断状态
public boolean isInterrupted() {
    return isInterrupted(false);
}

// 测试线程是否已被中断, ClearInterrupted为true时会清除线程的中断状态
private native boolean isInterrupted(boolean ClearInterrupted);
```

###### 判断是否存活

```java
// 判断该线程是否还存活
public final native boolean isAlive();
```

###### 判断是否为守护线程

```java
// 如果实例线程为守护线程，则为true; 如果实例线程为用户线程, 则为false
public final boolean isDaemon() {
    return daemon;
}
```

##### 其他辅助方法

###### 设置/获取线程优先级

```java
// 更改此线程的优先级(1<x<10), 如果指定的优先级大于线程组最大优先级, 则使用线程组最大的优先级
public final void setPriority(int newPriority) {
    ThreadGroup g;
    checkAccess();

    // 如果指定的优先级大于10, 或者小于1, 则抛出异常
    if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
        throw new IllegalArgumentException();
    }

    // 获取该线程组, 如果指定的优先级大于线程组最大优先级, 则使用线程组最大的优先级
    if((g = getThreadGroup()) != null) {
        if (newPriority > g.getMaxPriority()) {
            newPriority = g.getMaxPriority();
        }
        setPriority0(priority = newPriority);
    }
}

// 返回实例线程的优先级
public final int getPriority() {
    return priority;
}

// 一些私有native帮助方法
private native void setPriority0(int newPriority);
```

###### 设置/获取线程名称

```java
// 将实例线程名称更改指定名称
public final synchronized void setName(String name) {
    checkAccess();
    this.name = name.toCharArray();
    if (threadStatus != 0) {
        setNativeName(name);
    }
}

// 一些私有native帮助方法
private native void setNativeName(String name);
```

###### 获取线程组

```java
// 返回实例线程所属的线程组, 如果线程已死亡（已停止）, 则返回null
public final ThreadGroup getThreadGroup() {
    return group;
}
```

###### 标记为守护线程

```java
// 在线程启动之前, 将实例线程标记为守护线程/用户线程, 如果{@code true}, 则将此线程标记为守护线程
public final void setDaemon(boolean on) {
    checkAccess();
    if (isAlive()) {
        throw new IllegalThreadStateException();
    }
    daemon = on;
}
```

###### 设置/获取线程上下文类加载器

```java
// 设置实例线程的上下文类加载器, 以供实例线程中运行的代码在加载类和资源时使用
public void setContextClassLoader(ClassLoader cl) {
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        sm.checkPermission(new RuntimePermission("setContextClassLoader"));
    }
    contextClassLoader = cl;
}

// 获取实例线程的上下文类加载器, 以供实例线程中运行的代码在加载类和资源时使用
@CallerSensitive
public ClassLoader getContextClassLoader() {
    if (contextClassLoader == null)
        return null;
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        ClassLoader.checkClassLoaderPermission(contextClassLoader,
                                               Reflection.getCallerClass());
    }
    return contextClassLoader;
}
```

###### 获取线程ID

```java
// 线程 ID
private long tid;

// 返回实例线程的标识符, 线程ID是唯一的, 并且在其生命周期内保持不变, 而当一个线程终止时, 该线程ID可能会被重用
public long getId() {
    return tid;
}

// 用于生成线程ID
private static long threadSeqNumber;
private static synchronized long nextThreadID() {
    return ++threadSeqNumber;
}
```

###### 获取线程状态（生命周期）

```java
// 用于组合VM类常量, 以得出线程的State状态
private volatile int threadStatus = 0;

// 返回实例线程的状态, 用于监视系统状态, 而不是用于同步控制
public State getState() {
    // 获取当前线程状态
    return sun.misc.VM.toThreadState(threadStatus);
}

// sum.misc.VM
public class VM {
    // sum.misc.VM#静态常量
    private final static int JVMTI_THREAD_STATE_ALIVE = 0x0001;// 1
    private final static int JVMTI_THREAD_STATE_TERMINATED = 0x0002;// 2
    private final static int JVMTI_THREAD_STATE_RUNNABLE = 0x0004;// 4
    private final static int JVMTI_THREAD_STATE_BLOCKED_ON_MONITOR_ENTER = 0x0400;// 1024
    private final static int JVMTI_THREAD_STATE_WAITING_INDEFINITELY = 0x0010;// 16
    private final static int JVMTI_THREAD_STATE_WAITING_WITH_TIMEOUT = 0x0020;// 32

    // sum.misc.VM#toThreadState
    public static Thread.State toThreadState(int threadStatus) {
        // 4
        if ((threadStatus & JVMTI_THREAD_STATE_RUNNABLE) != 0) {
            return RUNNABLE;
        } 
        // 1024
        else if ((threadStatus & JVMTI_THREAD_STATE_BLOCKED_ON_MONITOR_ENTER) != 0) {
            return BLOCKED;
        } 
        // 16
        else if ((threadStatus & JVMTI_THREAD_STATE_WAITING_INDEFINITELY) != 0) {
            return WAITING;
        } 
        // 32
        else if ((threadStatus & JVMTI_THREAD_STATE_WAITING_WITH_TIMEOUT) != 0) {
            return TIMED_WAITING;
        } 
        // 2
        else if ((threadStatus & JVMTI_THREAD_STATE_TERMINATED) != 0) {
            return TERMINATED;
        } 
        // 1
        else if ((threadStatus & JVMTI_THREAD_STATE_ALIVE) == 0) {
            return NEW;
        } else {
            return RUNNABLE;
        }
    }  
}
```

##### 线程状态（生命周期）

![1628421924050](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1628421924050.png)

```java
   // Java线程的生命周期，即JVM线程状态，线程在某时刻只能处于一种状态（这些状态只为虚拟机状态，不代表任何操作系统的线程状态）
    public enum State {
        // 新建状态，尚未启动，new Thread(...)创建了线程，但未调用start（）启动线程
        NEW，
       
        // 可执行状态，包含操作系统的就绪、运行两种状态，线程的run()方法不一定会马上被并发执行，需要在线程获取了CPU时间片之后才真正启动并发执行。
        RUNNABLE，
            
        // 阻塞状态，处于阻塞状态的线程
        BLOCKED，
            
        // 等待状态，无时限的Object.wait（）、Thread.join()、LockSupport.park（）
        WAITING，
            
        // 等待超时状态，Thread.sleep(int n)，带时限的Object.wait()、Thread.join()、LockSupport.parkNanos()、LockSupport.parkUntil()
        TIMED_WAITING，
            
        // 终止状态，处于RUNNABLE状态的线程在run()方法执行完成之后就变成终止状态TERMINATED了。当然，如果在run()方法执行过程中发生了运行时异常而没有被捕获，run()方法将被异常终止，线程也会变成TERMINATED状态。
        TERMINATED；
    }
```

#### Unsafe

##### 特点

- Unsafe，一个执行低级、不安全操作的方法集合，可以为调用者提供执行不安全操作的能力，可用于**在任意内存地址读取和写入数据**。

##### 获取实例方法

```java
package sun.misc;
public final class Unsafe {
    // 私有构造、私有实例方法，用户无法实例化
    private Unsafe() {}
    private static final Unsafe theUnsafe = new Unsafe();
    
    // 只有受信任的代码才能调用该方法
    public static Unsafe getUnsafe() {
        Class<?> caller = Reflection.getCallerClass();
        if (!VM.isSystemDomainLoader(caller.getClassLoader()))
            throw new SecurityException("Unsafe");
        return theUnsafe;
    }
}

// 不受信任的代码, 只能通过反射获取Unsafe类并通过其分配直接内存
Field unsafeField = Unsafe.class.getDeclaredFields()[0];
unsafeField.setAccessible(true);
Unsafe unsafe = (Unsafe) unsafeField.get(null);
```

##### 操作Java变量方法

```java
// 64位地址偏移的方法
// 根据对象和偏移量, 获取类中返回值为int的某个字段的值
public native int getInt(Object o, long offset);

// 根据对象和偏移量, 将int值存储到Java变量中, 其字段类型必须为int类型
public native void putInt(Object o, long offset, int x);

// 从给定的Java变量中获取引用值
public native Object getObject(Object o, long offset);

// 根据对象和偏移量, 将x值存储到Java变量中, 其字段类型必须为Object类型
public native void putObject(Object o, long offset, Object x);

// 其他类型方法
public native boolean getBoolean(Object o, long offset);
public native void    putBoolean(Object o, long offset, boolean x);
public native byte    getByte(Object o, long offset);
public native void    putByte(Object o, long offset, byte x);
public native short   getShort(Object o, long offset);
public native void    putShort(Object o, long offset, short x);
public native char    getChar(Object o, long offset);
public native void    putChar(Object o, long offset, char x);
public native long    getLong(Object o, long offset);
public native void    putLong(Object o, long offset, long x);
public native float   getFloat(Object o, long offset);
public native void    putFloat(Object o, long offset, float x);
public native double  getDouble(Object o, long offset);
public native void    putDouble(Object o, long offset, double x);

// 32位地址偏移的方法，底层调用64位地址偏移的方法，已作废@Deprecated
public int getInt(Object o, int offset) {
    return getInt(o, (long)offset);
}
public void putInt(Object o, int offset, int x) {
    putInt(o, (long)offset, x);
}
public Object getObject(Object o, int offset) {
    return getObject(o, (long)offset);
}
public void putObject(Object o, int offset, Object x) {
    putObject(o, (long)offset, x);
}
public boolean getBoolean(Object o, int offset) {
    return getBoolean(o, (long)offset);
}
public void putBoolean(Object o, int offset, boolean x) {
    putBoolean(o, (long)offset, x);
}
public byte getByte(Object o, int offset) {
    return getByte(o, (long)offset);
}
public void putByte(Object o, int offset, byte x) {
    putByte(o, (long)offset, x);
}
public short getShort(Object o, int offset) {
    return getShort(o, (long)offset);
}
public void putShort(Object o, int offset, short x) {
    putShort(o, (long)offset, x);
}
public char getChar(Object o, int offset) {
    return getChar(o, (long)offset);
}
public void putChar(Object o, int offset, char x) {
    putChar(o, (long)offset, x);
}
public long getLong(Object o, int offset) {
    return getLong(o, (long)offset);
}
public void putLong(Object o, int offset, long x) {
    putLong(o, (long)offset, x);
}
public float getFloat(Object o, int offset) {
    return getFloat(o, (long)offset);
}
public void putFloat(Object o, int offset, float x) {
    putFloat(o, (long)offset, x);
}
public double getDouble(Object o, int offset) {
    return getDouble(o, (long)offset);
}
public void putDouble(Object o, int offset, double x) {
    putDouble(o, (long)offset, x);
}

// 返回字段的32位偏移量
@Deprecated
public int fieldOffset(Field f) {
    if (Modifier.isStatic(f.getModifiers()))
        return (int) staticFieldOffset(f);
    else
        return (int) objectFieldOffset(f);
}

// 返回访问给定类中某个静态字段的基地址
@Deprecated
public Object staticFieldBase(Class<?> c) {
    Field[] fields = c.getDeclaredFields();
    for (int i = 0; i < fields.length; i++) {
        if (Modifier.isStatic(fields[i].getModifiers())) {
            return staticFieldBase(fields[i]);
        }
    }
    return null;
}

// 返回给定静态字段的位置, 任何给定的字段将始终具有相同的偏移量, 并且同一类的两个不同字段永远不会具有相同的偏移量和基数
public native long staticFieldOffset(Field f);

// 返回给定字段在其类的存储分配中的位置, 任何给定的字段将始终具有相同的偏移量, 并且同一类的两个不同字段永远不会具有相同的偏移量和基数
public native long objectFieldOffset(Field f);

// 获取给定静态字段的基本“对象”(可能引用一个对象, 它是一个“cookie”, 不能保证是一个真正的对象, 它不应该以任何方式使用, 除了作为此类中get和put例程的参数), 如果有的话,可以通过 {@link #getInt(Object, long)} 之类的方法访问给定类的静态字段
public native Object staticFieldBase(Field f);

// 检测给定的类是否需要初始化
public native boolean shouldBeInitialized(Class<?> c);

// 确保给定的类已初始化
public native void ensureClassInitialized(Class<?> c);

// 返回给定数组Class的存储分配中第一个元素的偏移量
public native int arrayBaseOffset(Class<?> arrayClass);
/** The value of {@code arrayBaseOffset(boolean[].class)} */
public static final int ARRAY_BOOLEAN_BASE_OFFSET
            = theUnsafe.arrayBaseOffset(boolean[].class);
/** The value of {@code arrayBaseOffset(byte[].class)} */
public static final int ARRAY_BYTE_BASE_OFFSET
            = theUnsafe.arrayBaseOffset(byte[].class);
/** The value of {@code arrayBaseOffset(short[].class)} */
public static final int ARRAY_SHORT_BASE_OFFSET
            = theUnsafe.arrayBaseOffset(short[].class);
/** The value of {@code arrayBaseOffset(char[].class)} */
public static final int ARRAY_CHAR_BASE_OFFSET
            = theUnsafe.arrayBaseOffset(char[].class);
/** The value of {@code arrayBaseOffset(int[].class)} */
public static final int ARRAY_INT_BASE_OFFSET
            = theUnsafe.arrayBaseOffset(int[].class);
/** The value of {@code arrayBaseOffset(long[].class)} */
public static final int ARRAY_LONG_BASE_OFFSET
            = theUnsafe.arrayBaseOffset(long[].class);
/** The value of {@code arrayBaseOffset(float[].class)} */
public static final int ARRAY_FLOAT_BASE_OFFSET
            = theUnsafe.arrayBaseOffset(float[].class);
/** The value of {@code arrayBaseOffset(double[].class)} */
public static final int ARRAY_DOUBLE_BASE_OFFSET
            = theUnsafe.arrayBaseOffset(double[].class);
/** The value of {@code arrayBaseOffset(Object[].class)} */
public static final int ARRAY_OBJECT_BASE_OFFSET
            = theUnsafe.arrayBaseOffset(Object[].class);

// 返回在给定数组类的存储分配中寻址元素的比例因子
public native int arrayIndexScale(Class<?> arrayClass);
/** The value of {@code arrayIndexScale(boolean[].class)} */
public static final int ARRAY_BOOLEAN_INDEX_SCALE
            = theUnsafe.arrayIndexScale(boolean[].class);
/** The value of {@code arrayIndexScale(byte[].class)} */
public static final int ARRAY_BYTE_INDEX_SCALE
            = theUnsafe.arrayIndexScale(byte[].class);
/** The value of {@code arrayIndexScale(short[].class)} */
public static final int ARRAY_SHORT_INDEX_SCALE
            = theUnsafe.arrayIndexScale(short[].class);
/** The value of {@code arrayIndexScale(char[].class)} */
public static final int ARRAY_CHAR_INDEX_SCALE
            = theUnsafe.arrayIndexScale(char[].class);
/** The value of {@code arrayIndexScale(int[].class)} */
public static final int ARRAY_INT_INDEX_SCALE
            = theUnsafe.arrayIndexScale(int[].class);
/** The value of {@code arrayIndexScale(long[].class)} */
public static final int ARRAY_LONG_INDEX_SCALE
            = theUnsafe.arrayIndexScale(long[].class);
/** The value of {@code arrayIndexScale(float[].class)} */
public static final int ARRAY_FLOAT_INDEX_SCALE
            = theUnsafe.arrayIndexScale(float[].class);
/** The value of {@code arrayIndexScale(double[].class)} */
public static final int ARRAY_DOUBLE_INDEX_SCALE
            = theUnsafe.arrayIndexScale(double[].class);
/** The value of {@code arrayIndexScale(Object[].class)} */
public static final int ARRAY_OBJECT_INDEX_SCALE
            = theUnsafe.arrayIndexScale(Object[].class);
```

##### 操作C堆方法

```java
// 从给定的内存地址获取一个值
public native byte    getByte(long address);

// 将值存储到给定的内存地址中
public native void    putByte(long address, byte x);

// 其他类型方法
public native short   getShort(long address);
public native void    putShort(long address, short x);
public native char    getChar(long address);
public native void    putChar(long address, char x);
public native int     getInt(long address);
public native void    putInt(long address, int x);
public native long    getLong(long address);
public native void    putLong(long address, long x);
public native float   getFloat(long address);
public native void    putFloat(long address, float x);
public native double  getDouble(long address);
public native void    putDouble(long address, double x);

// 从给定的内存地址获取本地指针
public native long getAddress(long address);

// 将本地指针存储到给定的内存地址
public native void putAddress(long address, long x);
```

##### 分配内存方法

```java
// 分配一个新的本地内存块，以字节为单位给定大小
public native long allocateMemory(long bytes);

// 重新分配一个新的本地内存块，以字节为单位给定大小
public native long reallocateMemory(long address, long bytes);

// 将给定内存块中的所有字节设置为固定值(通常为零), 当对象引用为空时, 偏移量提供一个绝对基地址
public native void setMemory(Object o, long offset, long bytes, byte value);
public void setMemory(long address, long bytes, byte value) {
    setMemory(null, address, bytes, value);
}

// 将给定内存块中的所有字节设置为另一个块的副本, 当对象引用为空时, 偏移量提供一个绝对基地址
public native void copyMemory(Object srcBase, long srcOffset,
                              Object destBase, long destOffset,
                              long bytes);
public void copyMemory(long srcAddress, long destAddress, long bytes) {
    copyMemory(null, srcAddress, null, destAddress, bytes);
}

// 释放本地内存块
public native void freeMemory(long address);
```

##### 虚拟机/系统方法

```java
// 获取本地指针的字节大小
public native int addressSize();
public static final int ADDRESS_SIZE = theUnsafe.addressSize();

// 获取本地内存页面的字节大小
public native int pageSize();

// 告诉VM定义一个类
public native Class<?> defineClass(String name, byte[] b, int off, int len,
                                   ClassLoader loader,
                                   ProtectionDomain protectionDomain);
// 定义一个类，但不要让类加载器或系统字典知道它
public native Class<?> defineAnonymousClass(Class<?> hostClass, byte[] data, Object[] cpPatches);

// 分配一个实例，但不运行任何构造函数。如果尚未初始化类，则初始化该类
public native Object allocateInstance(Class<?> cls) throws InstantiationException;

// 锁定对象, 锁定后必须通过 {@link #monitorExit} 解锁
@Deprecated
public native void monitorEnter(Object o);

// 解锁对象, 必须先通过 {@link #monitorEnter} 锁定
@Deprecated
public native void monitorExit(Object o);

// 试图锁定对象。 返回 true 或 false 以指示锁定是否成功。 它必须通过 {@link #monitorExit} 解锁。
@Deprecated
public native boolean tryMonitorEnter(Object o);

// 在不告诉验证者的情况下抛出异常
public native void throwException(Throwable ee);

// 获取分配给可用处理器的系统运行队列中的平均负载在不同时间段内的平均值
public native int getLoadAverage(double[] loadavg, int nelems);

// 抛出非法访问错误, 供虚拟机使用
private static void throwIllegalAccessError() {
    throw new IllegalAccessError();
}
```

##### 并发方法

```java
// 如果当前保持预期状态，则将Java变量原子更新为x。
public final native boolean compareAndSwapObject(Object o, long offset, Object expected, Object x);

// 如果当前保持预期，则将 Java 变量原子更新为 x
public final native boolean compareAndSwapInt(Object o, long offset, int expected, int x);

// 如果当前保持预期，则将 Java 变量原子更新为 x
public final native boolean compareAndSwapLong(Object o, long offset, long expected, long x);

// 从给定具有 volatile 加载语义的Java变量中获取引用值, 否则等同于 {@link #getObject(Object, long)}
public native Object getObjectVolatile(Object o, long offset);
// 使用volatile存储语义将引用值存储到给定的Java变量中。否则等同于{@link #putObject(Object, long, Object)}, 该方法会保证存储对其他线程的立即可见性
public native void    putObjectVolatile(Object o, long offset, Object x);
/** Volatile version of {@link #getInt(Object, long)}  */
public native int     getIntVolatile(Object o, long offset);
/** Volatile version of {@link #putInt(Object, long, int)}  */
public native void    putIntVolatile(Object o, long offset, int x);
/** Volatile version of {@link #getBoolean(Object, long)}  */
public native boolean getBooleanVolatile(Object o, long offset);
/** Volatile version of {@link #putBoolean(Object, long, boolean)}  */
public native void    putBooleanVolatile(Object o, long offset, boolean x);
/** Volatile version of {@link #getByte(Object, long)}  */
public native byte    getByteVolatile(Object o, long offset);
/** Volatile version of {@link #putByte(Object, long, byte)}  */
public native void    putByteVolatile(Object o, long offset, byte x);
/** Volatile version of {@link #getShort(Object, long)}  */
public native short   getShortVolatile(Object o, long offset);
/** Volatile version of {@link #putShort(Object, long, short)}  */
public native void    putShortVolatile(Object o, long offset, short x);
/** Volatile version of {@link #getChar(Object, long)}  */
public native char    getCharVolatile(Object o, long offset);
/** Volatile version of {@link #putChar(Object, long, char)}  */
public native void    putCharVolatile(Object o, long offset, char x);
/** Volatile version of {@link #getLong(Object, long)}  */
public native long    getLongVolatile(Object o, long offset);
/** Volatile version of {@link #putLong(Object, long, long)}  */
public native void    putLongVolatile(Object o, long offset, long x);
/** Volatile version of {@link #getFloat(Object, long)}  */
public native float   getFloatVolatile(Object o, long offset);
/** Volatile version of {@link #putFloat(Object, long, float)}  */
public native void    putFloatVolatile(Object o, long offset, float x);
/** Volatile version of {@link #getDouble(Object, long)}  */
public native double  getDoubleVolatile(Object o, long offset);
/** Volatile version of {@link #putDouble(Object, long, double)}  */
public native void    putDoubleVolatile(Object o, long offset, double x);

// 使用volatile存储语义将引用值存储到给定的Java变量中。否则等同于{@link #putObject(Object, long, Object)}, 该方法不保证存储对其他线程的立即可见性
public native void    putOrderedObject(Object o, long offset, Object x);

// 使用volatile存储语义将引用值存储到给定的Java变量中。否则等同于{@link #putObject(Object, long, Object)}, 该方法不保证存储对其他线程的立即可见性
public native void    putOrderedInt(Object o, long offset, int x);

// 使用volatile存储语义将引用值存储到给定的Java变量中。否则等同于{@link #putObject(Object, long, Object)}, 该方法不保证存储对其他线程的立即可见性
public native void    putOrderedLong(Object o, long offset, long x);

// 唤醒在park上阻塞的指定线程, 如果该线程并未阻塞, 则它在后续调用park时不会被阻塞
public native void unpark(Object thread);

// 阻塞当前线程, 直到当前线程unpark被调用、被中断、time时间过去(非绝对时为纳秒, 绝对时为毫秒, 为0时代表无限阻塞)
public native void park(boolean isAbsolute, long time);

// 以原子方式将给定值添加到给定对象o中给定偏移量处的字段或数组元素的当前值
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!compareAndSwapInt(o, offset, v, v + delta));
    return v;
}

// 以原子方式将给定值添加到给定对象o中给定偏移量处的字段或数组元素的当前值
public final long getAndAddLong(Object o, long offset, long delta) {
    long v;
    do {
        v = getLongVolatile(o, offset);
    } while (!compareAndSwapLong(o, offset, v, v + delta));
    return v;
}

// 在给定的偏移量处以原子方式将给定值与给定对象 o 内的字段或数组元素的当前值进行交换
public final int getAndSetInt(Object o, long offset, int newValue) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!compareAndSwapInt(o, offset, v, newValue));
    return v;
}

// 在给定的偏移量处以原子方式将给定值与给定对象 o 内的字段或数组元素的当前值进行交换
public final long getAndSetLong(Object o, long offset, long newValue) {
    long v;
    do {
        v = getLongVolatile(o, offset);
    } while (!compareAndSwapLong(o, offset, v, newValue));
    return v;
}

// 在给定的偏移量处以原子方式将给定的参考值与给定对象 o 内的字段或数组元素的当前参考值进行交换
public final Object getAndSetObject(Object o, long offset, Object newValue) {
    Object v;
    do {
        v = getObjectVolatile(o, offset);
    } while (!compareAndSwapObject(o, offset, v, newValue));
    return v;
}

// 确保在栅栏之前不会对loads重排序, 在栅栏后不会对loads或stores重排序
public native void loadFence();

// 确保栅栏前不会对stores重排序, 在栅栏后不会对loads或stores重排序
public native void storeFence();

// 确保在栅栏之前不会对loads或stores进行重新排序，而在栅栏之后则不会对stores或loads进行重新排序
public native void fullFence();
```

#### LockSupport

##### 特点

- LockSupport，用于创建锁和其他同步类的基本**线程阻塞原语**，旨在用作创建更高级别同步实用程序的工具，本身对大多数并发控制应用程序没有用处，可以与使用它的每个线程相关联：
  - 如果许可证可用，调用{@code park} 将立即返回，否则可能会阻塞。
  - 如果许可证尚未可用，则调用 {@code unpark} 可使许可证可用。
- 方法{@code park}和{@code unpark}，提供了**阻塞和解除阻塞线程**的有效方法：
  - 如果调用者的线程被中断，{@code park}将返回，并且支持超时版本。
  - {@code park}方法也可能在任何其他时间“无缘无故”地返回，即发生**虚假唤醒**，因此通常必须在返回时重新检查条件的循环中调用。 从这个意义上说，{@code park}是“忙等待”的优化，不会浪费太多时间自旋，但必须与{@code unpark} 配对才能有效。
- {@code park}的三种形式都支持{@code blocker}对象参数，{@code blocker}对象在线程被阻塞时被记录，以允许监控和诊断工具识别线程被阻塞的原因。
  - 此{@code blocker}工具可以使用方法{@link #getBlocker(Thread)} 访问阻止程序。
  - 强烈建议使用这些形式而不是没有此参数的原始形式。
  - 在Lock实现中作为{@code blocker} 提供的正常参数是**{@code this}**。

##### 构造方法

```java
public class LockSupport {
    // 用户无法实例化
    private LockSupport() {}
    
    // 通过内在 API 实现热点
    private static final sun.misc.Unsafe UNSAFE;
    private static final long parkBlockerOffset;// Thread.parkBlocker
    private static final long SEED;// threadLocalRandomSeed
    private static final long PROBE;// threadLocalRandomProbe
    private static final long SECONDARY;// threadLocalRandomSecondarySeed
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> tk = Thread.class;
            parkBlockerOffset = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("parkBlocker"));
            SEED = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomSeed"));
            PROBE = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomProbe"));
            SECONDARY = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomSecondarySeed"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    // 返回伪随机初始化或更新的辅助种子。由于包访问限制，从ThreadLocalRandom复制。
    static final int nextSecondarySeed() {
        int r;
        Thread t = Thread.currentThread();
        if ((r = UNSAFE.getInt(t, SECONDARY)) != 0) {
            r ^= r << 13;   // xorshift
            r ^= r >>> 17;
            r ^= r << 5;
        }
        else if ((r = java.util.concurrent.ThreadLocalRandom.current().nextInt()) == 0)
            r = 1; // avoid zero
        UNSAFE.putInt(t, SECONDARY, r);
        return r;
    }

    // 设置负责指定线程阻塞的同步对象(通常使用this), 该对象会在线程被阻塞时被记录, 以允许监控和诊断工具识别线程被阻塞的原因, 强烈建议这种形式而不是没有此参数的原始形式
    private static void setBlocker(Thread t, Object arg) {
        // 根据对象和偏移量, 将x值存储到Java变量中, 其字段类型必须为Object类型
        UNSAFE.putObject(t, parkBlockerOffset, arg);
    }
}
```

##### 获取Blocker监控工具方法

```java
// 返回提供给尚未解除park阻塞方法的最近调用的同步对象快照, 如果未阻塞, 则返回null: 该对象会在线程被阻塞时被记录, 以允许监控和诊断工具识别线程被阻塞的原因, 强烈建议这种形式而不是没有此参数的原始形式
public static Object getBlocker(Thread t) {
    if (t == null)
        throw new NullPointerException();
    return UNSAFE.getObjectVolatile(t, parkBlockerOffset);
}
```

##### 阻塞线程方法(含Blocker)

```java
// 无限阻塞当前线程, 直到当前线程unpark被调用、被中断、time时间过去(非绝对时为纳秒, 绝对时为毫秒, 为0时代表无限阻塞)
public static void park(Object blocker) {
    // 获取当前线程
    Thread t = Thread.currentThread();

    // 设置负责指定线程阻塞的同步对象(通常使用this), 该对象会在线程被阻塞时被记录, 以允许监控和诊断工具识别线程被阻塞的原因, 强烈建议这种形式而不是没有此参数的原始形式
    setBlocker(t, blocker);

    // 阻塞当前线程, 直到当前线程unpark被调用、被中断、time时间过去(非绝对时为纳秒, 绝对时为毫秒, 为0时代表无限阻塞)
    UNSAFE.park(false, 0L);

    // 唤醒后清空负责指定线程阻塞的同步对象(通常使用this), 该对象会在线程被阻塞时被记录, 以允许监控和诊断工具识别线程被阻塞的原因, 强烈建议这种形式而不是没有此参数的原始形式
    setBlocker(t, null);
}

// 在指定的等待时间内阻塞当前线程, 直到当前线程unpark被调用、被中断、time时间过去(非绝对时为纳秒, 绝对时为毫秒, 为0时代表无限阻塞)
public static void parkNanos(Object blocker, long nanos) {
    if (nanos > 0) {
        // 获取当前线程
        Thread t = Thread.currentThread();

        // 设置负责指定线程阻塞的同步对象(通常使用this), 该对象会在线程被阻塞时被记录, 以允许监控和诊断工具识别线程被阻塞的原因, 强烈建议这种形式而不是没有此参数的原始形式
        setBlocker(t, blocker);

        // 阻塞当前线程, 直到当前线程unpark被调用、被中断、time时间过去(非绝对时为纳秒, 绝对时为毫秒, 为0时代表无限阻塞)
        UNSAFE.park(false, nanos);

        // 唤醒后清空负责指定线程阻塞的同步对象(通常使用this), 该对象会在线程被阻塞时被记录, 以允许监控和诊断工具识别线程被阻塞的原因, 强烈建议这种形式而不是没有此参数的原始形式
        setBlocker(t, null);
    }
}

// 在绝对时间前阻塞当前线程, 直到当前线程unpark被调用、被中断、time时间过去(非绝对时为纳秒, 绝对时为毫秒, 为0时代表无限阻塞)
public static void parkUntil(Object blocker, long deadline) {
    // 获取当前线程
    Thread t = Thread.currentThread();

    // 设置负责指定线程阻塞的同步对象(通常使用this), 该对象会在线程被阻塞时被记录, 以允许监控和诊断工具识别线程被阻塞的原因, 强烈建议这种形式而不是没有此参数的原始形式
    setBlocker(t, blocker);

    // 阻塞当前线程, 直到当前线程unpark被调用、被中断、time时间过去(非绝对时为纳秒, 绝对时为毫秒, 为0时代表无限阻塞)
    UNSAFE.park(true, deadline);

    // 唤醒后清空负责指定线程阻塞的同步对象(通常使用this), 该对象会在线程被阻塞时被记录, 以允许监控和诊断工具识别线程被阻塞的原因, 强烈建议这种形式而不是没有此参数的原始形式
    setBlocker(t, null);
}
```

##### 阻塞线程方法(不含Blocker)

```java
// 无同步对象记录方式地无限阻塞当前线程(不建议使用), 直到当前线程unpark被调用、被中断、time时间过去(非绝对时为纳秒, 绝对时为毫秒, 为0时代表无限阻塞)
public static void park() {
    UNSAFE.park(false, 0L);
}

// 无同步对象记录方式地在指定的等待时间内阻塞当前线程(不建议使用), 直到当前线程unpark被调用、被中断、time时间过去(非绝对时为纳秒, 绝对时为毫秒, 为0时代表无限阻塞)
public static void parkNanos(long nanos) {
    if (nanos > 0)
        UNSAFE.park(false, nanos);
}

// 无同步对象记录方式地在绝对时间前阻塞当前线程阻塞当前线程(不建议使用), 直到当前线程unpark被调用、被中断、time时间过去(非绝对时为纳秒, 绝对时为毫秒, 为0时代表无限阻塞)
public static void parkUntil(long deadline) {
    UNSAFE.park(true, deadline);
}
```

##### 解除线程阻塞方法

```java
// 发放许可给指定的线程, 如果该线程已经在park上被阻塞, 则立即解除阻塞; 否则会保证对它的下一次park调用不会进行阻塞; 但是如果该线程未启动, 则此操作没有任何效果
public static void unpark(Thread thread) {
    if (thread != null)
        // 唤醒在park上阻塞的指定线程, 如果该线程并未阻塞, 则它在后续调用park时不会被阻塞
        UNSAFE.unpark(thread);
}
```

#### AbstractQueuedSychronizer

##### AbstractOwnableSynchronizer

- AbstractOwnableSynchronizer，**线程独占拥有的同步器**，可以为创建可能需要**所有权概念**的锁和相关同步器提供了基础。
- AbstractOwnableSynchronizer本身不管理或使用此独占线程信息，但子类和其他工具可以使用，用于帮助控制和监视访问并提供诊断。

```java
public abstract class AbstractOwnableSynchronizer implements java.io.Serializable {
    // 供子类使用的空构造函数
    protected AbstractOwnableSynchronizer() { }
    
    // 独占模式同步的当前所有者
    private transient Thread exclusiveOwnerThread;
    
    // 设置当前拥有独占访问权限的线程。 {@code null} 参数表示没有线程拥有访问权限。 此方法不会以其他方式强加任何同步或 {@code volatile} 字段访问。
    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }
    
    // 返回最后由 {@code setExclusiveOwnerThread} 设置的线程，如果从未设置，则返回 {@code null}。 此方法不会以其他方式强加任何同步或 {@code volatile} 字段访问。
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
}
```

##### Condiction

- {@code Condition}将{@code Object}的监控方法（{@link Object#wait() wait}、{@link Object#notify notify}和{@link Object#notifyAll notifyAll}）**分解为不同的对象**，通过将它们与任意{@link Lock}实现的使用相结合，每个对象具有多个等待集的效果。
  - {@code Lock}代替了{@code synchronized}方法和语句的使用。
  - {@code Condition}代替了对象监视器方法的使用。
- Condiction，条件（也称为条件队列或条件变量），为一个线程提供了一种挂起执行（“等待”）的方法，直到另一个线程通知某个状态条件现在可能为真。
  - 由于对这个共享状态信息的访问发生在不同的线程中，因此这个共享状态必须受到保护，所以某种形式的Lock可以与Condition相关联。
  - 等待条件提供的关键属性是它以原子方式释放关联的锁并挂起当前线程，就像{@code Object.wait}一样。
  - **{@code Condition}实例本质上绑定到Lock**，要获取特定{@link Lock}实例的{@code Condition}实例，请使用其{@link Lock#newCondition newCondition()} 方法。
  - 在等待Condition之前，当前线程必须持有锁。调用 {@link Condition#await()}将在await之前自动释放锁，并在await返回之前重新获取锁。
- {@code Condition}实现可以提供与{@code Object}监视器方法不同的行为和语义，例如保证通知的顺序，或者在执行通知时不需要持有锁。如果实现提供了这样的专门语义，那么实现必须记录这些语义。
- 请注意，{@code Condition}实例只是普通对象，它们本身可以用作{@code synchronized} 语句中的目标，并且可以拥有自己的监视器{@link Object#wait wait}和{@link Object#notify 通知}方法调用，建议不要以这种方式使用{@code Condition} 实例以避免混淆，除非在它们自己的实现中。
- 除非另有说明，否则为任何参数传递 {@code null} 值将导致抛出 {@link NullPointerException}。
- 在等待{@code Condition}时，作为对底层平台语义的让步，通常允许发生“**虚假唤醒**”。因此，建议程序员应该始终假设它们可能发生，始终把{@code Condition}放在**循环中等待**，以实现可以自由地消除虚假唤醒的可能性。

```java
public interface Condition {
    // 当前线程阻塞等待, 直到收到信号或{@linkplain Thread#interrupt interrupted}, 在可以返回当前线程之前, 必须重新获取与此Condition关联的锁
    void await() throws InterruptedException;
    
    // 当前线程阻塞等待, 直到收到信号(如果期间有中断会先继续等待, 在返回后再重新设置回中断标记位), 在可以返回当前线程之前, 必须重新获取与此Condition关联的锁
    void awaitUninterruptibly();
    
    // 当前线程阻塞等待, 直到收到信号、中断或者指定的等待时间过去, 在可以返回当前线程之前, 必须重新获取与此Condition关联的锁
    long awaitNanos(long nanosTimeout) throws InterruptedException;
    
    // 等价于{@code awaitNanos(unit.toNanos(time)) > 0}, 当前线程阻塞等待, 直到收到信号、中断或者指定的等待时间过去, 在可以返回当前线程之前, 必须重新获取与此Condition关联的锁
    boolean await(long time, TimeUnit unit) throws InterruptedException;
    
    // 当前线程阻塞等待, 直到收到信号、中断或者指定的截止日期过去, 在可以返回当前线程之前, 必须重新获取与此Condition关联的锁
    boolean awaitUntil(Date deadline) throws InterruptedException;
    
    // 随机唤醒一个等待的线程
    void signal();
    
    // 唤醒所有等待的线程
    void signalAll();
}
```

![1629178466619](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1629178466619.png)

##### 特点

- AbstractQueuedSychronizer，**简称AQS**，它可以提供一个框架，用于实现**依赖先进先出 (FIFO) 等待队列**的阻塞锁和相关同步器（信号量、事件等），旨在成为大多数依赖**单个原子 {@code int} 值来表示状态**的同步器的有用基础。
- AQS支持独占模式（默认）和共享模式，在不同模式下等待的线程**共享同一个 FIFO 队列**。
  - 当以独占模式获取时，其他线程尝试获取不会成功。
  - 当以共享模式获取时， 多个线程尝试获取可能会成功。当共享模式获取成功时，下一个等待线程（如果存在）也必须确定它自己是否也可以获取。
  - 即使AQS内部基于FIFO队列，但它也不会自动执行FIFO采集策略，因为在入队之前会先调用获取的检查，所以新的获取线程可能会抢在其他被阻塞和排队的线程之前。
    - 虽然该策略无法保证公平或者无饥饿，但允许较早的排队线程在较晚的排队线程之前竞争，从而能够保持**最高的吞吐量和可扩展性**。
    - 如果需要可以定义{@code tryAcquire} 或者 {@code tryAcquireShared} ，通过内部调用一种或多种检查方法来禁用插入，从而提供公平的 FIFO 获取顺序。比如{@link #**hasQueuedPredecessors**}（一种专门设计用于公平同步器使用的方法）返回 {@code true}时不允许插入，因此大多数公平同步器可以基于该方法，从而定义 {@code tryAcquire} 返回 {@code false}，代表不允许提前插入。
- AQS定义了一个嵌套的 {@link ConditionObject} 类，{@link ConditionObject}可以被支持独占模式的子类用作 **{@link Condition} 实现**，而{@link ConditionObject} 的行为当然取决于其同步器实现的语义。
- AQS还为内部队列提供检查、检测和监视方法，同时也为ConditionObject提供类似方法。
- AQS子类应定义为非公共内部帮助类，用于实现其封闭类的同步属性，且必须定义更改此状态的protected方法，并定义该状态在获取或释放此对象方面的含义。
  - 如果要将AQS用作同步器的基础，需要根据适用情况重新定义以下方法，{@link #tryAcquire}、{@link #tryRelease}、{@link #tryAcquireShared}、{@link #tryReleaseShared}、{@link #isHeldExclusively}，默认情况下，这些方法中都会抛出 {@link UnsupportedOperationException}。 注意，实现它们必须是**内部线程安全**的，并且应该是**简短不阻塞**的。
  - 此外，子类还可以维护其他状态字段，但只有使用方法 {@link #getState}、{@link #setState} 和 {@link #compareAndSetState} 操作**原子更新的 {@code int} 值**才会在同步方面进行跟踪。
  - 基于这些，AQS中的其他方法执行所有**排队和阻塞机制**，比如定义了{@link #acquireInterruptably} 之类的方法，这些方法可以由具体锁和相关同步器适当调用以实现它们的公共方法。

##### 等待主队列与条件队列

```java
      +------+  prev +-----+       +-----+
 head |      | <---- |     | <---- |     |  tail
      +------+       +-----+       +-----+
```

- AQS等待队列，是**“CLH”锁定队列**（Craig、Landin 和 Hagersten）的变体（CLH 锁通常用于自旋锁），AQS将它们改为用于阻塞同步器，但使用相同的基本策略。
  - 即队列的每个节点都充当一个**特定的通知式监视器**，持有一个**等待线程**（用于保存后继有关控制信息），一个**状态字段**（用于跟踪线程是否应该阻塞）。
  - 在前驱结点Release时，当前结点将会收到信号，从而可能会尝试获取成为在队列中的第一个结点，但并不保证成功，只是给予当前结点参与竞争的权利。
  - 而要加入CLH锁，需要原子地将其拼接为新的尾部，要出列，需要设置该结点成为head结点。
  - “prev”前驱指针（未在原始CLH锁中使用），用于处理取消结点：如果一个结点被取消，则它的后继会重新链接到一个未取消的前驱。
  - “next”后继指针，用于实现阻塞与通知机制：每个节点的线程id保存在它自己的节点中，因此前驱通过遍历下一个链接来确定它是哪个线程来通知下一个节点唤醒。
- 此外，等待队列还需要一个**虚拟头结点**来启动，但是AQS不会在构建时创建它们，因为如果从不存在争用，那将是浪费精力。因此，AQS只会在第一次争用时，才构造结点并设置头指针和尾指针。
- 等待条件的线程也**使用相同的节点**，但使用了额外的指针来维护一个简单的**条件队列**。在等待时，一个结点被插入到条件队列中。然后根据信号，该结点将会被转移到主队列中。
  - 其中，AQS使用**status字段**来标记节点所在的队列。

```java
// AQS等待队列结点
static final class Node {
    static final Node SHARED = new Node();// 共享模式标记
    static final Node EXCLUSIVE = null;// 独占模式标记

    // Node等待状态，初始化结点(未设置状态)是为0
    volatile int waitStatus;
    static final int CANCELLED =  1;// 已取消状态
    static final int SIGNAL    = -1;// 需通知后继结点状态
    static final int CONDITION = -2;// 条件等待状态
    static final int PROPAGATE = -3;// 需传播状态

    volatile Node prev;// AQS等待主队列前驱
    volatile Node next;// AQS等待主队列后继
    volatile Thread thread;// ASQ等待主队列、条件队列中的排队线程
    Node nextWaiter;// 条件队列的后继, 或者代表等待主队列中的特殊值

    // 无参构造函数，用于建立初始头部或共享标记
    Node() { }

    // 根据指定线程和后继构造排队结点，用于addWaiter
    Node(Thread thread, Node mode) {
        this.nextWaiter = mode;
        this.thread = thread;
    }

    // 根据指定线程和等待状态构造排队结点，条件队列中使用
    Node(Thread thread, int waitStatus) {
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
    
    // 判断条件队列后继是否为特殊值SHARED
    final boolean isShared() {
        return nextWaiter == SHARED;
    }
    
    // volatile方式获取AQS等待队列前驱
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }
}
```

##### 构造方法

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {
    // AQS成员变量
    private transient volatile Node head;// AQS等待主队列头结点
    private transient volatile Node tail;// AQS等待队列尾结点
    private volatile int state;// AQS同步器状态， 其语义看具体的实现
    static final long spinForTimeoutThreshold = 1000L;// 自旋时可以进入阻塞的时间
    
    // protected构造器，抽象类无法实例化
    protected AbstractQueuedSynchronizer() { }
}
```

##### 同步器状态相关方法

```java
// volatile方式获取AQS同步器状态
protected final int getState() {
    return state;
}

// volatile方式设置AQS同步器状态
protected final void setState(int newState) {
    state = newState;
}

// CAS原子式更新AQS同步器状态
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

##### 核心源码 - 获取同步器

###### acquireQueued

```java
// 独占式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回; 否则放置node结点并阻塞其排队线程, 如果当前线程有被中断过, 则还需要继续自旋, 直到获取到同步器, 返回true代表有被中断过让上层API处理; 另外如果调用tryAcquire方法时发生了异常, 则还需要则取消node结点
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;

        // 开始自旋
        for (;;) {
            // volatile方式获取前驱p
            final Node p = node.predecessor();

            // 如果p为头结点, 则尝试以独占模式获取同步器
            if (p == head && tryAcquire(arg)) {
                // 如果获取同步器成功, 则volatile更新node为头结点, 并清空排队线程、前驱指针, 方便GC回收thread和prev
                setHead(node);
                p.next = null; // help GC

                // 更新失败为false, 返回interrupted, 为false代表当前线程自旋过程中没有被中断过, 为true代表当前线程自旋过程中有被中断过(含中断异常的方法调用时会在外面抛出中断异常)
                failed = false;
                return interrupted;
            }

            // 如果获取同步器失败, 则放置node结点并阻塞其排队线程是否合理, 如果返回true代表确定了前驱为通知结点, 此时node的排队线程需要阻塞; 如果返回false代表还没能确定前驱为通知结点, 需要返回继续自旋判断, 此时node的排队线程不能被阻塞
            if (shouldParkAfterFailedAcquire(p, node) &&
                // 如果node结点放置成功并且需要阻塞, 则park阻塞当前线程, 然后检查当前线程是否中断(当前线程被中断, 会解除线程阻塞状态)
                parkAndCheckInterrupt())
                // 如果当前线程被中断了(Java中中断不了线程, 只是设置了中断标记位为true), 则更新interrupted为true, 代表当前线程有被中断过, 然后继续自旋, 直到获取到同步器
                interrupted = true;
        }
    } finally {
        // 走到这里但failed为false, 说明在获取子类实现的tryAcquire方法时发生了异常
        if (failed)
            // 则取消node结点(前驱不为头结点且需要传播时), 或者唤醒node结点的排队线程(前驱为头结点时)
            cancelAcquire(node);
    }
}
// volatile更新node为头结点, 并清空排队线程、前驱指针, 方便GC回收thread和prev
private void setHead(Node node) {
    // volatile设置node为头结点
    head = node;

    // 清空排队线程、前驱指针, 方便GC回收thread和prev
    node.thread = null;
    node.prev = null;
}
```

###### doAcquireInterruptibly

```java
// 独占可中断式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回; 否则放置node结点并阻塞其排队线程, 如果调用tryAcquire方法时发生了异常或者自旋过程中发生了中断异常, 则还需要则取消node结点
private void doAcquireInterruptibly(int arg) throws InterruptedException {
    // 使用当前线程构建独占Node结点, 并CAS+自旋直至入队成功
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        // 开始自旋
        for (;;) {
            // volatile方式获取前驱p
            final Node p = node.predecessor();

            // 如果p为头结点, 则尝试以独占模式获取同步器
            if (p == head && tryAcquire(arg)) {
                // 如果获取同步器成功, 则volatile更新node为头结点, 并清空排队线程、前驱指针, 方便GC回收thread和prev
                setHead(node);
                p.next = null; // help GC

                // 更新失败为false并返回
                failed = false;
                return;
            }

            // 如果获取同步器失败, 则放置node结点并阻塞其排队线程是否合理, 如果返回true代表确定了前驱为通知结点, 此时node的排队线程需要阻塞; 如果返回false代表还没能确定前驱为通知结点, 需要返回继续自旋判断, 此时node的排队线程不能被阻塞
            if (shouldParkAfterFailedAcquire(p, node) &&
                // 如果node结点放置成功并且需要阻塞, 则park阻塞当前线程, 然后检查当前线程是否中断(当前线程被中断, 会解除线程阻塞状态)
                parkAndCheckInterrupt())
                // 如果当前线程被中断了, 则抛出中断异常, 进行内部消化, 而不像acquireQueued那样返回中断标记
                throw new InterruptedException();
        }
    } finally {
        // 走到这里但failed为false, 说明在获取子类实现的tryAcquire方法时发生了异常, 或者自旋过程中发生了中断异常
        if (failed)
            // 则取消node结点(前驱不为头结点且需要传播时), 或者唤醒node结点的排队线程(前驱为头结点时)
            cancelAcquire(node);
    }
}
```

###### doAcquireNanos

```java
// 独占定时可中断式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回; 否则放置node结点并阻塞其排队线程, 如果调用tryAcquire方法时发生了异常或者自旋过程中发生了中断异常, 则还需要则取消node结点
private boolean doAcquireNanos(int arg, long nanosTimeout) throws InterruptedException {
    // 超时时间小于等于0, 则直接返回false, 代表没获得同步状态
    if (nanosTimeout <= 0L)
        return false;

    // 系统纳米时间 + 超时时间 = 过期总时间
    final long deadline = System.nanoTime() + nanosTimeout;

    // 使用当前线程构建独占Node结点, 并CAS+自旋直至入队成功
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        // 开始自旋
        for (;;) {
            // volatile方式获取前驱p
            final Node p = node.predecessor();

            // 如果p为头结点, 则尝试以独占模式获取同步器
            if (p == head && tryAcquire(arg)) {
                // 如果获取同步器成功, 则volatile更新node为头结点, 并清空排队线程、前驱指针, 方便GC回收thread和prev
                setHead(node);
                p.next = null; // help GC

                // 更新失败为false并返回
                failed = false;
                return true;
            }

            // 如果获取同步器失败, 则重新获取系统纳米时间, 用于更新过期总时间, 如果过期总时间小于等于0, 则直接返回false, 代表没获得同步状态
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;

            // 如果过期总时间大于0, 说明仍有效, 则放置node结点并阻塞其排队线程是否合理, 如果返回true代表确定了前驱为通知结点, 此时node的排队线程需要阻塞; 如果返回false代表还没能确定前驱为通知结点, 需要返回继续自旋判断, 此时node的排队线程不能被阻塞
            if (shouldParkAfterFailedAcquire(p, node)
                // 如果过期总时间仍大于1s钟, 说明仍然需要阻塞线程, 则调用LockSupport的定时阻塞方法阻塞当前线程
                && nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);

            // 如果当前线程被中断或者退出阻塞, 则判断获取中断标志位, 如果为true则抛出中断异常
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        // 走到这里但failed为false, 说明在获取子类实现的tryAcquire方法时发生了异常, 或者自旋过程中发生了中断异常
        if (failed)
            // 则取消node结点(前驱不为头结点且需要传播时), 或者唤醒node结点的排队线程(前驱为头结点时)
            cancelAcquire(node);
    }
}
```

###### doAcquireShared

```java
// 共享式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回; 否则放置node结点并阻塞其排队线程, 如果当前线程有被中断过, 则还需要继续自旋, 直到获取到同步器, 返回true代表有被中断过让上层API处理; 另外如果调用tryAcquire方法时发生了异常, 则还需要则取消node结点
private void doAcquireShared(int arg) {
    // 使用当前线程构建共享模式的Node结点, 并CAS+自旋直至入队成功
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;

        // 开始自旋
        for (;;) {
            // volatile方式获取前驱p
            final Node p = node.predecessor();

            // 如果p为头结点, 则尝试以共享模式获取同步器
            if (p == head) {
                // 失败(<0), 获取成功但后续共享不成功(=0), 获取成功且后续共享成功(>0)
                int r = tryAcquireShared(arg);

                // 如果获取成功且后续共享成功(>0), 则volatile更新node为头结点, 并CAS更新为传播结点
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC

                    // 如果interrupted为true, 说明当前线程有被中断过, 则重新设置中断标记为true
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }

            // 如果获取同步器失败, 则放置node结点并阻塞其排队线程是否合理, 如果返回true代表确定了前驱为通知结点, 此时node的排队线程需要阻塞; 如果返回false代表还没能确定前驱为通知结点, 需要返回继续自旋判断, 此时node的排队线程不能被阻塞
            if (shouldParkAfterFailedAcquire(p, node) &&
                // 如果node结点放置成功并且需要阻塞, 则park阻塞当前线程, 然后检查当前线程是否中断(当前线程被中断, 会解除线程阻塞状态)
                parkAndCheckInterrupt())
                // 如果当前线程被中断了(Java中中断不了线程, 只是设置了中断标记位为true), 则更新interrupted为true, 代表当前线程有被中断过, 然后继续自旋, 直到获取到同步器
                interrupted = true;
        }
    } finally {
        // 走到这里但failed为false, 说明在获取子类实现的tryAcquire方法时发生了异常
        if (failed)
            // 则取消node结点(前驱不为头结点且需要传播时), 或者唤醒node结点的排队线程(前驱为头结点时)
            cancelAcquire(node);
    }
}
```

###### doAcquireSharedInterruptibly

```java
// 共享可中断式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回; 否则放置node结点并阻塞其排队线程, 如果调用tryAcquire方法时发生了异常或者自旋过程中发生了中断异常, 则还需要则取消node结点
private void doAcquireSharedInterruptibly(int arg) throws InterruptedException {
    // 使用当前线程构建共享Node结点, 并CAS+自旋直至入队成功
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        // 开始自旋
        for (;;) {
            // volatile方式获取前驱p
            final Node p = node.predecessor();

            // 如果p为头结点, 则尝试以独占模式获取同步器
            if (p == head) {
                // 失败(<0), 获取成功但后续共享不成功(=0), 获取成功且后续共享成功(>0)
                int r = tryAcquireShared(arg);

                // 如果获取成功且后续共享成功(>0), 则volatile更新node为头结点, 并CAS更新为传播结点
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }

            // 如果获取同步器失败, 则放置node结点并阻塞其排队线程是否合理, 如果返回true代表确定了前驱为通知结点, 此时node的排队线程需要阻塞; 如果返回false代表还没能确定前驱为通知结点, 需要返回继续自旋判断, 此时node的排队线程不能被阻塞
            if (shouldParkAfterFailedAcquire(p, node) &&
                // 如果node结点放置成功并且需要阻塞, 则park阻塞当前线程, 然后检查当前线程是否中断(当前线程被中断, 会解除线程阻塞状态)
                parkAndCheckInterrupt())
                // 如果当前线程被中断了, 则抛出中断异常, 进行内部消化
                throw new InterruptedException();
        }
    } finally {
        // 走到这里但failed为false, 说明在获取子类实现的tryAcquire方法时发生了异常, 或者自旋过程中发生了中断异常
        if (failed)
            // 则取消node结点(前驱不为头结点且需要传播时), 或者唤醒node结点的排队线程(前驱为头结点时)
            cancelAcquire(node);
    }
}
```

###### doAcquireSharedNanos

```java
// 共享定时可中断式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回; 否则放置node结点并阻塞其排队线程, 如果调用tryAcquire方法时发生了异常或者自旋过程中发生了中断异常, 则还需要则取消node结点
private boolean doAcquireSharedNanos(int arg, long nanosTimeout) throws InterruptedException {
    // 超时时间小于等于0, 则直接返回false, 代表没获得同步状态
    if (nanosTimeout <= 0L)
        return false;

    // 系统纳米时间 + 超时时间 = 过期总时间
    final long deadline = System.nanoTime() + nanosTimeout;

    // 使用当前线程构建共享Node结点, 并CAS+自旋直至入队成功
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        // 开始自旋
        for (;;) {
            // volatile方式获取前驱p
            final Node p = node.predecessor();

            // 如果p为头结点, 则尝试以共享模式获取同步器
            if (p == head) {
                // 失败(<0), 获取成功但后续共享不成功(=0), 获取成功且后续共享成功(>0)
                int r = tryAcquireShared(arg);

                // 如果获取成功且后续共享成功(>0), 则volatile更新node为头结点, 并CAS更新为传播结点
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return true;
                }
            }

            // 如果获取同步器失败, 则重新获取系统纳米时间, 用于更新过期总时间, 如果过期总时间小于等于0, 则直接返回false, 代表没获得同步状态
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;

            // 如果过期总时间大于0, 说明仍有效, 则放置node结点并阻塞其排队线程是否合理, 如果返回true代表确定了前驱为通知结点, 此时node的排队线程需要阻塞; 如果返回false代表还没能确定前驱为通知结点, 需要返回继续自旋判断, 此时node的排队线程不能被阻塞
            if (shouldParkAfterFailedAcquire(p, node) &&
                // 如果过期总时间仍大于1s钟, 说明仍然需要阻塞线程, 则调用LockSupport的定时阻塞方法阻塞当前线程
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);

            // 如果当前线程被中断或者退出阻塞, 则判断获取中断标志位, 如果为true则抛出中断异常
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        // 走到这里但failed为false, 说明在获取子类实现的tryAcquire方法时发生了异常, 或者自旋过程中发生了中断异常
        if (failed)
            // 则取消node结点(前驱不为头结点且需要传播时), 或者唤醒node结点的排队线程(前驱为头结点时)
            cancelAcquire(node);
    }
}
```

##### 核心源码 - 结点入队与出队

###### addWaiter

```java
// 使用当前线程构建独占/共享模式的Node结点, 并CAS+自旋直至入队成功
private Node addWaiter(Node mode) {
    // 当前线程作为等待队列结点中的排队线程, mode为传入的模式(Node.EXCLUSIVE为独占, Node.SHARED为共享)
    Node node = new Node(Thread.currentThread(), mode);

    // enq自旋入队前的尝试, 获取最新的尾结点, 尝试CAS当前线程结点加入队尾
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }

    }

    // 如果CAS失败, 说明队尾已被别的线程更新了, 则调用Node结点(含排队线程)自旋入队方法, 自旋 + CAS入队, 如果队列还没初始化则需要初始化
    enq(node);

    // node(含排队线程)自旋入队成功, 则返回node结点
    return node;
}
// CAS尾结点。仅由enq使用。
private final boolean compareAndSetTail(Node expect, Node update) {
    return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
}
```

###### enq

```java
// node结点(含排队线程)自旋入队方法, 自旋 + CAS入队, 如果队列还没初始化则需要初始化
private Node enq(final Node node) {
    // 自旋, 如果最新的尾结点为null, 说明需要初始化, 则CAS添加新结点作为头尾结点
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize 必须初始化
            if (compareAndSetHead(new Node()))
                tail = head;
        }
        // 如果最新的尾结点不为null, 说明已经被初始化了, 则CAS结点(含排队线程)加入队尾
        else {
            // 设置node前驱与t后继可以不是原子的, 保证t后继原子也能达到并发安全的效果, 因为t如果不为真正的前驱, 那么CAS设置后继也会失败, 从而导致下轮自旋重新更新node前驱
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
            // CAS失败, 说明队尾已被别的线程更新了, 则继续自旋
        }
    }
}
```

###### shouldParkAfterFailedAcquire

```java
// 放置node结点并阻塞其排队线程是否合理, 如果返回true代表确定了前驱为通知结点, 此时node的排队线程需要阻塞; 如果返回false代表还没能确定前驱为通知结点, 需要返回继续自旋判断, 此时node的排队线程不能被阻塞
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    // Node前驱的等待状态, 已取消的结点CANCELLED(1), 普通等待队列结点(0), 通知结点SIGNAL(-1), 条件队列结点CONDITION(-2), 传播结点PROPAGATE(-3)
    int ws = pred.waitStatus;

    // 如果前驱为通知结点, 由于已经设置了通知状态(释放时需要unpark后继结点的排队线程), 因此可以直接返回true即可, 代表前驱肯定为通知结点, 且node结点的排队线程需要阻塞
    if (ws == Node.SIGNAL)
        return true;

    // 如果前驱为已取消的结点, 则跳过已取消的前驱, 向前找到第一个有效结点作为前驱并链接
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    }
    // 如果前驱为普通结点|条件结点|传播结点, 则CAS更新前驱为通知结点, 代表需要前驱释放后需要unpark后继结点的排队线程
    else {
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }

    // 如果node前驱一开始不为通知结点, 则返回false, 代表node结点的排队线程不需要阻塞, 因为没能保证前驱为通知结点, 需要返回做自旋确认
    return false;
}
// 节点的 CAS waitStatus 字段。
private static final boolean compareAndSetWaitStatus(Node node,
                                                     int expect,
                                                     int update) {
    return unsafe.compareAndSwapInt(node, waitStatusOffset,
                                    expect, update);
}
```

###### parkAndCheckInterrupt

```java
// park阻塞当前线程, 然后检查当前线程是否中断(当前线程被中断, 会解除线程阻塞状态)
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

###### selfInterrupt

```java
// 中断该线程, 如果从Thread其他实例方法调用该方法, 则会清除中断状态, 然后会收到一个{@link InterruptedException}
static void selfInterrupt() {
    Thread.currentThread().interrupt();
}
```

###### cancelAcquire

```java
// 取消并脱钩node结点, 唤醒node后继中等待线程
private void cancelAcquire(Node node) {
    // Ignore if node doesn't exist
    // 如果节点不存在则忽略
    if (node == null)
        return;

    // 清空node的排队线程
    node.thread = null;

    // Skip cancelled predecessors
    // 跳过已取消的前驱, 直到找到为通知|条件|传播的结点作为新的前驱
    Node pred = node.prev;
    while (pred.waitStatus > 0)
        node.prev = pred = pred.prev;

    // volatile方式获取前驱的后继
    Node predNext = pred.next;

    // volatile方式更新node为取消结点
    node.waitStatus = Node.CANCELLED;

    // 如果我们是尾巴，请移开我们自己
    // If we are the tail, remove ourselves.
    // 如果node是最新的尾结点, 且尝试CAS更新尾结点为前驱
    if (node == tail && compareAndSetTail(node, pred)) {
        // 如果CAS更新尾结点成功, 则又CAS更新前驱的后继为null
        compareAndSetNext(pred, predNext, null);
        // 无论CAS更新前驱的后继结果怎么样, pred都为尾结点, 且它的next为null
    }
    // 如果node不是最新的尾结点
    else {
        int ws;

        // 如果前驱不为头结点, 且还没有被取消, 则CAS更新为通知结点
        if (pred != head &&
            ((ws = pred.waitStatus) == Node.SIGNAL || (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
            pred.thread != null) {
            // 如果CAS更新前驱为通知结点成功, 且前驱的排队线程存在, 则获取node最新的后继next
            Node next = node.next;

            // 如果后继next存在, 且还没有被取消, 则CAS更新前驱结点的后继为node最新的后继, 跳过了node结点, 说明node脱钩了
            if (next != null && next.waitStatus <= 0)
                compareAndSetNext(pred, predNext, next);
        }
        // 如果前驱为头结点, 或者已经被取消了, 或者CAS更新前驱为通知结点失败了
        else {
            // 则唤醒node后继结点中的排队线程, 如果不存在有效的后继结点, 则从最新的尾结点向前找第一个有效结点中的排队线程来进行唤醒
            unparkSuccessor(node);
        }

        // 如果node结点被取消了, 或者出队唤醒了, 则next自链接为自己, 方便GC
        node.next = node; // help GC
    }
}
```

###### setHeadAndPropagate

```java
// 设置node为最新的结点, 执行node结点的通知、传播特性
private void setHeadAndPropagate(Node node, int propagate) {
    // propagate: 失败(<0), 获取成功但后续共享不成功(=0), 获取成功且后续共享成功(>0)
    // 获取最新的头结点
    Node h = head; // Record old head for check below 记录下旧头以供检查

    // volatile更新node为头结点, 并清空排队线程、前驱指针, 方便GC回收thread和prev
    setHead(node);

    // 先看获得的同步器, 如果大于0, 说明需要传播
    if (propagate > 0
        // 如果没有同步器, 如果h为空, 说明队列中没有任何结点
        || h == null
        // 如果队列中有结点, 且h为通知|条件|传播结点, 说明有可能要传播
        || h.waitStatus < 0
        // 如果h也不需要传播, 则再重新获取最新头结点, 如果h为空, 则说明队列中没有任何结点
        || (h = head) == null
        // 如果队列中有结点, 且h为通知|条件|传播结点, 说明有可能要传播
        || h.waitStatus < 0) {
        // 获取指定node的后继s
        Node s = node.next;

        // 如果node后继的下一个等待条件的节点为特殊值SHARED, 则执行结点释放动作, 在共享模式下, CAS更新为传播结点, 代表发布传播成功;
        // 在独占模式下, 如果为SIGNAL结点, 则需要唤醒node后继结点中的排队线程或者从后往前找(非公平)的第一个有效结点的排队线程
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```

###### unparkSuccessor

```java
// 唤醒node后继结点中的排队线程, 如果不存在有效的后继结点, 则从最新的尾结点向前找(非公平方式)第一个有效结点中的排队线程来进行唤醒
private void unparkSuccessor(Node node) {
    // Node等待状态, 已取消的结点CANCELLED(1), 普通等待队列结点(0), 通知结点SIGNAL(-1), 条件队列结点CONDITION(-2), 传播结点PROPAGATE(-3)
    int ws = node.waitStatus;

    // 先更新node为普通结点
    // 如果Node等待状态小于0, 即可能是通知结点SIGNAL(-1), 条件队列结点CONDITION(-2), 传播结点PROPAGATE(-3), 则CAS更新等待状态为普通等待队列结点(0)
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    // volatile方式获取next结点作为唤醒的结点s
    Node s = node.next;

    // 如果s结点为null, 或者s为已取消的结点CANCELLED(1), 则无需唤醒
    if (s == null || s.waitStatus > 0) {
        // 清空s结点
        s = null;

        // 获取最新的尾结点t, 从后往前找第一个为有效的结点, 作为需要唤醒的结点s
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }

    // 如果确实有需要唤醒的s结点, 则LockSupport.unpark该结点的排队线程
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

##### 核心源码 - 释放同步器

###### doReleaseShared

```java
// 执行结点释放动作, 在共享模式下, CAS更新为传播结点, 代表发布传播成功; 在独占模式下, 如果为SIGNAL结点, 则需要唤醒node后继结点中的排队线程或者从后往前找(非公平方式)的第一个有效结点的排队线程
private void doReleaseShared() {
    // 开始自旋
    for (;;) {
        // volatile方式获取头结点h
        Node h = head;

        // 如果CLH队列中至少有1个以上的结点
        if (h != null && h != tail) {
            // Node等待状态, 已取消的结点CANCELLED(1), 普通等待队列结点(0), 通知结点SIGNAL(-1), 条件队列结点CONDITION(-2), 传播结点PROPAGATE(-3)
            int ws = h.waitStatus;

            // 如果h为通知结点SIGNAL(-1), 则CAS更新h等待状态为0
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    // CAS更新失败, 则继续自旋
                    continue;            // loop to recheck cases

                // CAS更新成功, 则唤醒node后继结点中的排队线程, 如果不存在有效的后继结点, 则从最新的尾结点向前找(非公平方式)第一个有效结点中的排队线程来进行唤醒
                unparkSuccessor(h);
            }
            // 如果h为普通等待队列结点(0), 则CAS更新为传播结点PROPAGATE(-3)
            else if (ws == 0 && !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                // CAS更新失败, 则继续自旋
                continue;                // loop on failed CAS
        }

        // 如果CLH队列只有1个结点 | 最新的head结点通知成功 | 更新为传播结点, 则退出自旋, 代表传播发布成功
        if (h == head)                   // loop if head changed
            break;
    }
}
```

##### 获取同步器 - 独占模式

- 独占模式下，tryAcquire返回的是boolean值，其含义取决于实现类定义的语义。
  - 经过推算，要实现独占就要保证：如果想要获取同步器，则当已存在独占线程时，则要返回false，当没存在独占线程时，则要返回true。
- 每个结点尝试获取同步器，获取失败的生成Node结点，并加入AQS主队列参与竞争，竞争失败的则会将前驱结点设置为SINGAL状态，然后阻塞。
- 每个结点（此时作为后继的前驱）释放同步器时，由于当前为SINGAL状态，所以会唤醒后继结点，重新参与AQS竞争。

###### tryAcquire

```java
// 尝试以独占模式获取同步器
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

###### acquire

```java
// 以阻塞、独占模式获取同步器
public final void acquire(int arg) {
    // 尝试以独占模式获取同步器
    if (!tryAcquire(arg) &&
        // 独占式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回; 否则放置node结点并阻塞其排队线程, 如果当前线程有被中断过, 则还需要继续自旋, 直到获取到同步器, 返回true代表有被中断过让上层API处理; 另外如果调用tryAcquire方法时发生了异常, 则还需要则取消node结点
        acquireQueued(
            // 使用当前线程构建独占模式的Node结点, 并CAS+自旋直至入队成功
            addWaiter(Node.EXCLUSIVE), arg
        )
       )
        // 中断当前线程
        selfInterrupt();
}
```

###### acquireInterruptibly

```java
// 以阻塞、独占可中断模式获取同步器
public final void acquireInterruptibly(int arg) throws InterruptedException {
    // 如果当前线程被中断了, 则抛出中断异常
    if (Thread.interrupted())
        throw new InterruptedException();

    // 尝试以独占模式获取同步器
    if (!tryAcquire(arg))
        // 独占可中断式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回; 否则放置node结点并阻塞其排队线程, 如果调用tryAcquire方法时发生了异常或者自旋过程中发生了中断异常, 则还需要则取消node结点
        doAcquireInterruptibly(arg);
}
```

###### tryAcquireNanos

```java
// 以阻塞、独占定时可中断模式获取同步器
public final boolean tryAcquireNanos(int arg, long nanosTimeout) throws InterruptedException {
    // 如果当前线程被中断了, 则抛出中断异常
    if (Thread.interrupted())
        throw new InterruptedException();
    // 尝试以独占模式获取同步器
    return tryAcquire(arg) ||
        // 独占定时可中断式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回; 否则放置node结点并阻塞其排队线程, 如果调用tryAcquire方法时发生了异常或者自旋过程中发生了中断异常, 则还需要则取消node结点
        doAcquireNanos(arg, nanosTimeout);
}
```

##### 获取同步器 - 共享模式

- 共享模式下，tryAcquireShared返回的是int值，其含义取决于实现类定义的语义。
- PROPAGATE只是占位符：
  - 获取到的资源非最后一个资源的结点，状态为PROPAGATE。
  - 获取到的资源最后一个资源的结点，状态一开始为0，不过很快就会在下一个结点阻塞前，设置为SINGAL；
- PROPAGATE和SINGAL结点的线程都在运行中，但PROPAGATE结点的next被清空了，相当于已经出队；而SINGAL结点的next则没有，相当于队头；
  - 当它们释放共享锁时，又会重复出现PROPAGATE和SINGAL结点，然后往复之前的操作。

###### tryAcquireShared

```java
// 尝试以共享模式获取同步器, 失败(<0), 获取成功但后续共享不成功(=0), 获取成功且后续共享成功(>0)
protected int tryAcquireShared(int arg) {
    throw new UnsupportedOperationException();
}
```

###### acquireShared

```java
// 以阻塞、共享模式获取同步器
public final void acquireShared(int arg) {
    // 尝试以共享模式获取同步器, 失败(<0), 获取成功但后续共享不成功(=0), 获取成功且后续共享成功(>0)
    if (tryAcquireShared(arg) < 0)
        // 共享式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回; 否则放置node结点并阻塞其排队线程, 如果当前线程有被中断过, 则还需要继续自旋, 直到获取到同步器, 返回true代表有被中断过让上层API处理; 另外如果调用tryAcquire方法时发生了异常, 则还需要则取消node结点
        doAcquireShared(arg);
}
```

###### acquireSharedInterruptibly

```java
// 以阻塞、共享可中断模式获取同步器
public final void acquireSharedInterruptibly(int arg) throws InterruptedException {
    // 如果当前线程被中断了, 则抛出中断异常
    if (Thread.interrupted())
        throw new InterruptedException();

    // 尝试以共享模式获取同步器, 失败(<0), 获取成功但后续共享不成功(=0), 获取成功且后续共享成功(>0)
    if (tryAcquireShared(arg) < 0)
        // 共享可中断式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回; 否则放置node结点并阻塞其排队线程, 如果调用tryAcquire方法时发生了异常或者自旋过程中发生了中断异常, 则还需要则取消node结点
        doAcquireSharedInterruptibly(arg);
}
```

###### tryAcquireSharedNanos

```java
// 以阻塞、共享定时可中断模式获取同步器
public final boolean tryAcquireSharedNanos(int arg, long nanosTimeout) throws InterruptedException {
    // 如果当前线程被中断了, 则抛出中断异常
    if (Thread.interrupted())
        throw new InterruptedException();

    // 尝试以共享模式获取同步器, 失败(<0), 获取成功但后续共享不成功(=0), 获取成功且后续共享成功(>0)
    return tryAcquireShared(arg) >= 0 ||
        // 共享定时可中断式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回; 否则放置node结点并阻塞其排队线程, 如果调用tryAcquire方法时发生了异常或者自旋过程中发生了中断异常, 则还需要则取消node结点
        doAcquireSharedNanos(arg, nanosTimeout);
}
```

##### 释放同步器 - 独占模式

###### tryRelease

```java
// 尝试释放独占模式的同步器
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
```

###### release

```java
// 尝试释放独占模式的同步器, 如果释放成功, 则返回true; 如果释放失败, 则返回false
public final boolean release(int arg) {
    // 尝试释放独占模式的同步器
    if (tryRelease(arg)) {
        Node h = head;

        // 释放成功, 如果头结点(也就是被释放的结点)不为普通结点
        if (h != null && h.waitStatus != 0)
            // 则唤醒node后继结点中的排队线程, 如果不存在有效的后继结点, 则从最新的尾结点向前找第一个有效结点中的排队线程来进行唤醒
            unparkSuccessor(h);

        // 唤醒后返回true, 代表已成功释放
        return true;
    }

    // 释放失败则返回false, 代表释放失败
    return false;
}
```

##### 释放同步器 - 共享模式

###### tryReleaseShared

```java
// 尝试释放共享模式的同步器
protected boolean tryReleaseShared(int arg) {
    throw new UnsupportedOperationException();
}
```

###### releaseShared

```java
// 释放共享模式的同步器, 如果释放成功, 则返回true; 如果释放失败, 则返回false
public final boolean releaseShared(int arg) {
    // 尝试释放共享模式的同步器
    if (tryReleaseShared(arg)) {
        // 执行结点释放动作, 在共享模式下, CAS更新为传播结点, 代表发布传播成功; 在独占模式下, 如果为SIGNAL结点, 则需要唤醒node后继结点中的排队线程或者从后往前找的第一个有效结点的排队线程
        doReleaseShared();

        // 传播后返回true, 代表已成功释放
        return true;
    }

    // 释放失败则返回false, 代表释放失败
    return false;
}
```

##### 其他辅助方法

###### isHeldExclusively

```java
// 判断同步模式是独占模式还是共享模式
protected boolean isHeldExclusively() {
    throw new UnsupportedOperationException();
}
```

###### hasQueuedThreads

```java
// 判断CLH队列中是否有线程在等待获取同步器, 由于中断和超时导致的结点取消随时都可能发生, 所以返回{@code true}并不能保证CLH队列中接下来都存在结点。
public final boolean hasQueuedThreads() {
    return head != tail;
}
```

###### hasContended

```java
// 判断CLH队头是否存在, 如果存在, 则代表同步器有被别的结点发生过争抢
public final boolean hasContended() {
    return head != null;
}
```

###### getFirstQueuedThread

```java
// 获取CLH队列中第一个排队线程(头后继检查两次+从尾到前遍历), 如果没有则返回null
public final Thread getFirstQueuedThread() {
    // 如果CLH队列中没有结点, 则返回null; 否则, 获取CLH中的第一个排队线程: 先检查从头检查两次头后继结点, 如果存在排队线程则返回; 如果不存在则从尾往前找, 返回第一个有效的不为头结点的排队结点的排队线程
    return (head == tail) ? null : fullGetFirstQueuedThread();
}
// 获取CLH中的第一个排队线程: 先检查从头检查两次头后继结点, 如果存在排队线程则返回; 如果不存在则从尾往前找, 返回第一个有效的不为头结点的排队结点的排队线程
private Thread fullGetFirstQueuedThread() {
    Node h, s;
    Thread st;

    // 头结点h, 头结点后继s作为第一个结点, 如果s前驱为头结点, 且存在排队线程st
    if (((h = head) != null && (s = h.next) != null &&
         s.prev == head && (st = s.thread) != null) ||
        // 如果不存在, 则再检查一次: 头结点h, 头结点后继s作为第一个结点, 如果s前驱为头结点, 且存在排队线程st
        ((h = head) != null && (s = h.next) != null &&
         s.prev == head && (st = s.thread) != null))
        // 检查两次了两次, 如果排队线程st存在, 则返回st作为CLH中的第一个排队线程
        return st;

    // 如果检查了两次都没找到第一个排队线程, 则获取最新的尾结点, 从后往前找
    Node t = tail;
    Thread firstThread = null;

    // 如果t不为null, 且不为头结点, 则停止向前遍历, 即向前找到第一个有效的不为头结点的排队结点
    while (t != null && t != head) {
        Thread tt = t.thread;
        if (tt != null)
            firstThread = tt;
        t = t.prev;
    }

    // 返回找到的第一个有效的不为头结点的排队结点的排队线程, 作为CLH中的第一个排队线程
    return firstThread;
}
```

###### isQueued

```java
// 判断目标线程是否在CLH队列中: 从尾往前遍历, 如果Thread为同一个地址, 在返回true, 否则返回false
public final boolean isQueued(Thread thread) {
    if (thread == null)
        throw new NullPointerException();

    // 从尾往前遍历, 如果Thread为同一个地址, 在返回true, 发欧泽返回false
    for (Node p = tail; p != null; p = p.prev)
        if (p.thread == thread)
            return true;
    return false;
}
```

###### apparentlyFirstQueuedIsExclusive

```java
// 判断第一个排队线程是否以独占模式等待
final boolean apparentlyFirstQueuedIsExclusive() {
    Node h, s;

    // 头结点h, 头结点后继s, 判断下一个等待条件的节点是否为特殊值SHARED, 如果不是且s的排队线程不为null, 则返回true; 否则返回false
    return (h = head) != null &&
        (s = h.next)  != null &&
        !s.isShared()         &&
        s.thread != null;
}
```

###### hasQueuedPredecessors

```java
// 判断此刻CLH中是否存在有等待时间比当前线程长的线程, 旨在由公平同步器使用, 以避免插入结点到CLH队列中
public final boolean hasQueuedPredecessors() {
    Node t = tail; // 以相反的初始化顺序读取字段
    Node h = head;
    Node s;

    // 如果CLH队列中多于一个结点, 如果头后继s刚刚释放, 或者头后继s的等待线程不为当前线程, 说明此刻CLH中确实存在有等待时间比当前线程长的线程
    return h != t && ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

###### getQueueLength

```java
// 从尾向前遍历统计非null的排队线程个数
public final int getQueueLength() {
    int n = 0;

    // 从尾向前遍历统计非null的排队线程个数
    for (Node p = tail; p != null; p = p.prev) {
        if (p.thread != null)
            ++n;
    }
    return n;
}
```

###### getQueuedThreads

```java
// 从尾向前遍历获取非null的排队线程列表
public final Collection<Thread> getQueuedThreads() {
    ArrayList<Thread> list = new ArrayList<Thread>();

    // 从尾向前遍历获取非null的排队线程列表
    for (Node p = tail; p != null; p = p.prev) {
        Thread t = p.thread;
        if (t != null)
            list.add(t);
    }
    return list;
}
```

###### getExclusiveQueuedThreads

```java
// 从尾向前遍历获取非共享的排队线程列表
public final Collection<Thread> getExclusiveQueuedThreads() {
    ArrayList<Thread> list = new ArrayList<Thread>();

    // 从尾向前遍历获取非共享的排队线程列表
    for (Node p = tail; p != null; p = p.prev) {
        if (!p.isShared()) {
            Thread t = p.thread;
            if (t != null)
                list.add(t);
        }
    }
    return list;
}
```

###### getSharedQueuedThreads

```java
// 从尾向前遍历获取共享的排队线程列表
public final Collection<Thread> getSharedQueuedThreads() {
    ArrayList<Thread> list = new ArrayList<Thread>();

    // 从尾向前遍历获取共享的排队线程列表
    for (Node p = tail; p != null; p = p.prev) {
        if (p.isShared()) {
            Thread t = p.thread;
            if (t != null)
                list.add(t);
        }
    }
    return list;
}
```

##### CondictionObject相关方法

###### isOnSyncQueue

```java
// 判断node结点的是不是在公平队列中排队
final boolean isOnSyncQueue(Node node) {
    // 如果node为条件结点, 或者node为头结点则返回false
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;

    // 如果有前驱不为头结点, 且有后继时，则认为node结点肯定在队列中, 此时返回true
    if (node.next != null) // If has successor, it must be on queue
        return true;

    // 如果有前驱不为头结点, 且没有后继, 可能该结点CAS失败还没入队(即设置了前驱但CAS后继失败时), 则获取最新尾结点开始自旋向前找, 如果找到对应的结点则返回true, 如果找不到则返回false
    return findNodeFromTail(node);
}
// 获取最新尾结点开始自旋向前找, 如果找到对应的结点则返回true, 如果找不到则返回false
private boolean findNodeFromTail(Node node) {
    Node t = tail;

    // 获取最新尾结点开始自旋向前找, 如果找到对应的结点则返回true, 如果找不到则返回false
    for (;;) {
        if (t == node)
            return true;
        if (t == null)
            return false;
        t = t.prev;
    }
}
```

###### transferForSignal

```java
// 将node从条件队列转移到了AQS同步队列, 入队后|入队唤醒成功后, 则返回true
final boolean transferForSignal(Node node) {
    // CAS更新为普通结点, 如果CAS更新失败, 则返回false, 代表结点已被更新(取消)
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    // node结点(含排队线程)自旋入队方法, 自旋 + CAS入队, 如果队列还没初始化则需要初始化
    Node p = enq(node);

    // node结点入队成功后, 如果结点状态为已取消, 或者CAS更新node结点为通知结点失败, 则唤醒node的排队线程
    int ws = p.waitStatus;
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);

    // 入队后|入队唤醒成功后, 则返回true, 代表node成功从条件队列转移到了同步队列
    return true;
}
```

###### transferAfterCancelledWait

```java
// 将已经取消了等待的节点转移到同步队列, 如果转移成功则返回true, 失败则返回false
final boolean transferAfterCancelledWait(Node node) {
    // CAS更新为普通结点
    if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
        // 如果CAS更新成功, 则node结点(含排队线程)自旋入队方法, 自旋 + CAS入队, 如果队列还没初始化则需要初始化
        enq(node);

        // 入队成功后返回true
        return true;
    }

    // 判断node结点的是不是在公平队列中排队, 如果不在则继续让步+自旋
    while (!isOnSyncQueue(node))
        Thread.yield();

    // 如果node结点在公平队列中排队了, 则返回false
    return false;
}
```

###### fullyRelease

```java
// volatile方式获取同步状态并尝试释放独占模式的同步器, 如果释放成功, 则返回刚刚获取到的同步状态; 如果失败则抛出异常后更改结点为取消结点
final int fullyRelease(Node node) {
    boolean failed = true;
    try {
        // volatile方式获取同步状态
        int savedState = getState();

        // 尝试释放独占模式的同步器, 如果释放成功, 则返回刚刚获取到的同步状态
        if (release(savedState)) {
            failed = false;
            return savedState;
        }
        // 如果释放失败, 则抛出IllegalMonitorStateException异常
        else {
            throw new IllegalMonitorStateException();
        }
    } finally {
        // 最后如果释放失败, 还需要volatile方式更改结点为取消结点
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}
```

###### owns

```java
// 查询给定的ConditionObject是否使用此同步器作为其锁
public final boolean owns(ConditionObject condition) {
    return condition.isOwnedBy(this);
}
```

###### hasWaiters

```java
// 查询是否有任何线程正在等待与此同步器关联的给定条件
public final boolean hasWaiters(ConditionObject condition) {
    if (!owns(condition))
        throw new IllegalArgumentException("Not owner");
    return condition.hasWaiters();
}
```

###### getWaitQueueLength

```java
// 返回等待与此同步器关联的给定条件的线程数的估计值
public final int getWaitQueueLength(ConditionObject condition) {
    if (!owns(condition))
        throw new IllegalArgumentException("Not owner");
    return condition.getWaitQueueLength();
}
```

###### getWaitingThreads

```java
// 返回一个包含可能正在等待与此同步器关联的给定条件的线程的集合
public final Collection<Thread> getWaitingThreads(ConditionObject condition) {
    if (!owns(condition))
        throw new IllegalArgumentException("Not owner");

    // 返回一个包含可能正在等待此Condition的线程的集合 => 从头遍历条件队列, 收集CONDITION结点的线程
    return condition.getWaitingThreads();
}
```

##### CondictionObject

###### 特点

- 等待条件的线程也**使用相同的节点**，但使用了额外的指针来维护一个简单的**条件队列**。在等待时，一个结点被插入到条件队列中。然后根据信号，该结点将会被转移到主队列中。
  - 其中，AQS使用**status字段**来标记节点所在的队列。
- CondictionObejct，作为{@link Lock}实现基础的{@link AbstractQueuedSynchronizer}的条件实现。
- 对于条件结点，await时会加入条件队列进行阻塞；signal时唤醒后会从条件队列转移到AQS主队列，去参与AQS竞争，竞争失败的会重新阻塞，阻塞后依赖于AQS机制实现唤醒。

###### 构造方法

```java
public class ConditionObject implements Condition, java.io.Serializable {
    
    private static final int REINTERRUPT =  1;// 模式意味着在退出等待时重新中断
    private static final int THROW_IE = -1;//模式意味着在退出等待时抛出nterruptedException
    
    private transient Node firstWaiter;// 条件队列的第一个节点
    private transient Node lastWaiter;// 条件队列的最后一个节点
    
    // 创建一个新的 {@code ConditionObject} 实例
    public ConditionObject() { }
}
```

###### 核心源码 - addConditionWaiter

```java
// 添加一个条件结点, 如果最后一个条件结点t不是条件结点, 则会从条件队列中取消非条件结点, 将其nextWaiter链接到最后一个条件结点末尾
private Node addConditionWaiter() {
    // 获取条件队列的最后一个节点t
    Node t = lastWaiter;

    // 如果最后一个条件结点t不是条件结点
    if (t != null && t.waitStatus != Node.CONDITION) {
        // 从条件队列中取消非条件结点, 将其nextWaiter链接到最后一个条件结点末尾
        unlinkCancelledWaiters();

        // 获取最后一个条件结点
        t = lastWaiter;
    }

    // 如果最后一个条件结点仍然是条件结点, 或者条件队列经过整理后, 则使用当前线程构造条件结点node
    Node node = new Node(Thread.currentThread(), Node.CONDITION);

    // 更新node为最后一个条件结点
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;

    // 返回新构建的node条件结点
    return node;
}
```

###### 核心源码 - unlinkCancelledWaiters

```java
// 从条件队列中取消非条件结点, 将其nextWaiter链接到最后一个条件结点末尾
private void unlinkCancelledWaiters() {
    // 条件队列的第一个节点t
    Node t = firstWaiter;
    Node trail = null;

    // 从头开始遍历原条件队列链表
    while (t != null) {
        // 条件队列的下一个节点next
        Node next = t.nextWaiter;

        // 如果结点不为条件结点
        if (t.waitStatus != Node.CONDITION) {
            // 清空原下一个结点
            t.nextWaiter = null;

            // 使用next结点重新构造新的条件trail链表
            if (trail == null)
                firstWaiter = next;
            else
                trail.nextWaiter = next;

            // 遍历原链表尾部后, 追加新构造的链表到最后一个条件结点末尾
            if (next == null)
                lastWaiter = trail;
        }
        // 如果为条件结点, 则更换trail链头, 代表使用最后一个条件结点
        else
            trail = t;

        // 继续遍历原条件队列链表
        t = next;
    }
}
```

###### 核心源码 - doSignal

```java
// 删除并传输节点到AQS同步队列, 直到命中未取消的一个结点(第一个转移成功的)或为null的(结尾的)结点
private void doSignal(Node first) {
    do {
        // 遍历first结点的条件队列节点, 如果为null, 则清空该结点的条件队列
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        first.nextWaiter = null;
    } while (
        // 清空后, 则将first结点从条件队列转移到了AQS同步队列, 入队后|入队唤醒成功后, 则返回true, 此时直接返回, 代表转移一次first结点即可
        !transferForSignal(first) &&
        // 如果CAS失败, 说明first结点转移失败, 则继续转移通知原first条件队列的结点, 直到结尾
        (first = firstWaiter) != null);
}
```

###### 核心源码 - doSignalAll

```java
// 遍历、清空、转移first结点的排队线程到AQS同步队列
private void doSignalAll(Node first) {
    lastWaiter = firstWaiter = null;
    do {
        Node next = first.nextWaiter;
        first.nextWaiter = null;

        // 将first从条件队列转移到了AQS同步队列, 入队后|入队唤醒成功后, 则返回true
        transferForSignal(first);

        // 遍历first结点的排队线程
        first = next;
    } while (first != null);
}
```

###### 核心源码 - checkInterruptWhileWaiting

```java
// 检查是否存在等待时中断, 如果当前线程被中断, 则将等待结点转移到同步队列, 如果转移成功则返回THROW_IE(-1), 如果转移失败则返回REINTERRUPT(1); 如果等待时没有发生异常, 则返回0
private int checkInterruptWhileWaiting(Node node) {
    // 测试当前线程是否被中断, 该方法会清除线程的中断状态
    return Thread.interrupted() ?
        // 将已经取消了等待的节点转移到同步队列, 如果转移成功则返回true, 失败则返回false
        (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) : 0;
}
```

###### 核心源码 - reportInterruptAfterWait

```java
// 根据模式，抛出 InterruptedException、重新中断当前线程或不执行任何操作
private void reportInterruptAfterWait(int interruptMode) throws InterruptedException {
    // 模式意味着在退出等待时抛出 InterruptedException
    if (interruptMode == THROW_IE)
        throw new InterruptedException();
    // 模式意味着在退出等待时重新中断
    else if (interruptMode == REINTERRUPT)
        // 中断该线程, 如果从Thread其他实例方法调用该方法, 则会清除中断状态, 然后会收到一个{@link InterruptedException}
        selfInterrupt();
}
```

###### 阻塞方法 - await

```java
// 阻塞等待当前线程, 如果线程被中断则加入同步队列, 且在获取到同步器后处理中断
public final void await() throws InterruptedException {
    // 测试当前线程是否被中断, 该方法会清除线程的中断状态
    if (Thread.interrupted())
        throw new InterruptedException();

    // 添加一个条件结点, 如果最后一个条件结点t不是条件结点, 则会从条件队列中取消非条件结点, 将其nextWaiter链接到最后一个条件结点末尾
    Node node = addConditionWaiter();

    // volatile方式获取同步状态并尝试释放独占模式的同步器, 如果释放成功, 则返回刚刚获取到的同步状态; 如果失败则抛出异常后更改结点为取消结点
    int savedState = fullyRelease(node);

    // 判断node结点的是不是在公平队列中排队, 如果不是则阻塞当前线程
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);

        // 检查是否存在等待时中断, 如果当前线程被中断, 则将等待结点转移到同步队列, 如果转移成功则返回THROW_IE(-1), 如果转移失败则返回REINTERRUPT(1); 如果等待时没有发生异常, 则返回0
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            // 如果抛出了异常, 则退出自旋
            break;
    }

    // 如果在同步队列中, 则独占式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回;
    // 否则放置node结点并阻塞其排队线程, 如果当前线程有被中断过, 则还需要继续自旋, 直到获取到同步器, 返回true代表有被中断过让上层API处理;
    // 另外如果调用tryAcquire方法时发生了异常, 则还需要则取消node结点
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;

    // 从条件队列中取消非条件结点, 将其nextWaiter链接到最后一个条件结点末尾
    if (node.nextWaiter != null) // clean up if cancelled 如果取消则清理
        unlinkCancelledWaiters();

    // 如果等待期间发生中断, 则抛出InterruptedException、重新中断当前线程或不执行任何操作
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

###### 阻塞方法 - awaitNanos

```java
// 定时阻塞等待当前线程, 如果线程被中断则加入同步队列, 且在获取到同步器后处理中断
public final long awaitNanos(long nanosTimeout) throws InterruptedException {
    // 测试当前线程是否被中断, 该方法会清除线程的中断状态
    if (Thread.interrupted())
        throw new InterruptedException();

    // 添加一个条件结点, 如果最后一个条件结点t不是条件结点, 则会从条件队列中取消非条件结点, 将其nextWaiter链接到最后一个条件结点末尾
    Node node = addConditionWaiter();

    // volatile方式获取同步状态并尝试释放独占模式的同步器, 如果释放成功, 则返回刚刚获取到的同步状态; 如果失败则抛出异常后更改结点为取消结点
    int savedState = fullyRelease(node);

    // 判断node结点的是不是在公平队列中排队, 如果不是则阻塞当前线程
    final long deadline = System.nanoTime() + nanosTimeout;
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        // 如果超时时间小于0, 说明发生了超时, 将已经取消了等待的节点转移到同步队列, 如果转移成功则返回true, 失败则返回false
        if (nanosTimeout <= 0L) {
            transferAfterCancelledWait(node);
            break;
        }
        // 如果过期总时间仍大于1s钟, 说明仍然需要阻塞线程, 则调用LockSupport的定时阻塞方法阻塞当前线程
        if (nanosTimeout >= spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanosTimeout);

        // 检查是否存在等待时中断, 如果当前线程被中断, 则将等待结点转移到同步队列, 如果转移成功则返回THROW_IE(-1), 如果转移失败则返回REINTERRUPT(1); 如果等待时没有发生异常, 则返回0
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;

        // 重新计算超时时间
        nanosTimeout = deadline - System.nanoTime();
    }

    // 如果在同步队列中, 则独占式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回;
    // 否则放置node结点并阻塞其排队线程, 如果当前线程有被中断过, 则还需要继续自旋, 直到获取到同步器, 返回true代表有被中断过让上层API处理;
    // 另外如果调用tryAcquire方法时发生了异常, 则还需要则取消node结点
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;

    // 从条件队列中取消非条件结点, 将其nextWaiter链接到最后一个条件结点末尾
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();

    // 如果等待期间发生中断, 则抛出InterruptedException、重新中断当前线程或不执行任何操作
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);

    // 返回剩余超时时间
    return deadline - System.nanoTime();
}
```

###### 阻塞方法 - awaitUntil

```java
// 绝对定时阻塞等待当前线程, 如果线程被中断则加入同步队列, 且在获取到同步器后处理中断
public final boolean awaitUntil(Date deadline) throws InterruptedException {
    // 获取指定超时的绝对时间
    long abstime = deadline.getTime();

    // 测试当前线程是否被中断, 该方法会清除线程的中断状态
    if (Thread.interrupted())
        throw new InterruptedException();

    // 添加一个条件结点, 如果最后一个条件结点t不是条件结点, 则会从条件队列中取消非条件结点, 将其nextWaiter链接到最后一个条件结点末尾
    Node node = addConditionWaiter();

    // volatile方式获取同步状态并尝试释放独占模式的同步器, 如果释放成功, 则返回刚刚获取到的同步状态; 如果失败则抛出异常后更改结点为取消结点
    int savedState = fullyRelease(node);

    // 判断node结点的是不是在公平队列中排队, 如果不是则阻塞当前线程
    boolean timedout = false;
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        // 如果发生了超时,  则将已经取消了等待的节点转移到同步队列, 如果转移成功则返回true, 失败则返回false
        if (System.currentTimeMillis() > abstime) {
            timedout = transferAfterCancelledWait(node);
            break;
        }

        // 如果还没超时, 则阻塞指定的时间
        LockSupport.parkUntil(this, abstime);

        // 检查是否存在等待时中断, 如果当前线程被中断, 则将等待结点转移到同步队列, 如果转移成功则返回THROW_IE(-1), 如果转移失败则返回REINTERRUPT(1); 如果等待时没有发生异常, 则返回0
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }

    // 如果在同步队列中, 则独占式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回;
    // 否则放置node结点并阻塞其排队线程, 如果当前线程有被中断过, 则还需要继续自旋, 直到获取到同步器, 返回true代表有被中断过让上层API处理;
    // 另外如果调用tryAcquire方法时发生了异常, 则还需要则取消node结点
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;

    // 从条件队列中取消非条件结点, 将其nextWaiter链接到最后一个条件结点末尾
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();

    // 如果等待期间发生中断, 则抛出InterruptedException、重新中断当前线程或不执行任何操作
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);

    // 返回是否发生了超时
    return !timedout;
}
```

###### 阻塞方法 - await(..，..)

```java
// 定时阻塞等待当前线程, 如果线程被中断则加入同步队列, 且在获取到同步器后处理中断
public final boolean await(long time, TimeUnit unit) throws InterruptedException {
    // 获取指定的超时时间
    long nanosTimeout = unit.toNanos(time);

    // 测试当前线程是否被中断, 该方法会清除线程的中断状态
    if (Thread.interrupted())
        throw new InterruptedException();

    // 添加一个条件结点, 如果最后一个条件结点t不是条件结点, 则会从条件队列中取消非条件结点, 将其nextWaiter链接到最后一个条件结点末尾
    Node node = addConditionWaiter();

    // volatile方式获取同步状态并尝试释放独占模式的同步器, 如果释放成功, 则返回刚刚获取到的同步状态; 如果失败则抛出异常后更改结点为取消结点
    int savedState = fullyRelease(node);

    // 判断node结点的是不是在公平队列中排队, 如果不是则阻塞当前线程
    final long deadline = System.nanoTime() + nanosTimeout;
    boolean timedout = false;
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        // 如果超时时间小于0, 说明发生了超时, 将已经取消了等待的节点转移到同步队列, 如果转移成功则返回true, 失败则返回false
        if (nanosTimeout <= 0L) {
            timedout = transferAfterCancelledWait(node);
            break;
        }
        // 如果过期总时间仍大于1s钟, 说明仍然需要阻塞线程, 则调用LockSupport的定时阻塞方法阻塞当前线程
        if (nanosTimeout >= spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanosTimeout);

        // 检查是否存在等待时中断, 如果当前线程被中断, 则将等待结点转移到同步队列, 如果转移成功则返回THROW_IE(-1), 如果转移失败则返回REINTERRUPT(1); 如果等待时没有发生异常, 则返回0
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;

        // 重新计算超时时间
        nanosTimeout = deadline - System.nanoTime();
    }

    // 如果在同步队列中, 则独占式自旋入队的核心逻辑, 如果node前驱刚释放, 则更新node为头结点且不用阻塞直接返回;
    // 否则放置node结点并阻塞其排队线程, 如果当前线程有被中断过, 则还需要继续自旋, 直到获取到同步器, 返回true代表有被中断过让上层API处理;
    // 另外如果调用tryAcquire方法时发生了异常, 则还需要则取消node结点
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;

    // 从条件队列中取消非条件结点, 将其nextWaiter链接到最后一个条件结点末尾
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();

    // 如果等待期间发生中断, 则抛出InterruptedException、重新中断当前线程或不执行任何操作
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);

    // 返回是否发生了超时
    return !timedout;
}
```

###### 唤醒方法 - signal

```java
// 独占模式下唤醒第一个条件等待队列结点
public final void signal() {
    // 判断当前线程是否为独占的线程, 如果不是则抛出异常
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();

    // 获取第一个条件等待队列结点first
    Node first = firstWaiter;
    if (first != null)
        // 删除并传输first节点到AQS同步队列, 直到命中未取消的一个结点(第一个转移成功的)或为null的(结尾的)结点
        doSignal(first);
}
```

###### 唤醒方法 - signalAll

```java
// 独占模式下唤醒所有条件等待队列结点
public final void signalAll() {
    // 判断当前线程是否为独占的线程, 如果不是则抛出异常
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();

    // 获取第一个条件等待队列结点first
    Node first = firstWaiter;
    if (first != null)
        // 遍历、清空、转移first结点的排队线程到AQS同步队列
        doSignalAll(first);
}
```

###### 辅助方法 - isOwnedBy

```java
// 如果由给定的AQS是否当前AQS创建的，则返回true
final boolean isOwnedBy(AbstractQueuedSynchronizer sync) {
    return sync == AbstractQueuedSynchronizer.this;
}
```

###### 辅助方法 - hasWaiters

```java
// 查询是否有线程在此条件下等待, 从头遍历条件队列, 如果结点状态为CONDITION结点, 则返回true, 否则返回false
protected final boolean hasWaiters() {
    // 判断当前线程是否为独占的线程, 如果不是则抛出异常
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();

    // 从头遍历条件队列, 如果结点状态为CONDITION结点, 则返回true, 否则返回false
    for (Node w = firstWaiter; w != null; w = w.nextWaiter) {
        if (w.waitStatus == Node.CONDITION)
            return true;
    }

    return false;
}
```

###### 辅助方法 - getWaitQueueLength

```java
// 返回等待此条件的线程数的估计值 => 从头遍历条件队列, 统计CONDITION结点的个数
protected final int getWaitQueueLength() {
    // 判断当前线程是否为独占的线程, 如果不是则抛出异常
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();

    // 从头遍历条件队列, 统计CONDITION结点的个数
    int n = 0;
    for (Node w = firstWaiter; w != null; w = w.nextWaiter) {
        if (w.waitStatus == Node.CONDITION)
            ++n;
    }
    return n;
}
```

###### 辅助方法 - getWaitingThreads

```java
// 返回一个包含可能正在等待此Condition的线程的集合 => 从头遍历条件队列, 收集CONDITION结点的线程
protected final Collection<Thread> getWaitingThreads() {
    // 判断当前线程是否为独占的线程, 如果不是则抛出异常
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();

    // 从头遍历条件队列, 收集CONDITION结点的线程
    ArrayList<Thread> list = new ArrayList<Thread>();
    for (Node w = firstWaiter; w != null; w = w.nextWaiter) {
        if (w.waitStatus == Node.CONDITION) {
            Thread t = w.thread;
            if (t != null)
                list.add(t);
        }
    }
    return list;
}
```

### 典型实现

#### ReentrantLock

##### Lock

- Lock，是用于**控制多个线程对共享资源的访问**的工具。
  - 通常的Lock实现，提供对共享资源的独占访问：一次只有一个线程可以获取锁，所有对共享资源的访问都需要先获取锁。
  - 但是，某些Lock实现，可能允许并发访问共享资源，例如 {@link ReadWriteLock} 的读锁。
- {@code Lock} 实现，提供了比使用 {@code synchronized} 方法和语句**更广泛**的锁定操作，允许**更灵活**的结构（可能具有完全不同的属性），并且可能支持多个关联的 {@link Condition} 对象。
  - {@code synchronized}方法或语句的使用，提供对与每个对象关联的**隐式监视器锁**的访问，但强制所有锁获取和释放以块结构方式发生：当获取多个锁时，它们必须在**相反的顺序**，并且所有锁必须在它们被获取的同一个词法范围内释放。
    - 虽然{@code synchronized}方法和语句的作用域机制，使使用监视器锁编程变得更加容易，并有助于避免许多涉及锁的常见编程错误，但在某些情况下，可能需要以更灵活的方式使用锁。
  - {@code Lock}接口的实现，允许在不同范围内获取和释放锁，并允许以任何顺序获取和释放多个锁，从而实现这种灵活的需求。
    - {@code Lock}实现，通过提供非阻塞的获取锁的尝试({@link #tryLock()})来提供比使用{@code synchronized}方法和语句更多的功能。比如尝试获取锁{@link #tryLock()}，允许被中断方式获取锁（{@link #lockInterruptably}，以及尝试获取可以超时的锁（{@link #tryLock(long, TimeUnit)}）。
    - 但这种增加的灵活性带来了额外的责任，一是**锁不不会自动释放**，必须手动调用unlock方法才释放；二是当lock和unlock发生在不同的作用域时，必须注意确保持有锁时执行的所有代码，都受到**try-finally或try-catch的保护，以确保在必要时unlock**。
    - 另外，{@code Lock}类还可以提供与隐式监视器锁完全不同的行为和语义，例如保证排序、不可重入使用或死锁检测。如果实现提供了这样的专门语义，那么实现必须记录这些语义。
- 请注意，{@code Lock}实例只是普通对象，它们本身可以用作{@code synchronized}语句中的目标。获取{@code Lock}实例的监视器锁与调用该实例的任何{@link #lock}方法没有指定的关系。**建议不要**以这种方式使用{@code Lock} 实例以避免混淆，除非在它们自己的实现中。

```java
public interface Lock {
    // 阻塞方式获取锁
    void lock();
    
    // 阻塞可中断方式获取锁
    void lockInterruptibly() throws InterruptedException;
    
    // 尝试获取锁, 如果可用则获取锁并立即返回值{@code true}; 如果锁不可用, 则立即返回值{@code false}
    boolean tryLock();
    
    // 定时可中断式尝试获取锁, 获得则返回true, 否则返回false
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    
    // 释放锁
    void unlock();
    
    // 返回绑定到此{@code Lock}实例的{@link Condition}实例 => eg: 在Condition#await之前，当前线程必须持有锁
    Condition newCondition();
}
```

![1629257284109](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1629257284109.png)

##### 特点

- ReentrantLock，**{@link Lock}接口的可重入、互斥锁实现**，与使用{@code synchronized}方法和语句访问的隐式监视器锁具有相同的基本行为和语义，但具有扩展功能，由上次成功lock但尚未unlock的线程拥有：
  - 当锁不属于另一个线程时，调用 {@code lock} 的线程将返回并成功获取锁。
  - 如果当前线程已经拥有锁，该方法将立即返回。 
  - 可以使用方法 {@link #isHeldByCurrentThread} 和 {@link #getHoldCount} 进行检查。
- ReentrantLock的构造函数接受一个可选的公平参数，当设置{@code true}时，在争用情况下，锁倾向于授予对**等待时间最长的线程**的访问权限，否则这个锁不能保证这个特定的访问顺序。与使用默认设置的程序相比，使用由多个线程访问的公平锁的程序可能会显示出较低的总体吞吐量（即更慢，通常慢得多），但是能避免线程饥饿问题。
  - 请注意，**锁的公平性并不能保证线程调度的公平性**。因此，使用公平锁的许多线程之一可能会连续多次获得它，而其他活动线程没有进行并且当前没有持有该锁。
  - 另请注意，**未计时的 {@link #tryLock()} 方法不符合公平性设置**。 即使其他线程正在等待，如果锁可用，它也会成功。
- ReentrantLock，建议的做法是，在lock成功获取锁后需要try-finally或try-catch的保护，以确保在必要时unlock。
- ReentrantLock，最多支持同一线程2147483647（7FFF FFFF = 2^32 - 1）个递归锁，尝试超过此限制会导致 {@link Error} 从锁定方法中抛出。

##### 构造方法

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    
    private final Sync sync;// ReentrantLock同步执行对象

    // ReentrantLock的同步控制基类
    abstract static class Sync extends AbstractQueuedSynchronizer {
        ...
    }
    
    // 非公平锁同步控制实现类
    static final class NonfairSync extends Sync {
        ...
    }
    
    // 公平锁同步控制实现类
    static final class FairSync extends Sync {
        ...
    }
    
    // 创建ReentrantLock实例, 默认使用非公平锁
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    
    // 创建ReentrantLock实例, 如果为true则使用公平锁, 如果为false则使用非公平锁
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
}
```

##### Sync

###### lock

```java
// ReentrantLock的同步控制基类
abstract static class Sync extends AbstractQueuedSynchronizer {
    // 获取锁，子类可以实现该方法，以支持非公平版本
    abstract void lock();
    ...
}
```

###### nonfairTryAcquire

```java
// 非公平方式获取锁
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();

    // 获取同步器状态作为锁持有次数
    int c = getState();

    // 如果锁持有次数为0, 则CAS更新锁次数为acquires, 并设置当前线程独占, 返回true代表成功获得锁
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 如果当前线程为独占中的线程, 则锁持有次数累加acquires, 代表可重入, 返回true代表锁重入成功
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }

    // 如果尝试获取锁失败, 则返回false
    return false;
}
```

###### tryRelease

```java
// 尝试释放独占模式的锁持有次数releases
protected final boolean tryRelease(int releases) {
    // 持有次数减去releases次
    int c = getState() - releases;

    // 如果当前线程不为当前独占线程, 则抛出异常
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();

    // 如果c为0, 则清空独占的线程, 并返回true, 代表锁释放成功; 否则返回fasle, 代表锁释放失败
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

###### isHeldExclusively

```java
// 判断当前线程是否为独占的线程
protected final boolean isHeldExclusively() {
    return getExclusiveOwnerThread() == Thread.currentThread();
}
```

###### newCondition

```java
// 构建AQS实现的Condition接口实现类
final ConditionObject newCondition() {
    return new ConditionObject();
}
```

###### getOwner

```java
// 获取当前独占的线程
final Thread getOwner() {
    return getState() == 0 ? null : getExclusiveOwnerThread();
}
```

###### getHoldCount

```java
// 获取同步器的状态(即锁获取次数)
final int getHoldCount() {
    return isHeldExclusively() ? getState() : 0;
}
```

###### isLocked

```java
// 判断是否存在同步器状态, 即锁获取次数是否为0
final boolean isLocked() {
    return getState() != 0;
}
```

###### readObject

```java
// 反序列化可重入锁, 反序列化后同步状态(锁获取次数为0)
private void readObject(java.io.ObjectInputStream s) throws java.io.IOException, ClassNotFoundException {
    s.defaultReadObject();
    setState(0); // reset to unlocked state 重置为解锁状态
}
```

##### NonfairSync

```java
// 非公平锁同步控制实现类
static final class NonfairSync extends Sync {
    // 非公平获取锁, 先CAS更新锁持有次数为1
    final void lock() {
        // CAS成功则设置当前线程为独占线程
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        // CAS失败则以阻塞、独占模式获取同步器状态
        else
            acquire(1);
    }

    // 尝试以独占模式获取同步器状态
    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

##### FairSync

```java
// 公平锁同步控制实现类
static final class FairSync extends Sync {
    // 以阻塞、独占模式获取同步器状态, 公平排队
    final void lock() {
        acquire(1);
    }

    // 尝试以独占模式获取同步器状态, 对比非公平锁的tryAcquire, 多了需要先判断是否存在排队结点再进行CAS更新同步器状态
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();

        // 获取同步器状态作为锁持有次数
        int c = getState();

        // 如果锁持有次数为0, 则CAS更新锁次数为acquires, 并设置当前线程独占, 返回true代表成功获得锁
        if (c == 0) {
            // 判断此刻CLH中是否存在有等待时间比当前线程长的线程, 旨在由公平同步器使用, 以避免插入结点到CLH队列中
            if (!hasQueuedPredecessors() &&
                // 如果没有结点在排队, 则CAS更新同步器状态
                compareAndSetState(0, acquires)) {
                // CAS更新成功, 则设置当前线程为独占状态, 并返回true, 表示公平锁获取成功
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 如果当前线程为独占中的线程, 则锁持有次数累加acquires, 代表可重入, 返回true代表锁重入成功
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }

        // 如果尝试获取锁失败, 则返回false
        return false;
    }
}
```

##### 获取锁方法

###### lock

```java
// 阻塞方式获取锁
public void lock() {
    sync.lock();
}
```

###### lockInterruptibly

```java
// 以阻塞、独占可中断模式获取同步器
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}
```

###### tryLock

```java
// 非公平方式尝试获取锁, 如果可用则获取锁并立即返回值{@code true}; 如果锁不可用, 则立即返回值{@code false}
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}
```

###### tryLock(..，..)

```java
// 定时可中断式尝试获取锁, 获得则返回true, 否则返回false
public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
```

##### 释放锁方法

###### unlock

```java
// 释放锁
public void unlock() {
    sync.release(1);
}
```

##### 获取Condition方法

###### newCondition

```java
// 返回绑定到此{@code Lock}实例的{@link Condition}实例 => eg: 在Condition#await之前，当前线程必须持有锁
public Condition newCondition() {
    return sync.newCondition();
}
```

##### 其他监控方法

###### getHoldCount

```java
// 查询当前线程持有该锁的次数
public int getHoldCount() {
    return sync.getHoldCount();
}
```

###### isHeldByCurrentThread

```java
// 查询当前线程是否持有此锁
public boolean isHeldByCurrentThread() {
    return sync.isHeldExclusively();
}
```

###### isLocked

```java
// 查询此锁是否被任何线程持有
public boolean isLocked() {
    return sync.isLocked();
}
```

###### isFair

```java
// 判断此锁是否为公平锁
public final boolean isFair() {
    return sync instanceof FairSync;
}
```

###### getOwner

```java
// 返回当前拥有此锁的线程，如果不拥有，则返回 {@code null}
protected Thread getOwner() {
    return sync.getOwner();
}
```

###### hasQueuedThreads

```java
// 查询是否有线程正在等待获取此锁
public final boolean hasQueuedThreads() {
    return sync.hasQueuedThreads();
}
```

###### hasQueuedThread

```java
// 查询给定线程是否正在等待获取此锁
public final boolean hasQueuedThread(Thread thread) {
    // 判断目标线程是否在AQS主队列中: 从尾往前遍历, 如果Thread为同一个地址, 在返回true, 否则返回false
    return sync.isQueued(thread);
}
```

###### getQueueLength

```java
// 返回等待获取此锁的线程数的估计值
public final int getQueueLength() {
    // 从尾向前遍历统计非null的排队线程个数
    return sync.getQueueLength();
}
```

###### getQueuedThreads

```java
// 返回一个包含可能正在等待获取此锁的线程的集合
protected Collection<Thread> getQueuedThreads() {
    // 从尾向前遍历获取非null的排队线程列表
    return sync.getQueuedThreads();
}
```

###### hasWaiters

```java
// 查询指定Condition对象下是否有等待线程 => 从头遍历条件队列, 如果结点状态为CONDITION结点, 则返回true, 否则返回false
public boolean hasWaiters(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");

    // 查询是否有线程在此条件下等待, 从头遍历条件队列, 如果结点状态为CONDITION结点, 则返回true, 否则返回false
    return sync.hasWaiters((AbstractQueuedSynchronizer.ConditionObject)condition);
}
```

###### getWaitQueueLength

```java
// 返回等待与此锁关联的给定条件的线程数的估计值
public int getWaitQueueLength(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");

    // 返回等待此条件的线程数的估计值 => 从头遍历条件队列, 统计CONDITION结点的个数
    return sync.getWaitQueueLength((AbstractQueuedSynchronizer.ConditionObject)condition);
}
```

###### getWaitingThreads

```java
// 返回一个包含可能正在等待此Condition的线程的集合 => 从头遍历条件队列, 收集CONDITION结点的线程
protected Collection<Thread> getWaitingThreads(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");

    // 返回一个包含可能正在等待此Condition的线程的集合 => 从头遍历条件队列, 收集CONDITION结点的线程
    return sync.getWaitingThreads((AbstractQueuedSynchronizer.ConditionObject)condition);
}
```

#### ReentrantReadWriteLock

##### ReadWriteLock

- {@code ReadWriteLock}维护一对关联的{@link Lock 锁}，一个用于**只读操作**，一个用于**写入操作**。
  - {@link #readLock 读锁}可以由多个读线程同时持有，只要不存在写线程即可。
  - {@link #writeLock写锁}是线程独占的。
  - 所有 {@code ReadWriteLock} 实现，都必须保证 {@code writeLock} 操作的内存同步效果也适用于相关联的 {@code readLock}。也就是说，**成功获取读锁的线程一定能看到在之前释放写锁时所做的所有更新**。
- 与互斥锁相比，**读写锁允许更高的并发访问共享数据**。 
  - 它利用了这样一个事实，即虽然一次只有一个线程（写入线程）可以修改共享数据，但在许多情况下，任意数量的线程都可以同时读取数据（因此是读取线程）。
  - 理论上，与使用互斥锁相比，使用读写锁所允许的并发性增加将导致性能改进。
  - 在实践中，这种并发性的增加只有在**多处理器上**才能完全实现，而且只有在共享数据的访问模式合适的情况下才能实现。
- 与使用互斥锁相比，读写锁是否会提高性能取决于**同时读取或写入数据的线程数**，常表现为：
  - 读取数据与修改数据的频率。
  - 读写操作的持续时间。
  - 线程对数据的争用。

```java
public interface ReadWriteLock {
    // 返回用于读取的锁
    Lock readLock();
    
    // 返回用于写入的锁
    Lock writeLock();
}
```

![1629264487023](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1629264487023.png)

##### 特点

- ReentrantReadWriteLock，{@link ReadWriteLock} 的实现，支持与 {@link ReentrantLock} 类似的语义。
- ReentrantReadWriteLock具有一下属性：
  - **非公平模式（默认）**: 
    - 当构造为**非公平（参数为false）**时，读写锁的进入顺序是未指定的，受重入约束。
    - 持续竞争的非公平锁，可能会无限期推迟一个或多个读或写线程（线程饥饿），但通常比公平锁具有更高的吞吐量。
  - **公平模式**：
    - 当构造为**公平（参数为true）**时，线程使用近似到达顺序策略竞争进入。只有当前持有的锁被释放时，等待时间最长的写线程才会被分配写锁。或者如果有一组读线程等待的时间比所有等待写入器线程都长，则该组将被分配读取锁。
    - 如果写锁被持有，如果有一个试图获取**公平读锁（非可重入）**的线程将被阻塞，那么它将会直到当前等待的最老的写线程获得并释放写锁之后，该线程才会获得读锁。
    - 同理，如果一个等待的写线程放弃等待，留下一个或多个读线程作为队列中最长的等待者，并且写入锁空闲，那么这些读取器将被分配读取锁。
    - 除非读锁和写锁都是空闲的（这意味着没有等待线程），否则尝试获取公平写锁（非可重入）的线程将阻塞。
    - 请注意，非阻塞 {@link ReadLock#tryLock()} 和 {@link WriteLock#tryLock()} 方法不遵守此公平设置，如果可能将会立即获取锁，而不管等待线程。
  - **可重入性**：
    - ReentrantReadWriteLock，允许读和写以 {@link ReentrantLock} 接口的样式重新获取读或写锁。 
    - 但在写线程持有的所有写锁都被释放之前，不允许非可重入读锁。
    - 此外，写线程可以获取读锁，但反之则不行。基于这点，当调用或回调在读锁下，执行读取的方法期间持有写锁时，可重入可能很有用。
    - 而如果读线程试图获取写锁，它将永远不会成功。
  - **锁降级**：
    - 根据可重入性，ReentrantReadWriteLock允许从写锁降级到读锁，通过获取写锁，然后获取读锁，然后释放写锁，此时仍然持有读锁。
    - 但是，无法从读锁升级到写锁，即**只支持锁降级，不支持锁升级**。
  - **锁获取中断**：读锁和写锁，都允许线程在获取锁期间发生中断。
  - **Condition支持**：
    - 写锁提供了一个 {@link Condition} 实现，它的行为方式与写锁相同，就像 {@link ReentrantLock#newCondition} 为 {@link ReentrantLock} 提供的{@link Condition} 实现一样。因此， **该{@link Condition} 只能与写锁一起使用**。
    - **读锁不支持 {@link Condition}**， 并且 {@code readLock().newCondition()} 抛出 {@code UnsupportedOperationException}。
  - **仪表监控**：
    - ReentrantReadWriteLock，支持确定是持有锁还是争用锁的方法。
    - 这些方法设计用于监视系统状态，而不是用于同步控制。
- ReentrantReadWriteLock，可用于在某些类型的集合的某些用途中**提高并发性**。这通常只有在**预期集合很大**、**访问的读取线程比写入线程多**、**读取时需要的开销超过同步的开销**时才有意义。
- ReentrantReadWriteLock，最多支持 65535（2^16 - 1） 个递归写锁和 65535（2^16 - 1） 个读锁，尝试超过这些限制会导致 {@link Error} 从锁定方法中抛出。

##### 构造方法

```java
public class ReentrantReadWriteLock implements ReadWriteLock, java.io.Serializable {
    
    private final ReentrantReadWriteLock.ReadLock readerLock;// 读锁
    private final ReentrantReadWriteLock.WriteLock writerLock;// 写锁
    final Sync sync;// 同步执行对象

    public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
    public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }
    
    // 创建ReentrantReadWriteLock实例, 同时实例化读锁和写锁对象, 默认为非公平的同步执行对象
    public ReentrantReadWriteLock() {
        this(false);
    }
    
    // 创建ReentrantReadWriteLock实例, 同时实例化读锁和写锁对象, 指定为true时为公平的同步执行对象, 指定为false时为公平的同步执行对象
    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);
        writerLock = new WriteLock(this);
    }
}
```

##### Sync

```java
// ReentrantReadWriteLock的同步控制基类
abstract static class Sync extends AbstractQueuedSynchronizer {
    
    static final int SHARED_SHIFT   = 16;// 共享位偏移16位
    static final int SHARED_UNIT    = (1 << SHARED_SHIFT);// 共享位单位65536, 1单位的共享计数(高位, 读锁)
    static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;// 最大计数65535
    static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;// 独占掩码65535
    
    // 获取以计数表示的共享保留数: 取int高16位作为共享保留数(读锁)
    static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
    
    // 获取以计数表示的独占保留数: 取int低16位作为独占保留数(写锁)
    static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
    
    // 每个线程读取保留计数的计数器。作为ThreadLocal维护；缓存在cachedHoldCounter中
    static final class HoldCounter {
        int count = 0;
        final long tid = getThreadId(Thread.currentThread());// 返回给定线程的线程ID
    }
    static final class ThreadLocalHoldCounter extends ThreadLocal<HoldCounter> {
        public HoldCounter initialValue() {
            return new HoldCounter();
        }
    }
    
    // 当前线程持有的可重入读锁的数量, 每当线程的读取保留计数降至0时删除
    private transient ThreadLocalHoldCounter readHolds;
    
    // 成功获取readLock的最后一个线程的读取保留计数的计数器, 可以节省ThreadLocal查找, 非常适合线程缓存
    private transient HoldCounter cachedHoldCounter;
    
    // 第一个获得读锁的线程, 它是最后一次将共享计数从0更改为1的唯一线程, 此后再没有释放读锁
    private transient Thread firstReader = null;

    // firstReader的保留计数, 允许跟踪无竞争读锁的读保持非常便宜
    private transient int firstReaderHoldCount;
    
    // 构造同步锁
    Sync() {
        readHolds = new ThreadLocalHoldCounter();
        setState(getState()); // 确保 readHolds 的可见性
    }
}
```

###### readerShouldBlock

```java
// 如果获取读锁应该被阻塞, 则返回true
abstract boolean readerShouldBlock();
```

###### writerShouldBlock

```java
// 如果获取写锁应该被阻塞, 则返回true
abstract boolean writerShouldBlock();
```

###### tryRelease

```java
// 尝试释放独占模式的同步器状态, 返回是否存在独占状态
protected final boolean tryRelease(int releases) {
    // 判断当前线程是否为独占线程, 如果不是则抛出异常
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();

    // 扣减同步器状态(锁持有次数)
    int nextc = getState() - releases;

    // 获取以计数表示的独占保留数: 取int低16位作为独占保留数(写锁)
    boolean free = exclusiveCount(nextc) == 0;

    // 如果独占状态为0, 则清空独占线程
    if (free)
        setExclusiveOwnerThread(null);

    // 更新同步器状态(锁持有次数)
    setState(nextc);

    // 返回是否存在独占状态
    return free;
}
```

###### tryAcquire

```java
// 尝试以独占模式获取同步器状态
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    int c = getState();

    // 获取以计数表示的独占保留数w: 取int低16位作为独占保留数(写锁)
    int w = exclusiveCount(c);
    if (c != 0) {
        // 如果存在共享计数, 或者独享计数不为空且当前线程不为独占线程, 则返回false, 代表独占锁获取失败
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;

        // 如果独享计数大于最大值65536, 则抛出错误
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");

        // 更新同步器状态(锁持有次数), 代表可重入
        setState(c + acquires);

        // 返回true, 代表独占锁获取成功
        return true;
    }

    // c为0, 如果写锁应该阻塞 或者 CAS更新锁持有次数失败, 则返回false, 代表独占锁获取失败
    if (writerShouldBlock() || !compareAndSetState(c, c + acquires))
        return false;

    // CAS更新锁持有次数成功, 则设置当前线程为独占线程, 然后返回true代表独占锁获取成功
    setExclusiveOwnerThread(current);
    return true;
}
```

###### tryReleaseShared

```java
// 尝试释放共享模式的同步器状态, 读锁计数完全释放成功, 返回true, 否则返回false
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();

    // 如果当前线程为第一个读线程, 则释放共享计数
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    }
    // 如果当前线程不为第一个读线程, 从缓存中获取当前线程的共享计数, 再减1
    else {
        // 先根据线程id从缓存中获取共享计数
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;

        // 如果共享计数小于1, 说明共享计数已失效, 则移除缓存, 如果为负数则抛出异常
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }

    // 开始自旋
    for (;;) {
        int c = getState();

        // CAS更新锁持有次数 - 1单位的共享计数(高位, 读锁)
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // 如果减到0, 则说明读锁释放成功, 返回true, 否则返回false
            return nextc == 0;
    }
}
private IllegalMonitorStateException unmatchedUnlockException() {
    return new IllegalMonitorStateException(
        "attempt to unlock read lock, not locked by current thread");
}
```

###### tryAcquireShared

```java
// 尝试以共享模式获取同步器状态, 失败(<0), 获取成功但后续共享不成功(=0), 获取成功且后续共享成功(>0)
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();

    // 如果写锁被持有, 且独占线程不为当前线程, 则返回-1, 代表读锁获取失败
    if (exclusiveCount(c) != 0 && getExclusiveOwnerThread() != current)
        return -1;

    // 否则说明发生锁降级或者没有写锁线程, 则获取共享计数r
    int r = sharedCount(c);

    // 如果读锁不应该被阻塞, 且共享计数还没达到最大值, 则CAS更新锁持有次数+1单位
    if (!readerShouldBlock() && r < MAX_COUNT && compareAndSetState(c, c + SHARED_UNIT)) {
        // 如果共享计数r为0, 则设置第一个获取读锁线程以及读锁持有数量
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        }
        // 如果当前线程为第一个读锁线程, 则累加锁持有数量
        else if (firstReader == current) {
            firstReaderHoldCount++;
        }
        // 如果当前线程不为第一个读取锁线程, 则操作缓存
        else {
            HoldCounter rh = cachedHoldCounter;

            // 如果线程id持有者rh为空, 或者持有者id不为当前线程id, 则从获取中获取
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            // 如果存在rh, 但rh技术为0, 则更新计数以及缓存
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }

        // 最后返回1, 代表读锁获取成功
        return 1;
    }

    // 如果读锁应该被阻塞, 或者共享计数达到了最大值, 或者CAS更新锁持有次数+1单位失败, 则完整版获取读取, 自旋更新缓存计数
    return fullTryAcquireShared(current);
}

// 如果读锁应该被阻塞, 或者共享计数达到了最大值, 或者CAS更新锁持有次数+1单位失败, 则完整版获取读取, 自旋更新缓存计数
final int fullTryAcquireShared(Thread current) {
    HoldCounter rh = null;

    // 开始自旋
    for (;;) {
        int c = getState();

        // 如果存在写锁, 且当前线程不为独占线程, 则返回-1, 代表读锁获取失败
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
        }
        // 如果读锁应该阻塞, 判断当前线程是否为第一个获取读锁的线程, 如果是则什么也不做, 如果不是则更新缓存, 此时如果缓存计数为0, 则返回-1, 代表读锁获取失败
        else if (readerShouldBlock()) {
            // 如果当前线程为第一个获取读锁线程, 则什么也不做
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
            }
            // 如果当前线程不为第一个获取读锁线程, 则操作缓存
            else {
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current)) {
                        rh = readHolds.get();
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }

                // 如果缓存为0, 则返回-1, 代表读锁获取失败
                if (rh.count == 0)
                    return -1;
            }
        }

        // 如果共享计数等于最大值, 则抛出异常
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");

        // 如果判断重入性, 如果满足则CAS更新+1共享计数单位, 如果CAS失败则继续自旋
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            // 如果CAS成功, 再更新缓存计数
            if (sharedCount(c) == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```

###### tryWriteLock

```java
// 执行 tryLock 以进行写, 与 tryAcquire 的效果相同，只是缺少对 writerShouldBlock 的调用
final boolean tryWriteLock() {
    Thread current = Thread.currentThread();
    int c = getState();

    // 如果存在锁计数
    if (c != 0) {
        // 如果不存在独占, 或者独占线程不为当前线程, 则返回false, 代表写锁获取失败
        int w = exclusiveCount(c);
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
    }

    // 如果不存在共享计数, 则CAS更新1单位的写锁, 如果CAS失败, 则返回false, 代表写锁获取失败
    if (!compareAndSetState(c, c + 1))
        return false;

    // CAS成功, 则设置当前线程为独享线程, 并返回true, 代表写锁获取成功
    setExclusiveOwnerThread(current);
    return true;
}
```

###### tryReadLock

```java
// 为读取执行tryLock, 这与 tryAcquireShared 的效果相同，除了缺少对 readerShouldBlock 的调用
final boolean tryReadLock() {
    Thread current = Thread.currentThread();

    // 开始自旋
    for (;;) {
        int c = getState();

        // 如果存在写锁, 且独占线程不为当前线程, 则返回false, 代表读锁获取失败
        if (exclusiveCount(c) != 0 && getExclusiveOwnerThread() != current)
            return false;

        // 获取共享计数r, 如果达到最大了, 则抛出异常
        int r = sharedCount(c);
        if (r == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");

        // CAS更新1单位共享计数, 如果CAS成功, 则更新缓存计数, 最后返回true, 代表读锁获取成功
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (r == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    cachedHoldCounter = rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
            }
            return true;
        }
        // 如果CAS失败, 则继续自旋
    }
}
```

###### isHeldExclusively

```java
// 判断当前线程是否为独占线程
protected final boolean isHeldExclusively() {
    return getExclusiveOwnerThread() == Thread.currentThread();
}
```

###### newCondition

```java
// 构建读写锁的ConditionObject实例
final ConditionObject newCondition() {
    return new ConditionObject();
}
```

###### getOwner

```java
// 获取独占的线程实例
final Thread getOwner() {
    return ((exclusiveCount(getState()) == 0) ? null : getExclusiveOwnerThread());
}
```

###### getReadLockCount

```java
// 获取共享计数
final int getReadLockCount() {
    return sharedCount(getState());
}
```

###### isWriteLocked

```java
// 是否存在写锁
final boolean isWriteLocked() {
    return exclusiveCount(getState()) != 0;
}
```

###### getWriteHoldCount

```java
// 获取独占计数
final int getWriteHoldCount() {
    return isHeldExclusively() ? exclusiveCount(getState()) : 0;
}
```

###### getReadHoldCount

```java
// 获取缓存计数
final int getReadHoldCount() {
    if (getReadLockCount() == 0)
        return 0;

    Thread current = Thread.currentThread();
    if (firstReader == current)
        return firstReaderHoldCount;

    HoldCounter rh = cachedHoldCounter;
    if (rh != null && rh.tid == getThreadId(current))
        return rh.count;

    int count = readHolds.get().count;
    if (count == 0) readHolds.remove();
    return count;
}
```

###### getCount

```java
// 获取锁持有次数
final int getCount() { return getState(); }
```

###### readObject

```java
// 从流中重构实例（即反序列化它）
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    s.defaultReadObject();
    readHolds = new ThreadLocalHoldCounter();
    setState(0); // reset to unlocked state
}
```

##### NonfairSync

```java
// 非公平锁同步控制实现类
static final class NonfairSync extends Sync {
    // 非公平下判断写锁是否应该阻塞, 非公平下写锁总是可以互斥的, 所以返回false
    final boolean writerShouldBlock() {
        return false; // writers can always barge
    }

    // 非公平下判断读锁是否应该阻塞, 如果队头存在独占线程, 则返回false
    final boolean readerShouldBlock() {
        // 判断第一个排队线程是否以独占模式等待
        return apparentlyFirstQueuedIsExclusive();
    }
}
```

##### FairSync

```java
// 公平锁同步控制实现类
static final class FairSync extends Sync {
    // 公平下判断写锁是否应该阻塞, 看前面是否有排队线程
    final boolean writerShouldBlock() {
        // 判断此刻AQS主队列中是否存在有等待时间比当前线程长的线程, 旨在由公平同步器使用, 以避免插入结点到CLH队列中
        return hasQueuedPredecessors();
    }

    // 非公平下判断读锁是否应该阻塞, 看前面是否有排队线程
    final boolean readerShouldBlock() {
        // 判断此刻AQS主队列中是否存在有等待时间比当前线程长的线程, 旨在由公平同步器使用, 以避免插入结点到CLH队列中
        return hasQueuedPredecessors();
    }
}
```

##### ReadLock

```java
// 读锁实现类
public static class ReadLock implements Lock, java.io.Serializable {
    
    private final Sync sync;// ReentrantReadWriteLock的同步控制基类
    
    // 构造ReentrantReadWriteLock的读锁实例
    protected ReadLock(ReentrantReadWriteLock lock) {
        sync = lock.sync;
    }
}
```

###### lock

```java
// 共享方式获取读锁, 如果写锁未被另一个线程持有，则获取读锁并立即返回; 如果写锁被另一个线程持有, 则当前线程会阻塞
public void lock() {
    sync.acquireShared(1);
}
```

###### lockInterruptibly

```java
// 可中断共享方式获取读锁, 如果写锁未被另一个线程持有，则获取读锁并立即返回; 如果写锁被另一个线程持有, 则当前线程会阻塞
public void lockInterruptibly() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

###### tryLock

```java
// 非公平、共享方式尝试获取锁, 如果写锁未被另一个线程持有, 则获取读锁并立即返回值 {@code true}; 如果写锁被另一个线程持有，则此方法将立即返回值 {@code false}
public boolean tryLock() {
    return sync.tryReadLock();
}
```

###### tryLock（..，..）

```java
// 非公平、定时、可中断、共享方式尝试获取锁, 如果写锁未被另一个线程持有, 则获取读锁并立即返回值 {@code true}; 如果写锁被另一个线程持有，则此方法将立即返回值 {@code false}
public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```

###### unlock

```java
// 共享方式释放锁, 如果读锁同步器状态现在为0, 则会尝试共享方式释放读锁
public void unlock() {
    sync.releaseShared(1);
}
```

###### newCondition

```java
// 读锁不支持newCondition
public Condition newCondition() {
    throw new UnsupportedOperationException();
}
```

##### WriteLock

```java
// 写锁实现类
public static class WriteLock implements Lock, java.io.Serializable {
    
    private final Sync sync;// ReentrantReadWriteLock的同步控制基类
    
    // 构造ReentrantReadWriteLock的写锁实例
    protected WriteLock(ReentrantReadWriteLock lock) {
        sync = lock.sync;
    }
}
```

###### lock

```java
// 独占方式获取写锁, 如果读锁和写锁都没有被另一个线程持有, 则获取写锁并立即返回, 将写锁持有计数设置为1; 如果当前线程已经持有写锁, 则持有计数加一并且该方法立即返回(重入性); 如果该锁由另一个线程持有，则当前线程将阻塞, 直到获得写锁为止, 然后写锁持有计数设置为1
public void lock() {
    sync.acquire(1);
}
```

###### lockInterruptibly

```java
// 独占、可中断方式获取写锁, 如果读锁和写锁都没有被另一个线程持有, 则获取写锁并立即返回, 将写锁持有计数设置为1; 如果当前线程已经持有写锁, 则持有计数加一并且该方法立即返回(重入性); 如果该锁由另一个线程持有，则当前线程将阻塞, 直到获得写锁为止, 然后写锁持有计数设置为1
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}
```

###### tryLock

```java
// 非公平、独占方式获取写锁, 如果读锁和写锁都没有被另一个线程持有, 则获取写锁并立即返回, 将写锁持有计数设置为1; 如果当前线程已经持有写锁, 则持有计数加一并且该方法立即返回(重入性); 如果该锁由另一个线程持有，则当前线程将阻塞, 直到获得写锁为止, 然后写锁持有计数设置为1
public boolean tryLock( ) {
    return sync.tryWriteLock();
}
```

###### tryLock（..，..）

```java
// 非公平、定时、可中断、独占方式获取写锁, 如果读锁和写锁都没有被另一个线程持有, 则获取写锁并立即返回, 将写锁持有计数设置为1; 如果当前线程已经持有写锁, 则持有计数加一并且该方法立即返回(重入性); 如果该锁由另一个线程持有，则当前线程将阻塞, 直到获得写锁为止, 然后写锁持有计数设置为1
public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
```

###### unlock

```java
// 独占方式释放锁, 如果当前线程是此锁的持有者, 则持有计数递减; 如果保持计数现在为零, 则锁定被释放; 如果当前线程不是此锁的持有者, 则抛出 {@link IllegalMonitorStateException}
public void unlock() {
    sync.release(1);
}
```

###### newCondition

```java
// 返回与此 {@link Lock} 实例一起使用的 {@link Condition} 实例; 如果在调用任何{@link Condition}方法时未持有此写锁, 则会引发 {@link IllegalMonitorStateException};
public Condition newCondition() {
    return sync.newCondition();
}
```

###### isHeldByCurrentThread

```java
// 查询当前线程是否持有此写锁。 与 {@link ReentrantReadWriteLock#isWriteLockedByCurrentThread} 效果相同
public boolean isHeldByCurrentThread() {
    return sync.isHeldExclusively();
}
```

###### getHoldCount

```java
// 查询当前线程持有该写锁的次数
public int getHoldCount() {
    return sync.getWriteHoldCount();
}
```

##### 其他监控方法

###### isFair

```java
// 如果此锁的公平性设置为 true，则返回 {@code true}。
public final boolean isFair() {
    return sync instanceof FairSync;
}
```

###### getOwner

```java
// 返回当前拥有写锁的线程，如果不拥有，则返回 {@code null}。 当此方法由不是所有者的线程调用时，返回值反映了当前锁定状态的尽力而为的近似值。
protected Thread getOwner() {
    return sync.getOwner();
}
```

###### getReadLockCount

```java
// 查询此锁持有的读锁数。 该方法设计用于监视系统状态，而不是用于同步控制
public int getReadLockCount() {
    return sync.getReadLockCount();
}
```

###### isWriteLocked

```java
// 查询写锁是否被任何线程持有。 该方法设计用于监视系统状态，而不是用于同步控制
public boolean isWriteLocked() {
    return sync.isWriteLocked();
}
```

###### isWriteLockedByCurrentThread

```java
// 查询当前线程是否持有写锁
public boolean isWriteLockedByCurrentThread() {
    return sync.isHeldExclusively();
}
```

###### getWriteHoldCount

```java
// 查询当前线程对该锁的可重入写持有次数
public int getWriteHoldCount() {
    return sync.getWriteHoldCount();
}
```

###### getReadHoldCount

```java
// 查询当前线程对该锁持有的可重入读次数
public int getReadHoldCount() {
    return sync.getReadHoldCount();
}
```

###### getQueuedWriterThreads

```java
// 返回一个包含可能正在等待获取写锁的线程的集合
protected Collection<Thread> getQueuedWriterThreads() {
    return sync.getExclusiveQueuedThreads();
}
```

###### getQueuedReaderThreads

```java
// 返回一个包含可能正在等待获取读锁的线程的集合
protected Collection<Thread> getQueuedReaderThreads() {
    return sync.getSharedQueuedThreads();
}
```

###### hasQueuedThreads

```java
// 查询是否有线程正在等待获取读锁或写锁
public final boolean hasQueuedThreads() {
    return sync.hasQueuedThreads();
}
```

###### hasQueuedThread

```java
// 查询给定线程是否正在等待获取读锁或写锁
public final boolean hasQueuedThread(Thread thread) {
    return sync.isQueued(thread);
}
```

###### getQueueLength

```java
// 返回等待获取读锁或写锁的线程数的估计值
public final int getQueueLength() {
    return sync.getQueueLength();
}
```

###### getQueuedThreads

```java
// 返回一个包含可能正在等待获取读锁或写锁的线程的集合
protected Collection<Thread> getQueuedThreads() {
    return sync.getQueuedThreads();
}
```

###### hasWaiters

```java
// 查询是否有任何线程正在等待与写锁关联的给定条件
public boolean hasWaiters(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");
    return sync.hasWaiters((AbstractQueuedSynchronizer.ConditionObject)condition);
}
```

###### getWaitQueueLength

```java
// 返回等待与写锁关联的给定条件的线程数的估计值
public int getWaitQueueLength(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");
    return sync.getWaitQueueLength((AbstractQueuedSynchronizer.ConditionObject)condition);
}
```

###### getWaitingThreads

```java
// 返回一个集合，其中包含那些可能正在等待与写锁关联的给定条件的线程
protected Collection<Thread> getWaitingThreads(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");
    return sync.getWaitingThreads((AbstractQueuedSynchronizer.ConditionObject)condition);
}
```

#### CountDownLatch

##### 特点

- CountDownLatch，一种同步辅助，**它允许一个或多个线程等待，直到执行的一组操作完成才唤醒这些线程**。
- {@code CountDownLatch} 使用给定的计数进行初始化，{@link #await await} 方法阻塞，直到当前计数由于 {@link #countDown} 方法的调用而达到零，之后所有等待线程都被释放，并且 {@link #await await} 的任何后续调用立即返回。这是一种**一次性现象**（即计数无法重置），如果需要重置计数的版本，请考虑使用 {@link CyclicBarrier}。
- {@code CountDownLatch} 是一种**多功能同步工具**，可用于多种用途。 
  - 计数初始化为1的 {@code CountDownLatch} 用作简单的开关锁存器或门闩。所有调用 {@link #await await} 的线程在门处等待，直到它被调用 {@link # countDown}。
  - 计数初始化为 N 的 {@code CountDownLatch} 可用于使一个线程等待，直到 N 个线程完成某个操作，或者某个操作已完成 N 次。
- {@code CountDownLatch} 的一个有用属性是，它不需要调用 {@code countDown} 的线程在继续之前等待计数达到零，它**只是阻止任何线程通过 {@link #await await}** ，直到计数为0，在await阻塞的才可以通过。

##### 构造方法

```java
public class CountDownLatch {
    
    private final Sync sync;// 同步执行对象
    
    // 构造一个用给定计数初始化的{@code CountDownLatch}实例
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }
}
```

##### Sync

```java
// CountDownLatch的同步控制对象, 它使用AQS状态来表示计数
private static final class Sync extends AbstractQueuedSynchronizer {
    
    // 构造CountDownLatch的同步控制对象实例, 使用传入的计数作为同步器状态
    Sync(int count) {
        setState(count);
    }
    
    // 获取计数, 使用同步器状态作为计数
    int getCount() {
        return getState();
    }
    
    // 尝试以共享模式获取同步器状态, 失败(<0), 获取成功但后续共享不成功(=0), 获取成功且后续共享成功(>0)
    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }
    
    // 尝试释放共享模式的同步器状态
    protected boolean tryReleaseShared(int releases) {
        // 递减计数； 过渡到零时的信号
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
}
```

##### 阻塞方法

###### await

```java
// 以阻塞、共享、可中断模式获取同步器状态, 直到当前线程等待直到闩锁倒计时为零, 或者线程被中断
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

###### await(..，..)

```java
// 以阻塞、共享、定时、可中断模式获取同步器状态, 直到当前线程等待直到闩锁倒计时为零, 或者线程被中断, 或者指定的等待时间已过
public boolean await(long timeout, TimeUnit unit) throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```

##### 计数方法

###### countDown

```java
// 如果当前计数大于零, 则以共享模式递减锁存器的计数; 如果计数达到0, 则释放所有等待的线程; 如果当时计数为0, 则什么也不会发生
public void countDown() {
    sync.releaseShared(1);
}
```

##### 其他监控方法

###### getCount

```java
// 返回当前计数, 通常用于调试和测试目的
public long getCount() {
    return sync.getCount();
}
```

#### CyclicBarrier

##### 特点

- CyclicBarrier，一种同步辅助工具，它允许一组线程全部等待彼此到达公共屏障点，在涉及固定大小的线程组的程序中很有用。
- CyclicBarrier屏障之所以被称为Cyclic循环的，因为它可以在等待线程被释放后**重新使用**。
- {@code CyclicBarrier} 支持一个可选的 {@link Runnable} 命令，该命令在每个屏障点运行一次，在参与者中的最后一个线程到达之前，其他参与者线程都会被阻塞。此屏障操作对于在任何一方继续之前更新共享状态很有用，为方便起见，每次调用 {@link #await}（被唤醒后） 都会返回该线程在屏障处的**到达索引**。
- {@code CyclicBarrier}的每次使用都表示为一个生成Generation实例，每当障碍物被触发或重置时，Generation都会发生变化，因此可以有许多Generation与使用{@code CyclicBarrier}的线程相关联。
  - 由于锁以不确定的方式分配给等待线程，但一次只能激活Generation其中之一，因此其余的Generation要么坏了要么被绊倒了。
  - 另外，如果有中断但没有后续复位，则该Generation将损坏。

##### 构造方法

```java
public class CyclicBarrier {
        
    private final ReentrantLock lock = new ReentrantLock();// 栅栏入口锁
    private final Condition trip = lock.newCondition();// 线程绊倒Condition
    private final int parties;// 参与线程数
    private final Runnable barrierCommand;// 栅栏跳闸时运行的任务
    private Generation generation = new Generation();// 当前代实例
    private int count;// 触发栅栏跳闸时的线程数
    
    // 创建一个新的 {@code CyclicBarrier}，当给定数量的参与方（线程）正在等待它时，它将跳闸，并且当屏障跳闸时不会执行其他预定义的操作。
    public CyclicBarrier(int parties) {
        this(parties, null);
    }
    
    // 创建一个新的 {@code CyclicBarrier}，当给定数量的参与方（线程）正在等待它时，它将触发，并在屏障被触发时会由进入屏障的最后一个线程执行给定的屏障操作。
    public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }
}
```

##### Generation

```java
// CyclicBarrier分代实例
private static class Generation {
    // 栅栏是否已损坏
    boolean broken = false;
}
```

###### nextGeneration

```java
// 唤醒所有等待的线程, 重置栅栏跳闸所需线程数, 并生成新代实例
private void nextGeneration() {
    // 上一代信号完成
    trip.signalAll();

    // 建立下一代
    count = parties;
    generation = new Generation();
}
```

###### breakBarrier

```java
// 标记当前代为已损坏, 重置栅栏跳闸所需线程数, 并唤醒所有等待的线程
private void breakBarrier() {
    generation.broken = true;
    count = parties;
    trip.signalAll();
}
```

###### isBroken

```java
// 查询CyclicBarrier(当前代)是否处于损坏状态
public boolean isBroken() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return generation.broken;
    } finally {
        lock.unlock();
    }
}
```

###### reset

```java
// 重置CyclicBarrier为初始状态, 标记当前代为已损坏, 生成新代实例
public void reset() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 标记当前代为已损坏, 重置栅栏跳闸所需线程数, 并唤醒所有等待的线程
        breakBarrier();   // break the current generation 打破当前的一代

        // 唤醒所有等待的线程, 重置栅栏跳闸所需线程数, 并生成新代实例
        nextGeneration(); // start a new generation 开始新的一代
    } finally {
        lock.unlock();
    }
}
```

##### 核心源码

###### await

```java
// 无限阻塞当前线程, 所有线程到达、或者任何线程中断、超时、或者栅栏被重置则被唤醒
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
```

###### await(..，..)

```java
// 阻塞当前线程指定时间, 所有线程到达、或者任何线程中断、超时、或者栅栏被重置则被唤醒
public int await(long timeout, TimeUnit unit) throws InterruptedException, BrokenBarrierException, TimeoutException {
    return dowait(true, unit.toNanos(timeout));
}
```

###### dowait

```java
// CyclicBarrier阻塞核心代码
private int dowait(boolean timed, long nanos) throws InterruptedException, BrokenBarrierException, TimeoutException {
    // 先获取栅栏入口锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 加锁后获取分代Generation
        final Generation g = generation;

        // 如果栅栏已损坏, 则抛出异常
        if (g.broken)
            throw new BrokenBarrierException();

        // 如果线程已中断, 则标记当前代为已损坏, 重置栅栏跳闸所需线程数, 并唤醒所有等待的线程
        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }

        // 如果栅栏没有损坏, 且线程也没有被中断, 则栅栏线程计数-1
        int index = --count;

        // 如果栅栏线程计数减到0了, 则触发栅栏回调
        if (index == 0) {  // tripped // 绊倒
            boolean ranAction = false;
            try {
                // 运行栅栏回调任务, 并生成新代Generation
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                ranAction = true;

                // 唤醒所有等待的线程, 重置栅栏跳闸所需线程数, 并生成新代实例
                nextGeneration();
                return 0;
            } finally {
                // 如果运行失败, 则将当前屏障生成设置为已破坏并唤醒所有人
                if (!ranAction)
                    breakBarrier();
            }
        }

        // 循环直到跳闸、损坏、中断或超时
        // loop until tripped, broken, interrupted, or timed out
        // 如果还没达到栅栏处, 则开始自旋
        for (;;) {
            try {
                // 如果不需要超时, 则调用Condition#await阻塞当前线程
                if (!timed)
                    trip.await();
                // 如果需要超时, 且超时时间大于0, 则调用Condition#awaitNanos阻塞当前线程nanos时间
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                // 如果发生异常, 则标记当前代为已损坏, 重置栅栏跳闸所需线程数, 并唤醒所有等待的线程
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    // 即使我们没有被中断，我们也即将完成等待，因此该中断被视为“属于”后续执行。
                    Thread.currentThread().interrupt();
                }
            }

            // 如果当前线程被唤醒后代被损坏, 则抛出异常
            if (g.broken)
                throw new BrokenBarrierException();

            // 如果当前线程被唤醒后不为同一代, 说明上一代到达了栅栏, 则返回到达栅栏的索引号(为0代表最后一个线程)
            if (g != generation)
                return index;

            // 如果当前线程被唤醒后仍为同一代, 说明上一代仍然没到达栅栏, 发生了虚假唤醒
            // 如果需要超时, 且发生了超时, 则将当前屏障生成设置为已破坏并唤醒所有人, 并抛出超时异常
            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
            // 如果不需要超时或者仍然没超时, 则继续自旋+阻塞
        }
    } finally {
        // 任何异常和返回都会先释放入口锁
        lock.unlock();
    }
}
```

##### 其他监控方法

###### getParties

```java
// 返回触发CyclicBarrier所需的参与线程数量
public int getParties() {
    return parties;
}
```

###### getNumberWaiting

```java
// 返回当前处于等待的线程数 = parties - count
public int getNumberWaiting() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return parties - count;
    } finally {
        lock.unlock();
    }
}
```

#### Semaphore

##### 特点

- Semaphore，计数信号量，从概念上讲，**信号量维护一组许可**。
  - 每个{@link#acquire}会直到许可证是可用的，然后获取。
  - 每个{@link#RELEASE}添加一个许可，释放阻塞的获取线程。
  - 实际上{@code Semaphore}有维护许可对象，它只是让计数可用，从而采取相应的行动。
- 信号量通常用于限制可以访问某些（物理或逻辑）资源的线程数。 
- Semaphore，可以初始化为 1 的信号量，并且使用时最多只有一个许可可用，用作互斥锁。 这通常被称为**二元信号量**，因为它只有两种状态：一个许可可用，或零个许可可用。以这种方式使用时，二进制信号量具有属性（与许多 {@link java.util.concurrent.locks.Lock} 实现不同），即“锁”可以由所有者以外的线程释放（因为**信号量具有没有所有权的概念**），这在某些特定上下文中很有用，例如**死锁恢复**。
- Semaphore的构造函数可以选择接受公平参数：
  - 当公平性设置为 false 时，为非公平模式，则不保证线程获取许可的顺序。特别是，允许插入，即调用 {@link #acquire} 的线程可以在一直等待的线程之前，分配一个许可（逻辑上，新线程将自己置于等待线程队列的头部）。
  - 当公平性设置为true时，为公平模式，信号量保证调用任何 {@link #acquire()acquire} 方法的线程被选择以按照它们对这些方法的调用的处理顺序（先进先出）获得许可。
  - 请注意，未计时的 {@link #tryAcquire() tryAcquire} 方法**不遵守公平设置**，但会采用任何可用的许可。
  - 通常，**用于控制资源访问的信号量应初始化为公平的**，以确保没有线程因访问资源而饿死。 当使用信号量进行其他类型的同步控制时，**非公平排序的吞吐量大于公平性排序的吞吐量**。

##### 构造方法

```java
public class Semaphore implements java.io.Serializable {
    
    private final Sync sync;// 同步执行对象
    
    // 使用给定数量的许可设置创建一个非公平版本的{@code Semaphore}
    public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }
    
    // 使用给定数量的许可设置创建一个{@code Semaphore}, 为true时为公平模式, 为false时为非公平模式
    public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }
}
```

##### Sync

```java
// Semaphore同步控制对象
abstract static class Sync extends AbstractQueuedSynchronizer {
    
    // 构造Semaphore同步控制对象, 设置许可数量, 使用同步器状态作为许可数量
    Sync(int permits) {
        setState(permits);
    }
}
```

###### getPermits

```java
// 获取当前许可数量
final int getPermits() {
    return getState();
}
```

###### nonfairTryAcquireShared

```java
// 非公平方式获取同步器状态
final int nonfairTryAcquireShared(int acquires) {
    // 开始自旋
    for (;;) {
        // 获取最新的同步器状态
        int available = getState();

        // 扣减要获取的同步器状态
        int remaining = available - acquires;

        // 如果同步器状态不够, 或者CAS成功扣减时, 则返回剩余计数
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            // 如果返回负数, 代表数量不够, 获取失败; 如果返回非负数, 代表数量足够, 获取成功
            return remaining;

        // 如果同步器够但CAS扣减失败, 则继续自旋
    }
}
```

###### tryReleaseShared

```java
// 尝试释放共享模式的同步器状态
protected final boolean tryReleaseShared(int releases) {
    // 开始自旋
    for (;;) {
        // 获取最新的同步器状态
        int current = getState();

        // 加上需要获取的同步器状态
        int next = current + releases;

        // 如果相加失败, 则抛出异常
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");

        // 如果相加成功, 则CAS更新同步器状态, 返回true, 代表释放成功
        if (compareAndSetState(current, next))
            return true;

        // 如果CAS更新失败, 则继续自旋
    }
}
```

###### reducePermits

```java
// 减少信号量许可证
final void reducePermits(int reductions) {
    // 开始自旋
    for (;;) {
        // 获取最新的同步器状态
        int current = getState();

        // 减去需要获取的同步器状态
        int next = current - reductions;

        // 如果扣减失败, 则抛出异常
        if (next > current) // underflow
            throw new Error("Permit count underflow");

        // 如果扣减成功, 则CAS更新同步器状态
        if (compareAndSetState(current, next))
            return;

        // 如果CAS更新成功, 则继续自旋
    }
}
```

###### drainPermits

```java
// 清空信号量许可证
final int drainPermits() {
    // 开始自旋
    for (;;) {
        // 获取最新的同步器状态
        int current = getState();

        // 如果同步器状态为0, 或者CAS更新为0成功, 则返回current(可能会为0, 可能不为0)
        if (current == 0 || compareAndSetState(current, 0))
            return current;

        // 如果CAS更新失败, 则继续自旋
    }
}
```

##### NonfairSync

```java
// 非公平版同步控制实现类
static final class NonfairSync extends Sync {
    // 构造非公平版同步控制实现类对象, 设置许可数量, 使用同步器状态作为许可数量
    NonfairSync(int permits) {
        super(permits);
    }
}
```

###### tryAcquireShared

```java
// 非公平获取同步器状态
protected int tryAcquireShared(int acquires) {
    return nonfairTryAcquireShared(acquires);
}
```

##### FairSync

```java
// 公平版同步控制实现类
static final class FairSync extends Sync {
    // 构造公平版同步控制实现类对象, 设置许可数量, 使用同步器状态作为许可数量
    FairSync(int permits) {
        super(permits);
    }
}
```

###### tryAcquireShared

```java
// 尝试以共享模式获取同步器状态, 失败(<0), 获取成功但后续共享不成功(=0), 获取成功且后续共享成功(>0)
protected int tryAcquireShared(int acquires) {
    // 开始自旋
    for (;;) {
        // 判断此刻CLH中是否存在有等待时间比当前线程长的线程, 旨在由公平同步器使用, 以避免插入结点到CLH队列中
        if (hasQueuedPredecessors())
            // 如果存在排队线程, 则返回-1, 代表同步器状态获取失败
            return -1;

        // 获取最新的同步器状态
        int available = getState();

        // 扣减需要获取的同步器状态
        int remaining = available - acquires;

        // 如果同步器状态不够, 或者CAS成功扣减时, 则返回剩余计数
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;

        // 如果同步器够但CAS扣减失败, 则继续自旋
    }
}
```

##### 获取信号量方法

###### acquire

```java
// 以阻塞、共享可中断模式获取同步器状态
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

###### acquireUninterruptibly

```java
// 以阻塞、不可中断、共享模式获取同步器状态
public void acquireUninterruptibly() {
    sync.acquireShared(1);
}
```

###### tryAcquire

```java
// 非公平快速失败方式获取同步器状态
public boolean tryAcquire() {
    return sync.nonfairTryAcquireShared(1) >= 0;
}
```

###### tryAcquire(..，..)

```java
// 以阻塞、共享、定时、可中断模式获取同步器状态
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```

###### acquir(int)

```java
// 以阻塞、共享可中断模式获取同步器状态
public void acquire(int permits) throws InterruptedException {
    if (permits < 0) throw new IllegalArgumentException();
    sync.acquireSharedInterruptibly(permits);
}
```

###### acquireUninterruptibly(int)

```java
// 以阻塞、不可中断、共享模式获取同步器状态
public void acquireUninterruptibly(int permits) {
    if (permits < 0) throw new IllegalArgumentException();
    sync.acquireShared(permits);
}
```

###### tryAcquire(int)

```java
// 非公平快速失败方式获取同步器状态
public boolean tryAcquire(int permits) {
    if (permits < 0) throw new IllegalArgumentException();
    return sync.nonfairTryAcquireShared(permits) >= 0;
}
```

###### tryAcquire(int, .., ..)

```java
// 以阻塞、共享、定时、可中断模式获取同步器状态
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException {
    if (permits < 0) throw new IllegalArgumentException();
    return sync.tryAcquireSharedNanos(permits, unit.toNanos(timeout));
}
```

##### 释放信号量方法

###### release

```java
// 尝试释放共享模式的同步器状态, 如果释放成功, 则返回true; 如果释放失败, 则返回false
public void release() {
    sync.releaseShared(1);
}
```

###### release(int)

```java
// 释放共享模式的同步器状态, 如果释放成功, 则返回true; 如果释放失败, 则返回false
public void release(int permits) {
    if (permits < 0) throw new IllegalArgumentException();
    sync.releaseShared(permits);
}
```

##### 其他监控方法

###### availablePermits

```java
// 返回此信号量中可用的当前许可数
public int availablePermits() {
    return sync.getPermits();
}
```

###### drainPermits

```java
// 获取并返回所有立即可用的许可证
public int drainPermits() {
    return sync.drainPermits();
}
```

###### reducePermits

```java
// 按指示的减少量减少可用许可证的数量, 此方法在使用信号量跟踪变得不可用的资源的子类中很有用
protected void reducePermits(int reduction) {
    if (reduction < 0) throw new IllegalArgumentException();
    sync.reducePermits(reduction);
}
```

###### isFair

```java
// 判断信号量是否为公平版本, 如果此信号量的公平性设置为 true，则返回 {@code true}
public boolean isFair() {
    return sync instanceof FairSync;
}
```

###### hasQueuedThreads

```java
// 查询是否有线程在等待获取
public final boolean hasQueuedThreads() {
    return sync.hasQueuedThreads();
}
```

###### getQueueLength

```java
// 返回等待获取的线程数的估计值
public final int getQueueLength() {
    return sync.getQueueLength();
}
```

###### getQueuedThreads

```java
// 返回一个包含可能正在等待获取的线程的集合
protected Collection<Thread> getQueuedThreads() {
    return sync.getQueuedThreads();
}
```

