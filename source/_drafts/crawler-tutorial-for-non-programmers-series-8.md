---
title: 给非程序员的爬虫教程（八）：Chrome 调试工具
tags:
  - python
  - 非程序员的爬虫教程
---

浏览器提供了一系列的工具，来辅助我们了解一个网站的加载过程。我们以 Chrome 为例，简单介绍浏览器调试工具的使用方式。我们选择 Chrome 的原因很简单：它的市场占有率最大，同时它的调试工具也足够强大。如果你的网络条件不允许，你可以选用一些国产“加壳”浏览器（例如QQ浏览器或 360 浏览器的“极速模式”），它们的浏览器内核基本都是采用的 Chromium 内核(开发版的 Chrome)，它们有着和 Chrome 类似的调试工具。

## 网页源代码
任意打开一个网页，在网页上右键＝>查看源代码，我们即可看到对应网站的 HTML 文档。这里看到的是网站的**源代码(source code)**，和我们看到网页时页面实际的 HTML 可能并不一致：因为 HTML 中`<script>`标签加载的 javascript 也会对 HTML 进行修改。这里看到的源代码，可以在一定程度上辅助我们梳理整个 Web 页面加载的流程。

## Chrome 开发者工具
Chrome 开发者工具可以通过下列几种方式打开：
1. 在网页可视区域点击右键，选择“检查”。
2. 使用键盘快捷键`Ctrl + Alt + I`打开。
3. 在菜单中，选择 **更多工具 > 开发者工具**。

[截图]

从图中，我们可以看到几个面板：
1. Elements：元素面板，用来实时显示网站当前的 HTML/CSS ，我们经常用它来即时修改页面的元素，可以马上看到结果。
2. Console：控制台面板是一个可以进行交互的 shell，用于输出一些调试 log 和执行一些 javascript 代码。
3. Sources：网站中用到的资源源代码。我们一般在这里调试我们的 javascript 代码。
4. Network：所有的网络请求。
5. Performance：性能面板，一般我们在这里监视网页的 CPU 占用。
6. Memory：内存面板，用于监视整个页面的内存占用。
7. Application：应用面板，用于展示网站所有存储、缓存的数据。
8. Security：安全面板，用于对网站中可能的安全问题做一些提示。
9. Audits：审计面板，为网站提供一些优化建议。

我们这里的目的是写爬虫，而爬虫最关注的是网页的各种资源，因此我们重点关注 Elements 和 Network 两个面板。

### Elements
### Network

## 扩展阅读
1. [Chrome 开发者工具官方文档](https://developers.google.com/web/tools/chrome-devtools/?hl=zh-cn)
2. [Chrome 开发者工具(第三方文档)](http://www.css88.com/doc/chrome-devtools/)
