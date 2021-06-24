## 源码篇

### List？

List，**有序集合**（也称为序列）。此接口的用户可以精确控制列表中每个元素的插入位置。用户**可以通过其整数索引（在列表中的位置）访问元素**，并在列表中搜索元素。

```java
public interface Iterable<T> {
	Iterator<T> iterator();
    default void forEach(Consumer<? super T> action) {...}
    default Spliterator<T> spliterator() {...}
}

public interface ListIterator<E> extends Iterator<E> {
    boolean hasNext();
    E next();
    boolean hasPrevious();
    E previous();
    int nextIndex();
    int previousIndex();
    void remove();
    void set(E e);
    void add(E e);
}

public interface Collection<E> extends Iterable<E> {
    int size()；
    boolean isEmpty();
    boolean add(E e);
    boolean remove(Object o)；
    Object[] toArray()；
    <T> T[] toArray(T[] a)；
    ...
}

public interface List<E> extends Collection<E> {
    void add(int index, E element);
    E remove(int index);
    E get(int index);
    E set(int index, E element);
    ...
}
```

- **典型实现**：**ArrayList、LinkedList、CopyOnWriteArrayList、SynchronizedList、Vector、Stack**

#### ArrayList

![1621431762562](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621431762562.png)

##### 特点

- ArrayList底层基于数组实现，**支持对元素进行快速随机访问，适合随机查找和遍历**，不适合大量插入和删除。**允许null元素**。
- **默认初始大小为10**，当数组容量不够时，会触发扩容机制（**扩大到当前的1.5倍**），需要将原来数组的数据复制到新的数组中。当从 ArrayList 的中间位置插入或者删除元素时，需要对数组进行复制、移动、代价比较高。
- 非线程安全的，如果要使用同步的List方法，可以考虑使用Collections＃synchronizedList进行包装。

- 迭代器是快速失败的。

##### 构造方法

- **无参的构造函数**：赋值初始化数组。
- **指定容量构造函数**：创建一个Object数组，并赋值给elementData成员变量。
- **指定复制集合的构造函数**：调用Collection接口toArray（）转换集合为数组，再用Arrays.copyOf（...）复制数组（底层调用的是System.arraycopy（...））。

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    
	private static final int DEFAULT_CAPACITY = 10;
	private static final Object[] EMPTY_ELEMENTDATA = {};
	private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
	transient Object[] elementData;
	private int size;
    
	// 无参的构造函数
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;// 代表初始化数组
    }

    // 指定容量的构造函数
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;// 代表空元素数组
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+initialCapacity);
        }
    }

    // 指定复制集合的构造函数
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();// 赋值数组， Collection接口的toArray()
        if ((size = elementData.length) != 0) {
            if (elementData.getClass() != Object[].class)
                // 底层调用System.arraycopy（...）
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            this.elementData = EMPTY_ELEMENTDATA;// 代表空元素数组
        }
    }
}
```

##### 迭代方法

> 此类的{@link #iterator（）}和{@link #listIterator（int）}方法返回的迭代器是**快速失败**的：即如果在创建迭代器后的任何时间都对结构进行了结构修改， 除了通过迭代器自己的{@link ListIterator＃remove（）remove}或{@link ListIterator＃add（Object）add}方法之外，迭代器都会抛出{@link ConcurrentModificationException}。 因此，面对并发修改，迭代器会快速干净地失败，而不会在未来的不确定时间内冒任意、不确定的行为的风险。 由{@link #elements（）elements}方法返回的{@link Enumeration Enumerations}不是快速失败的。
>
> 请注意，迭代器的快速失败行为无法得到保证，因为通常来说，在存在不同步的并发修改的情况下，不可能做出任何严格的保证。 **快速失败的迭代器会尽最大努力抛出{@code ConcurrentModificationException}**。 因此，编写依赖于此异常的程序的正确性是错误的：**迭代器的快速失败行为应仅用于检测错误**。

所有迭代操作都有：**并发修改检查**，检测对象有没有发生并发修改的操作。

- **iterator（）**：获取Collection迭代器。
- **listIterator（）**：获取默认的List迭代器。
- **listIterator（int）**：获取指定索引的List迭代器。

##### 扩容方法

- **扩容条件**：数组实际长度（size）大于当前数组长度（elementData.length）时。

- **grow（int）**：增长元素数组到指定容量。

1. 计算新容量： = 旧容量（当前元素数组实际长度） *  1.5。
2. 容量判断：如果新容量小于指定容量，则设置为指定容量。以及最大长度判断：Integer.MAX_VALUE - 8。
3. 移动元素到新数组：Arrays.copyOf（...）（底层调用的是System.arraycopy（...））。

```java
// 扩容条件
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);// minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
}

// 扩容方法
private void grow(int minCapacity) {
    // 旧数组容量（当前元素数组实际长度）
    int oldCapacity = elementData.length;

    // 新容量 = 旧容量 + 旧容量/2 = 旧容量 * 1.5 => 扩容为原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);

    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;

    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);// 最大长度判断：Integer.MAX_VALUE - 8
    
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

##### 增加元素方法

- **add（E）**：添加元素到末尾位置。

1. 扩容判断。
2. 添加元素、实际大小 + 1。
3. 返回true。

- **add（int，E）**：添加元素到指定位置。无返回值。

1. 索引越界判断。
2. 扩容判断。
3. 移动数组元素：System.arraycopy（...）。
4. 添加元素、实际大小 + 1。

```java
// 添加元素到末尾位置
public boolean add(E e) {
    ensureCapacityInternal(size + 1); // Increments modCount!! + 扩容判断
    elementData[size++] = e;
    return true;
}

// 添加元素到指定位置
public void add(int index, E element) {
    rangeCheckForAdd(index); // 索引越界校验
    ensureCapacityInternal(size + 1);  // Increments modCount!! + 扩容判断
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    elementData[index] = element;
    size++;
}
```

