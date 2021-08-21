源码篇

JUC，是**java.util.concurrent**⼯具包的简称，JDK 1.5开始出现，可以用于处理线程并发，包含atomic包、locks包和直接包下的其他工具类。

![1627786850701](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1627786850701.png)

- **并发基础**：Thread、Unsafe、Object、LockSupport、AQS、FutureTask、ThreadLocal。
- **典型实现**：ReentrantLock、ReentrantReadWriteLock、CountDownLatch、CyclicBarrier、Semaphore、AtomicInteger、ThreadPoolExecutor、ScheduledThreadPoolExecutor、Executors。

### 并发基础

#### FutureTask

##### Future

- {@code Future}，表示异步计算的结果，提供了**检查计算是否完成**、**等待计算结果完成**以及**检索计算结果**的方法。
  - 结果只能在计算完成后使用方法{@code get} 检索，必要时会阻塞直到它准备好。
  - 取消则由{@code cancel}方法执行，还提供了其他方法来确定任务是正常完成还是被取消，一旦计算完成，则不能取消计算。如果为了可取消性，而想使用{@code Future}但不提供可用结果，则可以声明{@code Future}形式的类型，并返回{@code null}作为底层任务的结果。
- {@link FutureTask}，是{@code Runnable}的{@code Future}的实现，因此可以由{@code Executor} 执行。

```java
public interface Future<V> {
    // 尝试取消此异步计算任务的执行, 如果任务已完成、已被取消或由于其他原因无法取消, 则返回false; 如果此任务在正常完成之前被取消, 则返回{@code true}; 如果任务已经开始, 那么{@code mayInterruptIfRunning}参数决定是否应该中断执行该任务的线程以尝试停止该任务
    boolean cancel(boolean mayInterruptIfRunning);
    
    // 判断此异步计算任务是否取消成功, 如果任务已完成、已被取消或由于其他原因无法取消, 则返回false; 如果在正常完成之前被取消，则返回{@code true}
    boolean isCancelled();
    
    // 判断此异步计算任务是否已经完成, 在正常终止、异常或取消时, 则返回true
    boolean isDone();
    
    // 阻塞获取异步计算结果
    V get() throws InterruptedException, ExecutionException;
    
    // 阻塞等待给定时间来获取异步计算结果
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```

##### RunnableFuture

- RunnableFuture，即是{@link Future}，也是{@link Runnable}，**{@code run}方法的成功执行，会导致{@code Future}的完成并允许访问其结果**。

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    // 将Future设置为其计算结果, 除非它已被取消
    void run();
}
```

##### Callable

- Callable，**一个允许返回结果并可能引发异常的任务**，实现者定义了一个没有参数的单一方法，称为 {@code call}。
- {@code Callable} 接口类似于{@link java.lang.Runnable}，因为两者都是为实例可能由另一个线程执行的类而设计的。但是，**{@code Runnable} 不会返回结果，也不会抛出已检查的异常**。
- {@link Executors} 类包含将其他常见形式转换为 {@code Callable} 类的实用方法。

```java
public interface Callable<V> {
    // 计算结果，如果无法计算则抛出异常
    V call() throws Exception;
}
```



![1629422568660](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1629422568660.png)

##### 特点

- FutureTask，可取消的异步计算任务，提供了{@link Future}的基本实现，具有启动和取消计算、查看计算是否完成、检索计算结果的方法。
- 计算完成后才能检索结果，如果计算尚未完成，{@code get}方法将阻塞。一旦计算完成，就不能重新开始或取消计算。
- {@code FutureTask}，可用于包装{@link Callable}或{@link Runnable}对象，因为{@code FutureTask}实现了{@code Runnable}，一个 {@code FutureTask}可以提交给一个{@link Executor}执行。
- 除了用作独立类之外，{@code FutureTask}还提供了{@code protected}功能，这在创建自定义任务类时可能很有用。

##### 构造方法

```java
public class FutureTask<V> implements RunnableFuture<V> {
    
    // 任务的运行状态，最初是NEW。运行状态仅在set、setException和cancel方法中转换为终止状态。在完成期间，状态可能采用COMPLETING（在设置结果时）或INTERRUPTING（仅在中断运行程序以满足取消（真）时）的瞬态值。 从这些中间状态到最终状态的转换使用更便宜的有序/惰性写入，因为值是唯一的并且无法进一步修改。
    private volatile int state;
    private static final int NEW          = 0;// 新增
    private static final int COMPLETING   = 1;// 正在完成
    private static final int NORMAL       = 2;// 已完成
    private static final int EXCEPTIONAL  = 3;// 发生异常
    private static final int CANCELLED    = 4;// 已取消
    private static final int INTERRUPTING = 5;// 发生中断
    private static final int INTERRUPTED  = 6;// 已中断
    // 可能的状态转换:
    // NEW -> COMPLETING -> NORMAL: 新增 -> 正在完成 -> 已完成
    // NEW -> COMPLETING -> EXCEPTIONAL: 新增 -> 正在完成 -> 发生异常
    // NEW -> CANCELLED: 新增 -> 已取消
    // NEW -> INTERRUPTING -> INTERRUPTED: 新增 -> 发生中断 -> 已中断
    
    private Callable<V> callable;// 底层可调用；运行后清零
    private Object outcome;// 从get()返回的结果或抛出的异常, 非volatile, 需要有状态读/写保护
    private volatile Thread runner;// 运行后可调用的线程, 在run()时被CAS为当前线程
    private volatile WaitNode waiters;// 等待线程WaitNode结点的链表
    
    // 包装可运行任务与成功返回的结果, 成为可调度任务, 然后设置将要运行的可调用任务, 并更新任务的运行状态为创建状态
    public FutureTask(Runnable runnable, V result) {
        // 包装可运行任务与成功返回的结果, 成为可调度任务, 然后设置将要运行的可调用任务, 并更新任务的运行状态为创建状态
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;       // ensure visibility of callable
    }
    
    // 设置将要运行的可调用任务, 并更新任务的运行状态为创建状态
    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();

        // 设置将要运行的可调用任务, 并更新任务的运行状态为创建状态
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable 确保可调用的可见性
    }
}
```

##### 运行方法

###### run

```java
// 本质上是调用Callable#call()运行目标任务, 并将计算结果设置到outcome中(包括正确的结果或者异常), 最后遍历等待线程堆栈结点, 并清空唤醒每个结点的线程, 并在完成前调用done方法触发子类的回调, 以及清空运行的任务
public void run() {
    // 如果任务为新增状态, 则CAS更新运行的线程为当前线程
    if (state != NEW || !UNSAFE.compareAndSwapObject(this, runnerOffset, null, Thread.currentThread()))
        // 如果CAS更新线程失败, 或者本来就是非新增状态, 则直接返回, 不用运行任务了
        return;
    
    // 如果CAS更新成功, 且任务为新增状态, 则获取要运行的目标任务, 调用call方法计算结果
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;

                // 如果计算时发生异常, 则为异步计算结果设置异常结果, 并更新任务状态为已发生异常状态, 最后遍历等待线程堆栈结点, 并清空唤醒每个结点的线程, 并在完成前调用done方法触发子类的回调, 以及清空运行的任务
                setException(ex);
            }
            // 如果计算成功, 则为异步计算结果设置value值, 并更新任务状态为已完成状态, 最后遍历等待线程堆栈结点, 并清空唤醒每个结点的线程, 并在完成前调用done方法触发子类的回调, 以及清空运行的任务
            if (ran)
                set(result);
        }
    } finally {
        // volatile方式清空运行线程, 以防止并发调用run()
        runner = null;

        // volatile方式获取任务状态, 如果为发生中断或者已中断, 则一直让步CPU保持等待, 直到为中断状态
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

###### runAndReset

```java
// 本质上是调用Callable#call()运行目标任务, 不将计算结果设置到outcome中, 但会设置异常结果, 返回true代表本次运行成功且仍然可以继续运行; 返回false代表本次运行失败且不可以继续运行
protected boolean runAndReset() {
    // 如果任务为新增状态, 则CAS更新运行的线程为当前线程
    if (state != NEW || !UNSAFE.compareAndSwapObject(this, runnerOffset, null, Thread.currentThread()))
        // 如果CAS更新线程失败, 或者本来就是非新增状态, 则直接返回false, 代表本次运行失败且不可以继续运行
        return false;

    // 如果CAS更新成功, 且任务为新增状态, 则获取要运行的目标任务, 调用call方法计算结果
    boolean ran = false;
    int s = state;
    try {
        Callable<V> c = callable;
        if (c != null && s == NEW) {
            try {
                c.call(); // don't set result
                ran = true;
            } catch (Throwable ex) {
                // 如果计算时发生异常, 则为异步计算结果设置异常结果, 并更新任务状态为已发生异常状态, 最后遍历等待线程堆栈结点, 并清空唤醒每个结点的线程, 并在完成前调用done方法触发子类的回调, 以及清空运行的任务
                setException(ex);
            }
        }
    } finally {
        // volatile方式清空运行线程, 以防止并发调用run()
        runner = null;
        
        // volatile方式获取任务状态, 如果为发生中断或者已中断, 则一直让步CPU保持等待, 直到为中断状态
        s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }

    // 如果运行完毕, 且任务状态仍为新增状态, 则返回true, 代表本次运行成功且仍然可以继续运行
    return ran && s == NEW;
}
```

###### handlePossibleCancellationInterrupt

```java
// 如果为正在中断状态, 则一直让步CPU保持等待, 直到为中断状态
private void handlePossibleCancellationInterrupt(int s) {
    // 如果为正在中断状态, 则一直让步CPU保持等待, 直到为中断状态
    if (s == INTERRUPTING)
        while (state == INTERRUPTING)
            Thread.yield(); // wait out pending interrupt // 等待挂起的中断
}
```

###### cancel

```java
// 尝试取消此异步计算任务的执行, 如果任务已完成、已被取消或由于其他原因无法取消, 则返回false; 如果此任务在正常完成之前被取消, 则返回{@code true}
public boolean cancel(boolean mayInterruptIfRunning) {
    // 如果此时为新增状态, 且应该中断, 则CAS更新状态为发生中断; 如果不应该中断, 则CAS更新状态为已取消
    if (!(state == NEW && UNSAFE.compareAndSwapInt(this, stateOffset, NEW, mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
        // CAS成功则返回false, 代表由于发生中断导致的取消
        return false;

    // 如果CAS更新失败, 且应该中断, 则中断该线程, 如果从Thread其他实例方法调用该方法, 则会清除中断状态, 然后会收到一个{@link InterruptedException}
    try {    // in case call to interrupt throws exception // 如果调用中断引发异常
        if (mayInterruptIfRunning) {
            try {
                Thread t = runner;
                if (t != null)
                    t.interrupt();
            } finally { // final state
                // 标记状态为已中断
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
        // 遍历线程堆栈结点, 并清空唤醒每个结点的线程, 最后完成前调用done方法触发子类的回调, 以及清空运行的任务
        finishCompletion();
    }

    // 如果正常取消则返回true
    return true;
}
```

###### finishCompletion

```java
// 遍历线程堆栈结点, 并清空唤醒每个结点的线程, 最后完成前调用done方法触发子类的回调, 以及清空运行的任务
private void finishCompletion() {
    // assert state > COMPLETING;
    // 遍历线程结点的堆栈
    for (WaitNode q; (q = waiters) != null;) {
        // CAS置null线程结点
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
            // 如果CAS成功, 说明当前线程抢到了清空操作的机会, 则开始自旋
            for (;;) {
                // volatile方式获取堆栈结点线程t
                Thread t = q.thread;

                // 置null该结点thread引用, 并唤醒该线程t
                if (t != null) {
                    q.thread = null;
                    LockSupport.unpark(t);
                }

                // 如果线程t为null, 或者唤醒t完毕, 则volatile方式获取下一个结点next
                WaitNode next = q.next;

                // 如果遍历到链尾, 则退出自旋
                if (next == null)
                    break;

                // 置null原next引用, 并继续遍历
                q.next = null; // unlink to help gc
                q = next;
            }
            break;
        }
        // 如果CAS失败, 说明当前没抢到清空操作的机会, 则重新CAS
    }

    // 正常结束或者已取消完毕时调用, 默认空实现, 子类可以覆盖此方法实现回调
    done();

    // 置null运行的任务
    callable = null;        // to reduce footprint 减少足迹
}
```

###### done

```java
// 正常结束或者已取消完毕时调用, 默认空实现, 子类可以覆盖此方法实现回调
protected void done() { }
```

##### 设置结果方法

###### set

```java
// 为异步计算结果设置value值, 并更新任务状态为已完成状态, 最后遍历等待线程堆栈结点, 并清空唤醒每个结点的线程, 并在完成前调用done方法触发子类的回调, 以及清空运行的任务
protected void set(V v) {
    // 先CAS更新任务状态为正在完成状态
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        // 为异步计算结果设置value值
        outcome = v;

        // 顺序更新任务状态为已完成状态
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state

        // 遍历线程堆栈结点, 并清空唤醒每个结点的线程, 最后完成前调用done方法触发子类的回调, 以及清空运行的任务
        finishCompletion();
    }
}
```

###### setException

```java
// 为异步计算结果设置异常结果, 并更新任务状态为已发生异常状态, 最后遍历等待线程堆栈结点, 并清空唤醒每个结点的线程, 并在完成前调用done方法触发子类的回调, 以及清空运行的任务
protected void setException(Throwable t) {
    // 先CAS更新任务状态为正在完成状态
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        // 为异步计算结果设置异常结果
        outcome = t;

        // 顺序更新任务状态为已发生异常状态
        UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL); // final state

        // 遍历线程堆栈结点, 并清空唤醒每个结点的线程, 最后完成前调用done方法触发子类的回调, 以及清空运行的任务
        finishCompletion();
    }
}
```

##### 获取结果方法

###### get

```java
// 阻塞获取异步计算结果, 如果任务处于未完成状态, 则会自旋+阻塞/中断/超时, 以等待任务已经完成、已取消、已发生异常或者已中断, 则清空等待结点, 并返回任务完成时的状态, 最后再为已完成的任务返回结果或者抛出异常
public V get() throws InterruptedException, ExecutionException {
    // volatile方式获取异步任务的状态
    int s = state;

    // 如果任务为新增或者正在完成的状态
    if (s <= COMPLETING)
        // 自旋+阻塞/中断/超时, 以等待任务已经完成、已取消、已发生异常或者已中断, 则清空等待结点, 并返回任务完成时的状态
        s = awaitDone(false, 0L);

    // 如果阻塞等待到任务完成后, 则为已完成的任务返回结果或者抛出异常
    return report(s);
}
```

###### get(..，..)

```java
// 定时阻塞获取异步计算结果, 如果任务处于未完成状态, 则会自旋+阻塞/中断/超时, 以等待任务已经完成、已取消、已发生异常或者已中断, 则清空等待结点, 并返回任务完成时的状态, 最后再为已完成的任务返回结果或者抛出异常
public V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException {
    if (unit == null)
        throw new NullPointerException();

    // volatile方式获取异步任务的状态
    int s = state;

    // 如果任务为新增或者正在完成的状态
    if (s <= COMPLETING &&
        // 自旋+阻塞/中断/超时, 以等待任务已经完成、已取消、已发生异常或者已中断, 则清空等待结点, 并返回任务完成时的状态
        (s = awaitDone(true, unit.toNanos(timeout))) <= COMPLETING)
        // 如果返回的是未完成的状态, 说明发生了超时, 则抛出TimeoutException
        throw new TimeoutException();

    // 如果阻塞等待到任务完成后, 则为已完成的任务返回结果或者抛出异常
    return report(s);
}
```

###### report

```java
// 为已完成的任务返回结果或者抛出异常
@SuppressWarnings("unchecked")
private V report(int s) throws ExecutionException {
    // 获取当前从get()返回的结果或抛出的异常x
    Object x = outcome;

    // 如果此异步计算任务的运行状态为正常状态, 说明结果已经计算好了, 则返回x
    if (s == NORMAL)
        return (V)x;

    // 如果此异步计算任务的运行状态为已取消、发生中断或者中断状态, 则抛出CancellationException异常
    if (s >= CANCELLED)
        throw new CancellationException();

    // 如果此异步计算任务的运行状态为其他状态, 则抛出ExecutionException异常
    throw new ExecutionException((Throwable)x);
}
```

###### awaitDone

```java
// 自旋+阻塞/中断/超时, 以等待任务已经完成、已取消、已发生异常或者已中断, 则清空等待结点, 并返回任务完成时的状态
private int awaitDone(boolean timed, long nanos) throws InterruptedException {
    // 如果需要定时, 则计算截止时间 = 系统当前纳秒数 + 定时纳秒数; 否则截止时间为0
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;

    // 开始自旋
    for (;;) {
        // 自旋过程中, 第一步就是测试当前线程是否被中断, 该方法会清除线程的中断状态
        if (Thread.interrupted()) {
            // 从头遍历等待线程堆栈链表, 已脱钩其中的垃圾结点(等待线程为null的结点)
            removeWaiter(q);

            // 清空完垃圾结点然后抛出中断异常
            throw new InterruptedException();
        }

        // 如果自旋过程中, 当前线程没有被中断, 则volatile方式获取任务的状态s
        int s = state;

        // 如果任务已经完成、已取消、已发生异常或者已中断, 则清空q结点的thread引用, 并返回s状态, 代表完成时的状态
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;
            return s;
        }
        // 如果任务正在完成中, 则当前线程先让出CPU, 缓一缓
        else if (s == COMPLETING) // cannot time out yet 还不能超时
            Thread.yield();
        // 如果q为null, 则使用当前线程初始化q为等待结点
        else if (q == null)
            q = new WaitNode();
        // 如果q不为null, 说明q已经在以上轮自旋中初始化完毕了, 且还没入栈的话, 则以volatile方式将q从头入栈, 并CAS更新waiters头结点为q
        else if (!queued)
            // CAS更新结果记录到入栈标识queued中
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset, q.next = waiters, q);
        // 如果需要定时时, 则还要判断是否超时, 剩余超时时间 = 截止时间 - 系统当前纳秒数
        else if (timed) {
            nanos = deadline - System.nanoTime();

            // 如果发生了超时, 则从头遍历等待线程堆栈链表, 已脱钩其中的垃圾结点(等待线程为null的结点), 并返回当前的任务状态, 代表完成时的状态
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }

            // 如果还没发生超时, 则阻塞当前线程nanos时间
            LockSupport.parkNanos(this, nanos);
        }
        // 如果不需要定时, 则直接阻塞当前线程
        else
            LockSupport.park(this);
    }
}
```

###### removeWaiter

```java
// 从头遍历等待线程堆栈链表, 已脱钩其中的垃圾结点(等待线程为null的结点)
private void removeWaiter(WaitNode node) {
    // 如果node结点不为null
    if (node != null) {
        // 则清空node结点的thread引用
        node.thread = null;

        // 开始自旋
        retry:
        for (;;) {          // restart on removeWaiter race // 在removeWaiter 比赛中重新开始
            // 遍历线程堆栈结点
            for (WaitNode pred = null, q = waiters, s; q != null; q = s) {
                // volatile方式获取结点q后继s
                s = q.next;

                // 如果q的thread引用不为null, 则继续往后遍历堆栈结点
                if (q.thread != null)
                    pred = q;
                // 如果q的thread引用为null, 且前驱不为null, 说明该结点是个垃圾结点, 且在堆栈链表中间
                else if (pred != null) {
                    // 则volatile方式链接前驱和后继, 脱钩结点q
                    pred.next = s;

                    // volatile方式获取前驱的thread引用, 如果此时thread引用也为null, 说明前驱也是个垃圾结点, 则重新自旋, 重头开始遍历
                    if (pred.thread == null) // check for race
                        continue retry;
                }
                // 如果q的thread引用为null, 且前驱也为null, 说明头结点为一个垃圾结点, 则CAS更新waiters引用指向下一个堆栈结点
                else if (!UNSAFE.compareAndSwapObject(this, waitersOffset, q, s))
                    // 然后重新自旋, 重头开始遍历
                    continue retry;
            }

            // 遍历结束, 代表垃圾结点(线程为null的结点)已被脱钩完毕, 则退出自旋
            break;
        }
    }
}
```

##### 其他监控方法

###### isDone

```java
// 判断任务是否完成中、已完成、已发生异常、已取消、发生中断或者已中断
public boolean isDone() {
    return state != NEW;
}
```

###### isCancelled

```java
// 判断任务是否已取消、发生中断或者已中断
public boolean isCancelled() {
    return state >= CANCELLED;
}
```

#### ThreadLocal

##### 特点

- ThreadLocal，提供线程局部变量，这些变量与它们的普通对应物不同，因为每个访问一个的线程（通过其 {@code get} 或 {@code set} 方法）都有**自己的、独立初始化的变量副本**。
  - {@code ThreadLocal} 实例，通常是希望将状态与Thread类中的私有静态字段（例如，用户 ID 或事务 ID）相关联。
- 只要线程处于活动状态，并且 {@code ThreadLocal} 实例可访问，每个线程都持有对其线程局部变量副本的**隐式引用**；线程消失后，它的所有线程本地实例副本都将进行**垃圾回收**（除非存在对这些副本的其他引用）。
  - ThreadLocals 依赖于附加到每个线程（Thread.threadLocals 和inheritableThreadLocals）的每线程线性探针哈希映射。
  - **ThreadLocal 对象充当键**，通过 threadLocalHashCode 进行搜索。 这是一个自定义哈希代码（仅在 ThreadLocalMaps 中有用），它消除了在相同线程使用连续构造的 ThreadLocals 的常见情况下的冲突，同时在不太常见的情况下保持良好行为。

##### 构造方法

```java
public class ThreadLocal<T> {
    // 线程本地副本hashCode
    private final int threadLocalHashCode = nextHashCode();
    // 要给出的下一个哈希码, 原子更新, 从零开始
    private static AtomicInteger nextHashCode = new AtomicInteger();
    // 连续生成的哈希码之间的差异 - 将隐式顺序线程本地 ID 转换为接近最优分布的乘法哈希值，用于 2 次方大小的表。
    private static final int HASH_INCREMENT = 0x61c88647;
    
    // 返回下一个哈希码
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }
    
    // 创建线程局部变量
    public ThreadLocal() {
        
    }
    
    // ThreadLocal 的扩展，它从指定的 {@code Supplier} 获取其初始值
    static final class SuppliedThreadLocal<T> extends ThreadLocal<T> {

        private final Supplier<? extends T> supplier;

        SuppliedThreadLocal(Supplier<? extends T> supplier) {
            this.supplier = Objects.requireNonNull(supplier);
        }

        @Override
        protected T initialValue() {
            return supplier.get();
        }
    }
    
    // 创建线程局部变量。 变量的初始值是通过调用 {@code Supplier} 上的 {@code get} 方法确定的。
    public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier) {
        return new SuppliedThreadLocal<>(supplier);
    }
}
```

##### 设置变量方法

###### set

```java
// 将此线程局部变量的当前线程副本设置为指定值
public void set(T value) {
    // 获取当前线程
    Thread t = Thread.currentThread();

    // 获取ThreadLocal.ThreadLocalMap threadLocals
    ThreadLocalMap map = getMap(t);

    // 如果threadLocals已被初始化, 则设置ThreadLocal-value条目, 如果ThreadLocal存在则替换旧值; 如果ThreadLocal存在空键, 则交换旧值到新桶并向后清空空键; 如果ThreadLocal不存在条目, 则新增Entry条目, 如果新增后实际大小超过了1/2则还需要清空所有弱键并扩容2倍散列表
    if (map != null)
        map.set(this, value);
    // 如果threadLocals还没被初始化, 则惰性构造默认容量16、默认负载因子16 * 2/3、当前ThreadLocal实例作为第一个Entry#Key的ThreadLocalMap, 只有在至少有一个条目可以放入时才创建
    else
        createMap(t, value);
}
```

###### createMap

```java
// 惰性构造默认容量16、默认负载因子16 * 2/3、当前ThreadLocal实例作为第一个Entry#Key的ThreadLocalMap, 只有在至少有一个条目可以放入时才创建
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

###### createInheritedMap

```java
// Thread#init调用, 创建继承线程局部变量映射的工厂方法
static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
    return new ThreadLocalMap(parentMap);
}
```

##### 获取变量方法

###### get

```java
// 先根据当前ThreadLocal实例的散列码获取ThreadLocalMap中的条目, 如果ThreadLocalMap还没被初始化, 则惰性构造默认容量16、默认负载因子16 * 2/3、当前ThreadLocal实例作为第一个Entry#Key的ThreadLocalMap, 并赋予初始值
public T get() {
    // 获取当前线程
    Thread t = Thread.currentThread();

    // 获取ThreadLocal.ThreadLocalMap threadLocals
    ThreadLocalMap map = getMap(t);

    // 如果ThreadLocal.ThreadLocalMap threadLocals已被初始化
    if (map != null) {
        // 则根据ThreadLocal中的散列码, 来获取ThreadLocalMap中的Entry, 如果根据散列码索引获取不到Entry, 则调用getEntryAfterMiss查找: 如果键匹配则返回e, 如果键为空则清空陈旧的条目然后返回null, 否则获取下一个索引i的Entry
        ThreadLocalMap.Entry e = map.getEntry(this);

        // 如果当前ThreadLocal实例不为null, 则返回其对应的value值
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }

    // 如果ThreadLocal.ThreadLocalMap threadLocals还没初始化, 则先获取初始值, 如果ThreadLocal.ThreadLocalMap threadLocals已被初始化, 则为其赋初值, 否则惰性构造默认容量16、默认负载因子16 * 2/3、当前ThreadLocal实例作为第一个Entry#Key的ThreadLocalMap
    return setInitialValue();
}
```

###### setInitialValue

```java
// 先获取初始值, 如果ThreadLocal.ThreadLocalMap threadLocals已被初始化, 则为其赋初值, 否则惰性构造默认容量16、默认负载因子16 * 2/3、当前ThreadLocal实例作为第一个Entry#Key的ThreadLocalMap
private T setInitialValue() {
    // 返回此线程局部变量的当前线程的“初始值”。 该方法将在线程第一次使用 {@link #get} 方法访问变量时被调用，除非线程之前调用了 {@link #set} 方法
    T value = initialValue();

    // 获取当前线程
    Thread t = Thread.currentThread();

    // 获取ThreadLocal.ThreadLocalMap threadLocals
    ThreadLocalMap map = getMap(t);

    // 如果threadLocals已被初始化, 则为map赋予"初始值"
    if (map != null)
        map.set(this, value);
    // 如果threadLocals还没被初始化, 则惰性构造默认容量16、默认负载因子16 * 2/3、当前ThreadLocal实例作为第一个Entry#Key的ThreadLocalMap, 只有在至少有一个条目可以放入时才创建
    else
        createMap(t, value);

    // 然后返回"初始值"
    return value;
}
```

###### initialValue

```java
// 返回此线程局部变量的当前线程的“初始值”。
// 该方法将在线程第一次使用 {@link #get} 方法访问变量时被调用, 除非线程之前调用了 {@link #set} 方法, 在这种情况下，{@code initialValue} 方法将不会被调用为线程调用。
// 通常, 每个线程最多调用此方法一次, 但在后续调用 {@link #remove} 后跟 {@link #get} 的情况下可能会再次调用。
// 此实现仅返回 {@code null}; 如果程序员希望线程局部变量具有 {@code null} 以外的初始值，则必须对 {@code ThreadLocal} 进行子类化，并覆盖此方法。
protected T initialValue() {
    return null;
}
```

###### childValue

```java
// 子类InheritableThreadLocal中定义的方法, 用于计算可继承线程局部变量的子级初始值, 作为创建子线程时父级值的函数
T childValue(T parentValue) {
    throw new UnsupportedOperationException();
}
```

##### 删除变量方法

###### remove

```java
// 如果threadLocals已经被初始化, 则删除ThreadLocal的条目, 如果找到e键为指定的ThreadLocal, 则清空其弱键条目, 否则什么也不做
public void remove() {
    // 获取ThreadLocal.ThreadLocalMap threadLocals
    ThreadLocalMap m = getMap(Thread.currentThread());

    // 如果threadLocals已经被初始化, 则删除ThreadLocal的条目, 如果找到e键为指定的ThreadLocal, 则清空其弱键条目, 否则什么也不做
    if (m != null)
        m.remove(this);
}
```

##### ThreadLocalMap

```java
static class ThreadLocalMap {
    
