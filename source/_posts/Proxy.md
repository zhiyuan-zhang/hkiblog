---
title: AOP之代理模式
date: 2019/8/18
categories:
  - 设计模式
  - java
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 61510
---



在我看来 代理模式 最常用 也最难理解

会得不难 , 难得不会, 在我看来现在理解了代理的运作模式也觉得没有以前想的那么复杂



代理模式说白了就是面向切面编程 AOP **业务和实际切点代码分割**



那么重点就在于 如何使在最小修改业务代码的情况下 来完成切点的插入





首先实现一个业务接口 

```java
interface Movable {
    void move();
}
```





接口具体实现类



```java
public class Tank implements Movable {

    /**
     * 模拟坦克移动了一段儿时间
     */
    @Override
    public void move() {
        System.out.println("Tank moving claclacla...");
        try {
            Thread.sleep(new Random().nextInt(10000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



这个时候我们有一个新的需求需要在 这段业务前后加入某些逻辑 不管是日志还是权限

这个时候我们需要新建一个代理类 

<font color = #F50A0A >实现 InvocationHandler 接口 然后实现具体的invoke方法</font>

具体逻辑看下图

```java
class TimeProxy implements InvocationHandler {
    Movable m;

    public TimeProxy(Movable m) {
        this.m = m;
    }

    public void before() {
        System.out.println("method start..");
    }

    public void after() {
        System.out.println("method stop..");
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //Arrays.stream(proxy.getClass().getMethods()).map(Method::getName).forEach(System.out::println);

        before();
        Object o = method.invoke(m, args);
        after();
        return o;
    }

}
```



最纠结的点可能是 这段代码 

method.invoke(m, args); 

先放一边 我们看是怎么调用的



main函数调用

```java
public static void main(String[] args) {
    Tank tank = new Tank();
		//返回的是ASM生成的$Proxy0的方法  后续调用也是调用这个方法的move 
    Movable m = (Movable)Proxy.newProxyInstance(
      			Tank.class.getClassLoader(),
            new Class[]{Movable.class}, //tank.class.getInterfaces()
            new TimeProxy(tank)
    );

    m.move();
}
```



我们给Proxy 传了三个参数  分别是 

Tank.class.getClassLoader() 

生成的代理对象$Proxy0 

我们传的是tank.class 实现的接口是movable 

所以ASM生成的代理类对象$Proxy0 也会实现接口movable 同时也会实现里面的方法 move()

可以说 <font color = #6AACDE >**我们把 Movable 传进去告诉ASM 你们给我生成一个实现了Movable接口的类** </font> 让我来调用



new Class[]{Movable.class}

也可以写成

```
Tank.class.getInterfaces()
```

reflection 通过二进制字节码分析类的属性和方法

就是拿到这个接口类的所有的属性和方法

不明白的话我举个例子 

就是 Tank这个类实现的接口是哪个接口类 你要告诉ASM

你也可以直接给我new出来 这个类 也可以通过 Tank拿到他的接口类



new TimeProxy(tank)

这个就是具体的实现类 

你具体要给这个接口的哪个实现类实现代理你要给ASM说

这三个参数涉及到 ASM newProxyInstance生成代理类对象的逻辑

可能会有点复杂在这里就不展开谈了





