## 源码篇

### Set

Set，不包含重复元素的集合，最多包含一个空元素。

```java
// 其接口方法都是Collection接口的方法
public interface Set<E> extends Collection<E> {
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    <T> T[] toArray(T[] a);
    boolean add(E e);
    boolean remove(Object o);
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean retainAll(Collection<?> c);
    boolean removeAll(Collection<?> c);
    void clear();
    boolean equals(Object o);
    int hashCode();
    default Spliterator<E> spliterator() {...}
}
```

- **典型实现**：HashSet、LinkedHashSet、SynchronizedSet

#### HashSet

![1621512944158](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621512944158.png)

##### 特点

- 实现Set接口，该接口由哈希表（HashMap实例）支持， **不保证集合的迭代顺序**，允许使用null元素。
- add，remove，contain和size方法时间性能恒定。但迭代时间与元素数量和哈希桶容量有关，最坏为O（m * n）（m为桶长度，n为桶中链表长度）。
- 非线程安全的，如果要使用同步的Set方法，可以考虑使用Collections＃synchronizedSet进行包装。
- 迭代器是快速失败的。
- **Set集合没有获取和替换元素的方法。**

##### 数据结构

![1625390302606](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625390302606.png)

##### 构造方法

- **无参的构造函数**：使用默认初始化容量16，默认负载因子0.75。
- **指定复制集合的构造函数**：根据集合大小计算初始容量（size / 0.75） 或者为默认值16。
- **指定初始容量的构造函数**：默认负载因子0.75。
- **指定初始容量和负载因子的构造函数**：该构造方法是defalut级别的，供LinkHashSet使用，同时底层数据结构改用LinkedHashMap存储，多维护了一个双向链表，使得迭代顺序是有序的。

```java
public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable
{
	private transient HashMap<E,Object> map;
	private static final Object PRESENT = new Object();// 用作HashMap中元素的假值
	
	// 无参的构造函数：使用默认初始化容量16，默认负载因子0.75
    public HashSet() {
        map = new HashMap<>();
    }
    
    // 指定复制集合的构造函数：根据集合大小计算初始容量（size / 0.75） 或者为默认值16
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }
    
    // 指定初始容量的构造函数：默认负载因子0.75
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }
    
    // 指定初始容量和负载因子的构造函数：该构造方法是defalut级别的，供LinkHashSet使用，同时底层数据结构改用LinkedHashMap存储，多维护了一个双向链表，使得迭代顺序是有序的
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        // dummy参数留给子类LinkedHashSet使用，注意这里构造的是LinkedHashMap，不是HashMap
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
}
```

##### 迭代方法

- **iterator（）**：获取迭代器，交由成员变量引用HashMap#HashIterator实现，用于遍历HashMap的Node结点的key。

```java
// 获取迭代器
public Iterator<E> iterator() {
    return map.keySet().iterator();
}
// HashMap#KeySet
final class KeySet extends AbstractSet<K> {
    public final Iterator<K> iterator()     { return new KeyIterator(); }
    ...
}
// HashMap#KeyIterator
final class KeyIterator extends HashIteratorimplements Iterator<K> {
    public final K next() { 
        return nextNode().key; 
    }
}
// HashMap#HashIterator
abstract class HashIterator {
    Node<K,V> next;        // next entry to return
    Node<K,V> current;     // current entry
    int expectedModCount;  // for fast-fail
    int index;             // current slot
    
   	...
        
    final Node<K,V> nextNode() {
        Node<K,V>[] t;
        Node<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        if ((next = (current = e).next) == null && (t = table) != null) {
            do {} while (index < t.length && (next = t[index++]) == null);
        }
        return e;
    }
}
```

##### 增加元素方法

- add（E）：添加元素到集合中，交由成员变量引用HashMap#put（K，V）实现。以元素的值为HashMap.Entry的键，以Object PRESENT = new Object();空对象作为HashMap.Entry的值。

```java
private static final Object PRESENT = new Object();// 用作HashMap中元素的假值

// 添加元素到集合中
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
} 
/**
 * HashMap#put（K，V）
 *
 * hash: 指key的hash值（hashCode扰动后的结果）
 * key：指key的值
 * value：指value的值
 * onlyIfAbsent：为true时代表不能替换旧值，为false时则可以替换旧值
 * evict：为true时代表散列表处于非创建模式中，为false时则散列表处于创建模式中（用于LinkedHashMap 
 *        结点插入后的回调访问函数，对于HashMap本身没用）
 */
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

##### 删除元素方法

- remove（Object）：删除集合中指定的元素，交由成员变量引用HashMap#remove（Object），同一元素的判断条件为：key的hash值相等 或者 Key值equal。

```java
// 删除集合中指定的元素
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}