	private static final int INITIAL_CAPACITY = 16;// 初始容量，必须是2的幂
    private Entry[] table;// 散列表，table.length必须始终是2的幂
    private int size = 0;// 散列表实际大小
    private int threshold;// 散列表阈值
	
    // 条目对象
    static class Entry extends WeakReference<ThreadLocal<?>> {
        Object value;
        
        // Entry#Key 强引用=> WeakReference 弱引用-> ThreadLocal:v
        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
    
    // 惰性构造默认容量16、默认负载因子16 * 2/3、ThreadLocal作为第一个Entry#Key的ThreadLocalMap, 只有在至少有一个条目可以放入时才创建
    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        // 构造默认容量16的散列表table
        table = new Entry[INITIAL_CAPACITY];

        // 根据线程本地副本hashCode计算所在散列表中的索引i
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);

        // 使用ThreadLocal作为Entry#Key, 使用firstValue作为Entry#Value, 构造Entry并放入散列表中指定索引的位置
        table[i] = new Entry(firstKey, firstValue);

        // 更新散列表实际大小
        size = 1;

        // 根据容量设置阈值 => 取容量的2/3
        setThreshold(INITIAL_CAPACITY);
    }
    
    // 从给定的父映射构造一个包含所有可继承线程本地的新映射。 仅由 createInheritedMap 调用
    private ThreadLocalMap(ThreadLocalMap parentMap) {
        // 获取父散列表parentTable, 父散列表容量len
        Entry[] parentTable = parentMap.table;
        int len = parentTable.length;

        // 根据len容量设置阈值 => 取容量的2/3, 构造len长的散列表
        setThreshold(len);
        table = new Entry[len];

        // 遍历父散列表, 调用childValue计算值, 以及重新散列, 重新构造子线程本地副本
        for (int j = 0; j < len; j++) {
            Entry e = parentTable[j];
            if (e != null) {
                @SuppressWarnings("unchecked")
                ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
                if (key != null) {
                    Object value = key.childValue(e.value);
                    Entry c = new Entry(key, value);
                    int h = key.threadLocalHashCode & (len - 1);
                    while (table[h] != null)
                        h = nextIndex(h, len);
                    table[h] = c;
                    size++;
                }
            }
        }
    }
}

// 根据容量设置阈值 => 取容量的2/3
private void setThreshold(int len) {
    threshold = len * 2 / 3;
}
```

###### 线性探测方法

```java
// 线性探测法: 用于解决此时产生的冲突, 查找散列表中离冲突单元最近的空闲单元，并且把新的键插入这个空闲单元。同样的，查找也同插入如出一辙：从散列函数给出的散列值对应的单元开始查找，直到找到与键对应的值或者是找到空单元。

// 线性探测法, 获取下一个索引, i+1和0循环获取
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}

// 线性探测法, 获取上一个索引, i-1和n-1循环获取
private static int prevIndex(int i, int len) {
    return ((i - 1 >= 0) ? i - 1 : len - 1);
}
```

###### 重新哈希方法

```java
// 清除表中的所有陈旧条目, 如果实际大小超过了1/2, 则扩容散列表为原来的2倍, 遍历旧表, 如果为空键, 则清空条目; 如果不是, 则重新哈希Entry放入到新表的新桶中
private void rehash() {
    // 清除表中的所有陈旧条目
    expungeStaleEntries();

    // 使用较低的加倍阈值以避免滞后
    // Use lower threshold for doubling to avoid hysteresis
    // 如果实际大小超过了2/3=8/12 - 2/12=6/12=1/2, 则扩容散列表为原来的2倍, 遍历旧表, 如果为空键, 则清空条目; 如果不是, 则重新哈希Entry放入到新表的新桶中
    if (size >= threshold - threshold / 4)
        resize();
}
```

###### 扩容方法

```java
// 扩容散列表为原来的2倍, 遍历旧表, 如果为空键, 则清空条目; 如果不是, 则重新哈希Entry放入到新表的新桶中
private void resize() {
    Entry[] oldTab = table;
    int oldLen = oldTab.length;
    int newLen = oldLen * 2;
    Entry[] newTab = new Entry[newLen];
    int count = 0;

    // 遍历旧表, 如果为空键, 则清空条目; 如果不是, 则重新哈希Entry放入到新表的新桶中
    for (int j = 0; j < oldLen; ++j) {
        Entry e = oldTab[j];
        if (e != null) {
            ThreadLocal<?> k = e.get();
            if (k == null) {
                e.value = null; // Help the GC
            } else {
                int h = k.threadLocalHashCode & (newLen - 1);
                while (newTab[h] != null)
                    h = nextIndex(h, newLen);
                newTab[h] = e;
                count++;
            }
        }
    }

    // 根据容量设置阈值 => 取容量的2/3
    setThreshold(newLen);
    size = count;
    table = newTab;
}
```

###### 设置条目方法

```java
// 设置ThreadLocal-value条目, 如果ThreadLocal存在则替换旧值; 如果ThreadLocal存在空键, 则交换旧值到新桶并向后清空空键; 如果ThreadLocal不存在条目, 则新增Entry条目, 如果新增后实际大小超过了1/2则还需要清空所有弱键并扩容2倍散列表
private void set(ThreadLocal<?> key, Object value) {

    // 根据ThreadLocal散列码获取表索引
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    // 根据表索引获取Entry e, 遍历散列表, 直到为null的e
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        // 获取e的弱键k
        ThreadLocal<?> k = e.get();

        // 如果k为指定的ThreadLocal, 则替换e值为value后, 直接返回即可
        if (k == key) {
            e.value = value;
            return;
        }

        // 如果k为null, 说明弱键已被清除
        if (k == null) {
            // 如果value存放的桶本来有弱键, 则交换弱键到新桶, 并向后清空弱键的条目; 如果没有则在staleSlot处构造key-value条目, 并向后清空弱键的条目
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    // 如果i索引或者向后索引处桶为null, 则根据key-value构造Entry, 并更新散列表的实际大小
    tab[i] = new Entry(key, value);
    int sz = ++size;

    // 从i处开始遍历散列表, 清空一路上的空键Entry, 如果有发生删除则返回true, 否则返回false
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        // 如果没有删除弱键且实际大小超过了阈值, 则清除表中的所有陈旧条目, 如果实际大小超过了1/2, 则扩容散列表为原来的2倍, 遍历旧表, 如果为空键, 则清空条目; 如果不是, 则重新哈希Entry放入到新表的新桶中
        rehash();
}

// 如果value存放的桶本来有弱键, 则交换弱键到新桶, 并向后清空弱键的条目; 如果没有则在staleSlot处构造key-value条目, 并向后清空弱键的条目
private void replaceStaleEntry(ThreadLocal<?> key, Object value, int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;
    Entry e;

    // 备份空键索引slotToExpunge, 向前遍历散列表, 如果遇到空键则赋予给slotToExpunge, 代表第一个空键索引
    int slotToExpunge = staleSlot;
    for (int i = prevIndex(staleSlot, len); (e = tab[i]) != null; i = prevIndex(i, len))
        if (e.get() == null)
            slotToExpunge = i;

    // 从staleSlot向后遍历散列表, 直到遇到null的条目
    for (int i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i, len)) {
        // 获取e的弱键k
        ThreadLocal<?> k = e.get();

        // 如果k与指定的ThreadLocal相等, 则将value交换到e上, 即索引staleSlot的条目与索引i的条目进行交换
        if (k == key) {
            e.value = value;

            tab[i] = tab[staleSlot];
            tab[staleSlot] = e;

            // 如果存在，则从前面的陈旧条目开始清除, 则从i位置开始向后清除空键Entry
            // Start expunge at preceding stale entry if it exists
            if (slotToExpunge == staleSlot)
                slotToExpunge = i;

            // 从i处开始遍历散列表, 清空一路上的空键Entry, 如果有发生删除则返回true, 否则返回false
            cleanSomeSlots(
                // 根据空键表索引清除陈旧的Entry, 清空staleSlot位置的Entry, 然后从后一位开始遍历清除所有空键的Entry, 或者将其整理到第一个为null的槽位中, 返回staleSlot下一个null的索引
                expungeStaleEntry(slotToExpunge), len);

            // 清除后则返回
            return;
        }

        // 如果k为null, 则从staleSlot处开始向后清除空键Entry
        if (k == null && slotToExpunge == staleSlot)
            slotToExpunge = i;
    }

    // 如果未找到密钥，则将新条目放入陈旧插槽中
    // If key not found, put new entry in stale slot
    tab[staleSlot].value = null;
    tab[staleSlot] = new Entry(key, value);

    // 在staleSlot构造完Entry后, 如果staleSlot不等于slotToExpunge, 说明散列表中至少有2个条目
    if (slotToExpunge != staleSlot)
        // 则从i处开始遍历散列表, 清空一路上的空键Entry, 如果有发生删除则返回true, 否则返回false
        cleanSomeSlots(
        // 根据空键表索引清除陈旧的Entry, 清空staleSlot位置的Entry, 然后从后一位开始遍历清除所有空键的Entry, 或者将其整理到第一个为null的槽位中, 返回staleSlot下一个null的索引
        expungeStaleEntry(slotToExpunge), len);
}
```

###### 清除旧条目方法

```java
// 删除ThreadLocal的条目, 如果找到e键为指定的ThreadLocal, 则清空其弱键条目, 否则什么也不做
private void remove(ThreadLocal<?> key) {
    // 根据ThreadLocal散列码获取表索引i
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    // 从i处向后遍历散列表, 直到为null的条目
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        // 如果e键为指定的ThreadLocal, 则清空其弱键条目
        if (e.get() == key) {
            // (非JVM调用)清空软/弱/虚 ref引用的实例对象(即data实际业务对象)
            e.clear();

            // 根据空键表索引清除陈旧的Entry, 清空staleSlot位置的Entry, 然后从后一位开始遍历清除所有空键的Entry, 或者将其整理到第一个为null的槽位中, 返回staleSlot下一个null的索引
            expungeStaleEntry(i);
            return;
        }
    }
}

