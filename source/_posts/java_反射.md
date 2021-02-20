---
title: Java-反射
categories: Java
tags: Java进阶
toc: true
---

### Java反射机制概述
Reflection(反射)是被视为动态语言的关键，反射机制允许程序在执行期借助于Reflection API取得任何类的内部消息，并能直接操作任意对象的内部属性及方法。
<!--more-->

正常方式获取对象：引入需要的"包类"名称---通过new实例化---取得实例化对象
反射方式获取对象：实例化对象---getClass()方法---得到完整的"包类"名称

**Java反射机制提供的功能**

*	在运行时判断任意一个对象所属的类
*	在运行时构造任意一个类的对象
*	在运行时判断任意一个类所具有的成员变量和方法
*	在运行时获取泛型信息
*	在运行时调用任意一个对象的成员变量和方法
*	在运行时处理注解
*	生成动态代理

**反射相关的主要API**

*	java.lang.Class：代表一个类(运行时类)
*	java.lang.reflect.Method：代表类的方法
*	java.lang.reflect.Field：代表类的成员变量
*	java.lang.reflect.Constructor：代表类的构造器

说明1：通过直接new的方式或反射的方式都可以调用公共的结构，开发中到底用哪个？
建议：直接new的方式的方式。

说明2：反射机制与面向对象中的封装性是不是矛盾的？如何看待两个技术？
不矛盾。

说明3：什么时候会使用反射的方式？
反射的特征：动态性


### 理解Class类并获取Class实例

```
/*
关于java.lang.Class类的理解
1.类的加载过程：
程序经过javac.exe命令以后，会生成一个或多个字节码文件(.class结尾)。
接着我们使用java.exe命令对某个字节码文件进行解释运行。相当于将某个字节码文件加载到内存中。
此过程就称为类的加载。加载到内存中的类，我们就称为运行时类，此运行时类，就作为Class的一个实例。

2.换句话说，Class的实例就对应着一个运行时类。
3.加载到内存中的运行时类，会缓存一定时间。在此时间之内，我们可以通过不同的方式来获取此运行时类。

万事万物皆对象
 */

//获取Class的实例的方式
@Test
public void test3() throws ClassNotFoundException {
    //方式一：调用运行时类的属性：.class
    Class<Person> clazz1 = Person.class;
    System.out.println(clazz1);

    //方式二：通过运行时类的对象，调用getClass()
    Person person = new Person();
    Class<? extends Person> clazz2 = person.getClass();
    System.out.println(clazz2);

    //用的较多，体现了反射的动态性
    //方式三：调用Class的静态方法：forName(String classPath)
    Class<?> clazz3 = Class.forName("reflection.Person");
    System.out.println(clazz3);

    System.out.println(clazz1 == clazz2);
    System.out.println(clazz1 == clazz3);

    //方式四：使用类的加载器：ClassLoader(了解)
    ClassLoader classLoader = ReflectionTest.class.getClassLoader();
    Class<?> clazz4 = classLoader.loadClass("reflection.Person");
    System.out.println(clazz4);
}
```



