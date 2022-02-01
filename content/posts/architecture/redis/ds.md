---
title: "Redis中的数据结构"
date: 2022-02-02T22:14:52+08:00
draft: false
categories: [架构设计,Redis]
---

## 键值如何组织

为了实现从键到值的快速访问O(1)，Redis使用hash表来组织键值映射。

### 全局hash表

![全局hash表](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/redis_hashtable.png)

hash表中每个桶中的元素（entry）保存了`*key`和`*value`，分别指向实际的键和值。这个hash表在内存中，维护了所有的键值对。当hash冲突时，Redis使用拉链法解决冲突：

![hash冲突](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/redis_hashtable_confict.png)

桶中链表上的元素，在查找时需要O(n)的遍历，当链表过长时，会直接导致元素查找时间过长，效率降低。此时，Redis会对hash表做rehash操作：增加现有的hash桶数量，让逐渐增多的entry元素能在更多的桶之间分散保存，减少单个桶中的元素数量。

### 渐进式rehash

为了使rehash操作更高效，Redis使用两个全局hash表，hash表1和hash表2。最开始，Redis默认使用hash表1，此时hash表2没有被分配空间。rehash时，会给hash表2分配更大的空间，然后将hash表1中的元素rehash到hash表2，最后切换为hash表2并释放hash表1的空间。

当下一次需要rehash时，会重复上面步骤（只是会从hash表2 rehahs到hash表1）。

因为rehash过程可能涉及大量的数据拷贝，一次性完成的话，会造成Redis阻塞，所以Redis采用了**渐进式rehash**：每处理一个请求时，从hash表1中第一个索引开始，将第一个索引下的所有entry rehash到hash表2，下一次请求时，处理下一个索引。

![rehash过程](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/redis_hashtable_rehash.png)

这里hash表1处理请求时，顺带从头开始rehash内容，而不是rehash当前处理的位置。有个全局变量记录rehash的进度。rehash过程，entry从原表**移动**到新表，而且rehash时，`PUT`操作只会在新表生效。

?> 这里需要从代码确认下 TODO

## 值的数据结构

### 简单动态字符串

先看它的结构：

```cpp
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len;   /* 已使用的长度 */
    uint32_t alloc; /* buf总可用（包括已使用）长度；不包括sdshdr结构体的大小，以及buf结尾'\0'占用的1字节 */
    unsigned char flags; /* 前3bits表示类型, 后5bits未使用 */
    char buf[];
};
```

为了更高效的（减少浪费）使用内存，`sdshdr`有不同的类型（用flag标识）：`sdshdr8`、`sdshdr16`、`sdshdr32`、`sdshdr64`（在生成新字符串时，可以动态的根据字符串长度，生成不同类型的结构）。不同的类型，其`len`、`alloc`的数据类型不同，例如`sdshdr16`对应`uint16_t`。节省hdr结构空间的同时，约束了buf的最大长度。另外，sds提供的API中，buf结构是二进制安全的，不会因为数组中有'\0'而被截断（使用len判断长度）。但是如果用于打印，那就不保证不被截断了。

在代码中，上面结构只是sds的**头**的定义，sbs本身被定义为`typedef char *sds;`，相当于在代码中，`sdshdr`结构中的`buf`作为sds传递，sds的API为了适配（或者说 是sds为了适配API？），会通过数组下标`-1`取到`flags`，再操作：

```cpp
static inline size_t sdslen(const sds s) {
    unsigned char flags = s[-1];
    switch(flags&SDS_TYPE_MASK) {
        case SDS_TYPE_5:
            return SDS_TYPE_5_LEN(flags);
        case SDS_TYPE_8:
            return SDS_HDR(8,s)->len;
        case SDS_TYPE_16:
            return SDS_HDR(16,s)->len;
        case SDS_TYPE_32:
            return SDS_HDR(32,s)->len;
        case SDS_TYPE_64:
            return SDS_HDR(64,s)->len;
    }
    return 0;
}
```

### 双向链表

### 压缩列表

在列表、散列和有序集合的长度较短或者体积较小的时候，Redis可以选择使用一种名为压缩列表（ziplist）的紧凑存储方式来存储这些结构。压缩列表一块连续的内存空间（可以减少内存碎片），所有元素紧挨在一起，头部有三个字段`zlbytes`、`zltail`和`zllen`，尾部有一个`zlend`。

![](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/redis_zlist.png)

+ zlbytes：总字节数（包含自身），`uint32_t`
+ zltail：列表尾部偏移量（列表头地址 到 最后一个元素），可以快速访问尾部元素，`uint32_t`
+ zllen：元素个数；`uint16_t`
+ zlend：列表结束标识，特殊值`0xFF`，`uint8_t`

压缩列表中entry的详细组织方式，可以参考[这里](https://www.cnblogs.com/hunternet/p/11306690.html)。

### hash表

### 跳表

### 整数数组

## 参考

1. 极客时间，《Redis核心技术与实战》
2. [An introduction to Redis data types and abstractions](https://redis.io/topics/data-types-intro)
3. [Redis代码](https://redis.io/download)