// 根据空键表索引清除陈旧的Entry, 清空staleSlot位置的Entry, 然后从后一位开始遍历清除所有空键的Entry, 或者将其整理到第一个为null的槽位中, 返回staleSlot下一个null的索引
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // 在 staleSlot 删除条目
    // expunge entry at staleSlot
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    // 重新哈希直到我们遇到 null
    // Rehash until we encounter null
    Entry e;
    int i;

    // 从staleSlot后一位开始遍历, 直到碰到为null的Entry
    for (i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i, len)) {
        // 获取其弱键k
        ThreadLocal<?> k = e.get();

        // 如果k为null, 说明k也被清除了, 则清除关联的条目
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        }
        // 如果没有被清除, 则根据k重新散列, 然后清空原i的槽, 然后从h扫描直到碰到为null的槽, 最后把e放入
        else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;

                // 与 Knuth 6.4 算法 R 不同，我们必须扫描直到为空，因为多个条目可能已经过时。
                // Unlike Knuth 6.4 Algorithm R, we must scan until
                // null because multiple entries could have been stale.
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }

    // 返回staleSlot下一个null的索引
    return i;
}

// 从i处开始遍历散列表, 清空一路上的空键Entry, 如果有发生删除则返回true, 否则返回false
private boolean cleanSomeSlots(int i, int n) {
    boolean removed = false;
    Entry[] tab = table;
    int len = tab.length;
    do {
        // 从i处开始遍历散列表, 碰到弱键则根据空键表索引清除陈旧的Entry, 清空staleSlot位置的Entry, 然后从后一位开始遍历清除所有空键的Entry, 或者将其整理到第一个为null的槽位中, 返回staleSlot下一个null的索引作为新的索引i
        i = nextIndex(i, len);
        Entry e = tab[i];
        if (e != null && e.get() == null) {
            n = len;
            removed = true;
            i = expungeStaleEntry(i);
        }
    } while ( (n >>>= 1) != 0);
    return removed;
}
```

###### 获取条目方法

```java
// 根据ThreadLocal中的散列码, 来获取ThreadLocalMap中的Entry, 如果根据散列码索引获取不到Entry, 则调用getEntryAfterMiss查找: 如果键匹配则返回e, 如果键为空则清空陈旧的条目然后返回null, 否则获取下一个索引i的Entry
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];

    // 如果弱键还没被清空, 则返回找到的Entry
    if (e != null && e.get() == key)
        return e;
    // 如果e不存在, 或者弱键已过期, 则调用getEntryAfterMiss获取
    else
        // 根据索引获取不到Entry后调用, 如果键匹配则返回e, 如果键为空则清空陈旧的条目然后返回null, 否则获取下一个索引i的Entry
        return getEntryAfterMiss(key, i, e);
}

// 根据索引获取不到Entry后调用, 如果键匹配则返回e, 如果键为空则清空陈旧的条目然后返回null, 否则获取下一个索引i的Entry
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    // 获取散列表tab, 散列表容量len
    Entry[] tab = table;
    int len = tab.length;

    // 如果e为null, 则返回null
    while (e != null) {
        // 获取e弱键中的ThreadLocal k
        ThreadLocal<?> k = e.get();

        // 如果弱键匹配, 说明弱引用还没被清除, 则返回e
        if (k == key)
            return e;

        // 如果弱键为null, 说明弱键已被清除
        if (k == null)
            // 则根据空键表索引清除陈旧的Entry, 清空staleSlot位置的Entry, 然后从后一位开始遍历清除所有空键的Entry, 或者将其整理到第一个为null的槽位中, 返回staleSlot下一个null的索引
            expungeStaleEntry(i);
        // 如果弱键不匹配也不为null, 则获取下一个索引的Entry e
        else
            i = nextIndex(i, len);
        e = tab[i];
    }

    // 如果e为null, 则返回null
    return null;
}
```

##### InheritableThreadLocal

- InheritableThreadLocal，**可继承的线程局部变量**，扩展 ThreadLocal 以提供从父线程到子线程的值继承，当创建子线程时，子线程接收父线程具有的所有可继承线程局部变量的初始值。
  - 通常孩子的价值观将与父母的相同， 但是，通过覆盖此类中的 childValue 方法，可以将子项的值设为父项的任意函数。
- 当在变量中维护的每个线程属性（例如，用户 ID、事务 ID）必须自动传输到创建的任何子线程时，可优先使用**可继承的线程局部变量**，而不是普通的线程局部变量。

```java
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    // 子类InheritableThreadLocal中定义的方法, 用于计算可继承线程局部变量的子级初始值, 作为创建子线程时父级值的函数
    protected T childValue(T parentValue) {
        return parentValue;
    }
    
    // 获取ThreadLocal.ThreadLocalMap inheritableThreadLocals
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }
    
    // 创建与 ThreadLocal 关联的映射
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```



### 典型实现

#### AtomicInteger

##### Number

- 抽象类 {@code Number}， 是**平台类的超类**，表示可转换为基本类型 {@code byte}、{@code double}、{@code float}、{@code int}、{@code long}和{@code short}。
- 从特定 {@code Number} 实现的数值到给定原始类型的转换的特定语义，由所讨论的 {@code Number} 实现定义。
- 对于平台类，转换通常类似于在 Java™ 语言规范中定义的、用于在原始类型之间转换的**缩小原语转换**或**扩大原语转换**。因此，转换可能会丢失有关数值整体大小的信息，**可能会丢失精度**，甚至可能返回与输入符号不同的结果。

```java
public abstract class Number implements java.io.Serializable {
    
    // 以 {@code int} 形式返回指定数字的值，这可能涉及舍入或截断
    public abstract int intValue();
    
    // 以 {@code long} 形式返回指定数字的值，这可能涉及舍入或截断。
    public abstract long longValue();
    
    // 以 {@code float} 形式返回指定数字的值，这可能涉及四舍五入。
    public abstract float floatValue();
    
    // 以 {@code double} 形式返回指定数字的值，这可能涉及四舍五入。
    public abstract double doubleValue();
    
    // 将指定数字的值作为 {@code byte} 返回，这可能涉及舍入或截断, 此实现将 {@link #intValue} 转换为 {@code byte} 的结果返回
    public byte byteValue() {
        return (byte)intValue();
    }
    
    // 将指定数字的值作为 {@code short} 返回，这可能涉及舍入或截断，此实现将 {@link #intValue} 转换为 {@code short} 的结果返回
    public short shortValue() {
        return (short)intValue();
    }
}
```

![1629376273292](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1629376273292.png)

##### 特点

- AtomicInteger，可以**原子更新的 {@code int} 值**，有关原子变量属性的描述，请参阅 {@link java.util.concurrent.atomic} 包规范。
- {@code AtomicInteger} 用于原子递增计数器等应用程序中，不能用作 {@link java.lang.Integer} 的替代品。但是，此类确实扩展了 {@code Number}抽象类， 以允许处理基于数字类的工具和实用程序进行统一访问。

##### 构造方法

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    
    private volatile int value;
    
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;// value
    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
    
    // 使用初始值 {@code 0} 创建一个新的 AtomicInteger
    public AtomicInteger() {
    }
    
    // 使用给定的初始值创建一个新的 AtomicInteger
    public AtomicInteger(int initialValue) {
        value = initialValue;
    }
}
```

##### 直接设置与获取方法

###### get

```java
// 获取当前值
public final int get() {
    return value;
}
```

###### set

```java
// 设置为给定值
public final void set(int newValue) {
    value = newValue;
}
```

###### lazySet

```java
// 最终设置为给定值
public final void lazySet(int newValue) {
    // 使用volatile存储语义将引用值存储到给定的Java变量中。否则等同于{@link #putObject(Object, long, Object)}, 该方法不保证存储对其他线程的立即可见性
    unsafe.putOrderedInt(this, valueOffset, newValue);
}
```

###### getAndSet

```java
// 原子地设置为给定值并返回旧值
public final int getAndSet(int newValue) {
    return unsafe.getAndSetInt(this, valueOffset, newValue);
}
```

###### compareAndSet

```java
// CAS设置为目标值
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

###### weakCompareAndSet

```java
// CAS设置为目标值, 可能会错误地失败并且不提供排序保证, 是 {@code compareAndSet} 的合适替代品
public final boolean weakCompareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

###### getAndIncrement

```java
// CAS递增1, 并返回旧值
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

###### getAndDecrement

```java
// CAS递减1, 并返回旧值
public final int getAndDecrement() {
    return unsafe.getAndAddInt(this, valueOffset, -1);
}
```

###### getAndAdd

```java
// CAS递增delta, 并返回旧值
public final int getAndAdd(int delta) {
    return unsafe.getAndAddInt(this, valueOffset, delta);
}
```

###### incrementAndGet

```java
// CAS递增1, 并返回新值
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```

###### decrementAndGet

```java
// CAS递减1, 并返回新值
public final int decrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, -1) - 1;
}
```

###### addAndGet

```java
// CAS递增delta, 并返回新值
public final int addAndGet(int delta) {
    return unsafe.getAndAddInt(this, valueOffset, delta) + delta;
}
```

##### 函数设置与获取方法

###### getAndUpdate

```java
// CAS递增updateFunction的结果, 并返回旧值
public final int getAndUpdate(IntUnaryOperator updateFunction) {
    int prev, next;
    do {
        prev = get();
        next = updateFunction.applyAsInt(prev);
    } while (!compareAndSet(prev, next));
    return prev;
}
```

###### updateAndGet

```java
// CAS递增updateFunction的结果, 并返回新值
public final int updateAndGet(IntUnaryOperator updateFunction) {
    int prev, next;
    do {
        prev = get();
        next = updateFunction.applyAsInt(prev);
    } while (!compareAndSet(prev, next));
    return next;
}
```

###### getAndAccumulate

```java
// CAS递增accumulatorFunction的结果, 并返回旧值
public final int getAndAccumulate(int x, IntBinaryOperator accumulatorFunction) {
    int prev, next;
    do {
        prev = get();
        next = accumulatorFunction.applyAsInt(prev, x);
    } while (!compareAndSet(prev, next));
    return prev;
}
```

###### accumulateAndGet

```java
// CAS递增accumulatorFunction的结果, 并返回新值
public final int accumulateAndGet(int x, IntBinaryOperator accumulatorFunction) {
    int prev, next;
    do {
        prev = get();
        next = accumulatorFunction.applyAsInt(prev, x);
    } while (!compareAndSet(prev, next));
    return next;
}
```

##### Number接口方法

###### intValue

```java
// 将此 {@code AtomicInteger} 的值作为 {@code int} 返回。
public int intValue() {
    return get();
}
```

###### longValue

```java
// 在扩展原始转换后，将此 {@code AtomicInteger} 的值作为 {@code long} 返回。
public long longValue() {
    return (long)get();
}
```

###### floatValue

```java
// 在扩展原始转换后，将此 {@code AtomicInteger} 的值作为 {@code float} 返回。
public float floatValue() {
    return (float)get();
}
```

###### doubleValue

```java
// 在扩展原始转换后，将此 {@code AtomicInteger} 的值作为 {@code double} 返回。
public double doubleValue() {
    return (double)get();
}
```

#### ThreadPoolExecutor

##### Executor

- **执行提交的 {@link Runnable} 任务的对象**，该接口提供了一种将**任务提交与每个任务将如何运行的机制分离**的方法，包括线程使用、调度等的细节，通常使用 {@code Executor} 而不是显式创建线程。
- JUC包中提供的{@link ExecutorService}实现了{@code Executor} ，这是一个更广泛的接口。
-  {@link ThreadPoolExecutor} 类提供了一个可扩展的线程池实现，{@link Executors} 类为这些 Executor 提供了方便的工厂方法。

```java
public interface Executor {
    // 在将来的某个时间执行给定的命令。 该命令可以在新线程、池线程或调用线程中执行，具体取决于 {@code Executor} 实现。
    void execute(Runnable command);
}
```

##### ExecutorService

- ExecutorService，一个 {@link Executor} **提供管理终止的方法和可以生成 {@link Future}** 以跟踪一个或多个异步任务进度的方法。
- 可以关闭未使用的{@code ExecutorService}以回收其资源，但将会导致它拒绝新任务。提供了两种不同的方法来关闭 {@code ExecutorService}：
  - {@link #shutdown} 方法，将允许先前提交的任务在终止之前执行。
  - {@link #shutdownNow} 方法，防止等待任务开始并尝试停止当前正在执行的任务。终止时，执行器没有正在执行的任务，没有等待执行的任务，也没有新的任务可以提交。
- 方法 {@code submit} 通过创建和返回可用于取消执行或者等待完成的 {@link Future} 来扩展基本方法 {@link Executor#execute(Runnable)}。
- 方法 {@code invokeAny} 和 {@code invokeAll} 执行最常用的批量执行形式，执行一组任务，然后等待至少一个或全部完成。
- {@link Executors} 类为此包中提供的执行程序服务提供工厂方法。

```java
public interface ExecutorService extends Executor {
    // 有序关闭Executor, 会执行先前提交的任务, 但不会接受新任务; 如果已经关闭, 则调用没有额外的效果; 此方法不等待先前提交的任务完成执行, 可以使用{@link #awaitTermination awaitTermination}来做到这一点
    void shutdown();
    
    // 有序关闭Executor, 会停止所有正在执行的任务, 停止等待任务的处理, 并返回等待执行的任务列表; 此方法不等待先前提交的任务完成执行, 可以使用{@link #awaitTermination awaitTermination}来做到这一点
    List<Runnable> shutdownNow();
    
    // 判断Executor是否已经关闭, 如果是则返回{@code true}
    boolean isShutdown();
    
    // 如果关闭后所有任务都已完成, 则返回true
    boolean isTerminated();
    
    // 阻塞直到所有任务在关闭请求后执行完成, 或发生超时, 或当前线程被中断, 以先发生者为准
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
    
    // 提交一个返回值的任务以供执行, 并返回一个表示任务未决结果的Future, Future的{@code get}方法将在成功完成后返回任务的结果
    <T> Future<T> submit(Callable<T> task);
    
    // 提交一个Runnable任务以供执行，并返回一个代表该任务的Future, Future的{@code get}方法将在成功完成后返回给定的结果
    <T> Future<T> submit(Runnable task, T result);
    
    // 提交一个 Runnable 任务以供执行，并返回一个代表该任务的 Future。 Future 的 {@code get} 方法将在成功完成后返回 {@code null}
    Future<?> submit(Runnable task);
    
