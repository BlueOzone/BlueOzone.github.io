---
title: Java-集合之ArrayList
date: 2020-05-16 21:02:15
categories: Java
tags: Java基础
toc: true
---

### 集合框架的概述
集合、数组都是对多个数据进行存储操作的结构，简称**Java容器**。 说明：此时的存储，主要指的是内存层面的存储，不涉及到持久化的存储（.txt .jpg等）。
<!--more-->
以下描述引用于onJava 8：
集合（Collection） ：一个独立元素的序列，这些元素都服从一条或多条规则。List 必须以插入的顺序保存元素， Set 不能包含重复元素， Queue 按照排队规则来确定对象产生的顺序（通常与它们被插入的顺序相同）


### 关于数组的概述
数组在存储多个数据方面的特点：
① 一旦初始化以后，其长度就确定了;
② 数组一旦定义好，其元素的类型也就确定了。我们也就只能操作指定类型的数据了。比如：`String[] string;int[] int`。

数组在存储多个数据方面的缺点：
① 一旦初始化以后，其长度就不可修改。
② 数组中提供的方法非常有限，对于添加、删除、插入数据等操作，非常不便，同时效率不高。
③ 获取数组中实际元素的个数的需求，数组没有现成的属性或方法可用。
④ 数组存储数据的特点：有序、可重复。对于无序、不可重复的需求，不能满足。

### 集合涉及的接口、实现类

|----Collection接口：单列集合，用来存储一个一个的对象
&emsp;|----List接口：存储有序的、可重复的数据。   
&emsp;&emsp;|----ArrayList、LinkedList、Vector
&emsp;|----Set接口：存储无序的、不可重复的数据。   
&emsp;&emsp;|----HashSet、LinkedHashSet、TreeSet

List是Collection的子接口，ArrayList、LinkedList、Vector是List接口的实现类。

|----Map接口：双列集合，用来存储一对（key - value）一对的数据。  
&emsp;|----HashMap、LinkedHashMap、TreeMap、Hashtable、Properties

HashMap、LinkedHashMap、TreeMap、Hashtable、Properties是Map接口的实现类。

### Collection接口中常用方法
```
增：add(Object obj)
删：remove(int index) / remove(Object obj)
改：set(int index, Object ele)
查：get(int index)
插：add(int index, Object obj)
长度：size()	//默认情况下返回的是元素的个数，而不是数组本身的长度
遍历：
①Iterator迭代器方式
②增强for循环
③普通的循环
```

### ArrayList、LinkedList、Vector三者的异同
相同点：三者都实现了List接口，存储的数据都是有序的、可重复的；

不同点：
ArrayList：作为List接口的主要实现类，线程不安全的，效率高；底层使用Object[] elementData存储（其实是对数组的封装）；
LinkedList：对于频繁的插入、删除操作，使用此类效率比ArrayList高，底层使用双向链表存储；
Vector：作为List接口的古老实现类，线程安全的，效率低；底层使用Object[] elementData存储。


### ArrayList初始化解析
**ArrayList源码分析之jdk 7：**
`ArrayList list = new ArrayList();`
执行该语句时，底层创建了长度是10的Object[]数组elementData;

`list.add(123);`相当于`elementData[0] = new Integer(123);`

**ArrayList源码分析之jdk 8：**
```
private static final int DEFAULT_CAPACITY = 10;

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```

上图为jdk 8中的ArrayList部分源码，在执行语句：`ArrayList list = new ArrayList();`时；底层Object[] elementData数组初始化为{},此时并没有创建长度为10的数组；
在当第一次调用add()时：`list.add(123);`底层才创建了长度为10的数组，并将数据123添加到elementData数组中。

小结：jdk 7 中的ArrayList的对象的数组创建类似于单例的饿汉式（提前造好）；而jdk8中的ArrayList的对象
的数组创建类似于单例的懒汉式（需要时才初始化），延迟了数组的创建，节省内存。



### ArrayList扩容解析
先附上jdk 8中ArrayList的有关扩容部分的源码，主要是grow()方法：

```
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private void ensureExplicitCapacity(int minCapacity) {
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

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}

```

