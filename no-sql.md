# no-sql

nosql和关系型数据库对比

https://blog.acecodeinterview.com/sql_vs_nosql/
## Redis

### 使用

#### 数据类型

redis是一个内存kv数据库，将redis看成一个内存哈希表，key是string类型，全局唯一，value一共有以下几种数据类型

##### string

内部存储：int，SDS

指令：字符串操作；批量设置；计数器；过期时间设置；不存在则插入；

应用场景：缓存对象；常规技术；分布式锁；共享session

##### list

内部存储：quicklist

命令：LPUSH，LPOP，RPUSH，RPOP，LRANGE，BLPOP，BRPOP

应用场景：消息队列

##### hash

内部存储：listpack，哈希表

命令：HSET，HGET，HMSET，HMGET，HDEL

应用场景：缓存对象；购物车

##### set

内部存储：整数集合，哈希表

命令：SADD，SREM，SCARD，SISMEMBER，SRANDOMEMBER，SPOP；集合操作

应用场景：点赞，关注

##### zset

内部存储：listpack，跳表和哈希表

命令：ZADD，ZREM，ZCARD，ZSCORE；集合操作（没有差集）

应用：排行榜

#### 接口

##### 全局命令

keys；dbsize；exists；del；expire；ping；eval；save；bgsave；

##### 数据类型相关

见上面数据类型命令




#### 应用场景

见上面数据类型应用场景

#### 分布式锁

https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/



#### 缓存

使用redis充当mysql缓存

##### 问题

缓存雪崩：大量key同时过期，redis宕机

缓存击穿：热点数据过期

缓存穿透：请求的key缓存和mysql都不存在

##### 一致性策略

先更新数据库，再删除缓存



### 原理

#### 底层数据结构

##### SDS

o1复杂度；二进制安全；不发生缓冲区溢出（自动扩容）

##### 链表

不必多说

##### 压缩列表

prevlen encoding data

缺点：连锁更新

##### 哈希表

链式哈希表，使用渐进rehash策略（两个哈希表）

##### 整数集合

类似于数组，存在升级

##### 跳表

不必多说

##### quicklist

将压缩列表使用链表串联

##### listpack

encoding data len



#### 持久化

##### RDB

某一时刻fork出子进程记录当前内存数据库内容（快照）
##### AOF

每执行完写命令后，将命令写入aof文件

aof重写：读取当前数据库所有键值对，将命令写入aof文件，然后使用新aof文件覆盖旧文件

aof重写机制：aof文件达到阈值之后，fork出子进程进行重写，重写结束后向父进程发送信号，父进程将aof缓冲区写入，然后覆盖

#### 过期删除和内存淘汰

##### 过期删除

所有设置过期时间的key还会存储在过期字典中

```
typedef struct redisDb {
    dict *dict;    /* 数据库键空间，存放着所有的键值对 */
    dict *expires; /* 键的过期时间 */
    ....
} redisDb;
```

redis过期删除策略：定期删除（5 / 20）+惰性删除

##### 数据淘汰

+ volatile-random

+ volatile-ttl

+ volatile-lru

+ volatile-lfu

+ allkeys-random

+ allkeys-lru

+ allkeys-lfu

每一个key存在lru字段，存储lru和lfu信息


#### 线程模型

Redis 6.0 之后引入了多线程处理网络请求，对于命令的执行仍然是单线程

https://www.xiaolincoding.com/redis/base/redis_interview.html#redis-%E6%98%AF%E5%8D%95%E7%BA%BF%E7%A8%8B%E5%90%97

#### 高可用