    // 执行给定的任务，返回一个 Futures 列表，在所有完成时保存它们的状态和结果。如果在此操作进行时修改了给定的集合，则此方法的结果未定义。
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
    
    // 执行给定的任务，当全部完成或超时到期时，返回一个保存其状态和结果的 Futures 列表，以先发生者为准。如果在此操作进行时修改了给定的集合，则此方法的结果未定义。
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException;
    
    // 执行给定的任务，返回成功完成的任务的结果（即不抛出异常），如果有的话。在正常或异常返回时，未完成的任务将被取消。如果在此操作进行时修改了给定的集合，则此方法的结果未定义。
    <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
    
    // 执行给定的任务，返回已成功完成的任务的结果（即不抛出异常），如果在给定的超时时间之前执行任何操作。 在正常或异常返回时，未完成的任务将被取消。如果在此操作进行时修改了给定的集合，则此方法的结果未定义。
    <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```

##### AbstractExecutorService

- AbstractExecutorService，提供 {@link ExecutorService} 执行方法的默认实现，并实现了{@code submit}、{@code invokeAny} 和 {@code invokeAll} 方法。
- AbstractExecutorService，使用 {@code newTaskFor} 返回的 {@link RunnableFuture} ，默认将传入的任务包裹为 {@link FutureTask} ，以实现RunnableFuture接口。
  - 子类可以覆盖 {@code newTaskFor} 方法以返回{@code RunnableFuture} 实现，而不是 {@code FutureTask}。

```java
public abstract class AbstractExecutorService implements ExecutorService {
    // 为给定的可运行任务和默认值返回{@code RunnableFuture}
    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }
    
    // 为给定的可调用任务返回 {@code RunnableFuture}
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new FutureTask<T>(callable);
    }
    
    // 实现ExecutorService方法, 运行给定的任务并返回{@code Future}对象
    public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();

        // 为给定的可运行任务和空的默认值返回{@code RunnableFuture}
        RunnableFuture<Void> ftask = newTaskFor(task, null);

        // Executor#execute方法, 交由子类实现
        execute(ftask);
        return ftask;
    }
    
    // 实现ExecutorService方法, 运行给定的任务并返回设置了result默认值的{@code Future}对象
    public <T> Future<T> submit(Runnable task, T result) {
        if (task == null) throw new NullPointerException();

        // 为给定的可运行任务和默认值返回{@code RunnableFuture}
        RunnableFuture<T> ftask = newTaskFor(task, result);

        // Executor#execute方法, 交由子类实现
        execute(ftask);
        return ftask;
    }
    
    // 实现ExecutorService方法, 运行给定的任务并返回{@code Future}对象
    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();

        // 为给定的可调用任务返回 {@code RunnableFuture}
        RunnableFuture<T> ftask = newTaskFor(task);

        // Executor#execute方法, 交由子类实现
        execute(ftask);
        return ftask;
    }
    
    // invokeAny主要机制, 执行给定的任务，当全部完成或超时到期时，返回一个保存其状态和结果的 Futures 列表，以先发生者为准。如果在此操作进行时修改了给定的集合，则此方法的结果未定义。
    private <T> T doInvokeAny(Collection<? extends Callable<T>> tasks, boolean timed, long nanos) throws InterruptedException, ExecutionException, TimeoutException {
        ...
    }
    
    // 实现ExecutorService方法, 执行给定的任务，返回成功完成的任务的结果（即不抛出异常），如果有的话。在正常或异常返回时，未完成的任务将被取消。 如果在此操作进行时修改了给定的集合，则此方法的结果未定义。
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException {
        try {
            return doInvokeAny(tasks, false, 0);
        } catch (TimeoutException cannotHappen) {
            assert false;
            return null;
        }
    }
    
    // 实现ExecutorService方法, 执行给定的任务，返回已成功完成的任务的结果（即，不抛出异常），如果在给定的超时时间之前执行任何操作。 在正常或异常返回时，未完成的任务将被取消。如果在此操作进行时修改了给定的集合，则此方法的结果未定义。
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException {
        return doInvokeAny(tasks, true, unit.toNanos(timeout));
    }
    
    // 实现ExecutorService方法, 执行给定的任务，返回一个 Futures 列表，在所有完成时保存它们的状态和结果。 {@link Future#isDone} 对于返回列表的每个元素都是 {@code true}。
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException {
    	...
    }
    
    // 实现ExecutorService方法, 执行给定的任务，当全部完成或超时到期时，返回一个保存其状态和结果的 Futures 列表，以先发生者为准。
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException {
        ...
    }
}
```

![1629461663248](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1629461663248.png)

##### 特点

- ThreadPoolExecutor，实现{@link ExecutorService}接口 ，**使用多个线程之一来执行每个提交的任务**，通常使用 {@link Executors} 工厂方法进行配置。

- 线程池解决两个不同的问题：

  - **减少了每个任务的调用开销**，在执行大量异步任务时提供改进的性能。
  - 提供了一种**限制和管理资源**的方法，包括在执行集合时消耗的线程任务。 
  - 此外，每个 {@code ThreadPoolExecutor} 还维护一些基本的统计信息，例如已完成的任务数。

- 为了在广泛的上下文中有用，ThreadPoolExecutor提供了许多**可调整的参数**和**可扩展性**挂钩。但是，JDK强烈建议（**实际工作中不建议**）程序员使用更方便的{@link Executors}工厂方法：

  - {@link Executors#newCachedThreadPool}：无界线程池，具有**自动线程回收**。
  - {@link Executors#newFixedThreadPool}：**固定大小**的线程池。
  - { @link Executors#newSingleThreadExecutor}：**单后台线程池**。

- **核心和最大池大小**： {@code ThreadPoolExecutor} 将根据**corePoolSize**（请参阅 {@link #getCorePoolSize}）和 **maximumPoolSize**（请参阅 {@link #getMaximumPoolSize}）设置的边界来自动调整池大小（请参阅 {@link #getPoolSize}）。

  - 当在方法 {@link #execute(Runnable)} 中提交新任务，并且运行的线程**少于 corePoolSize** 时，即使其他工作线程空闲，也会创建一个新线程来处理请求。
  - 如果有超过 corePoolSize 但小于 maximumPoolSize 的线程正在运行，则只有在**队列已满时**才会创建新线程。
  - 通过将 corePoolSize 和 maximumPoolSize 设置为相同，可以创建一个**固定大小**的线程池。
  - 通过将maximumPoolSize 设置为基本上无界的值，例如{@code Integer.MAX_VALUE}，可以允许池容纳**任意数量**的并发任务。
  - 核心和最大池大小仅在构造时设置，但也可以使用 {@link #setCorePoolSize} 和 {@link #setMaximumPoolSize} **动态更改**。

- **按需构建**：默认情况下，即使是核心线程也仅在**新任务到达时**最初创建和启动。

  - 如果使用非空队列构造池，想要**预启动线程**时，可以使用方法 {@link #prestartCoreThread} 或 {@link #prestartAllCoreThreads} 动态覆盖。

- **创建新线程**：使用 {@link ThreadFactory} 创建新线程。

  - 如果没有另外指定，则使用 {@link Executors#defaultThreadFactory}，它创建的线程都在同一个 {@link ThreadGroup} 中，并且具有相同的 {@code NORM_PRIORITY} 优先级和非守护进程状态。 
  - 通过提供不同的 ThreadFactory，可以更改线程的名称、线程组、优先级、守护进程状态等。如果 {@code ThreadFactory} 在通过从 {@code newThread} 返回 null 的线程，执行程序将会继续，但可能无法执行任何任务。
  -  线程应该拥有“modifyThread”{@code RuntimePermission}。 如果工作线程或其他使用池的线程不具备此权限，则服务可能会降级，而配置的更改可能无法及时生效，关闭池可能会一直处于可以终止但未完成的状态。

- **保活时间**：如果池中当前有超过 corePoolSize 的线程，则多余的线程如果空闲时间超过 keepAliveTime（请参阅 {@link #getKeepAliveTime(TimeUnit)}）将被终止，提供了一种在未积极使用池时**减少资源消耗**的方法，但如果池稍后变得更加活跃，则将构建新线程。

  - 可以使用方法 {@link #setKeepAliveTime(long, TimeUnit)} 动态更改此参数。
  - 使用 {@code Long.MAX_VALUE} {@link TimeUnit#NANOOSECONDS} 值可以有效地禁止空闲线程在关闭之前终止。 
  - 默认情况下，仅当有超过 corePoolSize 的线程时，保持活动策略才适用。 但是方法 {@link #allowCoreThreadTimeOut(boolean)} 也可用于将此超时策略应用于核心线程，只要 keepAliveTime 值不为零。

- **排队**：任何 {@link BlockingQueue} 都可用于**传输和保留提交的任务**。 

  - 此队列的使用与池大小**交互逻辑**为：

  1. 如果运行的线程数少于 corePoolSize，则 Executor 总是喜欢**添加新线程**而不是排队。
  2. 如果 corePoolSize 或更多线程正在运行，Executor 总是喜欢将**请求排队**而不是添加新线程。
  3. 如果请求无法排队，则会创建一个新线程，除非这会超过 maximumPoolSize，在这种情况下，**任务将被拒绝**。

  - **排队策略**一般有以下三种：
    - **直接交接**：
      - 工作队列的一个很好的默认选择是 {@link SynchronousQueue}，它将任务交给线程而不用其他方式保留它们。 在这里，如果没有线程可立即运行，则将任务排队的尝试将失败，因此将构建一个新线程。 在处理可能具有内部依赖性的请求集时，此策略可**避免锁定**。
      - 直接切换通常需要**无限的maximumPoolSizes**以避免拒绝新提交的任务，所以局限在于，当命令平均持续到达速度快于它们可以处理的速度时，可能会出现**无限线程的增长问题**。
    - **无界队列**： 
      - 使用无界队列（例如，没有预定义容量的 {@link LinkedBlockingQueue}）将导致新任务在所有 corePoolSize 线程都忙时在队列中等待。因此，**不会创建超过 corePoolSize 的线程**。 （因此maximumPoolSize的值没有任何影响。
      - 当**每个任务完全独立于其他任务**时，这可能是合适的，因此任务不会影响彼此的执行。例如，在网页服务器中。
      - 虽然这种排队方式**在平滑请求的瞬时爆发方面很有用**，但局限在于，当命令的平均到达速度超过它们的处理速度时，**工作队列可能会无限增长**。
    - **有界队列**：
      - 有界队列（例如，{@link ArrayBlockingQueue}）在与有限的 maximumPoolSizes 一起使用时有助于**防止资源耗尽**，但可能更**难以调整和控制**。
      - **队列大小和最大池大小可以相互权衡**：
        - **使用大队列和小池**：可以最大限度地减少CPU使用率、操作系统资源和上下文切换开销，但会导致人为地降低吞吐量。如果任务频繁阻塞（例如，如果它们受 I/O 限制），则系统可能能够为比您允许的更多线程安排时间。 
        - **使用小队列通常需要更大的池大小**：会使 CPU 更忙，但可能会遇到不可接受的调度开销，这也会降低吞吐量。

- **拒绝任务**：当 **Executor 已经关闭**，并且当 Executor 对最大线程和工作队列容量使用有限边界并且**饱和时**，在方法 {@link #execute(Runnable)} 中提交的新任务将被拒绝。

  - 在任何一种情况下，{@code execute} 方法都会调用其 {@link RejectedExecutionHandler} 的{@link RejectedExecutionHandler#rejectedExecution(Runnable, ThreadPoolExecutor)} 方法。
  - 提供了四个预定义的**拒绝策略**：
    -  {@link ThreadPoolExecutor.AbortPolicy} ：默认的拒绝策略，在任务被拒绝时，会**抛出运行时异常** {@link RejectedExecutionException}。
    -  {@link ThreadPoolExecutor.CallerRunsPolicy} ：调用 {@code execute} 时，线程自己会去运行任务，提供了一个简单的反馈控制机制，可以**减慢提交新任务的速度**。
    -  {@link ThreadPoolExecutor.DiscardPolicy} ：无法执行的任务被**简单地丢弃**。
    - {@link ThreadPoolExecutor.DiscardOldestPolicy}：如果Executor没有关闭，**工作队列头部的任务被丢弃**，然后重试执行，但可能会再次失败，导致重复执行。
    - 也可以**自定义**和使用其他类型的 {@link RejectedExecutionHandler} 类： 但这样做需要小心，特别是，在当策略设计是为仅在特定容量，或者排队策略下才工作时。

- **钩子方法**：ThreadPoolExecutor提供了 {@code protected} 可覆盖的 {@link #beforeExecute(Thread, Runnable)} 和 {@link #afterExecute(Runnable, Throwable)} 方法，这些方法在每个任务执行之前和之后调用，但是，如果钩子或回调方法抛出异常，内部工作线程**可能会失败并突然终止**。

  -  这些可用于操作执行环境， 例如，重新初始化 ThreadLocals、收集统计信息或添加日志条目。
  - 此外，方法 {@link #terminated} 可以被覆盖，以执行在 Executor 完全终止后需要完成的任何特殊处理。

- **队列维护**：

  - 方法 {@link #getQueue()} ：允许访问工作队列以进行监控和调试。强烈建议不要将此方法用于任何其他目的。
  -  {@link #remove(Runnable)} 和 {@link #purge}： 可用于在大量排队任务被取消时，进行协助存储回收。

- 最后注意，程序中**不再引用且没有剩余线程的池将会自动{@code shutdown}**。

  -  如果想确保即使忘记调用 {@link #shutdown} 能回收没有引用的池，则必须通过设置适当的保持活动时间，并使用为0的核心线程数，来安排未使用的线程最终死亡，或者设置 {@link #allowCoreThreadTimeOut(boolean)}。

##### 构造方法

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    // 主锁, 用于稳定工作线程集合
    private final ReentrantLock mainLock = new ReentrantLock();
    // 主锁的终止条件Condition
    private final Condition termination = mainLock.newCondition();
    // 任务队列: 保存任务、转交任务给工作线程
    private final BlockingQueue<Runnable> workQueue;
    // 工作线程集合
    private final HashSet<Worker> workers = new HashSet<Worker>();
    // 目前为止, 线程池中出现的最大线程数
    private int largestPoolSize;
    // 任务完成数
    private long completedTaskCount;
    // 线程工厂: 通过方法addWorker方法创建线程
    private volatile ThreadFactory threadFactory;
    // 拒绝策略: 核心线程数、任务队列、最大线程数饱和时, 再接受到新任务时会执行拒绝策略
    private volatile RejectedExecutionHandler handler;
    // 默认拒绝执行处理程序
    private static final RejectedExecutionHandler defaultHandler = new AbortPolicy();
    // 空闲线程的超时时间: 默认作用在非工作线程上
    private volatile long keepAliveTime;
    // 空闲线程的超时时间, 是否允许作用在核心工作线程上
    private volatile boolean allowCoreThreadTimeOut;
    // 核心线程数
    private volatile int corePoolSize;
    // 最大线程数
    private volatile int maximumPoolSize;

    // 指定基本线程池参数与任务队列, 使用默认的线程工厂和拒绝策略程序来创建线程池
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             // 默认创建优先级为5的非守护线程, 线程名称为pool-[?: poolNumber]-thread-[?: threadNumber]
             Executors.defaultThreadFactory(), defaultHandler);
    }
    
    // 指定基本线程池参数、任务队列与线程工厂, 使用默认的拒绝策略程序来创建线程池
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
    }

    // 指定基本线程池参数、任务队列与拒绝策略程序, 使用默认的线程工厂来创建线程池
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }
    
    // 指定基本线程池参数、任务队列、线程工厂和拒绝策略程序来创建线程池
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
}
```

##### ctl更新方法

- **主池控制状态ctl**，是一个原子整数，封装了两个概念字段：
  - workerCount，低29位，表示**有效线程数**。
  - runState，高3位，表示**线程池状态**，比如是否正在运行、正在关闭等。
- 为了将它们打包成一个int，workerCount 限制为 (2^29)-1（约 5 亿）个线程，而不是 (2^31)-1（20 亿）个其他可表示的线程。
  - 如果这在未来成为一个问题，可以将变量更改为 AtomicLong，并调整下面的移位/掩码常量。 但是在需要之前，这段代码使用 int 会更快更简单。
- workerCount，是允许启动以及不允许停止的工人数量。该值可能与实际的活动线程数暂时不同，例如当 ThreadFactory 在被询问时未能创建线程时，以及退出线程在终止前仍在ctl执行簿记时。 
  - 用户可通过getPoolSize获取为工作人员集的当前大小。
