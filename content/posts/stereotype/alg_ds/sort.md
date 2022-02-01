---
title: "排序"
date: 2022-02-02T21:33:23+08:00
draft: false
categories: [八股文,数据结构与算法]
---

## 冒泡排序

时间复杂度O(n^2)

1. 每次从尾到头（已排序列的尾部）迭代遍历，把相邻数字的更小（大）的元素放前面
2. 重复1步，直到所有元素均排序完毕。

## 选择排序

时间复杂度O(n^2)

1. 首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置（与之交换位置即可）。
2. 再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾（同1）。
3. 重复第2步，直到所有元素均排序完毕。

## 插入排序

插入排序由N-1此插入组成，对于第i趟，把下标i的数字插入到下标0~i的合适位置。时间复杂度O(n^2)。

```cpp
template <typename Comparable>
void insertionSort(std::vector<Comparable>& v) {
  int j;
  for (int p = 1; p < v.size(); p++) {
    Comparable tmp = v[p];
    for (j = p; j > 0 && tmp < v[j - 1]; j--) {
      v[j] = v[j - 1]; 
    }   
    v[j] = tmp;
  }
}
```

可能的STL实现：

```cpp
template <typename Iterator, typename Comparator, typename Object>
void _insertionSort(const Iterator& begin, const Iterator& end, Comparator lessThan, const Object& obj) {
  Iterator j;
  for (Iterator p = begin + 1; p != end; ++p) {
    Object tmp = *p; 
    for (j = p; j != begin && lessThan(tmp, *(j - 1)); --j) {
      *j = *(j - 1); 
    }   
    *j = tmp;
  }
}

template <typename Iterator, typename Comparator>
void insertionSort(const Iterator& begin, const Iterator& end, Comparator lessThan) {
  if (begin != end) _insertionSort(begin, end, lessThan, *begin);
}

template <typename Iterator, typename Object>
void _insertionSortHelp(const Iterator& begin, const Iterator& end, const Object& obj) {
  insertionSort(begin, end, less<Object>());
}

template <typename Iterator>
void insertionSort(const Iterator& begin, const Iterator& end) {
  if (begin != end) _insertionSortHelp(begin, end, *begin);
}
```

## 谢尔排序

通过多趟比较相距一定间隔的蒜素来工作：各趟比较所用的间隔随着算法的进行而减小，直到只比较相邻元素的最后一趟为止。谢尔也叫**缩减增量排序**。

谢尔排序使用一个序列 h1,h2,...hi，叫做**增量序列**。只要`h1=1`，任何增量序列都可以。不过，有些增量序列比另外一些增量序列更好。

使用谢尔增量的排序，时间复杂度O(n^2)。

```cpp
template <typename Comparable>
void shellSort(std::vector<Comparable>& v) {
  for (int gap = v.size() / 2; gap > 0; gap /= 2)
    for (int p = gap; p < v.size(); ++p) {
      Comparable tmp = v[p];
      int j = p;
      for (; j >= gap && tmp < v[j - gap]; j -= gap) {
        v[j] = v[j - gap];
      }   
      v[j] = tmp;
    }   
}
```

## 堆排序

构建大顶堆，然后逐个取走堆顶（放到数组最后）。时间复杂度O(nlogn)。

```cpp
inline int _leftChild(int p) { return 2 * (p + 1) - 1; }

template <typename Comparable>
void _percDown(std::vector<Comparable>& v, int p, int n) {
  int child;

  Comparable tmp = v[p];
  for (; _leftChild(p) < n; p = child) {
    child = _leftChild(p);
    // 找大的子
    if (child != n - 1 && v[child] < v[child + 1]) child++;
    if (tmp < v[child])  // 比大的子小，则该子上升，父继续下滤
      v[p] = v[child];
    else  // 比大的子大，子本身已合法，则当前合法，保持不变
      break;
  }
  v[p] = tmp;
}

template <typename Comparable>
void heapSort(std::vector<Comparable>& v) {
  // buildHeap
  // 从第一个不是叶子结点的结点开始
  for (int i = (v.size() - 1) / 2; i >= 0; --i) {
    _percDown(v, i, v.size());
  }

  // deleteMax
  for (int j = v.size() - 1; j >= 0; --j) {
    swap(v[0], v[j]);
    _percDown(v, 0, j); 
  }
}
```

## 归并排序

归并（`_merge`）是合并两个有序的数组，归并排序就是递归的分解&&合并数组。时间复杂度O(nlogn)。

```cpp
template <typename Comparable>
void _merge(std::vector<Comparable>& v, std::vector<Comparable>& tmp,
    int left_bg, int right_bg, int right_ed) {
  int left_ed = right_bg - 1;
  int num_cnt = right_ed - left_bg + 1;
  int tmp_p = left_bg;

  while (left_bg <= left_ed && right_bg <= right_ed) {
    if (v[left_bg] < v[right_bg])
      tmp[tmp_p++] = v[left_bg++];
    else
      tmp[tmp_p++] = v[right_bg++];
  }

  while (left_bg <= left_ed) {
    tmp[tmp_p++] = v[left_bg++];
  }

  while (right_bg <= right_ed) {
    tmp[tmp_p++] = v[right_bg++];
  }

  for (int p = right_ed; num_cnt--; --p) v[p] = tmp[p];
}

template <typename Comparable>
void _mergeSort(std::vector<Comparable>& v, std::vector<Comparable>& tmp, int left, int right) {
  if (left == right) return;

  int mid = left + (right - left) / 2;
  _mergeSort(v, tmp, left, mid);
  _mergeSort(v, tmp, mid + 1, right);
  _merge(v, tmp, left, mid + 1, right);
}

template <typename Comparable>
void mergeSort(std::vector<Comparable>& v) {
  std::vector<Comparable> tmp_v(v.size());
  _mergeSort(v, tmp_v, 0, v.size() - 1); 
}
```

