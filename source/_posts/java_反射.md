---
title: Java-反射
date: 2020-05-28 20:22:41
---


### 1、IO流
流是一种抽象概念，它代表了数据的无结构化传递。按照流的方式进行输入输出，数据被当成无结构的字节序或字符序列。从流中取得数据的操作称为提取操作，而向流中添加数据的操作称为插入操作。用来进行输入输出操作的流就称为IO流。换句话说，IO流就是以流的方式进行输入输出。<br>
输入input：读取外部数据（磁盘、光盘等存储设备的数据）到程序（内存）中；<br>
输出output：将程序（内存）数据输出到磁盘、光盘等存储设备中。<br>


### 2、流的分类
按操作数据单位不同分为：字节流（8bit），字符流（16bit）<br>
按数据流的流向不同分为：输入流，输出流<br>
按流的角色的不同分为：节点流，处理流<br>

### 3、流的体系结构
抽象基类&emsp;&emsp;&emsp;节点流（或文件流）&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;缓冲流（处理流的一种）<br>
InputStream&emsp;&emsp;FileInputStream(read(byte[] buffer))                &emsp;&emsp;&emsp;BufferedInputStream(read(byte[] buffer))<br>
OutputStream&emsp;&emsp;FileOutputStream(write(byte[] buffer,0,len))        BufferedOutputStream(write(byte[] buffer,0,len))<br>
Reader&emsp;&emsp;&emsp;&emsp;FileReader(read(char[] cbuf))                       &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;BufferedReader(read(char[] cbuf)|readline())<br>
Writer&emsp;&emsp;&emsp;&emsp;&emsp;FileWriter(write(char[] cbuf,0,len))&emsp;&emsp;&emsp;&emsp;BufferedWriter(write(char[] cbuf,0,len))<br>




### 4、使用FileInputStream从磁盘中读取文件

<pre><code>
/*
说明点：
1.read()的理解：返回读入的一个字符。如果达到文件末尾，返回-1
2.异常的处理：为了保证流资源一定可以执行关闭操作。需要使用try-catch-finally处理
3.读入的文件一定要存在，否则就会报FileNotFoundException
 */

@Test
public void test() throws IOException {
    File file1 = new File("C:\\Users\\Bangsun\\Desktop\\hello.txt");
    FileInputStream fileInputStream = new FileInputStream(file1);

    FileReader fileReader = new FileReader(file1);

    int data = fileReader.read();
    List<String> list = new ArrayList<>();
    while (data != -1){
        list.add(String.valueOf((char) data));
        data = fileReader.read();
    }

    fileReader.close();
    System.out.println(list);

}
</pre></code>


### 5、使用FileWriter把内容写进磁盘中

<pre><code>
/*
FileWriter
说明：
1.输出操作，对应的File可以不存在，并不会报异常；
        如果不存在，在输出的过程中，会自动创建此文件；
        如果存在:
            若流使用的构造器是：new FileWriter(file1,false)|new FileWriter(file1)：会对原有文件进行覆盖；
            若流使用的构造器是：new FileWriter(file1,true)：不会对原有文件进行覆盖，而是在原有文件基础上追加内容。
 */
@Test
public void test3(){

    FileWriter fileWriter = null;
    try {
        File file1 = new File("C:\\Users\\Bangsun\\Desktop\\hello.txt");

        fileWriter = new FileWriter(file1,true);

        fileWriter.write("ssssssssssstest");
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            fileWriter.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
</pre></code>




### 6、处理流之一：缓冲流的使用

* 1.缓冲流
* BufferedInputStream
* BufferedOutputStream
* BufferedReader
* BufferedWriter

2.作用：提高流的读取、写入的速度<br>
提高读写速度的原因：内部提供了一个缓冲区，读写的时候先将缓冲区装满后再继续。默认是8kb



### 7、FileInputStream与FileOutputStream

* 结论：
* 1.对于文本文件(.txt,.java,.c,.cpp)，使用字符流处理
* 2.对于非文本文件(.jpg,.mp4,.doc,.avi,.ppt)，需要使用字节流处理
*
* 对于单纯的复制操作，可以使用字节流复制文本文件的内容，输出到控制台的话可能会有中文乱码的情况
* 说明：
* 若使用字节流处理文本文件，是会可能出现乱码的

### 8、处理流之二：转换流的使用

* 处理流之二：转换流的使用
* 1.转换流：属于字符流
*   InputStreamReader：将一个字节的输入流转换为字符的输入流（可以理解为编码）
*   OnputStreamWriter：将一个字符的输出流转换为字节的输出流（可以理解为解码）
*
* 2.提供字节流与字符流之间的转换
*
* 3.解码：字节、字节数组 ---> 字符串、字符数组
*   编码：字符串、字符数组 ---> 字节、字节数组
*
* 4.字符集
* ASCII：美国标准信息交换码
*       用一个字节的7位可以表示
* ISO8859-1：拉丁码表。欧洲码表
*       用一个字节的8位表示
* GB2312：中国的中文编码表。最多两个字节编码所有字符
* GBK：中国的中文编码表升级，融合了更多的中文文字符号最多两个字节编码
* Unicode：国际标准码，融合了目前人类使用的所有字符。为每个字符分配唯一的字符码。所有的文字都用两个字节来表示
* UTF-8：变长的编码方式，可用1-4个字节来表示一个字符。



### 9、其他流的使用
* 1.标准的输入、输出流
* 2.打印流
* 3.数据流

1.标准的输入、输出流
1.1
System.in：标准的输入流，默认从键盘输入
System.out：标准的输出流，默认从控制台输出
1.2
System类的setIn(Inputstream is) / setOut(PrintStream in)方式重新指定输入和输出的流。

<br>
3.数据流<br>
3.1 DataInputStream 和 DataOutputStream;<br>
3.2 作用：用于读取或写出基本数据类型的变量或字符串。<br>