- runState提供主要的生命周期控制，提供5种取值，这些值之间的数字顺序很重要，以允许进行有序比较。
  - runState取值有：
    - **RUNNING**：接受新任务并处理排队任务。
    - **SHUTDOWN**：不接受新任务，但处理排队任务。
    - **STOP**：不接受新任务，不处理排队任务，并中断正在进行的任务。
    - **TIDYING**：所有任务都已终止，workerCount 为0，转换到状态 TIDYING 的线程将运行 terminate() 钩子方法。
    - **TERMINATED**： terminate() 钩子方法已完成。
  - runState 会随时间单调增加，但可以不用命中每个状态。转换有：
    - **RUNNING -> SHUTDOWN**：在调用 shutdown() 时，可能隐含在 finalize() 中。
    - **（RUNNING 或 SHUTDOWN）-> STOP**：在调用 shutdownNow() 时。
    - **SHUTDOWN -> TIDYING**：当队列和池都为空时。
    - **STOP -> TIDYING**：当池为空时。
    - **TIDYING -> TERMINATED**：当 terminate() 钩子方法完成时。

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
	// 线程池控制状态ctl, 高3位表示线程池状态, 低29位表示有效线程数, 初始为(1110, 0000, 0000, 0000, 0000, 0000, 0000, 0000) < 0
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
    
    // 29位
    private static final int COUNT_BITS = Integer.SIZE - 3;
    
    // 低29位存储有效线程数(约5亿个线程)
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
    
    private static final int RUNNING    = -1 << COUNT_BITS;// 运行状态, 高3位: 111
    private static final int SHUTDOWN   =  0 << COUNT_BITS;// 关闭状态, 高3位: 000
    private static final int STOP       =  1 << COUNT_BITS;// 停止状态, 高3位: 001
    private static final int TIDYING    =  2 << COUNT_BITS;// 整理状态, 高3位: 010
    private static final int TERMINATED =  3 << COUNT_BITS;// 终止状态, 高3位: 011
}
```

###### compareAndIncrementWorkerCount

```java
// CAS递增工作线程数
private boolean compareAndIncrementWorkerCount(int expect) {
    return ctl.compareAndSet(expect, expect + 1);
}
```

###### compareAndDecrementWorkerCount

```java
// CAS递减工作线程数
private boolean compareAndDecrementWorkerCount(int expect) {
    return ctl.compareAndSet(expect, expect - 1);
}
```

###### decrementWorkerCount

```java
// 减少一个工作线程, 底层依赖自旋+CAS递减工作线程数, 递减成功则退出自旋, 否则继续自旋
private void decrementWorkerCount() {
    do {} while (!compareAndDecrementWorkerCount(ctl.get()));
}
```

##### 生命周期方法

###### advanceRunState

```java
// 转换线程池状态为targetState
private void advanceRunState(int targetState) {
    // 开始自旋
    for (;;) {
        // 获取ctl控制位c
        int c = ctl.get();

        // 如果控制位c至少大于等于targetState, 则不做任何处理, 直接退出自旋; 如果c小于targetState, 则CAS更新为targetState状态
        if (runStateAtLeast(c, targetState) ||
            ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))))
            break;
    }
}
```

###### tryTerminate

```java
// 尝试线程池状态转换: STOP/SHUTDOWN -> 中断其中一个线程 -> TIDYING(工作线程数为0) -> TERMINATED状态 -> 通知所有等待线程池主锁Condition的线程
final void tryTerminate() {
    // 开始自旋
    for (;;) {
        // 获取状态控制位c
        int c = ctl.get();

        // 如果线程池为SHUTDOWN、TIDYING、TERMINATED状态, 或者为SHUTDOWN状态且任务队列非空时, 说明线程池不应该被终止或者已经被终止, 此时直接返回即可
        if (isRunning(c) ||
            runStateAtLeast(c, TIDYING) ||
            (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
            return;

        // 线程池为STOP状态 或者 为SHUTDOWN状态且队列为空, 如果工作线程数不为0, 此时确实需要清空工作线程
        if (workerCountOf(c) != 0) { // Eligible to terminate // 有资格终止
            // 中断可能正在等待任务的线程, 只中断其中一个线程, 中断后则返回
            interruptIdleWorkers(ONLY_ONE);
            return;
        }

        // 线程池为STOP状态 或者 为SHUTDOWN状态且队列为空, 如果工作线程数为0, 则获取线程池主锁
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            // CAS更新状态控制位为TIDYING, 并终止线程池
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                try {
                    terminated();
                } finally {
                    // 终止完毕则更新ctl为TERMINATED
                    ctl.set(ctlOf(TERMINATED, 0));

                    // 唤醒所有等待线程池主锁的终止条件Condition的线程, 然后返回
                    termination.signalAll();
                }
                return;
            }
        } finally {
            // 最后释放线程池主锁
            mainLock.unlock();
        }

        // 否则重试失败的 CAS
        // else retry on failed CAS
    }
}
```

###### shutdown

```java
// 启动有序关闭，其中执行先前提交的任务，但不会接受新任务, 此方法不等待先前提交的任务完成执行, 可使用 {@link #awaitTermination awaitTermination} 来做到这一点
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();

        // 转换线程池状态为SHUTDOWN
        advanceRunState(SHUTDOWN);

        // 中断可能正在等待任务的线程, 不止中断一个线程, 而是所有空闲的工作线程集合
        interruptIdleWorkers();

        // ScheduledThreadPoolExecutor 的钩子
        onShutdown(); // hook for ScheduledThreadPoolExecutor
    } finally {
        mainLock.unlock();
    }

    // 尝试线程池状态转换: STOP/SHUTDOWN -> 中断其中一个线程 -> TIDYING(工作线程数为0) -> TERMINATED状态 -> 通知所有等待线程池主锁Condition的线程
    tryTerminate();
}

// 如果有安全管理器，请确保调用者通常有权关闭线程（请参阅 shutdownPerm）。 如果这通过，还要确保调用者可以中断每个工作线程。
private void checkShutdownAccess() {
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkPermission(shutdownPerm);
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            for (Worker w : workers)
                security.checkAccess(w.thread);
        } finally {
            mainLock.unlock();
        }
    }
}
```

###### shutdownNow

```java
// 尝试停止所有正在执行的任务, 停止等待任务的处理, 并返回等待执行的任务列表, 此方法不会等待主动执行的任务终止, 可以使用 {@link #awaitTermination awaitTermination} 来做到这一点
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();

        // 转换线程池状态为STOP
        advanceRunState(STOP);

        // 中断所有线程，即使是活动线程。 忽略 SecurityExceptions（在这种情况下，某些线程可能保持不间断）。
        interruptWorkers();

        // 将任务队列排到一个新列表中
        tasks = drainQueue();
    } finally {
        mainLock.unlock();
    }

    // 尝试线程池状态转换: STOP/SHUTDOWN -> 中断其中一个线程 -> TIDYING(工作线程数为0) -> TERMINATED状态 -> 通知所有等待线程池主锁Condition的线程
    tryTerminate();

    // 返回所有排出的任务
    return tasks;
}
```

##### Worker

```java
// 线程工人, 实现Runnable本身可以作为一个任务, 实现AQS以简化获取和释放围绕每个任务执行的锁, 实现了一个简单的不可重入互斥锁
private final class Worker extends AbstractQueuedSynchronizer implements Runnable {
    
	final Thread thread;// 该工人正在运行的线程, 如果工厂生产线程失败, 则为null
    Runnable firstTask;// 该工人要运行的初始任务, 可能会为空
    volatile long completedTasks;// worker已完成任务计数器
    
    // 注意! 在构造Worker时, 使用了当前worker实例作为Thread#Runnable实例变量, 如果运行的目标任务不为null, 则调用Runnable#run方法
    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker 禁止中断直到 runWorker
        this.firstTask = firstTask;

        // 注意! 在构造Worker时, 使用了当前worker实例作为Thread#Runnable实例变量, 如果运行的目标任务不为null, 则调用Runnable#run方法
        this.thread = getThreadFactory().newThread(this);
    }
}
```

###### 核心方法 - run

```java
// 指定当前工人来运行任务: 先获取firstTask -> 从任务队列中获取任务 -> beforeExecute -> 运行获取到的任务 -> afterExecute -> 线程运行后的清理工作processWorkerExit
public void run() {
    runWorker(this);
}
```

###### 核心方法 - runWorker

```java
// 使用指定工人运行任务: 先获取firstTask -> 从任务队列中获取任务 -> beforeExecute -> 运行获取到的任务 -> afterExecute -> 线程运行后的清理工作processWorkerExit
final void runWorker(Worker w) {
    // 获取当前线程wt， 用于运行beforeExecute方法
    Thread wt = Thread.currentThread();

    // 获取要运行的初始任务task
    Runnable task = w.firstTask;
    w.firstTask = null;

    // 先释放锁, 同步器状态从-1更改为0, 允许中断当前线程
    w.unlock(); // allow interrupts 允许中断

    // worker需要突然死亡
    boolean completedAbruptly = true;
    try {
        // 执行阻塞或定时等待任务, 如果需要淘汰线程, 则使用存活时间定时获取任务, 在获取不到时则标记超时等待下一轮清空多余线程; 如果不需要淘汰线程, 则阻塞获取任务
        // 通过worker里的线程启动后, 自旋获取任务队列中的任务, 实现线程复用!!! 通过存活时间、核心线程与任务队列, 控制资源消耗
        while (task != null || (task = getTask()) != null) {
            // 如果任务不为空, 则获取worker锁, 设置当前线程为独占线程
            w.lock();

            // 如果运行状态为停止(不接受新任务)、整理(任务终止)、终止状态(已完成), 且线程已被中断, 但当前线程中断标记位不为true, 则中断当前线程
            if ((runStateAtLeast(ctl.get(), STOP) || (Thread.interrupted() && runStateAtLeast(ctl.get(), STOP))) && !wt.isInterrupted())
                // 中断当前线程, 如果从Thread其他实例方法调用该方法, 则会清除中断状态, 然后会收到一个{@link InterruptedException}
                wt.interrupt();
            try {
                // 执行任务前的钩子方法, 用于给子类实现回调
                beforeExecute(wt, task);

                // 运行任务
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    // 执行任务后的钩子方法, 用于给子类实现回调, 该方法由执行任务的线程调用, t为执行任务期间抛出的Throwable
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;

                // 更新worker的任务完成数
                w.completedTasks++;

                // 释放锁, 同步器状态从1更改为0
                w.unlock();
            }
        }

        // worker需要不突然死亡
        completedAbruptly = false;
    } finally {
        // 从工作线程集中删除线程, 并且可能会终止池或替换工作线程, 当指定的completedAbruptly为true时, 会先减少ctl工作线程数并替换工作线程
        processWorkerExit(w, completedAbruptly);
    }
}
```

###### 获取锁方法

```java
// 快速失败式获取锁, 只会将0 CAS更新为1(unused没用), CAS成功则设置当前线程为独占线程, 并返回true, 代表获取成功, 否则返回false, 代表获取失败
protected boolean tryAcquire(int unused) {
    if (compareAndSetState(0, 1)) {
        setExclusiveOwnerThread(Thread.currentThread());
        return true;
    }
    return false;
}

// 阻塞式获取锁, 快速失败失败还会进去AQS队列中排队
public void lock()        { acquire(1); }

// 快速失败式获取锁, 只会将0 CAS更新为1(unused没用), CAS成功则设置当前线程为独占线程, 并返回true, 代表获取成功, 否则返回false, 代表获取失败
public boolean tryLock()  { return tryAcquire(1); }
```

###### 释放锁方法

```java
// 释放锁, 只会将同步状态设置为0, 并清空独占线程以及返回true, 代表释放成功
protected boolean tryRelease(int unused) {
    setExclusiveOwnerThread(null);
    setState(0);
    return true;
}

// 释放锁, 只会将同步状态设置为0, 并清空独占线程以及返回true, 代表释放成功
public void unlock()      { release(1); }
```

###### 其他监控方法

```java
// 是否存在锁独占线程
protected boolean isHeldExclusively() {
    return getState() != 0;
}

// 判断worker是否会阻塞, 通过判断是否存在独占线程来判断
public boolean isLocked() { return isHeldExclusively(); }
```

###### 强制中断方法

```java
// 启动时中断
void interruptIfStarted() {
    Thread t;

    // 如果同步器状态大于0, 且t线程没有被中断, 则中断t线程
    if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
        try {
            t.interrupt();
        } catch (SecurityException ignore) {
        }
    }
}
```

##### Worker控制方法

###### interruptWorkers

```java
// 中断所有线程，即使是活动线程。 忽略 SecurityExceptions（在这种情况下，某些线程可能保持不间断）
private void interruptWorkers() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers)
            w.interruptIfStarted();
    } finally {
        mainLock.unlock();
    }
}
```

###### interruptIdleWorkers

```java
private static final boolean ONLY_ONE = true;