##### 获取元素方法

- **get（int）**：根据索引获取数组元素，O(1)。

```java
public E get(int index) {
    rangeCheck(index);// 索引越界检查
    return elementData(index);// 根据索引获取数组元素，O(1)
}

E elementData(int index) {
    return (E) elementData[index];
}
```

##### 删除元素方法

- **remove（int）**：删除指定索引元素。

1. 索引越界校验。
2. 根据索引获取数组元素，O(1)。
3. 移动数组元素：System.arraycopy（...）。
4. 清空元素、实际大小 - 1。
5. 返回旧元素。

- **remove（Object）**：删除指定元素，用equals（...）判断。

1. 元素为null：遍历数组、找到第一个为null的元素、移动数组元素、清空元素、实际大小 - 1、返回true。
2. 元素非null：遍历数组、找到第一个equal的元素、移动数组元素、清空元素、实际大小 - 1、返回true。
3. 否则代表删除失败，返回false。

- **removeRange（int，int）**：删除fromIndex~toIndex-1内的元素。无返回值。

1. 移动数组元素、清空元素、更新实际大小 。

```java
// 删除指定索引元素
public E remove(int index) {
    rangeCheck(index);// 索引越界校验
    modCount++;
    
    E oldValue = elementData(index);// 根据索引获取数组元素，O(1)

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}

// 删除指定元素: equals判断
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    
    return false;
}
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}

// 删除fromIndex~toIndex-1内的元素
protected void removeRange(int fromIndex, int toIndex) {
    modCount++;

    int numMoved = size - toIndex;
    System.arraycopy(elementData, toIndex, elementData, fromIndex,
                     numMoved);

    // clear to let GC do its work
    int newSize = size - (toIndex-fromIndex);
    for (int i = newSize; i < size; i++) {
        elementData[i] = null;
    }

    size = newSize;
}
```

##### 替换元素方法

- **set（int，E）**：替换指定位置的元素。

1. 索引越界检查、根据索引获取数组元素O(1)、设置数组元素O(1)、返回旧元素。

```java
// 替换指定位置的元素
public E set(int index, E element) {
    rangeCheck(index);// 索引越界检查
    E oldValue = elementData(index);// 根据索引获取数组元素, O(1)
    elementData[index] = element;
    return oldValue;
}
```

#### LinkedList

![1621435281320](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621435281320.png)

##### 特点

- **底层基于双向链表实现，适合数据的动态插入和删除。允许null元素。**
- 内部提供了List接口中没有定义的方法，**用于操作表头和表尾元素，可以当作栈、队列和双向队列使用**。比如jdk官方推荐使用基于LinkedList的Deque进行栈操作。
- 非线程安全的，如果要使用同步的List方法，可以考虑使用Collections＃synchronizedList进行包装。
- **实现了List、Queue、Deque接口，还提供了堆栈的方法**，所以可以把LinkedList看作是List、Queue、Deque和Stack。作为队列时，不会有替换元素的方法，且获取元素方法都是检索栈顶的元素。
- 迭代器是快速失败的。

##### 构造方法

- **空参的构造函数**：什么都不做。
- **指定复制集合的构造函数**：

1. 调用空参的构造函数：什么也不做。
2. 指针索引校验。
3. 根据索引获取插入结点的位置：succ为后结点, pred为前结点。
4. 遍历待插入的数组、插入元素，维护指针关系。
5. 更新succ、pred前后指针关系。
6. 更新实际大小。
7. 返回true。

```java
public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;
    transient Node<E> first;
    transient Node<E> last;
    
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    
    // 空参的构造函数
    public LinkedList() {
    }
    
    // 指定复制集合的构造函数
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }
    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);// 指针索引校验

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;
		
		// 根据索引获取插入结点的位置：succ为后结点, pred为前结点
        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }
		
		// 遍历待插入的数组、插入元素，维护指针关系
        for (Object o : a) {
            E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);

            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

		// 更新succ、pred前后指针关系
        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }
		
		// 更新实际大小
        size += numNew;
        modCount++;
        
        return true;
    }
}
```

##### 迭代方法

- **listIterator（int）**：获取指定索引的List迭代器。没有Collection迭代器方法。
  - 所有迭代操作都有：并发修改检查，检测对象有没有发生并发修改的操作。

##### 扩容方法

双向链表结构：无需系统扩容，新增结点，维护结点指针关系即可。

- **linkFirst（E）**：插入指定元素到头结点前。

1. 构造新结点：备份头指针、创建当前结点、链接前驱、链接后继。
2. 原头结点为空：更新尾指针（头指针）。
3. 原头结点不为空：设置该结点为原头结点的前驱。
4. 更新实际大小。

- **linkLast（E）**：插入指定元素到末尾结点后。

1. 构造新结点：备份尾指针、创建当前结点、链接前驱、链接后继。
2. 原尾结点为空：更新头指针（尾指针）。
3. 原尾结点不为空：设置该结点为原尾结点的后继。
4. 更新实际大小。

- **linkBefore（E，Node< E >）**：插入元素到某个结点前。

1. 构造新结点：备份后结点（指那个某结点）的前驱指针、创建当前结点、链接前驱、链接后继。
2. 原前驱结点为空：更新头指针（尾指针）。
3. 原前驱结点不为空：设置该结点为原后前驱结点的后继。
4. 更新实际大小。

```java
// 插入指定元素到头结点前
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;// 更新first指针指向新结点
    if (f == null)// 原头结点为空
        last = newNode;// 更新末尾指针
    else
        f.prev = newNode;// 设置该结点为原头结点的前驱

    // 更新实际大小
    size++;
    modCount++;
}

// 插入指定元素到末尾结点后
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;// 更新last指针为当前结点
    if (l == null)// 原尾结点为空
        first = newNode;// 更新头指针
    else
        l.next = newNode;// 设置该结点为原尾结点的后继

    // 更新实际大小
    size++;
    modCount++;
}

// 插入元素到某个结点前
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;// 更新后结点的前驱为当前结点

    if (pred == null)// 原前驱结点为空
        first = newNode;// 更新头指针
    else
        pred.next = newNode;// 设置该结点为原后前驱结点的后继

    // 更新实际大小
    size++;
    modCount++;
}
```

