---
title: 给非程序员的爬虫教程（十二）：实例：下载 bilibili 的标题栏小电影
tags:
  - python
  - 非程序员的爬虫教程
---

爬虫相关的基础知识，我们已经全部“介绍”完了。接下来，我们将通过一系列的实例，来综合运用之前学到的一些技能。同时从这些实例中，总结出一些通用的经验。

## 目标

用爬虫下载 Bilibili 标题栏的 GIF 动画。

[图片]

## 分析

要下载一张图片，首先要拿到这个图片的 URL。在图片上点击右键“检查”，进入 Chrome 的 Element 面板，我们可以很容易的在面板中找到图片的 URL。

[图片]

在拿到 URL 之后，下一步我们需要确定浏览器是如何拿到这个 URL 的。我们在浏览器中常见的数据来源以及对应的解析方式大概有三种：
1. 原本就是完整的 HTML 标签/属性：直接解析 HTML 即可拿到对应的数据；
2. 数据**直接**写在 HTML 内，但是经过 Javascript 进行了一次计算后得到的：需要用一系列的手段解析处理才能得到对应的数据；
3. 通过 Ajax 请求获取，然后再被 Javascript（经过一系列的计算后）添加到 HTML 中的：我们需要模拟对应的 ajax 请求，然后处理拿到的数据（一般是 html/xml 或 json）。

显然，我们这个例子中，我们需要的数据就是这个图片的 URL。我们首先要确定的是这个 URL 的数据源是哪里：在网页**右键 ＝> 查看网页源代码**，我们可以看到浏览器拿到的真实文本，这也是我们通过模拟浏览器请求拿到的真正结果。

[图片]

如果我们在源代码中可以找到图片 URL 的相关信息（即上述的情况1 或情况 2），意味着我们只需要发送`https://www.bilibili.com`这个请求并解析得到的响应，即可拿到我们想要的数据。查看源代码页面支持`Ctrl + F`进行搜索，可以节省一些我们人肉查找的时间。在使用搜索时，**我们需要找到合适的关键词**，因为我们在 Element 面板中看到的元素都可能是经过 Javascript 进行过处理的，原始的数据和我们看到的可能不完全一致。但是，一个简单的原则是：Javascript 一般对于我们要的关键数据，并不会做巨大的变动，一般只是简单的变换以及拼接。以我们这次需要搜的 URL 为例，`y4o19nyvmj`看上去就是一个合适的搜索关键词：它看上去像是一个唯一的 ID，不大可能由前端直接生成。当然，也有极少的情况下（例如做了前端加密），计算前和计算后的结果完全没有相似之处，这种就只能通过分析 HTML 和 Javascript 代码逻辑才能得到我们想要的数据。

我们在源代码中搜索的关键字并没有得到结果，基本可以确定，这个图片的 URL 是由 Ajax 请求得到的。接下来，我们需要打开 Network 面板，刷新页面，去寻找相关的请求。

[图片]

这里有非常多的请求，我们可以使用面板上的类型过滤。一般情况下，我们需要的数据都是 XHR 类型（Ajax 请求）或 Script 类型（直接把数据写在 Javascript 中）。为了维护方便，绝大多数的请求，我们都可以从命名和参数中，大致猜测出来接口的用途，因此并不需要挨个查看每个请求的详细信息。

[图片]

我们很容易能够找到名为 index-icon.json 的这个请求（实际对应的 URL 为 http://www.bilibili.com/index/index-icon.json），它包含了首页标题栏可能展示的所有 GIF 的详细信息，包括名称、URL、展示权重等等。我们只需要读取这个 JSON，就可以拿到所有我们所有想要的数据，而无需发送 http://www.bilibili.com 这个请求。

## 编码
知道了数据加载过程，我们接下来只需要用代码还原这个过程即可：
1. 请求 http://www.bilibili.com/index/index-icon.json ，拿到原始的 JSON 数据。
2. 格式化拿到的数据，从中取出我们想要的字段（名称和 URL）。
3. 对于每张图片分别进行处理：请求对应图片得到二进制数据，并写入一个空文件。文件名取对应的`title`字段即可。我们需要额外处理一下后缀名，这里只要取和 URL 相同的后缀名即可。

下面给出一个示例代码：
```python
import requests
import os
 
URL = 'http://www.bilibili.com/index/index-icon.json'

def get_ext(filepath):
  return os.path.splitext(filepath)[1]

def get():
  resp = requests.get(URL)
  arr = resp.json().get('fix',[])
  return [{
    'name': data["title"] + get_ext(data["icon"]),
    'url': data["icon"]
  } for data in arr]
 
def download():
  all_data = get()
  if not os.path.isdir('icons'):
    os.mkdir('icons')
  for data in all_data:
    print('downloading {}, url: {}'.format(data['name'],data['url']))
    with open(u'icons/{}'.format(data['name']),'wb') as f:
      f.write(requests.get(data['url']).content)

if __name__ == '__main__':
  download()
```

## 扩展思考
1. 我们下载文件时，用的方法是用 request 请求这个文件，再将请求到的内容写入一个空文件。而在 Python 的 urllib 中，有`urllib.request.urlretrive`方法用来处理"下载"这一行为。我们可以尝试使用这个方法，来对我们的代码稍稍进行简化。
2. index-icon 这个文件的内容可能会随着时间变更（例如加入新的动画），我们需要。最简单的方法，就是每过一段时间自动执行一次这段程序。在 Windows 下，我们可以采用计划任务来实现。
