---
title: 给非程序员的爬虫教程（六）：包和包管理工具
tags:
  - python
  - 非程序员的爬虫教程
date: 2017-12-31 21:20:01
---

## 模块(module)
随着我们的程序不断的复杂，我们可能将我们的程序拆分橙多个文件以便于维护。或者，我们还可能想在几个程序中，使用同一个函数或类，而不是将定义复制到每个程序中。我们将一个这样一个包含 Python 声明和定义的文件就称为**模块(module)**。例如，我们将一个`is_prime`函数保存为`prime.py`文件，这个文件就是一个 Python 模块。
<!-- more -->

```python
# prime.py
def is_prime(n):
  for i in range(2, n):
    if n % 2 == 0:
      return True
  return False

def is_not_prime(n):
  return not is_prime(n)
```
模块可以用`import`的方式引入，我们在和`prime.py`同文件夹下，建立一个`main.py`文件：
```python
# main.py
# 我们用 import 语法，引入 prime 这个模块
import prime
prime.is_prime(7) # 返回 True
prime.is_not_prime(8) # 返回 False

# 或者采用 from ... import 的语法，引入指定的方法、类或变量
from prime import is_prime
is_prime(7) # 返回 True

# 也可以同时引入多个，用逗号隔开即可
from prime import is_prime, is_not_prime
is_not_prime(999) # 返回 True
```

## 包(Package)
包是 Python 使用“点号连接的模块名”来区分不同**命名空间(namespace)**的方式（我们可以简单理解为沙箱）。例如名称`packA.packB`代表着名称为`packA`的包中，名称为`packB`的子模块。举个例子，假设我们想要编写一个声音处理的模块(包)，我们把格式处理、效果器、过滤器等等分开维护，可能有如下的文件结构：
```
sound/                          顶层包
      __init__.py               初始化时执行，可以为空
      formats/                  子包
            __init__.py
            wavread.py
            wavwrite.py
            aiffread.py
            aiffwrite.py
            auread.py
            auwrite.py
            ...
      effects/                  子包
            __init__.py
            echo.py
            surround.py
            reverse.py
            ...
      filters/                  子包
            __init__.py
            equalizer.py
            vocoder.py
            karaoke.py
            ...
```
所有的 Python 包(和其子包)的文件夹下，都**必须有`__init__.py`这个文件**，只有这样，Python 才会把该文件夹认定为合法的 Python 包。`__init__.py`文件可以为空，但是必须要有这个文件。

我们可以单独引入某个模块，例如：
```python
import sound.effect.echo
# 调用模块上的 echofilter 方法
sound.effects.echo.echofilter(input, output, delay=0.7, atten=4)
```
我们也可以从包中引入单个模块：
```python
from sound.effects import echo
# 调用模块上的 echofilter 方法
echo.echofilter(input, output, delay=0.7, atten=4)
```
我们也可以从模块中，单独引入某个函数/类/变量:
```python
from sound.effects.echo import echofilter
echofilter(input, output, delay=0.7, atten=4)
```

需要注意的是，使用`from package import item`语法时，item 可以是子模块（包），也可以是模块内的变量/函数/类。但是使用`import package`的语法时，package 必须是一个包（模块）。

## Python 内置模块
除了内置方法，Python 提供了很多内置模块（包）供我们调用。举个例子，Python 有个`datetime`模块，用来处理日期/时间，举个简单的例子：
```python
from datetime import datetime
# 输出当前时间
print(datetime.datetime.now())
```
我们也可以使用内置的`urllib`模块，来获取一个网站的 favicon (小图标)：
```python
import urllib.request
resp = urllib.request.urlopen('http://www.qq.com/favicon.ico')
with open('favicon.ico', 'wb') as f:
  f.write(resp.read())
```
Python 提供了非常多实用内置的模块，我们在[官方文档](https://docs.python.org/3/library/index.html)中查看所有的模块。

## 第三方包
除了 Python 内置的包，我们还可以使用第三方包，合理的使用第三方包，可以大大提高我们的开发效率。例如，我们可以使用第三方包`requests`来处理 http 请求，比起`urllib`可以节省大量的代码。

在使用第三方包前，我们必须先安装，我们通常用 pip 来管理 Python 的第三方模块。我们当前只需要了解安装命令：`pip install PACKAGE_NAME`，例如我们可以使用`pip install requests`来安装`requests`。

## 虚拟环境
所有的第三方包，都有**版本(version)**的概念：任何的软件开发，都会不断有 bug 修复、新功能添加、老功能的废弃等等。假如我们的机器上有很多个 Python 项目，有些开发的比较早，依赖了 requests 1.2 的版本，那么对于新开发的项目，我们想要使用 requests 1.3 的版本，应当如何处理呢？这时，我们需要引入**虚拟环境(virtual environment)**的概念，z在 Python 中，我们可以使用第三方模块`virtualenv`或者`venv`来进行虚拟环境的管理。我们这里不会对这两个的使用进行过多的描述，我们初期的开发也不需要对这个进行过多了解，如果感兴趣，可以自行查看[官方文档](https://packaging.python.org/tutorials/installing-packages/)。

## 扩展阅读
1. Python 的模块机制[官方文档](https://docs.python.org/3/tutorial/modules.html)
2. pip 的使用，以及虚拟环境(virtualenv)的[官方教程](https://packaging.python.org/tutorials/installing-packages/)