// 中断可能正在等待任务的线程, 不止中断一个线程, 而是所有空闲的工作线程集合
private void interruptIdleWorkers() {
    interruptIdleWorkers(false);
}
```

###### interruptIdleWorkers(boolean)

```java
// 中断可能正在等待任务的线程, 如果指定onlyOne, 则只中断其中一个线程
private void interruptIdleWorkers(boolean onlyOne) {
    // 获取线程池主锁
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 遍历工作线程集合
        for (Worker w : workers) {
            // 获取工人w中的线程t
            Thread t = w.thread;

            // 如果线程t没有被中断, 且获取工人w中的不可重入锁成功(快速失败获取, 如果获取到说明线程空闲需要被中断, 否则说明线程繁忙不需要被中断), 则中断t线程
            if (!t.isInterrupted() && w.tryLock()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                } finally {
                    // 释放工人w中的不可重入锁
                    w.unlock();
                }
            }

            // 如果只中断一个线程, 则退出遍历
            if (onlyOne)
                break;
        }
    } finally {
        // 释放线程池主锁
        mainLock.unlock();
    }
}
```

##### 核心工作方法

###### execute

```java
// 在未来的某个时间执行给定的任务, 任务可以在新线程或现有池线程中执行, 如果任务无法提交执行, 则该任务由当前 {@code RejectedExecutionHandler} 来处理
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();

    // 获取ctl控制位c
    int c = ctl.get();

    // 根据控制位ctl获取工作线程数, 如果工作线程数小于核心线程数
    if (workerCountOf(c) < corePoolSize) {
        // 则检查是否可以根据当前池状态和给定界限（核心或最大值）添加新的工作线程, 如果添加工作线程成功, 则返回true; 如果添加工作线程失败, 则回滚工作线程并返回false
        if (addWorker(command, true))
            // 如果启动工作线程成功, 则直接返回
            return;

        // 如果启动工作线程失败, 则重新获取ctl控制位c
        c = ctl.get();
    }

    // 如果线程池仍为运行状态, 则往任务队列填充任务command
    if (isRunning(c) && workQueue.offer(command)) {
        // 如果任务填充成功, 则再获取ctl控制位recheck
        int recheck = ctl.get();

        // 如果此时线程池不为运行状态, 则从执行程序的内部队列中删除该任务，从而导致它在尚未启动时无法运行
        if (! isRunning(recheck) && remove(command))
            // 如果删除成功, 则履行任务command和当前任务执行者executor的拒绝策略
            reject(command);
        // 如果此时线程池仍为运行状态, 但工作线程数为0, 则检查是否可以根据当前池状态和给定界限（核心或最大值）添加新的工作线程, 如果添加工作线程成功, 则返回true; 如果添加工作线程失败, 则回滚工作线程并返回false
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 如果线程池不为运行状态, 或者往任务队列填充任务command失败, 则检查是否可以根据当前池状态和给定界限（核心或最大值）添加新的工作线程, 如果添加工作线程成功, 则返回true; 如果添加工作线程失败, 则回滚工作线程并返回false
    else if (!addWorker(command, false))
        // 如果worker添加失败, 则履行任务command和当前任务执行者executor的拒绝策略
        reject(command);
}
```

###### addWorker

```java
// 检查是否可以根据当前池状态和给定界限（核心或最大值）添加新的工作线程, 如果添加工作线程成功, 则返回true; 如果添加工作线程失败, 则回滚工作线程并返回false
private boolean addWorker(Runnable firstTask, boolean core) {
    // 重试点retry
    retry:

    // 开始自旋, 先判断线程状态是否适合增加线程数, 再自旋增加ctl工作线程数
    for (;;) {
        // 获取ctl状态控制位c, 运行状态rs
        int c = ctl.get();
        int rs = runStateOf(c);

        // 仅在必要时检查队列是否为空。
        // Check if queue empty only if necessary.
        // 如果运行状态不为RUNNING状态
        if (rs >= SHUTDOWN &&
            // 如果不为SHUTDOWN状态, 或者为SHUTDOWN状态但指定了任务(因为此时不接受新的任务), 或者任务队列为非空时, 则返回false, 代表不需要添加新线程
            ! (rs == SHUTDOWN && firstTask == null && ! workQueue.isEmpty()))
            return false;

        // 如果状态为运行状态, 或者为SHUTDOWN状态状态但任务队列不为空, 则再次开始自旋, 开始增加ctl工作线程数
        for (;;) {
            // 根据ctl状态控制位c获取工作线程数wc
            int wc = workerCountOf(c);

            // 如果工作线程数wc已经大于最大线程容量(5亿)
            if (wc >= CAPACITY ||
                // 或者如果wc已经大于核心线程数或者最大线程数
                wc >= (core ? corePoolSize : maximumPoolSize))
                // 则返回false, 表示不需要添加新线程
                return false;

            // 如果需要添加新线程, 则CAS递增ctl工作线程数
            if (compareAndIncrementWorkerCount(c))
                // 如果CAS更新成功, 则退出最外层的自旋
                break retry;

            // 如果CAS更新失败, 则重新获取ctl状态控制位
            c = ctl.get();  // Re-read ctl 重读ctl

            // 如果运行状态发生了变化, 为防止线程池已关闭, 则重试最外层循环
            if (runStateOf(c) != rs)
                continue retry;

            // 否则 CAS 由于 workerCount 变化而失败； 重试内循环
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    // 更新完ctl工作线程数后, 真正开始创建新的worker
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        // 使用给定的第一个任务和来自 ThreadFactory 的线程创建
        // 注意! 在构造Worker时, 使用了当前worker实例作为Thread#Runnable实例变量, 如果运行的目标任务不为null, 则调用Runnable#run方法
        w = new Worker(firstTask);

        // 获取工人w中的线程t
        final Thread t = w.thread;

        // 如果t不为空, 则获取线程池主锁
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // 获取状态控制位ctl的运行状态rs
                int rs = runStateOf(ctl.get());

                // 如果rs为RUNNING状态 或者 为SHUTDOWN状态且firstTask为空, 则检查t线程是否已经启动, 如果已经启动则抛出异常
                if (rs < SHUTDOWN || (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable 预先检查 t 是否可启动
                        throw new IllegalThreadStateException();

                    // 如果t线程仍未启动, 说明线程t有效, 则将工人w添加到工作线程集合中
                    workers.add(w);

                    // 获取工作线程集合中大小s
                    int s = workers.size();

                    // 如果s 大于 能跟踪到的最大线程数, 则更新能跟踪到的最大线程数为s, 即目前为止, 线程池中出现的最大线程数
                    if (s > largestPoolSize)
                        largestPoolSize = s;

                    // 最后更新worker已添加成功
                    workerAdded = true;
                }
                // 如果rs不为RUNNING状态, 当为SHUTDOWN状态且还提交了任务firstTask, 则不做任何操作
            } finally {
                // 释放线程池主锁
                mainLock.unlock();
            }

            // 如果worker成功添加, 则启动t线程, 并更新worker已启动
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        // 如果最后worker仍未启动, 则回滚工作线程: 从工作线程集合中移除、减少ctl工作线程数、尝试线程池状态转换
        if (! workerStarted)
            addWorkerFailed(w);
    }

    // 返回worker是否启动成功
    return workerStarted;
}
```

###### addWorkerFailed

```java
// 回滚工作线程: 从工作线程集合中移除、减少ctl工作线程数、尝试线程池状态转换
private void addWorkerFailed(Worker w) {
    // 获取线程池主锁
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 如果需要回滚的线程不为空, 则从工作线程集合中移除
        if (w != null)
            workers.remove(w);

        // 减少一个工作线程, 底层依赖自旋+CAS递减工作线程数, 递减成功则退出自旋, 否则继续自旋
        decrementWorkerCount();

        // 尝试线程池状态转换: STOP/SHUTDOWN -> 中断其中一个线程 -> TIDYING(工作线程数为0) -> TERMINATED状态 -> 通知所有等待线程池主锁Condition的线程
        tryTerminate();
    } finally {
        // 释放线程池主锁
        mainLock.unlock();
    }
}
```

###### reject

```java
// 履行任务command和当前任务执行者executor的拒绝策略
final void reject(Runnable command) {
    handler.rejectedExecution(command, this);
}
```

###### processWorkerExit

```java
// 从工作线程集中删除线程, 并且可能会终止池或替换工作线程, 当指定的completedAbruptly为true时, 会先减少ctl工作线程数并替换工作线程
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    // 如果worker需要突然死亡, 则减少一个ctl中的工作线程, 底层依赖自旋+CAS递减工作线程数, 递减成功则退出自旋, 否则继续自旋
    if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted // 如果突然，则 workerCount 未调整
        decrementWorkerCount();

    // 获取线程池主锁
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 累加worker任务完成数作为线程池任务完成数
        completedTaskCount += w.completedTasks;

        // 将w工人从worker集合中移除
        workers.remove(w);
    } finally {
        // 释放线程池主锁
        mainLock.unlock();
    }

    // 尝试线程池状态的转换: STOP/SHUTDOWN -> 中断其中一个线程 -> TIDYING(工作线程数为0) -> TERMINATED状态 -> 通知所有等待线程池主锁Condition的线程
    tryTerminate();

    // 获取状态控制位ctl
    int c = ctl.get();

    // 如果状态为RUNNING或者SHUTDOWN
    if (runStateLessThan(c, STOP)) {
        // 如果之前没有减少过ctl的工作线程数
        if (!completedAbruptly) {
            // 获取最少线程数
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;

            // 如果最少线程数为0(此时肯定设置了核心线程数的过期时间), 同时任务队列不为空, 则至少保留一个线程
            if (min == 0 && ! workQueue.isEmpty())
                min = 1;

            // 如果工作线程数大于等于最少线程数, 则直接返回即可, 代表不需要更换worker(此时ctl)
            if (workerCountOf(c) >= min)
                return; // replacement not needed // 不需要更换
        }

        // 如果worker之前已经突然死亡了, 则检查是否可以根据当前池状态和给定界限（核心或最大值）添加新的工作线程, 如果添加工作线程成功, 则返回true; 如果添加工作线程失败, 则回滚工作线程并返回false
        addWorker(null, false);
    }
}
```

###### getTask

```java
// 执行阻塞或定时等待任务, 如果需要淘汰线程, 则使用存活时间定时获取任务, 在获取不到时则标记超时等待下一轮清空多余线程; 如果不需要淘汰线程, 则阻塞获取任务
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out? // 上次 poll() 是否超时？

    // 开始自旋
    for (;;) {
        // 获取ctl状态控制位c, 运行状态rs
        int c = ctl.get();
        int rs = runStateOf(c);

        // 仅在必要时检查队列是否为空。
        // Check if queue empty only if necessary.
        // 如果运行状态为停止(不接受新任务)、整理(任务终止)、终止状态(已完成), 或者任务队列为空
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            // 减少一个工作线程, 底层依赖自旋+CAS递减工作线程数, 递减成功则退出自旋, 否则继续自旋
            decrementWorkerCount();

            // 返回null, 表示当前线程必须退出
            return null;
        }

        // 根据状态控制位c获取工作线程数wc
        int wc = workerCountOf(c);

        // 工人会被淘汰吗？
        // Are workers subject to culling?
        // 如果允许核心线程超时, 或者工作线程数大于核心线程数(即存在非核心线程时)， 则认为可能有线程需要淘汰
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        // 如果(工作线程数大于最大线程数 或者 有线程需要淘汰) 同时 (工作线程数至少为1 或者 任务队列为空), 则CAS递减工作线程数
        if ((wc > maximumPoolSize || (timed && timedOut)) && (wc > 1 || workQueue.isEmpty())) {
            // 如果CAS成功, 则返回null, 表示当前线程必须退出
            if (compareAndDecrementWorkerCount(c))
                return null;

            // 如果CAS失败, 则继续自旋判断
            continue;
        }

        // 当前线程数正常
        try {
            Runnable r = timed ?
                    // 如果有线程需要淘汰(大于核心数 或者 核心数需要超时), 则根据存活时间定时从任务队列获取任务r
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    // 如果没有线程需要淘汰[!(大于核心数 或者 核心数需要超时)], 则阻塞获取任务队列中的任务r
                    workQueue.take();

            // 如果获取到任务r, 则返回任务r, 表示获取成功
            if (r != null)
                return r;

            // 如果没获取到任务r, 则标识timedOut为true, 代表获取超时(由于线程争抢激烈导致, 此时设置超时会在下轮自旋时进行清空多余线程)
            timedOut = true;
        } catch (InterruptedException retry) {
            // 如果定时获取期间抛出异常, 则任务还没超时
            timedOut = false;
        }
    }
}
```

###### awaitTermination

```java
// 阻塞直到所有任务在关闭请求后完成执行，或发生超时，或当前线程被中断，以先发生者为准。
public boolean awaitTermination(long timeout, TimeUnit unit)
    throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 开始自旋
        for (;;) {
            // 如果线程池为TERMINATED状态, 则返回true, 代表等待成功
            if (runStateAtLeast(ctl.get(), TERMINATED))
                return true;

            // 如果发生超时, 则返回false, 代表等待失败
            if (nanos <= 0)
                return false;

            // 如果没发生超时, 则调用主锁的Condition#awaitNanos阻塞当前线程nanos时间, 以方便其他线程执行完任务再中断它们的线程
            nanos = termination.awaitNanos(nanos);
        }
    } finally {
        mainLock.unlock();
    }
}
```

##### 其他辅助方法

###### setThreadFactory

```java
// 设置用于创建新线程的线程工厂
public void setThreadFactory(ThreadFactory threadFactory) {
    if (threadFactory == null)
        throw new NullPointerException();
    this.threadFactory = threadFactory;
}
```

###### getThreadFactory

```java
// 返回用于创建新线程的线程工厂
public ThreadFactory getThreadFactory() {
    return threadFactory;
}
```

###### setRejectedExecutionHandler

```java
// 为无法执行的任务设置新的处理程序
public void setRejectedExecutionHandler(RejectedExecutionHandler handler) {
    if (handler == null)
        throw new NullPointerException();
    this.handler = handler;
}
```

###### getRejectedExecutionHandler

```java
// 返回不可执行任务的当前处理程序
public RejectedExecutionHandler getRejectedExecutionHandler() {
    return handler;
}
```

###### setCorePoolSize

```java
// 设置核心线程数, 这会覆盖构造函数中设置的任何值, 如果属于线程减少, 则中断空闲线程; 如果属于线程增加, 则创建增量个数或者任务队列实际长度数的线程
public void setCorePoolSize(int corePoolSize) {
    if (corePoolSize < 0)
        throw new IllegalArgumentException();

    // 获取增量delta
    int delta = corePoolSize - this.corePoolSize;
    this.corePoolSize = corePoolSize;

    // 如果属于线程减少, 则中断可能正在等待任务的线程, 不止中断一个线程, 而是所有空闲的工作线程集合
    if (workerCountOf(ctl.get()) > corePoolSize)
        interruptIdleWorkers();
    // 如果属于线程增加
    else if (delta > 0) {
        // 则从增量delta和任务队列实际大小中取最小值k
        int k = Math.min(delta, workQueue.size());

        // 创建k个工作线程, 直到k为0, 或者线程添加失败, 或者任务队列为空
        while (k-- > 0 && addWorker(null, true)) {
            if (workQueue.isEmpty())
                break;
        }
    }
}
```

###### getCorePoolSize

```java
// 返回核心线程数
public int getCorePoolSize() {
    return corePoolSize;
}
```

###### prestartCoreThread

```java
// 启动一个核心线程，使其空闲等待工作, 这将覆盖仅在执行新任务时才启动核心线程的默认策略
public boolean prestartCoreThread() {
    return workerCountOf(ctl.get()) < corePoolSize && addWorker(null, true);
}
```

###### ensurePrestart

```java
// ScheduledThreadPoolExecutor调用, 无论如何(即使核心线程数为0)也会启动一个核心线程, 使其空闲等待工作, 这将覆盖仅在执行新任务时才启动核心线程的默认策略
void ensurePrestart() {
    int wc = workerCountOf(ctl.get());
    if (wc < corePoolSize)
        addWorker(null, true);
    else if (wc == 0)
        addWorker(null, false);
}
```

###### prestartAllCoreThreads

```java
// 启动所有核心线程，导致它们空闲等待工作, 这将覆盖仅在执行新任务时才启动核心线程的默认策略
public int prestartAllCoreThreads() {
    int n = 0;
    while (addWorker(null, true))
        ++n;
    return n;
}
```

###### allowsCoreThreadTimeOut

```java
// 判断核心线程是否会超时
public boolean allowsCoreThreadTimeOut() {
    return allowCoreThreadTimeOut;
}
```

###### allowCoreThreadTimeOut(boolean)

```java
// 是否允许核心线程过期, 如果允许核心线程过期, 则空闲线程存活时间必须大于0, 通常应在主动使用池之前调用此方法
public void allowCoreThreadTimeOut(boolean value) {
    // 如果允许核心线程过期, 则空闲线程存活时间必须大于0
    if (value && keepAliveTime <= 0)
        throw new IllegalArgumentException("Core threads must have nonzero keep alive times");

    // 更新允许标志, 如果允许过期, 则中断可能正在等待任务的线程, 不止中断一个线程, 而是所有空闲的工作线程集合
    if (value != allowCoreThreadTimeOut) {
        allowCoreThreadTimeOut = value;
        if (value)
            interruptIdleWorkers();
    }
}
```

###### setMaximumPoolSize

```java
// 设置允许的最大线程数, 这会覆盖构造函数中设置的任何值, 如果新值小于当前值, 则多余的现有线程将在下一次空闲时终止
public void setMaximumPoolSize(int maximumPoolSize) {
    if (maximumPoolSize <= 0 || maximumPoolSize < corePoolSize)
        throw new IllegalArgumentException();
    this.maximumPoolSize = maximumPoolSize;

    // 如果属于线程减少, 则中断可能正在等待任务的线程, 不止中断一个线程, 而是所有空闲的工作线程集合
    if (workerCountOf(ctl.get()) > maximumPoolSize)
        interruptIdleWorkers();
}
```

###### getMaximumPoolSize

```java
// 返回允许的最大线程数
public int getMaximumPoolSize() {
    return maximumPoolSize;
}
```

###### setKeepAliveTime

```java
// 设置空闲线程存活时间, 即如果线程池中的线程数超过核心数, 在等待存活时间得不到任务处理, 则会终止这些多余的线程
public void setKeepAliveTime(long time, TimeUnit unit) {
    if (time < 0)
        throw new IllegalArgumentException();
    if (time == 0 && allowsCoreThreadTimeOut())
        throw new IllegalArgumentException("Core threads must have nonzero keep alive times");
    long keepAliveTime = unit.toNanos(time);
    long delta = keepAliveTime - this.keepAliveTime;
    this.keepAliveTime = keepAliveTime;

    // 如果属于存活时间减少, 则中断可能正在等待任务的线程, 不止中断一个线程, 而是所有空闲的工作线程集合
    if (delta < 0)
        interruptIdleWorkers();
}
```

###### getKeepAliveTime

```java
// 返回线程保持活动时间, 这是超过核心池大小的线程在终止之前可能保持空闲的时间量
public long getKeepAliveTime(TimeUnit unit) {
    return unit.convert(keepAliveTime, TimeUnit.NANOSECONDS);
}
```

###### getQueue

```java
// 获取线程池所使用的任务队列, 该方法主要用于调试和监控
public BlockingQueue<Runnable> getQueue() {
    return workQueue;
}
```

###### remove

```java
// 如果此任务存在，则从执行程序的内部队列中删除该任务，从而导致它在尚未启动时无法运行
public boolean remove(Runnable task) {
    boolean removed = workQueue.remove(task);

    // 尝试线程池状态转换: STOP/SHUTDOWN -> 中断其中一个线程 -> TIDYING(工作线程数为0) -> TERMINATED状态 -> 通知所有等待线程池主锁Condition的线程
    tryTerminate(); // In case SHUTDOWN and now empty // 如果 SHUTDOWN 现在为空

    // 返回删除结果
    return removed;
}
```

###### purge

```java
// 尝试从工作队列中删除所有已取消的 {@link Future} 任务, 取消的任务永远不会执行, 但可能会在工作队列中累积, 直到工作线程可以主动删除它们
public void purge() {
    final BlockingQueue<Runnable> q = workQueue;
    try {
        // 迭代任务队列, 如果Future标记为已取消, 则从任务队列中移除掉
        Iterator<Runnable> it = q.iterator();
        while (it.hasNext()) {
            Runnable r = it.next();
            if (r instanceof Future<?> && ((Future<?>)r).isCancelled())
                it.remove();
        }
    } catch (ConcurrentModificationException fallThrough) {
        // 如果我们在遍历过程中遇到干扰，请走慢路径。为遍历制作副本并为取消的条目调用remove 。慢路径更有可能是 O(N*N)。
        // Take slow path if we encounter interference during traversal.
        // Make copy for traversal and call remove for cancelled entries.
        // The slow path is more likely to be O(N*N).
        for (Object r : q.toArray())
            if (r instanceof Future<?> && ((Future<?>)r).isCancelled())
                q.remove(r);
    }

    // 尝试线程池状态转换: STOP/SHUTDOWN -> 中断其中一个线程 -> TIDYING(工作线程数为0) -> TERMINATED状态 -> 通知所有等待线程池主锁Condition的线程
    tryTerminate(); // In case SHUTDOWN and now empty // 如果 SHUTDOWN 现在为空
}
```

###### drainQueue

```java
// 将任务队列排到一个新列表中
private List<Runnable> drainQueue() {
    BlockingQueue<Runnable> q = workQueue;
    ArrayList<Runnable> taskList = new ArrayList<Runnable>();
    q.drainTo(taskList);
    if (!q.isEmpty()) {
        for (Runnable r : q.toArray(new Runnable[0])) {
            if (q.remove(r))
                taskList.add(r);
        }
    }
    return taskList;
}
```

###### finalize

```java
// 线程池析构函数
protected void finalize() {
    // 启动有序关闭，其中执行先前提交的任务，但不会接受新任务, 此方法不等待先前提交的任务完成执行, 可使用 {@link #awaitTermination awaitTermination} 来做到这一点
    shutdown();
}
```

##### 其他监控方法

###### ctlOf

```java
// 根据运行状态和工作线程数组装控制位ctl
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

###### runStateOf

```java
// 根据控制位c获取运行状态: 与高3位(全1)相与, 得到高3位的值, 作为运行状态
private static int runStateOf(int c)     { return c & ~CAPACITY; }
```

###### workerCountOf

```java
// 根据控制位c获取工作线程数: 与低29位(全1)相与, 得到低29位的值, 作为工作线程数
private static int workerCountOf(int c)  { return c & CAPACITY; }
```

###### runStateLessThan

```java
// 判断控制位c是否小于某状态s
private static boolean runStateLessThan(int c, int s) {
    return c < s;
}
```

###### runStateAtLeast

```java
// 判断控制位c是否大于等于某状态s
private static boolean runStateAtLeast(int c, int s) {
    return c >= s;
}
```

###### isRunning

```java
// 判断控制位c是否小于SHUTDOWN(0), 即是否为运行状态
private static boolean isRunning(int c) {
    return c < SHUTDOWN;
}
```

###### isRunningOrShutdown

```java
// 如果线程池为运行状态, 或者为关闭状态且已经允许关闭了, 则返回true
final boolean isRunningOrShutdown(boolean shutdownOK) {
    int rs = runStateOf(ctl.get());
    return rs == RUNNING || (rs == SHUTDOWN && shutdownOK);
}
```