##### 增加元素方法

- **add（E）**：List接口，添加元素到末尾位置。作为队列时，由于是链表实现的，所以不会触发队列已满异常。

1. 插入指定元素到尾结点后、返回true。

- **add（int，E）**：List接口，添加元素到指定位置。无返回值。

1. 索引指针校验。
2. 索引等于实际大小：插入指定元素到末尾结点后。
3. 索引小于实际大小：插入元素到某个结点前。

- **offer（E）**：Queue接口，添加元素到队尾。
- **addFirst（E）**：Deque接口，添加元素到队头，由于是链表实现的，所以不会触发队列已满异常。
- **addLast（E）**：Deque接口，添加元素到队尾，由于是链表实现的，所以不会触发队列已满异常。
- **offerFirst（E）**：Deque接口，添加元素到队头。
- **offerLast（E）**：Deque接口，添加元素到队尾。
- **push（E）**：Deque接口，作为栈的方法使用，添加元素到队头（队列方法默认是添加到队尾）。

```java
// List、Queue接口，添加元素到队尾。作为队列时，由于是链表实现的，所以不会触发队列已满异常。
public boolean add(E e) {
    linkLast(e);
    return true;
}

// List接口，添加元素到指定位置
public void add(int index, E element) {
    checkPositionIndex(index);// 索引指针校验
    if (index == size)
        linkLast(element);// 添加元素到末尾位置
    else
        linkBefore(element, node(index));// 添加元素到结点前的位置
}

// Queue接口，添加元素到队尾
public boolean offer(E e) {
    return add(e);
}

// Deque接口，添加元素到队头，由于是链表实现的，所以不会触发队列已满异常。
public void addFirst(E e) {
    linkFirst(e);
}

// Deque接口，添加元素到队尾，由于是链表实现的，所以不会触发队列已满异常。
public void addLast(E e) {
    linkLast(e);
}

// Deque接口，添加元素到队头
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}

// Deque接口，添加元素到队尾
public boolean offerLast(E e) {
    addLast(e);
    return true;
}

// Deque接口，作为栈的方法使用，添加元素到队头（队列方法默认是添加到队尾）
public void push(E e) {
    addFirst(e);
}
```

##### 获取元素方法

作为队列时，不会有替换元素的方法，且获取元素方法都是检索栈顶的元素。

- **get（int）**：List接口，根据索引获取元素。

1. 索引越界校验。
2. 根据索引获取结点O(n/2)：如果指针在链表的前半部分，则从前半部分查找，返回index位置结点；否则，则从后半部分查找，返回index位置结点。
3. 返回该结点元素。

- **element（）**：Queue接口方法，获取队头元素，如果队列为空，则会抛出异常。
- **peek（）**：Queue接口方法，获取队头元素，如果队列为空，不会抛出异常，而是返回null。
- **getFirst（）**：Deque接口方法，获取队头元素，如果队列为空，则会抛出异常。
- **getLast（）**：Deque接口方法，获取队尾元素，如果队列为空，则会抛出异常。
- **peekFirst（）**：Deque接口方法，获取队头元素，如果队列为空，不会抛出异常，而是返回null。
- **peekLast（）**：Deque接口方法，获取队尾元素，如果队列为空，不会抛出异常，而是返回null。

```java
// List接口，根据索引获取元素
public E get(int index) {
    checkElementIndex(index);// 索引越界校验 
    return node(index).item;// 根据索引获取结点, 返回该结点元素, O(n/2)
}
Node<E> node(int index) {
    // assert isElementIndex(index);
    if (index < (size >> 1)) {
        // 如果指针在链表的前半部分, 则从前半部分查找, 返回index位置结点
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        // 否则，则从后半部分查找, 返回index位置结点
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}

// Queue接口，获取队头元素，如果队列为空，则会抛出异常
public E element() {
    return getFirst();
}

// Queue接口方法，获取队头元素，如果队列为空，不会抛出异常，而是返回null
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}

// Deque接口方法，获取队头元素，如果队列为空，则会抛出异常
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

// Deque接口方法，获取队尾元素，如果队列为空，则会抛出异常
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}

// Deque接口方法，获取队头元素，如果队列为空，不会抛出异常，而是返回null
public E peekFirst() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}

// Deque接口方法，获取队尾元素，如果队列为空，不会抛出异常，而是返回null
public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}
```

##### 删除元素方法

- **remove（int）**：List接口，删除指定位置元素。

1. 索引越界校验。
2. 根据索引获取结点O(n/2)：如果指针在链表的前半部分，则从前半部分查找，返回index位置结点；否则，则从后半部分查找，返回index位置结点。
3. 解除该结点：维护该结点原前驱、后继结点的指针，清空该结点、前驱、后继，更新实际大小，返回旧元素。

- **remove（Object）**：List，Deque接口，删除指定元素，使用equals（...）判断。

1. 元素为空：遍历找到第一个null，解除该结点。
2. 元素不为空：遍历找到第一个equal的元素，解除该结点。
3. 删除成功返回true，删除失败返回false。

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
// List接口，删除指定位置元素
public E remove(int index) {
    checkElementIndex(index);// 索引越界校验
    return unlink(node(index));// 根据索引获取结点, 然后解除结点
}
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;
    
    if (prev == null) {// 原前驱为空
        first = next;// 更新头结点为原后继
    } else {// 否则，更新原前驱的后继为原后继，清空该结点的前驱
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {// 原后继为空
        last = prev;// 更新尾结点为原前驱
    } else {// 否则，更新原后继的前驱为原前驱，清空该结点的后继
        next.prev = prev;
        x.next = null;
    }

    // 20201118 清空该节点，更新实际大小
    x.item = null;
    size--;
    modCount++;

    // 返回旧元素
    return element;
}

