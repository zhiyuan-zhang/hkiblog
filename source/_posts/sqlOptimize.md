---
title: 当问你sql优化的时候你能说什么
date: 2019年5月13日
categories:
  - 后端
tags:
  - sql
  - 优化
  - 数据库
  - db
comments: true
abbrlink: 49197
img:
---


本质上来讲 sql优化和数据库优化是两种优化

数据库优化包含的种类较多 软件优化,硬件优化

理论上 单个mysql数据库能够支撑的是每秒2000的并发 极限是5000

当然本次不说数据库优化 先说sql优化

sql优化一般是针对单个业务进行优化

比如秒杀 系统的订单查询 (当然一般用redis 这里只是举个例子)

或者说是对excel表格进行分析插入数据库

等等各种各样的业务



当然一个简单的sql 可能就是这样的 

```sql
SELECT * from test WHERE ASSIGNEE_ = 'user1'
```

但在实际业务环境中可能并不是这样

实际会复杂很多 以及很多条件

当我们在处理这样的sql的时候 应该怎么去优化以及从哪里入手

# 索引

我们大家都是知道索引可以很好的帮我们来提高效率

但是具体怎么用 以及针对某个业务或者单条sql怎么优化

再说这些之前我们先针对SQL进行一些常识性优化  比如


## 基本优化
**WHERE 子句里面的列尽量被索引**

**尽量避免使用 “SELECT *”**

**如果用到分页 尽量使用物理分页 并非逻辑分页**

**join列尽量使用索引**

**order by 使用索引**

等等  

总之是**为了避免全表扫描做出的各种操作**

大家应该发现索引的使用还是非常频繁的

那么具体某个sql使用了哪些索引 以及进行了什么处理操作

# EXPLAIN 优化

我们可以用  EXPLAIN 关键字去查看执行计划

```sql
EXPLAIN SELECT *
FROM professor a
WHERE NOT EXISTS (
		SELECT *
		FROM sys_attend b
		WHERE a.id = b.professor_id
			AND b.`type` = 1
			AND b.table_id = 93353728
	)
	AND a.polling_status = 1
	AND a.status = 1
	AND (a.Member_category = 1
		OR a.Member_category = 3)
	AND (organization_category IN (4, 7)
		OR organization_categoryvice IN (4, 7))
LIMIT 0, 2

```

 id	 | select_type	|table	|type |	possible_keys| key | key_len|	ref |	rows	| Extra 
:-: | :-: | :-: | :-: | :-:| :-: | :-: | :-: | :-:| :-:
1	|"PRIMARY"	| "a"|	"index_merge"	|"professor_level_index,professor_levelvice_index"	| "professor_level_index,professor_levelvice_index" |	"2,2"	| NULL	| 28 |	"Using sort_union(professor_level_index,professor_levelvice_index); | Using where" |
2	|"DEPENDENT SUBQUERY"|	"b"	|"ALL"	|NULL	|NULL	|NULL	|NULL	|85	|"Using where"

这张表里大概有这么几个字段

1. id 执行顺序 可以重复

  - id相同则从上往下执行
  - id从大到小执行

