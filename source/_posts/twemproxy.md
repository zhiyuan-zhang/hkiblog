---
title: 推特redis twemproxy集群搭建 
date: 2020/6/1
categories:
  - redis
  - java
  - linux
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 7
---





简单介绍可以查看https://github.com/twitter/twemproxy.git

推特redis twemproxy集群搭建 

twemproxy

```shell
git clone https://github.com/twitter/twemproxy.git
```


如果报错 yum update nss


进入twemproxy文件


```shell
yum install automake libtool -y
```



```shell
autoreconf -fvi
```



如果报错

更改centOS源
https://developer.aliyun.com/mirror/

```shell
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```





yum clean all 


- 进入twemproxy文件夹

```shell
yum search autoconf
```



搜索出来的 install 下



```shell
yum install autoreconf268
```




- autoreconf268 -fvi 
执行完之后 多了一个 configure



- 执行 ./configure

- 执行 make

进入scripts


```shell
cp nutcracker.init /etc/init.d/twemproxy
cd /etc/init.d/twemproxy
```



赋予权限

```shell
chmod +x twemproxy 
```



因为该文件依赖一个yml配置文件

所以我们需要拷贝一个源码目录的这个配置文件

进入twemproxy文件夹

```shell
cp ./conf/* /etc/nutcracker/
cd /twemproxy/nutcracker/src
```

拷贝可执行 cp nutcracker /sur/bin

- 修改配置文件


```shell

cd /etc/nutcracker/

vi nutcracker.yml 
```


配置对应的server

```
servers
	- 127.0.0.1:6379
	- 127.0.0.1:6380
```





- 启动刚刚配置好的 redis


启动nutcracker 


```shell
service nutcracker start 
```
如果你改名了的话就用改了名字的启动

```shell
service twemproxy start 
```

然后去启动

```shell
redis-cli -p 22121
```
端口是你刚刚那个配置文件里配置的端口


这个时候你连接22121 的redis 添加的数据会落在你之前配置的yml文件的server数据库中

缺点  
不支持 keys *
不支持 watch
不支持 multi