### 类的加载与ClassLoader的理解
**类的加载过程:**
当程序主动使用某个类时，如果该类还未被加载到内存中，则系统会通过如下三个步骤来对该类进行初始化。
![avatar](http://note.youdao.com/yws/public/resource/5f6100ed4f4214c3cb0465dfb3852db5/xmlnote/DDF1330A1CF94922A94D73793D8E13C4/12)

**类加载器的作用:**

*	类加载器的作用：将class文件字节码内容加载到内存中，并将这些静态数据转换成方法去的运行时数据结构，然后在堆中生成一个代表这个类的java.class.Class对象，作为方法区中类数据的访问入口。
*	类缓存：标准的JavaSE类加载器可以按要求查找类，但一旦某个类被加载到类加载器中，它将维持加载（缓存）一段时间。不过JVM垃圾回收机制可以回收这些Class对象。

**了解ClassLoader:**
类加载器作用是用来把类(Class)装载进内存的。JVM规范定义了如下类型的类加载器。
![avatar](http://note.youdao.com/yws/public/resource/eaf6e5b7723df5ab47ad98e0812a75f5/xmlnote/D56118B9361347F1946889FCA2638353/16)

```
@Test
public void test(){
    //对应自定义类，使用系统类加载器进行加载
    ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
    System.out.println(classLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2

    //调用系统类加载器的getParent()：获取扩展类的加载器
    ClassLoader classLoader1 = classLoader.getParent();
    System.out.println(classLoader1);//sun.misc.Launcher$ExtClassLoader@7229724f


    //调用扩展类的加载器getParent()：无法获取引导类加载器
    //引导类加载器主要负责加载Java的核心类库，无法加载自定义类。
    ClassLoader classLoader2 = classLoader1.getParent();
    System.out.println(classLoader2);//null

}
```


### 创建运行时类的对象
```
/*
newInstance():调用此方法，创建对应的运行时类的对象。内部调用了运行时类的空参构造器。

要想此方法正常的创建运行时类的对象，要求：
1.运行时类必须提供空参的构造器；
2.空参的构造器的访问权限得够。通常，设置为public。

在javabean中要求提供一个public的空参构造器。原因：
1.便于通过反射，创建运行时类的对象；
2.便于子类继承此运行时类时，默认调用super()时，保证父类有此构造器。

 */

@Test
public void test() throws IllegalAccessException, InstantiationException {
    Class<Person> clazz = Person.class;
    Person obj = clazz.newInstance();
    System.out.println(obj);
}
```


### 获取运行时类的完整结构

```
@Test
public void test2() throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException, NoSuchFieldException {

    //通过反射，创建Person类的对象
    Class clazz = Person.class;
    Constructor cons = clazz.getConstructor(String.class,int.class);
    Object obj = cons.newInstance("Tom", 12);
    Person p = (Person) obj;
    System.out.println(p.toString());
    //2.通过反射，调用对象指定的属性、方法
    //调用属性
    Field age = clazz.getDeclaredField("age");
    age.set(p,10);
    System.out.println(p.toString());

    //调用方法
    Method show = clazz.getDeclaredMethod("show");
    show.invoke(p);

    //通过反射，可以调用Person类的私有结构。比如：私有的构造器、方法、属性
    //调用私有属性
    Field name = clazz.getDeclaredField("name");
    //保证当前属性是可访问的
    name.setAccessible(true);
    name.set(p,"newTom");
    System.out.println(p.toString());

    //调用私有方法
    //getDeclaredMethod()：参数1：指明获取的方法的名称    参数2：指明获取的方法的形参列表
    Method showNation = clazz.getDeclaredMethod("showNation", String.class);
    //保证当前方法是可访问的
    showNation.setAccessible(true);
    //调用方法的invoke():参数1：方法的调用者  参数2：给方法形参赋值的实参
    showNation.invoke(p,"China");

    //调用私有构造器
    Constructor cons1 = clazz.getDeclaredConstructor(String.class);
    cons1.setAccessible(true);
    Object newJerry = cons1.newInstance("newJerry");
    Person p2 = (Person) newJerry;
    System.out.println(p2.toString());
}
```



### 调用运行时类的指定结构
获取属性结构：
```
@Test
public void test1(){
    Class clazz = Person.class;
    //获取属性结构
    //getFields():获取当前运行时类及其父类中声明为public访问权限的属性
    Field[] fields = clazz.getFields();
    for (Field f : fields){
        System.out.println(f);
    }
    System.out.println();
    //getDeclaredFields:获取当前运行时类中声明的所有属性。（不包含父类中声明的属性）
    Field[] declaredFields = clazz.getDeclaredFields();
    for (Field f : declaredFields){
        System.out.println(f);
    }
}
@Test
public void test2(){
    Class clazz = Person.class;

    Field[] declaredFields = clazz.getDeclaredFields();
    for (Field f : declaredFields){
        //获取属性的权限修饰符
        System.out.println(Modifier.toString(f.getModifiers()));
        //获取属性的数据类型
        System.out.println(f.getType().getName());
        //获取变量名
        System.out.println(f.getName());
    }
}
```


获取方法结构:
```
@Test
public void test1(){
    Class clazz = Person.class;

    //getMethods()：获取当前运行时类及其所有父类中声明为public权限的方法
    Method[] methods = clazz.getMethods();
    for (Method method : methods){
        System.out.println(method);
    }
    System.out.println();

    //getDeclaredMethods():获取当前运行时类中声明的所有方法。（不包含父类中声明的方法）
    Method[] declaredMethods = clazz.getDeclaredMethods();
    for (Method method : declaredMethods){
        System.out.println(method);
    }
}
```

### 反射的应用：动态代理

详情请查看Java-代理模式这篇文章















