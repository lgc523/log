---
title: "Kafka Client"
date: 2022-06-11T23:28:19+08:00
draft: true
type: post
tags: ['Kafka']
---

## 生产者分区

**消息只存在对应 topic 下的一个分区中，通过分区来提供负载均衡能力。**

自定义分区策略，显示配置**partitioner.class**参数为你自己实现类的 **Full Qualified Name**。

生产者可以编写一个类实现 ``org.apache.lafla.clients.producer.Partitioner`` 接口，实现主要的 **partiiton** 方法。

```java
int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster);
```

- 轮询策略

  - 顺序分配
  - 能够保证消息最大限度地被平均分配到所有分区上，默认是最合理的分区策略

- 随机策略

  ```java
  List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
  return ThreadLocalRandom.current().nextInt(partitions.size());
  ```

- 按消息键保存策略

  - Kafka 允许为每条消息定义消息键，可以是业务参数表明特殊含义
  - 定义了消息 key，可以保证同一个 key 的所有消息都进入到相同的分区里面。

  ```java
  List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
  return Math.abs(key.hashCode()) % partitions.size();
  ```

- 基于地理位置的分区策略

  - 针对大规模，跨城市、国家的集群

消息在Kafka 中处理逻辑结构中的第三层，上层的 分区可以通过 key 来切分流量，保证消息 key 的单分区顺序读写。

如果 topic 内定义太多 消息 key，在不同读写速率下，分区内的消息流量会倾斜，浪费分区空间，
