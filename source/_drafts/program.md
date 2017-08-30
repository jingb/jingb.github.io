---
title: program
tags:
---

# 查找类
## 二分查找
### 例题
> * [有序数组(0 1 2 4 5 6 7)在某个位置做了旋转，比方(4 5 6 7 0 1 2)，元素不重，找出最小的那个数](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/)
   * 最优是用二分查找
   * [题解](http://www.cnblogs.com/yuzhangcmu/p/4049030.html)
* [二分查找英文wiki](https://en.wikipedia.org/wiki/Binary_search_algorithm)
* 相关术语，前驱(predecessor, next-smallest element)，后继(successor，next-largest element)，秩(rank, the number of smaller elements)
* 适合解决的问题, 范围查询(Rang query)，最近邻搜索

<!-- more -->

### 固定模板
> 用固定模板减少边界值判断出错
```plain
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

### 实现
```plain
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

# 常问问题
## 字符串母串和子串是否匹配
> * [leetcode](https://leetcode.com/problems/implement-strstr/description/)
* 题解
 * 两层for循环，外层是母串，一次挪一个位置，内层循环是子串

```plain
L: for (int offset = 0; offset <= haystack.length() - needle.length(); offset++) {
    for (int i = 0; i < needle.length(); i++) {
    	int index = i + offset;
    	char ch = haystack.charAt(index);
    	char subCh = needle.charAt(i);
    	if (ch != subCh) {
    		continue L;
    	} 
    }
    return offset;
}
return -1;
```
 * KMP算法


## Anagrams(两个字符串间能否通过变换字符的顺序变成相同)
> * [题目](https://www.kancloud.cn/kancloud/data-structure-and-algorithm-notes/72934)
* 建一个int[256]的数组，分别遍历两个字符串，往数组里增减1，最后遍历数组，任意一个数不为0即两个字符串不能通过变换字符顺序变成相同
* ```plain
int letterCount[256] = {0};
for (int i = 0; i != s.size(); ++i) {
    ++letterCount[s[i]];
    --letterCount[t[i]];
}
for (int i = 0; i != t.size(); ++i) {
    if (letterCount[t[i]] != 0) {
        return false;
    }
}
```

### 变种
> 1. 两个字符串无需完全匹配，仅需A串的某个子串和B串是Anagrams即可 **[题目](https://www.kancloud.cn/kancloud/data-structure-and-algorithm-notes/72935)**
2. 按组Anagrams
 {% asset_img 1.png%}
 * [题目](https://www.kancloud.cn/kancloud/data-structure-and-algorithm-notes/72936)
 * 直观反应是两层for循环 O(n^2)，**慢在每个字符串要重复遍历**
 * Map的key是排好序的字符串。每个字符串可以先排序，O(lg K)，然后添加到map中，整体N\*O(lg K)

```plain
Map<String, List<String>> map;
for (String str : strs) {
    String sortStr = str排序;
    if (!map.containsKey(sortStr)) 
        map.put(sortStr, new ArrayList<String>());
    map.get(sortStr).add(s);
}
```


## Atoi实现
考虑如下问题
> * 去掉前导和后导空格(trim)
* 数值的正负号处理
* 数值溢出
```plain
while (str[i] >= '0' && str[i] <= '9') {
    // INT_MAX is 2147483647
    if (base >  INT_MAX / 10 || (base == INT_MAX / 10 && str[i] - '0' > 7)) {
        if (sign == 1) return INT_MAX;
        else return INT_MIN;
    }
    base  = 10 * base + (str[i++] - '0');
}
return base * sign;
```
* 不合法的输入

# 排序类
## 概览
> * 排序术语in place指no extra memory except perhaps for a small function-call stack or a constant number of instance variables，像归并排序需要额外开辟空间
* {% asset_img 2.png %}
* 一般情形下快排是第一选择，如果对stable有要求并且有足够空间，可以选择归并排序

## 归并排序
```plain
public static void mergeSort(int[] array, int low, int high) {
    if (high <= low) return;
    int mid = low + (high - low) / 2;
    mergeSort(array, low, mid);
    mergeSort(array, mid + 1, high);
    merge(array, low, mid, high);
}

private static void merge(int arr[], int left, int middle, int right) {
    int[] temp = new int[arr.length];
    int i = left;     
    int j = middle+1;
    while (i<=middle && j<=right) {
       // 逐个把两个数组里较小的元素存到temp至某个个数组遍历完
    }
    // 把未遍历完的数组剩余元素直接复制到temp数组
    // 把temp复制回原数组
}
```

## 快排
> 定基准——首先随机选择一个元素最为基准
划分区——所有比基准小的元素置于基准左侧，比基准大的元素置于右侧
递归调用——递归地调用此切分过程


# Greedy
> * 描述：在每一步选择中都采取在当前状态下最好或最优（即最有利）的选择，从而希望导致结果是最好或最优的算法
* 适用：在有最优子结构的问题中尤为有效。最优子结构是局部最优解能决定全局最优解。简单地说，问题能够分解成子问题来解决，子问题的最优解能递推到最终问题的最优解
* 动态规划的不同在于它对每个子问题的解决方案都做出选择，**不能回退**。动态规划则会保存以前的运算结果，并根据以前的结果对当前进行选择，有回退功能



