---
layout: mysql-write-ahead
title: Mysql中的WAL（预写日志）
date: 2021-07-27 15:38:21
categories: Mysql
tags: Mysql, WAL
---

WAL(Write Ahead Log)预写日志，是数据库系统中常见的一种手段，用于保证数据操作的原子性和持久性。

### Mysql数据页

mysql的page与文件系统的block类似，是mysql自己定义的读取和写入磁盘的最小单位，其目的是为了充分的利用磁盘的顺序访问速度快的优势。一般是磁盘扇区的4~16倍。通常Mysql的page大小为16kb。

当内存数据页跟磁盘数据页内容不一致的时候，我们成这个内存页为“脏页”。内存数据写入磁盘后，内存和磁盘上的数据页内容就一致了，称为“干净页”。MySQL从内存更新到磁盘的过程，称为刷脏页的过程（flush）。

<!--more-->

InnoDB刷脏页的时机:

- 内存中的redo log 写满了，这时系统就会停止所有更新操作，把checkoutpoint 往前推，redo log留出空间可以继续写。往前推进之后，就要把两个点之间的日志对应的所有脏页都 flush 到磁盘上。这种情况是 InnoDB 要尽量避免的。因为出现这种情况，整个系统都不能接受更新。更新数会跌为0。
- 系统中内存不足时，当这个时候需要新的数据页到内存中，就要淘汰掉一些数据页，如果淘汰的是“脏页”，就要先将“脏页”写到磁盘。
- 数据库空闲的时候刷脏页。
- 数据库正常关闭的时候，也要把内存中所有的脏页全都flush 到磁盘上。

刷脏页是常态，所以如果出现以下的情况，都会明明显影响性能：

```
1.一个查询要淘汰的脏页太多，会导致查询的响应时间明显变长；
2.日志写满，更新全部堵住，写性能跌为0，这种情况对于敏感业务来说是不能接受的。
```

如何合理的刷脏页可以通过一些参数来控制

1. innodb_io_capacity: 磁盘IO性能
2. innodb_max_dirty_pages_pct: 脏页比例上限，默认75%
3. innodb_flush_neighbors: 刷脏页的时候如果数据页旁边的数据页也是脏页就一起刷掉，使用机械硬盘时，这个优化很有意义。


### binlog

binlog的写入机制比较简单：事务执行的过程中，先把日志写到binlog cache，事务提交的时候，再把binlog cache写到binlog文件中。系统给 binlog cache 分配了一片内存，每个线程一个，参数 binglog_cache_size 用于控制单个线程内 binlog cache 的内存大小，超过就要暂存在磁盘。


![](https://upload-images.jianshu.io/upload_images/6578832-f121471641dca98d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1142/format/webp)

- write 指的是把日志写入到文件系统的 page cache，并没有吧数据持久化到磁盘，所以速度比较快。
- fsync 是持久化到磁盘的操作，一般情况下， fsync 才会占磁盘的 IOPS

write 和 fsync 的时机，是由参数 sync_binlog 控制的：

```
sync_binlog=0 的时候，表示每次提交事务都只 write，不 fsync；
sync_binlog=1 的时候，表示每次提交事务都会执行 fsync；
sync_binlog=N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才 fsync。
```

因此，在出现 IO 瓶颈的场景里，将 sync_binlog 设置成一个比较大的值，可以提升性能。在实际的业务场景中，考虑到丢失日志量的可控性，一般不建议将这个参数设成 0，比较常见的是将其设置为 100~1000 中的某个数值。

```
binlog_group_commit_sync_delay 参数，表示延迟多少微秒后才调用 fsync;
binlog_group_commit_sync_no_delay_count 参数，表示累积多少次以后才调用 fsync。
```


### REDO log

事务的执行过程中，生成的 redo log 是要先写到 redo log buffer 的。

redo log 三种状态：

- 存在 redo log buffer 中，物理上是在 MySQL 进程内存中
- 写到磁盘（write），但是没有持久化（fsync），物理上是在文件系统的 page cache 里
- 持久化磁盘，对应的是 hard disk


日志写到 redo log buffer 是很快的，write 到 page cache 也差不多，但是持久化到磁盘的速度就慢多了。

InnoDB 提供了 innodb_flush_log_at_trx_commit 参数，取值如下：


```
设置为 0 时，表示每次事务提交时都只是把 redo log 留在 redo log buffer 中；
设置为 1 时，表示每次事务提交时都将 redo log 直接持久化到磁盘；
设置为 2 时，表示每次事务提交时都只是把 redo log 写到 page cache。
```

InnoDB 有一个后台线程，每隔 1 秒，就会把 redo log buffer 中的日志，调用 write 写到文件系统的 page cache，然后调用 fsync 持久化到磁盘。



### undolog

undo log有两个作用：提供回滚和多个行版本控制(MVCC)。

在数据修改的时候，不仅记录了redo，还记录了相对应的undo，如果因为某些原因导致事务失败或回滚了，可以借助该undo进行回滚。

undo log和redo log记录物理日志不一样，它是逻辑日志。可以认为当delete一条记录时，undo log中会记录一条对应的insert记录，反之亦然，当update一条记录时，它记录一条对应相反的update记录。

undo log是采用段(segment)的方式来记录的，每个undo操作在记录的时候占用一个undo log segment。

另外，undo log也会产生redo log，因为undo log也要实现持久性保护。


### 事务执行过程中的日志存储

在MySQL5.6以前，当事务提交(即发出commit指令)后，MySQL接收到该信号进入commit prepare阶段；进入prepare阶段后，立即写内存中的二进制日志，写完内存中的二进制日志后就相当于确定了commit操作；然后开始写内存中的事务日志；最后将二进制日志和事务日志刷盘，它们如何刷盘，分别由变量 sync_binlog 和 innodb_flush_log_at_trx_commit 控制。

但因为要保证二进制日志和事务日志的一致性，在提交后的prepare阶段会启用一个prepare_commit_mutex锁来保证它们的顺序性和一致性。但这样会导致开启二进制日志后group commmit失效，特别是在主从复制结构中，几乎都会开启二进制日志。

在MySQL5.6中进行了改进。提交事务时，在存储引擎层的上一层结构中会将事务按序放入一个队列，队列中的第一个事务称为leader，其他事务称为follower，leader控制着follower的行为。虽然顺序还是一样先刷二进制，再刷事务日志，但是机制完全改变了：删除了原来的prepare_commit_mutex行为，也能保证即使开启了二进制日志，group commit也是有效的。

MySQL5.6中分为3个步骤：flush阶段、sync阶段、commit阶段。

![](https://images2018.cnblogs.com/blog/733013/201805/733013-20180508203426454-427168291.png)







