# [MySQL 实战 45 讲笔记](https://github.com/EruDev/blog/issues/2)

## 基础架构
![MySQL 基础架构](https://user-images.githubusercontent.com/31091355/106376530-241b0f80-63d1-11eb-9842-2ec186434885.png)
- 连接器：负责跟客户端建立连接、获取权限、维持和管理连接；
- 查询缓存：查询请求先访问缓存（key 是查询的语句，value 是查询的结果），命中直接返回。不推荐使用缓存，更新会把缓存清除（关闭缓存：参数 query_cache_type 设置为 DEMAND）；
- 分析器：对 SQL 语句解析，判断 sql 是否正确；
- 优化器：决定使用哪个索引，多表（join）的时候，决定各表的连接顺序；
- 执行器：执行语句，先判断用户有无权限，使用表定义的存储引擎。

## 日志系统

redo log (重做日志)，binlog（归档日志）
- redo log 是 InnoDB 特有，binlog 是 MySQL Server 层实现的，所有引擎都可以使用；
- redo log 是物理日志，记录的是 “在某个数据页做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如 “给 ID = 2 这一行的 c 字段加 1”；
- redo log 是循环写，空间固定会用完；binlog 是可以追加写。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

## 事务
- 原子性 Atomicity
- 一致性 Consistency
- 隔离性 Isolation
- 持久性 Durability

事务隔离级别：
- 读未提交 read uncommitted
- 读已提交 read committed
- 可重复读 repeatable read
- 串行化 serializable

事务隔离的实现
- 每条记录在更新的时候都会同时记录一条回滚操作。同一条记录在系统中存在多个版本，这就是数据库的多并发版本控制（MVCC）。
- 回滚日志会在系统判断当前没有事务用到这些回滚日志的时候删除。

尽量不要使用长事务
>长事务意味着系统里面会存在很老的事务视图，在这个事务提交之前，回滚记录都要保留，这会导致占用大量存储空间。除此之外，长事务还会占用锁资源，可能会拖垮库。

