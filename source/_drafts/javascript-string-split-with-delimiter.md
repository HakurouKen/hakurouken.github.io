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
需要注意的是，所有的捕获分组的值都会被插入到返回的数组中，因此我们的内层分组需要采用非捕获分组。

## `String.prototype.split`相关标准
在 [ECMAScript 标准](http://www.ecma-international.org/ecma-262/6.0/#sec-string.prototype.split)中，我们可以看到，在 split 使用正则表达式分割字符串时，实际使用的方法是 `GetMethod(separator, @@split)`，即[RegExp.prototype[@@split]()](http://www.ecma-international.org/ecma-262/6.0/#sec-regexp.prototype-@@split)。在标准中，对这一特性专门做了说明：
> 如果字符串是空字符串，（split 的）结果取决于正则表达式是否可以匹配空字符串。如果能，则返回空数组，否则返回包含一个空字符串的数组。
>
> 如果正则表达式包含匹配分组，每个匹配到的结果（包括`undefined`）都会被拼接到输出数组中。例如：
>
> `/<(\/)?([^<>]+)>/[Symbol.split]("A<B>bold</B>and<CODE>coded</CODE>")`
>
> 转化为:
>
> `["A",undefined,"B","bold","/","B","and",undefined,"CODE","coded","/","CODE",""]`

## 参考资料
1. [MDN String.prototype.split()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split)
2. [MDN RegExp.prototype\[@@split\]\(\)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/@@split)
3. [ECMA-262 String.prototype.split](http://www.ecma-international.org/ecma-262/6.0/#sec-string.prototype.split)
4. [ECMA-262 RegExp.prototype \[ @@split \]](http://www.ecma-international.org/ecma-262/6.0/#sec-regexp.prototype-@@split)
