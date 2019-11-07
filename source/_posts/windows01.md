---
title: 局域网共享文件夹如何换用户名登陆
date: 2015年10月8日
categories:
  - windwos
tags:
  - 共享文件夹
  - 登陆
comments: true
abbrlink: 49197
img:
---

你在运行里输入CMD
在命令提示符中输入：
```shell
net use * /delete  
```
OR
```shell
net use * /del /y
```
然后它会问你是否要删除网络连接，按Y，回车即可。

