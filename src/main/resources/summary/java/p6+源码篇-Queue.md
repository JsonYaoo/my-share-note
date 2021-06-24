## 源码篇

### Queue

- **特点：**
  - Queue，设计用于在处理之前容纳元素的集合，除Collection操作之外，还提供了其他插入、提取和检查的操作，一种在操作失败时会引发异常，一种是在操作失败时返回特殊值。
  - **队列通常不一定以FIFO（先进先出）的方式对元素进行排序**，比如优先级队列（元素的自然排序或者按照提供的比较器对元素进行排序），以及LIFO（后进先出）队列，如堆栈。
  - Queue接口未定义阻塞方法（用于并发编程），而在java.util.concurrent.BlockingQueue中有定义。
  - **不推荐使用Queue插入null元素**（尽管某些实现不禁止插入null），因为poll（）方法是将null作为特殊的返回值，指示队列不包含任何元素。

```java
public interface Queue<E> extends Collection<E> {
    // Collection接口
    // 添加元素到队列，插入成功后返回true，否则返回false，没有可用空间时会引发异常
    boolean add(E e);
    
    // 添加元素到队列，插入成功后返回true，否则返回false，没有可用空间时不会引发异常
    boolean offer(E e);
    E remove();// 删除并返回队列开头的元素，队列为空时会引发异常
    E poll();//删除并返回队列开头的元素，队列为空时会返回null
    E element();// 检索返回但不删除队列开头的元素，队列为空时会引发异常
    E peek();// 检索返回但不删除队列开头的元素，队列为空时会返回null
}
```

- **典型实现**：
  - *ArrayQueue（com.sun.jmx.remote.internal）*
  - ArrayDeque（java.util）
  - LinkList（java.util）
  - PriorityQueue（java.util）
  - ArrayBlockingQueue（java.util.concurrent）
  - LinkedBlockingQueue（java.util.concurrent）
  - LinkedBlockingDeque（java.util.concurrent）
  - PriorityBlockingQueue（java.util.concurrent）
  - DelayQueue（java.util.concurrent）
  - DelayedWorkQueue（static class in ScheduledThreadPoolExecutor）
  - SynchronousQueue（java.util.concurrent）
  - LinkedTransferQueue（java.util.concurrent）
  - ConcurrentLinkedQueue（java.util.concurrent）
  - ConcurrentLinkedDeque（java.util.concurrent）

#### Deque接口

