---
title: predixy
date: 2020/6.6
categories:
  - other
  - java
  - linux
  - db
  - 工具
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 7
---



predixy

去github 找最新的tar包下载到服务器 

https://github.com/joyieldInc/predixy.git


git clone  https://github.com/joyieldInc/predixy/releases/download/1.0.5/predixy-1.0.5-bin-amd64-linux.tar.gz



解压
tar xf predixy-1.0.5-bin-amd64-linux.tar.gz



进入主目录
cd bin 


配置文件
cd conf


vi predixy.conf 

Bind 127.0.0.1:7617


找到servers
Include sentinel.conf



vi sentinel.conf


复制examples

在sentinels 里面配置 你之前设置好的redis-sentinel

+ 127.0.0.1:26379
+ 127.0.0.1:26380
+ 127.0.0.1:26381


还需要配置你的主库名称

Group master1 {
	
}

Group master2 {
	
}