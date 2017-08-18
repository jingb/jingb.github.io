---
title: program
tags:
---

# 二分查找
## 例题
> * [有序数组(0 1 2 4 5 6 7)在某个位置做了旋转，比方(4 5 6 7 0 1 2)，元素不重，找出最小的那个数](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/)
   * 最优是用二分查找
   * [题解](http://www.cnblogs.com/yuzhangcmu/p/4049030.html)
* [英文wiki](https://en.wikipedia.org/wiki/Binary_search_algorithm)
* 相关术语，前驱(predecessor, next-smallest element)，后继(successor，next-largest element)，秩(rank, the number of smaller elements)
* 适合解决的问题, 范围查询(Rang query)，最近邻搜索

<!-- more -->

## 固定模板
```java
while (l < r - 1) {
    int m = l + (r - l) / 2;

    // means that there is no rotate.
    ... 这里添加各种退出条件，比如找到了目标值等 8

    // left side is sorted.
    if (A[m] > A[l]) {
        l = m;
    } else {
        r = m;
    }
}
```

## 实现
```java
public static int binarySearch(int[] arr, int value) {
    if (arr == null) return -1;
    int lo = 0, high = arr.length - 1;
    while (lo < high - 1) {
    	int mid = (lo + high) / 2;
    	if (arr[mid] < value) {
    		lo = mid + 1;
    	} else if (arr[mid] > value) {
    		high = mid - 1;
    	} else {
    		return mid;
    	}
    }
    return arr[lo] == value ? lo : (arr[high] == value ? high : -1);
}
```

# 字符串母串和子串是否匹配
> * [leetcode](https://leetcode.com/problems/implement-strstr/description/)
* 题解
 * 两层for循环，外层是母串，一次挪一个位置，内层循环是子串
 * KMP算法



