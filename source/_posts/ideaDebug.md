---
title: IDEA远程调试
date: 2020/2/25
categories:
  - 工具
tags:
  - idea
  - debug
comments: true
abbrlink: 49197
img:
---

## 远程debug是什么 ?

远程debug 原理是利用服务器端口监听来触发本地开发工具IDEA的一种调试手段 

## 什么情况下会用到 ?

一般我们本地调试的话会直接在代码中打断点然后debug模式运行代码就可以调试了.

但是我们在实际开发中会遇到一些比较复杂的问题我举几个例子

1. 涉及到调用生产环境会出现的网络问题.

2. 本地运行没问题,只有在测试服务器或者生产环境会出现报错

3. 因为开发系统和服务器系统不一致导致的问题

等等类似这样的问题我们在本地就没办法调试,这时候就需要用到远程服务器调试了.

## 如何使用远程服务器调试

远程调试主要有以下几个步骤

- 在调试之前我们需要你本地的代码和远程的代码要保持一致

- 部署上去之后需要加上特定的参数来启动 

- 服务器启动之后在启动本地环境


保持一致就不需要我们详细说了,部署最新版的就可以了.

接下来我们来加上启动参数

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


> 菜单 -> Run -> Edit Configurations…

![img](/themes/hexo-theme-snippet/source/img/git1.png)

> 在debug Configurations的弹出框里添加一个remote

![img](/themes/hexo-theme-snippet/source/img/git2.png)


> 之后会弹出一个如下图所示的界面 我们需要配置两个地方.图中的1是我们的服务器地址和端口,端口我们一般默认为4001.也可以自己配置.2是我们需要调试的项目.当这两个配置好后3的位置就会自动生成我们需要的启动参数

![img](/themes/hexo-theme-snippet/source/img/a.png)

> 之后启动jar的时候添加上 服务器启动好后 在启动本地的remote

## 注意事项
- 当发现启动报错 首先检查下你的命令 不要直接复制我的 看你自己生成的代码
- 保证端口没被占用
- 本地启动remote会发送请求连接服务器 如果本机无法连接服务器也不行