/**
 * HashMap#remove（Object）：key的hash值相等 或者 Key值equal
 *
 * hash：key的hash值 
 * key：key值
 * value：如果matchValue为true则匹配的value值，否则忽略value值。
 * matchValue：为true代表则仅在值相等时才删除，为false代表值不相等也可以删除
 * movable：为true代表删除时可以移动其他结点，为false代表在删除时不能移动其他结点
 *         （只有迭代器的删除方法movable才会为false）
 **/
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}
```

#### LinkedHashSet

![1621519230206](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621519230206.png)

##### 特点

- Set接口的**哈希表和链表**实现，具有可预测的迭代顺序。 此实现与HashSet的不同之处在于，它维护在其所有条目中运行的双向链接列表。 此链表定义了迭代顺序，即**将元素插入到集合中的顺序（插入顺序）**。 请注意，如果将元素重新插入到集合中，则插入顺序不会受到影响（如在调用之前s.contains（e）将返回true的情况下调用s.add（e），则将元素e重新插入到set s中）。
- 无论原始集的实现如何，都可以使用它来产生与原始集具有相同顺序的集合副本，如Set copy = new LinkedHashSet(Set)。
- 与HashSet一样，提供add，contains和remove基本操作，由哈希表（HashMap实例）支持，由于还维护了一个双向链表，所以性能可能会略低于HashSet。但迭代时间最坏为O（n），而HashSet的为最坏为O（m * n）（m为桶长度，n为桶中链表长度）。
- 非线程安全的，如果要使用同步的Set方法，可以考虑使用Collections＃synchronizedSet进行包装。
- 迭代器是快速失败的。
- **其迭代方法、增加与删除元素方法交由HashSet实现，没有获取和替换元素的方法。**

##### 构造方法

所有构造方法交由父类HashSet#HashSet(int, float, boolean)实现，底层数据结构改用LinkedHashMap存储，多维护了一个双向链表，使得迭代顺序是有序的。

- **无参的构造函数**：使用默认初始化容量16，默认负载因子0.75。
- **指定复制集合的构造函数**：根据集合大小计算初始容量（size / 0.75） 或者为默认值16。
- **指定初始容量的构造函数**：默认负载因子0.75。
- **指定初始容量和负载因子的构造函数**。

```java
public class LinkedHashSet<E> extends HashSet<E> implements Set<E>, Cloneable, java.io.Serializable {
    
    // 无参的构造函数
    public LinkedHashSet() {
        super(16, .75f, true);// true代表HashMap中的Key值使用假值
    }
    
    // 指定复制集合的构造函数
    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }

    // 指定初始容量的构造函数
    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }

    // 指定初始容量和负载因子的构造函数
    public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }
}

public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable
{
    // 指定初始容量和负载因子的构造函数：该构造方法是defalut级别的，供LinkHashSet使用，同时底层数据结构改用LinkedHashMap存储，多维护了一个双向链表，使得迭代顺序是有序的
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        // dummy参数留给子类LinkedHashSet使用，注意这里构造的是LinkedHashMap，不是HashMap
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
}
```

#### SynchronizedSet

![1621521638282](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1621521638282.png)

##### 特点

- java.util.Collections内部类

- 继承自SynchronizedCollection类，相当于拥有成员变量Collection引入和mutex对象锁，用于包装传入的集合，使（**其所有方法都能成为同步方法加synchronized关键字**），其元素操作方法与SynchronizedCollection类的一致，也就是**Set集合没有获取和替换元素的方法**。
- 迭代器是否具有快速失败的特性，取决于传入的Set引用的实际类型（**Set集合的都是快速失败的**）。

##### 构造方法

- **只传入Set集合的构造函数**：设置Collection引用，持有父类mutex对象锁。
- **传入Set集合与对象锁的构造函数**：设置Colleciton引用与对象锁引用。

```java
static class SynchronizedSet<E> extends SynchronizedCollection<E> implements Set<E> {
    private static final long serialVersionUID = 487447009682186044L;

    // 只传入Set集合的构造函数
    SynchronizedSet(Set<E> s) {
        super(s);
    }
    
    // 传入Set集合与对象锁的构造函数
    SynchronizedSet(Set<E> s, Object mutex) {
        super(s, mutex);
    }
    
    // 只重写了equals和hashCode方法: 加上synchronized关键字
    ...
}
```

