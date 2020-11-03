---
title: 浅析网路协议及LVS
date: 2015/5/2
categories:
  - 前端
tags:
  - hexo
comments: true
abbrlink: 49197
img:
---

# HTTP

## 七层协议

### <font color = #F50A0A >应用层</font>

-  用户态   - 浏览器 -生成各种 HTTP/1.1  只需要写好规则

### <font color = #F50A0A >表示层</font>

### <font color = #F50A0A >会话层</font>

### <font color = #F50A0A >传输控制层 (相当于操作系统的接口)</font>

- 给内核态 socket

	- 一台主机拥有的端口号是65535个

- tcp协议 (可靠协议)

	- 三次握手 之后建立资源消耗

		- 开辟资源后返回给应用层来 传输数据 操作

	- 四次挥手

		- 要求双方都发送一个分开链接和接受一次分开的包

	- tcp 是 基于下一跳的规则 每个节点只需要记录下一个节点的地址
	- 组织TCP协议的包然后发送给网络层 并且在当前阻塞住

- udp协议

### <font color = #F50A0A >网络层</font>

- netstat
- TCP/IP

	- 拿到包之后 通过本地的ip地址寻找相对于的路由表
	- route -n  
ip 和 Genmask(子网掩码)   按位与运算  得到 Destination 找到(gateWay)网关

### <font color = #F50A0A >链路层</font>

- TCP/IP

	- 找到mac 地址  封装包给下一层

		- arp -a  

### <font color = #F50A0A >物理层</font>

- 三层确定对方地址 

	- 源mac地址 -> 目标mac地址 (会不停地找下一跳  所以会变)
	- 源IP地址 -> 目标IP地址
	- 源端口号 -> 目标端口号
	- 建立连接的包

		- sync

			- sync/ack

				- ack

	- 断开连接的包

		- finl



##  七层协议出去之后

基于lvs的三种模式

### D-NAT

- 出去的时候肯定是 CIP 和 VIP 去寻找下一跳
- 然后会找到具体的VIP地址来确定DIP
- 最后访问RIP 回去的时候按照原路返回

### <font color = #6AACDE >DR模式 基于两层</font>

- CIP -> VIP -> DIP -> RIP -> VIP(对外隐藏)

	- 出去的时候肯定是 CIP 和 VIP 去寻找下一跳
	- 然后会找到具体的VIP地址来确定DIP
	- 在找到DIP的地址之后会封装一个  ( CIP VIP | RIP @ MAC  ) 的包
	- 最后访问RIP 然后RIP根据包里面的CIP来返回数据

- 路由器ip地址由 运营商提供 -> 去寻找VIP 
出去的时候为了防止同一局域网出去的ip一样会在路由器内部生成一个私有的端口号来区别
- 速度快, 成本低, 负载和RS要在同一局域网

### TUN隧道技术

- vpn & 翻墙软件



## LVS

主备



## nginx 

 nginx 和server建立三次连接  
 lvs 和nginx 建立通讯


七层协议每层都有相对应的规范

