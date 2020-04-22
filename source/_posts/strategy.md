---
title: 策略设计模式实际应用案例
date: 2020/1/18
categories:
  - 设计模式
  - java
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 64767
---





# 设计模式中的策略模式



 讲理论的话网上有很多现成的 以前也看了不少 这次特地在实际项目中抽取出来做成笔记

## 先说说需求



根据不同的单据类型选出选出五个教授

其中单据类型有可转债和挂牌两大类 目前可转债和挂牌各有几种选择方式 下面案例我一样拿出一种来

之前的模式在有多种策略选择相似的情况下，使用 if...else 复杂和难以维护,代码阅读起来也比较困难

现在使用策略模式的步骤及简化结果 





## 具体实现

### 步骤 1

创建一个接口。

ProfessorStrategy

```java
public interface ProfessorStrategy {

    List<Professor> five(Integer tableId);


}
```



### 步骤 2

创建实现接口的实体类。

EIMProfessorStrategy

```java
public class EIMProfessorStrategy implements ProfessorStrategy{

    @Override
    public List<Professor> five(Integer tableId) {
        //省略了我的筛选策略
        return null;
    }
}
```



IPDProfessorStrategy

```java
public class IPDProfessorStrategy implements ProfessorStrategy{


    @Override
    public List<Professor> five(Integer tableId) {
     		//省略了我的筛选策略
        return null;
    }
}
```



UserProfessorStrategy

```java
public class UserProfessorStrategy implements ProfessorStrategy{


    @Override
    public List<Professor> five(Integer tableId) {
     		//省略了我的筛选策略
        return null;
    }
}
```



### 步骤 3

创建具体的 *find* 方法, 你也可以创建一个有泛型的类来使用。

```java
    /**
     * 返回接口数据
     * @param tableId
     * @param professorStrategy
     * @return
     */
    public static   List<SysAttendsModel> find(Integer tableId, ProfessorStrategy professorStrategy) {

//        获取对应的五个专家
        List<Professor> five = professorStrategy.five(tableId);

        List<SysAttendsModel> sysAttendsModels = new ArrayList<>();

        return sysAttendsModels;
    }

```



### 步骤 4

使用 *find* 来查看当它改变策略 *Strategy* 时的行为变化。

main

```java
  public static void main(String[] args) {
				// 单据id
        int tableId = 10086;

				// 挂牌算法
        List<SysAttendsModel> list = find(tableId, new EIMProfessorStrategy());
        // 可转债算法
    		List<SysAttendsModel> list2 = find(tableId, new IPDProfessorStrategy());
        // 会员算法
    		List<SysAttendsModel> list2 = find(tableId, new IPDProfessorStrategy());


    }

```



## 代码讲解

讲讲上面的代码 

大概就和支付策略是一个逻辑



**支付前都需要找出二维码**

**支付后都需要查看是否到账**



中间走具体的 现金 or 微信支付 or 支付宝支付 我们不关心

不管后面有多少个支付手段我们就加上相对应的接口的实体类就行了.











