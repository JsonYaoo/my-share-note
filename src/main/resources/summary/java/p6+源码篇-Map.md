## 源码篇

### Map

- Map，是将**键映射到值**的对象，映射不能包含重复的键， 每个键最多可以映射到一个值。

```java
public interface Map<K,V> {
    int size();
    boolean isEmpty();
    boolean containsKey(Object key);
    boolean containsValue(Object value);
    V get(Object key);
    V put(K key, V value);
    V remove(Object key);
    void putAll(Map<? extends K, ? extends V> m);
    void clear();
    Set<K> keySet();
    Collection<V> values();
    Set<Map.Entry<K, V>> entrySet();
    boolean equals(Object o);
    int hashCode();
    default V putIfAbsent(K key, V value) {...}
    default boolean remove(Object key, Object value) {...}
    
    // 映射项(键值对)，Map.entrySet返回映射的集合视图，其元素属于此类。获取的唯一方法是从此集合视图的迭代器获取。
    interface Entry<K,V> {
        K getKey();
        V getValue();
        V setValue(V value);
        boolean equals(Object o);
        int hashCode();
        
        public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey(){...}
        public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {...}
        public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {...}
        public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {...}
    }
}
```

- 典型实现：HashMap、LinkedHashMap、TreeMap、WeakHashMap、SynchronizedMap、HashTable、ConcurrentHashMap、ConcurrentSkipListMap。

#### SortedMap接口

- **特点**：
  - SortedMap接口，**提供键的排序，默认按照键值进行自然排序**，也可以在创建时指定Comparator进行排序（Comparable用于内部指定排序规则，Comparator用于外部指定排序规则），因此**插入的所有键都必须实现{@code Comparable}接口或者被指定的比较器接受**。同时还提供了几个额外的操作来返回具有受限键范围的子Map。
  - 要注意的是，**如果SortedMap实现类要正确实现Map接口，则要保证维护的排序必须与equals一致**，这是因为Map接口是根据equals来定义的，而SortedMap是使用Comparable#compareTo方法、Comparator#compare方法来执行所有键的比较，因此需要保证比较结果为0时的两个对象也equals。
  - 所有SortedMap接口实现类都应该提供**四个标准的构造函数**（接口规定不了构造函数，只是建议这么实现）：
    - **无参构造函数**：创建一个空的排序映射，根据其键的**自然顺序**排序。
    - **指定{@code Comparator} 的构造函数**：创建一个空的排序映射，**根据指定的比较器排序**。
    - **指定复制集合的构造函数**：创建一个新映射，其键值映射与其参数相同，根据键的**自然顺序**排序。
    - **指定 {@code SortedMap} 复制集合的构造函数**：创建一个新的排序映射，其**键值映射和排序与输入排序映射相同**。

```java
public interface SortedMap<K,V> extends Map<K,V> {
    // 返回键排序的比较器
    Comparator<? super K> comparator();
    
    // 返回Map的中间视图：即键从fromKey（含）到toKey（不含）的部分视图，两者相等时返回null
    SortedMap<K,V> subMap(K fromKey, K toKey);
    
    // 返回Map的头部视图：即键小于toKey(不含)的部分视图
    SortedMap<K,V> headMap(K toKey);
    
    // 返回Map的尾部视图：即键大于或者等于fromKey（含）的部分视图
    SortedMap<K,V> tailMap(K fromKey);
    
    // 返回Map中的第一个键（最小）
    K firstKey();
    
    // 返回Map中的最后一个键（最大）
    K lastKey();
    
    // 返回键的升序视图
    Set<K> keySet();
    
    // 返回值的升序视图
    Collection<V> values();
    
    // 返回Entry条目的升序视图
    Set<Map.Entry<K, V>> entrySet();
}
```

- **典型实现**：
  - TreeMap（java.util）
  - ConcurrentSkipListMap（java.util.concurrent）

#### NavigableMap接口

- **特点**：
  - NavigableMap，扩展了SortedMap以提供导航的方法：**返回最接近给定搜索目标的匹配项**。下面这些方法都是为了**定位**而不是遍历条目而设计的。
    - **小于key的最大键**：lowerEntry & lowerKey。
    - **小于等于key的最大键**：floorEntry & floorKey。
    - **大于或者等于key的最小键**：ceilingEntry & ceilingKey。
    - **大于key的最小键**：higherEntry & higherKey。
  - NavigableMap，同时提供升序或者降序遍历，{@code DescingMap} 方法返回降序视图，但**升序操作可能会比降序操作快**。
  - 还定义了指定了以下方法：
    - **返回指定上下限（包含 | 不包含）的子视图**：{@code subMap}、{@code headMap} 和 {@code tailMap}方法。
    - **返回最小或者最大键值的条目**：{@code firstEntry}和{@code lastEntry} 。
    - **删除并返回最小或者最大键值的条目**：{@code pollFirstEntry}和 {@code pollLastEntry}。
  - NavigableMap接口的方法，返回的是 **{@code Map.Entry} 生成时的映射快照**，因此不支持可选的 {@code Entry#setValue} 方法，但{@code Map#put} 方法可以更改其映射值。

```java
public interface NavigableMap<K,V> extends SortedMap<K,V> {
    // 返回小于Key的最大键的Entry，如果没有这样的键，则返回 {@code null}
    Map.Entry<K,V> lowerEntry(K key);
    // 返回小于Key的最大键，如果没有这样的键，则返回 {@code null}
    K lowerKey(K key);
    
    // 返回小于或者等于Key的最大键的Entry，如果没有这样的键，则返回 {@code null}
    Map.Entry<K,V> floorEntry(K key);
    // 返回小于或者等于Key的最大键，如果没有这样的键，则返回 {@code null}
    K floorKey(K key);
    
    // 返回大于或者等于Key的最小键的Entry，如果没有这样的键，则返回 {@code null}
    Map.Entry<K,V> ceilingEntry(K key);
    // 返回大于或者等于Key的最小键，如果没有这样的键，则返回 {@code null}
    K ceilingKey(K key);
    
    // 返回大于Key的最小键的Entry，如果没有这样的键，则返回 {@code null}
    Map.Entry<K,V> higherEntry(K key);
    // 返回大于Key的最小键，如果没有这样的键，则返回 {@code null}
    K higherKey(K key);
    
    // 返回最小键的Entry，如果条目为空，则返回 {@code null}
    Map.Entry<K,V> firstEntry();
    
    // 返回最大键的Entry，如果条目为空，则返回 {@code null}
    Map.Entry<K,V> lastEntry();
    
    // 删除并返回最小键的Entry，如果条目为空，则返回 {@code null}
    Map.Entry<K,V> pollFirstEntry();
    
    // 删除并返回最大键的Entry，如果条目为空，则返回 {@code null}
    Map.Entry<K,V> pollLastEntry();
    
    // 返回Map的逆序（降序）视图
    NavigableMap<K,V> descendingMap();
    
    // 返回键的升序{@link NavigableSet}视图
    NavigableSet<K> navigableKeySet();
    
    // 返回Map的键的逆序{@link NavigableSet}视图 
    NavigableSet<K> descendingKeySet();
    
 	// 返回Map的中间视图：键从fromKey（不含）到toKey（不含）的部分视图，fromInclusive为true代表包含fromKey，toInclusive为true代表包含toKey
    NavigableMap<K,V> subMap(K fromKey, boolean fromInclusive,
                             K toKey,   boolean toInclusive);
    
    // 返回Map的头部视图：即键小于toKey(不含)的部分视图，inclusive为true代表包含toKey
    NavigableMap<K,V> headMap(K toKey, boolean inclusive);
    
    // 返回Map的尾部视图：即键大于fromKey（不含）的部分视图，inclusive为true代表包含fromKey
    NavigableMap<K,V> tailMap(K fromKey, boolean inclusive);
    
    // 返回Map的中间视图：即键从fromKey（fromInclusive含）到toKey（!toInclusive不含）的部分视图，两者相等时返回null(除非fromInclusive与toInclusive都为true)
    SortedMap<K,V> subMap(K fromKey, K toKey);
    
    // 返回Map的头部视图：即键小于toKey(!inclusive不含)的部分视图
    SortedMap<K,V> headMap(K toKey);
    
    // 返回Map的尾部视图：即键大于或者等于fromKey（inclusive含）的部分视图
    SortedMap<K,V> tailMap(K fromKey);
}
```

- **典型实现**：
  - TreeMap（java.util）
  - ConcurrentSkipListMap（java.util.concurrent）

#### ConcurrentMap接口

- **特点**：
  - ConcurrentMap接口，只是一个“标记”接口，“标记”实现该接口的类都是线程安全的。
  - 所有方法都是继承Map接口的，由实现类实现，以保证**线程安全与原子性**。

```java
public interface ConcurrentMap<K, V> extends Map<K, V> {
    
    default V getOrDefault(Object key, V defaultValue) {...}
    default void forEach(BiConsumer<? super K, ? super V> action) {...}
    V putIfAbsent(K key, V value);
    boolean remove(Object key, Object value);
    boolean replace(K key, V oldValue, V newValue);
    V replace(K key, V value);
    
    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function {
        ...
    }
                            
    default V computeIfAbsent(K key, Function<? super K, ? extends V> 	
                              mappingFunction) {
        ...
    }
    default V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> 
                               remappingFunction) {
		...
    }
                            
    default V compute(K key, BiFunction<? super K, ? super V, ? extends V> 
                      remappingFunction) {
        ...
    }
                            
    default V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> 
                     remappingFunction) {
        ...
    }
}
```

- **典型实现**：
  - ConcurrentHashMap（java.util.concurrent）
  - ConcurrentSkipListMap（java.util.concurrent）

#### ConcurrentNavigableMap接口

- **特点**：
  - ConcurrentNavigableMap接口，支持 {@link NavigableMap} 操作，并且可递归地用于其navigable子视图。

```java
public interface ConcurrentNavigableMap<K,V> extends ConcurrentMap<K,V>, NavigableMap<K,V>
{
	// 返回Map的中间视图：键从fromKey（不含）到toKey（不含）的部分视图，fromInclusive为true代表包含fromKey，toInclusive为true代表包含toKey
    ConcurrentNavigableMap<K,V> subMap(K fromKey, boolean fromInclusive,
                                       K toKey,   boolean toInclusive);
    
    // 返回Map的头部视图：即键小于toKey(不含)的部分视图，inclusive为true代表包含toKey
    ConcurrentNavigableMap<K,V> headMap(K toKey, boolean inclusive);
    
    // 返回Map的尾部视图：即键大于fromKey（不含）的部分视图，inclusive为true代表包含fromKey
    ConcurrentNavigableMap<K,V> tailMap(K fromKey, boolean inclusive);
    
    // 返回Map的中间视图：即键从fromKey（fromInclusive含）到toKey（!toInclusive不含）的部分视图，两者相等时返回null(除非fromInclusive与toInclusive都为true)
    ConcurrentNavigableMap<K,V> subMap(K fromKey, K toKey);
    
    // 返回Map的头部视图：即键小于toKey(!inclusive不含)的部分视图
    ConcurrentNavigableMap<K,V> headMap(K toKey);
    
    // 返回Map的尾部视图：即键大于或者等于fromKey（inclusive含）的部分视图
    ConcurrentNavigableMap<K,V> tailMap(K fromKey);
    
    // 返回Map的逆序（降序）视图
    ConcurrentNavigableMap<K,V> descendingMap();
    
    // 返回键的升序{@link NavigableSet}视图
    public NavigableSet<K> navigableKeySet();
    
    // SortedMap接口，返回键的升序视图
    NavigableSet<K> keySet();
    
    // 返回Map的键的逆序{@link NavigableSet}视图 
    public NavigableSet<K> descendingKeySet();
}
```

- **典型实现**：
  - ConcurrentSkipListMap（java.util.concurrent）

#### HashMap

![1622377026032](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1622377026032.png)

##### 特点

- HashMap，Map接口基于哈希表的实现，**允许空值（null）和空键（null），不保证映射的顺序**。
- 假设散列函数在存储桶中正确分散元素，get（）和put（E）方法的时间复杂度为O（1）。而其迭代所需的时间与HashMap实例的**容量（散列表桶数）**和**实际大小（键值对数）**成正比，因此，如果迭代性能重要，则不要设置过高的初始容量或者过低的负载因子（会导致大容量）。
- 两个影响性能的参数：
  - **初始容量**：当前容量是散列表中存储桶的数量，而初始容量只是创建散列表的容量。
  - **负载因子**：指HashMap自动增加散列表容量之前，允许散列表获得满意的度量。等于**实际大小 / 当前容量**，当散列表中的条目数超过负载因子和当前容量的乘积时，会重建内部数据结构，使散列表具有大约两倍的桶数。
- 默认提供的**负载因子0.75**，在时间和空间成本之间提供了很好的权衡，较高会减少空间的开销，但增加了查找的成本（由于高负载因子，导致扩容次数减少，桶拉链变长），较低会增加扩容的次数，增加空间的开销，但好在桶拉链变短，查找效率高，哈希冲突少。注意点有：
  - 在设置初始容量时，应考虑映射中**预期的条目数要及其负载因子**，以尽量减少重建散列表的次数，因为如果初始容量大于 考虑到的最大条目数/负载因子，则不会发生重建散列表的操作。
  - 如果要在一个HashMap实例中存储许多映射，应该创建时指定足够大的容量，而不是让它根据需要时，才执行重建散列表操作。
- **HashMap是非同步的**，在多个线程并发访问时，通常是通过同步一些自然封装映射的对象来完成，比如Collections.synchronizedMap。
- **HashMap的所有迭代器都是快速失败的**，即如果在创建迭代器后的任何时间对结构进行结构修改，则除了通过迭代器自己的remove方法之外，该迭代器都将抛出{ @link ConcurrentModificationException}，这在面对并发修改时，迭代器可以快速干净地失败，而不是在未来不确定的时间冒着任意、非确定性行为的风险去修改。但是**无法保证迭代器的快速失败行为**，因为一般而言，在存在非同步并发修改的情况下，不可能做出任何硬保证，**只会尽最大努力抛出**ConcurrentModificationException，因此，编写一个依赖于这个异常来保证其正确性的程序是错误的，**快速失败行为只适用于检测错误**。

##### 数据结构

![1625373206969](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625373206969.png)

##### 构造方法

- **空参的构造方法**：使用默认的负载因子0.75f，初始容量和阈值在第一次的resize()扩容方法中确定。
- **指定初始容量的构造方法**：使用指定的初始容量16和默认的负载因子0.75f。
- **指定初始容量和负载因子的构造方法**：赋值负载因子和计算2^n作为阈值。
- **指定复制集合的构造方法**：使用默认的负载因子0.75f，使用复制集合元素大小的2^n作为阈值。

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    static final int MAXIMUM_CAPACITY = 1 << 30;
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    static final int TREEIFY_THRESHOLD = 8;
    static final int UNTREEIFY_THRESHOLD = 6;
    static final int MIN_TREEIFY_CAPACITY = 64;
    
    transient Node<K,V>[] table;
    transient Set<Map.Entry<K,V>> entrySet;
    transient int size;
    transient int modCount;
    
    // 阈值：即要下一个容量值=当前容量（散列表实际大小）*负载因子，如果table==空表时，则在初始化散列表时，用作新建表的初始容量
    int threshold;
    final float loadFactor;// 负载因子
    
    // HashMap#Node，实现Map#Entry，以实现Entry#getKey | getValue等方法
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
        
        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        
        ...
    }
    
    // 空参的构造方法：使用默认的负载因子0.75f，初始容量和阈值在第一次的resize()扩容方法中确定
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
    
    // 指定初始容量的构造方法：使用指定的初始容量16和默认的负载因子0.75f
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    
    // 指定初始容量和负载因子的构造方法：赋值负载因子和计算2^n作为阈值
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        this.loadFactor = loadFactor;
        // 返回给定目标容量的2的幂次作为阈值
        this.threshold = tableSizeFor(initialCapacity);
    }
    
    // 指定复制集合的构造方法: 使用默认的负载因子0.75f，使用复制集合元素大小的2^n作为阈值
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        // 添加复制集合的的所有键值对，散列表处于创建模式中（用于LinkedHashMap结点插入后的回调访问函数，对于HashMap本身没用）
        putMapEntries(m, false);
    }
}
```

##### 迭代方法

- **抽象的HashMap.Node迭代器**：HashMap的迭代器基类，快速失败机制，遍历next指针，返回HashMap.Node对象。
- **Map.Entry迭代器**：继承HashMap的迭代器基类，快速失败机制，遍历next指针，返回Map.Entry对象。
- **HashMap.Node#Key迭代器**：继承HashMap的迭代器基类，快速失败机制，遍历next指针，返回HashMap.Node#Key对象。
- **HashMap.Node#Value迭代器**：继承HashMap的迭代器基类，快速失败机制，遍历next指针，返回HashMap.Node#Value对象。

```java
// 抽象的HashMap.Node迭代器：HashMap的迭代器基类，快速失败机制，遍历next指针，返回HashMap.Node对象
abstract class HashIterator {
    Node<K,V> next;        // next entry to return
    Node<K,V> current;     // current entry
    int expectedModCount;  // for fast-fail
    int index;             // current slot
    
    public final boolean hasNext() {
        return next != null;
    }
    
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
    
    ...
}

// Map.Entry迭代器：继承HashMap的迭代器基类，快速失败机制，遍历next指针，返回Map.Entry对象
final class EntryIterator extends HashIterator implements Iterator<Map.Entry<K,V>> {
	public final Map.Entry<K,V> next() { return nextNode(); }
}

// HashMap.Node#Key迭代器：继承HashMap的迭代器基类，快速失败机制，遍历next指针，返回HashMap.Node#Key对象
final class KeyIterator extends HashIterator implements Iterator<K> {
    public final K next() { return nextNode().key; }
}

// HashMap.Node#Value迭代器：继承HashMap的迭代器基类，快速失败机制，遍历next指针，返回HashMap.Node#Value对象
final class ValueIterator extends HashIterator implements Iterator<V> {
    public final V next() { return nextNode().value; }
}
```

##### HashCode扰动函数

- **hash（Object）**：HashCode右移16位，从而**混合HashCode的高位和低位，加大低位的随机性，减少哈希碰撞发生的概率**，是一种性能、效用和质量的折衷方案。
  - 使用简单的位移与异或操作，减少系统的计算损耗；使用高位异或，可以减少低位冲突的可能性，保证查找效率。
  - HashCode右移16位，使得高位能被利用起来，保证了效用性。
  - 使用高位异或，可以减少低位冲突的可能性，保证散列表的质量。

```java
// HashCode扰动函数
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

##### 计算哈希索引方法

- **（n - 1） & hash**：相当于hash % n，n指散列表的当前容量（桶数量），hash指获取hash（key）即key的hashCode扰动后结果。
  - **不能直接使用HashCode作为索引的原因**：int类型的hashCode，范围为[-2^32，2^32  - 1]，如果散列表数组与HashCode一一对应，**需要40亿的空间（int[40亿]），明显这在内存是放不下的**，也就是说明hashCode是不能直接作为数组索引的。因此，如果使用hashCode对散列表数组长度取模，那么就可以解决这个问题，从而保证较小的数组也还能利用上hashCode。
  - **n为2的幂次原因**：为了解决hashCode对散列表数组长度取模，设计了HashCode的扰动函数以及为2幂次的容量n，可以通过n-1来获得取模操作的低位掩码，此时**只需要通过低位掩码与扰动后的hashCode（hash值）进行一次与运算**，即可得到该hash值在散列表数组中的索引。其次通过在扩容方法中，经过hash值与低位掩码相与，可以保证扩容后，**只会移动少部分相与结果高位为1的桶链表**，其他保持不变，减少了扩容时的时间。

```java
if ((p = tab[i = (n - 1) & hash]) == null)
    tab[i] = newNode(hash, key, value, null);
```

##### 规范容量计算方法

- **tableSizeFor（int）**：返回给定目标容量的2的幂次，一是可以计算出规范的容量值，二是可以把规范的容量值作为阈值，以便在扩容时另新容量等于该阈值（也就规范了新容量）。
  - **右移的目的**：是为了取得每位都是1的结果（最高位为当前容量的最高位），从而可以通过右移结果+1后可以获得2的幂次容量。

```java
// 返回给定目标容量的2的幂次 => 根据容量获取指定容量的2^n
static final int tableSizeFor(int cap) {
    // 20201119 eg: cap = 0100 0011 1010 1001
    // 20201119 eg: n = 0100 0011 1010 1000
    int n = cap - 1;

    // 20201119 eg: 0100 0011 1010 1000
    //           |= 0010 0001 1101 0100 => 0110 0011 1111 1100
    n |= n >>> 1;

    // 2020119 eg:  0110 0011 1111 1100
    //           |= 0001 1000 1111 1111 => 0111 1011 1111 1111
    n |= n >>> 2;

    // 2020119 eg : 0111 1011 1111 1111
    //           |= 0000 0111 1011 1111 => 0111 1111 1111 1111
    n |= n >>> 4;

    // 2020119 eg : 0111 1111 1111 1111
    //           |= 0000 0000 0111 1111 => 0111 1111 1111 1111
    n |= n >>> 8;

    // 2020119 eg : 0111 1111 1111 1111
    //           |= 0000 0000 0000 0000 => 0111 1111 1111 1111
    n |= n >>> 16;
    // 20201119 eg: n => 0111 1111 1111 1111,
    // 实际容量再加1    => 1000 0000 0000 0000,
    // 因此右移这么位的目的是:是为了取得每位都是1的结果（最高位为当前容量的最高位），从而可以通过右移结果+1后可以获得2的幂次容量。

    // 根据容量获取指定容量的2^n => 大于最大容量则取最大容量2^30, 否则取n+1
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

##### 扩容方法

- **resize（）**：根据阈值调整散列表大小，更新容量和阈值，以及移动桶链表位置。

1. **容量&阈值判断**：
   - 当前容量 > 0，则重新重新计算新的容量和新的阈值（容量 * 2，阈值 *2）。
   - 当前容量 <=0，且阈值 > 0，说明当前容量 < 当前阈值，此时使用阈值作为新的容量即可。
   - 如果当前容量 <=0，且当前阈值 < 0，说明HashMap还没被初始化，则赋默认容量以及默认阈值。
2. **计算新的阈值**：如果新阈值还为0，说明使用了当前阈值替换了当前容量，此时需要重新计算新的阈值 => 新阈值 = 新容量 * 当前加载因子 = 当前阈值 * 当前加载因子。
3. **创建新容量的散列表**：(Node<K,V>[])new Node[newCap];
4. **转移元素**：
   - **如果原j桶链头不存在下一个结点**：即原来的桶中只有一个元素（即桶头元素）：此时只需要重新计算hash在散列表中的索引并移动元素即可。
   - **如果原j桶链头属于红黑树结点**：则根据新容量拆分当前桶链表, 会有两种结果 newIndex = oldIndex; newIndex = oldIndex + oldCap。
   - **如果原j桶链头不属于红黑树结点**：则使用lo链表来保存hash在数组中索引不变的元素，hi链表存放索引变了的元素，以保证顺序地移动元素。
5. 最后返回新容量的散列表：元素顺序不变（因为是顺序移动）。

```java
// 根据阈值调整散列表大小
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;// 获取当前Map的阈值(作为下一个目标容量) 
    int newCap, newThr = 0;// 初始化新容量和新阈值

    // 如果当前容量>0 => 重新计算新的容量和新的阈值
    if (oldCap > 0) {
        // 如果当前容量大于最大容量
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }

        // 否则, 如果将当前容量*2, 这时小于最大容量2^30 & 大于初始容量16，则阈值*2
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY) 
            newThr = oldThr << 1; // double threshold
    }
    
    // 如果当前容量<=0，且当前阈值>0，说明当前容量 < 当前阈值，此时使用阈值作为新的容量即可
    else if (oldThr > 0) // initial capacity was placed in threshold 
        newCap = oldThr;
    
    // 如果当前容量<=0，且当前阈值<0，说明HashMap还没被初始化，则赋默认容量以及默认阈值
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);//0.75*16=12
    }

    // 如果新阈值为0，说明使用了当前阈值替换了当前容量此时需要重新计算新的阈值
    if (newThr == 0) {
        // 新阈值 = 新容量 * 当前加载因子 = 当前阈值 * 当前加载因子
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;

    // 创建新容量散列表(Node数组, 每个元素作为链表头)
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;

    // 如果旧散列表（当前散列表）不为空 => 开始转移数据
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;

            // 如果j桶不为null, 同时备份桶链表到e中
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;// 清空j桶

                // 如果原j桶链头不存在下一个结点, 即原来的桶中只有一个元素（即桶头元素）
                if (e.next == null)
                    // e.hash & (newCap - 1) 相当于 e.hash % newCap, 其中后者最后需要转换为前者进行计算, 所以前者效率更高。巧妙之处在于：扩容前选择桶的位置 oldIndex: e.hash & (oldCap - 1)，扩容后选择桶的位置 newIndex: e.hash & (newCap - 1)，经过例子计算, 由于oldCap、newCap都是2^n, 且newCap比oldCap高1位, 那么oldCap - 1 与 newCap - 1两者实际就是差了最高位的1，这时与原来的hash相与只会有两种结果:1）原来的hash对应newCap-1的最高位为0, 那么扩容后桶的位置newIndex还是不变；2）原来的hash对应newCap-1的最高位为1, 那么扩容后痛的位置newIndex比oldIndex值差了高位的1, 即newIndex = oldIndex + 2^n, 其中2^n正好是oldCap, 即newIndex = oldIndex + oldCap => 这样能够在尽可能保证桶index的不变性, 减少结点的移动, 减少了resize的时间
                    newTab[e.hash & (newCap - 1)] = e;

                // 如果原j桶链头属于红黑树结点
                else if (e instanceof TreeNode)
                    // HashMap#resize()扩容时拆分index桶链表，会有两种结果lo链表newIndex = oldIndex，hi链表newIndex = oldIndex + oldCap，大于6时保持红黑树，小于等于6时拆除红黑树
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                
                // 如果不属于红黑树结点，保持顺序地移动元素
                else { // preserve order
                    // loHead链表存放newIndex = oldIndex的结点
                    Node<K,V> loHead = null, loTail = null;
                    // hiHead链表存放newInedx = oldIndex + oldCap的结点
                    Node<K,V> hiHead = null, hiTail = null;
                    // 只有next结点, 没有prev结点, prev红黑树结点才有
                    Node<K,V> next;

                    do {
             // 遍历分割j桶链表, 根据newIndex是否发生变化, 决定存放结点到lo链表还是hi链表
                        next = e.next;

             // (e.hash & oldCap) == 0, 说明hash在oldCap最高位为0, 即扩容后newIndex = oldIndex, 结点存放到lo链表中
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;// 初始化lo链表头指针
                            else
                                loTail.next = e;// 追加结点到lo链表
                            // lo链表移动到一个结点
                            loTail = e;
                        }

              // 20201120 (e.hash & oldCap) != 0, 说明hash在oldCap最高位为1, 即扩容后newIndex = oldIndex + oldCap, 结点存放到hi链表中
                        else {
                            if (hiTail == null)
                                hiHead = e;// 初始化hi链表
                            else
                                hiTail.next = e;// 追加结点到hi链表
                            // hi链表移动到一个结点
                            hiTail = e;
                        }
                    } while ((e = next) != null);

                    // 如果lo链表不为空, 则设置lo链表头指针到新的j桶中
                    if (loTail != null) {
                        loTail.next = null;
                        // 这里因为newIndex = oldIndex
                        newTab[j] = loHead;
                    }

                    // 如果hi链表不为空, 则设置hi链表头指针到桶偏移+旧容量的新桶中
                    if (hiTail != null) {
                        hiTail.next = null;
                        // 这里因为newIndex = oldIndex + oldCap
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;// 返回新的散列表
}
```

##### 添加元素方法

- **put（K，V）**：将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则**会替换旧值**。
- **putIfAbsent（K，V）**：将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则**不会替换旧值**。
- **putAll（Map）**：添加复制集合的的所有键值对，散列表处于非创建模式中（用于LinkedHashMap结点插入后的回调访问函数，对于HashMap本身没用）。

```java
// 将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则会替换旧值。
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// 将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值。
public V putIfAbsent(K key, V value) {
    return putVal(hash(key), key, value, true, true);
}

// 添加复制集合的的所有键值对，散列表处于非创建模式中（用于LinkedHashMap结点插入后的回调访问函数，对于HashMap本身没用）
public void putAll(Map<? extends K, ? extends V> m) {
    putMapEntries(m, true);
}
// 添加复制集合的的所有键值对，构造函数的evict为false，putAll的evict为true（用于LinkedHashMap结点插入后的回调访问函数，对于HashMap本身没用）
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        // 如果散列表为null，则更新阈值
        if (table == null) { // pre-size
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            // 如果通过复制集合大小计算出来的数值大于阈值，则返回给定目标容量的2的幂次作为阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        }

        // 如果散列表不为null，则复制集合大小大于阈值，则根据阈值调整散列表大小，更新容量和阈值，以及移动桶链表位置。
        else if (s > threshold)
            resize();

        // 最后遍历复制集合，添加键值对到散列表中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();

            // 添加key-value对, 可以替换旧值
            putVal(hash(key), key, value, false, evict);
        }
    }
}

/**
 * hash: 指key的hash值（hashCode扰动后的结果）
 * key：指key的值
 * value：指value的值
 * onlyIfAbsent：为true时代表不能替换旧值，为false时则可以替换旧值
 * evict：为true时代表散列表处于非创建模式中，为false时则散列表处于创建模式中（用于LinkedHashMap	*		 结点插入后的回调访问函数，对于HashMap本身没用）
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;

    // 如果当前散列表tab为null, 或者容量n为0，则根据阈值调整散列表大小，更新容量和阈值，以及移动桶链表位置
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;

    // 计算key的hash值在散列表数组中的索引，如果该处桶为null，说明没有桶链头，此时需要创建普通结点, 然后作为桶的链头
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);

    // 如果该处桶不为null，说明存在桶链头，发生了哈希冲突，需要在链头后追加结点
    else {
        Node<K,V> e; K k;

        // 如果桶链头结点p的hash值（int）相等且key引用地址相等或者equals, 则说明在桶链头位置，存在了相同的键，此时备份旧值e
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;

        // 如果桶链头位置不存在相同的键，且该链头结点为红黑树结点，则使用红黑树的结点添加方法
        else if (p instanceof TreeNode) 
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        
        // 如果桶链头位置不存在相同的键，该链头结点也不为红黑树结点，说明为普通结点, 则遍历当前桶链表，追加结点
        else {
            for (int binCount = 0; ; ++binCount) {
                // 遍历到了尾结点
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);

     // 如果桶链表长度达到了红黑树转换的阈值8时（到7代表时再算上本身结点，就是8了，所以减去1）
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        // 则对该桶链表进行红黑树化
                        treeifyBin(tab, hash);
                    
                    // 添加结点后，结束遍历，此时e为null
                    break;
                }

                // 如果还没到尾结点，且next结点e的hash值（int）相等且key引用地址相等或者equals，说明在e位置存在相同的键，此时遍历结束，e就是要找的结点
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break;// 

                // 如果什么都没找到，则继续遍历桶链表
                p = e;
            }
        }

        // 如果确实在当前桶链表中找到对应的e结点（键相同的结点）
        if (e != null) { // existing mapping for key
            V oldValue = e.value;

            // 如果可以替换旧值, 或者旧值为null，则填充新值
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;

            // 触发LinkedHashMap结点访问后的回调函数
            afterNodeAccess(e);

            // 返回旧值
            return oldValue;
        }
    }

    ++modCount;

    // 如果是插入到尾结点中，且插入后实际元素个数大于阈值(下一个目标的容量)时, 则还需要根据阈值调整散列表大小，更新容量和阈值，以及移动桶链表位置
    if (++size > threshold)
        resize();

    // 触发LinkedHashMap结点插入后的回调函数，evict用于LinkedHashMap结点插入后的回调访问函数，对于HashMap本身没用
    afterNodeInsertion(evict);
    
    return null;
}
```

##### 删除元素方法

- **remove（Object）**：如果key存在，则从此映射中删除该key对应的映射，**忽略value值**。
- **remove（Object，Object）**：删除key和value都equals的键值对，**value值必须匹配**。

```java
// 如果key存在，则从此映射中删除该key对应的映射，忽略value值
public V remove(Object key) {
    Node<K,V> e;
    // 忽略value值，允许移动其他结点
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

// 删除key和value都equals的键值对，value值必须匹配
public boolean remove(Object key, Object value) {
    // 需要匹配value值，允许移动其他结点
    return removeNode(hash(key), key, value, true, true) != null;
}

/**
 * hash：key的hash值 
 * key：key值
 * value：如果matchValue为true则匹配的value值，否则忽略value值。
 * matchValue：为true代表则仅在值相等时才删除，为false代表值不相等也可以删除
 * movable：为true代表删除时可以移动其他结点，为false代表在删除时不能移动其他结点
 *         （只有迭代器的删除方法movable才会为false）
 **/
final Node<K,V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;

    // 当前散列表tab、容量n、根据hash值计算的索引访问到的数组元素p桶，如果p桶链头不为null，说明很可能目标结点存在p桶中
    if ((tab = table) != null && (n = tab.length) > 0 && (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;

        // 如果p桶链头的hash值（int）相等且key引用地址相等或者equals, 则说明在桶链头位置，则标记该链头结点为待删除结点node
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        
        // 如果p桶链头的键不匹配，则继续遍历next结点
        else if ((e = p.next) != null) {
            // 如果p桶链表为红黑树链表，则使用红黑树方法来获取hash和key对应的结点，并标记为node
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            
            // 如果p桶链表不是红黑树链表，则继续普通遍历p桶链表，则到找到hash值（int）相等且key引用地址相等或者equals的结点，说明此结点为目标结点，并标记为node
            else {
                do {
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }

        // 如果存在待删除结点node，如果matchValue为true则还需要判断value是否equals
        if (node != null && (!matchValue || (v = node.value) == value || (value != null && value.equals(v)))) {
            // 如果node结点为红黑树结点，则使用红黑树方法删除该结点
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            
            // 如果node结点不为红黑树结点，且为桶头结点，则删除后还需要重新设置桶头链表的头结点
            else if (node == p)
                tab[index] = node.next;
            
            // 如果node结点不为红黑树结点，也不为桶头结点，则解除node与前面一个结点的链接
            else
                p.next = node.next;

            ++modCount;
            --size;

            // LinkedHashMap删除结点后的回调函数 
            afterNodeRemoval(node);

            // 返回当前结点
            return node;
        }
    }

    // 找不到键匹配的结点，则返回null
    return null;
}
```

##### 获取元素方法

- **get（Object）**：获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}。
- **getOrDefault（Object，V）**：带默认值的获取, 如果获取不到key的元素, 则返回默认值。

```java
// 获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}。
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

// 带默认值的获取, 如果获取不到key的元素, 则返回默认值
public V getOrDefault(Object key, V defaultValue) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? defaultValue : e.value;
}

// 根据key和key的hash值获取结点
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;

    // 散列表tab，当前容量n, 根据hash值计算的索引访问到的数组元素fist桶
    if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
        // 如果first桶链头的hash值（int）相等且key引用地址相等或者equals, 则说明在桶链头位置，则直接返回first结点即可
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        
        // 如果first结点不匹配，则继续遍历链表
        if ((e = first.next) != null) {
            // 如果结点属于红黑树结点，则使用红黑树结点获取方法获取，找到后返回结点即可
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            
           	// 如果结点不属于红黑树结点，则使用普通方式遍历链表，找到后返回结点即可
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    
    // 找不到匹配的结点，则返回null
    return null;
}
```

##### 红黑树化指定hash桶

- **treeifyBin（Node，int）**：红黑树化hash对应桶中的普通链表，重构hash桶中的普通单向Node链表为双向无环TreeNode链表后，底层调用红黑树结点的实例方法treeify（Node）来红黑树化实例结点链表。

```java
// 红黑树化hash对应桶中的普通链表
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;

    // 如果散列表tab为null, 或者散列表容量n 小于 红黑树化最小阈值64时，则根据阈值调整散列表大小，更新容量和阈值，以及移动桶链表位置。
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();

    // 如果散列表满足红黑树化条件（n >= 64）时, 且通过hash计算出的散列表索引e桶不为null，则遍历当前e桶链表
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;

        do {
            // 替换e桶头结点为TreeNode结点
            TreeNode<K,V> p = replacementTreeNode(e, null);

            // 如果tl指针还没被初始化，说明还在头结点处，则备份桶头指针p到hd中
            if (tl == null)
                hd = p;
            // 如果tl指针已经被初始化了，说明不在头结点处了，则使用tl结点作为p的prev结点
            else {
                p.prev = tl;
                tl.next = p;
            }

            // 继续遍历e桶链表，直到尾结点
            tl = p;
        } while ((e = e.next) != null);

        // 经过e桶链表的遍历后，原先的单向node链表变成了双向无环的TreeNode链表hd，如果hd链表不为null，则红黑树化当前桶链表
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
// 替换Node结点为TreeNode结点
TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
    return new TreeNode<>(p.hash, p.key, p.value, next);
}
```

##### 红黑树结点方法

###### TreeNode

- **性质1**：红黑树的结点**要么是红色，要么是黑色**。
- **性质2**：红黑树的**根结点是黑色**的。
- **性质3**：红黑树的**叶子结点（nil）都是黑色**的。
- **性质4**：红黑树的**红色结点必须有两个黑色结点**。
  - **推论**：从根结点到每个叶子结点的所有路径上，不可能存在两个连续的红色结点。
- **性质5**：红黑树是**黑色平衡**的，即从根结点到每个叶子结点的所有路径中，所经过的黑色结点数都是一样的。
  - **推论**：如果一个结点右黑色的子结点，那么该结点一定是有两个孩子结点，因为必须有另一半才能保证该结点黑色平衡。

```java
// LinkedHashMap#Entry，继承HashMap#Node，以获得hash、key、value、next等属性，同时还新增了before和after指针作为头尾指针
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;// 头尾指针（LinkedHashMap双向链表使用，对HashMap本身没用）
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}

// TreeNode继承LinkedHashMap#Entry，以获得hash、key、value、next等HashMap#Node属性，before、after等LinkedhashMap#Entry属性，同时还行了parent、left、right、prev、red等属性
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;// 父结点
    TreeNode<K,V> left;// 左孩子
    TreeNode<K,V> right;// 右孩子
    TreeNode<K,V> prev;// 前驱结点
    boolean red;// 是否为红黑树
    
    TreeNode(int hash, K key, V val, Node<K,V> next) {
       super(hash, key, val, next);
    }
    
    // 返回包含此节点的树的根。
    final TreeNode<K,V> root() {...}

    // 左旋root结点
    static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root, TreeNode<K,V> p) {...}
    
    // 右旋root结点
    static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root, TreeNode<K,V> p) {...}
    
    // 插入后平衡红黑树
    static <K,V> TreeNode<K,V> balanceInsertion(
        TreeNode<K,V> root, TreeNode<K,V> x) {...}
    
    // 删除后平衡红黑树
    static <K,V> TreeNode<K,V> balanceDeletion(
        TreeNode<K,V> root, TreeNode<K,V> x) {...}

    // 移动指定的root结点到散列表桶头位置
    static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {...}
    
  // 当hashCode相等且不可比较时, 用于打破比较平局的情况 => 比较a和b两个对象的ClassName相等以及原始hashCode
    static int tieBreakOrder(Object a, Object b) {...}
    
    // 红黑树结点检查(递归检查)
    static <K,V> boolean checkInvariants(TreeNode<K,V> t) {...}
    
    // 根据hash值和key，从根结点开始查找
    final TreeNode<K,V> find(int h, Object k, Class<?> kc) {...}
    
    // 递归调用find（int，Object，Class）查找根节点
    final TreeNode<K,V> getTreeNode(int h, Object k) {...}
    
    // 红黑树化实例结点链表
    final void treeify(Node<K,V>[] tab) {...}
    
    // 普通化红黑树结点链表
    final Node<K,V> untreeify(HashMap<K,V> map) {...}
    
    // 红黑树结点的添加方法
    final TreeNode<K,V> putTreeVal(
        HashMap<K,V> map, Node<K,V>[] tab, int h, K k, V v) {...}
    
    // 红黑树结点的删除方法
    final void removeTreeNode(
        HashMap<K,V> map, Node<K,V>[] tab, boolean movable) {...}
    
    // 拆分红黑树桶链表, 会有两种结果newIndex = oldIndex; newIndex = oldIndex + oldCap, HashMap#resize()方法中调用
    final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {...}
}
```

###### static方法 - 左旋

- **背景**：在结点的添加和删除后，**为了避免子树高度变化，需要通过子树内部调整来保证树达到平衡**，其中2-3-4树是通过结点旋转和结点元素变化实现的，红黑树是通过结点旋转和变色实现。
- **左右左右右**：以x结点作为旋转点进行**左**旋，旋转后，x的**右**结点p成为x的父结点，p原本的**左**结点成为x结点的**右**结点，p原本的**右**结点保持不变。

```java
// 左旋p结点，左右左右右
static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root, TreeNode<K,V> p) {
    TreeNode<K,V> r, pp, rl;

    // 如果p的右节点r不为null，则有必要进行左旋，否则直接返回root即可
    if (p != null && (r = p.right) != null) {
        // 如果p有右结点r，且r还有左结点rl, 则旋转后成为p的右结点，并连接上rl与p的关系
        if ((rl = p.right = r.left) != null)
            rl.parent = p;

        // 如果p有父结点，则关联p的父结点pp与p的右结点r的关系，且如果pp为null，说明需要更新r为根结点，以及变为黑色
        if ((pp = r.parent = p.parent) == null)
            (root = r).red = false;

        // 如果pp不为null，说明r不需要成为根结点，则关联pp与r的关系
        else if (pp.left == p)
            pp.left = r;
        else
            pp.right = r;

        // 关联p结点与r结点的关系
        r.left = p;
        p.parent = r;
    }
    
    return root;
}
```

###### static方法 - 右旋

- **背景**：在结点的添加和删除后，**为了避免子树高度变化，需要通过子树内部调整来保证树达到平衡**，其中2-3-4树是通过结点旋转和结点元素变化实现的，红黑树是通过结点旋转和变色实现。
- **右左右左左**：以x结点作为旋转点进行**右**旋，旋转后，x的**左**结点p成为x的父结点，p原本的**右**结点成为x结点的**左**结点，p原本的**左**结点保持不变。

```java
// 右旋p结点，右左右左左
static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root, TreeNode<K,V> p) {
    TreeNode<K,V> l, pp, lr;

    // 如果p的左节点l不为null，则有必要进行右旋，否则直接返回root即可
    if (p != null && (l = p.left) != null) {
        // 如果p有左结点l，且l还有右结点lr, 则旋转后成为p的左结点，并连接上rl与p的关系
        if ((lr = p.left = l.right) != null)
            lr.parent = p;

        // 如果p有父结点，则关联p的父结点pp与p的左结点r的关系，且如果pp为null，说明需要更新l为根结点，以及变为黑色
        if ((pp = l.parent = p.parent) == null)
            (root = l).red = false;

        // 如果pp不为null，说明l不需要成为根结点，则关联pp与l的关系
        else if (pp.right == p)
            pp.right = l;
        else
            pp.left = l;

        // 关联p结点与l结点的关系
        l.right = p;
        p.parent = l;
    }
    
    return root;
}
```

###### static方法 - 插入结点后平衡红黑树

- **balanceInsertion（TreeNode，TreeNode）**：对应2-3-4树的情况：
  - **a. 空结点新增**：成为一个2结点，插入前树为null，插入后x需要变黑色，作为根结点。
  - **b. 合并到2结点中**：成为一个3结点，插入前2结点为黑色，插入后无论是（上黑下左红 |  上黑下右红）, 都符合3结点要求，因此无需调整。
  - **c. 合并到3结点中**：成为一个4结点，插入前为3结点（上黑下左红 |  上黑下右红），插入后成为4结点黑红红的情况，根据x插入位置不同分为6种情况：
    - **c.2.1.**  左三(中左左*) ，黑红红，不符合红黑树定义 => 需要调整，则中1右旋，中1变红，左1变黑。
    - **c.2.2.** 中左右*(其实就相当于左三，因为对父结点进行左旋，即得到左三) ，黑红红，不符合红黑树定义 => 需要调整，则左1左旋（得到左三），中1右旋，中1变红，新左变黑。
    - **c.2.3.** 右三(中 右右*) ，黑红红，不符合红黑树定义 => 需要调整，则中1左旋，中1变红，右1变黑。
    - **c.2.4.** 中 右左*(其实就相当于右三，因为对父结点进行右旋，即得到右三) 黑红红，不符合红黑树定义 => 需要调整，则右1右旋（得到右三），中1左旋，中1变红，新右变黑。
    - **c.2.5.** 中左 右*，黑红 红，符合红黑树定义 => 无需调整。
    - **c.2.6.** 中左* 右，黑红 红，符合红黑树定义 => 无需调整。
  - **d. 合并到4结点中**：成为一个裂变状态（变色后相当于升元了），插入前为4结点（黑红红），插入后4结点颜色反转，爷结点成为新的x结点，准备下一轮的向上调整，根据x插入的位置不同分为4种情况：
    - **d.2.1.** 中左左* 右(黑红红 红)，不符合红黑树定义 => 需要调整，则中变红，左1变黑，左2保持为红， 右1变黑，中看作为“插入结点”，继续向上调整。
    - **d.2.2.** 中左右* 右(黑红红 红)，不符合红黑树定义 => 需要调整，则中变红，左1变黑，右1保持为红，右2变黑，中看作为“插入结点”，继续向上调整。
    - **d.2.3.** 中左 右左*(黑红 红红)，不符合红黑树定义 => 需要调整，则中变红，左1变黑，左2保持为红，右1变黑，中看作为“插入结点”，继续向上调整。
    - **d.2.4.** 中左 右右*(黑红 红红)，不符合红黑树定义 => 需要调整，则中变红，左1变黑，右1变黑，右2保持为红，中看作为“插入结点”，继续向上调整。

```java
// 插入后平衡红黑树，到了这步红黑树结点的二叉树指针关系已经确定好了的, 只需要进行调整和变色就可以了
static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root, TreeNode<K,V> x) {
    x.red = true;// 标记插入结点x为红结点

    // 从x结点向上遍历, 发现需要调整则进行调整
    for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
        // x结点父结点xp, 如果为null, 说明x为根结点，相当于情况a，则设置当前结点为黑色(性质2)
        if ((xp = x.parent) == null) {
            x.red = false;
            return x;
        }

        // 如果x结点不是根结点, 且父结点xp为黑色或者没有爷结点xpp时，相当于情况b，即合并到2结点中，所以无需调整, 直接返回根结点即可。
        else if (!xp.red || (xpp = xp.parent) == null)
            return root;

        // 如果x结点不是根结点，且父节点xp为红色，且有爷结点，且父结点为爷结点的左孩子时，相当于插入到3、4结点中，即情况c.2.1 & c.2.2 & d.2.1 & d.2.2
        if (xp == (xppl = xpp.left)) {// 这里的xppl是为了获取当父结点为爷结点右孩子时的叔结点
            // 如果xpp右孩子叔结点不为null, 且为红结点时，相当于情况d.2.1 和 d.2.2，即合并到4结点，因此x结点需要插入到了父结点xp的孩子结点中，此时标记叔结点为黑色, 父节点为黑色, 爷结点为红色（4结点颜色反转），且设置爷结点为新的x结点, 准备进行下一轮的向上调整
            if ((xppr = xpp.right) != null && xppr.red) {
                xppr.red = false;
                xp.red = false;
                xpp.red = true;
                x = xpp;
            }
            
            // 如果x没有叔结点，或者叔结点为黑色（不会出现，因为黑色不平衡），相当于插入到3结点中
            else {
                 // 如果x结点为父节点xp的右孩子，即父结点为爷结点的左孩子且为红结点, x结点插入到父结点的右孩子时(相当于c.2.2 中左右*, 合并到3结点)
                if (x == xp.right) {
                    // 此时将父结点xp左旋即可得到c.2.1左三的情况（相当于x结点与父结点交换了位置），x赋值为了xp
                    root = rotateLeft(root, x = xp);

                    // 左旋后更新指针：原来的x结点作为xp, 如果为null(不会出现), 则xpp设置为null, 否则xpp设置为原来的x结点的爷结点，x为原来x结点的父结点
                    xpp = (xp = x.parent) == null ? null : xp.parent;

                    // 这步if起始做的是把c.2.2的情况转换为c.2.1左三的情况
                }

                // 到这里，无论是哪种情况，如果xp不为null那肯定都是红色, 且x肯定为父亲的左孩子(相当于c.2.1, 左三, 合并到3结点) => 父变黑, 爷变红, 爷进行右旋, 变成4结点
                if (xp != null) {
                    xp.red = false;
                    if (xpp != null) {
                        xpp.red = true;
                        root = rotateRight(root, xpp);
                    }
                }
            }
        }
        
        // 如果x结点不是根结点，且父节点xp为红色，且有爷结点，且父结点为爷结点的右孩子时，相当于插入到3、4结点中，即情况c.2.3 & c.2.4 & d.2.3 & d.2.4
        else {
            // 如果xpp做孩子叔结点不为null, 且为红结点时，相当于情况d.2.3 和 d.2.4，即合并到4结点，因此x结点需要插入到了父结点xp的孩子结点中，此时标记叔结点为黑色, 父节点为黑色, 爷结点为红色（4结点颜色反转），且设置爷结点为新的x结点, 准备进行下一轮的向上调整
            if (xppl != null && xppl.red) {
                xppl.red = false;
                xp.red = false;
                xpp.red = true;
                x = xpp;
            }

            // 如果x没有叔结点，或者叔结点为黑色（不会出现，因为黑色不平衡），相当于插入到3结点中
            else {
                // 如果x结点为父节点xp的左孩子，即父结点为爷结点的右孩子且为红结点, x结点插入到父结点的左孩子时(相当于c.2.4 中右左*, 合并到3结点)
                if (x == xp.left) {      
                    // 此时将父结点xp右旋即可得到c.2.3右三的情况（相当于x结点与父结点交换了位置），x赋值为了xp
                    root = rotateRight(root, x = xp);

                    // 右旋后更新指针：原来的x结点作为xp, 如果为null(不会出现), 则xpp设置为null, 否则xpp设置为原来的x结点的爷结点，x为原来x结点的父结点
                    xpp = (xp = x.parent) == null ? null : xp.parent;

                    // 这步if起始做的是把c.2.4的情况转换为c.2.3右三的情况
                }
                
                // 到这里，无论是哪种情况，如果xp不为null那肯定都是红色, 且x肯定为父亲的右孩子(相当于c.2.3, 右三, 合并到3结点) => 父变黑, 爷变红, 爷进行左旋, 变成4结点
                if (xp != null) {
                    xp.red = false;
                    if (xpp != null) {
                        xpp.red = true;
                        root = rotateLeft(root, xpp);
                    }
                }
            }
        }
    }
}
```

###### static方法 - 删除结点前/后平衡红黑树

- **balanceDeletion（TreeNode，TreeNode）**：删除结点前/后平衡红黑树，如果x所在结点为2-3-4树的2结点，则平衡后再删除，如果x所在结点为3结点或者4结点，在平衡前对应的结点就已经删除了，此时x作为该结点的替代结点而保留下来：
  - **x自己搞得定**：
    - 自己搞得定的意思就是，可以**在自己结点内部处理完毕**（对应2-3-4树结构），不影响其他树的结构。
    - **a.1. x为3结点或者4结点的红结点**：直接置黑返回x结点即可调整完毕（因为x是作为替代结点而保留下来的），然后交由上层方法删除x结点。
  - **x自己搞不定，兄弟搞得定**：
    - 自己搞不定的意思就是，自身结点为黑结点，如果直接删除会导致父结点所在的树黑色不平衡。
    - 兄弟搞得定的意思就是，**兄弟结点存在多余的子结点**（即兄弟结点为3结点或者4结点），此时，x的父结点就可以借出结点下来合并到x结点中，兄弟结点再借出结点合并到父结点中，这样x就可以顺利删除了，同时2-3-4树的结构还保持不变。
    - 但是，**前提是x的兄弟结点是真正的兄弟结点，即为黑色的结点**，如果为红色的结点，说明其只是父结点（3结点）的红结点，此时需要对父结点进行旋转，以保证x有真正的兄弟结点。
    - **b.1. 兄弟结点为3结点，但无右（左）**：在x在左子树一方时，x的兄弟结点xpr为右子树，如果xpr无右孩子在对父结点左旋时，会导致xpr为null，导致2-3-4树的结构不正确，因此，**b.1是一个临时情况，需要对xpr进行右旋，转换为b.2有右进一步处理**。x为右子树一方时则相反。
    - **b.2. 兄弟结点为3结点，但有右（左）**：在x在左子树一方时，x的兄弟结点xpr为右子树，如果xpr有右，则可以顺利地对父结点xp进行左旋。左旋后，在2-3-4树结构看来，xp作为xpr的左孩子（相当于父结点借出去一个结点，合并到x结点中），xpr作为xp的父亲（**相当于兄弟结点借出去一个结点，合并到父结点中**），xpr借出去的结点颜色为xp借出去的结点颜色，xp借出去的结点颜色一定要为黑色（相当于3结点），xpr剩余结点一定要为黑色（相当于叶子结点），返回x结点即可调整完毕，交由上层方法删除x结点。x为右子树一方时则相反。
    - **b.3. 兄弟结点为4结点，肯定有右**：在x在左子树一方时，x的兄弟结点xpr为右子树，如果xpr有右，则可以顺利地对父结点xp进行左旋。左旋后，在2-3-4树结构看来，xp作为xpr的左孩子（相当于父结点借出去一个结点，合并到x结点中），xpr作为xp的父亲（**相当于兄弟结点借出去一个结点，合并到父结点中，而且还多借出左孩子合并到x结点中**），xpr合并到父结点的颜色为xp借出去的结点颜色（而借出去的左孩子本来为红色所以不用变），xp借出去的结点颜色一定要为黑色（相当于4结点），xpr剩余结点一定要为黑色（相当于叶子结点），返回x结点即可调整完毕，交由上层方法删除x结点。x为右子树一方时则相反。
    - 在b.3中对于兄弟结点为4结点时，兄弟结点可以借出1个结点（需要旋转两次）或者2个结点（只需要旋转一次），**在JDK中无论是HashMap还是TreeMap，都选择借出2个结点，因为可以减少花销。**
  - **x自己搞不定，而且兄弟也搞不定**：
    - 自己搞不定的意思就是，自身结点为黑结点，如果直接删除会导致父结点所在的树黑色不平衡。
    - 兄弟也搞不定的意思就是，**兄弟结点也为黑结点，没有多余的子结点**，如果直接删除x，则导致叔结点所在路径多了一个黑色结点，造成黑色不平衡。
    - **c.1. 兄弟结点为2结点**：此时，为了让x能够顺利删除，**兄弟结点需要置红（自损）**，这样x在删除后，x父结点所在树还是黑色平衡的。但是，如果x父结点为黑色，x爷结点所在树则不黑色平衡了（因为父结点这边少了一个黑色结点），所以父结点的叔结点要也要被置红。**因此需要一路向上自损，直到碰到任意一个终止条件即可结束**：
      - **自损的终止条件1（向上碰到根结点）**：经过一路置红叔结点（置红叔结点是没问题的，因为出现该情况是叶子结点为3结点黑黑黑的时候，此时如果叔结点没有孩子结点即为黑色，而对于更上层的叔结点来说，貌似不会出现叔为黑红红这种情况），直到循环到根结点时（因为上面已经没有父节点了），则代表自损完毕，此时整棵树都是黑色平衡的了（都减少了一个黑色结点）。
      - **自损的终止条件2（向上碰到红结点）**：如果碰到红色结点时，只需要把该结点置黑，则不需要在置红叔结点了，此时相当于在父结点这边子树补回了一个黑色结点，而不影响叔结点那边子树的黑色结点数目，因此整棵树还是黑色平衡的。

```java
// 删除结点前/后平衡红黑树，如果x所在结点为2-3-4树的2结点，则平衡后再删除，如果x所在结点为3结点或者4结点，在平衡前对应的结点就已经删除了，此时x作为该结点的替代结点而保留下来
static <K,V> TreeNode<K,V> balanceDeletion(TreeNode<K,V> root, TreeNode<K,V> x) {
    // 传入的根结点root，当前结点x，x的父结点xp，xp的左孩子xpl，xp的右孩子xpr
    for (TreeNode<K,V> xp, xpl, xpr;;)  {
        // 如果x为null，或者为指定的根结点root，则无需调整，直接返回root即可
        if (x == null || x == root)
            return root;

        // 如果x不为null，也不为root，且父节点xp为null，说明为自损的终止条件1（向上碰到根结点），此时x为根结点，此时置黑x结点即可（此时达到整棵树的黑色平衡，因为上一轮已经对叔结点进行置红了），终止循环向上调整，返回x结点作为根结点
        else if ((xp = x.parent) == null) {
            x.red = false;
            return x;
        }

        // 如果x不为null，也不为root，且父结点xp也不为null，且x为红结点，说明为自损的终止条件2（向上碰到红结点），此时置黑x结点即可（此时达到整棵树的黑色平衡，因为上一轮已经对叔结点进行置红了），终止循环向上调整，返回x结点作为根结点。还有一种情况就是，x自己搞的定（3、4结点的红结点时），置黑直接返回即可。
        else if (x.red) {
            x.red = false;
            return root;
        }

        // 如果x不为null，也不为根结点，且父结点也不为null，且x为黑色结点(删除2结点时，此时自己搞不定，需要兄弟和父亲帮忙)，且x为xp的左孩子（这里的xpl也等于x为右孩子时的叔结点），则按左的方式调整
        else if ((xpl = xp.left) == x) {
            
            // 叔结点xpr为xp的右孩子，如果为红结点时，说明xpr不是x真正的兄弟结点（xpr只是为xp的红结点，此时xp为3结点)，则置黑叔结点xpr，置红父结点xp，对xp进行左旋，左旋后xpr成为xp的父结点，原xpr的左结点成为xp的右结点（这时x与x真正的兄弟结点才一一对应）
            if ((xpr = xp.right) != null && xpr.red) {
                xpr.red = false;
                xp.red = true;
                root = rotateLeft(root, xp);
                xpr = (xp = x.parent) == null ? null : xp.right;
            }

            // 如果xpr为null，说明x没有叔结点，也就是兄弟没有得借，因此当x删除后，xp所在树会少一个黑结点，为了保证黑色平衡，所以把xp赋值给x，继续循环向上调整，直到碰到红结点或者根结点
            if (xpr == null)
                x = xp;// 可以看作xpr为红色（也相当于null），需要向上置红调整

            // 如果xpr不为null，说明x有真正的兄弟xpr，则继续判断xpr的左孩子sl，右孩子sr，看xpr是否有得借
            else {
                TreeNode<K,V> sl = xpr.left, sr = xpr.right;

                // 如果xpr没有孩子结点，则说明xpr没有得借，或者xpr的孩子结点都为黑色，说明xpr为叶子结点，也没有得借，相当于情况c.1，因此当x删除后，xp所在树会少一个黑结点，为了保证黑色平衡，所以把xp赋值给x，继续循环向上调整，直到碰到红结点或者根结点
                if ((sr == null || !sr.red) && (sl == null || !sl.red)) {
                    xpr.red = true;
                    x = xp;
                }

                // 如果兄弟结点xpr有得借，则继续判断兄弟结点到底是3结点还是4结点
                else {
                    // 如果xpr没有右孩子，为3结点时，对应情况b.1无右，此时对xpr进行右旋成b.2有右的情况，右旋后左孩子成为新的xpr
                    if (sr == null || !sr.red) {
                        if (sl != null)
                            sl.red = false;
                        xpr.red = true;
                        root = rotateRight(root, xpr);
                        xpr = (xp = x.parent) == null ? null : xp.right;
                    }

                    // 如果xpr不为null，说明此时对应情况b.2（xpr为3结点有右）和b.3（xpr为4结点），即xpr为3、4结点的情况，此时需要对xp进行左旋，左旋后xpr成为xp的父结点，sr置黑，xp置黑，并设置x为root结点，代表调整完毕，退出循环
                    if (xpr != null) {
                        xpr.red = (xp == null) ? false : xp.red;
                        if ((sr = xpr.right) != null)
                            sr.red = false;
                    }
                    if (xp != null) {
                        xp.red = false;
                        root = rotateLeft(root, xp);
                    }
                    x = root;// x会被删除，所以设置为root也没关系
                }
            }
        }

        // 同理右边情况，反向操作，如果x不为null，也不为根结点，且父结点也不为null，且x为黑色结点(删除2结点时，此时自己搞不定，需要兄弟和父亲帮忙)，且x为xp的右孩子，xpl为叔结点（上面已赋值），则按右的方式调整
        else { // symmetric
            
            // 叔结点xpl为xp的左孩子，如果为红结点时，说明xpl不是x真正的兄弟结点（xpl只是为xp的红结点，此时xp为3结点)，则置黑叔结点xpl，置红父结点xp，对xp进行右旋，右旋后xpl成为xp的父结点，原xpl的右结点成为xp的左结点（这时x与x真正的兄弟结点才一一对应）
            if (xpl != null && xpl.red) {
                xpl.red = false;
                xp.red = true;
                root = rotateRight(root, xp);
                xpl = (xp = x.parent) == null ? null : xp.left;
            }

            // 如果xpl为null，说明x没有叔结点，也就是兄弟没有得借，因此当x删除后，xp所在树会少一个黑结点，为了保证黑色平衡，所以把xp赋值给x，继续循环向上调整，直到碰到红结点或者根结点
            if (xpl == null)
                x = xp;// 可以看作xpl为红色（也相当于null），需要向上置红调整

            // 如果xpl不为null，说明x有真正的兄弟xpl，则继续判断xpl的左孩子sl，右孩子sr，看xpl是否有得借
            else {
                TreeNode<K,V> sl = xpl.left, sr = xpl.right;

                 // 如果xpl没有孩子结点，则说明xpl没有得借，或者xpl的孩子结点都为黑色，说明xpl为叶子结点，也没有得借，相当于情况c.1，因此当x删除后，xp所在树会少一个黑结点，为了保证黑色平衡，所以把xp赋值给x，继续循环向上调整，直到碰到红结点或者根结点
                if ((sl == null || !sl.red) && (sr == null || !sr.red)) {
                    xpl.red = true;
                    x = xp;
                }

				// 如果兄弟结点xpl有得借，则继续判断兄弟结点到底是3结点还是4结点
                else {
                    // 如果xpl没有左孩子，为3结点时，对应情况b.1无左，此时对xpl进行左旋成b.2有左的情况，左旋后右孩子成为新的xpl
                    if (sl == null || !sl.red) {
                        if (sr != null)
                            sr.red = false;
                        xpl.red = true;
                        root = rotateLeft(root, xpl);
                        xpl = (xp = x.parent) == null ? null : xp.left;
                    }
                    
                    // 如果xpl不为null，说明此时对应情况b.2（xpl为3结点有左）和b.3（xpl为4结点），即xpl为3、4结点的情况，此时需要对xp进行右旋，右旋后xpl成为xp的父结点，sl置黑，xp置黑，并设置x为root结点，代表调整完毕，退出循环
                    if (xpl != null) {
                        xpl.red = (xp == null) ? false : xp.red;
                        if ((sl = xpl.left) != null)
                            sl.red = false;
                    }
                    if (xp != null) {
                        xp.red = false;
                        root = rotateRight(root, xp);
                    }
                    x = root;// x会被删除，所以设置为root也没关系
                }
            }
        }
    }
}
```

###### static方法  -  移动指定根结点到散列表桶头位置

- **moveRootToFront（Node，TreeNode）**：移动指定root结点到散列表桶头位置，原来root的前驱和后继互相链接，跳过root结点，设置原本的桶头first成为root.next，并清空root.prev，递归检查root结点是否为红黑树。

```java
// 移动指定的root结点到散列表桶头位置，原本的桶头first成为root.next
static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
    
    // 根结点root，散列表tab，容量n，根据root的hash值计算出来的散列表数组索引index以及桶first
    int n;
    if (root != null && tab != null && (n = tab.length) > 0) {
        int index = (n - 1) & root.hash;
        TreeNode<K,V> first = (TreeNode<K,V>)tab[index];

        // 如果桶头first不为指定的root结点，则需要进行移动
        if (root != first) {
            Node<K,V> rn;
            tab[index] = root;// 设置root结点为桶头结点
            TreeNode<K,V> rp = root.prev;// 获取root的前驱rp

            // 如果root的后继rn不为null，则链接后继rn和前驱rp，跳过root结点
            if ((rn = root.next) != null)
                ((TreeNode<K,V>)rn).prev = rp;

            // 如果root的前驱rp不为null，则链接前驱rp和后继rn，跳过root结点
            if (rp != null)
                rp.next = rn;

            // 如果原桶头first结点不为null，则链接first与root
            if (first != null)
                first.prev = root;

            // 清空root与原前驱rp和后继rn的关系
            root.next = first;
            root.prev = null;
        }

        // 到这里，说明桶头已经是root结点了，这时递归检查红黑树结点root，是否为一颗合法的红黑树
        assert checkInvariants(root);
    }
}
```

###### static方法 - 递归检查指定结点是否为红黑树

- **checkInvariants（TreeNode）**：递归检查红黑树结点t，是否为一颗合法的红黑树。校验了前驱后继与t的关系、左右孩子与t的关系、左右孩子hash值与t的hash值关系、t结点颜色与左右孩子颜色关系（红结点两孩子结点必须为黑色）、递归校验左子树、递归校验右子树。

```java
// 递归检查红黑树结点t，是否为一颗合法的红黑树
static <K,V> boolean checkInvariants(TreeNode<K,V> t) {
    // 待检查结点t，父结点tp, 左孩子tl, 右孩子tr, 前驱tb, 后继tn
    TreeNode<K,V> tp = t.parent, tl = t.left, tr = t.right,
    tb = t.prev, tn = (TreeNode<K,V>)t.next;

    // 如果前驱tb不为null，但前驱的next结点不为t结点, 则返回false
    if (tb != null && tb.next != t)
        return false;

    // 如果后继tn不为null，但后继的prev结点不为t结点, 则返回false
    if (tn != null && tn.prev != t)
        return false;

    // 如果父结点tp不为null，但t又不是tp的左右孩子, 则返回false
    if (tp != null && t != tp.left && t != tp.right)
        return false;

    // 如果左孩子tl不为null，且左孩子的父亲不是t或者大于父亲的hashz值，则返回false（左孩子的hash值必须小于根结点的hash值）
    if (tl != null && (tl.parent != t || tl.hash > t.hash))
        return false;

    // 如果右孩子tr不为null, 且右孩子的父亲不是t或者小于父亲的hash值，则返回false（右孩子的hash值必须大于根结点的hash值）
    if (tr != null && (tr.parent != t || tr.hash < t.hash))
        return false;

    // 如果t是红节点, 且左右孩子都是红结点, 则返回false
    if (t.red && tl != null && tl.red && tr != null && tr.red)
        // bug？如果结点为红结点, 那么左右孩子结点肯定为黑结点! 节点红, 左孩子红, 右孩子黑, 也算红黑树？估计是不会出现这种情况的，因为不符合2-3-4树结点的特点
        return false;

    // 如果左孩子tl不为叶子结点，则继续校验左子树
    if (tl != null && !checkInvariants(tl))
        return false;

    // 如果右孩子tr不为叶子结点，则继续校验右子树
    if (tr != null && !checkInvariants(tr))
        return false;

    // 否则证明的确是一颗红黑树，返回true
    return true;
}
```

###### 实例方法  -  红黑树化实例结点链表

- **treeify（Node）**：红黑树化实例结点链表。遍历实例结点所在链，一路从根结点hash值比较当前结点x的hash值大小（小于等于的继续遍历左子树，大于的遍历右子树），链接x的父结点与x，插入x后平衡红黑树，遍历插入完成后移动指定的root结点到散列表桶头位置，完成实例结点链表的红黑树化。

```java
// 红黑树化实例结点链表
final void treeify(Node<K,V>[] tab) {
    TreeNode<K,V> root = null;

    // 从实例结点开始遍历链表
    for (TreeNode<K,V> x = this, next; x != null; x = next) {
        next = (TreeNode<K,V>)x.next;
        x.left = x.right = null;

        // 如果根结点还没初始化，说明还在根结点上，则设置当前结点为根结点(黑色，性质2)
        if (root == null) {
            x.parent = null;
            x.red = false;
            root = x;
        }
        
        // 如果根结点已经初始化了，说明不在根结点上，则备份实例结点的key值k，实例结点key的hash值h
        else {
            K k = x.key;
            int h = x.hash;
            Class<?> kc = null;// 比较器的Class对象

        // 从根结点开始遍历(相当于拿到链头x，然后遍历插入剩余结点，所以需要插入后平衡红黑树)，p为比较结点，比较结果dir（根据hash值比较），p的hash值ph，p的key值pk
            for (TreeNode<K,V> p = root;;) {
                int dir, ph;
                K pk = p.key;

                // 如果ph大于实例结点key的hash值，说明x应该为p的左子树，此时dir为-1
                if ((ph = p.hash) > h)
                    dir = -1;
                
                // 如果ph小于实例结点key的hash值，说明x应该为p的右子树，此时dir为1
                else if (ph < h)
                    dir = 1;
                
           // 如果ph等于key的hash值，则判断是否还x是否还实现了Comparable接口，如果是则继续比较
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0)
                    // 当hashCode相等且不可比较时, 用于打破比较平局的情况 => 比较a和b两个对象的ClassName相等以及原始hashCode
                    dir = tieBreakOrder(k, pk);

                // 到这里说明已经比较完毕，则备份比较结点p的指针为xp
                TreeNode<K,V> xp = p;

                // 如果比较结果dir<=0, 说明x应该为p的左孩子，则p取p的左孩子，否则说明x应该为p的右孩子，则p取p的右孩子
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    // 设置实例结点x的父结点为xp（即比较结点p原来的结点）
                    x.parent = xp;
                    
                    // 如果比较结果dir<=0, 说明x应该为xp的左孩子，此时设置x为xp的左孩子
                    if (dir <= 0)
                        xp.left = x;
                    
                    // 如果比较结果dir<=0, 说明x应该为xp的右孩子，此时设置x为xp的右孩子
                    else
                        xp.right = x;

                    // 插入后平衡红黑树
                    root = balanceInsertion(root, x);
                    break;
                }
            }
        }
    }

    // 红黑树化x桶链表后，则移动指定的root结点到散列表桶头位置，原本的桶头first成为root.next
    moveRootToFront(tab, root);
}
```

###### 实例方法 - 普通化红黑树结点链表

- **untreeify（HashMap）**：普通化红黑树结点链表。只需要使用红黑树结点的next指针，从桶头遍历到尾，返回链头指针hd即可。

```java
// 普通化红黑树结点链表
final Node<K,V> untreeify(HashMap<K,V> map) {
    // 链头指针hd，链尾指针tl
    Node<K,V> hd = null, tl = null;

    // 从结点实例q开始遍历
    for (Node<K,V> q = this; q != null; q = q.next) {
        // 替换TreeNode结点为Node结点
        Node<K,V> p = map.replacementNode(q, null);

        // 如果tl为null，说明链没有元素，此时把p作为链头，hd作为链头指针
        if (tl == null)
            hd = p;
        
        // 如果tl不为null，说明链中已经存在元素，此时把p追加到链后，tl作为链尾指针
        else
            tl.next = p;
        tl = p;
    }

    // 最后返回tl链的链头指针hd
    return hd;
}
// 替换TreeNode结点为Node结点
Node<K,V> replacementNode(Node<K,V> p, Node<K,V> next) {
    return new Node<>(p.hash, p.key, p.value, next);
}
```

###### 实例方法 - 分割红黑树桶链表

- **split（HashMap，Node，int，int）**：HashMap#resize()扩容时拆分index桶链表，会有两种结果lo链表newIndex = oldIndex，hi链表newIndex = oldIndex + oldCap，大于6时保持红黑树，小于等于6时拆除红黑树。

```java
// HashMap#resize()扩容时拆分index桶链表，会有两种结果lo链表newIndex = oldIndex，hi链表newIndex = oldIndex + oldCap，大于6时保持红黑树，小于等于6时拆除红黑树
final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
    TreeNode<K,V> b = this;// 实例结点

    // Relink into lo and hi lists, preserving order
    TreeNode<K,V> loHead = null, loTail = null;
    TreeNode<K,V> hiHead = null, hiTail = null;
    int lc = 0, hc = 0;// lo、hi链表容量

    // 遍历实例结点链表
    for (TreeNode<K,V> e = b, next; e != null; e = next) {
        next = (TreeNode<K,V>)e.next;
        e.next = null;// 置空next结点

        // (e.hash & oldCap) == 0，说明e.hash < oldCap，newIndex = oldIndex，则把结点放到lo链表
        if ((e.hash & bit) == 0) {
            if ((e.prev = loTail) == null)
                loHead = e;
            else
                loTail.next = e;
            loTail = e;
            ++lc;
        }
        
        // (e.hash & oldCap) == 1，说明e.hash > oldCap，newIndex = oldIndex + oldCap， 则把结点放到hi链表
        else {
            if ((e.prev = hiTail) == null)
                hiHead = e;
            else
                hiTail.next = e;
            hiTail = e;
            ++hc;
        }
    }

    // 如果lo链表不为null，则检查lo容量是否小于红黑树拆除阈值
    if (loHead != null) {
        // 如果lo链表容量 <= 拆除链表阈值6时，则拆除红黑树为普通结点链表
        if (lc <= UNTREEIFY_THRESHOLD)
            tab[index] = loHead.untreeify(map);
        
        // 否则不需要拆除红黑树, 则设置loHead为index桶头结点
        else {
            tab[index] = loHead;

            // 如果hi链表也不为null，说明拆除了原桶链，此时还需要重新红黑树化hi链表
            if (hiHead != null) // (else is already treeified)
                loHead.treeify(tab);
        }
    }

    // 如果hi链表不为null，则检查hc容量是否小于红黑树拆除阈值
    if (hiHead != null) {
        // 如果hi链表容量 <= 拆除链表阈值6时，则拆除红黑树为普通结点链表
        if (hc <= UNTREEIFY_THRESHOLD)
            tab[index + bit] = hiHead.untreeify(map);
        
        // 否则不需要拆除红黑树, 则设置loHead为index + cap桶头结点
        else {
            tab[index + bit] = hiHead;

            // 如果lo链表也不为null，说明拆除了原桶链，此时还需要重新红黑树化lo链表
            if (loHead != null)
                hiHead.treeify(tab);
        }
    }
}
```

###### 实例方法 - 红黑树结点的添加方法

- **putTreeVal（HashMap，Node，int，K，V）**：红黑树结点的添加方法（插入成功则返回null，插入失败则返回已经存在的结点）。从根结点遍历比较插入结点x的hash值（小于等于0的说明x应该在左边，大于0的说明x应该在右边），找到合适位置后（叶子结点），维护x与父结点、prev结点、next结点的关系，插入后平衡红黑树，移动root结点到桶头，最后返回成功标志null。

```java
// 红黑树结点的添加方法（插入成功则返回null，插入失败则返回已经存在的结点）
final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab, int h, K k, V v) {
    // 比较Class对象kc，是否已经找过了searched，根节点root，实例结点this，
    Class<?> kc = null;
    boolean searched = false;
    TreeNode<K,V> root = (parent != null) ? root() : this;

    // 从根结点开始查找，比较结果dir， 比较结点hash值ph，比较结点key值pk
    for (TreeNode<K,V> p = root;;) {
        int dir, ph; K pk;

        // 如果ph大于h，说明插入结点应该在p的左边，dir为-1
        if ((ph = p.hash) > h)
            dir = -1;

        // 如果ph小于h，说明插入结点应该在p的右边，dir为1
        else if (ph < h)
            dir = 1;

        // 如果ph等于h，且pk为插入结点的key值，或者equals，则说明p就是要插入的位置，此时返回p结点
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        
        // 如果ph等于h，且pk不为插入结点的key值，也不equals，则判断x是否还实现了Comparable接口，如果是则继续比较是否相等
        else if ((kc == null && (kc = comparableClassFor(k)) == null) ||(dir = compareComparables(kc, k, pk)) == 0) {
            // 如果还没找过，则递归查找左右子树, 直到找到指定hash和key匹配的结点q返回即可
            if (!searched) { 
                TreeNode<K,V> q, ch;
                searched = true;
                if (((ch = p.left) != null && (q = ch.find(h, k, kc)) != null) || ((ch = p.right) != null && (q = ch.find(h, k, kc)) != null))
                    return q;
            }

            // 如果已经递归找过了，当hashCode相等且不可比较时, 用于打破比较平局的情况 => 比较a和b两个对象的ClassName相等以及原始hashCode
            dir = tieBreakOrder(k, pk);
        }

        // 到这里，比较结果dir已经确定，如果dir<=0, 则从p的左子树开始找，否则从p的右子树开始找，p结点备份为xp, p继续作为左孩子或者右孩子，xp后继xpn，如果p为null，说明到了叶子结点，则构建TreeNode结点，next指向xp的next
        TreeNode<K,V> xp = p;
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            Node<K,V> xpn = xp.next;
            TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);

            // 如果dir<=0，说明x应该在xp的左边，则设置x为xp的左孩子
            if (dir <= 0)
                xp.left = x;
            
            // 如果dir>0，说明x应该在xp的右边，则设置x为xp的右孩子
            else
                xp.right = x;
            xp.next = x;
            x.parent = x.prev = xp;

            // 如果xp原后继xpn不为null，则关联x与xpn的关系
            if (xpn != null)
                ((TreeNode<K,V>)xpn).prev = x;

            // 插入x后平衡红黑树, 然后移动指定的root结点到散列表桶头位置
            moveRootToFront(tab, balanceInsertion(root, x));

            // 插入成功则返回null
            return null;
        }
    }
}

// 构建TreeNode结点
TreeNode<K,V> newTreeNode(int hash, K key, V value, Node<K,V> next) {
    return new TreeNode<>(hash, key, value, next);
}
```

###### 实例方法 - 红黑树结点的删除方法

- **removeTreeNode（HashMap，Node，boolean）**：
  - **替代结点**：红黑树是一种自平衡的二叉搜索树，而二叉搜索树删除，本质上是**找前驱或者后继结点来替代删除**（这里是replacement替代p然后删除p）。
  - **A. 如果要删除的结点是叶子结点**：则直接删除即可（肯定为黑色）。
  - **B. 如果要删除的结点只有一个孩子结点**：则使用孩子结点进行替代，然后删除"替代结点"。
  - **C. 如果要删除的结点有两个孩子结点**：则需要找到前驱或者后继进行替代，然后删除"替代结点"。
    - **C.1. 如果替代结点没有孩子结点**：此时所在的结点为2-3-4树的2结点，则直接要"替代结点"即可。
    - **C.2. 如果替代结点有孩子结点且孩子结点为替代方向**：此时所在的结点为2-3-4树的3结点或者4结点，则继续使用孩子结点进行替代，然后“替代结点”即可（二次替代）。
  - 无论是哪种情况，红黑树结点的删除方法，都要调用平衡红黑树的方法，在删除结点前/后平衡红黑树。

```java
// 红黑树结点的删除方法
final void removeTreeNode(HashMap<K,V> map, Node<K,V>[] tab, boolean movable) {
    // 散列表tab，容量n，实例结点hash值对应散列表数组索引index，index对应桶头结点first、root， root的左孩子rl，实例结点后继succ，实例结点前驱pred
    int n;
    if (tab == null || (n = tab.length) == 0)
        return;
    int index = (n - 1) & hash;
    TreeNode<K,V> first = (TreeNode<K,V>)tab[index], root = first, rl;
    TreeNode<K,V> succ = (TreeNode<K,V>)next, pred = prev;

    // 如果pred为null, 说明实例结点为桶头结点，由于是删除实例结点，则更新后继succ作为新的桶头
    if (pred == null)
        tab[index] = first = succ;
    // 如果pred不为null，说明实例结点不为桶头结点，则链接前驱和后继，跳过实例结点
    else
        pred.next = succ;
    if (succ != null)
        succ.prev = pred;

    // 链接pred、succ后，如果桶头结点为null，说明桶内已经没有数据了，所以不用再处理了，直接返回
    if (first == null)
        return;

    // 如果桶内还有数据，且root的还有父结点，说明传入的root并不是真正的根结点，此时继续遍历红黑树找到新的结点作为root结点
    if (root.parent != null)
        root = root.root();

    // 如果root为null或者没有孩子时，说明红黑树结点太少了，则拆除当前红黑树，将其退化成普通链表
    if (root == null || root.right == null || (rl = root.left) == null || rl.left == null) {
        tab[index] = first.untreeify(map);  // too small
        return;
    }

    // 实例结点p，实例结点左孩子pl，实例结点右孩子pr，替代结点replacement（null）
    TreeNode<K,V> p = this, pl = left, pr = right, replacement;

    // 如果实例结点p有两个孩子结点，则一路遍历右孩子pr的左孩子sl，直到sl为叶子结点，即查找p的后继s，并p和s的颜色
    if (pl != null && pr != null) {
        TreeNode<K,V> s = pr, sl;
        while ((sl = s.left) != null) // find successor
            s = sl;
        boolean c = s.red; s.red = p.red; p.red = c; // swap colors

        // 后继s的右孩子sr, 实例结点p的父结点pp
        TreeNode<K,V> sr = s.right;
        TreeNode<K,V> pp = p.parent;

        // 如果后继s为p的右孩子，说明s是p的直接后继，直接反转两者的父子关系即可，即s作为父亲，p作为右孩子，使得实例结点p交换到了后继s的位置
        if (s == pr) { // p was s's direct parent
            p.parent = s;
            s.right = p;
        } 
        // 如果后继s不是p的右孩子，说明s是p的间接后继，则维护s、p、sp、pr的指针关系，使得实例结点p交换到了后继s的位置
        else {
            TreeNode<K,V> sp = s.parent;
            if ((p.parent = sp) != null) {
                if (s == sp.left)
                    sp.left = p;
                else
                    sp.right = p;
            }
            if ((s.right = pr) != null)
                pr.parent = s;
        }

		// 交换了实例结点p与后继s的位置后，继续更新s、p、sr、sl、pl、pp的关系
        p.left = null;
        if ((p.right = sr) != null)
            sr.parent = p;
        if ((s.left = pl) != null)
            pl.parent = s;
        if ((s.parent = pp) == null)
            root = s;
        else if (p == pp.left)
            pp.left = s;
        else
            pp.right = s;

        // 到这里，p与s交换了位置，并完成了所有父结点、孩子节点的指针关系维护。如果sr不为null，说明s有右孩子（这里由于原s是通过遍历原p的右孩子的所有左孩子找到的，所以原s肯定是没有左孩子的，如果判断没有右孩子，则可以说明原s肯定是没有孩子的），则设置右孩子为替代结点(相当于情况C.2)
        if (sr != null)
            replacement = sr;
        // 否则，如果原s没有右孩子，则交换后的p就为替代结点(相当于情况C.1)
        else
            replacement = p;
    }
    
    // 如果实例结点p只有左孩子时，则替代结点为左孩子pl（相当于情况B）
    else if (pl != null)
        replacement = pl; 

    // 如果实例结点p只有右孩子时，则替代结点为右孩子pr（相当于情况B）
    else if (pr != null)
        replacement = pr;

    // 否则p没有孩子结点，则替代结点为实例结点p本身（相当于情况A）
    else
        replacement = p;

    // 到这一步，实例结点的替代结点replacement已经找到了。如果replacement不为p结点，说明为2-3-4树3结点或者4结点的红结点，即情况B和C.2，为了让replacement代替p结点，则链接pp与replacement并清空p所有的链接，此后p就没有被任何结点引用等待GC回收，相当于于p脱离了红黑树
    if (replacement != p) {
        TreeNode<K,V> pp = replacement.parent = p.parent;
        if (pp == null)
            root = replacement;
        else if (p == pp.left)
            pp.left = replacement;
        else
            pp.right = replacement;
        p.left = p.right = p.parent = null;
    }

    // 删除结点前/后平衡红黑树，如果x所在结点为2-3-4树的2结点，则平衡后再删除x，否则在平衡前对应的结点就已经删除了（此时x作为该结点的替代结点而保留下来），r为根结点
    TreeNode<K,V> r = p.red ? root : balanceDeletion(root, replacement);

    // 如果替代节点replacement为p结点，说明要删除的是2-3-4树的2结点（相当于情况A和C.1），则脱离p链接
    if (replacement == p) {  // detach
        TreeNode<K,V> pp = p.parent;
        p.parent = null;
        if (pp != null) {
            if (p == pp.left)
                pp.left = null;
            else if (p == pp.right)
                pp.right = null;
        }
    }

    // 至此，结点的删除与红黑树的平衡操作都已经处理完毕，如果允许删除时可以移动其他结点，则移动根结点到桶头（只有迭代器删除方法才会为false）
    if (movable)
        moveRootToFront(tab, r);
}
```

###### 实例方法 - 红黑树根结点的查找方法

- **root（）**：从实例结点向上遍历，找到根结点(没有父结点的结点)则返回。
- **getTreeNode（int，Object）**：根据hash值和key值，从根结点开始查找红黑树结点。底层调用find（int，Object，Class）方法进行查找。
- **find（int，Object，Class）**：根据hash值、key值和比较对象kc，从根结点开始查找查找红黑树结点。
  - **ph > h**：说明查找结点在左子树。
  - **ph < h**：说明查找结点在右子树。
  - **ph == h**：如果key相等或者equals，则说明p就为要找的hash值为h的结点。否则还说明哈希冲突了，还需要继续遍历查找右子树和左子树。

```java
// 从实例结点向上遍历，找到根结点(没有父结点的结点)则返回
final TreeNode<K,V> root() {
    for (TreeNode<K,V> r = this, p;;) {
        if ((p = r.parent) == null)
            return r;
        r = p;
    }
}

// 根据hash值和key值，从根结点开始查找红黑树结点
final TreeNode<K,V> getTreeNode(int h, Object k) {
    return ((parent != null) ? root() : this).find(h, k, null);
}

// 根据hash值、key值和比较对象kc，从根结点开始查找查找红黑树结点
final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
    TreeNode<K,V> p = this;
    do {
        // 实例结点p，p的hash值ph，比较结果dir，p的key值pk，p的左孩子pl，p的右孩子pr，要查找的结点q（null）
        int ph, dir; K pk;
        TreeNode<K,V> pl = p.left, pr = p.right, q;

        // 如果ph大于h，说明要查找的结点在左子树
        if ((ph = p.hash) > h)
            p = pl;
        
        // 如果ph小于h，说明要查找的结点在右子树  
        else if (ph < h)
            p = pr;
        
        // 如果ph等于h，且pk相等或者equals，说明p就是要找的结点
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        
        // 如果ph等于h，但pk不相等且不equals，说明哈希冲突了，需要往后继续遍历；如果pl为null，说明还没有左孩子，则往右子树继续遍历
        else if (pl == null)
            p = pr;
        
        // 如果ph等于h，但pk不相等且不equals，说明哈希冲突了，需要往后继续遍历；如果pl不为null但pr为null，说明还没有右孩子，则往左子树继续遍历
        else if (pr == null)
            p = pl;
        
        // 如果ph等于h，但pk不相等且不equals，说明哈希冲突了，需要往后继续遍历，但pl和pr都不为null，此时就需要判断x是否还实现了Comparable接口，如果是则继续比较是否相等，dir取其比较结果；如果dir<0，则p取左子树，否则取右子树
        else if ((kc != null ||
                  (kc = comparableClassFor(k)) != null) &&
                 (dir = compareComparables(kc, k, pk)) != 0)
            p = (dir < 0) ? pl : pr;
        
        // 如果ph等于h，但pk不相等且不equals，说明哈希冲突了，需要往后继续遍历，但pl和pr都不为null，且比较对象也为null，则递归查找右孩子，如果找到则返回
        else if ((q = pr.find(h, k, kc)) != null)
            return q;
        
        // 如果ph等于h，但pk不相等且不equals，说明哈希冲突了，需要往后继续遍历，但pl和pr都不为null，且比较对象也为null，则递归查找右孩子，如果没找到则p设置为左子树，继续循环查找
        else
            p = pl;
    } while (p != null);

    // 如果确实找不到则返回null
    return null;
}
```

#### LinkedHashMap

![1622987433370](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1622987433370.png)

##### 特点

- LinkedHashMap，**Map接口的哈希表和链表实现**，对于HashMap还维护了一个双向链表贯穿所有条目，因此迭代条目是顺序的，通常是条目被插入到映射中的顺序（即插入顺序）。注意，如果将键插入到已存在该键的映射中，其插入顺序不会收到影响。**允许null元素**。
- 可以使客户端免受HashMap和HashTable混乱的排序影响，也不会增加到与TreeMap相同的成本，可传入原始Map集合，生成与原始Map相同顺序的副本，从而**保证顺序地迭代条目**。
- 还提供了一个特殊的构造函数 LinkedHashMap（int，float，boolean）来创建容器，其迭代顺序为条目最后访问的顺序（**从最近最少访问到最近访问**，即时间倒序再次数倒序），**非常适合用于构建LRU缓存**，其中调用put、putIfAbsent、get、getOrDefault、compute、computeIfAbsent、computeIfPresent、merge等方法在调用完成后，都将视为对条目的访问。
- 与HashMap一样，在假设哈希函数能够正确地分散元素时，LinkedHashMap的添加、包含和删除方法性能为O（1），但由于维护了一个双向链表，在这点的性能可能会略低于HashMap。对于迭代性能，LinedHashMap所需的时间与条目实际大小成正比（不管散列表容量如何），而HashMap所需的时间与散列表容量和条目实际大小成正比。
- 两个影响性能的参数：其中，设置过高的初始容量的影响并没有HashMap那么严重，因为LinkedHashMap的迭代性能不受容量影响。
  - **初始容量**：当前容量是散列表中存储桶的数量，而初始容量只是创建散列表的容量。
  - **负载因子**：指HashMap自动增加散列表容量之前，允许散列表获得满意的度量。等于实际大小 / 当前容
    量，当散列表中的条目数超过负载因子和当前容量的乘积时，会重建内部数据结构，使散列表具有大约两
    倍的桶数。
- **LinkedHashMap是非同步的**，在多个线程并发访问时，通常是通过同步一些自然封装映射的对象来完成，比如Collections.synchronizedMap。
- **LinkedHashMap的所有迭代器都是快速失败的**，即如果在创建迭代器后的任何时间对结构进行结构修改，则除了通过迭代器自己的remove方法之外，该迭代器都将抛出{ @link ConcurrentModificationException}，这在面对并发修改时，迭代器可以快速干净地失败，而不是在未来不确定的时间冒着任意、非确定性行为的风险去修改。但是**无法保证迭代器的快速失败行为**，因为一般而言，在存在非同步并发修改的情况下，不可能做出任何硬保证，**只会尽最大努力抛出**ConcurrentModificationException，因此，编写一个依赖于这个异常来保证其正确性的程序是错误的，**快速失败行为只适用于检测错误**。

##### 数据结构

![1625373689741](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625373689741.png)

##### 构造方法

LinkedHashMap所有的构造方法都依赖父类HashMap的构造方法，除了有者同样定义的**默认初始容量16和默认负载因子0.75**外，还定义了一个条目“最近”的访问顺序（**访问顺序为true，代表访问后会移动元素到链表末尾，插入顺序为false，代表不作任何移动，保持为插入顺序**）。

- **空参的构造函数**：默认初始容量16，默认负载因子0.75，默认迭代顺序为插入顺序。
- **指定初始容量的构造函数**：默认负载因子0.75，默认迭代顺序为插入顺序。
- **指定初始容量和负载因子的构造函数**：默认迭代顺序为插入顺序。
- **指定初始容量、负载因子和迭代顺序的构造函数**。
- **指定复制集合的构造函数**：默认初始容量为复制集合的实际大小，默认负载因子0.75，默认迭代顺序为插入顺序。

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
{
	// LinkedHashMap#Entry普通结点对比HashMap#Node，多了before和after头尾指针
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;// 头尾指针（LinkedHashMap双向链表使用，对HashMap本身没用）
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
    
    transient LinkedHashMap.Entry<K,V> head;// 双向链表的头部（最年长的）
    transient LinkedHashMap.Entry<K,V> tail;// 双向链表的尾部（最年轻的）
    final boolean accessOrder;// 条目“最近”的访问顺序: 访问顺序为true，插入顺序为false
    
    // 空参的构造函数：默认初始容量16，默认负载因子0.75，默认迭代顺序为插入顺序
    public LinkedHashMap() {
        super();
        accessOrder = false;
    }

	// 指定初始容量的构造函数：默认负载因子0.75，默认迭代顺序为插入顺序
    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }

	// 指定初始容量和负载因子的构造函数：默认迭代顺序为插入顺序
    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }
    
    // 指定初始容量、负载因子和迭代顺序的构造函数
    public LinkedHashMap(int initialCapacity, float loadFactor,boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
    
    // 指定复制集合的构造函数：默认初始容量为复制集合的实际大小，默认负载因子0.75，默认迭代顺序为插入顺序
    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super();
        accessOrder = false;
        putMapEntries(m, false);
    }
}
```

##### 迭代方法

- **抽象的LinkedHashMap.Entry迭代器**：LinkedHashMap的迭代器基类，快速失败机制，遍历next指针，返回LinkedHashMap.Entry对象。
- **Map.Entry迭代器**：继承LinkedHashMap的迭代器基类，快速失败机制，遍历next指针，返回Map.Entry对象。
- **LinkedHashMap.Entry#Key迭代器**：继承LinkedHashMap的迭代器基类，快速失败机制，遍历next指针，返回LinkedHashMap.Entry#Key对象。
- **LinkedHashMap.Entry#Value迭代器**：继承LinkedHashMap的迭代器基类，快速失败机制，遍历next指针，返回LinkedHashMap.Entry#Value对象。

```java
// 抽象的LinkedHashMap.Entry迭代器：LinkedHashMap的迭代器基类，快速失败机制，遍历next指针，返回LinkedHashMap.Entry对象
abstract class LinkedHashIterator {
    LinkedHashMap.Entry<K,V> next;
    LinkedHashMap.Entry<K,V> current;
    int expectedModCount;
    
    public final boolean hasNext() {
        return next != null;
    }
    
    final LinkedHashMap.Entry<K,V> nextNode() {
        LinkedHashMap.Entry<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        current = e;
        next = e.after;
        return e;
    }
    
    ...
}

// Map.Entry迭代器：继承LinkedHashMap的迭代器基类，快速失败机制，遍历next指针，返回Map.Entry对象
final class LinkedEntryIterator extends LinkedHashIterator implements Iterator<Map.Entry<K,V>> {
    public final Map.Entry<K,V> next() { return nextNode(); }
}

// LinkedHashMap.Entry#Key迭代器：继承LinkedHashMap的迭代器基类，快速失败机制，遍历next指针，返回LinkedHashMap.Entry#Key对象
final class LinkedKeyIterator extends LinkedHashIterator implements Iterator<K> {
    public final K next() { return nextNode().getKey(); }
}

// LinkedHashMap.Entry#Value迭代器：继承LinkedHashMap的迭代器基类，快速失败机制，遍历next指针，返回LinkedHashMap.Entry#Value对象
final class LinkedValueIterator extends LinkedHashIteratorimplements Iterator<V> {
    public final V next() { return nextNode().value; }
}
```

##### 扩容方法

同HashMap，交由父类HashMap实现。

##### LRU淘汰机制

- **removeEldestEntry（Map.Entry）**：
  - 判断是否需要删除**最近最少访问条目**，该方法默认返回false代表该条目需要保留（此时类似于普通的Map，即永远不会删除最旧的元素），而true则代表该条目应该删除。
  - 在将新条目插入Map后，**put和putAll**会调用此方法，可以为实现者提供了**每次添加新条目时删除最旧条目的机会**。在把映射用作缓存时非常有用，因为可以通过删除陈旧条目来减少内存消耗。

```java
// 判断是否需要删除最近最少访问条目，该方法默认返回false代表该条目需要保留，而true则代表该条目应该删除
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

##### LRU访问机制

- **afterNodeAccess（Node）**：putVal（...）、get（...）、getOrDefault（...）访问结点后回调，移动结点到链表末尾，代表为最近访问。访问顺序为true，代表访问后会移动元素到链表末尾，插入顺序为false，代表不作任何移动，保持为插入顺序。

```java
// 访问结点后回调：移动结点到链表末尾，代表为最近访问
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    
    // 访问顺序为true，代表访问后会移动元素到链表末尾，插入顺序为false，代表不作任何移动，保持为插入顺序
    if (accessOrder && (last = tail) != e) {
        
        // e的备份p，p的前驱e，p的后继a
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        
        // p脱离后继
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;

        // 链接原p的前驱和后继
        if (a != null)
            a.before = b;
        else
            last = b;
        
        // p脱离原来的前驱，链接新的前驱
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        
        // 更新tail指针指向p
        tail = p;
        ++modCount;
    }
}
```

##### 添加元素方法

主要交由父类HashMap实现，其中HashMap提供插入结点后的回调方法（HashMap只做了空实现），便于子类做扩展。

- **afterNodeInsertion（boolean）**：插入结点后回调，put、putAll方法插入结点后调用，其实现主要用于判断是否需要删除最近最少访问的条目，是的话则删除，非常适合用于做缓存。

```java
// 插入结点后回调，evict：为true时代表散列表处于非创建模式中，为false时则散列表处于创建模式中（用于LinkedHashMap结点插入后的回调访问函数，对于HashMap本身没用），只有putMapEntries方法才会为false，表示在创建模式中
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;// 最旧的条目
    if (evict && (first = head) != null 
        // 判断是否需要删除最近最少访问条目，如果为true则根据hash值和key值删除该条目
        && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

##### 删除元素方法

主要交由父类HashMap实现，其中HashMap提供删除结点后的回调方法（HashMap只做了空实现），便于子类做扩展。

- afterNodeRemoval（Node）：脱离元素e，before和after指针链接前后结点。

```java
// 删除结点后回调，脱离元素e，before和after指针链接前后结点
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

##### 获取元素方法

重写了HashMap的get和getOrDefault方法，在其基础上添加了**条目“最近”的访问顺序**: 访问顺序为true，代表访问后会移动元素到链表末尾，插入顺序为false，代表不作任何移动，保持为插入顺序。

- **get（Object）**：获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}。
- **getOrDefault（Object，V）**：带默认值的获取, 如果获取不到key的元素, 则返回默认值。

```java
// 获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}。
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    
    // 访问顺序为true，代表访问后会移动元素到链表末尾，插入顺序为false，代表不作任何移动，保持为插入顺序
    if (accessOrder)
        afterNodeAccess(e);

    return e.value;
}

// 带默认值的获取, 如果获取不到key的元素, 则返回默认值。
public V getOrDefault(Object key, V defaultValue) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return defaultValue;
    
    // 访问顺序为true，代表访问后会移动元素到链表末尾，插入顺序为false，代表不作任何移动，保持为插入顺序
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}

```

#### TreeMap

![1623160630488](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1623160630488.png)

##### 特点

- TreeMap，**基于红黑树的NavigableMap实现，默认按照键值自然排序**，或者根据指定的Comparator进行排序，因此**插入的所有键都必须实现{@code Comparable}接口或者被指定的比较器接受**。{@code containsKey}、{@code get}、{@code put} 和 {@code remove} 操作，都提供有保证的 **log(n)** 时间成本。
- 要注意的是，**如果TreeMap要正确实现Map接口，则要保证维护的排序必须与equals一致**，这是因为Map接口是根据equals来定义的，而TreeMap是使用Comparable#compareTo方法、Comparator#compare方法来执行所有键的比较，因此需要保证比较结果为0时的两个对象也equals。
- **TreeMap是非同步的**，在多个线程并发访问时，通常是通过同步一些自然封装映射的对象来完成，比如
  Collections.synchronizedSortedMap。
- **TreeMap的所有迭代器都是快速失败的**，即如果在创建迭代器后的任何时间对结构进行结构修改，则除了通过迭代器自己的remove方法之外，该迭代器都将抛出{ @link ConcurrentModificationException}，这在面对并发修改时，迭代器可以快速干净地失败，而不是在未来不确定的时间冒着任意、非确定性行为的风险去修改。但是**无法保证迭代器的快速失败行为**，因为一般而言，在存在非同步并发修改的情况下，不可能做出任何硬保证，**只会尽最大努力抛出**ConcurrentModificationException，因此，编写一个依赖于这个异常来保证其正确性的程序是错误的，**快速失败行为只适用于检测错误**。
- NavigableMap接口的方法，返回的是 **{@code Map.Entry} 生成时的映射快照**，因此不支持可选的 {@code Entry#setValue} 方法，但{@code Map#put} 方法可以更改其映射值。

##### 数据结构

![1625374171000](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625374171000.png)

##### 构造方法

- **无参构造函数**：创建一个空的排序映射，根据其键的**自然顺序**排序。
- **指定{@code Comparator} 的构造函数**：创建一个空的排序映射，**根据指定的比较器排序**。
- **指定复制集合的构造函数**：O（n*logn），创建一个新映射（更新所有元素为传入集合的元素），其键值映射与其参数相同，根据键的**自然顺序**排序。
- **指定 {@code SortedMap} 复制集合的构造函数**：O（n），创建一个新的排序映射（即更新所有元素为传入集合的元素），其**键值映射和排序与输入排序映射相同**。

```java
public class TreeMap<K,V> extends AbstractMap<K,V> implements NavigableMap<K,V>, Cloneable, java.io.Serializable
{
	private final Comparator<? super K> comparator;
	private transient Entry<K,V> root;
	private transient int size = 0;
	private transient int modCount = 0;
	
    private static final boolean RED   = false;
    private static final boolean BLACK = true;
    
    static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        Entry<K,V> left;
        Entry<K,V> right;
        Entry<K,V> parent;
        boolean color = BLACK;
        
        Entry(K key, V value, Entry<K,V> parent) {
            this.key = key;
            this.value = value;
            this.parent = parent;
        }
		
		...
    }

	// 无参构造函数：创建一个空的排序映射，根据其键的自然顺序排序。
    public TreeMap() {
        comparator = null;
    }
    
    // 指定{@code Comparator} 的构造函数：创建一个空的排序映射，根据指定的比较器排序。
    public TreeMap(Comparator<? super K> comparator) {
        this.comparator = comparator;
    }
    
    // 指定复制集合的构造函数：创建一个新映射（即更新所有元素为传入集合的元素），其键值映射与其参数相同，根据键的自然顺序排序。
    public TreeMap(Map<? extends K, ? extends V> m) {
        comparator = null;
        putAll(m);// O(n * logn)
    }
 
 	// 指定 {@code SortedMap} 复制集合的构造函数：创建一个新的排序映射（即更新所有元素为传入集合的元素），其键值映射和排序与输入排序映射相同。
    public TreeMap(SortedMap<K, ? extends V> m) {
        comparator = m.comparator();
        try {
            buildFromSorted(m.size(), m.entrySet().iterator(), null, null);// O(n)
        } catch (java.io.IOException cannotHappen) {
        } catch (ClassNotFoundException cannotHappen) {
        }
    }
}

```

##### 构建新树方法

- **buildFromSorted（int，Iterator，ObjectInputStream，V）**：根据传入的迭代器it或者流str的复制元素构造红黑树(**只有最底层的结点才为红结点**)，更新所有元素为传入集合的元素，红黑树结点数目计算O(n)。
  - 取原序列的**中间元素作为根结点**，中间之前的元素作为左子树，中间之后的元素作为右子树。
  - 相关参数：
    - **size**：传入的迭代器或者流中，将要复制的元素数量。
    - **it**：如果传入的迭代器不为null，则从迭代器中获取元素。
    - **str**：如果传入的迭代器为null，则从流中获取元素。
    - **defaultVal**：如果为null，说明没指定默认值，则key和value设置为迭代器/流中的key和value，否则key设置为迭代器/流中的key，value设置为默认值。
    - **level**：递归辅助函数参数，当前子树所在root树中的深度。
    - **lo**：递归辅助函数参数，当前子树第一个元素的索引。
    - **hi**：递归辅助函数参数，当前子树最后第一个元素的索引。
    - **redLevel**：应该为红结点的深度（root树中叶子结点所在的深度）。

```java
// 根据传入的迭代器it或者流str, 复制元素构造红黑树(只有最底层的结点才为红结点), 更新所有元素为传入集合的元素, 红黑树结点数目计算O(n)
private void buildFromSorted(int size, Iterator<?> it,
                             java.io.ObjectInputStream str,
                             V defaultVal)
    throws  java.io.IOException, ClassNotFoundException {
    this.size = size;

    // 根据传入的迭代器it或者流str, 复制元素构造红黑树(只有最底层的结点才为红结点), 红黑树结点数目计算O(n)
    root = buildFromSorted(0, 0, size-1, computeRedLevel(size),
                           it, str, defaultVal);
}

// 计算传入结点高度中红黑树需要的红结点的数目
private static int computeRedLevel(int sz) {
    // 折半从最后一个结点开始计算, 直到根结点
    int level = 0;
    for (int m = sz - 1; m >= 0; m = m / 2 - 1)
        level++;
    return level;
}

// 根据传入的迭代器it或者流str, 复制元素构造红黑树(只有最底层的结点才为红结点)
private final Entry<K,V> buildFromSorted(int level, int lo, int hi,
                                         int redLevel,
                                         Iterator<?> it,
                                         java.io.ObjectInputStream str,
                                         V defaultVal)
    throws  java.io.IOException, ClassNotFoundException {

    // lo当前子树的最小索引, hi最大索引, mid中间索引, left左子树根结点，right右子树根结点
    if (hi < lo) return null;
    int mid = (lo + hi) >>> 1;
    Entry<K,V> left  = null;
    if (lo < mid)
        left = buildFromSorted(level+1, lo, mid - 1, redLevel,
                               it, str, defaultVal);

    // 取原序列的中间元素作为根结点, 中间之前的元素作为左子树, 中间之后的元素作为右子树
    // 如果指定的是迭代器, 则从迭代器中获取数据
    K key;
    V value;
    if (it != null) {
        // 如果没指定默认值, 则key设置为迭代器元素的key, value设置为迭代器元素的value
        if (defaultVal==null) {
            Map.Entry<?,?> entry = (Map.Entry<?,?>)it.next();
            key = (K)entry.getKey();
            value = (V)entry.getValue();
        }
        // 如果指定了默认值, 则key设置为迭代器元素的key, value设置为默认值
        else {
            key = (K)it.next();
            value = defaultVal;
        }
    }
    // 如果指定的是流, 则从流中获取数据
    else { // use stream
        // 如果没指定默认值, 则key设置为按顺序读取流中的key和value, 否则key设置为流中的key, value设置为默认值
        key = (K) str.readObject();
        value = (defaultVal != null ? defaultVal : (V) str.readObject());
    }

    // 构造每棵子树的根结点
    Entry<K,V> middle =  new Entry<>(key, value, null);

    // 最底层的结点设置红色
    if (level == redLevel)
        middle.color = RED;

    // 设置左子树
    if (left != null) {
        middle.left = left;
        left.parent = middle;
    }

    // 设置右子树
    if (mid < hi) {
        Entry<K,V> right = buildFromSorted(level+1, mid+1, hi, redLevel,
                                           it, str, defaultVal);
        middle.right = right;
        right.parent = middle;
    }

    // 返回当前子树的根结点
    return middle;
}

```

##### 迭代方法

- **抽象的TreeMap.Entry迭代器**：TreeMap的迭代器基类，快速失败机制，提供后继遍历和前驱遍历两种方法，返回TreeMap.Entry对象。
- **Map.Entry迭代器**：继承TreeMap的迭代器基类，快速失败机制，只提供后继遍历，返回Map.Entry对象。
- **TreeMap.Entry#Key迭代器**：继承TreeMap的迭代器基类，快速失败机制，只提供Key对象的后继遍历，返回TreeMap.Entry#Key对象。
- **TreeMap.Entry#Value迭代器**：继承TreeMap的迭代器基类，快速失败机制，只提供Value对象的后继遍历，返回TreeMap.Entry#Value对象。

```java
// 抽象的TreeMap.Entry迭代器：TreeMap的迭代器基类，快速失败机制，提供后继遍历和前驱遍历两种方法，返回TreeMap.Entry对象
abstract class PrivateEntryIterator<T> implements Iterator<T> {
    Entry<K,V> next;
    Entry<K,V> lastReturned;
    int expectedModCount;
    
    public final boolean hasNext() {
        return next != null;
    }
    
    // 后继遍历
    final Entry<K,V> nextEntry() {
        Entry<K,V> e = next;
        if (e == null)
            throw new NoSuchElementException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        next = successor(e);
        lastReturned = e;
        return e;
    }

    // 前驱遍历
    final Entry<K,V> prevEntry() {
        Entry<K,V> e = next;
        if (e == null)
            throw new NoSuchElementException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        next = predecessor(e);
        lastReturned = e;
        return e;
    }
    ...
}

// Map.Entry迭代器：继承TreeMap的迭代器基类，快速失败机制，只提供后继遍历，返回Map.Entry对象
final class EntryIterator extends PrivateEntryIterator<Map.Entry<K,V>> {
    EntryIterator(Entry<K,V> first) {
        super(first);
    }
    public Map.Entry<K,V> next() {
        return nextEntry();
    }
}

// TreeMap.Entry#Key迭代器：继承TreeMap的迭代器基类，快速失败机制，只提供Key对象的后继遍历，返回TreeMap.Entry#Key对象
final class KeyIterator extends PrivateEntryIterator<K> {
    KeyIterator(Entry<K,V> first) {
        super(first);
    }
    public K next() {
        return nextEntry().key;
    }
}

// TreeMap.Entry#Value迭代器：继承TreeMap的迭代器基类，快速失败机制，只提供Value对象的后继遍历，返回TreeMap.Entry#Value对象
final class ValueIterator extends PrivateEntryIterator<V> {
    ValueIterator(Entry<K,V> first) {
        super(first);
    }
    public V next() {
        return nextEntry().value;
    }
}

```

##### 扩容方法

基于树的数据结构实现，无需扩容机制。

##### 左旋方法

- **背景**：在结点的添加和删除后，**为了避免子树高度变化，需要通过子树内部调整来保证树达到平衡**，其中2-3-4树是通过结点旋转和结点元素变化实现的，红黑树是通过结点旋转和变色实现。
- **左右左右右**：以x结点作为旋转点进行**左**旋，旋转后，x的**右**结点p成为x的父结点，p原本的**左**结点成为x结点的**右**结点，p原本的**右**结点保持不变。

```java
// 左边的高度比右边的矮, 通过左旋可以增加左边高度
private void rotateLeft(Entry<K,V> p) {
    // 旋转结点p, p的右孩子r
    if (p != null) {
        Entry<K,V> r = p.right;

        // r的左孩子脱钩, 成为p的右孩子
        p.right = r.left;
        if (r.left != null)
            r.left.parent = p;

        // r结点作为p的父结点
        r.parent = p.parent;
        if (p.parent == null)
            root = r;
        else if (p.parent.left == p)
            p.parent.left = r;
        else
            p.parent.right = r;
        r.left = p;
        p.parent = r;
    }
}

```

##### 右旋方法

- **背景**：在结点的添加和删除后，**为了避免子树高度变化，需要通过子树内部调整来保证树达到平衡**，其中2-3-4树是通过结点旋转和结点元素变化实现的，红黑树是通过结点旋转和变色实现。
- **右左右左左**：以x结点作为旋转点进行**右**旋，旋转后，x的**左**结点p成为x的父结点，p原本的**右**结点成为x结点的**左**结点，p原本的**左**结点保持不变。

```java
// 右边的高度比左边的矮, 通过右旋可以增加右边高度
private void rotateRight(Entry<K,V> p) {
    // 旋转结点p, p的左孩子l
    if (p != null) {
        Entry<K,V> l = p.left;

        // l的右孩子脱钩, 成为p的左孩子
        p.left = l.right;
        if (l.right != null)
            l.right.parent = p;

        // l结点作为p的父结点
        l.parent = p.parent;
        if (p.parent == null)
            root = l;
        else if (p.parent.right == p)
            p.parent.right = l;
        else p.parent.left = l;
        l.right = p;
        p.parent = l;
    }
}

```

##### 插入结点后平衡红黑树

- **fixAfterInsertion（Entry）**：
  - **a. 空结点新增**：成为一个2结点，插入前树为null，插入后x需要变黑色，作为根结点。
  - **b. 合并到2结点中**：成为一个3结点，插入前2结点为黑色，插入后无论是（上黑下左红 | 上黑下右红）, 都符合3结点要求，因此无需调整。
  - **c. 合并到3结点中**：成为一个4结点，插入前为3结点（上黑下左红 | 上黑下右红），插入后成为4结点黑红红的情况，根据x插入位置不同分为6种情况：
    - **c.2.1.** 左三(中左左*) ，黑红红，不符合红黑树定义 => 需要调整，则中1右旋，中1变红，左1变黑。
    - **c.2.2.** 中左右*(其实就相当于左三，因为对父结点进行左旋，即得到左三) ，黑红红，不符合红黑树定
      义 => 需要调整，则左1左旋（得到左三），中1右旋，中1变红，新左变黑。
    - **c.2.3.** 右三(中 右右*) ，黑红红，不符合红黑树定义 => 需要调整，则中1左旋，中1变红，右1变黑。
    - **c.2.4.** 中 右左*(其实就相当于右三，因为对父结点进行右旋，即得到右三) 黑红红，不符合红黑树定
      义 => 需要调整，则右1右旋（得到右三），中1左旋，中1变红，新右变黑。
    - **c.2.5.** 中左 右*，黑红 红，符合红黑树定义 => 无需调整。
    - **c.2.6.** 中左* 右，黑红 红，符合红黑树定义 => 无需调整。
  - **d. 合并到4结点中**：成为一个裂变状态（变色后相当于升元了），插入前为4结点（黑红红），插入后4结
    点颜色反转，爷结点成为新的x结点，准备下一轮的向上调整，根据x插入的位置不同分为4种情况：
    - **d.2.1.** 中左左* 右(黑红红 红)，不符合红黑树定义 => 需要调整，则中变红，左1变黑，左2保持为
      红， 右1变黑，中看作为“插入结点”，继续向上调整。
    - **d.2.2.** 中左右* 右(黑红红 红)，不符合红黑树定义 => 需要调整，则中变红，左1变黑，右1保持为
      红，右2变黑，中看作为“插入结点”，继续向上调整。
    - **d.2.3.** 中左 左*右(黑红 红红)，不符合红黑树定义 => 需要调整，则中变红，左1变黑，左2保持为
      红，右1变黑，中看作为“插入结点”，继续向上调整。
    - **d.2.4.** 中左 左右*(黑红 红红)，不符合红黑树定义 => 需要调整，则中变红，左1变黑，右1变黑，右2
      保持为红，中看作为“插入结点”，继续向上调整。

```java
// 插入结点后平衡红黑树
private void fixAfterInsertion(Entry<K,V> x) {
    // 标记插入结点x为红结点
    x.color = RED;

    // 如果x不为root, 且x的父结点parent为红结点时才需要调整
    // x作为调整的逻辑终止条件分析: 1) 如果x为根结点, 则置黑root(即x)相当于把一个4结点成为了3个2结点); 2) 如果x父结点为黑, 此时x相当于插入到了一个2结点无需再作调整
    while (x != null && x != root && x.parent.color == RED) {
        // x的父结点为爷结点的左孩子时, x的叔结点y
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));

            // x的叔结点为红结点, 说明为合并到4结点中: 成为一个裂变状态（变色后相当于升元了）, 插入前为4结点（黑红红）, 插入后4结点颜色反转，爷结点成为新的x结点，准备下一轮的向上调整，根据x插入的位置不同分为: 中左左* 右(d.2.1)、中左右* 右(d.2.2)
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);// x的父结点置黑
                setColor(y, BLACK);// x的叔结点置黑
                setColor(parentOf(parentOf(x)), RED);// x的爷结点置红
                x = parentOf(parentOf(x));// 爷结点成为新的x结点(视为新插入的红结点), 准备下一轮的向上调整, 直到终止条件1或2的发生
            }
            // x的叔结点不为红结点, 说明叔结点不存在 或者 叔结点为黑色(不存在这情况)
            // 而x的叔结点不存在, 说明为合并到3结点中: 成为一个4结点, 插入前为3结点（上黑下左红）,
            // 插入后成为4结点黑红红的情况，根据x插入位置不同分为: 左三(c.2.1)、中左右*(c.2.2)
            else {
                // 如果x为父结点的右孩子时, 说明为 中左右*(c.2.2) 情况, 此时需要先对父结点左旋成 左三(c.2.1) 的情况
                if (x == rightOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateLeft(x);
                }
                // 来到这里, 说明肯定为黑红红 左三(c.2.1) 的情况, 为了使其成为3结点, 需要对爷结点置红, 父结点置黑, 然后爷结点右旋, 让父结点成为中
                setColor(parentOf(x), BLACK);// 父结点置黑
                setColor(parentOf(parentOf(x)), RED);// 爷结点置红
                rotateRight(parentOf(parentOf(x)));// 爷结点右旋, 让父结点成为中
            }
        }
        // x的父结点为爷结点的右孩子时, 叔结点y
        else {
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));

            // x的叔结点为红结点, 说明为合并到4结点中: 成为一个裂变状态（变色后相当于升元了）, 插入前为4结点（黑红红）,
            // 插入后4结点颜色反转，爷结点成为新的x结点，准备下一轮的向上调整，根据x插入的位置不同分为: 中左 右左*(d.2.3)、中左 右右*(d.2.4)
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);// x的父结点置黑
                setColor(y, BLACK);// x的叔结点置黑
                setColor(parentOf(parentOf(x)), RED);// x的爷结点置红
                x = parentOf(parentOf(x));// 爷结点成为新的x结点(视为新插入的红结点), 准备下一轮的向上调整, 直到终止条件1或2的发生
            }
            // x的叔结点不为红结点, 说明叔结点不存在 或者 叔结点为黑色(不存在这情况)
            // 而x的叔结点不存在, 说明为合并到3结点中: 成为一个4结点, 插入前为3结点（上黑下右红）,
            // 插入后成为4结点黑红红的情况，根据x插入位置不同分为: 右三(c.2.3)、中右左*(c.2.4)
            else {
                // 如果x为父结点的左孩子时, 说明为 中右左*(c.2.4) 情况, 此时需要先对父结点右旋成 右三(c.2.3) 的情况
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                // 来到这里, 说明肯定为黑红红 右三(c.2.3) 的情况, 为了使其成为3结点, 需要对爷结点置红, 父结点置黑, 然后爷结点左旋, 让父结点成为中
                setColor(parentOf(x), BLACK);// 父结点置黑
                setColor(parentOf(parentOf(x)), RED);// 爷结点置红
                rotateLeft(parentOf(parentOf(x)));// 爷结点右旋, 让父结点成为中
            }
        }
    }

    // 如果x为root, 或者x的父结点为黑结点时, 不需要调整
    // a. 空结点新增: x成为一个2结点, 作为根结点x变为黑色
    // b. 合并到2结点: x与parent成为一个3结点, 插入前2结点为黑色, 插入符合3结点条件(上黑下左红 | 上黑下右红), 因此直接返回即可
    // c. 合并到3结点中(此时x的父结点为上黑): 成为一个4结点, 插入前为3结点（上黑下红）, 插入后成为4结点黑红红的情况，根据x插入位置不同分为: 中左* 右(c.2.6)、中左 右*(c.2.5)
    root.color = BLACK;
}

```

##### 删除结点前/后平衡红黑树

- **fixAfterDeletion（Entry）**：删除结点前/后平衡红黑树，如果x所在结点为2-3-4树的2结点，则平衡后再删除，如果x所在结点为3结点或者4结点，在平衡前对应的结点就已经删除了，此时x作为该结点的替代结点而保留下来：
  - **x自己搞得定**：
    - 自己搞得定的意思就是，可以在自己结点内部处理完毕（对应2-3-4树结构），不影响其他树的结
      构。
    - **a.1. x为3结点或者4结点的红结点**：直接置黑返回x结点即可调整完毕（因为x是作为替代结点而保留
      下来的），然后交由上层方法删除x结点。
  - **x自己搞不定，兄弟搞得定**：
    - 自己搞不定的意思就是，自身结点为黑结点，如果直接删除会导致父结点所在的树黑色不平衡。
    - 兄弟搞得定的意思就是，**兄弟结点存在多余的子结点**（即兄弟结点为3结点或者4结点），此时，x的
      父结点就可以借出结点下来合并到x结点中，兄弟结点再借出结点合并到父结点中，这样x就可以顺利
      删除了，同时2-3-4树的结构还保持不变。
    - 但是，**前提是x的兄弟结点是真正的兄弟结点，即为黑色的结点**，如果为红色的结点，说明其只是父
      结点（3结点）的红结点，此时需要对父结点进行旋转，以保证x有真正的兄弟结点。
    - **b.1. 兄弟结点为3结点，但无右（左）**：在x在左子树一方时，x的兄弟结点xpr为右子树，如果xpr无
      右孩子在对父结点左旋时，会导致xpr为null，导致2-3-4树的结构不正确，因此，**b.1是一个临时情**
      **况，需要对xpr进行右旋，转换为b.2有右进一步处理**。x为右子树一方时则相反。
    - **b.2. 兄弟结点为3结点，但有右（左）**：在x在左子树一方时，x的兄弟结点xpr为右子树，如果xpr有
      右，则可以顺利地对父结点xp进行左旋。左旋后，在2-3-4树结构看来，xp作为xpr的左孩子（相当于父结点借出去一个结点，合并到x结点中），xpr作为xp的父亲（**相当于兄弟结点借出去一个结点，合**
      **并到父结点中**），xpr借出去的结点颜色为xp借出去的结点颜色，xp借出去的结点颜色一定要为黑色
      （相当于3结点），xpr剩余结点一定要为黑色（相当于叶子结点），返回x结点即可调整完毕，交由
      上层方法删除x结点。x为右子树一方时则相反。
    - **b.3. 兄弟结点为4结点，肯定有右**：在x在左子树一方时，x的兄弟结点xpr为右子树，如果xpr有右，
      则可以顺利地对父结点xp进行左旋。左旋后，在2-3-4树结构看来，xp作为xpr的左孩子（相当于父结
      点借出去一个结点，合并到x结点中），xpr作为xp的父亲（**相当于兄弟结点借出去一个结点，合并到**
      **父结点中，而且还多借出左孩子合并到x结点中**），xpr合并到父结点的颜色为xp借出去的结点颜色
      （而借出去的左孩子本来为红色所以不用变），xp借出去的结点颜色一定要为黑色（相当于4结
      点），xpr剩余结点一定要为黑色（相当于叶子结点），返回x结点即可调整完毕，交由上层方法删除
      x结点。x为右子树一方时则相反。
    - 在b.3中对于兄弟结点为4结点时，兄弟结点可以借出1个结点（需要旋转两次）或者2个结点（只需要
      旋转一次），**在JDK中无论是HashMap还是TreeMap，都选择借出2个结点，因为可以减少花销**。
  - **x自己搞不定，而且兄弟也搞不定**：
    - 自己搞不定的意思就是，自身结点为黑结点，如果直接删除会导致父结点所在的树黑色不平衡。
    - 兄弟也搞不定的意思就是，**兄弟结点也为黑结点，没有多余的子结点**，如果直接删除x，则导致叔结
      点所在路径多了一个黑色结点，造成黑色不平衡。
    - **c.1. 兄弟结点为2结点**：此时，为了让x能够顺利删除，**兄弟结点需要置红（自损）**，这样x在删除
      后，x父结点所在树还是黑色平衡的。但是，如果x父结点为黑色，x爷结点所在树则不黑色平衡了
      （因为父结点这边少了一个黑色结点），所以父结点的叔结点要也要被置红。**因此需要一路向上自**
      **损，直到碰到任意一个终止条件即可结束**：
      - **自损的终止条件1（向上碰到根结点）**：经过一路置红叔结点（置红叔结点是没问题的，因为出
        现该情况是叶子结点为3结点黑黑黑的时候，此时如果叔结点没有孩子结点即为黑色，而对于更
        上层的叔结点来说，貌似不会出现叔为黑红红这种情况），直到循环到根结点时（因为上面已
        经没有父节点了），则代表自损完毕，此时整棵树都是黑色平衡的了（都减少了一个黑色结
        点）。
      - **自损的终止条件2（向上碰到红结点）**：如果碰到红色结点时，只需要把该结点置黑，则不需要
        在置红叔结点了，此时相当于在父结点这边子树补回了一个黑色结点，而不影响叔结点那边子
        树的黑色结点数目，因此整棵树还是黑色平衡的。

```java
// 删除结点前/后平衡红黑树
private void fixAfterDeletion(Entry<K,V> x) {
    // 如果x不为root, 且x为黑结点时才需要调整
    while (x != root && colorOf(x) == BLACK) {
        // x为父结点的左孩子时, 兄弟结点sib
        if (x == leftOf(parentOf(x))) {
            Entry<K,V> sib = rightOf(parentOf(x));

            // 如果兄弟结点为红色, 说明不是真正的兄弟结点(只是父结点的红结点), 需要对父结点进行左旋, 左旋x出现了真正的兄弟结点, 原父结点位置被假的兄弟结点占据
            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);// 置黑假的兄弟结点(因为将要成为x的爷结点)
                setColor(parentOf(x), RED);// 置红x的父结点(设置为原兄弟结点的红色, 因为将要把原父结点3结点样式从中右改成中左)
                rotateLeft(parentOf(x));// 左旋父结点(将要把原父结点3结点样式从中右改成中左)
                sib = rightOf(parentOf(x));// sib更新为x真正的兄弟结点
            }
            // sib为x真正的兄弟结点, 如果兄弟结点没有可用的结点(2结点c.1时), 则 x设置为父结点, 继续向上自损, 置红叔结点, 直到根结点或者红结点
            if (colorOf(leftOf(sib))  == BLACK &&
                colorOf(rightOf(sib)) == BLACK) {
                setColor(sib, RED);// 自损, 置红叔结点
                x = parentOf(x);// x设置为父结点, 继续向上自损, 直到根结点或者红结点
            }
            // sib为x真正的兄弟结点, 如果兄弟结点有可用的孩子结点(3结点b.1、b.2或者4结点b.3时)
            else {
                // 如果叔结点不存在右结点, 说明为3结点无右b.1的情况, 为一个临时情况, 需要对叔结点进行右旋, 右转后成为有右b.2情况
                if (colorOf(rightOf(sib)) == BLACK) {
                    setColor(leftOf(sib), BLACK);// 置黑叔结点的左孩子(因为无右那肯定有左, 此时置黑为了成为父结点)
                    setColor(sib, RED);// 置红叔结点(为了成为有右的右结点)
                    rotateRight(sib);// 右旋叔结点
                    sib = rightOf(parentOf(x));// sib更新为最新的叔结点指针
                }
                // 到这里肯定有右即3结点b.2和4结点b.3的情况, 此时借出两个结点(最多), 则置叔为父结点颜色, 父结点置黑, 叔右结点置黑, 左旋父结点,
                // 让原父结点成为x和叔左结点的父结点, 叔结点站上原父结点的位置, 叔右结点保持为右孩子(2结点), 黑黑黑
                setColor(sib, colorOf(parentOf(x)));// 置叔为父结点颜色(因为将要站上父结点的位置)
                setColor(parentOf(x), BLACK);// 父结点置黑(因为左旋后借出去的父结点要成为3结点或者4结点的黑结点)
                setColor(rightOf(sib), BLACK);// 叔右结点置黑(因为原父结点置黑了, 叔右结点为黑(2结点), 才能保持黑黑黑的结构)
                rotateLeft(parentOf(x));// 左旋父结点
                x = root;// 设置x为根结点, 退出循环, 停止红黑树调整
            }
        }
        // x为父结点的右孩子时, 兄弟结点sib
        else { // symmetric // 对称的
            Entry<K,V> sib = leftOf(parentOf(x));

            // 如果兄弟结点为红色, 说明不是真正的兄弟结点(只是父结点的红结点), 需要对父结点进行右旋, 右旋x出现了真正的兄弟结点, 原父结点位置被假的兄弟结点占据
            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);// 置黑假的兄弟结点(因为将要成为x的爷结点)
                setColor(parentOf(x), RED);// 置红x的父结点(设置为原兄弟结点的红色, 因为将要把原父结点3结点样式从中左改成中右)
                rotateRight(parentOf(x));// 左旋父结点(将要把原父结点3结点样式从中左改成中右)
                sib = leftOf(parentOf(x));// sib更新为x真正的兄弟结点
            }
            // sib为x真正的兄弟结点, 如果兄弟结点没有可用的结点(2结点c.1时), 则 x设置为父结点, 继续向上自损, 置红叔结点, 直到根结点或者红结点
            if (colorOf(rightOf(sib)) == BLACK &&
                colorOf(leftOf(sib)) == BLACK) {
                setColor(sib, RED);// 自损, 置红叔结点
                x = parentOf(x);// x设置为父结点, 继续向上自损, 直到根结点或者红结点
            }
            // sib为x真正的兄弟结点, 如果兄弟结点有可用的孩子结点(3结点b.1、b.2或者4结点b.3时)
            else {
                // 如果叔结点不存在左结点, 说明为3结点无左b.1的情况, 为一个临时情况, 需要对叔结点进行左旋, 左转后成为有左b.2情况
                if (colorOf(leftOf(sib)) == BLACK) {
                    setColor(rightOf(sib), BLACK);// 置黑叔结点的左孩子(因为无左那肯定有右, 此时置黑为了成为父结点)
                    setColor(sib, RED);// 置红叔结点(为了成为有左的左结点)
                    rotateLeft(sib);// 左旋叔结点
                    sib = leftOf(parentOf(x));// sib更新为最新的叔结点指针
                }
                // 到这里肯定有左即3结点b.2和4结点b.3的情况, 此时借出两个结点(最多), 则置叔为父结点颜色, 父结点置黑, 叔右结点置黑, 右旋父结点,
                // 让原父结点成为x和叔右结点的父结点, 叔结点站上原父结点的位置, 叔左结点保持为左孩子(2结点), 黑黑黑
                setColor(sib, colorOf(parentOf(x)));// 置叔为父结点颜色(因为将要站上父结点的位置)
                setColor(parentOf(x), BLACK);// 父结点置黑(因为右旋后借出去的父结点要成为3结点或者4结点的黑结点)
                setColor(leftOf(sib), BLACK);// 叔左结点置黑(因为原父结点置黑了, 叔左结点为黑(2结点), 才能保持黑黑黑的结构)
                rotateRight(parentOf(x));// 右旋父结点
                x = root;// 设置x为根结点, 退出循环, 停止红黑树调整
            }
        }
    }

    // a. 自损终止条件打成: 碰到根结点或者红结点, 此时置黑x结点, 完成自损
    // b. 非自损调整完毕后, x被手动设置成了root结点, 表示需要结束调整, 此时对root结点置黑没有任何作用(因为root本身就是黑的), 只是为了重用代码而已
    setColor(x, BLACK);
}

```

##### 添加元素方法

- **put（K，V）**：将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则会替换旧值。
- **putIfAbsent（K，V）**：Map#putIfAbsent实现的默认方法，底层调用get和put方法。将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值。
- **putAll（Map）**：**根据复制集合元素构建新树/增量添加复制集合元素**，如果复制集合为SortedMap类型, 且比较器与当前的比较器不等, 则构建新树, O(n)；如果复制集合不为SortedMap类型, 或者比较器与当前的比较器相等，则增量添加元素，O(n * logn)。

```java
// 将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则会替换旧值
public V put(K key, V value) {
    Entry<K,V> t = root;

    // 如果root结点为null, 则构建root结点, 更新实际大小, 插入成功返回null
    if (t == null) {
        compare(key, key);// 如果没有指定比较器, 则使用键(已实现Comparable)来比较, 否则使用比较器比较
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }

    // 比较结果cmp, 父结点parent, 比较器cpr, 临时结点t
    int cmp;
    Entry<K,V> parent;
    Comparator<? super K> cpr = comparator;

    // 如果指定了比较器cpr, 则使用cpr比较, cmp小于0则遍历左子树并更新parent指针, cmp大于0则遍历右子树并更新parent指针, cmp等于0则替换结点的值并返回，O（logn）
    if (cpr != null) {
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    // 如果没有指定比较器cpr, 则使用键(已实现Comparable)来比较比较, cmp小于0则遍历左子树并更新parent指针, cmp大于0则遍历右子树并更新parent指针, cmp等于0则替换结点的值并返回
    else {
        if (key == null)
            throw new NullPointerException();
        Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }

    // 构建新结点, cmp小于0则设置/替换parent的左孩子, 否则说明大于0, 则设置/替换parent的右孩子
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;

    // 插入结点后平衡红黑树, 更新实际大小, 插入成功返回null
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}

// Map#putIfAbsent实现的默认方法， 底层调用get和put方法。将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值
default V putIfAbsent(K key, V value) {
    V v = get(key);
    if (v == null) {
        v = put(key, value);
    }

    return v;
}

// 根据复制集合元素构建新树/增量添加复制集合元素，如果复制集合为SortedMap类型, 且比较器与当前的比较器不等, 则构建新树, O(n); 如果复制集合不为SortedMap类型, 或者比较器与当前的比较器相等, 则增量添加元素, O(n * logn)
public void putAll(Map<? extends K, ? extends V> map) {
    int mapSize = map.size();

    // 如果复制集合为SortedMap类型, 且比较器与当前的比较器不等, 则构建新树, O(n)
    if (size==0 && mapSize!=0 && map instanceof SortedMap) {
        Comparator<?> c = ((SortedMap<?,?>)map).comparator();
        if (c == comparator || (c != null && c.equals(comparator))) {
            ++modCount;
            try {
                // 根据传入的迭代器it或者流str, 复制元素构造红黑树(只有最底层的结点才为红结点), 更新所有元素为传入集合的元素, 红黑树结点数目计算O(n)
                buildFromSorted(mapSize, map.entrySet().iterator(),
                                null, null);
            } catch (java.io.IOException cannotHappen) {
            } catch (ClassNotFoundException cannotHappen) {
            }
            return;
        }
    }

    // 如果复制集合不为SortedMap类型, 或者比较器与当前的比较器相等, 则增量添加元素, 复制集合遍历 + 二叉搜索树添加: O(n * logn)
    super.putAll(map);
}
// AbstractMap#putAll实现的默认方法， 底层调用put方法。将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则会替换旧值
public void putAll(Map<? extends K, ? extends V> m) {
    for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
        put(e.getKey(), e.getValue());
}
```

##### 删除元素方法

- **remove（Object）**：如果key存在，则从此映射中删除该key对应的映射，忽略value值。
- **remove（Object，Object）**：Map#remove实现的默认方法，删除key和value都equals的键值对，value值必须匹配。

```java
// 如果key存在，则从此映射中删除该key对应的映射，忽略value值。
public V remove(Object key) {
    Entry<K,V> p = getEntry(key);
    if (p == null)
        return null;

    V oldValue = p.value;
    deleteEntry(p);// 删除节点 p，然后重新平衡树
    return oldValue;
}
// 删除节点 p，然后重新平衡树
private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;

    // 如果p同时存在左右孩子, 则寻找后继s, 交换s和p, 此后p指向后继s结点
    if (p.left != null && p.right != null) {
        Entry<K,V> s = successor(p);
        p.key = s.key;
        p.value = s.value;
        p = s;
    } // p has 2 children

    // 如果p只有一个孩子, 或者没有孩子, 或者交换到了后继位置, 则选择替代结点: 如果此时p存在左孩子, 则替代结点为左孩子, 否则为右孩子
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);

    // 如果替代结点不为null, 说明p至少有一个孩子, 即p为非叶子结点为3结点或者4结点, 则p的父结点链接替代结点, 父结点脱钩p
    if (replacement != null) {
        replacement.parent = p.parent;
        if (p.parent == null)
            root = replacement;
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        else
            p.parent.right = replacement;

        // p脱钩父结点与孩子结点
        p.left = p.right = p.parent = null;

        // 这里p为3结点或者4结点, 如果p为黑结点, 则需要删除结点后平衡红黑树
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    }
    // 如果替代结点为null, 说明p没有孩子, 但p的父结点为null, 说明p为根结点, 直接置空根结点即可
    else if (p.parent == null) {
        root = null;
    }
    
    // 如果替代结点为null, 说明p没有孩子, 且p也不是根结点, 说明p为叶子结点
    else { 
        // 这里p为2结点, 如果p为黑结点, 则需要删除结点前平衡红黑树
        if (p.color == BLACK)
            fixAfterDeletion(p);

        // 红黑树平衡之后, 父结点脱钩p, p脱钩父结点
        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}

// Map#remove实现的默认方法，删除key和value都equals的键值对，value值必须匹配。
default boolean remove(Object key, Object value) {
    Object curValue = get(key);
    if (!Objects.equals(curValue, value) ||
        (curValue == null && !containsKey(key))) {
        return false;
    }
    remove(key);
    return true;
}
```

##### 获取元素方法

- **get（Object）**：获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}。底层使用比较器或者作为Comparable接口实现，进行二叉搜索查找**O(logn)**。
- **getOrDefault（Object，V）**：Map#getOrDefault实现的默认方法，底层依赖get和containsKey方法。带默认值的获取，如果获取不到key的元素，则返回默认值。

```java
// 获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}

// Map#getOrDefault实现的默认方法，底层依赖get和containsKey方法。带默认值的获取，如果获取不到key的元素, 则返回默认值
default V getOrDefault(Object key, V defaultValue) {
    V v;
    return (((v = get(key)) != null) || containsKey(key))
        ? v
        : defaultValue;
}
// 使用比较器或者作为Comparable接口实现，进行二叉搜索查找O(logn)
final Entry<K,V> getEntry(Object key) {
    if (comparator != null)
        // 使用比较器根据key值对进行二叉搜索查找, O(logn)
        return getEntryUsingComparator(key);

    if (key == null)
        throw new NullPointerException();

    // key作为Comparable接口实现, 根据key值对进行二叉搜索查找, O(logn)
    Comparable<? super K> k = (Comparable<? super K>) key;
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}
// 使用比较器根据key值对进行二叉搜索查找, O(logn)
final Entry<K,V> getEntryUsingComparator(Object key) {
    @SuppressWarnings("unchecked")
    K k = (K) key;
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = cpr.compare(k, p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
    }
    return null;
}

```

##### 实现NavigableMap接口方法

###### 获取小于Key的最大键方法

- **lowerEntry（K）**：返回小于Key的最大键的Entry，如果没有这样的键，则返回 {@code null}。返回的是 {@code Map.Entry} 生成时的**映射快照**，不支持setValue方法。
- **lowerKey（K）**：返回小于Key的最大键，如果没有这样的键，则返回 {@code null}。

```java
// 返回小于Key的最大键的Entry，如果没有这样的键，则返回 {@code null}
public Map.Entry<K,V> lowerEntry(K key) {
    return exportEntry(getLowerEntry(key));
}
// 返回小于指定键的最大键的条目； 如果不存在这样的条目（即树中的最小键大于指定的键），则返回 {@code null}
final Entry<K,V> getLowerEntry(K key) {
    // 指定键值key, 当前键值p, 比较结果cmp(为key - p的结果, 大于0说明key比p大, 小于0说明key比p小)
    Entry<K,V> p = root;
    while (p != null) {
        // 如果没有指定比较器, 则使用键(已实现Comparable)来比较, 否则使用比较器比较
        int cmp = compare(key, p.key);

        // 如果key > p, 则查找右子树, 如果达到右子树的叶子结点还没找到, 则返回最大的p
        if (cmp > 0) {
            if (p.right != null)
                p = p.right;
            else
                return p;
        }
        // 如果key <= p, 且p存在左孩子, 则继续查找左子树, 如果不存在左孩子, 说明碰到比key大的p了, 需要向上找p的父结点
        else {
            // 如果p存在左孩子, 说明存在比p小一点的键, 而由于key < p, 所以还需要找到再小一点的p, 则遍历左子树
            if (p.left != null) {
                p = p.left;
            }
            // 如果p不存在左孩子, 说明没有比p小一点的键了, 而由于key < p, 所以还需要找到再小一点的p, 则返回p的前驱(一定比key小)
            else {
                Entry<K,V> parent = p.parent;
                Entry<K,V> ch = p;
                while (parent != null && ch == parent.left) {
                    ch = parent;
                    parent = parent.parent;
                }
                return parent;
            }
        }
    }

    // 找不到则返回null
    return null;
}
// 返回的是 {@code Map.Entry} 生成时的映射快照，不支持setValue方法
static <K,V> Map.Entry<K,V> exportEntry(TreeMap.Entry<K,V> e) {
    return (e == null) ? null :
    new AbstractMap.SimpleImmutableEntry<>(e);
}

// 返回小于Key的最大键，如果没有这样的键，则返回 {@code null}
public K lowerKey(K key) {
    return keyOrNull(getLowerEntry(key));
}
// 返回输入键，如果为空则为空
static <K,V> K keyOrNull(TreeMap.Entry<K,V> e) {
    return (e == null) ? null : e.key;
}


```

###### 获取小于等于Key的最大键方法

- **floorEntry（K）**：返回小于或者等于Key的最大键的Entry，如果没有这样的键，则返回 {@code null}。返回的是 {@code Map.Entry} 生成时的**映射快照**，不支持setValue方法。
- **floorKey（K）**：返回小于或者等于Key的最大键，如果没有这样的键，则返回 {@code null}。

```java
// 返回小于或者等于Key的最大键的Entry，如果没有这样的键，则返回 {@code null}    
public Map.Entry<K,V> floorEntry(K key) {
    return exportEntry(getFloorEntry(key));
}
// 返回小于指定键的最大键的条目； 如果不存在这样的条目（即树中的最小键大于指定的键），则返回 {@code null}
final Entry<K,V> getFloorEntry(K key) {
    // 指定键值key, 当前键值p, 比较结果cmp(为key - p的结果, 大于0说明key比p大, 小于0说明key比p小)
    Entry<K,V> p = root;
    while (p != null) {
        // 如果没有指定比较器, 则使用键(已实现Comparable)来比较, 否则使用比较器比较
        int cmp = compare(key, p.key);

        // 如果key > p, 则查找右子树, 如果达到右子树的叶子结点还没找到, 则返回最大的p
        if (cmp > 0) {
            if (p.right != null)
                p = p.right;
            else
                return p;
        }
        // 如果key < p, 且p存在左孩子, 则继续查找左子树, 如果不存在左孩子, 说明碰到比key大的p了, 需要向上找p的父结点
        else if (cmp < 0) {
            // 如果p存在左孩子, 说明存在比p小一点的键, 而由于key < p, 所以还需要找到再小一点的p, 则遍历左子树
            if (p.left != null) {
                p = p.left;
            }
            // 如果p不存在左孩子, 说明没有比p小一点的键了, 而由于key < p, 所以还需要找到再小一点的p, 则返回p的前驱(一定比key小)
            else {
                Entry<K,V> parent = p.parent;
                Entry<K,V> ch = p;
                while (parent != null && ch == parent.left) {
                    ch = parent;
                    parent = parent.parent;
                }
                return parent;
            }
        }
        // 如果key = p, 返回p
        else
            return p;

    }

    // 找不到则返回null
    return null;
}

// 返回小于或者等于Key的最大键，如果没有这样的键，则返回 {@code null}
public K floorKey(K key) {
    return keyOrNull(getFloorEntry(key));
}

```

###### 获取大于等于key的最小键方法

- **ceilingEntry（K）**：返回大于或者等于Key的最小键的Entry，如果没有这样的键，则返回 {@code null}。返回的是 {@code Map.Entry} 生成时的**映射快照**，不支持setValue方法。
- **ceilingKey（K）**：返回大于或者等于Key的最小键，如果没有这样的键，则返回 {@code null}。

```java
// 返回大于或者等于Key的最小键的Entry，如果没有这样的键，则返回 {@code null}
public Map.Entry<K,V> ceilingEntry(K key) {
    return exportEntry(getCeilingEntry(key));
}
// 获取指定键对应的条目； 如果不存在这样的条目，则返回大于指定键的最小键的条目； 如果不存在这样的条目（即树中最大的键小于指定的键），则返回 {@code null}
final Entry<K,V> getCeilingEntry(K key) {
    // 指定键值key, 当前键值p, 比较结果cmp(为key - p的结果, 大于0说明key比p大, 小于0说明key比p小)
    Entry<K,V> p = root;
    while (p != null) {
        // 如果没有指定比较器, 则使用键(已实现Comparable)来比较, 否则使用比较器比较
        int cmp = compare(key, p.key);

        // 如果key < p, 则查找左子树, 如果达到左子树的叶子结点还没找到, 则返回最小的p
        if (cmp < 0) {
            if (p.left != null)
                p = p.left;
            else
                return p;
        }
        // 如果key > p, 且p存在右孩子, 则继续查找右子树, 如果不存在右孩子, 说明碰到比key小的p了, 需要向上找p的父结点
        else if (cmp > 0) {
            // 如果p存在右孩子, 说明存在比p大一点的键, 而由于key > p, 所以还需要找到再大一点的p, 则遍历右子树
            if (p.right != null) {
                p = p.right;
            }
            // 如果p不存在右孩子, 说明没有比p大一点的键了, 而由于key > p, 所以还需要找到再大一点的p, 则返回p的后继(一定比key大)
            else {
                Entry<K,V> parent = p.parent;
                Entry<K,V> ch = p;
                while (parent != null && ch == parent.right) {
                    ch = parent;
                    parent = parent.parent;
                }
                return parent;
            }
        }
        // 如果key = p, 返回p
        else
            return p;
    }

    // 找不到则返回null
    return null;
}

// 返回大于或者等于Key的最小键，如果没有这样的键，则返回 {@code null}
public K ceilingKey(K key) {
    return keyOrNull(getCeilingEntry(key));
}


```

###### 获取大于key的最小键方法

- **higherEntry（K）**：返回大于Key的最小键的Entry，如果没有这样的键，则返回 {@code null}。返回的是 {@code Map.Entry} 生成时的**映射快照**，不支持setValue方法。
- **higherKey（K）**：返回大于Key的最小键，如果没有这样的键，则返回 {@code null}。

```java
// 返回大于Key的最小键的Entry，如果没有这样的键，则返回 {@code null}
public Map.Entry<K,V> higherEntry(K key) {
    return exportEntry(getHigherEntry(key));
}
// 获取大于指定键的最小键的条目； 如果不存在这样的条目，则返回大于指定键的最小键的条目； 如果不存在这样的条目，则返回 {@code null}
final Entry<K,V> getHigherEntry(K key) {
    // 指定键值key, 当前键值p, 比较结果cmp(为key - p的结果, 大于0说明key比p大, 小于0说明key比p小)
    Entry<K,V> p = root;
    while (p != null) {
        // 如果没有指定比较器, 则使用键(已实现Comparable)来比较, 否则使用比较器比较
        int cmp = compare(key, p.key);

        // 如果key < p, 则查找左子树, 如果达到左子树的叶子结点还没找到, 则返回最小的p
        if (cmp < 0) {
            if (p.left != null)
                p = p.left;
            else
                return p;
        }
        // 如果key >= p, 且p存在右孩子, 则继续查找右子树, 如果不存在右孩子, 说明碰到比key小的p了, 需要向上找p的父结点
        else {
            // 如果p存在右孩子, 说明存在比p大一点的键, 而由于key > p, 所以还需要找到再大一点的p, 则遍历右子树
            if (p.right != null) {
                p = p.right;
            }
            // 如果p不存在右孩子, 说明没有比p大一点的键了, 而由于key > p, 所以还需要找到再大一点的p, 则返回p的后继(一定比key大)
            else {
                Entry<K,V> parent = p.parent;
                Entry<K,V> ch = p;
                while (parent != null && ch == parent.right) {
                    ch = parent;
                    parent = parent.parent;
                }
                return parent;
            }
        }
    }

    // 找不到则返回null
    return null;
}

// 返回大于Key的最小键，如果没有这样的键，则返回 {@code null}
public K higherKey(K key) {
    return keyOrNull(getHigherEntry(key));
}
```

###### 获取最小键方法

- **firstEntry（）**：返回最小键的Entry，如果条目为空，则返回 {@code null}。
- **firstKey（）**：返回最小键，如果没有这样的键，则抛出异常。
- **pollFirstEntry（）**：删除并返回最小键的Entry，如果条目为空，则返回 {@code null}。

```java
// 返回最小键的Entry，如果条目为空，则返回 {@code null}
public Map.Entry<K,V> firstEntry() {
    return exportEntry(getFirstEntry());
}
// 返回TreeMap中的第一个 Entry（根据TreeMap的键排序函数）。如果 TreeMap 为空，则返回 null
final Entry<K,V> getFirstEntry() {
    Entry<K,V> p = root;
    if (p != null)
        while (p.left != null)
            p = p.left;
    return p;
}

// 返回最小键，如果没有这样的键，则抛出异常
public K firstKey() {
    return key(getFirstEntry());
}
// 返回指定元素的键值，如果元素为null则抛出异常
static <K> K key(Entry<K,?> e) {
    if (e==null)
        throw new NoSuchElementException();
    return e.key;
}

// 删除并返回最小键的Entry，如果条目为空，则返回 {@code null}
public Map.Entry<K,V> pollFirstEntry() {
    Entry<K,V> p = getFirstEntry();
    Map.Entry<K,V> result = exportEntry(p);
    if (p != null)
        deleteEntry(p);
    return result;
}
```

###### 获取最大键方法

- **lastEntry（）**：返回最大键的Entry，如果条目为空，则返回 {@code null}。
- **lastKey（）**：返回最大键，如果没有这样的键，则抛出异常。
- **pollLastEntry（）**：删除并返回最大键的Entry，如果条目为空，则返回 {@code null}。

```java
// 返回最大键的Entry，如果条目为空，则返回 {@code null}
public Map.Entry<K,V> lastEntry() {
    return exportEntry(getLastEntry());
}
// 返回TreeMap中的最后一个 Entry（根据TreeMap的键排序函数）。如果TreeMap为空，则返回 null
final Entry<K,V> getLastEntry() {
    Entry<K,V> p = root;
    if (p != null)
        while (p.right != null)
            p = p.right;
    return p;
}

// 返回最大键，如果没有这样的键，则抛出异常。
public K lastKey() {
    return key(getLastEntry());
}
// 返回指定元素的键值，如果元素为null则抛出异常
static <K> K key(Entry<K,?> e) {
    if (e==null)
        throw new NoSuchElementException();
    return e.key;
}

// 删除并返回最大键的Entry，如果条目为空，则返回 {@code null}
public Map.Entry<K,V> pollLastEntry() {
    Entry<K,V> p = getLastEntry();
    Map.Entry<K,V> result = exportEntry(p);
    if (p != null)
        deleteEntry(p);
    return result;
}
```

###### NavigableSubMap抽象类

```java
// TreeMap#NavigableSubMap，导航视图抽象类
abstract static class NavigableSubMap<K,V> extends AbstractMap<K,V> implements NavigableMap<K,V>, java.io.Serializable {
    final TreeMap<K,V> m;// 全元素Map m
    final K lo, hi;// 指定的低位键lo，高位键hi
    final boolean fromStart, toEnd;// 是否绝对从头/尾开始
    final boolean loInclusive, hiInclusive;// 边界是否包含键值本身

    NavigableSubMap(TreeMap<K,V> m,
                    boolean fromStart, K lo, boolean loInclusive,
                    boolean toEnd,     K hi, boolean hiInclusive) {
        if (!fromStart && !toEnd) {
            if (m.compare(lo, hi) > 0)
                throw new IllegalArgumentException("fromKey > toKey");
        } else {
            if (!fromStart) // type check
                m.compare(lo, lo);
            if (!toEnd)
                m.compare(hi, hi);
        }

        this.m = m;
        this.fromStart = fromStart;
        this.lo = lo;
        this.loInclusive = loInclusive;
        this.toEnd = toEnd;
        this.hi = hi;
        this.hiInclusive = hiInclusive;
    }
    ...
}
```

###### 获取逆序视图方法

- **descendingMap（）**：返回Map的**逆序（降序）**视图。底层实现使用**父类的前驱迭代方法**，即取前驱作为next结点。

```java
// 返回Map的逆序（降序）视图
public NavigableMap<K, V> descendingMap() {
    NavigableMap<K, V> km = descendingMap;
    return (km != null) ? km :
    // 绝对从头/尾开始，没有指定键，包含边界
    (descendingMap = new DescendingSubMap<>(this,
                                            true, null, true,
                                            true, null, true));
}
// TreeMap#DescendingSubMap：逆序子视图
static final class DescendingSubMap<K,V>  extends NavigableSubMap<K,V> {
    DescendingSubMap(TreeMap<K,V> m,
                     boolean fromStart, K lo, boolean loInclusive,
                     boolean toEnd,     K hi, boolean hiInclusive) {
        super(m, fromStart, lo, loInclusive, toEnd, hi, hiInclusive);
    }
    
    Iterator<K> keyIterator() {
        return new DescendingSubMapKeyIterator(absHighest(), absLowFence());
    }
    ...
}

// TreeMap#NavigableSubMap#DescendingSubMapKeyIterator：NavigableSubMap内部逆序Key迭代器
final class DescendingSubMapKeyIterator extends SubMapIterator<K> implements Spliterator<K> {
    
    // 依赖父类的key迭代方法：取前驱的key
    public K next() {
        return prevEntry().key;
    }
    ...
}

// TreeMap#NavigableSubMap#SubMapIterator：NavigableSubMap内部子视图迭代器
abstract class SubMapIterator<T> implements Iterator<T> {
    TreeMap.Entry<K,V> lastReturned;
    TreeMap.Entry<K,V> next;
    final Object fenceKey;
    int expectedModCount;
    
    // 前驱迭代方法：取前驱作为next
    final TreeMap.Entry<K,V> prevEntry() {
        TreeMap.Entry<K,V> e = next;
        if (e == null || e.key == fenceKey)
            throw new NoSuchElementException();
        if (m.modCount != expectedModCount)
            throw new ConcurrentModificationException();
        next = predecessor(e);
        lastReturned = e;
        return e;
    }
    ...
}

```

###### 获取中间视图方法

底层实现**使用父类的后继迭代方法**，即取后继作为next结点，因此返回的视图是**顺序**的。

- **subMap（K，boolean，K，boolean）**：返回Map的中间视图，键从fromKey（不含）到toKey（不含）的部分视图，fromInclusive为true代表包含fromKey，toInclusive为true代表包含toKey。
- **subMap（K，K）**：返回Map的中间视图，即键从fromKey（fromInclusive含）到toKey（!toInclusive不含）的部分视图，两者相等时返回null(除非fromInclusive与toInclusive都为true)。

```java
// 返回Map的中间视图：键从fromKey（不含）到toKey（不含）的部分视图，fromInclusive为true代表包含fromKey，toInclusive为true代表包含toKey
public NavigableMap<K,V> subMap(K fromKey, boolean fromInclusive,
                                K toKey,   boolean toInclusive) {
    // 不从头/尾开始，指定fromKey和toKey，指定是否包含边界
    return new AscendingSubMap<>(this,
                                 false, fromKey, fromInclusive,
                                 false, toKey,   toInclusive);
}

// 返回Map的中间视图：即键从fromKey（fromInclusive含）到toKey（!toInclusive不含）的部分视图，两者相等时返回null(除非fromInclusive与toInclusive都为true)
public SortedMap<K,V> subMap(K fromKey, K toKey) {
    // 不从头/尾开始，指定fromKey和toKey，指定包含下边界不包含上边界
    return subMap(fromKey, true, toKey, false);
}

// TreeMap#AscendingSubMap：顺序子视图
static final class AscendingSubMap<K,V> extends NavigableSubMap<K,V> {
    AscendingSubMap(TreeMap<K,V> m,
                    boolean fromStart, K lo, boolean loInclusive,
                    boolean toEnd,     K hi, boolean hiInclusive) {
        super(m, fromStart, lo, loInclusive, toEnd, hi, hiInclusive);
    }
    
    Iterator<K> keyIterator() {
        return new SubMapKeyIterator(absLowest(), absHighFence());
    }
    ...
}

// TreeMap#NavigableSubMap#DescendingSubMapKeyIterator：NavigableSubMap内部顺序Key迭代器
final class SubMapKeyIterator extends SubMapIterator<K> implements Spliterator<K> {
    // 依赖父类的key迭代方法：取后继的key
    public K next() {
        return nextEntry().key;
    }
    ...
}

// TreeMap#NavigableSubMap#SubMapIterator：NavigableSubMap内部子视图迭代器
abstract class SubMapIterator<T> implements Iterator<T> {
    TreeMap.Entry<K,V> lastReturned;
    TreeMap.Entry<K,V> next;
    final Object fenceKey;
    int expectedModCount;
    
    // 后继迭代方法：取后继作为next
    final TreeMap.Entry<K,V> nextEntry() {
        TreeMap.Entry<K,V> e = next;
        if (e == null || e.key == fenceKey)
            throw new NoSuchElementException();
        if (m.modCount != expectedModCount)
            throw new ConcurrentModificationException();
        next = successor(e);
        lastReturned = e;
        return e;
    }
    ...
}
```

###### 获取头部视图方法

底层实现**使用父类的后继迭代方法**，即取后继作为next结点，因此返回的视图是**顺序**的。

- **headMap（K，boolean）**：返回Map的头部视图，即键小于toKey(不含)的部分视图，inclusive为true代表包含toKey。
- **headMap（K）**：返回Map的头部视图，即键小于toKey(!inclusive不含)的部分视图。

```java
// 返回Map的头部视图：即键小于toKey(不含)的部分视图，inclusive为true代表包含toKey
public NavigableMap<K,V> headMap(K toKey, boolean inclusive) {
    // 从头不从尾开始，指定toKey，指定包含下边界
    return new AscendingSubMap<>(this,
                                 true,  null,  true,
                                 false, toKey, inclusive);
}

// 返回Map的头部视图：即键小于toKey(!inclusive不含)的部分视图
public SortedMap<K,V> headMap(K toKey) {
    // 从头不从尾开始，指定toKey，指定包含下边界与不包含上边界
    return headMap(toKey, false);
}
```

###### 获取尾部视图方法

底层实现**使用父类的后继迭代方法**，即取后继作为next结点，因此返回的视图是**顺序**的。

- **tailMap（K，boolean）**：返回Map的尾部视图，即键大于fromKey（不含）的部分视图，inclusive为true代表包含fromKey。
- **tailMap（K）**：返回Map的尾部视图，即键大于或者等于fromKey（inclusive含）的部分视图。

```java
// 返回Map的尾部视图：即键大于fromKey（不含）的部分视图，inclusive为true代表包含fromKey
public NavigableMap<K,V> tailMap(K fromKey, boolean inclusive) {
    // 不从头从尾开始，指定fromKey，指定包含上边界
    return new AscendingSubMap<>(this,
                                 false, fromKey, inclusive,
                                 true,  null,    true);
}

// 返回Map的尾部视图：即键大于或者等于fromKey（inclusive含）的部分视图
public SortedMap<K,V> tailMap(K fromKey) {
    // 不从头从尾开始，指定fromKey，指定包含上边界与下边界
    return tailMap(fromKey, true);
}

```

#### WeakHashMap

![1623679998528](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1623679998528.png)

##### 特点

- WeakHashMap，**Map接口基于弱键的哈希表实现**，其条目会在键"不再正常使用时"**自动删除**。与HashMap类似，**支持null值和null键**，具有相同定义和大小的初始容量和负载因子。
- 一些熟悉的Map不变量不适用于WeakHashMap，因为WeakHashMap的Key对象都被间接存储为**弱引用**所指的对象，**弱引用“引用”的Key对象随时会被垃圾收集器回收**，然后其上的弱引用会被清除，WeakhashMap会删除该键，对应的条目也会从映射中删除。
- WeakHashMap表现得好像存在一个未知线程正在默默地删除条目一样，即随着时间推移，size方法可能返回更小的值，isEmpty方法返回false然后返回true，containsKey方法返回先为真后为假，get方法先返回值但后返回null等等。
- WeakHashMap中的**值对象由普通的强引用持有**，值对象如果直接或者间接强引用了键，可以防止键被丢弃。
- **WeakHashMap是非同步的**，在多个线程并发访问时，通常是通过同步一些自然封装映射的对象来完成，比如Collections.synchronizedMap。
- **WeakHashMap的所有迭代器都是快速失败的**，即如果在创建迭代器后的任何时间对结构进行结构修改，则除了通过迭代器自己的remove方法之外，该迭代器都将抛出{ @link ConcurrentModificationException}，这在面对并发修改时，迭代器可以快速干净地失败，而不是在未来不确定的时间冒着任意、非确定性行为的风险去修改。但是**无法保证迭代器的快速失败行为**，因为一般而言，在存在非同步并发修改的情况下，不可能做出任何硬保证，**只会尽最大努力抛出**ConcurrentModificationException，因此，编写一个依赖于这个异常来保证其正确性的程序是错误的，**快速失败行为只适用于检测错误**。

##### 数据结构

![1625375845586](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625375845586.png)

##### 构造方法

- **空参的构造方法**：使用默认初始容量(16) 和 默认负载因子(0.75) 构造一个新的空WeakHashMap(而HashMap的无参构造方法的初始容量是在resize方法中初始化的)。
- **指定初始容量的构造方法**：使用给定的初始容量和默认加载因子(0.75)构造一个新的空WeakHashMap。
- **指定初始容量和负载因子的构造方法**：使用给定的初始容量（会计算最接近的2^n）和给定的负载因子构造一个新的空 WeakHashMap，以及计算出阈值（容量 * 负载因子）。
- **指定复制集合的构造方法**：构造一个与指定映射具有相同映射关系的新WeakHashMap，使用默认负载因子 (0.75) 和足以容纳指定映射中的初始容量，同时会删除每个桶链上已被清除弱引用的Entry条目。

```java
public class WeakHashMap<K,V> extends AbstractMap<K,V> implements Map<K,V> {
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    private static final int MAXIMUM_CAPACITY = 1 << 30;
    private static final float DEFAULT_LOAD_FACTOR = 0.75f;
    Entry<K,V>[] table;
    private int size;
    private int threshold;
    private final float loadFactor;
    private final ReferenceQueue<Object> queue = new ReferenceQueue<>();
    int modCount;
    private static final Object NULL_KEY = new Object();
    private transient Set<Map.Entry<K,V>> entrySet;
  
    // 将null键的内部表示作为null返回给调用者。
    static Object unmaskNull(Object key) {
        return (key == NULL_KEY) ? null : key;
    }
    
    // Entry就是一个WeakReference, 没有Entry.key, 实际上是通过Entry --> Key弱引用Key对象, 通过WeakReference#get()来获取真实Key对象的强引用
    private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
        V value;
        final int hash;
        Entry<K,V> next;
        
        Entry(Object key, V value,
              ReferenceQueue<Object> queue,
              int hash, Entry<K,V> next) {
            super(key, queue);// Entry"弱引用“Key对象(table[i] -> Entry --> Key)
            this.value = value;
            this.hash  = hash;
            this.next  = next;
        }
        
        public K getKey() {
            return (K) WeakHashMap.unmaskNull(get());
        }
    }
    
    // 使用默认初始容量(16)和负载因子(0.75)构造一个新的空WeakHashMap(而HashMap的无参构造方法的初始容量是在resize方法中初始化的)
    public WeakHashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }
    
    // 使用给定的初始容量和默认加载因子(0.75)构造一个新的空WeakHashMap
    public WeakHashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    
    // 使用给定的初始容量和给定的负载因子构造一个新的空 WeakHashMap。
    public WeakHashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Initial Capacity: "+
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;

        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal Load factor: "+
                                               loadFactor);

        // 扩容到接近指定初始容量的2^n
        int capacity = 1;
        while (capacity < initialCapacity)
            capacity <<= 1;

        // 构造指定容量的散列表数组
        table = newTable(capacity);
        this.loadFactor = loadFactor;

        // 计算阈值
        threshold = (int)(capacity * loadFactor);
    }
    // 初始化散列表数组
    private Entry<K,V>[] newTable(int n) {
        return (Entry<K,V>[]) new Entry<?,?>[n];
    }

    // 构造一个与指定映射具有相同映射关系的新WeakHashMap，使用默认负载因子 (0.75) 和足以容纳指定映射中的初始容量
    public WeakHashMap(Map<? extends K, ? extends V> m) {
        // 取计算得到的容量((复制集合的实际大小 / 默认负载因子0.75) + 1) 和 默认容量16之间的最大值作为初始容量, 取默认负载因子0.75
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                DEFAULT_INITIAL_CAPACITY),
             DEFAULT_LOAD_FACTOR);

        // 遍历指定复制集合所有的条目, 并添加到WeakHashMap的散列表中: 添加前如果复制集合的大小大于当前阈值, 则会先对散列表进行扩容, 同时删除每个桶链上已被清除弱引用的Entry条目, 注意的是扩容后不一定会使用新容量的散列表(只有当删除的条目减少到大于阈值一半时才会)
        putAll(m);
    }
}
```

##### 迭代方法

- **抽象的WeakHashMap.Entry迭代器**：WeakHashMap的迭代器基类，快速失败机制，遍历next指针，返回WeakHashMap.Entry对象。
- **Map.Entry迭代器**：继承WeakHashMap的迭代器基类，快速失败机制，遍历next指针，返回Map.Entry对象。
- **WeakHashMap.Entry#getKey()迭代器**：继承WeakHashMap的迭代器基类，快速失败机制，遍历next指针，返回WeakHashMap.Entry#getKey()对象。
- **WeakHashMap.Entry#Value迭代器**：继承WeakHashMap的迭代器基类，快速失败机制，遍历next指针，返回WeakHashMap.Entry#Value对象。

```java
// 抽象的WeakHashMap.Entry迭代器：WeakHashMap的迭代器基类，快速失败机制，遍历next指针，返回WeakHashMap.Entry对象
private abstract class HashIterator<T> implements Iterator<T> {
    private int index;
    private Entry<K,V> entry;
    private Entry<K,V> lastReturned;
    private int expectedModCount = modCount;
    private Object nextKey;// 需要强引用以避免 hasNext 和 next 之间的键消失
    private Object currentKey;// 需要强引用以避免 nextEntry() 和条目的任何使用之间的键消失
    
    public boolean hasNext() {
        Entry<K,V>[] t = table;
        while (nextKey == null) {
            Entry<K,V> e = entry;
            int i = index;
            while (e == null && i > 0)
                e = t[--i];// 往回找桶索引
            entry = e;
            index = i;
            if (e == null) {
                currentKey = null;
                return false;
            }
            nextKey = e.get(); // hold on to key in strong ref
            if (nextKey == null)
                entry = entry.next;
        }
        return true;
    }
    
    protected Entry<K,V> nextEntry() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (nextKey == null && !hasNext())
            throw new NoSuchElementException();

        lastReturned = entry;
        entry = entry.next;
        currentKey = nextKey;
        nextKey = null;
        return lastReturned;
    }
}

// Map.Entry迭代器：继承WeakHashMap的迭代器基类，快速失败机制，遍历next指针，返回Map.Entry对象
private class EntryIterator extends HashIterator<Map.Entry<K,V>> {
    public Map.Entry<K,V> next() {
        return nextEntry();
    }
}

// WeakHashMap.Entry#getKey()迭代器：继承WeakHashMap的迭代器基类，快速失败机制，遍历next指针，返回WeakHashMap.Entry#getKey()对象
private class KeyIterator extends HashIterator<K> {
    public K next() {
        return nextEntry().getKey();
    }
}
// Entry#getKey()方法: 获取Key对象的强引用
public K getKey() {
    return (K) WeakHashMap.unmaskNull(get());
}

// WeakHashMap.Entry#Value迭代器：继承WeakHashMap的迭代器基类，快速失败机制，遍历next指针，返回WeakHashMap.Entry#Value对象
private class ValueIterator extends HashIterator<V> {
    public V next() {
        return nextEntry().value;
    }
}

```

##### HashCode扰动函数

- **hash（Object）**：HashCode扰动函数，把高位的特征和低位的特征组合起来计算键的哈希值，尽量做到任何一位的变化都能对最终得到的结果产生影响，降低哈希冲突的概率。

```java
// HashCode扰动函数: 尽量做到任何一位的变化都能对最终得到的结果产生影响, 降低哈希冲突的概率
final int hash(Object k) {
    int h = k.hashCode();

    // 为了把高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}

```

##### 计算哈希索引方法

- **（n - 1） & hash**：相当于hash % n，n指散列表的当前容量（桶数量），hash指获取hash（key）即key的
  hashCode扰动后结果。
  - **不能直接使用HashCode作为索引的原因**：int类型的hashCode，范围为[-2^32，2^32 - 1]，如果散列表
    数组与HashCode一一对应，**需要40亿的空间（int[40亿]），明显这在内存是放不下的**，也就是说明
    hashCode是不能直接作为数组索引的。因此，如果使用hashCode对散列表数组长度取模，那么就可以解决这个问题，从而保证较小的数组也还能利用上hashCode。
  - **n为2的幂次原因**：为了解决hashCode对散列表数组长度取模，设计了HashCode的扰动函数以及为2幂次的容量n，可以通过n-1来获得取模操作的低位掩码，此时只需要通过低位掩码与扰动后的
    **hashCode（hash值）进行一次与运算**，即可得到该hash值在散列表数组中的索引。其次通过在扩容方
    法中，经过hash值与低位掩码相与，可以保证扩容后，**只会移动少部分相与结果高位为1的桶链表**，其他
    保持不变，减少了扩容时的时间。

```java
// 返回哈希值h在散列表数组中的索引
private static int indexFor(int h, int length) {
    return h & (length-1);
}

```

##### 规范容量计算方法

- **newCapacity <<= 1**：循环左移1位，直到不小于目标容量，这样做的目的**计算出最接近当前容量的2^n**，规范化容量，而阈值则是扩容后再乘以负载因子。该代码在构造方法和putAll（）中执行。

```java
// 计算出最接近当前容量的2^n
int newCapacity = table.length;
while (newCapacity < targetCapacity)
    newCapacity <<= 1;

```

##### 扩容方法

- **resize（int）**：指定容量进行散列表扩容并移动条目，从表中删除桶链上已被清除弱引用的Entry条目后，如果**散列表实际大小减小到阈值一半**后，则继续使用旧散列表，否则使用新散列表并重新计算阈值。
  - **扩容条件**：当**实际大小大于阈值**时，则计算新容量传入（最接近实际大小的2^n）进行扩容，且如果使用新的散列表还需要根据新容量重新计算阈值（**不同于HashMap的阈值，该阈值很可能小于当前容量，不会作为下一个容量**）。

```java
// 指定容量进行散列表扩容并移动条目: 从表中删除桶链上已被清除弱引用的Entry条目后, 如果散列表实际大小减小到阈值一半后, 则继续使用旧散列表, 否则使用新散列表并重新计算阈值
void resize(int newCapacity) {
    // 从表中删除桶链上已被清除弱引用的Entry条目后, 返回散列表数组
    Entry<K,V>[] oldTable = getTable();

    // 如果容量达到极限, 则计算最大阈值
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    // 构造指定容量的新散列表数组
    Entry<K,V>[] newTable = newTable(newCapacity);

    // 把所有条目从oldTable数组移动到newTable数组(e.hash重新对数组取模), 同时清除每个桶链上已被清除弱引用的Entry条目
    transfer(oldTable, newTable);

    // 使用新容量的散列表
    table = newTable;
    
    // 如果清除陈旧条目后的实际大小 大于等于 当前阈值的一半, 说明散列表中至少有一半阈值的真实元素, 此时确认是要使用新容量的散列表了, 则需要准备下一个阈值, 根据指定的容量计算阈值
    if (size >= threshold / 2) {
        threshold = (int)(newCapacity * loadFactor);
    }
    // 如果清除陈旧条目后的实际大小 小于 当前阈值的一半, 说明散列表中一半阈值的真实元素都没有, 即之前清除过一批弱引用的Entry条目, 也就是以前的散列表还是可以装得下的(因为没超过一般的阈值), 此时还是继续使用旧的散列表, 把原本的条目移动回旧的散列表
    else {
        // 从表中删除桶链上的陈旧条目（已被清除弱引用的Entry条目）
        expungeStaleEntries();

        // 把所有条目从newTable数组还原到oldTable数组(e.hash重新对数组取模), 同时清除每个桶链上已被清除弱引用的Entry条目
        transfer(newTable, oldTable);

        // 继续使用旧的散列表
        table = oldTable;
    }
}
// 从表中删除桶链上已被清除弱引用的Entry条目后, 返回散列表数组
private Entry<K,V>[] getTable() {
    // 从表中删除桶链上的陈旧条目（已被清除弱引用的Entry条目）
    expungeStaleEntries();

    // 从表中删除桶链上已被清除弱引用的Entry条目后, 返回散列表数组
    return table;
}
// 将所有条目从src数组传输到dest数组(hash重新对数组取模), 同时清除每个桶链上已被清除弱引用的Entry条目
private void transfer(Entry<K,V>[] src, Entry<K,V>[] dest) {
    // 遍历老散列表数组, 当前遍历的条目e
    for (int j = 0; j < src.length; ++j) {
        Entry<K,V> e = src[j];

        // 清空e的原散列表上的桶
        src[j] = null;

        // 遍历e桶上的链表, 当前遍历结点e
        while (e != null) {
            Entry<K,V> next = e.next;

            // 如果e与Key之间的"弱引用"已经被清除, 也即Key对象被清除了, 说明e已经无效了, 则删除链表上的Entry e结点
            Object key = e.get();
            if (key == null) {
                e.next = null;  // Help GC
                e.value = null; //  "   "
                size--;
            }
            // 如果e与Key之间的"弱引用"还没被清除, 也即Key对象还没被清除, 说明e还是有效的, 则计算e.hash在新散列表数组中的索引, 进行头插法
            else {
                int i = indexFor(e.hash, dest.length);
                e.next = dest[i];// 头插, 后继链接到原来的链头(HashMap JDK7是头插, JDK8是尾插)
                dest[i] = e;// 头插(HashMap JDK7是头插, JDK8是尾插)
            }

            // 继续遍历当前链表
            e = next;
        }
    }
}

```

##### 删除陈旧条目方法

- **expungeStaleEntries（）**：从表中删除桶链上的陈旧条目（已被清除弱引用的Entry条目），在getTable（）和 size（）中被调用。
  - **要注意的Value对象内存泄露问题**：table[i] -> Entry --> Key，由于Key只被Entry"弱引用"，当Key被清除后，通过WeakReference#get（）获取真实Key强引用为null，相当于Entry.key==null了（虽然WeakHashMap的Entry没有key属性）。此时如果不删除Entry.value以及让Entry脱钩table[i]，则会存在**Value对象内存泄露问题**（因为key被置null了，在外界看来Value应该是被清空了的, 但实际上Value却还在）。因此还需要从引用队列获取条目Entry e进而删除该条目。

```java
// 从表中删除桶链上已被清除弱引用的Entry条目: 
private void expungeStaleEntries() {
    for (Object x; (x = queue.poll()) != null; ) {
        synchronized (queue) {
            Entry<K,V> e = (Entry<K,V>) x;

            // 重新取模条目e.hash与散列表长度, 计算条目e在散列表数组中的索引, 来获取e所在的桶链表prev
            int i = indexFor(e.hash, table.length);
            Entry<K,V> prev = table[i];
            Entry<K,V> p = prev;

            // 遍历table[i]桶链, p为当前遍历的结点, prev作为p的前驱
            while (p != null) {
                Entry<K,V> next = p.next;

                // 如果当前结点p为在引用队列获取到的条目, 说明遍历找到了e条目
                if (p == e) {
                    // 如果条目e是链头结点, 则next成为新的链头
                    if (prev == e)
                        table[i] = next;
                    // 如果条目e不是链头结点, 则链接p的前驱prev与next结点
                    else
                        prev.next = next;

                    // Must not null out e.next;
                    // stale entries may be in use by a HashIterator

                    // 链接e的前驱和后继完毕, 则清空e的value, 更新实际大小, 但不得将e.next置为空, 因为HashIterator可能正在使用陈旧的条目(桶链上已被清除弱引用的Entry条目)
                    e.value = null; // Help GC
                    size--;

                    // 退出循环, 此时Entry e已经没有了原来table[i]桶链的结点引用了, 也就是条目e被删除了, 很快该Entry就会被回收掉(但是e#next还在, 但是不影响e的回收)
                    break;
                }

                // 如果当前结点p不为在引用队列获取到的条目, 说明还没找到了e条目, 则更新prev和p指针, 继续往后找
                prev = p;
                p = next;
            }
        }
    }
}

```

##### 获取最新散列表数组方法

- **getTable（）**：从表中删除桶链上陈旧条目（已被清除弱引用的Entry条目）后，返回散列表数组。 该方法被get（Object）、getEntry（Object）、put（K，V）、resize（int）、remove（Object）、containsValue（Object）、containsNullValue（Object）、forEach（BiConsumer）、replaceAll（BiFunction）方法调用。其中，getEntry（Object）被constainsKey（Object）方法调用。

```java
// 从表中删除桶链上陈旧条目（已被清除弱引用的Entry条目）后，返回散列表数组
private Entry<K,V>[] getTable() {
    // 从表中删除桶链上的陈旧条目（已被清除弱引用的Entry条目）
    expungeStaleEntries();
    return table;
}

```

##### 添加元素方法

- **put（K，V）**：将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则会替换旧值。
- **putIfAbsent（K，V）**：Map#putIfAbsent实现的默认方法， 底层调用get和put方法。将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值。
- **putAll（Map）**：遍历指定复制集合所有的条目, 并添加到WeakHashMap的散列表中，添加前如果复制集合的大小大于当前阈值, 则会先对散列表进行扩容，**同时删除每个桶链上已被清除弱引用的Entry条目**，注意的是扩容后不一定会使用新容量的散列表(只有当删除的条目减少到大于阈值一半时才会)。

```java
// 将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则会替换旧值。
public V put(K key, V value) {
    // 如果键为null，则使用NULL_KEY作为键。
    Object k = maskNull(key);

    // HashCode扰动函数: 尽量做到任何一位的变化都能对最终得到的结果产生影响, 降低哈希冲突的概率
    int h = hash(k);

    // 从表中删除桶链上陈旧条目（已被清除弱引用的Entry条目）后，返回散列表数组
    Entry<K,V>[] tab = getTable();

    // 返回哈希值h在散列表数组中的索引
    int i = indexFor(h, tab.length);

    // 计算出的索引对应的桶e, 遍历e链表, 通过Entry#get()获取Key对象的强引用, 使用==和Object.equals比较Key对象是否相等
    for (Entry<K,V> e = tab[i]; e != null; e = e.next) {
        // 如果Entry.hash相等, 且Key对象相等, 则说明存在相同键的Entry, 此时需要替换条目, 即替换旧值, 返回旧值
        if (h == e.hash && eq(k, e.get())) {
            V oldValue = e.value;
            if (value != oldValue)
                e.value = value;
            return oldValue;
        }
    }

    modCount++;

    // 如果不存在相同键的Entry, 说明需要添加Entry, 此时构造传入Key、Value对象的Entry结点, 并插入到tab[i]桶的桶头中(HashMap JDK7是头插, JDK8是尾插), 原来的桶头e成为next结点
    Entry<K,V> e = tab[i];
    tab[i] = new Entry<>(k, value, queue, h, e);

    // 添加后, 如果实际大小大于阈值, 则指定容量(2倍实际大小)进行散列表扩容: 从表中删除桶链上已被清除弱引用的Entry条目后, 如果散列表实际大小减小到阈值一半后, 则继续使用旧散列表, 否则使用新散列表并重新计算阈值
    if (++size >= threshold)
        resize(tab.length * 2);
    return null;
}

// Map#putIfAbsent实现的默认方法， 底层调用get和put方法。将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值
default V putIfAbsent(K key, V value) {
    V v = get(key);
    if (v == null) {
        v = put(key, value);
    }

    return v;
}

// 遍历指定复制集合所有的条目, 并添加到WeakHashMap的散列表中，添加前如果复制集合的大小大于当前阈值, 则会先对散列表进行扩容，同时删除每个桶链上已被清除弱引用的Entry条目，注意的是扩容后不一定会使用新容量的散列表(只有当删除的条目减少到大于阈值一半时才会)
public void putAll(Map<? extends K, ? extends V> m) {
    // 如果复制集合的大小大于当前阈值, 则先计算新的容量(最接近targetCapacity的2^n), 而targetCapacity=复制集合实际大小 * 负载因子
    int numKeysToBeAdded = m.size();
    if (numKeysToBeAdded == 0) return;
    if (numKeysToBeAdded > threshold) {
        int targetCapacity = (int)(numKeysToBeAdded / loadFactor + 1);
        if (targetCapacity > MAXIMUM_CAPACITY)
            targetCapacity = MAXIMUM_CAPACITY;

        // 扩容到接近当前容量的2^n
        int newCapacity = table.length;
        while (newCapacity < targetCapacity)
            newCapacity <<= 1;

        // 如果新容量确实变大了, 则指定新容量(最接近targetCapacity的2^n)进行散列表扩容: 从表中删除桶链上已被清除弱引用的Entry条目后,
        // 如果散列表实际大小减小到阈值一半后, 则继续使用旧散列表, 否则使用新散列表并重新计算阈值
        if (newCapacity > table.length)
            resize(newCapacity);
    }

    // 遍历指定复制集合所有的条目, 并添加到WeakHashMap的散列表中
    for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
        put(e.getKey(), e.getValue());
}

```

##### 删除元素方法

- **remove（Object）**：如果key存在，则从此映射中删除该key对应的映射，忽略value值。
- **remove（Object，Object）**：Map#remove实现的默认方法，删除key和value都equals的键值对，value值必须匹配。

```java
// 如果key存在，则从此映射中删除该key对应的映射，忽略value值。
public V remove(Object key) {
    // 如果键为null，则使用 NULL_KEY 作为键。
    Object k = maskNull(key);

    // HashCode扰动函数: 尽量做到任何一位的变化都能对最终得到的结果产生影响, 降低哈希冲突的概率
    int h = hash(k);

    // 从表中删除桶链上陈旧条目（已被清除弱引用的Entry条目）后，返回散列表数组
    Entry<K,V>[] tab = getTable();

    // 返回哈希值h在散列表数组中的索引
    int i = indexFor(h, tab.length);

    // 计算出的索引对应散列表数组中的桶e
    Entry<K,V> prev = tab[i];
    Entry<K,V> e = prev;

    // 遍历e桶链表, e为当前链表遍历的结点, prev为e的前驱, next为e的后继
    while (e != null) {
        Entry<K,V> next = e.next;

        // 如果找到hash相等, 且Key对象也相等的结点, 说明e结点就是要删除的结点, 则脱钩e结点
        if (h == e.hash && eq(k, e.get())) {
            modCount++;
            size--;

            // 如果e为桶头结点, 则脱离桶头结点
            if (prev == e)
                tab[i] = next;
            // 如果e不为桶头结点, 则脱钩e结点
            else
                prev.next = next;

            // 返回e结点的value值
            return e.value;
        }

        // 如果还没找到要删除的结点, 则交换prev、e指针, 继续遍历链表
        prev = e;
        e = next;
    }

    // 如果最后都找不到要删除的结点, 则返回null
    return null;
}

// Map#remove实现的默认方法，删除key和value都equals的键值对，value值必须匹配。
default boolean remove(Object key, Object value) {
    Object curValue = get(key);
    if (!Objects.equals(curValue, value) ||
        (curValue == null && !containsKey(key))) {
        return false;
    }
    remove(key);
    return true;
}

```

##### 获取元素方法

- **get（Object）**：获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}。
- **getOrDefault（Object，V）**：Map#getOrDefault实现的默认方法，底层依赖get和containsKey方法。带默认值的获取，如果获取不到key的元素，则返回默认值。

```java
// 获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}
public V get(Object key) {
    // 如果键为null，则使用 NULL_KEY 作为键。
    Object k = maskNull(key);

    // HashCode扰动函数: 尽量做到任何一位的变化都能对最终得到的结果产生影响, 降低哈希冲突的概率
    int h = hash(k);

    // 从表中删除桶链上陈旧条目（已被清除弱引用的Entry条目）后，返回散列表数组
    Entry<K,V>[] tab = getTable();

    // 返回哈希值h在散列表数组中的索引
    int index = indexFor(h, tab.length);

    // 计算出的索引对应散列表数组中的桶e
    Entry<K,V> e = tab[index];

    // 遍历e桶链表, e为当前链表遍历的结点, 如果找到hash相等, 且Key对象也相等的结点, 说明e结点就是要找的结点, 此时返回e的value值
    while (e != null) {
        if (e.hash == h && eq(k, e.get()))
            return e.value;
        e = e.next;
    }

    // 如果最后都找不到要删除的结点, 则返回null
    return null;
}

// Map#getOrDefault实现的默认方法，底层依赖get和containsKey方法。带默认值的获取，如果获取不到key的元素, 则返回默认值
default V getOrDefault(Object key, V defaultValue) {
    V v;
    return (((v = get(key)) != null) || containsKey(key))
        ? v
        : defaultValue;
}
```

#### SynchronizedMap

![1623848363009](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1623848363009.png)

##### 特点

- java.util.Collections内部类。
- 实现Map接口，拥有成员变量Map的引用和mutex对象锁，用于包装传入的集合，使其**所有方法都能成为同步方法（加synchronized关键字）**。
- 没有实现同步的Entry、Key和Value迭代器，需要用户实现。

##### 构造方法

- **只传入Map集合的构造函数**：设置Map引用，持有父类mutex对象锁。
- **传入Map集合与对象锁的构造函数**：设置Map引用与对象锁引用。

```java
private static class SynchronizedMap<K,V> implements Map<K,V>, Serializable {
    private final Map<K,V> m;
    final Object      mutex;
    
    // 只传入Map集合的构造函数
    SynchronizedMap(Map<K,V> m) {
        this.m = Objects.requireNonNull(m);
        mutex = this;
    }
    
	// 传入Map集合与对象锁的构造函数
    SynchronizedMap(Map<K,V> m, Object mutex) {
        this.m = m;
        this.mutex = mutex;
    }
    ...
}
```

##### 迭代方法

没做实现，需要用户实现。

##### 扩容方法

所有方法都是获取mutex对象锁后，调用引用Map的方法，所以无独立实现的扩容机制。

##### 添加元素方法

所有方法都是只做了mutex对象锁的获取，其实现交由引用的Map集合。

- **put（K，V）**：将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则会替换旧值。
- **putIfAbsent（K，V）**：将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值。
- **putAll（Map）**：添加复制集合的的所有键值对。

```java
// 将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则会替换旧值
public V put(K key, V value) {
    synchronized (mutex) {return m.put(key, value);}
}

// 将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值
public V putIfAbsent(K key, V value) {
    synchronized (mutex) {return m.putIfAbsent(key, value);}
}

// 添加复制集合的的所有键值对
public void putAll(Map<? extends K, ? extends V> map) {
    synchronized (mutex) {m.putAll(map);}
}
```

##### 删除元素方法

所有方法都是只做了mutex对象锁的获取，其实现交由引用的Map集合。

- **remove（Object）**：如果key存在，则从此映射中删除该key对应的映射，忽略value值。
- **remove（Object，Object）**：删除key和value都equals的键值对，value值必须匹配。

```java
// 如果key存在，则从此映射中删除该key对应的映射，忽略value值
public V remove(Object key) {
    synchronized (mutex) {return m.remove(key);}
}

// 删除key和value都equals的键值对，value值必须匹配
public boolean remove(Object key, Object value) {
    synchronized (mutex) {return m.remove(key, value);}
}
```

##### 获取元素方法

所有方法都是只做了mutex对象锁的获取，其实现交由引用的Map集合。

- **get（Object）**：获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}。
- **getOrDefault（Object，V）**：带默认值的获取, 如果获取不到key的元素, 则返回默认值。

```java
// 获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}
public V get(Object key) {
    synchronized (mutex) {return m.get(key);}
}

// 带默认值的获取, 如果获取不到key的元素, 则返回默认值
public V getOrDefault(Object k, V defaultValue) {
    synchronized (mutex) {return m.getOrDefault(k, defaultValue);}
}
```

#### HashTable

![1623855125622](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1623855125622.png)

##### Dictionary抽象类

- Dictionary类可以将键映射到值，每个键和每个值都是一个对象，且每个键最多与一个值相关联，任何**非null**对象都可以用作键和值。
- 通常，Dictionary类的实现应使用**equals**方法来确定两个键是否相同。
- 注意：Dictionary类已经过时， 新的实现应该实现 Map 接口，而不是扩展这个类。
- Enumeration接口提供了方法来枚举向量的元素、哈希表的键和哈希表中的值，实现Enumeration接口的对象可以一次生成一系列元素，对nextElement方法的连续调用将返回系列的**连续元素**。
- 注意：Enumeration接口的功能由Iterator接口复制。此外，Iterator 添加了一个可选的删除操作，并具有较短的方法名称。新的实现应该考虑使用Iterator而不是Enumeration。

```java
// Dictionary相当于新版的Map接口
public abstract class Dictionary<K,V> {
    public Dictionary() { }
	abstract public int size();
    abstract public boolean isEmpty();
    abstract public Enumeration<K> keys();// Enumeration相当于新版的Iterator
    abstract public Enumeration<V> elements();
    abstract public V get(Object key);
    abstract public V put(K key, V value);// 键和值都不能为null
    abstract public V remove(Object key);
}
```

##### 特点

- HashTable，实现了一个哈希表，将键映射到值，任何**非null对象**都可以用作键或值。
- Hashtable的实例有两个影响其性能的参数：初始容量和负载因子。 
  - **容量**：哈希表中的桶数。
  - **初始容量**：哈希表创建时的容量，请注意哈希表是开放的：在“哈希冲突”的情况下，单个存储桶存储多个条目，必须顺序搜索。 
  - **负载因子**：衡量哈希表在其容量自动增加之前允许散列表达到多满的指标。
- **默认负载因子（0.75）** 在时间和空间成本之间提供了很好的权衡，较高的值会减少空间开销，但会增加查找条目的时间成本（这反映在大多数 Hashtable 操作中，包括 get 和 put）。
- **初始容量（11）**控制浪费空间和需要重新哈希操作之间的权衡（重新哈希操作是个非常耗时的操作）， 如果初始容量大于Hashtable将包含的最大条目数除以其负载因子，则不会发生重新哈希操作。但是，将初始容量设置得太高会浪费空间。如果要将许多条目放入Hashtable，创建具有足够大容量的条目，可以比让它根据需要执行自动重新散列来更有效地插入条目以增加表。
- **{@code Hashtable} 是同步的**，如果不需要线程安全的实现，建议使用 {@link HashMap} 代替 {@code Hashtable}， 如果需要线程安全的高并发实现，则建议使用 {@link java.util.concurrent.ConcurrentHashMap} 代替 {@code Hashtable}。
- **Hashtable的所有迭代器都是快速失败的**，即如果在创建迭代器后的任何时间对结构进行结构修改，则除了通过迭代器自己的remove方法之外，该迭代器都将抛出{ @link ConcurrentModificationException}，这在面对并发修改时，迭代器可以快速干净地失败，而不是在未来不确定的时间冒着任意、非确定性行为的风险去修改。但是**无法保证迭代器的快速失败行为**，因为一般而言，在存在非同步并发修改的情况下，不可能做出任何硬保证，**只会尽最大努力抛出**ConcurrentModificationException，因此，编写一个依赖于这个异常来保证其正确性的程序是错误的，**快速失败行为只适用于检测错误**。但**Hashtable 的keys()和elements()返回的Enumeration不是快速失败的**。

##### 数据结构

![1625376138026](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625376138026.png)

##### 构造方法

- **空参的构造方法**：构造一个具有默认初始容量 (11) 和负载因子 (0.75) 的新的空哈希表。
- **指定初始容量的构造方法**：使用指定的初始容量和默认加载因子 (0.75) 构造一个新的空哈希表。
- **指定初始容量和负载因子的构造方法**：使用指定的初始容量和指定的负载因子构造一个新的空哈希表。
- **指定复制集合的构造方法**：构造一个与给定 Map 具有相同映射的新哈希表。 哈希表的初始容量足以容纳给定Map中的映射和默认负载因子 (0.75)。

```java
public class Hashtable<K,V> extends Dictionary<K,V> implements Map<K,V>, Cloneable, java.io.Serializable {
    private transient Entry<?,?>[] table;
    private transient int count;
    private int threshold;
    private float loadFactor;
    private transient int modCount = 0;
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    private static class Entry<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Entry<K,V> next;
        
        protected Entry(int hash, K key, V value, Entry<K,V> next) {
            this.hash = hash;
            this.key =  key;
            this.value = value;
            this.next = next;
        }
        ...
    }
    
    // 构造一个具有默认初始容量 (11) 和负载因子 (0.75) 的新的空哈希表
    public Hashtable() {
        this(11, 0.75f);
    }
    
    // 使用指定的初始容量和默认加载因子 (0.75) 构造一个新的空哈希表
    public Hashtable(int initialCapacity) {
        this(initialCapacity, 0.75f);
    }
    
    // 使用指定的初始容量和指定的负载因子构造一个新的空哈希表
    public Hashtable(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal Load: "+loadFactor);

        if (initialCapacity==0)
            initialCapacity = 1;
        this.loadFactor = loadFactor;
        table = new Entry<?,?>[initialCapacity];
        threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    }
    
    // 构造一个与给定 Map 具有相同映射的新哈希表。 哈希表的初始容量足以容纳给定Map中的映射和默认负载因子 (0.75)
    public Hashtable(Map<? extends K, ? extends V> t) {
        this(Math.max(2*t.size(), 11), 0.75f);
        
        // 将所有映射从指定映射复制到此哈希表
        putAll(t);
    }
}
```

##### 迭代方法

- **Enumerator**：HashTable枚举迭代类，适配Enumeration与Iterator接口。
  - **int type**：Key为0，Values为1，Entries为2。
  - **boolean iterator**：为true代表是为Iterator，为fasle代表为Enumeration。

```java
private static final int KEYS = 0;
private static final int VALUES = 1;
private static final int ENTRIES = 2;

// HashTable枚举迭代类：适配Enumeration与Iterator接口
private class Enumerator<T> implements Enumeration<T>, Iterator<T> {
    Entry<?,?>[] table = Hashtable.this.table;
    int index = table.length;
    Entry<?,?> entry;
    Entry<?,?> lastReturned;
    int type;// Key: 0, Values: 1, Entries: 2
    boolean iterator;// 为true代表是为Iterator，为fasle代表为Enumeration
    
    // Iterator是快速失败的，Enumeration是非快速失败的
    protected int expectedModCount = modCount;

    // 指定int type和boolean iterator的构造方法
    Enumerator(int type, boolean iterator) {
        this.type = type;
        this.iterator = iterator;
    }
    
    // Enumeration接口方法，通过往前移动指针, 判断散列表数组是否存在元素
    public boolean hasMoreElements() {
        Entry<?,?> e = entry;
        int i = index;
        Entry<?,?>[] t = table;
        while (e == null && i > 0) {
            e = t[--i];
        }
        entry = e;
        index = i;
        return e != null;
    }

    // Enumeration接口方法，通过往前移动指针, 遍历散列表数组元素
    public T nextElement() {
        Entry<?,?> et = entry;
        int i = index;
        Entry<?,?>[] t = table;
        while (et == null && i > 0) {
            et = t[--i];
        }
        entry = et;
        index = i;
        if (et != null) {
            Entry<?,?> e = lastReturned = entry;
            entry = e.next;
            return type == KEYS ? (T)e.key : (type == VALUES ? (T)e.value : (T)e);
        }
        throw new NoSuchElementException("Hashtable Enumerator");
    }

    // Iterator接口方法，底层调用Enumeration接口的hasMoreElements()
    public boolean hasNext() {
        return hasMoreElements();
    }

    // Iterator接口方法，底层调用Enumeration接口的nextElement()，快速失败机制
    public T next() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        return nextElement();
    }
}
```

##### HashCode扰动函数

无扰动函数，只是简单的取HashCode的低31位。

```java
int hash = key.hashCode();
int index = (hash & 0x7FFFFFFF) % tab.length;
```

##### 计算哈希索引方法

取低31位HashCode后对散列表数组长度取模，得到该散列码在数组中的索引。

```java
int hash = key.hashCode();
int index = (hash & 0x7FFFFFFF) % tab.length;
```

##### 阈值计算方法

阈值，作为扩容判断的条件，取值方式为：新容量 * 负载因子，且上限为 Integer.MAX_VALUE - 8 + 1。

```java
threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
```

##### 扩容方法

- **rehash（）**：如果超过阈值，则扩容散列表, 并重新计算每个结点的hash索引、移动结点位置，如果旧容量还没达到上限，则扩容一倍，否则仍然使用旧的散列表。

```java
// 如果超过阈值，则扩容散列表, 并重新计算每个结点的hash索引、移动结点位置
protected void rehash() {
    int oldCapacity = table.length;
    Entry<?,?>[] oldMap = table;
    int newCapacity = (oldCapacity << 1) + 1;// 扩容一倍
    if (newCapacity - MAX_ARRAY_SIZE > 0) {
        if (oldCapacity == MAX_ARRAY_SIZE)
            return;// 如果旧容量已经达到上限，则无需再扩容，仍然使用旧的散列表
        newCapacity = MAX_ARRAY_SIZE;
    }

    Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];
    modCount++;
    // 计算新阈值
    threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    table = newMap;

    // 从最后一桶开始遍历到数组开头, 然后遍历每桶的结点, 并根据结点的hash值取模新容量得到新桶位置, 把该结点存入新桶完成rehash操作
    for (int i = oldCapacity ; i-- > 0 ;) {
        for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
            Entry<K,V> e = old;
            old = old.next;
            int index = (e.hash & 0x7FFFFFFF) % newCapacity;
            e.next = (Entry<K,V>)newMap[index];
            newMap[index] = e;
        }
    }
}
```

##### 添加元素方法

**所有方法都被synchronized修饰，为同步方法，且键和值都不能为null。**

- **put（K，V）**：将指定的键映射到此哈希表中的指定值，键和值都不能为null。
- **putIfAbsent（K，V）**：将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值，键和值都不能为null。
- **putAll（Map）**：将所有映射从指定映射复制到此哈希表。 这些映射将替换此哈希表对当前在指定映射中的任何键的任何映射，键和值都不能为null。

```java
// 将指定的键映射到此哈希表中的指定值，键和值都不能为null
public synchronized V put(K key, V value) {
    if (value == null) {
        throw new NullPointerException();
    }

    // 根据hash计算索引index，查找index桶是否存在该键的条目，是的话则替换旧值，返回旧值
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }

    // 如果HashTable没有对应的键，则在hash值对应的索引处添加key-value条目, 如果当前实际大小超过阈值, 则扩容散列表, 最后头插式插入到桶的位置
    addEntry(hash, key, value, index);
    return null;
}
// 在hash值对应的索引处添加key-value条目, 如果当前实际大小超过阈值, 则扩容散列表, 最后头插式插入到桶的位置
private void addEntry(int hash, K key, V value, int index) {
    modCount++;

    Entry<?,?> tab[] = table;
    if (count >= threshold) {
        // 如果超过阈值，则扩容散列表, 并重新计算每个结点的hash索引、移动结点位置
        rehash();

        tab = table;
        hash = key.hashCode();
        index = (hash & 0x7FFFFFFF) % tab.length;
    }

    // 头插法
    Entry<K,V> e = (Entry<K,V>) tab[index];
    tab[index] = new Entry<>(hash, key, value, e);
    count++;
}

// 将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值，键和值都不能为null
public synchronized V putIfAbsent(K key, V value) {
    Objects.requireNonNull(value);

    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for (; entry != null; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            
            // 只有旧值为null时才会添加value值
            if (old == null) {
                entry.value = value;
            }
            return old;
        }
    }

    // 只有不存在该key时，才会添加key-value条目
    addEntry(hash, key, value, index);
    return null;
}

// 将所有映射从指定映射复制到此哈希表。 这些映射将替换此哈希表对当前在指定映射中的任何键的任何映射，键和值都不能为null
public synchronized void putAll(Map<? extends K, ? extends V> t) {
    // 遍历复制的集合，将指定的键映射到此哈希表中的指定值，键和值都不能为null
    for (Map.Entry<? extends K, ? extends V> e : t.entrySet())
        put(e.getKey(), e.getValue());
}
```

##### 删除元素方法

**所有方法都被synchronized修饰，为同步方法。**

- **remove（Object）**：如果key存在，则从此映射中删除该key对应的映射，忽略value值。
- **remove（Object，Object）**：删除key和value都equals的键值对，value值必须匹配。

```java
// 如果key存在，则从此映射中删除该key对应的映射，忽略value值
public synchronized V remove(Object key) {
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    Entry<K,V> e = (Entry<K,V>)tab[index];

    // 遍历index桶链表, 如果hash值相等且key相等, 说明找到了对应结点, 则脱钩该结点, 并返回旧值
    for(Entry<K,V> prev = null ; e != null ; prev = e, e = e.next) {
        if ((e.hash == hash) && e.key.equals(key)) {
            modCount++;
            if (prev != null) {
                prev.next = e.next;
            } else {
                tab[index] = e.next;
            }
            count--;
            V oldValue = e.value;
            e.value = null;
            return oldValue;
        }
    }

    // 如果确实找不到对应结点, 则返回null
    return null;
}

// 删除key和value都equals的键值对，value值必须匹配
public synchronized boolean remove(Object key, Object value) {
    Objects.requireNonNull(value);

    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    Entry<K,V> e = (Entry<K,V>)tab[index];

    // 遍历index桶链表, 如果hash值相等、key相等且value相等, 说明找到了对应结点, 则脱钩该结点, 并返回true
    for (Entry<K,V> prev = null; e != null; prev = e, e = e.next) {
        if ((e.hash == hash) && e.key.equals(key) && e.value.equals(value)) {
            modCount++;
            if (prev != null) {
                prev.next = e.next;
            } else {
                tab[index] = e.next;
            }
            count--;
            e.value = null;
            return true;
        }
    }

    // 如果确实找不到对应结点, 则返回false
    return false;
}
```

##### 获取元素方法

**所有方法都被synchronized修饰，为同步方法。**

- **get（Object）**：获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}。
- **getOrDefault（Object，V）**：带默认值的获取, 如果获取不到key的元素, 则返回默认值，底层依赖于get（Object）方法。

```java
// 获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}
public synchronized V get(Object key) {
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;

    // 遍历index桶链表, 如果hash值相等且key相等, 说明找到了对应结点, 则返回该结点的value值
    for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
        if ((e.hash == hash) && e.key.equals(key)) {
            return (V)e.value;
        }
    }

    // 如果确实找不到对应结点, 则返回null
    return null;
}

// 带默认值的获取, 如果获取不到key的元素, 则返回默认值，底层依赖于get（Object）方法
public synchronized V getOrDefault(Object key, V defaultValue) {
    V result = get(key);
    return (null == result) ? defaultValue : result;
}
```

#### ConcurrentHashMap

![1624019246445](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1624019246445.png)

##### 特点

- ConcurrentHashMap，**一个支持完全并发检索和高预期并发更新的哈希表**。遵循与{@link java.util.Hashtable}相同的功能规范，并包含与{@code Hashtable}的每个方法对应的方法版本。与{@link Hashtable} 类似，但与 {@link HashMap} 不同的是，**此类不允许将 {@code null} 用作键或值**。主要设计目标是**保持并发可读性**（通常是方法get()，还有迭代器和相关方法），同时**最小化更新时的锁争用**。次要目标是**保持与java.util.HashMap相同或更好的空间消耗**，并**支持许多线程对Empty表的高初始插入率**。
- ConcurrentHashMap所有操作都是线程安全的，检索操作不需要也不支持锁，需要**原子读取、写入和CAS**，以防止所有访问的方式锁定整个表，在依赖其线程安全而不关注其同步细节的程序时，可完全视此类操作等同于{@code Hashtable}。
  - 检索操作（包括{@code get}）一般不会阻塞，可能与更新操作（包括{@code put} 和{@code remove}）同时执行，**反映的是最近完成更新操作的结果**。因此，对于诸如 {@code putAll} 和 {@code clear} 之类的聚合操作，并发检索可能只反映出插入或删除的某些条目。
  - 类似地，迭代器、拆分器和枚举返回，反映的是哈希表在迭代器/枚举创建时或创建后的**某个时刻的状态的元素**，它们不会抛出 {@link java.util.ConcurrentModificationException ConcurrentModificationException}。要注意的是，**迭代器被设计为一次仅由一个线程使用**。
  - {@code size}、{@code isEmpty} 和 {@code containsValue} 等聚合状态方法结果，通常仅在Map未在其他线程中进行并发更新时才有用，否则，这些方法的结果反映的只是**瞬态状态**，因此可以用于监测或估计目的，而不适用于程序控制。其中size并发计数是使用LongAdder的特殊化来维护的，**CounterCell计数数组机制**避免了更新计数时锁的争用，但如果在并发访问期间读取过于频繁，可能会遇到缓存抖动。
- ConcurrentHashMap，通常用分桶哈希表实现，每个键值映射都保存在一个**Node**中，大多数Node具有散列值、键、值和next Node指针。但是，存在各种子类：
  - **TreeNode**：为平衡树结点（红黑树的一种特殊形式）。
  - **TreeBins**：为持有TreeNode集合的根结点。**不保存散列值、键和值**。
  - **ForwardingNodes**：在调整大小期间，放置在桶的头部。**不保存散列值、键和值**。
  - **ReservationNodes**：用作占位符，同时在computeIfAbsent和相关方法中建立值。**不保存散列值、键和值**。
- ConcurrentHashMap，使用哈希字段值符号位进行控制，具有**负散列字段的结点**在map方法中将被特殊处理或忽略。对于添加结点的操作，由于不想浪费将不同的锁对象与每个桶相关联所需的空间，而是**使用桶链表本身的第一个结点作为锁**，即第一个结点插入（通过put或其变体）到空桶中，只需将其CASing到桶中即可，而其他结点的更新操作（插入、删除和替换）则需要获取锁。
  - 但是，使用链表的第一个结点作为锁本身还是不够完善的：当第一个结点被锁定时，任何更新都必须首先验证，**在锁定后该锁是否仍然是第一个结点**，如果不是则重试。
  - 新结点总是附加到链表中，一旦一个结点首先出现在一个桶中，就会**保持在第一个位置**，直到被删除或桶失效（调整大小时）。
  - per-bin锁的主要缺点是：受同一锁保护的桶链表中其他结点上的其他更新操作可能会停止，例如当用户equals()或映射函数需要很长时间时。然而在统计上，随机哈希码遵循泊松分布，**两个线程访问不同元素的锁争用概率大约为 1 / (8 * #elements)**。
  - 尽管即使在元素更新期间，用户也始终可以进行链表遍历，但红黑树遍历却不是，这主要是因为树旋转可能会改变根节点或其链接。因此，TreeBins需要一个简单的读写锁机制，即**与插入或移除相关的结构调整需要获取桶头结点的锁（因此不会与其他写入器冲突），且必须等待正在进行的读取操作完成**：由于只能有一个这样的服务者，可以使用一个简单的方案，使用单个“服务者”字段来阻止写入者，而读者永远不需要阻止。如果桶头锁被持有，读者可以沿着缓慢的遍历路径（通过下一个指针）继续，直到锁释放或到达链表尽头。 然而这些情况并不快，但可以最大限度地提高预期总吞吐量。
- 当ConcurrentHashMap散列表数组占用率超过阈值百分比时（ 0.75），需要进行**散列表扩容**，在启动线程分配和设置替换数组之后，任何注意到桶过满的线程都可以帮助调整散列表大小。然而，这些线程可能会继续进行插入等操作，而不是停顿。
  - 但是，线程在这样做之前要求传输小块索引（**transferIndex字段**），从而减少争用。
  - 字段**sizeCtl**中的生成戳，可确保调整大小不会重复执行。
  - 由于使用的是二次幂散列表容量扩展，所以每个桶中的元素必须**保持相同的索引**，或者以**二次幂的偏移量移动**。通过捕获可以重用的旧结点，来消除不必要的结点创建，因为它们的next指针不会改变。
  - 传输后，在旧散列表上的桶为一个特殊的**转发节点**（具有哈希字段“MOVED”），该节点包含新散列表作为其键，因此遇到转发节点时，访问和更新操作将使用新散列表表重新启动。
- ConcurrentHashMap可以用作**可扩展的频率图**（直方图或多重集的形式）：通过使用 {@link java.util.concurrent.atomic.LongAdder}以及通过{@link #computeIfAbsent computeIfAbsent} 初始化。
  - 例如，要将计数添加 {@code ConcurrentHashMap<String,LongAdder> freqs}，可以使用{@code freqs.computeIfAbsent(k -> new LongAdder()).increment();}。
- ConcurrentHashMaps支持一组顺序和并行的批量操作，与大多数 {@link Stream} 方法不同，这些操作旨在**线程安全**且明智地应用，即使其他线程并发更新映射，例如，在计算共享 注册表中值的快照摘要时。

##### 数据结构

![1625372983062](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625372983062.png)

##### 构造方法

- **空参的构造方法**：使用默认的初始表大小 (16) 创建一个新的空映射。
- **指定初始容量的构造方法**：创建一个新的空Map，其初始表大小可容纳指定数量的元素，而无需动态调整大小。
- **指定初始容量和负载因子的构造方法**：根据给定的元素数量 ({@code initialCapacity}) 和初始表密度 ({@code loadFactor}) 创建一个具有初始表大小的新空映射，估计并发更新线程数为1。
- **指定初始容量、负载因子和估计并发更新线程数的构造方法**：根据给定的元素数 ({@code initialCapacity})、表密度 ({@code loadFactor}) 和并发更新线程数 ({@code concurrencyLevel}) 创建一个具有初始表大小的新空映射。
- **指定复制集合的构造方法**：创建一个与给定Map具有相同映射的新Map。

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V> implements ConcurrentMap<K,V>, Serializable {
    
    private static final int MAXIMUM_CAPACITY = 1 << 30;
    private static final int DEFAULT_CAPACITY = 16;
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
    private static final float LOAD_FACTOR = 0.75f;
    static final int TREEIFY_THRESHOLD = 8;
    static final int UNTREEIFY_THRESHOLD = 6;
    static final int MIN_TREEIFY_CAPACITY = 64;
    private static final int MIN_TRANSFER_STRIDE = 16;
    private static int RESIZE_STAMP_BITS = 16;
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;
    static final int MOVED     = -1; // 转发节点的哈希
    static final int TREEBIN   = -2; // 树根的散列
    static final int RESERVED  = -3; // 临时保留的哈希
    static final int HASH_BITS = 0x7fffffff; // 正常节点哈希的可用位
    static final int NCPU = Runtime.getRuntime().availableProcessors();
    
    // Unsafe mechanics
    private static final sun.misc.Unsafe U;
    private static final long SIZECTL;// sizeCtl
    private static final long TRANSFERINDEX;// transferIndex
    private static final long BASECOUNT;// baseCount
    private static final long CELLSBUSY;// cellsBusy
    private static final long CELLVALUE;// value
    private static final long ABASE;// Node[].class
    // 31 - Integer.numberOfLeadingZeros(U.arrayIndexScale(Node[].class));
    private static final int ASHIFT;// 用于确定在Node[i]的值
    ...
    
    transient volatile Node<K,V>[] table;
    private transient volatile Node<K,V>[] nextTable;
    private transient volatile long baseCount;
    // 并发阈值：通常等于0.75 * 容量, 但在构造函数中等于初始容量, 散列表数组正在初始化时为-1
    private transient volatile int sizeCtl;
    private transient volatile int transferIndex;
    private transient volatile int cellsBusy;
    private transient volatile CounterCell[] counterCells;

    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;
        
        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }
        ...
    }
    
    // 使用默认的初始表大小 (16) 创建一个新的空映射。
    public ConcurrentHashMap() {
    }

    // 创建一个新的空Map，其初始表大小可容纳指定数量的元素，而无需动态调整大小。
    public ConcurrentHashMap(int initialCapacity) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException();
        int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
                   MAXIMUM_CAPACITY :
                   // 计算规范容量 > 1.5 * initialCapacity + 1
                   tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
        this.sizeCtl = cap;
    }
    
    // 根据给定的元素数量 ({@code initialCapacity}) 和初始表密度 ({@code loadFactor}) 创建一个具有初始表大小的新空映射，估计并发更新线程数为1。
    public ConcurrentHashMap(int initialCapacity, float loadFactor) {
        this(initialCapacity, loadFactor, 1);
    }
    
    // 根据给定的元素数 ({@code initialCapacity})、表密度 ({@code loadFactor}) 和并发更新线程数 ({@code concurrencyLevel}) 创建一个具有初始表大小的新空映射。
    public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        
        // 使用至少与估计线程一样多的桶
        if (initialCapacity < concurrencyLevel)
            initialCapacity = concurrencyLevel;
        
        // 计算规范容量 > 1 + initialCapacity / loadFactor
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
    }
    
    // 创建一个与给定Map具有相同映射的新Map。
    public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
        this.sizeCtl = DEFAULT_CAPACITY;
        putAll(m);
    }
}
```

##### 初始化散列表方法

- **initTable（）**：使用并发阈值初始化散列表，只在散列表为null或者散列表容量为0时，即**第一个元素添加时**才会调用。

```java
// 使用并发阈值初始化散列表
private final Node<K,V>[] initTable() {

    // 开始自旋, 散列表tab, 当前时刻阈值sc, 如果该轮自旋检测到tab已经有容量了, 说明tab被其他线程初始化了, 则当前线程结束自旋, 返回tab
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {

        // 如果sc小于0, 说明tab正在被其他线程初始化, 则当前线程让步时间片给CPU, 自己进入阻塞状态, 等待下一轮自旋
        if ((sc = sizeCtl) < 0)
            Thread.yield();

        // 如果sc不小于0, 说明tab没在初始化, 则CAS设置sizeCtl为-1(表示tab正在初始化)
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                // CAS成功, 再次检查tab是否还没被创建, 如果确实没创建, 则创建sc长或者默认容量16的散列表, 且更新并发阈值为0.75 * n, 结束自旋返回tab
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }

    return tab;
}

```

##### 迭代方法

- **Traverser**：代器和拆分器的基类，advance方法会逐个bin地遍历列表。
- **ConcurrentHashMap#BaseIterator**：键、值和条目迭代器公共类，相比Traverser基类，该类添加了字段以及iterator.remove方法。
- **Map.Entry迭代器**：继承迭代器基类BaseIterator，非快速失败机制，遍历next指针，返回MapEntry对象。
- **ConcurrentHashMap#Key迭代器**：继承迭代器基类BaseIterator，非快速失败机制，遍历next指针，返回ConcurrentHashMap.Node#Key对象。
- **ConcurrentHashMap#value迭代器**：继承迭代器基类BaseIterator，非快速失败机制，遍历next指针，返回ConcurrentHashMap.Node#Value对象。

```java
// 迭代器和拆分器的基类，advance方法会逐个bin地遍历列表
static class Traverser<K,V> {
    Node<K,V>[] tab;        // 当前表；如果调整大小则更新
    Node<K,V> next;         // 下一个要使用的条目
    TableStack<K,V> stack, spare; // 在ForwardingNodes上保存/恢复
    int index;              // 下一个要使用的 bin 索引
    int baseIndex;          // 初始表的当前索引
    int baseLimit;          // 初始表的索引边界
    final int baseSize;     // 初始表大小
    
    // 如果可能，则前进，返回下一个有效节点，如果没有，则返回null。
    final Node<K,V> advance() {
        ...
    }
    
    // 遇到转发节点时保存遍历状态。
    private void pushState(Node<K,V>[] t, int i, int n) {
        ...
    }
    
    // 可能会弹出遍历状态。
    private void recoverState(int n) {
        ...
    }
}

// ConcurrentHashMap#BaseIterator: 键、值和条目迭代器公共类，相比Traverser基类，该类添加了字段以及iterator.remove方法
static class BaseIterator<K,V> extends Traverser<K,V> {
    final ConcurrentHashMap<K,V> map;
    Node<K,V> lastReturned;

    public final boolean hasNext() { return next != null; }
	public final boolean hasMoreElements() { return next != null; }
    public final void remove() {
        Node<K,V> p;
        if ((p = lastReturned) == null)
            throw new IllegalStateException();
        lastReturned = null;
        map.replaceNode(p.key, null, null);
    }
}

// Map.Entry迭代器：继承迭代器基类BaseIterator，非快速失败机制，遍历next指针，返回MapEntry对象
static final class EntryIterator<K,V> extends BaseIterator<K,V> implements Iterator<Map.Entry<K,V>> {
    EntryIterator(Node<K,V>[] tab, int index, int size, int limit, ConcurrentHashMap<K,V> map) {
        super(tab, index, size, limit, map);
    }

    public final Map.Entry<K,V> next() {
        Node<K,V> p;
        if ((p = next) == null)
            throw new NoSuchElementException();
        K k = p.key;
        V v = p.val;
        lastReturned = p;
        advance();
        return new MapEntry<K,V>(k, v, map);
    }
}

// ConcurrentHashMap#Key迭代器：继承迭代器基类BaseIterator，非快速失败机制，遍历next指针，返回ConcurrentHashMap.Node#Key对象
static final class KeyIterator<K,V> extends BaseIterator<K,V> implements Iterator<K>, Enumeration<K> {
    KeyIterator(Node<K,V>[] tab, int index, int size, int limit,
                ConcurrentHashMap<K,V> map) {
        super(tab, index, size, limit, map);
    }

    public final K next() {
        Node<K,V> p;
        if ((p = next) == null)
            throw new NoSuchElementException();
        K k = p.key;
        lastReturned = p;
        advance();
        return k;
    }

    public final K nextElement() { return next(); }
}

// ConcurrentHashMap#value迭代器：继承迭代器基类BaseIterator，非快速失败机制，遍历next指针，返回ConcurrentHashMap.Node#Value对象
static final class ValueIterator<K,V> extends BaseIterator<K,V> implements Iterator<V>, Enumeration<V> {
    ValueIterator(Node<K,V>[] tab, int index, int size, int limit,
                  ConcurrentHashMap<K,V> map) {
        super(tab, index, size, limit, map);
    }

    public final V next() {
        Node<K,V> p;
        if ((p = next) == null)
            throw new NoSuchElementException();
        V v = p.val;
        lastReturned = p;
        advance();
        return v;
    }

    public final V nextElement() { return next(); }
}

```

##### HashCode扰动函数

- **spread（int）**：HashCode右移16位，从而**混合HashCode的高位和低位，加大低位的随机性，减少哈希碰**
  **撞发生的概率**，是一种性能、效用和质量的折衷方案。
  - 使用简单的位移与异或操作，减少系统的计算损耗；使用高位异或，可以减少低位冲突的可能性，保证查
    找效率。
  - HashCode右移16位，使得高位能被利用起来，保证了效用性。
  - 使用高位异或，可以减少低位冲突的可能性，保证散列表的质量。

```java
// 将散列的较高位传播 (XOR) 到较低位，并强制将最高位设为0。
static final int spread(int h) {
    return (h ^ (h >>> 16)) & HASH_BITS;
}

```

##### 计算哈希索引方法

- **（n - 1） & hash**：相当于hash % n，n指散列表的当前容量（桶数量），hash指获取hash（key）即key的
  hashCode扰动后结果。
  - **不能直接使用HashCode作为索引的原因**：int类型的hashCode，范围为[-2^32，2^32 - 1]，如果散列表
    数组与HashCode一一对应，需要40亿的空间（int[40亿]），明显这在内存是放不下的，也就是说明
    hashCode是不能直接作为数组索引的。因此，如果使用hashCode对散列表数组长度取模，那么就可以解决这个问题，从而保证较小的数组也还能利用上hashCode。
  - **n为2的幂次原因**：为了解决hashCode对散列表数组长度取模，设计了HashCode的扰动函数以及为2幂次的容量n，可以通过n-1来获得取模操作的低位掩码，此时**只需要通过低位掩码与扰动后的**
    **hashCode（hash值）进行一次与运算**，即可得到该hash值在散列表数组中的索引。其次通过在扩容方
    法中，经过hash值与低位掩码相与，可以保证扩容后，**只会移动少部分相与结果高位为1的桶链表**，其他
    保持不变，减少了扩容时的时间。

```java
e = tabAt(tab, (n - 1) & h)) != null

```

##### 获取/更新索引元素方法

- **tabAt（Node[]，int）**：volatile方式获取散列表tab中i位置的Node节点。
- **casTabAt（Node[]，int）**：CAS方式更新散列表tab中i位置的Node节点为v（原值必须为c才更新）。
- **setTabAt（Node[], int）**：volatile方式添加v到散列表tab中i位置。

```java
// volatile方式获取散列表tab中i位置的Node节点
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}

// CAS原子性替换散列表tab中i位置的Node节点为v(原值必须为c才更新)
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}

// volatile方式添加v到散列表tab中i位置
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
    U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
}

```

##### 规范容量计算方法

- **tableSizeFor（int）**：返回给定目标容量的2的幂次，一是可以计算出规范的容量值，二是可以把规范的容量
  值作为阈值，以便在扩容时另新容量等于该阈值（也就规范了新容量）。
  - **右移的目的**：是为了取得每位都是1的结果（最高位为当前容量的最高位），从而可以通过右移结果+1后
    可以获得2的幂次容量。

```java
// 为给定的所需容量返回表大小的 2 次幂
private static final int tableSizeFor(int c) {
    int n = c - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

##### 扩容方法

###### CounterCell

分布计数单元格，用于**分散并发叠加元素的计数**。

```java
// 用于分布计数的填充单元格，改编自LongAdder和Striped64。
static final class CounterCell {
    volatile long value;
    CounterCell(long x) { value = x; }
}

```

###### ForwardingNode

转发结点，**hash值为MOVED（-1），没有键和值，持有nextTab的引用**。

```java
static final class ForwardingNode<K,V> extends Node<K,V> {
    final Node<K,V>[] nextTable;
    
    // 创建转发结点, hash为MOVED(-1), 没有键和值, nextTab指向新散列表
    ForwardingNode(Node<K,V>[] tab) {
        super(MOVED, null, null, null);
        this.nextTable = tab;
    }
    
    // 以转发结点方式根据hash以及key对象查找结点: hash相等, 且Key相等或者equals
    Node<K,V> find(int h, Object k) {
 		// 循环以避免在转发节点上任意深度递归 
        outer: for (Node<K,V>[] tab = nextTable;;) {
            Node<K,V> e; int n;

            // 键k, 键hash值h, 新散列表tab, tab容量n, h取模n对应的结点e, 如果任意为null, 则返回null, 代表以转发结点方式没找到对应的桶头结点
            if (k == null || tab == null || (n = tab.length) == 0 || (e = tabAt(tab, (n - 1) & h)) == null)
                return null;

            // 如果找到对应的桶头结点, 则开始自旋, e的hash值eh, e的键ek
            for (;;) {
                int eh; K ek;

                // 如果e的hash值相等, e的Key相等或者e的Key equals, 说明在新tab中的e链表上找到了对应结点, 此时返回e结点即可
                if ((eh = e.hash) == h && ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;

                // 如果e的hash值还小于0, 说明e可能为转发结点、红黑树结点、computeIfAbsent临时结点
                if (eh < 0) {
                    // 如果e仍为转发结点, 说明新表tab又被扩容转移了, 此时更新tab指针, 继续自旋
                    if (e instanceof ForwardingNode) {
                        tab = ((ForwardingNode<K,V>)e).nextTable;
                        continue outer;
                    }
                    // 如果e为红黑树结点, 则按根据hash值和key值，从根结点开始查找红黑树结点; 如果e为computeIfAbsent临时结点, 则返回null
                    else
                        return e.find(h, k);
                }

                // 桶头没找到e结点, 则继续遍历e链表, 如果找到链尾还没找到对应的结点, 则返回null
                if ((e = e.next) == null)
                    return null;
            }
        }
    }
}


```

###### 获取容量n的扩容标记

helpTransfer、transfer方法调用前、调用时需要获取扩容标记时调用。

- **resizeStamp（int）**：获取容量n的扩容标记位，用于更新并发阈值为。**高16为扩容标记，第16位为并发扩容线程数**(从2开始, 步长+1)。结果肯定为负数。

```java
// 获取容量n的扩容标记位, 用于更新并发阈值为: 高16为扩容标记, 第16位为并发扩容线程数(从2开始, 步长+1), 结果肯定为负数
static final int resizeStamp(int n) {
    // n的二进制补码最高位前的为零的位数 | 1 << 15
    // eg: n = 10: 0000 0000 0000 0000 0000 0000 0000 1010
    // => 0000 0000 0000 0000, 0000 0000 0001 1100 | 0000 0000 0000 0000, 1000 0000 0000 0000
    // => 0000 0000 0000 0000, 1000 0000 0001 1100
    return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
}


```

###### 并发扩容线程数相关方法

在tryPresize、addCount、transfer等扩容方法有执行。

```java
// 第一个扩容线程时，CAS更新并发阈值sizeCtl，此时sc为rs <<< 16 + 2
U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2)

// 其他协助转移线程加入结点转移工作时，CAS更新并发阈值sizeCtl，此时sc + 1
U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)
    
// 通过sizeCtl判断当前线程扩容是否为ConcurrentHashMap的同一次扩容
(sc >>> RESIZE_STAMP_SHIFT) == rs 

// 当前线程完成转移工作后，CAS更新并发阈值sizeCtl，此时sc-1
U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)
    
// 通过sizeCtl判断当前线程是否为最后一个提交转移工作的线程
(sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT

```

###### 元素个数统计方法

在putVal或者replaceNode方法中，判断到i桶结点为**ForwardingNode结点**时调用。

- **addCount（long，int）**：
  - **baseCount或者CounterCell叠加x**，叠加后检查是否需要扩容，如果需要则启动扩容并转移结点；如果判断到其他线程正在扩容转移结点，则当前线程进行协助扩容转移结点。
  - **int check代表是否检查阈值**，大于等于0代表需要，一般为添加结点调用；小于0代表不需要，一般为删除结点调用。
  - 底层依赖**fullAddCount（long，boolean）**：
    - boolean wasUncontended代表调用fullAddCount前是否CAS叠加过一次，false说明CAS失败了。
    - **在baseCount或者counterCells上叠加或者存放x**，如果单元格为空 -> 竞争初始化as -> 竞争初始化as失败则叠加到baseCount；如果单元格不为空 -> 竞争叠加到单元格a -> 竞争叠加a失败则重新哈希再次确认叠加，还失败则扩容as。

```java
// baseCount或者CounterCell叠加x, 叠加后检查是否需要扩容, 如果需要则启动扩容并转移结点; 如果判断到其他线程正在扩容转移结点, 则当前线程进行协助扩容转移结点
private final void addCount(long x, int check) {
    // 要添加的数量x, 分布计数填充单元格数组as, ConcurrentHashMap实例字段上的元素个数b, 叠加b后的元素个数s
    CounterCell[] as; long b, s;

    // 如果CAS更新baseCount为s失败, 说明当前线程竞争b失败, 此时需要竞争as单元格a, 从而叠加x
    if ((as = counterCells) != null || !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        CounterCell a; long v; int m;
        boolean uncontended = true;

        // 如果as还没被初始化, 或者对as掩码m取模后获得的a单元格没被初始化, 或者a有被初始化但CAS叠加x值到a单元格失败, 则在as上并发叠加x
        if (as == null || (m = as.length - 1) < 0 || (a = as[ThreadLocalRandom.getProbe() & m]) == null || !(uncontended = U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            // 在baseCount或者counterCells上叠加或者存放x: 单元格为空 -> 竞争初始化as -> 竞争初始化as失败则叠加到baseCount -> || 单元格不为空 -> 竞争叠加到单元格a -> 竞争叠加a失败则重新哈希再次确认叠加, 还失败则扩容as
            fullAddCount(x, uncontended);
            return;
        }

        // 如果CAS叠加x到a单元格成功, 且只需要检查有无竞争, 则直接可以返回无需扩容了, 这种情况发生在删除、替换结点中, 因为操作后肯定不需要扩容
        if (check <= 1)
            return;

        // 如果需要扩容, 则s等于叠加ConcurrentHashMap实例字段上的元素个数baseCount, 与计数填充单元格数组CounterCell[]的所有数值
        s = sumCount();
    }

    // 到这里, 说明CAS更新baseCount为s成功, 或者竞争as成功了, 即x已经被成功叠加了, 此时需要进行扩容判断和扩容
    if (check >= 0) {
        // 旧散列表tab, 新散列表nt, tab容量n, 并发阈值sc
        Node<K,V>[] tab, nt; int n, sc;

        // 如果叠加后的s大于sc, 且旧散列表容量n还没到最大容量, 说明需要扩容
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null && (n = tab.length) < MAXIMUM_CAPACITY) {
            // 获取容量n的扩容标记位rs, 用于更新并发阈值为: 高16为扩容标记, 第16位为并发扩容线程数(从2开始, 步长+1), 结果肯定为负数
            int rs = resizeStamp(n);

            // 如果并发阈值sc小于0, 说明散列表正在被其他线程扩容, 则当前线程加入一起转移结点
            if (sc < 0) {
                // 如果sc高16位扩容标记已经改变(说明不是本次扩容了), 或者并发阈值为非法状态(并发扩容线程数从+2开始), 或者并发线程数超过了最大值2^16-1, 或者nextTable不存在, 或者transferIndex到头了, 代表非法条件或者终止条件, 则退出自旋
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 || sc == rs + MAX_RESIZERS || (nt = nextTable) == null || transferIndex <= 0)
                    break;

                // 如果sc合法, 且当前处于非终止条件, 则CAS更新sc+1, 也就是低16位并发扩容线程数+1(从2开始, 步长+1)
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    // 如果CAS更新成功, 说明当前线程争抢到协助转移结点的机会, 则转移旧散列表tab中的结点到新散列表nextTab中
                    transfer(tab, nt);
                // 如果CAS更新失败, 说明同一时刻协助转移结点的机会被其他线程抢占了, 则当前线程继续自旋, 争抢协助机会
            }
            // 如果散列表没有在被其他线程扩容, 说明当前线程为扩容的第一个线程, 则CAS更新并发阈值为1000 0000 0001 1100, 0000, 0000, 0000, 0010(eg: n=16)
            else if (U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2))
                // 如果CAS更新成功, 说明当前线程争抢到扩容转移结点的机会, 则转移旧散列表tab中的结点到新散列表nextTab中, 如果nextTab还没创建则先扩容(创建nextTab)
                transfer(tab, null);
            // 如果CAS更新失败, 说明同一时刻扩容转移结点的机会被其他线程抢占了, 则当前线程继续自旋, 争抢扩容转移机会

            // 每轮自旋都重新计算s, s等于叠加ConcurrentHashMap实例字段上的元素个数baseCount, 与计数填充单元格数组CounterCell[]的所有数值, 准备下一次自旋, 看当前nt是否还需要扩容
            s = sumCount();
        }
    }
}

// 在baseCount或者counterCells上叠加或者存放x: 单元格为空 -> 竞争初始化as -> 竞争初始化as失败则叠加到baseCount -> || 单元格不为空 -> 竞争叠加到单元格a -> 竞争叠加a失败则重新哈希再次确认叠加, 还失败则扩容as
private final void fullAddCount(long x, boolean wasUncontended) {

    // 获取当前线程哈希值h(同一个线程的h相同)
    int h;
    if ((h = ThreadLocalRandom.getProbe()) == 0) {
        ThreadLocalRandom.localInit();
        h = ThreadLocalRandom.getProbe();
        wasUncontended = true;
    }

    // 开始自旋, 是否已冲突collide(fasle代表已冲突), 分布计算单元格数组as, 当前计算单元格a, as长度n, 单元格a的值或者baseCount的值v
    boolean collide = false;
    for (;;) {
        CounterCell[] as; CounterCell a; int n; long v;

        // 如果分布计算单元格数组as已经创建, 且长度n大于0, 说明as有效, 则需要争抢counterCells数组来叠加x
        if ((as = counterCells) != null && (n = as.length) > 0) {

            // 如果h取模n-1得到的单元格a为null, 说明当前位置可以直接存放x, 如果这里h变化了, a可能会变化, 相当于随机得到新的索引位置去判断
            if ((a = as[(n - 1) & h]) == null) {

                // 如果cellsBusy标记为0, 说明as数组空闲, 可以操作as, 则构造值为x的CounterCell单元格r
                if (cellsBusy == 0) {
                    CounterCell r = new CounterCell(x);

                    // 再次检查as数组是否空闲, 如果空闲则CAS更新cellsBusy标记位为1
                    if (cellsBusy == 0 && U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {

                        // 如果CAS成功, 说明当前线程竞争到操作as的机会, counterCells数组rs, rs长度m, h取模m-1索引j, 单元格a是否创建成功created
                        boolean created = false;
                        try {               // Recheck under lock 在锁定状态下重新检查
                            // 再次检查rs不为空, 长度m大于0, 且j位置单元格为空, 说明a单元格没有被其他线程抢占, 则当前线程直接把新建的单元格r添加到数组as中
                            CounterCell[] rs; int m, j;
                            if ((rs = counterCells) != null && (m = rs.length) > 0 && rs[j = (m - 1) & h] == null) {
                                rs[j] = r;
                                created = true;
                            }
                        } finally {
                            // as操作完成后, 则更新cellsBusy为0, 代表counterCells数组已经空闲了
                            cellsBusy = 0;
                        }

                        // 如果a单元格被当前线程成功创建, 则结束自旋并返回, 代表x已经存放成功了
                        if (created)
                            break;

                        // 如果a单元格没有被当前线程创建, 则当前线程还需要继续自旋
                        continue;
                    }
                }

                // 如果cellsBusy标记为1, 说明as数组在忙, 不可以操作as, 则标记a单元格已冲突, 获取新的h和as再自旋去竞争
                collide = false;
            }

            // 如果h取模n-1得到的单元格a不为null, 说明a单元格需要竞争, 如果wasUncontended为false, 说明当前线程调用方法前已经CAS竞争a单元格失败过了, 则标记wasUncontended为true, 获取新的h和as再自旋去竞争(应该是为了缓一缓, 减少竞争)
            else if (!wasUncontended)
                wasUncontended = true;

            // 如果h取模n-1得到的单元格a不为null, 说明a单元格需要竞争, 如果当前线程已经置true了wasUncontended, 说明已经至少重刷了一遍, 则CAS叠加x到单元格a中
            else if (U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))
                // 如果CAS成功, 说明当前线程争抢叠加x到了a单元格, 此时可以结束自旋并返回了
                break;

            // CAS叠加x到a单元格失败, 如果as有变化或者长度n超出CPU核心数, 则更新collide为已冲突, 获取新的h和as再自旋去竞争
            else if (counterCells != as || n >= NCPU)
                // 对于长度n超出CPU核心数, 说明as已经达到最长了, collide只能一直为false, 一直获取新的h再自旋去竞争, 此时是永远不会走collide = true分支的
                // 而对于as有变化的情况, 说明当前as已经落后了, 不能再叠加x了, 必须重新自旋获取新的as来叠加, 获取新的as后如果还冲突, 是会走collide = true分支的
                collide = false;

            // 如果collide为已冲突且长度还没达到最大, 则更新collide为true, 代表即将无冲突了, 扩容前先让当前线程再去获取h, 确认一下as是否有空格子, 或者是否能够竞争格子成功, 或者as是否有发生变化
            else if (!collide)
                collide = true;

            // 当前线程最后一次已经确认完毕, 说明当前as大概率没有空格子(碰到两次格子都不为空), 且as大概率冲突很严重(竞争格子失败至少2次 或者 数组繁忙时至少竞争3次 或者as变化时至少竞争3次)
            // 如果cellsBusy为0, 说明as数组空闲, 可以被操作, 则CAS更新cellsBusy为1, 代表as在忙
            else if (cellsBusy == 0 && U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                try {
                    // 再次检查as是否为counterCells, 如果as不变, 则对as数组扩容2倍, 并转移旧的as数据到新的数组, 转移完成后更新as为新的数组, 完成as扩容
                    if (counterCells == as) {
                        CounterCell[] rs = new CounterCell[n << 1];
                        for (int i = 0; i < n; ++i)
                            rs[i] = as[i];
                        counterCells = rs;
                    }
                } finally {
                    // as操作完毕后, 更新cellsBusy为0, 说明as空闲了, 可以被操作了
                    cellsBusy = 0;
                }

                // 重新置collide为已冲突, 让当前线程继续去竞争, 当识别到as冲突严重且可以扩容时, 则再次扩容as
                collide = false;

                // 以当前h, 并获取新的as再自旋去竞争
                continue;
            }

            // 如果竞争a冲突, 或者方法调用前CAS单元格a失败, 则重新获取当前线程哈希值h(更新同一个线程的h, 与之前的h不一样了)
            h = ThreadLocalRandom.advanceProbe(h);
        }

        // 如果as还没创建, 或者长度n为0了, 说明as无效需要创建, 且cellsBusy标志位为0, 说明as空闲, 即数组没有其他线程在使用, 则CAS更新cellsBusy标志为1, 代表as在忙
        else if (cellsBusy == 0 && counterCells == as && U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
            // 如果CAS成功, 说明当前线程竞争到操作as的机会
            boolean init = false;
            try {
                // 重新检查as指针, 如果仍然指向counterCells, 说明还没被更新, 则构造长度为2的CounterCell[], h取模n-1得到单元格并存放往里面x, 此后init为true, 代表as已经被初始化了
                if (counterCells == as) {
                    CounterCell[] rs = new CounterCell[2];
                    rs[h & 1] = new CounterCell(x);
                    counterCells = rs;
                    init = true;
                }
            } finally {
                // as操作完成后, 则更新cellsBusy为0, 代表counterCells数组已经空闲了
                cellsBusy = 0;
            }

            // 如果as被当前线程初始化成功, 则结束自旋并返回, 代表x已经存放成功了; 如果as指针已经改变, 说明as并没有被当前线程初始化, 此时当前线程继续自旋
            if (init)
                break;
        }

        // 如果as为空, 且在初始化as时如果CAS更新cellsBusy标志为1失败, 说明初始化as操作被别的线程抢占了, 则当前线程CAS更新baseCount叠加x, 即叠加到baseCount
        else if (U.compareAndSwapLong(this, BASECOUNT, v = baseCount, v + x))
            // 如果CAS成功, 此时结束并返回, 代表x已经叠加成功了; 如果CAS失败, 则还需继续自旋找机会叠加x
            break;
    }
}
```

###### 获取元素个数方法

在size、isEmpty、addCount等方法中，会底层调用sumCount方法。

- **sumCount（）**：分布计数核心逻辑，叠加ConcurrentHashMap实例字段上的元素个数baseCount, 与计数填充单元格数组CounterCell[]的所有数值。

```java
// 分布计数核心逻辑，叠加ConcurrentHashMap实例字段上的元素个数baseCount, 与分布计数填充单元格数组CounterCell[]的所有数值
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}


```

###### 指定容量的扩容方法

在putAll和treeifyBin方法中调用。

- **tryPersize（int）**：初始化或者扩容当前散列表tab容量，到最接近指定容量的2次幂次容量，可能会初始化tab、无需扩容tab、协助转移tab结点、扩容tab并转移结点。

```java
// 初始化或者扩容当前散列表tab容量, 到最接近指定容量的2次幂次容量, 可能会初始化tab、无需扩容tab、协助转移tab结点、扩容tab并转移结点
private final void tryPresize(int size) {
    // 获取最接近指定容量的2次幂次容量c
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY : tableSizeFor(size + (size >>> 1) + 1);

    // 开始自旋, 如果并发阈值sc大于等于0, 说明散列表没有在做扩容或者初始化, 当前散列表tab, tab容量n
    int sc;
    while ((sc = sizeCtl) >= 0) {
        Node<K,V>[] tab = table; int n;

        // 如果散列表还没初始化, 则判断并发阈值sc与新容量c, 取两者的大者来创建散列表tab的长度
        if (tab == null || (n = tab.length) == 0) {
            n = (sc > c) ? sc : c;

            // 如果CAS成功更新sc为-1, 说明当前线程争抢到创建散列表tab的机会
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    // CAS成功则再次检查, 如果table指针没有被改变, 则创建n长的散列表为table, 且更新并发阈值为0.75 * n
                    if (table == tab) {
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
            }
        }

        // 如果散列表已存在, 且c小于等于sc, 说明并不需要扩容这么快, 或者容量n已经超出了最大容量, 则直接返回即可, 无需扩容
        else if (c <= sc || n >= MAXIMUM_CAPACITY)
            break;

        // 如果散列表已存在, 且c大于sc, 且n还没达到最大容量,
        else if (tab == table) {
            // 获取容量n的扩容标记位, 用于更新并发阈值为: 高16为扩容标记, 第16位为并发扩容线程数(从2开始, 步长+1), 结果肯定为负数
            int rs = resizeStamp(n);

            // 如果并发阈值sc小于0, 说明散列表正在被其他线程扩容, 则当前线程加入一起转移结点
            if (sc < 0) {
                Node<K,V>[] nt;

                // 如果sc高16位扩容标记已经改变(说明不是本次扩容了), 或者并发阈值为非法状态(并发扩容线程数从+2开始), 或者并发线程数超过了最大值2^16-1, 或者nextTable不存在, 或者transferIndex到头了, 代表非法条件或者终止条件, 则退出自旋
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 || sc == rs + MAX_RESIZERS || (nt = nextTable) == null || transferIndex <= 0)
                    break;

                // 如果sc合法, 且当前处于非终止条件, 则CAS更新sc+1, 也就是低16位并发扩容线程数+1(从2开始, 步长+1)
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    // 如果CAS更新成功, 说明当前线程争抢到协助转移结点的机会, 则转移旧散列表tab中的结点到新散列表nextTab中
                    transfer(tab, nt);
                // 如果CAS更新失败, 说明同一时刻协助转移结点的机会被其他线程抢占了, 则当前线程继续自旋, 争抢协助机会
            }

            // 如果散列表没有在被其他线程扩容, 说明当前线程为扩容的第一个线程, 则CAS更新并发阈值为1000 0000 0001 1100, 0000, 0000, 0000, 0010(eg: n=16)
            else if (U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2))
                // 如果CAS更新成功, 说明当前线程争抢到扩容转移结点的机会, 则转移旧散列表tab中的结点到新散列表nextTab中, 如果nextTab还没创建则先扩容(创建nextTab)
                transfer(tab, null);
            // 如果CAS更新失败, 说明同一时刻扩容转移结点的机会被其他线程抢占了, 则当前线程继续自旋, 争抢扩容转移机会
        }
    }
}


```

###### 协助转移结点方法

在putVal或者replaceNode方法中，判断到i桶结点为**ForwardingNode结点**时调用。

- **helpTransfer（Node，Node）**：如果旧tab正在被其他线程扩容转移中，则当前线程加入一起转移结点, 协助完成后返回新tab。

```java
// 如果旧tab正在被其他线程扩容转移中, 则当前线程加入一起转移结点, 协助完成后返回新tab
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
    // 旧散列表tab, i桶头结点f, 新散列表nextTab, 并发阈值sc
    Node<K,V>[] nextTab; int sc;

    // 如果在tab碰到ForwardingNode结点, 且该结点指向的新散列表不为null, 说明确实有线程正在扩容转移tab上的结点(理论上都是成立), 则当前线程也加入结点转移工作中
    if (tab != null && (f instanceof ForwardingNode) && (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
        // 获取容量n的扩容标记位, 用于更新并发阈值为: 高16为扩容标记, 第16位为并发扩容线程数(从2开始, 步长+1), 结果肯定为负数
        int rs = resizeStamp(tab.length);

        // 如果nextTable指针没变, 且table指针没变, 并发阈值sc小于0, 说明tab正在被其他线程扩容转移中, 则当前线程加入一起转移结点
        while (nextTab == nextTable && table == tab && (sc = sizeCtl) < 0) {
            // 如果sc高16位扩容标记已经改变(说明不是本次扩容了), 或者并发阈值为非法状态(并发扩容线程数从+2开始), 或者并发线程数超过了最大值2^16-1, 或者transferIndex到头了, 代表非法条件或者终止条件, 则退出自旋
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 || sc == rs + MAX_RESIZERS || transferIndex <= 0)
                break;

            // 如果sc合法, 且当前处于非终止条件, 则CAS更新sc+1, 也就是低16位并发扩容线程数+1(从2开始, 步长+1)
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                // 如果CAS更新成功, 说明当前线程争抢到协助转移结点的机会, 则转移旧散列表tab中的结点到新散列表nextTab中
                transfer(tab, nextTab);

                // 当前线程转移完毕, 则结束自旋
                break;
            }
            // 如果CAS更新失败, 说明同一时刻协助转移结点的机会被其他线程抢占了, 则当前线程继续自旋, 争抢协助机会
        }

        // 自旋结束, 返回新散列表nextTab
        return nextTab;
    }

    // 一般来说永远不会走到这里
    return table;
}


```

###### 散列表扩容或者转移结点方法

扩容的核心方法，在addCount、helpTransfer方法中，会底层调用transfer方法。

- **transfer（Node[]，Node[]）**：
  - **转移旧散列表tab中的结点到新散列表nextTab中**，如果nextTab还没创建则先扩容(创建nextTab)。
  - 如果nextTab已创建，则转移线程步骤为：
    - 划分转移区间 -> i为转移结点 -> 继续前进划分转移区间。
    - 划分转移区间 -> i为业务结点-> 转移普通链表/红黑树 -> 转移完成，继续前进划分转移区间。
    - 如果当前线程不为最后一个转移完成线程，即**(sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT**，则直接返回即可，无需提交新散列表nextTab。
    - 如果当前线程为最后一个转移完成线程，即**(sc - 2) == resizeStamp(n) << RESIZE_STAMP_SHIFT**，则进行最后一次检查i与transferIndex，没问题则提交新散列表nextTab（设置为table）。
    - 注意，在addCount（long，int）方法中，转移tab到nextTab完成返回后，还要继续判断nextTab是否需要扩容。

```java
// 转移旧散列表tab中的结点到新散列表nextTab中, 如果nextTab还没创建则先扩容(创建nextTab)
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    // 旧散列表tab, 新散列表nextTab, tab容量n, 步长stride
    int n = tab.length, stride;

    // 步长计算公式(多核) = n/8 / cores, 步长计算公式(单核) = 1, 如果步长小于16, 则默认为16
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE;

    // 如果nextTab还没被创建, 说明当前线程为扩容发起的线程(由于经过了CAS才进入的, 所以nextTab为null的只有一个线程), 则创建2倍容量的散列表数组, 更新为nextTable开始扩容, 且更新转移索引为n(即从n开始转移)
    if (nextTab == null) {
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }

    // 到这里nextTab已经被创建, nextTab容量nextn, 创建转发结点fwd(hash为MOVED(-1), 没有键和值, nextTab指向新散列表, 下面转移完结点都会复用这个转发结点) => put时会判断, 判断到则会协助转移结点
    int nextn = nextTab.length;
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);

    // 当前线程是否还需要前进划分转移区间advance, 当成线程转移工作是否已经完成finishing
    boolean advance = true;
    boolean finishing = false;

    // 当前线程开始转移结点, 当前索引i, 左边界bound, i桶的头结点f, f的hash值fh
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;

        // 如果当前线程还需要前进划分转移区间, 则继续前进, 当前转移索引nextIndex, 下一个转移边界nextBound
        while (advance) {
            int nextIndex, nextBound;

            // 索引i前移1, 如果超过了左边界bound, 或者转移已经完成了, 则更新advance为false, 代表不需要前进划分转移区间了
            if (--i >= bound || finishing)
                advance = false;
            // 如果i还没超出左边界bound, 且nextIndex小于等于0了, 说明所有转移区间都被其他线程划分完毕了, 则更新advance为false, 代表不需要前进划分转移区间了
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            // 如果i还没超出左边界bound, 且nextIndex也还没小于等于0, 说明当前线程存在转移区间, 则transferIndex前移一个步长, 并使用CAS更新, 新的左边界nextBound
            else if (U.compareAndSwapInt(this, TRANSFERINDEX, nextIndex, nextBound = (nextIndex > stride ? nextIndex - stride : 0))) {
                // 如果CAS成功, 说明当前线程抢到转移区间, 则更新左边界bound为nextBound, 更新i为nextIndex-1, 更新advance为false, 代表当前转移区间已经确定(为bound~i), 不需要前进划分转移区间了
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }

            // 如果CAS失败, 说明当前线程没抢到转移区间, 则需要重新advance自旋: 前移i、判断transferIndex是否合法、CAS更新transferIndex...
        }

        // 到这里, 说明当前线程的转移区间(bound~i)已经确定好了
        // 如果i小于0, 或者i大于tab容量n, 或者i+n新的索引大于nextn, 说明i转移起来不合法了(i"越界"了), 也就是当前转移工作已经完成了
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;

            // 如果当前线程转移工作已经完成了, 且通过了最后一次的校验(遍历完旧表tab), 则发起最后的扩容提交, 正式更新table为新散列表nextTab, 清空nextTable, 更新并发阈值sizeCtl为(0.75 * 新表容量), 结束扩容转移操作并返回
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }

            // 如果当前线程转移工作已经完成了, 则CAS更新并发阈值sizeCtl-1, 代表扩容/转移并发线程数-1, 如果CAS成功, 说明当前线程抢到了提交工作的名额
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                // 如果容量n的扩容标记位rs不一致, 说明sc低位还有别的线程在工作, 此时直接返回即可, 把nextTab的赋值操作留给最后一个提交线程来实现
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;

                // 如果容量n的扩容标记位rs一致, 说明当前线程为最后一个提交工作的线程, 此时置finishing=advance为true, 进行提交前最后的校验
                finishing = advance = true;
                i = n;
            }

            // 如果CAS失败, 说明当前线程没抢到了提交工作的名额, 需要自旋等待下一轮名额的竞争
        }

        // i没有"越界", 当前线程适合转移结点, 则volatile方式获取散列表tab中i位置的Node节点, 如果为null, 则CAS标记为ForwardingNode结点, 代表已经转移过了(其他线程看到会跳过的)
        else if ((f = tabAt(tab, i)) == null)
            // 如果CAS标记成功, 说明当前线程成功转移了i结点, 则置advance为true, 当前线程可以继续前进划分区间(为了保持stride不长的区间); 如果CAS失败, 则保持i位置不动进行下一轮自旋(一定是进入MOVED的判断分支)
            advance = casTabAt(tab, i, null, fwd);

        // 如果当前线程转移i结点碰到ForwardingNode结点, 说明该结点已经被其他线程转移了, 则置advance为true, 当前线程可以继续前进划分区间(为了保持stride不长的区间)
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed 已经处理

        // 如果当前线程转移i结点碰到非ForwardingNode结点, 说明该结点是个有效的业务结点, 则对结点加锁老老实实地转移, 如果获取不到f锁, 则阻塞等待其他线程释放f锁, 保证再判断一次f结点是否转移完毕
        else {
            synchronized (f) {
                // 拿到f锁后, 判断如果f结点确实没有改变, 说明f结点适合转移; 如果不是即说明f结点被改变了(也就是被f转移了), 则释放f锁, 保持i位置不动进行下一轮自旋(一定是进入MOVED的判断分支)
                if (tabAt(tab, i) == f) {
                    // 新散列表nextTab[0, n-1]低区间链表ln, nextTab[n-1, 2n-1]高区间链表hn
                    Node<K,V> ln, hn;

                    // 如果f的hash值fh(在上面赋值了)大于等于0, 说明f为普通结点, 则以普通链表方式转移结点
                    if (fh >= 0) {
                        // 旧散列表tab容量n, 通过fh & n计算f结点位标记runBit, lastRun对应该位标记的结点
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;

                        // f后继p, 遍历f链表, 位标记runBit取最后结点的位标记(如果不同的话), lastRun对应该位标记的结点
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }

                        // 如果最后位标记runBit为0, 说明lastRun结点的hash值"不变", 即lastRun结点转移后需要在新散列表nextTab[0, n-1]低区间链表ln内
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        // 如果最后位标记runBit不为0, 说明lastRun结点的hash值"变了", 即lastRun结点转移后需要在新散列表nextTab[n-1, 2n-1]高区间链表hn
                        else {
                            hn = lastRun;
                            ln = null;
                        }

                        // 再次遍历f链表, 切割f~lastRun之间的结点为ln链表, lastRun到剩余结点为hn链表
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }

                        // 划分ln和hn链表后, volatile方式添加ln链表到新散列表nextTab的i桶中
                        setTabAt(nextTab, i, ln);

                        // volatile方式添加hn链表到新散列表nextTab的i+n桶中
                        setTabAt(nextTab, i + n, hn);

                        // volatile方式添加ForwardingNode结点到旧散列表tab中i位置, 代表i桶结点已经转移完毕
                        setTabAt(tab, i, fwd);

                        // 置advance为true, 当前线程可以继续前进划分区间(为了保持stride不长的区间)
                        advance = true;
                    }

                    // 如果fh小于0, 且f为TreeBin类型, 说明f链为红黑树, f.root为红黑树的根结点, 则以"红黑树方式"转移结点
                    else if (f instanceof TreeBin) {
                        // f结点备份指针t, 低区间链表lo, 高区间链表hi, lo尾指针loTail, hi尾指针hiTail, lo链表容量lc, hi链表容量hc
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;

                        // 以链表方式遍历t链表, 当前遍历结点e, e的hash值h, 根据e重新构造的结点p
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>(h, e.key, e.val, null, null);

                            // 通过h & n计算结点位标记, 如果为0, 说明结点的hash值"不变", 即结点转移后需要在新散列表nextTab[0, n-1]低区间链表lo内, 则维护lo链表结点的前驱和后继
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            // 通过h & n计算结点位标记, 如果不为0, 说明结点的hash值"变了", 即结点转移后需要在新散列表nextTab[n-1, 2n-1]高区间链表hi内, 则维护hi链表结点的前驱和后继
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }

                        // lo链表维护完毕后, 如果lc小于6, 则普通化lo链表; 否则lc大于等于6, 如果hi链表存在, 说明有必要红黑树化lo链表, 如果hi链表不存在, 说明红黑树保持不变为t即可
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) : (hc != 0) ? new TreeBin<K,V>(lo) : t;

                        // hi链表维护完毕后, 如果hc小于6, 则普通化hi链表; 否则hc大于等于6, 如果lo链表存在, 说明有必要红黑树化hi链表, 如果lo链表不存在, 说明红黑树保持不变为t即可
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) : (lc != 0) ? new TreeBin<K,V>(hi) : t;

                        // 红黑树化/普通化完毕后, volatile方式添加ln链表到新散列表nextTab的i桶中
                        setTabAt(nextTab, i, ln);

                        // volatile方式添加hn链表到新散列表nextTab的i+n桶中
                        setTabAt(nextTab, i + n, hn);

                        // volatile方式添加ForwardingNode结点到旧散列表tab中i位置, 代表i桶结点已经转移完毕
                        setTabAt(tab, i, fwd);

                        // 置advance为true, 当前线程可以继续前进划分区间(为了保持stride不长的区间)
                        advance = true;
                    }
                }
            }
        }
    }
}
```

##### 添加元素方法

- **put（K，V）**：将指定的键映射到此哈希表中的指定值，键和值都不能为null。
- **putIfAbsent（K，V）**：将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值，键和值都不能为null。
- **putAll（Map）**：将所有映射从指定映射复制到此哈希表。 这些映射将替换此哈希表对当前在指定映射中的任何键的任何映射，键和值都不能为null。

```java
// 将指定的键映射到此哈希表中的指定值，键和值都不能为null
public V put(K key, V value) {
    return putVal(key, value, false);
}

// 将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值，键和值都不能为null
public V putIfAbsent(K key, V value) {
    return putVal(key, value, true);
}

// 将所有映射从指定映射复制到此哈希表。 这些映射将替换此哈希表对当前在指定映射中的任何键的任何映射，键和值都不能为null
public void putAll(Map<? extends K, ? extends V> m) {
    // 初始化或者扩容当前散列表tab容量, 到最接近m.size的2次幂次容量, 可能会初始化tab、无需扩容tab、协助转移tab结点、扩容tab并转移结点
    tryPresize(m.size());

    // 遍历复制集合m, 将指定值和指定键相关联, 形成key-value键值对, 如果包含相同的键会替换旧值
    for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
        putVal(e.getKey(), e.getValue(), false);
}

// put和putIfAbsent的核心逻辑，将指定值和指定键相关联，形成key-value键值对，如果包含相同的键且onlyIfAbsent为false，则会替换旧值
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();

    // 将散列的较高位传播 (XOR) 到较低位，并强制将最高位设为0
    int hash = spread(key.hashCode());

    // 开始自旋, 散列表tab, 桶头结点f, 当前散列表容量n, Key的hash值索引i, 桶头结点f的hash值fh, f桶链上结点数量binCount
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;

        // 如果tab为null或者tab容量n为0, 说明散列表还没被创建, 则创建散列表
        if (tab == null || (n = tab.length) == 0)
            // 使用并发阈值初始化散列表
            tab = initTable();

        // 如果tab已被初始化, 且桶头结点f为null, 则CAS方式更新散列表tab中i位置的Node节点为v, 设置成功则结束自旋, 否则继续下轮自旋
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                break;
        }

        // 如果tab已被初始化, 且桶头结点f不为null, 但f为转发结点, 说明当前散列表已经是旧的散列表了, 则当前线程协助转移tab元素到新的散列表中
        else if ((fh = f.hash) == MOVED)
            // 如果旧tab正在被其他线程扩容转移中, 则当前线程加入一起转移结点, 协助完成后返回新tab, 则基于新tab进行下一轮自旋(会把结点插入到新tab中)
            tab = helpTransfer(tab, f);

        // 如果tab已被初始化, 且桶头结点f不为null, 且f也不为转发结点, 说明f为正常结点, 可以在桶链上追加元素
        else {
            V oldVal = null;

            // 此时对桶头结点f进行加锁, 只有抢到锁的线程才能进行追加结点
            synchronized (f) {
                // 再次检查当前桶头结点是否还为f, 如果是才继续追加, 否则说明桶头结点已被其他线程更新了, 则释放f锁, 继续自旋
                if (tabAt(tab, i) == f) {
                    // 如果f的hash值fh大于等于0, 说明f为普通Node结点, 则更新binCount为1(1个Node), 然后按普通链表方式添加结点
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;

                            // 如果已经存在该元素, 如果只允许在不存在时才添加则什么也不做, 否则替换旧值, 结束遍历
                            if (e.hash == hash && ((ek = e.key) == key || (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }

                            // 如果不存在该元素, 则直接添加到链尾, 结束遍历
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key, value, null);
                                break;
                            }
                        }
                    }

                    // 如果f的hash值fh小于0, 且为TreeBin类型, 说明f链为红黑树, 则更新binCount为2(1个TreeBin + 1个TreeNode), 然后按照红黑树结点的添加方法添加该结点
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        // 红黑树结点的添加方法（插入成功则返回null，插入失败则返回已经存在的结点）=> 类似HashMap#putTreeVal（HashMap，Node，int，K，V）, 但这里还需要竞争TreeBin桶头结点的写锁
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                            // 如果插入失败, 说明已经存在该元素, 此时p.val为旧值, 如果只允许在不存在时才添加则什么也不做, 否则替换旧值
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }

            // 释放桶头结点f的锁, 如果binCount不为0, 说明结点已经添加成功
            if (binCount != 0) {

                // 如果f链上结点个数是否大于红黑树化阈值8, 则需要红黑树化f链
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);

                // 否则说明还不需要红黑树化, 如果存在旧值, 则说明发生的不是结点添加, 而是结点值替换, 此时只需要返回旧值即可, 无需更新实际大小和扩容
                if (oldVal != null)
                    return oldVal;

                // 否则说明为结点添加, 则结束自旋, 开始更新实际大小和扩容
                break;
            }
        }
    }

    // 到这里说明发生了结点添加, 则baseCount或者CounterCell叠加x, 叠加后检查是否需要扩容, 如果需要则启动扩容并转移结点; 如果判断到其他线程正在扩容转移结点, 则当前线程进行协助扩容转移结点
    addCount(1L, binCount);

    // 结点添加成功, 返回null作为标志位
    return null;
}


```

##### 删除元素方法

- **remove（Object）**：根据key删除结点，删除成功会返回旧值，key可以为null。
- **remove（Object，Object）**：根据key和value删除结点，删除成功会返回旧值，key不可以为null。

```java
// 根据key删除结点, 删除成功会返回旧值, key可以为null
public V remove(Object key) {
    // 根据key和cv删除结点, 删除成功会返回旧值
    return replaceNode(key, null, null);
}

// 根据key和value删除结点, 删除成功会返回旧值, key不可以为null
public boolean remove(Object key, Object value) {
    // 根据key和cv删除结点, 删除成功会返回旧值
    if (key == null) throw new NullPointerException();
    return value != null && replaceNode(key, null, value) != null;
}

// 根据key和cv删除/替换结点核心逻辑, 如果指定了value说明为替换模式, 此时会替换目标结点的value值为指定的value并返回null, 否则删除成功会返回旧值
final V replaceNode(Object key, V value, Object cv) {
    // 将散列的较高位传播 (XOR) 到较低位，并强制将最高位设为0
    int hash = spread(key.hashCode());

    // 开始自旋, 当前散列表tab, tab容量n, hash值索引i, hash值索引对应的桶头结点f, f的hash值fh
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;

        // 如果tab不存在, 或者n为0, 或者f为null, 则直接结束自旋, 返回null
        if (tab == null || (n = tab.length) == 0 || (f = tabAt(tab, i = (n - 1) & hash)) == null)
            break;

        // 如果tab已被初始化, 且桶头结点f不为null, 但f为转发结点, 说明当前散列表已经是旧的散列表了, 则当前线程协助转移tab元素到新的散列表中
        else if ((fh = f.hash) == MOVED)
            // 如果旧tab正在被其他线程扩容转移中, 则当前线程加入一起转移结点, 协助完成后返回新tab, 则基于新tab进行下一轮自旋(会新tab中删除对应的结点)
            tab = helpTransfer(tab, f);

        // 如果tab已被初始化, 且桶头结点f不为null, 且f也不为转发结点, 说明f为正常结点, 可以在tab上删除结点
        else {
            V oldVal = null;
            boolean validated = false;// 是否有真正做了结点删除, 有的话为true, 代表需要更新散列表实际大小并返回旧值

            // 此时对桶头结点f进行加锁, 只有抢到锁的线程才能进行删除结点
            synchronized (f) {
                // 再次检查当前桶头结点是否还为f, 如果是才继续删除, 否则说明桶头结点已被其他线程更新了, 则释放f锁, 继续自旋
                if (tabAt(tab, i) == f) {
                    // 如果f的hash值fh大于等于0, 说明f为普通Node结点, 则普通链表方式删除结点
                    if (fh >= 0) {
                        validated = true;

                        // 以链表方式遍历f链表, 当前遍历结点e, e键ek, 根据e重新构造的结点p
                        for (Node<K,V> e = f, pred = null;;) {
                            K ek;

                            // 如果找到e的hash值相等, 且e键相等的或者e键equals的结点, 则继续比较e值与cv是否相等或者e值e与cv是否quals
                            if (e.hash == hash && ((ek = e.key) == key || (ek != null && key.equals(ek)))) {
                                V ev = e.val;

                                // 如果k和cv都相等或者equals, 则说明找到了对应的结点
                                if (cv == null || cv == ev || (ev != null && cv.equals(ev))) {
                                    oldVal = ev;

                                    // 如果指定了不为null的value, 则替换e结点的value
                                    if (value != null)
                                        e.val = value;
                                    // 如果没指定value, 且e结点还有前驱, 则链接e的前驱和后继, 脱钩e结点
                                    else if (pred != null)
                                        pred.next = e.next;
                                    // 如果没指定value, 但e结点没有了前驱, 说明e为头结点, 则volatile方式添加后继为头结点的链表到散列表tab的i桶中
                                    else
                                        setTabAt(tab, i, e.next);
                                }

                                // 结束链表遍历, 返回null, 再去更新散列表实际大小并返回旧值
                                break;
                            }
                            pred = e;

                            // 如果连k匹配的结点都找不到, 则结束链表遍历, 返回null, 再去更新散列表实际大小并返回旧值
                            if ((e = e.next) == null)
                                break;
                        }
                    }

                    // 如果f的hash值fh小于0, 且为TreeBin类型, 说明f链为红黑树, 则按照红黑树结点方式删除结点
                    else if (f instanceof TreeBin) {
                        validated = true;
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> r, p;

                        // 根据hash值和key值, 从根结点r开始查找红黑树结点p, 如果找到继续匹配cv, 否则释放f锁, 再去散列表实际大小并返回旧值
                        if ((r = t.root) != null && (p = r.findTreeNode(hash, key, null)) != null) {
                            V pv = p.val;

                            // 如果k和cv都相等或者equals, 则说明找到了对应的结点
                            if (cv == null || cv == pv || (pv != null && cv.equals(pv))) {
                                oldVal = pv;

                                // 如果指定了不为null的value, 则替换e结点的value, 再去散列表实际大小并返回旧值
                                if (value != null)
                                    p.val = value;

                                // 如果没指定value, 则调用红黑树结点的删除方法, 返回false, 说明正常插入结点到红黑树了, 不需要拆除当前红黑树退化成普通链表
                                else if (t.removeTreeNode(p))
                                    // 如果返回true, 则普通化t.first链表, 并volatile方式添加到散列表tab中i位置, 再去散列表实际大小并返回旧值
                                    setTabAt(tab, i, untreeify(t.first));
                            }
                        }
                    }
                }
            }

            // 为true说明真正做了结点删除, 需要更新散列表实际大小并返回旧值
            if (validated) {
                if (oldVal != null) {
                    // 如果没指定value, 说明不是替换模式, 则返回旧值
                    if (value == null)
                        // baseCount或者CounterCell叠加x, 删除操作时这里无需协助扩容
                        addCount(-1L, -1);
                    return oldVal;
                }
                break;
            }
        }
    }

    // 没有做到结点删除, 返回null即可(比如替换操作或者没找要删除的结点时)
    return null;
}

```

##### 获取元素方法

- **get（Object）**：获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}，需要判断桶头结点e是否为转发结点、红黑树结点、computeIfAbsent临时结点。
- **getOrDefault（Object，V）**：带默认值的获取, 如果获取不到key的元素, 则返回默认值，底层依赖于get（Object）方法，需要判断桶头结点e是否为转发结点、红黑树结点、computeIfAbsent临时结点。

```java
// 获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}：需要判断桶头结点e是否为转发结点、红黑树结点、computeIfAbsent临时结点
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;

    // 将散列的较高位传播 (XOR) 到较低位，并强制将最高位设为0
    int h = spread(key.hashCode());

    // 当前散列表tab, Key的hash值h, tab容量n, h取模n后的桶e, e的hash值eh, e的键ek
    if ((tab = table) != null && (n = tab.length) > 0 && (e = tabAt(tab, (n - 1) & h)) != null) {

        // 如果e的hash值相等, 且Key相等或者Key equals, 说明桶头结点e就是要找到的结点, 此时返回e的value
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }

        // 如果e的hash小于0, 说明e可能为转发结点、红黑树结点、computeIfAbsent临时结点
        else if (eh < 0)
            // 1. 如果e为转发结点, 则以转发结点方式根据hash以及key对象查找结点: hash相等, 且Key相等或者equals
            // 2. 如果e为红黑树结点, 则根据hash值和key值，从根结点开始查找红黑树结点 => 同HashMap#getTreeNode（int，Object）
            // 3. 如果e为computeIfAbsent临时结点, 则返回null
            return (p = e.find(h, key)) != null ? p.val : null;

        // 如果桶头没找e结点, 则继续遍历e链表, 找到hash值相等, 且Key相等或者equals结点并返回
        while ((e = e.next) != null) {
            if (e.hash == h && ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }

    // 如果实在找不到, 则返回null
    return null;
}

// 带默认值的获取, 如果获取不到key的元素, 则返回默认值，底层依赖于get（Object）方法: 需要判断桶头结点e是否为转发结点、红黑树结点、computeIfAbsent临时结点
public V getOrDefault(Object key, V defaultValue) {
    V v;
    return (v = get(key)) == null ? defaultValue : v;
}


```

##### 红黑树化指定index桶

- **treeifyBin（Node[]，int）**：红黑树化index对应桶中的普通链表，重构hash桶中的普通单向Node链表为**双向无环TreeNode链表**，并在最后使用TreeBin包装root结点放到index桶中。

```java
// 红黑树化index对应桶中的普通链表
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;

    // 当前散列表tab, 待红黑树化的桶索引index, 桶结点b, tab容量n, 并发阈值sc
    if (tab != null) {
        // 如果容量n小于最小树化容量64, 则停止红黑树化链表, 转而初始化或者扩容当前散列表tab容量到n的2倍, 可能会初始化tab、无需扩容tab、协助转移tab结点、扩容tab并转移结点
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            tryPresize(n << 1);

        // 如果容量n满足最小树化容量64, 说明可以进行红黑树化b桶链表, 则对b结点加锁进行红黑树化
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {
                // 再次检查, 如果b结点还是为index桶的头结点, 说明b没有变化, 则对b结点加锁进行红黑树化
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;

                    // 以链表方式遍历b链表, 当前遍历结点e, 根据e重新构造的结点p, 维护hd链表结点的前驱和后继
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p = new TreeNode<K,V>(e.hash, e.key, e.val, null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }

                    // volatile方式添加hd链表到散列表tab的index桶中
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}


```

##### 普通化红黑树链表

- **unTreeify（Node）**：普通化红黑树链表，重构双向无环TreeNode链表为**普通单向Node链表**。

```java
// 普通化红黑树链表，传入的参数b是root结点，而非TreeBin结点
static <K,V> Node<K,V> untreeify(Node<K,V> b) {
    Node<K,V> hd = null, tl = null;

    // 以链表方式遍历b链表, 当前遍历结点q, 根据q重新构造的结点p, 删除hd链表结点的前驱, 只维护hd链表结点的后继
    for (Node<K,V> q = b; q != null; q = q.next) {
        Node<K,V> p = new Node<K,V>(q.hash, q.key, q.val, null);
        if (tl == null)
            hd = p;
        else
            tl.next = p;
        tl = p;
    }

    // 返回普通结点hd链表
    return hd;
}


```

##### 红黑树结点方法

###### TreeNode

- **性质1**：红黑树的结点**要么是红色，要么是黑色**。
- **性质2**：红黑树的**根结点是黑色**的。
- **性质3**：红黑树的**叶子结点（nil）都是黑色**的。
- **性质4**：红黑树的**红色结点必须有两个黑色结点**。
  - **推论**：从根结点到每个叶子结点的所有路径上，不可能存在两个连续的红色结点。
- **性质5**：红黑树是**黑色平衡**的，即从根结点到每个叶子结点的所有路径中，所经过的黑色结点数都是一样的。
  - **推论**：如果一个结点右黑色的子结点，那么该结点一定是有两个孩子结点，因为必须有另一半才能保证该
    结点黑色平衡。

```java
// 红黑树业务结点
static final class TreeNode<K,V> extends Node<K,V> {
    TreeNode<K,V> parent;
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;
    boolean red;

    TreeNode(int hash, K key, V val, Node<K,V> next,
             TreeNode<K,V> parent) {
        super(hash, key, val, next);
        this.parent = parent;
    }

    // 递归调用find（int，Object，Class）查找根节点
    Node<K,V> find(int h, Object k) {...}

    // 根据hash值和key，从根结点开始查找
    final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {...}
}


```

###### TreeBin

下面红黑树的方法如果除了find（int，Object）以及find（int，Object，Class）方法为TreeNode的实例方法，其他的都为TreeBin的方法，包括读写锁控制的方法。

- 为红黑树桶头结点，内部含真实红黑树的根结点root指针和链表头first指针，**不保存键和值**。
- 维护了读写锁，强迫**写线程必须等待所有读线程**完成后才能进行红黑树结点操作，而对于读线程存在一下优化：
  - 如果当前红黑树存在写线程或者等待写锁线程，则**以链表的方式去遍历**出红黑树结点并返回。
  - 如果当前红黑树没有写线程或者等待写锁线程，则**叠加读锁状态后，以红黑树方式去遍历**红黑树结点并返回。
  - 最后释放当前读锁状态，如果释放读锁前的状态为读状态，或者为等待写锁状态且存在等待写锁线程， 说明当前线程为最后一个读线程，需要**唤醒因调用contendedLock（）争抢不到写锁而进入阻塞等待的线程**，使其重新争抢写锁。

```java
// 维护了读写锁的红黑树桶头结点
static final class TreeBin<K,V> extends Node<K,V> {
    TreeNode<K,V> root;// 红黑树根结点
    volatile TreeNode<K,V> first;// 链表形式的头结点(等于红黑树根结点)
    volatile Thread waiter;// 等待写锁的线程
    volatile int lockState;// 读写锁状态
    static final int WRITER = 1;// 持有写锁时设置
    static final int WAITER = 2;// 等待写锁时设置
    static final int READER = 4;// 设置读锁的增量值
    
    // 根据b链表构建TreeBin结点 => 类似于HashMap#treeify(Node)
    TreeBin(TreeNode<K,V> b) {
        // 创建null键、null值、next指针也为null、hash值为TREEBIN(-2)的Node结点
        super(TREEBIN, null, null, null);

        // b链表头结点设置为first结点
        this.first = b;

        // First结点b, 当前遍历结点x, x后继next, 根结点r, 直到遍历到b链尾
        TreeNode<K,V> r = null;
        for (TreeNode<K,V> x = b, next; x != null; x = next) {
            next = (TreeNode<K,V>)x.next;

            // 置空x的左右孩子
            x.left = x.right = null;

            // 如果r为null, 说明根结点还没设置, 则设置x为根结点, 更新r指针指向x
            if (r == null) {
                x.parent = null;
                x.red = false;
                r = x;
            }

            // 如果r不为null, 说明根结点已经设置了, 此时x不再为根结点, 则获取x的键k, x的hash值h, kc为x的Class对象
            else {
                K k = x.key;
                int h = x.hash;
                Class<?> kc = null;

                // 从根结点r开始遍历, 当前遍历结点p, 比较结果dir, p的hash值ph, p的键pk
                for (TreeNode<K,V> p = r;;) {
                    int dir, ph;
                    K pk = p.key;

                    // 如果ph大于x的hash值h, 说明x应该往左子树方向插入, 则dir为-1
                    if ((ph = p.hash) > h)
                        dir = -1;

                    // 如果ph小于x的hash值h, 说明x应该往右子树方向插入, 则dir为1
                    else if (ph < h)
                        dir = 1;

                    // 如果ph等于x的hash值h, 则还需要比较键的Class, 此时dir为其比较结果
                    else if ((kc == null &&
                              (kc = comparableClassFor(k)) == null) ||
                             (dir = compareComparables(kc, k, pk)) == 0)
                        // 当 hashCode 相等且不可比较时，用于对插入进行排序的打破平局实用程序
                        dir = tieBreakOrder(k, pk);

                    // 到这里, x与p的比较结果dir已经知道了, 如果dir小于等于0, 则x作为p左孩子, 否则x作为p右孩子, 并插入结点后平衡红黑树
                    TreeNode<K,V> xp = p;
                    if ((p = (dir <= 0) ? p.left : p.right) == null) {
                        x.parent = xp;
                        if (dir <= 0)
                            xp.left = x;
                        else
                            xp.right = x;
                        // 插入结点后平衡红黑树
                        r = balanceInsertion(r, x);
                        break;
                    }
                }
            }
        }

        // b链表遍历完毕, 说明r结点上的红黑树已经构建完毕, 则设置r为TreeBin桶的root指针
        this.root = r;

        // 递归检查指定结点是否为红黑树
        assert checkInvariants(root);
    }
}

```

###### 读写锁控制

- **lockRoot（）**：获取写锁，底层依赖contendedLock（）来争抢写锁。
  - CAS方式设置lockState为1(持有写锁)，如果CAS设置成功，说明当前线程获取到写锁，此时直接返回即可。 
  - 如果CAS设置失败，说明当前线程获取不到写锁，此时需要继续争抢写锁。
- **contendedLock（）**：争抢写锁，争抢不到写锁的会进入阻塞状态，直到所有调用TreeBin#find（int，Object）的线程调用完毕后唤醒，从而重新争抢写锁。
  - 如果当前红黑树不存在写或者读线程，则当前线程去**竞争写锁**，如果竞争成功则返回。
  - 如果当前红黑树写锁或者读锁正在被持有，且不存在等待写锁的线程，则当前线程去**竞争成为等待锁的线程**，竞争成功则成为等待锁的线程。
  - 如果当前线程成为等待写锁的线程，但**竞争写锁失败，则进入阻塞状态**。
  - 而那些争抢不到写锁，也进入不了阻塞状态成为等待写锁的线程，会**一直自旋等待锁状态变更**。
- **unlockRoot（）**：重置锁状态，释放读锁/写锁。

```java
// CAS方式设置lockState为1(持有写锁), 如果CAS设置成功, 说明当前线程获取到写锁, 此时直接返回即可; 如果CAS设置失败, 说明当前线程获取不到写锁, 此时需要继续争抢写锁
private final void lockRoot() {
    if (!U.compareAndSwapInt(this, LOCKSTATE, 0, WRITER))
        contendedLock();
}

/**
 * 争抢写锁，争抢不到写锁的会进入阻塞状态，直到所有调用TreeBin#find（int，Object）的线程调用完毕后唤醒，从而重新争抢写锁。
 * a. 如果当前红黑树不存在写或者读线程, 则当前线程去竞争写锁, 如果竞争成功则返回;
 * b. 如果当前红黑树写锁或者读锁正在被持有, 且不存在等待写锁的线程，则当前线程去竞争成为等待锁的线程, 竞争成功则成为等待锁的线程;
 * c. 如果当前线程成为等待锁的线程, 但竞争读/写锁失败, 则进入阻塞状态;
 * d. 而那些争抢不到写锁, 也进入不了阻塞状态成为等待写锁的线程, 会一直自旋等待锁状态变更;
 */
private final void contendedLock() {
    boolean waiting = false;

    // 开始自旋
    for (int s;;) {
        // 如果锁状态非101, 说明当前红黑树不存在写或者读线程, 此时允许当前线程去竞争锁
        if (((s = lockState) & ~WAITER) == 0) {
            // CAS设置lockState为1(持有写锁), 如果CAS成功, 说明当前线程成功获取到写锁, 此时置空等待线程waiter, 然后返回即可
            if (U.compareAndSwapInt(this, LOCKSTATE, s, WRITER)) {
                if (waiting)
                    waiter = null;
                return;
            }
        }

        // 如果锁状态非010, 说明当前红黑树不存在等待写锁的线程, 此时允许当前线程成功为等待锁的线程
        else if ((s & WAITER) == 0) {
            // CAS设置lockState为011或者110(等待读锁或者等待写锁的状态), 如果CAS成功说明当前线程成功成为等待锁的线程, 此时设置waiter为当前线程
            if (U.compareAndSwapInt(this, LOCKSTATE, s, s | WAITER)) {
                waiting = true;
                waiter = Thread.currentThread();
            }
        }
        
        // 如果当前线程成为等待写锁的线程, 但竞争写锁失败, 则进入阻塞状态
        else if (waiting)
            LockSupport.park(this);
    }
}

// 重置锁状态, 释放读锁/写锁
private final void unlockRoot() {
    lockState = 0;
}


```

###### static方法 - 左旋

- **背景**：在结点的添加和删除后，**为了避免子树高度变化，需要通过子树内部调整来保证树达到平衡**，其中2-3-4树是通过结点旋转和结点元素变化实现的，红黑树是通过结点旋转和变色实现。
- **左右左右右**：以x结点作为旋转点进行**左**旋，旋转后，x的**右**结点p成为x的父结点，p原本的**左**结点成为x结点的**右**结点，p原本的**右**结点保持不变。

```java
// 左旋p结点，左右左右右 => 同HashMap#rotateLeft(TreeNode, TreeNode)
static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root,
                                      TreeNode<K,V> p) {
    TreeNode<K,V> r, pp, rl;

    // 如果p的右节点r不为null，则有必要进行左旋，否则直接返回root即可
    if (p != null && (r = p.right) != null) {
        // 如果p有右结点r，且r还有左结点rl, 则旋转后成为p的右结点，并连接上rl与p的关系
        if ((rl = p.right = r.left) != null)
            rl.parent = p;

        // 如果p有父结点，则关联p的父结点pp与p的右结点r的关系，且如果pp为null，说明需要更新r为根结点，以及变为黑色
        if ((pp = r.parent = p.parent) == null)
            (root = r).red = false;

        // 如果pp不为null，说明r不需要成为根结点，则关联pp与r的关系
        else if (pp.left == p)
            pp.left = r;
        else
            pp.right = r;

        // 关联p结点与r结点的关系
        r.left = p;
        p.parent = r;
    }

    return root;
}


```

###### static方法 - 右旋

- **背景**：在结点的添加和删除后，**为了避免子树高度变化，需要通过子树内部调整来保证树达到平衡**，其中2-3-4树是通过结点旋转和结点元素变化实现的，红黑树是通过结点旋转和变色实现。
- **右左右左左**：以x结点作为旋转点进行**右**旋，旋转后，x的**左**结点p成为x的父结点，p原本的**右**结点成为x结点的**左**结点，p原本的**左**结点保持不变。

```java
// 右旋p结点，右左右左左 => 同HashMap#rotateRight(TreeNode, TreeNode)
static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root,
                                       TreeNode<K,V> p) {
    TreeNode<K,V> l, pp, lr;

    // 如果p的左节点l不为null，则有必要进行右旋，否则直接返回root即可
    if (p != null && (l = p.left) != null) {
        // 如果p有左结点l，且l还有右结点lr, 则旋转后成为p的左结点，并连接上rl与p的关系
        if ((lr = p.left = l.right) != null)
            lr.parent = p;

        // 如果p有父结点，则关联p的父结点pp与p的左结点r的关系，且如果pp为null，说明需要更新l为根结点，以及变为黑色
        if ((pp = l.parent = p.parent) == null)
            (root = l).red = false;

        // 如果pp不为null，说明l不需要成为根结点，则关联pp与l的关系
        else if (pp.right == p)
            pp.right = l;
        else
            pp.left = l;

        // 如果pp不为null，说明l不需要成为根结点，则关联pp与l的关系
        l.right = p;
        p.parent = l;
    }

    return root;
}


```

###### static方法 - 插入结点后平衡红黑树

**balanceInsertion（TreeNode，TreeNode）**：对应2-3-4树的情况：

- **a. 空结点新增**：成为一个2结点，插入前树为null，插入后x需要变黑色，作为根结点。
- **b. 合并到2结点中**：成为一个3结点，插入前2结点为黑色，插入后无论是（上黑下左红 |  上黑下右红）, 都符合3结点要求，因此无需调整。
- **c. 合并到3结点中**：成为一个4结点，插入前为3结点（上黑下左红 |  上黑下右红），插入后成为4结点黑红红的情况，根据x插入位置不同分为6种情况：
  - **c.2.1.**  左三(中左左*) ，黑红红，不符合红黑树定义 => 需要调整，则中1右旋，中1变红，左1变黑。
  - **c.2.2.** 中左右*(其实就相当于左三，因为对父结点进行左旋，即得到左三) ，黑红红，不符合红黑树定义 => 需要调整，则左1左旋（得到左三），中1右旋，中1变红，新左变黑。
  - **c.2.3.** 右三(中 右右*) ，黑红红，不符合红黑树定义 => 需要调整，则中1左旋，中1变红，右1变黑。
  - **c.2.4.** 中 右左*(其实就相当于右三，因为对父结点进行右旋，即得到右三) 黑红红，不符合红黑树定义 => 需要调整，则右1右旋（得到右三），中1左旋，中1变红，新右变黑。
  - **c.2.5.** 中左 右*，黑红 红，符合红黑树定义 => 无需调整。
  - **c.2.6.** 中左* 右，黑红 红，符合红黑树定义 => 无需调整。
- **d. 合并到4结点中**：成为一个裂变状态（变色后相当于升元了），插入前为4结点（黑红红），插入后4结点颜色反转，爷结点成为新的x结点，准备下一轮的向上调整，根据x插入的位置不同分为4种情况：
  - **d.2.1.** 中左左* 右(黑红红 红)，不符合红黑树定义 => 需要调整，则中变红，左1变黑，左2保持为红， 右1变黑，中看作为“插入结点”，继续向上调整。
  - **d.2.2.** 中左右* 右(黑红红 红)，不符合红黑树定义 => 需要调整，则中变红，左1变黑，右1保持为红，右2变黑，中看作为“插入结点”，继续向上调整。
  - **d.2.3.** 中左 右左*(黑红 红红)，不符合红黑树定义 => 需要调整，则中变红，左1变黑，左2保持为红，右1变黑，中看作为“插入结点”，继续向上调整。
  - **d.2.4.** 中左 右右*(黑红 红红)，不符合红黑树定义 => 需要调整，则中变红，左1变黑，右1变黑，右2保持为红，中看作为“插入结点”，继续向上调整。

```java
// 插入后平衡红黑树，到了这步红黑树结点的二叉树指针关系已经确定好了的, 只需要进行调整和变色就可以了 => 同HashMap#balanceInsertion(TreeNode, TreeNode)
static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root,
                                            TreeNode<K,V> x) {
    x.red = true;// 标记插入结点x为红结点

    // 从x结点向上遍历, 发现需要调整则进行调整
    for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
        // x结点父结点xp, 如果为null, 说明x为根结点，相当于情况a，则设置当前结点为黑色(性质2)
        if ((xp = x.parent) == null) {
            x.red = false;
            return x;
        }

        // 如果x结点不是根结点, 且父结点xp为黑色或者没有爷结点xpp时，相当于情况b，即合并到2结点中，所以无需调整, 直接返回根结点即可。
        else if (!xp.red || (xpp = xp.parent) == null)
            return root;

        // 如果x结点不是根结点，且父节点xp为红色，且有爷结点，且父结点为爷结点的左孩子时，相当于插入到3、4结点中，即情况c.2.1 & c.2.2 & d.2.1 & d.2.2
        if (xp == (xppl = xpp.left)) {// 这里的xppl是为了获取当父结点为爷结点右孩子时的叔结点
            // 如果xpp右孩子叔结点不为null, 且为红结点时，相当于情况d.2.1 和 d.2.2，即合并到4结点，因此x结点需要插入到了父结点xp的孩子结点中，此时标记叔结点为黑色, 父节点为黑色, 爷结点为红色（4结点颜色反转），且设置爷结点为新的x结点, 准备进行下一轮的向上调整
            if ((xppr = xpp.right) != null && xppr.red) {
                xppr.red = false;
                xp.red = false;
                xpp.red = true;
                x = xpp;
            }

            // 如果x没有叔结点，或者叔结点为黑色（不会出现，因为黑色不平衡），相当于插入到3结点中
            else {
                // 如果x结点为父节点xp的右孩子，即父结点为爷结点的左孩子且为红结点, x结点插入到父结点的右孩子时(相当于c.2.2 中左右*, 合并到3结点)
                if (x == xp.right) {
                    // 此时将父结点xp左旋即可得到c.2.1左三的情况（相当于x结点与父结点交换了位置），x赋值为了xp
                    root = rotateLeft(root, x = xp);

                    // 左旋后更新指针：原来的x结点作为xp, 如果为null(不会出现), 则xpp设置为null, 否则xpp设置为原来的x结点的爷结点，x为原来x结点的父结点
                    xpp = (xp = x.parent) == null ? null : xp.parent;

                    // 这步if其实做的是把c.2.2的情况转换为c.2.1左三的情况
                }

                // 到这里，无论是哪种情况，如果xp不为null那肯定都是红色, 且x肯定为父亲的左孩子(相当于c.2.1, 左三, 合并到3结点) => 父变黑, 爷变红, 爷进行右旋, 变成4结点
                if (xp != null) {
                    xp.red = false;
                    if (xpp != null) {
                        xpp.red = true;
                        root = rotateRight(root, xpp);
                    }
                }
            }
        }

        // 如果x结点不是根结点，且父节点xp为红色，且有爷结点，且父结点为爷结点的右孩子时，相当于插入到3、4结点中，即情况c.2.3 & c.2.4 & d.2.3 & d.2.4
        else {
            // 如果xpp做孩子叔结点不为null, 且为红结点时，相当于情况d.2.3 和 d.2.4，即合并到4结点，因此x结点需要插入到了父结点xp的孩子结点中，此时标记叔结点为黑色, 父节点为黑色, 爷结点为红色（4结点颜色反转），且设置爷结点为新的x结点, 准备进行下一轮的向上调整
            if (xppl != null && xppl.red) {
                xppl.red = false;
                xp.red = false;
                xpp.red = true;
                x = xpp;
            }

            // 如果x没有叔结点，或者叔结点为黑色（不会出现，因为黑色不平衡），相当于插入到3结点中
            else {
                // 如果x结点为父节点xp的左孩子，即父结点为爷结点的右孩子且为红结点, x结点插入到父结点的左孩子时(相当于c.2.4 中右左*, 合并到3结点)
                if (x == xp.left) {
                    // 此时将父结点xp右旋即可得到c.2.3右三的情况（相当于x结点与父结点交换了位置），x赋值为了xp
                    root = rotateRight(root, x = xp);

                    // 右旋后更新指针：原来的x结点作为xp, 如果为null(不会出现), 则xpp设置为null, 否则xpp设置为原来的x结点的爷结点，x为原来x结点的父结点
                    xpp = (xp = x.parent) == null ? null : xp.parent;

                    // 这步if其实做的是把c.2.4的情况转换为c.2.3右三的情况
                }

                // 到这里，无论是哪种情况，如果xp不为null那肯定都是红色, 且x肯定为父亲的右孩子(相当于c.2.3, 右三, 合并到3结点) => 父变黑, 爷变红, 爷进行左旋, 变成4结点
                if (xp != null) {
                    xp.red = false;
                    if (xpp != null) {
                        xpp.red = true;
                        root = rotateLeft(root, xpp);
                    }
                }
            }
        }
    }
}

```

###### static方法 - 删除结点前/后平衡红黑树

**balanceDeletion（TreeNode，TreeNode）**：删除结点前/后平衡红黑树，如果x所在结点为2-3-4树的2结点，则平衡后再删除，如果x所在结点为3结点或者4结点，在平衡前对应的结点就已经删除了，此时x作为该结点的替代结点而保留下来：

- **x自己搞得定**：
  - 自己搞得定的意思就是，可以**在自己结点内部处理完毕**（对应2-3-4树结构），不影响其他树的结构。
  - **a.1. x为3结点或者4结点的红结点**：直接置黑返回x结点即可调整完毕（因为x是作为替代结点而保留下来的），然后交由上层方法删除x结点。
- **x自己搞不定，兄弟搞得定**：
  - 自己搞不定的意思就是，自身结点为黑结点，如果直接删除会导致父结点所在的树黑色不平衡。
  - 兄弟搞得定的意思就是，**兄弟结点存在多余的子结点**（即兄弟结点为3结点或者4结点），此时，x的父结点就可以借出结点下来合并到x结点中，兄弟结点再借出结点合并到父结点中，这样x就可以顺利删除了，同时2-3-4树的结构还保持不变。
  - 但是，**前提是x的兄弟结点是真正的兄弟结点，即为黑色的结点**，如果为红色的结点，说明其只是父结点（3结点）的红结点，此时需要对父结点进行旋转，以保证x有真正的兄弟结点。
  - **b.1. 兄弟结点为3结点，但无右（左）**：在x在左子树一方时，x的兄弟结点xpr为右子树，如果xpr无右孩子在对父结点左旋时，会导致xpr为null，导致2-3-4树的结构不正确，因此，**b.1是一个临时情况，需要对xpr进行右旋，转换为b.2有右进一步处理**。x为右子树一方时则相反。
  - **b.2. 兄弟结点为3结点，但有右（左）**：在x在左子树一方时，x的兄弟结点xpr为右子树，如果xpr有右，则可以顺利地对父结点xp进行左旋。左旋后，在2-3-4树结构看来，xp作为xpr的左孩子（相当于父结点借出去一个结点，合并到x结点中），xpr作为xp的父亲（**相当于兄弟结点借出去一个结点，合并到父结点中**），xpr借出去的结点颜色为xp借出去的结点颜色，xp借出去的结点颜色一定要为黑色（相当于3结点），xpr剩余结点一定要为黑色（相当于叶子结点），返回x结点即可调整完毕，交由上层方法删除x结点。x为右子树一方时则相反。
  - **b.3. 兄弟结点为4结点，肯定有右**：在x在左子树一方时，x的兄弟结点xpr为右子树，如果xpr有右，则可以顺利地对父结点xp进行左旋。左旋后，在2-3-4树结构看来，xp作为xpr的左孩子（相当于父结点借出去一个结点，合并到x结点中），xpr作为xp的父亲（**相当于兄弟结点借出去一个结点，合并到父结点中，而且还多借出左孩子合并到x结点中**），xpr合并到父结点的颜色为xp借出去的结点颜色（而借出去的左孩子本来为红色所以不用变），xp借出去的结点颜色一定要为黑色（相当于4结点），xpr剩余结点一定要为黑色（相当于叶子结点），返回x结点即可调整完毕，交由上层方法删除x结点。x为右子树一方时则相反。
  - 在b.3中对于兄弟结点为4结点时，兄弟结点可以借出1个结点（需要旋转两次）或者2个结点（只需要旋转一次），**在JDK中无论是HashMap还是TreeMap，都选择借出2个结点，因为可以减少花销。**
- **x自己搞不定，而且兄弟也搞不定**：
  - 自己搞不定的意思就是，自身结点为黑结点，如果直接删除会导致父结点所在的树黑色不平衡。
  - 兄弟也搞不定的意思就是，**兄弟结点也为黑结点，没有多余的子结点**，如果直接删除x，则导致叔结点所在路径多了一个黑色结点，造成黑色不平衡。
  - **c.1. 兄弟结点为2结点**：此时，为了让x能够顺利删除，**兄弟结点需要置红（自损）**，这样x在删除后，x父结点所在树还是黑色平衡的。但是，如果x父结点为黑色，x爷结点所在树则不黑色平衡了（因为父结点这边少了一个黑色结点），所以父结点的叔结点要也要被置红。**因此需要一路向上自损，直到碰到任意一个终止条件即可结束**：
    - **自损的终止条件1（向上碰到根结点）**：经过一路置红叔结点（置红叔结点是没问题的，因为出现该情况是叶子结点为3结点黑黑黑的时候，此时如果叔结点没有孩子结点即为黑色，而对于更上层的叔结点来说，貌似不会出现叔为黑红红这种情况），直到循环到根结点时（因为上面已经没有父节点了），则代表自损完毕，此时整棵树都是黑色平衡的了（都减少了一个黑色结点）。
    - **自损的终止条件2（向上碰到红结点）**：如果碰到红色结点时，只需要把该结点置黑，则不需要在置红叔结点了，此时相当于在父结点这边子树补回了一个黑色结点，而不影响叔结点那边子树的黑色结点数目，因此整棵树还是黑色平衡的。

```java
// 删除结点前/后平衡红黑树，如果x所在结点为2-3-4树的2结点，则平衡后再删除，如果x所在结点为3结点或者4结点，在平衡前对应的结点就已经删除了，此时x作为该结点的替代结点而保留下来 => 同HashMap#balanceDeletion(TreeNode, TreeNode)
static <K,V> TreeNode<K,V> balanceDeletion(TreeNode<K,V> root,
                                           TreeNode<K,V> x) {
    // 传入的根结点root，当前结点x，x的父结点xp，xp的左孩子xpl，xp的右孩子xpr
    for (TreeNode<K,V> xp, xpl, xpr;;)  {
        // 如果x为null，或者为指定的根结点root，则无需调整，直接返回root即可
        if (x == null || x == root)
            return root;

        // 如果x不为null，也不为root，且父节点xp为null，说明为自损的终止条件1（向上碰到根结点），此时x为根结点，此时置黑x结点即可（此时达到整棵树的黑色平衡，因为上一轮已经对叔结点进行置红了），终止循环向上调整，返回x结点作为根结点
        else if ((xp = x.parent) == null) {
            x.red = false;
            return x;
        }

        // 如果x不为null，也不为root，且父结点xp也不为null，且x为红结点，说明为自损的终止条件2（向上碰到红结点），此时置黑x结点即可（此时达到整棵树的黑色平衡，因为上一轮已经对叔结点进行置红了），终止循环向上调整，返回x结点作为根结点。还有一种情况就是，x自己搞的定（3、4结点的红结点时），置黑直接返回即可。
        else if (x.red) {
            x.red = false;
            return root;
        }

        // 如果x不为null，也不为根结点，且父结点也不为null，且x为黑色结点(删除2结点时，此时自己搞不定，需要兄弟和父亲帮忙)，且x为xp的左孩子（这里的xpl也等于x为右孩子时的叔结点），则按左的方式调整
        else if ((xpl = xp.left) == x) {
            // 叔结点xpr为xp的右孩子，如果为红结点时，说明xpr不是x真正的兄弟结点（xpr只是为xp的红结点，此时xp为3结点)，则置黑叔结点xpr，置红父结点xp，对xp进行左旋，左旋后xpr成为xp的父结点，原xpr的左结点成为xp的右结点（这时x与x真正的兄弟结点才一一对应）
            if ((xpr = xp.right) != null && xpr.red) {
                xpr.red = false;
                xp.red = true;
                root = rotateLeft(root, xp);
                xpr = (xp = x.parent) == null ? null : xp.right;
            }

            // 如果xpr为null，说明x没有叔结点，也就是兄弟没有得借，因此当x删除后，xp所在树会少一个黑结点，为了保证黑色平衡，所以把xp赋值给x，继续循环向上调整，直到碰到红结点或者根结点
            if (xpr == null)
                x = xp;// 可以看作xpr为红色（也相当于null），需要向上置红调整

            // 如果xpr不为null，说明x有真正的兄弟xpr，则继续判断xpr的左孩子sl，右孩子sr，看xpr是否有得借
            else {
                TreeNode<K,V> sl = xpr.left, sr = xpr.right;

                // 如果xpr没有孩子结点，则说明xpr没有得借，或者xpr的孩子结点都为黑色，说明xpr为叶子结点，也没有得借，相当于情况c.1，因此当x删除后，xp所在树会少一个黑结点，为了保证黑色平衡，所以把xp赋值给x，继续循环向上调整，直到碰到红结点或者根结点
                if ((sr == null || !sr.red) &&
                    (sl == null || !sl.red)) {
                    xpr.red = true;
                    x = xp;
                }

                // 如果兄弟结点xpr有得借，则继续判断兄弟结点到底是3结点还是4结点
                else {
                    // 如果xpr没有右孩子，为3结点时，对应情况b.1无右，此时对xpr进行右旋成b.2有右的情况，右旋后左孩子成为新的xpr
                    if (sr == null || !sr.red) {
                        if (sl != null)
                            sl.red = false;
                        xpr.red = true;
                        root = rotateRight(root, xpr);
                        xpr = (xp = x.parent) == null ?
                            null : xp.right;
                    }

                    // 如果xpr不为null，说明此时对应情况b.2（xpr为3结点有右）和b.3（xpr为4结点），即xpr为3、4结点的情况，此时需要对xp进行左旋，左旋后xpr成为xp的父结点，sr置黑，xp置黑，并设置x为root结点，代表调整完毕，退出循环
                    if (xpr != null) {
                        xpr.red = (xp == null) ? false : xp.red;
                        if ((sr = xpr.right) != null)
                            sr.red = false;
                    }
                    if (xp != null) {
                        xp.red = false;
                        root = rotateLeft(root, xp);
                    }
                    x = root;// 更新x为root结点, 退出循环, 结束调整
                }
            }
        }

        // 同理右边情况，反向操作，如果x不为null，也不为根结点，且父结点也不为null，且x为黑色结点(删除2结点时，此时自己搞不定，需要兄弟和父亲帮忙)，且x为xp的右孩子，xpl为叔结点（上面已赋值），则按右的方式调整
        else { // symmetric
            // 叔结点xpl为xp的左孩子，如果为红结点时，说明xpl不是x真正的兄弟结点（xpl只是为xp的红结点，此时xp为3结点)，则置黑叔结点xpl，置红父结点xp，对xp进行右旋，右旋后xpl成为xp的父结点，原xpl的右结点成为xp的左结点（这时x与x真正的兄弟结点才一一对应）
            if (xpl != null && xpl.red) {
                xpl.red = false;
                xp.red = true;
                root = rotateRight(root, xp);
                xpl = (xp = x.parent) == null ? null : xp.left;
            }

            // 如果xpl为null，说明x没有叔结点，也就是兄弟没有得借，因此当x删除后，xp所在树会少一个黑结点，为了保证黑色平衡，所以把xp赋值给x，继续循环向上调整，直到碰到红结点或者根结点
            if (xpl == null)
                x = xp;// 可以看作xpl为红色（也相当于null），需要向上置红调整

            // 如果xpl不为null，说明x有真正的兄弟xpl，则继续判断xpl的左孩子sl，右孩子sr，看xpl是否有得借
            else {
                TreeNode<K,V> sl = xpl.left, sr = xpl.right;

                // 如果xpl没有孩子结点，则说明xpl没有得借，或者xpl的孩子结点都为黑色，说明xpl为叶子结点，也没有得借，相当于情况c.1，因此当x删除后，xp所在树会少一个黑结点，为了保证黑色平衡，所以把xp赋值给x，继续循环向上调整，直到碰到红结点或者根结点
                if ((sl == null || !sl.red) &&
                    (sr == null || !sr.red)) {
                    xpl.red = true;
                    x = xp;
                }

                // 如果兄弟结点xpl有得借，则继续判断兄弟结点到底是3结点还是4结点
                else {
                    // 如果xpl没有左孩子，为3结点时，对应情况b.1无左，此时对xpl进行左旋成b.2有左的情况，左旋后右孩子成为新的xpl
                    if (sl == null || !sl.red) {
                        if (sr != null)
                            sr.red = false;
                        xpl.red = true;
                        root = rotateLeft(root, xpl);
                        xpl = (xp = x.parent) == null ?
                            null : xp.left;
                    }

                    // 如果xpl不为null，说明此时对应情况b.2（xpl为3结点有左）和b.3（xpl为4结点），即xpl为3、4结点的情况，此时需要对xp进行右旋，右旋后xpl成为xp的父结点，sl置黑，xp置黑，并设置x为root结点，代表调整完毕，退出循环
                    if (xpl != null) {
                        xpl.red = (xp == null) ? false : xp.red;
                        if ((sl = xpl.left) != null)
                            sl.red = false;
                    }
                    if (xp != null) {
                        xp.red = false;
                        root = rotateRight(root, xp);
                    }
                    x = root;// 更新x为root结点, 退出循环, 结束调整
                }
            }
        }
    }
}

```

###### static方法 - 递归检查指定结点是否为红黑树

- **checkInvariants（TreeNode）**：递归检查红黑树结点t，是否为一颗合法的红黑树。校验了前驱后继与t的关系、左右孩子与t的关系、左右孩子hash值与t的hash值关系、t结点颜色与左右孩子颜色关系（红结点两孩子结点必须为黑色）、递归校验左子树、递归校验右子树。

```java
// 递归检查指定结点是否为红黑树 => 同HashMap#checkInvariants(TreeNode)
static <K,V> boolean checkInvariants(TreeNode<K,V> t) {
    // 待检查结点t，父结点tp, 左孩子tl, 右孩子tr, 前驱tb, 后继tn
    TreeNode<K,V> tp = t.parent, tl = t.left, tr = t.right,
    tb = t.prev, tn = (TreeNode<K,V>)t.next;

    // 待检查结点t，父结点tp, 左孩子tl, 右孩子tr, 前驱tb, 后继tn
    if (tb != null && tb.next != t)
        return false;

    // 如果后继tn不为null，但后继的prev结点不为t结点, 则返回false
    if (tn != null && tn.prev != t)
        return false;

    // 如果父结点tp不为null，但t又不是tp的左右孩子, 则返回false
    if (tp != null && t != tp.left && t != tp.right)
        return false;

    // 如果父结点tp不为null，但t又不是tp的左右孩子, 则返回false
    if (tl != null && (tl.parent != t || tl.hash > t.hash))
        return false;

    // 如果右孩子tr不为null, 且右孩子的父亲不是t或者小于父亲的hash值，则返回false（右孩子的hash值必须大于根结点的hash值）
    if (tr != null && (tr.parent != t || tr.hash < t.hash))
        return false;

    // 如果t是红节点, 且左右孩子都是红结点, 则返回false
    if (t.red && tl != null && tl.red && tr != null && tr.red)
        // bug？如果结点为红结点, 那么左右孩子结点肯定为黑结点! 节点红, 左孩子红, 右孩子黑, 也算红 黑树？估计是不会出现这种情况的，因为不符合2-3-4树结点的特点
        return false;

    // 如果左孩子tl不为叶子结点，则继续校验左子树
    if (tl != null && !checkInvariants(tl))
        return false;

    // 如果右孩子tr不为叶子结点，则继续校验右子树
    if (tr != null && !checkInvariants(tr))
        return false;

    // 如果右孩子tr不为叶子结点，则继续校验右子树
    return true;
}
```

###### 实例方法 - 红黑树结点的添加方法

- **putTreeVal（int，K，V）**：红黑树结点的添加方法（插入成功则返回null，插入失败则返回已经存在的结点）。从根结点遍历比较插入结点x的hash值（小于等于0的说明x应该在左边，大于0的说明x应该在右边），找到合适位置后（叶子结点），维护x与父结点、prev结点、next结点的关系，**插入后竞争TreeBin的写锁**，成功抢到则平衡红黑树，最后返回成功标志null。

```java
// 红黑树结点的添加方法（插入成功则返回null，插入失败则返回已经存在的结点） => 类似HashMap#putTreeVal（HashMap，Node，int，K，V）, 但这里还需要竞争TreeBin桶头结点的写锁
final TreeNode<K,V> putTreeVal(int h, K k, V v) {
    // 比较键的Class对象kc，是否已经找过了searched，根节点root
    Class<?> kc = null;
    boolean searched = false;

    // 从根结点root开始查找，比较结果dir， 比较结点hash值ph，比较结点key值pk，当前遍历结点p
    for (TreeNode<K,V> p = root;;) {
        int dir, ph; K pk;

        // 如果p为null且p不为叶子结点的孩子结点, 说明红黑树根结点还没被初始化, 则把k和v当做根结点插入
        if (p == null) {
            first = root = new TreeNode<K,V>(h, k, v, null, null);
            break;
        }

        // 如果ph大于h，说明插入结点应该在p的左边，dir为-1
        else if ((ph = p.hash) > h)
            dir = -1;

        // 如果ph小于h，说明插入结点应该在p的右边，dir为1
        else if (ph < h)
            dir = 1;

        // 如果ph等于h，且pk为插入结点的key值，或者equals，则说明p就是要插入的位置，此时返回p结点
        else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
            return p;

        // 如果ph等于h，且pk不为插入结点的key值，也不equals，则判断x是否还实现了Comparable接口，如果是则继续比较是否相等
        else if ((kc == null &&
                  (kc = comparableClassFor(k)) == null) ||
                 (dir = compareComparables(kc, k, pk)) == 0) {
            // 如果还没找过，则递归查找左右子树, 直到找到指定hash和key匹配的结点q返回即可
            if (!searched) {
                TreeNode<K,V> q, ch;
                searched = true;
                if (((ch = p.left) != null &&
                     (q = ch.findTreeNode(h, k, kc)) != null) ||
                    ((ch = p.right) != null &&
                     (q = ch.findTreeNode(h, k, kc)) != null))
                    return q;
            }

            // 如果已经递归找过了，当hashCode相等且不可比较时, 用于打破比较平局的情况 => 比较a和b两个对象的ClassName相等以及原始hashCode
            dir = tieBreakOrder(k, pk);
        }

        // 到这里，比较结果dir已经确定，如果dir<=0, 则从p的左子树开始找，否则从p的右子树开始找，p结点备份为xp, p继续作为左孩子或者右孩子，链表头结点f，如果p为null，说明到了叶子结点，则构建TreeNode结点，next指向链表头结点f
        TreeNode<K,V> xp = p;
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            TreeNode<K,V> x, f = first;
            first = x = new TreeNode<K,V>(h, k, v, f, xp);

            // 如果f不为null, 则关联x与f的关系
            if (f != null)
                f.prev = x;

            // 如果dir<=0，说明x应该在xp的左边，则设置x为xp的左孩子
            if (dir <= 0)
                xp.left = x;
            // 如果dir>0，说明x应该在xp的右边，则设置x为xp的右孩子
            else
                xp.right = x;

            // 如果父结点不为红结点, 则插入x结点并标记为红结点即可退出遍历
            if (!xp.red)
                x.red = true;
            // 如果父结点为红结点, 此时如果插入会出现红红情况, 不符合红黑树性质, 因此需要对root结点加锁平衡, 平衡后释放锁结束遍历
            else {
                // CAS方式设置lockState为1(持有写锁时设置), 如果CAS设置成功, 说明当前线程获取到写锁, 此时直接返回即可; 如果CAS设置失败, 说明当前线程获取不到写锁, 此时需要继续争抢写锁
                lockRoot();
                try {
                    // 如果当前线程成功争抢到写锁后, 则进行插入后平衡红黑树
                    root = balanceInsertion(root, x);
                } finally {
                    // 重置锁状态, 释放读锁/写锁
                    unlockRoot();
                }
            }
            break;
        }
    }

    // 递归检查指定结点是否为红黑树
    assert checkInvariants(root);

    // 插入成功则返回null
    return null;
}

```

###### 实例方法 - 红黑树结点的删除方法

**removeTreeNode（HashMap，Node，boolean）**：

- **替代结点**：红黑树是一种自平衡的二叉搜索树，而二叉搜索树删除，本质上是**找前驱或者后继结点来替代删除**（这里是replacement替代p然后删除p）。
- **A. 如果要删除的结点是叶子结点**：则直接删除即可（肯定为黑色）。
- **B. 如果要删除的结点只有一个孩子结点**：则使用孩子结点进行替代，然后删除"替代结点"。
- **C. 如果要删除的结点有两个孩子结点**：则需要找到前驱或者后继进行替代，然后删除"替代结点"。
  - **C.1. 如果替代结点没有孩子结点**：此时所在的结点为2-3-4树的2结点，则直接要"替代结点"即可。
  - **C.2. 如果替代结点有孩子结点且孩子结点为替代方向**：此时所在的结点为2-3-4树的3结点或者4结点，则继续使用孩子结点进行替代，然后“替代结点”即可（二次替代）。
- 无论是哪种情况，红黑树结点的删除方法，都要调用平衡红黑树的方法，在删除结点前/后平衡红黑树。

```java
// 红黑树结点的删除方法 => 类似HashMap#removeTreeNode（HashMap，Node，boolean）, 但这里还需要竞争TreeBin桶头结点的写锁
final boolean removeTreeNode(TreeNode<K,V> p) {
    // p的后继next, p的前驱pred, 根结点r, 根结点的左孩子rl
    TreeNode<K,V> next = (TreeNode<K,V>)p.next;
    TreeNode<K,V> pred = p.prev;  // unlink traversal pointers 取消链接遍历指针
    TreeNode<K,V> r, rl;

    // 如果pred为null, 说明p为桶头结点，则更新后继succ作为新的桶头
    if (pred == null)
        first = next;
    // 如果pred不为null，说明p不为桶头结点，则链接前驱和后继，跳过p结点
    else
        pred.next = next;
    if (next != null)
        next.prev = pred;

    // 链接pred、succ后，如果桶头结点为null，说明桶内已经没有数据了，所以不用再处理了，直接返回true, 说明可以拆除当前红黑树，将其退化成普通链表
    if (first == null) {
        root = null;
        return true;
    }

    // 如果root为null或者没有孩子时，说明红黑树结点太少了，则返回true, 说明可以拆除当前红黑树，将其退化成普通链表
    if ((r = root) == null || r.right == null || // too small
        (rl = r.left) == null || rl.left == null)
        return true;

    // 到这里, 说明真的是要添加p到红黑树中, 则CAS方式设置lockState为1(持有写锁时设置), 如果CAS设置成功, 说明当前线程获取到写锁, 此时直接返回即可; 如果CAS设置失败, 说明当前线程获取不到写锁, 此时需要继续争抢写锁
    lockRoot();
    try {
        // p左孩子pl，p右孩子pr，替代结点replacement（null）
        TreeNode<K,V> replacement;
        TreeNode<K,V> pl = p.left;
        TreeNode<K,V> pr = p.right;

        // 如果p有两个孩子结点，则一路遍历右孩子pr的左孩子sl，直到sl为叶子结点，即查找p的后继s，并交换p和s的颜色
        if (pl != null && pr != null) {
            TreeNode<K,V> s = pr, sl;
            while ((sl = s.left) != null) // find successor
                s = sl;
            boolean c = s.red; s.red = p.red; p.red = c; // swap colors

            // 后继s的右孩子sr, p的父结点pp
            TreeNode<K,V> sr = s.right;
            TreeNode<K,V> pp = p.parent;

            // 如果后继s为p的右孩子，说明s是p的直接后继，直接反转两者的父子关系即可，即s作为父亲，p作为右孩子，使得p交换到了后继s的位置
            if (s == pr) { // p was s's direct parent
                p.parent = s;
                s.right = p;
            }
            // 如果后继s不是p的右孩子，说明s是p的间接后继，则维护s、p、sp、pr的指针关系，使得p交换到了后继s的位置
            else {
                TreeNode<K,V> sp = s.parent;
                if ((p.parent = sp) != null) {
                    if (s == sp.left)
                        sp.left = p;
                    else
                        sp.right = p;
                }
                if ((s.right = pr) != null)
                    pr.parent = s;
            }

            // 交换了p与后继s的位置后，继续更新s、p、sr、sl、pl、pp的关系
            p.left = null;
            if ((p.right = sr) != null)
                sr.parent = p;
            if ((s.left = pl) != null)
                pl.parent = s;
            if ((s.parent = pp) == null)
                r = s;
            else if (p == pp.left)
                pp.left = s;
            else
                pp.right = s;

            // 到这里，p与s交换了位置，并完成了所有父结点、孩子节点的指针关系维护。如果sr不为null，说明s有右孩子（这里由于原s是通过遍历原p的右孩子的所有左孩子找到的，所以原s肯定是没有左孩子的，如果判断没有右孩子，则可以说明原s肯定是没有孩子的），则设置右孩子为替代结点(相当于情况C.2)
            if (sr != null)
                replacement = sr;
            // 否则，如果原s没有右孩子，则交换后的p就为替代结点(相当于情况C.1)
            else
                replacement = p;
        }

        // 否则，如果原s没有右孩子，则交换后的p就为替代结点(相当于情况C.1)
        else if (pl != null)
            replacement = pl;

        // 如果实例结点p只有右孩子时，则替代结点为右孩子pr（相当于情况B）
        else if (pr != null)
            replacement = pr;

        // 如果实例结点p只有右孩子时，则替代结点为右孩子pr（相当于情况B）
        else
            replacement = p;

        // 到这一步，原p的替代结点replacement已经找到了。如果replacement不为p结点，说明为2-3-4树3结点或者4结点的红结点，即情况B和C.2，为了让replacement代替p结点，则链接pp与replacement并清空p所有的链接，此后p就没有被任何结点引用等待GC回收，相当于p脱离了红黑树
        if (replacement != p) {
            TreeNode<K,V> pp = replacement.parent = p.parent;
            if (pp == null)
                r = replacement;
            else if (p == pp.left)
                pp.left = replacement;
            else
                pp.right = replacement;
            p.left = p.right = p.parent = null;
        }

        // 删除结点前/后平衡红黑树，如果x所在结点为2-3-4树的2结点，则平衡后再删除x，否则在平衡前对应的结点就已经删除了（此时x作为该结点的替代结点而保留下来），r为根结点
        root = (p.red) ? r : balanceDeletion(r, replacement);

        // 如果替代节点replacement为p结点，说明要删除的是2-3-4树的2结点（相当于情况A和C.1），则脱离p链接
        if (p == replacement) {  // detach pointers
            TreeNode<K,V> pp;
            if ((pp = p.parent) != null) {
                if (p == pp.left)
                    pp.left = null;
                else if (p == pp.right)
                    pp.right = null;
                p.parent = null;
            }
        }
    } finally {
        // 重置锁状态, 释放读锁/写锁
        unlockRoot();
    }

    // 递归检查指定结点是否为红黑树
    assert checkInvariants(root);

    // 返回false, 说明正常插入结点到红黑树了, 不需要拆除当前红黑树退化成普通链表
    return false;
}
```

###### 实例方法 - 红黑树结点的查找方法

- **find（int，Object）**：TreeNode实例方法，根据hash值和key值，从根结点开始查找红黑树结点。底层调用find（int，Object，Class）方法进行查找。
- **find（int，Object，Class）**：TreeNode实例方法，根据hash值、key值和比较对象kc，从根结点开始查找查找红黑树结点。
  - **ph > h**：说明查找结点在左子树。
  - **ph < h**：说明查找结点在右子树。
  - **ph == h**：如果key相等或者equals，则说明p就为要找的hash值为h的结点。否则还说明哈希冲突了，还需要继续遍历查找右子树和左子树。
- **find（int，Object）**：ConcurrentHashMap#TreeBin实例方法，根据hash值和key值，从根结点开始查找红黑树结点。
  - 如果当前红黑树存在写线程或者等待写锁线程，则**以链表的方式去遍历**出红黑树结点并返回。
  - 如果当前红黑树没有写线程或者等待写锁线程，则**叠加读锁状态后，以红黑树方式去遍历**红黑树结点并返回。
  - 最后释放当前读锁状态，如果释放读锁前的状态为读状态，或者为等待写锁状态且存在等待写锁线程， 说明当前线程为最后一个读线程，需要**唤醒因调用contendedLock（）争抢不到写锁而进入阻塞等待的线程**，使其重新争抢写锁。

```java
// 根据hash值和key值，从根结点开始查找红黑树结点 => 同HashMap#getTreeNode（int，Object）
Node<K,V> find(int h, Object k) {
    return findTreeNode(h, k, null);
}

// 根据hash值、key值和比较对象kc，从根结点开始查找查找红黑树结点 => 同HashMap#find（int，Object，Class）
final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {
    if (k != null) {
        TreeNode<K,V> p = this;
        do  {
            // 实例结点p，p的hash值ph，比较结果dir，p的key值pk，p的左孩子pl，p的右孩子pr，要查找的结点q（null）
            int ph, dir; K pk; TreeNode<K,V> q;
            TreeNode<K,V> pl = p.left, pr = p.right;

            // 如果ph大于h，说明要查找的结点在左子树
            if ((ph = p.hash) > h)
                p = pl;

            // 如果ph小于h，说明要查找的结点在右子树
            else if (ph < h)
                p = pr;

            // 如果ph等于h，且pk相等或者equals，说明p就是要找的结点
            else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                return p;

            // 如果ph等于h，但pk不相等且不equals，说明哈希冲突了，需要往后继续遍历；如果pl为null，说明还没有左孩子，则往右子树继续遍历
            else if (pl == null)
                p = pr;

            // 如果ph等于h，但pk不相等且不equals，说明哈希冲突了，需要往后继续遍历；如果pl不为null但pr为null，说明还没有右孩子，则往左子树继续遍历
            else if (pr == null)
                p = pl;

            // 如果ph等于h，但pk不相等且不equals，说明哈希冲突了，需要往后继续遍历，但pl和pr都不为null，此时就需要判断x是否还实现了Comparable接口，如果是则继续比较是否相等，dir取其比较结果；如果dir<0，则p取左子树，否则取右子树
            else if ((kc != null ||
                      (kc = comparableClassFor(k)) != null) &&
                     (dir = compareComparables(kc, k, pk)) != 0)
                p = (dir < 0) ? pl : pr;

            // 如果ph等于h，但pk不相等且不equals，说明哈希冲突了，需要往后继续遍历，但pl和pr都不为null，且比较对象也为null，则递归查找右孩子，如果找到则返回
            else if ((q = pr.findTreeNode(h, k, kc)) != null)
                return q;

            // 如果ph等于h，但pk不相等且不equals，说明哈希冲突了，需要往后继续遍历，但pl和pr都不为null，且比较对象也为null，则递归查找右孩子，如果没找到则p设置为左子树，继续循环查找
            else
                p = pl;
        } while (p != null);
    }

    // 如果确实找不到则返回null
    return null;
}

// 根据hash值和key值，从根结点开始查找红黑树结点: 如果当前红黑树存在写线程或者等待写锁线程, 则以遍历链表的方式去遍历出红黑树结点并返回; 如果当前红黑树没有写线程或者等待写锁线程, 则叠加读锁状态后以红黑树方式去遍历红黑树结点并返回
final Node<K,V> find(int h, Object k) {
    if (k != null) {
        // 从根结点开始遍历, 当前遍历结点e, 锁状态s, e键ek,
        for (Node<K,V> e = first; e != null; ) {
            int s; K ek;

            // 如果锁状态为011, 说明当前红黑树存在写线程或者等待写锁线程, 为了减少锁竞争以便写操作尽快完成, 则以链表的方式去遍历出红黑树结点并返回
            if (((s = lockState) & (WAITER|WRITER)) != 0) {
                if (e.hash == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
                e = e.next;
            }

            // 否则, 说明当前红黑树没有写线程或者等待写锁线程, 则CAS叠加lockState读锁状态(每个读线程叠加一次), 然后再以红黑树方式去遍历红黑树结点并返回
            else if (U.compareAndSwapInt(this, LOCKSTATE, s, s + READER)) {
                TreeNode<K,V> r, p;
                try {
                    // 根据hash值、key值和比较对象kc，从根结点开始查找查找红黑树结点
                    p = ((r = root) == null ? null :
                         r.findTreeNode(h, k, null));
                } finally {
                    // 最后释放当前读锁状态, 如果释放读锁前的状态为读状态, 或者为等待写锁状态且存在等待写锁线程, 说明当前线程为最后一个读线程, 需要唤醒因调用contendedLock（）争抢不到写锁而进入阻塞等待的线程，使其重新争抢写锁
                    Thread w;
                    if (U.getAndAddInt(this, LOCKSTATE, -READER) ==
                        (READER|WAITER) && (w = waiter) != null)
                        LockSupport.unpark(w);
                }
                return p;
            }
        }
    }

    // 如果确实找不到则返回null
    return null;
}
```

#### ConcurrentSkipListMap

![1624537265896](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1624537265896.png)

##### 特点

- ConcurrentSkipListMap，扩展的并发{@link ConcurrentNavigableMap} 实现，**其键是有序的**，默认按照对键进行自然排序，或者根据Map创建时提供的{@link Comparator} 进行排序，具体取决于使用的构造函数。**不允许使用 {@code null} 键或值**，因为无法可靠地区分某些 null 返回值与元素的缺失。
- 相比Collections.synchronizedMap（**TreeMap**），ConcurrentSkipListMap同样也是线程安全的键有序的Map，但ConcurrentSkipListMap支持更高的并发。
- ConcurrentSkipListMap，实现了**SkipLists的并发变体**，为 {@code containsKey}、{@code get}、{@code put} 和 {@code remove} 操作及其变体提供预期的平均 **log(n)** 时间成本。插入、移除、更新和访问操作由多个**线程安全**地并发执行。
- ConcurrentSkipListMap，**迭代器和拆分器是弱一致的**，而升序键有序视图及其迭代器会比降序的更快。映射生成时的快照包含所有的 {@code Map.Entry} ，不支持 {@code Entry.setValue} 方法。
- ConcurrentSkipListMap，与大多数集合不同，**{@code size} 方法不是恒定时间O（n）操作**，因为映射的异步性质，确定当前元素数量需要遍历元素，因此如果在遍历期间修改此集合，则可能会报告不准确的结果。
- 此外，**批量操作** {@code putAll}、{@code equals}、{@code toArray}、{@code containsValue} 和 {@code clear} **并不能保证以原子方式执行**。例如，与 {@code putAll} 操作并发运行的迭代器可能仅查看一些添加的元素。

##### SkipList跳表性质

- 跳表由**多层链表**组成：
  - 一层数据层，多层索引层，**head指针指向最高层的索引头结点**。
  - 数据结点含有3个指针，key指针指向Key对象，value指针指向Value对象，next指针指向下一个数据结点。
  - 索引结点含有3个指针，node指针指向level 0的数据结点，down指针指向level-1的索引结点，right指针指向相同level的索引结点。
  - 其中level是通过一定的概率随机产生的（**索引层级越高，出现的概率越低**，level 1: 50%, level 2: 25%, level 3: 12.5%...）。
- 每一层都是一个**有序链表**，默认为自然排序，也可以指定Comparator排序。
- level 0为**数据层**，数据层含有所有元素。level 0以上都为**索引层**，索引层的结点随机选取。
- 高级别索引的元素集是低级别索引元素集的一个子集，索引的元素集又是数据的元素集的一个子集，即**level 0是任何level的一个并集**。

##### 数据结构

![1624546428005](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1624546428005.png)

##### 构造方法

- **无参构造函数**：创建一个空的排序映射，根据其键的自然顺序排序。
- **指定{@code Comparator} 的构造函数**：创建一个空的排序映射，根据指定的比较器排序。
- **指定复制集合的构造函数**：O（n*logn），创建一个新映射（更新所有元素为传入集合的元素），其键值映射
  与其参数相同，根据键的自然顺序排序。
- **指定 {@code SortedMap} 复制集合的构造函数**：O（n），创建一个新的排序映射（即更新所有元素为传入集合的元素），其键值映射和排序与输入排序映射相同。

```java
public class ConcurrentSkipListMap<K,V> extends AbstractMap<K,V> implements ConcurrentNavigableMap<K,V>, Cloneable, Serializable {
    
    private static final Object BASE_HEADER = new Object();// 基础Level头结点的特殊值
    private transient volatile HeadIndex<K,V> head;// CAS => headOffset
    final Comparator<? super K> comparator;
    private static final int EQ = 1;
    private static final int LT = 2;
    private static final int GT = 0;

    // 数据结点
    static final class Node<K,V> {
        final K key;
        volatile Object value;
        volatile Node<K,V> next;
        
        // 创建一个新的常规节点
        Node(K key, Object value, Node<K,V> next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
        
        // 创建一个新的标记节点，标记的区别在于其value字段指向自身
        Node(Node<K,V> next) {
            this.key = null;
            this.value = this;
            this.next = next;
        }
    }
    
    // 索引结点
    static class Index<K,V> {
        final Node<K,V> node;
        final Index<K,V> down;
        volatile Index<K,V> right;
        
        // 创建具有给定值的索引节点
        Index(Node<K,V> node, Index<K,V> down, Index<K,V> right) {
            this.node = node;
            this.down = down;
            this.right = right;
        }
    }
    
    // 带level的索引头结点
    static final class HeadIndex<K,V> extends Index<K,V> {
        final int level;
        HeadIndex(Node<K,V> node, Index<K,V> down, Index<K,V> right, int level) {
            super(node, down, right);
            this.level = level;
        }
    }
    
    // 创建一个空的排序映射，根据其键的自然顺序排序
    public ConcurrentSkipListMap() {
        this.comparator = null;
        initialize();// 初始化或重置跳跃表状态
    }
    
    // 创建一个空的排序映射，根据指定的比较器排序
    public ConcurrentSkipListMap(Comparator<? super K> comparator) {
        this.comparator = comparator;
        initialize();// 初始化或重置跳跃表状态
    }
    
    // O（n*logn），创建一个新映射（更新所有元素为传入集合的元素），其键值映射与其参数相同，根据键的自然顺序排序
    public ConcurrentSkipListMap(Map<? extends K, ? extends V> m) {
        this.comparator = null;
        initialize();// 初始化或重置跳跃表状态
        putAll(m);
    }
    
    // O（n），创建一个新的排序映射（即更新所有元素为传入集合的元素），其键值映射和排序与输入排序映射相同
    public ConcurrentSkipListMap(SortedMap<K, ? extends V> m) {
        this.comparator = m.comparator();
        initialize();// 初始化或重置跳跃表状态
        buildFromSorted(m);
    }
    
    // 初始化或重置跳跃表状态，会被构造函数、clone、clear、readObject调用，注意比较器必须单独初始
    private void initialize() {
        keySet = null;
        entrySet = null;
        values = null;
        descendingMap = null;

        // 创建level为1的索引结点, 指向新创建的BASE_HEADER数据头结点
        // HeadIndex: { node: [Node: {key: null, value: BASE_HEADER, next: null}], down: null, right: null, level: 1 }
        head = new HeadIndex<K,V>(
                new Node<K,V>(null, BASE_HEADER, null),
                null, null, 1
        );
    }
}
```

##### 顺序Map构建SkipList

- **buildFromSorted（SortedMap）**：
  - 根据顺序Map重新建立跳表，head指针设置为最高层的索引头结点。
  - 索引层级越高，出现的概率越低：level 1: 50%, level 2: 25%, level 3: 12.5%...
  - 上图跳表数据结构可以如下解释：
    - 数据结点Node链表为：BASE_HEADER->1->2->3->4->5。
    - level 1的索引结点Index链表为：BASE_HEADER->2->4->5。
    - level 2的索引结点Index链表为：BASE_HEADER->2->4。
    - level 3的索引结点Index链表为：BASE_HEADER->2，head指针落在该链表的头结点HeadIndex上。

```java
// 根据顺序Map重新建立跳表, head指针设置为最高层的索引头结点 => level 1: 50%, level 2: 25%, level 3: 12.5%, 索引层级越高, 出现的概率越低
private void buildFromSorted(SortedMap<K, ? extends V> map) {
    if (map == null)
        throw new NullPointerException();

    // 构造函数&clone()调用时, h为 HeadIndex: { node: { Node: {key: null, value: BASE_HEADER, next: null} }, down: null, right: null, level: 1 }
    HeadIndex<K,V> h = head;

    // basepred表示最右边的数据结点, basepred为 Node: {key: null, value: BASE_HEADER, next: null}
    Node<K,V> basepred = h.node;

    // Track the current rightmost node at each level. Uses an
    // ArrayList to avoid committing to initial or maximum level.
    // 在每个级别跟踪当前最右边的节点。 使用 ArrayList 避免提交到初始或最大级别。
    // preds第i桶存放第i级的最右边的索引结点(i为0时为null)
    ArrayList<Index<K,V>> preds = new ArrayList<Index<K,V>>();

    // initialize
    // 为每层的最右索引结点占空位
    // eg: h.level=1时, preds[0: null, 1: null]
    for (int i = 0; i <= h.level; ++i)
        preds.add(null);

    // 设置preds数组元素, 设置每层最右的索引结点(i、level一一对应), 此时只有1个结点, 所以为HeadIndex结点
    // eg: h.level=1时, preds[0: null, 1: h]
    Index<K,V> q = h;
    for (int i = h.level; i > 0; --i) {
        preds.set(i, q);
        q = q.down;
    }

    // 使用Node的有序迭代器遍历, 迭代器it, 当前迭代结点e, 伪随机数rnd, 确定到的层级j
    Iterator<? extends Map.Entry<? extends K, ? extends V>> it = map.entrySet().iterator();
    while (it.hasNext()) {
        Map.Entry<? extends K, ? extends V> e = it.next();
        int rnd = ThreadLocalRandom.current().nextInt();
        int j = 0;

        // 0x8000_0001: 最高位为1, 最低位为1 => 排除了负数和奇数, 也就是rnd肯定为正偶数
        if ((rnd & 0x80000001) == 0) {
            do {
                // j为rnd连续1的个数, 因此增加层级的概率是1/2 * 1/2 = 1/4(本层级为1占1/2, +1层级为1占1/2)
                ++j;
            } while (((rnd >>>= 1) & 1) != 0);

            // 当j大于当前层级, 则层级+1赋值给j => level 1: 50%, level 2: 25%, level 3: 12.5% => 索引层级越高, 出现的概率越低
            if (j > h.level) j = h.level + 1;
        }

        // e键k, e值v, 构造数据结点z, 由于map是有序的, 则按顺序建立数据结点
        // z为Node: {key: e.k, value: e.v, null},
        K k = e.getKey();
        V v = e.getValue();
        if (k == null || v == null)
            throw new NullPointerException();
        Node<K,V> z = new Node<K,V>(k, v, null);

        // 链接最右数据结点basepred与z, 此时basepred为 Node: {key: null, value: BASE_HEADER, next: { Node: {key: e.k, value: e.v, null} }}
        basepred.next = z;

        // z成为新的最右结点, 此时basepred为 Node: {key: e.k, value: e.v, null}
        basepred = z;

        // 如果确定到的层级j大于0, 说明j有效, 则从低到高建立索引结点
        if (j > 0) {
            Index<K,V> idx = null;

            // i、j表示从下到上的层级(j为0时表示数据结点), i从1开始表示从第1级的索引结点开始处理
            for (int i = 1; i <= j; ++i) {
                // 基于数据结点z, i=1时, idx表示新建的索引结点, i=2时, idx表示i=1的索引结点
                // eg: i=1时, idx1为 Index: node: { Node: {key: e.k, value: e.v, null} }, down: null, right: null }
                // eg: i=2时, idx2为 Index: node: { Node: {key: e.k, value: e.v, null} }, down: idx1, right: null }
                idx = new Index<K,V>(z, idx, null);

                // 如果层级比h的层级还高, 说明需要建立更高层的HeadIndex结点
                if (i > h.level)
                    // eg: h=1, i=2时, h为 HeadIndex: { node: { Node: {key: null, value: BASE_HEADER, next: null} }, down: {h}, right: idx2, level: 2 }
                    h = new HeadIndex<K,V>(h.node, h, idx, i);
                // 如果i<=h, 则保持h不变
                // eg: h=1, i=1时, h为 HeadIndex: { node: { Node: {key: null, value: BASE_HEADER, next: null} }, down: null, right: null, level: 1 }

                // preds#size-1可以表示索引结点的级高(0表示数据结点), 如果i小于size, 表示i层还没超出以前的层级, 则需要把i层idx结点右移一位
                // eg: h.level=1时, preds[0: null, 1: h], 此时i确实小于size
                if (i < preds.size()) {
                    // eg: i=1时, preds[1].right => h.right = idx1, 相当于i层结点right指针追加idx
                    preds.get(i).right = idx;

                    // 再设置preds[1] = idx1, 相当于i层的结点右移一个结点
                    preds.set(i, idx);
                }
                // 如果i等于size(永远不会大于), 表示i层超出了1层最高层, 则往preds继续追加一个索引结点, 待后面遍历时补上
                // eg: i=2时, 则preds[0: null, 1: h, 2: idx2]
                else
                    preds.add(idx);
            }
        }
    }

    // head指针设置为最高层的索引头结点
    head = h;
}
```

##### 迭代方法

- **抽象的ConcurrentSkipListMap迭代器**：ConcurrentSkipListMap迭代器基类，**非快速失败机制**，提供advance方法提前提前缓存nextValue，因此ConcurrentSkipListMap迭代器是**弱一致性**的。
- **Map.Entry迭代器**：继承ConcurrentSkipListMap迭代器基类，**非快速失败机制**，利用父类的advance方法提前提前缓存nextValue，因此该迭代器是**弱一致性**的。返回AbstractMap.SimpleImmutableEntry即Map.Entry**快照**，该快照不支持 {@code Entry.setValue} 方法。
- **ConcurrentSkipListMap.Node#Key迭代器**：继承ConcurrentSkipListMap迭代器基类，**非快速失败机制**，利用父类的advance方法提前提前缓存nextValue，因此该迭代器是**弱一致性**的。返回ConcurrentSkipListMap.Node#Key对象。
- **ConcurrentSkipListMap.Node#Value迭代器**：继承ConcurrentSkipListMap迭代器基类，**非快速失败机制**，利用父类的advance方法提前提前缓存nextValue，因此该迭代器是**弱一致性**的。返回ConcurrentSkipListMap.Node#Value对象。

```java
// ConcurrentSkipListMap迭代器基类
abstract class Iter<T> implements Iterator<T> {
    
    Node<K,V> lastReturned;// next() 返回的最后一个节点
    Node<K,V> next;// 从 next() 返回的下一个节点；
    V nextValue;// 缓存下一个值字段以保持弱一致性

    public final boolean hasNext() {
        return next != null;
    }

    // 提前缓存nextValue
    final void advance() {
        if (next == null)
            throw new NoSuchElementException();
        lastReturned = next;
        while ((next = next.next) != null) {
            Object x = next.value;
            if (x != null && x != next) {
                @SuppressWarnings("unchecked") V vv = (V)x;
                nextValue = vv;
                break;
            }
        }
    }
    ...
}

// Map.Entry迭代器
final class EntryIterator extends Iter<Map.Entry<K,V>> {
    public Map.Entry<K,V> next() {
        Node<K,V> n = next;
        V v = nextValue;
        advance();// 提前缓存nextValue
        return new AbstractMap.SimpleImmutableEntry<K,V>(n.key, v);
    }
}

// ConcurrentSkipListMap.Node#Key迭代器
final class KeyIterator extends Iter<K> {
    public K next() {
        Node<K,V> n = next;
        advance();
        return n.key;
    }
}

// ConcurrentSkipListMap.Node#Value迭代器
final class ValueIterator extends Iter<V> {
    public V next() {
        V v = nextValue;
        advance();
        return v;
    }
}
```

##### 扩容方法

链表方式实现，不需要扩容机制。

##### 清除索引结点方法

- **findPredecessor（Object，Comparator）**：查找比刚好key小一点的数据结点(**key前驱**)，如果找不到则会返回BASE_HEADER，查找同时还会**清除value为null的数据结点的索引结点**。

```java
// 查找比刚好key小一点的数据结点(key前驱), 如果找不到则会返回BASE_HEADER, 查找同时还会清除value为null的数据结点的索引结点
private Node<K,V> findPredecessor(Object key, Comparator<? super K> cmp) {
    if (key == null)
        throw new NullPointerException(); // don't postpone errors

    // 开始自旋, 键key, 比较器cmp
    for (;;) {
        // 从head指针开始遍历, 当前遍历结点q, q的后继r, q的下结点d, 右结点r对应的数据结点n, n键k
        for (Index<K,V> q = head, r = q.right, d;;) {
            // 如果q存在后继r, 分两种情况处理, value值为null的会被清除, 否则比较键值继续往后遍历同一层索引
            if (r != null) {
                Node<K,V> n = r.node;
                K k = n.key;

                // 如果n结点value为null, 说明n结点是删除的, 则CAS解除后继r的链接
                if (n.value == null) {
                    if (!q.unlink(r))
                        // 如果CAS更新失败, 说明q或者r已被其他线程改变, 则进入下一轮自旋, 重新获取head指针, 重新遍历
                        break;           // restart

                    // 如果CAS更新成功, 则获取最新r指针为q最新的后继, 继续下一轮遍历
                    r = q.right;         // reread r
                    continue;
                }

                // n结点value不为null, 则使用比较器或者自然排序比较key与r.node的键, 如果比较结果大于0, 说明r.node键k小了(用于定位r的位置, 保证r.key刚好小于key), 则继续往前遍历, 直到r.node键刚好等于key或者大于key
                if (cpr(cmp, key, k) > 0) {
                    q = r;
                    r = r.right;
                    continue;
                }
            }

            // 经过上面的判断, 因为r.key刚好等于或者大于key, 此时q肯定为比key小的结点, 如果q下结点为null, 说明最后一层索引层, 此时返回q的数据结点即可
            if ((d = q.down) == null)
                return q.node;

            // 经过上面的判断, 因为r.key刚好等于或者大于key, 此时q肯定为比key小的结点, 如果q下结点不为null, 说明还没达到最后一层索引层, 此时索引往下走一层
            q = d;
            r = d.right;
        }
    }
}
```

##### 清除空索引层方法

- **tryReduceLevel（）**：
  - 尝试减少索引层级, 如果head、head.down、head.down.down**3层都没有索引结点**了（即只剩下HeadIndex时），则删除head层，使head指向下一层。
  - 但如果删除后复检时发现原head又多了索引结点, 则又会恢复原状, 取消删除。

```java
// 尝试减少索引层级, 如果head、head.down、head.down.down3层都没有索引结点了(即只剩下HeadIndex时), 则删除head层, 使head指向下一层; 但如果删除后复检时发现原head又多了索引结点, 则又会恢复原状, 取消删除
private void tryReduceLevel() {
    // head指针h, h下结点d, d下结点e
    HeadIndex<K,V> h = head;
    HeadIndex<K,V> d;
    HeadIndex<K,V> e;

    // 如果h的层级大于3, 且d、e不为null, 且h、d、e层只有HeadIndex结点(即没有索引结点时), 则CAS更新h到h.down
    if (h.level > 3 &&
        (d = (HeadIndex<K,V>)h.down) != null &&
        (e = (HeadIndex<K,V>)d.down) != null &&
        e.right == null &&
        d.right == null &&
        h.right == null &&
        casHead(h, d) && // try to set
        h.right != null) // recheck

        // CAS更新h为d成功后, 再次检查h层是否还有索引结点, 如果确实还有, 那再把h改为原来的h(即取消删除), 否则直接返回
        casHead(d, h);   // try to backout
}
```

##### 清除数据结点方法

- **helpDelete（Node，Node）**：通过标记后继结点辅助删除实例结点，在实例结点**value为null时调用**。

```java
// 如果后继n值value为null, 说明n已经被删除了, 则通过标记后继结点辅助删除实例结点, 在实例结点value为null时调用, 删除后重新自旋, 以获取新的b
if ((v = n.value) == null) {    // n is deleted
    n.helpDelete(b, f);
    break;
}

// 通过标记后继结点辅助删除实例结点, 在实例结点value为null时调用
void helpDelete(Node<K,V> b, Node<K,V> f) {
    
    // 前驱b, 后继f, 当前要删除的结点this, 如果f还为后继, b仍为前驱, 说明this还没被其他线程删除
    if (f == next && this == b.next) {
        
        // 如果f为null(bug?), 或者f不为null且f值不为自身, 说明f仍未标记, 则构造新的后继结点, 另原本n的后继f为标记结点
        if (f == null || f.value != f) // not already marked 尚未标记
            casNext(f, new Node<K,V>(f));

        // 如果f不为null, 且f值为自身, 则链接n的前驱与f.next(新构建的f结点, 其next还是指向正常的node结点), 此时完成n与旧f的脱钩
        else
            b.casNext(this, f.next);

        // 而f为null, 且n也为null的情况, 可视它们为null的链尾, 不需要处理
    }
}
```

##### 清除索引结点与数据结点方法

- **findNode（Object）**：
  - 返回持有key的结点(如果没有则返回null)，同时清除遍历key沿途看到的任何已删除节点，包括**数据结点和索引结点**。
  - 底层调用了findPredecessor（Object，Comparator）来清除索引结点，调用了helpDelete（Node，Node）来清除数据结点。

```java
// 返回持有key的结点(如果没有则返回null), 同时清除遍历key沿途看到的任何已删除节点, 包括数据结点和索引结点
private Node<K,V> findNode(Object key) {
    if (key == null)
        throw new NullPointerException(); // don't postpone errors
    Comparator<? super K> cmp = comparator;

    // 开始自旋, 比较器comparator, 键key, key前驱b, b后继n, n后继f, n的值v, 比较结果c
    outer: for (;;) {
        // 查找比刚好key小一点的数据结点(key前驱), 如果找不到则会返回BASE_HEADER, 查找同时还会清除value为null的数据结点的索引结点
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            Object v; int c;

            // 如果key要所在位置为null, 说明没找到key对应的结点, 此时结束自旋, 返回null
            if (n == null)
                break outer;

            // 如果n不为b后继了, 则说明b的后继被其他线程更改了, 此时重新自旋, 以获取新的b
            Node<K,V> f = n.next;
            if (n != b.next)                // inconsistent read
                break;

            // 如果后继n值value为null, 说明n已经被删除了, 则通过标记后继结点辅助删除实例结点, 在实例结点value为null时调用, 删除后重新自旋, 以获取新的b
            if ((v = n.value) == null) {    // n is deleted
                n.helpDelete(b, f);
                break;
            }

            // 如果key前驱b值为null, 或者n为标记结点, 说明b是要被删除的, 此时重新自旋, 以获取新的b
            if (b.value == null || v == n)  // b is deleted
                break;

            // 如果b和n都不是已删除的结点, 则比较key与n.key, 如果key == n.key, 说明n就是要找结点结点, 此时返回n结点
            if ((c = cpr(cmp, key, n.key)) == 0)
                return n;

            // 如果c小于0, 即key < n.key, 说明b遍历到了链尾也没找到key对应的结点, 此时结束自旋, 返回null
            if (c < 0)
                break outer;

            // 如果c大于0, 即key > n.key, 说明key对应的结点可能还在b后面, 则继续遍历b链表
            b = n;
            n = f;
        }
    }
    return null;
}
```

##### 添加元素方法

**所有方法键都不能为null**。

- **put（K，V）**：将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则会替换旧值。
- **putIfAbsent（K，V）**：将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值。
- **putAll（Map）**：根据复制集合元素添加元素，AbstractMap#putAll实现的默认方法，底层调用put方法。

```java
// 将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则会替换旧值
public V put(K key, V value) {
    if (value == null)
        throw new NullPointerException();
    return doPut(key, value, false);
}

// 将指定值和指定键相关联，形成key-value键值对，如果包含相同的键，则不会替换旧值
public V putIfAbsent(K key, V value) {
    if (value == null)
        throw new NullPointerException();
    return doPut(key, value, true);
}

// 根据复制集合元素添加元素，AbstractMap#putAll实现的默认方法，底层调用put方法
public void putAll(Map<? extends K, ? extends V> m) {
    for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
        put(e.getKey(), e.getValue());
}

// put方法核心逻辑, onlyIfAbsent为true代表只允许不存在时插入(即存在时不允许插入或者替换), 此时返回n值; 否则替换n值, 返回value值
private V doPut(K key, V value, boolean onlyIfAbsent) {
    Node<K,V> z;             // added node
    if (key == null)
        throw new NullPointerException();
    Comparator<? super K> cmp = comparator;

    // 开始自旋, 要添加的结点z, 比较器comparator, 键key, 值value, key前驱b, b后继n, n后继f, n的值v, 比较结果c
    outer: for (;;) {
        // 查找比刚好key小一点的数据结点(key前驱), 如果找不到则会返回BASE_HEADER, 查找同时还会清除value为null的数据结点的索引结点
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            // 如果key要插入位置存在后继
            if (n != null) {
                Object v; int c;
                Node<K,V> f = n.next;

                // 如果n不为b后继了, 则说明b的后继被其他线程更改了, 此时重新自旋, 以获取新的b
                if (n != b.next)
                    break;

                // 如果后继n值value为null, 说明n已经被删除了, 则通过标记后继结点辅助删除实例结点, 在实例结点value为null时调用, 删除后重新自旋, 以获取新的b
                if ((v = n.value) == null) {
                    n.helpDelete(b, f);
                    break;
                }

                // 如果key前驱b值为null, 或者n为标记结点, 说明b是要被删除的, 此时重新自旋, 以获取新的b
                if (b.value == null || v == n)
                    break;

                // 如果b和n都不是已删除的结点, 则比较key与n.key, 如果key > n.key, 说明key的位置应该在n的后面, 也就是跳表追加了更接近key的结点, 则继续遍历
                if ((c = cpr(cmp, key, n.key)) > 0) {
                    b = n;
                    n = f;
                    continue;
                }

                // 如果c为0, 说明key == n.key, 说明n就是key对应的结点
                if (c == 0) {
                    // 如果只允许不存在时插入(即存在时不允许插入或者替换), 则返回n的值, 否则替换n值, 返回value值
                    if (onlyIfAbsent || n.casValue(v, value)) {
                        @SuppressWarnings("unchecked") V vv = (V)v;
                        return vv;
                    }

                    // 如果允许存在时插入或者替换, 但CAS更新n值失败了, 说明被其他线程替换掉了, 此时重新自旋, 以获取新的b
                    break;
                }

                // 如果c小于0, 即key < n.key, 且key != b.key, 说明b确实是key的前驱, 则继续往下走, 随机生成并维护索引结点
            }

            // 到这里, key已经找到了插入的位置, 则构造node结点z, key -> n
            z = new Node<K,V>(key, value, n);

            // CAS链接b与key, b -> key, 如果CAS失败, 说明key插入失败, 则重新自旋, 以获取新的b
            if (!b.casNext(n, z))
                break;
            
            // CAS链接b与key, b -> key, 如果CAS成功, 说明key插入成功, 则退出自旋
            break outer;
        }
    }

    // 到这里, key对应的数据结点z已创建并维护完毕, 此时需要随机生成level并维护索引结点
    int rnd = ThreadLocalRandom.nextSecondarySeed();

    // 0x8000_0001: 最高位为1, 最低位为1 => 排除了负数和奇数, 也就是rnd肯定为正偶数
    if ((rnd & 0x80000001) == 0) {
        int level = 1, max;

        // level为rnd连续1的个数, 因此增加层级的概率是1/2 * 1/2 = 1/4(本层级为1占1/2, +1层级为1占1/2)
        while (((rnd >>>= 1) & 1) != 0)
            ++level;

        // 索引结点idx, 顶层索引头结点h
        Index<K,V> idx = null;
        HeadIndex<K,V> h = head;

        // 如果随机生成的level小于等于h的层级, 说明不用更新顶层索引头结点h的指针, 则依次建立1~level级的索引结点, 但还没维护与前驱索引结点的链接
        if (level <= (max = h.level)) {
            for (int i = 1; i <= level; ++i)
                idx = new Index<K,V>(z, idx, null);
        }

        // 如果随机生成的level大于h的层级, 说明需要更新顶层索引头结点head的指针, 则更新head指针(只维护head指针与z对应的idx结点关系, 因为这层只需要维护这个关系即可)
        else {
            level = max + 1;

            // 创建level+1索引结点数组, size-1可以表示索引结点的级高(0表示数据结点), 如果i小于size, 表示i层还没超出以前的层级, 则需要把i层idx结点右移一位
            @SuppressWarnings("unchecked")
            Index<K,V>[] idxs = (Index<K,V>[])new Index<?,?>[level+1];

            // 依次建立1~level级的索引结点, 并存进idxs数组(但还没维护与前驱索引结点的链接), idxs[1~level]分别表示i层z对应的索引结点
            for (int i = 1; i <= level; ++i)
                idxs[i] = idx = new Index<K,V>(z, idx, null);

            // 开始自旋, 再次检查顶层索引头结点h是否被其他线程改变了
            for (;;) {
                h = head;
                int oldLevel = h.level;

                // 如果level不在大于h.level, 说明h确实被其他线程改变了, 则结束自旋, 此后直接维护前驱索引结点即可, 无需更新head指针了
                if (level <= oldLevel) 
                    break;

                // 如果level确实还大于h.level, 说明h确实没被其他线程更改过, 则更新h指针为level层的HeadIndex, 这里只是维护好head指针(方便后面利用head进行每个HeadIndex与每层z对应的idx结点的指针维护)
                HeadIndex<K,V> newh = h;
                Node<K,V> oldbase = h.node;
                for (int j = oldLevel+1; j <= level; ++j)
                    newh = new HeadIndex<K,V>(oldbase, newh, idxs[j], j);

                // 如果CAS更新head指针成功, 则更新h指针指向最新的HeadIndex, level为以前h的level, idx为以前h的level层z对应的索引结点
                if (casHead(h, newh)) {
                    h = newh;
                    idx = idxs[level = oldLevel];
                    break;
                }
                // 如果CAS更新h指针失败, 则h还是为以前那个HeadIndex, level为max+1, idx为max+1层z对应的索引结点, 此时重新自旋, 获取最新的head指针
            }
        }

        // find insertion points and splice in 找到插入点并拼接
        // 开始自旋, insertionLevel为以前h的level, j为新h的level, q为新的h, r为q的后继, t为以前h的level层z对应的索引结点
        splice: for (int insertionLevel = level;;) {
            int j = h.level;

            // 从h开始遍历, 维护每个t的索引结点前驱
            for (Index<K,V> q = h, r = q.right, t = idx;;) {
                // 如果q为null, 或者t为null, 说明到了level 0索引结点(不存在), 代表维护结束, 结束自旋
                if (q == null || t == null)
                    break splice;

                // q不为null, 且t不为null, 说明索引结点q和t都是正常的, 如果r不为null, 说明还没找到原本最右的索引结点
                if (r != null) {
                    Node<K,V> n = r.node;

                    // compare before deletion check avoids needing recheck 在删除检查之前进行比较避免需要重新检查
                    // 使用比较器或者自然排序比较key和后继的数据结点n的键, 比较结果c
                    int c = cpr(cmp, key, n.key);

                    // 如果n结点value为null, 说明n结点是删除的, 则CAS解除后继r的链接
                    if (n.value == null) {
                        // 如果CAS更新失败, 说明q或者r已被其他线程改变, 则进入下一轮自旋, 从头开始遍历
                        if (!q.unlink(r))
                            break;

                        // 如果CAS更新成功, 则获取最新r指针为q最新的后继, 接着遍历即可
                        r = q.right;
                        continue;
                    }

                    // n结点value不为null, 如果比较结果大于0, 说明r.node键k小了(用于定位r的位置, 保证r.key刚好小于key), 则继续往前遍历, 直到r.node键刚好等于key或者大于key
                    if (c > 0) {
                        q = r;
                        r = r.right;
                        continue;
                    }
                }

                // 经过上面的判断, 因为r.key刚好等于或者大于key, 此时q肯定为比key小的结点
                // 如果j减小到了以前h的level, 说明j层适合维护t的前驱, 则CAS更新实例结点q最右结点r为t
                if (j == insertionLevel) {
                    // 如果CAS成功, 则q链表上, 最右索引结点为t
                    if (!q.link(r, t))
                        // 如果CAS更新失败, 说明r已经被更改了, 则重新自旋, 重新从head指针位置开始找过来
                        break; // restart

                    // 如果t的数据结点已经删除, 则清除遍历key沿途看到的任何已删除节点, 包括数据结点和索引结点, 代表无需再维护索引结点了, 结束自旋返回null, 表明插入完成(实际上是被删除了懒得插了)
                    if (t.node.value == null) {
                        findNode(key);
                        break splice;
                    }
                    // t的数据结点没有被删除, 说明该层z对应的idx结点前驱已经维护完毕, 则insertionLevel-1, 继续维护下一层z对应的idx结点前驱, 此时如果insertionLevel-1为0, 说明所有层的idx结点前驱都维护完毕了, 所以结束自旋返回null即可, 代表插入完毕
                    if (--insertionLevel == 0)
                        break splice;
                }

                // 如果j大于以前h的level, 说明j层不是要维护的索引层, 则z对应的idx结点向下走一层
                if (--j >= insertionLevel && j < level)
                    t = t.down;

                // 到这里, 前一层z对应的idx结点前驱已经维护完毕了, 重新初始化HeadIndex q和Index r, q往下走一层, r为q的后继
                q = q.down;
                r = q.right;
            }
        }
    }

    // 插入成功, 返回null
    return null;
}
```

##### 删除元素方法

- **remove（Object）**：如果key存在，则从此映射中删除该key对应的映射，忽略value值。
- **remove（Object，Object）**：删除key和value都equals的键值对，value值必须匹配。

```java
// 如果key存在，则从此映射中删除该key对应的映射，忽略value值
public V remove(Object key) {
    return doRemove(key, null);
}

// 删除key和value都equals的键值对，value值必须匹配
public boolean remove(Object key, Object value) {
    if (key == null)
        throw new NullPointerException();
    return value != null && doRemove(key, value) != null;
}

// remove方法核心逻辑, 如果value不为null, 则必须key和value都匹配才删除该结点, 否则key匹配即可删除
final V doRemove(Object key, Object value) {
    if (key == null)
        throw new NullPointerException();
    Comparator<? super K> cmp = comparator;

    // 开始自旋, 比较器comparator, 键key, 值value, key前驱b, b后继n, n值value, key比较结果c, n后继f
    outer: for (;;) {
        // 查找比刚好key小一点的数据结点(key前驱), 如果找不到则会返回BASE_HEADER, 查找同时还会清除value为null的数据结点的索引结点
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            Object v; int c;
            if (n == null)
                break outer;
            Node<K,V> f = n.next;

            // 如果n不为b后继了, 则说明b的后继被其他线程更改了, 此时重新自旋, 以获取新的b
            if (n != b.next)                    // inconsistent read
                break;

            // 如果后继n值value为null, 说明n已经被删除了, 则通过标记后继结点辅助删除实例结点, 在实例结点value为null时调用, 删除后重新自旋, 以获取新的b
            if ((v = n.value) == null) {        // n is deleted
                n.helpDelete(b, f);
                break;
            }

            // 如果key前驱b值为null, 或者n为标记结点, 说明b是要被删除的, 此时重新自旋, 以获取新的b
            if (b.value == null || v == n)      // b is deleted
                break;

            // 如果b和n都不是已删除的结点, 则比较key与n.key, 如果key < n.key, 说明key不存在(有可能真的不存在、也有可能被之前存在但被其他线程删除了), 则直接结束自旋返回null即可
            if ((c = cpr(cmp, key, n.key)) < 0)
                break outer;

            // 如果key > n.key, 说明key的位置应该在n的后面, 则继续遍历
            if (c > 0) {
                b = n;
                n = f;
                continue;
            }

            // 如果key == n.key, 说明n就是key要找的结点, 如果指定了value, 但value不匹配, 则直接结束自旋返回null即可
            if (value != null && !value.equals(v))
                break outer;

            // 如果key == n.key, 且不指定value或者value匹配了, 则CAS更新n.value为null, 代表删除结点n
            if (!n.casValue(v, null))
                // 如果CAS失败, 说明v.value已经被其他线程更新掉了, 即删除失败, 则重新自旋
                break;

            // 如果CAS删除n结点成功, 则再CAS更新f(即n.next)的next指针指向f自身, 代表使f为标记结点, 如果失败了, 则再CAS更新b.next为f结点
            if (!n.appendMarker(f) || !b.casNext(n, f))
                // 如果CAS更新f后继失败, 且再CAS更新b后继失败, 则清除遍历key沿途看到的任何已删除节点, 包括数据结点和索引结点
                findNode(key);

            //  如果CAS更新f后继成功, 或者f后继更新失败但b后继更新成功
            else {
                // 清除value为null的数据结点的索引结点
                findPredecessor(key, cmp);      // clean index

                // 如果head没有索引结点了, 则尝试减少索引层级
                if (head.right == null)
                    // 如果head、head.down、head.down.down3层都没有索引结点了(即只剩下HeadIndex时), 则删除head层, 使head指向下一层; 但如果删除后复检时发现原head又多了索引结点, 则又会恢复原状, 取消删除
                    tryReduceLevel();
            }

            // 删除结点并清除索引结点后, 返回n.value的深复制vv
            @SuppressWarnings("unchecked") V vv = (V)v;
            return vv;
        }
    }
    return null;
}
```

##### 获取元素方法

- **get（Object）**：获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}。
- **getOrDefault（Object，V）**：带默认值的获取，如果获取不到key的元素，则返回默认值。

```java
// 获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}
public V get(Object key) {
    return doGet(key);
}

// 带默认值的获取，如果获取不到key的元素，则返回默认值
public V getOrDefault(Object key, V defaultValue) {
    V v;
    return (v = doGet(key)) == null ? defaultValue : v;
}

// get方法核心逻辑, 与findNode方法几乎相同, 但返回的是结点的值
private V doGet(Object key) {
    if (key == null)
        throw new NullPointerException();
    Comparator<? super K> cmp = comparator;

    // 开始自旋, 比较器comparator, 键key, key前驱b, b后继n, n后继f, n的值v, 比较结果c
    outer: for (;;) {
        // 查找比刚好key小一点的数据结点(key前驱), 如果找不到则会返回BASE_HEADER, 查找同时还会清除value为null的数据结点的索引结点
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            Object v; int c;

            // 如果key要所在位置为null, 说明没找到key对应的结点, 此时结束自旋, 返回null
            if (n == null)
                break outer;

            // 如果n不为b后继了, 则说明b的后继被其他线程更改了, 此时重新自旋, 以获取新的b
            Node<K,V> f = n.next;
            if (n != b.next)                // inconsistent read
                break;

            // 如果后继n值value为null, 说明n已经被删除了, 则通过标记后继结点辅助删除实例结点, 在实例结点value为null时调用, 删除后重新自旋, 以获取新的b
            if ((v = n.value) == null) {    // n is deleted
                n.helpDelete(b, f);
                break;
            }

            // 如果key前驱b值为null, 或者n为标记结点, 说明b是要被删除的, 此时重新自旋, 以获取新的b
            if (b.value == null || v == n)  // b is deleted
                break;

            // 如果b和n都不是已删除的结点, 则比较key与n.key, 如果key == n.key, 说明n就是要找结点结点, 此时返回n结点的值
            if ((c = cpr(cmp, key, n.key)) == 0) {
                @SuppressWarnings("unchecked") V vv = (V)v;
                return vv;
            }

            // 如果c小于0, 即key < n.key, 说明b遍历到了链尾也没找到key对应的结点, 此时结束自旋, 返回null
            if (c < 0)
                break outer;

            // 如果c大于0, 即key > n.key, 说明key对应的结点可能还在b后面, 则继续遍历b链表
            b = n;
            n = f;
        }
    }
    return null;
}
```

##### 实现NavigableMap接口方法

###### 获取最近元素方法

- **findNear（K，int，Comparator）**：返回指定key和拟合关系的最近节点(通过LT|EQ或者GT|EQ 来控制 小于等于或者大于等于), 如果没有则为 null, 用于lower、floor、ceiling、higher方法。
- **getNear（K，int）**：为findNear的结果封装成SimpleImmutableEntry对象，生成时的映射快照，不支持setValue方法。

```java
// 控制值或作为findNear的参数
private static final int EQ = 1;
private static final int LT = 2;
private static final int GT = 0; // Actually checked as !LT

// 返回指定key和拟合关系的最近节点(通过LT|EQ或者GT|EQ 来控制 小于等于或者大于等于), 如果没有则为 null, 用于lower、floor、ceiling、higher方法
final Node<K,V> findNear(K key, int rel, Comparator<? super K> cmp) {
    if (key == null)
        throw new NullPointerException();

    // 开始自旋, 比较器comparator, 键key, key前驱b, b后继n, n后继f, n的值v, 比较结果c
    for (;;) {
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            Object v;

            // 如果key要所在位置为null, 说明没找到key对应的结点, 此时如果rel不为LT || rel为LT/LT|EQ, 但b为BASE_HEADER, 则返回null, 代表找不到后继或者前驱结点; 否则如果rel为LT/LT|EQ且b不为BASE_HEADER, 则返回key前驱b结点
            if (n == null)
                return ((rel & LT) == 0 || b.isBaseHeader()) ? null : b;

            // 如果n不为b后继了, 则说明b的后继被其他线程更改了, 此时重新自旋, 以获取新的b
            Node<K,V> f = n.next;
            if (n != b.next)                  // inconsistent read
                break;

            // 如果后继n值value为null, 说明n已经被删除了, 则通过标记后继结点辅助删除实例结点, 在实例结点value为null时调用, 删除后重新自旋, 以获取新的b
            if ((v = n.value) == null) {      // n is deleted
                n.helpDelete(b, f);
                break;
            }

            // 如果key前驱b值为null, 或者n为标记结点, 说明b是要被删除的, 此时重新自旋, 以获取新的b
            if (b.value == null || v == n)      // b is deleted
                break;

            // 如果b和n都不是已删除的结点, 则比较key与n.key
            int c = cpr(cmp, key, n.key);

            // 1. 如果key == n.key, 说明n就是要找结点结点, 如果此时rel为LT|EQ(小于等于)或者GT|EQ(大于等于)时, 则返回n
            // 2. 如果key < n.key, 说明b遍历到了链尾也没找到key对应的结点, 如果此时rel为GT(大于)或者GT|EQ(大于等于)时, 则返回n
            if ((c == 0 && (rel & EQ) != 0) || (c < 0 && (rel & LT) == 0))
                return n;

            // 3. 如果key < n.key, 如果此时rel为LT(小于)或者LT|EQ(小于等于)时
            if ( c <= 0 && (rel & LT) != 0)
                // 如果为BASE_HEADER, 则返回null, 代表找不到后继或者前驱结点; 否则返回key前驱b结点
                return b.isBaseHeader() ? null : b;

            // 如果c大于0, 即key > n.key, 说明key对应的结点可能还在b后面, 则继续遍历b链表
            b = n;
            n = f;
        }
    }
}

// 为findNear的结果封装成SimpleImmutableEntry对象，生成时的映射快照，不支持setValue方法
final AbstractMap.SimpleImmutableEntry<K,V> getNear(K key, int rel) {
    Comparator<? super K> cmp = comparator;
    for (;;) {
        Node<K,V> n = findNear(key, rel, cmp);
        if (n == null)
            return null;
        AbstractMap.SimpleImmutableEntry<K,V> e = n.createSnapshot();
        if (e != null)
            return e;
    }
}
```

###### 获取小于Key的最大键方法

- **lowerEntry（K）**：
  - 底层**LT**依赖findNear。
  - 返回小于Key的最大键的Entry，如果没有这样的键，则返回 {@code null}。
  - 返回的是{@code Map.Entry} 生成时的**映射快照**，不支持setValue方法。
- **lowerKey（K）**：
  - 底层**LT**依赖findNear。
  - 返回小于Key的最大键，如果没有这样的键，则返回 {@code null}。

```java
// 底层LT依赖findNear，返回小于Key的最大键的Entry，如果没有这样的键，则返回 {@code null}
public Map.Entry<K,V> lowerEntry(K key) {
    return getNear(key, LT);
}

// 底层LT依赖findNear，返回小于指定键的最大键的条目； 如果不存在这样的条目（即树中的最小键大于指定的键），则返回 {@code null}
public K lowerKey(K key) {
    Node<K,V> n = findNear(key, LT, comparator);
    return (n == null) ? null : n.key;
}
```

###### 获取小于等于Key的最大键方法

- **floorEntry（K）**：
  - 底层**LT|EQ**依赖findNear。
  - 返回小于或者等于Key的最大键的Entry，如果没有这样的键，则返回 {@code null}。
  - 返回的是 {@code Map.Entry} 生成时的**映射快照**，不支持setValue方法。
- **floorKey（K）**：
  - 底层**LT|EQ**依赖findNear。
  - 返回小于或者等于Key的最大键，如果没有这样的键，则返回 {@code null}。

```java
// 底层LT|EQ依赖findNear，返回小于或者等于Key的最大键的Entry，如果没有这样的键，则返回 {@code null}，返回的是 {@code Map.Entry} 生成时的映射快照，不支持setValue方法。
public Map.Entry<K,V> floorEntry(K key) {
    return getNear(key, LT|EQ);
}

// 底层LT|EQ依赖findNear，返回小于或者等于Key的最大键，如果没有这样的键，则返回 {@code null}。
public K floorKey(K key) {
    Node<K,V> n = findNear(key, LT|EQ, comparator);
    return (n == null) ? null : n.key;
}
```

###### 获取大于等于key的最小键方法

- **ceilingEntry（K）**：
  - 底层**GT|EQ**依赖findNear。
  - 返回大于或者等于Key的最小键的Entry，如果没有这样的键，则返回 {@code null}。
  - 返回的是 {@code Map.Entry} 生成时的**映射快照**，不支持setValue方法。
- **ceilingKey（K）**：
  - 底层**GT|EQ**依赖findNear。
  - 返回大于或者等于Key的最小键，如果没有这样的键，则返回 {@code null}。

```java
// 底层GT|EQ依赖findNear，返回大于或者等于Key的最小键的Entry，如果没有这样的键，则返回 {@code null}。返回的是 {@code Map.Entry} 生成时的映射快照，不支持setValue方法
public Map.Entry<K,V> ceilingEntry(K key) {
    return getNear(key, GT|EQ);
}

// 底层GT|EQ依赖findNear，返回大于或者等于Key的最小键，如果没有这样的键，则返回 {@code null}
public K ceilingKey(K key) {
    Node<K,V> n = findNear(key, GT|EQ, comparator);
    return (n == null) ? null : n.key;
}
```

###### 获取大于key的最小键方法

- **higherEntry（K）**：
  - 底层**GT**依赖findNear。
  - 返回大于Key的最小键的Entry，如果没有这样的键，则返回 {@code null}。
  - 返回的是{@code Map.Entry} 生成时的**映射快照**，不支持setValue方法。
- **higherKey（K）**：
  - 底层**GT**依赖findNear。
  - 返回大于Key的最小键，如果没有这样的键，则返回 {@code null}。

```java
// 底层GT依赖findNear，返回大于Key的最小键的Entry，如果没有这样的键，则返回 {@code null}。返回的是{@code Map.Entry} 生成时的映射快照，不支持setValue方法。
public Map.Entry<K,V> higherEntry(K key) {
    return getNear(key, GT);
}

// 底层GT依赖findNear，返回大于Key的最小键，如果没有这样的键，则返回 {@code null}。
public K higherKey(K key) {
    Node<K,V> n = findNear(key, GT, comparator);
    return (n == null) ? null : n.key;
}
```

###### 获取最小键方法

- **firstEntry（）**：返回最小键的Entry，如果条目为空，则返回 {@code null}。
- **firstKey（）**：返回最小键，如果没有这样的键，则抛出异常。
- **pollFirstEntry（）**：删除并返回最小键的Entry，如果条目为空，则返回 {@code null}。

```java
// 返回最小键的Entry，如果条目为空，则返回 {@code null}
public Map.Entry<K,V> firstEntry() {
    for (;;) {
        // findNode的特殊变体, 用于获取第一个有效节点, 如果返回之前刚好n被置null了, 则通过标记后继结点辅助删除实例结点
        Node<K,V> n = findFirst();
        if (n == null)
            return null;
        AbstractMap.SimpleImmutableEntry<K,V> e = n.createSnapshot();
        if (e != null)
            return e;
    }
}

// 返回最小键，如果没有这样的键，则抛出异常
public K firstKey() {
    // findNode的特殊变体, 用于获取第一个有效节点
    Node<K,V> n = findFirst();
    if (n == null)
        throw new NoSuchElementException();
    return n.key;
}
// findNode的特殊变体, 用于获取第一个有效节点, 如果返回之前刚好n被置null了, 则通过标记后继结点辅助删除实例结点
final Node<K,V> findFirst() {
    // 从head开始遍历node, 直到第一个不为null的后继n, 并返回n.value
    for (Node<K,V> b, n;;) {
        if ((n = (b = head.node).next) == null)
            return null;
        if (n.value != null)
            return n;
        // 如果返回之前刚好n被置null了, 则通过标记后继结点辅助删除实例结点, 在实例结点value为null时调用
        n.helpDelete(b, n.next);
    }
}

// 删除并返回最小键的Entry，如果条目为空，则返回 {@code null}
public Map.Entry<K,V> pollFirstEntry() {
    // 删除第一个条目的数据结点以及与之对应的每层的索引结点, 并返回其键和值的快照
    return doRemoveFirstEntry();
}
// 删除第一个条目的数据结点以及与之对应的每层的索引结点, 并返回其键和值的快照
private Map.Entry<K,V> doRemoveFirstEntry() {
    for (Node<K,V> b, n;;) {
        // 从head开始遍历node, 找到第一个不为null的后继n, 如果找不到则返回null
        if ((n = (b = head.node).next) == null)
            return null;

        // 如果n不为b后继了, 则说明b的后继被其他线程更改了, 此时重新自旋, 以获取新的b
        Node<K,V> f = n.next;
        if (n != b.next)
            continue;

        // 如果后继n值value为null, 说明n已经被删除了, 则通过标记后继结点辅助删除实例结点, 在实例结点value为null时调用, 删除后重新自旋, 以获取新的b
        Object v = n.value;
        if (v == null) {
            n.helpDelete(b, f);
            continue;
        }

        // 如果后继n没有发生改变, 则CAS更新n.value为null, 即删除n结点
        if (!n.casValue(v, null))
            // 如果CAS失败, 说明v.value已经被其他线程更新掉了, 即删除失败, 则重新自旋
            continue;

        // 如果CAS删除n结点成功, 则再CAS更新f(即n.next)的next指针指向f自身, 代表使f为标记结点, 如果失败了, 则再CAS更新b.next为f结点
        if (!n.appendMarker(f) || !b.casNext(n, f))
            // 通过标记后继结点辅助删除实例结点, 在实例结点value为null时调用
            findFirst(); // retry

        // 清除数据结点后, 继续清除与删除每层第一个条目关联的索引节点
        clearIndexToFirst();

        // 数据结点和索引结点都删除成功后, 则返回n键、n值的快照
        @SuppressWarnings("unchecked") V vv = (V)v;
        return new AbstractMap.SimpleImmutableEntry<K,V>(n.key, vv);
    }
}
// 清除与删除每层第一个条目关联的索引节点
private void clearIndexToFirst() {
    // 开始自旋
    for (;;) {
        // 从head开始遍历索引层, head指针q, q的后继r
        for (Index<K,V> q = head;;) {
            Index<K,V> r = q.right;
            // 如果r不为null, 且r对应的数据结点为null, 说明找到了q层第一个需要删除的的索引结点, 则CAS解除后继r的链接
            if (r != null && r.indexesDeletedNode() && !q.unlink(r))
                // 如果r无效, 或者r数据结点有效, 或者CAS解除r链接失败, 则重新自旋
                break;

            // CAS解除r链接成功, 如果q下结点为null, 说明清除到了最后一层, 即每层的第一个需要删除的索引已经清理完毕了
            if ((q = q.down) == null) {
                // 再判断是否需要减少索引层级, 如果head没有索引结点了, 则尝试减少索引层级
                if (head.right == null)
                    // 如果head、head.down、head.down.down3层都没有索引结点了(即只剩下HeadIndex时), 则删除head层, 使head指向下一层; 但如果删除后复检时发现原head又多了索引结点, 则又会恢复原状, 取消删除
                    tryReduceLevel();

                // 如果不需要减少索引层级, 或者减少成功后, 则返回即可
                return;
            }
        }
    }
}
```

###### 获取最大键方法

- **lastEntry（）**：返回最大键的Entry，如果条目为空，则返回 {@code null}。
- **lastKey（）**：返回最大键，如果没有这样的键，则抛出异常。
- **pollLastEntry（）**：删除并返回最大键的Entry，如果条目为空，则返回 {@code null}。

```java
// 返回最大键的Entry，如果条目为空，则返回 {@code null}
public Map.Entry<K,V> lastEntry() {
    for (;;) {
        // 获取最后一个有效节点: 先走到最后一个索引结点, 再从该索引的数据结点遍历到最后一个数据结点，对比findPredecessor，该方法不需要比较key大小
        Node<K,V> n = findLast();
        if (n == null)
            return null;
        AbstractMap.SimpleImmutableEntry<K,V> e = n.createSnapshot();
        if (e != null)
            return e;
    }
}

// 返回最大键，如果没有这样的键，则抛出异常
public K lastKey() {
    // 获取最后一个有效节点: 先走到最后一个索引结点, 再从该索引的数据结点遍历到最后一个数据结点，对比findPredecessor，该方法不需要比较key大小
    Node<K,V> n = findLast();
    if (n == null)
        throw new NoSuchElementException();
    return n.key;
}
// 获取最后一个有效节点: 先走到最后一个索引结点, 再从该索引的数据结点遍历到最后一个数据结点，对比findPredecessor，该方法不需要比较key大小
final Node<K,V> findLast() {
    Index<K,V> q = head;

    // 开始自旋, head指针q, q下结点d, q后继r
    for (;;) {
        Index<K,V> d, r;

        // 如果q存在后继r, 说明还没到最右一个索引结点, 则继续遍历q层索引
        if ((r = q.right) != null) {
            // 如果r已被删除, 则返回CAS解除后继succ的链接, 更新q为head指针, 继续自旋
            if (r.indexesDeletedNode()) {
                q.unlink(r);
                q = head; // restart
            }
            // 如果r没被删除, 则继续遍历q层索引结点
            else
                q = r;
        }
        // 如果q不存在后继r, 说明q层索引遍历到尾了, 则q往下走一层, 继续自旋
        else if ((d = q.down) != null) {
            q = d;
        }
        // 如果q层索引到尾了, 且q为最后一层索引, 说明找到了最后一个索引结点
        else {
            // 从最后一个索引的数据结点开始遍历, q的数据结点b, b后继n, n后继f
            for (Node<K,V> b = q.node, n = b.next;;) {
                // 如果n为null了, 说明数据结点遍历到尾了, 则返回b; 如果b为BASE_HEADER, 则返回null, 表示没有任何数据结点
                if (n == null)
                    return b.isBaseHeader() ? null : b;

                // 如果n不为b后继了, 则说明b的后继被其他线程更改了, 此时重新自旋, 以获取新的b
                Node<K,V> f = n.next;            // inconsistent read
                if (n != b.next)
                    break;

                // 如果后继n值value为null, 说明n已经被删除了, 则通过标记后继结点辅助删除实例结点, 在实例结点value为null时调用, 删除后结束遍历, 重新获取b, 重新自旋
                Object v = n.value;
                if (v == null) {                 // n is deleted
                    n.helpDelete(b, f);
                    break;
                }

                // 如果key前驱b值为null, 或者n为标记结点, 说明b是要被删除的, 此时结束遍历, 重新获取b, 重新自旋
                if (b.value == null || v == n)      // b is deleted
                    break;

                // 如果b和n都是正常的结点, 继续遍历数据结点链表
                b = n;
                n = f;
            }

            // 重新获取b, 重新自旋
            q = head; // restart
        }
    }
}

// 删除并返回最大键的Entry，如果条目为空，则返回 {@code null}。
public Map.Entry<K,V> pollLastEntry() {
    // 删除最后一个条目的数据结点以及与之对应的每层的索引结点, 并返回其键和值的快照
    return doRemoveLastEntry();
}
// 删除最后一个条目的数据结点以及与之对应的每层的索引结点, 并返回其键和值的快照
private Map.Entry<K,V> doRemoveLastEntry() {
    // 开始自旋
    for (;;) {
        // 获取最后一层最后一个索引的数据结点
        Node<K,V> b = findPredecessorOfLast();

        // b后继n, 如果n为null了, 说明数据结点遍历到尾了, 则继续自旋; 如果b为BASE_HEADER, 则返回null, 表示没有任何数据结点, 删除失败
        Node<K,V> n = b.next;
        if (n == null) {
            if (b.isBaseHeader())               // empty
                return null;
            else
                continue;// all b's successors are deleted; retry
        }

        // 如果b还有后继, 则继续遍历
        for (;;) {
            // 如果n不为b后继了, 则说明b的后继被其他线程更改了, 此时重新自旋, 以获取新的b
            Node<K,V> f = n.next;
            if (n != b.next)                    // inconsistent read
                break;

            // 如果后继n值value为null, 说明n已经被删除了, 则通过标记后继结点辅助删除实例结点, 在实例结点value为null时调用, 删除后结束遍历, 重新获取b, 重新自旋
            Object v = n.value;
            if (v == null) {                    // n is deleted
                n.helpDelete(b, f);
                break;
            }

            // 如果key前驱b值为null, 或者n为标记结点, 说明b是要被删除的, 此时结束遍历, 重新获取b, 重新自旋
            if (b.value == null || v == n)      // b is deleted
                break;

            // 如果b和n都是正常的结点, 且f不为null, 说明n还有后继, 则继续遍历数据结点链表
            if (f != null) {
                b = n;
                n = f;
                continue;
            }

            // 如果b和n都是正常的结点, 但f为null, 说明n为最后一个结点了, 则CAS更新v值为null
            if (!n.casValue(v, null))
                // 如果CAS失败, 则结束遍历, 重新获取b, 重新自旋
                break;

            // 如果CAS删除n结点成功, 则再CAS更新f(即n.next)的next指针指向f自身, 代表使f为标记结点, 如果失败了, 则再CAS更新b.next为f结点
            K key = n.key;
            if (!n.appendMarker(f) || !b.casNext(n, f))
                // 如果CAS更新f后继失败, 且再CAS更新b后继失败, 则清除遍历key沿途看到的任何已删除节点, 包括数据结点和索引结点
                findNode(key);                  // retry via findNode

            //  如果CAS更新f后继成功, 或者f后继更新失败但b后继更新成功
            else {                              // clean index
                // 清除value为null的数据结点的索引结点
                findPredecessor(key, comparator);

                // 如果head没有索引结点了, 则尝试减少索引层级
                if (head.right == null)
                    // 如果head、head.down、head.down.down3层都没有索引结点了(即只剩下HeadIndex时), 则删除head层, 使head指向下一层; 但如果删除后复检时发现原head又多了索引结点, 则又会恢复原状, 取消删除
                    tryReduceLevel();
            }

            // 数据结点和索引结点都删除成功后, 则返回n键、n值的快照
            @SuppressWarnings("unchecked") V vv = (V)v;
            return new AbstractMap.SimpleImmutableEntry<K,V>(key, vv);
        }
    }
}
```

###### SubMap实现类

```java
static final class SubMap<K,V> extends AbstractMap<K,V> implements ConcurrentNavigableMap<K,V>, Cloneable, Serializable {
    
    private final ConcurrentSkipListMap<K,V> m;// 底层Map
    private final K lo;// 下限键，如果从头开始，则为 null
    private final K hi;// 上限键，如果结束则为空
    private final boolean loInclusive;// lo 的包含标志
    private final boolean hiInclusive;// hi 的包含标志
    private final boolean isDescending;// 方向

    // 延迟初始化的视图持有者
    private transient KeySet<K> keySetView;
    private transient Set<Map.Entry<K,V>> entrySetView;
    private transient Collection<V> valuesView;
    
    // 创建一个新的子图，初始化所有字段。
    SubMap(ConcurrentSkipListMap<K,V> map,
           K fromKey, boolean fromInclusive,
           K toKey, boolean toInclusive,
           boolean isDescending) {
        Comparator<? super K> cmp = map.comparator;
        if (fromKey != null && toKey != null && cpr(cmp, fromKey, toKey) > 0)
            throw new IllegalArgumentException("inconsistent range");
        this.m = map;
        this.lo = fromKey;
        this.hi = toKey;
        this.loInclusive = fromInclusive;
        this.hiInclusive = toInclusive;
        this.isDescending = isDescending;
    }
    
    // 创建子图的实用程序，其中给定的边界覆盖无界（空）的或根据有界的进行检查，subMap、headMap、tailMap方法调用
    SubMap<K,V> newSubMap(K fromKey, boolean fromInclusive,
                          K toKey, boolean toInclusive) {
        Comparator<? super K> cmp = m.comparator;
        if (isDescending) { // flip senses
            K tk = fromKey;
            fromKey = toKey;
            toKey = tk;
            boolean ti = fromInclusive;
            fromInclusive = toInclusive;
            toInclusive = ti;
        }
        if (lo != null) {
            if (fromKey == null) {
                fromKey = lo;
                fromInclusive = loInclusive;
            }
            else {
                int c = cpr(cmp, fromKey, lo);
                if (c < 0 || (c == 0 && !loInclusive && fromInclusive))
                    throw new IllegalArgumentException("key out of range");
            }
        }
        if (hi != null) {
            if (toKey == null) {
                toKey = hi;
                toInclusive = hiInclusive;
            }
            else {
                int c = cpr(cmp, toKey, hi);
                if (c > 0 || (c == 0 && !hiInclusive && toInclusive))
                    throw new IllegalArgumentException("key out of range");
            }
        }
        return new SubMap<K,V>(m, fromKey, fromInclusive,
                               toKey, toInclusive, isDescending);
    }
    
    abstract class SubMapIter<T> implements Iterator<T>, Spliterator<T> {
        Node<K,V> lastReturned;// next() 返回的最后一个节点
        Node<K,V> next;// 从 next() 返回的下一个节点
        V nextValue;// 缓存下一个值字段以保持弱一致性
        
        public final boolean hasNext() {
            return next != null;
        }
        
        // 子Map的遍历放啊
        final void advance() {
            if (next == null)
                throw new NoSuchElementException();
            lastReturned = next;
            
            // 如果isDescending为true，则逆序遍历
            if (isDescending)
                descend();
            // 如果isDescending为fasle，则顺序遍历
            else
                ascend();
        }

        // 顺序遍历子Map
        private void ascend() {
            Comparator<? super K> cmp = m.comparator;
            for (;;) {
                next = next.next;
                if (next == null)
                    break;
                Object x = next.value;
                if (x != null && x != next) {
                    if (tooHigh(next.key, cmp))
                        next = null;
                    else {
                        @SuppressWarnings("unchecked") V vv = (V)x;
                        nextValue = vv;
                    }
                    break;
                }
            }
            
        }
        
        // 逆序遍历子Map
        private void descend() {
            Comparator<? super K> cmp = m.comparator;
            for (;;) {
                // 类似于ConcurrentSkipListMap的findNear，返回小于Key的最大键的Entry，如果没有这样的键，则返回 {@code null}
                next = m.findNear(lastReturned.key, LT, cmp);
                if (next == null)
                    break;
                Object x = next.value;
                if (x != null && x != next) {
                    if (tooLow(next.key, cmp))
                        next = null;
                    else {
                        @SuppressWarnings("unchecked") V vv = (V)x;
                        nextValue = vv;
                    }
                    break;
                }
            }
        }
    }
}
```

###### 获取逆序视图方法

- **descendingMap（）**：返回Map的逆序（降序）视图。

```java
// 返回Map的逆序（降序）视图
public ConcurrentNavigableMap<K,V> descendingMap() {
    ConcurrentNavigableMap<K,V> dm = descendingMap;
    return (dm != null) ? dm :
    // 本Map作为底层Map, 没有下限键, 不包含lo, 没有上限键, 不包含hi, 反向
    (descendingMap = new SubMap<K,V>(this, null, false, null, false, true));
}
```

###### 获取中间视图方法

没有指定isDescending，isDescending默认为false，代表所有方法都是**顺序**的。

- **subMap（K，boolean，K，boolean）**：返回Map的中间视图，键从fromKey（不含）到toKey（不含）的
  部分视图，fromInclusive为true代表包含fromKey，toInclusive为true代表包含toKey。
- **subMap（K，K）**：返回Map的中间视图，即键从fromKey（fromInclusive含）到toKey（!toInclusive不含）的部分视图，两者相等时返回null(除非fromInclusive与toInclusive都为true)。

```java
// 返回Map的中间视图，键从fromKey（不含）到toKey（不含）的部分视图，fromInclusive为true代表包含fromKey，toInclusive为true代表包含toKey
public SubMap<K,V> subMap(K fromKey, boolean fromInclusive,
                          K toKey, boolean toInclusive) {
    if (fromKey == null || toKey == null)
        throw new NullPointerException();
    
    // 指定上限键，指定是否包含lo，指定上限键，指定是否包含hi
    return newSubMap(fromKey, fromInclusive, toKey, toInclusive);
}

// 返回Map的中间视图，即键从fromKey（fromInclusive含）到toKey（!toInclusive不含）的部分视图，两者相等时返回null(除非fromInclusive与toInclusive都为true)。
public SubMap<K,V> subMap(K fromKey, K toKey) {
    // 指定下限键， 包含lo，指定上限键，不包含hi
    return subMap(fromKey, true, toKey, false);
}
```

###### 获取头部视图方法

没有指定isDescending，isDescending默认为false，代表所有方法都是**顺序**的。

- **headMap（K，boolean）**：返回Map的头部视图，即键小于toKey(不含)的部分视图，inclusive为true代表包含toKey。
- **headMap（K）**：返回Map的头部视图，即键小于toKey(!inclusive不含)的部分视图。

```java
// 返回Map的头部视图，即键小于toKey(不含)的部分视图，inclusive为true代表包含toKey
public SubMap<K,V> headMap(K toKey, boolean inclusive) {
    if (toKey == null)
        throw new NullPointerException();
    
    // 不指定下限键，不包含lo，指定上限键，指定是否包含hi
    return newSubMap(null, false, toKey, inclusive);
}

// 返回Map的头部视图，即键小于toKey(!inclusive不含)的部分视图
public SubMap<K,V> headMap(K toKey) {
    // 指定上限键，不包含hi
    return headMap(toKey, false);
}
```

###### 获取尾部视图方法

没有指定isDescending，isDescending默认为false，代表所有方法都是**顺序**的。

- **tailMap（K，boolean）**：返回Map的尾部视图，即键大于fromKey（不含）的部分视图，inclusive为true代表包含fromKey。
- **tailMap（K）**：返回Map的尾部视图，即键大于或者等于fromKey（inclusive含）的部分视图。

```java
// 返回Map的尾部视图，即键大于fromKey（不含）的部分视图，inclusive为true代表包含fromKey
public SubMap<K,V> tailMap(K fromKey, boolean inclusive) {
    if (fromKey == null)
        throw new NullPointerException();
    
    // 指定下限键，指定是否包含lo，不指定上限键，不包含hi
    return newSubMap(fromKey, inclusive, null, false);
}

// 返回Map的尾部视图，即键大于或者等于fromKey（inclusive含）的部分视图
public SubMap<K,V> tailMap(K fromKey) {
    // 指定下限键，包含lo
    return tailMap(fromKey, true);
}
```

#### JDK7 HashMap

![1624883193030](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1624883193030.png)

##### 特点

- JDK7 HashMap，JDK1.7中基于哈希表的Map接口实现，提供所有可选的映射操作，**允许null值和null键，但不保证映射的顺序**。
- 假设散列函数在存储桶中正确分散元素，get 和 put提供恒定时间性能，即O（1）。 迭代集合视图需要的时间与 HashMap 实例的**“容量”（桶的数量）**加上**它的大小（键值映射的数量）**成正比。 因此，如果迭代性能很重要，则不要将初始容量设置得太高或负载因子太低（会导致大容量），这一点非常重要。
- 两个影响性能的参数：
  - **初始容量**：当前容量指哈希表中的桶数，而初始容量指哈希表创建时的容量。
  - **负载因子**：是衡量哈希表在其容量自动增加之前允许达到多满的指标，当 **哈希表中的条目数 > 负载因子 * 当前容量**时，重新哈希表（即重建内部数据结构），使哈希表具有大约两倍的桶数。
- 作为一般规则，**默认负载因子 (0.75)** 在时间和空间成本之间提供了很好的权衡。 较高的值会减少空间开销，但会增加查找成本（反映在 HashMap 类的大多数操作中，包括get 和put）。
  - 因此，在设置其初始容量时，应考虑映射中的**预期条目数及其负载因子**，以尽量减少重新哈希操作的次数。 
  - 而如果要在一个HashMap实例中存储许多映射，则创建具有足够大容量的映射将允许更有效地存储映射，而不是让它根据需要执行自动重新散列以增加表。
  - 其中要注意的是，如果初始容量大于最大条目数除以负载因子，则不会发生重新哈希操作。
- **HashMap是非同步的**，在多个线程并发访问时，通常是通过同步一些自然封装映射的对象来完成，比如
  Collections.synchronizedMap。
- **HashMap的所有迭代器都是快速失败的**，即如果在创建迭代器后的任何时间对结构进行结构修改，则除了通过迭代器自己的remove方法之外，该迭代器都将抛出{ @link ConcurrentModificationException}，这在面对并发修改时，迭代器可以快速干净地失败，而不是在未来不确定的时间冒着任意、非确定性行为的风险去修改。但是**无法保证迭代器的快速失败行为**，因为一般而言，在存在非同步并发修改的情况下，不可能做出任何硬保证，**只会尽最大努力抛出**ConcurrentModificationException，因此，编写一个依赖于这个异常来保证其正确性的程序是错误的，**快速失败行为只适用于检测错误**。

##### 数据结构

![1625372230001](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625372230001.png)

##### 构造方法

- **空参的构造方法**：构造一个具有默认初始容量 (16) 和默认负载因子 (0.75) 的空 HashMap。
- **指定初始容量的构造方法**：构造一个具有指定初始容量和默认负载因子 (0.75) 的空 HashMap。
- **指定初始容量和负载因子的构造方法**：构造一个具有指定初始容量和负载因子的空HashMap。
- **指定复制集合的构造方法**：构造一个与指定Map具有相同映射关系的新HashMap。HashMap是使用默认负载因子(0.75)和足以在指定Map中保存映射的初始容量创建的。

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable
{
	static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;// 默认容量
	static final int MAXIMUM_CAPACITY = 1 << 30;// 最大容量
	static final float DEFAULT_LOAD_FACTOR = 0.75f;// 默认负载因子
	static final Entry<?,?>[] EMPTY_TABLE = {};// 空散列表
	transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;// 散列表
	transient int size;// 实际大小
	
	// 阈值：即要下一个容量值=当前容量（散列表实际大小）*负载因子，如果table==EMPTY_TABLE，则在初始化散列表时，用作新建表的初始容量
	int threshold;
	
	final float loadFactor;// 负载因子
	transient int modCount;// 修改模数
	
	// 哈希种子阈值的默认值
	static final int ALTERNATIVE_HASHING_THRESHOLD_DEFAULT = Integer.MAX_VALUE;
	
	// 哈希种子可以让哈希算法更复杂一点，从而减少哈希碰撞的概率，如果为0代表哈希种子不生效
	transient int hashSeed = 0;

	// HashMap元素结点
    static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;
        
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }
    }
    
    // 构造一个具有默认初始容量 (16) 和默认负载因子 (0.75) 的空 HashMap。
    public HashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }
    
    // 构造一个具有指定初始容量和默认负载因子 (0.75) 的空 HashMap。
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    
    // 构造一个具有指定初始容量和负载因子的空HashMap。
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        this.loadFactor = loadFactor;
        threshold = initialCapacity;
        init();
    }
    
    // LinkedHashMap初始化钩子, 在初始化HashMap之后但在插入任何条目之前调用
    void init() {
    }
    
    // 构造一个与指定Map具有相同映射关系的新HashMap。 HashMap 是使用默认负载因子(0.75)和足以在指定Map中保存映射的初始容量创建的。
    public HashMap(Map<? extends K, ? extends V> m) {
        // 使用默认容量16或者根据复制集合大小计算的容量, 以及默认负载因子0.75构造HashMap, 其中由于table=[], 所以使用指定的容量作为阈值
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                      DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);

        // 根据指定容量初始化散列表(只在散列表为空时调用)，根据规范化后的容量计算新阈值以及构造新容量的散列表
        inflateTable(threshold);
        
        // 添加复制集合所有元素, 但不会调整散列表的大小
        putAllForCreate(m);
    }
    // 添加复制集合所有元素, 但不会调整散列表的大小
    private void putAllForCreate(Map<? extends K, ? extends V> m) {
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
            // 添加指定key、value元素, 但不会调整散列表的大小
            putForCreate(e.getKey(), e.getValue());
    }
    // 添加指定key、value元素, 但不会调整散列表的大小
    private void putForCreate(K key, V value) {
        int hash = null == key ? 0 : hash(key);
        int i = indexFor(hash, table.length);

        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {
                e.value = value;
                return;
            }
        }

        // 使用hash值以及哈希索引, 在散列表中创建条目, 不会调整散列表大小
        createEntry(hash, key, value, i);
    }
}
```

##### 初始化散列表方法

- **inflateTable（int）**：根据指定容量初始化散列表(只在散列表为空时调用)，根据规范化后的容量计算新阈值以及构造新容量的散列表。

```java
// 根据指定容量初始化散列表(只在散列表为空时调用)，根据规范化后的容量计算新阈值以及构造新容量的散列表
private void inflateTable(int toSize) {
    // 返回给定目标容量的2的幂次, 规范化容量: 取2的(number - 1) <<< 1最高位幂次容量
    int capacity = roundUpToPowerOf2(toSize);

    // 根据规范化后的容量计算新阈值
    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);

    // 构造新容量的散列表
    table = new Entry[capacity];

    // 生成/切换哈希种子(会推迟初始化, 默认为0, 代表不生效), 直到指定的容量大于哈希种子阈值(可通过改变JVM参数提前触发更换时机), 返回值用作判断是否重新hash的依据(因为返回true代表了更换了哈希算法), 一般为false
    initHashSeedAsNeeded(capacity);
}
```

##### 迭代方法

- **抽象的HashMap.HashIterator迭代器**：HashMap的迭代器基类，快速失败机制，遍历next指针，返回
  HashMap.Entry对象。
- **Map.Entry迭代器**：继承HashMap的迭代器基类，快速失败机制，遍历next指针，返回Map.Entry对象。
- **HashMap.Entry#Key迭代器**：继承HashMap的迭代器基类，快速失败机制，遍历next指针，返回
  HashMap.Entry#Key对象。
- **HashMap.Entry#Value迭代器**：继承HashMap的迭代器基类，快速失败机制，遍历next指针，返回
  HashMap.Entry#Value对象。

```java
// 抽象的HashMap.HashIterator迭代器：HashMap的迭代器基类，快速失败机制，遍历next指针，返回HashMap.Entry对象。
private abstract class HashIterator<E> implements Iterator<E> {
    Entry<K,V> next;        // 下一个返回的条目
    int expectedModCount;   // 对于快速失败
    int index;              // 当前插槽
    Entry<K,V> current;     // 当前条目

    HashIterator() {
        expectedModCount = modCount;
        // 初始化next指针
        if (size > 0) { // advance to first entry
            Entry[] t = table;
            while (index < t.length && (next = t[index++]) == null);
        }
    }

    public final boolean hasNext() {
        return next != null;
    }

    // 向前遍历链表、散列表
    final Entry<K,V> nextEntry() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        Entry<K,V> e = next;
        if (e == null)
            throw new NoSuchElementException();

        // 遍历链表
        if ((next = e.next) == null) {
            Entry[] t = table;
            // 遍历散列表
            while (index < t.length && (next = t[index++]) == null);
        }
        current = e;
        return e;
    }
}

// Map.Entry迭代器：继承HashMap的迭代器基类，快速失败机制，遍历next指针，返回Map.Entry对象。
private final class EntryIterator extends HashIterator<Map.Entry<K,V>> {
    public Map.Entry<K,V> next() {
        return nextEntry();
    }
}

// HashMap.Entry#Key迭代器：继承HashMap的迭代器基类，快速失败机制，遍历next指针，返回HashMap.Entry#Key对象。
private final class KeyIterator extends HashIterator<K> {
    public K next() {
        return nextEntry().getKey();
    }
}

// HashMap.Entry#Value迭代器：继承HashMap的迭代器基类，快速失败机制，遍历next指针，返回HashMap.Entry#Value对象。
private final class ValueIterator extends HashIterator<V> {
    public V next() {
        return nextEntry().value;
    }
}
```

##### HashCode扰动函数

- **hash（Object）**：HashCode扰动函数, key为String类型则返回key的32位哈希码即可, 否则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响。

```java
// HashCode扰动函数, key为String类型则返回key的32位哈希码即可, 否则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
final int hash(Object k) {
    // 哈希种子可以让哈希算法更复杂一点，从而减少哈希碰撞的概率，如果为0代表哈希种子不生效
    int h = hashSeed;

    // 如果k为String类型, 则返回k的32位哈希值
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    // 如果k不为String类型, 则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    // 如果h为默认0, 则h异或k的散列表码, 还是会等于k的散列码
    h ^= k.hashCode();
    
    // 确保在每个位位置仅相差常数倍的hashCode，从而具有有限数量的冲突（在默认加载因子下约为8）=>把高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

##### 计算哈希索引方法

- **indexFor（int，int）**：返回哈希码h的索引。其实现是利用**（n - 1） & hash**，相当于hash % n，n指散列表的当前容量（桶数量），hash指获取hash（key）即key的hashCode扰动后结果。
  - **不能直接使用HashCode作为索引的原因**：int类型的hashCode，范围为[-2^32，2^32 - 1]，如果散列表
    数组与HashCode一一对应，**需要40亿的空间（int[40亿]），明显这在内存是放不下的**，也就是说明
    hashCode是不能直接作为数组索引的。因此，如果使用hashCode对散列表数组长度取模，那么就可以解决这个问题，从而保证较小的数组也还能利用上hashCode。
  - **n为2的幂次原因**：为了解决hashCode对散列表数组长度取模，设计了HashCode的扰动函数以及为2幂次的容量n，可以通过n-1来获得取模操作的低位掩码，此时只需要通过低位掩码与扰动后的
    **hashCode（hash值）进行一次与运算**，即可得到该hash值在散列表数组中的索引。其次通过在扩容方
    法中，经过hash值与低位掩码相与，可以保证扩容后，**只会移动少部分相与结果高位为1的桶链表**，其他
    保持不变，减少了扩容时的时间。

```java
// 返回哈希码h的索引
    static int indexFor(int h, int length) {
        // 如果n为2的幂次数, 则n-1可以得到散列表容量n的掩码, 0~n-1表示散列表的下标范围, h & (n-1)相当于h % n, 因此这就是n为2的幂次的原因1, 因此与操作比取与操作快
        return h & (length-1);
    }
```

##### 规范容量计算方法

- **roundUpToPowerOf2（int）**：返回给定目标容量的2的幂次，规范化容量，取2的(number - 1) <<< 1最高位幂次容量。

```java
// 返回给定目标容量的2的幂次, 规范化容量: 取2的(number - 1) <<< 1最高位幂次容量
private static int roundUpToPowerOf2(int number) {
    return number >= MAXIMUM_CAPACITY
        ? MAXIMUM_CAPACITY
        // 返回最高位为1其他位补0的数值
        : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
}

// Integer方法，返回最高位为1其他位补0的数值
public static int highestOneBit(int i) {
    // HD, Figure 3-1
    // 右移1位, 让最高位后(含)2位为1, 移出去的不管
    i |= (i >>  1);

    // 右移2位, 让最高位后(含)4位为1, 移出去的不管
    i |= (i >>  2);

    // 右移4位, 让最高位后(含)8位为1, 移出去的不管
    i |= (i >>  4);

    // 右移8位, 让最高位后(含)16位为1, 移出去的不管
    i |= (i >>  8);

    // 右移16位, 让最高位后(含)32位为1, 移出去的不管
    i |= (i >> 16);

    // 最后 - 再右移1位的结果, 得到最高位为1的数值
    return i - (i >>> 1);
}
```

##### 初始化哈希种子

- **initHashSeedAsNeeded（int）**：
  - **生成/切换哈希种子**(会推迟初始化, 默认为0, 代表不生效), 直到指定的容量大于**哈希种子阈值(可通过改变JVM参数提前触发更换时机)**, 返回值用作判断是否重新hash的依据(因为返回true代表了更换了哈希算法), 一般为false。
  - 被初始化散列表inflateTable（int）方法和扩容方法resize（int）调用，用于初始化哈希种子，或者转移结点前判断是否需要**重新计算桶的hash值**。

```java
// 哈希种子阈值的默认值
static final int ALTERNATIVE_HASHING_THRESHOLD_DEFAULT = Integer.MAX_VALUE;

// 哈希种子可以让哈希算法更复杂一点，从而减少哈希碰撞的概率，如果为0代表哈希种子不生效
transient int hashSeed = 0;

// Holder保存直到JVM启动后才能初始化的值。
private static class Holder {
    // 哈希种子阈值, 默认为Integer.MAX_VALUE, 或者为-Djdk.map.althashing.threshold指定的值
    static final int ALTERNATIVE_HASHING_THRESHOLD;

    static {
        // 取当前虚拟机的环境变量阈值-Djdk.map.althashing.threshold
        String altThreshold = java.security.AccessController.doPrivileged(
            new sun.security.action.GetPropertyAction(
                "jdk.map.althashing.threshold"));

        // 如果指定了-Djdk.map.althashing.threshold, 则threshold为指定的阈值, 否则threshold为Integer.MAX_VALUE
        int threshold;
        try {
            threshold = (null != altThreshold)
                ? Integer.parseInt(altThreshold)
                : ALTERNATIVE_HASHING_THRESHOLD_DEFAULT;

            // disable alternative hashing if -1
            if (threshold == -1) {
                threshold = Integer.MAX_VALUE;
            }

            if (threshold < 0) {
                throw new IllegalArgumentException("value must be positive integer.");
            }
        } catch(IllegalArgumentException failed) {
            throw new Error("Illegal value for 'jdk.map.althashing.threshold'", failed);
        }

        // 最后更新ALTERNATIVE_HASHING_THRESHOLD为threshold, 默认为Integer.MAX_VALUE
        ALTERNATIVE_HASHING_THRESHOLD = threshold;
    }
}

// 生成/切换哈希种子(会推迟初始化, 默认为0, 代表不生效), 直到指定的容量大于哈希种子阈值(可通过改变JVM参数提前触发更换时机), 返回值用作判断是否重新hash的依据(因为返回true代表了更换了哈希算法), 一般为false
final boolean initHashSeedAsNeeded(int capacity) {
    // 当前是否存在哈希种子，默认为0 => 即currentAltHashing默认为0
    boolean currentAltHashing = hashSeed != 0;

    // 如果JVM在运行, 且指定的容量大于哈希种子阈值, 说明认为当前哈希算法不太均匀, 则需要生成/切换哈希种子, 从而更换哈希算法
    boolean useAltHashing = sun.misc.VM.isBooted() && (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);

    // 如果currentAltHashing与useAltHashing不相等, 则返回true, 代表需要生成/切换哈希种子
    boolean switching = currentAltHashing ^ useAltHashing;
    if (switching) {
        // 返回一个非零的32位伪随机值, 其中当前HashMap实例用作值的一部分, 构造新的哈希种子作为掩码
        hashSeed = useAltHashing ? sun.misc.Hashing.randomHashSeed(this) : 0;
    }

    // 返回是否生成/切换哈希种子
    return switching;
}
```

##### 扩容方法

- **resize（int）**：根据指定容量扩容散列表(当实际大小超过阈值时调用), 使用头插法将当前表中的所有条目传输到newTable, 会重新计算每个结点的hash值以及新的阈值。

```java
// 根据指定容量扩容散列表(当实际大小超过阈值时调用), 使用头插法将当前表中的所有条目传输到newTable, 会重新计算每个结点的hash值以及新的阈值
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry[] newTable = new Entry[newCapacity];

    // 使用头插法将当前表中的所有条目传输到newTable, 可能会重新计算hash值
    transfer(newTable,
             // 生成/切换哈希种子(会推迟初始化, 默认为0, 代表不生效), 直到指定的容量大于哈希种子阈值(可通过改变JVM参数提前触发更换时机), 返回值用作判断是否重新hash的依据(因为返回true代表了更换了哈希算法), 一般为false
             initHashSeedAsNeeded(newCapacity));

    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}

// 使用头插法将当前表中的所有条目传输到newTable, 可能会重新计算hash值
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;

    // 遍历散列表, 当前遍历到的桶e
    for (Entry<K,V> e : table) {
        // 遍历e链表, 如果哈希种子有变化, 则重新每个e结点的hash值, 重新计算哈希索引i, 并设置到新散列表i桶中
        while(null != e) {
            Entry<K,V> next = e.next;
            
            // 一般为false
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            
            int i = indexFor(e.hash, newCapacity);

            // 头插法：注意！头插法转移元素会使用的当前链表元素顺序反转，导致并发扩容时会出现next出现在e前面的情况，从而出现循环链表。而JDK8 HashMap中的分高低位链表转移方式，既可以解决key需要重新计算索引，也可以解决循环链表的出现。
            e.next = newTable[i];
            newTable[i] = e;

            // 继续遍历e链表
            e = next;
        }
    }
}
```

##### 添加元素方法

- **put（K，V）**：将指定值与此映射中的指定键相关联, 如果映射先前包含键的映射，则旧值将被替换。
- **putAll（Map）**：添加复制集合的的所有键值对。

```java
// 将指定值与此映射中的指定键相关联, 如果映射先前包含键的映射，则旧值将被替换
public V put(K key, V value) {
    // 如果散列表为空, 则根据指定容量初始化散列表(只在散列表表为空时调用)，根据规范化后的容量计算新阈值以及构造新容量的散列表
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }

    // 如果key为null, 则将null键、value值添加到第1个桶, 其中底层会调整散列表大小
    if (key == null)
        return putForNullKey(value);

    // HashCode扰动函数, key为String类型则返回key的32位哈希码即可, 否则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    int hash = hash(key);

    // 返回哈希码 h 的索引
    int i = indexFor(hash, table.length);

    // 从i桶开始遍历链表, 当前遍历结点e, e键k
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;

        // 如果e的hash值等于key的hash值, 且e键等于key或者e键equalskey, 说明e结点就是要找的结点, 则替换e值并返回旧值
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);// LinkedHashMap值覆盖回调方法

            // 替换成功, 返回旧值
            return oldValue;
        }
    }

    modCount++;

    // 将具有指定键、值和哈希码的新条目添加到指定存储桶， 其中此方法会负责调整散列表大小
    addEntry(hash, key, value, i);

    // 添加成功, 返回null
    return null;
}
// 将null键、value值添加到第1个桶, 其中底层会调整散列表大小
private V putForNullKey(V value) {
    // 从第1个桶开始遍历链表, 当前遍历结点e
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        // 如果e键确实为null, 说明e结点就是要找的结点, 则替换e值并返回旧值
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);// LinkedHashMap值覆盖回调方法

            // 替换成功, 返回旧值
            return oldValue;
        }
    }
    modCount++;

    // 将null键、value值添加到第1个桶， 其中此方法会负责调整散列表大小
    addEntry(0, null, value, 0);

    // 添加成功, 返回null
    return null;
}
// 将具有指定键、值和哈希码的新条目添加到指定存储桶， 其中此方法会负责调整散列表大小
void addEntry(int hash, K key, V value, int bucketIndex) {
    // 如果实例大小大于当前阈值, 且bucketIndex桶结点不为null, 说明HashMap需要扩容了
    if ((size >= threshold) && (null != table[bucketIndex])) {
        // 根据指定容量扩容散列表(当实际大小超过阈值时调用), 使用头插法将当前表中的所有条目传输到newTable, 会重新计算每个结点的hash值以及新的阈值
        resize(2 * table.length);

        // 重新计算hash值, 以及哈希索引
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    // 添加复制集合所有元素, 但不会调整散列表的大小
    createEntry(hash, key, value, bucketIndex);
}
// 使用hash值以及哈希索引, 在散列表中创建条目, 不会调整散列表大小
void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}

// 添加复制集合的的所有键值对
public void putAll(Map<? extends K, ? extends V> m) {
    int numKeysToBeAdded = m.size();
    if (numKeysToBeAdded == 0)
        return;

    // 如果散列表为空, 则根据计算出来的阈值初始化散列表(只在散列表表为空时调用)，根据规范化后的容量计算新阈值以及构造新容量的散列表
    if (table == EMPTY_TABLE) {
        inflateTable((int) Math.max(numKeysToBeAdded * loadFactor, threshold));
    }

    // 如果复制集合的大小大于当前阈值, 则先计算新的容量(最接近targetCapacity的2^n), 而targetCapacity=复制集合实际大小 * 负载因子
    if (numKeysToBeAdded > threshold) {
        int targetCapacity = (int)(numKeysToBeAdded / loadFactor + 1);
        if (targetCapacity > MAXIMUM_CAPACITY)
            targetCapacity = MAXIMUM_CAPACITY;

        // 扩容到接近当前容量的2^
        int newCapacity = table.length;
        while (newCapacity < targetCapacity)
            newCapacity <<= 1;

        // 根据指定容量扩容散列表(当实际大小超过阈值时调用), 使用头插法将当前表中的所有条目传输到newTable, 会重新计算每个结点的hash值以及新的阈值
        if (newCapacity > table.length)
            resize(newCapacity);
    }

    // 遍历指定复制集合所有的条目, 并添加到WeakHashMap的散列表中
    for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
        put(e.getKey(), e.getValue());
}
```

##### 删除元素方法

- **remove（Object）**：如果存在指定的键，则从此映射中删除指定键的映射。

```java
// 如果存在指定的键，则从此映射中删除指定键的映射
public V remove(Object key) {
    // 移除并返回与HashMap中指定键关联的条目, 如果HashMap不包含此键的映射, 则返回 null
    Entry<K,V> e = removeEntryForKey(key);
    return (e == null ? null : e.value);
}
// 移除并返回与HashMap中指定键关联的条目, 如果HashMap不包含此键的映射, 则返回 null
final Entry<K,V> removeEntryForKey(Object key) {
    if (size == 0) {
        return null;
    }

    // HashCode扰动函数, key为String类型则返回key的32位哈希码即可, 否则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    int hash = (key == null) ? 0 : hash(key);

    // 返回哈希码h的索引i, 获取i桶结点e、前驱prev(初始时为e)
    int i = indexFor(hash, table.length);
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;

    // 遍历e链表, 后继next, e键k
    while (e != null) {
        Entry<K,V> next = e.next;
        Object k;

        // 如果e的hash值等于key的hash值, 且e键等于key或者e键equals key, 则说明找到了结点e
        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
            modCount++;
            size--;

            // 如果prev等于e, 说明要删除的结点正好为桶头结点, 则替换next为新的桶头
            if (prev == e)
                table[i] = next;
            // 如果prev不等于e, 说明要删除的结点不为桶头结点, 则脱钩e结点
            else
                prev.next = next;

            // LinkedHashMap结点删除回调方法
            e.recordRemoval(this);

            // 删除成功, 则返回e结点
            return e;
        }

        // 如果还没找到要删除的结点e, 则继续遍历e链表
        prev = e;
        e = next;
    }

    // 如果最后还没找到e结点, 则返回null
    return e;
}
```

##### 获取元素方法

- **get（Object）**：返回指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}。

```java
// 返回指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}
public V get(Object key) {
    // 如果key为null, 则从第1个桶结点获取null键以及对应的value值, 第1个桶结点e, 如果e键为null, 则返回e值, 否则返回null
    if (key == null)
        return getForNullKey();

    // 如果key不为null, 返回与 HashMap 中指定键关联的条目。 如果 HashMap 不包含键的映射，则返回 null
    Entry<K,V> entry = getEntry(key);
    return null == entry ? null : entry.getValue();
}
// 从第1个桶结点获取null键以及对应的value值, 第1个桶结点e, 如果e键为null, 则返回e值, 否则返回null
private V getForNullKey() {
    if (size == 0) {
        return null;
    }

    // 第1个桶结点e, 如果e键为null, 则返回e值, 否则返回null
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null)
            return e.value;
    }
    return null;
}
// 返回与 HashMap 中指定键关联的条目。 如果 HashMap 不包含键的映射，则返回 null。
final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }

    // HashCode扰动函数, key为String类型则返回key的32位哈希码即可, 否则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    int hash = (key == null) ? 0 : hash(key);

    // 返回哈希码 h 的索引, 获取该索引所在的桶结点e, 遍历e链表
    for (Entry<K,V> e = table[indexFor(hash, table.length)]; e != null; e = e.next) {
        Object k;

        // 如果e的hash等于key的hash值, 且e键等于key或者e键equals key, 说明e就是要找的结点, 则返回e结点即可
        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }

    // 如果确实找不到e结点, 则返回null
    return null;
}
```

#### JDK7 ConcurrentHashMap

![1625053979148](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625053979148.png)

##### 特点

- JDk7 ConcurrentHashMap，JDK 1.7中**哈希表的并发实现，支持检索的完全并发性和可调整的更新预期并发性**。遵循与 {@link java.util.Hashtable} 相同的功能规范，并包含与 Hashtable 的每个方法对应的方法版本。与 {@link Hashtable} 类似但与 {@link HashMap} 不同的是，**此类不允许将 null 用作键或值**。
- 与 {@link Hashtable}  不同，虽然所有操作也是线程安全的，但是**检索操作不需要对整个散列表加锁**。在依赖其线程安全但不依赖其同步细节的程序中，此类与 Hashtable 完全可互操作。
- 检索操作（包括获取）一般不会阻塞，因此可能与更新操作（包括放置和删除）重叠。 检索反映了**最近完成的更新操作的结果**。 对于诸如 putAll 和 clear 之类的聚合操作，并发检索可能仅反映某些条目的插入或删除。 
- 类似地，迭代器和枚举返回反映哈希表在迭代器/枚举创建时或创建后的**某个时刻的状态的元素**。它们不会抛出 {@link ConcurrentModificationException}。 然而，**迭代器被设计为一次只能被一个线程使用**。
- 更新操作之间允许的并发性，由可选的 **concurrencyLevel** 构造函数参数（默认为 16）来控制，用作内部调整的提示。散列表在内部进行了分区，以尝试允许**指定数量的并发更新**而不会发生争用。
  - 由于散列表中的放置本质上是随机的，所以实际的并发性会有所不同。
  - 理想情况下，您应该选择一个值来容纳尽可能多的线程同时修改表。使用**明显高于需要的值会浪费空间和时间**，而**明显较低的值会导致线程争用**。但一个数量级内的高估和低估通常不会产生太大的影响。当已知只有一个线程会修改而所有其他线程只会读取时，值1是合适的。
  - 此外，调整此散列表或任何其他类型的散列表的大小是一个相对较慢的操作，因此，如果可能，最好在构造函数中提供预期表大小的估计值。
  - concurrencyLevel在**JDK8 ConcurrentHashMap**中，为了能够向后兼容，其作用更改为**只要不为负数即可**，且当指定的initialCapacity < concurrencyLevel，将使用concurrencyLevel作为新的initialCapacity。
- JDK7 ConcurrentHashMap基本策略是**将散列表细分到Segment中，每个Segment本身就是一个并发可读的散列表**。
  - 为了减少占用空间，只有在第一次需要时才构建除一个段之外的所有段。
  - 为了在存在延迟构造的情况下保持可见性，对段以及段表的元素的访问必须使用**volatile访问**，这是通过 segmentAt 等方法中的 Unsafe 完成的，**提供了 AtomicReferenceArrays 的功能，但减少了间接级别**。 
  - 此外，锁定操作中表元素和条目**“next”字段的volatile**写入使用更便宜的“lazySet”形式的写入（通过 **putOrderedObject**），因为这些写入总是跟随锁释放，以保持表更新的顺序一致性。

##### 数据结构

![1625286571593](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1625286571593.png)

##### 构造方法

- **空参的构造方法**：使用默认初始容量 (16)、负载因子 (0.75) 和 concurrencyLevel (16) 创建一个新的空映射。
- **指定初始容量的构造方法**：使用指定的初始容量、默认负载因子 (0.75) 和 concurrencyLevel (16) 创建一个新的空映射。
- **指定初始容量和负载因子的构造方法**：使用指定的初始容量和负载因子以及默认的 concurrencyLevel (16) 创建一个新的空映射。
- **指定初始容量、负载因子和并发数量的构造方法**：使用指定的初始容量、负载因子和并发数量创建一个新的空映射, segment[]长度sszie为2的幂次(concurrencyLevel), 每个segment里table的长度为max(2的幂次(initialCapacity/sszie), 2)。
- **指定复制集合的构造方法**：创建一个与给定Map具有相同映射的新Map。该映射的创建容量为给定映射中映射数的1.5倍或16（以较大者为准），以及默认负载因子(0.75)和concurrencyLevel(16)。

```java
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V> implements ConcurrentMap<K, V>, Serializable {
    
    static final int DEFAULT_INITIAL_CAPACITY = 16;
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    static final int DEFAULT_CONCURRENCY_LEVEL = 16;
    static final int MAXIMUM_CAPACITY = 1 << 30;
    static final int MIN_SEGMENT_TABLE_CAPACITY = 2;// 每段表的最小容量，至少为2的2的幂
    static final int MAX_SEGMENTS = 1 << 16;// 允许的最大段数，必须是小于 1 << 24 的2的幂
    
    // 避免size（）、containsValue （）在表进行连续修改时无限制的重试
    static final int RETRIES_BEFORE_LOCK = 2;
    
    // 哈希种子可以让哈希算法更复杂一点，从而减少哈希碰撞的概率，如果为0代表哈希种子不生效
    private transient final int hashSeed = randomHashSeed(this);
    
    // ssize-1, 得到Segment[]长度的掩码, 在(hash(key) >>> segmentShift) & segmentMask中, 用来判断key到底落在哪个segment上
    final int segmentMask;
   
    // 32-幂次sshift, 得到剩余高位的位数, 在(hash(key) >>> segmentShift) & segmentMask中, 用来判断key到底落在哪个segment上
    final int segmentShift;
    
    // Segment[], 代表ConcurrentHashMap的直接数据结构, 里面每个Segment含有table, 可以看作是一个小的HashMap
    final Segment<K,V>[] segments;

    // Unsafe mechanics
    private static final sun.misc.Unsafe UNSAFE;
    
    // Segment[]数组起始偏移, ConcurrentHashMap中的segments
    private static final long SBASE;
    
    // numberOfLeadingZeros得到ss高位1前的0的位数, 31-numberOfLeadingZeros得到ss高位1后的位数 => 如果ss为2的幂次数, 则J << SSHIFT + BASE 等价于 j * ss + BASE
    private static final int SSHIFT;
    
    // HashEntry[]数组起始偏移, 每个Segement中的table
    private static final long TBASE;
    
    // numberOfLeadingZeros得到ss高位1前的0的位数, 31-numberOfLeadingZeros得到ss高位1后的位数 => 如果ss为2的幂次数, 则J << SSHIFT + BASE 等价于 j * ss + BASE
    private static final int TSHIFT;
    
    // hashSeed, 哈希种子
    private static final long HASHSEED_OFFSET;
    
    // segmentShift, 用来判断key到底落在哪个segment上
    private static final long SEGSHIFT_OFFSET;
    
    // segmentMask, 用来判断key到底落在哪个segment上
    private static final long SEGMASK_OFFSET;
    
    // segments, Segment[]
    private static final long SEGMENTS_OFFSET;
    ...

    // 使用默认初始容量 (16)、负载因子 (0.75) 和 concurrencyLevel (16) 创建一个新的空映射。
    public ConcurrentHashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
    }
    
    // 使用指定的初始容量、默认负载因子 (0.75) 和 concurrencyLevel (16) 创建一个新的空映射。
    public ConcurrentHashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
    }
    
    // 使用指定的初始容量和负载因子以及默认的 concurrencyLevel (16) 创建一个新的空映射。
    public ConcurrentHashMap(int initialCapacity, float loadFactor) {
        this(initialCapacity, loadFactor, DEFAULT_CONCURRENCY_LEVEL);
    }
    
    // 使用指定的初始容量、负载因子和并发数量创建一个新的空映射, Segment[]长度sszie为2的幂次(concurrencyLevel), 每个segment里table的长度为max(2的幂次(initialCapacity/sszie), 2)
    public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        
        // 如果指定的concurrencyLevel 大于 允许的最大段数1<< 16, 则设置concurrencyLevel为允许的最大段数
        if (concurrencyLevel > MAX_SEGMENTS)
            concurrencyLevel = MAX_SEGMENTS;

        // 查找最佳匹配参数的二的幂
        int sshift = 0;// ssize的幂次sshift, 代表Segment[]长度在低位占据几个1
        int ssize = 1;// ssize=2的幂次(concurrencyLevel), 用作Segment[]长度
        while (ssize < concurrencyLevel) {
            ++sshift;
            ssize <<= 1;
        }
        
        // 32-幂次sshift, 得到剩余高位的位数, 在(hash(key) >>> segmentShift) & segmentMask中, 用来判断key到底落在哪个segment上
        this.segmentShift = 32 - sshift;
        
        // ssize-1, 得到segment[]长度的掩码, 在(hash(key) >>> segmentShift) & segmentMask中, 用来判断key到底落在哪个segment上
        this.segmentMask = ssize - 1;

        // c向上取整initialCapacity/ssize; cap取最接近c的2次幂, 用作segment中table的长度(至少为2)
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        int c = initialCapacity / ssize;
        if (c * ssize < initialCapacity)
            ++c;
        int cap = MIN_SEGMENT_TABLE_CAPACITY;
        while (cap < c)
            cap <<= 1;

        // create segments and segments[0]
        // 创建第一个segment s0, 其中的散列表table(长度为cap), 并指定负载因子等于传入的负载因子, 阈值等于cap*loadFactor
        Segment<K,V> s0 = new Segment<K,V>(loadFactor, (int)(cap * loadFactor), (HashEntry<K,V>[])new HashEntry[cap]);

        // 构造ssize长度的Segment[] ss, 有序地写入段[0], 并更新segments指针
        Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
        UNSAFE.putOrderedObject(ss, SBASE, s0);
        this.segments = ss;
    }
    
    // 创建一个与给定Map具有相同映射的新Map。该映射的创建容量为给定映射中映射数的1.5倍或16（以较大者为准），以及默认负载因子(0.75)和concurrencyLevel(16)。
    public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                      DEFAULT_INITIAL_CAPACITY),
             DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
        
        // 添加复制集合的的所有键值对
        putAll(m);
    }
}
```

##### Segment[]长度规范计算方法

- **初始长度**：规范化Segment数组长度ssize，逻辑在三参数的构造方法里，其中如果 concurrencyLevel为2的幂次，则 concurrencyLevel就作为Segment[]的长度，否则**ssize取大于且最接近concurrencyLevel的2的幂次数**。
- **Segment[]长度一但确定后，则不会再改变了**。

```java
// 查找最佳匹配参数的二的幂
int sshift = 0;// ssize的幂次sshift, 代表Segment[]长度在低位占据几个1
int ssize = 1;// ssize=2的幂次(concurrencyLevel), 用作Segment[]长度
while (ssize < concurrencyLevel) {
    ++sshift;
    ssize <<= 1;
}
```

##### 迭代方法

- **ConcurrentHashMap.HashIterator迭代器**：HashMap的迭代器基类，**非快速失败机制**，advance方法提前提前缓存nextValue，因此该迭代器是**弱一致性**的。返回ConcurrentHashMap.HashEntry对象。
- **Map.Entry迭代器**：继承ConcurrentHashMap.HashIterator迭代器，**非快速失败机制**，advance方法提前提前缓存nextValue，因此该迭代器是**弱一致性**的。返回ConcurrentHashMap.WriteThroughEntry对象。
- **ConcurrentHashMap.HashEntry#Key迭代器**：继承ConcurrentHashMap.HashIterator迭代器，**非快速失败机制**，advance方法提前提前缓存nextValue，因此该迭代器是**弱一致性**的。返回ConcurrentHashMap.HashEntry#Key对象。
- **ConcurrentHashMap.HashEntry#Value迭代器**：继承ConcurrentHashMap.HashIterator迭代器，**非快速失败机制**，advance方法提前提前缓存nextValue，因此该迭代器是**弱一致性**的。返回ConcurrentHashMap.HashEntry#Valuie对象。

```java
// HashMap的迭代器基类，非快速失败机制，advance方法提前提前缓存nextValue，因此该迭代器是弱一致性的。返回ConcurrentHashMap.HashEntry对象。
abstract class HashIterator {
    int nextSegmentIndex;// Segment[]索引
    int nextTableIndex;// Segment#HashEntry[]索引
    HashEntry<K,V>[] currentTable;// Segment#HashEntry[]
    HashEntry<K, V> nextEntry;// 缓存下一个要返回的结点
    HashEntry<K, V> lastReturned;// 当前返回的结点

    HashIterator() {
        nextSegmentIndex = segments.length - 1;
        nextTableIndex = -1;
        advance();// 缓存下一个要返回的结点
    }

    // 缓存下一个要返回的结点
    final void advance() {
        for (;;) {
            // 倒序遍历Segment中的散列表
            if (nextTableIndex >= 0) {
                if ((nextEntry = entryAt(currentTable, nextTableIndex--)) != null)
                    break;
            }
            // 倒序遍历Segment[]
            else if (nextSegmentIndex >= 0) {
                Segment<K,V> seg = segmentAt(segments, nextSegmentIndex--);
                if (seg != null && (currentTable = seg.table) != null)
                    nextTableIndex = currentTable.length - 1;
            }
            else
                break;
        }
    }

    public final boolean hasNext() { return nextEntry != null; }
    
    // 返回nextEntry
    final HashEntry<K,V> nextEntry() {
        HashEntry<K,V> e = nextEntry;
        if (e == null)
            throw new NoSuchElementException();
        lastReturned = e; // cannot assign until after null check
        if ((nextEntry = e.next) == null)
            advance();// 缓存下一个要返回的结点
        return e;
    }
}

// 继承ConcurrentHashMap.HashIterator迭代器，非快速失败机制，advance方法提前提前缓存nextValue，因此该迭代器是弱一致性的。返回ConcurrentHashMap.WriteThroughEntry对象。
final class EntryIterator extends  HashIterator implements Iterator<Entry<K,V>> {
    public Map.Entry<K,V> next() {
        HashEntry<K,V> e = super.nextEntry();
        return new WriteThroughEntry(e.key, e.value);
    }
}
// EntryIterator.next() 使用的自定义条目类
final class WriteThroughEntry extends AbstractMap.SimpleEntry<K,V> {
    WriteThroughEntry(K k, V v) {
        super(k,v);
    }

    // 替换当前Entry值为value后, 再调用ConcurrentHashMap#put方法替换key对应结点的值
    public V setValue(V value) {
        if (value == null) throw new NullPointerException();
        V v = super.setValue(value);
        ConcurrentHashMap.this.put(getKey(), value);
        return v;
    }
}

// 继承ConcurrentHashMap.HashIterator迭代器，非快速失败机制，advance方法提前提前缓存nextValue，因此该迭代器是弱一致性的。返回ConcurrentHashMap.HashEntry#Key对象。
final class KeyIterator extends HashIterator implements Iterator<K>, Enumeration<K> {
    public final K next()        { return super.nextEntry().key; }
    public final K nextElement() { return super.nextEntry().key; }
}

// 继承ConcurrentHashMap.HashIterator迭代器，非快速失败机制，advance方法提前提前缓存nextValue，因此该迭代器是弱一致性的。返回ConcurrentHashMap.HashEntry#Valuie对象。
final class ValueIterator extends HashIterator implements Iterator<V>, Enumeration<V> {
    public final V next()        { return super.nextEntry().value; }
    public final V nextElement() { return super.nextEntry().value; }
}
```

##### 初始化哈希种子

- **randomHashSeed（ConcurrentHashMap）**：返回一个非零的32位伪随机值, 其中当前ConcurrentHashMap实例用作值的一部分, 构造新的哈希种子。只在这里被调用。

```java
// 哈希种子可以让哈希算法更复杂一点，从而减少哈希碰撞的概率，如果为0代表哈希种子不生效
private transient final int hashSeed = randomHashSeed(this);

// 返回一个非零的32位伪随机值, 其中当前ConcurrentHashMap实例用作值的一部分, 构造新的哈希种子
private static int randomHashSeed(ConcurrentHashMap instance) {
    if (sun.misc.VM.isBooted() && Holder.ALTERNATIVE_HASHING) {
        return sun.misc.Hashing.randomHashSeed(instance);
    }

    return 0;
}
```

##### HashCode扰动函数

- **hash（Object）**：
  - HashCode扰动函数, key为String类型则返回key的32位哈希码即可, 否则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响。
  - **产生的Hash值既会用来定位Segment索引, 又会定位Segment#table中的索引**。

```java
// HashCode扰动函数, key为String类型则返回key的32位哈希码即可, 否则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
private int hash(Object k) {
    // 哈希种子可以让哈希算法更复杂一点，从而减少哈希碰撞的概率，如果为0代表哈希种子不生效
    int h = hashSeed;

    // 如果k为String类型, 则返回k的32位哈希值
    if ((0 != h) && (k instanceof String)) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    // 如果k不为String类型, 则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    // 如果h为默认0, 则h异或k的散列表码, 还是会等于k的散列码
    h ^= k.hashCode();

    // 产生的Hash值既会用来定位Segment索引, 又会定位Segment#table中的索引 => 把高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    h += (h <<  15) ^ 0xffffcd7d;
    h ^= (h >>> 10);
    h += (h <<   3);
    h ^= (h >>>  6);
    h += (h <<   2) + (h << 14);
    return h ^ (h >>> 16);
}
```

##### 获取/创建Segment方法

- **ensureSegment（int）**：
  - 返回k段, 如果不存在, 则**创建**它，并通过CAS记录在segment[k]中。
  - 被size、containsValue、put、putIfAbsent、writeObject方法调用。
- **segmentForHash（int）**：
  - 获取给定hash值的段，**不会去创建k段Segment**。
  - 被remove、replace方法调用。
- **segmentAt（Segment[]，int）**：
  - 获取主内存的Segment[j]对象。
  - 被isEmpty、size、containsValue、clear、HashIterator#advance、writeObject方法调用。

```java
// 返回k段, 如果不存在, 则创建它并通过CAS记录在segment[k]中
private Segment<K,V> ensureSegment(int k) {
    // Segment数组segments, k段偏移u, k段seg
    final Segment<K,V>[] ss = this.segments;
    long u = (k << SSHIFT) + SBASE; // raw offset
    Segment<K,V> seg;

    // 如果k段seg为null, 说明k段Segment还没初始化, 则使用段0作为原型创建Segment
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
        // 第0段proto, proto散列表容量cap, proto负载因子lf, cap*lf计算出阈值threshold
        Segment<K,V> proto = ss[0];
        int cap = proto.table.length;
        float lf = proto.loadFactor;
        int threshold = (int)(cap * lf);

        // 使用第0段proto作为原型, 构造出同样容量cap的散列表tab
        HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];

        // 再次检查k段是否为null, 如果为null, 说明还没被其他线程赋值, 则构建Segment对象, 并CAS更新到k段位置
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
            Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
            while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
                if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                    break;
            }
        }
    }

    // 最后无论当前线程实际有没有创建k段Segment, 都返回k段Segment
    return seg;
}

// 获取给定hash值的段
private Segment<K,V> segmentForHash(int h) {
    // h >>> segmentShift, 相当于取 hash的高ssize位数值 % (ssize-1), 得到hash在segment[]中的索引, 而SBASE为Segment[]的起始内存地址, ss为数组中每个对象的内存块大小, SSHIFT为sc高位1后的位数, 当ss为2的幂次时, j<<SSHIFT相当于j*ss, (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE, 直接获取Segment[]中hash值对应的索引j个对象
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    return (Segment<K,V>) UNSAFE.getObjectVolatile(segments, u);
}

// 获取主内存的Segment[j]对象
static final <K,V> Segment<K,V> segmentAt(Segment<K,V>[] ss, int j) {
    // j为在Segment[]中的索引, 而SBASE为Segment[]的起始内存地址, ss为数组中每个对象的内存块大小, SSHIFT为sc高位1后的位数, 当ss为2的幂次时, j<<SSHIFT相当于j*ss, 因此u相当于Segment[j]对象的偏移
    long u = (j << SSHIFT) + SBASE;

    // 获取主内存的Segment[j]
    return ss == null ? null : (Segment<K,V>) UNSAFE.getObjectVolatile(ss, u);
}
```

##### 获取/更新HashEntry方法

- **entryAt（HashEntry[]，int）**：
  - 使用volatile读取语义，获取给定表tab中的第i个元素。
  - 被Segment#put、Segment#remove、containsValue、HashIterator#advance、writeObject方法调用。
- **entryForHash（Segment，int）**：
  - 获取指定段seg中散列表的给定hash值的条目。
  - Segment#scanAndLockForPut、Segment#scanAndLock、Segment#replace方法调用。
- **setEntryAt（HashEntry[]，int， HashEntry）**：
  - 设置给定表tab中的第i个元素。
  - 被Segment#put、Segment#remove、Segment#clear方法调用。

```java
// 使用volatile读取语义，获取给定表tab中的第i个元素
static final <K,V> HashEntry<K,V> entryAt(HashEntry<K,V>[] tab, int i) {
    return (tab == null) ? null : 
    (HashEntry<K,V>) UNSAFE.getObjectVolatile(tab, ((long)i << TSHIFT) + TBASE);
}

// 获取指定段seg中散列表的给定hash值的条目
static final <K,V> HashEntry<K,V> entryForHash(Segment<K,V> seg, int h) {
    HashEntry<K,V>[] tab;
    return (seg == null || (tab = seg.table) == null) ? null :
    // (tab.length - 1) & h, 计算出h所在散列表的索引, 再((tab.length - 1) & h <<< TSHIFT) + TBASE获取当前索引在table数组中的对象
    (HashEntry<K,V>) UNSAFE.getObjectVolatile(tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
}

// 设置给定表tab中的第i个元素
static final <K,V> void setEntryAt(HashEntry<K,V>[] tab, int i, HashEntry<K,V> e) {
    UNSAFE.putOrderedObject(tab, ((long)i << TSHIFT) + TBASE, e);
}
```

##### Segment

###### Segment & HashEntry

```java
// Segment[]中的每段Segment, 类似于一个HashMap, 但本身又继承了ReentrantLock, 所以每段Segment还是一把锁, 更新Segment需要获取该锁, 从而保证线程安全
static final class Segment<K,V> extends ReentrantLock implements Serializable {
    // 预扫描中tryLock的最大次数
    static final int MAX_SCAN_RETRIES = 
        Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;

    // Segment[]中的每段Segment中的散列表, 相当于HashMap中的table
    transient volatile HashEntry<K,V>[] table;

    // Segment[]中的每段Segment中的散列表元素个数, 相当于HashMap中的table的实际大小
    transient int count;

    // Segment[]中的每段Segment中的散列表修改模数, 相当于HashMap中的table的修改模数
    transient int modCount;

    // Segment[]中的每段Segment中的散列表阈值, 相当于HashMap中的table的阈值
    transient int threshold;

    // Segment[]中的每段Segment中的散列表负载因子, 相当于HashMap中的table的负载因子
    final float loadFactor;

    // 指定负载因子、阈值、散列表来构造段
    Segment(float lf, int threshold, HashEntry<K,V>[] tab) {
        this.loadFactor = lf;
        this.threshold = threshold;
        this.table = tab;
    }
}

// Segment[]中的每段Segment中table的结点类型, 相当于HashMap中的Entry
static final class HashEntry<K,V> {
    final int hash;
    final K key;
    volatile V value;
    volatile HashEntry<K,V> next;

    HashEntry(int hash, K key, V value, HashEntry<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```

###### table容量规范计算方法

- **table初始容量**：规范化Segment#table即HashEntry[]长度，逻辑在三参数的构造方法里，其中**c向上取整initialCapacity/ssize**，table初始容量为**cap取大于且最接近c的2的幂次数**。
- **table扩容后的容量**：int newCapacity = oldCapacity << 1，即**新容量等于旧容量的2倍**。

```java
// c向上取整initialCapacity/ssize; cap取最接近c的2次幂, 用作segment中table的长度(至少为2)
if (initialCapacity > MAXIMUM_CAPACITY)
    initialCapacity = MAXIMUM_CAPACITY;
int c = initialCapacity / ssize;
if (c * ssize < initialCapacity)
    ++c;
int cap = MIN_SEGMENT_TABLE_CAPACITY;
while (cap < c)
    cap <<= 1;
```

###### 扩容方法

- **rehash（HashEntry）**：
  - 扩容当前segment的散列表，重新计算每个链表结点hash值对应新表的索引，并**分成高低链表转移到新表中**，最后把要插入的node结点也插入到新表中，更新当前segment的散列表为新表。
  - 对于比JDK7 HashMap，JDK7 ConcurrentHashMap扩容转移元素时，并不需要判断是否已经切换了哈希算法（哈希种子），而是**仍然使用原key的hash值**，通过低位链表分块的转移到新表中。
  - 只被Segment#put方法调用。

```java
// 扩容当前segment的散列表, 重新计算每个链表结点hash值对应新表的索引, 并分成高低链表转移到新表中, 最后把要插入的node结点也插入到新表中, 更新当前segment的散列表为新表
private void rehash(HashEntry<K,V> node) {
    // 旧表oldTable, 旧表容量oldCapacity, 新表容量newCapacity, 新表阈值threshold, 新表掩码sizeMask
    HashEntry<K,V>[] oldTable = table;
    int oldCapacity = oldTable.length;
    int newCapacity = oldCapacity << 1;
    threshold = (int)(newCapacity * loadFactor);
    HashEntry<K,V>[] newTable =
        (HashEntry<K,V>[]) new HashEntry[newCapacity];
    int sizeMask = newCapacity - 1;

    // 从头开始遍历旧表oldTable, 当前遍历结点e
    for (int i = 0; i < oldCapacity ; i++) {
        HashEntry<K,V> e = oldTable[i];

        // 如果e结点不为null, 说明e桶链表存在结点, 则遍历e链表, 后继next, e桶头结点在新表中的索引idx
        if (e != null) {
            HashEntry<K,V> next = e.next;
            int idx = e.hash & sizeMask;

            // 如果后继为null, 说明e链表只有一个桶头结点e, 则移动e到新表即可
            if (next == null)
                newTable[idx] = e;
            // 如果后继不为null, 说明需要转移链表到新表中
            else {
                HashEntry<K,V> lastRun = e;
                int lastIdx = idx;

                // 遍历e链表, 找出最后一段头开始的结点lastRun(一般来说只有两段, 因为扩容成两倍只差1位), 对应索引lastIdx
                for (HashEntry<K,V> last = next; last != null; last = last.next) {
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }

                // 找到lastRun后, 转移lastRun链表到新表的lastIdx位置, 完成高位链表转移
                newTable[lastIdx] = lastRun;

                // Clone remaining nodes 克隆剩余节点
                // 如果低位还有结点, 则从头开始遍历到lastRun之前, 头插法插入每个遍历到的结点到新表的k位置
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }

    // 重新计算要插入node结点在新表中的索引, 并更新node.next为新表索引中的桶头结点, 头插法把node结点插入新表中
    int nodeIndex = node.hash & sizeMask;
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;

    // 更新当前segment的散列表为新表
    table = newTable;
}
```

###### put自旋获取当前Segment锁方法

- **scanAndLockForPut（K，int，V）**：自旋获取当前segment锁, **自旋期间会尝试创建node结点**，如果自旋次数超过最大值还会阻塞获取锁，直到获取到锁才返回。

```java
// 自旋获取当前segment锁, 自旋期间会尝试创建node结点, 如果自旋次数超过最大值还会阻塞获取锁, 直到获取到锁才返回
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    // 获取指定段seg中散列表的给定hash值的条目first, first备份条目e, 要返回的条目node, 新桶头结点f, 重试次数retries(初始时为-1)
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1;

    // 快速失败自旋获取当前segment锁, 如果获取不到, 则期间会尝试创建node结点
    while (!tryLock()) {
        HashEntry<K,V> f; 

        // 如果retries < 0, 说明node结点还没找到或者创建, 则遍历e链表
        if (retries < 0) {
            // 如果e为null, 说明要么桶头结点first为null, 要么遍历到了e链尾还没找到node结点, 则创建node结点
            if (e == null) {
                if (node == null)
                    node = new HashEntry<K,V>(hash, key, value, null);

                // 更新retries为0, 代表node结点已创建
                retries = 0;
            }
            // 如果e不为null, 说明正在遍历链表, 如果key euqals e键, 说明e就是要找的结点, 则更新retries为0, 代表node结点已找到, 此时返回node结点为不为null也没关系, 因为外层逻辑不会走node设置到桶头结点的逻辑
            else if (key.equals(e.key))
                retries = 0;
            // 如果e不为null, 且key不equals e键, 则继续遍历e链表
            else
                e = e.next;
        }

       // 如果retries >= 0, 说明node结点已找到或者已创建, 则继续自旋获取锁, 直到达到最大重试次数
        else if (++retries > MAX_SCAN_RETRIES) {
            // 否则, 阻塞获取锁, 直到获取到锁才返回
            lock();
            break;
        }

        // 如果自旋次数达到了最大值, 且重试次数为奇数次时, 则重新获取指定段seg中散列表的给定hash值的条目f, 如果f不等于first, 说明桶头结点被更改了, 则重新更新first和e为新的桶头结点, 且重新自旋
        else if ((retries & 1) == 0 && (f = entryForHash(this, hash)) != first) {
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }

    // 自旋结束, 代表当前线程获取到了当前segment锁
    // node结点为null代表在e链表中找到了key对应的结点;
    // node结点不为null, 可能代表无论是新旧e链表中, 肯定不存在key对应的结点, 此时node结点返回后肯定会被当做桶头结点插入
    // node结点不为null, 可能还代表原e链表中找不到且创建了node结点, 但原e链表被其他线程更新了, 且新e链表存在key对应的结点, 此时返回node结点为不为null也没关系, 因为外层逻辑不会走node设置到桶头结点的逻辑
    return node;
}

```

###### remove/replace自旋获取当前Segment锁方法

- **scanAndLock（Object，int）**：自旋获取当前segment锁，**自旋期间不会尝试创建node结点但会遍历e链表**，如果自旋次数超过最大值还会阻塞获取锁，直到获取到锁才返回。

```java
// 自旋获取当前segment锁, 自旋期间不会尝试创建node结点但会遍历e链表, 如果自旋次数超过最大值还会阻塞获取锁, 直到获取到锁才返回
private void scanAndLock(Object key, int hash) {
    // similar to but simpler than scanAndLockForPut 类似于但比 scanAndLockForPut 更简单
    // 获取指定段seg中散列表的给定hash值的条目first, first备份条目e, 新桶头结点f, 重试次数retries(初始时为-1)
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    int retries = -1;

    // 快速失败自旋获取当前segment锁, 对比scanAndLockForPut, 如果获取不到, 则期间并不会尝试创建node结点
    while (!tryLock()) {
        HashEntry<K,V> f;

        // 如果retries < 0, 说明node结点还没已找到或者e链表还没遍历完, 则遍历e链表
        if (retries < 0) {
            // 如果e为null, 说明遍历到了e链尾还没找到node结点, 或者如果找到了key对应的node, 则停止遍历
            if (e == null || key.equals(e.key))
                // 更新retries为0, 代表node结点已找到或者已经遍历完了
                retries = 0;
            // 如果e不为null, 且key不equals e键, 则继续遍历e链表
            else
                e = e.next;
        }

        // 如果retries >= 0, 说明node结点已找到或者已经遍历完了, 则继续自旋获取锁, 直到达到最大重试次数
        else if (++retries > MAX_SCAN_RETRIES) {
            // 否则, 阻塞获取锁, 直到获取到锁才返回
            lock();
            break;
        }

        // 如果自旋次数达到了最大值, 且重试次数为奇数次时, 则重新获取指定段seg中散列表的给定hash值的条目f, 如果f不等于first, 说明桶头结点被更改了, 则重新更新first和e为新的桶头结点, 且重新自旋
        else if ((retries & 1) == 0 && (f = entryForHash(this, hash)) != first) {
            e = first = f;
            retries = -1;
        }
    }
}
```

###### 添加元素方法

- **put（K，int，V，boolean）**：**获取当前segment锁后**, 往其散列表添加key-value条目, onlyIfAbsent为true代表只允许key对应结点不存在时才添加, 为false代表key对应结点已存在时可以发生值替换; 如果散列表实际大小超过阈值, 则还会**发生扩容**。

```java
// 获取当前segment锁后, 往其散列表添加key-value条目, onlyIfAbsent为true代表只允许key对应结点不存在时才添加, 为false代表key对应结点已存在时可以发生值替换; 如果散列表实际大小超过阈值, 则还会发生扩容
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 快速失败方式获取当前segment锁, 如果获取到, 则node为null, 代表node结点没有创建; 否则自旋获取当前segment锁, 自旋期间会尝试创建node结点, 如果自旋次数超过最大值还会阻塞获取锁, 直到获取到锁才返回
    HashEntry<K,V> node = tryLock() ? null : scanAndLockForPut(key, hash, value);
    V oldValue;

    try {
        // 到这里, 当前线程获取到了当前segment锁, 提前创建的结点node, segment散列表tab, hash值表索引为index
        HashEntry<K,V>[] tab = table;
        int index = (tab.length - 1) & hash;

        // 通过Unsafe方法获取主内存中tab表index位置的元素first, 此时first为最新的桶头结点, 且当前处于锁定状态中, 肯定不会被其他线程更新了
        HashEntry<K,V> first = entryAt(tab, index);

        // 遍历first桶链表, 当前遍历元素e
        for (HashEntry<K,V> e = first;;) {
            // 如果e不为null, 说明需要判断e链表中是否已存在key对应的元素
            if (e != null) {
                K k;

                // 如果key等于e键或者e的hash值等于hash且key equals e键, 说明找到了key对应的结点e
                if ((k = e.key) == key || (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;

                    // 此时如果需要果只允许在不存在时才添加, 则什么也不做, 否则替换旧值, 结束遍历
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }

                // 如果没找到key对应结点e, 则继续遍历
                e = e.next;
            }

            // 如果e为null, 说明first链表中不存在key对应的元素, 则头插法插入一个新的entry
            else {
                // 如果node结点不为null, 说明node结点已经提前创建了, 则头插法插入到桶头结点即可
                if (node != null)
                    node.setNext(first);
                // 如果node结点为null, 说明node结点没有提前创建, 则在桶头位置创建node结点
                else
                    node = new HashEntry<K,V>(hash, key, value, first);

                // node结点添加到桶头结点后, 如果当前segment的实际大小大于阈值, 且容量小于最大容量, 则扩容当前segment的散列表table
                int c = count + 1;
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    // 由于这里已经对当前segment进行上锁了, 也就是当前table不会被其他线程更改, 因此扩容也是线程安全的
                    // 扩容当前segment的散列表, 重新计算每个链表的hash值, 并分成高低链表转移到新表中, 最后把要插入的node结点也插入到新表中, 更新当前segment的散列表为新表
                    rehash(node);

                // 如果当前segment的散列表不需要扩容, 则通过Unsafe方法更新主内存中tab表i位置的元素为e
                else
                    setEntryAt(tab, index, node);

                // 最后更新修改模数、实际大小, 设置旧值为null, 代表插入成功, 结束遍历
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        // 最后释放segment锁
        unlock();
    }

    // 返回null代表添加成功, 返回旧值代表key对应结点已存在, 当onlyIfAbsent为false则还发生了值替换
    return oldValue;
}
```

###### 删除元素方法

- **remove（Object，int，Object）**：**获取当前segment锁后**， 删除散列表中指定key对应的结点，如果指定了value则key、value都匹配才能删除结点。删除成功则返回旧值，否则返回null。

```java
// 获取当前segment锁后, 删除散列表中指定key对应的结点, 如果指定了value则key、value都匹配才能删除结点, 删除成功则返回旧值, 否则返回null
final V remove(Object key, int hash, Object value) {
    // 快速失败方式获取当前segment锁, 如果获取失败, 则自旋获取当前segment锁, 自旋期间不会尝试创建node结点但会遍历e链表, 如果自旋次数超过最大值还会阻塞获取锁, 直到获取到锁才返回
    if (!tryLock())
        scanAndLock(key, hash);
    V oldValue = null;

    try {
        // 到这里, 当前线程获取到了当前segment锁, segment散列表tab, hash值表索引为index
        HashEntry<K,V>[] tab = table;
        int index = (tab.length - 1) & hash;

        // 通过Unsafe方法获取主内存中tab表i位置的元素e, 前驱pred
        HashEntry<K,V> e = entryAt(tab, index);
        HashEntry<K,V> pred = null;

        // 如果e不为null, 说明需要遍历e链表, e键k, e后继next, e值v
        while (e != null) {
            K k;
            HashEntry<K,V> next = e.next;

            // 如果e键等于key, 或者e的hash值等于hash且e键equals key, 说明e就是要找的结点
            if ((k = e.key) == key || (e.hash == hash && key.equals(k))) {
                V v = e.value;

                // 如果没有指定value, 说明e就是要找的结点, 需要删除
                // 如果指定了value, 则还需要匹配value是否相等, 如果e值等于value, 或者e值equals value, 说明e就是要找的结点, 需要删除
                if (value == null || value == v || value.equals(v)) {
                    // 如果前驱为null, 说明e为桶头结点, 则更新后继作为桶头结点, 脱钩e结点
                    if (pred == null)
                        setEntryAt(tab, index, next);
                    // 如果前驱不为null, 说明e不为桶头结点, 则链接前驱与后继, 脱钩e结点
                    else
                        pred.setNext(next);

                    // 删除成功后, 则更新修改模数、更新实际大小、设置旧值为e值v, 结束遍历
                    ++modCount;
                    --count;
                    oldValue = v;
                }
                break;
            }
            pred = e;
            e = next;
        }
    } finally {
        // 最后释放segment锁
        unlock();
    }

    // 删除成功则返回旧值, 否则返回null
    return oldValue;
}

```

##### 添加元素方法

所有方法**key和value都不能为null**，且底层都依赖Segment#put（K，int，V，boolean）方法。

- **put（K，V）**：根据key的hash值选择segment后, 获取当前segment锁后, 往其散列表添加key-value条目(**允许值替换**), 并且如果插入后散列表实际大小超过阈值, 则还会发生扩容。
- **putIfAbsent（K，V）**：根据key的hash值选择segment后, 获取当前segment锁后, 往其散列表添加key-value条目(**不允许值替换**), 并且如果插入后散列表实际大小超过阈值, 则还会发生扩容。
- **putAll（Map）**：添加复制集合的的所有键值对，**底层依赖put（K，V）允许值替换**。

```java
// 根据key的hash值选择segment后, 获取当前segment锁后, 往其散列表添加key-value条目(允许值替换), 并且如果插入后散列表实际大小超过阈值, 则还会发生扩容
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null) throw new NullPointerException();

    // HashCode扰动函数, key为String类型则返回key的32位哈希码即可, 否则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    int hash = hash(key);

    // 相当于取hash的高ssize位数值 % (ssize-1)，得到hash在segment中的索引
    int j = (hash >>> segmentShift) & segmentMask;

    // 获取索引元素，通过UNSAFE方法，直接获取Segment[]中第j个对象
    if ((s = (Segment<K,V>)UNSAFE.getObject (segments, (j << SSHIFT) + SBASE)) == null)
        // 如果第j个对象为null, 则创建它并通过CAS记录在segment[j]中
        s = ensureSegment(j);

    // 获取第j个Segment后, 则获取当前segment锁后, 往其散列表添加key-value条目, onlyIfAbsent为true代表只允许key对应结点不存在时才添加, 为false代表key对应结点已存在时可以发生值替换; 如果插入后散列表实际大小超过阈值, 则还会发生扩容
    return s.put(key, hash, value, false);
}

// 根据key的hash值选择segment后, 获取当前segment锁后, 往其散列表添加key-value条目(不允许值替换), 并且如果插入后散列表实际大小超过阈值, 则还会发生扩容
public V putIfAbsent(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();

    // HashCode扰动函数, key为String类型则返回key的32位哈希码即可, 否则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    int hash = hash(key);

    // 相当于取hash的高ssize位数值 % (ssize-1)，得到hash在segment中的索引
    int j = (hash >>> segmentShift) & segmentMask;

    // 获取索引元素，通过UNSAFE方法，直接获取Segment[]中第j个对象
    if ((s = (Segment<K,V>)UNSAFE.getObject(segments, (j << SSHIFT) + SBASE)) == null)
        // 如果第j个对象为null, 则创建它并通过CAS记录在segment[j]中
        s = ensureSegment(j);

    // 获取第j个Segment后, 则获取当前segment锁后, 往其散列表添加key-value条目, onlyIfAbsent为true代表只允许key对应结点不存在时才添加, 为false代表key对应结点已存在时可以发生值替换; 如果插入后散列表实际大小超过阈值, 则还会发生扩容
    return s.put(key, hash, value, true);
}

// 添加复制集合的的所有键值对
public void putAll(Map<? extends K, ? extends V> m) {
    for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
        // 根据key的hash值选择segment后, 获取当前segment锁后, 往其散列表添加key-value条目(允许值替换), 并且如果插入后散列表实际大小超过阈值, 则还会发生扩容
        put(e.getKey(), e.getValue());
}

```

##### 删除元素方法

- **remove（Object）**：如果key存在，则从此映射中删除该key对应的映射，忽略value值。
- **remove（Object，Object）**：删除key和value都equals的键值对，value值必须匹配。

```java
// 如果key存在，则从此映射中删除该key对应的映射，忽略value值
public V remove(Object key) {
    // HashCode扰动函数, key为String类型则返回key的32位哈希码即可, 否则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    int hash = hash(key);

    // 获取给定hash值的段
    Segment<K,V> s = segmentForHash(hash);

    // 获取当前segment锁后, 删除散列表中指定key对应的结点, 删除成功则返回旧值, 否则返回null
    return s == null ? null : s.remove(key, hash, null);
}

// 删除key和value都equals的键值对，value值必须匹配
public boolean remove(Object key, Object value) {
    // HashCode扰动函数, key为String类型则返回key的32位哈希码即可, 否则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    int hash = hash(key);
    Segment<K,V> s;

    // 获取给定hash值的段, 获取当前segment锁后, 删除散列表key、value都匹配的结点, 删除成功则返回旧值, 否则返回null
    return value != null && (s = segmentForHash(hash)) != null && s.remove(key, hash, value) != null;
}
```

##### 获取元素方法

- **get（Object）**：获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}。

```java
// 获取指定键映射到的值，如果此映射不包含键的映射，则返回 {@code null}
public V get(Object key) {
    Segment<K,V> s; // 手动集成访问方法，以减少开销
    HashEntry<K,V>[] tab;

    // HashCode扰动函数, key为String类型则返回key的32位哈希码即可, 否则哈希种子异或k的散列码, 再把结果高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
    int h = hash(key);

    // 获取h对应Segment[]中的对象偏移u, 根据u获取主内存中的Segment对象s
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null && (tab = s.table) != null) {
        // 然后根据h对应s中散列表tab中的对象偏移, 获取主内存中的HashEntry结点e, 遍历e链表
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile(tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE); e != null; e = e.next) {
            K k;

            // 如果e键等于key, 或者e的hash值等于h且e键equals k, 说明e结点就是要找的结点, 则返回e值即可
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }

    // 确认没找到key对应的结点, 则返回null
    return null;
}
```

##### 元素统计方法

- **size（）**：
  - 返回此映射中键值映射的数量，如果Map包含多个Integer.MAX_VALUE元素，则只返回**Integer.MAX_VALUE**。
  - **先不加锁统计2次**，取前2次统计到的修改模数作比较：
    - 如果相等，说明没有发生并发修改，size统计结果正确，则结束自旋，返回size或Integer.MAX_VALUE。
    - 如果不等，说明发生了并发修改，则创建每个Segment并加锁统计，统计后返回size或Integer.MAX_VALUE。

```java
// 避免size（）、containsValue （）在表进行连续修改时无限制的重试
static final int RETRIES_BEFORE_LOCK = 2;

// 返回此映射中键值映射的数量Map包含多个Integer.MAX_VALUE元素，则只返回Integer.MAX_VALUE: 先不加锁统计2次, 如果前2次统计到的修改模数不等, 说明发生了并发修改, 则创建每个Segment并加锁统计
public int size() {
    // 段数组segments, ConcurrentHashMap实际大小size, int数值是否溢出overflow, 每次统计的修改模数sum, 上一次的修改模数last, 自旋retries
    final Segment<K,V>[] segments = this.segments;
    int size;
    boolean overflow;
    long sum;
    long last = 0L;
    int retries = -1;

    try {
        // 开始自旋
        for (;;) {
            // 如果自旋次数达到加锁前最大自旋次数2, 说明前2次的统计模数发生了变化, 此时需要加锁统计了, 则遍历segments, 对于j段如果不存在, 则创建它并通过CAS记录在segment[j]中, 并阻塞式地对j段Segment进行加锁
            if (retries++ == RETRIES_BEFORE_LOCK) {
                for (int j = 0; j < segments.length; ++j)
                    ensureSegment(j).lock(); // force creation
            }

            // 初始化每次的修改模数统计sum，实际大小size，是否溢出overflow
            sum = 0L;
            size = 0;
            overflow = false;

            // 如果segment都加锁完毕后, 则再次遍历每一段Segment
            for (int j = 0; j < segments.length; ++j) {
                // 获取主内存的Segment[j]对象seg
                Segment<K,V> seg = segmentAt(segments, j);

                // 如果esg不为null, 说明其上的散列表元素需要做统计, 统计其修改模数sum, 统计其实际大小c, size叠加每个c的结果
                if (seg != null) {
                    sum += seg.modCount;
                    int c = seg.count;

                    // 如果最高位为1, 此时为负数, 小于0, 说明溢出了
                    if (c < 0 || (size += c) < 0)
                        overflow = true;
                }
            }

            // 如果统计两次, 修改模数都相等, 说明size统计正确, 则结束自旋
            if (sum == last)
                break;

            // 如果修改模数不相等, 或者还没统计两次以上, 则需要再次统计
            last = sum;
        }
    } finally {
        // 统计完毕, 如果自旋次数有大于2, 说明每段Segment都上了锁, 则释放每段Segment的锁
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                segmentAt(segments, j).unlock();
        }
    }

    // 如果int值溢出, 则返回MAX_VALUE, 否则返回正常的统计结果size
    return overflow ? Integer.MAX_VALUE : size;
}
```