2. select_type 搜索类型 一共有十一种 具体可以看这篇博客[SQL_explain操作解释](https://blog.csdn.net/y1193329479/article/details/78821126#27_DEPENDENT_SUBQUERY_134) 这里就简单的介绍常见的几种 

  - 第一种也是最常见到的一种 SIMPLE 简单的select查询,查询中不包含子查询或者union;

  - PRIMARY:查询中若包含任何复杂的子查询,最外层查询则被标记; 就是刚刚的优先级最外层则会显示PRIMARY

  - SUBQUERY:在select或者where列表中包含了子查询;


3. type 显示的是访问类型，一般我们优化sql就是着重优化这个 , 是较为重要的一个指标，结果值从好到坏依次是：
system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL。 一般来说，得保证查询至少达到range级别，最好能达到ref。 那么怎么能达到这个级别呢 

  - ref:非唯一性索引扫描,返回匹配某个单独值得所有行,本质上也是一种索引访问,它返回所有匹配某个单独值的行,然而,它可能会找到多个符合条件的行,所以它应该属于查找和扫描的混合体;

  - range:只检索给定范围的行,使用一个索引来选择行,key列显示使用哪个索引,一般就是在你的where语句中出现了between,<,>,in等的查询；这种范围索引扫描比全表扫描要好,因为它只需要开始于索引的某一个点,结束于另一个点,不用扫描全部索引; 之前说的避免全表扫描就是为了这样



4. possible_keys:显示可能会被应用到这张表的索引,一个或者多个;查询涉及到的字段上若存在索引,则该索引将被列出,但不一定被查询实际使用到;

5. key:实际使用到的索引.如果为null,则没有使用索引;查询中若使用了覆盖索引,则该索引仅出现在key列表中;

6. key_len:表示索引中使用的字节数,可通过该列计算查询中使用的索引的长度,在不损失精确性的情况下,长度越短越好; key_len显示的值为索引字段的最大可能长度,并非实际使用长度,即key_len是根据表定义计算而得,不是通过表内检索出的;

7. ref:显示索引的哪一列被使用了,如果可能的话,是一个常数,哪些列或常量别用于查找索引列上的值;

8. rows:根据表统计信息及索引选用情况,大致估算出找到所需的记录所需要读取的行数;

9. Extra:包含不适合在其它列中显示但十分重要的额外信息:
  - 对于这个消息栏里展示的东西有很多 一般是你的sql违反的数据库相对应的算法,他认为你的不合理
  - 在这里就不详细解释了 当大家出现后可以百度相对应的信息 [MySQL中explain执行计划中额外信息字段(Extra)详解](https://blog.csdn.net/poxiaonie/article/details/77757471)


那么根据这些信息我们可以查到需要优化哪些地方

说完 explain 我们可以再说说 explain extended + show warnings 

在执行explain extended  之后我们在 show warnings 可以看到在数据库中我们sql的执行方式, 我们可以在这个基础上再次进行优化


我们可以看到sql的执行方式，对于分析sql还是很有帮助的。

( 8 ) SELECT ( 9 ) DISTINCT ( 11 ) < Top Num > < select list > 

( 1 ) FROM [ left_table ] 

( 3 ) < join_type > JOIN < right_table > 

( 2 ) ON < join_condition > 

( 4 ) WHERE < where_condition > 

( 5 ) GROUP BY < group_by_list > 

( 6 ) WITH < CUBE | RollUP > 

( 7 ) HAVING < having_condition > 

( 10 ) ORDER BY < order_by_list >

从优先级我们可以看出为什么order by 排序在 group by 之前不生效了

之前那条sql之后后数据库warnings的执行方式是下面这样的
```sql
SELECT `zwkj_zhxt`.`a`.`id` AS `id`,
        `zwkj_zhxt`.`a`.`name` AS `name`,
        `zwkj_zhxt`.`a`.`sex` AS `sex`,
FROM `zwkj_zhxt`.`professor` `a`
WHERE ((`zwkj_zhxt`.`a`.`status` = 1)
        AND (`zwkj_zhxt`.`a`.`polling_status` = 1)
        AND (not(exists(/* select#2 */SELECT 1
FROM `zwkj_zhxt`.`sys_attend` `b`
WHERE ((`zwkj_zhxt`.`b`.`table_id` = 93353728)
        AND (`zwkj_zhxt`.`b`.`sex` = 1)
        AND (`zwkj_zhxt`.`a`.`id` = `zwkj_zhxt`.`b`.`professor_id`)))))
        AND ((`zwkj_zhxt`.`a`.`name` = 1)
        OR (`zwkj_zhxt`.`a`.`name` = 3))
        AND ((`zwkj_zhxt`.`a`.`sex` IN (4,7))
        OR (`zwkj_zhxt`.`a`.`sex` IN (4,7)))) limit 0,2
```

从上面还可以看出 and 的优先级 总是高于 or 

但有一点需要注意的是 exteneded得到的sql并不是 最终优化执行的sql
这一点可以在官方文档中得到确认[Extended EXPLAIN Output Format](https://dev.mysql.com/doc/refman/8.0/en/explain-extended.html)
但从优化的角度上来讲也能帮助我们.


一般企业数据库的优化 基本上是

配硬件+差不多的sql优化+分布式+分库分表+读写分离

等等 反正一系列的操作 基本满足80%的业务场景

当然我们优化个别业务需要根据实际场景来优化

如果确实说需要高级别的维护和优化 公司会有相对应的运维或者DB来做很少需要开发者担心




