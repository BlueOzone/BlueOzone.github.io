---
title: Java-IO流
---


### 1. IO流
流是一种抽象概念，它代表了数据的无结构化传递。按照流的方式进行输入输出，数据被当成无结构的字节序或字符序列。从流中取得数据的操作称为提取操作，而向流中添加数据的操作称为插入操作。用来进行输入输出操作的流就称为IO流。换句话说，IO流就是以流的方式进行输入输出。<br>
输入input：读取外部数据（磁盘、光盘等存储设备的数据）到程序（内存）中；<br>
输出output：将程序（内存）数据输出到磁盘、光盘等存储设备中。<br>


### 2. 流的分类
按操作数据单位不同分为：字节流（8bit），字符流（16bit）<br>
按数据流的流向不同分为：输入流，输出流<br>
按流的角色的不同分为：节点流，处理流<br>

### 3. 流的体系结构
![avatar](https://note.youdao.com/yws/api/personal/file/54BA996D2B5C4A2E937A5CCE815D5933?method=download&shareKey=b1c1ad2537bf98c1df68e333cfd7534f)

### 4. 字节流-FileInputStream与FileOutputStream

* 对于文本文件(.txt,.java,.c,.cpp)，使用字符流处理
* 对于非文本文件(.jpg,.mp4,.doc,.avi,.ppt)，需要使用字节流处理
* 对于单纯的复制操作，可以使用字节流复制文本文件的内容，输出到控制台的话可能会有中文乱码的情况<br>

说明：若使用字节流处理文本文件，是会可能出现乱码的

<pre><code>
@Test
public void test(){
    FileInputStream fileInputStream = null;
    try {
        File file1 = new File("C:\\Users\\Bangsun\\Desktop\\hello.txt");
        fileInputStream = new FileInputStream(file1);
        int data;
        while ((data = fileInputStream.read()) != -1){
            System.out.print((char) data);
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (fileInputStream != null) {
            try {
                fileInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
</pre></code>


### 5. 字符流之FileReader-从磁盘中读取文件

<pre><code>
/*
说明点：
1.read()的理解：返回读入的一个字符。如果达到文件末尾，返回-1
2.异常的处理：为了保证流资源一定可以执行关闭操作。需要使用try-catch-finally处理
3.读入的文件一定要存在，否则就会报FileNotFoundException
4.read(char cbuf[])方法可以一次读指定数量的字符
 */

@Test
public void test() throws IOException {
    File file1 = new File("C:\\Users\\Bangsun\\Desktop\\hello.txt");
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


### 6. 字符流之FileWriter-把内容写进磁盘中

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


### 7. 处理流之缓冲流

* 缓冲流
<br>BufferedInputStream
<br>BufferedOutputStream
<br>BufferedReader
<br>BufferedWriter

缓冲流的作用：提高流的读取、写入的速度<br>
提高读写速度的原因：内部提供了一个缓冲区，读写的时候先将缓冲区装满后再继续。默认是8kb

<pre><code>
/*
实现非文本文件的复制
 */
@Test
public void test(){
    BufferedInputStream bufferedInputStream = null;
    BufferedOutputStream bufferedOutputStream = null;
    FileInputStream fileInputStream = null;
    FileOutputStream fileOutputStream = null;
    try {
        //1.造文件
        File file1 = new File("C:\\Users\\Bangsun\\Desktop\\hello.txt");
        File file2 = new File("C:\\Users\\Bangsun\\Desktop\\hello123.txt");
        //2.造流
        //2.1 造节点流
        fileInputStream = new FileInputStream(file1);
        fileOutputStream = new FileOutputStream(file2);
        //2.2造缓冲流
        bufferedInputStream = new BufferedInputStream(fileInputStream);
        bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
        //3.复制的细节：读取、写入
        int len;
        while ((len = bufferedInputStream.read()) != -1){
            bufferedOutputStream.write(len);
//                bufferedOutputStream.flush();
        }

    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        //4.资源关闭
        //要求：先关闭外层的流，再关闭内层的流
        //即先关闭bufferedInputStream、bufferedOutputStream，在关闭fileInputStream、fileOutputStream
        //说明：在关闭外层的流的同时，内层流也会自动的进行关闭。关于内层流的关闭，可以省略
        if (bufferedInputStream != null){
            try {
                bufferedInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (bufferedOutputStream != null){
            try {
                bufferedInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (fileInputStream != null){
            try {
                fileInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (fileOutputStream != null){
            try {
                fileOutputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
</pre></code>


### 8. 处理流之转换流

转换流(属于字符流)：<br>
InputStreamReader：将一个字节的输入流转换为字符的输入流（可以理解为编码）<br>
OnputStreamWriter：将一个字符的输出流转换为字节的输出流（可以理解为解码）

提供字节流与字符流之间的转换<br>
解码：字节、字节数组 ---> 字符串、字符数组<br>
编码：字符串、字符数组 ---> 字节、字节数组

字符集：<br>
ASCII：美国标准信息交换码,用一个字节的7位可以表示<br>
ISO8859-1：拉丁码表。欧洲码表,用一个字节的8位表示<br>
GB2312：中国的中文编码表。最多两个字节编码所有字符<br>
GBK：中国的中文编码表升级，融合了更多的中文文字符号最多两个字节编码<br>
Unicode：国际标准码，融合了目前人类使用的所有字符。为每个字符分配唯一的字符码。所有的文字都用两个字节来表示<br>
UTF-8：变长的编码方式，可用1-4个字节来表示一个字符。


### 9. 对象流
对象流跟序列化有关，放到其他文章中再介绍。


### 10. 其他流的使用
#### 10.1 标准的输入、输出流

System.in：标准的输入流，默认从键盘输入<br>
System.out：标准的输出流，默认从控制台输出<br>

System类的setIn(Inputstream is) / setOut(PrintStream in)方式重新指定输入和输出的流。

<pre><code>
public static void main(String[] args) throws IOException {
    InputStreamReader isr = new InputStreamReader(System.in);
    BufferedReader br = new BufferedReader(isr);
    while (true){
        String data = br.readLine();
        if ("exit".equals(data)){
            break;
        }
        System.out.println(data.toUpperCase());
    }
    br.close();
}
</pre></code>
#### 10.2 打印流

作用：实现将基本数据类型的数据格式转化为字符串输出<br>
分类：PrintStream和PrintWriter

#### 10.3 数据流

数据流<br>
作用：用于读取或写出基本数据类型的变量或字符串;<br>
分类：DataInputStream 和 DataOutputStream;<br>
DataInputStream读取的顺序要与DataOutputStream存储的顺序一致，否则会抛出异常。

<pre><code>
@Test
public void test(){
    DataOutputStream dataOutputStream = null;
    try {
        dataOutputStream = new DataOutputStream(new FileOutputStream("hello.txt"));
        dataOutputStream.writeInt(113);
        dataOutputStream.flush();//刷新操作，将内存中的数据写入文件
        dataOutputStream.writeUTF("甲乙丙丁");
        dataOutputStream.flush();
        dataOutputStream.writeBoolean(true);
        dataOutputStream.flush();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (dataOutputStream != null){
            try {
                dataOutputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

@Test
public void test3() {
    DataInputStream dataInputStream = null;
    try {
        dataInputStream = new DataInputStream(new FileInputStream("hello.txt"));
        System.out.println(dataInputStream.readInt());
        System.out.println(dataInputStream.readUTF());
        System.out.println(dataInputStream.readBoolean());
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (dataInputStream != null){
            try {
                dataInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
</pre></code>