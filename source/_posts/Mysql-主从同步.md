---
title: Mysql-主从同步
date: 2021-07-14 17:07:31
tags: Mysql
Categories: Mysql
---


数据库随着越来越多的请求，我们将数据库的写操作和读操作进行分离， 使用多个从库副本（Slaver Replication）负责读，使用主库（Master）负责写， 从库从主库同步更新数据，保持数据一致。架构上就是数据库主从同步。 从库可以水平扩展，所以更多的读请求不成问题。

<!--more-->

## 主从同步

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMwLmNuYmxvZ3MuY29tL2kvMzgxNDEyLzIwMTQwOC8wMTAwMDkxNDkxNTkwNTMuanBn?x-oss-process=image/format,png)

1. Master将改变记录到二进制日志(binary log)中（这些记录叫做二进制日志事件，binary log events）；

2. Slave将Master的二进制日志事件(binary log events)拷贝到它的中继日志(relay log)；

3. Slave重做中继日志(Relay Log)中的事件，将Master上的改变反映到它自己的数据库中。


## 同步延迟

mysql的主从复制都是单线程的操作，主库对所有DDL和 DML产生binlog，binlog是顺序写，所以效率很高，slave的Slave_IO_Running线程到主库取日志，效率很比较高，下一步， 问题来了，slave的Slave_SQL_Running线程将主库的DDL和DML操作在slave实施。DML和DDL的IO操作是随即的，不是顺 序的，成本高很多，还可能可slave上的其他查询产生lock争用，由于Slave_SQL_Running也是单线程的，所以一个DDL卡主了，需要 执行10分钟，那么所有之后的DDL会等待这个DDL执行完才会继续执行，这就导致了延时。有朋友会问：“主库上那个相同的DDL也需要执行10分，为什 么slave会延时？”，答案是master可以并发，Slave_SQL_Running线程却不可以。

当主库的TPS并发较高时，产生的DDL数量超过slave一个sql线程所能承受的范围，那么延时就产生了，当然还有就是可能与slave的大型query语句产生了锁等待。


```
sync_binlog”：这个参数是对于MySQL系统来说是至关重要的，他不仅影响到Binlog对MySQL所带来的性能损耗，而且还影响到MySQL中数据的完整性。对于“sync_binlog”参数的各种设置的说明如下：

sync_binlog=0，当事务提交之后，MySQL不做fsync之类的磁盘同步指令刷新binlog_cache中的信息到磁盘，而让Filesystem自行决定什么时候来做同步，或者cache满了之后才同步到磁盘。

sync_binlog=n，当每进行n次事务提交之后，MySQL将进行一次fsync之类的磁盘同步指令来将binlog_cache中的数据强制写入磁盘。
```

### 如何判断主从延迟

MySQL提供了从服务器状态命令，可以通过 show slave status 进行查看，  比如可以看看Seconds_Behind_Master参数的值来判断，是否有发生主从延时。

- NULL - 表示io_thread或是sql_thread有任何一个发生故障，也就是该线程的Running状态是No,而非Yes.
- 0 - 该值为零，是我们极为渴望看到的情况，表示主从复制状态正常
