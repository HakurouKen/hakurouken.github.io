---
title: "LeetCode 2: Two Sum"
tags: leetcode
---

## Question
You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Input:** (2 -> 4 -> 3) + (5 -> 6 -> 4)

**Output:** 7 -> 0 -> 8

## 题目
指定两个表示非负整数的**非空**链表。数字以逆序存储在链表中，且每一位和链表的每个节点一一对应。将这两个数字想家，并返回一个链表。

你可以假定两个数字都不以 0 开头（除了数字本身为 0）。

**输入:** (2 -> 4 -> 3) + (5 -> 6 -> 4)

**输出:** 7 -> 0 -> 8

## 分析
对于每一位数字，有`sum = x + y + carry`，其中 carry 为上一位的进位。需要注意的边界情况有：
1. 两个列表长度不同。如`list1 = [0, 1, 2, 3]`, `list2 = [1, 9, 5]`，这时要把 list2 的末尾用0补足。
2. 最后有一个额外进位。如`list1 = [9, 9, 9]`, `list2 = [1]`。最后要额外做判断。
在实际代码中，我们会用到一个“哑节点”（dummy node）来简化链表的操作，这是一个在链表操作中的常用技巧：由于头节点的特异性（不是任何一个节点的“下一个节点”），我们创建一个没有实际用途的哑节点，用它指向实际的头节点，这样就可以统一头节点和其它节点的行为，减少一些额外判断。

时间复杂度：O(max(m, n)) (遍历链表的所有节点需要循环的次数)

空间复杂度：O(max(m, n)) (结果最多为 max(m, n)+1 位)

## 题解
```javascript
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *   this.val = val;
 *   this.next = null;
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var addTwoNumbers = function (l1, l2) {
  var dummy = new ListNode(0),
    cur = dummy,
    value,
    carry = 0;
  while (l1 || l2) {
    value = (l1 ? l1.val : 0) + (l2 ? l2.val : 0) + carry;
    carry = (value / 10) ^ 0;
    cur.next = new ListNode(value % 10);
    cur = cur.next;
    l1 = l1 && l1.next;
    l2 = l2 && l2.next;
  }

  if (carry) {
    cur.next = new ListNode(carry);
  }
  return dummy.next;
};
```
