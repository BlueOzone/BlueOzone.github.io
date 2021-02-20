---
title: Java-泛型
date: 2020-05-18 20:52:11
---


### 1、泛型类、泛型接口
系。


### 2、泛型方法
1、泛型方法：在方法中出现了泛型的结构，泛型参数与类的泛型参数没有任何关系；<br>
2、泛型方法所属的类是不是泛型类都没有关系；<br>
3、泛型方法可以声明为静态的。原因：泛型参数是在调用泛型方法时确定的，并非在实例化类时确定。<br>
泛型方法举例：
<pre><code>public <E> List<E> method(E[] arr){
	ArrayList<E> list = new ArrayList<E>();
}
</code></pre>
其中`<E>`表明这是个泛型方法，确实的话编译器会认为E是一个自定义的类。

### 3、泛型
1、泛型在继承方面的体现：<br>
虽然类A是类B的父类，但是`G<A>`和`G<B>`二者不具备子父类关系，二者是并列关系；<br>
补充：类A是类B的父类，`A<G>`是`B<G>`的父类。


2、通配符的使用
通配符：?

类A是类B的父类，`G<A>`和`G<B>`是没有关系的，二者共同的父类是：`G<?>`




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
