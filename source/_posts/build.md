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

但是这么写在《effective java 》中是不推荐这么写的 

书中写到如果你要构建复杂对象的话 最好使用build模式来做

```java
public interface SysMessageBuilder{
			// some 
  SysMessageBuilder addName();
  
  SysMessageBuilder addPhone();
  
  SysMessageBuilder createSysMessageBuilder();

  SysMessageBuilder send();
  
  
  
}
```





```java
public static class SysMessageBuilderImpl  implements SysMessageBuilder, Serializable {
        
  			SysMessage sysMessage = new SysMessage();
  
  
				public SysMessage addPhone(String phone) {
          	//somothing
  					sysMessage.setPhone(phone);
            return sysMessage;
        }
  
  			public SysMessage addName(String name) {
          	//somothing
  					sysMessage.setName(name);
            return sysMessage;
        }
  
  
        public SysMessage createSysMessageBuilder() {
            return sysMessage;
        }
  
  			public SysMessage send(){
           return repositoryService.deploy(this);
        };
  

```





使用的话

```java


SysMessageBuilder.createSysMessageBuilder().addName("张三").addPhone("18812341234").send();


// 复杂一些的类似activity框架 这样调用
		 repositoryService.createDeployment().key("6513546").name(deploymentNameHint).deploy();


```













