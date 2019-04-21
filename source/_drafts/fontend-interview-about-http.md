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

## 请求跨域

### JSONP

JSONP 的本质是动态插入 script 标签，因此仅支持 GET 请求传递参数，不适合于做用户相关的请求。因为涉及到直接插入 script 标签并执行，因此它也是 XSRF 的重灾区，必须使用 CSP 策略严格限制来源。

### 服务端反向代理

避免跨域问题，最简单的方式就是不跨域。我们可以简单的使用一个反向代理即可完成。以 nginx 为例，如果我们要把 www.example.com 域名下 `/api` 路径下所有的请求，代理到 api.example.com 下，nginx 配置大致如下：

```
server {
  listen 80;
  server_name  www.example.com;
  location /api {
    proxy_pass_header Server;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_pass http://api.example.com;
  }
}
```

### CORS

#### CORS 的基本过程

1. 在请求中携带 `Origin` 头，它的值包含了当前的协议路径信息（例如：`https://example.com/`），由浏览器自动设置，使用 javascript 无法更改。
2. 服务器读取 `Origin` 头，如果允许跨域，根据条件返回 `Access-Control-Allow-Origin`, 如果允许访问，可以设置为和来源一致或者 \* 。否则，不返回 `Access-Control-Allow-Origin` 的头。

如果想要携带凭证(Cookie)，还需要额外做两个事情：

1. 客户端的 XHR 请求，设置 `withCredentials = true` 开关。
2. 服务端返回 `Access-Control-Allow-Credentials: true` 的 HTTP 响应头。
3. `Access-Control-Allow-Origin` 不允许为 \*，必须和请求的 Origin 一致

注意，上述基本流程，仅适用于**简单请求**。

#### 简单请求

一个简单请求，必须满足下列所有条件：

1. GET/POST 请求
2. 没有自定义 HTTP 头
3. Content-Type 只能是 application/x-www-form-urlencoded, multipart/form-data, text/plain

标准如此规定，主要是为了向后的兼容性。

#### 预检请求(CORS-preflight request)

如果不是简单请求，那么在真实请求发送前，浏览器会先行发送一个预检请求，用于询问服务器，可以使用哪些 HTTP 方法和头信息。

预检请求是一个 OPTIONS 请求，除了 `Origin` 头之外，还包括 `Access-Control-Request-Method` 和 `Access-Control-Request-Headers`（可选） 两个头，分别代表当前使用的 HTTP 方法（例如 POST、DELETE）和额外携带的 HTTP 头信息。

与简单请求类似，如果服务器如果不允许跨域，返回的请求中不带 `Access-Control-Allow-Origin` 头即可。否则，返回的头会有如下额外字段：

1. `Access-Control-Allow-Origin`: 和请求的 `Origin` 一致，或者是 \* （与简单请求一致）
2. `Access-Control-Allow-Credentials`: 可选，是否允许发送 Cookie。如果有这个头，值只能为 `true` （与简单请求一致）
3. `Access-Control-Allow-Methods`：逗号分割的字符串，标识**所有**支持跨域请求的方法（不限于浏览器发出的特定方法），避免发出多次预检请求。
4. `Access-Control-Allow-Headers`: 可选，对应请求中的 `Access-Control-Request-Method`，标识**所有**支持的头信息字段。
5. `Access-Control-Max-Age`：可选，标识预检请求的有效期。类似于 `Cache-Control` 的 max-age。

预检请求完成后，后续发送的真实请求，和简单请求一致。

### 参考资料与扩展阅读

1. [MDN: Origin](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Origin)
2. [MDN: Server-Side Access Control](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Server-Side_Access_Control)
3. [MDN: HTTP 访问控制（CORS）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)
4. [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)

## HTTP 方法

|  方法   | 初次出现的 HTTP 版本 | 描述                                                                                                                                                                                                         |
| :-----: | :------------------: | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|   GET   |       HTTP 0.9       | 只用于获取资源，一般要求是幂等且安全无副作用的，这样浏览器可以对同样路径和参数的资源进行缓存。                                                                                                               |
|  POST   |       HTTP 1.0       | 向指定的资源位置提交数据，一般有副作用，可能创建/删除/修改对应的资源。                                                                                                                                       |
|  HEAD   |       HTTP 1.0       | 和 GET 请求类似，也用于获取资源，不同的是，它只需要返回请求的头部。常用于获取数据元信息（例如图片大小），或是检测 URL 的合法性等。                                                                           |
| OPTIONS |       HTTP 1.1       | OPTIONS 方法用于获取资源请求/响应相关的通信选项。例如预检请求就是一个 OPTIONS 请求。                                                                                                                         |
| DELETE  |       HTTP 1.1       | 请求删除对应的资源。                                                                                                                                                                                         |
|   PUT   |       HTTP 1.1       | 请求服务器把当前的数据实体保存到指定资源位。                                                                                                                                                                 |
|  TRACE  |       HTTP 1.1       | 要求服务器原样返回任何客户端请求的内容（可能会附加路由中间的代理服务器的信息）。一般仅用于调试。由于 HTTP 头中有 Cookie 等敏感信息，所以可能用于欺骗合法用户并获取其隐私信息。一般建议服务器直接关闭此方法。 |
| CONNECT |       HTTP 1.1       | CONNECT 方法用于建立一个到由目标资源标识的服务器的隧道。不常用。                                                                                                                                             |
|  PATCH  | HTTP 1.1 (RFC 5789)  | 用于部分修改指定资源。                                                                                                                                                                                       |

补充说明：

1. 幂等方法：对于同一个 URL，使用同样的参数，进行一次或多次操作，造成的副作用是一样的。
2. 所有 HTTP 方法，从定义上的作用对象都是 URI，即资源。从 HTTP 回包的结构也能看出：HTTP 的请求行只包括“请求方法”、“请求 URL”、HTTP 协议版本三个部分。例如 RESTFUL 规范是一种将所有 URI 视为资源，并充分利用 HTTP 方法（作为动词）的规范。但是以资源的方式组织数据有一定的弊端（例如字段冗余、多端点往往需要多请求等）。
3. 所有的 HTTP 方法的作用，都是**约定**，具体的实现是由业务控制，协议本身并没有强制要求。例如，即使是在 HTTP 1.1 协议中，我们依然可以使用 POST 来替代 DELETE/PUT/PATCH 的行为。

参考资料及扩展阅读：
1. [HTTP Method详细解读(`GET` `HEAD` `POST` `OPTIONS` `PUT` `DELETE` `TRACE` `CONNECT`](https://www.cnblogs.com/machao/p/5788425.html)
2. [HTTP协议详解（一）](https://www.jianshu.com/p/d7a97cc74203)
3. [MDN: HTTP Methods](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods)

