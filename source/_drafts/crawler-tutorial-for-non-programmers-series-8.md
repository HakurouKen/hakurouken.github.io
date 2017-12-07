---
title: 给非程序员的爬虫教程（八）：Chrome 调试工具
tags:
  - python
  - 非程序员的爬虫教程
---

浏览器提供了一系列的工具，来辅助我们了解一个网站的加载过程。我们以 Chrome 为例，简单介绍浏览器调试工具的使用方式。我们选用 Chrome 的原因很简单：它的市场占有率最大，同时它的调试工具也足够强大。如果你的网络条件不允许，你可以选用一些国产“加壳”浏览器（例如QQ浏览器或 360 浏览器的“极速模式”），它们的浏览器内核基本都是采用的 Chromium 内核(开发版的 Chrome)，它们有着和 Chrome 类似的调试工具。

## 网页源代码
任意打开一个网页，在网页上右键＝>查看源代码，我们即可看到对应网站的 HTML 文档。这里看到的是网站的**源代码(source code)**，和我们看到网页时页面实际的 HTML 可能并不一致：因为 HTML 中`<script>`标签加载的 javascript 也会对 HTML 进行修改。这里看到的源代码，可以在一定程度上辅助我们梳理整个 Web 页面加载的流程。

## Chrome 开发者工具

## 扩展阅读
1. [Chrome 开发者工具官方文档](https://developers.google.com/web/tools/chrome-devtools/?hl=zh-cn)
2. [Chrome 开发者工具(第三方文档)](http://www.css88.com/doc/chrome-devtools/)
