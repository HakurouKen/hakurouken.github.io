---
title: 给非程序员的爬虫教程（十三）：实例：好奇心日报长文章列表
tags:
  - python
  - 非程序员的爬虫教程
date: 2017-12-31 22:43:24
---

## 目标

入口 URL：http://www.qdaily.com/tags/1068.html

![截图](http://ww1.sinaimg.cn/large/9f9426adgy1fn0bxvmcdsj21z4166e13.jpg)

批量获取好奇心日报长文的标题、正文链接、更新时间等基础信息。
<!-- more -->

## 分析

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

![XHR 请求](http://ww1.sinaimg.cn/large/9f9426adgy1fn0bthln1rj213w0vugv0.jpg)

通过观察，我们不难发现如下的规律：上一页返回的数据中的`last_key`字段，就是下一页的请求的 `.json` 的文件名。这是在这种“无限滚动”的场景下，十分常见的一种翻页模式：它的特点是，在返回一页数据的同时，返回一个特征值，Web 可以利用这个特征值来安全的获取当前数据的下一页数据。有些人形象的将这个参数称之为“接力参数”。

利用这个`last_key`，我们可以通过第 2 页的数据来获取第 3 页的数据，再通过第 3 页的数据通过第 4 页的数据，依次类推。同时我们不难想到，第二页的`last_key`的相关数据，一定位于 HTML 内的某处。我们用第二页的 JSON 文件名中的数字（例如`1513381308`）来在源代码中进行搜索，可以发现一个`data-lastkey="1513381308"`的属性，我们可以利用它将所有的页码串联起来。

## 编码
根据我们上文分析的结果，我们代码大致可以采取下列流程实现：
1. 发送首页的请求，解析 HTML 拿到第一页的数据和 lastkey
2. 根据 lastkey，拿到下一页的数据，并根据返回值更新 lastkey
3. 循环执行步骤 2（直至满意为止，例如我们可以设定取 10 页数据）

下面是一段示例代码：
```python
import requests
from pyquery import PyQuery

def get_first_page():
  ''' 获取第一页的数据以及 lastkey '''
  response = requests.get('http://www.qdaily.com/tags/1068.html')
  doc = PyQuery(response.content)

  infos = []
  infos = [{
    'url': 'http://www.qdaily.com{}'.format(link.attr('href')),
    'title': link('.title').text(),
    'date': link('.smart-date').attr('data-origindate')
  } for link in doc('.packery-item>a').items()]

  lastkey = doc('.packery-container.articles').attr('data-lastkey')

  return (lastkey, infos)

def get_more(lastkey):
  ''' 根据 lastkey 获取数据，以及下一页的接力 lastkey '''
  nexturl = 'http://www.qdaily.com/tags/tagmore/1068/{}.json'.format(lastkey)
  response = requests.get(nexturl)
  data = response.json()

  infos = [{
    'url': 'http://www.qdaily.com/articles/{}.html'.format(feed['post']['id']),
    'title': feed['post']['title'],
    'date': feed['post']['publish_time']
  } for feed in data['data']['feeds']]

  lastkey = data["data"]["last_key"]
  return (lastkey, infos)

def get(page=1):
  ''' 获取前 page 页的数据，如果不传参数，默认只取第一页 '''
  lastkey, infos = get_first_page()
  p = 1

  while p < page:
    lastkey, page_infos = get_more(lastkey)
    infos += page_infos
    p += 1

  return infos

print(get(5))
```

这段代码写的比较冗长，主要原因是要对第一页进行特殊处理。我们有没有办法简化掉这第一页的特殊处理的步骤呢？换位思考一下，如果你是这个网站的开发者，为了用户体验，我们可以把第一页的数据直接写到 HTML 中（这样省去了 ajax 再请求数据所花费的时间），但我们每一页的数据来源应该是一致的，从数据层面上讲，我们没有必要对第一页的数据做特殊处理。这意味着，我们有可能使用和获取第二页的数据类似的方式获取第一页的数据。我们第一页并没有`lastkey`这个数据，但所有的 lastkey 是数字，我们可以尝试使用默认值`0`试一试：访问`http://www.qdaily.com/tags/tagmore/1068/0.json`，我们发现成功返回了第一页的数据。

接下来，我们就可以去掉`get_first_page`函数，来简化我们的代码(`get_more`函数不做任何修改)：
```python
def get(page=1):
  p = 0
  lastkey = 0
  infos = []
  while p < page:
    lastkey, page_infos = get_more(lastkey)
    infos += page_infos
    p += 1
  return infos
```

不过，值得说明的是，这里的方法并不是一个通用的解决方案。在多数场景下，我们可能需要反复尝试才能得到最优的解决方案。

## 扩展思考
1. 我们这里获取到的日期字符串是一个复杂格式的字符串，我们可以使用`datetime`类的`strptime`将其转化为 datetime 对象，然后再用`strftime`处理为我们需要的格式。
2. 我们现在是直接将结果 print 到控制台，无法进行持久存储。对于这种对象列表，我们自然而然的想到使用表格存储。我们可以使用一些读写 xls 文件的第三方库（例如`XlsxWriter`）来读写一个 Excel 表格；或者采用简单的 csv 格式来保存。绝大多数表格工具（例如 Excel 和 WPS 等）都可以读写 csv。Python 内置了 csv 的读写库，我们可以使用它来存储一些简单的结果。如果涉及到大量数据，我们可以考虑存储到**数据库**中。
