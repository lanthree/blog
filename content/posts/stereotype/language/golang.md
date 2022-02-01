---
title: "[WIP] golang"
date: 2022-02-02T21:49:17+08:00
draft: true
categories: [八股文,编程语言]
---

## 流程语句

### for

```go
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

遍历容器
```go
for key, value := range oldMap {
    newMap[key] = value
}

// 甚至可以遍历字符串
for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
    fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}
// print
// character U+65E5 '日' starts at byte position 0
// character U+672C '本' starts at byte position 3
// character U+FFFD '�' starts at byte position 6
// character U+8A9E '語' starts at byte position 7

// 只要key
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}

// 只要value
sum := 0
for _, value := range array {
    sum += value
}
```

### switch

没有switch value等同于switch true
```go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

switch只会命中第一个case并执行它的代码段，不会因为没有break而fall through，但是你可以写case列表clumsy
```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```

## Defer

defer的参数在defer语句出决定

```go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
//4 3 2 1 0
```

## 数据

### new

`new T`为类型T分配一块初始化为0的空间，并返回`*T`类型的地址。

### make

用于创建slice、map、和channel，并返回初始化后的对象本身（而非地址）

## 常用类型

### 数组

+ 数组是值，把一个数组赋值给另一个数组，回copy所有元素
    + 特别的，如果传数组给函数，函数拿到的是他的一个copy
+ 数组的大小是类型的一部分：`[10]int`和`[20]int`是不同的类型

### slice



```go
type slice struct {
    array unsafe.Pointer
    len int
    cap int
}
```

```golang
// slice
var s [] int  // 此时s为nil
              // 如果写成 s := []int{} 或者 s := make([]int) s不为nil
s := make([]int, len, cap)

// array
var a [len]int // 长度是类型的一部分，即长度不一样不能作为相应的函数参数传入
```

growslice：
+ 当cap小于1024，每次double
+ 当cap大于等于1024，每次\*1.25

特别注意：初次append时（cap=0），以append的个数为准。

当slice作为参数传入函数：
+ 如果没有发生扩容，修改在原来的内存中
+ 如果发生了扩容，修改会在新的内存中

#### 二维slice

```go
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.
```

```go
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
```

### map

map持有底层数据结构的引用，所以在作为参数传递到函数后，函数内的修改会影响到函数外的原始map对象

```go
func modifykv(kv map[string]string) {
    kv["k1"] = "modify_v1"
}

kv := map[string]string{
    "k1": "v1",
    "k2": "v2",
}
modifykv(kv)
fmt.Println(kv)
// map[k1:modify_v1 k2:v2]
```

map删除key时[不会自动缩容](https://github.com/golang/go/issues/20135)

[深度解密Go语言之map](https://qcrao.com/2019/05/22/dive-into-go-map/)

### itoa

https://segmentfault.com/a/1190000000656284

### channel

+ 有锁
    + 高并发、高性能编程不适合使用
    + https://github.com/golang/go/issues/8899
+ 底层是ringbuffer
+ 会触发调度

```go
func main() {
}
```

### interface{}

## Embedding


[深度解密Go语言之channel](https://qcrao.com/2019/07/22/dive-into-go-channel/)


## 并发安全问题&&解决方案

DataRace
+ 原因：多个gorontine同时接触一个变量，行为不可预知
+ 认定条件：两个及以上goroutine同时接触一个变量，其中至少一个gorontine为写
+ 检测方案：go run/test -race
+ 结局方案：atomic，sync.Mutex/sync.Mutex等，限制访问数据

## sync.Mutex

+ 减少持有时间
+ 优化锁的粒度
    + 并发获取随机数（rand.Rand中有锁），通过拆分rand.Rand缩小锁粒度
    ```go
    type SafeRander struct {
        pos     uint32
        randers [128]*rand.Rand
        locks   [128]sync.Mutex
    }

    func (sr *SafeRander) Init() {
        for i := 0; i < 128; i++ {
            seed := time.Now().UnixNano()
            sr.randers[i] = rand.New(rand.NewSource(seed))
        }

    }

    func (sr *SafeRander) Intn(n int) int {
        x := atomic.AddUint32(&sr.pos, 1)
        x %= 128
        sr.locks[x].Lock()
        n = sr.randers[x].Intn(n)
        sr.locks[x].Unlock()
        return n
    }
    ```
+ 读写分离 
    + RWMutex
    + sync.Map
+ 使用原子操作 Lock Free
    + sync.atomic

race detector

自旋锁
```go
type SpinLock int32

func (s *SpinLock) Lock() {
	for !atomic.CompareAndSwapInt32((*int32)(s), 0, 1) {
	}
}

func (s *SpinLock) UnLock() {
	atomic.StoreInt32((*int32)(s), 0)
}
```

[Mutex](https://cs.opensource.google/go/go/+/refs/tags/go1.17.5:src/sync/mutex.go;drc=refs%2Ftags%2Fgo1.17.5;l=25) 效率优先，兼顾公平
+ 正常模式
    + 等待队列：先进先出
    + 新手优势：先抢再排（不用进入等待队列，减少调度开销，充分利用缓存）
    + 等到超1ms：饥饿模式
+ 饥饿模式
    + 严格排队，队收接盘
        + 牺牲效率，保P99
    + 适时回归（是最后一个 or 等待不到1ms） -> 正常模式


## 并发数据结构和算法

+ 并发写，通过锁来约束只有一个能写
+ 并发读，使用atomic（要求写也要是atomic）

TODO：有序链表、跳表

## schedule

### 调度循环

GM模型 GPM模型

如何建立调度循环

TODO：逻辑图，模型图、源码阅读

### 协作与抢占

+ （同步）协作式调度：依靠被调度方主动弃权
    + 主动用户让权：通过runtime.Gosched调用主动让出执行机会
    + 主动调度弃权：执行栈分段时检查自身的枪战标记，决定是否继续执行
+ （异步）抢占是调度：依靠调度器强制将被调度方中断
    + 当G执行时间过长时，sysmon会抢占G
    + 被动GC抢占，当需要进行垃圾回收时，强制停止所有G

## 垃圾回收 - garbage collection



## 内存管理

## 性能分析

## 高效代码

## 应用

