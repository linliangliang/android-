# HashMap和Hashtable的区别
HashMap和Hashtable都实现了Map接口，但决定用哪一个之前先要弄清楚它们之间的分别。主要的区别有：线程安全性，同步(synchronization)，以及速度。

1. HashMap几乎可以等价于Hashtable，除了HashMap是非synchronized的，并可以接受null(HashMap可以接受为null的键值(key)和值(value)，而Hashtable则不行)。
2. HashMap是非synchronized，而Hashtable是synchronized，这意味着Hashtable是线程安全的，多个线程可以共享一个Hashtable；而如果没有正确的同步的话，多个线程是不能共享HashMap的。Java 5提供了ConcurrentHashMap，它是HashTable的替代，比HashTable的扩展性更好
3. HashMap不能保证随着时间的推移Map中的元素次序是不变的。

4. sychronized意味着在一次仅有一个线程能够更改Hashtable。就是说任何线程要更新Hashtable时要首先获得同步锁，其它线程要等到同步锁被释放之后才能再次获得同步锁更新Hashtable。

**我们能否让HashMap同步？**
HashMap可以通过下面的语句进行同步：
Map m = Collections.synchronizeMap(hashMap);

- 如果你使用Java 5或以上的话，请使用ConcurrentHashMap吧。ConcurrentHashMap虽然也是线程安全的，但是它的效率比Hashtable要高好多倍。因为ConcurrentHashMap使用了分段锁，并不对整个数据进行锁定。

# HashMap和Hashtable的底层实现
## HashMap和Hashtable的底层实现都是数组+链表结构实现的，这点上完全一致
1. HashMap基于hashing原理，我们通过put()和get()方法储存和获取对象，当我们将键值对传递给put()方法时，它调用键对象的hashCode()方法来计算hashcode，让后找到bucket位置来储存值对象。当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。HashMap使用链表来解决碰撞问题，当发生碰撞了，对象将会储存在链表的下一个节点中。 HashMap在每个链表节点中储存键值对对象。

```/**
     * HashMap的默认初始容量 必须为2的n次幂
     */
    static final int DEFAULT_INITIAL_CAPACITY = 16;

    /**
     * HashMap的最大容量，可以认为是int的最大值    
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 默认的加载因子
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * HashMap用来存储数据的数组
     */
    transient Entry[] table;
```

capacity为当前HashMap的Entry数组的大小，**为什么Entry数组的大小是2的N次方？**
```
// Find a power of 2 >= initialCapacity
int capacity = 1;
while (capacity < initialCapacity)
      capacity <<= 1;
```
源于一个数学规律，就是如果length是2的N次方，那么数h对length的模运算结果等价于a和(length-1)的按位与运算，也就是 h%length <=> h&(length-1)。
位运算当然比取余效率高，因此这就解释了：为什么Entry数组的大小是2的N次方？
1. 构造HashMap中的loadFactor（装填因子）
```
threshold = (int)(capacity * loadFactor);
```
threshold为HashMap的size最大值，注意不是HashMap内部数组的大小。

**为什么需要loadFactor，怎么合理的设置loadFactor？**
如果loadFactor很小很小，那么map中的table需要不断的扩容，导致除数越来越大，冲突越来越小！
如果loadFactor很大很大，那么当map中table放满了也不要求扩容，导致冲突越来越多，解决冲突而起的链表越来越长！


- key的hashCode经过hash后，为了让其在table（table为hashMap的entry[]）的范围内，需要再hash一次。这里实际上是采用的“除余法”（h%length）。
