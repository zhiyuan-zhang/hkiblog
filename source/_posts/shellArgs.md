---
title: Shell 传递参数
date: 2015/10/8
categories:
  - java
tags:
  - shell
comments: true
abbrlink: 49197
img:
---

脚本内获取参数的格式为：$n。n 代表一个数字，1 为执行脚本的第一个参数，2 为执行脚本的第二个参数，以此类推……

新建shell
```
#!/bin/bash

echo "Shell 传递参数实例！";
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";
```



调用
```
$ ./test.sh 1 2 3
```