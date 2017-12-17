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
# 拿到 response 之后，其它的操作和上文一样
# 例如读取 html 响应正文：`response.read().encode('utf-8')`
```
如果我们想要添加多个请求头，调用多次`add_header`方法即可。在实际请求中，我们经常使用的头有 `Referer`、`Host`、`Cookie`、`User-Agent`等等。

如果我们想添加 GET 请求的参数，直接拼接到 URL 后即可。需要注意的是，我们需要自行对参数进行转义。Python 也提供了内置的转义方法供我们使用：
```python
from urllib import request, parse
url = 'http://www.zhihu.com/'
url += parse.urlencode([
  ('type', 'content'),
  ('q', 'Python 爬虫')
])
response = request.urlopen(url)
```

### POST 请求
POST 请求和 GET 请求类似，我们只需要将 POST 数据显式由`data`参数显式传入即可。我们以豆瓣 FM 的登录接口为例：
```python
from urllib import request, parse

data = parse.urlencode([
  ('source', 'fm'),
  ('referer', 'https://douban.fm/'),
  ('ck', ''),
  ('name', ''),
  ('password', ''),
  ('captcha_solution', ''),
  ('captcha_id', '')
])

req = request.Request('https://accounts.douban.com/j/popup/login/basic')
```
注意：豆瓣 FM 的登录接口还是用的明文密码传输，尽管采用了 HTTPS 来保证了传输过程中的安全，但是仍然有些安全隐患。目前比较成熟的登录服务，都会有一些复杂的机制来保证**除了用户自己，没有任何一个环节（包括网站自己）有可能拿到用户的密码明文**。因此，往往会引入一些前端的不对称加密。这种情况比较复杂，我们就需要去了解客户端的 javascript 逻辑，模拟这一加密过程，计算得到真实的请求。

### Handler/Opener
对于一些复杂的场景，我们就需要接触 urllib 中的各种 handler。例如，有的公司会有一套自己的网络环境，内部所有的网络请求都要通过代理来访问。这时我们就会用到`ProxyHandler`。又例如，有些 HTTP 服务器访问时需要认证，这时我们可能会用到`HTTPBasicAuthHandler`。

在构建好一个 Handler 之后，我们需要使用一个自定义的 Opener 来代替 urlopen 生成的 Opener 来处理 url。以 Github 的第三方 API 调用为例：
```python
from urllib import request

url = 'https://api.github.com'

req = request.Request(url)

password_manager = request.HTTPPasswordMgrWithDefaultRealm()
password_manager.add_password(None, url, 'user', 'pass')

auth_manager = request.HTTPBasicAuthHandler(password_manager)
opener = request.build_opener(auth_manager)

request.install_opener(opener)

response = request.urlopen(req)

print(response.getcode())
print(response.headers['content-type'])
print(response.read())
```

## requests
为了简化一些繁琐的 HTTP 请求的处理，我们可以引入一个第三方模块`requests`。这是一个在 Python 中非常实用的 HTTP 请求库，可以大大简化我们 HTTP 请求的构造过程。requests 有着详细的[官方上手指南](http://docs.python-requests.org/zh_CN/latest/user/quickstart.html)，这里不再进行赘述。

## 扩展阅读
1. [Python urllib 模块官方文档](https://docs.python.org/3/library/urllib.request.html)
2. [requests 官方文档](http://docs.python-requests.org/zh_CN/latest/index.html)
