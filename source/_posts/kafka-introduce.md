---
title: Kakfa简介
date: 2021-06-07 15:33:14
tags: kafka
categories: 系统架构
---

## 简介

Apache Kafka 是一个分布式发布-订阅消息系统。是大数据领域消息队列中唯一的王者。最初由 linkedin 公司使用 scala 语言开发，在2010年贡献给了Apache基金会并成为顶级开源项目。至今已有十余年，仍然是大数据领域不可或缺的并且是越来越重要的一个组件。

- [x] 可靠性：具有副本及容错机制。
- [x] 可扩展性：kafka无需停机即可扩展节点及节点上线。
- [x] 持久性：数据存储到磁盘上，持久性保存。
- [x] 性能：kafka具有高吞吐量。达到TB级的数据，也有非常稳定的性能。
- [x] 速度快：顺序写入和零拷贝技术使得kafka延迟控制在毫秒级。

<!--more-->

```
消息系统分类:

Peer-to-Peer
消息生产者Producer1生产消息到Queue，然后Consumer1从Queue中取出并且消费消息。消息被消费后，Queue将不再存储消息，其它所有Consumer不可能消费到已经被其它Consumer消费过的消息。Queue支持存在多个Producer，但是对一条消息而言，只会有一个Consumer可以消费，其它Consumer则不能再次消费。但Consumer不存在时，消息则由Queue一直保存，直到有Consumer把它消费。

Publish/Subscribe（Topic）
简称发布/订阅模式。例如我微博有30万粉丝，我今天更新了一条微博，那么这30万粉丝都可以接收到我的微博更新，大家都可以消费我的消息。
```

## 架构

![](https://pic4.zhimg.com/80/v2-385cea5d5e8e02b8d71645b3310071db_1440w.jpg)

kafka支持消息持久化，消费端是主动拉取数据，消费状态和订阅关系由客户端负责维护，消息消费完后，不会立即删除，会保留历史消息。因此支持多订阅时，消息只会存储一份就可以。

- broker：kafka集群中包含一个或者多个服务实例（节点），这种服务实例被称为broker（一个broker就是一个节点/一个服务器）；
- topic：每条发布到kafka集群的消息都属于某个类别，这个类别就叫做topic；
- partition：partition是一个物理上的概念，每个topic包含一个或者多个partition；
- segment：一个partition当中存在多个segment文件段，每个segment分为两部分，.log文件和 .index 文件，其中 .index 文件是索引文件，主要用于快速查询， .log 文件当中数据的偏移量位置；
- producer：消息的生产者，负责发布消息到 kafka 的 broker 中；
- consumer：消息的消费者，向 kafka 的 broker 中读取消息的客户端；
- consumer group：消费者组，每一个 consumer 属于一个特定的 consumer group（可以为每个consumer指定 groupName）；
- .log：存放数据文件；
- .index：存放.log文件的索引数据。

## 组件

### Partation

kafka当中，topic是消息的归类，一个topic可以有多个分区（partition），每个分区保存部分topic的数据，所有的partition当中的数据全部合并起来，就是一个topic当中的所有的数据。

一个broker服务下，可以创建多个分区，broker数与分区数没有关系；
在kafka中，每一个分区会有一个编号：编号从0开始。
每一个分区内的数据是有序的，但全局的数据不能保证是有序的。（有序是指生产什么样顺序，消费时也是什么样的顺序）


### CosumerGroup

消费者组由一个或者多个消费者组成，同一个组中的消费者对于同一条消息只消费一次。

每个消费者都属于某个消费者组，如果不指定，那么所有的消费者都属于默认的组。

每个消费者组都有一个ID，即group ID。组内的所有消费者协调在一起来消费一个订阅主题( topic)的所有分区(partition)。当然，每个分区只能由同一个消费组内的一个消费者(consumer)来消费，可以由不同的消费组来消费。

partition数量决定了每个consumer group中并发消费者的最大数量。

![](https://pic2.zhimg.com/80/v2-1cf1cf01715c113b6dba9bc33755edb9_1440w.jpg)

如上面左图所示，如果只有两个分区，即使一个组内的消费者有4个，也会有两个空闲的。

### replicas

![](https://pic3.zhimg.com/80/v2-9c6842085ec000033ce4f95a5658d91a_1440w.jpg)

副本数（replication-factor）：控制消息保存在几个broker（服务器）上，一般情况下副本数等于broker的个数。主副本叫做leader，从副本叫做 follower（在有多个副本的情况下，kafka会为同一个分区下的所有分区，设定角色关系：一个leader和N个 follower），处于同步状态的副本叫做in-sync-replicas(ISR);follower通过拉的方式从leader同步数据。消费者和生产者都是从leader读写数据，不与follower交互。


### segment文件

一个partition当中由多个segment文件组成，每个segment文件，包含两部分，一个是 .log 文件，另外一个是 .index 文件，其中 .log 文件包含了我们发送的数据存储，.index 文件，记录的是我们.log文件的数据索引值，以便于我们加快数据的查询速度。

.index 与 .log 对应关系如下：

![](https://pic3.zhimg.com/80/v2-cce29042ec33e8a9724ef70da24b8aca_1440w.jpg)

这是因为index文件中并没有为数据文件中的每条消息都建立索引，而是采用了稀疏存储的方式，每隔一定字节的数据建立一条索引。
这样避免了索引文件占用过多的空间，从而可以将索引文件保留在内存中。
但缺点是没有建立索引的Message也不能一次定位到其在数据文件的位置，从而需要做一次顺序扫描，但是这次顺序扫描的范围就很小了。
