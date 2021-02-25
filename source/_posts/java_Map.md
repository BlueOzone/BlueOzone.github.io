---
title: Java-Map之HashMap
categories: Java
tags: Java基础
toc: true
---


### Map概述
Map 提供了一个更通用的元素存储方法。
Map 集合类用于存储元素对（称作“键”和“值”），其中每个键映射到一个值。<!--more-->

### Map的实现类
① HashMap：作为Map的主要实现类；无序的，**线程不安全**的，效率高；可以存储null的key和value。
② LinkedHashMap extends HashMap：有序的；保证在遍历map元素时，可以按照添加的顺序实现遍历。原因：在原有的HashMap底层结构基础上，添加了一对指针，指向前一个和后一个元素。对于频繁的遍历操作，此类执行效率高于HashMap。
③ TreeMap：保证按照添加的key-value键值对进行排序，实现排序遍历。此时key的自然排序或定制排序；底层使用红黑树存储。
④ Hashtable：作为古老的实现类；线程安全的，效率低；不能存储null的key和value。
⑤ Properties extends Hashtable：常用来处理配置文件。key和value都是String类型。

### HashMap的底层实现原理

首先以jdk7为例说明：
`HashMap map = new HashMap();`执行后：
底层创建了长度是16的一维数组`Entry[] table`。
`map.put(key1,value1);`执行后：



在不断的添加过程中，会涉及扩容问题，当超出临界值（且要存放的位置非空）时，扩容。默认的扩容方式：扩容为原来容量的2倍，并将原有数据复制过来


jdk8 相较于jdk7在底层实现方面的不同：
1.new HashMap()：底层没有创建一个长度为16的数组
2.jdk 8底层的数组是：Node[]，而非Entry[]
3.首次调用put()方法时，底层创建长度为16的数组
4.jdk 7底层结构只有：数组+链表。jdk 8中底层结构：数组+链表+红黑树。
当数组的某一个索引位置上的元素以链表形式存在的数据个数 > 8且当前数组的长度 >64时，
此时此索引位置上的所有数据改为使用红黑树存储。

DEFAULT\_INITIAL\_CAPACITY：HashMap的默认容量：16
DEFAULT\_LOAD\_FACTOR：HashMap的默认加载因子：0.75
threadold：扩容的临界值=容量\*填充因子：16\*0.75 => 12
TREEIFY\_THRESHOLD：Bucket中链表长度大于该默认值，转化为红黑树：8
MIN\_TREEIFY\_CAPACITY：桶中的Node被树化时最小的hash表容量：64

以下附上HashMap部分源码及其解析：

```
//底层使用的数组
transient Node<K,V>[] table;

//HashMap的默认容量：16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

static final int MAXIMUM_CAPACITY = 1 << 30;

//HashMap的默认加载因子：0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//Bucket中链表长度大于该默认值，转化为红黑树：8
static final int TREEIFY_THRESHOLD = 8;

static final int UNTREEIFY_THRESHOLD = 6;

//桶中的Node被树化时最小的hash表容量：64
static final int MIN_TREEIFY_CAPACITY = 64;
//put方法
public V put(K key, V value) {
	//计算key的hash值传入putVal方法
    return putVal(hash(key), key, value, false, true);
}
static final int hash(Object key) {
    int h;
	//hash值计算方法
	//若key==null，hash=0;
	//key.hashCode()函数调用的是key键值类型自带的哈希函数，返回int型散列值，然后做一次16位右位移异或混合计算得出hash值；右位移16位，正好是32bit的一半，自己的高半区和低半区做异或，就是为了混合原始哈希码的高位和低位，以此来加大低位的随机性。
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    HashMap.Node<K,V>[] tab; HashMap.Node<K,V> p; int n, i;
    //tab = table;判断当前table数组是否为空
    if ((tab = table) == null || (n = tab.length) == 0)
        //若为空，调用resize方法，table为空时resize返回一个初始化过后的数组，长度为16
        n = (tab = resize()).length;
    //通过哈希值计算索引值，判断数组上该索引上是否有值
    if ((p = tab[i = (n - 1) & hash]) == null)
        //若为null，则直接插入新的数据
        tab[i] = newNode(hash, key, value, null);
    //若以上条件不满足，则需要比较hash值和key值是否相等
    else {
        HashMap.Node<K,V> e; K k;
        //先判断p处的hash值和传进来的hash值、再判断p.key和传进来的key的地址值是否相等、再通过equals方法判断两者的值是否相等
        //注意第一个逻辑与后面的条件，若地址值都相等了就不需要equals判断了
        if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
            //将p指向的地址赋给e;
            e = p;
        //判断p处是否使用红黑树存储
        else if (p instanceof HashMap.TreeNode)
            e = ((HashMap.TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //若以上条件不满足，进入以下
        else {
            for (int binCount = 0; ; ++binCount) {
                //判断node节点p是否存在下一个元素，即使判断p.next是否为空
                if ((e = p.next) == null) {
                    //若p.next为空，则把新的元素放到p.next里，原本的值还在数组索引的第一处
                    p.next = newNode(hash, key, value, null);
                    //判断链表长度，决定是否使用红黑树存储；若大于TREEIFY_THRESHOLD-1，则使用红黑树进行存储
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    //跳出循环，e=p.next;
                    break;
                }
                //先判断e=p.next处的hash值和传进来的hash值、再判断e.key和传进来的key的地址值是否相等、再通过equals方法判断两者的值是否相等
                if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                    //若相等，跳出循环,e=p.next
                    break;
                //若以上条件不符合，则p=e;即p = p.next;
                p = e;
            }
        }
        //经过以上条件后，判断e是否为空
        if (e != null) { // existing mapping for key
            //若不为空，取出e的value值给变量oldValue;
            V oldValue = e.value;

            if (!onlyIfAbsent || oldValue == null)
                //更新e的value值
                e.value = value;
            afterNodeAccess(e);
            //返回旧的value值
            return oldValue;
        }
    }
    ++modCount;
    //添加新元素后，判断++size是否大于临界值
    if (++size > threshold)
        //若大于临界值，调用resize()扩容
        resize();
    afterNodeInsertion(evict);
    return null;
}

```


