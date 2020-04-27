---
title: 享元模式
date: 2019/10/8
categories:
  - 设计模式
  - java
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 29384
---



重点是重复利用对象 





类似安卓 的list优化



维持一个池子里的key 

for循环查看key的标签是否可以重复利用 

如果可以则返回

不可以说明都在用 这个时候需要new出来了



