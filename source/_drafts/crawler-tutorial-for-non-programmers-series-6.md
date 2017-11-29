---
title: 给非程序员的爬虫教程（六）：包和包管理工具
tags:
  - python
  - 非程序员的爬虫教程
---

## 模块(module)
随着我们的程序不断的复杂，我们可能将我们的程序拆分橙多个文件以便于维护。或者，我们还可能想在几个程序中，使用同一个函数或类，而不是将定义复制到每个程序中。我们将一个这样一个包含 Python 声明和定义的文件就称为**模块(module)**。例如，我们将一个`is_prime`函数保存为`prime.py`文件，这个文件就是一个 Python 模块。
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

## Python 内置模块

## 第三方包

## 包管理

## 扩展阅读
1. Python 的模块机制[官方文档](https://docs.python.org/3/tutorial/modules.html)
2. pip 的使用，以及虚拟环境(virtualenv)的[官方教程](https://packaging.python.org/tutorials/installing-packages/)