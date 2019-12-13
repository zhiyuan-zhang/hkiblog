---
title: swagger生成项目离线文档
date: 2015/10/8
categories:
  - java
tags:
  - swagger
comments: true
abbrlink: 49197
img:
---

最近需要写项目文档来记录下把swagger转换成adoc 然后根据adoc的格式依次转换成 pdf html 

首先导入 swagger包
```xml
   <!-- swagger start -->
        <dependency>
            <groupId>com.battcn</groupId>
            <artifactId>swagger-spring-boot-starter</artifactId>
            <version>2.1.5-RELEASE</version>
            <!---->
            <exclusions>
                <exclusion>
                    <groupId>com.battcn</groupId>
                    <artifactId>swagger-vue-ui</artifactId>
                </exclusion>
            </exclusions>

        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>${springfox.version}</version>
        </dependency>
```


导入转换插件  注意两个路径
```xml
 <plugin>
                <groupId>io.github.swagger2markup</groupId>
                <artifactId>swagger2markup-maven-plugin</artifactId>
                <version>1.3.1</version>
                <configuration>
                    <swaggerInput>http://localhost:8088/zhxtotc/v2/api-docs</swaggerInput><!---swagger-api-json路径-->
                    <outputFile>src/docs/asciidoc/generated/swagger</outputFile><!---生成路径-->
                    <config>
                        <swagger2markup.markupLanguage>ASCIIDOC</swagger2markup.markupLanguage><!--生成格式-->
                    </config>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.asciidoctor</groupId>
                <artifactId>asciidoctor-maven-plugin</artifactId>
                <version>1.5.6</version>
                <configuration>
                    <!--asciidoc文件目录-->
                    <sourceDirectory>src/docs/asciidoc/generated</sourceDirectory>
                    <!---生成html的路径-->
                    <outputDirectory>docs/asciidoc/html</outputDirectory>
                    <backend>html</backend>
                    <sourceHighlighter>coderay</sourceHighlighter>
                    <attributes>
                        <!--导航栏在左-->
                        <toc>left</toc>
                        <!--显示层级数-->
                        <!--<toclevels>3</toclevels>-->
                        <!--自动打数字序号-->
                        <sectnums>true</sectnums>
                    </attributes>
                </configuration>
            </plugin>

```
