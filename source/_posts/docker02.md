---
title: 浅析docker
date: 2020/3/10
categories:
  - other
tags:
  - docker
comments: true
abbrlink: 49197
typora-root-url: ../
typora-copy-images-to: ../static
img:

---



# docker

## Docker 安装

Mac https://docs.docker.com/docker-for-mac/install/Windows 

https://docs.docker.com/docker-for-windows/install/Ubuntu 

https://docs.docker.com/install/linux/docker-ce/ubuntu/Debian 

https://docs.docker.com/install/linux/docker-ce/debian/CentOS 

https://docs.docker.com/install/linux/docker-ce/centos/Fedora 

https://docs.docker.com/install/linux/docker-ce/fedora/

其他 Linux 发行版 

https://docs.docker.com/install/linux/docker-ce/binaries/

## 镜像加速

https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors 可以按照阿里云的步骤 操作

## 容器与镜像

![docker](https://docs.docker.com/engine/images/architecture.svg)

docker pull  拉取镜像image 然后根据镜像image 产生具体的容器

我们最后操作的是容器并不是镜像image

client 指的是我们的操作命令 各种pull run 等等, 
我们可以根据命令来操作dockerHost,
registry 是我们的远程仓库,或者说是dockerhub接下来就是我们的整体dockerHost

这里面包含了 整个docker组件, images CONTAINERS  

你可以把daemon当做是一个管家来管理docker组件你在client端上的所有操作都会由它来执行 

## 容器生命周期

![image-20200512143758893](/static/image-20200512143758893.png)

## 基础命令

yum install docker

service docker start



### pull

- 拉取镜像文件命令
- docker pull mysql:latestdocker pull mysql:8.0.20格式是 : docker pull NAME[:TAG] TAG 默认是 latest 

### images

- docker images 查看所有已经下拉到本地的镜像
- docker images  | grep 'test'

### run

- docker run tomcat
- 端口转发docker run -p 8000:8080 tomcat
- -d 后台运行
- --name  赋予容器名称
- --ip 172.17.0.10 指定启动的ip地址

### logs

- docker logs [容器ID]  查看一个容器的启动日志

### ps

- docker ps 查看所有正在运行容器 (并不是镜像)
- -a 查看所有容器

### stop

- 将一个已经在运行的容器停止 docker stop [容器ID]

### kill

- 与stop不同的是 kill 会将这个容器删除 stop是将这个容器停止 后续还可以继续start使用

### rm

- 使用前容器必须在stop状态下docker rm [容器id] 
- 无需容器在stop状态强制删除 -fdocker rm -f [容器ID]
- docker rm $(docker ps -a -q)  删除所有容器
- 强制删除镜像imagesdocker rmi -f [容器ID]
- 注意点：1. 删除前需要保证容器是停止的  stop2. 需要注意删除镜像和容器的命令不一样。 docker rmi ID  ,其中 容器(rm)  和 镜像(rmi)3. 顺序需要先删除容器

### exec

- docker exec -it [id] /bin/bash   进入一个已经存在的容器

### inspect

- docker inspect [容器ID] 查看容器的详细信息

### network

- docker network ls 列出当前存在的网络连接
- docker network create  my_net 创建一个网桥 类似于网关
- -d 设置创建的网络驱动

### build

- docker build -t  机构/镜像名&lt;版本&gt; Dockerfile目录

### help

- docker -h 
- docker run -h

## DockerFile

### 构建docker build -t  机构/镜像名&lt;版本&gt; Dockerfile目录

### FROM 基准镜像 基于哪一个来构建的image

- FROM centos
- FROM scratch 不依赖任何镜像
- FROM mysql:8.0.20

### 说明信息

- MAINTAINER 当前镜像由谁来维护
- LABEL version="V1.0"
- LABEL description = ""

### WORKDIR 工作目录 类似cd  

### ADD &amp; COPY

- ADD text.txt / 将当前text文件添加到容器的根目录
- ADD text.gz / 添加并解压缩

### ENV 

- ENV JAVA_HOME /sur/local/jdk  设置容器环境变量

### RUN 

- shell 模式

	- RUN yum install -y vim 

- exec 模式(推荐)

	- RUN ["yum","install","-y","vim"]

### ENTRYPOINT

- 只有最后一个有效
- ENTRYPOINT ["ps"]

### CMD 

- 类似ENTRYPOINT 但是有时候不一定会被执行 建议用ENTRYPOINT
- CMD 当你的容器启动的时候最后附带参数的话就不会被执行 如果不附带参数的话就会执行当前CMD指令

### EXPOSE

- 对外暴露端口

### docker build -t  机构/镜像名&lt;版本&gt; Dockerfile目录docker build -t hki.com/myapp:1.0 .

## 容器间通讯

### 单向通讯

- 在启动的时候加上一个参数 --link [需要通讯的容器名称]

### bridge网桥 双向通讯

- 启动的时候指定网桥来通讯

	-  --network my_net启动的时候多加一个参数来指定当前容器的 网桥是哪个

- 创建一个网桥 用来通讯

	- 1. 创建一个网桥docker network create my_net
	- 2. 将容器containerr1连接到新建网络my_netdocker network connect my_net container1
	- 3. 将容器containerr2连接到新建网络my_netdocker network connect my_net container2

## volume数据共享

### 数据共享分两种 1. 在启动的时候加上-v参数来设置数据卷 2.  先设置一个存档点,然后后续容器启动的时候直接指定这个存档点

### -v 宿主机文件地址:容器的文件地址

### 创建共享容器来存放数据

- docker create --name my_volume -v /usr/test:/usr/local/tomcat/webapps/ tomcat
- docker run -d -p  8000:8080 --volumes-from my_volume --name test tomcat

## DockerCompose

### Windows 和 mac 不需要安装 linux需要

- sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
- sudo chmod +x /usr/local/bin/docker-compose

### k8s 代替了 

