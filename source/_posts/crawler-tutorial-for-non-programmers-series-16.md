---
title: 给非程序员的爬虫教程（十六）：接下来需要学习什么
tags:
  - python
  - 非程序员的爬虫教程
date: 2017-12-31 23:04:57
---
通过前面的学习和例子，我们已经能够编写一些简单的爬虫，足以我们应付大多数场景下的数据获取。但是，为了应付一些复杂的场景，我们仍需要接触一些其它的知识。
<!-- more -->
## 正则表达式

正则表达式是我们在对**满足特定规则的大量字符串**进行处理时的利器。我们盗用 XKCD 的一幅漫画，来描述正则表达式的强大：

![正则表达式](https://imgs.xkcd.com/comics/regular_expressions.png)

Python 中内置了正则表达式的库`re`。不仅是 Python ，绝大多数编程语言和文本编辑器中的搜索/替换都支持正则表达式。但正则表达式强大同时也带来一些副作用，相比不用正则表达式的代码，它的“信息密度”较大，过于复杂的正则表达式规则，不太利于后续的维护。

## 反反爬虫措施
绝大多数网站，都会有一定的反爬虫(anti-spider)的措施。因此，如果我们要在短时间内从同一个网站收集大量的信息，就需要采取一些反反爬虫措施。这里列举一些基础的反反爬虫措施供参考：
1. 填写尽可能完整的 HTTP 头。我们的请求和真实用户的数据越相似，服务器就越难以分辨。
2. 适当限制爬虫获取数据的频率。这样可以在一定程度上避开服务器的单个请求的频率限制。
3. 使用多账号、多 IP 做分布式爬虫。这样可以使我们的每个爬虫和正常用户的请求无异。
4. 使用一些现成的方案，对抗验证码。

## 爬虫框架
我们目前编写的爬虫都是针对单个或少量页面获取特定数据，场景相对简单，请求数也相对较少。对于一些复杂的场景，我们可能需要借助爬虫框架来简化逻辑。例如 Python 中的 [scrapy](https://scrapy.org/) 和 [pyspider](https://github.com/binux/pyspider)，都是构建大型爬虫不错的选择。

## 更多通用技能
1. Python 的一些核心库和一些常用的第三方库。了解这些的使用方法，有助于我们编写更加简洁的代码。
2. 一些基本的 JavaScript 。了解 JavaScript 的逻辑，有助于我们了解页面是如何获取数据的。
3. 一些简单的数据库知识。对于稍微大量的数据，如果存成纯文本，增删改查将会十分繁琐。为了管理方便，我们就会将这些数据存入数据库。

## 结语
“给非程序员的爬虫教程”这个系列到此就结束了。虽然本系列文章是“面向非程序员”，但是回头看去，还是很多地方代入了程序员的惯性思维。爬虫作为一种应用程序，编写的思路和通用的应用开发是相通的。如果能够系统学习一些编程的知识，将会有助于我们理解和开发爬虫程序。