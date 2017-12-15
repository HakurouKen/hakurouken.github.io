---
title: 给非程序员的爬虫教程（十）：使用 Python 发送 HTTP 请求
tags:
  - python
  - 非程序员的爬虫教程
---

要写一个爬虫，第一步就是要模拟一个 HTTP 请求，我们需要用到 Python 内置的`urllib`库。

## urllib
### GET 请求
我们使用`urllib`中的`request`来做发送请求的基础。
```python
from urllib import request
response = request.urlopen('http://www.zhihu.com/')

print('HTTP Status Code: ', response.status)

print('HTTP Response Headers: ')
for k, v in response.getheaders():
  print('{}: {}'.format(k, v))

print('Content: ')
print(response.read().encode('utf-8'))
```
上面描述了一个最简单的不含任何 HTTP 请求头的请求。如果我们需要添加一些 HTTP 头，则不能只传一个 url，而是要自己构造一个 Request 请求：
```python
from urllib import request
req = request.Request('http://www.zhihu.com/')
req.add_header('User-Agent','Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36')

response = request.urlopen(req)
# ...
```

### POST 请求
### Handler 与 Opener

## requests

## 参考资料
