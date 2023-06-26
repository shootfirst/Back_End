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

#### 索引

##### 索引分类

+ 数据结构：b+树，哈希表

+ 物理存储：聚簇，二级

+ 字段特性：主键，唯一，普通，前缀

+ 字段个数：单列，联合

##### 适合索引

+ 唯一

+ where

+ group by

##### 不适合索引

+ 修改频繁

+ 数据大量重复

+ 数据过少

+ where过少

##### 优化索引方法

+ 前缀索引

+ 覆盖索引

+ 自增

+ not null

##### 索引失效

+ 模糊匹配

+ 计算和函数

+ 联合索引

+ where中的or

## mysql原理



### 数据组织

### sql优化

### 查询模型
### 索引

### 内存池

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

+ update Undo Log：提交时放入undo log链表，等待过期，由perge线程删除

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
### 锁

### sql语句生命周期


