---
title: "[WIP] Kafka"
date: 2022-02-02T22:15:02+08:00
draft: true
categories: [架构设计,MQ]
---

## 设计理念

1. 低延迟：时间复杂度O(1)
2. 高吞吐
3. 水平扩展
4. 顺序性：保证每个partition内的消息顺序传输
5. 多场景：支持离线数据和实时数据

## 架构

Broker（kafka svr）依赖Zookeeper做：
1. 元数据管理
2. 领导选举

消费是PULL模型：
|消费模型|优点|缺点|
|--|--|--|
|PUSH|延迟低|容易怼挂消费者|
|PULL|削峰效果好：消费者根据处理能力拉数据；<br/> 可以批量PULL = 性能更好|延迟相对高|

数据
+ Record
    + key - value
    + timestamp
+ Topic
    + 逻辑概念
    + 发布-订阅均基于Topic
+ Partition
    + 一个Topic包含一个或多个Partion
        + 均匀分布在多个Broker - 以达到高并行处理能力
    + 每个Partition物理上对应一个文件夹（每一段对应一个Segment，及一个文件）

Producer
+ 何时发送消息
    + 异步（默认）
    + 同步：flush()
+ 如何保证消息的顺序性
    + Queue与Retry机制，可能会导致先发送的消息（因失败退避重试）后到Broker
    + `max.in.flight.requests.per.connection`
+ 消息路由策略，partitioner

Consumer
+ Low level API/Assign
    + 指定目标Partition
    + 指定消费的起始Offset
    + 指定每次消费的消息长度
    + 可只消费某Topic内的特定Partion的特定消息
+ High level API/Subscriber
    + 每个Consumer实例属于特定的Consumer Group
        + 每个Partion只会分配给一个Consumer（Consumer数 > Partition数 会导致有Consumer没有分配Partition）
    + 默认情况下，Consumer Group会顺序消费（Partition内顺序）某Topic的所有消息
    + Offset存于Zookeeper或kafka或者自定义存储
    + 实现了Rebalance机制

Consumer Group
+ 组内单播
+ 组内广播

## Consumer Group Rebalance

增减Consumer自适应的调整Partion分配给Consumer的情况

|Rebalance方式|说明|优点|缺点|
|--|--|--|--|
|自治式|Consumer监听Broker和Consumer变更事件触发Rebalance|简单|1. 任何Broker或者Consumer的增减都会触发所有的Consumer的Rebalance<br/>2. 脑裂：不同的Consumer看到的数据可能不一样（Zookeeper特性导致）<br/>3. 调整结果不可控，Consumer消费有问题，其他Consumer可能并不知道|
|**集中式**|基于Coordinator的Rebalance||解决结果一致|

## 高可用原理

> [CAP理论]()

备份，数据复制

Producer -> Partion Leader -ISR-> Partion Follower（PULL）

基于ISR的动态平衡
+ Leader会维护一个与其基本保持同步的Replica列表（包括他自己），该列表称为ISR（in-sync Replica）
    + 也有Replica不在ISR中，当所有ISR中的节点故障，可选其为Leader
+ 如果一个Follower比Leader落后太多，或者超过一定时间未发起数据复制请求，则Leader将其从ISR中移除
+ 当ISR中所有Replica都向Leader发送ACK时，Leader即Commit

Leader Failover过程：

## Exactly Once

... link to mq_comm?