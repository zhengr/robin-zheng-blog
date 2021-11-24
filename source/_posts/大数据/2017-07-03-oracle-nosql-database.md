---
title: Oracle NoSQL Database介绍及部署
date: 2017-07-03 00:00:00
author: robin zheng
img: https://www.oracle.com/us/assets/obic-campaign-oracle-cloud-services-3889624.svg
categories: 大数据
tags:
  - 大数据
  - Oracle
  - Nosql
---

Oracle于2011年发布NoSQL,它是基于Oracle Berkeley DB Java Edition的新键-值存储系统,设计时考虑到了高扩展性和高可用性，并可部署于多个互相复制的节点上，以便进行快速故障切换及负载均衡。

## Oracle NoSQL database

### 1、特性

#### 数据模型简单

Key/Value式的存储。

#### 扩展性强

支持自动地基于hash函数的数据分片策略。
提供基于数据节点拓扑结构和访问延迟的智能控制，以提供最佳的数据访问性能。

#### 行为可预测性

提供ACID的事务性支持，并且支持基于全局和单个操作的事务级别设置。
通过B-tree数据结构构成的Cache层和高效的查询调度机制，提供可控的请求延迟。          

#### 高可用性

没有单点故障。
提供内置且可配置的数据复制备份机制。         
通过跨数据中心的数据备份，提供数据的灾难恢复。


#### 简单的管理与维护

除了命令行之外，还提供基于Web的界面管理工具。提供对系统及数据节点的控制控制。
可以查看系统的拓扑结构、系统状态参数、当前负载情况、请求延迟记录、内部事件及通知等信息。

### 2、存储术语

Partition/分区：整个数据库有多个分区组成。分区数在创建数据库时指定。运行时，键通过Hash计算。分配到特定的分区。

Replicationgroup/复制组：多个分区组合成一个复制组。

Store/数据库：多个复制组组成的一个完整的OracleNoSQL数据库。

Masterreplication node/主节点：每个复制组里有一个主节点。对该复制组的写操作在主节点中完成，然后复制到其他节点。

Replicanode/复制节点：每个复制组里有零到多个复制节点，同时提供分布性读和高可用性，在主节点故障时，某个复制节点会被选举为新的主节点。

Storagenode/存储节点：物理/虚拟服务器上运行的NoSQL的实例。通常每个机器对应一个节点。

Replicationfactor/复制因子：每个复制组的存储节点数目。

### 3、数据模型

#### key

OracleNoSQL使用key来组织数据，且Key呈现一种树形结构。数据组织方式对效率有重要影响，采用什么样的组织方式取决于数据的访问模式。Key中不能带有空格。根据key来分区：

#### value

Oracle NoSQL还支持以Table形式组织数据，并提供了Tables API

### 4、分区架构


### 5、安装部署

测试中按照3X3的网格建立了，1个Zone/3个storage node, 每个storage node里有一个master及两个Replication.

