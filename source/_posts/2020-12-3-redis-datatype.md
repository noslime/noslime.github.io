---
title: redis datatype and command
date: 2020-12-3 11:14:23
author: noslime
top: false
cover: false
categories: 数据库
tags: REDIS
keywords:  redis命令
summary: 因为redis的数据类型大都比较简明易懂，也拥有着极其丰富中英文社区文档，所以仅以此文总结下常用的数据类型及其操作命令简介，主要目的为学习一遍数据类型及其相关命令，也方便今后自己查阅。
---

因为redis的数据类型大都比较简明易懂，也拥有着极其丰富中英文社区文档，所以仅以此文总结下**常用的数据类型**及其操作命令==简介== ，主要目的为学习一遍数据类型及其相关命令，方便今后自己查阅。若要查阅命令详情，请访问[redis Commands](https://redis.io/commands)  或 [redis 中文](http://www.redis.cn/commands.html)。

> 当前redis版本 ==6.2.0==



# 概述

redis的数据类型常见的有字符串（String）、哈希表（Hash）、列表（List）、集合（Set）、有序集合（zset）五种，另外还有位图（bitmaps）、HyperLogLogs、Streams。本文主要记录下前五种数据类型及命令简介。

---

## Redis keys

redis键是二进制安全的，这意味着可以使用任何二进制序列作为键，比如ipg图片或者序列化的对象， 空字符串也是有效的键。

键允许的最大值为512M。

### 相关命令

| 命令      | 起始版本  | 时间复杂度 | 基本语法                             | 简述                                                         |
| :-------- | --------- | ---------- | ------------------------------------ | :----------------------------------------------------------- |
| TYPE      | **1.0.0** | O(1)       | TYPE  key                            | 用于返回 key 所储存的值的类型。                              |
| PEXPIREAT | **2.6.0** | O(1)       | PEXPIREAT key milliseconds-timestamp | 设置 key 到什么时候过期，以毫秒计。                          |
| RENAME    | **1.0.0** | O(1)       | RENAME key newkey                    | 修改 key 的名称                                              |
| PERSIST   | **2.2.0** | O(1)       | PERSIST  key                         | 移除 key 的过期时间，key 将持久保持。                        |
| MOVE      | **1.0.0** | O(1)       | MOVE key db                          | 将当前数据库的 key 移动到给定的数据库 db 当中。              |
| RANDOMKEY | **1.0.0** | O(1)       | RANDOMKEY                            | 从当前数据库中随机返回一个 key 。                            |
| DUMP      | **2.6.0** | O(1)       | DUMP  key                            | 序列化给定 key ，并返回被序列化的值。                        |
| TTL       | **1.0.0** | O(1)       | TTL  key                             | 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。 |
| EXPIRE    | **1.0.0** | O(1)       | EXPIRE key seconds                   | 为给定 key 设置过期时间。                                    |
| DEL       | **1.0.0** | O(N)       | DEL key [key ...]                    | 该命令用于在 key 存在时删除 key。                            |
| PTTL      | **2.6.0** | O(1)       | PTTL key                             | 以毫秒为单位返回 key 的剩余的过期时间。                      |
| RENAMENX  | **1.0.0** | O(1)       | RENAMENX key newkey                  | 仅当 newkey 不存在时，将 key 改名为 newkey 。                |
| EXISTS    | **1.0.0** | O(1)       | EXISTS key [key ...]                 | 判断当前key是否存在                                          |
| EXPIREAT  | **1.2.0** | O(1)       | EXPIREAT key timestamp               | EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。 |
| KEYS      | **1.0.0** | O(N)       | KEYS pattern                         | 查找所有符合给定模式( pattern)的 key 。                      |

---

## Redis Strings

字符串是最基本的Redis值。 Redis Strings是二进制安全的，这意味着Redis字符串可以包含任何类型的数据，例如JPEG图像或序列化的JAVA对象。字符串值的长度可以达到最大512兆字节。

### 相关命令

| 命令        | 起始版本  | 时间复杂度 | 基本语法                                                     | 简述                                                         |
| :---------- | --------- | ---------- | ------------------------------------------------------------ | :----------------------------------------------------------- |
| SETNX       | **1.0.0** | O(1)       | SETNX key value                                              | 只有在 key 不存在时设置 key 的值。                           |
| GETRANGE    | **2.4.0** | O(N)       | GETRANGE key start end                                       | 返回 key 中字符串值的子字符 起始0，结束-1                    |
| MSET        | **1.0.1** | O(N)       | MSET key value [key value ...]                               | 同时设置一个或多个 key-value 对，是原子操作。                |
| MSETNX      | **1.0.1** | O(N)       | MSETNX key value [key value ...]                             | 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。 |
| SETEX       | **2.0.0** | O(1)       | SETEX key seconds value                                      | 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。 |
| PSETEX      | **2.6.0** | O(1)       | PSETEX key milliseconds value                                | 这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间 |
| SET         | **1.0.0** | O(1)       | SET key value [EX seconds\|PX milliseconds\|KEEPTTL] [NX\|XX] [GET] | 设置指定 key 的值<br />>= 2.6.12: Added the EX, PX, NX and XX options.<br/>>= 6.0: Added the KEEPTTL option.<br/>>= 6.2: Added the GET option. |
| GET         | **1.0.0** | O(1)       | GET key                                                      | 获取指定 key 的值。                                          |
| GETBIT      | **2.2.0** | O(1)       | GETBIT key offset                                            | 对 key 所储存的字符串值，获取指定偏移量上的位(bit)。是二进制位的偏移量。 |
| SETBIT      | **2.2.0** | O(1)       | SETBIT key offset value                                      | 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。是二进制位的偏移量。 |
| DECR        | **1.0.0** | O(1)       | DECR key                                                     | 将 key 中储存的数字值减一。                                  |
| DECRBY      | **1.0.0** | O(1)       | DECRBY key decrement                                         | key 所储存的值减去给定的减量值（decrement） 。               |
| STRLEN      | **2.2.0** | O(1)       | STRLEN key                                                   | 返回 key 所储存的字符串值的长度。                            |
| INCR        | **1.0.0** | O(1)       | INCR key                                                     | 将 key 中储存的数字值增一。                                  |
| Incrby      | **1.0.0** | O(1)       | INCRBY key increment                                         | 将 key 所储存的值加上给定的增量值（increment） 。            |
| INCRBYFLOAT | **2.6.0** | O(1)       | INCRBYFLOAT key increment                                    | 将 key 所储存的值加上给定的浮点增量值（increment） 。        |
| SETRANGE    | **2.2.0** | O(1)       | SETRANGE key offset value                                    | 用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。 |
| APPEND      | **2.0.0** | O(1)       | APPEND key value                                             | 如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原来的值的末尾。 |
| GETSET      | **1.0.0** | O(1)       | GETSET key value                                             | 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。   |
| MGET        | **1.0.0** | O(N)       | MGET key [key ...]                                           | 获取所有(一个或多个)给定 key 的值。                          |

---

## Redis List

Redis lists允许重复值，并且基于Linked Lists实现，。这意味着即使在一个list中有数百万个元素，在头部或尾部添加一个元素的操作，其时间复杂度也是常数级别的。LPUSH 命令插入一个新元素到列表头部，而RPUSH命令 插入一个新元素到列表的尾部。当对一个空key执行其中某个命令时，将会创建一个新表。

### 相关命令

| 命令       | 起始版本  | 时间复杂度   | 基本语法                                                     | 简述                                                         |
| :--------- | --------- | ------------ | ------------------------------------------------------------ | :----------------------------------------------------------- |
| LPUSH      | **1.0.0** | O(1) or O(N) | LPUSH key element [element ...]                              | 将一个或多个值插入到列表头部                                 |
| LPUSHX     | **2.2.0** | O(1) or O(N) | LPUSHX key element [element ...]                             | 将一个或多个值插入到列表头部，当且仅当列表存在时插入         |
| RPUSH      | **1.0.0** | O(1) or O(N) | RPUSH key element [element ...]                              | 在列表尾部插入所有指定的值。                                 |
| RPUSHX     | **2.2.0** | O(1) or O(N) | RPUSHX key element [element ...]                             | 只有当键已经存在并持有列表时，才在存储在键的列表尾部插入指定值 |
| LPOP       | **1.0.0** | O(1)         | LPOP key                                                     | 移除并返回存储在列表的第一个元素。                           |
| RPOP       | **1.0.0** | O(1)         | RPOP key                                                     | 移除并返回存储在列表的最后一个元素                           |
| RPOPLPUSH  | **1.2.0** | O(1)         | RPOPLPUSH source destination                                 | 移除列表的最后一个元素，并将该元素添加到另一个列表头部并返回 |
| LREM       | **1.0.0** | O(N+M)       | LREM key count element                                       | 移除列表中指定count数量的与element相同的元素<br /> count=0 移除所有相等元素<br /> count>0 从队首开始移除指定个数<br /> count<0 从队尾开始移除指定个数 |
| LLEN       | **1.0.0** | O(1)         | LLEN key                                                     | 返回列表长度                                                 |
| LINDEX     | **1.0.0** | O(N)         | LINDEX key index                                             | 返回指定索引的元素                                           |
| LINSERT    | **2.2.0** | O(N)         | LINSERT key BEFORE\|AFTER pivot element                      | 在指定元素前后插入元素                                       |
| LSET       | **1.0.0** | O(N)         | LSET key index element                                       | 设置指定索引处的元素                                         |
| LRANGE     | **1.0.0** | O(S+N)       | LRANGE key start stop                                        | 获取列表指定范围内的元素                                     |
| LTRIM      | **1.0.0** | O(N)         | LTRIM key start stop                                         | 截取列表，只保留指定范围内的元素                             |
| LPOS       | **6.0.6** | O(N)         | LPOS key element [RANK rank] [COUNT num-matches] [MAXLEN len] | 返回列表中匹配列表的索引，rank指定返回第几个匹配元素的索引，count指定返回几个匹配元素的索引， maxlen指定比较的次数。 |
| BLPOP      | **2.0.0** | O(1)         | BLPOP key [key ...] timeout                                  | 该命令返回Redis列表中匹配元素的索引该命令返回Redis列表中匹配元素的索引移除并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| BRPOP      | **2.0.0** | O(1)         | BRPOP key [key ...] timeout                                  | 移除并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| BRPOPLPUSH | **2.2.0** | O(1)         | BRPOPLPUSH source destination timeout                        | 移除列表的最后一个元素，并将该元素添加到另一个列表头部并返回，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| LMOVE      | **6.2.0** | O(1)         | LMOVE source destination LEFT\|RIGHT LEFT\|RIGHT             | 原子性地返回并移除存储在源中的列表的第一个/最后一个元素，并将元素推送到存储在目标位置的列表的第一个/最后一个元素。<br /> 此命令可以用来代替不太推荐的`RPOPLPUSH`命令 |
| BLMOVE     | **6.2.0** | O(1)         | BLMOVE source destination LEFT\|RIGHT LEFT\|RIGHT timeout    | 此命令是`LMOVE`的阻塞变体                                    |

---

## Redis Hash

Redis Hashes是字符串字段和字符串值之间的映射，hash特别适合用于存储对象。类似java里面的Map<String,Object>，每个散列最多可存储(2^32)-1个字段值对（超过40亿）。

### 相关命令

| 命令         | 起始版本  | 时间复杂度   | 基本语法                                       | 简述                                                         |
| :----------- | --------- | ------------ | ---------------------------------------------- | :----------------------------------------------------------- |
| HSET         | **2.0.0** | O(1) or O(N) | HSET key field value [field value ...]         | 设置存储在哈希表中一个或多个字段的值                         |
| HSETNX       | **2.0.0** | O(1)         | HSETNX key field value                         | 设置哈希表中一个字段的值，当且仅当字段不存在时起作用。       |
| ~~HMSET~~    | **2.0.0** | O(N)         | HMSET key field value [field value ...]        | 4.0.0以后已弃用，作用同`HSET`                                |
| HGET         | **2.0.0** | O(1)         | HGET key field                                 | 返回存储在键处的哈希中与字段关联的值                         |
| HGETALL      | **2.0.0** | O(N)         | HGETALL key                                    | 返回存储在键处的哈希的所有字段和值。                         |
| HMGET        | **2.0.0** | O(N)         | HMGET key field [field ...]                    | 返回存储在哈希中与指定字段关联的一个或多个值。               |
| HEXISTS      | **2.0.0** | O(1)         | HEXISTS key field                              | 返回字段在哈希中是否存在。1存在 0不存在                      |
| HDEL         | **2.0.0** | O(N)         | HDEL key field [field ...]                     | 删除哈希表中指定的字段                                       |
| HLEN         | **2.0.0** | O(1)         | HLEN key                                       | 返回哈希表中字段的数量                                       |
| HSTRLEN      | **3.2.0** | O(1)         | HSTRLEN key field                              | 返回存储在键处的哈希中与字段关联的值的字符串长度。如果键或字段不存在，则返回0。 |
| HINCRBY      | **2.0.0** | O(1)         | HINCRBY key field increment                    | 为哈希表 key 中的指定字段的整数值加上增量 increment 。返回增长后的值 |
| HINCRBYFLOAT | **2.6.0** | O(1)         | HINCRBYFLOAT key field increment               | 为哈希表 key 中的指定字段的浮点数值加上增量 increment 。     |
| HKEYS        | **2.0.0** | O(N)         | HKEYS key                                      | 返回存储在哈希表中所有的字段名称                             |
| HVALS        | **2.0.0** | O(N)         | HVALS key                                      | 返回存储在哈希表中所有的字段的值                             |
| HSCAN        | **2.8.0** | O(1) or O(N) | HSCAN key cursor [MATCH pattern] [COUNT count] | 用于迭代哈希中的键值对。数据量较小时count不工作。            |

---

## Redis Set

Redis集合是一个无序的字符串合集。它是通过HashTable实现的，你可以以**O(1)** 的时间复杂度完成 添加、删除以及测试元素是否存在的操作。

### 相关命令

| 命令        | 起始版本  | 时间复杂度   | 基本语法                                       | 简述                                                         |
| :---------- | --------- | ------------ | ---------------------------------------------- | :----------------------------------------------------------- |
| SADD        | **1.0.0** | O(1) or O(N) | SADD key member [member ...]                   | 向集合中添加一个或多个元素                                   |
| SISMEMBER   | **1.0.0** | O(1)         | SISMEMBER key member                           | 判断元素是否是集合的成员                                     |
| SPOP        | **1.0.0** | O(1)         | SPOP key [count]                               | 移除并返回集合中的一个或指定数量随机元素                     |
| SRANDMEMBER | **1.0.0** | O(1) or O(N) | SRANDMEMBER key [count]                        | 随机访问集合中的一个或指定数量随机元素并返回                 |
| SREM        | **1.0.0** | O(N)         | SREM key member [member ...]                   | 移除列出的元素                                               |
| SMOVE       | **1.0.0** | O(1)         | SMOVE source destination member                | 将一个集合中的指定元素移动到另一个集合中去                   |
| SCARD       | **1.0.0** | O(1)         | SCARD key                                      | 获取集合的成员数                                             |
| SMEMBERS    | **1.0.0** | O(N)         | SMEMBERS key                                   | 返回集合中的所有成员                                         |
| SSCAN       | **2.8.0** | O(1) or O(N) | SSCAN key cursor [MATCH pattern] [COUNT count] | 迭代集合中的元素                                             |
| SINTER      | **1.0.0** | O(N*M)       | SINTER key [key ...]                           | 返回给定所有集合的交集                                       |
| SINTERSTORE | **1.0.0** | O(N*M)       | SINTERSTORE destination key [key ...]          | 返回给定所有集合的交集并存储在 destination 中                |
| SUNION      | **1.0.0** | O(N)         | SUNION key [key ...]                           | 返回给定所有集合的并集                                       |
| SUNIONSTORE | **1.0.0** | O(N)         | SUNIONSTORE destination key [key ...]          | 返回给定所有集合的并集并存储在 destination 中                |
| SDIFF       | **1.0.0** | O(N)         | SDIFF key [key ...]                            | 返回给定所有集合的差集，只返回第一个集合的差集               |
| SDIFFSTORE  | **1.0.0** | O(N)         | SDIFFSTORE destination key [key ...]           | 返回给定所有集合的差集并存储在 destination 中                |
| SMISMEMBER  | **6.2.0** | O(N)         | SMISMEMBER key member [member ...]             | 返回每个成员是否是存储在集合的成员。 对于每个成员，如果值是集合的成员，则返回1，如果元素不是集合的成员，或者如果键不存在，则返回0。 |

---

## Redis Sorted sets

类似Sets,但是每个字符串元素都关联到一个叫*score*浮动数值（floating number value）。里面的元素总是通过score进行着排序，其中zset的成员是唯一的，但分数（score）却可以重复。

### 相关命令

| 命令             | 起始版本  | 时间复杂度         | 基本语法                                                     | 简述                                                         |
| :--------------- | --------- | ------------------ | ------------------------------------------------------------ | :----------------------------------------------------------- |
| ZADD             | **1.2.0** | O(log(N))          | ZADD key [NX\|XX] [GT\|LT] [CH] [INCR] score member [score member ...] | 将具有指定分数的所有指定成员添加到存储在键处的有序集中。<br />>=2.4：接受多个元素。在2.4版之前的Redis版本中，每次调用都只能添加或更新一个成员。<br/>>=3.0.2：增加了XX、NX、CH和INCR选项。<br/>>=6.2:添加了GT和LT选项。 |
| ZSCORE           | **1.2.0** | O(1)               | ZSCORE key member                                            | 返回有序集合中单个成员的分数                                 |
| ZMSCORE          | **6.2.0** | O(N)               | ZMSCORE key member [member ...]                              | 返回有序集合中一个或多个成员的分数                           |
| ZRANK            | **2.0.0** | O(log(N))          | ZRANK key member                                             | 返回有序集合中指定成员的索引                                 |
| ZINCRBY          | **1.2.0** | O(log(N))          | ZINCRBY key increment member                                 | 有序集合中对指定成员的分数加上增量 increment                 |
| ZCARD            | **1.2.0** | O(1)               | ZCARD key                                                    | 获取有序集合的成员数                                         |
| ZCOUNT           | **2.0.0** | O(log(N))          | ZCOUNT key min max                                           | 计算在有序集合中指定区间分数的成员数                         |
| ZLEXCOUNT        | **2.8.9** | O(log(N))          | ZLEXCOUNT key min max                                        | 在有序集合中计算指定字典区间内成员数量                       |
| ZRANGE           | **1.2.0** | O(log(N)+M)        | ZRANGE key start stop [WITHSCORES]                           | 返回存储在键处的排序集中的指定元素范围。                     |
| ZRANGEBYLEX      | **2.8.9** | O(log(N)+M)        | ZRANGEBYLEX key min max [LIMIT offset count]                 | ZRANGEBYLEX 返回指定成员区间内的成员，按成员字典正序排序, 分数必须相同。 |
| ZRANGEBYSCORE    | **1.0.5** | O(log(N)+M)        | ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]  | 通过分数返回有序集合指定区间内的成员                         |
| ZREVRANGE        | **1.2.0** | O(log(N)+M)        | ZREVRANGE key start stop [WITHSCORES]                        | 返回有序集中指定区间内的成员，通过索引，分数从高到底         |
| ZREVRANGEBYLEX   | **2.8.9** | O(log(N)+M)        | ZREVRANGEBYLEX key max min [LIMIT offset count]              | ZREVRANGEBYLEX 返回指定成员区间内的成员，按成员字典倒序排序, 分数必须相同。 |
| ZREVRANGEBYSCORE | **2.2.0** | O(log(N)+M)        | ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count] | 返回有序集合中指定分数区间内的成员，分数由高到低排序。       |
| ZREVRANK         | **2.0.0** | O(log(N))          | ZREVRANK key member                                          | 返回有序集合中指定成员的索引，有序集成员按分数值递减(从大到小)排序 |
| ZREM             | **1.2.0** | O(M*log(N))        | ZREM key member [member ...]                                 | 从存储在键处的有序集中移除指定的成员。                       |
| ZREMRANGEBYSCORE | **1.2.0** | O(log(N)+M)        | ZREMRANGEBYSCORE key min max                                 | 移除有序集合中给定的分数区间的所有成员                       |
| ZREMRANGEBYLEX   | **2.8.9** | O(log(N)+M)        | ZREMRANGEBYLEX key min max                                   | 移除有序集合中给定的字典区间的所有成员                       |
| ZREMRANGEBYRANK  | **2.0.0** | O(log(N)+M)        | ZREMRANGEBYRANK key start stop                               | 移除有序集合中给定的索引区间的所有成员                       |
| ZPOPMAX          | **5.0.0** | O(log(N)*M)        | ZPOPMAX key [count]                                          | 删除并返回最多count个排序集中得分最高的成员。                |
| ZPOPMIN          | **5.0.0** | O(log(N)*M)        | ZPOPMIN key [count]                                          | 删除并返回最多count个排序集中得分最低的成员。                |
| BZPOPMAX         | **5.0.0** | O(log(N))          | BZPOPMAX key [key ...] timeout                               | `BZPOPMAX`是有序集合`ZPOPMAX`原语的阻塞版本。                |
| BZPOPMIN         | **5.0.0** | O(log(N))          | BZPOPMIN key [key ...] timeout                               | `BZPOPMIN`是有序集合`ZPOPMIN`原语的阻塞版本。                |
| ZUNION           | **6.2.0** | O(N)+O(M*log(M))   | ZUNION numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM\|MIN\|MAX] [WITHSCORES] | 计算给定的一个或多个有序集的并集                             |
| ZUNIONSTORE      | **2.0.0** | O(N)+O(M log(M))   | ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM\|MIN\|MAX] | 计算给定的一个或多个有序集的并集，并存储在新的有序集合中     |
| ZINTER           | **6.2.0** | O(N*K)+O(M*log(M)) | ZINTER numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM\|MIN\|MAX] [WITHSCORES] | 计算给定的一个或多个有序集的交集                             |
| ZINTERSTORE      | **2.0.0** | O(N*K)+O(M*log(M)) | ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM\|MIN\|MAX] | 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合中 |
| ZDIFF            | **6.2.0** | O(L + (N-K)log(N)) | ZDIFF numkeys key [key ...] [WITHSCORES]                     | 计算给定的一个或多个有序集的差集                             |
| ZDIFFSTORE       | **6.2.0** | O(L + (N-K)log(N)) | ZDIFFSTORE destination numkeys key [key ...]                 | 计算给定的一个或多个有序集的差集并将结果集存储在新的有序集合中 |
| ZSCAN            | **2.8.0** | O(1) or O(N)       | ZSCAN key cursor [MATCH pattern] [COUNT count]               | 迭代有序集合中的元素（包括元素成员和元素分值）               |

