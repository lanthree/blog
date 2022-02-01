---
title: "跳表"
date: 2022-02-02T21:33:29+08:00
draft: false
categories: [八股文,数据结构与算法]
---

## 简介

跳表（skip list）是一个有序链表的随机化的变体，有附加的、平行的链表。在上层的平行链表指数级的跳过多个元素。为了快速找到正确的部分，搜索从最高层开始，递进地进入底层。加入元素时，先随机选择层高，然后给该层以及所有下层的链表的正确顺序的位置加入该元素。

![一个典型的跳表示意图](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/skip_list.png)

|算法|平均|最差|
|--|--|--|
|Space|$O(n)$|$O(n\log n)$|
|Search|$n\log n)$|$O(n)$|
|Insert|$n\log n)$|$O(n)$|
|Delete|$n\log n)$|$O(n)$|

## 实现

### 数据结构以及初始化

```cpp
template <typename Comparable, typename Value>
class SkipList {

  struct Node {
    Comparable key;
    Value val;
    Node* forward[MAXLEVEL];
  };

 public:
  SkipList() {}

 private:
  Node* hdr_;       // 链表头
  int list_level_;  // 链表当前level
};
```

定义`Node`为跳表的一个结点（注意，这个结点包含跳表的多个层级的指针`forward`，最低0，最高`MAXLEVEL-1`）。下图左边为一个有两个key的跳表的内部结构示意图。第0个结点只用于维护表头，不存储数据；每个结点都有一个`forward`数组，维护该结点对应层级的下一个结点的地址：例如`node1`层级是0，那么只有`forward[0]`有效，并指向了`node2`；而`node2`的层级是2，那么`forward[0~2]`都有效，虽然都指向了`nullptr`：

![MAXLEVEL=4；为了防止结点太多后箭头不清晰，下面讨论查找、插入、删除时用右面图替代](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/skip_list_sample.png)

初始化的时候，新建`node0`，并把其`forward`均设置指向`nullptr`，跳表高度设置为0；

![](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/skip_list_init.png)

```cpp
StatusEnum Init() {
  hdr_ = new (std::nothrow) Node();
  if (hdr_ == 0) return STATUS_MEM_EXHAUSTED;

  for (int i = 0; i < MAXLEVEL; ++i) hdr_->forward[i] = nullptr;
  list_level_ = 0;
  return STATUS_OK;
}
```

### 查找

查找时，从高层（跳表当前高度）向下查找，每当当前层的当前结点的下一项的key 大于 要查找的key时，就下降一层，否则在当前层前进；走到最后一层的时候停止：

![](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/skip_list_find.png)

```cpp
StatusEnum Find(const Comparable& key, Value& val) const {
  Node* x = hdr_;
  for (int i = list_level_; i >= 0; --i) {
    // x 的下一项是空 或者 x的下一项大于key 则停止
    // x 本身小于key
    while (x->forward[i] != nullptr && x->forward[i]->key < key) {
      x = x->forward[i];
    }   
  }   
  x = x->forward[0];  // 检查前置项的下一项
  if (x != nullptr && x->key == key) {
    val = x->val;
    return STATUS_OK;
  }   
  return STATUS_KEY_NOT_FOUND;
}
```

### 插入

插入时，首先跟查找一样，要逐层渐进式下降查找要插入的位置。另外，因为需要在链表插入一个结点，所以需要记录前置结点，插入时，新结点指向前置结点的`forward`，前置结点指向新结点；因为有多层结构，所以要记录一个“前置结点”数组：

```cpp
StatusEnum Insert(const Comparable& key, const Value& val) {
  Node* update[MAXLEVEL];
  Node* x = hdr_;
  // 找应该插在哪
  for (int i = list_level_; i >= 0; --i) {
    // x 的下一项是空 或者 x的下一项大于key 则停止
    // x 本身小于key
    while (x->forward[i] != nullptr && x->forward[i]->key < key) {
      x = x->forward[i];
    }
    update[i] = x;  // 要更新的是小于key的项；记录第i层要更新的项，即新插入结点的前置项
  }
  x = x->forward[0];  // 检查前置项的下一项
  // 如果前置项的下一项的值与插入值的key相等，则key重复
  if (x != nullptr && x->key == key) return STATUS_DUPLICATE_KEY;

  // 随机新结点的层级
  int new_level = _genLevel();
  // 如果新结点的层级 比 跳表当前层级还高
  if (new_level > list_level_) {
    // 那比当前层级还高的那些层级的 前置项 就是hdr_
    for (int i = list_level_ + 1; i <= new_level; ++i) update[i] = hdr_;
    list_level_ = new_level;
  }

  // 新建结点
  x = new (std::nothrow) Node();
  if (x == 0) return STATUS_MEM_EXHAUSTED;
  x->key = key;
  x->val = val;

  for (int i = 0; i <= new_level; ++i) {
    // 新结点每一层级的下一结点 就是 每一层级前置项的下一结点
    x->forward[i] = update[i]->forward[i];
    // 每一层级前置项的下一结点 都是 新结点
    update[i]->forward[i] = x;
  }
  return STATUS_OK;
}
```

插入结点的层高是随机出来的，例如用如下随机函数：

```cpp
int _genLevel() {
  int level = 0;
  while (rand() < RAND_MAX / 4 && level < MAXLEVEL - 1) level++;
  return level;
}
```

### 删除

删除跟插入是一样的过程，也需要记录前置结点数组，只需要更新前置结点指向 要删除结点 的下一项，最后删除该结点即可：

```cpp
StatusEnum Delete(const Comparable& key) {
  Node* update[MAXLEVEL];
  Node* x = hdr_;
  // 找要删除的项
  for (int i = list_level_; i >= 0; --i) {
    // x 的下一项是空 或者 x的下一项大于key 则停止
    // x 本身小于key
    while (x->forward[i] != nullptr && x->forward[i]->key < key) {
      x = x->forward[i];
    }
    update[i] = x;  // 要更新的是小于key的项；记录第i层要更新的项，即删除结点的前置项
  }
  x = x->forward[0];  // 检查前置项的下一项
  // 如果前置项的下一项的值与要删除的key不相等，则没有该key
  if (x != nullptr && x->key != key) return STATUS_KEY_NOT_FOUND;

  // 查看更新数组（记录要删除项的前置项）中的每个层级
  for (int i = 0; i <= list_level_; ++i) {
    if (update[i]->forward[i] != x) break;
    // 把前置项该层级的forward 设置为 要删除结点的对应层的forward
    update[i]->forward[i] = x->forward[i];
  }

  // 删除该项
  delete x;

  while ((list_level_ > 0) && (hdr_->forward[list_level_] == nullptr)) {
    --list_level_;
  }
  return STATUS_OK;
}
```

## 性能参考

即使是上面的简单实现（`MAXLEVEL=10`），性能跟基础库的`set`以及`unordered_map`也不会差太多（甚至插入时性能优于`set`）：

```
# 2w items
set           insert: 202ms
unordered_map insert: 109ms
SkipList      Insert: 151ms

set           count: 175ms
unordered_map count: 69ms
SkipList      Find:  186ms
```

可执行代码[参考](https://wandbox.org/permlink/wr5FlLjB1oXLvEGY)。

## 参考

1. [skiplist from xlinux.nist.gov](https://xlinux.nist.gov/dads/HTML/skiplist.html)
2. [wikipedia Skip_list](https://en.wikipedia.org/wiki/Skip_list)
