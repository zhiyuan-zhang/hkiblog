---
title: build 复杂对象构造器
date: 2019/8/18
categories:
  - 设计模式
  - java
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 7
---







Build 模式一般是用来对复杂对象的构造器



一般我们可以对一个复杂对象这样new 

```java

    public SysMessage(String phone, String comm, LocalDateTime sendTime, Integer status, Integer templateId, String tableId, String name, String role) {
        this.phone = phone;
        this.comm = comm;
        this.sendTime = sendTime;
        this.status = status;
        this.templateId = templateId;
        this.tableId = tableId;
        this.name = name;
        this.role = role;


    }
```



使用的时候是这样的

```java
new SysMessage(getTel(), comm, LocalDateTime.now()
                                    , getDeleted(), 1006,
                                    "10086", getName(), "专家")
```





上面这个例子是一个短信模板对象的生成 

这样写也可以  如果不想要其中某个我们可以传空 

但是这么写在<<effective java >>中是不推荐这么写的 

书中写到如果你要构建复杂对象的话 最好使用build模式来做

```java
public static class SysMessageBuild{
        SysMessage sysMessage = new SysMessage();

        public SysMessageBuild basicInfo(String role,String name,String phone,String comm) {
            sysMessage.role = role;
            sysMessage.name = name;
            sysMessage.phone = phone;
            sysMessage.comm = comm;
            return this;
        }

        public SysMessageBuild sendTime(LocalDateTime sendTime){
            sysMessage.sendTime = sendTime;
            return this;
        }
        public SysMessageBuild status(Integer status){
            sysMessage.status = status;
            return this;
        }

        public SysMessageBuild templateId(Integer templateId){
            sysMessage.templateId = templateId;
            return this;
        }

        public SysMessageBuild tableId(String tableId){
            sysMessage.tableId = tableId;
            return this;
        }
        public SysMessage build() {
            return sysMessage;
        }

```



最后用一个build方法返回 