```
root@ubuntu:~/scripts# java -jar $KVHOME/lib/kvstore.jar runadmin -port 5000 -host kvhost01

kv-> show plans
     1 Deploy Zone (1)          SUCCEEDED
     2 Deploy Storage Node (2)  SUCCEEDED
     3 Deploy Admin Service (3) SUCCEEDED
     4 Deploy Storage Node (4)  SUCCEEDED
     5 Deploy Admin Service (5) SUCCEEDED
     6 Deploy Storage Node (6)  SUCCEEDED
     7 Deploy Admin Service (7) SUCCEEDED
     8 Deploy Topo (8)          SUCCEEDED
     9 CreateTable:user_test    SUCCEEDED
    10 CreateIndex:user_test:idx1 SUCCEEDED

kv-> show topology
store=kvstore  numPartitions=120 sequence=136
  zn: id=zn1 name=Houston repFactor=3 type=PRIMARY allowArbiters=false

  sn=[sn1] zn:[id=zn1 name=Houston] kvhost01:5000 capacity=3 RUNNING
    [rg1-rn1] RUNNING
             single-op avg latency=0.0 ms   multi-op avg latency=0.0 ms
    [rg2-rn1] RUNNING
             single-op avg latency=0.0 ms   multi-op avg latency=0.0 ms
    [rg3-rn1] RUNNING
             single-op avg latency=0.43896294 ms   multi-op avg latency=0.0 ms
  sn=[sn2] zn:[id=zn1 name=Houston] kvhost02:6000 capacity=3 RUNNING
    [rg1-rn2] RUNNING
             single-op avg latency=0.46328074 ms   multi-op avg latency=0.0 ms
    [rg2-rn2] RUNNING
             single-op avg latency=0.0 ms   multi-op avg latency=0.0 ms
    [rg3-rn2] RUNNING
             single-op avg latency=0.0 ms   multi-op avg latency=0.0 ms
  sn=[sn3] zn:[id=zn1 name=Houston] kvhost03:7000 capacity=3 RUNNING
    [rg1-rn3] RUNNING
             single-op avg latency=0.0 ms   multi-op avg latency=0.0 ms
    [rg2-rn3] RUNNING
             single-op avg latency=0.4089808 ms   multi-op avg latency=0.0 ms
    [rg3-rn3] RUNNING
             single-op avg latency=0.0 ms   multi-op avg latency=0.0 ms

  shard=[rg1] num partitions=40
    [rg1-rn1] sn=sn1
    [rg1-rn2] sn=sn2
    [rg1-rn3] sn=sn3
  shard=[rg2] num partitions=40
    [rg2-rn1] sn=sn1
    [rg2-rn2] sn=sn2
    [rg2-rn3] sn=sn3
  shard=[rg3] num partitions=40
    [rg3-rn1] sn=sn1
    [rg3-rn2] sn=sn2
    [rg3-rn3] sn=sn3


kv-> verify
The command:

        verify [-silent]

is deprecated and has been replaced by:

        verify configuration [-silent]

Verify: starting verification of store kvstore based upon topology sequence #136
120 partitions and 3 storage nodes
Time: 2017-07-02 20:58:09 UTC   Version: 12.2.4.4.6
See kvhost01:/tmp/data/sn1/kvroot/kvstore/log/kvstore_{0..N}.log for progress messages
Verify: Shard Status: healthy:3 writable-degraded:0 read-only:0 offline:0
Verify: Admin Status: healthy
Verify: Zone [name=Houston id=zn1 type=PRIMARY allowArbiters=false]   RN Status: online:9 offline:0 maxDelayMillis:0 maxCatchupTimeSecs:0
Verify: == checking storage node sn1 ==
Verify: Storage Node [sn1] on kvhost01:5000    Zone: [name=Houston id=zn1 type=PRIMARY allowArbiters=false]    Status: RUNNING   Ver: 12cR2.4.4.6 2017-04-13 06:54:25 UTC  Build id: d6a9b947763f
Verify:         Admin [admin1]          Status: RUNNING,MASTER
Verify:         Rep Node [rg1-rn1]      Status: RUNNING,REPLICA sequenceNumber:2,822 haPort:5011 delayMillis:0 catchupTimeSecs:0
Verify:         Rep Node [rg2-rn1]      Status: RUNNING,REPLICA sequenceNumber:2,925 haPort:5012 delayMillis:0 catchupTimeSecs:0
Verify:         Rep Node [rg3-rn1]      Status: RUNNING,MASTER sequenceNumber:2,893 haPort:5013
Verify: == checking storage node sn2 ==
Verify: Storage Node [sn2] on kvhost02:6000    Zone: [name=Houston id=zn1 type=PRIMARY allowArbiters=false]    Status: RUNNING   Ver: 12cR2.4.4.6 2017-04-13 06:54:25 UTC  Build id: d6a9b947763f
Verify:         Admin [admin2]          Status: RUNNING,REPLICA
Verify:         Rep Node [rg1-rn2]      Status: RUNNING,MASTER sequenceNumber:2,822 haPort:6011
Verify:         Rep Node [rg2-rn2]      Status: RUNNING,REPLICA sequenceNumber:2,925 haPort:6012 delayMillis:0 catchupTimeSecs:0
Verify:         Rep Node [rg3-rn2]      Status: RUNNING,REPLICA sequenceNumber:2,893 haPort:6013 delayMillis:0 catchupTimeSecs:0
Verify: == checking storage node sn3 ==
Verify: Storage Node [sn3] on kvhost03:7000    Zone: [name=Houston id=zn1 type=PRIMARY allowArbiters=false]    Status: RUNNING   Ver: 12cR2.4.4.6 2017-04-13 06:54:25 UTC  Build id: d6a9b947763f
Verify:         Admin [admin3]          Status: RUNNING,REPLICA
Verify:         Rep Node [rg1-rn3]      Status: RUNNING,REPLICA sequenceNumber:2,822 haPort:7011 delayMillis:0 catchupTimeSecs:0
Verify:         Rep Node [rg2-rn3]      Status: RUNNING,MASTER sequenceNumber:2,925 haPort:7012
Verify:         Rep Node [rg3-rn3]      Status: RUNNING,REPLICA sequenceNumber:2,893 haPort:7013 delayMillis:0 catchupTimeSecs:0

Verification complete, no violations.

```
