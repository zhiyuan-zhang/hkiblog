---
title: idea远程debug
date: 2017/4/16
categories:
  - java
tags:
  - 模板
comments: true
abbrlink: 49197
img:
---


远程debug 原理是利用端口监听来触发本地idea 

所以你本地的代码和远程的代码要保持一致

部署上去之后需要加上特定的参数来启动 

原来启动jar包需要的命令
```shell
java -jar xxx.jar
```

如果是远程debug 则需要加上一些参数
```shell
java -jar -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=4001 xxx.jar 
```

这个参数在idea里可以直接获取到

接下来我们去找如何获取debug参数


菜单 -> Run -> Edit Configurations…

![img](https://victorblog.nos-eastchina1.126.net/2072/1.png)

在debug Configurations的弹出框里添加一个remote

![img](https://victorblog.nos-eastchina1.126.net/2072/2.png)


添加后需要注意配置好 1-2 后就生成了启动参数

![img](/themes/hexo-theme-snippet/source/img/a.png)

之后启动jar的时候添加上 服务器启动好后 在启动本地的


QAQ
- 当发现启动报错 首先检查下你的命令 不要直接复制我的 看你自己生成的代码
- 保证端口没被占用
- 本地启动remote会发送请求连接服务器 如果本机无法连接服务器也不行