- **特点**：

  - Deque，double-ended queue, **双端队列**，通常发音为“deck”，支持在两端插入和删除元素的线性集合。继承于Queue接口，拥有其所有方法，除Collection操作之外，还提供了其他插入、提取和检查的操作，一种在操作失败时会引发异常，一种是在操作失败时返回特殊值。但该接口还定义了**从两端访问双端队列元素的方法**：

  |              | First Element (Head)                    | First Element (Head)                   | Last Element (Tail)                   | Last Element (Tail)                  |
  | ------------ | --------------------------------------- | -------------------------------------- | ------------------------------------- | ------------------------------------ |
  | **Features** | *Throws exception*                      | *Special value*                        | *Throws exception*                    | *Special value*                      |
  | **Insert**   | {@link Deque#addFirst addFirst(e)}      | {@link Deque#offerFirst offerFirst(e)} | {@link Deque#addLast addLast(e)}      | {@link Deque#offerLast offerLast(e)} |
  | **Remove**   | {@link Deque#removeFirst removeFirst()} | {@link Deque#pollFirst pollFirst()}    | {@link Deque#removeLast removeLast()} | {@link Deque#pollLast pollLast()}    |
  | **Examine**  | {@link Deque#getFirst getFirst()}       | {@link Deque#peekFirst peekFirst()}    | {@link Deque#getLast getLast()}       | {@link Deque#peekLast peekLast()}    |

  - 当双端队列用作**队列**时，将导致**FIFO（先进先出）**行为：元素在双端队列的**末尾添加，并从开头删除**，peek（）用于检索队列开头。其中，从{@code Queue}接口继承的方法与{@code Deque}方法**完全等效**：

  | **{@code Queue} Method**                  | **Equivalent {@code Deque} Method** |
  | ----------------------------------------- | ----------------------------------- |
  | {@link java.util.Queue#add add(e)}        | {@link #addLast addLast(e)}         |
  | {@link java.util.Queue#offer offer(e)}    | {@link #offerLast offerLast(e)}     |
  | {@link java.util.Queue#remove remove()}   | {@link #removeFirst removeFirst()}  |
  | {@link java.util.Queue#poll poll()}       | {@link #pollFirst pollFirst()}      |
  | {@link java.util.Queue#element element()} | {@link #getFirst getFirst()}        |
  | {@link java.util.Queue#peek peek()}       | {@link #peek peekFirst()}           |

  - 当双端队列用作**堆栈**时，将导致**LIFO（后进先出）**行为：元素从双端队列的**开头被压入并在开头弹出**，peek（）用于检索队列开头。其中，{@code Stack}堆栈方法完全等同于{@code Deque}方法：

  | **Stack Method**      | **Equivalent {@code Deque} Method** |
  | --------------------- | ----------------------------------- |
  | {@link #push push(e)} | {@link #addFirst addFirst(e)}       |
  | {@link #pop pop()}    | {@link #removeFirst removeFirst()}  |
  | {@link #peek peek()}  | {@link #peekFirst peekFirst()}      |

  - **不推荐使用Queue插入null元素**（尽管某些实现不禁止插入null），因为poll（）方法是将null作为特殊的返回值，指示队列不包含任何元素。

```java
public interface Deque<E> extends Queue<E> {
    // 一堆Queue接口的方法
    ...
        
    void addFirst(E e);// true/false，会异常
    void addLast(E e);// true/false，会异常
    boolean offerFirst(E e);// true/false，不会异常
    boolean offerLast(E e);// true/false，不会异常
    
    E removeFirst();// 会异常
    E removeLast();// 会异常
    E pollFirst();// 不会异常，会null
    E pollLast();// 不会异常，会null
    
    E getFirst();// 会异常
    E getLast();// 会异常
    E peekFirst();// 不会异常，会null
    E peekLast();// 不会异常，会null
    
    boolean removeFirstOccurrence(Object o);// 删除第一个指定元素的方法
    boolean removeLastOccurrence(Object o);// 删除最后一个指定元素的方法
}
```

- 典型实现：
  - ArrayDeque（java.util）
  - LinkedBlockingDeque（java.util.concurrent）
  - ConcurrentLinkedDeque（java.util.concurrent）

#### BlockingQueue接口

- **特点**：

  - BlockingQueue，阻塞队列，继承自Queue接口，除了Queue接口方法外，还提供了以下操作：在检索**元素时等待队列变为非空，并在存储元素时等待队列中的空间变为可用。不允许null元素。**
  - BlockingQueue方法有四种形式：
    - 抛出异常。
    - 返回一个特殊值（{@code null}或{@code false}，具体取决于操作）。
    - 将无限期阻塞当前线程，直到操作成功为止。
    - 阻塞仅放弃给定的最大限制的时间。

  |             | *Throws exception*         | *Special value*         | *Blocks*             | *Times out*                                                 |
  | ----------- | -------------------------- | ----------------------- | -------------------- | ----------------------------------------------------------- |
  | **Insert**  | {@link #add add(e)}        | {@link #offer offer(e)} | {@link #put put(e)}  | {@link #offer(Object, long, TimeUnit) offer(e, time, unit)} |
  | **Remove**  | {@link #remove remove()}   | {@link #poll poll()}    | {@link #take take()} | {@link #poll(long, TimeUnit) poll(time, unit)}              |
  | **Examine** | {@link #element element()} | {@link #peek peek()}    | *not applicable*     | *not applicable*                                            |

  - BlockingQueue可能会受容量限制：这时底层通过**leftCapacity剩余容量**，来控制put（）添加时是否阻塞。
  - BlockingQueue是线程安全的，所有排队方法都是使用内部锁或其他形式的并发控制来原子地实现其效果的。本质上不支持任何类型的“关闭”操作，但可以让生产者插入**特殊的流尾对象或有毒对象**，当消费者采取这种方法时会对其进行相应的解释。

```java
public interface BlockingQueue<E> extends Queue<E> {
    // Queue接口方法
    boolean add(E e);// 为空时抛异常
    boolean offer(E e);// 为空时返回null，不抛异常
    ...
    
    // BlockingQueue接口方法
    void put(E e) throws InterruptedException;// 添加时阻塞
    E take() throws InterruptedException;// 删除时阻塞
    // 超时式添加
    boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException;
    // 超时式删除
    E poll(long timeout, TimeUnit unit) throws InterruptedException;
    ...
}
```

- **典型实现**：
  - ArrayBlockingQueue（java.util.concurrent）
  - LinkedBlockingQueue（java.util.concurrent）
  - LinkedBlockingDeque（java.util.concurrent）
  - PriorityBlockingQueue（java.util.concurrent）
  - DelayQueue（java.util.concurrent）
  - DelayedWorkQueue（static class in ScheduledThreadPoolExecutor）
  - SynchronousQueue（java.util.concurrent）
  - LinkedTransferQueue（java.util.concurrent）

#### BlockingDeque接口

- 特点：

  - BlockingDeque，在Deque接口基础上，还提供了支持阻塞的操作，**这些操作将在检索元素时等待双端队列变为非空，并在存储元素时等待双端队列中的空间可用**。
  - BlockingDeque方法有四种形式，它们以不同的方式处理操作，这些方法无法立即满足，但将来可能会满足：
    - 第一种是抛出异常。
    - 第二种是返回一个特殊值（两种方法之一 null}或{@code false}，具体取决于操作）。
    - 第三种是无限期阻塞当前线程，直到操作成功为止。
    - 第四种是阻塞仅一个给定的最大时间限制，然后再放弃。

  | **First Element (Head)** |                                    |                                           |                                |                                                              |
  | ------------------------ | ---------------------------------- | ----------------------------------------- | ------------------------------ | ------------------------------------------------------------ |
  |                          | *Throws exception*                 | *Special value*                           | *Blocks*                       | *Times out*                                                  |
  | **Insert**               | {@link #addFirst addFirst(e)}      | {@link #offerFirst(Object) offerFirst(e)} | {@link #putFirst putFirst(e)}  | {@link #offerFirst(Object, long, TimeUnit) offerFirst(e, time, unit)} |
  | **Remove**               | {@link #removeFirst removeFirst()} | {@link #pollFirst pollFirst()}            | {@link #takeFirst takeFirst()} | {@link #pollFirst(long, TimeUnit) pollFirst(time, unit)}     |
  | **Examine**              | {@link #getFirst getFirst()}       | {@link #peekFirst peekFirst()}            | *not applicable*               | *not applicable*                                             |
  | **Last Element (Tail)**  |                                    |                                           |                                |                                                              |
  |                          | *Throws exception*                 | *Special value*                           | *Blocks*                       | *Times out*                                                  |
  | **Insert**               | {@link #addLast addLast(e)}        | {@link #offerLast(Object) offerLast(e)}   | {@link #putLast putLast(e)}    | {@link #offerLast(Object, long, TimeUnit) offerLast(e, time, unit)} |
  | **Remove**               | {@link #removeLast() removeLast()} | {@link #pollLast() pollLast()}            | {@link #takeLast takeLast()}   | {@link #pollLast(long, TimeUnit) pollLast(time, unit)}       |
  | **Examine**              | {@link #getLast getLast()}         | {@link #peekLast peekLast()}              | *not applicable*               | *not applicable*                                             |

  - BlockingQueue与BlockingDeque方法等效关系：

  | **{@code BlockingQueue} Method**                            | **Equivalent {@code BlockingDeque} Method**                  |
  | ----------------------------------------------------------- | ------------------------------------------------------------ |
  | **Insert**                                                  |                                                              |
  | {@link #add(Object) add(e)}                                 | {@link #addLast(Object) addLast(e)}                          |
  | {@link #offer(Object) offer(e)}                             | {@link #offerLast(Object) offerLast(e)}                      |
  | {@link #put(Object) put(e)}                                 | {@link #putLast(Object) putLast(e)}                          |
  | {@link #offer(Object, long, TimeUnit) offer(e, time, unit)} | {@link #offerLast(Object, long, TimeUnit) offerLast(e, time, unit)} |
  | **Remove**                                                  |                                                              |
  | {@link #remove() remove()}                                  | {@link #removeFirst() removeFirst()}                         |
  | {@link #poll() poll()}                                      | {@link #pollFirst() pollFirst()}                             |
  | {@link #take() take()}                                      | {@link #takeFirst() takeFirst()}                             |
  | {@link #poll(long, TimeUnit) poll(time, unit)}              | {@link #pollFirst(long, TimeUnit) pollFirst(time, unit)}     |
  | **Examine**                                                 |                                                              |
  | {@link #element() element()}                                | {@link #getFirst() getFirst()}                               |
  | {@link #peek() peek()}                                      | {@link #peekFirst() peekFirst()}                             |

```java
public interface BlockingDeque<E> extends BlockingQueue<E>, Deque<E> {
    // 一堆Queue接口的方法
    ...
        
    // Deque接口的方法
    void addFirst(E e);
    void addLast(E e);
    boolean offerFirst(E e);
    boolean offerLast(E e);
    boolean removeFirstOccurrence(Object o);
    boolean removeLastOccurrence(Object o);
    
    // BlockingDeque接口的方法
    void putFirst(E e) throws InterruptedException;
    void putLast(E e) throws InterruptedException;
    boolean offerFirst(E e, long timeout, TimeUnit unit) throws InterruptedException;
    boolean offerLast(E e, long timeout, TimeUnit unit) throws InterruptedException;
    E takeFirst() throws InterruptedException;
    E takeLast() throws InterruptedException;
    E pollFirst(long timeout, TimeUnit unit) throws InterruptedException;
    E pollLast(long timeout, TimeUnit unit) throws InterruptedException;
}
```

- **典型实现**：LinkedBlockingDeque（java.util.concurrent）。

#### TransferQueue接口

- **特点**：
  - TransferQueue，**消息传输阻塞队列**，生产者可以在其中等待消费者接收元素。如在消息传递应用程序中可能很有用。
  - 与其他阻塞队列一样，{@code TransferQueue} 可能有容量限制。 其中，对于在零容量队列中，例如 {@link SynchronousQueue}，{@code put} 和 {@code transfer} 实际上是同义词。

```java
public interface TransferQueue<E> extends BlockingQueue<E> {
    
    // TransferQueue接口方法，快速失败，如果存在等待消息的消费者，则立即传输该元素，否则返回false元素不入队
    boolean tryTransfer(E e);
    
    // TransferQueue接口方法，同步阻塞，如果存在等待消息的消费者，则立即传输该元素，否则等待直到被消费者接收
    void transfer(E e) throws InterruptedException;
    
     // TransferQueue接口方法，延迟消息，如果存在等待消息的消费者，则立即传输该元素，否则在限定时间内等待过后元素才可以被传输
    boolean tryTransfer(E e, long timeout, TimeUnit unit) throws InterruptedException;
    
    // 判断是否存在等待消息的消费者，返回值表示事务的瞬时状态，如果消费者已完成或放弃等待，其数值可能不准确。
    boolean hasWaitingConsumer();
    
    // 获取等待消息的消费者的估计数量，返回值表示事务的瞬时状态，如果消费者已完成或放弃等待，其数值可能不准确。
    int getWaitingConsumerCount();
}
```



- **典型实现**：
  - LinkedTransferQueue（java.util.concurrent）

#### *ArrayQueue*

![1621606963948](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621606963948.png)

##### 特点

- 位于**com.sun.jmx.remote.internal**包中，不是Open JDK里的工具类，了解就好。
- 继承AbstractList抽象类，实质上是个List集合。**没有实现Queue接口，只是通过在方法内抛出异常，来局限只能执行队列操作，从而看起来像是遵守Queue接口的规范**。
- 底层通过数组实现，head和tail指针都是数组索引，而不是链表结点。
- 队列没有替换元素的方法，且获取元素方法都是检索栈顶的元素。

##### 构造方法

- **指定容量的构造函数**：数组实际容量 = 指定的容量 + 1，而head和tail是数组索引。

```java
package com.sun.jmx.remote.internal;
public class ArrayQueue<T> extends AbstractList<T> {
    
    private int capacity;
    private T[] queue;
    private int head;// head和tail是数组索引
    private int tail;// head和tail是数组索引
    
    // 指定容量的构造函数：数组实际容量 = 指定的容量 + 1
    public ArrayQueue(int capacity) {
        this.capacity = capacity + 1;
        this.queue = newArray(capacity + 1);
        this.head = 0;
        this.tail = 0;
    }
    private T[] newArray(int size) {
        return (T[]) new Object[size];
    }
}
```

##### 迭代方法

同AbstractList的迭代方法。

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    public Iterator<E> iterator() {
        return new Itr();
    }
}
```

##### 扩容方法

当前类不会触发扩容，而是在com.sun.jmx.remote.internal.ArrayNotificationBuffer#resize中触发。

```java
public void resize(int newcapacity) {
    int size = size();
    if (newcapacity < size)
        throw new IndexOutOfBoundsException("Resizing would lose data");
    newcapacity++;
    if (newcapacity == this.capacity)
        return;
    T[] newqueue = newArray(newcapacity);// 扩容时，创建新的指定容量的数组
    for (int i = 0; i < size; i++)
        newqueue[i] = get(i);
    this.capacity = newcapacity;
    this.queue = newqueue;
    this.head = 0;
    this.tail = size;
}
```

##### 添加元素方法

- **add（T）**：添加元素到队列，插入成功后返回true，否则返回false，**没有可用空间时会引发异常**。

```java
// 添加元素到队列
public boolean add(T o) {
    queue[tail] = o;
    int newtail = (tail + 1) % capacity;
    if (newtail == head)
        throw new IndexOutOfBoundsException("Queue full");
    tail = newtail;
    return true; // we did add something
}
```

##### 删除元素方法

- **remove（int）**：删除并返回队列开头（i必须为0）的元素，**队列为空时会引发异常**。

```java
// 删除并返回队列开头的元素
public T remove(int i) {
    if (i != 0)
        throw new IllegalArgumentException("Can only remove head of queue");
    if (head == tail)
        throw new IndexOutOfBoundsException("Queue empty");
    T removed = queue[head];
    queue[head] = null;
    head = (head + 1) % capacity;
    return removed;
}
```

##### 获取元素方法

- **get（int）**：检索返回但不删除队列开头（i必须为0）的元素，**队列为空时会引发异常**。

```java
// 检索返回但不删除队列开头的元素
public T get(int i) {
    int size = size();
    if (i < 0 || i >= size) {
        final String msg = "Index " + i + ", queue size " + size;
        throw new IndexOutOfBoundsException(msg);
    }
    int index = (head + i) % capacity;
    return queue[index];
}
```

#### ArrayDeque

![1621610556634](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621610556634.png)

##### 特点

- Deque接口可调整大小的数组实现，没有容量限制，会根据需要增长以支持使用，因此add（）插入时不会触发异常。**禁止使用null元素**。
- 底层通过数组实现，head和tail指针都是数组索引，而不是链表结点。
- 不是线程安全的，在没有外部同步的情况下，不支持多个线程并发访问。
- 用作堆栈时，此类**可能**比{@link Stack}更快，而用作队列时，此类**可能**比{@link LinkedList}更快。
- 队列没有替换元素的方法，且获取元素方法都是检索栈顶的元素。

##### 构造方法

- **无参的构造函数**：构造长度为16的数组。
- **指定容量的构造函数**：当指定容量至少为8时，会构造长度为16以上的数组（即当前数组长度 > 容量），而自定容量少于8时，只会构造最小长度数组（即当前数组长度 = 容量）。
- **指定复制集合的构造函数**：容量 = 复制集合的长度，当前数组长度 > 容量（至少一个2的指数）。

```java
public class ArrayDeque<E> extends AbstractCollection<E> implements Deque<E>, Cloneable, Serializable
{
	transient Object[] elements;
	transient int head;
	transient int tail;
	private static final int MIN_INITIAL_CAPACITY = 8;
	
	// 无参的构造函数
    public ArrayDeque() {
        elements = new Object[16];
    }
    
    // 指定容量的构造函数
    public ArrayDeque(int numElements) {
        allocateElements(numElements);
    }
    
    // 指定复制集合的构造函数
    public ArrayDeque(Collection<? extends E> c) {
        allocateElements(c.size());
        addAll(c);
    }
    private void allocateElements(int numElements) {
        int initialCapacity = MIN_INITIAL_CAPACITY;// 2^3
        // Find the best power of two to hold elements.
        // Tests "<=" because arrays aren't kept full.
        if (numElements >= initialCapacity) {
            // eg: 2^3 = 0000 1000
            initialCapacity = numElements;
            // eg: 0000 1000 | 0000 0100 = 0000 1100
            initialCapacity |= (initialCapacity >>>  1);
            // eg: 0000 1100 | 0000 0011 = 0000 1111
            initialCapacity |= (initialCapacity >>>  2);
            // eg: 0000 1111 | 0000 0000 = 0000 1111
            initialCapacity |= (initialCapacity >>>  4);
            // eg: 0000 1111 | 0000 0000 = 0000 1111
            initialCapacity |= (initialCapacity >>>  8);
            // eg: 0000 1111 | 0000 0000 = 0000 1111
            initialCapacity |= (initialCapacity >>> 16);
            // eg: 0000 1111 + 1 = 0001 0000 => 16
            initialCapacity++;
            // => 右移这么位的目的是: 为了取得容纳CAP最高位且保证是2的幂次, 是保证求出的最高的那个1，在16位指定容量中最高位的那个1之前

            if (initialCapacity < 0)   // Too many elements, must back off
                initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
        }
        elements = new Object[initialCapacity];
    }
}
```

##### 迭代方法

- **iterator（）**：返回此双端队列中的元素的迭代器，元素将从头到尾的顺序遍历，与元素出队（通过连续调用{@link #remove}或弹出（通过连续调用{@link #pop}）的顺序相同。

```java
// 返回此双端队列中的元素的迭代器
public Iterator<E> iterator() {
    return new DeqIterator();
}
private class DeqIterator implements Iterator<E> {
    ...
}
```

##### 扩容方法

- **扩容条件**：**当head索引等于tail索引时**，即实际长度（size）等于当前数组长度（elements.length）时。
- **doubleCapacity（）**：扩容时，**容量 * 2**，调用System.arraycopy（...）复制右左两边的数组，设置新数组指针、head、tail指针。

```java
// 扩容方法
private void doubleCapacity() {
    assert head == tail;
    int p = head;
    int n = elements.length;
    int r = n - p; // number of elements to the right of p
    int newCapacity = n << 1;
    if (newCapacity < 0)
        throw new IllegalStateException("Sorry, deque too big");
    Object[] a = new Object[newCapacity];
    // Object src, int srcPos, Object dest, int destPos, int length
    System.arraycopy(elements, p, a, 0, r);// 复制右边部分数组
    System.arraycopy(elements, 0, a, r, p);// 复制左边部分数组
    elements = a;
    head = 0;
    tail = n;
}
```

##### 添加元素方法

- **add（E）**：Queue接口方法，添加元素到队尾，由于有扩容机制，所以不会触发队列已满异常。
- **offer（E）**：Queue接口方法，添加元素到队尾。
- **addFirst（E）**：Deque接口方法，添加元素到队头，由于有扩容机制，所以不会触发队列已满异常。
- **addLast（E）**：Deque接口方法，添加元素到队尾，由于有扩容机制，所以不会触发队列已满异常。
- **offerFirst（E）**：Deque接口方法，添加元素到队头。
- **offerLast（E）**：Deque接口方法，添加元素到队尾。
- **pust（E）**：Deque接口方法，作为栈的方法使用，添加元素到队头（队列方法默认是添加到队尾）。

```java
// Queue接口方法，添加元素到队尾，由于有扩容机制，所以不会触发队列已满异常
public boolean add(E e) {
    addLast(e);
    return true;
}

// Queue接口方法，添加元素到队尾
public boolean offer(E e) {
    return offerLast(e);
}

// Deque接口方法，添加元素到队头，由于有扩容机制，所以不会触发队列已满异常
public void addFirst(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;
    if (head == tail)
        doubleCapacity();
}

// Deque接口方法，添加元素到队尾，由于有扩容机制，所以不会触发队列已满异常
public void addLast(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[tail] = e;
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)
        doubleCapacity();
}

// Deque接口方法，添加元素到队头
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}

// Deque接口方法，添加元素到队尾
public boolean offerLast(E e) {
    addLast(e);
    return true;
}

// Deque接口方法，作为栈的方法使用，添加元素到队头（队列方法默认是添加到队尾）
public void push(E e) {
    addFirst(e);
}
```

##### 删除元素方法

- **remove（）**：Queue接口方法，删除队头元素，如果队列为空，则会抛出异常。
- **poll（）**：Queue接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null。
- **removeFirst（）**：Deque接口方法，删除队头元素，如果队列为空，则会抛出异常。
- **removeLast（）**：Deque接口方法，删除队尾元素，如果队列为空，则会抛出异常。
- **pollFirst（）**：Deque接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null。
- **pollLast（）**：Deque接口方法，删除队尾元素，如果队列为空，不会抛出异常，而是返回null。
- **removeFirstOccurrence（Object）**：Deque接口方法，删除第一个指定的元素(从头到尾找)，找到并删除成功返回true，否则返回false。
- **removeLastOccurrence（Object）**：Deque接口方法，删除最后一个指定的元素(从尾到头找)，找到并删除成功返回true，否则返回false。
- **pop（）**：Deque接口方法，作为栈的方法使用，删除队头元素（队列方法默认也是删除队头元素）。

```java
// Queue接口方法，删除队头元素，如果队列为空，则会抛出异常
public E remove() {
    return removeFirst();
}

// Queue接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null
public E poll() {
    return pollFirst();
}

// Deque接口方法，删除队头元素，如果队列为空，则会抛出异常
public E removeFirst() {
    E x = pollFirst();
    if (x == null)
        throw new NoSuchElementException();
    return x;
}

// Deque接口方法，删除队尾元素，如果队列为空，则会抛出异常
public E removeLast() {
    E x = pollLast();
    if (x == null)
        throw new NoSuchElementException();
    return x;
}

// Deque接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null
public E pollFirst() {
    int h = head;
    E result = (E) elements[h];
    // Element is null if deque empty
    if (result == null)
        return null;
    elements[h] = null;     // Must null out slot
    head = (h + 1) & (elements.length - 1);
    return result;
}

// Deque接口方法，删除队尾元素，如果队列为空，不会抛出异常，而是返回null
public E pollLast() {
    int t = (tail - 1) & (elements.length - 1);
    E result = (E) elements[t];
    if (result == null)
        return null;
    elements[t] = null;
    tail = t;
    return result;
}

// Deque接口方法，删除第一个指定的元素(从头到尾找)，找到并删除成功返回true，否则返回false
public boolean removeFirstOccurrence(Object o) {
    if (o == null)
        return false;
    int mask = elements.length - 1;
    int i = head;
    Object x;
    while ( (x = elements[i]) != null) {
        if (o.equals(x)) {
            delete(i);
            return true;
        }
        i = (i + 1) & mask;
    }
    return false;
}

// Deque接口方法，删除最后一个指定的元素(从尾到头找)，找到并删除成功返回true，否则返回false
public boolean removeLastOccurrence(Object o) {
    if (o == null)
        return false;
    int mask = elements.length - 1;
    int i = (tail - 1) & mask;
    Object x;
    while ( (x = elements[i]) != null) {
        if (o.equals(x)) {
            delete(i);
            return true;
        }
        i = (i - 1) & mask;
    }
    return false;
}

// Deque接口方法，作为栈的方法使用，删除队头元素（队列方法默认也是删除队头元素）
public E pop() {
    return removeFirst();
}
```

##### 获取元素方法

- **element（）**：Queue接口方法，获取队头元素，如果队列为空，则会抛出异常。
- **peek（）**：Queue接口方法，获取队头元素，如果队列为空，不会抛出异常，而是返回null。
- **getFirst（）**：Deque接口方法，获取队头元素，如果队列为空，则会抛出异常。
- **getLast（）**：Deque接口方法，获取队尾元素，如果队列为空，则会抛出异常。
- **peekFirst（）**：Deque接口方法，获取队头元素，如果队列为空，不会抛出异常，而是返回null。
- **peekLast（）**：Deque接口方法，获取队尾元素，如果队列为空，不会抛出异常，而是返回null。

```java
// Queue接口方法，获取队头元素，如果队列为空，则会抛出异常
public E element() {
    return getFirst();
}

// Queue接口方法，获取队头元素，如果队列为空，不会抛出异常，而是返回null
public E peek() {
    return peekFirst();
}

// Deque接口方法，获取队头元素，如果队列为空，则会抛出异常
public E getFirst() {
    E result = (E) elements[head];
    if (result == null)
        throw new NoSuchElementException();
    return result;
}

// Deque接口方法，获取队尾元素，如果队列为空，则会抛出异常
public E getLast() {
    E result = (E) elements[(tail - 1) & (elements.length - 1)];
    if (result == null)
        throw new NoSuchElementException();
    return result;
}

// Deque接口方法，获取队头元素，如果队列为空，不会抛出异常，而是返回null
public E peekFirst() {
    // elements[head] is null if deque empty
    return (E) elements[head];
}

// Deque接口方法，获取队尾元素，如果队列为空，不会抛出异常，而是返回null
public E peekLast() {
    return (E) elements[(tail - 1) & (elements.length - 1)];
}
```

#### LinkedList

见List#LinkedList。

#### PriorityQueue

![1621669611615](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621669611615.png)

##### 特点

- PriorityQueue，**基于优先级堆的无界优先级队列**，根据自然排序或者队列构造时提供的{@link Comparator}进行排序，具体取决于所使用的构造函数。**不允许{@code null}元素。**
- **队头的元素是队列中“最小”的元素**，如果并列存在多个相同的元素，则此时是不稳定的。
- 队列是无界的，具有内部容量来控制用于在队列上存储元素的数组的大小。 它总是至少与队列大小一样大。 将元素添加到优先级队列时，其**容量会自动增长**。
- 迭代器**不保证**方法{@link #iterator（）}中提供的Iterator以任何特定顺序遍历优先级队列的元素。 如果需要有序遍历，请考虑使用{@code Arrays.sort（pq.toArray（））}。
- 是非线程安全的，需要线程安全时考虑{@link java.util.concurrent.PriorityBlockingQueue}。
- 入队和出队方法（{@code offer}，{@code poll}，{@code remove（）}和{@code add}）提供O（log（n））时间； {@code remove（Object）}和{@code contains（Object）}方法的线性时间，即O(n)；检索方法的固定时间（{@code peek}，{@code element}和{@code size}）即O(1)。
- 队列没有替换元素的方法，且获取元素方法都是检索栈顶的元素。

##### 构造方法

- **空参的构造函数**：使用默认容量11，不指定任何比较器。
- **指定容量的构造函数**：使用指定的容量，不指定任何比较器。
- **指定比较器的构造函数**：使用默认容量11，使用指定的比较器。
- **指定容量与比较器的构造函数**：构造指定容量的queue数组，以及设置指定的比较器。
- **指定复制集合的构造函数**：判断集合类型、根据类型初始化队列元素、设置实际大小、将数组最小堆化。
- **指定复制优先级队列的构造函数**：设置比较器为指定队列的比较器、初始化队列元素、设置实际大小。
- **指定排序集合的构造函数**：设置比较器为指定队列的比较器、初始化队列元素、设置实际大小。

```java
public class PriorityQueue<E> extends AbstractQueue<E> implements java.io.Serializable {
    
	private static final int DEFAULT_INITIAL_CAPACITY = 11;
    transient Object[] queue;
    private int size = 0;
    private final Comparator<? super E> comparator;
    transient int modCount = 0;

    // 空参的构造函数
    public PriorityQueue() {
        this(DEFAULT_INITIAL_CAPACITY, null);
    }

    // 指定容量的构造函数
    public PriorityQueue(int initialCapacity) {
        this(initialCapacity, null);
    }

    // 指定比较器的构造函数
    public PriorityQueue(Comparator<? super E> comparator) {
        this(DEFAULT_INITIAL_CAPACITY, comparator);
    }
    
    // 指定容量与比较器的构造函数
    public PriorityQueue(int initialCapacity, Comparator<? super E> comparator) {
        // Note: This restriction of at least one is not actually needed,
        // but continues for 1.5 compatibility
        if (initialCapacity < 1)
            throw new IllegalArgumentException();
        this.queue = new Object[initialCapacity];
        this.comparator = comparator;
    }
    
    // 指定复制集合的构造函数
    public PriorityQueue(Collection<? extends E> c) {
        if (c instanceof SortedSet<?>) {
            SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
            this.comparator = (Comparator<? super E>) ss.comparator();
            initElementsFromCollection(ss);
        }
        else if (c instanceof PriorityQueue<?>) {
            PriorityQueue<? extends E> pq = (PriorityQueue<? extends E>) c;
            this.comparator = (Comparator<? super E>) pq.comparator();
            initFromPriorityQueue(pq);
        }
        else {
            this.comparator = null;
            initFromCollection(c);
        }
    }
    private void initFromCollection(Collection<? extends E> c) {
        initElementsFromCollection(c);
        heapify();
    }

    // 指定复制优先级队列的构造函数
    public PriorityQueue(PriorityQueue<? extends E> c) {
        this.comparator = (Comparator<? super E>) c.comparator();
        initFromPriorityQueue(c);
    }
    private void initFromPriorityQueue(PriorityQueue<? extends E> c) {
        if (c.getClass() == PriorityQueue.class) {
            this.queue = c.toArray();
            this.size = c.size();
        } else {
            initFromCollection(c);
        }
    }
    
    // 指定排序集合的构造函数
    public PriorityQueue(SortedSet<? extends E> c) {
        this.comparator = (Comparator<? super E>) c.comparator();
        initElementsFromCollection(c);
    }
    
    // 构造实现1：复制指定集合的元素，调用Arrays.copyOf（...），校验元素不能为空
    private void initElementsFromCollection(Collection<? extends E> c) {
        Object[] a = c.toArray();
        // If c.toArray incorrectly doesn't return Object[], copy it.
        if (a.getClass() != Object[].class)
            a = Arrays.copyOf(a, a.length, Object[].class);
        int len = a.length;
        if (len == 1 || this.comparator != null)
            for (int i = 0; i < len; i++)
                if (a[i] == null)
                    throw new NullPointerException();
        this.queue = a;
        this.size = a.length;
    }
}
```

##### 数组最小堆化

- **相关公式**：堆是建立在满二叉树上的, 对于长度为n的序列, 最后一个非叶子结点序号i = n/2 - 1, 其左孩子序号为2i+1, 右孩子序号为2i+2, 父结点序号为(i-1)/2。
- **最小堆化步骤**：

1. 从最后一个非叶子结点K位置（叶子结点没有孩子不用比较）开始。
2. 选择K位置的左右孩子，判断当前元素的值、左右孩子的值之中的最小者。
3. 如果最小者落在K位置，则可以直接插入当前元素；否则把最小者的值设置到K位置，K交换到最小者的位置，继续向下调整。
4. 直到原来K位置所在的树的所有结点都遍历完后，原K位置便是这棵树最小的值了。
5. 接着从下到上，从右到左，一直遍历到序列的根结点queue[0]，经过调整后，queue[0]即是整个序列中最小的值了。这时便完成了序列元素的最小堆化。

```java
// 构造实现2：数组最小堆化
private void heapify() {
    for (int i = (size >>> 1) - 1; i >= 0; i--)
        // 每一步都设置最小值到k根结点处, 直到非叶子结点都调整过最小值后, 整个树成为最小堆
        siftDown(i, (E) queue[i]);
}
```

##### 最小堆调整

**PriorityQueue的精髓所在，用于初始化、插入、删除时的堆调整，使其数组一直保持最小堆的结构**。

- **siftDown（int，E）**：向下调整小顶堆, 使得插入后，当前树成为最小堆。

1. 判断没有指定比较器，没有比较器的使用自然排序比较规则，否则使用比较器的比较规则。
2. 选择当前位置的左右孩子，判断当前元素的值、左右孩子的值之中的最小者。
3. 如果最小者落在当前位置，则可以直接插入当前元素；否则把最小者的值设置到当前位置，当前位置交换到最小者的位置，继续向下调整。
4. 直到碰到合适的位置**（比下小时），插入当前元素**。

- **siftUp（int，E）**：向上调整小顶堆, 插入指定元素到合适位置。

1. 判断没有指定比较器，没有比较器的使用自然排序比较规则，否则使用比较器的比较规则。
2. 选择当前位置的父结点，判断当前的值与父结点的值之中的最大者。
3. 如果最大者落在当前位置，则可以直接插入当前元素；否则把最大者的值设置到当前位置，当前位置交换到父结点位置，继续向上调整。
4. 直到碰到合适的位置**（比上大时），插入当前元素**。

```java
// 1、向下调整小顶堆, 使得插入后，当前树成为最小堆
private void siftDown(int k, E x) {
    if (comparator != null)
        siftDownUsingComparator(k, x);
    else
        siftDownComparable(k, x);
}

// 没有指定比较器时, 使用自然排序筛选比较: 向下调整小顶堆, 使得插入后, 当前树成为最小堆
private void siftDownComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>)x;
    int half = size >>> 1;        // loop while a non-leaf
    while (k < half) {
        int child = (k << 1) + 1; // assume left child is least 左孩子: 2k + 1
        Object c = queue[child];
        int right = child + 1;// 右孩子: 2k + 2

        // 如果存在右孩子, 且左孩子比右孩子大, 则取右孩子(较小的一个)设置为待交换结点
        if (right < size && ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
            c = queue[child = right];
        // 如果当前元素的值比待交换结点的值还小, 说明当前元素的值已经是最小的了, 适合插入, 则退出循环
        if (key.compareTo((E) c) <= 0)
            break;
        // 否则说明, 当前元素的值比左(右)孩子的大, 则带交换结点插入当前位置, 当前位置走下到待交换结点的位置, 继续循环比较新的左右孩子
        queue[k] = c;
        k = child;
    }
    // 插入当前元素, 经过了上面的调整, 该位置一定是符合小顶堆的位置(比下小时)
    queue[k] = key;
}

// 有指定比较器时, 使用指定比较器进行筛选比较: 向下调整小顶堆, 使得插入后, 当前树成为最小堆
private void siftDownUsingComparator(int k, E x) {
    int half = size >>> 1;
    while (k < half) {
        int child = (k << 1) + 1;// 左孩子: 2k + 1
        Object c = queue[child];
        int right = child + 1;// 右孩子: 2k + 2

        // 如果存在右孩子, 且左孩子比右孩子大, 则取右孩子(较小的一个)设置为待交换结点
        if (right < size && comparator.compare((E) c, (E) queue[right]) > 0)
            c = queue[child = right];
        // 如果当前元素的值比待交换结点的值还小, 说明当前元素的值已经是最小的了, 适合插入, 则退出循环
        if (comparator.compare(x, (E) c) <= 0)
            break;
        // 否则说明, 当前元素的值比左(右)孩子的大, 则带交换结点插入当前位置, 当前位置走下到待交换结点的位置, 继续循环比较新的左右孩子
        queue[k] = c;
        k = child;
    }
    // 插入当前元素, 经过了上面的调整, 该位置一定是符合小顶堆的位置(比下小时)
    queue[k] = x;
}

// 2、向上调整小顶堆, 插入指定元素到合适位置
private void siftUp(int k, E x) {
    if (comparator != null)
        siftUpUsingComparator(k, x);
    else
        siftUpComparable(k, x);
}

// 没有指定比较器时, 使用自然排序筛选比较: 向上调整小顶堆, 直到有合适位置插入指定元素
private void siftUpComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>) x;
    while (k > 0) {
        // 父结点序号 = (k - 1) / 2
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];

        // 如果当前元素的值大于父结点, 则可以直接跳出循环
        if (key.compareTo((E) e) >= 0)
            break;

        // 否则说明当前元素的值小于父结点, 则父结点插入当前位置, 当前位置走上父结点位置, 继续循环比较下一个父结点
        queue[k] = e;
        k = parent;
    }

    // 插入当前元素, 经过了上面的调整, 该位置一定是符合小顶堆的位置(比上大时)
    queue[k] = key;
}

// 有指定比较器时, 使用指定比较器进行筛选比较: 向上调整小顶堆, 直到有合适位置插入指定元素
private void siftUpUsingComparator(int k, E x) {
    while (k > 0) {
        // 父结点序号 = (k - 1) / 2
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];

        // 如果当前元素的值大于父结点, 则可以直接跳出循环
        if (comparator.compare(x, (E) e) >= 0)
            break;

        // 否则说明当前元素的值小于父结点, 则父结点插入当前位置, 当前位置走上父结点位置, 继续循环比较下一个父结点
        queue[k] = e;
        k = parent;
    }

    // 插入当前元素, 经过了上面的调整, 该位置一定是符合小顶堆的位置(比上大时)
    queue[k] = x;
}
```

##### 迭代方法

- iterator（）：返回对该队列中的元素进行迭代的迭代器，**该迭代器不会以任何特定顺序返回元素**，因为最小堆只能保证堆顶最小，而不能保证孩子结点中的顺序。

```java
public Iterator<E> iterator() {
    return new Itr();
}
```

##### 扩容方法

- **扩容条件**：size >= queue.length，即当实际大小等于当前数组大小时。
- **grow（int）**：当旧容量（当前数组大小） < 64 时，扩容为容量 * 2 + 2；否则扩容为容量 * 1.5。

1. 最后进行容量界限校验:0 < c < Integer.MAX_VALUE - 8。
2. 调用Arrays.copyOf（...）复制元素到新的数组。

```java
private void grow(int minCapacity) {
    int oldCapacity = queue.length;
    // Double size if small; else grow by 50%
    int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                     (oldCapacity + 2) :
                                     (oldCapacity >> 1));
    // overflow-conscious code
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        // 容量界限校验:0 < c < Integer.MAX_VALUE - 8
        newCapacity = hugeCapacity(minCapacity);
    queue = Arrays.copyOf(queue, newCapacity);
}
```

##### 添加元素方法

由于Queue数组保持着最小堆的结构，这时**插入元素到队尾**后，需要进行向上调整。

- **add（E）**：Queue接口方法，添加元素到队尾，由于有扩容机制，所以不会触发队列已满异常。
- **offer（E）**：Queue接口方法，添加元素到队尾。

```java
// Queue接口方法，添加元素到队尾，由于有扩容机制，所以不会触发队列已满异常
public boolean add(E e) {
    return offer(e);
}

// Queue接口方法，添加元素到队尾
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);
    size = i + 1;
    if (i == 0)
        queue[0] = e;
    else
        siftUp(i, e);// 向上调整小顶堆, 插入指定元素到合适位置。
    return true;
}
```

##### 删除元素方法

由于Queue数组保持着最小堆的结构，这时**删除元素后，需要以队尾作为替代结点，插入当前位置并向下进行调整**。

- **remove（）**：Queue接口方法，由AbstractQueue抽象类实现，删除队头元素，如果队列为空，则会抛出异常。
- **offer（）**：Queue接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null。
- **remove（Object）**：Collection接口方法，删除第一个指定的元素，使用equals比较。

```java
// Queue接口方法，由AbstractQueue抽象类实现，删除队头元素，如果队列为空，则会抛出异常
public E remove() {
    E x = poll();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null
public E poll() {
    if (size == 0)
        return null;
    int s = --size;
    modCount++;
    E result = (E) queue[0];
    E x = (E) queue[s];// 使用队尾元素作为替代结点
    queue[s] = null;
    if (s != 0)
        siftDown(0, x);// 从根位置向下调整小顶堆, 使得插入替代结点后, 当前树成为最小堆
    return result;
}

// Collection接口方法，删除第一个指定的元素
public boolean remove(Object o) {
    int i = indexOf(o);
    if (i == -1)
        return false;
    else {
        removeAt(i);
        return true;
    }
}
// 找到指定元素在队列中的索引
private int indexOf(Object o) {
    if (o != null) {
        for (int i = 0; i < size; i++)
            if (o.equals(queue[i]))
                return i;
    }
    return -1;
}
// 从队列中删除第ith个元素
private E removeAt(int i) {
    // assert i >= 0 && i < size;
    modCount++;
    int s = --size;
    if (s == i) // removed last element
        queue[i] = null;
    else {
        E moved = (E) queue[s];// 设置末尾结点为替代结点
        queue[s] = null;// 清空末尾结点
        siftDown(i, moved);// 向下调整小顶堆, 使得替代结点的值插入后，当前树成为最小堆

        // PriorityQueue.iterator#remove()才会触发下面代码
        // 如果当前树的根结点的值为替代结点的值，说明当前数组有可能不是最小堆的结构
        if (queue[i] == moved) {
            siftUp(i, moved);// 所以接着向上调整小顶堆, 插入替代结点的值到合适位置
            if (queue[i] != moved)// 如果当前位置的值已经不为替代结点的值了, 说明向上调整成功
                return moved;// 返回替代结点的元素, 标识一下经过了向上调整
        }
    }
    return null;
}
```

##### 获取元素方法

- **element（）**：Queue接口方法：由AbstractQueue抽象类实现，检索队头元素，如果队列为空，则会抛出异常。
- **peek（）**：Queue接口方法，检索队头元素，如果队列为空，不会抛出异常，而是返回null。

```java
// Queue接口方法，由AbstractQueue抽象类实现，检索队头元素，如果队列为空，则会抛出异常
public E element() {
    E x = peek();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，检索队头元素，如果队列为空，不会抛出异常，而是返回null
public E peek() {
    return (size == 0) ? null : (E) queue[0];
}
```

#### ArrayBlockingQueue

![1621742345924](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621742345924.png)

##### 特点

- ArrayBlockingQueue，**由数组支持的有界阻塞队列**，对元素进行FIFO（先进先出）排序：
  - 队列的头是已经在队列中最长时间的元素。
  - 队列的尾部是最短时间出现在队列中的元素。
  - 新元素插入到队列的尾部。
  - 队列检索操作在队列的开头来获取元素。
- ArrayBlockingQueue是**经典的“有界缓冲区”**，其中保存的固定大小的数组（由生产者插入并由消费者提取元素），数组创建后，容量将无法更改。 尝试将元素{@code put}到完整队列中将导致操作阻塞，尝试从空队列中{@code pull]一个元素也会类似地被阻塞。
- ArrayBlockingQueue支持可选的公平性策略，**默认为非公平的**，但可以使用为{@code true}构造（参数交由ReentrantLock实现公平性）的队列，进行公平性设置，此时队列将按FIFO顺序授予线程访问权限。**公平通常会降低吞吐量，但会减少可变性并避免饥饿。**
- **允许队列操作更新迭代器状态**。
- 队列没有替换元素的方法，且获取元素方法都是检索栈顶的元素。

##### 构造方法

- **指定容量的构造函数**：默认非公平方式。
- **指定容量与公平性设置的构造函数**：创建指定容量的数组，以及创建指定的公平性的可重入锁。
- **指定容量、公平性设置与复制集合的构造函数**：创建数组、可重入锁，加锁复制集合元素（越界会抛异常）。

```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {
    
    final Object[] items;// 元素队列
    int takeIndex;// 元素的删除索引
    int putIndex;// 元素的添加索引
    int count;// 实际大小
    final ReentrantLock lock;// 可重入锁
    private final Condition notEmpty;// 非空等待条件
    private final Condition notFull;// 可用等待条件
    transient Itrs itrs = null;// 当前活动的迭代器
    
    // 指定容量的构造函数: 默认非公平方式
    public ArrayBlockingQueue(int capacity) {
        this(capacity, false);
    }
    
    // 指定容量与公平性设置的构造函数：创建指定容量的数组，以及创建指定的公平性的可重入锁
    public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }
    
    // 指定容量、公平性设置与复制集合的构造函数：创建数组、可重入锁，加锁复制集合元素（越界会抛异常）
    public ArrayBlockingQueue(int capacity, boolean fair,
                              Collection<? extends E> c) {
        this(capacity, fair);

        final ReentrantLock lock = this.lock;
        lock.lock(); // Lock only for visibility, not mutual exclusion
        try {
            int i = 0;
            try {
                for (E e : c) {
                    checkNotNull(e);
                    items[i++] = e;
                }
            } catch (ArrayIndexOutOfBoundsException ex) {
                throw new IllegalArgumentException();
            }
            count = i;
            putIndex = (i == capacity) ? 0 : i;
        } finally {
            lock.unlock();
        }
    }
}
```

##### 迭代方法

- **iterator（）**：以适当的顺序返回对该队列中的元素的迭代器。 元素将按照从头到尾的顺序返回。返回的迭代器是**弱一致性的**。

1. 为了保持关于puts和takes的弱一致性，该迭代器预读了一个槽位，以免报告hasNext为true，但是没有要返回的元素。

```java
public Iterator<E> iterator() {
    return new Itr();// $$ 很复杂，没有仔细研究 $$
}
```

##### 扩容方法

ArrayBlockingQueue是有界队列，**没有扩容机制**，数组在构造时就指定了容量，不能动态更改。

##### 添加元素方法

所有方法都是使用ReentrantLock加锁同步实现。而**在元素入队（插入队尾）后，唤醒等待非空条件的线程**。

- **add（E）**：Queue接口方法，添加一个元素，由于队列有界，当队列满时会抛出异常。
- **offer（E）**：Queue接口方法，添加一个元素，由于队列有界，不会抛出异常，而是返回false。
- **put（E）**：BlockingQueue接口方法，队列满时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
- **offer（E，long，TimeUnit）**：BlockingQueue接口方法，队列满时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）。

```java
// Queue接口方法，添加一个元素，由于队列有界，当队列满时会抛出异常
public boolean add(E e) {
    return super.add(e);
}
// AbstractQueue#add（E）
public boolean add(E e) {
    if (offer(e))
        return true;
    else
        throw new IllegalStateException("Queue full");
}

// Queue接口方法，添加一个元素，由于队列有界，不会抛出异常，而是返回false
public boolean offer(E e) {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count == items.length)
            return false;
        else {
            enqueue(e);// 入队
            return true;
        }
    } finally {
        lock.unlock();
    }
}
// 入队，唤醒等待非空条件的线程
private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;
    notEmpty.signal();// 唤醒一个等待线程：通知队列非空了
}

// BlockingQueue接口方法，队列满时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            notFull.await();// 队列满时，会一直阻塞等待非满条件
        enqueue(e);// 队列非满时，入队
    } finally {
        lock.unlock();
    }
}

// BlockingQueue接口方法，队列满时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）
public boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException {
    checkNotNull(e);
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length) {
            if (nanos <= 0)
                return false;
            nanos = notFull.awaitNanos(nanos);// 队列满时，会在限定时间内一直阻塞等待非满条件
        }
        enqueue(e);// 队列非满时，入队
        return true;
    } finally {
        lock.unlock();
    }
}
```

##### 删除元素方法

所有方法都是使用ReentrantLock加锁同步实现。而**在元素出队（删除队头）后，唤醒等待非满条件的线程**。

- **remove（）**：Queue接口方法，交由抽象类实现AbstractQueue#remove（），当队列为空时，会抛出异常。
- **poll（）**：Queue接口方法，当队列为空时，不会抛出异常，而是返回null。
- **take（）**：BlockingQueue接口方法，队列为空时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
- **poll（long，TimeUnit）**：BlockingQueue接口方法，队列为空时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
- **remove（Object）**：Collection接口方法，删除第一个指定的元素，使用equals判断。

```java
// Queue接口方法，交由抽象类实现AbstractQueue#remove（），当队列为空时，会抛出异常
public E remove() {
    E x = poll();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，当队列为空时，不会抛出异常，而是返回null
public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return (count == 0) ? null : dequeue();// 出队
    } finally {
        lock.unlock();
    }
}
// 入队，唤醒等待非满条件的线程
private E dequeue() {
    // assert lock.getHoldCount() == 1;
    // assert items[takeIndex] != null;
    final Object[] items = this.items;
    E x = (E) items[takeIndex];
    items[takeIndex] = null;
    if (++takeIndex == items.length)
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();// 每当元素已出队（在takeIndex处）时调用。
    notFull.signal();// 唤醒一个等待线程：通知队列非满了
    return x;
}

// BlockingQueue接口方法，队列为空时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            notEmpty.await();// 队列为空时，会一直阻塞等待非空条件
        return dequeue();// 队列非空时，出队
    } finally {
        lock.unlock();
    }
}

// BlockingQueue接口方法，队列为空时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0) {
            if (nanos <= 0)
                return null;
            // 队列为空时，会在限定时间内阻塞等待非空条件
            nanos = notEmpty.awaitNanos(nanos);
        }
        return dequeue();// 队列非空时，出队
    } finally {
        lock.unlock();
    }
}

// Collection接口方法，删除第一个指定的元素，使用equals判断
public boolean remove(Object o) {
    if (o == null) return false;
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count > 0) {
            final int putIndex = this.putIndex;
            int i = takeIndex;
            do {
                if (o.equals(items[i])) {
                    // 移除第一个equal的元素, 唤醒一个等待线程：通知队列非满了, 然后返回true
                    removeAt(i);
                    return true;
                }
                if (++i == items.length)
                    i = 0;
            } while (i != putIndex);
        }
        return false;
    } finally {
        lock.unlock();
    }
}
```

##### 获取元素方法

连检索的所有方法都是使用ReentrantLock加锁同步实现，用于检索队头元素。

- **element（）**：Queue接口方法，交由抽象类实现AbstractQueue#element（），当队列为空时，会抛出异常。
- **peek（）**：Queue接口方法，当队列为空时，不会抛出异常，而是返回null。

```java
// Queue接口方法，交由抽象类实现AbstractQueue#element（），当队列为空时，会抛出异常
public E element() {
    E x = peek();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，当队列为空时，不会抛出异常，而是返回null
public E peek() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return itemAt(takeIndex); // null when queue is empty
    } finally {
        lock.unlock();
    }
}
final E itemAt(int i) {
    return (E) items[i];
}
```

#### LinkedBlockingQueue

![1621757385625](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621757385625.png)

##### 特点

- LinkedBlockingQueue，**基于单向链表实现的有界阻塞队列**，对元素进行FIFO（先进先出排序）：
  - 队列的头是已经在队列中最长时间的元素。
  - 队列的尾部是最短时间出现在队列中的元素。
  - 新元素插入到队列的尾部。
  - 队列检索操作在队列的开头获取元素。
- LinkedBlockingQueue通常比基于ArrayBlockingQueue具有更高的吞吐量（无需维护数组指针），但是在大多数并发应用程序中，可预测的性能较差（结点数多？）。
- LinkedBlockingQueue容量未指定时，默认为Integer＃MAX_VALUE，将会导致每次插入时动态创建链接节点，直至超出容量，所以一般往往在构造时指定容量，防止队列拥有过多元素。
- 队列没有替换元素的方法，且获取元素方法都是检索栈顶的元素。
- 简而言之，其实现可参考LinkedBlocking + 实现Deque的思路，由于太类似，所以不做多描述了。

##### 构造方法

- **空参的构造函数**：默认容量为Integer.MAX_VALUE = 0x7fffffff。
- **指定容量的构造函数**：设置容量、创建头结点。
- **指定复制集合的构造函数**：默认容量为Integer.MAX_VALUE，加锁，遍历，入队。

```java
public class LinkedBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {
    
    static class Node<E> {
        E item;
        Node<E> next;
        Node(E x) { item = x; }
    }
    
    private final int capacity;
    private final AtomicInteger count = new AtomicInteger();
    transient Node<E> head;
    private transient Node<E> last;
    
    private final ReentrantLock takeLock = new ReentrantLock();
    private final Condition notEmpty = takeLock.newCondition();
    private final ReentrantLock putLock = new ReentrantLock();
    private final Condition notFull = putLock.newCondition();
    
    // 空参的构造函数：默认容量为Integer.MAX_VALUE = 0x7fffffff
    public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }
    
    // 指定容量的构造函数：设置容量、创建头结点
    public LinkedBlockingQueue(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException();
        this.capacity = capacity;
        last = head = new Node<E>(null);
    }

    // 指定复制集合的构造函数：默认容量为Integer.MAX_VALUE，加锁，遍历，入队
    public LinkedBlockingQueue(Collection<? extends E> c) {
        this(Integer.MAX_VALUE);
        final ReentrantLock putLock = this.putLock;
        putLock.lock(); // Never contended, but necessary for visibility
        try {
            int n = 0;
            for (E e : c) {
                if (e == null)
                    throw new NullPointerException();
                if (n == capacity)
                    throw new IllegalStateException("Queue full");
                enqueue(new Node<E>(e));// 入队
                ++n;
            }
            count.set(n);
        } finally {
            putLock.unlock();
        }
    }
}
```

##### 迭代方法

- **iterator（）**：以适当的顺序返回对该队列中的元素的迭代器。 元素将按照从头到尾的顺序返回。返回的迭代器是**弱一致性的**。

```java
public Iterator<E> iterator() {
    return new Itr();// $$ 很复杂，没有仔细研究 $$
}
```

##### 扩容方法

LinkedBlockingQueue是链表实现，**无需扩容机制**，只需要维护链表结点指针即可。

##### 添加元素方法

所有方法都是使用ReentrantLock加锁同步实现。而**在元素入队（插入队尾）后，如果实际大小还没达到容量时，唤醒等待非满条件的线程；如果该元素为第一个元素并添加成功后，唤醒等待非空条件的线程**。

- **add（E）**：Queue接口方法，添加一个元素，由于队列有界，当队列满时会抛出异常。
- **offer（E）**：Queue接口方法，添加一个元素，由于队列有界，不会抛出异常，而是返回false。
- **put（E）**：BlockingQueue接口方法，队列满时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
- **offer（E，long，TimeUnit）**：BlockingQueue接口方法，队列满时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）。

```java
// Queue接口方法，添加一个元素，由于队列有界，当队列满时会抛出异常
public boolean add(E e) {
    if (offer(e))
        return true;
    else
        throw new IllegalStateException("Queue full");
}

// Queue接口方法，添加一个元素，由于队列有界，不会抛出异常，而是返回false
public boolean offer(E e) {
    if (e == null) throw new NullPointerException();
    final AtomicInteger count = this.count;
    if (count.get() == capacity)
        return false;
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        if (count.get() < capacity) {
            enqueue(node);// 队列非满时，入队
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();// 如果实际大小还没达到容量时，唤醒等待非满条件的线程
        }
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();// 如果该元素为第一个元素并添加成功后，唤醒等待非空条件的线程
    return c >= 0;
}
// 入队
private void enqueue(Node<E> node) {
    // assert putLock.isHeldByCurrentThread();
    // assert last.next == null;
    last = last.next = node;
}
// 唤醒等待非空条件的线程
private void signalNotEmpty() {
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
}

// BlockingQueue接口方法，队列满时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    // Note: convention in all put/take/etc is to preset local var
    // holding count negative to indicate failure unless set.
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
        while (count.get() == capacity) {
            notFull.await();// 队列满时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）
        }
        enqueue(node);// 队列非满时，入队
        c = count.getAndIncrement();
        if (c + 1 < capacity)
            notFull.signal();// 如果实际大小还没达到容量时，唤醒等待非满条件的线程
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();// 如果该元素为第一个元素并添加成功后，唤醒等待非空条件的线程
}

// BlockingQueue接口方法，队列满时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）
public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {

    if (e == null) throw new NullPointerException();
    long nanos = unit.toNanos(timeout);
    int c = -1;
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
        while (count.get() == capacity) {
            if (nanos <= 0)
                return false;
            nanos = notFull.awaitNanos(nanos);// 队列满时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）
        }
        enqueue(new Node<E>(e));// 队列非满时，入队
        c = count.getAndIncrement();
        if (c + 1 < capacity)
            notFull.signal();// 如果实际大小还没达到容量时，唤醒等待非满条件的线程
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();// 如果该元素为第一个元素并添加成功后，唤醒等待非空条件的线程
    return true;
}
```

##### 删除元素方法

所有方法都是使用ReentrantLock加锁同步实现。而**在元素出队（删除队头）后，如果剩余结点大于1，则唤醒等待非空条件的线程；如果原本是满的且删除成功，则唤醒等待非满条件的线程**。

- **remove（）**：Queue接口方法，交由抽象类实现AbstractQueue#remove（），当队列为空时，会抛出异常。
- **poll（）**：Queue接口方法，当队列为空时，不会抛出异常，而是返回null。
- **take（）**：BlockingQueue接口方法，队列为空时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
- **poll（long，TimeUnit）**：BlockingQueue接口方法，队列为空时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
- **remove（Object）**：Collection接口方法，删除第一个指定的元素，使用equals判断。

```java
// Queue接口方法，交由抽象类实现AbstractQueue#remove（），当队列为空时，会抛出异常
public E remove() {
    E x = poll();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，当队列为空时，不会抛出异常，而是返回null
public E poll() {
    final AtomicInteger count = this.count;
    if (count.get() == 0)
        return null;
    E x = null;
    int c = -1;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        if (count.get() > 0) {
            x = dequeue();// 队列非空时，出队
            c = count.getAndDecrement();
            if (c > 1)
                notEmpty.signal();// 如果剩余结点大于1，则唤醒等待非空条件的线程
        }
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();// 如果原本是满的且删除成功，则唤醒等待非满条件的线程
    return x;
}
// 出队
private E dequeue() {
    // assert takeLock.isHeldByCurrentThread();
    // assert head.item == null;
    Node<E> h = head;
    Node<E> first = h.next;
    h.next = h; // help GC
    head = first;
    E x = first.item;
    first.item = null;
    return x;
}
// 唤醒等待非满条件的线程
private void signalNotFull() {
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        notFull.signal();
    } finally {
        putLock.unlock();
    }
}

// BlockingQueue接口方法，队列为空时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {
            notEmpty.await();// 队列为空时，会一直阻塞等待非空条件
        }
        x = dequeue();// 队列非空时，出队
        c = count.getAndDecrement();
        if (c > 1)
            notEmpty.signal();// 如果剩余结点大于1，则唤醒等待非空条件的线程
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();// 如果原本是满的且删除成功，则唤醒等待非满条件的线程
    return x;
}

