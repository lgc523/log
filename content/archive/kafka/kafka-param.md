---
title: "Kafka Param"
date: 2022-06-11T16:59:25+08:00
draft: true
type: post
tags: ['Kafka']
---

## Broker 参数

### log.dirs

- 没有默认值

- /data/kafka1,/data/kafka2,..
- 最好保证挂在到**不同物理磁盘上**
  1. 提升读写性能，有更高的吞吐量
  2. 实现故障转移，坏掉的磁盘上的数据会自动转移到其他正常的磁盘上，broker 还能正常工作

### zookeeper.connect

- **chroot** 别名 隔离环境
- zk1:2181，zk2:2181，zk3:2181/kafka1
- zk1:2181，zk2:2181，zk3:2181/kafka2

## listener

- listeners 内网ip
- advertised.listeners 外网ip
- host.name/port 过期参数
- listener
  - 协议名称、主机名、端口号
  - **PLAINTEXT** 表示明文传输、SSL 使用 SSL/TLS 加密传输
  - 自定义协议名称，必须指定 listener.security.protocol.map
    - Listener.security.protocol.map=CONTROLL:PLAINTEXT 表示自定义协议使用明文不加密

## topic

- **auto.create.topics.enable**           是否允许自动创建 topic
- **unclean.leader.election.enable**  是否允许 unclean leader 选举
  - 分区副本中只有**leader 副本对外提供服务**
  - 只有保存数据比较多的副本才有资格竞选，落后太多的副本没有资格
  - **false** 不让落后太多的副本竞选，会导致这个**分区没有 leader 不可用**
  - True 允许落后副本当选 leader，会导致**数据丢失**
- **auto.leader.rebalance.enable**    是否允许定期进行 leader 选举
  - true 允许定期对一些 **topic 分区进行 leader 重选举**，换 leader
  - 换 leader 代价很高，流量要切换节点，没有任何性能收益

Topic 级别参数会覆盖全局 broker 参数的值，可以为每一个 topic 设置一个数据留存时间。

- retention.ms
  - 只设置该 topic 的的保留时间 默认 7 天
- retention.bytes
  - 为该 topic 预留多大的磁盘空间

### 创建topic设置

```
bin/kafka-topics.sh --bootstrap-server localhost:9092 --create \
--topic transaction --partitions 1 --replication-factor 1 \
--config retention.ms=15552000000 \
--config max.message.bytes=5242880
```

### 修改topic

```
bin/kafka-configs.sh --zookeeper localhost:2181 \
--entity-type topics \
--entity-name transaction \
--alter \
--add-config max.message.bytes=10485760
```



## log

- **log.retention.{hours|minutes|ms}**
  - 控制一条消息被保存多长时间，优先级 ms > minutes > hours
- **log.retention.bytes**
  - 指定 broker 为消息保存的总磁盘容量大小
  - -1 可以无限使用磁盘空间，在多租户的集群环境中有意义
- **message.max.bytes**
  - 控制 broker 能够接收的最大消息大小

## jvm

### heap

6GB，默认 1GB，Kafka broker 在与客户端进行交互时会在 JVM 堆上创建大量的 ByteBuffer 实例。

### GC

G1 垃圾回收器，更少的 Full GC。

```
export KAFKA_HEAP_OPTS=--Xms6g  --Xmx6g
export KAFKA_JVM_PERFORMANCE_OPTS= -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -Djava.awt.headless=true
bin/kafka-server-start.sh config/server.properties
```

## os

- 文件描述符限制
  - ulimit -n
- 文件系统类型
  - XFS 性能优于 ext4
  - [Kafka on ZFS](https://www.confluent.io/kafka-summit-sf18/kafka-on-zfs/)
- Swappiness
  - 设置一个接近0 的数，eg: 1，能够观测到 broker 性能开始出现下降的救火处理时间
  - 避免无力内存耗尽时，os OOM killer 随机 kill 进程
- 提交时间
  - 提交时间/ flush 落盘时间
  - 发送数据并不是真的要等数据被写入磁盘才会认为成功，只要数据被写入到操作系统的页缓存上就可以了
  - 操作系统会根据  LRU 算法定期将页缓存上的脏数据落盘到物理磁盘上，这个定期由提交时间确定。
  - 默认 5 秒，由于存在多副本来保证高可用，可以稍微拉大提交间隔来换取性能。
  - Flush 之前机器宕机，数据在 broker 中就丢失了。

> Kafka 配额管理 https://www.cnblogs.com/huxi2b/p/8609453.html



