---
title: linux开放端口
date: 2018/5/18
categories:
  - linux
abbrlink: 29383
tags:
---
文件local
```
/etc/sysconfig/iptables
```
添加

```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 5236 -j ACCEPT
```

生效
```
service iptables restart
```



-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 3000:5000 -j ACCEPT
