---
title: "拓扑排序"
date: 2022-02-02T21:33:19+08:00
draft: false
categories: [八股文,数据结构与算法]
---

拓扑排序（topological sort）是对有向无环图的顶点的一种排序。[LeetCode 210.课程表II](https://engineers.cool/#/pages/LeetCode/LeetCode/210)基本是一道模板题，可以参考这道题。


伪代码：
```cpp
void Graph::toposort() {
  queue<Vertex> q;
  int counter = 0;

  for Vectex &v : vectexes_:
    if v.indegree == 0:
      q.push(v);
    
  while not q.empty():
    Vertex& v = q.top(); q.pop();
    v.topoNum = ++counter;

    for Vectex &w : v.linked:
      if (--w.indegree == 0):
        q.push(w);

  if counter != VECTEX_NUM:
    throw CycleFoundException();
}
```