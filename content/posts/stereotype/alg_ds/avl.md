---
title: "AVL树"
date: 2022-02-02T21:34:05+08:00
draft: false
categories: [八股文,数据结构与算法]
---

AVL（Adelson-Velskii and Landis）树是带有平衡条件的（balance condition）的二叉查找树：一颗AVL树是其每个节点的左子树和右子树的高度最多差1的二叉查找树。

> 高度是指从该节点到叶子节点的最长简单路径边的条数。叶子结点高度为0，NIL结点高度-1。

![左边是AVL树，右边不是](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/avl_demo.svg)

可以证明，数高为O(logN)。在假设 删除为懒惰删除 情况下，删除和查找都可以在简单地在O(logN)复杂度内执行。只有插入操作，因为可能会破坏平衡条件，需要做操作，我们称其为*旋转*（rotation）。在插入以后，只有那些从插入点到根结点的路径上的结点的平衡可能被改变，因此只有这些结点的可能需要平衡，我们沿着这条路径平衡该树。

可以如下，简单定义AVL树：
```cpp
template <typename Comparable>
class AvlTree {
  struct AvlNode {
    Comparable element;
    AvlNode* left;
    AvlNode* right;
    int height;

    AvlNode(const Comparable& that_ele, AvlNode* lt, AvlNode* rt, int h = 0)
        : element(that_ele), left(lt), right(rt), height(h) {}
  };

 public:
  AvlTree() : root(nullptr) {}
  int Height() const { return root == nullptr ? -1 : _height(root); }

 private:
  int _height(AvlNode* t) const { return t == nullptr ? -1 : t->height; }
 private:
  AvlNode* root;
};
```

## 插入

把当前需要平衡的结点记为N，AVL保证每个节点的两颗子树的高度最多差1，那么结点N两颗子树的高度差2。这种不平衡可能有以下4中情况：

1. N的**左**儿子的**左**子树进行一次插入
2. N的**左**儿子的**右**子树进行一次插入
3. N的**右**儿子的**右**子树进行一次插入
4. N的**右**儿子的**左**子树进行一次插入

情况1与情况3、情况2与情况4互为镜像问题，前者通过**单旋转**（single rotation）完成平衡，后者通过稍复杂的**双旋转**（double rotation）完成平衡。

### 单旋转

首先考虑情况1，“N-左-左”插入导致不平衡（N两子树高相差2）：“N-左-左”插入后肯定有导致“N-左”树高+1（树高不变 不会不平衡），而且插入前 “N-左”就比“N-右”高1，插入后“N-左”比“N-右”高2。如下图所示，通过右旋 减少“N-左”、增加“N-右” 达到平衡，且可以保证N的高度不变：

![](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/avl_single_rotation_R.svg)

