---
title: maven多环境配置文件
date: 2015/10/8
categories:
  - java
tags:
  - 模板
comments: true
abbrlink: 49197
img:
---

```yml

# 多环境配置文件激活属性---开发、测试、生产
spring.profiles.active=@activatedProperties@


```



```xml


    <profiles>
        <profile>
            <!-- 测试环境 -->
            <id>test</id>
            <properties>
                <profiles.active>test</profiles.active>
            </properties>
        </profile>
        <profile>
            <!-- 本地开发环境 -->
            <id>dev</id>
            <!-- 默认环境为开发环境 -->
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        <properties>
            <profiles.active>dev</profiles.active>
        </properties>
        </profile>
    </profiles>


```

// 2019/11/22 测试不需要这个
```xml
      <!-- 打包后的名字(test.war) -->
        <resources>
            <!-- 打包时要把mapper.xml也打进去！ -->
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <!-- 资源根目录排除各环境的配置，使用单独的资源目录来指定 -->
                <excludes>
                    <exclude>test/*</exclude>
                    <exclude>dev/*</exclude>
                </excludes>
            </resource>
            <resource>
                <directory>src/main/resources/${profiles.active}</directory>
            </resource>
        </resources>


```

如果先远程部署后本地启动需要先maven install

// 没有-P 选择默认的
mvn clean package -Pdev  -Dmaven.test.skip=true


// 命令优先度高
java -jar  /xxx/xxx.jar  --spring.profiles.active=dev 

yml环境下有些区别 @换成#

详细情况可以参考 [SpringBoot + Maven实现多环境动态切换yml配置及配置文件拆分](https://blog.csdn.net/colton_null/article/details/82145467)