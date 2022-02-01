+++
title = "学习笔记"
+++

## 学习计划

+ [DONE] LeetCode 序号前100的 简单、中等难度题目 **重点准备**
    + [ING] 倒刷[高频算法题](https://zhuanlan.zhihu.com/p/386929820)
+ [ING] 分布式系统 **重点准备**
    + [ING] 分布式论文
        + [Google三宝 - MapReduce](https://engineers.cool/#/pages/分布式系统/Papers/MapReduce)
        + [Google三宝 - GFS](https://engineers.cool/#/pages/分布式系统/Papers/GFS)
        + [Raft](https://engineers.cool/#/pages/分布式系统/Papers/Raft)
    + [ING] DDIA
    + [PAUSE] [MIT 6.824](https://pdos.csail.mit.edu/6.824/schedule.html)
    + [CAP](https://engineers.cool/#/pages/分布式系统/CAP)
    + [TODO] 一致性协议：Paxos Raft gossip
    + [TODO] 分布式锁
    + [TODO] ID生成器
+ [TODO] 八股文
    1. [TODO] TCP/IP协议
        1. [协议栈 TPC握手/挥手/状态转移](https://engineers.cool/#/pages/八股文/网络/cheatsheet?id=tcp)
        2. 常见问题准备
    2. [TODO] 数据结构与算法 **重点准备**
        1. 算法思想 NP完全性
        2. [排序](https://engineers.cool/#/pages/八股文/数据结构与算法/sort)
        3. 数组、链表（[跳表](https://engineers.cool/#/pages/八股文/数据结构与算法/skip_list)）、队列（二项队列）、栈、 堆、优先队列
        4. 树：[AVL树](https://engineers.cool/#/pages/八股文/数据结构与算法/avl.md) 伸展树 [B树](https://engineers.cool/#/pages/八股文/数据结构与算法/btree) [B+树](https://engineers.cool/#/pages/八股文/数据结构与算法/b+tree) [红黑树](https://engineers.cool/#/pages/八股文/数据结构与算法/rbtree) 线段树
        5. 散列
        6. 图论：最小生成树、[拓扑排序](https://engineers.cool/#/pages/八股文/数据结构与算法/graph/tp_sort) 、最短路径、二分图
    3. [TODO] 现代操作系统
        + Unix环境编程-锁、段页
    4. [TODO] 数据库
        + mysql原理（innodb、[事务隔离级别](https://engineers.cool/#/pages/八股文/数据库/transaction)、自增主键的好处、[锁](https://engineers.cool/#/pages/八股文/数据库/lock)） **重点准备**
        + LSM-tree Bitcask
    5. [TODO] [C/C\+\+](https://engineers.cool/#/pages/八股文/编程语言/cpp) [《Effective C++》](https://engineers.cool/#/pages/八股文/编程语言/effective_cpp)
    6. [TODO] 设计模式
    7. [TODO] FAQ
+ [TODO] 开源软件学习
    + level db
+ [TODO] 架构设计
    1. Redis vs Memcached - TODO **重点准备**
    2. 名字服务/服务发现 - TODO
    3. mq 原理 **重点准备**
    4. chubby / zookeeper
    5. UML：类图

## About Me

{{< figure class="avatar" src="/avatar.jpg" alt="avatar">}}

This is a Hugo based resume template. You can find the full source code on
[GitHub](https://github.com/ojroques/hugo-researcher).

## Research Interest

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aliquam finibus ipsum
ac erat aliquam dapibus. Vestibulum vehicula placerat ex, a consectetur odio
pharetra quis[^1]. Mauris id urna ante.

Fusce pharetra diam ac nisi aliquet, velegestas ex iaculis. Pellentesque
laoreet cursus tellus sed pellentesque. Praesent a rhoncus elit[^2]. Nunc
ipsum nisl, consequat sit amet pretium quis, gravida id ipsum.

## Publications

In chronological order:
1. F.Bar, J.Doe: Effects of having a placeholder of a name
2. S.Holmes, J.Watson: Consequences of living with a sociopath in London

## Typography

This is a [link](http://google.com). Something *italics* and something **bold**.

Here is a table:

Year | Award | Category
-----|-------|--------
2014 | Emmy  | Won Outstanding Lead Actor in a miniseries or a movie
2015 | BAFTA | Nominated for Best Leading Actor for Sherlock
2014 | Satellite | Won Best Actor miniseries or television film

Here is a horizontal rule:

---

Here is a blockquote:

> To a great mind, nothing is little

Here is a `code` block:

```python
def is_elementary():
  return True
```

```c++
template <typename Comparable>
class AvlTree {
 private:
  void _singleRotateWithLeftChild(AvlNode*& N) {
    AvlNode* L = N->left;

    N->left = L->right;
    L->right = N;
    N->height = max(_height(N->left), _height(N->right)) + 1;
    L->height = max(_height(L->left), N->height) + 1;

    N = L;
  }
};
```

## References

* Foo Bar: Head of Department, Placeholder Names, Lorem
* John Doe: Associate Professor, Department of Computer Science, Ipsum

[^1]: This is the first footnote.
[^2]: This is the second footnote.
