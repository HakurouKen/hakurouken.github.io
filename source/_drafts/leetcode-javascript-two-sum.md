---
title: "LeetCode 1: Two Sum"
tags: leetcode
---

## Question
Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.

You may assume that each input would have **exactly** one solution, and you may not use the same element twice.

** Example: **
```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

## 题目
给定一个整型数组，求两个能够加起来成为一个特定的目标的数字，返回它们的 **索引**。

你可以假定每个输入中**仅有**一组结果，同时注意，一个元素不能使用两次。

**例如:**
```
给定: nums = [2, 7, 11, 15] ， target = 9
由于: nums[0] + nums[1] = 2 + 7 = 9
返回: [0, 1]
```

## 分析
要寻找这样一对数字，我们最少需要遍历一次数组（复杂度至少为O(n)）。由于需要返回的是索引，在遍历数组的过程中，我们可以把已经遍历的数字的 value:index 缓存存起来，对于 nums[i], 我们可以快速知道 target - nums[i] 是否在已经被遍历的元素内。由于**有且仅有**一组解，我们可以无需考虑有两个相同数字索引不同的情况（这样的数字一定不是一组解中的一个）。

时间复杂度： O(n) (遍历一次数组)

空间复杂度： O(n) (对遍历的所有数字做哈希存储)

## 题解
```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function (nums, target) {
  var i,l,
    value, another,
    m = {};

  for (i = 0, l = nums.length; i < l; i++) {
  	value = nums[i];
  	another = target - value;
  	if (m[another] !== undefined) {
  		return [m[another], i];
  	} else {
  		m[value] = i;
  	}
  }
};
```
