---
title: Java-Map之HashMap
date: 2020-05-18 20:52:11
---


### 1、Map概述
Map 提供了一个更通用的元素存储方法。Map 集合类用于存储元素对（称作“键”和“值”），其中每个键映射到一个值。从概念上而言，您可以将 List 看作是具有数值键的 Map。而实际上，除了 List 和 Map 都在定义 java.util 中外，两者并没有直接的联系。


### 2、Map的实现类
① HashMap：作为Map的主要实现类；线程不安全的，效率高；存储null的key和value。<br>
② LinkedHashMap extends HashMap：保证在遍历map元素时，可以按照添加的顺序实现遍历。原因：在原有的HashMap底层结构基础上，添加了一对指针，指向前一个和后一个元素。对于频繁的遍历操作，此类执行效率高于HashMap。<br>
③ TreeMap：保证按照添加的key-value键值对进行排序，实现排序遍历。此时key的自然排序或定制排序；底层使用红黑树存储。<br>
④ Hashtable：作为古老的实现类；线程安全的，效率低；不能存储null的key和value。<br>
⑤ Properties extends Hashtable：常用来处理配置文件。key和value都是String类型。

### 3、HashMap的底层实现原理

以jdk7为例说明：<br>
`HashMap map = new HashMap();`<br>
在实例化以后，底层创建了长度是16的一维数组`Entry[] table`。<br>
`map.put(key1,value1);`<br>
首先，调用key1所在类的hashCode()计算key1哈希值，此哈希值经过某种算法计算以后，得到在Entry数组中的存放位置。<br>
如果此位置上的数据为空，此时key1和value1添加成功。	--->情况1<br>
如果此位置上的数据不为空，（意味着此位置上存在一个或多个数据（以链表形式存在）），比较key1和已经存在的一个或多个数据的哈希值：<br>
&emsp;如果key1的哈希值与已经存在的数据的哈希值都不相同；此时key1和value1添加成功。	--->情况2<br>
&emsp;如果key1的哈希值与已经存在的某个数据的哈希值相同。继续比较：调用key1所在类的equals()方法，比较：<br>
&emsp;&emsp;如果equals返回false，此时key1和value1添加成功；	--->情况3<br>
&emsp;&emsp;如果equals返回true，使用新的value替换旧的value。<br>

补充：关于情况2和情况3：此时key1-value1和原来的数据以链表的方式存储。

在不断的添加过程中，会涉及扩容问题，当超出临界值（且要存放的位置非空）时，扩容。默认的扩容方式：扩容为原来容量的2倍，并将原有数据复制过来


jdk8 相较于jdk7在底层实现方面的不同：<br>
1.new HashMap()：底层没有创建一个长度为16的数组<br>
2.jdk 8底层的数组是：Node[]，而非Entry[]<br>
3.首次调用put()方法时，底层创建长度为16的数组<br>
4.jdk 7底层结构只有：数组+链表。jdk 8中底层结构：数组+链表+红黑树。<br>
当数组的某一个索引位置上的元素以链表形式存在的数据个数 > 8且当前数组的长度 >64时，
此时此索引位置上的所有数据改为使用红黑树存储。<br>

DEFAULT\_INITIAL\_CAPACITY：HashMap的默认容量：16<br>
DEFAULT\_LOAD\_FACTOR：HashMap的默认加载因子：0.75<br>
threadold：扩容的临界值=容量\*填充因子：16\*0.75 => 12<br>
TREEIFY\_THRESHOLD：Bucket中链表长度大于该默认值，转化为红黑树：8<br>
MIN\_TREEIFY\_CAPACITY：桶中的Node被树化时最小的hash表容量：64<br>

以下附上HashMap部分源码及其解析：

<pre><code>
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

</pre></code>


### 4、Map中的方法

**添加、删除、修改操作：**<br>
Object put(Object key,Object value)：将指定的key-value添加到（或修改到）当前map对象中<br>
void putAll(Map m)：将m中的所有key-value复制到当前map中<br>
Object remove(Object key)：移除指定key的key-value数据，并返回value<br>
void clear()：清空当前map的所有数据<br>
**元素查询的操作：**<br>
Object get(Object key)：获取指定key对应的value<br>
boolean containsKey(Object key)：map对象中是否包含指定的key<br>
boolean containsValue(Object value)：map对象中是否包含指定的value<br>
int Size()：返回当前map中key-value键值对的个数<br>
boolean isEmpty()：判断当前map是否为空<br>
boolean equals(Object obj)：判断当前map和参数obj是否相等<br>

**元视图操作的方法：**<br>
Set keySet()：返回所有key构成的Set集合<br>
Collection values()：返回所有value构成的Collection集合<br>
Set entrySet()：返回所有key-value键值对构成的Set集合<br>