// List，Deque接口，删除指定元素：equals判断
public boolean remove(Object o) {
    if (o == null) {
        // 遍历找到第一个null, 解除该结点
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        // 遍历找到同一个元素, 解除该结点
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }

    return false;
}

// Queue接口方法，删除队头元素，如果队列为空，则会抛出异常
public E remove() {
    return removeFirst();
}

// Queue接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

// Deque接口方法，删除队头元素，如果队列为空，则会抛出异常
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

// Deque接口方法，删除队尾元素，如果队列为空，则会抛出异常
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

// Deque接口方法，删除队头元素，如果队列为空，不会抛出异常，而是返回null
public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

// Deque接口方法，删除队尾元素，如果队列为空，不会抛出异常，而是返回null
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}

// Deque接口方法，删除第一个指定的元素(从头到尾找)，找到并删除成功返回true，否则返回false
public boolean removeFirstOccurrence(Object o) {
    return remove(o);
}

// Deque接口方法，删除最后一个指定的元素(从尾到头找)，找到并删除成功返回true，否则返回false
public boolean removeLastOccurrence(Object o) {
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

// Deque接口方法，作为栈的方法使用，删除队头元素（队列方法默认也是删除队头元素）
public E pop() {
    return removeFirst();
}
```

##### 替换元素方法

作为队列时，不会有替换元素的方法，且获取元素方法都是检索栈顶的元素。

- **set（int，E）**：替换指定位置的元素。

1. 索引越界校验、根据索引获取结点、替换结点元素、返回旧元素。

```java
// 替换指定位置的元素
public E set(int index, E element) {
    checkElementIndex(index);// 索引越界校验
    Node<E> x = node(index);// 根据索引获取结点
    E oldVal = x.item;
    x.item = element;// 替换结点元素
    return oldVal;
}
```

#### CopyOnWriteArrayList

![1621484957471](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621484957471.png)

##### 特点

- ArrayList的线程安全变体，其中所有可变操作（add、remove、set方法）都**通过对基础数组进行全新复制来实现**，不需要扩容机制。
- 在**无法或不想同步遍历**而又需要防止并发线程之间的干扰时很有用。
- **基于数组快照方式查找数据**，所有迭代操作都没有并发修改检查，不需要快速失败，但不支持add、remove、set方法，会抛出**UnsupportedOperationException**。

##### 构造方法

- **无参的构造函数**：创建一个0长度的数组。
- **指定复制数组的构造函数**：Arrays.copyOf（...）复制数组（底层调用的是System.arraycopy（...））。
- **指定复制集合的构造函数**：调用Collection接口toArray（）转换集合为数组，再用Arrays.copyOf（...）复制数组（底层调用的是System.arraycopy（...））。

```java
public class CopyOnWriteArrayList<E>
implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
	final transient ReentrantLock lock = new ReentrantLock();
	private transient volatile Object[] array;// volatile修饰
   
    final void setArray(Object[] a) {
        array = a;
    }
    
	// 无参的构造函数
	public CopyOnWriteArrayList() {
        setArray(new Object[0]);
    }
    
    // 指定复制数组的构造函数
    public CopyOnWriteArrayList(E[] toCopyIn) {
        setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
    }
    
    // 指定复制集合的构造函数
    public CopyOnWriteArrayList(Collection<? extends E> c) {
        Object[] elements;
        if (c.getClass() == CopyOnWriteArrayList.class)
            elements = ((CopyOnWriteArrayList<?>)c).getArray();
        else {
            elements = c.toArray();
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elements.getClass() != Object[].class)
                elements = Arrays.copyOf(elements, elements.length, Object[].class);
        }
        setArray(elements);
    }
}
```

##### 迭代方法

所有迭代操作**都没有并发修改检查**，检测对象有没有发生并发修改的操作。

next（）、previous（）、forEachRemaining（Consumer）：**基于snapshot查找数据**。

不支持remove（）、set（）、add（）方法，会抛出**UnsupportedOperationException**。

- **iterator（）**：获取Collection迭代器。
- **listIterator（）**：获取默认的List迭代器。
- **listIterator（int）**：获取指定索引的List迭代器。

```java
// 获取Collection迭代器
public Iterator<E> iterator() {
    return new COWIterator<E>(getArray(), 0);
}

// 获取默认的List迭代器
public ListIterator<E> listIterator() {
    return new COWIterator<E>(getArray(), 0);
}

// 获取指定索引的List迭代器
public ListIterator<E> listIterator(int index) {
    Object[] elements = getArray();
    int len = elements.length;
    if (index < 0 || index > len)
        throw new IndexOutOfBoundsException("Index: "+index);

    return new COWIterator<E>(elements, index);
}

static final class COWIterator<E> implements ListIterator<E> {
    /** Snapshot of the array */
    private final Object[] snapshot;
    /** Index of element to be returned by subsequent call to next.  */
    private int cursor;

    private COWIterator(Object[] elements, int initialCursor) {
        cursor = initialCursor;
        snapshot = elements;
    }

    public boolean hasNext() {
        return cursor < snapshot.length;
    }

    public boolean hasPrevious() {
        return cursor > 0;
    }
	
    public E next() {
        if (! hasNext())
            throw new NoSuchElementException();
        return (E) snapshot[cursor++];// 基于snapshot查找数据
    }

    public E previous() {
        if (! hasPrevious())
            throw new NoSuchElementException();
        return (E) snapshot[--cursor];// 基于snapshot查找数据
    }

    public int nextIndex() {
        return cursor;
    }

