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

## 格式化上下文

FC(Formatting Context，格式化上下文)，是指 Web 页面可视化渲染的一部分，它决定了其子元素如何定位以及元素之间的相互关系和作用。

格式化上下文分 BFC（Block Formatting Context）和 IFC（Inline Formatting Context）两种。CSS3 中又增加了 FFC（Flex Formatting Context）和 GFC（Grid Formatting Context）。

### BFC

块级格式化上下文是一个独立的块级渲染区域，用一系列渲染规则约束块级盒子的布局，这部分区域的布局和外部无关。

#### BFC 的约束规则

1. 内部的盒子在垂直方向上依次放置（每个占一行）
2. 盒子间距由 margin 决定，且同一个 BFC 的两个相邻盒子的 margin 会合并（margin collapse）
3. 每个元素的左外边距与包含块的左边界相接触（从左到右），即使浮动元素也是如此（BFC 中的子元素不会超出它的包含块）。
4. BFC 的区域不会和 `float` 重叠。
5. 计算 BFC 的高度时，浮动元素也参与计算
6. BFC 是格力的独立容器，内外的元素互不影响

#### BFC 的生成条件

下列方式都会创建 BFC：

1. 根元素
2. 浮动元素(`float` 不为 `none`)
3. 绝对定位(`position`是 `absolute` 或 `fixed`)
4. `overflow` 不为 `visible`
5. `display` 的值为 `inline-block`，`table-cell`或 `table-caption` 等

#### BFC 的实际应用

1. 清除浮动，使浮动的内容和周围的内容等高
2. 创建新的 BFC，避免两个相邻的 div 的 margin collapse

### IFC

IFC，即行内格式化上下文。IFC 的线框(`line box`)高度由内部的行内元素中最高的实际高度计算而来，竖直方向的 margin/padding 无实际影响。

### IFC 的排版规则

1. IFC 的区域一般就是整个线框的区域，除非有 `float` 元素的存在。`float` 元素会卫浴 IFC 和线框之间，使得线框宽度缩短。
2. IFC 内不能有块级元素。如果一个 IFC 中，插入了一个块级元素（例如 `<p>some<div></div>text</p>` ），则会产生两个匿名的 IFC，每个 IFC 对外表现为块级元素，和插入的块级元素垂直排列。

### IFC 的应用实例

1. 水平居中：当一个块要在水平居中时，设置其为`inline-block` 则会在外层产生 IFC，通过`text-align:center` 则可以使其水平居中。
2. 垂直居中：创建一个 IFC，用其中一个元素撑开父元素的高度，然后设置其 `vertical-align:middle`，其他行内元素则可以在此父元素下垂直居中。

### FFC

从字面意思就可以看出，FFC 由 `display: flex` 或 `display: inline-flex` 的元素创建，其内部的容器即为一个 FFC，按照 flexbox 的规则进行排版。

### GFC

和 FFC 类似，GFC 由 `disable: grid` 的元素创建，容器内部是一个独立的渲染区域，满足网格布局。

### 参考资料

1. [CSS 的 BFC 详解 - shadow_zed 的博客 - CSDN 博客](https://blog.csdn.net/shadow_zed/article/details/72811975)
2. [块格式化上下文 - Web 开发者指南 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)
3. [css 布局的各种 FC 简单介绍：BFC，IFC，GFC，FFC - 小楼兰的梦想 - SegmentFault 思否](https://segmentfault.com/a/1190000014886753#articleHeader11)
4. [CSS 布局 - 学习 Web 开发 | MDN](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/CSS_layout)
5. [Understanding Block Formatting Contexts in CSS — SitePoint](https://www.sitepoint.com/understanding-block-formatting-contexts-in-css/)

## Flex 布局

## CSS 垂直居中