###### isShutdown

```java
// 判断线程池是否已经关闭
public boolean isShutdown() {
    return !isRunning(ctl.get());
}
```

###### isTerminating

```java
// 如果此执行程序在 {@link #shutdown} 或 {@link #shutdownNow} 之后正在终止但尚未完全终止，则返回 true。
public boolean isTerminating() {
    int c = ctl.get();
    return ! isRunning(c) && runStateLessThan(c, TERMINATED);
}
```

###### isTerminated

```java
// 判断线程池是否为TERMINATED状态
public boolean isTerminated() {
    return runStateAtLeast(ctl.get(), TERMINATED);
}
```

###### getPoolSize

```java
// 返回池中的当前线程数, 如果线程池状态为TIDYING或者TERMINATED, 则返回0; 否则返回工作线程集合的实际大小
public int getPoolSize() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 删除 isTerminated() && getPoolSize() > 0 的罕见和令人惊讶的可能性
        // Remove rare and surprising possibility of
        // isTerminated() && getPoolSize() > 0

        // 如果线程池状态为TIDYING或者TERMINATED, 则返回0; 否则返回工作线程集合的实际大小
        return runStateAtLeast(ctl.get(), TIDYING) ? 0 : workers.size();
    } finally {
        mainLock.unlock();
    }
}
```

###### getActiveCount

```java
// 返回正在积极执行任务的线程的大致数量, 通过判断工人worker是否持有锁来判断
public int getActiveCount() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        int n = 0;
        for (Worker w : workers)
            if (w.isLocked())
                ++n;
        return n;
    } finally {
        mainLock.unlock();
    }
}
```

###### getLargestPoolSize

```java
// 返回到目前为止, 线程池中出现的最大线程数
public int getLargestPoolSize() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        return largestPoolSize;
    } finally {
        mainLock.unlock();
    }
}
```

###### getTaskCount

```java
// 返回已安排执行的大致任务总数 = 加锁的工人worker数 + 任务队列的实际大小
public long getTaskCount() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        long n = completedTaskCount;
        for (Worker w : workers) {
            n += w.completedTasks;
            if (w.isLocked())
                ++n;
        }
        return n + workQueue.size();
    } finally {
        mainLock.unlock();
    }
}
```

###### getCompletedTaskCount

```java
// 返回已完成执行的大致任务总数, 累加每个worker完成的任务数
public long getCompletedTaskCount() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        long n = completedTaskCount;
        for (Worker w : workers)
            n += w.completedTasks;
        return n;
    } finally {
        mainLock.unlock();
    }
}
```

##### 钩子方法

###### onShutdown

```java
// 在调用关闭时运行状态转换后执行任何进一步的清理。 此处无操作，但被 ScheduledThreadPoolExecutor 用于取消延迟任务。
void onShutdown() {
}
```

###### beforeExecute

```java
// 执行任务前的钩子方法, 用于给子类实现回调
protected void beforeExecute(Thread t, Runnable r) { }
```

###### afterExecute

```java
// 执行任务后的钩子方法, 用于给子类实现回调, 该方法由执行任务的线程调用, t为执行任务期间抛出的Throwable
protected void afterExecute(Runnable r, Throwable t) { }
```

###### terminated

```java
// tryTerminate方法中, 尝试将线程池状态TIDYING->TERMINATED转换的中间钩子方法
protected void terminated() { }
```

##### 拒绝策略实现

###### CallerRunsPolicy

```java
// 拒绝策略一: 直接在 {@code execute} 方法的调用线程中运行被拒绝的任务, 如果执行程序已关闭, 则会丢弃任务
public static class CallerRunsPolicy implements RejectedExecutionHandler {
    public CallerRunsPolicy() { }

    // 履行拒绝策略: 如果没有线程池没有关闭, 则使用当前线程直接运行任务r
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            r.run();
        }
    }
}
```

###### AbortPolicy

```java
// 拒绝策略二: 抛出RejectedExecutionException异常, 从而拒绝任务的执行
public static class AbortPolicy implements RejectedExecutionHandler {
    public AbortPolicy() { }

    // 总是抛出 RejectedExecutionException
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                                             " rejected from " +
                                             e.toString());
    }
}
```

###### DiscardPolicy

```java
// 拒绝策略三: 不做任何事情, 丢弃所有的任务
public static class DiscardPolicy implements RejectedExecutionHandler {
    public DiscardPolicy() { }

    // 不做任何事情，具有丢弃任务 r 的效果。
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
}
```

###### DiscardOldestPolicy

```java
// 拒绝策略四: 丢弃任何任务最旧的未处理请求, 然后重试 {@code execute}, 如果执行程序已关闭, 则会丢弃任务
public static class DiscardOldestPolicy implements RejectedExecutionHandler {
    public DiscardOldestPolicy() { }

    // 如果线程池没被关闭, 则删除队头任务, 删除后重新执行execute方法
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            e.getQueue().poll();
            e.execute(r);
        }
    }
}
```

#### ScheduledThreadPoolExecutor

##### Delayed

- Delayed，用于标记在**给定延迟**后应作用的对象，其实现必须定义一个{@code compareTo}方法，用于提供与其{@code getDelay}方法一致的排序。

```java
public interface Delayed extends Comparable<Delayed> {
    // 以给定的时间单位返回与此对象关联的剩余延迟
    long getDelay(TimeUnit unit);
}
```

##### ScheduledFuture

- ScheduledFuture，可以**取消的、延迟的、能产生结果的任务**，通常是使用 {@link ScheduledExecutorService} 设置调度任务的结果。

```java
public interface ScheduledFuture<V> extends Delayed, Future<V> {}
```

##### RunnableScheduledFuture

- RunnableScheduledFuture，既是一个{@link Rundable}，也是一个{@link ScheduledFuture}，其延迟成功执行的{@code run}方法，将会导致{@code Future}完成并允许访问其结果。

```java
public interface RunnableScheduledFuture<V> extends RunnableFuture<V>, ScheduledFuture<V> {
    // 判断任务是否为周期性的, 如果是则返回true, 否则返回false; 定期任务可以根据一些时间表重新运行, 而非定期任务只能运行一次
    boolean isPeriodic();
}
```

##### ScheduledExecutorService

- ScheduledExecutorService，继承{@link ExecutorService}接口，可以安排命令在给定的**延迟后运行**，或**定期执行**。
- {@code schedule} 方法，创建具有各种延迟的任务，并返回可用于取消或检查执行的任务对象。
  - 允许0延迟和负延迟（但不是周期），并被视为立即执行的请求。
  - 接受相对延迟和时间段作为参数，而不是绝对时间或日期，其中将表示为 {@link java.util.Date} 的绝对时间，转换为所需的相对时间形式是一件简单的事情，如{@code schedule(task, date.getTime() - System.currentTimeMillis(), TimeUnit.MILLISECONDS)}。
