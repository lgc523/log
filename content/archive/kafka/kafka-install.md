---
title: "Kafka Install"
date: 2022-06-07T00:23:42+08:00
draft: true
type: post
tags: ["Kafka"]
---

- **server.properties**

  - **broker.id** 
  - **listeners=PLAINTEXT://{ip}:9092**
  - **log.dirs=/data/kafka/kafka-logs**
  - **zookeeper.connect={ZK_HOST}:{PORT},.../{$chroot}****

- **start**

  - bin/kafka-server-start.sh [-daemon] config/server.properties [&]
  - **bin/kafka-server-start.sh -daemon config/server.properties**

- **Topic**

  ```
  bin/kafka-topics.sh --zookeeper localhost:2181/kafka --create --topic topic-demo \
  --replication-factor 3 \
  --partitions 3
  #创建主题 topic-demo 4 分区，3 副本
  bin/kafka-topics.sh --bootstrap-server 172.21.0.9:9092  --create --topic topic-demo \ 
  --replication-factor 3 \
  --partitions 3
  ```

- **list**

  ```
  bin/kafka-topics.sh --list --zookeeper localhost:2181,c2:2181,c3:2181
  bin/kafka-topics.sh --list --bootstrap-server localhost:9092
  ```

  

- **describe**

  ```
  bin/kafka-topics.sh --zookeeper localhost:2181/kafka --describe --topic topic-demo
  bin/kafka-topics.sh --bootstrap-server 172.21.0.9:9092 \
  --describe --topic topic-demo
  ```

  ```
  [spider@c1 kafka_2.13-3.2.0]$ bin/kafka-topics.sh --bootstrap-server 172.21.0.9:9092 \
  > --describe --topic topic-demo
  Topic: topic-demo	TopicId: OJKFfWxyQNq79M4L3LU-AQ	PartitionCount: 3	ReplicationFactor: 3	Configs: segment.bytes=1073741824
  	Topic: topic-demo	Partition: 0	Leader: 2	Replicas: 2,3,1	Isr: 2,3,1
  	Topic: topic-demo	Partition: 1	Leader: 3	Replicas: 3,1,2	Isr: 3,1,2
  	Topic: topic-demo	Partition: 2	Leader: 1	Replicas: 1,2,3	Isr: 1,2,3
  [spider@c1 kafka_2.13-3.2.0]$
  ```

- console-consumer

  ```
  bin/kafka-console-consumer.sh --bootstrap-server 172.24.40.89:9092 \
  --topic topic-demo
  ```

  ```
  bin/kafka-console-producer.sh --broker-list 172.17.16.2:9092 \
  --topic topic-demo
  ```

- Java-client

  ```java
  <!-- https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients -->
  <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-clients</artifactId>
      <version>3.2.0</version>
  </dependency>
  ```

  1. Init client & config
  2. Build ProducerRecord （contain topic、msg）
  3. Client send ProducerRecord
  4. Close client

- produer

  ```java
  package cn.spider.kafka.simple;
  
  import org.apache.kafka.clients.producer.KafkaProducer;
  import org.apache.kafka.clients.producer.ProducerRecord;
  import org.apache.kafka.common.serialization.StringSerializer;
  import org.junit.Before;
  import org.junit.Test;
  
  import java.util.Properties;
  
  public class ProducerFastStartTest {
      public static final String brokerList = "c1:9092,c2:9092,c3:9092";
      public static final String topic = "topic-demo";
  
      public Properties properties;
      public KafkaProducer<String, String> producer;
  
      @Before
      public void init() {
          properties = new Properties();
          properties.put("key.serializer", StringSerializer.class.getName());
          properties.put("value.serializer", StringSerializer.class.getName());
          properties.put("bootstrap.servers", brokerList);
  
          //config producer client
          producer = new KafkaProducer<String, String>(properties);
  
      }
  
      @Test
      public void send() {
          //build record
          ProducerRecord<String, String> record = new ProducerRecord<String, String>(topic, "hello,kafka!!!");
          //send
          try {
              producer.send(record);
              System.out.println("send over...");
          } catch (Exception e) {
              e.printStackTrace();
          }
          producer.close();
      }
  }
  
  ```

- consumer

  ```java
  package cn.spider.kafka.simple;
  
  import org.apache.kafka.clients.consumer.ConsumerRecord;
  import org.apache.kafka.clients.consumer.ConsumerRecords;
  import org.apache.kafka.clients.consumer.KafkaConsumer;
  import org.apache.kafka.common.serialization.StringDeserializer;
  import org.junit.Before;
  import org.junit.Test;
  
  import java.time.Duration;
  import java.util.Collections;
  import java.util.Properties;
  
  public class ConsumerFastStartTest {
      public static final String brokerList = "c2:9092,c1:9092,c3:9092";
      public static final String topic = "topic-demo";
      public static final String groupId = "group.demo";
      public KafkaConsumer<String, String> consumer;
  
      public Properties properties;
  
      @Before
      public void init() {
          properties = new Properties();
          properties.put("key.deserializer", StringDeserializer.class.getName());
          properties.put("value.deserializer", StringDeserializer.class.getName());
          properties.put("bootstrap.servers", brokerList);
  
          //consumer group
          properties.put("group.id", groupId);
          //config consumer client
          consumer = new KafkaConsumer<>(properties);
          //subscribe topic
          consumer.subscribe(Collections.singletonList(topic));
      }
  
      @Test
      public void testConsumer() {
          while (true) {
              ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(500));
              for (ConsumerRecord<String, String> record : records) {
                  System.out.println(record.value());
              }
          }
      }
  }
  
  ```

  
