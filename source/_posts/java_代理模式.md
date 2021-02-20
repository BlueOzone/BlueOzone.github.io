---
title: 细说Java的代理模式
date: 2020-05-28 20:22:41
categories: Java
tags: Java进阶
toc: true
---


### 代理模式
代理(Proxy)是一种设计模式，提供了对目标对象另外的访问方式；即通过代理对象访问目标对象。这样做的好处是：可以在目标对象实现的基础上，增强额外的功能操作，即扩展目标对象的功能。<!--more-->
这里使用到编程中的一个思想：不要随意去修改别人已经写好的代码或者方法，如果需改修改，可以通过代理的方式来扩展该方法。

### 静态代理
静态代理特点：代理类和被代理类在编译期间，就确定下来了。
示例代码：
```
interface ClothFactory{
    void produceCloth();
}

//代理类
class ProxyClothFactory implements ClothFactory{
    private ClothFactory factory;

    public ProxyClothFactory(ClothFactory factory){
        this.factory = factory;
    }

    @Override
    public void produceCloth() {
        System.out.println("代理类做一些准备工作");
        factory.produceCloth();
        System.out.println("代理类做一些收尾工作");
    }
}

//被代理类
class NikeClothFactory implements ClothFactory{
    @Override
    public void produceCloth() {
        System.out.println("Nike工厂生产一批衣服");
    }
}

public class StaticProxyTest {
    public static void main(String[] args) {
        NikeClothFactory nikeClothFactory = new NikeClothFactory();
        ProxyClothFactory proxyClothFactory = new ProxyClothFactory(nikeClothFactory);
        proxyClothFactory.produceCloth();
        /*
        输出：
        代理类做一些准备工作
        Nike工厂生产一批衣服
        代理类做一些收尾工作
         */
    }
}
```

### 动态代理
动态代理是指客户通过代理类来调用其他对象的方法，并且是在程序运行时根据需要动态创建目标类的代理对象。
**动态代理相比于静态代理的优点：**
抽象角色中（接口）声明的所有方法都被转移到调用处理器一个集中的方法中处理，这样，我们可以更加灵活和统一的处理众多的方法。

在本例中使用了**基于JDK的动态代理**；
JDK的动态代理主要涉及到java.lang.reflect包中的两个类：Proxy和InvocationHandler。
其中 InvocationHandler是一个接口，可以通过实现该接口定义横切逻辑，在并通过反射机制调用目标类的代码，动态将横切逻辑和业务逻辑编织在一起。
而Proxy为InvocationHandler实现类动态创建一个符合某一接口的代理实例。

实现代理时，Proxy类会使用到newProxyInstance方法,但是该方法需要接收三个参数,完整的写法是：

```
static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,InvocationHandler h )
```

注意该方法是在Proxy类中是静态方法,且接收的三个参数依次为：

`ClassLoader loader` ：指定当前目标对象使用类加载器,获取加载器的方法是固定的；
`Class<?>[] interfaces` ：目标对象实现的接口的类型,使用泛型方式确认类型；
`InvocationHandler h` ：事件处理,执行目标对象的方法时,会触发事件处理器的方法,会把当前执行目标对象的方法作为参数传入。

实现代理时，需要用到InvocationHandler接口，该接口定义了一个 invoke(Object proxy, Method method, Object[] args)的方法：
`proxy` ：是代理实例，一般不会用到；
`method` ：是代理实例上的方法，通过它可以发起对目标类的反射调用；
`args` ：是通过代理类传入的方法参数，在反射调用时使用。

示例代码：
```
interface Human{
    String getBelief();
    void eat(String food);
}

//被代理类
class SuperMan implements Human{
    @Override
    public String getBelief() {
        return "I believe I can fly";
    }

    @Override
    public void eat(String food) {
        System.out.println("我喜欢吃" + food);
    }
}

class ProxyFactory{
    //调用此方法，返回一个代理类的对象，解决问题一
    public static Object getProxyInstance(Object obj){//obj:被代理类的对象
        MyInvocationHandler handler = new MyInvocationHandler();
        handler.bind(obj);

        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), handler);
    }
}

class MyInvocationHandler implements InvocationHandler{
    
    private Object obj;//需要使用被代理类的对象进行赋值

    public void bind(Object obj){
        this.obj = obj;
    }

    //当我们通过代理类的对象，调用方法a时，同时会自动的调用如下的方法：invoke()
    //将被代理类要执行的方法a的功能，声明在invoke()中
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //method:即为代理类对象调用的方法，此方法也就作为了被代理类对象要调用的方法
        //obj:被代理类的对象
		//通过反射机制调用目标对象的方法
        Object returnValue = method.invoke(obj, args);
        //此返回值就是被代理类对象调用相应方法的返回值
        return returnValue;
    }
}

public class ProxyTest {
    public static void main(String[] args) {
        SuperMan superMan = new SuperMan();
        //注意，这里的Human不能写成SuperMan；因为动态代理会根据你实现的接口去生成对应的代理对象，不保证一定是SuperMan类型的
        Human proxyInstance = (Human) ProxyFactory.getProxyInstance(superMan);
        //当通过代理类对象调用方法时，会自动的调用被代理类中同名的方法
        String belief = proxyInstance.getBelief();
        System.out.println(belief);
        proxyInstance.eat("apple");
    }
}
```
输出如下：
```
I believe I can fly
我喜欢吃appale
```

