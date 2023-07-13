# no-sql

nosql和关系型数据库对比

https://blog.acecodeinterview.com/sql_vs_nosql/
# Redis

## 使用

### 数据类型

+ string

    - 内部存储：int，SDS

    - 命令：SET GET DEL STRLEN MSET MGET INCR INCRBY DECR DECRBY EXPIRE TTL EX SETEX

    - 应用场景：缓存对象；常规计数；分布式锁；共享session

+ list

    - 内部存储：quicklist

    - 命令：LPUSH，LPOP，RPUSH，RPOP，LRANGE，BLPOP，BRPOP

    - 应用场景：消息队列

+ hash

    - 内部存储：listpack，哈希表

    - 命令：HSET，HGET，HMSET，HMGET，HDEL，HLEN，HGETALL，HINCRBY

    - 应用场景：缓存对象；购物车

+ set

    - 内部存储：整数集合，哈希表

    - 命令：SADD SREM SMEMBERS SCARD SISMEMBER SPOP 集合运算

    - 应用场景：点赞，关注

+ zset

    - 内部存储：listpack，跳表和哈希表

    - 命令：ZADD ZREM ZSCORE ZCARD 集合操作

    - 应用：排行榜

### 接口

##### 全局命令

ping bgsave eval dbsize


##### 数据类型相关

见上面数据类型命令


### 应用场景
#### 分布式锁

https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/

#### 缓存

使用redis充当mysql缓存

##### 问题

缓存雪崩：大量key同时过期，redis宕机

缓存击穿：热点数据过期

缓存穿透：请求的key缓存和mysql都不存在

##### 一致性策略

先更新数据库，再更新缓存

#### 分布式限流器


#### 一般应用

见上面数据类型



## 原理

### 底层数据结构

redis是一个巨大的内存哈希表，key与value统称为对象object，key类型为SDS，value类型如下

+ SDS

+ 链表

+ 压缩列表：prevlen encoding data

+ quicklist：将压缩列表作为链表节点

+ listpack：encoding data len

+ 哈希表：渐进式rehash

+ 整数集合：集合升级

+ 跳表
##### SDS

o1复杂度；二进制安全；不发生缓冲区溢出（自动扩容）

##### 链表

不必多说

##### 压缩列表

prevlen encoding data

##### quicklist

将压缩列表使用链表串联

缺点：连锁更新

##### listpack

encoding data len

##### 哈希表

链式哈希表，使用渐进rehash策略（两个哈希表）

##### 整数集合

类似于数组，存在升级

##### 跳表

不必多说






### 持久化

#### RDB

某一时刻（时间内修改次数达到阈值）fork出子进程记录当前内存数据库内容（快照）
#### AOF

时机：每执行完写命令后，将命令写入aof文件

刷盘时机：总是 每秒 从不

aof重写过程：

aof重写机制：aof文件达到阈值之后，fork出子进程读取当前数据库所有键值对，将命令写入aof文件，然后使用新aof文件覆盖旧文件。重写结束后向父进程发送信号，父进程将aof缓冲区写入，然后覆盖老aof文件

### 过期删除和内存淘汰

#### 过期删除

所有设置过期时间的key还会存储在过期字典中

```
typedef struct redisDb {
    dict *dict;    /* 数据库键空间，存放着所有的键值对 */
    dict *expires; /* 键的过期时间 */
    ....
} redisDb;
```

redis过期删除策略：定期删除（5 / 20）+ 惰性删除

#### 数据淘汰

+ volatile-random

+ volatile-ttl

+ volatile-lru

+ volatile-lfu

+ allkeys-random

+ allkeys-lru

+ allkeys-lfu

每一个key存在lru字段，存储lru和lfu信息


### 线程模型

对于命令解析，执行，返回，是单线程

Redis 6.0 之后引入了多线程处理网络请求

对于写aof，关闭文件，内存回收等耗时任务，发送到任务队列由后台线程消费

https://www.xiaolincoding.com/redis/base/redis_interview.html#redis-%E6%98%AF%E5%8D%95%E7%BA%BF%E7%A8%8B%E5%90%97

### 高可用

#### 主从复制

##### 第一次同步：全量复制

+ 建立连接：从服务器发送runid与offset，主服务器发回

+ 主服务器执行bgsave，生成rdb文件，将其发送给从服务器，从服务器收到后回复

+ 主服务器接到回复后，发送replication buffer中的命令给从数据库

##### 命令传播

双方在第一次同步结束后会维护一个tcp长连接，传输写命令

##### 网络故障后恢复：增量复制

+ 从服务器发送自己offset

+ 根据从服务器offset是否在缓冲区，来决定增量还是全量同步


#### 哨兵

##### 判断主节点故障

+ 单个哨兵发现主观下线，发起投票

+ 刚好超过二分之一认为下线，该哨兵成为候选者，向其他哨兵发起投票

##### 主从故障转移

+ 哨兵ld选出主节点

+ 将从节点指向主节点

+ 通知客户端主节点发生变化（发布订阅者模式）

+ 监听旧主节点，等其上线将其变为从节点




