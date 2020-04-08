---
title: redis-bitmap
date: 2020/04/03
categories:
  - java
  - redis
comments: true
abbrlink: 49197
img:

---



 # 1. BitMap 是什么


在redis中可以设置用一个bit的位置来标识某个元素的0-1状态, 

具体的操作指令是  指定某个key的offset的0或者1的状态

```redis
SETBIT key offset value
```



计算某个key的被设置为 `1` 的比特位的数量

```
BITCOUNT key [start end]
```

# 2. BitMap 怎么用

```java
/**
 * 设置reids的bitmap位置
 * @param key
 * @param offset 根据需求设置long int
 * @param value
 * @return
 */
public Boolean setBit(String key,int offset, Boolean value) {
    return redisTemplate.execute((RedisCallback<Boolean>) con -> con.setBit(key.getBytes(),(long)offset,value));
}

```



```java

/**
 * 统计key的所有1值
 * @param key
 * @return
 */
public long bitCount(String key) {
    return redisTemplate.execute((RedisCallback<Long>) con -> con.bitCount(key.getBytes()));
}
```



# 3. BitMap 什么时候用



- ## 短信验证一分钟不能发送5次

### 之前我们的做法

调用reids的increment方法来自增完成需求

### 现在我们的做法

将当前用户当做key ,拿到当前分钟的毫秒数做offset,最后用bitcount来统计用户在当前分钟数访问次数.



- ## 记录用户当前是否在线

### 之前我们的做法

我们需要在数据库字段里标志处是否离线

### 现在我们的做法

找到我们需要的key, 拿用户id做offset,最后我们标记出是否在线 还可以统计当前在线总人数





