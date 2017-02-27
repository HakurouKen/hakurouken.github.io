---
title: "Leetcode 4: Median of Two Sorted Arrays"
tags: leetcode
---

## Question
There are two sorted arrays `nums1` and `nums2` of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

**Example 1:**
```
nums1 = [1, 3]
nums2 = [2]

The median is 2.0
```

**Example 2:**
```
nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5
```

## 题目
给定两个长度分别为 m 和 n 的已排序数组`nums1`和`nums2`。

找出这两个数组共同的中位数。总得事件复杂度应该为 O(log(m+n))。

**示例1：**
```
nums1 = [1, 3]
nums2 = [2]

中位数为 2.0
```

**示例2:**
```
nums1 = [1, 2]
nums2 = [3, 4]

中位数为 (2 + 3)/2 = 2.5
```
## 分析
时间复杂度为 O(log(m+n)) 已经提示了我们要采用分治算法。中位数的定义比较复杂（需要分总个数是奇数还是偶数讨论），我们因此先抽象出一个“找到两个排序数列中第 k 大的数字”的函数，将函数简化如下：
```javascript
function findMedianSortedArrays(nums1, nums2) {
  var l1 = nums1.length,
  l2 = nums2.length,
  len = l1 + l2;
  if( len % 2 === 0 ){
    return ( findKthElem(nums1, 0, l1 - 1 , nums2, 0, l2 - 1, len/2 - 1) + findKthElem(nums1, 0, l1 - 1, nums2, 0, l2 - 1, len/2) ) / 2;
  } else {
    return findKthElem(nums1, 0, l1 - 1, nums2, 0, l2 - 1, (len-1)/2 );
  }
}

function findKthElem(nums1, start1, end1, nums2, start2, end2, k) {
  //@TODO: return the kth element.
}
```

我们剩下只需考虑这个“找到第 k 大的数字”的函数实现即可。
## 题解
