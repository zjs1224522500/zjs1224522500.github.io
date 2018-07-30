---
title: 'Redis数据类型'
date: 2017-12-7 20:00:29
tags: Redis
---
## Redis数据类型
- Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)

<!-- more -->
#### string
- `string`是`redis`最基本的类型，你可以理解成与`Memcached`一模一样的类型，一个`key`对应一个`value`。
- `string`类型是二进制安全的。意思是`redis`的`string`可以包含任何数据。比如jpg图片或者序列化的对象 。
- `string`类型是`Redis`最基本的数据类型，一个键最大能存储**512MB**。
```
127.0.0.1:6739 > set name elvis
OK
127.0.0.1:6739 > get name
"elvis"
127.0.0.1:6739 > getset name shunzi
"elvis"
127.0.0.1:6739 > get name
"shunzi"
127.0.0.1:6739 > del name
(interger) 1
127.0.0.1:6739 > get name
(nil)

// 使用自增自减符，若变量不存在则初始化为0 
127.0.0.1:6739 > incr num
(interger) 1
127.0.0.1:6739 > get num
"1"
127.0.0.1:6739 > incr num
(interger) 2
127.0.0.1:6739 > get num
"2"
127.0.0.1:6739 > incr name
(error) ERR value is not an integer or out of range
127.0.0.1:6739 > decr num
(interger) 1
127.0.0.1:6739 > get num
"1"
127.0.0.1:6739 > decr num2
(interger) -1

// incryby decrby
127.0.0.1:6739 > incrby num 5
(interger) 6
127.0.0.1:6739 > incrby num3 5
(interger) 5
127.0.0.1:6739 > decrby num 3
(interger) 3
127.0.0.1:6739 > decrby num4 3
(interger) -3

// append返回拼接的字符串长度
127.0.0.1:6739 > append num 5
(interger) 2
127.0.0.1:6739 > get num
"35"
127.0.0.1:6739 > append num5 123
(interger) 3
127.0.0.1:6739 > get num5 
"123"
```

#### Hash
- Redis hash 是一个键值(key=>value)对集合。
- Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。
- 每个 hash 可以存储 232 -1 键值对（40多亿）。
---
- `HSET key field value`//设置键以及其中的属性（单个属性）
- `HMSET key field value key value...`//设置键以及其中的属性(多个属性)
- `HGETALL key`// 获取该键对应的所有属性名和属性值
- `HGET key field` //获取该健对应的属性对应的值
- `HMGET key field1 field2`//获取该键对应的多个属性的值
- `HDEL key field1 field2`//删除对应的键对应的属性
- `del key`//删除键对应的value
- `HEXISTS key field`//判断键对应的对象中是否存在field属性,存在返回1 否返回0
- `HLEN key`//获取键对应的属性个数
- `hkeys key`//获取键对应的属性的属性名
- `hvals key`//获取键对应的属性 的属性值
```
127.0.0.1:6379> HMSET user:1 username zhangjianshun password 123456 age 18 college UESTC
OK
127.0.0.1:6379> HGETALL user:1
1) "username"
2) "zhnagjianshun"
3) "password"
4) "123456"
5) "age"
6) "18"
7) "college"
8) "UESTC"
127.0.0.1:6379> HGET user:1 college
"UESTC"
127.0.0.1:6379> hincrby user:1 age 2
(interger) 20
127.0.0.1:6379> hexists user:1 username
(interger) 1
```

#### List
- Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。
- 列表最多可存储 232 - 1 元素 (4294967295, 每个列表可存储40多亿)。
- `lpush key value1 value2 ...`//从左开始添加
- `rpush key value1 value2 ...`//从右开始添加
- `lrange key index1 index2`//从左开始查看index1 ~ index2,若index为负数，则倒数对应的个数
- `lpop key`//左端弹出，之后list中该元素
- `rpop key`//右端弹出
- `llen key`//获取元素个数
- `lpushx key value`//仅仅当key存在时才进行插入，同理rpushx
- `lremo key count value`//删除指定count个值为value的元素，若count<0，删除顺序反向，若count=0，则删除所有的对应的value
- `lset key index value`//设定对应的角标的值，index若为-1，则为末尾
- `linsert key before/after value newValue`//在指定元素前后插入新元素 
---
##### 常用于消息队列的使用场景
- `rpoplpush key1 key2`//弹出key1的一个元素压入key2

