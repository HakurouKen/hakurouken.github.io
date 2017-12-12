---
title: 给非程序员的爬虫教程（八）：Web 中常见的数据格式
tags:
  - python
  - 非程序员的爬虫教程
---

我们通过一个 HTTP 请求拿到数据之后，需要进行的下一步操作就是从我们拿到的“源代码”中提取有效信息。大多数我们拿到的文本信息/二进制信息都需要按照一定的规则进行解析，我们称这种规则为**数据格式**。例如我们之前提到的 HTML，也是一种格式。大多数格式都是通用的，我们可以调用现成的解析库来简化我们的读取逻辑。下面将简单介绍在 Web 中常见的几种格式，以及相应的读取逻辑。

## HTML
前文已经介绍过 HTML 的基本格式，HTML 的基本组成结构是标签(tag)。一个 HTML 标签的结构大致可以表示为：
```
<标签 属性1="值1" 属性2="值2">内容</标签>
```
或
```
<标签 属性1="值1" 属性2="值2" />
```
只要能够定位到某个标签，我们就可以获得标签的指定内容。如果你曾经接触过 CSS，你一定会对选择器这一概念有印象。需要注意的是，CSS 中有很多高级选择器，并不能直接在编程语言的相关库中使用。原因是实现这些高级选择器过于复杂，而且和 CSS 不同，编程语言自身是有处理逻辑的能力的，我们可以将选择器配合一些逻辑使用。下面仅介绍在 CSS 中最基础的、常用的选择器。以下面的示例 HTML 为例：
```html
<div class="container">
  <div class="header">
    <p class="title" id="popup-title" data-title="标题">标题</p>
  </div>
  <div class="body">
    <p>一段文字</p>
  </div>
  <div class="footer">
    <button class="button button-confirm">确定</button>
    <button class="button button-cancel">取消</button>
  </div>
</div>
```

| 选择器 | 示例 | 说明 |
| --- | --- | --- |
| #id | `#popup-title` | 选择 id 为`popup-title`的元素 |
| .class | `.button` | 选择 class 中包含`button`的元素 |
| tag | `button` | 选择`<button>`元素 |
| [attribute] | `[data-title]` | 选择带有`data-title`属性的元素 |
| [attribute="value"] | `[data-title="标题"]` | 选择`data-title`属性值为`标题`的元素 |

上面的各个选择器可以进行任意组合，我们这里只介绍3个组合的规则：

| 连接规则 | 示例 | 说明 |
| 直接连接 | `p.title` | 所有 class 中含有`title` 的`<p>`元素 |
| 空格连接 | `.header p` | 所有有 class 中含有`header`的元素内的`<p>`元素 |
| > 连接 | `.footer>button` | 所有父元素的 class 中含有`footer`的元素 |

这些规则也可以任意组合，例如想要选中“标题”，可以使用：`#popup-title`,`.title` 或`.header p`或`.header p.title` 等等。想要选中"确定"按钮，可以使用`.button.button-confirm` 或者`.button-confirm`等。

## XML

## JSON