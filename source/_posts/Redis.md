---
title: 浅析redis
date: 2020/06/19
categories:
  - java
  - redis
comments: true
abbrlink: 49197
img:
---

# Redis

## 帮助文档

 https://cloud.tencent.com/developer/section/1374167

 redis.cn



## 安装流程

### 1. yum install wget

### 2. wget  www.***redis .tar.gz(去github找最新的包)

### 3. tar xf    redis.tar.gz

### 4. cd redis-src

### 5. README.md

### 6. make

make yum install gcc   
make distclean

### 7. make

cd src  ... 生成了可执行程序

### 8. cd ..

### 9. make install  PREFIX=/opt/xxxx/redis5

### 10. vi /etc/profile

### 11. cd utils

最后写入
... export REDIS_HOME =/opt/xxx/redis5
... export PATH=$PATH:$REDIS_HOME/bin
source /etc/profile

### 12. ./install_server.sh

- 一个物理机中可以有多个redis实例. 通过port区分
- 可执行程序就有一份在目录, 但是内存中需要多个实例有各自的配置文件
- 批处理server redi_6379 start/stop/status > linux /etc/init.d/***

## 



## 存储类型

### key

### value

- String(byte)

	- 字符类型

		- set k1 test  nx

			- nx 如果存在则不能插入成功, 只能用来新增

		- set k1 test xx

			- xx 如果不存在的话就失败, 只能用来更新

		- append

			- 追加

		- range

			- setrange
			- getrange

		- strlen
		- type

			- 命令是哪个分组的就是哪种类型

		- getset

			- 更新值并且拿到老值

		- MSETNX

			- 原子性批量操作  要么都成功 要么都失败

	- 数值类型

		- incr
		- incrby

	- bitmap

		- 一个字节(bit)有8个二进制位

			- 1B=8 Bit
1KB＝1024B 
1MB＝1024KB 
1GB＝1024MB 
1TB=1024GB

		- SETBIT key offset value

			- 设置key 偏移量offset 的 0 或者1

		- BITPOS key bit [start] [end]

			- 查找字符串中第一个设置为1或0的bit位 ,返回该位置

				- bitpos k1 1 0 0 

					- 1

				- bitpos k1 1 1 1 

					- 9

		- BITCOUNT key [start end]

			- 返回是1的数量
			- start 和end是字节位置

		- BITOP operation destkey key [key ...]

			-  AND  

				- 按位与操作  全1是1  其余为0

			- OR

				- 按位或操作  有1是1  其余为0

			- XOR
			- NOT

- lists(双向链表)

	- lpush

		- 左添加 进入队列在链表的左边依次开始添加

	- lpop

		- 左弹出

	- rpush

		- 右添加 进入队列在链表的最右边依次添加

	- rpop 

		- 右弹出

	- lrange 

		- 得到固定范围内的所有值

	- lrem 

		- 移除几个什么类型的元素

	- linsert

		- 只在找到第一个值的地方

			- after 后面插入
			- before 前面插入

	- blpop (阻塞,单播队列)

		- 拿到里面的第一个数据 拿不到则阻塞

- Hashes ( 点赞,收藏,详情页)

	- 存储对象
name::age
name::sex
name::weight
	- hset key  field value 

		- 添加一个对象的属性值

	- hmset 

		- 添加一个对象的多个属性值

	- hmget

		- 获取一个对象的多个属性值

	- hvals

		- 获取这个对象的所有属性值

	- hgetall

		- 拿到这个对象的所有key value

	- HINCRBYFLOAT key field increment

		- 对指定的一个属性进行 +1 -1

- sets ( 去重)

	- SADD key member [member ...] 

		- 添加元素

	- smembers

		- 去重查看

	- srem 

		- 去除某个value

	- sinter key...

		- 返回  多个key 的交集

	- sinterstore  

		- 将交集结果 保存到指定key

	- sunion

		- 去重并且打印出 并集

	- sdiff (差集)

		- 前面的key 与后面的key 做差集

	- Srandmember key count(随机事件)

		- count 正数 得到一个去重的结果集
		- count 负数 得到一个带重复的结果集

	- spop

		- 随机取出一个数据,并删除集合里面的该值

- sorted sets(排序)

	- zadd 

		- 添加一个带数值的set

	- ZRANGE key start stop [WITHSCORES]

		- 得到一个带有起始位置的排序组合 

	- ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]

		- 最小值和最大值的一个区间组合

	- zrevrange

		- 逆序输出

	- zscore  

		- 通过元素取到分值

	- ZRANGE key start stop [WITHSCORES]

		- 通过元素取到排名

	- ZINCRBY key increment member

		- 对指定key的分值进行加减

	- ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]

		- 获取交集和并集 权重和集合指令

## 管道

### http://redis.cn/topics/pipelining.html

## 发布订阅

###  client

- 历史数据

	- 全量数据  -- 数据库
	-  		短时间内  -- sorted set 

- 实时数据

	- PUBLISH key value 发布
	- SUBSCRIBE  key  监听

		- 先监听才能收到发布的消息

## 事务

### mutil

- 开始事务

### exec

- 执行 哪个先到就执行哪个事务

### watch

- 乐观锁 CAS 监控某个值是否发生了变更

## 布隆过滤器

### https://github.com/RedisBloom/RedisBloom

### redis-server --loadmodule /path/to/redisbloom.so

### 黑名单拦截

### 垃圾邮箱拦截

### 凡是某个值需要经过判断大量数据是否存在并且误报的影响较小的情况下可以使用

### 一旦一个值必定不存在的话，我们可以不用进行后续昂贵的查询请求。会有误判就是可能存在但是不一定存在的情况,

## Redis使用

### 安装使用

- redis-cli

	- 进入redis
	- -p 指定端口
	- -d 指定DB
	- help 帮助文档

- 在使用前 需要规定编码格式  因为是字节流 二进制安全, 不存在客户端解析问题, 所以需要统一加码解码
- kernel内存响应时间是纳秒  如果在同一时间有十多万个并发来请求redis服务端 那么会有秒级的延迟

	- 所以内存寻址每个大约在纳秒级别响应

- install_server.sh

	- 创建指定端口的配置文件

### 作为缓存使用

- maxmemory 配置内存尽量控制在1G-10G 
- 设置内存满了之后的拒绝策略

	- lfu 碰了多少次
	- lru 多久没碰

- key的有效期设置

	- 主动

		- 主动访问的时候发现过期删除

	- 被动

		- 随机抽查20个key
		
		- 删除已经过期的
		
		- 如果发现超过25%的key过期那么重复第一步骤
		
		  

### 作为数据库使用

​	目前作为缓存的较多 ,作为数据库比较少 因为存在分布式情况下的数据一致性问题 以及备份问题不推荐使用redis做数据库



## 缓存集群



### 目前单节点的问题

- 单点故障

  看集群搭建

- 压力有限

  看集群搭建

- 容量有限

	- 业务节点拆分不同的redis
	- 算法拆分 (sharding)

		- hash+ 取模 (modula )
		- random

			- 消息队列 一边lpush  一边只需要去rpop

		- kemata (一致性hash算法)

			- 1. 有一个首尾相连的环 在这个环上有你的物理节点
			- 2. 业务数据过来后通过hash算法最后会有一个结果落在这个环的某个位置,也就是虚拟点
			- 3. 虚拟点去找后面的物理节点然后保存到redis集群中
			- 优点 : 后续加节点 会分担其他节点的压力,并不会造成全局洗牌
			- 缺点: 新增的节点有部分数据不能命中缓存 导致缓存击穿, 压到mysql数据库中

				- 解决方案 : 去寻找最近的两个节点的缓存数据

### AKF

- X 轴 全量 镜像
- Y 轴 业务 功能
- Z轴 指定规则将指定的key放在不同的库里面



### 集群搭建

- 主节点

	- 当子节点挂在主节点后 主节点会受到一个子节点的消息

- 子节点

	- 启动的时候 添加配置  

		- redis-server ./6380.conf  --REPLICAOF  IP  port --appendonly yes

- redis 默认使用的是弱一致性
- 主从复制配置

  - replica-serve-stale-data yes

  	- 是否在同步期间提供给第三方数据

  - replica-read-only yes 

  - repl-backiog-size 1mb

### 监控(Sentinel)

- 监控

	- port 26379
sentinel monitor master1 127.0.0.1 6381 2
	- redis-server  ./26379.conf --sentinel

## 分布式锁

​		目前纯redis 的分布式锁都不是很好实现 

​		可以结合zookeeper