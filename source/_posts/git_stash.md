---
title: git stash 在多分支下的使用
date: 2019/3/8
categories:
  - git
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 29384
---



# 1. 前言

stash命令叫暂存 平常开发很多情况都会遇到 开发一半的代码线上环境要紧急修复个bug这个时候我们可以把代码暂存起来

常用的有

```sh
gsta='git stash save'

gstaa='git stash apply'

gstc='git stash clear'

gstd='git stash drop'

gstl='git stash list'

gstp='git stash pop'

gsts='git stash show --text'
```



## 问题

这一段时间频繁在各个分支上开发及修改bug导致用到的stash命令挺多的

最近出现一个挺让我纠结的问题就是 在各个分支下stash存放的栈是一样的所以在dev分支下和master分支下你保存的数据都是一个地方 

如果你在dev 下save了stash 然后切换master分支你仍然可以apply到你刚刚在dev下保存的代码

下面我们看具体代码

# 2. 出现原因





现在我们文件下有两个文档 123 和 abc 

```sh
master > ll
-rw-r--r--  1 apple  staff    21B  4 23 15:30 123.txt
-rw-r--r--  1 apple  staff    66B 10 29 10:44 abc.txt
```



现在我们删除一个

```sh
master > rm -rf abc.txt
total 8
-rw-r--r--  1 apple  staff    21B  4 23 15:30 123.txt
```







不提交当前代码 stash保存起来


```sh
master > git stash save "master save"
Saved working directory and index state On master: master save
```



切换到生产分支


```sh
master > git checkout prod
切换到分支 'prod'
prod > ll
total 16
-rw-r--r--  1 apple  staff    21B  4 23 15:30 123.txt
-rw-r--r--  1 apple  staff   121B  4 23 16:11 abc.txt
```



再次修改代码 并且stash保存


```sh
 prod > vi 123.txt
 prod > git stash save "prod save"
Saved working directory and index state On prod: prod save
```





切换分支查看stash list



```sh
master > git checkout master
切换到分支 'master'
master > git stash list
stash@{0}: On prod: prod save
stash@{1}: On master: master save
```



这个时候显示出了两个保存栈 分别是我在master和prod上保存的代码

如果这个时候你直接git stash apply 会使用最近的一个 也就是你在prod下保存的代码而不是你master的代码


# 3. 优化使用



Stash 有两种保存方式 一种是带消息保存和不带消息保存 上面我演示的是带消息所以能看出来我保存的是哪个分支

所以在使用satsh的时候最好带上当前保存的一个标记语 这样方便我们下次使用的时候拉取

```sh
git stash save "markword"
```