根据源码知道，ArrayList扩容最主要的在grow()方法，以下解释下该方法：
```
private void grow(int minCapacity) {
    //minCapacity = size + 1；可以理解为需要的数组长度+1
    // 获取当前数组的长度
    int oldCapacity = elementData.length;
    // 通过位移运算（计算机底层中位移运算更快），将新的数组长度定义为原来的1.5倍
    //为什么扩容的是1.5倍而不是其他倍？后面会解释
    //此处若溢出，newCapacity将为负值，不影响后续条件的判断
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //比较计算出的新长度与minCapacity
    if (newCapacity - minCapacity < 0)
        //如果扩容1.5倍后的长度仍比需要的长度小，则把newCapacity定义为需要的长度+1：minCapacity = size + 1
        newCapacity = minCapacity;

    //此条件是为了处理溢出的情况
    //比较计算出的新长度与MAX_ARRAY_SIZE：ArrayList允许的最大长度
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        //如果扩容后的长度大于允许的最大长度，调用hugeCapacity方法
        //根据hugeCapacity方法源码可知，当minCapacity > MAX_ARRAY_SIZE时，将返回Integer.MAX_VALUE
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    //调用Arrays.copyOf方法，传入旧的数组、扩容后的长度；返回一个拥有新的长度的数组（原有值不变）
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

**为什么扩容的是1.5倍而不是其他倍？**
\>因为一次性扩容太大(例如2.5倍)可能会浪费更多的内存(1.5倍最多浪费33%，而2.5被最多会浪费60%，3.5倍则会浪费71%……)。
\>但是一次性扩容太小，需要多次对数组重新分配内存，对性能消耗比较严重。
\>所以1.5倍是个经验值，它既能满足性能需求，也不会造成很大的内存消耗。
\>所以扩容扩多少，是JDK开发人员在时间、空间上做的一个权衡，提供出来的一个比较合理的数值。

**总结：**
创建集合的时候，如果能知道总长度是多少，最好定义集合的长度，避免因扩容带来的资源浪费和内存消耗。

### ArrayList遍历方式解析

ArrayList遍历总的有三种：
1、通过普通for循环的方式；
2、foreach的方式；
3、Iterator迭代器的方式。


这三种方式相信大家都知道，所以就不写相关的代码了，但是对于ArrayList来说哪种方式效率更高，更快呢？
ArrayList实现了RandomAccess接口，其中RandomAccess接口又这么一段描述：
```
* for typical instances of the class, this loop:
* 
*     for (int i=0, n=list.size(); i &lt; n; i++)
*         list.get(i);
* 
* runs faster than this loop:
* 
*     for (Iterator i=list.iterator(); i.hasNext(); )
*         i.next();
* 
```

从描述中，可以看出实现RandomAccess接口的集合类，使用for循环的效率会比Iterator高。
RandomAccess接口为ArrayList带来的好处：
1、可以快速随机访问集合。
2、使用快速随机访问（for循环）效率可以高于Iterator。

**注意**：ArrayList的iterator和listIterator方法返回的迭代器是fail-fast的：在创建迭代器之后，除非通过迭代器自身的remove或add方法从结构上对列表进行修改，否则在任何时间以任何方式对列表进行修改，迭代器都会抛出ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

注意，迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败迭代器会尽最大努力抛出ConcurrentModificationException。因此，为提高这类迭代器的正确性而编写一个依赖于此异常的程序是错误的做法：迭代器的快速失败行为应该仅用于检测bug。

### ArrayList之多线程

ArrayList本身是线程不安全的，在多线程操作ArrayList时，需要使用：
`List list1 = Collections.synchronizedList(list);`
该方法可使将指定集合包装成线程同步的集合，从而可以解决多线程并发访问集合时的线程安全问题；以下附上synchronizedList()方法的源码，可以看到，返回的list对象里，大部分方法内部都使用了synchronized关键字保证同步，锁mutex是对象本身。
但是注意，关于迭代器的方法并没有加上synchronized同步锁，所以在多线程的场景下，需要在外部加上synchronized同步锁保证线程安全。


```

public static <T> List<T> synchronizedList(List<T> list) {
    return (list instanceof RandomAccess ?
            new SynchronizedRandomAccessList<>(list) :
            new SynchronizedList<>(list));
}

static class SynchronizedList<E>
    extends SynchronizedCollection<E>
    implements List<E> {
    private static final long serialVersionUID = -7754090372962971524L;

    final List<E> list;

    SynchronizedList(List<E> list) {
        super(list);
        this.list = list;
    }
    SynchronizedList(List<E> list, Object mutex) {
        super(list, mutex);
        this.list = list;
    }

    public boolean equals(Object o) {
        if (this == o)
            return true;
        synchronized (mutex) {return list.equals(o);}
    }
    public int hashCode() {
        synchronized (mutex) {return list.hashCode();}
    }

    public E get(int index) {
        synchronized (mutex) {return list.get(index);}
    }
    public E set(int index, E element) {
        synchronized (mutex) {return list.set(index, element);}
    }
    public void add(int index, E element) {
        synchronized (mutex) {list.add(index, element);}
    }
    public E remove(int index) {
        synchronized (mutex) {return list.remove(index);}
    }

    public int indexOf(Object o) {
        synchronized (mutex) {return list.indexOf(o);}
    }
    public int lastIndexOf(Object o) {
        synchronized (mutex) {return list.lastIndexOf(o);}
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        synchronized (mutex) {return list.addAll(index, c);}
    }

    public ListIterator<E> listIterator() {
        return list.listIterator(); // Must be manually synched by user
    }

    public ListIterator<E> listIterator(int index) {
        return list.listIterator(index); // Must be manually synched by user
    }

    public List<E> subList(int fromIndex, int toIndex) {
        synchronized (mutex) {
            return new SynchronizedList<>(list.subList(fromIndex, toIndex),
                                        mutex);
        }
    }
}

```

