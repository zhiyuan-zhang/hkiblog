---
title: 基于SLF4J MDC机制配合AOP实现日志的链路追踪
date: 2019年05月31日
categories:
  - other
tags:
  - AOP
  - SLF4J
  - MDC
comments: true
abbrlink: 49197
img:
---


# 问题描述
一个合格的项目必须要有日志来支撑,日志不但能记录输入输出,当系统有问题的时候我们还需要做线上问题的排查.

在一个正常的项目中日志里包含了各种各样的接口及其他无关的数据日志,那么我们如何快速定位单次请求中所有的日志呢 ?

# 问题分析

当我们设计一个系统日志的时候

首先我们需要解决以下几个问题



1. 哪些数据需要写进日志中

2. 日志如何分类 按天还是按周

3. 如何区分每次请求

4. 请求参数及返回值需不需要打印

5. 如何进行多环节配置



# 解决方案

按照上面的问题我们来一个一个解决

主要思路是
1. AOP负责切入每个请求及参数打印

2. 在进入controller之前打印本次请求的各种参数

3. MDC添加hashCode来做参数校验

4. 日志使用logback配置

5. 日志按天分类,每天生成一个日志

6. 利用thread来区分每次请求

7. springProfile来做多环境配置

aop有两种 
CGLIB,JDK 都是动态代理 今天不讨论这两者的区别 我使用的是CGLIB




pom中引入SpringBoot的web模块和使用AOP相关的依赖：


定义切面类，实现web层的日志切面

对所有的web请求做切面来记录日志





# 第一种 AOP上 logback的输出

```java
   // 接收到请求，记录请求内容
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        if (attributes != null && joinPoint != null) {

            HttpServletRequest request = attributes.getRequest();
            
            // 记录请求内容
            log.info( "1. 对象请求的URL : " + request.getRequestURL().toString());
            log.info( "2. 请求方法名称 : " + request.getMethod());
            log.info( "3. 对方IP地址 : " + request.getRemoteAddr());
            log.info( "4. 运行的java类 : " + joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
            try{
                log.info("5. 请求参数 : " + JSONObject.toJSONString(joinPoint.getArgs() ));
            }catch (Exception e){
                log.info("请求参数切点无法切入");
            }

        }else{
            throw new CheckException("网络请求出错, 请清空缓存重新尝试. ");
        }
```


# 第二种 request.HashCode 唯一标示

在获得
```java

log.info(request.hashCode()+ "5. 请求参数 : " + JSONObject.toJSONString(joinPoint.getArgs() ));
```


# 第三种 基于SLF4J 的MDC 机制

```java

MDC.put("THREAD_ID", "userId"+ userService.getId() );

```

```xml
 <pattern>%date [%thread] %-5level %logger{80}  %X{THREAD_ID} || %msg%n</pattern>
```



# 第四种 结合HashCode 和MDC 

```java

MDC.put("THREAD_ID", ""+request.hashCode());

```




# 最终效果

