---
layout: post
title:  Cassandra之初识
categories: [Distribution,Nosql]
date: 2020-03-17 10:58:30 +0800
keywords: [Cassandra,Distribution]
---

>本文基于Cassandra 4.0 简单介绍cassandra。

### 简介

Cassandra由facebook开源，是一款分布式、高可用、去中心化、容错的noSql数据库，它提出了一种基于最终一致的语义的分区宽列存储模型。结合亚马逊的Dynamo的分布式存储和谷歌的BigData的数据模型,采用了一种分阶段事件驱动架构（SEDA）来实现。

### 特性

* 分布式(Distributed)

  可以在多台机器上运行，呈现给用户一致的整体。

* 去中心化(Decentralized)

  每个节点对等，cassandra的协议是P2P的，并使用gossip来维护存活和死亡的列表，去中心化意味着不会单点失效，因此支持高可用。

* 弹性可扩展(Elastic Scalability)

  可以在不中断服务的情况下，扩展或缩减服务的规模。

* 高效写

  Netflix测试

* 高效读

* 弱一致性

* 高可用(High Availability)
可以在不中断服务的情况下替换故障节点，还可以把数据分布到多个cassandra数据中心，提供很好的本地访问，并且在某一个数据中心发生故障，不会中断服务，防止系统瘫痪。

* 容错(Fault Tolerance)

* 可调节的一致性(Tuneable Consistency)

  允许通过参数调节一致性水平和可用性水平，通过副本因子(replication factor)设置更新在集群中传播的节点数，此外，客户端还需要设置一致性级别（consistency level）参数，决定写入多少个副本成功才判定写操作是成功的。

* 面向行(Row-Oriented)  
  数据结构为多维稀疏哈希表，可以在运行中添加、删除字段，不会造成服务中断。

- 持久化

节点可部署在不同数据中心，即使数据中心挂了，也无损数据，适用于对数据持久化要求高的系统

* 灵活的模式(Flexible Schema)

  CQL，类似SQL的语法，早期通过Thrift API来实现，3.0版本后，

* 高性能(High Performance)
  
  cassandra为写吞吐量专门做了优化。
  
### 缺点

不支持关联查询和子查询，因为在分布式系统中跨越硬件进行关联查询的性能很差，推荐从单表中查询数据或者使用提供的集合工具简化操作。

### 适用场景

* 大规模部署的场景
 
  cassandra的很多特性都是基于高可用、可调节一致性、P2P协议等，这些特性如果单点部署没有意义。

* 适用于大量写入、分析、统计的场景

  cassandra为优异的写吞吐量特别优化。

* 地区分布
  
  cassandra支持多地分布存储，很容易可以将数分布到多个数据中心。

* 快速变化的应用

  灵活的模式让应用在初始阶段数据结构能跟上业务的变化。

### 用户案例

1. apple 75000个节点，存储数据超过10PB
2. Netflix 2500个节点，存储数据达420TB，每天上万亿次请求