---

## Redis Bitmaps

位图不是实际的数据类型，而是在字符串类型上定义的一组面向位的操作。由于字符串是二进制安全blob，其最大长度为512MB，因此它们适合设置多达232个不同的位。
位操作分为两组：恒定时间的单位操作，如将位设置为1或0，或获取其值，以及对位组的操作，例如在给定的位范围内计算设定位的数量（例如，总体计数）。
位图的最大优点之一是，在存储信息时，它们通常可以极大地节省空间。例如，在一个用递增的用户id表示不同用户的系统中，仅使用512MB内存就可以记住40亿用户的一个比特信息（例如，知道用户是否希望接收新闻稿）。

## Redis HyperLogLogs

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

## Redis Streams

Stream是Redis 5.0版本引入的一个新的数据类型，它以更抽象的方式模拟*日志数据结构*，但日志仍然是完整的：就像一个日志文件，通常实现为以只附加模式打开的文件，Redis流主要是一个仅附加数据结构。至少从概念上来讲，因为Redis流是一种在内存表示的抽象数据类型，他们实现了更加强大的操作，以此来克服日志文件本身的限制。

Stream是Redis的数据类型中最复杂的，尽管数据类型本身非常简单，它实现了额外的非强制性的特性：提供了一组允许消费者以阻塞的方式等待生产者向Stream中发送的新消息，此外还有一个名为**消费者组**的概念。

消费者组最早是由名为Kafka（TM）的流行消息系统引入的。Redis用完全不同的术语重新实现了一个相似的概念，但目标是相同的：允许一组客户端相互配合来消费同一个Stream的不同部分的消息。