```java
2019-06-04 10:14:27,743 [main] INFO  com.zwkj.zhxt.ZhxtApplication || Started ZhxtApplication in 48.489 seconds (JVM running for 56.375)
2019-06-04 10:15:28,310 [qtp1387878879-21] INFO  com.zwkj.zhxt.common.ControllerAOP || 1. 对象请求的URL : http://localhost:8088/zhxtotc/sys-user/login
2019-06-04 10:15:28,313 [qtp1387878879-21] INFO  com.zwkj.zhxt.common.ControllerAOP || 2. 请求方法名称 : POST
2019-06-04 10:15:28,314 [qtp1387878879-21] INFO  com.zwkj.zhxt.common.ControllerAOP || 3. 对方IP地址 : 0:0:0:0:0:0:0:1
2019-06-04 10:15:28,323 [qtp1387878879-21] INFO  com.zwkj.zhxt.common.ControllerAOP || 4. 运行的java类 : com.zwkj.zhxt.controller.SysUserController.findSysUser
2019-06-04 10:15:28,585 [qtp1387878879-21] INFO  com.zwkj.zhxt.common.ControllerAOP || 2825564545. 请求参数 : [{"code":"string","password":"123456","phone":"string","username":"admin12"}]
 Time：168 ms - ID：com.zwkj.zhxt.mvc.sysuser.mapper.SysUserMapper.selectOne
Execute SQL：SELECT id,username,password,nickname,role_id,create_time,update_time,delete_status,user_id FROM sys_user WHERE username = 'admin12' AND password = '4QrcOUm6Wau+VuBX8g+IPg=='

2019-06-04 10:15:30,860 [qtp1387878879-21] INFO  com.zwkj.zhxt.common.ControllerAOP || @Around:结果是 :ResultBean com.zwkj.zhxt.controller.SysUserController.findSysUser(UserModel) use time: 2551
2019-06-04 10:15:30,861 [qtp1387878879-21] INFO  com.zwkj.zhxt.common.ControllerAOP || 282556454: 方法的返回值 : ResultBean(msg=success, code=0, data=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJub3ciOiIxNTU5NjE0NTMwMjE2emh4dCIsImtleSI6InVzZXIxMiJ9.ae7ra7hvDSp5N6YfuGYzr8ULeq7Zr5OuC_4PbybqceY)
```

```java
2019-06-04 10:15:41,401 [qtp1387878879-19] INFO  com.zwkj.zhxt.common.ControllerAOP || 1. 对象请求的URL : http://localhost:8088/zhxtotc/sys-user/login
2019-06-04 10:15:41,401 [qtp1387878879-19] INFO  com.zwkj.zhxt.common.ControllerAOP || 2. 请求方法名称 : POST
2019-06-04 10:15:41,402 [qtp1387878879-19] INFO  com.zwkj.zhxt.common.ControllerAOP || 3. 对方IP地址 : 0:0:0:0:0:0:0:1
2019-06-04 10:15:41,402 [qtp1387878879-19] INFO  com.zwkj.zhxt.common.ControllerAOP || 4. 运行的java类 : com.zwkj.zhxt.controller.SysUserController.findSysUser
2019-06-04 10:15:41,402 [qtp1387878879-19] INFO  com.zwkj.zhxt.common.ControllerAOP || 2825564545. 请求参数 : [{"code":"string","password":"123456","phone":"string","username":"admin12"}]
 Time：151 ms - ID：com.zwkj.zhxt.mvc.sysuser.mapper.SysUserMapper.selectOne
Execute SQL：SELECT id,username,password,nickname,role_id,create_time,update_time,delete_status,user_id FROM sys_user WHERE username = 'admin12' AND password = '4QrcOUm6Wau+VuBX8g+IPg=='

```



> # \\*literal asterisks\\\* 转义

\*literal asterisks\*

> # MD语法 加粗 \*\*加粗**
 
**比较粗**
># MD语法 斜体 \*斜体* 

*本来就是斜的*  

># MD语法  列表1. 2. 3.

1. ni shuo ni shi shui ?
2. ni bushi shuo ni me ?
3. wo xi huan de shi ni !

># MD语法  列表 -XXX -XXX -XXX

- wo bu shi yi ge haoren 
- na ni shi shen me ?
- wo shi dog

># MD语法 分割线  \*\*\* 代表一条分割线

***
***
***


># MD语法 A标签 \[A标签显示的名称]\(链接地址)


[baidu](http://www.baidu.com)


># MD语法  图片 \!\[ 图片说明 ](图片地址)

![img](/img/banner.jpg)


>#  MD语法 编写代码 三个 `

``` python
def test()
  print('asd')
test()

```


># MD语法 表格

 id	 | select_type	|table	
:-: | :-: | :-: 
 1	 | "SIMPLE"	|"professor"	
 2	 | "SIMPLE"	|"professor"	
 3	 | "SIMPLE"	|"professor"	


># MD语法 标题  一个#号是一级标题 依此类推

# 一级标题

## 二级标题

### 三级标题

#### 四级标题

##### 五级标题

###### 六级标题