// BlockingQueue接口方法，队列为空时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    E x = null;
    int c = -1;
    long nanos = unit.toNanos(timeout);
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {
            if (nanos <= 0)
                return null;
            // 队列为空时，会限定时间内一直阻塞等待非空条件
            nanos = notEmpty.awaitNanos(nanos);
        }
        x = dequeue();// 队列非空时，出队
        c = count.getAndDecrement();
        if (c > 1)
            notEmpty.signal();// 如果剩余结点大于1，则唤醒等待非空条件的线程
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();// 如果原本是满的且删除成功，则唤醒等待非满条件的线程
    return x;
}

// Collection接口方法，删除第一个指定的元素，使用equals判断
public boolean remove(Object o) {
    if (o == null) return false;
    fullyLock();
    try {
        for (Node<E> trail = head, p = trail.next;
             p != null;
             trail = p, p = p.next) {
            if (o.equals(p.item)) {
                unlink(p, trail);// 删除第一个equal的结点
                return true;
            }
        }
        return false;
    } finally {
        fullyUnlock();
    }
}
// 删除结点
void unlink(Node<E> p, Node<E> trail) {
    // assert isFullyLocked();
    // p.next is not changed, to allow iterators that are
    // traversing p to maintain their weak-consistency guarantee.
    p.item = null;
    trail.next = p.next;
    if (last == p)
        last = trail;
    if (count.getAndDecrement() == capacity)
        notFull.signal();// 如果原本是满的且删除成功，则唤醒等待非满条件的线程
}
```

##### 获取元素方法

连检索的所有方法都是使用ReentrantLock加锁同步实现，用于检索队头元素。

- **element（）**：Queue接口方法，交由抽象类实现AbstractQueue#element（），当队列为空时，会抛出异常。
- **peek（）**：Queue接口方法，当队列为空时，不会抛出异常，而是返回null。

```java
// Queue接口方法，交由抽象类实现AbstractQueue#element（），当队列为空时，会抛出异常
public E element() {
    E x = peek();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，当队列为空时，不会抛出异常，而是返回null
public E peek() {
    if (count.get() == 0)
        return null;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        Node<E> first = head.next;
        if (first == null)
            return null;
        else
            return first.item;
    } finally {
        takeLock.unlock();
    }
}
```

#### LinkedBlockingDeque

![1621764421090](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621764421090.png)

##### 特点

- LinkedBlockingDeque，**基于双向链表实现的阻塞双端队列**。
- LinkedBlockingDeque容量未指定时，默认为Integer＃MAX_VALUE，将会导致每次插入时动态创建链接节点，直至超出容量，所以一般往往在构造时指定容量，防止队列拥有过多元素。
- 队列没有替换元素的方法，且获取元素方法都是检索栈顶的元素。

##### 构造方法

- **空参的构造函数**：默认容量为Integer.MAX_VALUE = 0x7fffffff。
- **指定容量的构造函数**：设置容量、创建头结点。
- **指定复制集合的构造函数**：默认容量为Integer.MAX_VALUE，加锁，遍历，入队。

```java
public class LinkedBlockingDeque<E> extends AbstractQueue<E> implements BlockingDeque<E>, java.io.Serializable {
    
    static final class Node<E> {
        E item;
        Node<E> prev;
        Node<E> next;
       	Node(E x) {
            item = x;
        }
    }
    
    transient Node<E> first;
    transient Node<E> last;
    private transient int count;
    private final int capacity;
    final ReentrantLock lock = new ReentrantLock();
    private final Condition notEmpty = lock.newCondition();
    private final Condition notFull = lock.newCondition();
    
    // 空参的构造函数
    public LinkedBlockingDeque() {
        this(Integer.MAX_VALUE);
    }
    
    // 指定容量的构造函数
    public LinkedBlockingDeque(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException();
        this.capacity = capacity;
    }
    
    // 指定复制集合的构造函数
    public LinkedBlockingDeque(Collection<? extends E> c) {
        this(Integer.MAX_VALUE);
        final ReentrantLock lock = this.lock;
        lock.lock(); // Never contended, but necessary for visibility
        try {
            for (E e : c) {
                if (e == null)
                    throw new NullPointerException();
                if (!linkLast(new Node<E>(e)))
                    throw new IllegalStateException("Deque full");
            }
        } finally {
            lock.unlock();
        }
    }
  	// 添加元素到队尾
    private boolean linkLast(Node<E> node) {
        // assert lock.isHeldByCurrentThread();
        if (count >= capacity)
            return false;
        Node<E> l = last;
        node.prev = l;
        last = node;
        if (first == null)
            first = node;
        else
            l.next = node;
        ++count;
        notEmpty.signal();// 唤醒等待非空条件的线程
        return true;
    }
}
```

#### PriorityBlockingQueue

![1621768802057](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621768802057.png)

##### 特点

- PriorityBlockingQueue，**基于优先级堆的无界优先级阻塞队列**，并提供**阻塞检索**操作。**不允许{@code null}元素。**根据自然排序或者队列构造时提供的{@link Comparator}进行排序，具体取决于所使用的构造函数。
- 尽管此队列在逻辑上是不受限制的，但是尝试添加可能会由于资源耗尽而失败（导致{@code OutOfMemoryError}）。
- **队头的元素是队列中“最小”的元素**，如果并列存在多个相同的元素，则此时是不稳定的。
- 迭代器**不保证**方法{@link #iterator（）}中提供的Iterator以任何特定顺序遍历优先级队列的元素。 如果需要有序遍历，请考虑使用{@code Arrays.sort（pq.toArray（））}。
- 队列没有替换元素的方法，且获取元素方法都是检索栈顶的元素。

##### 构造方法

- **空参的构造函数**：使用默认容量11，不指定任何比较器。
- **指定容量的构造函数**：使用指定的容量，不指定任何比较器。
- **指定容量与比较器的构造函数**：构造指定容量的queue数组，以及设置指定的比较器，创建可重入锁，创建等待非空条件。
- **指定复制集合的构造函数**：创建可重入锁、创建等待非空条件、判断集合类型、根据类型初始化队列元素、设置实际大小（调用Arrays.copyOf（...））、将数组最小堆化。

```java
public class PriorityBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {
    
    private static final int DEFAULT_INITIAL_CAPACITY = 11;
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8; 
    private transient Object[] queue;
    private transient int size;
    private transient Comparator<? super E> comparator;
    private final ReentrantLock lock;
    private final Condition notEmpty;
    private transient volatile int allocationSpinLock;// 自旋锁进行分配，通过CAS获取
    private PriorityQueue<E> q;// 一个普通的PriorityQueue，仅用于序列化
    
    // 空参的构造函数
    public PriorityBlockingQueue() {
        this(DEFAULT_INITIAL_CAPACITY, null);
    }
    
    // 指定容量的构造函数
    public PriorityBlockingQueue(int initialCapacity) {
        this(initialCapacity, null);
    }
    
    // 指定容量与比较器的构造函数
    public PriorityBlockingQueue(int initialCapacity,
                                 Comparator<? super E> comparator) {
        if (initialCapacity < 1)
            throw new IllegalArgumentException();
        this.lock = new ReentrantLock();
        this.notEmpty = lock.newCondition();
        this.comparator = comparator;
        this.queue = new Object[initialCapacity];
    }
    
    // 指定复制集合的构造函数
    public PriorityBlockingQueue(Collection<? extends E> c) {
        this.lock = new ReentrantLock();
        this.notEmpty = lock.newCondition();
        boolean heapify = true; // true if not known to be in heap order
        boolean screen = true;  // true if must screen for nulls
        if (c instanceof SortedSet<?>) {
            SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
            this.comparator = (Comparator<? super E>) ss.comparator();
            heapify = false;
        }
        else if (c instanceof PriorityBlockingQueue<?>) {
            PriorityBlockingQueue<? extends E> pq =
                (PriorityBlockingQueue<? extends E>) c;
            this.comparator = (Comparator<? super E>) pq.comparator();
            screen = false;
            if (pq.getClass() == PriorityBlockingQueue.class) // exact match
                heapify = false;
        }
        Object[] a = c.toArray();
        int n = a.length;
        // If c.toArray incorrectly doesn't return Object[], copy it.
        if (a.getClass() != Object[].class)
            a = Arrays.copyOf(a, n, Object[].class);
        if (screen && (n == 1 || this.comparator != null)) {
            for (int i = 0; i < n; ++i)
                if (a[i] == null)
                    throw new NullPointerException();
        }
        this.queue = a;
        this.size = n;
        if (heapify)
            heapify();// 数组最小堆化
    }
}
```

##### 数组最小堆化

- **相关公式**：堆是建立在满二叉树上的, 对于长度为n的序列, 最后一个非叶子结点序号i = n/2 - 1, 其左孩子序号为2i+1, 右孩子序号为2i+2, 父结点序号为(i-1)/2。
- **最小堆化步骤**：

1. 从最后一个非叶子结点K位置（叶子结点没有孩子不用比较）开始。
2. 选择K位置的左右孩子，判断当前元素的值、左右孩子的值之中的最小者。
3. 如果最小者落在K位置，则可以直接插入当前元素；否则把最小者的值设置到K位置，K交换到最小者的位置，继续向下调整。
4. 直到原来K位置所在的树的所有结点都遍历完后，原K位置便是这棵树最小的值了。
5. 接着从下到上，从右到左，一直遍历到序列的根结点queue[0]，经过调整后，queue[0]即是整个序列中最小的值了。这时便完成了序列元素的最小堆化。

```java
// 数组最小堆化
private void heapify() {
    Object[] array = queue;
    int n = size;
    int half = (n >>> 1) - 1;// n/2 - 1, 从最后一个非结点开始
    Comparator<? super E> cmp = comparator;
    if (cmp == null) {
        for (int i = half; i >= 0; i--)// 直到根结点, 经过调整后, 整个树就成为了最小堆
            siftDownComparable(i, (E) array[i], array, n);// 自然排序
    }
    else {
        for (int i = half; i >= 0; i--)
            siftDownUsingComparator(i, (E) array[i], array, n, cmp);// 使用比较器
    }
}
```

##### 最小堆调整

**PriorityQueue的精髓所在，用于初始化、插入、删除时的堆调整，使其数组一直保持最小堆的结构**。

- **siftDownComparable（...）/siftDownUsingComparator（...）**：没有/有指定比较器的**向下调整小顶堆**, 使得插入后，当前树成为最小堆。

1. 判断没有指定比较器，没有比较器的使用自然排序比较规则，否则使用比较器的比较规则。
2. 选择当前位置的左右孩子，判断当前元素的值、左右孩子的值之中的最小者。
3. 如果最小者落在当前位置，则可以直接插入当前元素；否则把最小者的值设置到当前位置，当前位置交换到最小者的位置，继续向下调整。
4. 直到碰到合适的位置**（比下小时），插入当前元素**。

- **siftUpComparable（...）/siftUpUsingComparator（...）**：没有/有指定比较器的**向上调整小顶堆**, 插入指定元素到合适位置。

1. 判断没有指定比较器，没有比较器的使用自然排序比较规则，否则使用比较器的比较规则。
2. 选择当前位置的父结点，判断当前的值与父结点的值之中的最大者。
3. 如果最大者落在当前位置，则可以直接插入当前元素；否则把最大者的值设置到当前位置，当前位置交换到父结点位置，继续向上调整。
4. 直到碰到合适的位置**（比上大时），插入当前元素**。

```java
// 没有指定比较器时, 使用自然排序筛选比较: 向下调整小顶堆, 使得插入后, 当前树成为最小堆
// 参数解析: [当前根结点索引, 待插入元素, 原数组, 原数组长度]
private static <T> void siftDownComparable(int k, T x, Object[] array, int n) {
    if (n > 0) {
        Comparable<? super T> key = (Comparable<? super T>)x;
        int half = n >>> 1;           // loop while a non-leaf
        while (k < half) {
            int child = (k << 1) + 1; // assume left child is least 左孩子: 2k + 1
            Object c = array[child];
            int right = child + 1;// 右孩子: 2k + 2
            // 如果存在右孩子, 且左孩子比右孩子大, 则取右孩子(较小的一个)设置为待交换结点
            if (right < n && ((Comparable<? super T>) c).compareTo((T) array[right]) > 0)
                c = array[child = right];
            // 如果当前元素的值比待交换结点的值还小, 说明当前元素的值已经是最小的了, 适合插入, 则退出循环
            if (key.compareTo((T) c) <= 0)
                break;
            // 否则说明, 当前元素的值比左(右)孩子的大, 则带交换结点插入当前位置, 当前位置走下到待交换结点的位置, 继续循环比较新的左右孩子
            array[k] = c;
            k = child;
        }
        // 插入当前元素, 经过了上面的调整, 该位置一定是符合小顶堆的位置(比下小时)
        array[k] = key;
    }
}

