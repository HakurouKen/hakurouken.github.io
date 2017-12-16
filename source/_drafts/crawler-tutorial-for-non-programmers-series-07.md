---
title: 给非程序员的爬虫教程（七）：HTTP 协议与 Web 基础概念
tags:
  - python
  - 非程序员的爬虫教程
---

爬虫程序设计的初衷，就是让程序来替代人来获取数据。人获取数据一般都是通过浏览器，爬虫也就是**模拟浏览器获取数据**。具体的说，我们需要模拟的就是从地址栏输入网址到浏览器拿到数据的过程。为此，我们需要先要引入一系列的基础概念。

本章的概念相对抽象，我们可以结合下一章的“浏览器的调试工具”，在浏览器中通过真实网站的请求，辅助理解本章的概念。

## URL
我们平时输入的网址，是一种**统一资源定位符(Uniform Resource Locator)**，简称 URL。它有一套标准的格式：
```
协议类型:[//服务器地址[:端口号]][/资源层级UNIX文件路径]文件名[?查询][#片段ID]
```
以`https://en.wikipedia.org/w/index.php?title=URL`为例：
1. 协议类型(protocol)为`https`
2. 服务器地址/域名(domain)是`en.wikipedia.org`
3. 端口号(port)这里已经省略了，对于 https 来讲默认是`443`
4. 路径(path)为`/w/index.php`
5. 查询(search)为`?title=URL`（以`?`为起点，每个参数用`&`隔开，用`=`分隔参数和名称）

大部分浏览器中，浏览器并不强制要求我们输入协议类型，默认为 http（同时默认端口是80）。服务器可以根据我们的 URL 来推断需要返回给我们的资源。注意，即使 url 完全相同，服务器返回给我们的内容可能也有所不同，这和我们的请求中其它的参数，以及服务器的逻辑决定。

## HTTP 与 HTTPS
HTTP(HyperText Transfer Protocol，超文本传输协议)是在 Web 中最常见的通信协议，它的整个通信过程，我们可以简化为“发送请求，得到响应”：

[示例图]

输入网址后，HTTP 客户端(通常是浏览器，或者是我们的爬虫程序)会通过这个网址找到对应的**服务器(server)**，并将对应的数据发送过去，这一过程我们通常称之为**请求(request)**。服务器会根据我们输入的路径，通过一系列的计算，返回给我们对应的数据，这一过程我们称之为**响应(response)**。我们将分别从请求和响应两方面来详细介绍下 HTTP 协议。

### HTTP 请求
#### 请求方法
HTTP 协议中定义一系列的方法，来方便客户端以不同方式操作 URL 定位到的对应资源。
1. GET：“读取”对应的资源。一般情况下，GET 请求不应当产生“副作用”，即不会对服务器资源产生影响。
2. HEAD：和 GET 类似，HEAD 请求也是获取指定资源的请求。和 GET 不同的是，HEAD 只需要该资源的 HTTP 头信息（元信息）。
3. POST：向服务器提交数据，并请求服务器进行处理。
4. PUT：向服务器上传新内容。
5. DELETE：请求服务器删除对应资源。
6. TRACE：显示服务器收到的请求（一般用于测试）。
7. OPTIONS：使服务器返回所有支持的 HTTP 方法。
8. CONNECT：和服务器建立双向连接。
9. PATCH： 局部更新对应资源。  

在上文列出的方法中，最常用的是 GET 和 POST。需要注意的是：上文对这些方法的作用，仅仅是一种约定，具体的实现还是要看服务器端的具体实现。然而，很多服务器端的实现并不规范，我们写爬虫时，只需将请求方法保持和网页端的请求保持一致即可。

#### 请求头(request header)
HTTP 协议允许我们使用请求头，来附加一些请求的额外信息。标准中规定了一系列的请求头，我们列出几个写爬虫时经常需要注意的头：
* Cookie: 浏览器 Cookie，一般用于标识用户身份，以及储存一些数据。
* Host: 服务器的域名。
* User-Agent: 浏览器的身份标识字符串。
* Referer: 表示浏览器访问的前一个页面。

除了标准中定义的请求头，很多网站也会自定义一些请求头，例如很多 Javascript 框架会用`X-Requested-With`来标识是否是 **ajax 请求**。

#### GET 请求参数
我们平时在浏览器中，绝大部分时间是在“获取数据”，GET 是使用频次最高的请求。所有的请求都可以带**参数(parameter)**，GET 请求也不例外。GET 请求的参数有一点比较特殊：它是直接拼接在 URL 中的。GET 参数在 URL 中有一个特征：处在`?`之后，用`&`符号来分隔每一对值，关键词和值用`=`分隔。我们以知乎的搜索页面为例，在知乎中用“Python”关键词进行搜索，我们从地址栏中得到的 URL 如下：
```
https://www.zhihu.com/search?type=content&q=Python
```
其中`type=content&q=Python`部分就是我们的搜索参数。我们用 Python 中的 dict 来描述这个请求想要表达的内容，大致就是：
```python
{
  "type": "content",
  "q": "Python"
}
```
如果我们请求的值中含有一些特殊符号（例如`&`或`=`），则需要进行**URL 转义**，例如我们把搜索关键词改成"Python&&"，可以看到 URL 变为：
```
https://www.zhihu.com/search?type=content&q=Python%26%26
```
这里的`%26`就是`&`的 URL 转义结果。类似的，汉字也会被进行 URL转义，例如我们搜索“Python 爬虫”，实际的 URL 会是:
```
https://www.zhihu.com/search?type=content&q=Python%20%E7%88%AC%E8%99%AB
```
注：如果你使用的是比较高版本的 Chrome，浏览器在地址栏中会“智能”的对转义的 URL 参数进行格式化，让请求看上去更加直观。我们可以把它复制出来，就能够看到真实的 URL。

