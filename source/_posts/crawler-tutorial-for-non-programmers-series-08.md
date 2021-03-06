---
title: 给非程序员的爬虫教程（八）：Chrome 调试工具
tags:
  - python
  - 非程序员的爬虫教程
date: 2017-12-31 21:26:54
---

浏览器提供了一系列的工具，来辅助我们了解一个网站的加载过程。我们以 Chrome 为例，简单介绍浏览器调试工具的使用方式。我们选择 Chrome 的原因很简单：它的市场占有率最大，同时它的调试工具也足够强大。如果你的网络条件不允许，你可以选用一些国产“加壳”浏览器（例如QQ浏览器或 360 浏览器的“极速模式”），它们的浏览器内核基本都是采用的 Chromium 内核(开发版的 Chrome)，它们有着和 Chrome 类似的调试工具。

<!-- more -->
## 网页源代码
任意打开一个网页，在网页上右键＝>查看源代码，我们即可看到对应网站的 HTML 文档。这里看到的是网站的**源代码(source code)**，和我们看到网页时页面实际的 HTML 可能并不一致：因为 HTML 中`<script>`标签加载的 javascript 也会对 HTML 进行修改。这里看到的源代码，可以在一定程度上辅助我们梳理整个 Web 页面加载的流程。

## Chrome 开发者工具
Chrome 开发者工具可以通过下列几种方式打开：
1. 在网页可视区域点击右键，选择“检查”。
2. 使用键盘快捷键`Ctrl + Alt + I`打开。
3. 在菜单中，选择 **更多工具 > 开发者工具**。

![Chrome 开发者工具](http://ww1.sinaimg.cn/large/9f9426adgy1fn09orcgh2j20n10is47h.jpg)

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

注意：在不同 Chrome 的版本下，这些 TAB 的表现会有一些小差异。

### Elements
![Chrome 开发者工具：Element 面板](http://ww1.sinaimg.cn/large/9f9426adgy1fn09orcgh2j20n10is47h.jpg)

Elements 面板分为两个区域，一部分展示当前的 HTML ，一部分展示当前选中的节点的样式，javascript 事件监听等等。这里的 HTML 已经被生成为一种树形结构的 DOM (Document Object Model)，我们可以右键编辑对应节点的属性和内容。

Elements 面板集成了搜索功能，我们可以在面板按下`Ctrl + F`，搜索并定位到指定的文字。

注意，这里的 HTML 和源代码中看到的不同：它是**实时**的。这就意味着，这里显示的 HTML 是经过 javascript 修改的，和我们当前在浏览器可视区域看到的界面完全对应的。我们鼠标移到对应的 dom 节点上，页面中的对应元素就会高亮；同时，我们也可以通过左上角的inspect 按钮，选定对应的元素来定位到对应的节点。因此，我们可以很方便的定位到我们所需要的信息所在的位置。但是爬虫获取到的数据是网页的源代码，因此，我们一般结合网页源代码和元素面板来定位资源。

### Network
Network 对于爬虫来说，是最重要的一个 TAB，我们的爬虫的本质就是为了模拟一个个的请求，拿到数据并解析。

![Chrome 开发者工具：Network 面板](http://ww1.sinaimg.cn/large/9f9426adgy1fn09pzxtzvj20mx0istdv.jpg)

Network 表格中，每一行就是一个请求。这里只会显示当前页面在调试工具打开之后产生的请求，如果需要查看这个页面的所有请求，尝试刷新页面即可。我们可以在表头右键编辑需要显示的列，这里简单介绍一下默认显示的几列内容的含义。

* Name：资源的名称和 URL 路径
* Status: 响应的 HTTP 状态码
* Type: 响应的类型(Content-Type)，又叫 Mime 类型
* Initiator: 请求是由谁发起的。常见的有四类：
  1. Parser: HTML 发起，说明这个资源是以标签形式写在 HTML 中的。
  2. Redirect：由页面重定向而来。
  3. Script：由 javascript 产生的请求。
  4. Other：其它。
* Size：显示这个请求的实际大小，和传输时的大小。一般服务器都会对文件进行压缩，来减小传输时的耗时。
* Time：请求的耗时（开始下载第一个字节经历的时间 / 下载完成经历的时间）。
* Waterfall：网络加载耗时的瀑布图。

![Chrome 开发者工具：Network 面板请求详情](http://ww1.sinaimg.cn/large/9f9426adgy1fn09r3a9gbj20n40is0yx.jpg)

点击某个请求的名称，我们还可以看到这个请求的详细内容，包括请求的内容、HTTP 请求头、HTTP 响应头、响应具体信息等等。

![Chrome 开发者工具：Network 面板筛选器](http://ww1.sinaimg.cn/large/9f9426adgy1fn09rt3gyij20n20iqtbv.jpg)

我们也可以根据自己的要求，来过滤(filter)我们需要的请求。Chrome 支持按名称/响应类型过滤请求。这里要特别说明一下 XHR 请求的全称为 (XmlHttpRequest)，也就是我们常说的 Ajax 请求。我们关注的数据，多数都在 Doc/XHR/JS 三个面板内。

## 扩展阅读
1. [Chrome 开发者工具官方文档](https://developers.google.com/web/tools/chrome-devtools/?hl=zh-cn)
2. [Chrome 开发者工具(第三方文档)](http://www.css88.com/doc/chrome-devtools/)
