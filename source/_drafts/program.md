---
title: program
tags:
---

# 二分查找
## 例题
> * [有序数组(0 1 2 4 5 6 7)在某个位置做了旋转，比方(4 5 6 7 0 1 2)，元素不重，找出最小的那个数](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/)
   * 最优是用二分查找
   * [题解](http://www.cnblogs.com/yuzhangcmu/p/4049030.html)
* [二分查找英文wiki](https://en.wikipedia.org/wiki/Binary_search_algorithm)
* 相关术语，前驱(predecessor, next-smallest element)，后继(successor，next-largest element)，秩(rank, the number of smaller elements)
* 适合解决的问题, 范围查询(Rang query)，最近邻搜索

<!-- more -->

## 固定模板
> 用固定模板减少边界值判断出错
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

```java
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


# Anagrams(两个字符串间能否通过变换字符的顺序变成相同)
> * [题目](https://www.kancloud.cn/kancloud/data-structure-and-algorithm-notes/72934)
* 建一个int[256]的数组，分别遍历两个字符串，往数组里增减1，最后遍历数组，任意一个数不为0即两个字符串不能通过变换字符顺序变成相同
* ```C++
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

## 变种
> 1. 两个字符串无需完全匹配，仅需A串的某个子串和B串是Anagrams即可 **[题目](https://www.kancloud.cn/kancloud/data-structure-and-algorithm-notes/72935)**
2. 按组Anagrams
 {% asset_img 1.png%}
 * [题目](https://www.kancloud.cn/kancloud/data-structure-and-algorithm-notes/72936)
 * 直观反应是两层for循环 O(n^2)，**慢在每个字符串要重复遍历**
 * Map<String, List<String>> key是排好序的字符串。每个字符串可以先排序，O(lg K)，然后添加到map中，整体N\*O(lg K)
```java
Map<String, List<String>> map;
for (String str : strs) {
  String sortStr = str排序;
  if (!map.containsKey(sortStr)) 
    map.put(sortStr, new ArrayList<String>());
  map.get(sortStr).add(s);
}
```



