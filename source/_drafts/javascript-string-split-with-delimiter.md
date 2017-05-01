---
title: Javascript 中保留分隔符的 split
tags:
  - javascript
---

## 问题
在 Javascript 中使用正则分隔字符串时，有时会需要保留分隔符。比如我需要在一个帖子的信息中，分割提取所有的表情占位符。如果需求只是简单的字符串替换，这里用一个 replace 即可解决问题。但是后续会对文字和表情做更复杂的处理，这里更加期望的是能够将不同的类型分割开。即我们想要将下列字符串：
```javascript
const str = '表情[哈哈]列表[假表情][嘿嘿]示例'
```
分割为：
```javascript
const result = ['表情', '[哈哈]', '列表[假表情]', '嘿嘿', '示例']
```
这里涉及到一个 split 方法比较少用到的方法，在这里记录一下。

## 解决方案
在 MDN 的 [String.prototype.split()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split)说明中，有下列的描述：
> 当找到一个 seperator 时，separator 会从字符串中被移除，返回存进一个数组当中的子字符串。如果忽略 separator 参数，则返回的数组包含一个元素，该元素是原字符串。如果 separator 是一个空字符串，则 str 将被转换为由字符串中字符组成的一个数组。
>
> 如果 separator 是一个正则表达式，且包含捕获括号（capturing parentheses），则每次匹配到 separator 时，捕获括号匹配的结果将会插入到返回的数组中。然而，不是所有浏览器都支持该特性。

```javascript
const emoji = ['哈哈', '微笑', '嘿嘿'];
const str = '表情[哈哈]列表[假表情][嘿嘿]示例'
const emojiRegex = new RegExp(`(\\[(?:${emoji.join('|')})\\])`)
const result = str.split(emojiRegex)
```

## `String.prototype.split`相关


## 参考资料
