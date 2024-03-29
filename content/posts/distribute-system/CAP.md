---
title: "CAP"
date: 2022-02-02T09:36:38+08:00
draft: false
categories: 分布式系统
---

## CAP理论[^1]

在理论计算机科学中，CAP定理（CAP theorem），又被称作布鲁尔定理（Brewer's theorem），它指出对于一个分布式计算系统来说，不可能同时满足以下三点：

+ 一致性（Consistency） （等同于所有节点访问同一份最新的数据副本）
+ 可用性（Availability）（每次请求都能获取到非错的响应——但是不保证获取的数据为最新数据）
+ 分区容错性（Partition tolerance）（以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。）

根据定理，分布式系统只能满足三项中的两项而不可能满足全部三项[4]。理解CAP理论的最简单方式是想象两个节点分处分区两侧。允许至少一个节点更新状态会导致数据不一致，即丧失了C性质。如果为了保证数据一致性，将分区一侧的节点设置为不可用，那么又丧失了A性质。除非两个节点可以互相通信，才能既保证C又保证A，这又会导致丧失P性质。

## 数据一致性[^2]

分布式系统中，一般为了容错性，会引入多副本，多个副本间需要同步状态，就引入了一致性问题。一致性问题包含**数据一致性**和**事务一致性**（指ACID）。

DDIA第五章更详细的给出了集中供暖需要备份数据的情况：
+ 为了保持需要地理上离用户更近（降低延迟）
+ 为了允许系统在某些部分故障时仍然能持续工作（增加可用性）
+ 为了扩充读服务的机器数量（增加读性能）

其实只有两类数据一致性，强一致性与弱一致性。强一致性也叫做线性一致性，除此以外，所有其他的一致性都是弱一致性的特殊情况。所谓强一致性，即复制是同步的，弱一致性，即复制是异步的。

### 强一致性

强一致性可以保证从库有与主库一致的数据。如果主库突然宕机，我们仍可以保证数据完整。但如果从库宕机或网络阻塞，主库就无法完成写入操作。

在实践中，我们通常使一个从库是同步的，而其他的则是异步的。如果这个同步的从库出现问题，则使另一个异步从库同步。这可以确保永远有两个节点拥有完整数据：主库和同步从库。 这种配置称为半同步。


### 弱一致性

#### 最终一致

容忍节点故障只是需要复制的一个原因。另两个原因是可扩展性和降低延迟。

单领导者的主从复制算法要求所有写入都由单个节点处理，但只读查询可以由任何节点处理。对于读多写少的场景，我们往往创建很多从库，并将读请求分散到所有的从库上去。这样能减小主库的负载，并允许向最近的节点发送读请求。当然这只适用于异步复制——如果尝试同步复制，则单个节点故障将使整个系统无法写入。

当用户从异步从库读取时，如果此异步从库落后，他可能会看到过时的信息。这种不一致只是一个暂时的状态——如果等待一段时间，从库最终会赶上并与主库保持一致。这称为最终一致性。

最终两个字用得很微妙，因为从写入主库到反映至从库之间的延迟，可能仅仅是几分之一秒，也可能是几个小时。

#### 读写一致（读己之写一致）

手机刷虎扑的时候经常遇到，回复某人的帖子然后想马上查看，但我刚提交的回复可能尚未到达从库，看起来好像是刚提交的数据丢失了，很不爽。

在这种情况下，我们需要读写一致性，也称为读己之写一致性。它可以保证，如果用户刷新页面，他们总会看到自己刚提交的任何更新。它不会对其他用户的写入做出承诺，其他用户的更新可能稍等才会看到，但它保证用户自己提交的数据能马上被自己看到。

如何实现读写一致性？

最简单的方案，对于某些特定的内容，都从主库读。举个例子，知乎个人主页信息只能由用户本人编辑，而不能由其他人编辑。因此，永远从主库读取用户自己的个人主页，从从库读取其他用户的个人主页。

如果应用中的大部分内容都可能被用户编辑，那这种方法就没用了。在这种情况下可以使用其他标准来决定是否从主库读取，例如可以记录每个用户最后一次写入主库的时间，一分钟内都从主库读，同时监控从库的最后同步时间，任何超过一分钟没有更新的从库不响应查询。

还有一种更好的方法是，客户端可以在本地记住最近一次写入的时间戳，发起请求时带着此时间戳。从库提供任何查询服务前，需确保该时间戳前的变更都已经同步到了本从库中。如果当前从库不够新，则可以从另一个从库读，或者等待从库追赶上来。

#### 单调读

用户从某从库查询到了一条记录，再次刷新后发现此记录不见了，就像遇到时光倒流。如果用户从不同从库进行多次读取，就可能发生这种情况。

单调读可以保证这种异常不会发生。单调读意味着如果一个用户进行多次读取时，绝对不会遇到时光倒流，即如果先前读取到较新的数据，后续读取不会得到更旧的数据。单调读比强一致性更弱，比最终一致性更强。

实现单调读取的一种方式是确保每个用户总是从同一个节点进行读取（不同的用户可以从不同的节点读取），比如可以基于用户ID的哈希值来选择节点，而不是随机选择节点。

#### 因果一致性

在本文中阐述因果一致性可能并不是一个很好的时机，因为它往往发生在分区（也称为分片）的分布式数据库中。

分区后，每个节点并不包含全部数据。不同的节点独立运行，因此不存在全局写入顺序。如果用户A提交一个问题，用户B提交了回答。问题写入了节点A，回答写入了节点B。因为同步延迟，发起查询的用户可能会先看到回答，再看到问题。

为了防止这种异常，需要另一种类型的保证：因果一致性。 即如果一系列写入按某个逻辑顺序发生，那么任何人读取这些写入时，会看见它们以正确的逻辑顺序出现。

这是一个听起来简单，实际却很难解决的问题。一种方案是应用保证将问题和对应的回答写入相同的分区。但并不是所有的数据都能如此轻易地判断因果依赖关系。如果有兴趣可以搜索向量时钟深入此问题。

[^1]: [CAP定理](https://zh.wikipedia.org/wiki/CAP%E5%AE%9A%E7%90%86)
[^2]: [通俗易懂 强一致性、弱一致性、最终一致性、读写一致性、单调读、因果一致性 的区别与联系](https://zhuanlan.zhihu.com/p/67949045)

