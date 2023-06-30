# Mysql

## mysql使用

#### 表设计

##### 数据建模

https://blog.csdn.net/zhangliushi/article/details/127549003

##### 范式

https://blog.csdn.net/weixin_47414034/article/details/123855916

+ 列的原子性

+ 不在主键的字段必须完全依赖主键

+ 不在主键的字段必须完全直接依赖主键

反范式：允许适当冗余，空间换时间


#### SQL语句

+ DDL

+ DQL

+ DML

+ DCL

#### 分库分表

##### 方案

+ 垂直分库

+ 垂直分表

+ 水平分库

+ 水平分表

##### 适用场景

##### 副作用

https://www.51cto.com/article/709614.html



## mysql原理


### 数据组织

类似于文件系统

### 索引

#### 分类

+ 数据结构：b+树，哈希表

+ 物理存储：聚簇，二级

+ 字段特性：主键，唯一，普通，前缀

+ 字段个数：单列，联合

#### 适合索引

+ 唯一

+ where

+ group by

#### 不适合索引

+ 修改频繁

+ 数据大量重复

+ 数据过少

+ where过少

#### 优化索引

+ 前缀索引

+ 覆盖索引

+ 自增

+ not null

#### 索引失效

+ 模糊匹配

+ 计算和函数

+ 联合索引

+ where中的or

#### B+树索引

##### 插入

##### 删除

##### 并发控制

crabbing协议：

对于查询操作，从根节点开始，首先获取根节点的读锁，然后在根节点中查找key应该出现的孩子节点，获取孩子节点的读锁，然后释放根节点的读锁，以此类推，直到找到目标叶子节点，此时该叶子节点获取了读锁。

对于删除和插入操作，也是从根节点开始，先获取根节点的写锁，一旦孩子节点也获取了写锁，检查根节点是否安全，如果安全释放孩子节点所有祖先节点的写锁，以此类推，直到找到目标叶子节点。节点安全定义如下：如果对于插入操作，如果再插入一个元素，不会产生分裂，或者对于删除操作，如果再删除一个元素，不会产生并合。

### sql优化器

sql优化器对AST形态进行修改，最终生成查询树

#### 逻辑优化

找出SQL语句等价的变换形式，使得SQL执行更高效。

https://zhuanlan.zhihu.com/p/149497447

#### 物理优化

基于代价的查询优化方式对查询执行计划做了定量的分析，对每一个可能的执行方式进行评估，挑出代价最小的作为最优的计划。

https://zhuanlan.zhihu.com/p/150444554



### 查询模型

https://blog.csdn.net/u011436427/article/details/121809259

+ Iteraor Model 火山模型

+ Materialization Model 物化模型

+ Vector Model 向量化模型


### 内存池

+ 作为和磁盘之间的缓存，存储：数据页，索引页，undo页，锁信息等等（不包括redo 页！）

+ 整个buffer pool分为前面的控制块和后面的数据页

+ free list

+ flush list

+ lru链表：young区域和old区域（63:37）



### 日志

#### redo log

https://www.cnblogs.com/kuangtf/articles/16353184.html#1redo

作用：记录对一个页面（索引页，数据页，undo页）的修改

过程：对一个页面修改完后（还在内存），写入redo页

##### 日志格式

+ 逻辑日志

+ 物理日志

采用逻辑物理日志结合（每个页面的逻辑日志）

##### LSN与Mtr

+ 每一个上层的逻辑事务都对应底层多个Mtr

+ 每一个页面修改对应一个Mtr，有一个唯一LSN，生成若干redo log

几个关键的LSN

+ flushed_to_disk_lsn

+ flush链表，页面控制块的oldest_modification

+ flush链表，页面控制块的newest_modification

+ checkpoint_lsn

##### redo log block

+ 缓冲区由多个 log block组成

+ 存储所有LSN，同一个事务LSN不一定连续，但是使用链表串联

##### checkpoint

存储flush链表中最小的oldest_modification（之前的一定刷盘，之后的可能刷盘）

##### 崩溃后恢复过程

+ 确定恢复起点（checkpoint_lsn）和终点

+ 从起点开始，对比lsn和对应页面的newest_modification，若`lsn>newest_modification`，则执行redo（幂等）



#### undo log

作用：事务在执行一半，忽然宕机，开机后，必须回滚到该事务执行之前的状态

过程：对记录更新之前，写入undo log

##### 分类

+ insert Undo Log：事务被提交便删除

+ update Undo Log：提交时放入undo log链表，等待过期，由perge线程删除（当最小活跃事务id大于该log的事务id，即可删除）

##### undo log存储

b+树页面中记录，会存储trx_id和roll pointer

​128个​回滚段（rollback segment）​​​。每个回滚段记录了​​1024 个Undo Log segment​​​，在每个Undo Log segment段中进行​​Undo页​​的申请。

#### bin log


#### 流程

##### 日志写入

写入undo log

写入undo log的redo log

修改数据

写入修改数据的redo log

##### 崩溃恢复

获取崩溃前的活跃事务列表和脏页（通过check point获取二者）

进行redo：通过redo log写入脏页

进行undo：通过undo log撤销活跃事务的写入





### 事务

#### ACID

+ A 原子性

+ C 一致性

+ I 隔离性

+ D 持久性

#### 问题

https://blog.csdn.net/promsing/article/details/126002074

+ 脏写

+ 脏读

+ 不可重复读

+ 幻读

#### 事务隔离级别

+ 读未提交

+ 读已提交

+ 可重复读

+ 串行化

#### MVCC

只有快照读才使用mvcc，需要使用每条记录的undo log版本链

##### read view

+ creator_trx_id：创建这个 Read View 的事务 ID。

+ trx_ids：表示在生成ReadView时当前系统中活跃的读写事务的 事务id列表 。  

+ up_limit_id：活跃的事务中最小的事务 ID。

+ low_limit_id：表示生成ReadView时系统中应该分配给下一个事务的 id 值

##### 读取机制

+ 等于creator_trx_id可以读取

+ 小于up_limit_id可以读取

+ 大于low_limit_id不能读取

+ 在up_limit_id和low_limit_id之间则判断是否在trx_ids中

##### 生成机制

+ 读未提交：不生成rv，每次读最新

+ 读已提交：每次读取生成rv

+ 可重复读：第一次读取生成rv

+ 可串行化：加锁




### 锁

#### 分类

##### 粒度

+ 全局锁

+ 表级锁

+ 页级锁

+ 行级锁

##### 属性

+ 共享锁

+ 排它锁

##### 其他

+ 意向锁（协调行锁与表锁）

+ 间隙锁（实现串行化关键）

+ 记录锁（行锁）

+ 临键锁（记录锁加间隙锁）

#### 加锁策略

https://www.cnblogs.com/rjzheng/p/9950951.html

##### 快照读与当前读

可串行化都是当前读，加锁读取

##### 加锁策略考虑

+ 隔离级别

+ 是否走索引：无索引，聚簇，非聚簇

+ 更新还是读取

+ 精确查找与范围查找


### sql语句生命周期