### 基于CGLIB的动态代理

使用JDK创建代理有一个限制，即JDK动态代理只能为接口创建代理，这一点我们从Proxy的接口方法`newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)`就看得很清楚，第三个入参interfaces就是为代理实例指定的实现接口。

现在的问题是：对于没有通过接口定义业务方法的类，如何动态创建代理实例呢？JDK的代理技术显然已经黔驴技穷，CGLib作为一个替代者，填补了这个空缺。

CGLib实现动态代理时，需要实现MethodInterceptor接口，需要重写intercept方法，这里先来解释一下该方法的参数含义：

`@param1 obj` ：代理对象本身
`@param2 method` ： 被拦截的方法对象
`@param3 args`：方法调用入参
`@param4 proxy`：用于调用被拦截方法的方法代理对象

示例代码：

```
//被代理类
class SuperMan{
    public void getBelief() {
        System.out.println("I believe I can fly");
    }

    public void eat() {
        System.out.println("我喜欢吃");
    }
}

class ProxyFactory implements MethodInterceptor {
    private Object obj;//需要使用被代理类的对象进行赋值

    public ProxyFactory(Object obj) {
        this.obj = obj;
    }

    //给目标对象创建一个代理对象
    public Object getProxyInstance(){
        //Enhancer可以用来为无接口的类创建代理；它会根据某个给定的类创建子类，并且所有非final的方法都带有回调钩子。
        //1.工具类
        Enhancer en = new Enhancer();
        //2.设置父类
        en.setSuperclass(obj.getClass());
        //3.设置回调对象为本身
        en.setCallback(this);
        //4.通过字节码技术动态创建子类实例
        return en.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("开始...");
        //写法一：
        /*
        通过反射机制调用目标对象的方法
        Object invoke = method.invoke(obj, objects);
        */
        //写法二：（正确）
        /*
        通过FastClass的机制来实现对被拦截方法的调用
         */
        Object returnValue = methodProxy.invokeSuper(o, objects);
        System.out.println("结束...");
        return returnValue;
    }
}

public class CGLIBProxyTest {
    public static void main(String[] args) {
        ProxyFactory proxyFactory = new ProxyFactory(new SuperMan());
        SuperMan proxyInstance = (SuperMan) proxyFactory.getProxyInstance();

        proxyInstance.getBelief();
        proxyInstance.eat();
    }
}
```

输出如下：

```
开始...
I believe I can fly
结束...
开始...
我喜欢吃
结束...
```

**总结：**

intercept方法因为具有MethodProxy proxy参数的原因，不再需要代理类的引用对象了，直接通过proxy对象访问被代理对象的方法(这种方式更快)。
当然也可以通过反射机制，通过method引用实例 `Object result = method.invoke(target, args)` 的形式反射调用被代理类方法。

Jdk动态代理的拦截对象是通过反射的机制来调用被拦截方法的，反射的效率比较低，所以cglib采用了FastClass的机制来实现对被拦截方法的调用。FastClass机制就是对一个类的方法建立索引，通过索引来直接调用相应的方法。

FastClass使用：动态生成一个继承FastClass的类，并向类中写入委托对象，直接调用委托对象的方法。
FastClass逻辑：在继承FastClass的动态类中，根据方法签名(方法名字+方法参数)得到方法索引，根据方法索引调用目标对象方法。
FastClass优点：FastClass用于代替Java反射，避免了反射造成调用慢的问题。


### 基于JDK和基于CGLIB的动态代理的区别
**JDK的动态代理必须具备四个条件：**

* 目标接口
* 目标类
* 拦截器（处理器）
* 代理类

1、因为利用JDKProxy生成的代理类实现了接口，所以目标类中所有的方法在代理类中都有。
2、生成的代理类的所有的方法都拦截了目标类的所有的方法。而拦截器（处理器）中invoke方法的内容正好就是代理类的各个方法的组成体。
3、利用JDKProxy方式必须有接口的存在。
4、invoke方法中的三个参数可以访问目标类的被调用方法的API、被调用方法的参数、被调用方法的返回类型。

**基于CGLIB的动态代理：**
1、 CGlib是一个强大的,高性能,高质量的Code生成类库。它可以在运行期扩展Java类与实现Java接口。

2、 用CGlib生成代理类是目标类的子类。

3、 用CGlib生成代理类不需要接口。

4、 用CGLib生成的代理类重写了父类的各个方法。

5、 拦截器中的intercept方法内容正好就是代理类中的方法体。


### 动态代理总结
JDK动态代理所创建的代理对象，在JDK 1.3下，性能强差人意。虽然在高版本的JDK中，动态代理对象的性能得到了很大的提高，但是有研究表明：

CGLib所创建的动态代理对象的性能依旧比JDK的所创建的代理对象的性能高不少（大概10倍）。

而CGLib在创建代理对象时性能却比JDK动态代理慢很多（大概8倍），

所以对于singleton的代理对象或者具有实例池的代理，因为不需要频繁创建代理对象，所以比较适合用CGLib动态代理技术，反之适合用JDK动态代理技术。此外，由于CGLib采用生成子类的技术创建代理对象，所以不能对目标类中的final方法进行代理。