    public int previousIndex() {
        return cursor-1;
    }
	
    // 不支持remove（）、set（）、add（）方法，会抛出UnsupportedOperationException
    public void remove() {
        throw new UnsupportedOperationException();
    }
    public void set(E e) {
        throw new UnsupportedOperationException();
    }
    public void add(E e) {
        throw new UnsupportedOperationException();
    }

    @Override
    public void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        Object[] elements = snapshot;// 基于snapshot查找数据
        final int size = elements.length;
        for (int i = cursor; i < size; i++) {
            E e = (E) elements[i];
            action.accept(e);
        }
        cursor = size;
    }
}
```

##### 扩容方法

**不需要扩容机制**，因为每个数组都是“刚需”，而每次添加又都全量复制。

##### 增加元素方法

所有添加动作之前都需要**ReentrantLock加锁**，在添加完成后解锁。

所有添加动作都是通过Arrays.copyOf（...）**复制新数组**来完成的（底层调用的是System.arraycopy（...））。

- add（E）：添加元素到末尾位置。
- add（int，E）：添加元素到指定位置。
- addIfAbsent（E）：如果**当前快照不存在指定元素**，才添加该元素到末尾位置。

```java
// 添加元素到末尾位置
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();// ReentrantLock加锁
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

// 添加元素到指定位置
public void add(int index, E element) {
    final ReentrantLock lock = this.lock;
    lock.lock();// ReentrantLock加锁
    try {
        Object[] elements = getArray();
        int len = elements.length;
        if (index > len || index < 0)
            throw new IndexOutOfBoundsException("Index: "+index+
                                                ", Size: "+len);
        Object[] newElements;
        int numMoved = len - index;
        if (numMoved == 0)
            newElements = Arrays.copyOf(elements, len + 1);
        else {
            newElements = new Object[len + 1];
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index, newElements, index + 1,
                             numMoved);
        }
        newElements[index] = element;
        setArray(newElements);
    } finally {
        lock.unlock();
    }
}

