---
title: redis常见知识点总结
date: 2018-03-21 17:40:25
tags: [redis,PHP]
categories: PHP
---
由于redis在项目中的广泛使用，几乎所有的后端技术都会受用到redis，一直以来也没有总结过，今天，算是总结一下redis的常用知识点，对于自己的了解巩固下吧。
<!--more-->

## redis常用的数据结构

字符串string、哈希表hash、链表list、集合set、有序集合sorted set

### string字符串类型

>使用场景

1. 字符串类型是redis最基础的数据结构，他最经典的使用场景是作为数据缓存层，mysql或者其他db作为落地存储层，绝大部分的请求是都在redis缓存层中获取，所以能降低后端的压力和加速拉取速度。
2. 许多计数需求也会使用redis string来计数，可有实现高并发下的快速计数功能

>常用命令

set 直接将指定的key设置字符串value，如果该key已经存在则覆盖其原有的值

```
#将key=value对应存储
set key value 
```

get 获取指定key的value

```
get key
```

append 给指定的key的value追加一个string

```
##给指定的key的value后面追加一个.bak 为value.bak
append key .bak 
```

mget,mset 批量设置和获取

```
#批量设置
mset key1 value1 key2 value2 key3 value3 ....

#批量获取 返回指定的key的value
mget key1 key2 key3....

```

DECR将key值减一操作如果该key不存在默认初始化为0后减一操作
INCR将key值加一操作如果该key不存在默认初始化为0后加一操作

```
# -1
DECR key 
# +1
INCR KEY
```

### Hash 哈希表

>使用场景

主要使用储存对象信息，不需要将对象序列化存储后再反序列化后修改在存储，直接使用hash存储的hashmap存储可以减少开销，同时维护起来也比较方便

>常用命令

Hset命令用于为哈希表中的字段赋值 。如果哈希表不存在，一个新的哈希表被创建并进行 HSET 操作。如果字段已经存在于哈希表中，旧值将被覆盖。如果字段是哈希表中的一个新建字段，并且值设置成功，返回 1 ，如果哈希表中域字段已经存在且旧值已被新值覆盖，返回 0 

```
HSET KEY FIELD VALUE 
```

Hget 命令用于返回哈希表中指定字段的值,返回给定字段的值。如果给定的字段或 key 不存在时，返回 nil 
Hgetall 用于返回哈希表中，所有的字段和值在返回值里，紧跟每个字段名(field name)之后是字段的值(value)

```
HGET myhashkey

HGETALL myhashkey

```

其他命令 :
key field1 \[field2\] 删除HDEL 
HEXISTS key field 查看哈希表 key 中，指定的字段是否存在。
HINCRBY key field increment 为哈希表 key 中的指定字段的整数值加上增量 increment 。如果哈希表不存在，一个新的哈希表被创建并进行

### List 

>使用场景

list的使用场景很多主要的有几种一个是存储消息，列表之类的，比如最新消息，关注列表等还有一个比较重要的就是消息队列使用

>常用命令

```
#将一个或多个值插入到列表头部
LPUSH/RPUSH key value1 [value2] 

#移出并获取列表的第一个元素
LPOP/RPOP key 

#获取列表长度
LLEN key 

```


### Set

>使用场景

set的功能和list十分类似，set的特殊之处在于在对于内部的值会进行排重功能，同时还可以判断改值是否在这个集合当中，可以用户关注 粉丝等一些应用中

>常用命令

```

#向集合添加一个或多个成员
SADD key member1 [member2] 

#移除并返回集合中的一个随机元素
SPOP key 

#返回集合中的所有成员
SMEMBERS key 

#返回所有给定集合的并集
SUNION key1 [key2] 

#移除集合中一个或多个成员
SREM key member1 [member2] 
```

### Sorted Set

>使用场景

Sorted Set和set是类似的，但是不同之处在于Sorted Set提供一个score参数来为成员排序，并且插入的数据都是有序的，可以应用为时间线等应用

>常用命令

```
#向有序集合添加一个或多个成员，或者更新已存在成员的分数
ZADD key score1 member1 [score2 member2] 

#通过索引区间返回有序集合成指定区间内的成员
ZRANGE key start stop [WITHSCORES]

#移除有序集合中的一个或多个成员
ZREM key member [member ...] 

#返回有序集中，成员的分数值
ZSCORE key member 
```


