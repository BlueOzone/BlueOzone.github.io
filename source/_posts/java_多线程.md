---
title: Java-多线程
categories: Java
tags: Java进阶
toc: true
---


### 基本概念
**程序：**由特定语言编写，完成某项功能的指令集合。
**进程：**程序的一次执行过程，或者是正在运行的程序；是一个动态的过程：有自身的产生、存在和消亡的过程。<!--more-->
**线程：**可以理解进程的执行单位，是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位；
Java中，每个线程拥有独立的运行栈和程序计数器(pc)。
**多线程：**Java的main方法是一个线程，当你想main方法与别的方法同时执行时，就需要创建线程来执行对应的方法；
简而言之，多线程就是Java中多个线程共同运行，分别或共同完成特定的功能，充分利用CPU。
**线程优先级：**线程存在1-10的优先级。
**线程的生命周期（几种状态）**：
新建 -- 就绪 -- 运行 -- 阻塞 -- 死亡

### 为什么需要多线程和多线程的优点
多线程在特定场景下可以提高系统的响应速度，完成一些高并发场景下的需求。

**多线程的优点：**
1、提高应用程序的响应。图像化页面情况下可增强用户的体验；
2、提高计算机系统CPU的利用率；
3、改善程序结构。将既长又复杂的进程分为多个线程，独立运行，利于理解和修改。

### 多线程创建方式一：继承Thread类
1. 创建一个继承于Thread类的子类
2. 重写Thread类的run() --> 将此线程执行的操作声明在run()中
3. 创建Thread类的子类的对象
4. 通过此对象调用start()

```
public class Demo {
    public static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("MyThread");
        }
    }

    public static void main(String[] args) {
        Thread myThread = new MyThread();
        myThread.start();
    }
}
```
注意要调用start()方法后，该线程才算启动！
我们在程序里面调用了start()方法后，虚拟机会先为我们创建一个线程，然后等到这个线程第一次得到时间片时再调用run()方法。
注意不可多次调用start()方法。在第一次调用start()方法后，再次调用start()方法会抛出异常。

### 多线程创建方式二：实现Runanle接口
1. 创建一个实现了Runnable接口的类
2. 实现类去实现Runnable中的抽象方法：run()
3. 创建实现类的对象
4. 将此对象作为参数传递到Thread类的构造器中，创建Thread类的对象
5. 通过Thread类的对象调用start()方法

先看下Runnable接口：
```
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```
可以看到Runnable是一个函数式接口，这意味着我们可以使用Java 8的函数式编程来简化代码。
示例代码：
```
public class Demo {
    public static class MyThread implements Runnable {
        @Override
        public void run() {
            System.out.println("MyThread");
        }
    }

    public static void main(String[] args) {
        new Thread(new MyThread()).start();
        // Java 8 函数式编程，可以省略MyThread类
        new Thread(() -> {
            System.out.println("Java 8 匿名内部类");
        }).start();
    }
}
```

### 多线程创建方式三：实现Callable接口
示例代码：
```
public class CallableTest {
    public static void main(String[] args) {
        MyCallable myCallable = new MyCallable();
        FutureTask futureTask = new FutureTask(myCallable);
        new Thread(futureTask).start();

        try {
            Object sum = futureTask.get();
            System.out.println(sum);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}

class MyCallable implements Callable{

    @Override
    public Object call() throws Exception {
        int sum = 0;
        for (int i = 0; i <= 100; i++) {
            if (i % 2 ==0){
                System.out.println(i);
                sum +=i;
            }
            Thread.sleep(100);
        }
        return sum;
    }
}
```

### 多线程创建方式四：使用线程池
示例代码：
```
public class ThreadPoolTest {
    public static void main(String[] args) {
        //1、提供指定线程数量的线程池
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        ThreadPoolExecutor executorService1 = (ThreadPoolExecutor)executorService;
        //设置线程池的属性
        System.out.println(executorService.getClass());
//        executorService1.setCorePoolSize(10);

        //2、执行指定的线程的操作。需要提供实现Runable接口或Callable接口的实现类的对象
        executorService.execute(new NumberThread());//适合使用于Runable
        for (int i = 0; i < 3; i++) {
            executorService.execute(new NumberThread());
        }
//        executorService.execute(new NumberThread1());
//        executorService.submit();//适合使用于Callable
        //3、关闭线程池
        executorService.shutdown();
    }
}

class NumberThread implements Runnable{
    @Override
    public void run() {
        /*for (int i = 0; i <=100 ; i++) {
            if (i % 2 == 0){
                System.out.println(Thread.currentThread().getName() + "：" + i);
            }
        }*/
        while (true){
            System.out.println(Thread.currentThread().getName() + "：");
            System.out.println(">>>>>>>>");
        }
    }
}
```
### Thread类的几个常用方法
currentThread()：静态方法，返回对当前正在执行的线程对象的引用；
start()：开始执行线程的方法，java虚拟机会调用线程内的run()方法；
yield()：yield在英语里有放弃的意思，同样，这里的yield()指的是当前线程愿意让出对当前处理器的占用。这里需要注意的是，就算当前线程调用了yield()方法，程序在调度的时候，也还有可能继续运行这个线程的；
sleep()：静态方法，使当前线程睡眠一段时间；
join()：使当前线程等待另一个线程执行完毕之后再继续执行，内部调用的是Object类的wait方法实现的；
run()：通常需要重写thread类中的此方法，将创建的线程要执行的操作写在此方法中；
getName()：获取线程名字；
setName()：设置线程名字；
stop()：已过时；强制结束线程的生命周期；
isAlive()：判断当前线程是否存活。

