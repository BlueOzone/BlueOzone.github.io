---
title: JVM之类加载子系统(JVM系列一)
toc: true
date: 2021-01-19 15:37:41
categories: JVM
tags: Java进阶
---


## JVM基本概念
Java虚拟机（英语：Java Virtual Machine，缩写为JVM），一种能够运行Java bytecode的虚拟机，以堆栈结构机器来进行实做。最早由Sun微系统所研发并实现第一个实现版本，是Java平台的一部分，能够运行以Java语言写作的软件程序。<!--more-->
Java虚拟机作用：就是二进制字节码的运行环境，负责装载字节码到其内部，解释/编译为对应平台上的机器指令执行。
由于跨平台性的设计，Java的指令都是根据栈来设计的。
JVM的生命周期：启动、运行、退出。

## JVM整体架构
![JVM整体架构](https://github.com/BlueOzone/BlueOzone.github.io/blob/hexo/source/_posts/img/JVM%E6%95%B4%E4%BD%93%E7%BB%93%E6%9E%84.png?raw=true)

## 类加载器与类的加载过程

类加载器子系统负责从文件系统或者网络中加载Class文件，Class文件在文件开头有特定的文件标识。
ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定。


### 类加载过程

![类加载过程1](https://github.com/BlueOzone/BlueOzone.github.io/blob/hexo/source/_posts/img/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B-%E6%95%B4%E4%BD%93.png?raw=true)

![类加载过程2](https://github.com/BlueOzone/BlueOzone.github.io/blob/hexo/source/_posts/img/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B-%E7%BB%86%E8%8A%82.png?raw=true)

#### 加载
1. 通过一个类的全限定名获取定义此类的二进制字节流；
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构；
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

#### 链接
**验证：**
目的在于确保Class文件的字节流中包含的信息符合当前虚拟机要求，保证被加载类的正确性；主要包括四种验证：文件格式验证、元数据验证、字节码验证、符号引用验证。

**准备：**
为类变量分配内存，并设置该类变量的默认初始值。
这里不包含用final修饰的static，因为final在编译的时候就会分配了，准备阶段会显示初始化。
注意，这里不会为实例变量分配初始化。

**解析：**
将常量池内的符号引用转换为直接引用的过程。

#### 初始化
初始化阶段就是执行类构造器方法`<clinit>()`的过程。
`<clinit>()`不同于类的构造器。
若该类具有父类，JVM会保证子类的`<clinit>()`执行前，父类的`<clinit>()`已经执行完毕。
虚拟机必须保证一个类的`<clinit>()`方法在多线程下被同步加锁，确保只被加载一次。

### 类加载器的分类
JVM支持两种类型的类加载器，分别为引导类加载器（Bootstrap ClassLoader）和自定义类加载器（User-Defined ClassLoader）。
Java虚拟机规范将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器。

#### 引导类加载器（启动类加载器，Bootstrap ClassLoader）
用C++编写的，是JVM自带的类加载器，负责Java平台核心库，用来装载核心类库。
该加载器无法直接获取。
并不继承自java.lang.ClassLoader，没有父加载器。
出于安全考虑，Bootstrap启动类加载器只加载包名为java、javax、sun等开头的类。

#### 扩展类加载器（Extension ClassLoader）
负责jre/lib/ext目录下的jar包或-D java.ext.dirs 指定目录下的jar包装入工作库。派生于ClassLoader类。
父类加载器为启动类加载器。

#### 系统类加载器（应用程序类加载器，System ClassLoader）
负责java -classpath 或 -D java.class.path所指的目录下的类与jar包装入工作库，是最常用的加载器。
派生于ClassLoader类。
父类加载器为扩展类加载器。
该类加载时程序中默认的类加载器，一般来说，Java应用的类都是由它来完成加载。

### 获取ClassLoader的方式
方式一：获取当前类的ClassLoader
clazz.getClassLoader()

方式二：获取当前线程上下文的ClassLoader
Thread.currentThread().getContextClassLoader()

方式三：获取系统的ClassLoader
ClassLoader.getSystemClassLoader()

方式四：获取调用者的ClassLoader
DriverManager.getCallerClassLoader()


## 双亲委派机制
### 工作原理
![双亲委派机制](https://github.com/BlueOzone/BlueOzone.github.io/blob/hexo/source/_posts/img/%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%9C%BA%E5%88%B6.png?raw=true)


(1)如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行；
(2)如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器；
(3)如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会自己去加载，这就是双亲委派模式。

优势：
1、避免类的重复加载；
2、保护程序安全，防止核心API被随意篡改。

### 沙箱安全机制

如果你自定义了String类，但是在加载自定义String类的时候会率先使用引导类加载器加载，而引导类加载器在加载的过程中会先加载jdk自带的文件(rt.jar包中java\lang\String.class)；此时会报错，报错信息说没有main方法，因为加载的是rt.jar包中的String类。这样就可以保证对java核心源代码的保护，避免核心类被篡改，这就是沙箱安全机制。


## 其他

在JVM中表示两个class对象是否为同一类，有两个必要条件：
1. 类的完整类名必须一致，包括包名。
2. 加载这个类的ClassLoader(指ClassLoader实例对象)必须相同。

换句话说，在JVM中，即使这两个类对象(class对象)来源同一个class文件，被同一个虚拟机所加载，但只要加载它们的ClassLoader实例对象不同，那么这两个类对象也是不相等的。

### 对类加载器的引用
JVM必须知道一个类型是由启动类加载器加载的还是由用户类加载器加载的。如果一个类型是由用户类加载器加载的，那么JVM会将这个类加载器的一个引用作为类型信息的一部分保存在方法区中。当解析一个类型到另一个类型的引用的时候，JVM需要保证这两个类型的类加载器是相同的。


### 类的主动使用和被动使用
**Java程序对类的使用方式分为：主动使用和被动使用。**
主动使用，分为七种情况：
1、创建类的实例
2、访问某个类或接口的静态变量，或者对该静态变量赋值
3、调用类的静态方法
4、反射（比如：Class.forName("com.aa.Test")）
5、初始化一个类的子类
6、Java虚拟机启动时被标明为启动类的类
7、JDK 7 开始提供的动态语言支持：
   java.lang.invoke.MethodHandle实例的解析结果

除了以上七种情况，其他使用Java类的方式都被看作是对类的被动使用，都不会导致类的初始化。