// 有指定比较器时, 使用指定比较器进行筛选比较: 向下调整小顶堆, 使得插入后, 当前树成为最小堆
// 参数解析: [当前根结点索引, 待插入元素, 原数组, 原数组长度, 指定的比较器]
private static <T> void siftDownUsingComparator(int k, T x, Object[] array, int n, Comparator<? super T> cmp) {
    if (n > 0) {
        int half = n >>> 1;
        while (k < half) {
            int child = (k << 1) + 1;// 左孩子: 2k + 1
            Object c = array[child];
            int right = child + 1;// 右孩子: 2k + 2
            // 如果存在右孩子, 且左孩子比右孩子大, 则取右孩子(较小的一个)设置为待交换结点
            if (right < n && cmp.compare((T) c, (T) array[right]) > 0)
                c = array[child = right];
            // 如果当前元素的值比待交换结点的值还小, 说明当前元素的值已经是最小的了, 适合插入, 则退出循环
            if (cmp.compare(x, (T) c) <= 0)
                break;
            // 否则说明, 当前元素的值比左(右)孩子的大, 则带交换结点插入当前位置, 当前位置走下到待交换结点的位置, 继续循环比较新的左右孩子
            array[k] = c;
            k = child;
        }
        // 插入当前元素, 经过了上面的调整, 该位置一定是符合小顶堆的位置(比下小时)
        array[k] = x;
    }
}


// 没有指定比较器时, 使用自然排序筛选比较: 向上调整小顶堆, 直到有合适位置插入指定元素
private static <T> void siftUpComparable(int k, T x, Object[] array) {
    Comparable<? super T> key = (Comparable<? super T>) x;
    while (k > 0) {
        // 父结点序号 = (k - 1) / 2
        int parent = (k - 1) >>> 1;
        Object e = array[parent];
        // 如果当前元素的值大于父结点, 则可以直接跳出循环
        if (key.compareTo((T) e) >= 0)
            break;
        // 否则说明当前元素的值小于父结点, 则父结点插入当前位置, 当前位置走上父结点位置, 继续循环比较下一个父结点
        array[k] = e;
        k = parent;
    }
    // 插入当前元素, 经过了上面的调整, 该位置一定是符合小顶堆的位置(比上大时)
    array[k] = key;
}

// 有指定比较器时, 使用指定比较器进行筛选比较: 向上调整小顶堆, 直到有合适位置插入指定元素
private static <T> void siftUpUsingComparator(int k, T x, Object[] array, Comparator<? super T> cmp) {
    while (k > 0) {
        // 父结点序号 = (k - 1) / 2
        int parent = (k - 1) >>> 1;
        Object e = array[parent];
        // 如果当前元素的值大于父结点, 则可以直接跳出循环
        if (cmp.compare(x, (T) e) >= 0)
            break;
        // 否则说明当前元素的值小于父结点, 则父结点插入当前位置, 当前位置走上父结点位置, 继续循环比较下一个父结点
        array[k] = e;
        k = parent;
    }
    // 插入当前元素, 经过了上面的调整, 该位置一定是符合小顶堆的位置(比上大时)
    array[k] = x;
}
```

##### 迭代方法

- iterator（）：返回对该队列中的元素进行迭代的迭代器，**该迭代器不会以任何特定顺序返回元素**，因为最小堆只能保证堆顶最小，而不能保证孩子结点中的顺序。且返回的迭代器是**弱一致性的**。

```java
public Iterator<E> iterator() {
    return new Itr();
}
```

##### 扩容方法

- **扩容条件**：size >= queue.length，即当实际大小等于当前数组大小时。
- **tryGrow（Object[]，int）**：当旧容量（当前数组大小） < 64 时，扩容为容量 * 2 + 2；否则扩容为容量 * 1.5。

1. 释放主锁，对扩容自旋锁进行CAS操作，防止并发扩容。
2. 容量界限校验（内存溢出校验）：c < Integer.MAX_VALUE - 8。
3. 创建新容量数组，还原扩容自旋锁标志位为0。
4. 重新获取主锁，调用Arrays.copyOf（...）复制元素到新的数组，防止并发扩容以及扩容时其他线程并发做队列操作。

```java
private static final sun.misc.Unsafe UNSAFE;// 用于操作堆外内存
private static final long allocationSpinLockOffset;// 自旋锁对外内存偏移
static {
    try {
        UNSAFE = sun.misc.Unsafe.getUnsafe();
        Class<?> k = PriorityBlockingQueue.class;
        // native方法，获取自旋锁对外内存偏移量
        allocationSpinLockOffset = UNSAFE.objectFieldOffset
            (k.getDeclaredField("allocationSpinLock"));
    } catch (Exception e) {
        throw new Error(e);
    }
}

// 扩容方法
private void tryGrow(Object[] array, int oldCap) {
    // 这里必须先释放掉主锁的目的是为了提高性能：
    // 因为如果不释放主锁，CAS失败而走到了Thread.yield()方法，将会导致线程持有主锁进入阻塞态，从而导致其他线程都工作不了；但如果释放了锁，扩容期间，队列还是可以正常加减元素，直到扩容完成，重新获取主锁，复制元素到新数组，完成扩容
    lock.unlock(); // must release and then re-acquire main lock
    Object[] newArray = null;
    if (allocationSpinLock == 0 &&
        // CAS操作：将自旋锁从0修改为1
        UNSAFE.compareAndSwapInt(this, allocationSpinLockOffset, 0, 1)) {
        try {
            int newCap = oldCap + ((oldCap < 64) ?
                                   (oldCap + 2) : // grow faster if small
                                   (oldCap >> 1));
            if (newCap - MAX_ARRAY_SIZE > 0) {    // possible overflow
                int minCap = oldCap + 1;
                if (minCap < 0 || minCap > MAX_ARRAY_SIZE)
                    throw new OutOfMemoryError();// 内存溢出校验
                newCap = MAX_ARRAY_SIZE;
            }
            if (newCap > oldCap && queue == array)
                newArray = new Object[newCap];
        } finally {
            allocationSpinLock = 0;// 还原自旋锁标志为0
        }
    }
    if (newArray == null) // back off if another thread is allocating
        Thread.yield();
    lock.lock();
    if (newArray != null && queue == array) {
        queue = newArray;
        System.arraycopy(array, 0, newArray, 0, oldCap);
    }
}
```

##### 添加元素方法

所有方法都是使用ReentrantLock加锁同步实现。由于Queue数组保持着最小堆的结构，这时**插入元素到队尾**后，需要进行向上调整，插入后还需唤醒等待非空条件的线程。

- **add（E）**：Queue接口方法，添加一个元素，由于有扩容机制，所以不会触发队列已满异常。
- **offer（E）**：Queue接口方法，添加一个元素，添加元素到队尾。
- **put（E）**：BlockingQueue接口方法，添加一个元素，由于有扩容机制，所以永远不会阻塞。
- **offer（E，long，TimeUnit）**：BlockingQueue接口方法，添加一个元素，由于有扩容机制，所以永远不会阻塞。

```java
// Queue接口方法，添加元素到队尾，由于有扩容机制，所以不会触发队列已满异常
public boolean add(E e) {
    return offer(e);
}

// Queue接口方法，添加元素到队尾
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    final ReentrantLock lock = this.lock;
    lock.lock();
    int n, cap;
    Object[] array;
    while ((n = size) >= (cap = (array = queue).length))
        tryGrow(array, cap);// 扩容：当队列实际大小等于当前数组大小时触发
    try {
        Comparator<? super E> cmp = comparator;
        if (cmp == null)
            siftUpComparable(n, e, array);// 向上调整最小堆
        else
            siftUpUsingComparator(n, e, array, cmp);// 向上调整最小堆
        size = n + 1;
        notEmpty.signal();// 唤醒等待非空条件的线程
    } finally {
        lock.unlock();
    }
    return true;
}

// BlockingQueue接口方法，由于有扩容机制，所以永远不会阻塞
public void put(E e) {
    offer(e); // never need to block
}

// BlockingQueue接口方法，由于有扩容机制，所以永远不会阻塞
public boolean offer(E e, long timeout, TimeUnit unit) {
    return offer(e); // never need to block
}
```

##### 删除元素方法

所有方法都是使用ReentrantLock加锁同步实现。而**在元素出队（删除队头）后，唤醒等待非满条件的线程**。

由于Queue数组保持着最小堆的结构，这时**删除元素后，需要以队尾作为替代结点，插入当前位置并向下进行调整**。

- **remove（）**：Queue接口方法，交由抽象类实现AbstractQueue#remove（），当队列为空时，会抛出异常。
- **poll（）**：Queue接口方法，当队列为空时，不会抛出异常，而是返回null。
- **take（）**：BlockingQueue接口方法，队列为空时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
- **poll（long，TimeUnit）**：BlockingQueue接口方法，队列为空时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
- **remove（Object）**：Collection接口方法，删除第一个指定的元素，使用equals判断。

```java
// Queue接口方法，交由抽象类实现AbstractQueue#remove（），当队列为空时，会抛出异常。
public E remove() {
    E x = poll();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，当队列为空时，不会抛出异常，而是返回null。
public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return dequeue();// 队头出队
    } finally {
        lock.unlock();
    }
}
// 队头出队
private E dequeue() {
    int n = size - 1;
    if (n < 0)
        return null;
    else {
        Object[] array = queue;
        E result = (E) array[0];
        E x = (E) array[n];
        array[n] = null;
        Comparator<? super E> cmp = comparator;
        if (cmp == null)
            siftDownComparable(0, x, array, n);// 队头向下调整最小堆
        else
            siftDownUsingComparator(0, x, array, n, cmp);// 队头向下调整最小堆
        size = n;
        return result;
    }
}

// BlockingQueue接口方法，队列为空时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    E result;
    try {
        while ( (result = dequeue()) == null)
            notEmpty.await();// 队列为空时，会一直阻塞等待非满条件
    } finally {
        lock.unlock();
    }
    return result;
}

// BlockingQueue接口方法，队列为空时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    E result;
    try {
        while ( (result = dequeue()) == null && nanos > 0)
            // 队列为空时，会在限定时间内一直阻塞等待非满条件
            nanos = notEmpty.awaitNanos(nanos);
    } finally {
        lock.unlock();
    }
    return result;
}

// Collection接口方法，删除第一个指定的元素，使用equals判断
public boolean remove(Object o) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        int i = indexOf(o);// equals判断指定元素的索引
        if (i == -1)
            return false;
        removeAt(i);// 删除指定索引的元素
        return true;
    } finally {
        lock.unlock();
    }
}
// equals判断指定元素的索引
private int indexOf(Object o) {
    if (o != null) {
        Object[] array = queue;
        int n = size;
        for (int i = 0; i < n; i++)
            if (o.equals(array[i]))
                return i;
    }
    return -1;
}
// 删除指定索引的元素
private void removeAt(int i) {
    Object[] array = queue;
    int n = size - 1;
    if (n == i) // removed last element
        array[i] = null;
    else {
        E moved = (E) array[n];// 设置末尾结点为替代结点
        array[n] = null;// 清空末尾结点
        Comparator<? super E> cmp = comparator;
        if (cmp == null)
            // 向下调整小顶堆, 使得替代结点的值插入后，当前树成为最小堆
            siftDownComparable(i, moved, array, n);
        else
            siftDownUsingComparator(i, moved, array, n, cmp);

        // PriorityQueue.iterator#remove()才会触发下面代码
        // 如果当前树的根结点的值为替代结点的值，说明当前数组有可能不是最小堆的结构
        if (array[i] == moved) {
            if (cmp == null)
                // 所以接着向上调整小顶堆, 插入替代结点的值到合适位置
                siftUpComparable(i, moved, array);
            else
                siftUpUsingComparator(i, moved, array, cmp);
        }
    }
    size = n;
}
```

##### 获取元素方法

连检索的所有方法都是使用ReentrantLock加锁同步实现，用于检索队头元素。

- **element（）**：Queue接口方法，交由抽象类实现AbstractQueue#element（），当队列为空时，会抛出异常。
- **peek（）**：Queue接口方法，当队列为空时，不会抛出异常，而是返回null。

```java
// Queue接口方法，交由抽象类实现AbstractQueue#element（），当队列为空时，会抛出异常。
public E element() {
    E x = peek();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，当队列为空时，不会抛出异常，而是返回null。
public E peek() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return (size == 0) ? null : (E) queue[0];
    } finally {
        lock.unlock();
    }
}
```

#### DelayQueue

![1621778303598](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621778303598.png)

##### 特点

- DelayQueue，Delayed元素（标记了给定延迟后应作用的对象）的无界阻塞队列，其元素只能在其延迟到期后才能使用。**此队列不允许使用null元素。**
- 队列的头是该{@code Delayed}元素，其延迟在过去最远时过期。如果没有延迟，则没有Head，{@code poll}将返回{@code null}。
- 当元素的{@code getDelay（TimeUnit.NANOSECONDS）}方法返回的值小于或等于零时，就会发生过期。即使无法使用{@code take}或{@code poll}删除未过期的元素，也应该将未过期的元素视为普通元素。例如，{@code size}方法返回已过期和未过期元素的计数。
- 基于**数组快照方式**的迭代器，但不保证方法{@link #iterator（）}中提供的Iterator以任何特定顺序遍历DelayQueue的元素。
- 底层依赖PriorityQueue无界优先级队列，所以DelayQueue也是无界的。
- 队列没有替换元素的方法，且获取元素方法都是检索栈顶的元素。

##### 构造方法

- **空参的构造函数**：什么也不做。
- **指定复制集合的构造函数**：交由父类AbstractQueue#addAll(Collecion)实现，底层调用add（E）方法实现。

```java
public class DelayQueue<E extends Delayed> extends AbstractQueue<E> implements BlockingQueue<E> {
    
    private final transient ReentrantLock lock = new ReentrantLock();
    private final PriorityQueue<E> q = new PriorityQueue<E>();
    private Thread leader = null;// 用于等待队列开头元素的Leader线程
    private final Condition available = lock.newCondition();// 队列开头可用信号
    
    // 空参的构造函数
    public DelayQueue() {}
    
    // 指定复制集合的构造函数
    public DelayQueue(Collection<? extends E> c) {
        this.addAll(c);
    }
    // AbstractQueue#addAll(Collecion)
    public boolean addAll(Collection<? extends E> c) {
        if (c == null)
            throw new NullPointerException();
        if (c == this)
            throw new IllegalArgumentException();
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
}
```

##### 迭代方法

- iterator（）：回此队列中所有元素（**已过期和未过期**）的迭代器。 迭代器不会以任何特定顺序返回元素。且返回的迭代器是**弱一致性的**。

```java
public Iterator<E> iterator() {
    return new Itr(toArray());// 基于数组快照方式的迭代器
}
```

##### 扩容方法

底层依赖PriorityQueue无界优先级队列，扩容机制同PriorityQueue#扩容方法

##### 添加元素方法

所有方法都是使用ReentrantLock加锁同步实现。底层依赖PriorityQueue#offer（）实现。

添加后如果队头刚好为该Delay元素，需要**发出队头可用信号，唤醒等待线程**。

- **add（E）**：Queue接口方法，添加一个元素，由于有扩容机制，所以不会触发队列已满异常。
- **offer（E）**：Queue接口方法，添加一个元素，添加元素到队尾。
- **put（E）**：BlockingQueue接口方法，由于有扩容机制，所以永远不会阻塞。
- **offer（E，long，TimeUnit）**：BlockingQueue接口方法，由于有扩容机制，所以永远不会阻塞。

```java
// Queue接口方法，添加一个元素，由于有扩容机制，所以不会触发队列已满异常。
public boolean add(E e) {
    return offer(e);
}

// Queue接口方法，添加一个元素，添加元素到队尾。
public boolean offer(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 底层依赖PriorityQueue#offer（）实现
        q.offer(e);
        if (q.peek() == e) {
            leader = null;
            available.signal();// 发出队头可用信号，唤醒等待线程
        }
        return true;
    } finally {
        lock.unlock();
    }
}

// BlockingQueue接口方法，由于有扩容机制，所以永远不会阻塞。
public void put(E e) {
    offer(e);
}

// BlockingQueue接口方法，由于有扩容机制，所以永远不会阻塞。
public boolean offer(E e, long timeout, TimeUnit unit) {
    return offer(e);
}
```

##### 删除元素方法

所有方法都是使用ReentrantLock加锁同步实现。底层依赖PriorityQueue#peek（）与PriorityQueue#poll（）实现。

删除都重新检查了一遍队头，如果队头存在Delay元素，则发出队头可用信号，唤醒等待线程。

- **remove（）**：Queue接口方法，交由抽象类实现AbstractQueue#remove（），当队头没有Delay元素时，会抛出异常。
- **poll（）**：Queue接口方法，队头有过期元素，则底层调用PriorityQueue#offer（）返回，否则返回null。
- **take（）**：BlockingQueue接口方法，当队头没有Delay元素时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
- **poll（long，TimeUnit）**：BlockingQueue接口方法，当队头没有Delay元素时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
- **remove（Object）**：Collection接口方法，删除第一个指定的元素（过期的与未过期的都包括），使用equals判断。

```java
// Queue接口方法，交由抽象类实现AbstractQueue#remove（），当队头没有Delay元素时，会抛出异常。
public E remove() {
    E x = poll();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，队头有过期元素，则底层调用PriorityQueue#offer（）返回，否则返回null。
public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        E first = q.peek();
        if (first == null || first.getDelay(NANOSECONDS) > 0)
            return null;
        else
            return q.poll();// 队头有过期元素，则底层调用PriorityQueue#offer（）返回
    } finally {
        lock.unlock();
    }
}

// BlockingQueue接口方法，当队头没有Delay元素时，会一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            E first = q.peek();
            if (first == null)
                available.await();// 阻塞等待队头有过期元素
            else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0)
                    // 队头有过期元素，则底层调用PriorityQueue#offer（）返回
                    return q.poll();
                
                // 等待时不要保留引用
                first = null; // don't retain ref while waiting
                
                if (leader != null)
                    available.await();// 队头有Delay元素，但还没过期，继续等待信号
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        // 阻塞等待delay毫秒后，自旋重新poll（）
                        available.awaitNanos(delay);
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && q.peek() != null)
            available.signal();// 如果队头存在Delay元素，则发出队头可用信号，唤醒等待线程
        lock.unlock();
    }
}

// BlockingQueue接口方法，当队头没有Delay元素时，会在限定时间内一直阻塞等待非满条件（阻塞等待锁时可以被中断）。
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            E first = q.peek();
            if (first == null) {
                if (nanos <= 0)
                    return null;
                else
                    // 阻塞等待delay毫秒后，自旋重新poll（）
                    nanos = available.awaitNanos(nanos);
            } else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0)
                    // 队头有过期元素，则底层调用PriorityQueue#offer（）返回
                    return q.poll();
                if (nanos <= 0)
                    return null;
                first = null; // don't retain ref while waiting// 等待时不要保留引用
                if (nanos < delay || leader != null)
                    // 队头有Delay元素，但还没过期，继续等待信号
                    nanos = available.awaitNanos(nanos);
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        // 阻塞等待delay毫秒后，自旋重新poll（）
                        long timeLeft = available.awaitNanos(delay);
                        nanos -= delay - timeLeft;
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && q.peek() != null)
            available.signal();// 如果队头存在Delay元素，则发出队头可用信号，唤醒等待线程
        lock.unlock();
    }
}

// Collection接口方法，删除第一个指定的元素（过期的与未过期的都包括），使用equals判断。
public boolean remove(Object o) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return q.remove(o);
    } finally {
        lock.unlock();
    }
}
```

##### 获取元素方法

连检索的所有方法都是使用ReentrantLock加锁同步实现，用于检索队头Delay元素（含过期时间），注意队头元素还有可能还没过期，需要getDelay（TimeUnit）进行判断。

- **element（）**：Queue接口方法，交由抽象类实现AbstractQueue#element（），当队头没有Delay元素时，会抛出异常。
- **peek（）**：Queue接口方法，当队头没有Delay元素时，不会抛出异常，而是返回null。

```java
// Queue接口方法，交由抽象类实现AbstractQueue#element（），当队头没有Delay元素时，会抛出异常。
public E element() {
    E x = peek();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，当队头没有Delay元素时，不会抛出异常，而是返回null。
public E peek() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return q.peek();
    } finally {
        lock.unlock();
    }
}
```

#### DelayedWorkQueue

![1621781086648](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621781086648.png)

##### 特点

- java.util.concurrent.ScheduledThreadPoolExecutor.DelayedWorkQueue，是一个延迟线程池的内部类。
- DelayedWorkQueue，专门的延迟队列，基于小顶堆的数据结构，这样就消除了在取消时查找任务的需要，极大地加快了清除速度（从O（n）降到O（log n）），并减少了垃圾保留，否则将通过等待元素在清除之前上升到顶部而发生垃圾保留。
- 由于调加到队列中的元素，可能不是ScheduledFutureTasks的RunnableScheduledFuture（底层的Queue是RunnableScheduledFuture类型），所以并不能保证有这样的索引可用。
- **队列操作方法实现与DelayQueue、PriorityBlockingQueue类似**，都是线程安全，且基于小顶堆实现。
- 基于**数组快照方式**的迭代器，但不会以任何特定顺序返回元素，因为最小堆只能保证堆顶最小，而不能保证孩子结点中的顺序。
- 队列没有替换元素的方法，且获取元素方法都是检索栈顶的元素。

##### 构造方法

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

##### 迭代方法

- iterator（）：基于**数组快照方式**的迭代器，但不会以任何特定顺序返回元素，因为最小堆只能保证堆顶最小，而不能保证孩子结点中的顺序。

```java
public Iterator<Runnable> iterator() {
    return new Itr(Arrays.copyOf(queue, size));// 基于数组快照方式的迭代器
}
```

##### 扩容方法

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

#### SynchronousQueue

![1621785755857](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621785755857.png)

##### 特点

- SynchronousQueue，**同步队列**，每个插入操作必须等待另一个线程进行相应的删除操作，反之亦然。同步队列没有任何内部容量，甚至没有一个容量（**即该容器不能获取元素**）：
  - 无法在同步队列中{@code peek}，因为仅在尝试删除它时，该元素才存在。
  - 不能使用任何方法插入元素，除非另一个线程试图将其删除。
  - 无法进行迭代，因为没有要迭代的内容。 
  - 队列的头部是第一个排队的插入线程试图添加到队列中的元素，如果没有这样的排队线程，则没有元素可用于删除，即{@code poll（）}将返回{@code null}。
  - 出于其他{@code Collection}方法（例如{@code contains}）的目的，{@code SynchronousQueue}用作空集合。
  - **不允许{@code null}元素**。
- 同步队列的四种操作方法还是维持着队列方法的特性，经测试可得以下结论：
  - add(E)与remove()： 还是会抛出异常, 一对且非并发使用时, 会永远抛出异常。
  - offer(E)与poll()：还是不会抛出异常, 一对且非并发使用时, 会永远返回false。
  - put(E)与take()：还是会阻塞等待直到成功, 一对且非并发使用时, take可以取到值, 而put没有返回值。
  - offer(E, long, TimeUnit)与poll(long, TimeUnit)：还是会等待指定时间后返回, 一对且非并发使用时, poll可以取到值, offer也可以为true。
- 同步队列**非常适合切换设计**，在该设计中，在一个线程中运行的对象必须与在另一个线程中运行的对象同步以便向其传递一些信息，事件或任务。
- 支持可选的公平性策略，用于正在等待的已订阅的生产者和使用者线程。 **默认情况下是非公平的**。可以通过使用公平性设置为{@code true}构造的队列将按FIFO顺序授予线程访问权限。
- 利用自旋 + LockSupport方式阻塞 + CAS乐观锁方式实现栈或者队列结构，全程没有使用悲观锁也能保证同步，吞吐量高。

##### 构造方法

- **无参的构造函数**：默认非公平方式，即栈结构（后进先出）。
- **指定公平方式的构造函数**：true代表公平（队列结构），false代表非公平（栈结构）。

```java
public class SynchronousQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {
    
