#### HashMap的源码，实现原理，底层结构
HashMap基于哈希表的Map接口的实现。此实现提供所有可选的映射操作，并允许使用null值和null键。（除了不同步和允许使用null之外，HashMap类与Hashtable大致相同。）null值hashcode为0，所以以null为键的键值对存储在table[0]。

HashMap在1.6中使用链表(Entry<K,V>)加数组(Entry数组)的方式实现，HashMap中主要是通过key的hashCode来计算hash值的，只要hashCode相同，计算出来的hash值就一样。如果发生hash冲突通过链表解决。HashMap中的负载因子loadFactor默认为0.75，resize过程中一般保证为大于initialCapacity的最小的2的n次幂（一般容量初始为16，之后每次乘2）。
>- JDK 1.6 当数量大于容量 * 负载因子即会扩充容量。
- JDK 1.7 初次扩充为：当数量大于容量时扩充；第二次及以后为：当数量大于容量 * 负载因子时扩充。
- JDK 1.8 初次扩充为：与负载因子无关；第二次及以后为：与负载因子有关。

JDK1.8中使用位桶 + 链表 + 红黑树实现，当链表长度超过阈值（8）时，将链表转换为红黑树。
![](http://images2015.cnblogs.com/blog/616953/201603/616953-20160304192851940-1880633940.png)

>HashMap中使用 **`hash&(length-1)`** 计算table数组的索引位置。
因为HashMap底层数组的长度总是2的n次方，当length总是2的n次方时，hash&(length-1)运算等价于对length取模，也就是h%length，但是&比%具有更高的效率。


#### Fail-Fast机制
我们知道java.util.HashMap不是线程安全的，因此如果在使用迭代器的过程中有其他线程修改了map，那么将抛出ConcurrentModificationException，这就是所谓fail-fast策略。

fail-fast机制是java集合(Collection)中的一种错误机制。当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。
例如：当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。

这一策略在源码中的实现是通过modCount域，modCount顾名思义就是修改次数，对HashMap内容（当然不仅仅是HashMap才会有，其他例如ArrayList也会）的修改都将增加这个值（大家可以再回头看一下其源码，在很多操作中都有modCount++这句），那么在迭代器初始化过程中会将这个值赋给迭代器的expectedModCount。
```java
HashIterator() {
    expectedModCount = modCount;
    if (size > 0) { // advance to first entry
    Entry[] t = table;
    while (index < t.length && (next = t[index++]) == null)
        ;
    }
}
```
在迭代过程中，判断modCount跟expectedModCount是否相等，如果不相等就表示已经有其他线程修改了Map：注意到modCount声明为volatile，保证线程之间修改的可见性。
```java
final Entry<K,V> nextEntry() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
```

在HashMap的API中指出：
由所有HashMap类的“collection视图方法”所返回的迭代器都是快速失败的：在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器本身的remove方法，其他任何时间任何方式的修改，迭代器都将抛出ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不冒在将来不确定的时间发生任意不确定行为的风险。
>注意，迭代器的快速失败行为不能得到保证，一般来说，存在非同步的并发修改时，不可能作出任何坚决的保证。快速失败迭代器尽最大努力抛出ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测程序错误。


#### HashMap的遍历方式
```java
Map map = new HashMap();
Iterator iter = map.entrySet().iterator();
while (iter.hasNext()) {
    Map.Entry entry = (Map.Entry) iter.next();
    Object key = entry.getKey();
    Object val = entry.getValue();
}
// http://www.cnblogs.com/imzhj/p/5981665.html
```


#### Hashtable与HashMap的简单比较
1. HashTable基于Dictionary类，而HashMap是基于AbstractMap。Dictionary是任何可将键映射到相应值的类的抽象父类，而AbstractMap是基于Map接口的实现，它以最大限度地减少实现此接口所需的工作。

2. HashMap的key和value都允许为null，而Hashtable的key和value都不允许为null。HashMap遇到key为null的时候，调用putForNullKey方法进行处理，而对value没有处理；Hashtable遇到null，直接返回NullPointerException。

3. Hashtable方法是同步，而HashMap则不是。我们可以看一下源码，Hashtable中的几乎所有的public的方法都是synchronized的，而有些方法也是在内部通过synchronized代码块来实现。所以有人一般都建议如果是涉及到多线程同步时采用HashTable，没有涉及就采用HashMap，但是在Collections类中存在一个静态方法：synchronizedMap，该方法创建了一个线程安全的Map对象，并把它作为一个封装的对象来返回，该对象将传入的Map设置成一个Mutex通过对其加锁实现同步。


#### Hashtable,HashMap,ConcurrentHashMap底层实现原理与线程安全问题
HashMap线程不安全，HashTable通过对主要的方法如put，get加入关键字synchronized进行修饰保证其线程安全。

ConcurrentHashMap在jdk1.7中使用segment进行分段，每个segment包含了一个Entry的table，segment继承与ReentrantLock可以加锁保证其线程安全。
![](http://images2015.cnblogs.com/blog/764863/201606/764863-20160620202714522-1795796503.png)
而在JDK1.8中取消了segment，直接采用transient volatile Node<K,V>[] table 保存数据，对应的数据操作则使用CAS + 同步的方式进行加锁。
>HashMap中的数组数据成员使用transient修饰防止序列化，数组还有很多的空间没有被使用，没有被使用到的空间被序列化没有意义。所以需要手动使用 writeObject() 方法，只序列化实际存储元素的数组。并且由于不同的虚拟机对于相同hashCode产生的Code值可能是不一样的，如果你使用默认的序列化，那么反序列化后，元素的位置和之前的是保持一致的，可是由于hashCode的值不一样了，那么定位函数indexOf（）返回的元素下标就会不同，这样不是我们所想要的结果。


#### LinkedHashMap的实现
**LinkedHashMap**(public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>)底层使用哈希表与双向链表来保存所有元素。其基本操作与父类HashMap相似，它通过重写父类相关的方法，来实现自己的链接列表特性。

LinkedHashMap采用的hash算法和HashMap相同，但是它重新定义了数组中保存的元素Entry，该Entry除了保存当前对象的引用外，还保存了其上一个元素before和下一个元素after的引用，从而在哈希表的基础上又构成了双向链接列表。

```java
/**
* LinkedHashMap的Entry元素。
* 继承HashMap的Entry元素，又保存了其上一个元素before和下一个元素after的引用。
*/
private static class Entry<K,V> extends HashMap.Entry<K,V> {
    Entry<K,V> before, after;
    ……
}
/**
* The iteration ordering method for this linked hash map: <tt>true</tt>
* for access-order, <tt>false</tt> for insertion-order.
* 如果为true，则按照访问顺序；如果为false，则按照插入顺序。
*/
private final boolean accessOrder;
/**
* 双向链表的表头元素。
*/
private transient Entry<K,V> header;
/**
 * The tail (youngest) of the doubly linked list.
*/
transient LinkedHashMap.Entry<K,V> tail;
```

>LinkedHashMap的构造方法中，实际调用了父类HashMap的相关构造方法来构造一个底层存放的table数组，但额外可以增加accessOrder这个参数，如果不设置，默认为false，代表按照插入顺序进行迭代；当然可以显式设置为true，代表以访问顺序进行迭代。

LinkedHashMap并未重写父类HashMap的put方法，而是重写了父类HashMap的put方法调用的子方法void recordAccess(HashMap m)，void addEntry(int hash, K key, V value, int bucketIndex)和void createEntry(int hash, K key, V value, int bucketIndex)，提供了自己特有的双向链接列表的实现。
```java
void recordAccess(HashMap<K,V> m) {
    LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
    if (lm.accessOrder) {
        lm.modCount++;
        remove();
        addBefore(lm.header);
        }
}

void addEntry(int hash, K key, V value, int bucketIndex) {
    // 调用create方法，将新元素以双向链表的的形式加入到映射中。
    createEntry(hash, key, value, bucketIndex);

    // 删除最近最少使用元素的策略定义
    Entry<K,V> eldest = header.after;
    if (removeEldestEntry(eldest)) {
        removeEntryForKey(eldest.key);
    } else {
        if (size >= threshold)
            resize(2 * table.length);
    }
}

void createEntry(int hash, K key, V value, int bucketIndex) {
    HashMap.Entry<K,V> old = table[bucketIndex];
    Entry<K,V> e = new Entry<K,V>(hash, key, value, old);
    table[bucketIndex] = e;
    // 调用元素的addBrefore方法，将元素加入到哈希、双向链接列表。
    e.addBefore(header);
    size++;
}

private void addBefore(Entry<K,V> existingEntry) {
    after  = existingEntry;
    before = existingEntry.before;
    before.after = this;
    after.before = this;
}
```

LinkedHashMap重写了父类HashMap的get方法，实际在调用父类getEntry()方法取得查找的元素后，再判断当排序模式accessOrder为true时，记录访问顺序，将最新访问的元素添加到双向链表的表头，并从原来的位置删除。由于的链表的增加、删除操作是常量级的，故并不会带来性能的损失。

>LinkedHashMap能够做到按照插入顺序或者访问顺序进行迭代