### Thread类和Runnable接口方式的比较
对于这两种方法，在开发中优先选择：实现Runnable接口的方式；
原因：
1. Runnable接口实现的方式没有类的单继承性的限制；
2. Runnable接口实现的方式更适合来处理多个线程由共享数据的情况(继承方式的话共享数据需要声明为static，有时候还需要额外的处理)。

所以，我们通常**优先使用“实现Runnable接口”**这种方式来自定义线程类。

### 线程安全问题
#### 什么是线程安全？
当多个线程访问同个一资源的时候，就会出现线程安全问题。比如售票系统，多个线程同时访问库存，如果没有做到线程的同步就很有可能会出现超卖（比如卖了-1这个号的票），或者重复卖（票号为2的卖了多次）的问题。
#### 线程安全解决方式一：同步代码块
同步代码块，即有synchronized关键字修饰的语句块。被该关键字修饰的语句块会自动被加上内置锁，从而实现同步。
同步是一种高开销的操作，因此应该尽量减少同步的内容。通常没有必要同步整个方法，使用synchronized代码块同步关键代码即可。
```
synchronized (同步监视器){
	//需要被同步的代码(操作共享数据的代码)
	//共享数据：多个线程共同操作的变量
	//同步监视器：俗称为锁。任何一个类的对象都可以充当锁。多个线程必须共用一把锁。
}
```
示例代码：
```
public class RunnableImpl implements Runnable {
    int ticket = 10;
    Object obj = new Object();

    @Override
    public void run() {
        while (true) {
            synchronized (obj) {    //同步代码块
                if (ticket > 0) {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("saling : " + ticket + " ticket.");
                    ticket--;
                }
                else{
                    break;
                }
            }
        }
    }
}
```
#### 线程安全解决方式二：同步方法
同步方法，即有synchronized关键字修饰的方法。 由于java的每个对象都有一个内置锁，当用此关键字修饰方法时,内置锁会保护整个方法。在调用该方法前，需要获得内置锁，否则就处于阻塞状态。
synchronized关键字也可以修饰静态方法，此时如果调用该静态方法，将会锁住整个类。
示例代码：
```
public class RunnableImpl implements Runnable {
    int ticket = 10;
    @Override
    public void run() {
        while (true) {
            int iRet = payTicket(); //调用同步方法
            if (iRet == 0) break;
        }
    }
    public synchronized int payTicket() {
        if (ticket > 0) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " saling : " + ticket + " ticket.");
            ticket--;
            return 1;
        } else {
            return 0;
        }
    }
}

```
#### 线程安全解决方式三：lock 锁机制
jdk开始，Java提供了更强大的线程同步机制————通过显示定义同步锁对象来实现同步。同步锁使用Lock对象来充当。
java.util.concurrent.locks.Lock 接口是控制多个线程对共享资源进行访问的工具。
ReentrantLock 类实现了Lock，它拥有与synchronized相同的并发性和内存语义，在实现线程安全的控制中，比较常用的是ReentrantLock，可以显示加锁，释放锁。
示例代码：
```
public class LockTest {
    public static void main(String[] args) {
        Window window = new Window();

        Thread t1 = new Thread(window);
        Thread t2 = new Thread(window);
        Thread t3 = new Thread(window);

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();
    }
}

class Window implements Runnable{
    private int ticket = 100;
    //1.实例化ReentrantLock
    private ReentrantLock lock = new ReentrantLock();

    @Override
    public void run() {
        while (true){
            try {
                //2.调用锁定方法：lock()
                lock.lock();
                if (ticket > 0){
                    System.out.println(Thread.currentThread().getName() + "：售票，票号为：" + ticket);
                    ticket--;
                }else {
                    break;
                }
            }finally {
                //3.调用解锁方法：unlock()
                lock.unlock();
            }
        }

    }
}
```
#### synchronized与Lock的对比
1、synchronized与Lock都可以解决线程安全问题；
2、Lock是显示锁（手动开启和关闭锁，不要忘记关闭锁），synchronized是隐式锁，出了作用域自动释放。
3、Lock只有代码块锁，synchronized有代码块锁和方法锁。
4、使用Lock锁，JVM将花费较少的时间来调度线程，性能更好。并且具有更好的扩展性（因为提供了更多的子类）。

**优先使用顺序：**
Lock——>同步代码块（已经进入了方法体，分配了相应资源）——>同步方法（在方法体之外）

### 线程的死锁问题
**死锁：**不同的线程分别占用对方的需要的同步资源不放弃，都在等待对方放弃自己需要的同步资源，就形成了线程的死锁。
出现死锁后，不会出现异常，不会出现提示，只是所有的线程都处于阻塞状态，无法继续处理其他事情。

**解决方法：**
1、专门的算法、原则；
2、尽量减少同步资源的定义；
3、尽量避免嵌套同步。

### 线程安全的单例模式
饿汉式原本就是线程安全的；懒汉式需要使用同步代码块的方式去保证线程安全。
```
public class SingleTest {
	//方法一：饿汉式
    /*private static Person person = new Person();
    public static Person getPersion(){
        return person;
    }*/

    private static Person person2 = null;
    //方法二：懒汉式
    public static Person getPerson2(){
        if (person2 == null){
            synchronized (SingleTest.class) {
                if (person2 == null){
                    person2 = new Person();
                }
            }
        }
        return person2;
    }
}
```

