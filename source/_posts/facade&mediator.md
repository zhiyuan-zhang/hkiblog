---
title: 门面模式和调停者模式
date: 2017/1/27
categories:
  - 设计模式
  - java
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 29384
---

# facade mediator



## 对外 门面模式



## 对内 调停者模式



### 著名的消息中间件就是这个思路



## 案例 : 

```java

package facade;
public class FacadePattern{
    public static void main(String[] args){
        Facade f=new Facade();
        f.method();
    }
}
//外观角色
class Facade
{
    private SubSystem01 obj1=new SubSystem01();
    private SubSystem02 obj2=new SubSystem02();
    private SubSystem03 obj3=new SubSystem03();
    public void method(){
        obj1.method1();
        obj2.method2();
        obj3.method3();
    }
}
//子系统角色
class SubSystem01{
    public  void method1(){
        System.out.println("子系统01的method1()被调用！");
    }   
}
//子系统角色
class SubSystem02{
    public  void method2(){
        System.out.println("子系统02的method2()被调用！");
    }   
}
//子系统角色
class SubSystem03{
    public  void method3(){
        System.out.println("子系统03的method3()被调用！");
    }   
}
```







参考资料: http://c.biancheng.net/view/1369.html









