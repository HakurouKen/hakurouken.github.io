---
title: 给非程序员的爬虫教程（九）：Web 中常见的数据格式
tags:
  - python
  - 非程序员的爬虫教程
date: 2017-12-31 21:28:32
---
我们通过一个 HTTP 请求拿到数据之后，需要进行的下一步操作就是从我们拿到的“源代码”中提取有效信息。大多数我们拿到的文本信息/二进制信息都需要按照一定的规则进行解析，我们称这种规则为**数据格式**。例如我们之前提到的 HTML，也是一种格式。大多数格式都是通用的，我们可以调用现成的解析库来简化我们的读取逻辑。下面将简单介绍在 Web 中常见的几种格式，以及相应的读取逻辑。
<!-- more -->
## HTML
前文已经介绍过 HTML 的基本格式，HTML 的基本组成结构是标签(tag)。一个 HTML 标签的结构大致可以表示为：
```
<标签 属性1="值1" 属性2="值2">内容</标签>
```
或
```
<标签 属性1="值1" 属性2="值2" />
```
只要能够定位到某个标签，我们就可以获得标签的指定内容。如果你曾经接触过 CSS，你一定会对选择器这一概念有印象。需要注意的是，CSS 中有很多高级选择器，并不能直接在编程语言的相关库中使用。原因是实现这些高级选择器过于复杂，而且和 CSS 不同，编程语言自身是有处理逻辑的能力的，我们可以将选择器配合一些逻辑使用。下面仅介绍在 CSS 中最基础的、常用的选择器，这些选择器，一般相关的解析库都会做支持。下文的示例，全部基于下列的 HTML 片段：
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
XML(Xtensible Markup Language) ，中文翻译为可扩展标记语言，是一个和 HTML 非常类似的格式。不同的是，HTML 中允许有“自闭合标签”（例如 img），而且 HTML 已经有预设语义（例如 p 标签表示段落、b 标签表示加粗等等）。我们可以把 HTML 理解为一个不严格的 XML 的子集，比起 HTML 来，XML 不仅限于描述 Web，是一种更加通用的数据结构。例如，我们想要描述一个学生的数据，可能会使用下列 XML：
```
<student>
  <name>leroy</name>
  <age>18</age>
  <score>100</score>
</student>
```
类比 Python 中内置的数据结构，我们可能会用`dict`这样表示：
```python
{
  "name": "leroy",
  "age": 10,
  "score": 100
}
```
XML 一般使用 XPath 来选定指定的元素。但是由于 XML 的格式书写相对复杂，而且较为“重量”，近些年在 Web 中应用已经越来越少。除了 RSS 源之外，已经比较少见到应用 XML 的场景了。

## JSON 
JSON(JavaScript Object Notation) 是基于 javascript 的对象字面量来定义的一种数据格式。它的优点是轻量、易读写（无论是对于人类还是对于机器），因此现在非常流行，在 Web 开发中，基本取代了 XML 的位置。虽然它的名字中带有 Javascript, 但是它已经是一种通用的数据格式，绝大多数语言都有原生的 JSON 读取/解析的库。JSON 的格式也非常直观，我们同样以上文的 student 为例，JSON 格式表示大概是：
```json
{
  "name": "leroy",
  "age": 10,
  "score": 100
}
```
这里为了可读性，我们人为加上了空格和换行。在实际传输过程中，我们的 JSON 文件一般都不带换行和空格。可以看到，它和 Python 的 dict 极其类似。我们可以用 Python 的 json 库的`loads`方法直接将其转成 python 内置的格式：
```python
import json
data = json.loads('{"name":"leroy","age":10,"score":100}')
print(data)
```
类似的，我们可以用`dumps`方法，将一个 Python 的内置对象转为 JSON：
```python
import json
data = {
  "name": "leroy",
  "age": 10,
  "score": 100
}
json_str = json.dumps(data)
print(json_str)
```

## 扩展阅读
1. [w3cschool CSS 选择器](http://www.w3school.com.cn/cssref/css_selectors.asp)
2. [维基百科：XML](https://zh.wikipedia.org/wiki/XML)
3. [w3cshool XML 教程](https://www.w3cschool.cn/xml/)
4. [w3cshool XPath 教程](https://www.w3cschool.cn/xpath/)
5. [JSON 官网(中文)](http://json.org/json-zh.html)
6. [Python 的 JSON 库官方文档](https://docs.python.org/3.6/library/json.html)