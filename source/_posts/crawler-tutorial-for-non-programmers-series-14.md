---
title: 给非程序员的爬虫教程（十四）：实例：500px 首页风景图
tags:
  - python
  - 非程序员的爬虫教程
date: 2017-12-31 22:55:19
---

## 目标
入口 URL：http://500px.com/
![截图](http://ww1.sinaimg.cn/large/9f9426adgy1fn0cb7s8nhj21z20zi1gl.jpg)

获取 500px 首页的风景(landscape)图。

<!-- more -->
## 分析

![查看源代码](http://ww1.sinaimg.cn/large/9f9426adgy1fn0c4r6bz9j21z415u1kx.jpg)

通过查看源代码，我们很容易发现，我们要的数据就写在 HTML 代码中。准确的说，这个数据被`<script>`标签包裹着，是一段 Javascript 代码。我们首先需要定位到这段代码，由于网页中有很多`<script>`标签，它们明显的差异只有内容部分：我们可以尝试通过内容(`.text()`)来区分。例如，我们可以通过是不是以`window.photos`开头，来定位到我们需要找的代码所在的标签。
定位到这个标签之后，我们可以看到我们需要的数据，都被赋给了一个 javascript 变量。Python 是无法直接执行 JavaScript 代码的，我们只能将其作为**普通字符串**来处理。不过 JavaScript 的对象和 JSON 的格式非常类似，我们也可以考虑经过一些手段变换/提取之后，再使用`json.load`处理。

通过观察发现，我们需要的数据都在第 360 行，在`landscape:` 后到结尾逗号之前的部分是个标准的 JSON 字符串。我们可以将这段字符串匹配出来，然后采用标准的 JSON 进行解析。

## 编码
1. 拿到页面 HTML ，遍历所有的`<script>`标签，找到文本以`window.photo`开头的代码段。
2. 从中找到`landscape`的所在行，处理得到中间的部分，使用`json.load`处理。
3. 处理得到的数据中，我们可以方便的得到图片的名称、URL 等信息，利用这些可以下载图片到本地。

示例代码：
```python
import requests
from pyquery import PyQuery
import json

def get_info():
  response = requests.get('https://500px.com/')
  doc = PyQuery(response.content)

  for item in doc('script').items():
    text = item.text().strip()
    # 如果以 `window.photo` 开头，说明是我们需要的 script 脚本
    if text.startswith('window.photo'):
      for line in text.split('\n'):
        line = line.strip()
        # 我们只需要 `landscapes` 开头的行
        if line.startswith('landscapes:'):
          dumped = line.lstrip('landscapes:').rstrip(',').strip()
          return json.loads(dumped)

info = get_info()
print(info)
```

## 扩展思考
1. 上文的示例代码省略了通过 URL 下载图片的部分，请通过之前的例子尝试自行补全。
2. 除了使用`json.load`来处理数据，形如**从固定的格式中，提取特定的文本**的需求，我们还可以尝试使用**正则表达式**来实现。正则表达式的上手比较困难，但是它可以大大简化很多文本的逻辑。
