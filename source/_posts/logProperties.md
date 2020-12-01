---
title: 日志配置文档说明
date: 2020/11/30
categories:
  - java
  - log
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 7
---



# 配置文件说明

## log4j2

```xml
<!--全局参数-->
    <Properties>
       <!-- 配置日志文件输出目录 -->
        <Property name="LOG_HOME">/opt/data/log/hunavi</Property>
        <!-- 日志输出文件名 -->
        <property name="FILE_NAME">info.log</property>
        <!-- info日志格式化 -->
        <property name="pattern_layout">
            %d{yyyy-MM-dd HH:mm:ss.SSS/zzz} %-5p [TMS] [%X{hostIp}] [%X{taskId}] [%X{taskRecordId}]  [%X{point}]   [%c, %L] %m%n
        </property>
        <!-- 业务日志格式化 -->
        <property name="biz_pattern_layout">
            %d{yyyy-MM-dd HH:mm:ss.SSS/zzz} %-5p [TMS] [%X{hostIp}] [%X{taskId}] [%X{taskRecordId}]   [%c,  %L] %m%n
        </property>
    </Properties>
```



```json
 2020-08-18 18:08:56  Biz  [TMS] [32.112.31.21]  [] []  {"biz":true, "module":"xxx","functionPoint":"xxx" , "objectType":"task" , "objectId": "task_xxx", "output":{ "msg": "下载注册任务", "code": "00000"  } }
```



# MDC配置说明



在日志文档中需要输出**taskId** 和**taskCode** 这里是利用了 log中的MDC 为“Mapped Diagnostic Context”（映射诊断上下文）功能实现 下面简述下具体实现逻辑

MDC类基本原理其实非常简单，其内部持有一个InheritableThreadLocal实例，用于保存context数据，MDC提供了put/get/clear等几个核心接口，用于操作ThreadLocal中的数据；ThreadLocal中的K-V，可以在logback.xml中声明，最终将会打印在日志中



简单描述就是 在代码中配置

```java
MDC.put("taskId",1000);	
```



那么在logback.xml中，即可在layout中通过声明"%X{taskId}"来打印此信息。



# 多环境配置说明

 将不同级别的日志文件输出到不同的文件中



```xml
<RollingRandomAccessFile name="ManageBiz"
                         fileName="${LOG_HOME}/ManageBiz.log"
                         filePattern="${LOG_HOME}/ManageBiz.%d{yyyy-MM-dd}_%i.log">
    <Filters>
        <ThresholdFilter level="FATAL" onMatch="ACCEPT" onMismatch="DENY"/>
    </Filters>
    <PatternLayout pattern="${biz_pattern_layout}" />
    <Policies>
        <TimeBasedTriggeringPolicy interval="1"/>
        <SizeBasedTriggeringPolicy size="100 MB" />
    </Policies>
</RollingRandomAccessFile>
```



**RollingRandomAccessFile基本属性**

  name：Appender名称

  fileName：日志存储路径

  filePattern：历史日志封存路径。其中%d{yyyyMMddHH}表示了封存历史日志的时间单位（目前单位为小时，yyyy表示年，MM表示月，dd表示天，HH表示小时，mm表示分钟，ss表示秒，SS表示毫秒）。注意后缀，log4j2自动识别zip等后缀，表示历史日志需要压缩。



**ThresholdFilter**

ThresholdFilter：配置的日志过滤
               如果要输出的日志级别在当前级别及以上，则为match，否则走mismatch
               ACCEPT： 执行日志输出；
               DENY： 不执行日志输出，结束过滤；
               NEUTRAL： 不执行日志输出，执行下一个过滤器

 日志级别说明

```shell
日志级别：TRACE < DEBUG < INFO < WARN < ERROR < FATAL
```



**PatternLayout**

格式化输出日志的规则定义



**pattern详解**



%d 时间

%X 参数

%F 文件名

%l 输出完整位置

%L 错误行号

%m 错误信息



具体详细配置请看

https://blog.csdn.net/yu870646595/article/details/88743033

https://blog.csdn.net/iteye_19607/article/details/82677252



**Policies**

`Policy`是用来控制日志文件何时(When)进行滚动的；



每天记录一个日志 

TimeBasedTriggeringPolicy

按照大小区分日志记录

SizeBasedTriggeringPolicy



