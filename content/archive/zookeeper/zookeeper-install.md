---
title: "Zookeeper Install"
date: 2022-06-06T23:17:07+08:00
draft: true
Type: post
Tags: ["zookeeper"]
---

``3.7.1``

- server.A=B:C:D
  - **A 服务器编号 = myid**
  - **B 服务器 IP 地址**
  - **C leader 交互端口**
  - **D 选举服务器通信端口**

```
dataDir=/data/zookeeper/data
server.1={1_IP}:2888:3888
server.2={2_IP}:2888:3888
server.3={3_IP}:2888:3888
quorumListenOnAllIPs=true
audit.enable=true
```

```
myid /data/zookeeper/data range server.{i}
```

