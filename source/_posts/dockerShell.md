---
title: docker基本语法
date: 2019/10/8
categories:
  - other
tags:
  - 
comments: true
abbrlink: 49197
img:
---

-  	查看所有正在运行容器 
```shell
$ docker ps 
```

-  	查看所有容器 
```shell
$ docker ps -a 
```

-	查看所有容器ID 
```
$ docker ps -a -q 
```

- 启动容器
```
$ docker start [NAME]	
```

-	stop停止所有容器 
```
$ docker stop 
$(docker ps -a -q)
```


- 	remove删除所有容器
```
$ docker rm $(docker ps -a -q) 
```

- run创建一个新的服务
```
docker run -it [NAME] /bin/bash	
```

- Exec 进入一个已经存在的
```
docker exec -it 217a29f3495a /bin/bash	
```

-	查看容器启动日志
```
docker logs [容器ID]
```

-删除容器镜像  image
```
sudo docker rmi IMAGE [IMAGE...]	
sudo docker rmi centos:latest
```
- 强制删除 实例
```
docker rm -f 
```

- 启动一个实例Container 
```
sudo docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
sudo docker run -t -i contos /bin/bash
sudo docker run  -p 8080:8080 -d [name]
```