```cmd
redis 127.0.0.1:6379> lpush database redis
(integer) 1
redis 127.0.0.1:6379> lpush database mongodb
(integer) 2
redis 127.0.0.1:6379> lpush database rabitmq
(integer) 3
redis 127.0.0.1:6379> lrange database 0 10
1) "rabitmq"
2) "mongodb"
3) "redis"
redis 127.0.0.1:6379>
```

#### Set
- `Redis`的`Set`是`string`类型的无序集合。不允许出现重复的元素，
- 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。
- 集合中最大的成员数为 232 - 1(4294967295, 每个集合可存储40多亿个成员)。
---
- `sadd key member`// 添加一个string元素到,key对应的set集合中，成功返回1,如果元素已经在集合中返回0,key对应的set不存在返回错误。
- `srem key value1 value2`//删除key对应的set中指定的value
- `smembers key`//列举key对应的set中的所有value
- `sismember key value`//判断在该集合中是否存在该元素，返回布尔值
- `sdiff key1 key2`//两个集合的差集运算
- `sinter key1 key2`//两个集合的交集运算
- `sunion key1 key2`//两个集合的并集运算，自动去重
- `scard key`//集合对应的元素个数
- `srandmember key`//随机返回集合中的成员
- `sdiffstore key1 key2 key3`//求交集之后存储到kwy3，同理差集并集

##### 应用场景
- 获取唯一性数据
- 用于维护数据对象之间的关联关系（集合运算）


```cmd
redis 127.0.0.1:6379> sadd runoob redis
(integer) 1
redis 127.0.0.1:6379> sadd runoob mongodb
(integer) 1
redis 127.0.0.1:6379> sadd runoob rabitmq
(integer) 1
redis 127.0.0.1:6379> sadd runoob rabitmq
(integer) 0
redis 127.0.0.1:6379> smembers runoob

1) "rabitmq"
2) "mongodb"
3) "redis"
```

#### zset (sorted set：有序集合)
- Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。
- 不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。
- zset的成员是唯一的,但分数(score)却可以重复。
- `zadd key score member `//添加元素到集合，元素在集合中存在则更新对应score
- `zscore key member`//获取对应的分数
- `zcard key`//获取对应的元素个数
- `zrem key mem1 mem2`//删除集合中对应的元素
- `zrange key index1 index2`//显示对应的下表范围内的元素，可使用参数`withscores`来决定是否显示对应的参数。默认由小到大排序
- `zrevrange key index1 index2`//从大到小
- `zrenrangebyrank key index1 index2`//根据顺序删除指定范围的元素
- `zrenrangebyscore key index1 index2`//根据分数范围删除指定的元素
- `zrangebyscore key1 index1 index2 withscores`//显示分数在指定范围内的元素，还可以使用`limit index1 index2`限定返回的个数
- `zincrby key score member`//指定成员的分数加分score
- `zcount key score1 score2`//显示指定的分数范围内的元素个数
---
##### 应用场景：
- 积分排行榜
- 构建数据索引

```cmd
redis 127.0.0.1:6379> zadd runoob 0 redis
(integer) 1
redis 127.0.0.1:6379> zadd runoob 0 mongodb
(integer) 1
redis 127.0.0.1:6379> zadd runoob 0 rabitmq
(integer) 1
redis 127.0.0.1:6379> zadd runoob 0 rabitmq
(integer) 0
redis 127.0.0.1:6379> ZRANGEBYSCORE runoob 0 1000

1) "redis"
2) "mongodb"
3) "rabitmq"
```