// 如果当前快照不存在指定元素，才添加该元素到末尾位置
public boolean addIfAbsent(E e) {
    Object[] snapshot = getArray();
    return indexOf(e, snapshot, 0, snapshot.length) >= 0 ? false :
    addIfAbsent(e, snapshot);
}
private boolean addIfAbsent(E e, Object[] snapshot) {
    final ReentrantLock lock = this.lock;
    lock.lock();// ReentrantLock加锁
    try {
        Object[] current = getArray();
        int len = current.length;
        if (snapshot != current) {
            // Optimize for lost race to another addXXX operation
            int common = Math.min(snapshot.length, len);
            for (int i = 0; i < common; i++)
                if (current[i] != snapshot[i] && eq(e, current[i]))
                    return false;
            if (indexOf(e, current, common, len) >= 0)
                return false;
        }
        Object[] newElements = Arrays.copyOf(current, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

##### 获取元素方法

- get（int）：根据索引获取元素。

1. 获取当前数组快照、根据索引获取快照中的元素。

```java
// 根据索引获取元素
public E get(int index) {
    return get(getArray(), index);
}
final Object[] getArray() {
    return array;
}
private E get(Object[] a, int index) {
    return (E) a[index];
}

```

##### 删除元素方法

所有删除动作之前都需要**ReentrantLock加锁**，在删除完成后解锁。

所有删除动作都是通过Arrays.copyOf（...）**复制新数组**来完成的（底层调用的是System.arraycopy（...））。

- **remove（int）**：删除指定索引元素。
- **remove（Object）**：删除指定元素，用equals（...）判断**数组快照**。
- **removeRange（int，int）**：删除fromIndex~toIndex-1内的元素。无返回值。

```java
// 删除指定索引元素
public E remove(int index) {
    final ReentrantLock lock = this.lock;
    lock.lock();// ReentrantLock加锁
    try {
        Object[] elements = getArray();
        int len = elements.length;
        E oldValue = get(elements, index);
        int numMoved = len - index - 1;
        if (numMoved == 0)
            setArray(Arrays.copyOf(elements, len - 1));
        else {
            Object[] newElements = new Object[len - 1];
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index + 1, newElements, index,
                             numMoved);
            setArray(newElements);
        }
        return oldValue;
    } finally {
        lock.unlock();
    }
}

// 删除指定元素
public boolean remove(Object o) {
    Object[] snapshot = getArray();
    int index = indexOf(o, snapshot, 0, snapshot.length);
    return (index < 0) ? false : remove(o, snapshot, index);
}
private static int indexOf(Object o, Object[] elements,
                           int index, int fence) {
    if (o == null) {
        for (int i = index; i < fence; i++)
            if (elements[i] == null)
                return i;
    } else {
        for (int i = index; i < fence; i++)
            if (o.equals(elements[i]))
                return i;
    }
    return -1;
}
private boolean remove(Object o, Object[] snapshot, int index) {
    final ReentrantLock lock = this.lock;
    lock.lock();// ReentrantLock加锁
    try {
        Object[] current = getArray();
        int len = current.length;
        if (snapshot != current) findIndex: {
            int prefix = Math.min(index, len);
            for (int i = 0; i < prefix; i++) {
                if (current[i] != snapshot[i] && eq(o, current[i])) {
                    index = i;
                    break findIndex;
                }
            }
            if (index >= len)
                return false;
            if (current[index] == o)
                break findIndex;
            index = indexOf(o, current, index, len);
            if (index < 0)
                return false;
        }
        Object[] newElements = new Object[len - 1];
        System.arraycopy(current, 0, newElements, 0, index);
        System.arraycopy(current, index + 1,
                         newElements, index,
                         len - index - 1);
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

// 删除fromIndex~toIndex-1内的元素
void removeRange(int fromIndex, int toIndex) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;

        if (fromIndex < 0 || toIndex > len || toIndex < fromIndex)
            throw new IndexOutOfBoundsException();
        int newlen = len - (toIndex - fromIndex);
        int numMoved = len - toIndex;
        if (numMoved == 0)
            setArray(Arrays.copyOf(elements, newlen));
        else {
            Object[] newElements = new Object[newlen];
            System.arraycopy(elements, 0, newElements, 0, fromIndex);
            System.arraycopy(elements, toIndex, newElements,
                             fromIndex, numMoved);
            setArray(newElements);
        }
    } finally {
        lock.unlock();
    }
}
```

##### 替换元素方法

- **set（int，E）**：替换指定位置的元素。
  - 需要**ReentrantLock加锁**，在完成后解锁。
  - 通过Arrays.copyOf（...）**复制新数组**来完成的（底层调用的是System.arraycopy（...））。

```java
// 替换指定位置的元素
public E set(int index, E element) {
    final ReentrantLock lock = this.lock;
    lock.lock();// ReentrantLock加锁
    try {
        Object[] elements = getArray();
        E oldValue = get(elements, index);

        if (oldValue != element) {
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len);
            newElements[index] = element;
            setArray(newElements);
        } else {
            // Not quite a no-op; ensures volatile write semantics
            setArray(elements);
        }
        return oldValue;
    } finally {
        lock.unlock();
    }
}
```

#### SynchronizedList

![1621503899100](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621503899100.png)

##### SynchronizedCollection

- java.util.Collections内部类
- 成员变量有Collection引入和一个mutex对象锁，用于包装传入的集合，使其所有方法都能成为同步方法（**加synchronized关键字**）。
- 获取集合的迭代器后，需要自己实现同步的迭代方法。

```java
public class Collections {
    static class SynchronizedCollection<E> implements Collection<E>, Serializable {
        private static final long serialVersionUID = 3053995032091335093L;

        final Collection<E> c;  // Backing Collection
        final Object mutex;     // Object on which to synchronize

        SynchronizedCollection(Collection<E> c) {
            this.c = Objects.requireNonNull(c);
            mutex = this;
        }

        SynchronizedCollection(Collection<E> c, Object mutex) {
            this.c = Objects.requireNonNull(c);
            this.mutex = Objects.requireNonNull(mutex);
        }

        public int size() {
            synchronized (mutex) {return c.size();}
        }
        public boolean isEmpty() {
            synchronized (mutex) {return c.isEmpty();}
        }
        public boolean contains(Object o) {
            synchronized (mutex) {return c.contains(o);}
        }
        public boolean add(E e) {
            synchronized (mutex) {return c.add(e);}
        }
        public boolean remove(Object o) {
            synchronized (mutex) {return c.remove(o);}
        }
        // Override default methods in Collection
        @Override
        public void forEach(Consumer<? super E> consumer) {
            synchronized (mutex) {c.forEach(consumer);}
        }
		...
    }
}
```

##### 特点

- java.util.Collections内部类。
- 继承自SynchronizedCollection类，相当于拥有成员变量Collection引用和mutex对象锁，用于包装传入的集合，使**其所有方法都能成为同步方法（加synchronized关键字）**，其在SynchronizedCollection类上扩展了属于List集合的同步方法。
- 在获取集合的迭代器后，需要自己实现同步的迭代方法。

##### 构造方法

- **只传入List集合的构造函数**：设置List引用，持有父类mutex对象锁。
- **传入List集合与对象锁的构造函数**：设置List引用与对象锁引用。

```java
static class SynchronizedList<E> extends SynchronizedCollection<E> implements List<E> {
    final List<E> list;
    
    // 只传入List集合的构造函数
    SynchronizedList(List<E> list) {
        super(list);
        this.list = list;
    }
    // 传入List集合与对象锁的构造函数
    SynchronizedList(List<E> list, Object mutex) {
        super(list, mutex);
        this.list = list;
    }
    ...
}
```

##### 迭代方法

- **listIterator（）**：获取List引用的默认迭代器，其同步迭代方法需要自己实现。
- **listIterator（int）**：获取指定索引的迭代器，其同步迭代方法需要自己实现。

```java
// 获取默认迭代器
public ListIterator<E> listIterator() {
    return list.listIterator(); // Must be manually synched by user
}

// 获取指定索引的迭代器
public ListIterator<E> listIterator(int index) {
    return list.listIterator(index); // Must be manually synched by user
}
```

##### 扩容方法

所有方法都是获取mutex对象锁后，调用引用List的方法，所以无独立实现的扩容机制。

##### 增加元素方法

- **add（E）**：父类方法，添加元素到末尾位置，只做了mutex对象锁的获取，其实现交由引用的List集合。
- **add（int，E）**：添加元素到指定索引的位置，只做了mutex对象锁的获取，其实现交由引用的List集合。

```java
// 父类方法，添加元素到末尾位置
public boolean add(E e) {
    synchronized (mutex) {return c.add(e);}
}

// 添加元素到指定索引的位置
public void add(int index, E element) {
    synchronized (mutex) {list.add(index, element);}
}
```

##### 获取元素方法

- get（int）：获取指定索引的元素，获取方法也需要加mutex锁，其实现同样交由List引用实现。

```java
// 获取指定索引的元素
public E get(int index) {
    synchronized (mutex) {return list.get(index);}
}
```

##### 删除元素方法

- **remove（Object）**：父类方法，删除指定元素，只做了mutex对象锁的获取，其实现交由引用的List集合。
- **remove（int）**：删除指定位置的元素，只做了mutex对象锁的获取，其实现交由引用的List集合。

```java
// 父类方法，删除指定元素
public boolean remove(Object o) {
    synchronized (mutex) {return c.remove(o);}
}

// 删除指定位置的元素
public E remove(int index) {
    synchronized (mutex) {return list.remove(index);}
}
```

##### 替换元素方法

- **set（int，E）**：替换指定位置元素，只做了mutex对象锁的获取，其实现交由引用的List集合。

```java
// 替换指定位置元素
public E set(int index, E element) {
    synchronized (mutex) {return list.set(index, element);}
}
```

#### Vector

![1621505440028](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621505440028.png)

##### 特点

- {@code Vector}类实现对象的**可增长数组**。 像数组一样，它包含可以使用整数索引访问的组件。 但是，{@ code Vector}的大小可以根据需要增大或缩小，以适应在创建{@code Vector}之后添加和删除项目。
- 每个Vector都尝试通过维持{@code Capacity}和{@code CapacityIncrement}来优化存储管理。 Capacity始终至少与Vector大小一样大； 它通常较大，因为随着向Vector中添加分量，Vector的存储量**将以{@code CapacityIncrement}的大小逐块增加**。 应用程序可以在插入大量组件之前增加Vector的容量。 这减少了增量重新分配的数量。
- 与新的集合实现不同，**{@ code Vector}是同步的**。 如果不需要线程安全的实现，建议使用{@link ArrayList}代替{@code Vector}。
- 迭代器是快速失败的。

##### 构造方法

- **空参的构造函数**：默认容量为10，默认容量增速为0，即扩容时增长2倍。
- **指定容量的构造函数**：默认容量增速为0，即扩容时增长2倍。
- **指定容量以及容量增速的构造函数**。
- **指定复制集合的构造函数**：默认容量增速为0，即扩容时增长2倍。

```java
public class Vector<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
	protected Object[] elementData;
	protected int elementCount;
	protected int capacityIncrement;
	
	// 空参的构造函数，默认容量为10，默认容量增速为0，即扩容时增长2倍
	public Vector() {
        this(10);
    }

	// 指定容量的构造函数，默认容量增速为0，即扩容时增长2倍
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }
    
    // 指定容量以及容量增速的构造函数
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }
    
    // 指定复制集合的构造函数，默认容量增速为0，即扩容时增长2倍
    public Vector(Collection<? extends E> c) {
        elementData = c.toArray();
        elementCount = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
    }
}
```

##### 迭代方法

与Arraylist的相同，所有迭代操作都有：**并发修改检查**，检测对象有没有发生并发修改的操作。

- **iterator（）**：获取Collection迭代器。
- **listIterator（）**：获取默认的List迭代器。
- **listIterator（int）**：获取指定索引的List迭代器。

##### 扩容方法

- **扩容条件**：数组实际长度（size）大于当前数组长度（elementData.length）时。

- **grow（int）**：增长元素数组到指定容量。

1. 计算新容量： = 旧容量（当前元素数组实际长度） *  2。
2. 容量判断：如果新容量小于指定容量，则设置为指定容量。以及最大长度判断：Integer.MAX_VALUE - 8。
3. 移动元素到新数组：Arrays.copyOf（...）（底层调用的是System.arraycopy（...））。

```java
// 扩容条件
private void ensureCapacityHelper(int minCapacity) {
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

// 扩容方法
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

##### 增加元素方法

- **add（E）**：添加元素到末尾位置。方法开始前需要获取synchronized可重入锁，所以是线程安全的。

1. 更新模数（修改次数）、扩容判断、插入元素到数组末尾、更新实际大小、返回true。

- **add（int，E）**：添加元素到指定位置。无返回值。方法开始前需要获取synchronized可重入锁，所以是线程安全的。

1. 更新模数（修改次数）、索引指针校验、扩容判断、System.arraycopy（...）复制移动数组、插入元素到指定位置、更新实际大小。

- **addElement（E）**：独立实现的方法，添加元素到末尾位置。方法开始前需要获取synchronized可重入锁，所以是线程安全的。

1. 更新模数（修改次数）、扩容判断、插入元素到数组末尾、更新实际大小、返回true。

```java
// 添加元素到末尾位置
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}

// 添加元素到指定位置
public void add(int index, E element) {
    insertElementAt(element, index);
}
public synchronized void insertElementAt(E obj, int index) {
    modCount++;
    if (index > elementCount) {
        throw new ArrayIndexOutOfBoundsException(index
                                                 + " > " + elementCount);
    }
    ensureCapacityHelper(elementCount + 1);
    System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
    elementData[index] = obj;
    elementCount++;
}

// 独立实现的方法，添加元素到末尾位置
public synchronized void addElement(E obj) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = obj;
}
```

##### 获取元素方法

- **get（int）**：获取指定索引位置的元素O（1）。方法开始前需要获取synchronized可重入锁，所以是线程安全的。

```java
// 获取指定索引位置的元素
public synchronized E get(int index) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    return elementData(index);
}
```

##### 删除元素方法

- **remove（int）**：删除指定索引元素。方法开始前需要获取synchronized可重入锁，所以是线程安全的。

1. 索引越界校验。
2. 根据索引获取数组元素，O(1)。
3. 移动数组元素：System.arraycopy（...）。
4. 清空元素、实际大小 - 1。
5. 返回旧元素。

- **remove（Object）**：删除指定元素，用equals（...）判断。方法开始前需要获取synchronized可重入锁，所以是线程安全的。

1. 元素为null：遍历数组、找到第一个为null的元素、移动数组元素、清空元素、实际大小 - 1、返回true。
2. 元素非null：遍历数组、找到第一个equal的元素、移动数组元素、清空元素、实际大小 - 1、返回true。
3. 否则代表删除失败，返回false。

- **removeRange（int，int）**：删除fromIndex~toIndex-1内的元素。无返回值。方法开始前需要获取synchronized可重入锁，所以是线程安全的。

1. 移动数组元素、清空元素、更新实际大小 。

```java
// 删除指定索引元素
public synchronized E remove(int index) {
    modCount++;
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);
    E oldValue = elementData(index);

    int numMoved = elementCount - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--elementCount] = null; // Let gc do its work

    return oldValue;
}

