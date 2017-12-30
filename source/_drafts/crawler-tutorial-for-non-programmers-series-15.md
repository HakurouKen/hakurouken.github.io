---
title: 给非程序员的爬虫教程（十五）：实例：微博热搜榜
tags:
  - python
  - 非程序员的爬虫教程
---

## 目标

[图片]

获取微博的实时热搜榜。

http://s.weibo.com/top/summary

## 分析

[图片]

我们直接通过查看源码([view-source:http://s.weibo.com/top/summary](view-source:http://s.weibo.com/top/summary))可以看到，整个页面的 HTML 部分（不包括 JavaScript 脚本）非常少，但同时，JavaScript 代码中有很多的 HTML 代码。我们需要的数据也正位于一段以`STK && STK.pageletM && STK.pageletM.view`开头的 JavaScript 代码中。

和我们 500px 的例子类似，我们需要先匹配出对应的`<script>`标签的内容，然后再从这部分内容中提取我们需要的部分来进行解析：在这里，我们需要先把对应的 HTML 提取出来，然后丢给 HTML 库（例如 PyQuery）解析即可。

这里特别说明下，代码中的汉字都被转义成了`\uxxxx`的格式，这是用 unicode 码的一种表示形式，我们可以在输出的时候将这些编码还原为汉字。类似的，在有些场景下，我们也会对一些字符进行 URL 编码，它的特征是`%`，例如“测试”经过 URL 编码后的结果为`%E6%B5%8B%E8%AF%95`。

## 编码
根据上述的分析，我们这里的思路相对比较清晰，因此不再赘述具体的逻辑步骤。我们需要额外注意一下[转义](https://zh.wikipedia.org/wiki/%E8%BD%AC%E4%B9%89%E5%AD%97%E7%AC%A6)的细节即可。

```python
import requests
from pyquery import PyQuery

def get_doc():
  response = requests.get('http://s.weibo.com/top/summary')
  doc = PyQuery(response.content)

  for script in doc('script').items():
    text = script.text()
    if text.startswith('STK && STK.pageletM && STK.pageletM.view({"pid":"pl_top_realtimehot"'):
      # 我们需要取得`{"html": "{CONTENT}"}`中的`{CONTENT}`部分
      html = text.split('"html":"', 1)[1].rstrip('"})')
      # 还原字符串中的转义部分
      html = html.replace('\\"', '"').replace('\\/', '/')
      return PyQuery(html)

def get():
  doc = get_doc()
  infos = []
  for i, item in enumerate(doc('tr').items()):
    # 第一个元素是标题，过滤掉
    if i == 0:
      continue
    infos.append({
      'rank': int(item('.td_01').text(), 0),
      'title': item('a').text(),
      'count': item('.star_num').text()
    })
  return infos

print(get())
```

## 扩展
1. 微博的大部分页面，都是采用类似的方式来实现的，我们可以将类似的逻辑应用到热门微博、发现页卡等等。
2. 如果我们想要获取时间线等**用户相关的信息**，需要拿到标识用户的 Cookie。涉及到登录的接口一般比较复杂，我们采用偷懒的方式：在浏览器上登录，然后复制登录后的 Cookie 到代码中即可。
