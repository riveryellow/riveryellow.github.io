---
title: Redis架构分析
cdn: header-off
date: 2019-01-09 15:50:16
header-img: "/img/redis-banner.jpg"
tags:
---

## 普通主从模式
![](./img/redis-master-slaves.png)
Redis 的复制（replication）功能允许用户根据一个 Redis 服务器来创建任意多个该服务器的复制品，其中被复制的服务器为主服务器（master），而通过复制创建出来的服务器备份则为从服务器（slave）。 只要主从服务器之间的网络连接正常，主从服务器两者会具有相同的数据，主服务器就会一直将发生在自己身上的数据更新同步给从服务器，从而一直保证主从服务器的数据相同。
![](./img/redis-master-slaves-sequence.png)
**优点**
能较好地避免单点故障问题，以及提出了读写分离，降低了master节点的读压力。
**缺点**
1. 没有降低master节点的写压力；
2. 没有实现高可用，当主节点挂了没办法快速选举出新的主节点；
3. 每个节点都存储全量数据，浪费资源。

## 哨兵(Sentinel)模式
Redis-Sentinel是Redis官方推荐的高可用性(HA)解决方案。实际上这意味着你可以使用Sentinel模式创建一个可以不用人为干预而应对各种故障的Redis分布式系统。

Redis提供的sentinel（哨兵）机制，通过sentinel模式启动redis后，自动监控master/slave的运行状态，基本原理是：__心跳机制+投票裁决__
+ **监控（Monitoring）**：Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
+ **提醒（Notification）**：当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
+ **自动故障迁移（Automatic failover）**：当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。
**故障转移过程**
![](./img/sentinel1.png) ![](./img/sentinel2.png)
![](./img/sentinel3.png)

**优点**
1. 保证分布式系统的高可用
2. 实时监控各个节点的健康状态
3. 自动故障迁移
**缺点**
1. 没有解决master的写压力
2. 切换主节点的过程中可能会有写数据的丢失

## Redis-Cluster集群高可用架构
![](./img/redis-cluster.jpg)
Redis-Cluster架构中，被设计成共有16384(2^14)个hash slot。每个master分得一部分slot，其算法为：hash_slot = crc16(key) mod 16384 ，这就找到对应slot。采用hash slot的算法，实际上是解决了Redis-Cluster架构下多个master节点的数据分布问题。群集至少需要3主3从，且每个实例使用不同的配置文件。

**note**
Redis-Cluster主从架构的从节点默认不支持读写，官方不建议在此模式下进行读写分离。Redis Cluster的核心的理念主要是用slave做高可用，每个master挂一两个slave用作数据的热备，当master故障时的作为主备切换，实现高可用的。