    // 双重数据结构：栈 & 队列结构的共同API
    abstract static class Transferer<E> {
        // 添加或者删除元素
        abstract E transfer(E e, boolean timed, long nanos);
    }
    
    static final int NCPUS = Runtime.getRuntime().availableProcessors();
    static final int maxTimedSpins = (NCPUS < 2) ? 0 : 32;
    static final int maxUntimedSpins = maxTimedSpins * 16;
    static final long spinForTimeoutThreshold = 1000L;
    private transient volatile Transferer<E> transferer;// 转移者（双重数据结构）
    
    // 无参的构造函数：默认非公平方式，即栈结构（后进先出）
    public SynchronousQueue() {
        this(false);
    }
    
    // 指定公平方式的构造函数：true代表公平（队列结构），false代表非公平（栈结构）
    public SynchronousQueue(boolean fair) {
        transferer = fair ? new TransferQueue<E>() : new TransferStack<E>();
    }
}
```

##### 栈结构实现

![1621954377695](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621954377695.png)

**栈结构的实现是同步队列非公平方式实现的灵魂所在，使用自旋锁提高吞吐量，结合LockSupport提供线程的阻塞等待，利用添加和删除的结点配对方式实现两操作的同步。**

上图为put（E）+ put（E）+ poll（）三步操作的流程概况。

下面是put（E） + poll（）的解释：

1. put线程：构造结点 s=snode[item=e, next=null, mode=1] ，CAS移动头结点。
2. put线程：CAS移动头结点成功, 则自旋（512次） + 设置s结点的waiter阻塞线程 + LockSupport.park阻塞当前线程。
3. poll线程：构造结点 s=snode[item=e, next=原来的头结点, mode=0]，CAS移动头结点(相当于压栈)。
4. poll线程：压栈成功后, 自旋，**设置put结点的配对结点为当前poll结点** + 唤醒put()阻塞的线程。如果配对成功，则交换put结点的下一个结点成为新的头结点，返回put结点的元素；否则继续配对put结点的下一个结点。
5. put线程：此时与唤醒当前线程的poll线程同步进行，由于put结点的配对结点已经不为null了，所以返回配对的**poll结点**，结束用于put阻塞等待的自旋。返回后经过判断，最终返回put结点的元素，结束put的自旋。

而take（）+ offer（E）的流程与put（E） + poll（）的相反，take（）返回的是offer（E）的元素，offer（E）返回的还是offer（E）的元素。

```java
E transfer(E e, boolean timed, long nanos) {
    SNode s = null; // constructed/reused as needed
    int mode = (e == null) ? REQUEST : DATA;

    for (;;) {
        SNode h = head;

        // 33. eg: 再offer(E), 这时的mode为1, h为take()的s, h.mode=0
        // 24. eg: 先take(), 这时的mode为0, h为null
        // 11. eg: poll(): 这时的mode为0, h为put()的s, h.mode = 1
        // 0. 一开始头结点为空
        if (h == null || h.mode == mode) {  // empty or same-mode
            // 1. put()与take()为false, 其他为true, 且offer(E)与poll()的超时时间为0
            if (timed && nanos <= 0) {      // can't wait
                if (h != null && h.isCancelled())
                    casHead(h, h.next);     // pop cancelled node
                else
                    return null;
            }
            
            // 25. eg: take(): 构造结点 s=snode[item=e, next=null, mode=0] => 再CAS移动头结点
            // 2. eg: put(): 构造结点 s=snode[item=e, next=null, mode=1] => 再CAS移动头结点
            else if (casHead(h, s = snode(s, e, h, mode))) {
                
                // 26.CAS移动头结点成功, 则自旋 + 设置s结点的阻塞线程 + LockSupport.park阻塞当前线程
                // 3.CAS移动头结点成功, 则自旋 + 设置s结点的阻塞线程 + LockSupport.park阻塞当前线程
                SNode m = awaitFulfill(s, timed, nanos);

                // 41. 与唤醒当前线程的take()线程同步进行: m为take()时的匹配结点, eg: m != s
                // 21. 与唤醒当前线程的poll()线程同步进行: m为poll()时的匹配结点, eg: m != s
                if (m == s) {               // wait was cancelled
                    clean(s);
                    return null;
                }
                
                // 42. 与唤醒当前线程的take()线程同步进行: eg: 此时head为null
                // 22. 与唤醒当前线程的poll()线程同步进行: eg: 此时head为null
                if ((h = head) != null && h.next == s)
                    casHead(h, s.next);     // help s's fulfiller

                // 43. 与唤醒当前线程的take()线程同步进行: eg: take(): mode=0, 返回offer(E)时的元素
                // 23. 与唤醒当前线程的poll()线程同步进行: eg: put(): mode=1, 返回put()时的元素
                return (E) ((mode == REQUEST) ? m.item : s.item);
            }
        }
        
        // 34. 判断是否为配对模式, eg: offer(E): false
        // 12. 判断是否为配对模式, eg: poll(): false
        else if (!isFulfilling(h.mode)) { // try to fulfill
            
            // 13. 如果当前头结点的配对结点等于头结点自身, 则认为可以取消头结点了(见6. 如果当前线程被中断, 则将设置当前配对结点为当前结点)
            // eg: poll(): false
            if (h.isCancelled())            // already cancelled
                casHead(h, h.next);         // pop and retry
            
            // 35. 再次判断当前模式是否为配对模式, 构造结点 s=snode[item=e, next=take()的结点, mode=0] => 再CAS移动头结点(相当于压栈)
            // 14. 再次判断当前模式是否为配对模式, 构造结点 s=snode[item=e, next=原来的头结点, mode=0] => 再CAS移动头结点(相当于压栈)
            else if (casHead(h, s=snode(s, e, h, FULFILLING|mode))) {
                
                // 36. 压栈成功后, 自旋
                // 15. 压栈成功后, 自旋
                for (;;) { // loop until matched or waiters disappear
                    // 37. 获取原来的头结点, eg: offer(E): s
                    // 16. 获取原来的头结点, eg: poll(): s
                    SNode m = s.next;       // m is s's match
                    if (m == null) {        // all waiters are gone
                        casHead(s, null);   // pop fulfill node
                        s = null;           // use new node next time
                        break;              // restart main loop
                    }

                    // 38. 获取原来头结点的下一个结点, eg: offer(E): null, 将原来结点配对当前结点 + 唤醒take()阻塞的线程
                    // 17. 获取原来头结点的下一个结点, eg: poll(): null, 将原来结点配对当前结点 + 唤醒put()阻塞的线程
                    SNode mn = m.next;
                    if (m.tryMatch(s)) {
                        
                        // 39. 与唤醒的take()线程同时进行: 配对成功, eg: 交换null作为新的头结点
                        // 19. 与唤醒的put()线程同时进行: 配对成功, eg: 交换null作为新的头结点
                        casHead(s, mn);     // pop both s and m
                        // 40. 与唤醒的take()线程同时进行: offer(E)添加成功, 返回当前offer(E)的元素。
                        // 20. 与唤醒的put()线程同时进行: poll()删除成功, 返回put()时的元素
                        return (E) ((mode == REQUEST) ? m.item : s.item);
                    } else                  // lost match
                        // 19. 如果没匹配上, 则继续找下一个结点匹配
                        s.casNext(m, mn);   // help unlink
                }
            }
        } else {                            // help a fulfiller
            SNode m = h.next;               // m is h's match
            if (m == null)                  // waiter is gone
                casHead(h, null);           // pop fulfilling node
            else {
                SNode mn = m.next;
                if (m.tryMatch(h))          // help match
                    casHead(h, mn);         // pop both h and m
                else                        // lost match
                    h.casNext(m, mn);       // help unlink
            }
        }
    }
}

// 自旋/阻塞，直到节点s通过执行操作被匹配。
SNode awaitFulfill(SNode s, boolean timed, long nanos) {
    // 27. eg: take(): time=false
    // 4. eg: put(): timed=false
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    Thread w = Thread.currentThread();

    // 28. 是否应该自旋: 当前结点为头结点 | 头结点为null | 当前模式为配对模式时 => true, 自旋次数为32 * 16 = 512
    // 5. 是否应该自旋: 当前结点为头结点 | 头结点为null | 当前模式为配对模式时 => true, 自旋次数为32 * 16 = 512
    int spins = (shouldSpin(s) ? (timed ? maxTimedSpins : maxUntimedSpins) : 0);

    // 39. 与唤醒当前线程的offer(E)线程同步进行: 继续自旋
    // 19. 与唤醒当前线程的poll()线程同步进行: 继续自旋
    for (;;) {
        
        // 6. 如果当前线程被中断, 则将设置当前配对结点为当前结点, eg: put() => false
        if (w.isInterrupted())
            s.tryCancel();

        // 40. 与唤醒当前线程的offer(E)线程同步进行: 见(38. 如果当前结点为null, 则还需要CAS设置为传入的结点为匹配结点 + 唤醒take()阻塞的线程),
        //     eg: 此时m=take()时的结点, 并返回m, 结束自旋
        // 29. eg: take(), 配对结点为null
        // 20. 与唤醒当前线程的poll()线程同步进行: 见(18. 如果当前结点为null, 则还需要CAS设置为传入的结点为匹配结点 + 唤醒put()阻塞的线程),
        //     eg: 此时m=poll()时的结点, 并返回m, 结束自旋
        // 7. 获取配对的结点, eg: put() => null
        SNode m = s.match;
        if (m != null)
            return m;
        if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                s.tryCancel();
                continue;
            }
        }
        
        // 30. 重新计算自旋次数, eg: put() => 512 - 1 = 511...
        // 8. 重新计算自旋次数, eg: put() => 512 - 1 = 511...
        if (spins > 0)
            spins = shouldSpin(s) ? (spins-1) : 0;
        
        // 31. 如果自旋结束(阻塞512次或者头结点更新了), 判断当前结点有没有设置阻塞线程, eg: take(): null => 设置waiter为当前线程
        // 9. 如果自旋结束(阻塞512次或者头结点更新了), 判断当前结点有没有设置阻塞线程, eg: put(): null => 设置waiter为当前线程
        else if (s.waiter == null)
            s.waiter = w; // establish waiter so can park next iter
        
        // 32. 如果自旋结束了, 且也设置了阻塞线程, eg: take(): false => 调用LockSupport.part进行阻塞
        // 10. 如果自旋结束了, 且也设置了阻塞线程, eg: put(): false => 调用LockSupport.part进行阻塞
        else if (!timed)
            LockSupport.park(this);
        else if (nanos > spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanos);
    }
}
```

##### 队列结构实现

![1622033076546](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1622033076546.png)

**队列结构的实现是同步队列公平方式实现的灵魂所在，使用自旋锁提高吞吐量，结合LockSupport提供线程的阻塞等待，利用删除时清空添加的元素，添加时设置删除的元素，实现两操作的同步，其中后一个操作不会真正地构造元素，一次匹配只有前一个操作才会构造结点，同时构造出来的结点匹配后会成为新的头结点，可以减少GC的次数。**

上图为put（E）+ poll（）操作的流程概况。

下面是put（E） + poll（）的解释：

1. put线程：构造结点 s=Qnode[e, true] ，CAS设置next指针，相当于s入队。
2. put线程：入队成功后, 则自旋（512次） + 设置s结点的waiter阻塞线程+ LockSupport.park阻塞当前线程。
3. poll线程：清空put的结点元素，交换put结点为新的头结点，唤醒put线程，返回put时的元素。
4. put线程：此时与唤醒当前线程的poll线程同步进行，由于put的元素被poll置空了，所以不等于原来的put节点了，接着返回put时的元素。

而take（）+ offer（E）的流程与put（E） + poll（）的相反，take（）返回的是offer（E）设置的元素，offer（E）返回的还是offer（E）本身的元素。

```java
E transfer(E e, boolean timed, long nanos) {
    // 35. 最后再offer(E), isData=true
    // 23. 再反过来, 先take(), isData=false
    // 13. poll(), isData=false
    // 0. put(E), isData=true
    QNode s = null; // constructed/reused as needed
    boolean isData = (e != null);

    for (;;) {
        // 36. offer(E), 此时head=上次put的结点, tail=take时的结点
        // 24. 这时head和tail都为上次put的结点, isData=true
        // 14. poll(), 此时head=初始的head, tail=put时的结点
        // 1. head和tail在构造函数就创建了, head=tail=new node(null, false)
        QNode t = tail;
        QNode h = head;
        if (t == null || h == null)         // saw uninitialized value
            continue;                       // spin

        // 37. t.isData为take的isData即false, 不等于现在的true
        // 25. 这里的h也等于t
        // 15. t.isData为put的isData即true, 不等于现在的false
        // 2. 这里h等于t
        if (h == t || t.isData == isData) { // empty or same-mode
            
            // 26. tn也为null
            // 3. tn为null
            QNode tn = t.next;
            if (t != tail)                  // inconsistent read
                continue;
            if (tn != null) {               // lagging tail
                advanceTail(t, tn);
                continue;
            }
            if (timed && nanos <= 0)        // can't wait
                return null;
            
            // 27. s为null, 构造s结点[null, false]
            // 4. s为null, 构造s结点[e, true]
            if (s == null)
                s = new QNode(e, isData);
            
            // 28. s入队, 入队成功则CAS社会队尾为s结点, 设置s为新的队尾
            // 5. s入队, 入队成功则CAS社会队尾为s结点, 设置s为新的队尾
            if (!t.casNext(null, s))        // failed to link in
                continue;
            advanceTail(t, s);              // swing tail and wait

            // 43. 与offer(E)线程同时进行: item被offer(E)线程赋值了, x为offer的元素e
            // 34. 自旋 + 设置waiter阻塞线程 + LockSupport.park阻塞take线程
            // 20. 与poll()线程同时进行: item被poll()线程置空了, x为null
            // 12. 自旋 + 设置waiter阻塞线程 + LockSupport.park阻塞put线程
            Object x = awaitFulfill(s, e, timed, nanos);

            // 44. x为e, s为take的结点
            // 21. x为null, s为put的结点
            if (x == s) {                   // wait was cancelled
                clean(t, s);
                return null;
            }

            // 45. s为新的头和尾结点, x为e, 则返回x, 即为offer(E)时的元素
            // 22. s为新的头和尾结点, x为null, 则返回put时的元素
            if (!s.isOffList()) {           // not already unlinked
                advanceHead(t, s);          // unlink if head
                if (x != null)              // and forget fields
                    s.item = s;
                s.waiter = null;
            }
            return (x != null) ? (E)x : e;

        }
        else {                            // complementary-mode
            
            // 38. t等于tail, m为t, h为head
            // 16. t等于tail, m为t, h为head
            QNode m = h.next;               // node to fulfill
            if (t != tail || m == null || h != head)
                continue;                   // inconsistent read

            // 39. x为take时的元素, isData=true, x为空, x不为take的结点即还没取消put的线程, 则尝试CAS交换x为当前offer(E)的e
            // 17. x为put时的元素, isData=false, x不为空, x不为put的结点即还没取消put的线程, 则尝试CAS交换x为null
            Object x = m.item;
            if (isData == (x != null) ||    // m already fulfilled
                x == m ||                   // m cancelled
                !m.casItem(x, e)) {         // lost CAS
                advanceHead(h, m);          // dequeue and retry
                continue;
            }

            // 40. 设置了take时的元素为当前元素e后, 继续交换take的结点为头结点(这时与原头结点差不多, 但isData为false)
            // 18. 置空put时的元素后, 继续交换put的结点为头结点(这时与原头结点差不多, 但isData为true)
            advanceHead(h, m);              // successfully fulfilled
            
            // 41. 唤醒take()线程
            // 19. 唤醒put(E)线程
            LockSupport.unpark(m.waiter);
            
            // 42. 与take()线程同时进行: x为null, 这时返回当前元素e, 结束offer(E)方法
            // 20. 与put(E)线程同时进行: x不为null, 为put时的元素, 这时返回x, 结束poll()方法
            return (x != null) ? (E)x : e;
        }
    }
}

// 自旋/阻塞，直到满足节点s为止。
Object awaitFulfill(QNode s, E e, boolean timed, long nanos) {
    // 29. take(): 插入的s结点, 当前插入的元素, false, 0 => deadline = 0
    // 6. put(E): 插入的s结点, 当前插入的元素, false, 0 => deadline = 0
    /* Same idea as TransferStack.awaitFulfill */
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    Thread w = Thread.currentThread();

    // 30. take(): 此时head.next确实等于s, timed=false => 自旋32*16=512次
    // 7. put(E): 此时head.next确实等于s, timed=false => 自旋32*16=512次
    int spins = ((head.next == s) ? (timed ? maxTimedSpins : maxUntimedSpins) : 0);
    for (;;) {
        if (w.isInterrupted())
            // 如果线程被中断的话, 则设置item为结点
            s.tryCancel(e);

        // 42. 与offer(E)线程同时进行: item被offer(E)线程赋值了, x不为null了, 代表与offer(E)配对了, 这时返回offer的元素e
        // 31. take(): 这时x确实为s的元素, 即没有被中断也没有offer(E)进行配对
        // 20. 与poll()线程同时进行: item被poll()线程置空了, x不为e了, 代表与poll()配对了, 这时返回null
        // 8. put(E): 这时x确实为s的元素, 即没有被中断也没有poll()进行配对
        Object x = s.item;
        if (x != e)
            return x;
        if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                s.tryCancel(e);
                continue;
            }
        }
        
        // 32. 更新自旋次数, 继续自旋
        // 9. 更新自旋次数, 继续自旋
        if (spins > 0)
            --spins;
        
        // 33. 自旋结束后, 设置waiter阻塞线程为当前take()线程
        // 10. 自旋结束后, 设置waiter阻塞线程为当前put(E)线程
        else if (s.waiter == null)
            s.waiter = w;
        
        // 33. 如果自旋结束, 也设置了waiter线程, timed又为false, 则调用LockSupport.park阻塞当前线程
        // 11. 如果自旋结束, 也设置了waiter线程, timed又为false, 则调用LockSupport.park阻塞当前线程
        else if (!timed)
            LockSupport.park(this);
        else if (nanos > spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanos);
    }
}
```

##### 迭代方法

- **iterator（）**：始终返回一个空的迭代器。因为同步队列没有元素获取。

```java
public Iterator<E> iterator() {
    return Collections.emptyIterator();
}
```

##### 扩容方法

同步队列没有容量，即获取不了元素，没有扩容方法。而同步队列的实现基于链表的结构，实现时也不需要扩容。

##### 添加元素方法

所有方法成功执行后，都不是真正的添加元素，而是会阻塞到删除操作的执行，且阻塞都是基于自旋锁实现。

- **add（E）**：Queue接口方法，添加一个元素，如果没有同时删除元素，则会抛出异常。
- **offer（E）**：Queue接口方法，添加一个元素，如果没有同时删除元素，则会返回false。
- **put（E）**：BlockingQueue接口方法，添加一个元素，会一直阻塞到中断或者删除元素。
- **offer（E，long，TimeUnit）**：BlockingQueue接口方法，添加一个元素，会在限定时间内阻塞到中断或者删除元素，否则返回false。

```java
// Queue接口方法，添加一个元素，如果没有同时删除元素，则会抛出异常
public boolean add(E e) {
    if (offer(e))
        return true;
    else
        throw new IllegalStateException("Queue full");
}

// Queue接口方法，添加一个元素，如果没有同时删除元素，则会返回false
public boolean offer(E e) {
    if (e == null) throw new NullPointerException();
    return transferer.transfer(e, true, 0) != null;
}

// BlockingQueue接口方法，添加一个元素，会一直阻塞到中断或者删除元素
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    if (transferer.transfer(e, false, 0) == null) {
        Thread.interrupted();
        throw new InterruptedException();
    }
}

// BlockingQueue接口方法，添加一个元素，会在限定时间内阻塞到中断或者删除元素，否则返回false
public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {
    if (e == null) throw new NullPointerException();
    if (transferer.transfer(e, true, unit.toNanos(timeout)) != null)
        return true;
    if (!Thread.interrupted())
        return false;
    throw new InterruptedException();
}
```

##### 删除元素方法

所有方法成功执行后，都会成对删除元素（一个删除时新增的空元素与一个与之配对的元素），会唤醒一个等待山删除的添加线程，同样都是删除时也会阻塞，也是基于自旋锁实现。

- **remove（）**：Queue接口方法，删除头元素，如果没有同时添加元素，则会抛出异常。
- **poll（）**：Queue接口方法，删除头元素，如果没有同时添加元素，则会返回false。
- **take（）**：BlockingQueue接口方法，删除头元素，会一直阻塞到中断或者添加元素的执行。
- **poll（long，TimeUnit）**：BlockingQueue接口方法，删除头元素，会在限定时间内阻塞到中断或者添加元素的执行，否则返回false。
- **remove（Object）**：Collection接口方法，不允许删除指定的元素，统一返回false。

```java
// Queue接口方法，删除头元素，如果没有同时添加元素，则会抛出异常
public E remove() {
    E x = poll();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，删除头元素，如果没有同时添加元素，则会返回false
public E poll() {
    return transferer.transfer(null, true, 0);
}

// BlockingQueue接口方法，删除头元素，会一直阻塞到中断或者添加元素的执行
public E take() throws InterruptedException {
    E e = transferer.transfer(null, false, 0);
    if (e != null)
        return e;
    Thread.interrupted();
    throw new InterruptedException();
}

// BlockingQueue接口方法，删除头元素，会在限定时间内阻塞到中断或者添加元素的执行，否则返回false
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    E e = transferer.transfer(null, true, unit.toNanos(timeout));
    if (e != null || !Thread.interrupted())
        return e;
    throw new InterruptedException();
}

