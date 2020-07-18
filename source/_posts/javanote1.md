
---
title: java学习笔记
date: 2020-05-17 23:03:19
tags:

<!--more-->
---


## 1、list列表添加实例对象重复的问题
list.add(对象)这种情况时，如以下，list元素里存放的是对象的地址(可打印元素值印证)；对象的地址值是不变的，每次循环变的只是对象里面的引用值；故像这样的写法，list里面的元素就是最后一次循环里面对象的引用值。<br />
要解决这个问题，可以将对象的实例化放在循环里面(以下代码的注释)，在每次循环开始时实例化，这样保证了list列表元素存放的是每个不同的对象的地址，这样就可以找到每次设置的值了。


<pre><code>List<LogShow> logShows = new ArrayList<>();
LogShow logShow = new LogShow();
for (OperateLog map:logresp){
    //logShow = new LogShow();
    logShows.add(logShow);
}
datamap.put("list",logShows);
</code></pre>

## 2、判断为空串的问题
判断空字符串应该是这个比较基础的问题，有时候前端传值过来的时候可能是这样<code>string=</code>，也可能是这样<code>string=null</code>,这两种的实际情况都应该是空串，所以应该需要像以下这样比较标准：
<pre><code>if (string!=null && string.length()!=0)
</code></pre>
特殊情况时，还有另一种判断方法：
<pre><code>if ("0".equals(logParams.getOperType()))
</code></pre>
这样可省去上一步的判断。
## 3、接收数据格式的问题
一般来说，为了数据安全，对外的接口一般是另外定义一个DO对象来接收，不用PO对象来接收。

4、Date数据格式，有三个类，sql.date仅包含时间
5、数据类型转换、解析的问题，不知道什么样的格式怎样去解析
6、报错JSONArray cannot be cast to java.lang.String] with root如何解决
7、java占位符替换问题
## 4、Date类说明
java.util.Date日期格式为：年月日时分秒 <br />
java.sql.Date日期格式为：年月日<br />
java.sql.Time日期格式为：时分秒 <br />
java.sql.Timestamp日期格式为：年月日时分秒纳秒（毫微秒）<br />

java.util.Date 就是在除了SQL语句的情况下面使用；<br />
java.sql.Date 是针对SQL语句使用的，new java.sql.Date(new java.util.Date().getTime()，它只包含日期而没有时间部分。<br />
## 5、数据类型转换、解析的问题
需要熟知各种常用的数据格式是怎么样的，比如json、list等，才会知道用什么方法去解析；
json用json.parse方法解析。
## 6、报错JSONArray cannot be cast to java.lang.String with root如何解决
如下，将需要解析的数据先进行JSON.toJSONString，才不会报这个错：
<pre><code>List<Map<String,Object>> mapList = (List<Map<String,Object>>) JSON.parse(JSON.toJSONString(respmap.get("series")));
</code></pre>
## 7、java占位符替换问题
背景：echarts的option存放在数据库里，其中的数据未填充，需要通过替换的方式来填充；但是用{0}类似的方法替换时，由于option格式中也有{}中括号，所以导致这种替换方式不能用；需要用以下的方式来替换，前面的参数为需要替换的参数，在option里面设置好就行<pre><code>ss.replace("replace3", (String) JSON.toJSONString(mapList.get(1).get("data")));</pre><code>

