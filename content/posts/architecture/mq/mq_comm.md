---
title: "[WIP] MQ"
date: 2022-02-02T22:15:10+08:00
draft: true
categories: [架构设计,MQ]
---

## Why MQ？

+ 异步处理
+ 解耦服务
+ 削峰填谷
+ 顺序消息

## 术语

|||
|--|--|
|Producer||
|Consumer||
|Topic||
|MessageQueue/Partition||
|Broker||
|Consumer Group||
|Rebalance||
|Message ordering||

## mq选型

|MQ|场景|设计目标|CAP|硬件|
|--|--|--|--|--|
|kafka|||||
|NSQ/RocketMQ|||||

[RocketMQ vs. ActiveMQ vs. Kafka](https://rocketmq.apache.org/docs/motivation/)

## Exactly Once

> [At least once, at most once, exactly once]()

||||
|--|--|--|
|两阶段提交||耗时长，不适用于低延时场景；<br/>需要参与的所有系统皆实现XA接口|
|at least once 加 下游幂等处理|实现简单|要求下有系统提供幂等处理接口|
|Offset更新与数据处理放在同一事物中||要求下有处理系统可回滚；<br/>需要系统支持事务功能|