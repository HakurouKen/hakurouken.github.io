---
title: Web 前端常见的 HTTP 相关面试题
tags:
  - javascript
  - http
  - 面试
---

## TCP 三次握手 四次挥手

## 3xx 重定向

### 301 Moved Permanently

一般用于资源永久转移时使用，默认情况下（不指定 Cache-Control 头的情况），浏览器会缓存 301 请求。同时 301 也是 SEO 友好的跳转方式，搜索引擎会转移所有原页面的权重到新的页面。**只有 GET 和 HEAD 的请求 301 响应，浏览器才会重定向**，否则禁止重定向。

### 302 Found

临时重定向，默认不缓存，除非显式指定 Cache-Control 。按照规范，302 只对 GET 和 HEAD 请求进行重定向，其余的请求(例如 POST)除非得到用户的确认，否则不自动重定向。对于非 HEAD/GET 请求，很多老浏览器的实现并不规范（很多是按照 303 实现的）。因此，在 HTTP 1.1 中，新增了 303 和 307 两个状态码。

### 303 See Other

HTTP 1.1 新增，与 302 类似，不同的地方在于，对于 POST/DELETE/PUT 等方法，使用 GET 重定向。这个场景主要用于 **发送表单后，重定向到新的资源**。

### 307 Temporary Redirect

HTTP 1.1 新增，类似 302 类似，不同的地方在于，所有重定向请求，维持原 HTTP 方法.

### 参考资料与扩展阅读

1. [维基百科: HTTP 状态码](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81#3xx%E9%87%8D%E5%AE%9A%E5%90%91)
2. [你所知道的 3xx 状态码](https://aotu.io/notes/2016/01/28/3xx-of-http-status/index.html)

## HTTP 请求/响应格式

### HTTP 请求

1. 请求行，包括 请求方法、URL、协议版本
2. 请求头
3. 请求体

### HTTP 响应

1. 状态行
2. 消息头
3. 消息正文

### 参考资料与扩展阅读

1. [HTTP 消息结构](http://www.runoob.com/http/http-messages.html)
2. [HTTP Headers](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)

## 缓存

### Cache-Control

Cache-Control 是用于控制缓存的最重要的一个 HTTP 头。**虽然多数情况下，我们都是在服务器返回的响应头中使用，但它也支持 HTTP 的请求头**。

#### no-store(禁止缓存)

no-store 代表完全不缓存，一般用于隐私数据之类的场景。

#### no-cache(强制确认缓存)

允许客户端缓存，但在使用前，必须向服务端进行确认。如果服务器确认可使用缓存（返回 304），即可使用缓存。

与 no-cache 类似的，HTTP 1.0 的请求头中有一个 Pargma，它仅有一个可选值 no-cache，且仅支持请求头（因此并不常用）。

#### max-age

max-age 的属性值是“可缓存时间(秒)”，格式大概为 `Cache-Control: max-age=86400`。和这个属性类似的，HTTP 1.0 协议中，有一个类似的 Expires 头，返回的是时间，它代表着"缓存在此时间之前都有效"。但是客户端的时间往往可以被手动调整，因此并不准确。Expires 一般仅作为向后兼容使用。

#### private/public

private 代表此请求是用户相关的，任何的中间节点不允许缓存，只有客户端可以缓存。

public(默认) 代表可以被传输中的任何节点（中间代理/运营商/CDN 等）缓存。一般用于缓存各种公有资源，例如 js、css 等等。

### 强缓存

Cache-Control 给的 max-age 未到过期时间，此时浏览器不会发起请求，调试控制台会显示 200 (from cache)。在 Chrome 中，缓存分为内存缓存和硬盘缓存两类(from disk cache / from memory cache)，这两类缓存和服务器无关，完全取决于浏览器自己的策略。浏览器内部对于缓存的逻辑非常复杂，但是大原则上，内存缓存一般是和渲染进程（即浏览器 TAB）绑定的。

### 协商缓存

在服务器端下发的 max-age 已经到期之后，浏览器会和服务器端进行“协商缓存”。当协商成功的情况下，服务器端只返回 304，只有 HTTP 头，内容由客户端从缓存中读取。

协商缓存涉及到两组重要的 HTTP 头：Last-Modified/If-Modified-Since 和 Etag/If-None-Match。它们的使用方式大致如下：

Last-Modified：Response 头中，返回上次修改时间
If-Modified-Since：Request 头中，上报上次返回的修改时间(Last-Modified)。服务器通过对比它和真实的文件修改时间，判断资源是否修改，如果没有修改，返回 304，否则返回 200 。

Etag：Response 头中，返回文件签名
If-None-Match: Request 头中，带上缓存的文件签名(Etag)。服务器通过对比它和真实的文件签名，判断资源是否修改，如果没有修改，返回 304，否则返回 200 。

### 参考资料与扩展阅读

1. [MDN: HTTP 缓存](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching_FAQ)
2. [你应该知道的浏览器缓存知识](https://excaliburhan.com/post/things-you-should-know-about-browser-cache.html)
3. [MDN: Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)