### Map中的方法

**添加、删除、修改操作：**
Object put(Object key,Object value)：将指定的key-value添加到（或修改到）当前map对象中
void putAll(Map m)：将m中的所有key-value复制到当前map中
Object remove(Object key)：移除指定key的key-value数据，并返回value
void clear()：清空当前map的所有数据
**元素查询的操作：**
Object get(Object key)：获取指定key对应的value
boolean containsKey(Object key)：map对象中是否包含指定的key
boolean containsValue(Object value)：map对象中是否包含指定的value
int Size()：返回当前map中key-value键值对的个数
boolean isEmpty()：判断当前map是否为空
boolean equals(Object obj)：判断当前map和参数obj是否相等

**元视图操作的方法：**
Set keySet()：返回所有key构成的Set集合
Collection values()：返回所有value构成的Collection集合
Set entrySet()：返回所有key-value键值对构成的Set集合


### 线程安全的几种Map

#### ConcurrentHashMap - 推荐
* 当你程序需要高度的并行化的时候，你应该使用ConcurrentHashMap；
* 尽管没有同步整个Map，但是它仍然是线程安全的；
* 读操作非常快，而写操作则是通过加锁完成的；
* 在对象层次上不存在锁（即不会阻塞线程）；
* 锁的粒度设置的非常好，只对哈希表的某一个key加锁；
* ConcurrentHashMap不会抛出ConcurrentModificationException，即使一个线程在遍历的同时，另一个线程尝试进行修改；
* ConcurrentHashMap会使用多个锁。

#### SynchronizedMap
这种是直接使用工具类里面的方法创建SynchronizedMap，把传入进行的HashMap对象进行了包装同步。
用法：`Map<String, Object> map22 = Collections.synchronizedMap(new HashMap<String, Object>());`

* 会同步整个对象；
* 每一次的读写操作都需要加锁；
* 对整个对象加锁会极大降低性能；
* 这相当于只允许同一时间内至多一个线程操作整个Map，而其他线程必须等待；
* 它有可能造成资源冲突（某些线程等待较长时间）；
* SynchronizedHashMap会返回Iterator，当遍历时进行修改会抛出异常。

#### HashTable
HashTable作为古老的实现类；线程安全的，效率低；不能存储null的key和value。
用法：`Map<String, Object> map33 = new Hashtable<>();`
通过查阅HashTable的源码可以知道，HashTable初始化的容量是11，加载因子为0.75：

```/**
 * Constructs a new, empty hashtable with a default initial capacity (11)
 * and load factor (0.75).
 */
public Hashtable() {
    this(11, 0.75f);
}
```

再继续查看HashTable的源码发现，get/put方法都被synchronized关键字修饰，说明它们是方法级别阻塞的，它们占用共享资源锁，所以导致同时只能一个线程操作get或者put，而且get/put操作不能同时执行，所以这种同步的集合效率非常低，一般不建议使用这个集合。
```
public synchronized V get(Object key) {
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
        if ((e.hash == hash) && e.key.equals(key)) {
            return (V)e.value;
        }
    }
    return null;
}

public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }

    addEntry(hash, key, value, index);
    return null;
}
```








