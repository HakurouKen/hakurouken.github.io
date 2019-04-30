---
title: Web 前端常见的 CSS 面试基础题
tags:
  - css
  - 面试
---

## 块级元素和行内元素

### 块级元素(block element)

1. 总是在新行上开始（独占一行）
2. height/width/padding/border/margin 可控制
3. 宽度默认是 100%
4. 可以容纳 inline element 和 block element

### 内联元素(inline element)

1. height 无效，一般只能控制 line-height
2. margin/padding 左右生效，上下不生效（值变化，但不影响布局）
3. 宽度由内部元素撑起
4. 只能容纳其余内联元素（或者文本）

### 常见块级元素

- div / p
- h1 - h6
- table
- ul / ol

### 常见内联元素

- a
- img
- span
- b em i strong
- input / textarea

## CSS 盒模型(box model)

CSS 的盒模型是文档布局的基础。每个盒子分为 4 个部分，从内到外依次为： content(实际内容)/padding(内边距)/border(边框)/margin(外边距).

![MDN: CSS 盒模型](<https://mdn.mozillademos.org/files/8685/boxmodel-(3).png>)

默认情况下，我们在 CSS 中通过 width/height 指定的值以及通过 offsetWidth/offsetHeight 拿到的值，都是 content-box 的大小。CSS3 中，允许我们使用 `box-sizing` 来指定盒子的大小的包含范围，它有这样几个可选值：

- `content-box`(默认)：content
- `padding-box`: content + padding
- `border-box`: content + padding + margin

需要注意的是，外边距(margin)不属于盒子内容的一部分，它是盒子之间的分隔（永远是透明的），因此没有 margin-box 这一属性值的存在。

更多参考：

1. [CSS 基础框盒模型介绍 - CSS（层叠样式表） | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Introduction_to_the_CSS_box_model)
2. [CSS 盒模型详解 - 掘金](https://juejin.im/post/59ef72f5f265da4320026f76)

## BFC

## Flex 布局

## CSS 垂直居中