## 快速排序

简单的，递归时选第一个元素为锚点，时间复杂度最坏O(n^2)：

```cpp
template <typename Comparable>
void _quickSort(std::vector<Comparable>& v, int left, int right) {
  if (left >= right) return;
  // 选第一个元素为锚点
  Comparable pivot = v[left];

  int l = left;
  int r = right;
  while (l < r) {
    while (r > l && v[r] > pivot) r--;
    if (l < r) v[l++] = v[r];
    while (r > l && v[l] < pivot) l++;
    if (l < r) v[r--] = v[l];
  }
  v[l] = pivot;
  _quickSort(v, left, l - 1); 
  _quickSort(v, l + 1, right);
}

template <typename Comparable>
void quickSort(std::vector<Comparable>& v) {
  _quickSort(v, 0, v.size() - 1); 
}
```

如果输入参数本身是有序的，那么这个算法的递归层次等于数组大小，时间复杂度达到最差。

一个可能的思路是，随机取这段`_quickSort`的一个数字作为锚点，但是随机数的消耗不小。另外一个思路是**三数中值分割法**：最好的锚点之是第`N/2`个大的数，但是未排序数组不好得出，一个简化的方法是取三个数字，取其中间的未锚点：

```cpp
template <typename Comparable>
void insertSort(std::vector<Comparable>& v, int left, int right) {
  for (int i = left + 1; i <= right; i++) {
    Comparable tmp = v[i];

    int j = i;
    for (; j > left && tmp < v[j - 1]; --j) v[j] = v[j - 1]; 
    v[j] = tmp;
  }
}

template <typename Comparable>
const Comparable& _median3(std::vector<Comparable>& v, int left, int right) {
  int mid = left + (right - left) / 2;
  if (v[mid] < v[left]) swap(v[mid], v[left]);
  if (v[right] < v[left]) swap(v[right], v[left]);
  if (v[right] < v[mid]) swap(v[right], v[mid]);
  // 已中间值为锚点，v[left]、v[right]已经处于合适的位置

  swap(v[mid], v[left + 1]);  // 先放这里，相当于有个空位
  return v[left + 1]; 
}

template <typename Comparable>
void _quickSort(std::vector<Comparable>& v, int left, int right) {
  if (left >= right) return;

  // 不够三个值了，换插入排序
  if (right - left < 3) {
    insertSort(v, left, right);
    return;
  }

  Comparable pivot = _median3(v, left, right);

  int l = left + 1;
  int r = right - 1;
  while (l < r) {
    while (r > l && v[r] > pivot) r--;
    if (l < r) v[l++] = v[r];
    while (r > l && v[l] < pivot) l++;
    if (l < r) v[r--] = v[l];
  }
  v[l] = pivot;
  _quickSort(v, left, l - 1); 
  _quickSort(v, l + 1, right);
}

template <typename Comparable>
void quickSort(std::vector<Comparable>& v) {
  _quickSort(v, 0, v.size() - 1); 
}
```

## 间接排序

这里考虑的问题主要是，以上排序过程中，有大量的复制操作，如果复制操作消耗很大，那么需要引入指针数组，对其排序后，最后只按排序后的顺序挪动一次对象。

```cpp
template <typename Comparable>
class Pointer {
 public:
  Pointer(Comparable* rhs = nullptr) : pointee(rhs) {}

  bool operator<(const Pointer& rhs) const { return *pointee < *rhs.pointee; }

  operator Comparable*() const { return pointee; }

 private:
  Comparable* pointee;
};

template <typename Comparable>
void largeObjectSort(std::vector<Comparable>& v) {
  std::vector<Pointer<Comparable>> p(v.size());
  for (int i = 0; i < v.size(); ++i) p[i] = &v[i];

  quickSort(p);

  for (int i = 0; i < v.size(); ++i)
    if (p[i] != &v[i]) {
      Comparable tmp = v[i];
      int j = i, nextj;
      for (; p[j] != &v[i]; j = nextj) {
        nextj = p[j] - &v[0];
        v[j] = *p[j];
        p[j] = &v[j];
      }   
      v[j] = tmp;
      p[j] = &v[j];
    }   
}
```

注意：`Pointer`只重载了`operator<`，没有重载`operator>`，这要求`quickSort`的实现是能用`<`比较大小！因为上面实现的`quickSort`用了到`>`，所以我在测试时总是不正确...



## 计数排序

如果序列中值的值域范围比较小，可以使用计数排序排序，即统计每个值出现的次数，然后展开成序列。

## 桶排序

先把序列hash到K个桶中，每个桶单独排序，最后再归并即可。

## 基数排序

基数排序（radix sort）属于“分配式排序”（distribution sort），是一种非比较型整数排序算法，其原理是将 待排key 按段切割成不同的部分，然后按每个部分分别比较。基数排序的方式可以采用LSD（Least significant digital）或MSD（Most significant digital），LSD的排序方式由键值的最右边开始，而MSD则相反，由键值的最左边开始。

正整数序列的LSD的过程例如：将所有待比较数值统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后, 数列就变成一个有序序列。

## 外部排序

如果输入数据过大，以至于不能全部载入内容，这时就需要外部排序。基本的外部排序算法使用归并排序中的合并算法。基本思路是：
1. 分段排序
    1. 分段载入内存能处理的最大长度，排序后写入磁盘
2. 合并
    1. 给每两段有序序列做合并，合并成2被大小的有序序列，边合并边写磁盘
    2. 重复合并过程，直至合并成一个序列
