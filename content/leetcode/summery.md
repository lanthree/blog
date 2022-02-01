---
title: "总结"
date: 2020-07-03T08:13:53+08:00
draft: false
categories: []
---

## 随便找的一个面试题出现次数

+ [206. 反转链表](https://engineers.cool/#/pages/LeetCode/LeetCode/206)
+ [146. LRU缓存机制](https://engineers.cool/#/pages/LeetCode/LeetCode/146)
+ [8. 字符串转换整数（atoi）](https://engineers.cool/#/pages/LeetCode/LeetCode/8)
+ [153. 寻找旋转排序数组中的最小值](https://engineers.cool/#/pages/LeetCode/LeetCode/153)
+ [3. 无重复自负的最长子串](https://engineers.cool/#/pages/LeetCode/LeetCode/2)
+ [剑指Offer 54. 二叉搜索树的第k大结点](https://engineers.cool/#/pages/LeetCode/%E5%89%91%E6%8C%87Offer/jz_54)
+ [300. 最长上升子序列](https://engineers.cool/#/pages/LeetCode/LeetCode/300)
+ [2. 两数相加](https://engineers.cool/#/pages/LeetCode/LeetCode/2)
+ [470. 用Rand7()实现Rand10()](https://engineers.cool/#/pages/LeetCode/LeetCode/470)
+ [112. 路径总和](https://engineers.cool/#/pages/LeetCode/LeetCode/112)

## 旋转数据查找

## 二分查找框架

```cpp
int binary_search(int[] nums, int target) {
    int left = 0, right = nums.length - 1; 
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1; 
        } else if(nums[mid] == target) {
            // 直接返回
            return mid;
        }
    }
    // 直接返回
    return -1;
}

int left_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 别返回，锁定左侧边界
            right = mid - 1;
        }
    }
    // 最后要检查 left 越界的情况
    if (left >= nums.length || nums[left] != target)
        return -1;
    return left;
}


int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 别返回，锁定右侧边界
            left = mid + 1;
        }
    }
    // 最后要检查 right 越界的情况
    if (right < 0 || nums[right] != target)
        return -1;
    return right;
}
```
