---
title: 给非程序员的爬虫教程（十一）：使用 Python 解析 HTML
tags:
  - python
  - 非程序员的爬虫教程
---

我们通过模拟 HTTP 请求拿到请求之后，下一步就是对拿到的内容进行解析。对于 JSON 格式，我们一般用内置的`json`库即可完美处理；XML 也可以用 Python 内置的`xml.etree.ElementTree`模块来读取；同样对于 HTML，Python 也提供了`html.parse`模块来处理。

不过，`html.parse`使用起来并不是很方便，我们需要自己做很多额外工作。这里介绍一个第三方库`PyQuery`，它可以让我们使用 CSS 选择器方便的选择指定的元素，并提供了一系列简单的 API 来获取指定节点的信息。

## PyQuery

## BeautifulSoup

## 扩展阅读