```cpp
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

类似的，考虑镜像问题“情况3”，左旋即可：

![](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/avl_single_rotation_L.svg)

```cpp
template <typename Comparable>
class AvlTree {
 private:
  void _singleRotateWithRightChild(AvlNode*& N) {
    AvlNode* R = N->right;

    N->right = R->left;
    R->left = N;
    N->height = max(_height(N->left), _height(N->right)) + 1;
    R->height = max(_height(R->right), N->height) + 1;

    N = R;
  }
};
```

### 双旋转

接下来考虑情况2，“N-左-右”插入导致不平衡（N两子树高相差2）：“N-左-右”插入后肯定有导致“N-左”树高+1（树高不变 不会不平衡），而且插入前 “N-左”就比“N-右”高1，插入后“N-左”比“N-右”高2。如下图所示，通过可以先对左子树左选，把内部高度差 转换为 边上高度差，再按情况1处理：

![](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/avl_double_rotation_LR.svg)

```cpp
template <typename Comparable>
class AvlTree {
 private:
  void _doubleRotateWithRightChild(AvlNode*& N) {
    _singleRotateWithLeftChild(N->right);
    _singleRotateWithRightChild(N);
  }
};
```

类似的，考虑镜像问题“情况4”，右旋+左旋 即可：

![](https://engineers-cool-1251518258.cos.ap-chengdu.myqcloud.com/avl_double_rotation_RL.svg)

```cpp
template <typename Comparable>
class AvlTree {
 private:
  void _doubleRotateWithLeftChild(AvlNode*& N) {
    _singleRotateWithRightChild(N->left);
    _singleRotateWithLeftChild(N);
  }
};
```

最后，汇总插入函数：

```cpp
template <typename Comparable>
class AvlTree {
 public:
  void Insert(const Comparable& x) { _insert(x, root); }

private:
  // 向子树t插入值x，注意t值可能被修改
  void _insert(const Comparable& x, AvlNode*& t) {
    if (t == nullptr) {
      t = new AvlNode(x, nullptr, nullptr);
    } else if (x < t->element) {
      _insert(x, t->left);
      if (_height(t->left) - _height(t->right) == 2) {
        if (x < t->left->element)
          _singleRotateWithLeftChild(t);
        else
          _doubleRotateWithLeftChild(t);
      }
    } else if (t->element < x) {
      _insert(x, t->right);
      if (_height(t->right) - _height(t->left) == 2) {
        if (t->right->element < x)
          _singleRotateWithRightChild(t);
        else
          _doubleRotateWithRightChild(t);
      }
    } else {
      // 重复了，忽略
    }
    t->height = max(_height(t->left), _height(t->right)) + 1;
  }
};
```

## 删除

最后，如果删除次数很多，懒惰删除不适用，实时删除操作，也会导致树不平衡，此时也需要平衡。可以同样适用 单旋转 与 双旋转 处理高度差的问题，但是这会是的结点N的高度差少1，所以还会需要继续向上递归处理。

首先回顾一搜索二叉树的删除操作：
1. 要删除结点 是 叶子结点：直接删除
2. 要删除结点 不是叶子结点 且 右子树为空：父结点指向被删除结点的左子树
3. 要删除结点 不是叶子结点 且 左子树为空：父结点指向被删除结点的右子树
4. 要删除结点 不是叶子结点 且 双子树均不为空：交换 被删除结点 和 其左子树的最大值/其右子树的最小值，转化为情况1～3

删除过程代码如下：

```cpp
template <typename Comparable>
class AvlTree {
 public:
  void Delete(const Comparable& x) { _delete(x, root); }

private:
  void _delete(const Comparable& x, AvlNode*& t) {
    if (t == nullptr) {
      return;
    } else if (x < t->element) {
      _delete(x, t->left);
      if (_height(t->right) - _height(t->left) == 2) {
        if (_height(t->right->right) > _height(t->right->left))
          _singleRotateWithRightChild(t);
        else
          _doubleRotateWithRightChild(t);
      }
    } else if (t->element < x) {
      _delete(x, t->right);
      if (_height(t->left) - _height(t->right) == 2) {
        if (_height(t->left->left) > _height(t->left->right))
          _singleRotateWithLeftChild(t);
        else
          _doubleRotateWithLeftChild(t);
      }
    } else {
      // 就是删除这个结点
      if (t->left == nullptr && t->right == nullptr) {  // 要删除结点 是 叶子结点
        delete t;
        t = nullptr;

      } else if (t->right == nullptr) {  // 要删除结点 不是叶子结点 且 右子树为空
        AvlNode* L = t->left;
        delete t;
        t = L;  // 父结点指向被删除结点的左子树

      } else if (t->left == nullptr) {  // 要删除结点 不是叶子结点 且 左子树为空
        AvlNode* R = t->right;
        delete t;
        t = R;  // 父结点指向被删除结点的右子树

      } else {  // 要删除结点 不是叶子结点 且 双子树均不为空
        // 交换 被删除结点 和 其左子树的最大值
        Comparable max_v = _findMax(t->left);
        t->element = max_v;
        _delete(max_v, t->left);
      }
    }
    if (t) t->height = max(_height(t->left), _height(t->right)) + 1;
  }
};
```

## 查找

跟普通二叉查找树一样的查找过程：

```cpp
template <typename Comparable>
class AvlTree {
 public:
  bool Find(const Comparable& x) const { return _find(x, root); }

 private:
  bool _find(const Comparable& x, AvlNode* t) const {
    if (t == nullptr) {
      return false;
    } else if (x < t->element) {
      return _find(x, t->left);
    } else if (t->element < x) {
      return _find(x, t->right);
    }
    return true;
  }
};
```

最后附上整体的[可执行代码](https://wandbox.org/permlink/nrLtPzvt3svh45uo)。

## 参考

1. 《数据结构与算法分析 C++描述》