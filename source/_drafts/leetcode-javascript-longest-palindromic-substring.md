---
title: "LeetCode 5: Longest Palindromic Substring"
tags: leetcode
---

## Question
Given a string `s`, find the longest palindromic substring in `s`. You may assume that the maximum length of `s` is 1000.

**Example:**
```
Input: "babad"

Output: "bab"

Note: "aba" is also a valid answer.
```

**Example:**
```
Input: "cbbd"

Output: "bb"
```

## 题目
给定一个字符串`s`, 找到`s`的最长回文子串。你可以假定`s`的最大长度为1000。

**示例：**
```
输入: "babad"

输出: "bab"

注: "aba" 也是一个合法的答案。
```

**示例：**
```
输入： "cbbd"

输出："bb"
```

## 分析
这里提供一种动态规划的解法，我们定义一个函数，以 javascript 的语法描述如下：
```javascript
const d = function (i, j) {
  return isPalindromic(s.slice(i, j))
}
```
则有下列等价关系，即 DP 的**初始状态**：
```javascript
d(i, i) === true
d(i, i+1) === (s[i] === s[i+1])
```
同时又有**状态转移方程**
```javascript
d(i, j) === d(i+1, j-1) && (s[i] === s[j])
```

空间复杂度：O(n^2)

时间复杂度：O(n^2)

## 题解
