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