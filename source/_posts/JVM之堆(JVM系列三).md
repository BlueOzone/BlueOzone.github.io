---
title: JVM之堆(JVM系列三)
toc: true
date: 2021-02-02 14:32:29
categories: Java
tags: Java进阶
---

## JVM整体架构
![JVM整体架构](https://github.com/BlueOzone/BlueOzone.github.io/blob/hexo/source/_posts/img/JVM%E6%95%B4%E4%BD%93%E7%BB%93%E6%9E%84.png?raw=true)

## 堆的核心概述
一个JVM实例只存在一个堆内存，堆也是Java内存管理的核心区域。
Java堆区在JVM启动的时候即被创建，其空间大小也就确定了。是JVM管理的最大一块内存空间。
堆内存的大小是可以通过启动参数调节的。
《Java虚拟机规范》规定，堆可以处于物理上不连续的内存空间中，但在逻辑上它应该被视为连续的。
所有的线程共享Java堆，在这里还可以划分线程私有的缓冲区（Thread Local Allocation Buffer, TLAB）。

《Java虚拟机规范》中对Java堆的描述是：所有的对象实例以及数组都应当在运行时分配在堆上。
数组和对象可能永远不会存储在栈上，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置。
在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除。
堆，是GC（Garbage Collection，垃圾收集器）执行垃圾回收的重点区域。

### 内存细分
现代垃圾收集器大部分都基于分代收集理论设计，堆空间细分为：
Java 7及其之前堆内存逻辑上分为三部分：新生区+养老区+永久区。
Java 8及其之后堆内存逻辑上分为三部分：新生区+养老区+元空间。

约定：
新生区==新生代==年轻代
养老区==老年区==老年代
永久区==永久代

## 设置堆内存大小与OOM
### 堆空间大小的设置
Java堆区用于存储Java对象实例，堆的大小在JVM启动时就已经设定好了，大家可以通过选项"-Xmx"和"-Xms"来进行设置。
`"-Xms"用于表示堆区的起始内存，等价于-XX:InitialHeapSize`

`"-Xmx"则用于表示堆区的最大内存，等价于-XX:MaxHeapSize`

一旦堆区中的内存大小超过"-Xmx"所指定的最大内存时，将会抛出OutOfMemoryError异常。
通常会将-Xms和-Xmx两个参数配置相同的值，其目的是为了能够在java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，从而提高性能。

默认情况下，初识内存大小：物理电脑内存大小 / 64；
最大内存大小：物理电脑内存大小 / 4。

```
-Xms 用来设置堆空间（年轻代+老年代）的初识内存大小
	 -X  是JVM运行参数
	 -ms 是memory start

-Xmx 用来设置堆空间（年轻代+老年代）的最大内存大小

开发中建议起始内存和最大内存设置成一样的，避免在扩、缩内存的时候给系统造成过多的压力。
```

### OutOfMemory举例

```
public class OOMTest {
    public static void main(String[] args) {
        ArrayList<Picture> list = new ArrayList<>();
        while (true){
            try {
                Thread.sleep(20);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            list.add(new Picture(new Random().nextInt(1024*1024)));
        }

    }
}

class Picture{
    private byte[] pixels;
    public Picture(int length){
        this.pixels = new byte[length];
    }
}
```
执行以上代码，将会抛出如下错误：
```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at jvm.Picture.<init>(OOMTest.java:31)
	at jvm.OOMTest.main(OOMTest.java:22)
```

## 年轻代与老年代
存储在JVM中的Java对象可以被划分为两类：
1. 一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速
2. 另外一类对象的生命周期却非常长，在某些极端的情况下还能够与JVM的生命周期保持一致。

Java堆区进一步细分的话，可以划分为年轻代（YoungGen）和老年代（OldGen）。
其中年轻代又可以划分为Eden空间、Survivor 0空间和Survivor 1空间（有时也叫做from区、to区）

![年轻代与老年代](https://github.com/BlueOzone/BlueOzone.github.io/blob/hexo/source/_posts/img/%E5%B9%B4%E8%BD%BB%E4%BB%A3%E5%92%8C%E8%80%81%E5%B9%B4%E4%BB%A3.png?raw=true)

默认情况下，新生代与老年代在堆结构的占比为：
-XX:NewRatio=2，表示新生代占1，老年代占2，新生代占整个堆的1/3。
可以修改参数：-XX:NewRatio=4，表示新生代占1，老年代占4，新生代占整个堆的1/5。

**·** 在HotSpot中，Eden空间和另外两个Survivor空间缺省所占的比例是8:1:1。
**·** 当然开发人员可以通过选项`-XX:SurvivorRatio`调整这个空间比例。比如`-XX:SurvivorRatio=8`。
**·** 几乎所有的Java对象都是在Eden区被new出来的。
**·** 绝大部分的Java对象的销毁都在新生代进行了。
**·** 可以使用选项"-Xmn"设置新生代最大内存大小。

## 图解对象分配过程
### 概述
为新对象分配内存是一件非常严谨和复杂的任务，JVM的设计者们不仅需要考虑内存如何分配、在哪里分配等问题，并且由于内存分配算法与内存回收算法密切相关，所以还需要考虑GC执行完内存回收后是否会在内存空间中产生内存碎片。以下为描述对象分配以及垃圾回收的相关过程：

![对象分配过程](https://github.com/BlueOzone/BlueOzone.github.io/blob/hexo/source/_posts/img/%E5%AF%B9%E8%B1%A1%E5%88%86%E9%85%8D%E8%BF%87%E7%A8%8B.png?raw=true)
1. new的对象先放伊甸园区(Eden)。此区有大小限制。
2. 当Eden的空间填满时，程序还需要继续创建对象，JVM的垃圾回收器将对新生代进行垃圾回收(Minor GC)，将新生代中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到Eden。
3. 然后将Eden中的剩余对象移动到Survivor 0区。
4. 如果Eden的空间继续填满，再次触发垃圾回收，此时上次幸存下来的放到Survivor 0区的，没有回收的，就会放到Survivor 1区。
5. 如果再次经历垃圾回收，此时会重新放回Survivor 0区，接着再去Survivor 1区；每经历一次垃圾回收，from区的数据就会放到to区。
6. 如果Survivor区中某个对象经历垃圾回收后一直存在，那它啥时候能去养老区呢？默认是15次垃圾回收后会被放到养老区，这个次数可以通过参数：`-XX:MaxTenuringThreshold=<N>`进行设置。
7. 当养老区的内存也不足以存放新对象时，会触发GC：Major GC，进行养老区的内存清理。
8. 若养老区执行了Major GC之后发现依然无法进行对象的保存，就会产生OOM异常。

总结：
针对Survivor 0和Survivor 1区的总结：复制之后有交换，谁空谁是to区。
关于垃圾回收：频繁在新生区收集，很少在养老区收集，几乎不在永久区/元空间收集。

## Minor GC、Major GC、Full GC
JVM在进行GC时，并非每次都对上面三个内存区域一起回收的，大部分时候回收的都是指新生代。
针对HotSpot VM的实现，它里面的GC按照回收区域又分为两大种类型：一种是部分收集（Partial GC），一种是整堆收集（Full GC）。

部分收集：不是完整收集整个Java堆的垃圾收集。其中又分为：
`新生代收集（Minor GC / Young GC）：只是新生代（Eden/S0,S1）的垃圾收集。`
`老年代收集（Major GC / Old GC）：只是老年代的垃圾收集。`

目前，只有CMS GC会有单独收集老年代的行为。
注意，很多时候Major GC会和Full GC混淆使用，需要具体分辨是老年代还是整堆回收。

`混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集。`

整堆收集（Full GC）：收集整个Java堆和方法区的垃圾收集。

### 最简单的分代式GC策略的触发条件
**年轻代GC（Minor GC）**触发机制：
（1）当年轻代空间不足时，就会触发Minor GC，这里的年轻代满指的是Eden区满了，单单的Survivor区满不会主动触发GC。（但是每次Minor GC都会清理年轻代的内存）。
（2）因为Java对象大多都具备朝生夕死的特性，所以Minor GC非常频繁，一般回收速度也比较快。
（3）Minor GC会引发STW，暂停其他用户的线程，等垃圾回收结束，用户线程才恢复运行。


**老年代GC（Major GC/Full GC）**触发机制：
（1）指发生在老年代的GC，对象从老年代消失时，我们说"Major GC"或"Full GC"发生了。
（2）出现了Major GC，经常会伴随至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）。
也就是在老年代空间不足时，会先尝试触发Minor GC。如果之后空间还不足，则触发Major GC。
（3）Major GC的速度一般会比Minor GC 慢10倍以上，STW的时间更长。
（4）如果Major GC后，内存还不足，就报OOM了。


**Full GC**触发机制：
触发Full GC执行的情况有如下五种：
（1）调用System.gc()时，系统建议执行Full GC，但是不必然执行。
（2）老年代空间不足。
（3）方法区空间不足。
（4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存。
（5）由Eden区、Survivor Space 0（From Space）区向Survivor Space 1（To Space）区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小。

说明：Full GC是开发或调优中尽量要避免的。这样暂停时间会短一些。


## 堆空间分代思想
为什么需要把Java堆分代？不分代就不能正常工作了吗？

经研究，不同对象的生命周期不同。70%-99%的对象是临时对象。
新生代：有Eden、两块大小相同的Survivor（又称为from/to，s0/s1）构成，to总为空。
老年代：存放新生代中经历多次GC仍然存活的对象。

其实不分代完全可以，分代的唯一理由就是优化GC性能。如果没有分代，那所有的对象都在一块，就如同把一个学校的人都关在一个教室。GC的时候要找到哪些对象没用，这样就会对堆的所有区域进行扫描。
而很多对象都是朝生夕死的，如果分代的话，把新创建的对象放到某一地方，当GC的时候先把这块存储朝生夕死对象的区域进行回收，这样就会腾出很大空间出来。


## 内存分配策略

如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被Survivor区容纳的话，将被移动到Survivor空间中，并将对象年龄设为1。对象在Survivor区中每熬过一次Minor GC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁，其实每个JVM、每个GC都有所不同）时，就会被晋升到老年代中。
对象晋升老年代的年龄阈值，可以通过选项`-XX:MaxTenuringThreshold`来设置。

针对不同年龄段的对象分配原则如下所示：
· 优先分配到Eden
· 大对象直接分配到老年代

尽量避免程序中出现过多的大对象
· 长期存活的对象分配到老年代
· 动态对象年龄判断
如果Survivor区中相同的年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代，无须等到MaxTenuringThreshold中要求的年龄。
· 空间分配担保
-XX:HandlePromotionFailure


## 为对象分配内存：TLAB
为什么有TLAB(Thread Local Allocation Buffer)？
· 堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据
· 由于对象实例的创建在JVM中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的
· 为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度。

什么是TLAB？
· 从内存模型而不是垃圾收集的角度，对Eden区域继续进行划分，JVM为每个线程分配了一个私有缓存区域，它包含在Eden空间内；**每个线程私有的缓存区，称为TLAB(Thread Local Allocation Buffer)**。
· 多线程同时分配内存时，会优先使用TLAB，这样可以避免一系列的非线程安全问题，同时还能够提升内存分配的吞吐量，因此我们可以将这种内存分配方式称之为快速分配策略。

TLAB的再说明：
· 尽管不是所有的对象实例都能够在TLAB中成功分配内存，但JVM确实是将TLAB作为内存分配的首选。
· 在程序中，开发人员可以通过选项`-XX:UseTLAB`设置是否开启TLAB空间。
· 默认情况下，TLAB默认开启。TLAB空间的内存非常小，仅占有整个Eden空间的1%，当然我们可以通过选项`-XX:TLABWasteTargetPercent`设置TLAB空间所占用Eden空间的百分比大小。
· 一旦对象在TLAB空间分配内存失败时，JVM就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在Eden空间中分配内存。

## 小结堆空间的参数设置

-XX:+PrintFlagsInitial：查看所有的参数的默认初始值。
-XX:+PrintFlagsFinal：查看所有的参数的最终值（可能会存在修改，不再是初始值）。
-Xms：初识堆空间内存（默认为物理内存的1/64）。
-Xmx：最大堆空间内存（默认为物理内存的1/4）。
-Xmn：设置新生代的大小（初始值及最大值）。
-XX:NewRatio：配置新生代与老年代在堆结构的占比。
-XX:SurvivorRatio：设置新生代中Eden和S0/S1空间的比例。
-XX:MaxTenuringThreshold：设置新生代垃圾的最大年龄。
-XX:+PrintGCDetails：输出详细的GC处理日志。
-XX:+PrintGC：打印GC简要信息
-verbose:gc：打印GC简要信息
-XX:HandlerPromotionFailure：是否设置空间分配担保。

在发生Minor GC之前，只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小，就会进行Minor GC，否则将进行Full GC。

## 堆是分配对象存储的唯一选择吗


在Java虚拟机中，对象是在Java堆中分配内存的，这是一个普遍的常识。但是，有一种特殊情况，那就是如果经过逃逸分析（Escape Analysis）后发现，一个对象并没有逃逸出方法的话，那么就可能被优化成栈上分配。这样就无需在堆上分配内存，也无须进行垃圾回收了。这也是最常见的堆外存储技术。


### 逃逸分析概述

· 如何将堆上的对象分配到栈，需要使用逃逸分析手段。
· 这是一种可以有效减少Java程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法。
· 通过逃逸分析，Java HotSpot编译器能够分析出一个新的对象的引用的使用范围从而决定是否要将这个对象分配到堆上。

· 逃逸分析的基本行为就是分析对象动态作用域：
（1）当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸。
（2）当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸。例如作为调用参数传递到其他地方中。


没有发生逃逸的对象，则可以分配到栈上，随着方法执行的结束，栈空间就被移除。


```
public class EscapeAnalysis {
    public EscapeAnalysis obj;
    /**
    方法返回EscapeAnalysis对象，发送逃逸
     */
    public EscapeAnalysis getInstance(){
        return obj == null? new EscapeAnalysis() : obj;
    }
    /**
    为成员属性赋值，发生逃逸
     */
    public void setObj(){
        this.obj = new EscapeAnalysis();
    }
    /*
    对象的作用域仅在当前方法中有效，没有发生逃逸
     */
    public void useEscapeAnalysis(){
        EscapeAnalysis e = new EscapeAnalysis();
    }
    /*
    引用成员变量的值，发生逃逸
     */
    public void useEscapeAnalys2(){
        EscapeAnalysis e = getInstance();
    }
}
```


参数设置：
在JDK 7及其以后，HotSpot中默认就已经开启了逃逸分析。
如果使用的是较早的版本，开发人员则可以通过：
选项`-XX:+DoEscapeAnalysis`显示开启逃逸分析。
通过选项`-XX:+PrintEscapeAnalysis`查看逃逸分析的筛选结果。

结论：
开发中要善用局部变量和全局变量。

### 逃逸分析：代码优化
根据逃逸分析，编译器可以对代码做如下优化：
一、**栈上分配。**将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配。
二、**同步省略。**如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步。
三、**分离对象或标量替换。**有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

#### 代码优化之栈上分配

JIT编译器在编译期间根据逃逸分析的结果，发现如果一个对象并没有逃逸出方法的话，就可能被优化成栈上分配。分配完成后，继续在调用栈内执行，最后线程结束，栈空间回收，局部变量对象也被回收。这样就无须进行垃圾回收了。

常见的栈上分配的场景：
在逃逸分析中，已经说明了。分别是给成员变量赋值、方法返回值、实例引用传递。

实例代码分析：
```
public class StackAllocation {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        long end = System.currentTimeMillis();
        System.out.println("花费时间：" + (end - start) + "ms");
        //此处暂停是为了jvisualvm等工具可视化分析
        try {
            Thread.sleep(10000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private static void alloc() {
        User user = new User();
    }

    static class User{}
}
```

```
设置启动参数(关闭逃逸分析)：
-Xmx1G -Xmn1G -XX:-DoEscapeAnalysis -XX:+PrintGCDetails
输出：
花费时间：199ms


设置启动参数(开启逃逸分析)：
-Xmx1G -Xmn1G -XX:+DoEscapeAnalysis -XX:+PrintGCDetails
输出：
花费时间：10ms
```

可以看到，开启逃逸分析后，使用了栈上分配，花费的时间、内存都极大的减少了；
且在关闭逃逸分析的情况下，还可能会产生GC。


#### 代码优化之同步省略

线程同步的代价是相当高的，同步的后果是降低并发性和性能。

在动态编译同步块的时候，JIT编译器可以借助逃逸分析来判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程。如果没有，那么JIT编译器在编译这个同步块的时候就会取消对这部分代码的同步。这样就能大大提高并发性和性能。这个取消同步的过程就叫同步省略，也叫锁消除。

例如以下代码：
```
public void f() {
    Object hollis = new Object();
    synchronized (hollis) {
        System.out.println(hollis);
    }
}
```
代码中对hollis这个对象进行加锁，但是hollis对象的生命周期只在f()方法中，并不会被其他线程所访问到，所以在JIT编译阶段就会被优化掉。优化成：
```
public void f() {
    Object hollis = new Object();
    System.out.println(hollis);
}
```

#### 代码优化之标量替换

标量（Scalar）是指一个无法再分解成更小的数据的数据。Java中的原始数据类型就是标量。
相对的，那些还可以分解的数据叫做聚合量（Aggregate），Java中的对象就是聚合量，因为他可以分解成其他聚合量和标量。
在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来代替。这个过程就是标量替换。


例如以下代码：
```
public static void main(String[] args) {
        alloc();
    }

private static void alloc() {
    Point point = new Point(1, 2);
    System.out.println(point.x + point.y);
}

static class Point {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
```

以上代码，经过标量替换后，就会变成：

```
private static void alloc() {
    int x = 1;
    int y = 2;
    System.out.println(x + y);
    System.out.println(point.x + point.y);
}
```

可以看到，Point这个聚合量经过逃逸分析后，发现他并没有逃逸，就被替换成两个聚合量了。那么标量替换有什么好处呢？
就是可以大大减少堆内存的占用。因为一旦不需要创建对象了，那么就不再需要分配堆内存了、
标量替换为栈上分配提供了良好的基础。


标量替换参数设置：
参数`-XX:+EliminateAllocations`；开启标量替换（默认开启），允许将对象打散分配在栈上。


实例代码分析：
```
public class ScalarTest {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        long end = System.currentTimeMillis();
        System.out.println("花费时间：" + (end - start) + "ms");
        //此处暂停是为了jvisualvm等工具可视化分析
        try {
            Thread.sleep(10000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private static void alloc() {
        User user = new User();
        user.id = 5;
        user.name = "www.baidu.com";
    }

    public static class User {
        public int id;
        public String name;
    }
}
```

根据以下输出，可以看到，在关闭逃逸分析的情况下，还可能会产生GC。

```
设置启动参数(关闭标量替换)：
-Xmx100m -Xmn100m -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:-EliminateAllocations
输出：
[GC (Allocation Failure)  77312K->848K(90112K), 0.0432396 secs]
[GC (Allocation Failure)  78160K->864K(90112K), 0.0018051 secs]
[GC (Allocation Failure)  78176K->816K(90112K), 0.0023638 secs]
花费时间：153ms


设置启动参数(开启标量替换)：
-Xmx100m -Xmn100m -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:+EliminateAllocations
输出：
花费时间：8ms
```

### 小结

所以，根据以上结论，堆并不是分配对象存储的唯一选择。
**但是!**
目前逃逸分析并不成熟，原因是无法保证逃逸分析的性能消耗一定能高于它的消耗。虽然经过逃逸分析可以做标量替换、栈上分配和同步省略。但是逃逸分析自身也是需要一系列复杂分析的，这也是一个相对耗时的过程。

且目前在HotSpot虚拟机上，并没有真正的实现栈上分配，我们的例子中，对象没有在堆上分配的效果，其实是标量替换实现的。

所以目前仍可以认为：**对象实例都是分配在堆上。**


## 总结
年轻代是对象的诞生、成长、消亡的区域，一个对象在这里产生、应用，最后被垃圾回收器收集、结束生命。

老年代放置长生命周期的对象，通常都是从Survivor区域筛选拷贝过来的Java对象，当然，也有特殊情况，我们知道普通的对象会被分配在TLAB上；如果对象较大，JVM会试图直接分配在Eden其他位置上；如果对象太大，完全无法在新生代找到足够长的连续空闲空间，JVM就会直接分配到老年代。

当GC只发生在年轻代中，回收年轻代对象的行为被称为Minor GC。当GC发生在老年代时则被称为Major GC或者Full GC。一般的，Minor GC的发生频率要比Major GC高很多，即老年代中垃圾回收发生的频率将大大低于年轻代。