// Collection接口方法，不允许删除指定的元素，统一返回false
public boolean remove(Object o) {
    return false;
}
```

##### 获取元素方法

同步队列没有容量，获取元素、判断元素、获取实际大小等除了添加和删除的方法外，统统都返回0/false/null。

- **element（）**：Queue接口方法，交由抽象类实现AbstractQueue#element（），同步队列没有元素，默认会抛出异常。
- **peek（）**：Queue接口方法，同步队列没有元素，默认返回null。

```java
// Queue接口方法，交由抽象类实现AbstractQueue#element（），同步队列没有容量，默认会抛出异常。
public E element() {
    E x = peek();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，同步队列没有容量，默认返回null。
public E peek() {
    return null;
}
```

#### LinkedTransferQueue

![1622347602561](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1622347602561.png)

##### 特点

- LinkedTransferQueue，**基于链表实现的无界消息阻塞队列**，该队列根据任何给定的生产者对元素FIFO（先进先出）进行排序。队列的头部是某个生产者在队列中停留时间最长的元素。队列的尾部是某个生产者在队列中停留时间最短的那个元素，因为是从尾部添加，从头部删除结点的，其中**一个新增的元素会与一个删除的空元素配对**。
- LinkedTransferQueue，既有**SynchronousQueue的同步队列中生产者消费者**的特性，也包含了**LinkedTransferQueue对高并发无锁CAS + 自旋的优化，通过跳过2个结点才更新head指针**，减少一半线程的CPU自旋损耗，又利用LockSupport.park（）来实现**BlockingQueue的阻塞特性**，还提供了**异步非阻塞、延迟消息/消费**的方法。
- 请注意，与大多数集合不同，{@code size} 方法不是恒定时间操作。 由于这些队列的异步性质，确定当前元素数量需要遍历元素，因此如果在遍历期间修改此集合，则可能会报告不准确的结果。 
- 此外，批量操作 {@code addAll}、{@code removeAll}、{@code retainAll}、{@code containsAll}、{@code equals} 和{@code toArray} 不能保证以原子方式执行。 例如，与 {@code addAll} 操作同时运行的迭代器可能只查看一些添加的元素。
- 队列没有替换元素的方法，且获取元素方法都是检索队头的元素。

##### 构造方法

- **无参的构造函数**：什么也不做。
- **指定复制集合的构造函数**：遍历集合、调用add（）调价每一个元素。

```java
public class LinkedTransferQueue<E> extends AbstractQueue<E> implements TransferQueue<E>, java.io.Serializable {
    
    private static final boolean MP = Runtime.getRuntime().availableProcessors() > 1;
    private static final int FRONT_SPINS   = 1 << 7;
    private static final int CHAINED_SPINS = FRONT_SPINS >>> 1;
    static final int SWEEP_THRESHOLD = 32;
    
    static final class Node {
        final boolean isData;
        volatile Object item;
        volatile Node next;
        volatile Thread waiter;
        
        private static final sun.misc.Unsafe UNSAFE;
        private static final long itemOffset;
        private static final long nextOffset;
        private static final long waiterOffset;
        static {
            try {
                UNSAFE = sun.misc.Unsafe.getUnsafe();
                Class<?> k = Node.class;
                itemOffset = UNSAFE.objectFieldOffset
                    (k.getDeclaredField("item"));
                nextOffset = UNSAFE.objectFieldOffset
                    (k.getDeclaredField("next"));
                waiterOffset = UNSAFE.objectFieldOffset
                    (k.getDeclaredField("waiter"));
            } catch (Exception e) {
                throw new Error(e);
            }
        } 
    }
    
    transient volatile Node head;
    private transient volatile Node tail;
    private transient volatile int sweepVotes;
    
    private static final int NOW   = 0; // for untimed poll, tryTransfer
    private static final int ASYNC = 1; // for offer, put, add 
    private static final int SYNC  = 2; // for transfer, take
    private static final int TIMED = 3; // for timed poll, tryTransfer
    
    private static final sun.misc.Unsafe UNSAFE;
    private static final long headOffset;
    private static final long tailOffset;
    private static final long sweepVotesOffset;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> k = LinkedTransferQueue.class;
            headOffset = UNSAFE.objectFieldOffset
               (k.getDeclaredField("head"));
            tailOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("tail"));
            sweepVotesOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("sweepVotes"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
    
    // 无参的构造函数
    public LinkedTransferQueue() {
        
    }
    
    // 指定复制集合的构造函数
    public LinkedTransferQueue(Collection<? extends E> c) {
        this();
        addAll(c);
    }
    public boolean addAll(Collection<? extends E> c) {
        if (c == null)
            throw new NullPointerException();
        if (c == this)
            throw new IllegalArgumentException();
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
    public boolean add(E e) {
        xfer(e, true, ASYNC, 0);
        return true;
    }
}
```

##### 迭代方法

- **iterator（）**：以适当的顺序返回此队列中元素的迭代器。 元素将按从第一个（头）到最后一个（尾）的顺序返回。**返回的迭代器弱一致**。

```java
public Iterator<E> iterator() {
    return new Itr();
}
```

##### 扩容方法

LinkedTransferQueue，无界消息阻塞队列，基于链表实现，没有边界，无需扩容。

##### 核心方法

由此可见，LinkedTransferQueue，既有**SynchronousQueue的同步队列中生产者消费者**的特性，也包含了**LinkedTransferQueue对高并发无锁CAS + 自旋的优化，通过跳过2个结点才更新head指针**，减少一半线程的CPU自旋损耗，又利用LockSupport.park（）来实现**BlockingQueue的阻塞特性**，还提供了**异步非阻塞、延迟消息/消费**的方法。

下面是offer（E）* 2 + poll（）* 2的解释：

1. offer 1线程：构造结点 s=Node[e, true] ，调用tryAppend（Node，boolean），CAS设置head指针，相当于s入队，并成为head结点，返回添加的元素。
2. offer 2线程：构造结点 s=Node[e, true] ，调用tryAppend（Node，boolean），CAS设置head.next指针，返回的元素。
3. poll 1线程：CAS置null head结点的item，唤醒head的阻塞线程，但由于offer 1线程添加是异步非阻塞的添加，所以此时head的阻塞线程为null，即什么也不做，然后返回offer 1添加的元素。而head指针的移动交由下一个poll线程完成。
4. poll 2线程：判断到当前head的item为null，不匹配它自身的isData为true，所以继续向下遍历找到offer 2的元素，CAS置null该结点的item。然后CAS设置head指针为当前结点offer 2的结点，即更新了head指针，并自链接offer 1的结点。唤醒当前offer 2添加结点的阻塞线程，但由于offer 2线程添加是异步非阻塞的添加，所以此结点的阻塞线程为null，即什么也不做，然后返回offer 2添加的元素。

```java
private E xfer(E e, boolean haveData, int how, long nanos) {

    // 35. eg: poll() * 2: null, false, NOW, 0
    // 26. eg: poll(): null, false, NOW, 0
    // 9. eg: offer(E): e, true, ASYNC, 0
    // 0. eg: offer(E): e, true, ASYNC, 0
    if (haveData && (e == null))
        throw new NullPointerException();
    Node s = null;                        // the node to append, if needed

    retry:
    for (;;) {                            // restart on append race
        // 40. eg: poll() * 2: 继续自旋后, h=p=head=offer(E) * 2的s, 不为null
        // 36. eg: poll() * 2: h=p=head=offer(E)的s(这时s的item已经被上一个poll线程置null了), 不为null
        // 27. eg: poll(): h=p=head=offer(E)的s, 不为null
        // 10. eg: offer(E) * 2: h=p=head=offer(E)的s, 不为null
        // 1. eg: offer(E): h=p=head == null
        for (Node h = head, p = h; p != null;) { // find & match first node

            // 41. eg: poll() * 2: isData = true, item=offer(E) * 2的s.item
            // 37. eg: poll() * 2: isData = true, item=offer(E)的s.item, 但已经被上一个poll线程置null了, 所以item=null
            // 28. eg: poll(): isData = true, item=offer(E)的s.item
            // 11. eg: offer(E) * 2: isData=true, item=offer(E)的s.item
            boolean isData = p.isData;
            Object item = p.item;

            // 42. eg: poll() * 2: item=offer(E) * 2的s.item, p=head.next, item确实不为p, item!=null => true, 符合isData
            // 38. eg: poll() * 2: item=null, p=head, item确实不为p, item!=null => false, 不符合isData
            // 29. eg: poll(): item=offer(E)的s.item, p=head, item确实不为p, item!=null => true, 符合isData
            // 12. eg: offer(E) * 2: item确实不等于p, item不为null, 确认符合isData=true
            if (item != p && (item != null) == isData) { // unmatched

                // 43. eg: poll() * 2: isData=true, haveData=false => false
                // 30. eg: poll(): isData=true, haveData=false => false
                // 13. eg: offer(E) * 2: isData == haveData == true, 所以退出循环, 表示不能匹配
                if (isData == haveData)   // can't match
                    break;

                // 44. eg: poll() * 2: CAS设置head.next的item为null
                // 31. eg: poll(): CAS设置head的item为null
                if (p.casItem(item, e)) { // match

                    // 45. eg: poll() * 2: q=p=head.next, h=head => q确实不等于h, 说明该线程需要移动head指针
                    // 32. eg: poll(): q=p=head, h=head => false
                    for (Node q = p; q != h;) {
                        Node n = q.next;  // update by 2 unless singleton   // 除非单例，否则更新 2
                        // 46. eg: poll() * 2: h=head=offer(E)的s, n=null, CAS设置head为q=p=head.next
                        if (head == h && casHead(h, n == null ? q : n)) {
                            // 46, eg: poll() * 2: CAS设置head为q=p=head.next成功, 则自链接旧的head, 退出循环即可
                            h.forgetNext();
                            break;
                        }                 // advance and retry  // 前进并重试
                        // 46. eg: poll() * 2: CAS设置head为q=p=head.next失败, 说明head指着已经被更新了,
                        //     则更新最新的head指针, 判断head.next是否已经匹配, 如果匹配了, 则继续向下寻找, 如果没有匹配则说明可以匹配, 退出循环即可
                        if ((h = head)   == null ||
                            (q = h.next) == null || !q.isMatched())
                            break;        // unless slack < 2
                    }

                    // 47, eg: poll() * 2: 退出循环说明head指针已经移动完成, 这时唤醒p的线程, 由于p为offer(E) * 2的s, 是异步非阻塞的添加, 没有线程 => 什么也不做
                    // 33. eg: poll(): 唤醒p的线程, 由于p为offer(E)的s, 是异步非阻塞的添加, 没有线程 => 什么也不做
                    LockSupport.unpark(p.waiter);

                    // 48, eg: poll() * 2: 返回配对的那个item
                    // 34. eg: poll(): 返回配对的那个item => 移动指针交由下一个线程来处理
                    return LinkedTransferQueue.<E>cast(item);
                }
            }

            // 39. eg: poll() * 2: item不匹配isData, 说明被上一个线程置null了, 这时需要p右移一个结点, 继续自旋
            // 30. eg: poll(): CAS设置head的item为null, 如果设置失败, 说明被别的线程匹配了, 这时需要p右移一个结点, 继续自旋
            Node n = p.next;
            p = (p != n) ? n : (h = head); // Use head if p offlist
        }

        // 14. eg: offer(E) * 2: SYNC != NOW
        // 2. eg: offer(E): SYNC != NOW
        if (how != NOW) {                 // No matches available
            // 15. eg: offer(E) * 2: s为null => s[e, true]
            // 3. eg: offer(E): s为null => s[e, true]
            if (s == null)
                s = new Node(e, haveData);

            // 16. eg: offer(E) * 2: CAS设置s为head.next, 成功则返回head => head != null, how == ASYNC
            // 4. eg: offer(E): CAS设置s为head指针, 成功则返回s => pred不为null, how == ASYNC
            Node pred = tryAppend(s, haveData);
            if (pred == null)
                continue retry;           // lost race vs opposite mode
            if (how != ASYNC)
                return awaitMatch(s, pred, e, (how == TIMED), nanos);
        }

        // 25. eg: offer(E) * 2: 返回添加的元素
        // 8. eg: offer(E): 返回添加的元素
        return e; // not waiting
    }
}

// 尝试将节点 s 附加为尾部。
private Node tryAppend(Node s, boolean haveData) {
    // 17. eg: offer(E) * 2: p=tail == null
    // 5. eg: offer(E): p=tail == null
    for (Node t = tail, p = t;;) {        // move p to last node and append // 将 p 移动到最后一个节点并追加
        Node n, u;                        // temps for reads of next & tail // 读取 next 和 tail 的温度

        // 18. eg: offer(E) * 2: p为null, 但head不为null => p=head
        // 6. eg: offer(E): p=heda == null
        if (p == null && (p = head) == null) {
            // 7. eg: offer(E): CAS设置s为head指针, 成功则返回s
            if (casHead(null, s))
                return s;                 // initialize
        }

        // 19. eg: offer(E) * 2: head的isData = havaData = true, 返回false, 说明可以处理, 不用返回false
        else if (p.cannotPrecede(haveData))
            return null;                  // lost race vs opposite mode // 输了比赛 vs 对立模式

        // 20. eg: offer(E) * 2: head.next == null, n=null
        else if ((n = p.next) != null)    // not last; keep traversing  // 不持久； 继续穿越
            p =
            p != t && t != (u = tail) ?
            (t = u) :
        (p != n)?
            n :
        null;

        // 21. eg: offer(E) * 2: p=head, CAS设置head的next为s
        else if (!p.casNext(null, s))
            p = p.next;                   // re-read on CAS failure // 重读CAS失败

        else {
            // 22. eg: offer(E) * 2: p.next设置成功, 则判断p=head, t=null => p确定不等于t
            if (p != t) {                 // update if slack now >= 2   // 如果现在松弛 >= 2 则更新
                // 23. eg: offer(E) * 2: tail=t=null, CAS设置t为s => t=tail != null => 退出循环
                while ((tail != t || !casTail(t, s)) &&
                       (t = tail)   != null &&
                       (s = t.next) != null && // advance and retry
                       (s = s.next) != null && s != t);
            }

            // 24. eg: offer(E) * 2: 返回p=head
            return p;
        }
    }
}
```

##### 添加元素方法

所有BlockingQueue接口的添加方法都是**异步非阻塞**的实现。一个新增的元素会与一个删除的空元素配对。

- **add（E）**：Queue接口方法，添加一个元素，由于队列是无界的，所以不会触发队列已满异常。
- **offer（E）**：Queue接口方法，添加一个元素，由于队列是无界的，所以不会返回null。
- **put（E）**：BlockingQueue接口方法，添加一个元素，由于队列是无界的，所以不会阻塞。
- **offer（E，long，TimeUnit）**：BlockingQueue接口方法，添加一个元素，由于队列是无界的，所以不会阻塞或者返回null。

```java
// Queue接口方法，添加一个元素，由于队列是无界的，所以不会触发队列已满异常。
public boolean add(E e) {
    xfer(e, true, ASYNC, 0);
    return true;
}

// Queue接口方法，添加一个元素，由于队列是无界的，所以不会返回null。
public boolean offer(E e) {
    xfer(e, true, ASYNC, 0);
    return true;
}

// BlockingQueue接口方法，添加一个元素，由于队列是无界的，所以不会阻塞。
public void put(E e) {
    xfer(e, true, ASYNC, 0);
}

// BlockingQueue接口方法，添加一个元素，由于队列是无界的，所以不会阻塞或者返回null。
public boolean offer(E e, long timeout, TimeUnit unit) {
    xfer(e, true, ASYNC, 0);
    return true;
}
```

##### 移交元素方法

tryTransfer（E）是**快速失败**的，transfer（E）是**同步阻塞**的，tryTransfer（E，long，TimeUnit）是**延迟消息**的。如果已经存在等待接收的消费者，比如{@link #take} 或 {@link #poll(long,TimeUnit) }，则会立即传输元素，否则各走的特殊实现。

- **tryTransfer（E）** ：TransferQueue接口方法，快速失败，如果存在等待消息的消费者，则立即传输该元素，否则返回false元素不入队。
- **transfer（E）**：TransferQueue接口方法，同步阻塞，如果存在等待消息的消费者，则立即传输该元素，否则等待直到被消费者接收。
- tryTransfer（E，long，TimeUnit）：TransferQueue接口方法，延迟消息，如果存在等待消息的消费者，则立即传输该元素，否则在尾部插入元素，并在限定时间内等待过后元素才可以被传输，并返回false。

```java
// TransferQueue接口方法，快速失败，如果存在等待消息的消费者，则立即传输该元素，否则返回false元素不入队
public boolean tryTransfer(E e) {
    return xfer(e, true, NOW, 0) == null;
}

// TransferQueue接口方法，同步阻塞，如果存在等待消息的消费者，则立即传输该元素，否则等待直到被消费者接收
void transfer(E e) throws InterruptedException;
public void transfer(E e) throws InterruptedException {
    if (xfer(e, true, SYNC, 0) != null) {
        Thread.interrupted(); // failure possible only due to interrupt
        throw new InterruptedException();
    }
}

// TransferQueue接口方法，延迟消息，如果存在等待消息的消费者，则立即传输该元素，否则在尾部插入元素，并在限定时间内等待过后元素才可以被传输，并返回false
public boolean tryTransfer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {
    if (xfer(e, true, TIMED, unit.toNanos(timeout)) == null)
        return true;
    if (!Thread.interrupted())
        return false;
    throw new InterruptedException();
}
```

##### 删除元素方法

remove（）、poll（）是**快速失败**的，take（）是**同步阻塞**的，poll（long，TimeUnit）是**延迟消费**的。一个删除的空元素会与新增的元素配对。

- **remove（）**：Queue接口方法，删除头元素，快速失败，如果没有同时添加元素，则会抛出异常。
- **poll（）**：Queue接口方法，删除头元素，快速失败，如果没有同时添加元素，则会返回false。
- **take（）**：BlockingQueue接口方法，删除头元素，同步阻塞，会一直阻塞到中断或者添加元素的执行。
- **poll（long，TimeUnit）**：BlockingQueue接口方法，删除头元素，延迟消费，会在限定时间内阻塞到中断或者添加元素的执行，否则返回false。

```java
// Queue接口方法，删除头元素，交由AbstractQueue实现，快速失败，如果没有同时添加元素，则会抛出异常。
public E remove() {
    E x = poll();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，删除头元素，快速失败，如果没有同时添加元素，则会抛出异常。
public E poll() {
    return xfer(null, false, NOW, 0);
}

// BlockingQueue接口方法，删除头元素，同步阻塞，会一直阻塞到中断或者添加元素的执行。
public E take() throws InterruptedException {
    E e = xfer(null, false, SYNC, 0);
    if (e != null)
        return e;
    Thread.interrupted();
    throw new InterruptedException();
}

// BlockingQueue接口方法，删除头元素，延迟消费，会在限定时间内阻塞到中断或者添加元素的执行，否则返回false。
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    E e = xfer(null, false, TIMED, unit.toNanos(timeout));
    if (e != null || !Thread.interrupted())
        return e;
    throw new InterruptedException();
}
```

##### 获取元素方法

- **element（）**：Queue接口方法，交由抽象类实现AbstractQueue#element（），返回第一个isData=true结点的元素，如果没有则抛出异常。
- **peek（）**：Queue接口方法，返回第一个isData=true结点的元素，如果没有则返回false。

```java
// Queue接口方法，交由抽象类实现AbstractQueue#element（），，返回第一个isData=true结点的元素，如果没有则抛出异常。
public E element() {
    E x = peek();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，返回第一个isData=true结点的元素，如果没有则返回false。
public E peek() {
    return firstDataItem();
}
// 找出第一个isData=true结点的元素，如果没有则返回false。
private E firstDataItem() {
    for (Node p = head; p != null; p = succ(p)) {
        Object item = p.item;
        if (p.isData) {
            if (item != null && item != p)
                return LinkedTransferQueue.<E>cast(item);
        }
        else if (item == null)
            return null;
    }
    return null;
}
// 获取下一个结点，如果碰到自链接的，则返回头指针，重头再来。
final Node succ(Node p) {
    Node next = p.next;
    return (p == next) ? head : next;
}
```

#### ConcurrentLinkedQueue

![1622036465996](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1622036465996.png)

##### 特点

- ConcurrentLinkedQueue，是**基于链表实现的无界的无锁的线程安全的队列**，支持对元素的FIFO（先进先出）进行排序。队列的开头是该元素在队列中停留时间最长的元素。 队列的尾部是最短时间出现在队列中的元素。新元素插入到队列的尾部，并且队列检索操作在队列的开头获取元素。当**多线程共享对一个公共集合的访问权限时**，{@code ConcurrentLinkedQueue}是一个适当的选择。**不允许使用{@code null}元素**。
- 采用一种**有效的自适应的阻塞并发队列算法**进行实现：
  - 全程没有使用悲观锁使得线程进入阻塞态，而是设计了**CAS + 自旋**来实现高并发无阻塞方式添加/删除元素，这样减少了线程切换的时间，提高吞吐量。且更新next指针使用了**UNSAFE.putOrderedObject（...）**实现延迟更新，进一步提高吞吐量。
  - 添加/删除元素方法中，还设计了**跳两个结点才更新tail/head指针**，来优化多线程并发自旋等待tail/head指针，浪费CPU资源严重的问题。
  - 还设计了**删除元素不是真正地删除而是把next指针指向自身（自链接）** + first（） 获取第一个item不为null的结点指针 + succ（）获取next活动的结点，来实现从头遍历。
- **迭代器是弱一致性的**，即仅反映返回的元素在创建迭代器时或创建迭代器后的某个时刻的队列状态。不会抛出{@link java.util.ConcurrentModificationException}，并且可能与其他操作并发进行。 自创建迭代器以来，队列中包含的元素将仅返回一次。
- 与大多数集合不同，**{@code size}方法不是恒定时间操作O（1）**， 由于这些队列的异步性质，**确定当前元素数需要对元素进行遍历**，因此，如果在遍历期间修改了此集合，则可能会出现不同的结果。
- 同时，也不能保证批量操作{@code addAll}，{@code removeAll}，{@code keepAll}，{@code containsAll}与{@code equals}和{@code toArray}的原子执行。例如，**与{@code addAll}操作同时运行的迭代器可能仅能查看一部分添加的元素**。
- 队列没有替换元素的方法，且获取元素方法都是检索队头的元素。

##### 构造方法

- **无参的构造函数**：构造空的头（尾）结点。
- **指定复制集合的构造函数**：遍历集合、UNSAFE.putOrderedObject设置链表结点的后继结点（线程可见有延迟，下次设置生效）。

```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E> implements Queue<E>, java.io.Serializable {
    
    private static class Node<E> {
        volatile E item;
        volatile Node<E> next;
        
        Node(E item) {
            UNSAFE.putObject(this, itemOffset, item);
        }
        
        private static final sun.misc.Unsafe UNSAFE;
        private static final long itemOffset;
        private static final long nextOffset;
        
        static {
            try {
                UNSAFE = sun.misc.Unsafe.getUnsafe();
                Class<?> k = Node.class;
                itemOffset = UNSAFE.objectFieldOffset(k.getDeclaredField("item"));
                nextOffset = UNSAFE.objectFieldOffset(k.getDeclaredField("next"));
            } catch (Exception e) {
                throw new Error(e);
            }
        }

        // CAS设置Next结点
        void lazySetNext(Node<E> val) {
            // putOrderedObject是有延迟的putIntVolatile方法，并且不保证值的改变被其他线程立即看到，只有字段在被volatile修饰并且期望被修改（下次修改）的时候使用才会生效。
            UNSAFE.putOrderedObject(this, nextOffset, val);
        }
    }
    
    private transient volatile Node<E> head;// next永远不会指向自身
    private transient volatile Node<E> tail;// next可能会指向自身
    
    private static final sun.misc.Unsafe UNSAFE;
    private static final long headOffset;
    private static final long tailOffset;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> k = ConcurrentLinkedQueue.class;
            headOffset = UNSAFE.objectFieldOffset(k.getDeclaredField("head"));
            tailOffset = UNSAFE.objectFieldOffset(k.getDeclaredField("tail"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
    
    // 空参的构造函数
    public ConcurrentLinkedQueue() {
        head = tail = new Node<E>(null);
    }
    
    // 指定复制集合的构造函数
    public ConcurrentLinkedQueue(Collection<? extends E> c) {
        Node<E> h = null, t = null;
        for (E e : c) {
            checkNotNull(e);
            Node<E> newNode = new Node<E>(e);
            if (h == null)
                h = t = newNode;
            else {
                t.lazySetNext(newNode);// 设置t的Next结点(线程可见有延迟，下次设置生效)
                t = newNode;
            }
        }
        if (h == null)
            h = t = new Node<E>(null);
        head = h;
        tail = t;
    } 
}
```

##### 迭代方法

- **iterator（）**：
  - 以适当的顺序返回对该队列中的元素的迭代器。 元素将按照从头（头）到尾（头）的顺序返回。
  - **返回的迭代器是弱一致性的**。
  - 一旦hasNext（）声明了中存在某个元素，即使在调用hasNext（）时该元素正处于删除过程中，也必须在接下来的next（）调用中将该元素返回。

```java
public Iterator<E> iterator() {
    return new Itr();
}

```

##### 扩容方法

链表结构无需扩容，维护指针关系即可。

##### 添加元素方法

由于要实现高并发无阻塞地添加元素，所以采用**CAS（写死null值） + 自旋**方式来实现，但又考虑到了高并发添加时的**多线程为了等待tail指针做CAS，从而导致大量线程一直自旋**，消耗大量的CPU资源，所以又设计**跳两个结点才进行更新tail指针**，从而减少一般的线程自旋损耗，提升了吞吐量。

- **add（E）**：Queue接口方法，添加元素到队尾，由于队列是无界的，所以不会触发队列已满异常。
- **offer（E）**：Queue接口方法，添加元素到队尾，由于队列是无界的，所以永远不会返回false。

```java
// Queue接口方法，添加元素到队尾，由于队列是无界的，所以不会触发队列已满异常。
public boolean add(E e) {
    return offer(e);
}

// Queue接口方法，添加元素到队尾，由于队列是无界的，所以永远不会返回false。
public boolean offer(E e) {
    checkNotNull(e);
    final Node<E> newNode = new Node<E>(e);

    // 1. t、p为当前尾指针
    for (Node<E> t = tail, p = t;;) {
        // 2. 防止并发, 所以每轮需要重新获取next结点, q为当前尾指针p的next结点
        // 5. (重新自旋)最新一次的自旋, 获取最新p结点的next结点q
        Node<E> q = p.next;

        // 3. 高并发无阻塞队列的插入是写死CAS为null才能插入的, 所以要自旋判断下是否为null
        if (q == null) {
            // 4. 如果q是null的话, 说明是能够CAS的, 这时CAS当前结点为next结点
            if (p.casNext(null, newNode)) {
                
                // 5. CAS next结点成功, 则判断p!=t, 说明当前线程是一个需要修改tail指针的线程, 来完成上一个线程直接返回true的工作。
                if (p != t) // hop two nodes at a time  // 一次跳两个节点
                    casTail(t, newNode);  // Failure is OK. // 失败是可以的。

                // 5. CAS next结点成功, 则可以直接返回true了, 无需再修改tail指针, 是一种优化机制: 只有两跳才会修改tail指针, 把修改工作交给下一个线程来做, 提高吞吐量
                return true;
            }
            // Lost CAS race to another thread; re-read next
        }
        
        // 3. 如果为自身的话, 说明自身结点被删除了(由于不想影响其他线程, 所以删除不能真的把结点删除, 而是把next指针指向它自己)
        else if (p == q)
            
            // 4. 如果t==tail, 说明已经最后一个结点都被删除了, 这时需要p=tail=head,
            //      而t!=tail说明当前结点被删除了且别的线程插入了新的结点还更新尾指针, 这时需要把最新的尾指针赋值给p, 重新自旋
            p = (t != (t = tail)) ? t : head;
        
        // 3. 如果q不是null, 又不是自身的话, 则继续判断
        else
            // 在两跳之后检查尾部更新。
            // Check for tail updates after two hops.

            // 4. 如果p==t或者tail不是最新的尾结点, 则赋值q给p, 说明别的线程添加了next结点, 但还没移动tail指针, 这时需要跳到q结点, 自旋到下一次重新获取q
            // 5. 如果p!=t且tail为最新的尾结点, 则赋值t给p, 即让p成为最新的尾结点, 继续自旋插入
            p = (p != t && t != (t = tail)) ? t : q;
    }
}
```

##### 删除元素方法

由于要实现高并发无阻塞地添加元素，所以采用**CAS（item为null） + 自旋**方式来实现，但又考虑到了高并发添加时的**多线程为了等待head指针做CAS，从而导致大量线程一直自旋**，消耗大量的CPU资源，所以又设计**跳两个结点才进行更新head指针**，从而减少一般的线程自旋损耗，提升了吞吐量。

- **remove（）**：Queue接口方法，由AbstractQueue抽象类实现，删除队头元素，如果队列为空，则会抛出异常。
- **offer（）**：Queue接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null。
- **remove（Object）**：Collection接口方法，删第一个指定的元素，使用equals比较。

```java
// Queue接口方法，由AbstractQueue抽象类实现，删除队头元素，如果队列为空，则会抛出异常。
public E remove() {
    E x = poll();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null。
public E poll() {
    restartFromHead:
    for (;;) {
        // 1. h为头结点, p=h, q = null
        for (Node<E> h = head, p = h, q;;) {

            // 2. 防止并发, 所以每轮需要重新获取item值
            // 5. (重新自旋)最新的一次的自旋, 获取item值
            E item = p.item;

            // 3. 如果item不为null, 且能够CAS item为null, 说明p确实为头结点
            if (item != null && p.casItem(item, null)) {
                // 4. CAS item为null后, 如果p!=h, 说明上一个线程只做了CAS item操作, 还没做移动head指针的操作 所以当前线程需要做head指针的移动操作
                if (p != h) // hop two nodes at a time // 一次跳两个节点
                    
                    // 5. 如果p的next结点为null, 说明p为尾结点了, 则更新当前head指针为当前结点, 否则更新head为next结点
                    //    CAS旧的头结点h为最新的头结点p, 把原来的h结点的next结点设置为它本身, 实现结点的“删除" => 两个线程才移动一次head指针, 提高吞吐量
                    updateHead(h, ((q = p.next) != null) ? q : p);

                // 4. CAS item为null后, 可以直接返回了, 把head指针的更改交给下一个线程, 减少其他线程的自旋次数, 提高吞吐量
                return item;
            }
            
            // 3. 如果item为null或者CAS item为null失败, 且p的next结点为null, 说明p为尾结点且当前线程为移动head指针线程
            else if ((q = p.next) == null) {
                // 4. 这时CAS旧的头结点h为最新的头结点p, 把原来的h结点的next结点设置为它本身, 实现结点的“删除"
                updateHead(h, p);

                // 5. 返回null是因为, 有数据的结点已经在上个线程置空并返回了, 当前线程只需要为移动head指针即可, 所以返回null
                return null;
            }
            
            // 3. 如果item为null或者CAS item为null失败, 且p又不是尾结点, 且p的next结点等于p自身, 说明p结点已经被删除, 且head指针已经被移动过了
            else if (p == q)
                
                // 4. 由于poll()是CAS+自旋操作, 是要肯定成功的, 所以继续自旋, 去poll下一个元素, 但又要获取最新的head结点, 所以直接重新进入自旋, 而不是下一轮自旋
                continue restartFromHead;
            
            // 3. 否则p item为null 捉着 CAS失败、且又不是尾结点、且又还没被删除, 说明上一个线程做了CAS item为null的操作, 但是还没移动head指针
            else
                
                // 4. 所以把p跳到下一个结点q去, 由于poll()是CAS+自旋操作, 是要肯定成功的, 所以继续自旋, 以在适当的时候置null元素、移动head指针
                p = q;
        }
    }
}

// Collection接口方法，删第一个指定的元素，使用equals比较。
public boolean remove(Object o) {
    if (o == null) return false;
    Node<E> pred = null;

    // 找出第一个item不为null的结点 或者head=tail的结点 => 从头开始遍历
    for (Node<E> p = first(); p != null;
         // 返回p的next结点, 如果为p自身则返回头结点 => 每次遍历结束, 都需要把p移动到下一个结点, 开启下一轮遍历
         p = succ(p)) {

        // 当前item
        E item = p.item;
        if (item != null && // 当前item不为null
            o.equals(item) && // 且equals指定元素
            p.casItem(item, null)) {// 这时则CAS置item为null

            // CAS置空item成功后, 如果为第一个结点或者为尾结点, 则直接返回就好, 移动head指针交由下一个线程负责, 而如果不是第一个结点也不是尾结点, 说明需要移动head指针
            Node<E> next = succ(p);
            if (pred != null && next != null)
                // 移动head指针到p的next结点处
                pred.casNext(p, next);
            return true;
        }

        // pred为p的前驱结点, 为null代表p为第一个结点
        pred = p;
    }

    // 如果遍历完还没找到, 则返回false
    return false;
}
// 找出第一个item不为null的结点 或者head=tail的结点
Node<E> first() {
    restartFromHead:
    for (;;) {
        // 1. h、p为头结点
        for (Node<E> h = head, p = h, q;;) {
            // 2. 判断item是否为null
            boolean hasItem = (p.item != null);

            // 3. 如果item不为null, 或者p为尾结点
            if (hasItem || (q = p.next) == null) {
                // 4. 如果item不为null, 更新h相当于没做任何更新; 如果p为尾结点, 更新h相当于head=tail
                updateHead(h, p);

                // 5. 如果item不为null, 则返回头结点h=p, 即第一个有item的结点指针
                return hasItem ? p : null;
            }

            // 3. 如果p已经被删除了, 则重新更新head指针, 重新进入自旋
            else if (p == q)
                continue restartFromHead;

            // 3. 如果item为null且p也不是尾结点也没有被删除, 则移动p指针到next结点, 继续自旋, 找出第一个item不为null的结点
            else
                p = q;
        }
    }
}
// 返回p的next结点, 如果为p自身则返回头结点
final Node<E> succ(Node<E> p) {
    Node<E> next = p.next;
    return (p == next) ? head : next;
}
```

##### 获取元素方法

与first()方法实现类似，**只不过first()返回的是第一个item不为null的结点, 而peek()返回的是第一个不为null的item**。

虽然可以使 peek() 成为 first() 的包装器，但item读取不稳定，这将需要**额外的花费（根据item重新构造结点）**，并且**需要添加重试循环来处理与并发 poll() 竞争失败的可能性**（因为重新构造了结点要循环去判断到底这个结点的指针是什么）。所以first()并没有使用peek()作为包装器，而是独立又实现了一遍与peek()差不多的逻辑。

- **element（）**：Queue接口方法：由AbstractQueue抽象类实现，检索队头元素，如果队列为空，则会抛出异常。
- **peek（）**：Queue接口方法，检索队头元素，如果队列为空，不会抛出异常，而是返回null。

```java
// Queue接口方法：由AbstractQueue抽象类实现，检索队头元素，如果队列为空，则会抛出异常。
public E element() {
    E x = peek();
    if (x != null)
        return x;
    else
        throw new NoSuchElementException();
}

// Queue接口方法，检索队头元素，如果队列为空，不会抛出异常，而是返回null。
public E peek() {
    restartFromHead:
    for (;;) {
        // 1. h、p为头结点
        for (Node<E> h = head, p = h, q;;) {
            E item = p.item;

            // 2. 如果item不为null, 或者p为尾结点
            if (item != null || (q = p.next) == null) {
                // 4. 如果item不为null, 更新h相当于没做任何更新; 如果p为尾结点, 更新h相当于head=tail
                updateHead(h, p);

                // 5. 如果item不为null, 则返回头结点h=p, 即第一个有item的结点指针, 否则返回null
                return item;
            }

            // 3. 如果p已经被删除了, 则重新更新head指针, 重新进入自旋
            else if (p == q)
                continue restartFromHead;

            // 如果item为null且p也不是尾结点也没有被删除, 则移动p指针到next结点, 继续自旋, 找出第一个item不为null的结点
            else
                p = q;
        }
    }
}
```

#### ConcurrentLinkedDeque

![1622269704245](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1622269704245.png)

##### 特点

- ConcurrentLinkedDeque，**基于链接节点的无界并发的无阻塞的双端队列**，并发插入、删除和访问操作跨多个线程安全地执行。 当许多线程将共享对公共集合的访问时，{@code ConcurrentLinkedDeque} 是合适的选择。 与大多数其他并发集合实现一样，**此类不允许使用{@code null}元素**。
- 采用一种**有效的阻塞并发队列算法**进行实现：
  - 全程没有使用悲观锁使得线程进入阻塞态，而是设计了**CAS + 自旋**来实现高并发无阻塞方式添加/删除元素，这样减少了线程切换的时间，提高吞吐量。且更新next指针使用了**UNSAFE.putOrderedObject（...）**实现延迟更新，进一步提高吞吐量。
  - 添加元素方法中，还设计了**跳两个结点才更新tail/head指针**，来优化多线程并发自旋等待tail/head指针，浪费CPU资源严重的问题。
  - 删除方法中，设计了三步删除的方式，分别为**logical deletion、unlink和gc-unlink**，来优化多线程并发自旋等待tail/head指针，浪费CPU资源严重的问题。
  - 还设计了**删除元素不是真正地删除而是把next指针指向自身（自链接）** + first（） 获取第一个item不为null的结点指针 + succ（）获取next活动的结点，来实现从头遍历。
  - 如果节点是**活动节点**，或者是第一个或最后一个结点，则该节点被认为是“活动的”。 活动结点不能取消链接。当且仅当以下情况，结点p才是活动的：
    - p.item != null：p表示活动结点，即实际元素的结点。
    - || (p.prev == null && p.next != p)：p表示虚拟结点 NEXT_TERMINATOR，而不是最后一个活动结点，而是最后一个结点，用于判断迭代是否到了结束的时候。 队尾结点p出队时, 会把p.prev=p，p.next=NEXT_TERMINATOR。
    - || (p.next == null && p.prev != p)：p表示虚拟结点 PREV_TERMINATOR，而不是第一个活动结点，而是第一个结点，用于判断是否迭代开始。 队头结点p出队时, 会把p.next=p，p.prev=PREV_TERMINATOR。
- **迭代器是弱一致性的**，即仅反映返回的元素在创建迭代器时或创建迭代器后的某个时刻的队列状态。不会抛出{@link java.util.ConcurrentModificationException}，并且可能与其他操作并发进行。 自创建迭代器以来，队列中包含的元素将仅返回一次。
- 与大多数集合不同，**{@code size}方法不是恒定时间操作O（1）**， 由于这些队列的异步性质，**确定当前元素数需要对元素进行遍历**，因此，如果在遍历期间修改了此集合，则可能会出现不同的结果。
- 同时，也不能保证批量操作{@code addAll}，{@code removeAll}，{@code keepAll}，{@code containsAll}与{@code equals}和{@code toArray}的原子执行。例如，**与{@code addAll}操作同时运行的迭代器可能仅能查看一部分添加的元素**。
- 队列没有替换元素的方法，且获取元素方法都是检索队头的元素。

##### 构造方法

- **无参的构造函数**：构造空的头（尾）结点。
- **指定复制集合的构造函数**：遍历集合、UNSAFE.putOrderedObject设置链表结点的前驱和后继结点（线程可见有延迟，下次设置生效）。

```java
public class ConcurrentLinkedDeque<E> extends AbstractCollection<E> implements Deque<E>, java.io.Serializable {
    
    static final class Node<E> {
        volatile Node<E> prev;
        volatile E item;
        volatile Node<E> next;
        
        private static final sun.misc.Unsafe UNSAFE;
        private static final long prevOffset;
        private static final long itemOffset;
        private static final long nextOffset;

        static {
            try {
                UNSAFE = sun.misc.Unsafe.getUnsafe();
                Class<?> k = Node.class;
                prevOffset = UNSAFE.objectFieldOffset(k.getDeclaredField("prev"));
                itemOffset = UNSAFE.objectFieldOffset(k.getDeclaredField("item"));
                nextOffset = UNSAFE.objectFieldOffset(k.getDeclaredField("next"));
            } catch (Exception e) {
                throw new Error(e);
            }
        }
        
        // CAS设置next后继结点
        void lazySetNext(Node<E> val) {
            // putOrderedObject是有延迟的putIntVolatile方法，并且不保证值的改变被其他线程立即看到，只有字段在被volatile修饰并且期望被修改（下次修改）的时候使用才会生效。
            UNSAFE.putOrderedObject(this, nextOffset, val);
        }
        
        // CAS设置prev前驱结点
        void lazySetPrev(Node<E> val) {
            // putOrderedObject是有延迟的putIntVolatile方法，并且不保证值的改变被其他线程立即看到，只有字段在被volatile修饰并且期望被修改（下次修改）的时候使用才会生效。
            UNSAFE.putOrderedObject(this, prevOffset, val);
        }
    }
    
    private transient volatile Node<E> head;// next永远不会指向自身
    private transient volatile Node<E> tail;// next可能会指向自身
    
    // PREV_TERMINATOR, prev终止结点: 队头结点p出队时, 会把p.next=p,p.prev=PREV_TERMINATOR
    // NEXT_TERMINATOR, next终止结点: 队尾结点p出队时, 会把p.prev=p,p.next=NEXT_TERMINATOR
    private static final Node<Object> PREV_TERMINATOR, NEXT_TERMINATOR;

    private static final sun.misc.Unsafe UNSAFE;
    private static final long headOffset;
    private static final long tailOffset;
    static {
        PREV_TERMINATOR = new Node<Object>();
        PREV_TERMINATOR.next = PREV_TERMINATOR;// Prev终止结点next为自身，自身为null
        NEXT_TERMINATOR = new Node<Object>();
        NEXT_TERMINATOR.prev = NEXT_TERMINATOR;// Next终止结点prev为自身，自身为null
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> k = ConcurrentLinkedDeque.class;
            headOffset = UNSAFE.objectFieldOffset(k.getDeclaredField("head"));
            tailOffset = UNSAFE.objectFieldOffset(k.getDeclaredField("tail"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }

    // 无参的构造函数
    public ConcurrentLinkedDeque() {
        head = tail = new Node<E>(null);
    }
    
    // 指定复制集合的构造函数
    public ConcurrentLinkedDeque(Collection<? extends E> c) {
        // Copy c into a private chain of Nodes
        Node<E> h = null, t = null;
        for (E e : c) {
            checkNotNull(e);
            Node<E> newNode = new Node<E>(e);
            if (h == null)
                h = t = newNode;
            else {
                t.lazySetNext(newNode);// 设置t的Next结点(线程可见有延迟，下次设置生效)
                newNode.lazySetPrev(t);// 设置node的Prev结点(线程可见有延迟，下次设置生效)
                t = newNode;// 移动t指针
            }
        }
        initHeadTail(h, t);
    }
}

```

##### 迭代方法

- **iterator（）**：
  - 以适当的顺序返回对该队列中的元素的迭代器。 元素将按照从头（头）到尾（头）的顺序返回。
  - **返回的迭代器是弱一致性的**。

```java
public Iterator<E> iterator() {
    return new Itr();
}
```

##### 扩容方法

链表结构无需扩容，维护指针关系即可。

##### 高并发无阻塞添加原理

由于要实现高并发无阻塞地添加元素，所以采用**CAS（写死null值） + 自旋**方式来实现，但又考虑到了高并发添加时的**多线程为了等待head/tail指针做CAS，从而导致大量线程一直自旋**，消耗大量的CPU资源，所以又设计**跳两个结点才进行更新head/tail指针**，从而减少一般的线程自旋损耗，提升了吞吐量。

```java
// 高并发无阻塞地添加头结点的主要逻辑
private void linkFirst(E e) {
    checkNotNull(e);
    final Node<E> newNode = new Node<E>(e);

    restartFromHead:
    for (;;)
        // 1. h = p = head, 为head指针
        for (Node<E> h = head, p = h, q;;) {
            // 2. 如果p的前驱不为null, 且p的前驱的前驱也不为null, 相当于p、q都左移一个结点,
            //    这里巧妙的地方在于, 如果p前驱为null了就不会再找前驱的前驱了, 而如果前驱不为null还会去更新p为p的前驱, 从而保证了p一定视为第一个结点, 而q永远只代表p的前驱
            if ((q = p.prev) != null &&
                (q = (p = q).prev) != null)
                // Check for head updates every other hop.  // 每隔一跳检查头部更新。
                // If p == q, we are sure to follow head instead.   // 如果 p == q，我们肯定会跟随 head。

                // 3. 如果此时还不为空, 说明前面还有结点, 则判断h是否为最新的head指针, 如果不是, 则说明head指针被更新过了, 设置p为最新的head指针继续自旋;
                //    如果是, 则说明head指针还没更新, 说明了别的线程插入了两结点, 但最后的线程还没更新head指针, 此时设置p为q即p的前驱, 继续自旋, 等待插入或者继续往前找
                p = (h != (h = head)) ? h : q;

            // 2. 如果p的前驱为null, 且p的后继为它本身, 说明p结点已经被删除了, 需要重新获取head指针, 此时重新进入自旋
            else if (p.next == p) // PREV_TERMINATOR
                continue restartFromHead;

            // 2. 如果p的前驱为null, 且p还没有被删除, 说明p是第一个结点
            else {
                // 3. 此时把当前结点CAS插入到p前面
                // p is first node
                newNode.lazySetNext(p); // CAS piggyback

                // 4. 然后CAS设置p的前驱(写死为null)为当前结点
                if (p.casPrev(null, newNode)) {
                    // 成功的 CAS 是 e 成为这个双端队列的一个元素，以及 newNode 成为“活”的线性化点。
                    // Successful CAS is the linearization point
                    // for e to become an element of this deque,
                    // and for newNode to become "live".

                    // 5. 如果CAS设置前驱成功, 如果p!=h, 说明当前线程需要移动head指针, 此时移动head指针为当前结点
                    if (p != h) // hop two nodes at a time  // 一次跳两个节点
                        casHead(h, newNode);  // Failure is OK. // 失败是可以的。

                    // 5. 如果CAS设置前驱成功, 则直接返回, 移动head指针交由下一个线程完成
                    return;
                }

                // 失去了到另一个线程的 CAST 竞赛； 重读上一篇
                // Lost CAS race to another thread; re-read prev
            }
        }
}

// 高并发无阻塞地添加尾结点的主要逻辑
private void linkLast(E e) {
    checkNotNull(e);
    final Node<E> newNode = new Node<E>(e);

    restartFromTail:
    for (;;)
        // 1. h = p = head, 为tail指针
        for (Node<E> t = tail, p = t, q;;) {
            // 2. 如果p的后继不为null, 且p的后继的后继也不为null, 相当于p和q都右移了一个结点
            //    这里巧妙的地方在于, 如果p后继为null了就不会再找后继的后继了, 而如果后继不为null还会去更新p为p的后继, 从而保证了p一定视为最后结点, 而q永远只代表p的后继
            if ((q = p.next) != null &&
                (q = (p = q).next) != null)
                // Check for tail updates every other hop.  // 每隔一跳检查尾部更新。
                // If p == q, we are sure to follow tail instead.   // 如果 p == q，我们肯定会跟随tail。

                // 3. 如果此时还不为空, 说明后面还有结点, 再判断t是否为最新的tail指针, 如果不是, 则更新t为最新的tail指针, 继续自旋
                //    如果是, 则说明tail指针还没更新, 说明有两个线程插入了结点, 但一个线程还没更新tail指针, 此时设置p为p的后继, 等待插入或者继续往后找
                p = (t != (t = tail)) ? t : q;

            // 2. 如果p的后继为null, 且p的前驱为它本身, 说明p被删除了, 此时需要获取最新的tail指针, 所以重新进入自旋
            else if (p.prev == p) // NEXT_TERMINATOR
                continue restartFromTail;

            // 2. 如果p的后继为null, 且p还没有被删除, 说明p是最后一个结点
            else {
                // 3. 此时把当前结点CAS插入到p后面
                // p is last node
                newNode.lazySetPrev(p); // CAS piggyback

                // 4. 然后CAS设置p的后继(写死为null)为当前结点
                if (p.casNext(null, newNode)) {
                    // Successful CAS is the linearization point
                    // for e to become an element of this deque,
                    // and for newNode to become "live".

                    // 5. 如果CAS设置后继成功, 如果p!=h, 说明当前线程需要移动tail指针, 此时移动tail指针为当前结点
                    if (p != t) // hop two nodes at a time
                        casTail(t, newNode);  // Failure is OK.

                    // 5. 如果CAS设置后继成功, 则直接返回, 移动tail指针交由下一个线程完成
                    return;
                }
                // Lost CAS race to another thread; re-read next
            }
        }
}
```

##### 高并发无阻塞删除原理

由于要实现高并发无阻塞地删除元素，所以采用三步删除的方式实现：

1. **logical deletion**：**逻辑删除**，通过在unlink（Object）外层实现元素CAS置空，以直接返回item值。
2. **unlink**：**断开活动结点到逻辑删除结点的链接**，其中本次要unlink的first结点，虽然被标记为了逻辑删除结点，但本次主要用来圈起来其他逻辑删除的结点，并不会被unlink掉。此后，由于逻辑删除结点没有别人引用它们，不久后就会被GC找到了的。
3. **gc-unlink**：断开逻辑删除结点与活动结点的链接，本次是一次优化，主要让逻辑删除结点（被圈起来的那些结点）的prev/next指向自身或者为PREV_TERMINATOR（存在first时）或者为NEXT_TERMINATOR（存在last时），以便让GC更快地发现，保持GC的健壮性。**其中first代表x/o前逻辑删除的结点，last代表x/o后逻辑删除的结点。**

而对于删除情况又分为了三种：

1. unlink的x是第一个结点（prev==null），此时需要unlinkFirst（x，next），当有大量的线程并发pollFirst()，则会在unlinkFirst中走unlink + gc-unlink的逻辑。而如果非并发pollFirst()，则先调用的线程会生成first结点，剩下的线程都走unlink内部节点的逻辑。
2. unlink的x是第一个结点（next==null），此时需要unlinkLast（x，prev），当有大量的线程并发pollLast()，则会在unlinkLast中走unlink + gc-unlink的逻辑。而如果非并发pollLast()，则先调用的线程会生成last结点，剩下的线程都走unlink内部节点的逻辑。
3. unlink内部结点的删除逻辑：主要是用于找出first和last结点，没有话有item的活动结点也可以。通过前后first和last建立链接走unlink + gc-unlink的逻辑。

```java
// unlink指定活动结点的链接
void unlink(Node<E> x) {
    // assert x != null;
    // assert x.item == null;
    // assert x != PREV_TERMINATOR;
    // assert x != NEXT_TERMINATOR;

    final Node<E> prev = x.prev;
    final Node<E> next = x.next;

    // 1. 如果当前结点的前驱为null, 说明x为第一个活动结点, 则unlink x, 且x是已经被标记为逻辑删除的结点(logical deletion),
    //    只有在第一次并发同端删除才会进入, 且first=head并不会被gc-unlink, 但会被unlink
    if (prev == null) {
        unlinkFirst(x, next);
    }

    // 2. 如果当期结点的后继为null, 说明x为最后一个活动结点, 则unlink X, 且X是已经被标记为逻辑删除的结点(logical deletion),
    //    只有在第一次并发同端删除才会进入, 且last=tail并不会被gc-unlink, 但会被unlink
    else if (next == null) {
        unlinkLast(x, prev);
    }

    // 3. 如果当前结点的前驱和后继都不为null, 说明x既不是第一个也不是最后一个活动结点, 非第一次同端删除都属于内部结点的删除, 用于删除中间标记了逻辑删除但为gc-unlink的o结点们
    else {
      
        // 4. activePred为x前第一个活动的前驱, activeSucc为x后第一个活动的后继, isFist代表x前是否存在待删除的first结点, isLast代表x后是否存在待删除的last结点
        //    x是已经被标记为逻辑删除的结点(logical deletion)
        Node<E> activePred, activeSucc;
        boolean isFirst, isLast;
        int hops = 1;

        // Find active predecessor

        // 查找x前第一个活动的前驱
        for (Node<E> p = prev; ; ++hops) {
            // 5. 如果x的前驱p的item不为null, 则说明p为x前第一个活动的前驱, 此时x前不存在待删除的first结点, 所以isFirst为ifasle
            if (p.item != null) {
                activePred = p;
                isFirst = false;
                break;
            }

            // 5. 如果x的前驱p的item为null, 说明p是一个活动的结点, 但也是一个待删除的结点, 则再获取p的前驱q
            Node<E> q = p.prev;

            // 6. 如果q为null, 说明p为第一个活动的结点
            if (q == null) {
                // 7. 但p.next=p, 说明p为PREV_TERMINATOR, 因为找不到一个x的活动的前驱结点, 说明还没经过第一个同端删除, 这时直接返回即可, 把unlink x看作为第一个的同端删除
                if (p.next == p)
                    return;

                // 7. 如果p不是PREV_TERMINATOR, 则说明找到了第一个活动结点也是待删除的结点, 此时设置activePred为p, 所以isFirst为true, 退出循环
                activePred = p;
                isFirst = true;
                break;
            }

            // 6. 如果q不为null, 说明q前还有结点, 但p==q, 即p=p.prev, 说明p为NEXT_TERMINATOR, 其实不可能发生
            else if (p == q)
                return;

            // 6. 如果q不为null, 说明q前还有结点, 也不为NEXT_TERMINATOR, 此时p往前移动一个结点, hops记录经过的结点数, 此后就是2了
            else
                p = q;
        }

        // Find active successor

        // 查找x前第一个活动的后继
        for (Node<E> p = next; ; ++hops) {
            // 8. 如果x的后继p的item不为null, 则说明p为x前第一个活动的后继, 此时x前不存在待删除的last结点, 所以isLast为ifasle
            if (p.item != null) {
                activeSucc = p;
                isLast = false;
                break;
            }

            // 8. 如果x的后继p的item为null, 说明p并不是一个活动的结点, 则再获取p的后继q
            Node<E> q = p.next;

            // 10. 如果q为null, 说明p为最后一个活动结点
            if (q == null) {
                // 11. 但p.prev=p, 说明p为NEXT_TERMINATOR, 因为找不到一个x的活动的后继结点, 说明还没经过第一个同端删除, 这时直接返回即可, 把unlink x看作为第一个的同端删除
                if (p.prev == p)
                    return;

                // 11. 如果p为活动结点, 则说明找到了最后一个活动结点也是待删除的结点, 此时设置activeSucc为p, 所以isLast为true, 退出循环
                activeSucc = p;
                isLast = true;
                break;
            }

            // 10. 如果q不为null, 说明q后还有结点, 但p==q, 即p=p.next, 说明p为PREV_TERMINATOR, 其实不可能发生
            else if (p == q)
                return;

            // 10. 如果q不为null, 说明q后还有结点, 也不为PREV_TERMINATOR, 此时p往后移动一个结点, hops记录经过的结点数, 此后就是3了
            else
                p = q;
        }

        // 12. 如果中间跳过的结点数小于2, 则退出, 如果x前不存在待删除结点, 则也会退出
        if (hops < HOPS && (isFirst | isLast))
            return;
        
        // Squeeze out deleted nodes between activePred and
        // activeSucc, including x.
        
        // 13. activePred往后跳过已经逻辑删除的后继结点, 使得activePred的next一定是个活动结点
        skipDeletedSuccessors(activePred);

        // 14. activeSucc往前跳过已经逻辑删除的前驱结点, 使得activeSucc的prev一定是个活动结点
        skipDeletedPredecessors(activeSucc);

        // Try to gc-unlink, if possible

        // 15. 如果x前存在待删除结点, 且activePred和activeSucc保持链接,
        //     如果存在待删除的first结点, 则判断activePred是否为第一个结点, 否则判断activePred是否为存活结点
        //     如果存在待删除的last结点, 则判断activeSucc是否为最后一个结点, 否则判断activeSucc是否为存活结点
        //     说明x确实在activePred与activeSucc的中间, 且activePred和activeSucc都是合法的
        if ((isFirst | isLast) &&

            // Recheck expected state of predecessor and successor
            (activePred.next == activeSucc) &&
            (activeSucc.prev == activePred) &&
            (isFirst ? activePred.prev == null : activePred.item != null) &&
            (isLast  ? activeSucc.next == null : activeSucc.item != null)) {

            // 17. 通过当前head指针, 去向前找到第一个活动结点, 并设置为最新的head指针, 这里保证了head一定是activePred或者activePred前面, 从而确保x在中间
            updateHead(); // Ensure x is not reachable from head

            // 18. 通过当前tail指针, 去向后找到最后一个活动结点, 并设置为最新的tail指针, 这里保证了head一定是activeSucc或者activeSucc后面, 从而确保x在中间
            updateTail(); // Ensure x is not reachable from tail

            // Finally, actually gc-unlink

            // 19. gc-unlink: 如果存在待删除的first结点, 则x.prev链接PREV_TERMINATOR, 否则自链接自身结点x
            x.lazySetPrev(isFirst ? prevTerminator() : x);

            // 20. gc-unlink: 如果存在待删除的last结点, 则x.next链接NEXT_TERMINATOR, 否则自链接自身结点x
            x.lazySetNext(isLast  ? nextTerminator() : x);

            // first和last结点会不清理, 或者留到下次清理
        }
    }
}

// unlink第一个非null结点, 且该结点是已经被标记为逻辑删除的结点(logical deletion), 只有在第一次并发同端删除才会进入, 且first=head并不会被gc-unlink, 但会被unlink
private void unlinkFirst(Node<E> first, Node<E> next) {
    // assert first != null;
    // assert next != null;
    // assert first.item == null;

    // 1. first为当前要解除链接的结点, p为解除结点的next结点
    for (Node<E> o = null, p = next, q;;) {

        // 2. 如果p的item不为null, 说明p为活动结点, 或者p的next结点为null, 说明p为尾结点了
        if (p.item != null || (q = p.next) == null) {

            // 3. 此时p为活动结点或者为尾结点, 如果o不为null, 说明p经过了移动, 且p的前驱不是它自己, 说明新的p还没有被gc-unlink, 即p是first后的第一个活动结点,
            //    此时设置待删除的结点first的next为p, 保证只能从first结点才能访问活动结点,
            //    => 如果下面还将p的prev设置为first, 那么无论是从head还是tail, 都无法访问first与p之间的待删除结点了, 这就是unlink的第一步
            if (o != null && p.prev != p && first.casNext(next, p)) {

                // 4. 如果first.next设置成功了, 则p往前跳过已经逻辑删除的前驱结点, 使得p的prev为first结点， 这步属于unlink的第二步
                //    => unlink后, 中间的逻辑删除结点o就被first和p圈起来了, first的next为p, p的prev为frist, 尽管o还可以访问first和p,
                //       但由于o没有了引用, GC最后都会发现它们
                skipDeletedPredecessors(p);

                // 5. 如果first的前驱为null, 说明first为第一个结点, 且p的next为null或者p的item不为null, 说明p是个活动结点
                //    且p的前驱为first, 说明p与first的链接还是正常的
                if (first.prev == null &&
                    (p.next == null || p.item != null) &&
                    p.prev == first) {

                    // 6. 通过当前head指针, 去向前找到第一个活动结点, 并设置为最新的head指针, 这里保证了head一定是first或者first前面, 从而确保o在中间
                    updateHead(); // Ensure o is not reachable from head    // 确保 o 无法从头部到达, o为上一个线程留下的

                    // 7. 通过当前tail指针, 去向后找到最后一个活动结点, 并设置为最新的tail指针, 这里保证了tail一定是p或者p后面, 从而确保o在中间
                    updateTail(); // Ensure o is not reachable from tail    // 确保 o 无法从尾部到达, o为上一个线程留下的

                    // Finally, actually gc-unlink
                    // 8. gc-unlink: 自链接中间的o结点, 设置o的前驱为PREV_TERMINATOR, 即将o进行最后一步删除
                    o.lazySetNext(o);
                    o.lazySetPrev(prevTerminator());
                }
            }

            // 3. 如果p为活动结点或者为尾结点, 如果o为null, 说明first到p之间不存在逻辑删除的结点, 直接返回即可,
            //    由于first标记为了逻辑删除, 下一个线程再poll时, 会把该first结点gc-unlink(自链接)掉的
            return;
        }

        // 2. 如果p为非活动结点、也不为尾结点, 且p=q, 即p=p.next, 说明p已经被gc-unlink了, 则直接返回,
        //    由于first标记为了逻辑删除, 下一个线程再poll时, 会把该first结点gc-unlink(自链接)掉的
        else if (p == q)
            return;

        // 3. 如果p为非活动结点、也不为尾结点、也还没有被删除, 说明first到p之间存在逻辑删除的结点, O只会有一个
        else {
            o = p;
            p = q;
        }
    }
}

// unlink最后一个非null结点, 且该结点是已经被标记为逻辑删除的结点(logical deletion), 只有在第一次并发同端删除才会进入, 且last=tail并不会被gc-unlink, 但会被unlink
private void unlinkLast(Node<E> last, Node<E> prev) {
    // assert last != null;
    // assert prev != null;
    // assert last.item == null;

    // 1. last为当前要解除链接的结点, p为解除结点的prev结点
    for (Node<E> o = null, p = prev, q;;) {

        // 2. 如果p的item不为null, 说明p为活动结点, 或者p的prev结点为null, 说明p为头结点了
        if (p.item != null || (q = p.prev) == null) {

            // 3. 此时p为活动结点或者为头结点, 如果o不为null, 说明p经过了移动, 且p的后继不是它自己, 说明新的p还没有被gc-unlink, 即p是last前的第一个活动结点,
            //    此时设置待删除的结点last的prev为p, 保证只能从last结点才能访问活动结点,
            //    => 如果下面还将p的next设置为last, 那么无论是从head还是tail, 都无法访问last与p之间的待删除结点了, 这就是unlink的第一步
            if (o != null && p.next != p && last.casPrev(prev, p)) {

                // 4. 如果last.prev设置成功了, 则p往后跳过已经逻辑删除的后继结点, 使得p的next为last结点， 这步属于unlink的第二步
                //    => unlink后, 中间的逻辑删除结点o就被last和p圈起来了, last的prev为p, p的next为last, 尽管o还可以访问last和p,
                //       但由于o没有了引用, GC最后都会发现它们
                skipDeletedSuccessors(p);

                // 5. 如果last的后继为null, 说明last为最后一个结点, 且p的prev为null或者p的item不为null, 说明p是个活动结点
                //    且p的后继为last, 说明p与last的链接还是正常的
                if (last.next == null &&
                    (p.prev == null || p.item != null) &&
                    p.next == last) {

                    // 6. 通过当前head指针, 去向前找到第一个活动结点, 并设置为最新的head指针, 这里保证了head一定是p或者p前面, 从而确保o在中间
                    updateHead(); // Ensure o is not reachable from head    // 确保 o 无法从头部到达, o为上一个线程留下的

                    // 7. 通过当前tail指针, 去向后找到最后一个活动结点, 并设置为最新的tail指针, 这里保证了tail一定是last或者last后面, 从而确保o在中间
                    updateTail(); // Ensure o is not reachable from tail    // 确保 o 无法从尾部到达, o为上一个线程留下的

                    // Finally, actually gc-unlink
                    // 8. gc-unlink: 自链接中间的o结点, 设置o的后继为NEXT_TERMINATOR, 即将o进行最后一步删除
                    o.lazySetPrev(o);
                    o.lazySetNext(nextTerminator());
                }
            }

            // 3. 如果p为活动结点或者为头结点, 如果o为null, 说明last到p之间不存在逻辑删除的结点, 直接返回即可,
            //    由于last标记为了逻辑删除, 下一个线程再poll时, 会把该last结点gc-unlink(自链接)掉的
            return;
        }

        // 2. 如果p为非活动结点、也不为头结点, 且p=q, 即p=p.prev, 说明p已经被gc-unlink了, 则直接返回,
        //    由于last标记为了逻辑删除, 下一个线程再poll时, 会把该last结点gc-unlink(自链接)掉的
        else if (p == q)
            return;

        // 3. 如果p为非活动结点、也不为头结点、也还没有被删除, 说明last到p之间存在逻辑删除的结点, O只会有一个
        else {
            o = p;
            p = q;
        }
    }
}
```

##### 高并发无阻塞获取原理

- **first（）**：获取第一个不为null的结点, 不会返回PREV_TERMINATOR。
- **succ（Node）**：返回p的后继结点, 如果碰到自链接, 则返回第一个不能null的结点, 从头再来。
- **last（）**：返回最后一个不能null的结点，不会返回NEXT_TERMINATOR。
- **prev（）**：返回p的前驱结点, 如果碰到自链接, 则返回最后一个不为null的结点, 从尾再来。

```java
// 获取第一个不为null的结点, 不会返回PREV_TERMINATOR
Node<E> first() {
    restartFromHead:
    for (;;)
        // 1. h、p为head指针
        for (Node<E> h = head, p = h, q;;) {
            
            // 2. p、q前移一个结点, 同时保证p永远是第一个不为null的结点, q为p的前驱
            if ((q = p.prev) != null &&
                (q = (p = q).prev) != null)
                
                // 3. 如果h不为最新的head指针, 则更新p为最新的head指针;
                //    如果为最新的head指针, 说明有两个线程插入了结点, 且最后一个线程还没有更新head指针, 此时更新p为p的前驱, 继续自旋, 继续往前找或者等待更新p为最新的h
                p = (h != (h = head)) ? h : q;
            
            // 2. 如果p的前驱为null, 且p==h, 说明此时p就是头结点了, 此时直接返回即可; 如果p!=h, 说明此时p已经移动到前面了，为第一个不为null的结点了,
            //    即别的线程插入了新的结点, 此时CAS设置h为p, 如果设置成功了说明h为有效结点, 此时返回p即可; 如果失败, 说明h已经移动过了, 不能CAS更新h指针
            else if (p == h
                     // p 可能是 PREV_TERMINATOR，但如果是这样，CAS 肯定会失败。
                     // It is possible that p is PREV_TERMINATOR,
                     // but if so, the CAS is guaranteed to fail.
                     || casHead(h, p))
                return p;
            
          // 2. 如果p的前驱为null, 且p和h已经移动过了, 此时需要重新进入自旋, 获取最新的head指针
            else
                continue restartFromHead;
        }
}

// 返回p的后继结点, 如果碰到自链接, 则返回第一个不能null的结点, 从头再来
final Node<E> succ(Node<E> p) {
    // TODO: should we skip deleted nodes here?
    Node<E> q = p.next;
    return (p == q) ? first() : q;
}

// 返回最后一个不能null的结点，不会返回NEXT_TERMINATOR
Node<E> last() {
    restartFromTail:
    for (;;)
        // 1. t、p为tail指针
        for (Node<E> t = tail, p = t, q;;) {

            // 2. p、q后移一个结点, 同时保证p永远是最后一个不为null的结点, q为p的后继
            if ((q = p.next) != null &&
                (q = (p = q).next) != null)
                // Check for tail updates every other hop.
                // If p == q, we are sure to follow tail instead.

                // 3. 如果t不为最新的tail指针, 则更新p为最新的tail指针;
                //    如果为最新的tail指针, 说明有两个线程插入了结点, 且最后一个线程还没有更新tail指针, 此时更新p为p的后继, 继续自旋, 继续往后找或者等待更新p为最新的t
                p = (t != (t = tail)) ? t : q;

            // 2. 如果p的后继为null, 且p==t, 说明此时p就是尾结点了, 此时直接返回即可; 如果p!=h, 说明此时p已经移动到后面了, 为最后一个不为null的结点了,
            //    即别的线程插入了新的结点, 此时CAS设置t为p, 如果设置成功了说明t为有效结点, 此时返回p即可; 如果失败, 说明t已经移动过了, 不能CAS更新t指针
            else if (p == t
                     // It is possible that p is NEXT_TERMINATOR,
                     // but if so, the CAS is guaranteed to fail.
                     || casTail(t, p))
                return p;

            // 2. 如果p的后继为null, 且p和t已经移动过了, 此时需要重新进入自旋, 获取最新的tail指针
            else
                continue restartFromTail;
        }
}

// 返回p的前驱结点, 如果碰到自链接, 则返回最后一个不为null的结点, 从尾再来
final Node<E> pred(Node<E> p) {
    Node<E> q = p.prev;
    return (p == q) ? last() : q;
}
```

##### 添加元素方法

- **add（E）**：Queue接口方法，添加元素到队尾，由于队列是无界的，所以不会触发队列已满异常。
- **offer（E）**：Queue接口方法，添加元素到队尾，由于队列是无界的，所以不会返回false。
- **addFirst（E）**：Deque接口方法，添加元素到队头，由于队列是无界的，所以不会触发队列已满异常。
- **addLast（E）**：Deque接口方法，添加元素到队尾，由于队列是无界的，所以不会触发队列已满异常。
- **offerFirst（E）**：Deque接口方法，添加元素到队头，由于队列是无界的，所以不会返回false。
- **offerLast（E）**：Deque接口方法，添加元素到队尾，由于队列是无界的，所以不会返回false。
- **pust（E）**：Deque接口方法，作为栈的方法使用，添加元素到队头（队列方法默认是添加到队尾）。

```java
// Queue接口方法，添加元素到队尾，由于队列是无界的，所以不会触发队列已满异常。
public boolean add(E e) {
    return offerLast(e);
}

// Queue接口方法，添加元素到队尾，由于队列是无界的，所以不会返回false。
public boolean offer(E e) {
    return offerLast(e);
}

// Deque接口方法，添加元素到队头，由于队列是无界的，所以不会触发队列已满异常。
public void addFirst(E e) {
    linkFirst(e);
}

// Deque接口方法，添加元素到队尾，由于队列是无界的，所以不会触发队列已满异常。
public void addLast(E e) {
    linkLast(e);
}

// Deque接口方法，添加元素到队头，由于队列是无界的，所以不会返回false。
public boolean offerFirst(E e) {
    linkFirst(e);
    return true;
}

// Deque接口方法，添加元素到队尾，由于队列是无界的，所以不会返回false。
public boolean offerLast(E e) {
    linkLast(e);
    return true;
}

// Deque接口方法，作为栈的方法使用，添加元素到队头（队列方法默认是添加到队尾）。
public void push(E e) { 
    addFirst(e);
}
```

##### 删除元素方法

- **remove（）**：Queue接口方法，删除队头元素，如果队列为空，则会抛出异常。
- **poll（）**：Queue接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null。
- **removeFirst（）**：Deque接口方法，删除队头元素，如果队列为空，则会抛出异常。
- **removeLast（）**：Deque接口方法，删除队尾元素，如果队列为空，则会抛出异常。
- **pollFirst（）**：Deque接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null。
- **pollLast（）**：Deque接口方法，删除队尾元素，如果队列为空，不会抛出异常，而是返回null。
- **removeFirstOccurrence（Object）**：Deque接口方法，删除第一个指定的元素(从头到尾找)，找到并删除成功返回true，否则返回false。
- **removeLastOccurrence（Object）**：Deque接口方法，删除最后一个指定的元素(从尾到头找)，找到并删除成功返回true，否则返回false。
- **remove（Object）**：Collection接口方法， 删除指定元素(从头到尾找)，找到并删除成功返回true，否则返回false。
- **pop（）**：Deque接口方法，作为栈的方法使用，删除队头元素（队列方法默认也是删除队头元素）。

```java
// Queue接口方法，删除队头元素，如果队列为空，则会抛出异常。
public E remove() { 
    return removeFirst(); 
}

// Queue接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null。
public E poll() { 
    return pollFirst(); 
}

// Deque接口方法，删除队头元素，如果队列为空，则会抛出异常。
public E removeFirst() {
    return screenNullResult(pollFirst());
}

// Deque接口方法，删除队尾元素，如果队列为空，则会抛出异常。
public E removeLast() {
    return screenNullResult(pollLast());
}
private E screenNullResult(E v) {
    if (v == null)
        throw new NoSuchElementException();
    return v;
}

// Deque接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null。
public E pollFirst() {
    for (Node<E> p = first(); p != null; p = succ(p)) {
        E item = p.item;
        if (item != null && p.casItem(item, null)) {
            unlink(p);
            return item;
        }
    }
    return null;
}

// Deque接口方法，删除队尾元素，如果队列为空，不会抛出异常，而是返回null。
public E pollLast() {
    for (Node<E> p = last(); p != null; p = pred(p)) {
        E item = p.item;
        if (item != null && p.casItem(item, null)) {
            unlink(p);
            return item;
        }
    }
    return null;
}

// Deque接口方法，删除第一个指定的元素(从头到尾找)，找到并删除成功返回true，否则返回false。
public boolean removeFirstOccurrence(Object o) {
    checkNotNull(o);
    for (Node<E> p = first(); p != null; p = succ(p)) {
        E item = p.item;
        if (item != null && o.equals(item) && p.casItem(item, null)) {
            unlink(p);
            return true;
        }
    }
    return false;
}

// Deque接口方法，删除最后一个指定的元素(从尾到头找)，找到并删除成功返回true，否则返回false。
public boolean removeLastOccurrence(Object o) {
    checkNotNull(o);
    for (Node<E> p = last(); p != null; p = pred(p)) {
        E item = p.item;
        if (item != null && o.equals(item) && p.casItem(item, null)) {
            unlink(p);
            return true;
        }
    }
    return false;
}

// Collection接口方法， 删除指定元素(从头到尾找)，找到并删除成功返回true，否则返回false。 
public boolean remove(Object o) {
    return removeFirstOccurrence(o);
}

// Deque接口方法，作为栈的方法使用，删除队头元素（队列方法默认也是删除队头元素）。
public E pop()  { 
    return removeFirst(); 
}
```

##### 获取元素方法

- **element（）**：Queue接口方法，获取队头元素，如果队列为空，则会抛出异常。
- **peek（）**：Queue接口方法，获取队头元素，如果队列为空，不会抛出异常，而是返回null。
- **getFirst（）**：Deque接口方法，获取队头元素，如果队列为空，则会抛出异常。
- **getLast（）**：Deque接口方法，获取队尾元素，如果队列为空，则会抛出异常。
- **peekFirst（）**：Deque接口方法，获取队头元素，如果队列为空，不会抛出异常，而是返回null。
- **peekLast（）**：Deque接口方法，获取队尾元素，如果队列为空，不会抛出异常，而是返回null。

```java
// Queue接口方法，获取队头元素，如果队列为空，则会抛出异常。
public E element() { 
    return getFirst();
}

// Queue接口方法，获取队头元素，如果队列为空，不会抛出异常，而是返回null。
public E peek()  { 
    return peekFirst(); 
}

// Deque接口方法，获取队头元素，如果队列为空，则会抛出异常。
public E getFirst() {
    return screenNullResult(peekFirst());
}
private E screenNullResult(E v) {
    if (v == null)
        throw new NoSuchElementException();
    return v;
}

// Deque接口方法，获取队尾元素，如果队列为空，则会抛出异常。
public E getLast() {
    return screenNullResult(peekLast());
}

// Deque接口方法，获取队头元素，如果队列为空，不会抛出异常，而是返回null。
public E peekFirst() {
    for (Node<E> p = first(); p != null; p = succ(p)) {
        E item = p.item;
        if (item != null)
            return item;
    }
    return null;
}

// Deque接口方法，获取队尾元素，如果队列为空，不会抛出异常，而是返回null。
public E peekLast() {
    for (Node<E> p = last(); p != null; p = pred(p)) {
        E item = p.item;
        if (item != null)
            return item;
    }
    return null;
}
```

