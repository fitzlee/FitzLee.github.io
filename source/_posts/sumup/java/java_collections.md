---
title: java collections源码分析原理
urlname: java_jdk_base_collections_analysis
#date: 2016-03-06
top: true
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 00Java&JDK
tags:
    - java
    - jdk
    - collections
    - 源码
    - 原理
---

## collections大纲
![image.png](https://upload-images.jianshu.io/upload_images/11010834-d3932f2726ac3e77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- List
  ArrayList
  LinkedList
  CopyOnWriteArrayList
- Queue
  LinkedList
  ArrayDeque
  PriorityQueue
  ConcurrentLinkedQueue/Deque
  ArrayBlockingQueue
  LinkedBlockingQueue/Deque
  PriorityBlockingQueue
  SynchronousQueue
  DelayQueue
- Set
  HashSet
  LinkedHashSet
  TreeSet
  ConcurrentSkipListSet
  CopyOnWriteArraySet
- Stack

![image.png](https://upload-images.jianshu.io/upload_images/11010834-a4847c318e44f943.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- Map
  HashMap 
  LinkedHashMap
  TreeMap
  ConcurrentHashMap
  ConcurrentSkipListMap
  WeakHashMap



## ArrayList概述

重要知识点：
1. 实现原理private transient Object[] elementData;
2. 构造函数`public ArrayList()`可以构造一个默认初始容量为10的空列表；
3. 实现接口List<E>, RandomAccess, Cloneable, java.io.Serializable
4. 扩容原理，如下：
    **复制system.copyof Arrays.copyOf**，`System.arraycopy(elementData, index+1, elementData, index, numMoved);  elementData = Arrays.copyOf(elementData, size, Object[].class);`
    **扩容条件**：在add(`ensureCapacityInternal(size + 1);`)或者addAll(`ensureCapacityInternal(size + 1);`)进行判断是否需要扩容。
    **扩容方法**：
```java
    private void ensureCapacityInternal(int minCapacity) {
        modCount++;
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
4. fail-fast机制
    ArrayList也采用了快速失败的机制，通过记录modCount参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。
    机制详解：http://www.cnblogs.com/skywang12345/p/3308762.html

iterater初始化过程中保存一个int expectedModCount = modCount;当remove会修改expectedModCount, 每次执行next或者remove都会checkForComodification，看expectedModCount和modCount是否匹配， modCount会被其他线程在list.add或remove操作给修改掉，就变得不一致了。

```java
package java.util;

public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {

    ...

    // AbstractList中唯一的属性
    // 用来记录List修改的次数：每修改一次(添加/删除等操作)，将modCount+1
    protected transient int modCount = 0;


    public void ensureCapacity(int minCapacity) {
        modCount++; //修改modCount
        int oldCapacity = elementData.length;
        if (minCapacity > oldCapacity) {
            Object oldData[] = elementData;
            int newCapacity = (oldCapacity * 3)/2 + 1;
            if (newCapacity < minCapacity)
                newCapacity = minCapacity;
            // minCapacity is usually close to size, so this is a win:
            elementData = Arrays.copyOf(elementData, newCapacity);
        }
    }

        // 删除指定位置的元素 
    public E remove(int index) {
        RangeCheck(index);

        // 修改modCount
        modCount++;
        E oldValue = (E) elementData[index];

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        elementData[--size] = null; // Let gc do its work

        return oldValue;
    }

    // 返回List对应迭代器。实际上，是返回Itr对象。
    public Iterator<E> iterator() {
        return new Itr();
    }

    // Itr是Iterator(迭代器)的实现类
    private class Itr implements Iterator<E> {
        int cursor = 0;

        int lastRet = -1;

        // 修改数的记录值。
        // 每次新建Itr()对象时，都会保存新建该对象时对应的modCount；
        // 以后每次遍历List中的元素的时候，都会比较expectedModCount和modCount是否相等；
        // 若不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件。
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size();
        }

        public E next() {
            // 获取下一个元素之前，都会判断“新建Itr对象时保存的modCount”和“当前的modCount”是否相等；
            // 若不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件。
            checkForComodification();
            try {
                E next = get(cursor);
                lastRet = cursor++;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public void remove() {
            if (lastRet == -1)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    ...
}
```


<!-- more -->


## LinkedList
1. 实现接口，implements List<E>, Deque<E>, Cloneable, java.io.Serializable
2. 基本原理是双端队列，注意下Node的新建方式，直接赋值pre和next节点, 代码结果如下：
```java
    transient int size = 0;
    transient Node<E> first; //链表的头指针
    transient Node<E> last; //尾指针
    //存储对象的结构 Node, LinkedList的内部类
    private static class Node<E> {
        E item;
        Node<E> next; // 指向下一个节点
        Node<E> prev; //指向上一个节点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```
4. add方法处理，主要是一些判断和链表操作。单Node注意pre和next，整个LinkedList注意first和last的变化。
```java


public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}

/**
    * Links e as first element.
    */
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}

/**
* Links e as last element.
*/
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}

void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

## CopyOnWriteArrayList
在线程对其进行些操作的时候，会拷贝一个新的数组以存放新的字段。
1. `private volatile transient Object[] array;`//保证了线程的可见性，所有操作直接操作主存。
2. 写操作加锁，读操作不加锁，相对来说读的效率更高
3. 可变操作（add()、set() 和 remove() 等等）的开销很大, 因为通常需要复制整个基础数组。
4. 
5. 写操作`Object[] newElements = Arrays.copyOf(elements, len + 1);`直接加锁，复制，新建一个数组，写回主存，get操作不加锁直接操作

```java
    /** The lock protecting all mutators */
    transient final ReentrantLock lock = new ReentrantLock();
 
    /** The array, accessed only via getArray/setArray. */
    private volatile transient Object[] array;//保证了线程的可见性

	public boolean add(E e) {
        final ReentrantLock lock = this.lock;//ReentrantLock 保证了线程的可见性和顺序性，即保证了多线程安全。
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);//在原先数组基础之上新建长度＋1的数组，并将原先数组当中的内容拷贝到新数组当中。
            newElements[len] = e;//设值
            setArray(newElements);//对新数组进行赋值
            return true;
        } finally {
            lock.unlock();
        }
    }

    public E get(int index) {
        return (E)(getArray()[index]);
    }

```

3. 迭代不会出现ConcurrentModificationException,  因为迭代器会保存一个快照`private final Object[] snapshot;`, 而不是直接操作原来的Object[] array，不可变，不会发生同步问题。
    迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等操作, 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照
```java
    public Iterator<E> iterator() {
        return new COWIterator<E>(getArray(), 0);
    }
    public ListIterator<E> listIterator() {
        return new COWIterator<E>(getArray(), 0);
    }
    public ListIterator<E> listIterator(final int index) {
        Object[] elements = getArray();
        int len = elements.length;
        if (index<0 || index>len)
            throw new IndexOutOfBoundsException("Index: "+index);
 
        return new COWIterator<E>(elements, index);
    }
 
    private static class COWIterator<E> implements ListIterator<E> {
        private final Object[] snapshot; // 保存数组的快照，是一个不可变的对象
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
 
        @SuppressWarnings("unchecked")
        public E next() {
            if (! hasNext())
                throw new NoSuchElementException();
            return (E) snapshot[cursor++];
        }
 
        @SuppressWarnings("unchecked")
        public E previous() {
            if (! hasPrevious())
                throw new NoSuchElementException();
            return (E) snapshot[--cursor];
        }
 
        public int nextIndex() {
            return cursor;
        }
 
        public int previousIndex() {
            return cursor-1;
        }
        public void remove() {
            throw new UnsupportedOperationException();
        }
        public void set(E e) {
            throw new UnsupportedOperationException();
        }
        public void add(E e) {
            throw new UnsupportedOperationException();
        }
    }

```

参考：https://blog.csdn.net/mazhimazh/article/details/19210547


## synchronized List

1. 线程安全原理，继承层级
    mutex就是当前的SynchronizedCollection对象，而SynchronizedRandomAccessList继承自SynchronizedList，SynchronizedList又继承自SynchronizedCollection，所以SynchronizedRandomAccessList中的mutex也就是SynchronizedRandomAccessList的this对象。所以在GoodListHelper中使用的锁list对象，和SynchronizedRandomAccessList内部的锁是一致的，所以它可以实现线程安全性。

```java
public static <T> List<T> synchronizedList(List<T> list) {  
    return (list instanceof RandomAccess ?  
                new SynchronizedRandomAccessList<T>(list) :  
                new SynchronizedList<T>(list));  
    }  

 通过源码，我们还需要知道ArrayList是否实现了RandomAccess接口：

public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable  

  查看ArrayList的源码，可以看到它实现了RandomAccess，所以上面的synchronizedList放回的应该是SynchronizedRandomAccessList的实例。接下来看看SynchronizedRandomAccessList这个类的实现：

static class SynchronizedRandomAccessList<E>  extends SynchronizedList<E>  implements RandomAccess {  
        SynchronizedRandomAccessList(List<E> list) {  
            super(list);  
        }  
    SynchronizedRandomAccessList(List<E> list, Object mutex) {  
            super(list, mutex);  
        }  
    public List<E> subList(int fromIndex, int toIndex) {  
        synchronized(mutex) {  
                return new SynchronizedRandomAccessList<E>(  
                    list.subList(fromIndex, toIndex), mutex);  
            }  
        }  
        static final long serialVersionUID = 1530674583602358482L;  
        private Object writeReplace() {  
            return new SynchronizedList<E>(list);  
        }  
    }  

 因为SynchronizedRandomAccessList这个类继承自SynchronizedList，而大部分方法都在SynchronizedList中实现了，所以源码中只包含了很少的方法，但是通过subList方法，我们可以看到这里使用的锁对象为mutex对象，而mutex是在SynchronizedCollection类中定义的，所以再看看SynchronizedCollection这个类中关于mutex的定义部分源码：

static class SynchronizedCollection<E> implements Collection<E>, Serializable {  
    private static final long serialVersionUID = 3053995032091335093L;  
    final Collection<E> c;  // Backing Collection  
    final Object mutex;     // Object on which to synchronize  

    SynchronizedCollection(Collection<E> c) {  
            if (c==null)  
                throw new NullPointerException();  
        this.c = c;  
            mutex = this;  
        }  

    SynchronizedCollection(Collection<E> c, Object mutex) {  
        this.c = c;  
            this.mutex = mutex;  
        }  
}  
```

2. 两种写法造成的线程安全问题
```java
@ThreadSafe  
class GoodListHelper <E> {  
    public List<E> list = Collections.synchronizedList(new ArrayList<E>());  

    public boolean putIfAbsent(E x) {  
        synchronized (list) {  
            boolean absent = !list.contains(x);  
            if (absent)  
                list.add(x);  
            return absent;  
        }  
    }  
}
//mutex也就是SynchronizedRandomAccessList=SynchronizedCollection的this对象。所以在GoodListHelper中使用的锁list=mutex对象，和SynchronizedRandomAccessList内部的锁是一致的，所以它可以实现线程安全性。

@NotThreadSafe  
class BadListHelper <E> {  
    public List<E> list = Collections.synchronizedList(new ArrayList<E>());  

    public boolean putIfAbsent(E x) {  
        boolean absent = !list.contains(x);  
        if (absent)  
            list.add(x);  
        return absent;  
    }  
}
// 不安全 contains 和 add 单独安全，组合就不安全了。   

@NotThreadSafe  
class BadListHelper <E> {  
    public List<E> list = Collections.synchronizedList(new ArrayList<E>());  

    public synchronized boolean putIfAbsent(E x) {  
        boolean absent = !list.contains(x);  
        if (absent)  
            list.add(x);  
        return absent;  
    }  
} 
// 不安全，list是public，可能被外部进行add和remove操作

```

https://www.cnblogs.com/yaowen/p/5983136.html


## Vector
1. 相当于给每一个ArrayList操作加了一个synchronize，
2. 但是接口组合调用依然是不安全的，需要注意，这种类似情况可以追溯到ConcurrentHashMap等一些集合的调用。


## 接口组合调用-线程安全问题

ConcurrentHashMap只能保证单次操作的Atomicity，如
`
T1: concurrent_map.insert(key1,val1)           
T2: concurrent_map.contains(key1)
`
T1 和T2 的操作不会导致race condition.但如果T1 和T2 组合调用是
```java
if (! concurrent_map.contains(key1) {
    concurrent_map.insert(key1,val1)
}
```
即使 `contains` 和 `insert` 他们本身是thread-safe的，由于interleaving会发生在`contains`和`insert`中间，你没办法保证这两个操作组合起来的atomicity. 这是用目前几乎所有的thread-safe entity 的一大问题之一：小的thread-safe entity 如果要组合成大的thread-safe entity, 小entity的抽象便没办法维持；比如说，在你这个例子里，当你单独使用contains 或者 insert 的时候，不需要给 key1上锁，因为这个已经抽象在concurrent_map 里面了，但当你想把这两个操作合起来使用并继续保持atomicity，就必须要给key1 上锁，就破坏了concurrent_map 的抽象。有兴趣的话可以看 此书（http://bt.nitk.ac.in/c/15a/co403/notes/SLoCA-TM.pdf）的1.1.2小节，专门讲了这个composition 问题。

参考：
http://a123159521.iteye.com/blog/1245116
https://www.jianshu.com/p/a20052ac48f1



## HashMap
1. HashMap底层就是一个数组结构，数组中的每一项又是一个链表.
```java
public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        // Find a power of 2 >= initialCapacity
        int capacity = 1;
        while (capacity < initialCapacity)
            capacity <<= 1;

        this.loadFactor = loadFactor;
        threshold = (int)Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
        table = new Entry[capacity];
        useAltHashing = sun.misc.VM.isBooted() &&
                (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
        init();
}
```
我们着重看一下第18行代码`table = new Entry[capacity];`。这不就是Java中数组的创建方式吗？也就是说在构造函数中，其创建了一个Entry的数组，其大小为capacity（目前我们还不需要太了解该变量含义），那么Entry又是什么结构呢？看一下源码：
```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    final int hash;
    ……
}
//jdk1.8
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;    //用来定位数组索引位置
        final K key;
        V value;
        Node<K,V> next;   //链表的下一个node
        Node(int hash, K key, V value, Node<K,V> next) { ... }
        public final K getKey(){ ... }
        public final V getValue() { ... }
        public final String toString() { ... }
        public final int hashCode() { ... }
        public final V setValue(V newValue) { ... }
        public final boolean equals(Object o) { ... }
}
```

2. 扩容机制threshold和loadFactor，hashcode等
    int threshold;             // 所能容纳的key-value对极限 
    final float loadFactor;    // 负载因子

首先,Node[] table的初始化长度length(默认值是16),Load factor为负载因子(默认值是0.75),threshold是HashMap所能容纳的最大数据量的Node(键值对)个数。threshold = length * Load factor。也就是说,在数组定义好长度之后,负载因子越大,所能容纳的键值对个数越多。

结合负载因子的定义公式可知,threshold就是在此Load factor和length(数组长度)对应下允许的最大元素数目,超过这个数目就重新resize(扩容),扩容后的HashMap容量是之前容量的两倍。

3. 确定哈希桶数组索引位置
```java
//方法一:
static final int hash(Object key) {   //jdk1.8 & jdk1.7
     int h;
     // h = key.hashCode() 为第一步 取hashCode值
     // h ^ (h >>> 16)  为第二步 高位参与运算
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
//在JDK1.8的实现中,优化了高位运算的算法,通过hashCode()的高16位异或低16位实现的:(h = k.hashCode()) ^ (h >>> 16),主要是从速度、功效、质量来考虑的,这么做可以在数组table的length比较小的时候,也能保证考虑到高低Bit都参与到Hash的计算中,同时不会有太大的开销。
```java
//方法二:
static int indexFor(int h, int length) {  //jdk1.7的源码,jdk1.8没有这个方法,但是实现原理一样的
     return h & (length-1);  //第三步 取模运算
}
```
它通过h & (table.length -1)来得到该对象的保存位,而HashMap底层数组的长度总是2的n次方,这是HashMap在速度上的优化。当length总是2的n次方时,h& (length-1)运算等价于对length取模,也就是h%length,但是&比%具有更高的效率。


4. 扩容方法
    newTable[i]的引用赋给了e.next,也就是使用了单链表的头插入方式,同一位置上新元素总会被放在链表的头部位置；这样先放在一个索引上的元素终会被放到Entry链的尾部(如果发生了hash冲突的话),这一点和Jdk1.8有区别,下文详解。在旧数组中同一条Entry链上的元素,通过重新计算索引位置后,有可能被放到了新数组的不同位置上。
    扩容技巧：我们使用的是2次幂的扩展(指长度扩为原来2倍),所以,元素的位置要么是在原位置,要么是在原位置再移动2次幂的位置
```java
void resize(int newCapacity) {   //传入新的容量
    Entry[] oldTable = table;    //引用扩容前的Entry数组
    int oldCapacity = oldTable.length;         
    if (oldCapacity == MAXIMUM_CAPACITY) {  //扩容前的数组大小如果已经达到最大(2^30)了
        threshold = Integer.MAX_VALUE; //修改阈值为int的最大值(2^31-1),这样以后就不会扩容了
        return;
    }
 
    Entry[] newTable = new Entry[newCapacity];  //初始化一个新的Entry数组
    transfer(newTable);                         //!!将数据转移到新的Entry数组里
    table = newTable;                           //HashMap的table属性引用新的Entry数组
    threshold = (int)(newCapacity * loadFactor);//修改阈值
}

void transfer(Entry[] newTable) {
    Entry[] src = table;                   //src引用了旧的Entry数组
    int newCapacity = newTable.length;
    for (int j = 0; j < src.length; j++) { //遍历旧的Entry数组
        Entry<K,V> e = src[j];             //取得旧Entry数组的每个元素
        if (e != null) {
            src[j] = null;//释放旧Entry数组的对象引用(for循环后,旧的Entry数组不再引用任何对象)
            do {
                Entry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity); //!!重新计算每个元素在数组中的位置
                e.next = newTable[i]; //标记[1]
                newTable[i] = e;      //将元素放在数组上
                e = next;             //访问下一个Entry链上的元素
            } while (e != null);
        }
    }
}
```

5. 冲突解决方法: **链地址，开放地址(线性探测、二次探测、再哈希法)**
6. put操作和红黑树性能优化
```java
/**
 * Associates the specified value with the specified key in this map.
 * If the map previously contained a mapping for the key, the old
 * value is replaced.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
 *         (A <tt>null</tt> return can also indicate that the map
 *         previously associated <tt>null</tt> with <tt>key</tt>.)
 */
public V put(K key, V value) {
    // 对key的hashCode()做hash
    return putVal(hash(key), key, value, false, true);
}
/**
 * Implements Map.put and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode.
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 步骤①:tab为空则创建
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 步骤②:计算index,并对null做处理
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 步骤③:节点key存在,直接覆盖value
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 步骤④:判断该链为红黑树
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 步骤⑤:该链为链表
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //链表长度大于8转换为红黑树进行处理
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // key已经存在直接覆盖value
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 步骤⑥:超过最大容量 就扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
!()[http://atecher.net/2017/09/21/Java-Java8-%E9%87%8D%E6%96%B0%E8%AE%A4%E8%AF%86HashMap/hashMap_put%E6%96%B9%E6%B3%95%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B%E5%9B%BE.png]
6. fail-fast通过modeCount体现
    fail-fast 机制是java集合(Collection)中的一种错误机制。 当多个线程对同一个集合的内容进行操作时，就可能会产生 fail-fast 事件。
    例如：当某一个线程A通过 iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出 ConcurrentModificationException异常，产生 fail-fast 事件。

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
在迭代过程中，判断modCount跟expectedModCount是否相等，如果不相等就表示已经有其他线程修改了Map：

注意到modCount声明为volatile，保证线程之间修改的可见性。
```java
final Entry<K,V> nextEntry() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
```
在HashMap的API中指出：

由所有HashMap类的“collection 视图方法”所返回的迭代器都是快速失败的：在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器本身的 remove 方法，其他任何时间任何方式的修改，迭代器都将抛出 ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不冒在将来不确定的时间发生任意不确定行为的风险。

注意，迭代器的快速失败行为不能得到保证，一般来说，存在非同步的并发修改时，不可能作出任何坚决的保证。快速失败迭代器尽最大努力抛出ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是错误的，正确做法是：<font color="#ff0000">迭代器的快速失败行为应该仅用于检测程序错误。


7. 参考：
    http://atecher.net/2017/09/21/Java-Java8-%E9%87%8D%E6%96%B0%E8%AE%A4%E8%AF%86HashMap/
    https://github.com/atecher/atecher.github.io/blob/blog-source/source/_posts/Java-Java8-%E9%87%8D%E6%96%B0%E8%AE%A4%E8%AF%86HashMap.md

8. 遍历
    HashMap的两种遍历方式
    第一种
```java
　　Map map = new HashMap();
　　Iterator iter = map.entrySet().iterator();
　　while (iter.hasNext()) {
    　　Map.Entry entry = (Map.Entry) iter.next();
    　　Object key = entry.getKey();
    　　Object val = entry.getValue();
　　}
```
效率高,以后一定要使用此种方式！
第二种
```java
　　Map map = new HashMap();
　　Iterator iter = map.keySet().iterator();
　　while (iter.hasNext()) {
    　　Object key = iter.next();
    　　Object val = map.get(key);Object key = iter.next();
    　　Object val = map.get(key);
　　}
```
效率低,以后尽量少使用！

## LinkedHashMap
1. 排序模式accessOrder默认插入顺序，构造函数设置为true后，变成访问顺序。new LinkedHashMap<String, String>(16,0.75f,true);
2. 继承与HashMap(`public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>`)、底层使用哈希表与双向链表来保存所有元素。其基本操作与父类HashMap相似，它通过重写父类相关的方法，来实现自己的链接列表特性。
3. LinkedHashMap采用的hash算法和HashMap相同，但是它重新定义了数组中保存的元素Entry，该Entry除了保存当前对象的引用外，还保存了其上一个元素before和下一个元素after的引用，从而在哈希表的基础上又构成了**双向链接**列表。
```java
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
* LinkedHashMap的Entry元素。
* 继承HashMap的Entry元素，又保存了其上一个元素before和下一个元素after的引用。
 */
private static class Entry<K,V> extends HashMap.Entry<K,V> {
    Entry<K,V> before, after;
    ……
}
```
LinkedHashMap中的Entry集成与HashMap的Entry，但是其增加了before和after的引用，指的是上一个元素和下一个元素的引用。

4. removeEldestEntry(Map.Entry<K,V> eldest)方法。该方法可以提供在每次添加新条目时移除最旧条目的实现程序，默认返回false，这样，此映射的行为将类似于正常映射，即永远不能移除最旧的元素

5. put/get操作。
    **HashMap.put**:
```java
public V put(K key, V value) {
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
}
```

重写方法：
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

**读取get**
LinkedHashMap重写了父类HashMap的get方法，实际在调用父类getEntry()方法取得查找的元素后，再判断当排序模式accessOrder为true时，记录访问顺序，将最新访问的元素添加到双向链表的表头，并从原来的位置删除。由于的链表的增加、删除操作是常量级的，故并不会带来性能的损失。
```java
public V get(Object key) {
    // 调用父类HashMap的getEntry()方法，取得要查找的元素。
    Entry<K,V> e = (Entry<K,V>)getEntry(key);
    if (e == null)
        return null;
    // 记录访问顺序。
    e.recordAccess(this);
    return e.value;
}

void recordAccess(HashMap<K,V> m) {
    LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
    // 如果定义了LinkedHashMap的迭代顺序为访问顺序，
    // 则删除以前位置上的元素，并将最新访问的元素添加到链表表头。  
    if (lm.accessOrder) {
        lm.modCount++;
        remove();
        addBefore(lm.header);
    }
}


/**
* Removes this entry from the linked list.
*/
private void remove() {
    before.after = after;
    after.before = before;
}

/**clear链表，设置header为初始状态*/
public void clear() {
 super.clear();
 header.before = header.after = header;
}
```

6. cache的两种实现
    lrucache->linkedHashMap(false) 插入顺序排序
    fifocache->linkedHashMap(true) 访问顺序排序

其中有两种比较简单的方法，如下：
```
public class LRU<K,V> extends LinkedHashMap<K, V> implements Map<K, V>{

    private static final long serialVersionUID = 1L;
    private static final int maxSize = 0;

    public LRU(int maxSize) {
        this.maxSize = maxSize;
        super(16, 0.75, true);  // lru
        super(16, 0.75, false);  // fifo
    }

    /** 
     * @description 重写LinkedHashMap中的removeEldestEntry方法，当LRU中元素多余maxSize个时，
     *              删除最不经常使用的元素
     * @param eldest
     * @return     
     * @see java.util.LinkedHashMap#removeEldestEntry(java.util.Map.Entry)     
     */  
    @Override
    protected boolean removeEldestEntry(java.util.Map.Entry<K, V> eldest) {
        if(size() > maxSize){
            return true;
        }
        return false;
    }
}
```

## HashTable
HashTable和HashMap采用相同的存储机制，二者的实现基本一致，不同的是：

（1）HashMap是非线程安全的，HashTable是线程安全的，内部的方法基本都经过synchronized修饰。

（2）因为同步、哈希性能等原因，性能肯定是HashMap更佳，因此HashTable已被淘汰。

（3） HashMap允许有null值的存在，而在HashTable中put进的键值只要有一个null，直接抛出NullPointerException。

（4）HashMap默认初始化数组的大小为16，HashTable为11。前者扩容时乘2，使用位运算取得哈希，效率高于取模。而后者为乘2加1，都是素数和奇数，这样取模哈希结果更均匀。

（5）HashMap可以允许插入null key和null value； HashTable和ConcurrentHashMap都不可以插入null key和null value


## ConcurrentHashMap
1. 参考：
    https://www.cnblogs.com/study-everyday/p/6430462.html#autoid-2-2-3
    https://blog.csdn.net/m0_37135421/article/details/80551884#t11
    http://www.jasongj.com/java/concurrenthashmap/
    https://www.cnblogs.com/huaizuo/p/5413069.html
2. ConcurrentHashMap的结构中包含的Segment的数组，在默认的并发级别会创建包含16个Segment对象的数组。通过我们上面的知识，我们知道每个Segment又包含若干个散列表的桶，每个桶是由HashEntry链接起来的一个链表。如果key能够均匀散列，每个Segment大约守护整个散列表桶总数的1/16。
    `private static final int DEFAULT_CONCURRENCY_LEVEL = 16;`
3. jdk1.7, ConcurrentHashMap的数据结构是由一个Segment数组和多个HashEntry组成, Segment数组的意义就是将一个大的table分割成多个小的table来进行加锁，也就是上面的提到的锁分离技术，而每一个Segment元素存储的是HashEntry数组+链表，这个和HashMap的数据存储结构一样。
    put操作
    对于ConcurrentHashMap的数据插入，这里要进行两次Hash去定位数据的存储位置
    `static class Segment<K,V> extends ReentrantLock implements Serializable {`

从上Segment的继承体系可以看出，Segment实现了ReentrantLock,也就带有锁的功能，当执行put操作时，会进行第一次key的hash来定位Segment的位置，如果该Segment还没有初始化，即通过CAS操作进行赋值，然后进行第二次hash操作，找到相应的HashEntry的位置，这里会利用继承过来的锁的特性，在将数据插入指定的HashEntry位置时（链表的尾端），会通过继承ReentrantLock的tryLock（）方法尝试去获取锁，如果获取成功就直接插入相应的位置，如果已经有线程获取该Segment的锁，那当前线程会以自旋的方式去继续的调用tryLock（）方法去获取锁，超过指定次数就挂起，等待唤醒。
它使用了自旋锁，如果tryLock获取锁失败，说明锁被其它线程占用，此时通过循环再次以tryLock的方式申请锁。如果在循环过程中该Key所对应的链表头被修改，则重置retry次数。如果retry次数超过一定值，则使用lock方法申请锁。

get操作
ConcurrentHashMap的get操作跟HashMap类似，只是ConcurrentHashMap第一次需要经过一次hash定位到Segment的位置，然后再hash定位到指定的HashEntry，遍历该HashEntry下的链表进行对比，成功就返回，不成功就返回null

size操作
计算ConcurrentHashMap的元素大小是一个有趣的问题，因为他是并发操作的，就是在你计算size的时候，他还在并发的插入数据，可能会导致你计算出来的size和你实际的size有相差（在你return size的时候，插入了多个数据），要解决这个问题，JDK1.7版本用两种方案
第一种方案他会使用不加锁的模式去尝试多次计算ConcurrentHashMap的size，最多三次，比较前后两次计算的结果，结果一致就认为当前没有元素加入，计算的结果是准确的
第二种方案是如果第一种方案不符合，他就会给每个Segment加上锁，然后计算ConcurrentHashMap的size返回。
```java
try {
    for (;;) {
        if (retries++ == RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j) ensureSegment(j).lock(); // force creation
        }
        sum = 0L;
        size = 0;
        overflow = false;
        for (int j = 0; j < segments.length; ++j) {
            Segment<K,V> seg = segmentAt(segments, j);
            if (seg != null) { sum += seg.modCount; int c = seg.count; if (c < 0 || (size += c) < 0)
               overflow = true;
            } }
        if (sum == last) break;
        last = sum; } }
finally {
    if (retries > RETRIES_BEFORE_LOCK) {
        for (int j = 0; j < segments.length; ++j)
            segmentAt(segments, j).unlock();
    }
}
```
6. JDK1.8的实现已经摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来实现，并发控制使用Synchronized和CAS来操作，整个看起来就像是优化过且线程安全的HashMap，虽然在JDK1.8中还能看到Segment的数据结构，但是已经简化了属性，只是为了兼容旧版本

put操作

如果没有初始化就先调用initTable（）方法来进行初始化过程
如果没有hash冲突就直接CAS插入
如果还在进行扩容操作就先进行扩容
如果存在hash冲突，就加锁来保证线程安全，这里有两种情况，一种是链表形式就直接遍历到尾端插入，一种是红黑树就按照红黑树结构插入，
最后一个如果该链表的数量大于阈值8，就要先转换成黑红树的结构，break再一次进入循环
如果添加成功就调用addCount（）方法统计size，并且检查是否需要扩容

ConcurrentHashMap的get操作的流程很简单，也很清晰，可以分为三个步骤来描述

计算hash值，定位到该table索引位置，如果是首节点符合就返回
如果遇到扩容的时候，会调用标志正在扩容节点ForwardingNode的find方法，查找该节点，匹配就返回
以上都不符合的话，就往下遍历节点，匹配就返回，否则最后就返回null

size操作
```java
public int size() {
    long n = sumCount();
    return ((n < 0L) ? 0 :
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
            (int)n);
}
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a; //变化的数量
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
7. 通过对同一个 Volatile 变量的写 / 读访问，协调不同线程间读 / 写操作的内存可见性。



## TreeMap

## ConcurrentSkipListMap

到目前为止，我们在Java世界里看到了两种实现key-value的数据结构：Hash、TreeMap，这两种数据结构各自都有着优缺点。

1. Hash表：插入、查找最快，为O(1)；如使用链表实现则可实现无锁；数据有序化需要显式的排序操作。
2. 红黑树：插入、查找为O(logn)，但常数项较小；无锁实现的复杂性很高，一般需要加锁；数据天然有序。

然而，这次介绍第三种实现key-value的数据结构：SkipList。SkipList有着不低于红黑树的效率，但是其原理和实现的复杂度要比红黑树简单多了。

### SkipList

什么是SkipList？Skip List ，称之为跳表，它是一种可以替代平衡树的数据结构，其数据元素默认按照key值升序，天然有序。Skip list让已排序的数据分布在多层链表中，以0-1随机数决定一个数据的向上攀升与否，通过“空间来换取时间”的一个算法，在每个节点中增加了向前的指针，在插入、删除、查找时可以忽略一些不可能涉及到的结点，从而提高了效率。

我们先看一个简单的链表，如下：

[![1499585893174201707090001](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/1499585893174201707090001_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/1499585893174201707090001.png)

如果我们需要查询9、21、30，则需要比较次数为3 + 6 + 8 = 17 次，那么有没有优化方案呢？有！我们将该链表中的某些元素提炼出来作为一个比较“索引”，如下：

[![1499586063109201707090002](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/1499586063109201707090002_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/1499586063109201707090002.png)

我们先与这些索引进行比较来决定下一个元素是往右还是下走，由于存在“索引”的缘故，导致在检索的时候会大大减少比较的次数。当然元素不是很多，很难体现出优势，当元素足够多的时候，这种索引结构就会大显身手。

### SkipList的特性

SkipList具备如下特性：

1. 由很多层结构组成，level是通过一定的概率随机产生的
2. 每一层都是一个有序的链表，默认是升序，也可以根据创建映射时所提供的Comparator进行排序，具体取决于使用的构造方法
3. 最底层(Level 1)的链表包含所有元素
4. 如果一个元素出现在Level i 的链表中，则它在Level i 之下的链表也都会出现
5. 每个节点包含两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素

我们将上图再做一些扩展就可以变成一个典型的SkipList结构了

[![1499590828559201707090003](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/1499590828559201707090003_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/1499590828559201707090003.png)

### SkipList的查找

SkipListd的查找算法较为简单，对于上面我们我们要查找元素21，其过程如下：

1. 比较3，大于，往后找（9），
2. 比9大，继续往后找（25），但是比25小，则从9的下一层开始找（16）
3. 16的后面节点依然为25，则继续从16的下一层找
4. 找到21

如图

[![201707090004](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201707090004_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201707090004.png)

红色虚线代表路径。

### SkipList的插入

SkipList的插入操作主要包括：

1. 查找合适的位置。这里需要明确一点就是在确认新节点要占据的层次K时，采用丢硬币的方式，完全随机。如果占据的层次K大于链表的层次，则重新申请新的层，否则插入指定层次
2. 申请新的节点
3. 调整指针

假定我们要插入的元素为23，经过查找可以确认她是位于25后，9、16、21前。当然需要考虑申请的层次K。

**如果层次K > 3**

需要申请新层次（Level 4）

[![201707090005](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201707090005_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201707090005.png)

**如果层次 K = 2**

直接在Level 2 层插入即可

[![201707090006](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201707090006_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201707090006.png)

这里会涉及到以个算法：通过丢硬币决定层次K，该算法我们通过后面ConcurrentSkipListMap源码来分析。还有一个需要注意的地方就是，在K层插入元素后，需要确保所有小于K层的层次都应该出现新节点。

### SkipList的删除

删除节点和插入节点思路基本一致：找到节点，删除节点，调整指针。

比如删除节点9，如下：

[![201707090007](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201707090007_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201707090007.png)

### ConcurrentSkipListMap

通过上面我们知道SkipList采用空间换时间的算法，其插入和查找的效率O(logn)，其效率不低于红黑树，但是其原理和实现的复杂度要比红黑树简单多了。一般来说会操作链表List，就会对SkipList毫无压力。

ConcurrentSkipListMap其内部采用SkipLis数据结构实现。为了实现SkipList，ConcurrentSkipListMap提供了三个内部类来构建这样的链表结构：Node、Index、HeadIndex。其中Node表示最底层的单链表有序节点、Index表示为基于Node的索引层，HeadIndex用来维护索引层次。到这里我们可以这样说ConcurrentSkipListMap是通过HeadIndex维护索引层次，通过Index从最上层开始往下层查找，一步一步缩小查询范围，最后到达最底层Node时，就只需要比较很小一部分数据了。在JDK中的关系如下图：

[![201707090008](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201707090008_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201707090008.png)

** Node **

```
    static final class Node<K,V> {
        final K key;
        volatile Object value;
        volatile ConcurrentSkipListMap.Node<K, V> next;
        
        /** 省略些许代码 */
    }
```

Node的结构和一般的单链表毫无区别，key-value和一个指向下一个节点的next。

**Index**

```
    static class Index<K,V> {
        final ConcurrentSkipListMap.Node<K,V> node;
        final ConcurrentSkipListMap.Index<K,V> down;
        volatile ConcurrentSkipListMap.Index<K,V> right;

        /** 省略些许代码 */
    }
```

Index提供了一个基于Node节点的索引Node，一个指向下一个Index的right，一个指向下层的down节点。

**HeadIndex**

```
    static final class HeadIndex<K,V> extends Index<K,V> {
        final int level;  //索引层，从1开始，Node单链表层为0
        HeadIndex(Node<K,V> node, Index<K,V> down, Index<K,V> right, int level) {
            super(node, down, right);
            this.level = level;
        }
    }
```

HeadIndex内部就一个level来定义层级。

ConcurrentSkipListMap提供了四个构造函数，每个构造函数都会调用initialize()方法进行初始化工作。

```
    final void initialize() {
        keySet = null;
        entrySet = null;
        values = null;
        descendingMap = null;
        randomSeed = seedGenerator.nextInt() | 0x0100; // ensure nonzero
        head = new ConcurrentSkipListMap.HeadIndex<K,V>(new ConcurrentSkipListMap.Node<K,V>(null, BASE_HEADER, null),
                null, null, 1);
    }
```

注意，initialize()方法不仅仅只在构造函数中被调用，如clone，clear、readObject时都会调用该方法进行初始化步骤。这里需要注意randomSeed的初始化。

```
 private transient int randomSeed;
 randomSeed = seedGenerator.nextInt() | 0x0100; // ensure nonzero
```

randomSeed一个简单的随机数生成器（在后面介绍）。

### put操作

CoucurrentSkipListMap提供了put()方法用于将指定值与此映射中的指定键关联。源码如下：

```
    public V put(K key, V value) {
        if (value == null)
            throw new NullPointerException();
        return doPut(key, value, false);
    }
```

首先判断value如果为null，则抛出NullPointerException，否则调用doPut方法，其实如果各位看过JDK的源码的话，应该对这样的操作很熟悉了，JDK源码里面很多方法都是先做一些必要性的验证后，然后通过调用do**()方法进行真正的操作。

doPut()方法内容较多，我们分步分析。

```
    private V doPut(K key, V value, boolean onlyIfAbsent) {
        Node<K,V> z;             // added node
        if (key == null)
            throw new NullPointerException();
        // 比较器
        Comparator<? super K> cmp = comparator;
        outer: for (;;) {
            for (Node<K, V> b = findPredecessor(key, cmp), n = b.next; ; ) {

            /** 省略代码 */
```

doPut()方法有三个参数，除了key,value外还有一个boolean类型的onlyIfAbsent，该参数作用与如果存在当前key时，该做何动作。当onlyIfAbsent为false时，替换value，为true时，则返回该value。用代码解释为：

```
   if (!map.containsKey(key))
      return map.put(key, value);
  else
       return map.get(key);
```

首先判断key是否为null，如果为null，则抛出NullPointerException，从这里我们可以确认ConcurrentSkipList是不支持key或者value为null的。然后调用findPredecessor()方法，传入key来确认位置。findPredecessor()方法其实就是确认key要插入的位置。

```
   private Node<K,V> findPredecessor(Object key, Comparator<? super K> cmp) {
        if (key == null)
            throw new NullPointerException(); // don't postpone errors
        for (;;) {
            // 从head节点开始，head是level最高级别的headIndex
            for (Index<K,V> q = head, r = q.right, d;;) {

                // r != null，表示该节点右边还有节点，需要比较
                if (r != null) {
                    Node<K,V> n = r.node;
                    K k = n.key;
                    // value == null，表示该节点已经被删除了
                    // 通过unlink()方法过滤掉该节点
                    if (n.value == null) {
                        //删掉r节点
                        if (!q.unlink(r))
                            break;           // restart
                        r = q.right;         // reread r
                        continue;
                    }

                    // value != null，节点存在
                    // 如果key 大于r节点的key 则往前进一步
                    if (cpr(cmp, key, k) > 0) {
                        q = r;
                        r = r.right;
                        continue;
                    }
                }

                // 到达最右边，如果dowm == null，表示指针已经达到最下层了，直接返回该节点
                if ((d = q.down) == null)
                    return q.node;
                q = d;
                r = d.right;
            }
        }
    }
```

findPredecessor()方法意思非常明确：寻找前辈。从最高层的headIndex开始向右一步一步比较，直到right为null或者右边节点的Node的key大于当前key为止，然后再向下寻找，依次重复该过程，直到down为null为止，即找到了前辈，看返回的结果注意是Node，不是Item，所以插入的位置应该是最底层的Node链表。

在这个过程中ConcurrentSkipListMap赋予了该方法一个其他的功能，就是通过判断节点的value是否为null，如果为null，表示该节点已经被删除了，通过调用unlink()方法删除该节点。

```
        final boolean unlink(Index<K,V> succ) {
            return node.value != null && casRight(succ, succ.right);
        }
```

删除节点过程非常简单，更改下right指针即可。

通过findPredecessor()找到前辈节点后，做什么呢？看下面：

```
 for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
        // 前辈节点的next != null
        if (n != null) {
            Object v; int c;
            Node<K,V> f = n.next;

            // 不一致读，主要原因是并发，有节点捷足先登
            if (n != b.next)               // inconsistent read
                break;

            // n.value == null，该节点已经被删除了
            if ((v = n.value) == null) {   // n is deleted
                n.helpDelete(b, f);
                break;
            }

            // 前辈节点b已经被删除
            if (b.value == null || v == n) // b is deleted
                break;

            // 节点大于，往前移
            if ((c = cpr(cmp, key, n.key)) > 0) {
                b = n;
                n = f;
                continue;
            }

            // c == 0 表示，找到一个key相等的节点，根据onlyIfAbsent参数来做判断
            // onlyIfAbsent ==false，则通过casValue，替换value
            // onlyIfAbsent == true，返回该value
            if (c == 0) {
                if (onlyIfAbsent || n.casValue(v, value)) {
                    @SuppressWarnings("unchecked") V vv = (V)v;
                    return vv;
                }
                break; // restart if lost race to replace value
            }
            // else c < 0; fall through
        }

        // 将key-value包装成一个node，插入
        z = new Node<K,V>(key, value, n);
        if (!b.casNext(n, z))
            break;         // restart if lost race to append to b
        break outer;
    }
```

找到合适的位置后，就是在该位置插入节点咯。插入节点的过程比较简单，就是将key-value包装成一个Node，然后通过casNext()方法加入到链表当中。当然是插入之前需要进行一系列的校验工作。

在最下层插入节点后，下一步工作是什么？新建索引。前面博主提过，在插入节点的时候，会根据采用抛硬币的方式来决定新节点所插入的层次，由于存在并发的可能，ConcurrentSkipListMap采用ThreadLocalRandom来生成随机数。如下：

```
int rnd = ThreadLocalRandom.nextSecondarySeed();
```

抛硬币决定层次的思想很简单，就是通过抛硬币如果硬币为正面则层次level + 1 ，否则停止，如下：

```
            // 抛硬币决定层次
            while (((rnd >>>= 1) & 1) != 0)
                ++level;
```

在阐述SkipList插入节点的时候说明了，决定的层次level会分为两种情况进行处理，一是如果层次level大于最大的层次话则需要新增一层，否则就在相应层次以及小于该level的层次进行节点新增处理。

**level <= headIndex.level**

```
            // 如果决定的层次level比最高层次head.level小，直接生成最高层次的index
            // 由于需要确认每一层次的down，所以需要从最下层依次往上生成
            if (level <= (max = h.level)) {
                for (int i = 1; i <= level; ++i)
                    idx = new ConcurrentSkipListMap.Index<K,V>(z, idx, null);
            }
```

 

从底层开始，小于level的每一层都初始化一个index，每次的node都指向新加入的node，down指向下一层的item，右侧next全部为null。整个处理过程非常简单：为小于level的每一层初始化一个index，然后加入到原来的index链条中去。

**level > headIndex.level**

```
           // leve > head.level 则新增一层
            else { // try to grow by one level
                // 新增一层
                level = max + 1;

                // 初始化 level个item节点
                @SuppressWarnings("unchecked")
                ConcurrentSkipListMap.Index<K,V>[] idxs =
                        (ConcurrentSkipListMap.Index<K,V>[])new ConcurrentSkipListMap.Index<?,?>[level+1];
                for (int i = 1; i <= level; ++i)
                    idxs[i] = idx = new ConcurrentSkipListMap.Index<K,V>(z, idx, null);

                //
                for (;;) {
                    h = head;
                    int oldLevel = h.level;
                    // 层次扩大了，需要重新开始（有新线程节点加入）
                    if (level <= oldLevel) // lost race to add level
                        break;
                    // 新的头结点HeadIndex
                    ConcurrentSkipListMap.HeadIndex<K,V> newh = h;
                    ConcurrentSkipListMap.Node<K,V> oldbase = h.node;
                    // 生成新的HeadIndex节点，该HeadIndex指向新增层次
                    for (int j = oldLevel+1; j <= level; ++j)
                        newh = new ConcurrentSkipListMap.HeadIndex<K,V>(oldbase, newh, idxs[j], j);

                    // HeadIndex CAS替换
                    if (casHead(h, newh)) {
                        h = newh;
                        idx = idxs[level = oldLevel];
                        break;
                    }
                }
```

当抛硬币决定的level大于最大层次level时，需要新增一层进行处理。处理逻辑如下：

1. 初始化一个对应的index数组，大小为level + 1，然后为每个单位都创建一个index，个中参数为：Node为新增的Z，down为下一层index，right为null
2. 通过for循环来进行扩容操作。从最高层进行处理，新增一个HeadIndex，个中参数：节点Node，down都为最高层的Node和HeadIndex，right为刚刚创建的对应层次的index，level为相对应的层次level。最后通过CAS把当前的head与新加入层的head进行替换。

通过上面步骤我们发现，尽管已经找到了前辈节点，也将node插入了，也确定确定了层次并生成了相应的Index，但是并没有将这些Index插入到相应的层次当中，所以下面的代码就是将index插入到相对应的层当中。

```
  // 从插入的层次level开始
    splice: for (int insertionLevel = level;;) {
        int j = h.level;
        //  从headIndex开始
        for (ConcurrentSkipListMap.Index<K,V> q = h, r = q.right, t = idx;;) {
            if (q == null || t == null)
                break splice;

            // r != null；这里是找到相应层次的插入节点位置，注意这里只横向找
            if (r != null) {
                ConcurrentSkipListMap.Node<K,V> n = r.node;

                int c = cpr(cmp, key, n.key);

                // n.value == null ，解除关系，r右移
                if (n.value == null) {
                    if (!q.unlink(r))
                        break;
                    r = q.right;
                    continue;
                }

                // key > n.key 右移
                if (c > 0) {
                    q = r;
                    r = r.right;
                    continue;
                }
            }

            // 上面找到节点要插入的位置，这里就插入
            // 当前层是最顶层
            if (j == insertionLevel) {
                // 建立联系
                if (!q.link(r, t))
                    break; // restart
                if (t.node.value == null) {
                    findNode(key);
                    break splice;
                }
                // 标志的插入层 -- ，如果== 0 ，表示已经到底了，插入完毕，退出循环
                if (--insertionLevel == 0)
                    break splice;
            }

            // 上面节点已经插入完毕了，插入下一个节点
            if (--j >= insertionLevel && j < level)
                t = t.down;
            q = q.down;
            r = q.right;
        }
    }
```

这段代码分为两部分看，一部分是找到相应层次的该节点插入的位置，第二部分在该位置插入，然后下移。

至此，ConcurrentSkipListMap的put操作到此就结束了。代码量有点儿多，这里总结下：

1. 首先通过findPredecessor()方法找到前辈节点Node
2. 根据返回的前辈节点以及key-value，新建Node节点，同时通过CAS设置next
3. 设置节点Node，再设置索引节点。采取抛硬币方式决定层次，如果所决定的层次大于现存的最大层次，则新增一层，然后新建一个Item链表。
4. 最后，将新建的Item链表插入到SkipList结构中。

### get操作

相比于put操作 ，get操作会简单很多，其过程其实就只相当于put操作的第一步：

```
  private V doGet(Object key) {
        if (key == null)
            throw new NullPointerException();
        Comparator<? super K> cmp = comparator;
        outer: for (;;) {
            for (ConcurrentSkipListMap.Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
                Object v; int c;
                if (n == null)
                    break outer;
                ConcurrentSkipListMap.Node<K,V> f = n.next;
                if (n != b.next)                // inconsistent read
                    break;
                if ((v = n.value) == null) {    // n is deleted
                    n.helpDelete(b, f);
                    break;
                }
                if (b.value == null || v == n)  // b is deleted
                    break;
                if ((c = cpr(cmp, key, n.key)) == 0) {
                    @SuppressWarnings("unchecked") V vv = (V)v;
                    return vv;
                }
                if (c < 0)
                    break outer;
                b = n;
                n = f;
            }
        }
        return null;
    }
```

与put操作第一步相似，首先调用findPredecessor()方法找到前辈节点，然后顺着right一直往右找即可，同时在这个过程中同样承担了一个删除value为null的节点的职责。

### remove操作

remove操作为删除指定key节点，如下：

```
    public V remove(Object key) {
        return doRemove(key, null);
    }
```

直接调用doRemove()方法，这里remove有两个参数，一个是key，另外一个是value，所以doRemove方法即提供remove key，也提供同时满足key-value。

```
 final V doRemove(Object key, Object value) {
        if (key == null)
            throw new NullPointerException();
        Comparator<? super K> cmp = comparator;
        outer: for (;;) {
            for (ConcurrentSkipListMap.Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
                Object v; int c;
                if (n == null)
                    break outer;
                ConcurrentSkipListMap.Node<K,V> f = n.next;

                // 不一致读，重新开始
                if (n != b.next)                    // inconsistent read
                    break;

                // n节点已删除
                if ((v = n.value) == null) {        // n is deleted
                    n.helpDelete(b, f);
                    break;
                }

                // b节点已删除
                if (b.value == null || v == n)      // b is deleted
                    break;

                if ((c = cpr(cmp, key, n.key)) < 0)
                    break outer;

                // 右移
                if (c > 0) {
                    b = n;
                    n = f;
                    continue;
                }

                /*
                 * 找到节点
                 */

                // value != null 表示需要同时校验key-value值
                if (value != null && !value.equals(v))
                    break outer;

                // CAS替换value
                if (!n.casValue(v, null))
                    break;
                if (!n.appendMarker(f) || !b.casNext(n, f))
                    findNode(key);                  // retry via findNode
                else {
                    // 清理节点
                    findPredecessor(key, cmp);      // clean index

                    // head.right == null表示该层已经没有节点，删掉该层
                    if (head.right == null)
                        tryReduceLevel();
                }
                @SuppressWarnings("unchecked") V vv = (V)v;
                return vv;
            }
        }
        return null;
    }
```

调用findPredecessor()方法找到前辈节点，然后通过右移，然后比较，找到后利用CAS把value替换为null，然后判断该节点是不是这层唯一的index，如果是的话，调用tryReduceLevel()方法把这层干掉，完成删除。

其实从这里可以看出，remove方法仅仅是把Node的value设置null，并没有真正删除该节点Node，其实从上面的put操作、get操作我们可以看出，他们在寻找节点的时候都会判断节点的value是否为null，如果为null，则调用unLink()方法取消关联关系，如下：

```
                    if (n.value == null) {
                        if (!q.unlink(r))
                            break;           // restart
                        r = q.right;         // reread r
                        continue;
                    }
```

### size操作

ConcurrentSkipListMap的size()操作和ConcurrentHashMap不同，它并没有维护一个全局变量来统计元素的个数，所以每次调用该方法的时候都需要去遍历。

```
    public int size() {
        long count = 0;
        for (Node<K,V> n = findFirst(); n != null; n = n.next) {
            if (n.getValidValue() != null)
                ++count;
        }
        return (count >= Integer.MAX_VALUE) ? Integer.MAX_VALUE : (int) count;
    }
```

调用findFirst()方法找到第一个Node，然后利用node的next去统计。最后返回统计数据，最多能返回Integer.MAX_VALUE。注意这里在线程并发下是安全的。

ConcurrentSkipListMap过程其实不复杂，相比于ConcurrentHashMap而言，是简单的不能再简单了。对跳表SkipList熟悉的话，ConcurrentSkipListMap 应该是盘中餐了。



我们知道线程Thread可以调用setPriority(int newPriority)来设置优先级的，线程优先级高的线程先执行，优先级低的后执行。而前面介绍的ArrayBlockingQueue、LinkedBlockingQueue都是采用FIFO原则来确定线程执行的先后顺序，那么有没有一个队列可以支持优先级呢？ PriorityBlockingQueue 。

PriorityBlockingQueue是一个支持优先级的无界阻塞队列。默认情况下元素采用自然顺序升序排序，当然我们也可以通过构造函数来指定Comparator来对元素进行排序。需要注意的是PriorityBlockingQueue不能保证同优先级元素的顺序。



## PriorityBlockingQueue

### 二叉堆

由于PriorityBlockingQueue底层采用二叉堆来实现的，所以有必要先介绍下二叉堆。

二叉堆是一种特殊的堆，就结构性而言就是完全二叉树或者是近似完全二叉树，满足树结构性和堆序性。树机构特性就是完全二叉树应该有的结构，堆序性则是：父节点的键值总是保持固定的序关系于任何一个子节点的键值，且每个节点的左子树和右子树都是一个二叉堆。它有两种表现形式：最大堆、最小堆。

最大堆：父节点的键值总是大于或等于任何一个子节点的键值（下右图）

最小堆：父节点的键值总是小于或等于任何一个子节点的键值（下走图）

[![201703270001](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270001_thumb.jpg)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270001.jpg)

二叉堆一般用数组表示，如果父节点的节点位置在n处，那么其左孩子节点为：2 * n + 1 ，其右孩子节点为2 * (n + 1)，其父节点为（n - 1） / 2 处。上左图的数组表现形式为：

[![201703270002_2](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270002_2_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270002_2.png)

二叉堆的基本结构了解了，下面来看看二叉堆的添加和删除节点。二叉堆的添加和删除相对于二叉树来说会简单很多。

### 添加元素

首先将要添加的元素N插添加到堆的末尾位置（在二叉堆中我们称之为空穴）。如果元素N放入空穴中而不破坏堆的序（其值大于跟父节点值（最大堆是小于父节点）），那么插入完成。否则，我们则将该元素N的节点与其父节点进行交换，然后与其新父节点进行比较直到它的父节点不在比它小（最大堆是大）或者到达根节点。

假如有如下一个二叉堆

[![201703270003_3](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270003_3_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270003_3.png)

这是一个最小堆，其父节点总是小于等于任一一个子节点。现在我们添加一个元素2。

第一步：在末尾添加一个元素2，如下：

第二步：元素2比其父节点6小，进行替换，如下：

[![201703270005_3](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270005_3_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270005_3.png)
第三步：继续与其父节点5比较，小于，替换：

[![201703270006_3](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270006_3_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270006_3.png)

第四步：继续比较其跟节点1，发现跟节点比自己小，则完成，到这里元素2插入完毕。所以整个添加元素过程可以概括为：在元素末尾插入元素，然后不断比较替换直到不能移动为止。

复杂度：Ο(logn)

### 删除元素

删除元素与增加元素一样，需要维护整个二叉堆的序。删除位置1的元素（数组下标0），则把最后一个元素空出来移到最前边，然后和它的两个子节点比较，如果两个子节点中较小的节点小于该节点，就将他们交换，知道两个子节点都比该元素大为止。

就上面二叉堆而言，删除的元素为元素1。

第一步：删掉元素1，元素6空出来，如下：

第二步：与其两个子节点（元素2、元素3）比较，都小，将其中较小的元素（元素2）放入到该空穴中：

[![201703270008](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270008_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270008.png)
第三步：继续比较两个子节点（元素5、元素7），还是都小，则将较小的元素（元素5）放入到该空穴中：[![201703270004_4](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270004_4_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270004_4.png)[![201703270007_4](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270007_4_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270007_4.png)

第四步：比较其子节点（元素8），比该节点小，则元素6放入该空穴位置不会影响二叉堆的树结构，放入：

[![201703270010_3](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270010_3_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270010_3.png)
到这里整个删除操作就已经完成了。

二叉堆的添加、删除操作还是比较简单的，很容易就理解了。下面我们就参考该内容来开启PriorityBlockingQueue的源代码研究。

### PriorityBlockingQueue

PriorityBlockingQueue继承AbstractQueue，实现BlockingQueue接口。

```
public class PriorityBlockingQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable 
```

定义了一些属性

```
// 默认容量
    private static final int DEFAULT_INITIAL_CAPACITY = 11;

    // 最大容量
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    // 二叉堆数组
    private transient Object[] queue;

    // 队列元素的个数
    private transient int size;

    // 比较器，如果为空，则为自然顺序
    private transient Comparator<? super E> comparator;

    // 内部锁
    private final ReentrantLock lock;

    private final Condition notEmpty;
    
    // 
    private transient volatile int allocationSpinLock;

    // 优先队列：主要用于序列化，这是为了兼容之前的版本。只有在序列化和反序列化才非空
    private PriorityQueue<E> q;
```

内部仍然采用可重入锁ReentrantLock来实现同步机制，但是这里只有一个notEmpty的Condition，了解了ArrayBlockingQueue我们知道它定义了两个Condition，之类为何只有一个呢？原因就在于PriorityBlockingQueue是一个无界队列，插入总是会成功，除非消耗尽了资源导致服务器挂。

### 入列

PriorityBlockingQueue提供put()、add()、offer()方法向队列中加入元素。我们这里从put()入手：put(E e) ：将指定元素插入此优先级队列。

```
    public void put(E e) {
        offer(e); // never need to block
    }
```

PriorityBlockingQueue是无界的，所以不可能会阻塞。内部调用offer(E e)：

```
public boolean offer(E e) {
        // 不能为null
        if (e == null)
            throw new NullPointerException();
        // 获取锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        int n, cap;
        Object[] array;
        // 扩容
        while ((n = size) >= (cap = (array = queue).length))
            tryGrow(array, cap);
        try {
            Comparator<? super E> cmp = comparator;
            // 根据比较器是否为null，做不同的处理
            if (cmp == null)
                siftUpComparable(n, e, array);
            else
                siftUpUsingComparator(n, e, array, cmp);
            size = n + 1;
            // 唤醒正在等待的消费者线程
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
        return true;
    }
```

**siftUpComparable**
当比较器comparator为null时，采用自然排序，调用siftUpComparable方法：

```
private static <T> void siftUpComparable(int k, T x, Object[] array) {
        Comparable<? super T> key = (Comparable<? super T>) x;
        // “上冒”过程
        while (k > 0) {
            // 父级节点 （n - ） / 2
            int parent = (k - 1) >>> 1;
            Object e = array[parent];
            
            // key >= parent 完成（最大堆）
            if (key.compareTo((T) e) >= 0)
                break;
            // key < parant 替换
            array[k] = e;
            k = parent;
        }
        array[k] = key;
    }
```

这段代码所表示的意思：将元素X插入到数组中，然后进行调整以保持二叉堆的特性。

**siftUpUsingComparator**
当比较器不为null时，采用所指定的比较器，调用siftUpUsingComparator方法：

```
private static <T> void siftUpUsingComparator(int k, T x, Object[] array,
                                       Comparator<? super T> cmp) {
        while (k > 0) {
            int parent = (k - 1) >>> 1;
            Object e = array[parent];
            if (cmp.compare(x, (T) e) >= 0)
                break;
            array[k] = e;
            k = parent;
        }
        array[k] = x;
    }
```

**扩容：tryGrow**

```
    private void tryGrow(Object[] array, int oldCap) {
        lock.unlock();      // 扩容操作使用自旋，不需要锁主锁，释放
        Object[] newArray = null;
        // CAS 占用
        if (allocationSpinLock == 0 && UNSAFE.compareAndSwapInt(this, allocationSpinLockOffset, 0, 1)) {
            try {

                // 新容量  最小翻倍
                int newCap = oldCap + ((oldCap < 64) ? (oldCap + 2) :  (oldCap >> 1));

                // 超过
                if (newCap - MAX_ARRAY_SIZE > 0) {    // possible overflow
                    int minCap = oldCap + 1;
                    if (minCap < 0 || minCap > MAX_ARRAY_SIZE)
                        throw new OutOfMemoryError();
                    newCap = MAX_ARRAY_SIZE;        // 最大容量
                }
                if (newCap > oldCap && queue == array)
                    newArray = new Object[newCap];
            } finally {
                allocationSpinLock = 0;     // 扩容后allocationSpinLock = 0 代表释放了自旋锁
            }
        }
        // 到这里如果是本线程扩容newArray肯定是不为null，为null就是其他线程在处理扩容，那就让给别的线程处理
        if (newArray == null)
            Thread.yield();
        // 主锁获取锁
        lock.lock();
        // 数组复制
        if (newArray != null && queue == array) {
            queue = newArray;
            System.arraycopy(array, 0, newArray, 0, oldCap);
        }
    }
```

整个添加元素的过程和上面二叉堆一模一样：先将元素添加到数组末尾，然后采用“上冒”的方式将该元素尽量往上冒。

### 出列

PriorityBlockingQueue提供poll()、remove()方法来执行出对操作。出对的永远都是第一个元素：array[0]。

```
   public E poll() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
```

先获取锁，然后调用dequeue()方法：

```
   private E dequeue() {
        // 没有元素 返回null
        int n = size - 1;
        if (n < 0)
            return null;
        else {
            Object[] array = queue;
            // 出对元素
            E result = (E) array[0];
            // 最后一个元素（也就是插入到空穴中的元素）
            E x = (E) array[n];
            array[n] = null;
            // 根据比较器释放为null，来执行不同的处理
            Comparator<? super E> cmp = comparator;
            if (cmp == null)
                siftDownComparable(0, x, array, n);
            else
                siftDownUsingComparator(0, x, array, n, cmp);
            size = n;
            return result;
        }
    }
```

**siftDownComparable**

如果比较器为null，则调用siftDownComparable来进行自然排序处理：

```
  private static <T> void siftDownComparable(int k, T x, Object[] array,
                                               int n) {
        if (n > 0) {
            Comparable<? super T> key = (Comparable<? super T>)x;
            // 最后一个叶子节点的父节点位置
            int half = n >>> 1;
            while (k < half) {
                int child = (k << 1) + 1;       // 待调整位置左节点位置
                Object c = array[child];        //左节点
                int right = child + 1;          //右节点
                
                //左右节点比较，取较小的
                if (right < n &&
                        ((Comparable<? super T>) c).compareTo((T) array[right]) > 0)
                    c = array[child = right];
                
                //如果待调整key最小，那就退出，直接赋值
                if (key.compareTo((T) c) <= 0)
                    break;
                //如果key不是最小，那就取左右节点小的那个放到调整位置，然后小的那个节点位置开始再继续调整
                array[k] = c;
                k = child;
            }
            array[k] = key;
        }
    }
```

处理思路和二叉堆删除节点的逻辑一样：就第一个元素定义为空穴，然后把最后一个元素取出来，尝试插入到空穴位置，并与两个子节点值进行比较，如果不符合，则与其中较小的子节点进行替换，然后继续比较调整。

**siftDownUsingComparator**

如果指定了比较器，则采用比较器来进行调整：

```
    private static <T> void siftDownUsingComparator(int k, T x, Object[] array,
                                                    int n,
                                                    Comparator<? super T> cmp) {
        if (n > 0) {
            int half = n >>> 1;
            while (k < half) {
                int child = (k << 1) + 1;
                Object c = array[child];
                int right = child + 1;
                if (right < n && cmp.compare((T) c, (T) array[right]) > 0)
                    c = array[child = right];
                if (cmp.compare(x, (T) c) <= 0)
                    break;
                array[k] = c;
                k = child;
            }
            array[k] = x;
        }
    }
```

PriorityBlockingQueue采用二叉堆来维护，所以整个处理过程不是很复杂，添加操作则是不断“上冒”，而删除操作则是不断“下掉”。掌握二叉堆就掌握了PriorityBlockingQueue，无论怎么变还是不离其宗。对于PriorityBlockingQueue需要注意的是他是一个无界队列，所以添加操作是不会失败的，除非资源耗尽。
[![201703270009](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270009_thumb.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/07/201703270009.png)



前面的BlockingQueue都是单向的FIFO队列，而LinkedBlockingDeque则是一个由链表组成的双向阻塞队列，双向队列就意味着可以从对头、对尾两端插入和移除元素，同样意味着LinkedBlockingDeque支持FIFO、FILO两种操作方式。

LinkedBlockingDeque是可选容量的，在初始化时可以设置容量防止其过度膨胀，如果不设置，默认容量大小为Integer.MAX_VALUE。



DelayQueue是一个支持延时获取元素的无界阻塞队列。里面的元素全部都是“可延期”的元素，列头的元素是最先“到期”的元素，如果队列里面没有元素到期，是不能从列头获取元素的，哪怕有元素也不行。也就是说只有在延迟期到时才能够从队列中取元素。

DelayQueue主要用于两个方面：
\- 缓存：清掉缓存中超时的缓存数据
\- 任务超时处理

## DelayQueue

DelayQueue实现的关键主要有如下几个：

1. 可重入锁ReentrantLock
2. 用于阻塞和通知的Condition对象
3. 根据Delay时间排序的优先级队列：PriorityQueue
4. 用于优化阻塞通知的线程元素leader

ReentrantLock、Condition这两个对象就不需要阐述了，他是实现整个BlockingQueue的核心。PriorityQueue是一个支持优先级线程排序的队列（参考[【死磕Java并发】-----J.U.C之阻塞队列：PriorityBlockingQueue](http://cmsblogs.com/?p=2407)），leader后面阐述。这里我们先来了解Delay，他是实现延时操作的关键。

### Delayed

Delayed接口是用来标记那些应该在给定延迟时间之后执行的对象，它定义了一个long getDelay(TimeUnit unit)方法，该方法返回与此对象相关的的剩余时间。同时实现该接口的对象必须定义一个compareTo 方法，该方法提供与此接口的 getDelay 方法一致的排序。

```
public interface Delayed extends Comparable<Delayed> {
    long getDelay(TimeUnit unit);
}
```

如何使用该接口呢？上面说的非常清楚了，实现该接口的getDelay()方法，同时定义compareTo()方法即可。

### 内部结构

先看DelayQueue的定义：

```
    public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
            implements BlockingQueue<E> {
        /** 可重入锁 */
        private final transient ReentrantLock lock = new ReentrantLock();
        /** 支持优先级的BlockingQueue */
        private final PriorityQueue<E> q = new PriorityQueue<E>();
        /** 用于优化阻塞 */
        private Thread leader = null;
        /** Condition */
        private final Condition available = lock.newCondition();

        /**
         * 省略很多代码
         */
    }
```

看了DelayQueue的内部结构就对上面几个关键点一目了然了，但是这里有一点需要注意，DelayQueue的元素都必须继承Delayed接口。同时也可以从这里初步理清楚DelayQueue内部实现的机制了：以支持优先级无界队列的PriorityQueue作为一个容器，容器里面的元素都应该实现Delayed接口，在每次往优先级队列中添加元素时以元素的过期时间作为排序条件，最先过期的元素放在优先级最高。

### offer()

```
    public boolean offer(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            // 向 PriorityQueue中插入元素
            q.offer(e);
            // 如果当前元素的对首元素（优先级最高），leader设置为空，唤醒所有等待线程
            if (q.peek() == e) {
                leader = null;
                available.signal();
            }
            // 无界队列，永远返回true
            return true;
        } finally {
            lock.unlock();
        }
    }
```

offer(E e)就是往PriorityQueue中添加元素，具体可以参考（[【死磕Java并发】-----J.U.C之阻塞队列：PriorityBlockingQueue](http://cmsblogs.com/?p=2407)）。整个过程还是比较简单，但是在判断当前元素是否为对首元素，如果是的话则设置leader=null，这是非常关键的一个步骤，后面阐述。

### take()

```
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            for (;;) {
                // 对首元素
                E first = q.peek();
                // 对首为空，阻塞，等待off()操作唤醒
                if (first == null)
                    available.await();
                else {
                    // 获取对首元素的超时时间
                    long delay = first.getDelay(NANOSECONDS);
                    // <=0 表示已过期，出对，return
                    if (delay <= 0)
                        return q.poll();
                    first = null; // don't retain ref while waiting
                    // leader != null 证明有其他线程在操作，阻塞
                    if (leader != null)
                        available.await();
                    else {
                        // 否则将leader 设置为当前线程，独占
                        Thread thisThread = Thread.currentThread();
                        leader = thisThread;
                        try {
                            // 超时阻塞
                            available.awaitNanos(delay);
                        } finally {
                            // 释放leader
                            if (leader == thisThread)
                                leader = null;
                        }
                    }
                }
            }
        } finally {
            // 唤醒阻塞线程
            if (leader == null && q.peek() != null)
                available.signal();
            lock.unlock();
        }
    }
```

首先是获取对首元素，如果对首元素的延时时间 delay <= 0 ，则可以出对了，直接return即可。否则设置first = null，这里设置为null的主要目的是为了避免内存泄漏。如果 leader != null 则表示当前有线程占用，则阻塞，否则设置leader为当前线程，然后调用awaitNanos()方法超时等待。

**first = null**

这里为什么如果不设置first = null，则会引起内存泄漏呢？线程A到达，列首元素没有到期，设置leader = 线程A，这是线程B来了因为leader != null，则会阻塞，线程C一样。假如线程阻塞完毕了，获取列首元素成功，出列。这个时候列首元素应该会被回收掉，但是问题是它还被线程B、线程C持有着，所以不会回收，这里只有两个线程，如果有线程D、线程E...呢？这样会无限期的不能回收，就会造成内存泄漏。

这个入队、出对过程和其他的阻塞队列没有很大区别，无非是在出对的时候增加了一个到期时间的判断。同时通过leader来减少不必要阻塞。



## LinkedBlockingDeque

LinkedBlockingDeque 继承AbstractQueue，实现接口BlockingDeque，而BlockingDeque又继承接口BlockingQueue，BlockingDeque是支持两个附加操作的 Queue，这两个操作是：获取元素时等待双端队列变为非空；存储元素时等待双端队列中的空间变得可用。这两类操作就为LinkedBlockingDeque 的双向操作Queue提供了可能。BlockingDeque接口提供了一系列的以First和Last结尾的方法，如addFirst、addLast、peekFirst、peekLast。

```
public class LinkedBlockingDeque<E>
    extends AbstractQueue<E>
    implements BlockingDeque<E>, java.io.Serializable {

    // 双向链表的表头
    transient Node<E> first;

    // 双向链表的表尾
    transient Node<E> last;

    // 大小，双向链表中当前节点个数
    private transient int count;

    // 容量，在创建LinkedBlockingDeque时指定的
    private final int capacity;

    final ReentrantLock lock = new ReentrantLock();

    private final Condition notEmpty = lock.newCondition();

    private final Condition notFull = lock.newCondition();

}
```

通过上面的Lock可以看出，LinkedBlockingDeque底层实现机制与LinkedBlockingQueue一样，依然是通过互斥锁ReentrantLock 来实现，notEmpty 、notFull 两个Condition做协调生产者、消费者问题。

与其他BlockingQueue一样，节点还是使用内部类Node：

```
    static final class Node<E> {
        E item;

        Node<E> prev;

        Node<E> next;

        Node(E x) {
            item = x;
        }
    }
```

双向嘛，节点肯定得要有前驱prev、后继next咯。

### 基础方法

LinkedBlockingDeque 的add、put、offer、take、peek、poll系列方法都是通过调用XXXFirst，XXXLast方法。所以这里就仅以putFirst、putLast、pollFirst、pollLast分析下。

### **putFirst**

putFirst(E e) :将指定的元素插入此双端队列的开头，必要时将一直等待可用空间。

```
    public void putFirst(E e) throws InterruptedException {
        // check null
        if (e == null) throw new NullPointerException();
        Node<E> node = new Node<E>(e);
        // 获取锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            while (!linkFirst(node))
                // 在notFull条件上等待，直到被唤醒或中断
                notFull.await();
        } finally {
            // 释放锁
            lock.unlock();
        }
    }
```

先获取锁，然后调用linkFirst方法入列，最后释放锁。如果队列是满的则在notFull上面等待。linkFirst设置Node为对头：

```
    private boolean linkFirst(Node<E> node) {
        // 超出容量
        if (count >= capacity)
            return false;

        // 首节点
        Node<E> f = first;
        // 新节点的next指向原first
        node.next = f;
        // 设置node为新的first
        first = node;

        // 没有尾节点，设置node为尾节点
        if (last == null)
            last = node;
        // 有尾节点，那就将之前first的pre指向新增node
        else
            f.prev = node;
        ++count;
        // 唤醒notEmpty
        notEmpty.signal();
        return true;
    }
```

linkFirst主要是设置node节点队列的列头节点，成功返回true，如果队列满了返回false。整个过程还是比较简单的。

### **putLast**

putLast(E e) :将指定的元素插入此双端队列的末尾，必要时将一直等待可用空间。

```
    public void putLast(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        Node<E> node = new Node<E>(e);
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            while (!linkLast(node))
                notFull.await();
        } finally {
            lock.unlock();
        }
    }
```

调用linkLast将节点Node链接到队列尾部：

```
    private boolean linkLast(Node<E> node) {
        if (count >= capacity)
            return false;
        // 尾节点
        Node<E> l = last;

        // 将Node的前驱指向原本的last
        node.prev = l;

        // 将node设置为last
        last = node;
        // 首节点为null，则设置node为first
        if (first == null)
            first = node;
        else
        //非null，说明之前的last有值，就将之前的last的next指向node
            l.next = node;
        ++count;
        notEmpty.signal();
        return true;
    }
```

### **pollFirst**

pollFirst()：获取并移除此双端队列的第一个元素；如果此双端队列为空，则返回 null。

```
    public E pollFirst() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return unlinkFirst();
        } finally {
            lock.unlock();
        }
    }
```

调用unlinkFirst移除队列首元素：

```
    private E unlinkFirst() {
        // 首节点
        Node<E> f = first;

        // 空队列，直接返回null
        if (f == null)
            return null;

        // first.next
        Node<E> n = f.next;

        // 节点item
        E item = f.item;

        // 移除掉first ==> first = first.next
        f.item = null;
        f.next = f; // help GC
        first = n;

        // 移除后为空队列，仅有一个节点
        if (n == null)
            last = null;
        else
        // n的pre原来指向之前的first，现在n变为first了，pre指向null
            n.prev = null;
        --count;
        notFull.signal();
        return item;
    }
```

### **pollLast**

pollLast():获取并移除此双端队列的最后一个元素；如果此双端队列为空，则返回 null。

```
    public E pollLast() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return unlinkLast();
        } finally {
            lock.unlock();
        }
    }
```

调用unlinkLast移除尾结点，链表空返回null ：

```
    private E unlinkLast() {
        // assert lock.isHeldByCurrentThread();
        Node<E> l = last;
        if (l == null)
            return null;
        Node<E> p = l.prev;
        E item = l.item;
        l.item = null;
        l.prev = l; // help GC
        last = p;
        if (p == null)
            first = null;
        else
            p.next = null;
        --count;
        notFull.signal();
        return item;
    }
```

LinkedBlockingDeque大部分方法都是通过linkFirst、linkLast、unlinkFirst、unlinkLast这四个方法来实现的，因为是双向队列，所以他们都是针对first、last的操作，看懂这个整个LinkedBlockingDeque就不难了。

**掌握了双向队列的插入、删除操作，LinkedBlockingDeque就没有任何难度可言了，数据结构的重要性啊！！！！**



## **SynchronousQueue数据结构**

　　由于SynchronousQueue的支持公平策略和非公平策略，所以底层可能两种数据结构：队列（实现公平策略）和栈（实现非公平策略），队列与栈都是通过链表来实现的。具体的数据结构如下

![img](data:image/png;base64,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)　　说明：数据结构有两种类型，栈和队列；栈有一个头结点，队列有一个头结点和尾结点；栈用于实现非公平策略，队列用于实现公平策略。

### **SynchronousQueue源码分析**

　　3.1 类的继承关系　　

```
public class SynchronousQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable {}
```

　　说明：SynchronousQueue继承了AbstractQueue抽象类，AbstractQueue定义了对队列的基本操作；同时实现了BlockingQueue接口，BlockingQueue表示阻塞型的队列，其对队列的操作可能会抛出异常；同时也实现了Searializable接口，表示可以被序列化。

　　3.2 类的内部类

　　SynchronousQueue的内部类框架图如下

![img](data:image/png;base64,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)

　　说明：其中比较重要的类是左侧的三个类，Transferer是TransferStack栈和TransferQueue队列的公共类，定义了转移数据的公共操作，由TransferStack和TransferQueue具体实现，WaitQueue、LifoWaitQueue、FifoWaitQueue表示为了兼容JDK1.5版本中的SynchronousQueue的序列化策略所遗留的，这里不做具体的讲解。下面着重看左侧的三个类。

　　① Transferer　　

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    abstract static class Transferer<E> {
        /**
         * Performs a put or take.
         *
         * @param e if non-null, the item to be handed to a consumer;
         *          if null, requests that transfer return an item
         *          offered by producer.
         * @param timed if this operation should timeout
         * @param nanos the timeout, in nanoseconds
         * @return if non-null, the item provided or received; if null,
         *         the operation failed due to timeout or interrupt --
         *         the caller can distinguish which of these occurred
         *         by checking Thread.interrupted.
         */
        // 转移数据，put或者take操作
        abstract E transfer(E e, boolean timed, long nanos);
    }
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　说明：Transferer定义了transfer操作，用于take或者put数据。transfer方法由子类实现。

　　② TransfererStack

  　　1. 类的继承关系　　

```
static final class TransferStack<E> extends Transferer<E> {}
```

http://www.cnblogs.com/leesf456/p/5560362.html



作为BlockingQueue中的一员，SynchronousQueue与其他BlockingQueue有着不同特性：

1. SynchronousQueue没有容量。与其他BlockingQueue不同，SynchronousQueue是一个不存储元素的BlockingQueue。每一个put操作必须要等待一个take操作，否则不能继续添加元素，反之亦然。
2. 因为没有容量，所以对应 peek, contains, clear, isEmpty ... 等方法其实是无效的。例如clear是不执行任何操作的，contains始终返回false,peek始终返回null。
3. SynchronousQueue分为公平和非公平，默认情况下采用非公平性访问策略，当然也可以通过构造函数来设置为公平性访问策略（为true即可）。
4. 若使用 TransferQueue, 则队列中永远会存在一个 dummy node（这点后面详细阐述）。

SynchronousQueue非常适合做交换工作，生产者的线程和消费者的线程同步以传递某些信息、事件或者任务。

与其他BlockingQueue一样，SynchronousQueue同样继承AbstractQueue和实现BlockingQueue接口：

```
public class SynchronousQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable
```

SynchronousQueue提供了两个构造函数：

```
    public SynchronousQueue() {
        this(false);
    }

    public SynchronousQueue(boolean fair) {
        // 通过 fair 值来决定公平性和非公平性
        // 公平性使用TransferQueue，非公平性采用TransferStack
        transferer = fair ? new TransferQueue<E>() : new TransferStack<E>();
    }
```

TransferQueue、TransferStack继承Transferer，Transferer为SynchronousQueue的内部类，它提供了一个方法transfer()，该方法定义了转移数据的规范，如下：

```
    abstract static class Transferer<E> {
        abstract E transfer(E e, boolean timed, long nanos);
    }
```

transfer()方法主要用来完成转移数据的，如果e != null，相当于将一个数据交给消费者，如果e == null，则相当于从一个生产者接收一个消费者交出的数据。

SynchronousQueue采用队列TransferQueue来实现公平性策略，采用堆栈TransferStack来实现非公平性策略，他们两种都是通过链表实现的，其节点分别为QNode，SNode。TransferQueue和TransferStack在SynchronousQueue中扮演着非常重要的作用，SynchronousQueue的put、take操作都是委托这两个类来实现的。

### TransferQueue

TransferQueue是实现公平性策略的核心类，其节点为QNode，其定义如下：

```
    static final class TransferQueue<E> extends Transferer<E> {
        /** 头节点 */
        transient volatile QNode head;
        /** 尾节点 */
        transient volatile QNode tail;
        // 指向一个取消的结点
        //当一个节点中最后一个插入时，它被取消了但是可能还没有离开队列
        transient volatile QNode cleanMe;

        /**
         * 省略很多代码O(∩_∩)O
         */
    }
```

在TransferQueue中除了头、尾节点外还存在一个cleanMe节点。该节点主要用于标记，当删除的节点是尾节点时则需要使用该节点。

同时，对于TransferQueue需要注意的是，其队列永远都存在一个dummy node，在构造时创建：

```
        TransferQueue() {
            QNode h = new QNode(null, false); // initialize to dummy node.
            head = h;
            tail = h;
        }
```

在TransferQueue中定义了QNode类来表示队列中的节点，QNode节点定义如下：

```
    static final class QNode {
        // next 域
        volatile QNode next;
        // item数据项
        volatile Object item;
        //  等待线程，用于park/unpark
        volatile Thread waiter;       // to control park/unpark
        //模式，表示当前是数据还是请求，只有当匹配的模式相匹配时才会交换
        final boolean isData;

        QNode(Object item, boolean isData) {
            this.item = item;
            this.isData = isData;
        }

        /**
         * CAS next域，在TransferQueue中用于向next推进
         */
        boolean casNext(QNode cmp, QNode val) {
            return next == cmp &&
                    UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
        }

        /**
         * CAS itme数据项
         */
        boolean casItem(Object cmp, Object val) {
            return item == cmp &&
                    UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
        }

        /**
         *  取消本结点，将item域设置为自身
         */
        void tryCancel(Object cmp) {
            UNSAFE.compareAndSwapObject(this, itemOffset, cmp, this);
        }

        /**
         * 是否被取消
         * 与tryCancel相照应只需要判断item释放等于自身即可
         */
        boolean isCancelled() {
            return item == this;
        }


        boolean isOffList() {
            return next == this;
        }

        private static final sun.misc.Unsafe UNSAFE;
        private static final long itemOffset;
        private static final long nextOffset;

        static {
            try {
                UNSAFE = sun.misc.Unsafe.getUnsafe();
                Class<?> k = QNode.class;
                itemOffset = UNSAFE.objectFieldOffset
                        (k.getDeclaredField("item"));
                nextOffset = UNSAFE.objectFieldOffset
                        (k.getDeclaredField("next"));
            } catch (Exception e) {
                throw new Error(e);
            }
        }
    }
```

上面代码没啥好看的，需要注意的一点就是isData，该属性在进行数据交换起到关键性作用，两个线程进行数据交换的时候，必须要两者的模式保持一致。

### TransferStack

TransferStack用于实现非公平性，定义如下：

```
    static final class TransferStack<E> extends Transferer<E> {

        static final int REQUEST    = 0;

        static final int DATA       = 1;

        static final int FULFILLING = 2;

        volatile SNode head;

        /**
         * 省略一堆代码  O(∩_∩)O~
         */

    }
```

TransferStack中定义了三个状态：REQUEST表示消费数据的消费者，DATA表示生产数据的生产者，FULFILLING，表示匹配另一个生产者或消费者。任何线程对TransferStack的操作都属于上述3种状态中的一种（对应着SNode节点的mode）。同时还包含一个head域，表示头结点。

内部节点SNode定义如下：

```
    static final class SNode {
        // next 域
        volatile SNode next;
        // 相匹配的节点
        volatile SNode match;
        // 等待的线程
        volatile Thread waiter;
        // item 域
        Object item;                // data; or null for REQUESTs

        // 模型
        int mode;

        /**
         * item域和mode域不需要使用volatile修饰，因为它们在volatile/atomic操作之前写，之后读
         */

        SNode(Object item) {
            this.item = item;
        }

        boolean casNext(SNode cmp, SNode val) {
            return cmp == next &&
                    UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
        }

        /**
         * 将s结点与本结点进行匹配，匹配成功，则unpark等待线程
         */
        boolean tryMatch(SNode s) {
            if (match == null &&
                    UNSAFE.compareAndSwapObject(this, matchOffset, null, s)) {
                Thread w = waiter;
                if (w != null) {    // waiters need at most one unpark
                    waiter = null;
                    LockSupport.unpark(w);
                }
                return true;
            }
            return match == s;
        }

        void tryCancel() {
            UNSAFE.compareAndSwapObject(this, matchOffset, null, this);
        }

        boolean isCancelled() {
            return match == this;
        }

        // Unsafe mechanics
        private static final sun.misc.Unsafe UNSAFE;
        private static final long matchOffset;
        private static final long nextOffset;

        static {
            try {
                UNSAFE = sun.misc.Unsafe.getUnsafe();
                Class<?> k = SNode.class;
                matchOffset = UNSAFE.objectFieldOffset
                        (k.getDeclaredField("match"));
                nextOffset = UNSAFE.objectFieldOffset
                        (k.getDeclaredField("next"));
            } catch (Exception e) {
                throw new Error(e);
            }
        }
    }
```

上面简单介绍了TransferQueue、TransferStack，由于SynchronousQueue的put、take操作都是调用Transfer的transfer()方法，只不过是传递的参数不同而已，put传递的是e参数，所以模式为数据（公平isData = true，非公平mode= DATA），而take操作传递的是null，所以模式为请求（公平isData = false，非公平mode = REQUEST），如下：

```
    // put操作
    public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        if (transferer.transfer(e, false, 0) == null) {
            Thread.interrupted();
            throw new InterruptedException();
        }
    }

    // take操作
    public E take() throws InterruptedException {
        E e = transferer.transfer(null, false, 0);
        if (e != null)
            return e;
        Thread.interrupted();
        throw new InterruptedException();
    }
```

### 公平模式

公平性调用TransferQueue的transfer方法：

```
    E transfer(E e, boolean timed, long nanos) {
        QNode s = null;
        // 当前节点模式
        boolean isData = (e != null);

        for (;;) {
            QNode t = tail;
            QNode h = head;
            // 头、尾节点 为null，没有初始化
            if (t == null || h == null)
                continue;

            // 头尾节点相等（队列为null） 或者当前节点和队列节点模式一样
            if (h == t || t.isData == isData) {
                // tn = t.next
                QNode tn = t.next;
                // t != tail表示已有其他线程操作了，修改了tail，重新再来
                if (t != tail)
                    continue;
                // tn != null，表示已经有其他线程添加了节点，tn 推进，重新处理
                if (tn != null) {
                    // 当前线程帮忙推进尾节点，就是尝试将tn设置为尾节点
                    advanceTail(t, tn);
                    continue;
                }
                //  调用的方法的 wait 类型的, 并且 超时了, 直接返回 null
                // timed 在take操作阐述
                if (timed && nanos <= 0)
                    return null;

                // s == null，构建一个新节点Node
                if (s == null)
                    s = new QNode(e, isData);

                // 将新建的节点加入到队列中，如果不成功，继续处理
                if (!t.casNext(null, s))
                    continue;

                // 替换尾节点
                advanceTail(t, s);

                // 调用awaitFulfill, 若节点是 head.next, 则进行自旋
                // 若不是的话, 直接 block, 直到有其他线程 与之匹配, 或它自己进行线程的中断
                Object x = awaitFulfill(s, e, timed, nanos);

                // 若返回的x == s表示，当前线程已经超时或者中断，不然的话s == null或者是匹配的节点
                if (x == s) {
                    // 清理节点S
                    clean(t, s);
                    return null;
                }

                // isOffList：用于判断节点是否已经从队列中离开了
                if (!s.isOffList()) {
                    // 尝试将S节点设置为head，移出t
                    advanceHead(t, s);
                    if (x != null)
                        s.item = s;
                    // 释放线程 ref
                    s.waiter = null;
                }

                // 返回
                return (x != null) ? (E)x : e;

            }

            // 这里是从head.next开始，因为TransferQueue总是会存在一个dummy node节点
            else {
                // 节点
                QNode m = h.next;

                // 不一致读，重新开始
                // 有其他线程更改了线程结构
                if (t != tail || m == null || h != head)
                    continue;

                /**
                 * 生产者producer和消费者consumer匹配操作
                 */
                Object x = m.item;
                // isData == (x != null)：判断isData与x的模式是否相同，相同表示已经匹配了
                // x == m ：m节点被取消了
                // !m.casItem(x, e)：如果尝试将数据e设置到m上失败
                if (isData == (x != null) ||  x  == m || !m.casItem(x, e)) {
                    // 将m设置为头结点，h出列，然后重试
                    advanceHead(h, m);
                    continue;
                }

                // 成功匹配了，m设置为头结点h出列，向前推进
                advanceHead(h, m);
                // 唤醒m上的等待线程
                LockSupport.unpark(m.waiter);
                return (x != null) ? (E)x : e;
            }
        }
    }
```

整个transfer的算法如下：
\1. 如果队列为null或者尾节点模式与当前节点模式一致，则尝试将节点加入到等待队列中（采用自旋的方式），直到被匹配或、超时或者取消。匹配成功的话要么返回null（producer返回的）要么返回真正传递的值（consumer返回的），如果返回的是node节点本身则表示当前线程超时或者取消了。
\2. 如果队列不为null，且队列的节点是当前节点匹配的节点，则进行数据的传递匹配并返回匹配节点的数据
\3. 在整个过程中都会检测并帮助其他线程推进

当队列为空时，节点入列然后通过调用awaitFulfill()方法自旋，该方法主要用于自旋/阻塞节点，直到节点被匹配返回或者取消、中断。

```
    Object awaitFulfill(QNode s, E e, boolean timed, long nanos) {

        // 超时控制
        final long deadline = timed ? System.nanoTime() + nanos : 0L;
        Thread w = Thread.currentThread();
        // 自旋次数
        // 如果节点Node恰好是head节点，则自旋一段时间，这里主要是为了效率问题，如果里面阻塞，会存在唤醒、线程上下文切换的问题
        // 如果生产者、消费者者里面到来的话，就避免了这个阻塞的过程
        int spins = ((head.next == s) ?
                (timed ? maxTimedSpins : maxUntimedSpins) : 0);
        // 自旋
        for (;;) {
            // 线程中断了，剔除当前节点
            if (w.isInterrupted())
                s.tryCancel(e);

            // 如果线程进行了阻塞 -> 唤醒或者中断了，那么x != e 肯定成立，直接返回当前节点即可
            Object x = s.item;
            if (x != e)
                return x;
            // 超时判断
            if (timed) {
                nanos = deadline - System.nanoTime();
                // 如果超时了，取消节点,continue，在if(x != e)肯定会成立，直接返回x
                if (nanos <= 0L) {
                    s.tryCancel(e);
                    continue;
                }
            }

            // 自旋- 1
            if (spins > 0)
                --spins;

            // 等待线程
            else if (s.waiter == null)
                s.waiter = w;

            // 进行没有超时的 park
            else if (!timed)
                LockSupport.park(this);

            // 自旋次数过了, 直接 + timeout 方式 park
            else if (nanos > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanos);
        }
    }
```

在自旋/阻塞过程中做了一点优化，就是判断当前节点是否为对头元素，如果是的则先自旋，如果自旋次数过了，则才阻塞，这样做的主要目的就在如果生产者、消费者立马来匹配了则不需要阻塞，因为阻塞、唤醒会消耗资源。在整个自旋的过程中会不断判断是否超时或者中断了，如果中断或者超时了则调用tryCancel()取消该节点。

**tryCancel**

```
            void tryCancel(Object cmp) {
                UNSAFE.compareAndSwapObject(this, itemOffset, cmp, this);
            }
```

取消过程就是将节点的item设置为自身（itemOffset是item的偏移量）。所以在调用awaitFulfill()方法时，如果当前线程被取消、中断、超时了那么返回的值肯定时S，否则返回的则是匹配的节点。如果返回值是节点S，那么if(x == s)必定成立，如下：

```
                    Object x = awaitFulfill(s, e, timed, nanos);
                    if (x == s) {                   // wait was cancelled
                        clean(t, s);
                        return null;
                    }
```

如果返回的x == s成立，则调用clean()方法清理节点S：

```
    void clean(QNode pred, QNode s) {
        //
        s.waiter = null;

        while (pred.next == s) {
            QNode h = head;
            QNode hn = h.next;
            // hn节点被取消了，向前推进
            if (hn != null && hn.isCancelled()) {
                advanceHead(h, hn);
                continue;
            }

            // 队列为空，直接return null
            QNode t = tail;
            if (t == h)
                return;

            QNode tn = t.next;
            // 不一致，说明有其他线程改变了tail节点，重新开始
            if (t != tail)
                continue;

            // tn != null 推进tail节点，重新开始
            if (tn != null) {
                advanceTail(t, tn);
                continue;
            }

            // s 不是尾节点 移出
            if (s != t) {
                QNode sn = s.next;
                // 如果s已经被移除退出循环，否则尝试断开s
                if (sn == s || pred.casNext(s, sn))
                    return;
            }

            // s是尾节点，则有可能会有其他线程在添加新节点，则cleanMe出场
            QNode dp = cleanMe;
            // 如果dp不为null，说明是前一个被取消节点，将其移除
            if (dp != null) {
                QNode d = dp.next;
                QNode dn;
                if (d == null ||               // 节点d已经删除
                        d == dp ||                 // 原来的节点 cleanMe 已经通过 advanceHead 进行删除
                        !d.isCancelled() ||        // 原来的节点 s已经删除
                        (d != t &&                 // d 不是tail节点
                                (dn = d.next) != null &&  //
                                dn != d &&                //   that is on list
                                dp.casNext(d, dn)))       // d unspliced
                    // 清除 cleanMe 节点, 这里的 dp == pred 若成立, 说明清除节点s，成功, 直接return, 不然的话要再次循环
                    casCleanMe(dp, null);
                if (dp == pred)
                    return;
            } else if (casCleanMe(null, pred))  // 原来的 cleanMe 是 null, 则将 pred 标记为 cleamMe 为下次 清除 s 节点做标识
                return;
        }
    }
```

这个clean()方法感觉有点儿难度，我也看得不是很懂。这里是引用<http://www.jianshu.com/p/95cb570c8187>

1. 删除的节点不是queue尾节点, 这时 直接 pred.casNext(s, s.next) 方式来进行删除(和ConcurrentLikedQueue中差不多)
2. 删除的节点是队尾节点
   - 此时 cleanMe == null, 则 前继节点pred标记为 cleanMe, 为下次删除做准备
   - 此时 cleanMe != null, 先删除上次需要删除的节点, 然后将 cleanMe至null, 让后再将 pred 赋值给 cleanMe

### 非公平模式

非公平模式transfer方法如下：

```
    E transfer(E e, boolean timed, long nanos) {
        SNode s = null; // constructed/reused as needed
        int mode = (e == null) ? REQUEST : DATA;

        for (;;) {
            SNode h = head;
            // 栈为空或者当前节点模式与头节点模式一样，将节点压入栈内，等待匹配
            if (h == null || h.mode == mode) {
                // 超时
                if (timed && nanos <= 0) {
                    // 节点被取消了，向前推进
                    if (h != null && h.isCancelled())
                        //  重新设置头结点（弹出之前的头结点）
                        casHead(h, h.next);
                    else
                        return null;
                }
                // 不超时
                // 生成一个SNode节点，并尝试替换掉头节点head (head -> s)
                else if (casHead(h, s = snode(s, e, h, mode))) {
                    // 自旋，等待线程匹配
                    SNode m = awaitFulfill(s, timed, nanos);
                    // 返回的m == s 表示该节点被取消了或者超时、中断了
                    if (m == s) {
                        // 清理节点S，return null
                        clean(s);
                        return null;
                    }

                    // 因为通过前面一步将S替换成了head，如果h.next == s ，则表示有其他节点插入到S前面了,变成了head
                    // 且该节点就是与节点S匹配的节点
                    if ((h = head) != null && h.next == s)
                        // 将s.next节点设置为head，相当于取消节点h、s
                        casHead(h, s.next);

                    // 如果是请求则返回匹配的域，否则返回节点S的域
                    return (E) ((mode == REQUEST) ? m.item : s.item);
                }
            }

            // 如果栈不为null，且两者模式不匹配（h != null && h.mode != mode）
            // 说明他们是一队对等匹配的节点，尝试用当前节点s来满足h节点
            else if (!isFulfilling(h.mode)) {
                // head 节点已经取消了，向前推进
                if (h.isCancelled())
                    casHead(h, h.next);

                // 尝试将当前节点打上"正在匹配"的标记，并设置为head
                else if (casHead(h, s=snode(s, e, h, FULFILLING|mode))) {
                    // 循环loop
                    for (;;) {
                        // s为当前节点，m是s的next节点，
                        // m节点是s节点的匹配节点
                        SNode m = s.next;
                        // m == null，其他节点把m节点匹配走了
                        if (m == null) {
                            // 将s弹出
                            casHead(s, null);
                            // 将s置空，下轮循环的时候还会新建
                            s = null;
                            // 退出该循环，继续主循环
                            break;
                        }
                        // 获取m的next节点
                        SNode mn = m.next;
                        // 尝试匹配
                        if (m.tryMatch(s)) {
                            // 匹配成功，将s 、 m弹出
                            casHead(s, mn);     // pop both s and m
                            return (E) ((mode == REQUEST) ? m.item : s.item);
                        } else
                            // 如果没有匹配成功，说明有其他线程已经匹配了，把m移出
                            s.casNext(m, mn);
                    }
                }
            }
            // 到这最后一步说明节点正在匹配阶段
            else {
                // head 的next的节点，是正在匹配的节点，m 和 h配对
                SNode m = h.next;

                // m == null 其他线程把m节点抢走了，弹出h节点
                if (m == null)
                    casHead(h, null);
                else {
                    SNode mn = m.next;
                    if (m.tryMatch(h))
                        casHead(h, mn);
                    else 
                        h.casNext(m, mn);
                }
            }
        }
    }
```

整个处理过程分为三种情况，具体如下：
\1. 如果当前栈为空获取节点模式与栈顶模式一样，则尝试将节点加入栈内，同时通过自旋方式等待节点匹配，最后返回匹配的节点或者null（被取消）
\2. 如果栈不为空且节点的模式与首节点模式匹配，则尝试将该节点打上FULFILLING标记，然后加入栈中，与相应的节点匹配，成功后将这两个节点弹出栈并返回匹配节点的数据
\3. 如果有节点在匹配，那么帮助这个节点完成匹配和出栈操作，然后在主循环中继续执行

当节点加入栈内后，通过调用awaitFulfill()方法自旋等待节点匹配：

```
    SNode awaitFulfill(SNode s, boolean timed, long nanos) {
        // 超时
        final long deadline = timed ? System.nanoTime() + nanos : 0L;
        // 当前线程
        Thread w = Thread.currentThread();

        // 自旋次数
        // shouldSpin 用于检测当前节点是否需要自旋
        // 如果栈为空、该节点是首节点或者该节点是匹配节点，则先采用自旋，否则阻塞
        int spins = (shouldSpin(s) ?
                (timed ? maxTimedSpins : maxUntimedSpins) : 0);
        for (;;) {
            // 线程中断了，取消该节点
            if (w.isInterrupted())
                s.tryCancel();

            // 匹配节点
            SNode m = s.match;

            // 如果匹配节点m不为空，则表示匹配成功，直接返回
            if (m != null)
                return m;
            // 超时
            if (timed) {
                nanos = deadline - System.nanoTime();
                // 节点超时，取消
                if (nanos <= 0L) {
                    s.tryCancel();
                    continue;
                }
            }

            // 自旋;每次自旋的时候都需要检查自身是否满足自旋条件，满足就 - 1，否则为0
            if (spins > 0)
                spins = shouldSpin(s) ? (spins-1) : 0;

            // 第一次阻塞时，会将当前线程设置到s上
            else if (s.waiter == null)
                s.waiter = w;

            // 阻塞 当前线程
            else if (!timed)
                LockSupport.park(this);
            // 超时
            else if (nanos > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanos);
        }
    }
```

awaitFulfill()方法会一直自旋/阻塞直到匹配节点。在S节点阻塞之前会先调用shouldSpin()方法判断是否采用自旋方式，为的就是如果有生产者或者消费者马上到来，就不需要阻塞了，在多核条件下这种优化是有必要的。同时在调用park()阻塞之前会将当前线程设置到S节点的waiter上。匹配成功，返回匹配节点m。

shouldSpin()方法如下：

```
        boolean shouldSpin(SNode s) {
            SNode h = head;
            return (h == s || h == null || isFulfilling(h.mode));
        }
```

同时在阻塞过程中会一直检测当前线程是否中断了，如果中断了，则调用tryCancel()方法取消该节点，取消过程就是将当前节点的math设置为当前节点。所以如果线程中断了，那么在返回m时一定是S节点自身。

```
            void tryCancel() {
                UNSAFE.compareAndSwapObject(this, matchOffset, null, this);
            }
```

awaitFullfill()方法如果返回的m == s，则表示当前节点已经中断取消了，则需要调用clean()方法，清理节点S：

```
    void clean(SNode s) {

        // 清理item域
        s.item = null;
        // 清理waiter域
        s.waiter = null;

        // past节点
        SNode past = s.next;
        if (past != null && past.isCancelled())
            past = past.next;

        // 从栈顶head节点，取消从栈顶head到past节点之间所有已经取消的节点
        // 注意：这里如果遇到一个节点没有取消，则会退出while
        SNode p;
        while ((p = head) != null && p != past && p.isCancelled())
            casHead(p, p.next);     // 如果p节点已经取消了，则剔除该节点

        // 如果经历上面while p节点还没有取消，则再次循环取消掉所有p 到past之间的取消节点
        while (p != null && p != past) {
            SNode n = p.next;
            if (n != null && n.isCancelled())
                p.casNext(n, n.next);
            else
                p = n;
        }
    }
```

clean()方法就是将head节点到S节点之间所有已经取消的节点全部移出。【不清楚为何要用两个while，一个不行么】

## ArrayBlockingQueue 

ArrayBlockingQueue，一个由数组实现的有界阻塞队列。该队列采用FIFO的原则对元素进行排序添加的。

ArrayBlockingQueue为有界且固定，其大小在构造时由构造函数来决定，确认之后就不能再改变了。ArrayBlockingQueue支持对等待的生产者线程和使用者线程进行排序的可选公平策略，但是在默认情况下不保证线程公平的访问，在构造时可以选择公平策略（fair = true）。公平性通常会降低吞吐量，但是减少了可变性和避免了“不平衡性”。

### ArrayBlockingQueue

先看ArrayBlockingQueue的定义：

```
    public class ArrayBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, Serializable {
        private static final long serialVersionUID = -817911632652898426L;
        final Object[] items;
        int takeIndex;
        int putIndex;
        int count;
        // 重入锁
        final ReentrantLock lock;
        // notEmpty condition
        private final Condition notEmpty;
        // notFull condition
        private final Condition notFull;
        transient ArrayBlockingQueue.Itrs itrs;
    }
```

可以清楚地看到ArrayBlockingQueue继承AbstractQueue，实现BlockingQueue接口。看过java.util包源码的同学应该都认识AbstractQueue，改类在Queue接口中扮演着非常重要的作用，该类提供了对queue操作的骨干实现（具体内容移驾其源码）。BlockingQueue继承java.util.Queue为阻塞队列的核心接口，提供了在多线程环境下的出列、入列操作，作为使用者，则不需要关心队列在什么时候阻塞线程，什么时候唤醒线程，所有一切均由BlockingQueue来完成。

ArrayBlockingQueue内部使用可重入锁ReentrantLock + Condition来完成多线程环境的并发操作。

- items，一个定长数组，维护ArrayBlockingQueue的元素
- takeIndex，int，为ArrayBlockingQueue对首位置
- putIndex，int，ArrayBlockingQueue对尾位置
- count，元素个数
- lock，锁，ArrayBlockingQueue出列入列都必须获取该锁，两个步骤公用一个锁
- notEmpty，出列条件
- notFull，入列条件

### 入队

ArrayBlockingQueue提供了诸多方法，可以将元素加入队列尾部。

- add(E e) ：将指定的元素插入到此队列的尾部（如果立即可行且不会超过该队列的容量），在成功时返回 true，如果此队列已满，则抛出 IllegalStateException
- offer(E e) :将指定的元素插入到此队列的尾部（如果立即可行且不会超过该队列的容量），在成功时返回 true，如果此队列已满，则返回 false
- offer(E e, long timeout, TimeUnit unit) :将指定的元素插入此队列的尾部，如果该队列已满，则在到达指定的等待时间之前等待可用的空间
- put(E e) :将指定的元素插入此队列的尾部，如果该队列已满，则等待可用的空间

方法较多，我们就分析一个方法：add(E e)：

```
    public boolean add(E e) {
        return super.add(e);
    }
	
    public boolean add(E e) {
        if (offer(e))
            return true;
        else
            throw new IllegalStateException("Queue full");
    }
```

add方法调用offer(E e)，如果返回false，则直接抛出IllegalStateException异常。offer(E e)为ArrayBlockingQueue实现：

```
    public boolean offer(E e) {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            if (count == items.length)
                return false;
            else {
                enqueue(e);
                return true;
            }
        } finally {
            lock.unlock();
        }
    }
```

方法首先检查是否为null，然后获取lock锁。获取锁成功后，如果队列已满则直接返回false，否则调用enqueue(E e)，enqueue(E e)为入列的核心方法，所有入列的方法最终都将调用该方法在队列尾部插入元素：

```
    private void enqueue(E x) {
        // assert lock.getHoldCount() == 1;
        // assert items[putIndex] == null;
        final Object[] items = this.items;
        items[putIndex] = x;
        if (++putIndex == items.length)
            putIndex = 0;
        count++;
        notEmpty.signal();
    }
```

该方法就是在putIndex（对尾）为知处添加元素，最后使用notEmpty的signal()方法通知阻塞在出列的线程（如果队列为空，则进行出列操作是会阻塞）。

### 出队

ArrayBlockingQueue提供的出队方法如下：

- poll() :获取并移除此队列的头，如果此队列为空，则返回 null
- poll(long timeout, TimeUnit unit) :获取并移除此队列的头部，在指定的等待时间前等待可用的元素（如果有必要）
- remove(Object o) :从此队列中移除指定元素的单个实例（如果存在）
- take() :获取并移除此队列的头部，在元素变得可用之前一直等待（如果有必要）

**poll()**

```
    public E poll() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return (count == 0) ? null : dequeue();
        } finally {
            lock.unlock();
        }
    }
```

如果队列为空返回null，否则调用dequeue()获取列头元素：

```
 private E dequeue() {
        final Object[] items = this.items;
        E x = (E) items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
        notFull.signal();
        return x;
    }
```

该方法主要是从列头（takeIndex 位置）取出元素，同时如果迭代器itrs不为null，则需要维护下该迭代器。最后调用notFull.signal()唤醒入列线程。

**take()**

```
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
```

take()与poll()存在一个区别就是count == 0 时的处理，poll()直接返回null，而take()则是在notEmpty上面等待直到被入列的线程唤醒。



## ConcurrentLinkedQueue



ConcurrentLinkedQueue是一个基于链接节点的无界线程安全队列，它采用先进先出的规则对节点进行排序，当我们添加一个元素的时候，它会添加到队列的尾部，当我们获取一个元素时，它会返回队列头部的元素。它采用了“wait－free”算法来实现，该算法在Michael & Scott算法上进行了一些修改, Michael & Scott算法的详细信息可以参见[参考资料一](http://www.cs.rochester.edu/~scott/papers/1996_PODC_queues.pdf)。
上一篇文章介绍了JDK java.util.concurrent包下很重要的一个类：[ConcurrentHashMap](http://vickyqi.com/2015/10/26/JDK%E5%B9%B6%E5%8F%91%E5%B7%A5%E5%85%B7%E7%B1%BB%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94ConcurrentHashMap/)，今天来看下另一个重要的类——ConcurrentLinkedQueue。
在多线程编程环境下并发安全队列是不可或缺的一个重要工具类，为了实现并发安全可以有两种方式：一种是阻塞式的，例如：LinkedBlockingQueue；另一种即是我们将要探讨的非阻塞式，例如：ConcurrentLinkedQueue。相比较于阻塞式，非阻塞的最显著的优点就是性能，非阻塞式算法使用CAS来原子性的更新数据，避免了加锁的时间，同时也保证了数据的一致性。

#### **简单介绍**

ConcurrentLinkedQueue是一个基于链接节点的无界线程安全队列，它采用先进先出的规则对节点进行排序，当我们添加一个元素的时候，它会添加到队列的尾部，当我们获取一个元素时，它会返回队列头部的元素。它采用了“wait－free”算法来实现，该算法在Michael & Scott算法上进行了一些修改, Michael & Scott算法的详细信息可以参见[参考资料一](http://www.cs.rochester.edu/~scott/papers/1996_PODC_queues.pdf)。

#### **结构预览**

首先看看结构图：

**图1：ConcurrentLinkedQueue结构图：**
[![ConcurrentLinkedQueue结构图](http://7xnnj7.com1.z0.glb.clouddn.com/blogConcurrentLinkedQueue%E7%BB%93%E6%9E%84%E5%9B%BE.png)](http://7xnnj7.com1.z0.glb.clouddn.com/blogConcurrentLinkedQueue%E7%BB%93%E6%9E%84%E5%9B%BE.png)
从图中可以看到ConcurrentLinkedQueue中包含两个内部类：Node<E>和Itr。Node<E>用来表示ConcurrentLinkedQueue链表中的一个节点，通过Node<E>的next字段指向下一个节点，从而形成一个链表结构；Itr实现Iterator<E>接口，用来遍历ConcurrentLinkedQueue。ConcurrentLinkedQueue中的方法不多，其中最主要的两个方法是：offer(E)和poll()，分别实现队列的两个重要的操作：入队和出队。

| 方法             | 含义                         |
| ---------------- | ---------------------------- |
| offer(E)         | 插入一个元素到队列尾部       |
| poll()           | 从队列头部取出一个元素       |
| add(E)           | 同offer(E)                   |
| peek()           | 获取头部元素，但不删除       |
| isEmpty()        | 判断队列是否为空             |
| size()           | 获取队列长度(元素个数)       |
| contains(Object) | 判断队列是否包含指定元素     |
| remove(Object)   | 删除队列中指定元素           |
| toArray(T[])     | 将队列的元素复制到一个数组   |
| iterator()       | 返回一个可遍历该队列的迭代器 |

下面会着重分析offer(E)和poll()两个方法，同时会讲解remove(Object)和iterator()方法。

#### **常用方法解读**

##### **入队——offer**

首先看看入队操作，由于是无阻塞的队列，所以整个入队操作是在无锁模式下进行的，下面来分析下JDK到底是如何实现无锁并保证安全性的。

```
/**
 * Inserts the specified element at the tail of this queue.
 *
 * @return <tt>true</tt> (as specified by {@link Queue#offer})
 * @throws NullPointerException if the specified element is null
 */
public boolean offer(E e) {
    if (e == null) throw new NullPointerException();
    Node<E> n = new Node<E>(e, null);
    for (;;) {//①
        Node<E> t = tail;//②
        Node<E> s = t.getNext();//②
        if (t == tail) {//③
            if (s == null) {//④
                if (t.casNext(s, n)) {//⑥
                    casTail(t, n);//⑦
                    return true;
                }
            } else {
                casTail(t, s);//⑤
            }
        }
    }
}
```

代码不长，但是思路还是很巧妙的，下面我们逐句深入分析每一行代码。`if (e == null) throw new NullPointerException(); Node n = new Node(e, null);`检查NULL，避免NullPointerException，然后创建一个Node，该Node的item为传入的参数e，next为NULL。`for (;;) {}`接着是一个死循环，死循环保证该入队操作能够一直重试直至入队成功。`Node t = tail; Node s = t.getNext();`使用局部变量t引用tail节点，同时获取tail节点的next节点，赋予变量s。`if (t == tail) {}`只有在t==tail的情况下才会执行入队操作，否则进行下一轮循环，直到t==tail，因为是无锁模式，所以如果同时有多个线程在执行入队操作，那么在一个线程读取了tail之后，很可能会有其他线程已经修改了tail（**此处的修改是指将tail指向另一个节点，所以t还引用着原来的节点，导致t!=tail，而并非是修改了tail所指向的节点的值**），此处的判断避免了一开始的错误，但是并不能保证后续的执行过程中不会插入其他线程的操作，其实ConcurrentLinkedQueue的设计使得if内的代码即使在有其他线程插入的情况下依旧能够很好地执行，下面我们接着分析。

`if (s == null) {} else { casTail(t, s); }`这里判断s（tail的next是否为NULL），如果不为NULL，则直接将tail指向s。这里需要说明一下：由于tail指向的是队列的尾部，所以tail的next应该始终是NULL，那么当发生tail的next不为NULL，则说明当前队列处于不一致状态，这时当前线程需要帮助队列进入一致性状态，这就是ConcurrentLinkedQueue设计的巧妙之处！那么如果帮助队列进入一致性状态呢？这个问题我们先留着，继续看什么情况下会导致队列进入不一致状态！

```
if (t.casNext(s, n)) {
	casTail(t, n);
    return true;
}
```

这几句代码完成了入队的操作，第一步CAS的设置t（指向tail）的next为n（新创建的节点），该更新操作能够完成的前提是t的next值==s，即tail的next值在该线程首次读取期间并未发生变化。此处的CAS操作保证了tail的next值更新的原子性，所以不会出现不一致情况。当成功更新了tail的next节点之后，接下来就是原子性的更新tail为n，此处如果更新成功，则入队顺利完成完成，但是奇怪的是如果此处更新失败，入队依旧是成功的！为什么呢？看下文。

我们试想如果一个线程成功的原子性更新了tail的next值为新创建的节点，由于Node的next是volatile修饰的，所以会立即被之后的所有线程可见，那么就会出现tail未变化但是tail的next已经不是NULL了，此时就会出现上面提到的tail的next不为NULL的情况了，现在我们再来看看上面是如何处理这种情况的，`casTail(t, s);`，从这句可以看出当一个线程看到tail的next不为NULL时就会直接将tail更新成s（tail的next所指向的节点），即将tail指向其next节点，当然这里的更新也是CAS保证的原子性更新。为什么敢这么大胆，正是因为如果当前线程（T1）看到tail的next不为NULL，那么必然是有一个线程（T2）处于入队操作中，且成功执行了`t.casNext(s, n)`（将新创建的节点赋到tail的next上），正准备执行`casTail(t, n);`（将tail执行其next指向的节点），那么T1直接将T2准备做的工作完成，然后再进入循环重新进行入队操作，而T2也不在乎自己这一步是否顺利完成，反正只要有人完成了就行，所以T2就直接返回入队成功，最终T1帮助T2顺利完成了入队操作，并且全程无锁，此设计真的是巧妙啊~~~

下面我们使用流程图形象的描绘下入队过程，整个入队方法被划分成7步（见上面的代码中的注释）。说明：虽然入队是在无锁模式下进行，但是由于使用CAS进行原子性更新，所以很多地方其实还是实现了线程安全的，除了⑥->⑦，下面的图描绘的也正是⑥->⑦这一步可能出现的冲突情况。

**图2：ConcurrentLinkedQueue入队流程图：**
[![ConcurrentLinkedQueue入队流程图](http://7xnnj7.com1.z0.glb.clouddn.com/blog/ConcurrentLinkedQueue%E5%85%A5%E9%98%9F%E6%B5%81%E7%A8%8B%E5%9B%BE.png)](http://7xnnj7.com1.z0.glb.clouddn.com/blog/ConcurrentLinkedQueue%E5%85%A5%E9%98%9F%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

上面介绍了ConcurrentLinkedQueue是如何实现无锁入队的，但是我们只说明了多个线程同时入队操作是线程安全的，但是如果多个线程同时进行入队和出队，以及删除操作呢？这个问题在下面分析另外两个方法时会提到，同时最后也会进行一个总结，下面我们先看看删除操作是如何实现的。

##### **删除——remove**

先介绍删除，是因为出队操作有个地方需要在这里提前介绍下。

```
public boolean remove(Object o) {
    if (o == null) return false;// ①
    for (Node<E> p = first(); p != null; p = p.getNext()) {// ②
        E item = p.getItem();// ③
        if (item != null &&
            o.equals(item) &&
            p.casItem(item, null))// ④
            return true;
    }
    return false;
}
```

源码中的注释申明了remove方法会使用equals()判断两个节点的值与待删除的值是否相同，同时如果队列有多个与待删除值相同的节点则只删除最前面的一个节点。

同样remove()方法也是无锁模式，①判断是否为NULL，②从队列头部开始查找，③获取每个节点的item值，用于跟o进行equals比较，前面三步都很平常，重点在④，`if (item != null && o.equals(item) && p.casItem(item, null))`这里首先判断item不为NULL，然后判断item与o相等，前面两个都满足的话，那说明已经查找到一个节点的值与待删除的值一样，后面就是删除该节点，这里删除其实并非真的删除，而只是原子性的将节点的item值设置为NULL。从上面的分析可以看出ConcurrentLinkedQueue的删除只是将队列中的某个节点值置为NULL，由于Node的item是volatile的，所以不存在线程安全问题，同时由于remove并未修改队列的结构，所以多个线程同时进行remove，或者同其他方法一起进行也不会发生线程安全性问题。

##### **出队——poll**

出队从逻辑上来说就是从队列的头部往外取出数据并删除，下面看看ConcurrentLinkedQueue是如何实现无锁出队的。

```
public E poll() {
    for (;;) {// ①
        Node<E> h = head;// ②
        Node<E> t = tail;// ②
        Node<E> first = h.getNext();// ②
        if (h == head) {// ③
            if (h == t) {// ④
                if (first == null)// ⑤
                    return null;
                else
                    casTail(t, first);// ⑥
            } else if (casHead(h, first)) {// ⑦
                E item = first.getItem();// ⑧
                if (item != null) {// ⑨
                    first.setItem(null);// ⑩
                    return item;
                }
                // else skip over deleted item, continue loop,
            }
        }
    }
}
```

出队的步骤略多些，不过理解了也就很简单了。首先①是一个死循环；②的三步分别是获取head/tail/head.next三个节点；③判断h==head，避免操作过程中已有其他线程移动了head；④判断head是否等于tail，即队列是否为NULL，说到这里我们先来看看head和tail在队列中到底处于什么位置。我们用一个队列入队出队的时序图来描绘下在入队和出队过程中head和tail到底是如何变化的。

**图3：ConcurrentLinkedQueue队列时序图：**
[![ConcurrentLinkedQueue队列](http://7xnnj7.com1.z0.glb.clouddn.com/blog/ConcurrentLinkedQueue%E9%98%9F%E5%88%97%E6%97%B6%E5%BA%8F%E5%9B%BE.png)](http://7xnnj7.com1.z0.glb.clouddn.com/blog/ConcurrentLinkedQueue%E9%98%9F%E5%88%97%E6%97%B6%E5%BA%8F%E5%9B%BE.png)
从图中我们可以看出head的next指向的是队列的第一个元素，我们出队也是将head的next指向的元素出队，同时head==tail说明队列已经没有元素了。明白了这两点我们再接着④分析，如果④这里为真，说明队列已经为NULL，接着⑤判断f（head的next指向的节点）是否为NULL，不为NULL则执行⑥将tail指向f，到这里如果理解了上面入队操作，那么应该是可以理解这一步的用意的——帮助其他线程执行入队操作，跟入队时的⑤是一样的，因为head==tail，head的next不为NULL，则说明tail的next不为NULL，所以要将tail重新指向他的next，帮助正在执行入队的线程完成入队工作。理解了这一步那么出队操作就已经理解了一大半了，下面继续看⑦⑧⑨⑩。

如果head!=tail，则队列不为NULL，那么直接将head指向下一个节点，将当前节点踢出队列即可，当然需要CAS保证原子性更新，然后将踢出队列的节点的item取出返回，并置为NULL即完成了出队操作。这里需要注意的是如果被踢出队列的节点的item是NULL，说明该节点已经被删除了（因为remove()方法只是将节点的item设置为NULL，而不将节点踢出队列），那就只能再次循环了。再提一点，为什么⑦⑧⑨⑩能够被线程安全的执行，因为在⑦这一步是原子更新的，而且更新之后这个节点就立即不会被其他任何线程访问到了，所以后面⑧⑨⑩想怎么处理都是安全的。

到这里出队操作应该很清楚了，下面就来综合分析下为什么针对ConcurrentLinkedQueue的整个入队/出队/删除都是不需要锁的。

1. 上面已经分析了如果多个线程同时访问其中任一个方法（offer/poll/remove）都是无需加锁而且线程安全的
2. 由于remove方法不修改ConcurrentLinkedQueue的结构，所以跟其他两个方法都不会有冲突
3. 如果同时两个线程，一个入队，一个出队，在队列不为NULL的情况下是不是有任何问题的，因为一个操作tail，一个操作head，完全不相关。但是如果队列为NULL时还是会发生冲突的，因为tail==head。这里我们在分析出队时也提到了，如果出队线程发现tail的next不为NULL，那么就会感知到当前有一个线程在执行入队操作，所以出队线程就会帮助入队线程完成入队操作，而且每个操作都是通过CAS保证原子性更新，所以就算同时两个线程，一个入队，一个出队也不会发生冲突。

综上，ConcurrentLinkedQueue最终实现了无锁队列。

#### **使用场景**

ConcurrentLinkedQueue适合在对性能要求相对较高，同时对队列的读写存在多个线程同时进行的场景，即如果对队列加锁的成本较高则适合使用无锁的ConcurrentLinkedQueue来替代。下面我们来简单对比下ConcurrentLinkedQueue与我们常用的阻塞队列LinkedBlockingQueue的性能。
**表1：入队性能对比**

| 线程数 | ConcurrentLinkedQueue耗时(ms) | LinkedBlockingQueue耗时(ms) |
| ------ | ----------------------------- | --------------------------- |
| 5      | 22                            | 29                          |
| 10     | 50                            | 59                          |
| 20     | 99                            | 112                         |
| 30     | 139                           | 171                         |

测试数据：N个线程，每个线程入队10000个元素。



参考：

http://vickyqi.com/2015/10/29/JDK%E5%B9%B6%E5%8F%91%E5%B7%A5%E5%85%B7%E7%B1%BB%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94ConcurrentLinkedQueue/

http://cmsblogs.com/?p=2611





## Set

HashSet
LinkedHashSet
TreeSet
ConcurrentSkipListSet
CopyOnWriteArraySet





## 参考Reference:
https://github.com/CarpenterLee/JCFInternals
http://calvin1978.blogcn.com/articles/collection.html
http://wiki.jikexueyuan.com/project/java-collection/hashmap.html
http://wiki.jikexueyuan.com/project/java-enhancement/java-twentytwo.html
https://zhuanlan.zhihu.com/p/32997606
https://blog.csdn.net/qq_34448345/article/details/79835190
https://essviv.github.io/2017/01/25/%E9%9B%86%E5%90%88/java%E9%9B%86%E5%90%88%E5%AD%A6%E4%B9%A0%E4%B9%8BQueue%E7%9A%84%E5%AE%9E%E7%8E%B05/
http://welkinbai.coding.me/2017/06/15/jdk-collection/#%E5%85%B3%E7%B3%BB%E7%BB%93%E6%9E%84
https://blog.csdn.net/u010887744/article/category/6126950AC



## java concurrent
https://blog.csdn.net/qq_34448345/article/details/80087738
http://wiki.jikexueyuan.com/project/java-memory-model/basic.html
http://wiki.jikexueyuan.com/project/java-concurrent/introduction.html
http://wiki.jikexueyuan.com/project/java-concurrency/concurrency-multithreading.html
https://blog.csdn.net/column/details/wenniuwuren-jdk.html