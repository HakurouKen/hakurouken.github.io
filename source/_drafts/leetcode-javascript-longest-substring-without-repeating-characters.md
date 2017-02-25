---
title: "LeetCode 3: Longest Substring Without Repeating Characters"
tags: leetcode
---

## Question
Given a string, find the length of the longest substring without repeating characters.

Examples:

Given "abcabcbb", the answer is "abc", which the length is 3.

Given "bbbbb", the answer is "b", with the length of 1.

Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a substring, "pwke" is a subsequence and not a substring.

## 题目
给定一个字符串，找出其中最长的没有重复字符的子串的长度。

例如：

对于"abcabcbb"，最长的子串是"abc", 长度是 3。

对于"bbbbb"，最长的子串是"b"，长度是 1。

对于"pwwwkew"，最长的子串是"wke"，长度是 3。需要注意的是答案必须是子串，"pwke"是一个子序列但不是一个子字符串

## 分析
为了解决这个子串问题，我们使用滑动窗口的算法。具体操作如下：

1. 初始化窗口 ，窗口的起点和终点均为`s[0]`，初始化最大长度`result = 0`
2. 滑动窗口右端，直至出现重复字符，记此时的长度为 l，则`result = Math.max(l, result)`
3. 收回窗口左端，直至没有重复字符
4. 重复步骤 2 和 3，直至窗口的终点抵达最后一个元素
5. 返回`result`

由于我们不可能获知还未遍历的数组信息，因此步骤 2 无法进行优化，只能逐个遍历。而步骤 3 是对已遍历过的数组进行操作，我们可以用缓存机制来加快它的执行。具体的说，我们可以用一个哈希表，存储我们所有遍历的字符的索引，当收回窗口时，只需从哈希表中直接取出对应索引即可，无需逐个字符移动。

时间复杂度：O(n) （遍历一次数组）

空间复杂度：O(1) （由于一共有26个字母，哈希表最大长度也只有26，为常数空间）

## 题解
```javascript
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function(s) {
  var start = 0, end = 0,
    n = s.length,
    result = 0,
    map = {},
    c;

  while (end < n) {
    c = s.charAt(end);
    if (map[c] !== undefined) {
      start = Math.max(map.get(c), start);
    }
    result = Math.max(result, end - start + 1);
    map.set(c, end+1);
  }

  result = Math.max(result, n - start);
  return result;
};
```
