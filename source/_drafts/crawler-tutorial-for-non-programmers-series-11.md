---
title: 给非程序员的爬虫教程（十一）：使用 Python 解析 HTML
tags:
  - python
  - 非程序员的爬虫教程
---

我们通过模拟 HTTP 请求拿到请求之后，下一步就是对拿到的内容进行解析。对于 JSON 格式，我们一般用内置的`json`库即可完美处理；XML 也可以用 Python 内置的`xml.etree.ElementTree`模块来读取；同样对于 HTML，Python 也提供了`html.parse`模块来处理。

不过，`html.parse`使用起来并不是很方便，我们需要自己做很多额外工作。这里介绍一个第三方库`PyQuery`，它可以让我们使用 CSS 选择器方便的选择指定的元素，并提供了一系列简单的 API 来获取指定节点的信息。

## PyQuery
PyQuery 的命名来源于一个著名的 javascript 库`jQuery`。jQuery 以其简介的 API，方便的 DOM 操作和优秀的跨浏览器兼容性，曾一度是最受欢迎的 javascript 库。PyQuery 的 DOM 操作相关的 API，基本也都参照类似 jQuery 的方式来设计。我们来看一个 PyQuery 使用的简单例子：
```python
from pyquery import PyQuery
html = '''
<ul id="fruits">
  <li class="apple">Apple</li>
  <li class="orange fruit">Orange</li>
  <li class="pear"><b>Pear</b></li>
</ul>
'''
doc = PyQuery(html)

# ’fruits‘
doc.attr('id')

# ’Apple'
doc('.apple').text()

# 'orange fruit'
doc('li.orange').attr('class')

# 'Pear'
doc('li.pear').text()

# '<b>Pear</b>'
doc('li.pear').html()

fruits = []
for item in doc('li').items():
  fruits.append(item.text())

# `Apple,Orange,Pear`
','.join(fruits)
```
我们全文中，几乎所有的代码都是在进行`pyquery.PyQuery`的对象的相关操作(即`doc`这个变量)。这个对象有些特殊：它既可以像函数一样调用（例如`doc('.apple')`），也可以像一般的示例一样，调用对应的方法（例如`doc.attr('id')`）。这个是因为 PyQuery 的类在开发时，为了使 API 更加简洁，用了一些特殊的技巧(重写了`__call__`方法)，使得这个对象可以像函数一样调用。这里形如`pyquery(selector)`的调用，返回的是一个满足 selector 选择器的、全新的 PyQuery 的对象，我们可以方便的在对象上进行“链式操作”。

上文中列举了 PyQuery 对象几个常见的 API：
* `.attr(attribute)`: 取当前 HTML 上，的名为 "attribute" 的属性。
* `.text()`: 取当前 HTML 节点内部的文本内容（不包括任何 HTML 标签）。
* `.html()`: 取当前 HTML 节点内的 HTML 内容。
* `.items()`: 取 PyQuery 对象中，所有的 HTML 根节点，包装成独立的 PyQuery 对象并返回。

关于`.items()`的用法，这里额外补充一下说明：以上文的`doc('li')`为例，满足`"li"`这个选择器的有三个元素，我们想分别把每个元素取出来，执行一些特定的操作（比如 Demo 中的取文本）。items 方法就是我们这一思路的封装，我们用 for 循环遍历，变量`item`得到的值依次等价于下列结果：
```python
# 第一次
PyQuery('<li class="apple">Apple</li>')
# 第二次
PyQuery('<li class="orange fruit">Orange</li>')
# 第三次
PyQuery('<li class="pear"><b>Pear</b></li>')
```

这里列出的 API 仅仅是 PyQuery 用法的冰山一角，更多相关的 API 使用，我们可以参照[官方文档](https://pythonhosted.org/pyquery/api.html)进行学习。

## 其它第三方库选择
1. [Beautifulsoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)：一个同样强大的 HTML/XML 解析工具。我们这里没有将其作为首选，是因为它的执行速度比 PyQuery 低一些，同时 API 比较“多样”（我们可以用很多中不同的方式，来完成同一个效果），不太适合新手上手。不过这些并不能掩盖它是一个优秀的库的事实，感兴趣的读者可以自行研究。
2. [lxml](http://lxml.de/)：这是一个相对“底层”的库，pyquery 内部就是使用的它来进行的文档解析。它拥有最高的解析效率，但同时使用起来也稍微复杂一些。如果对效率极其看重，可以尝试使用它来解析 HTML、XML。

## 扩展阅读
1. [Python html.parser 的官方文档](https://docs.python.org/3/library/html.parser.html)
2. [Python xml.etree.elementtree 的官方文档](https://docs.python.org/3/library/xml.etree.elementtree.html)
3. [PyQuery 的官方文档](https://pythonhosted.org/pyquery/api.html)
4. [BeautifulSoup 官方文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)
5. [lxml 官方文档](http://lxml.de/)