// 删除指定元素，用equals（...）判断
public boolean remove(Object o) {
    return removeElement(o);
}
public synchronized boolean removeElement(Object obj) {
    modCount++;
    int i = indexOf(obj);
    if (i >= 0) {
        removeElementAt(i);
        return true;
    }
    return false;
}
public synchronized void removeElementAt(int index) {
    modCount++;
    if (index >= elementCount) {
        throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                 elementCount);
    }
    else if (index < 0) {
        throw new ArrayIndexOutOfBoundsException(index);
    }
    int j = elementCount - index - 1;
    if (j > 0) {
        System.arraycopy(elementData, index + 1, elementData, index, j);
    }
    elementCount--;
    elementData[elementCount] = null; /* to let gc do its work */
}

// 删除fromIndex~toIndex-1内的元素
protected synchronized void removeRange(int fromIndex, int toIndex) {
    modCount++;
    int numMoved = elementCount - toIndex;
    System.arraycopy(elementData, toIndex, elementData, fromIndex,
                     numMoved);

    // Let gc do its work
    int newElementCount = elementCount - (toIndex-fromIndex);
    while (elementCount != newElementCount)
        elementData[--elementCount] = null;
}
```

##### 替换元素方法

- **set（int，E）**：替换指定位置的元素。方法开始前需要获取synchronized可重入锁，所以是线程安全的。

1. 索引越界检查、根据索引获取数组元素O(1)、设置数组元素O(1)、返回旧元素。

- **setElementAt（E，int）**：独立实现的方法，替换指定位置的元素。无返回值。方法开始前需要获取synchronized可重入锁，所以是线程安全的。

```java
// 替换指定位置的元素
public synchronized E set(int index, E element) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}