- {@code scheduleAtFixedRate} 和 {@code scheduleWithFixedDelay} 方法，创建和执行定期运行直到被取消的任务。
- {@link Executor#execute(Runnable)} 和 {@link ExecutorService} {@code submit} 方法，提交的命令的调度请求延迟为0。
- {@link Executors} 类为此包中提供的 ScheduledExecutorService 实现提供了方便的工厂方法。

```java
public interface ScheduledExecutorService extends ExecutorService {
    // 在给定延迟后执行一次性的command任务
    public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);

    // 在给定延迟后执行一次性的command任务
    public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit);
    
    // (相对周期)创建并执行一个周期性动作, 任务会在initialDelay后执行, 之后则以period为周期执行, 如果遇到异常, 则会终止后续执行; 如果执行时间超过period, 后续执行则会延后而不是并发执行
    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit);
    
    // (绝对周期)创建并执行一个周期性动作, 任务会在initialDelay后执行, 任务执行结束之后则延迟delay后再次执行, 任务执行慢时可能会有并发执行
    public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit);
}
```

![1629513070975](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1629513070975.png)

##### 特点

- ScheduledThreadPoolExecutor，一个 {@link ThreadPoolExecutor} 子类实现，可以额外安排命令在给定**延迟后运行**，或**定期执行**。
  - 当需要多个工作线程时，或者需要 {@link ThreadPoolExecutor}（此类扩展）的额外灵活性或功能时，此类比 {@link java.util.Timer} 更可取。
  - 虽然ScheduledThreadPoolExecutor继承自 {@link ThreadPoolExecutor}，但一些继承的调优方法对它没有用。 特别是，因为它充当使用 {@code corePoolSize} 线程和无界队列的固定大小的池，因此对 {@code maximumPoolSize} 的调整没有任何有用的效果。
  - 此外，将 {@code corePoolSize} 设置为0，或使用 {@code allowCoreThreadTimeOut} 几乎从来都不是一个好主意，因为一旦它们有资格运行，这可能会使池没有线程来处理任务。、
- 延迟任务，在启用后立即执行，但没有任何关于启用后何时开始的实时保证，计划执行时间完全相同的任务按提交的先进先出 (FIFO) 顺序启用。
- 当提交的任务在运行之前被取消时，执行会被抑制。默认情况下，此类取消的任务不会自动从工作队列中删除，会直到其延迟结束。虽然这可以实现进一步的检查和监控，但它也可能导致取消任务的无限保留。 
  - 为避免这种情况，请将 {@link #setRemoveOnCancelPolicy} 设置为 {@code true}，这会导致任务在取消时立即从工作队列中删除。
- 通过 {@code scheduleAtFixedRate} 或 {@code scheduleWithFixedDelay} 安排的任务的连续执行不重叠。虽然不同的执行可能由不同的线程执行，但先前执行的效果发生在后续执行的效果之前。
- ScheduledThreadPoolExecutor覆盖了 {@link ThreadPoolExecutor#execute(Runnable) execute} 和 {@link AbstractExecutorService#submit(Runnable) submit} 方法来生成内部{@link ScheduledFuture} 对象来控制每个任务的延迟和调度。
  -  为了保留功能，子类中这些方法的任何进一步覆盖都必须调用超类版本，这有效地禁用了额外的任务自定义。
  - 但是，此类提供了替代的受保护扩展方法 {@code decorateTask}（{@code Runnable} 和 {@code Callable} 各有一个版本），可用于自定义用于执行通过{@code 输入的命令的具体任务类型 执行}、{@code submit}、{@code schedule}、{@code scheduleAtFixedRate} 和 {@code scheduleWithFixedDelay}。
  - 默认情况下，{@code ScheduledThreadPoolExecutor} 使用扩展 {@link FutureTask} 的任务类型。
- ScheduledThreadPoolExecutor专门用于 ThreadPoolExecutor实现，其不同点主要在于：
  - **使用自定义任务类型 ScheduledFutureTask** ：即使是那些不需要调度的任务（即使用 ExecutorService 执行提交的任务，而不是 ScheduledExecutorService 方法），这些任务被视为延迟为0的延迟任务。
  - **使用自定义队列（DelayedWorkQueue），无界DelayQueue的变种**：与 ThreadPoolExecutor 相比，缺少容量约束以及 corePoolSize 和 maximumPoolSize 实际上相同的事实简化了一些执行机制（请参阅 delayExecute）。
  - **支持可选的run-after-shutdown参数**：这会导致覆盖关闭方法以删除和取消关闭后不应运行的任务，以及当任务（重新）提交与关闭重叠时不同的重新检查逻辑。
  - **允许拦截和检测的任务装饰方法**：这是必需的，因为子类不能以其他方式覆盖提交方法以获得此效果。 不过，这些对池控制逻辑没有任何影响。

##### 构造方法

```java
public class ScheduledThreadPoolExecutor extends ThreadPoolExecutor implements ScheduledExecutorService {
    // 判断是否应该在关闭时取消定期任务, 如果需要取消则为false, 如果不需要取消则为true
    private volatile boolean continueExistingPeriodicTasksAfterShutdown;
    // 判断是否应该在关闭时取消非周期性任务, 如果需要取消则为false, 如果不需要取消则为true
    private volatile boolean executeExistingDelayedTasksAfterShutdown = true;
    // 判断ScheduledFutureTask取消后是否应该从任务队列中删除, 如果需要则为true, 如果不需要则为false
    private volatile boolean removeOnCancel = false;
    // 用于打破调度关系的序列号，进而保证绑定条目之间的 FIFO 顺序
    private static final AtomicLong sequencer = new AtomicLong();
    
    // 使用指定核心线程数、默认线程工厂、默认拒绝策略以及专门的延迟队列, 构造无界最大线程数、无界任务队列的ScheduledThreadPoolExecutor
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue());
    }
    
    // 使用指定核心线程数、指定线程工厂、默认拒绝策略以及专门的延迟队列, 构造无界最大线程数、无界任务队列的ScheduledThreadPoolExecutor
    public ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory threadFactory) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue(), threadFactory);
    }
    
    // 使用指定核心线程数、默认线程工厂、指定拒绝策略以及专门的延迟队列, 构造无界最大线程数、无界任务队列的ScheduledThreadPoolExecutor
    public ScheduledThreadPoolExecutor(int corePoolSize, RejectedExecutionHandler handler) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue(), handler);
    }

    // 使用指定核心线程数、指定线程工厂、指定拒绝策略以及专门的延迟队列, 构造无界最大线程数、无界任务队列的ScheduledThreadPoolExecutor
    public ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory threadFactory, RejectedExecutionHandler handler) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue(), threadFactory, handler);
    }
}
```

##### 获取时间方法

###### now

```java
// 返回当前纳秒时间
final long now() {
    return System.nanoTime();
}
```

###### triggerTime(..，..)

```java
// 返回延迟动作的触发时间
private long triggerTime(long delay, TimeUnit unit) {
    return triggerTime(unit.toNanos((delay < 0) ? 0 : delay));
}
```

###### triggerTime(long)

```java
// 返回延迟动作的触发时间
long triggerTime(long delay) {
    return now() + ((delay < (Long.MAX_VALUE >> 1)) ? delay : overflowFree(delay));
}

// 将队列中所有延迟的值限制在彼此的 Long.MAX_VALUE 之内，以避免在 compareTo 中溢出。如果某个任务有资格出队但尚未出队，则可能会发生这种情况，而其他一些任务已添加，延迟为 Long.MAX_VALUE
private long overflowFree(long delay) {
    Delayed head = (Delayed) super.getQueue().peek();
    if (head != null) {
        long headDelay = head.getDelay(NANOSECONDS);
        if (headDelay < 0 && (delay - headDelay < 0))
            delay = Long.MAX_VALUE + headDelay;
    }
    return delay;
}
```

##### ScheduledFutureTask

```java
private class ScheduledFutureTask<V> extends FutureTask<V> implements RunnableScheduledFuture<V> {
    // 用于打破调度关系的序列号，进而保证绑定条目之间的 FIFO 顺序
    private final long sequenceNumber;
    // 启用任务以纳米时间, 作为单位执行的时间
    private long time;
    // 重复任务的周期(以纳秒为单位), 正值表示固定速率(相对周期)执行, 负值表示固定延迟(绝对周期)执行, 0值表示非重复任务(非周期)
    private final long period;
    // 对于周期任务, 在reExecutePeriodic重新入队的实际任务
    RunnableScheduledFuture<V> outerTask = this;
    // 索引到延迟队列中，以支持更快的取消
    int heapIndex;
    
    // 使用给定的基于nanoTime的触发时间创建一次性动作
    ScheduledFutureTask(Runnable r, V result, long ns) {
        super(r, result);
        this.time = ns;
        this.period = 0;
        this.sequenceNumber = sequencer.getAndIncrement();
    }

    // 创建具有给定纳米时间和周期的周期性动作
    ScheduledFutureTask(Runnable r, V result, long ns, long period) {
        super(r, result);
        this.time = ns;
        this.period = period;
        this.sequenceNumber = sequencer.getAndIncrement();
    }

    // 使用给定的基于nanoTime的触发时间创建一次性动作
    ScheduledFutureTask(Callable<V> callable, long ns) {
        super(callable);
        this.time = ns;
        this.period = 0;
        this.sequenceNumber = sequencer.getAndIncrement();
    }
}
```

###### getDelay

```java
// 以给定的时间单位返回与此对象关联的剩余延迟。
public long getDelay(TimeUnit unit) {
    return unit.convert(time - now(), NANOSECONDS);
}
```

###### compareTo

```java
// Delayed接口继承Comparable接口, 比较延迟甚至序列号
public int compareTo(Delayed other) {
    // 相同对象, 延迟相等
    if (other == this) // compare zero if same object // 如果相同的对象比较零
        return 0;

    // 如果为ScheduledFutureTask, 则继续比较
    if (other instanceof ScheduledFutureTask) {
        ScheduledFutureTask<?> x = (ScheduledFutureTask<?>)other;

        // 比较剩余延迟(纳秒), 如果当前对象的小于other对象的, 则返回-1, 代表小
        long diff = time - x.time;
        if (diff < 0)
            return -1;
        // 如果当前对象的大于other对象的, 则返回1, 代表大
        else if (diff > 0)
            return 1;
        // 如果延迟相等, 则继续比较序列号, 序列号小的则小, 序列号大的则大
        else if (sequenceNumber < x.sequenceNumber)
            return -1;
        else
            return 1;
    }

    // 如果不为ScheduledFutureTask, 则比较剩余延迟(纳秒), 如果当前对象的小于other对象的, 则返回-1, 代表小; 如果当前对象的大于other对象的, 则返回1, 代表大; 否则返回0, 代表相等
    long diff = getDelay(NANOSECONDS) - other.getDelay(NANOSECONDS);
    return (diff < 0) ? -1 : (diff > 0) ? 1 : 0;
}
```

###### 核心方法 - run

```java
// 如果为延迟任务, 则调用父类run方法运行; 如果为周期任务, 则调用runAndReset, 在运行后清空任务的线程、设置下一次运行的时间、以及重新到任务队列中排队
    public void run() {
        boolean periodic = isPeriodic();

        // 如果为周期任务, 且指定了退出不关闭周期任务, 则线程池为SHUTDOWN状态时返回false; 同理, 如果为延迟任务, 且指定了退出不关闭周期任务, 则线程池为SHUTDOWN状态时返回false
        if (!canRunInCurrentRunState(periodic))
            // 如果线程池为SHUTDOWN状态时不需要关闭周期或者延迟线程, 则不用中断FutureTask任务中的线程, 只需要从线程池任务队列中移除即可
            cancel(false);
        // 如果任务可以继续运行, 且为延迟任务时, 则调用父类FutureTask正常运行run方法
        else if (!periodic)
            ScheduledFutureTask.super.run();
        // 如果任务可以继续运行, 且为周期任务时, 则本质上是调用Callable#call()运行目标任务, 不将计算结果设置到outcome中, 但会设置异常结果, 返回true代表本次运行成功且仍然可以继续运行; 返回false代表本次运行失败且不可以继续运行
        else if (ScheduledFutureTask.super.runAndReset()) {
            // 如果本次运行成功且仍然可以继续运行, 则设置下一次运行周期性任务的时间
            setNextRunTime();

            // 周期任务重新排队, 排队后会再次检查是否需要继续运行, 如果需要则会保证一定有一个核心线程在运行, 否则会取消该任务、中断任务的线程
            reExecutePeriodic(outerTask);
        }
    }
}
```

###### isPeriodic

```java
// 判断该任务是否为周期任务, 如果是则返回true, 否则返回false
public boolean isPeriodic() {
    return period != 0;
}
```

###### cancel

```java
// 中断FutureTask任务中的线程, 并从线程池任务队列中移除
public boolean cancel(boolean mayInterruptIfRunning) {
    // 尝试取消此异步计算任务的执行, 如果任务已完成、已被取消或由于其他原因无法取消, 则返回false; 如果此任务在正常完成之前被取消, 则返回{@code true}
    boolean cancelled = super.cancel(mayInterruptIfRunning);

    // 如果此任务存在，则从执行程序的内部队列中删除该任务，从而导致它在尚未启动时无法运行
    if (cancelled && removeOnCancel && heapIndex >= 0)
        remove(this);

    return cancelled;
}
```

###### setNextRunTime

```java
// 设置下一次运行周期性任务的时间
private void setNextRunTime() {
    long p = period;

    // 如果p为正数, 说明为固定速率, 则执行时间time累加周期时间p(相对延迟)
    if (p > 0)
        time += p;
    // 如果p为0代表没有周期, 如果p为负数, 说明为固定延迟, 则当前时间+p延迟(绝对延迟)
    else
        time = triggerTime(-p);
}
```

##### DelayedWorkQueue

![1621781086648](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621781086648.png)

###### 特点

- java.util.concurrent.ScheduledThreadPoolExecutor.DelayedWorkQueue，是一个延迟线程池的内部类。
- DelayedWorkQueue，专门的延迟队列，基于小顶堆的数据结构，这样就消除了在取消时查找任务的需要，极大地加快了清除速度（从O（n）降到O（log n）），并减少了垃圾保留，否则将通过等待元素在清除之前上升到顶部而发生垃圾保留。
- 由于调加到队列中的元素，可能不是ScheduledFutureTasks的RunnableScheduledFuture（底层的Queue是RunnableScheduledFuture类型），所以并不能保证有这样的索引可用。
- **队列操作方法实现与DelayQueue、PriorityBlockingQueue类似**，都是线程安全，且基于小顶堆实现。
- 基于**数组快照方式**的迭代器，但不会以任何特定顺序返回元素，因为最小堆只能保证堆顶最小，而不能保证孩子结点中的顺序。
- 队列没有替换元素的方法，且获取元素方法都是检索栈顶的元素。

###### 数据结构

其实现参考PriorityBlockingQueue，也是最小堆实现。但存储的元素为Runnable元素。

构造方法

- 无参的构造函数：默认的，什么也没做。

```java
static class DelayedWorkQueue extends AbstractQueue<Runnable> implements BlockingQueue<Runnable> {
    
    private static final int INITIAL_CAPACITY = 16;
    private RunnableScheduledFuture<?>[] queue = new RunnableScheduledFuture<?>[INITIAL_CAPACITY];
    private final ReentrantLock lock = new ReentrantLock();
    private int size = 0;
    private Thread leader = null;
    private final Condition available = lock.newCondition();
    
    // 只有一个默认的无参构造器
}
```

###### 迭代方法

- iterator（）：基于**数组快照方式**的迭代器，但不会以任何特定顺序返回元素，因为最小堆只能保证堆顶最小，而不能保证孩子结点中的顺序。

```java
public Iterator<Runnable> iterator() {
    return new Itr(Arrays.copyOf(queue, size));// 基于数组快照方式的迭代器
}
```

###### 扩容方法

- **扩容条件**：获取到主锁、实际大小大于等于当前数组大小时。
- **grow（）**：扩容1.5倍，通过Arrays.copyOf（...）复制到新的数组。

```java
private void grow() {
    int oldCapacity = queue.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1); // grow 50%
    if (newCapacity < 0) // overflow
        newCapacity = Integer.MAX_VALUE;
    queue = Arrays.copyOf(queue, newCapacity);
}
```

##### 核心工作方法

###### decorateTask(Runnable, ..)

```java
// 修改或替换用于执行可运行的任务, 默认实现只是返回给定的任务
protected <V> RunnableScheduledFuture<V> decorateTask(Runnable runnable, RunnableScheduledFuture<V> task) {
    return task;
}
```

###### decorateTask(Callable, ..)

```java
// 修改或替换用于执行可运行的任务, 默认实现只是返回给定的任务
protected <V> RunnableScheduledFuture<V> decorateTask(Callable<V> callable, RunnableScheduledFuture<V> task) {
    return task;
}
```

###### schedule(Runable, .., ..)

```java
// 延迟delay时间执行一次command任务
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit) {
    if (command == null || unit == null)
        throw new NullPointerException();

    // 修改或替换用于执行可运行的任务, 默认实现只是返回ScheduledFutureTask, 没设置周期
    RunnableScheduledFuture<?> t = decorateTask(command, new ScheduledFutureTask<Void>(command, null, triggerTime(delay, unit)));

    // 执行延迟或者周期任务, 在任务队列追加任务后, 需要再次检查线程池以及任务状态, 如果任务还需要继续运行, 则要确保至少还有一个核心线程运; 如果任务不需要继续运行, 则从任务队列中移除并中断任务线程
    delayedExecute(t);
    return t;
}
```

###### schedule(Callable, .., ..)

```java
// 延迟delay时间执行一次callable任务
public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit) {
    if (callable == null || unit == null)
        throw new NullPointerException();

    // 修改或替换用于执行可运行的任务, 默认实现只是返回ScheduledFutureTask, 没设置周期
    RunnableScheduledFuture<V> t = decorateTask(callable, new ScheduledFutureTask<V>(callable, triggerTime(delay, unit)));

    // 执行延迟或者周期任务, 在任务队列追加任务后, 需要再次检查线程池以及任务状态, 如果任务还需要继续运行, 则要确保至少还有一个核心线程运; 如果任务不需要继续运行, 则从任务队列中移除并中断任务线程
    delayedExecute(t);
    return t;
}
```

###### scheduleAtFixedRate(Runable, .., ..)

```java
// (相对周期)创建并执行一个周期性动作, 任务会在initialDelay后执行, 之后则以period为周期执行, 如果遇到异常, 则会终止后续执行; 如果执行时间超过period, 后续执行则会延后而不是并发执行
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit) {
    if (command == null || unit == null)
        throw new NullPointerException();

    // 周期必须大于0
    if (period <= 0)
        throw new IllegalArgumentException();

    // 修改或替换用于执行可运行的任务, 默认实现只是返回ScheduledFutureTask, period作为运行周期(相对周期)
    ScheduledFutureTask<Void> sft = new ScheduledFutureTask<Void>(command, null, triggerTime(initialDelay, unit), unit.toNanos(period));
    RunnableScheduledFuture<Void> t = decorateTask(command, sft);
    sft.outerTask = t;

    // 执行延迟或者周期任务, 在任务队列追加任务后, 需要再次检查线程池以及任务状态, 如果任务还需要继续运行, 则要确保至少还有一个核心线程运; 如果任务不需要继续运行, 则从任务队列中移除并中断任务线程
    delayedExecute(t);
    return t;
}
```

###### scheduleWithFixedDelay(Runable, .., ..)

```java
// (绝对周期)创建并执行一个周期性动作, 任务会在initialDelay后执行, 任务执行结束之后则延迟delay后再次执行, 任务执行慢时可能会有并发执行
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                 long initialDelay,
                                                 long delay,
                                                 TimeUnit unit) {
    if (command == null || unit == null)
        throw new NullPointerException();

    // 延迟必须大于0
    if (delay <= 0)
        throw new IllegalArgumentException();

    // 修改或替换用于执行可运行的任务, 默认实现只是返回ScheduledFutureTask, -delay作为运行周期(绝对周期)
    ScheduledFutureTask<Void> sft = new ScheduledFutureTask<Void>(command, null, triggerTime(initialDelay, unit), unit.toNanos(-delay));
    RunnableScheduledFuture<Void> t = decorateTask(command, sft);
    sft.outerTask = t;

    // 执行延迟或者周期任务, 在任务队列追加任务后, 需要再次检查线程池以及任务状态, 如果任务还需要继续运行, 则要确保至少还有一个核心线程运; 如果任务不需要继续运行, 则从任务队列中移除并中断任务线程
    delayedExecute(t);
    return t;
}
```

###### execute

```java
// 延迟0时间执行一次command任务
public void execute(Runnable command) {
    schedule(command, 0, NANOSECONDS);
}
```

###### submite(Runable)

```java
// 延迟0时间执行一次command任务
public Future<?> submit(Runnable task) {
    return schedule(task, 0, NANOSECONDS);
}
```

###### submit(Runable, T)

```java
// 延迟0时间执行一次command任务
public <T> Future<T> submit(Runnable task, T result) {
    return schedule(Executors.callable(task, result), 0, NANOSECONDS);
}
```

###### submit(Callable)

```java
// 延迟0时间执行一次command任务
public <T> Future<T> submit(Callable<T> task) {
    return schedule(task, 0, NANOSECONDS);
}
```

###### 核心方法 - delayedExecute

```java
// 执行延迟或者周期任务, 在任务队列追加任务后, 需要再次检查线程池以及任务状态, 如果任务还需要继续运行, 则要确保至少还有一个核心线程运; 如果任务不需要继续运行, 则从任务队列中移除并中断任务线程
private void delayedExecute(RunnableScheduledFuture<?> task) {
    // 判断线程池是否已经关闭, 如果线程池已经关闭, 则履行任务command和当前任务执行者executor的拒绝策略
    if (isShutdown())
        reject(task);
    // 如果线程池还没关闭, 则将任务入队
    else {
        super.getQueue().add(task);

        // 再次检查线程池是否已经关闭, 如果已经关闭, 且任务不需要继续运行, 则从任务队列移除任务, 并中断任务线程取消任务的执行
        if (isShutdown() &&
            !canRunInCurrentRunState(task.isPeriodic()) &&
            remove(task))
            task.cancel(false);
        // 还没关闭, 或者任务还需要继续运行, 则确保至少还有一个核心线程运行
        else
            ensurePrestart();
    }
}
```

###### canRunInCurrentRunState

```java
// 如果为周期任务, 且指定了退出不关闭周期任务, 则线程池为SHUTDOWN状态时返回false; 同理, 如果为延迟任务, 且指定了退出不关闭周期任务, 则线程池为SHUTDOWN状态时返回false
boolean canRunInCurrentRunState(boolean periodic) {
    // 如果线程池为运行状态, 或者为关闭状态且已经允许关闭了, 则返回true
    return isRunningOrShutdown(periodic ?
                               continueExistingPeriodicTasksAfterShutdown :
                               executeExistingDelayedTasksAfterShutdown);
}
```

###### reExecutePeriodic

```java
// 周期任务重新排队, 排队后会再次检查是否需要继续运行, 如果需要则会保证一定有一个核心线程在运行, 否则会取消该任务、中断任务的线程
void reExecutePeriodic(RunnableScheduledFuture<?> task) {
    // 如果为周期任务, 且指定了退出不关闭周期任务, 则线程池为SHUTDOWN状态时返回false; 同理, 如果为延迟任务, 且指定了退出不关闭周期任务, 则线程池为SHUTDOWN状态时返回false
    if (canRunInCurrentRunState(true)) {
        // 线程池为SHUTDOWN状态时, 周期任务仍需继续运行, 则往任务队列中添加该周期任务
        super.getQueue().add(task);

        // 再次检查周期任务是否需要运行, 如果不需要继续运行, 且此任务存在，则从执行程序的内部队列中删除该任务，从而导致它在尚未启动时无法运行
        if (!canRunInCurrentRunState(true) && remove(task))
            // 尝试取消此异步计算任务的执行, 如果任务已完成、已被取消或由于其他原因无法取消, 则返回false; 如果此任务在正常完成之前被取消, 则返回{@code true}
            task.cancel(false);
        // 如果还需要继续运行, 则无论如何(即使核心线程数为0)也会启动一个核心线程, 使其空闲等待工作, 这将覆盖仅在执行新任务时才启动核心线程的默认策略
        else
            ensurePrestart();
    }
}
```

##### 其他辅助方法

###### setContinueExistingPeriodicTasksAfterShutdownPolicy

```java
// 设置是否应该在关闭时取消定期任务, 如果需要取消则为false, 如果不需要取消则为true
public void setContinueExistingPeriodicTasksAfterShutdownPolicy(boolean value) {
    continueExistingPeriodicTasksAfterShutdown = value;
    if (!value && isShutdown())
        onShutdown();
}
```

###### getContinueExistingPeriodicTasksAfterShutdownPolicy

```java
// 判断是否应该在关闭时取消定期任务, 如果需要取消则为false, 如果不需要取消则为true
public boolean getContinueExistingPeriodicTasksAfterShutdownPolicy() {
    return continueExistingPeriodicTasksAfterShutdown;
}
```

###### setExecuteExistingDelayedTasksAfterShutdownPolicy

```java
// 设置是否应该在关闭时取消非周期性任务, 如果需要取消则为false, 如果不需要取消则为true
public void setExecuteExistingDelayedTasksAfterShutdownPolicy(boolean value) {
    executeExistingDelayedTasksAfterShutdown = value;
    if (!value && isShutdown())
        onShutdown();
}
```

###### getExecuteExistingDelayedTasksAfterShutdownPolicy

```java
// 判断是否应该在关闭时取消非周期性任务, 如果需要取消则为false, 如果不需要取消则为true
public boolean getExecuteExistingDelayedTasksAfterShutdownPolicy() {
    return executeExistingDelayedTasksAfterShutdown;
}
```

###### setRemoveOnCancelPolicy

```java
// 设置ScheduledFutureTask取消后是否应该从任务队列中删除, 如果需要则为true, 如果不需要则为false
public void setRemoveOnCancelPolicy(boolean value) {
    removeOnCancel = value;
}
```

###### getRemoveOnCancelPolicy

```java
// 判断ScheduledFutureTask取消后是否应该从任务队列中删除, 如果需要则为true, 如果不需要则为false
public boolean getRemoveOnCancelPolicy() {
    return removeOnCancel;
}
```

###### getQueue

```java
public BlockingQueue<Runnable> getQueue() {
    return super.getQueue();
}
```

##### 钩子实现方法

###### onShutdown

```java
// 取消并清除由于关闭策略而不应运行的所有任务的队列。 在 super.shutdown 中调用
@Override
void onShutdown() {
    // 获取任务队列
    BlockingQueue<Runnable> q = super.getQueue();

    // 判断是否应该在关闭时取消非周期性任务, 如果需要取消则为false, 如果不需要取消则为true
    boolean keepDelayed = getExecuteExistingDelayedTasksAfterShutdownPolicy();

    // 判断是否应该在关闭时取消定期任务, 如果需要取消则为false, 如果不需要取消则为true
    boolean keepPeriodic = getContinueExistingPeriodicTasksAfterShutdownPolicy();

    // 如果需要同时取消非周期以及定期任务, 则遍历任务队列, 取消所有任务, 并清空任务队列
    if (!keepDelayed && !keepPeriodic) {
        for (Object e : q.toArray())
            if (e instanceof RunnableScheduledFuture<?>)
                ((RunnableScheduledFuture<?>) e).cancel(false);
        q.clear();
    }
    else {
        // 遍历快照以避免迭代器异常, 如果keepPeriodic或者keepDelayed为false, 则根据isPeriodic来取消并删除任务
        // Traverse snapshot to avoid iterator exceptions
        for (Object e : q.toArray()) {
            if (e instanceof RunnableScheduledFuture) {
                RunnableScheduledFuture<?> t = (RunnableScheduledFuture<?>)e;
                if ((t.isPeriodic() ? !keepPeriodic : !keepDelayed) ||
                    t.isCancelled()) { // also remove if already cancelled // 如果已经取消，也删除
                    if (q.remove(t))
                        t.cancel(false);
                }
            }
        }
    }

    // 尝试线程池状态转换: STOP/SHUTDOWN -> 中断其中一个线程 -> TIDYING(工作线程数为0) -> TERMINATED状态 -> 通知所有等待线程池主锁Condition的线程
    tryTerminate();
}
```