### HTTP 响应
#### 状态码
HTTP 的响应状态码是一个三位数字，描述了当前返回响应的状态。例如我们常见的 404，表示页面不存在；200 表示请求成功等等。状态码的第一位，标识了该状态所属的类型：
1. 1xx —— 请求已被服务器接收，继续处理
2. 2xx —— 请求已成功被服务器接收、理解、并接受
3. 3xx —— 需要后续操作才能完成这一请求
4. 4xx —— 请求含有词法错误或者无法被执行
5. 5xx —— 服务器在处理某个正确请求时发生错误

下面列举几个常见的错误码：
* 200(OK): 请求成功
* 301(Moved Permanently)：请求重定向(永久)
* 302(Moved Temporarily)：请求重定向(临时)
* 400(Bad Request): 请求格式有错误，服务器不能处理
* 403(Forbidden): 服务器拒绝执行该请求(一般是权限不足)
* 404(Not Found): 资源未找到
* 500(Server Internal Error): 服务器异常

#### 响应头
和 HTTP 请求类似，HTTP 响应也有头部。不过大多数响应头都是自定义的，而且我们需要的信息，多数也不在响应头内，下面只简单列出两个常见的响应头`Content-Length`和`Content-Type`：
* Content-Length: 请求体的长度
* Content-Type: 返回的请求体类型。例如`text/javascript`,`text/html`,`image/png`,`image/jpeg` 等等。

#### 响应体
HTTP 响应体就是我们拿到的数据，它有可能是文本/脚本，也有可能是二进制数据（例如图片）等等。

### SSL
HTTPS 的通信过程和 HTTP 完全一致，不同的是它传输的数据都是加密的。它利用一种特殊的算法(RSA 加密算法)，保证只有发出方能够解密出 HTTP 的信息。这一过程，在底层已经做了处理，我们的上层应用不感知，在我们编写爬虫或网站时，就默认 HTTP 和 HTTPS 传输的信息格式一致即可。

## HTML、CSS 与 Javascript
HTML、CSS 和 Javascript 是构成 Web 站点的三个不可或缺的组件。想要构建一个 Web 爬虫，我们也必须对这三个组件有些基本了解。

### HTML
HTML(HyperText Markup Language，超文本标记语言)是用来描述网页的**标记语言**。它并不是一种编程语言，因为它本身没有包含逻辑。HTML 由一系列的嵌套的**标签(tag)**组成，例如，我们可以写一个简单的 HTML 页面：
```html
<!DOCTYPE html> 
<html> 
<body> 
  <h1>标题</h1> 
  <p>段落</p> 
</body> 
</html>
```
把上述代码复制到文件中，并重命名为`test.html`，在浏览器内即可打开。上文中的`<h1>标题</h1>`和`<p>段落</p>`，都是一对标签。**我们能够在页面上看到的数据，必定会有对应的 HTML 标签，因此有人把 HTML 称为 Web 的“骨架”。**有关 HTML 的更多细节，我们可以参照[W3Cschool HTML 教程](https://www.w3cschool.cn/html/)进行进一步的了解。在看这个教程的时候，无需刻意记住每个标签对应的含义，最重要的是通过一系列的示例，去切实的理解“标签”这一概念。

### CSS
CSS(Cascading Style Sheets，层叠样式表) 是用来描述页面样式的一种标记语言。我们可以简单把 CSS 理解为“皮肤”：它并不会影响我们看到的核心数据。在写爬虫时，我们基本无需关注 CSS 的影响。

### Javascript
javascript 是在 Web 页中使用的**脚本语言**。和 HTML/CSS 不同，javascript 是一个拥有逻辑的完整语言。最早的网页中，javascript 只负责在网页中实现一些小的动画和交互效果使用，然而随着浏览器的不断发展和 Web 页的复杂度不断的提升，javascript 在 Web 中的角色变得越来越重要。javascript 可以在页面不刷新的情况下，从服务器获取数据，同时也可以动态插入 HTML/CSS。今天，javascript 承担了互联网中大部分的交互需求，我们已经很难找到不使用 javascript 的网站了。

尽管 javascript 在 Web 中占有如此重要的地位，但是值得注意的是：**数据是不会无中生有的**。绝大部分有价值的数据，都是存储在服务器上（如果只是存储在本地，则无法多地同步），一定有数据下载到客户端（即浏览器）这一过程。因此，对于爬虫，我们只要关注对应的请求，无需完整的了解 Web 的 javascript 逻辑。由于 HTTP 是**无状态协议**，我们的每个请求都是相对独立的，我们完全可以绕过 javascript 逻辑，直接调用接口，也可以拿到正常的结果。

## 扩展阅读
1. [维基百科：统一资源标志符(URI)](https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6)
2. [维基百科：统一资源定位符(URL)](https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E5%AE%9A%E4%BD%8D%E7%AC%A6)
3. [维基百科：超文本传输协议(HTTP)](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)
4. [维基百科: 超文本传输安全协议(HTTPS)](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8%E5%8D%8F%E8%AE%AE)
5. [维基百科：HTTP头字段](https://zh.wikipedia.org/zh-hans/HTTP%E5%A4%B4%E5%AD%97%E6%AE%B5)
6. [维基百科：HTTP状态码](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)
7. [W3Cschool HTML 教程](https://www.w3cschool.cn/html/)
8. [维基百科：无状态协议](https://zh.wikipedia.org/zh-hans/%E6%97%A0%E7%8A%B6%E6%80%81%E5%8D%8F%E8%AE%AE)