// 独立实现的方法，替换指定位置的元素
public synchronized void setElementAt(E obj, int index) {
    if (index >= elementCount) {
        throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                 elementCount);
    }
    elementData[index] = obj;
}
```

#### Stack

![1621509325202](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621509325202.png)

##### 特点

- **Stack类表示对象的后进先出（LIFO）堆栈**。 它通过五个操作扩展了Vector类，这些操作允许将Vector视为堆栈。 提供了通常的push和pop方法，一种查看堆栈顶部元素的peek方法，一种判断堆栈是否为空的empty方法，和一种查找指定元素距离堆栈底部高度的search方法。 
- 首次创建堆栈时，不做任何实现。添加元素时，从顶部开始添加。

##### 构造方法

- **只有无参构造方法**：该方法没有任何实现。

```java
public class Stack<E> extends Vector<E> {
    public Stack() {
    }
}
```

##### push方法

- **push（E）**：添加元素到栈顶，交由父类Vector#addElement（E）实现。父类方法开始前需要获取synchronized可重入锁，所以是线程安全的。

1. 更新模数（修改次数）、扩容判断、插入元素到数组末尾、更新实际大小、返回true。

```java
// 添加元素到栈顶
public E push(E item) {
    addElement(item);
    return item;
}
// Vector#addElement（E）
public synchronized void addElement(E obj) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = obj;
}
```

##### pop方法

- **pop（）**：弹出栈顶元素，交由父类Vector#removeElementAt（int）实现。父类方法开始前需要获取synchronized可重入锁，所以是线程安全的。

1. 备份栈顶元素、移动数组元素、清空元素、实际大小 - 1、返回旧元素。

```java
public synchronized E pop() {
    E obj;
    int len = size();
    obj = peek();
    removeElementAt(len - 1);
    return obj;
}
// Vector#removeElementAt（int）
public synchronized void removeElementAt(int index) {
    modCount++;
    if (index >= elementCount) {
        throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                 elementCount);
    }
    else if (index < 0) {
        throw new ArrayIndexOutOfBoundsException(index);
    }
    int j = elementCount - index - 1;
    if (j > 0) {
        System.arraycopy(elementData, index + 1, elementData, index, j);
    }
    elementCount--;
    elementData[elementCount] = null; /* to let gc do its work */
}
```

##### peek方法

- **peek（）**：查看栈顶元素O（1），交由父类Vector#elementData（int）实现，O（1）。父类方法开始前需要获取synchronized可重入锁，所以是线程安全的。

```java
// 查看栈顶元素O（1）
public synchronized E peek() {
    int len = size();

    if (len == 0)
        throw new EmptyStackException();
    return elementAt(len - 1);
}
// Vector#elementAt（int）
public synchronized E elementAt(int index) {
    if (index >= elementCount) {
        throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
    }

    return elementData(index);
}
// Vector#elementData（int），O（1）
E elementData(int index) {
    return (E) elementData[index];
}
```

##### empty方法

- **empty（）**：判断栈是否为空，即实际元素个数是否为0。获取实际元素个数方法交由父类Vector#size（），其方法开始前需要获取synchronized可重入锁，所以是线程安全的。

```java
// 判断栈是否为空，即实际元素个数是否为0
public boolean empty() {
    return size() == 0;
}
// Vector#size（）
public synchronized int size() {
    return elementCount;
}
```

##### search方法

- **search（）**：查找元素距离堆栈底部的高度，交由父类Vector#lastIndexOf（Object，int）实现。父类方法开始前需要获取synchronized可重入锁，所以是线程安全的。

```java
// 查找元素距离堆栈底部的高度
public synchronized int search(Object o) {
    int i = lastIndexOf(o);

    if (i >= 0) {
        return size() - i;
    }
    return -1;
}
// Vector#lastIndexOf（Object）
public synchronized int lastIndexOf(Object o) {
    return lastIndexOf(o, elementCount-1);
}
// Vector#lastIndexOf（Object，int）
public synchronized int lastIndexOf(Object o, int index) {
    if (index >= elementCount)
        throw new IndexOutOfBoundsException(index + " >= "+ elementCount);

    if (o == null) {
        for (int i = index; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = index; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

