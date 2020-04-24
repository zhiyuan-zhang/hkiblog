---
title: 工厂设计模式实际应用案例
date: 2020/4/24
categories:
  - 设计模式
  - java
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 64767
---





# 设计模式中的工厂模式



  工厂模式在实际应用中也很常见 各种框架的factory也很多 



## 先说说需求



我们这个项目有一个发送通知的需求

通知有各种各样的 举几个例子

手机验证码通知

项目审批通知

出函通知

确认函通知

会议取消通知

待处理通知

... 



发送方式也有很多种

邮件通知

短信通知

微信通知

钉钉通知

...



所以在设计的时候一个合格的通知必须经过三个步骤

1. 确定发送类型
2. 确定发送人
3. 确定发送方式



我们可以把通知类做成一个factory 专门做通知的发送

// 2020年修改

后来我觉得这么设计有点不妥

改成

factory 制作各种通知

由策略模式来负责发送短信



工厂只负责短信的生产,不负责发送





## 具体实现

### 步骤 1

创建一个接口。

ProfessorStrategy

```java
public interface notice {
   void type();
  void choose();
  void send();
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

创建一个*工厂*，生成基于给定信息的实体类的对象。

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













