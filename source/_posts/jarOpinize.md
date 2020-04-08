---
title: 外部jar包打包方式
date: 2020/1/2
categories:
  - other
abbrlink: 29383
tags:
---


- 将原本 jar 包解压缩找到 BOOT-INF 下的 lib ,该目录下包含当前项目中用到的所有 jar包 ,复制出来放到jar包启动位置

- 在 pom 中 spring-boot-maven-plugin 打包插件设置打包时排除所有 jar 包
```
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <includes>
                        <include>
                            <!-- 排除所有Jar -->
                            <groupId>nothing</groupId>
                            <artifactId>nothing</artifactId>
                        </include>
                    </includes>
                </configuration>
            </plugin>
```

- 这个时候需要在jar包启动参数上添加一个 -Dloader.path
```
java -Dloader.path="lib/" -jar test.jar
```





