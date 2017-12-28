---
title: 给非程序员的爬虫教程（十三）：实例：好奇心日报长文章列表
tags:
  - python
  - 非程序员的爬虫教程
---

## 目标

入口 URL：http://www.qdaily.com/tags/1068.html

[图片]

批量获取好奇心日报长文的标题、正文链接、更新时间等基础信息。

## 分析

[图片]

直接查看源代码，我们很容易看到，我们需要的数据都清楚的写在 HTML 中。我们需要的就是伪造 HTTP 请求获取页面，然后使用 PyQuery 取出需要的值/属性即可。我们可以很快的写出下列代码片：
```python
import requests
from pyquery import PyQuery

response = requests.get('http://www.qdaily.com/tags/1068.html')
doc = PyQuery(response.content)

infos = []

for link in doc('.packery-item>a').items():
  infos.append({
    'url': 'http://www.qdaily.com{}'.format(link.attr('href')),
    'title': link('.title').text(),
    'date': link('.smart-date').attr('data-origindate')
  })
```

但是，我们这里得到的只有第一页的数据，第二页之后的是后续滚动到页面底部才使用 ajax 加载的。通过筛选 XHR 请求，我们也可以看到对应的请求如下：

[图片]

通过观察，我们不难发现如下的规律：上一页返回的数据中的`last_key`字段，就是下一页的请求的 `.json` 的文件名。这是在这种“无限滚动”的场景下，十分常见的一种翻页模式：它的特点是，在返回一页数据的同时，返回一个特征值，Web 可以利用这个特征值来安全的获取当前数据的下一页数据。有些人形象的将这个参数称之为“接力参数”。

利用这个`last_key`，我们可以通过第 2 页的数据来获取第 3 页的数据，再通过第 3 页的数据通过第 4 页的数据，依次类推。同时我们不难想到，第二页的`last_key`的相关数据，一定位于 HTML 内的某处。我们用第二页的 JSON 文件名中的数字（这里为`1513381308`）来在源代码中进行搜索，可以发现一个`data-lastkey="1513381308"`的属性，我们可以利用它将所有的页码串联起来。
