---
title: Web 前端常见的 HTTP 相关面试题
tags:
  - javascript
  - http
  - 面试

date: 2019-04-28 23:11:07
---

<!-- more -->

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

## CORB

### CORB 的表现

CORB(Cross-Origin Read Blocking，屏蔽跨域读写)是浏览器加载。如果我们的请求被 CORB 拦截了，会在浏览器中有类似下列的提示（根据浏览器版本不同，提示会有一些不同）：

```
Cross-Origin Read Blocking (CORB) blocked cross-origin response https://www.example.com/example.html with MIME type text/html. See https://www.chromestatus.com/feature/5629709824032768 for more details.
```

当一个请求被 CORB 拦截时，它的请求体和请求头都会被置为空（在版本低一些的 Chrome 版本中，会保留部分 HTTP 头）。

### 为什么要有 CORB

事实上，由于有 CORS/CSP 规则的存在，我们的应用层并不会拿到第三方的数据，但是他们仍然会被加载到内存里。利用 Intel 的幽灵熔断漏洞（硬件级别），黑客可以进行旁路攻击，来越界获取到我们内存中的信息。为了增加漏洞的利用成本，Chrome 引入了 CORB，将不合法的返回直接置空。有关这个漏洞，可以参考下面这个视频（如果视频不能加载，可以[点击这里](https://www.bilibili.com/video/av18144159/)）：

<iframe src="//player.bilibili.com/player.html?aid=18144159&cid=29622092&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"></iframe>

### 受 CORB 保护的内容

在[Chromium 的官方说明中](https://chromium.googlesource.com/chromium/src/+/master/services/network/cross_origin_read_blocking_explainer.md#Determining-whether-a-response-is-CORB_protected)中，对受保护的内容有详细的描述，不精确的简单概括如下：

1. 如果响应头中有`X-Content-Type-Options: nosniff`，则直接使用 Content-Type 来判断，否则浏览器会根据 `Content-Type` 和响应体来“嗅探”。
2. 受保护的类型有 html/xml/json/普通文本(text/plain).

### CORB 的影响

事实上，我们的业务几乎不需要对 CORB 做针对性处理，因为绝大多数场景都不受影响，包括但不限于：

1. 正常的跨域请求的图片/资源。
2. 普通的 XHR/fetch 请求。屏蔽掉非法的 CORS 请求会被屏蔽掉，但原本这些请求在 javascript 中就无法获取。
3. 上报请求（例如使用 img 标签来进行上报），这种情况由于我们本来就不需要回包，所以不关注。
4. Blob 和 File API. 在没有 CORB 的情况下，这些跨域请求也会被屏蔽。

### 参考资料与扩展阅读
1. [Cross-Origin Read Blocking (CORB)](https://chromium.googlesource.com/chromium/src/+/master/services/network/cross_origin_read_blocking_explainer.md)
2. [CORB for Developer](https://www.chromium.org/Home/chromium-security/corb-for-developers)
3. [30 分钟理解 CORB 是什么](https://segmentfault.com/a/1190000016126079)

## gzip

### gzip 启用过程

1. 浏览器端在 HTTP 头中，带上 HTTP 头 `Accept-Encoding: gzip`，表明自己支持 gzip. Accept-Encoding 除了 gzip 之外还支持其它的压缩格式。
2. 服务器端检则到客户端支持 gzip，启用 gzip 压缩，并传输给浏览器，并在响应头追加当前编码格式`Content-Encoding: gzip`。
3. 浏览器根据 `Content-Encoding` 的值进行解包。

### gzip 压缩原理

gzip 默认采用 Deflate 算法来压缩字符串。deflate 算法可以简单理解为先使用 LZ77 压缩，再使用哈弗慢编码压缩。

### nginx 中开启 gzip

```conf
# 开启
gzip on;

# 压缩等级，1-9。值越大，压缩率越高
# 但对应的压缩/解压负载会变大
# 对于静态文件，经验值是 3 或 4
gzip_comp_level 3;

# UA 检测，针对特定 UA 禁用 gzip
gzip_disable regex ...

# 最小压缩文件长度
gzip_min_length 20;

# 使用 GZIP 压缩的最小 HTTP 版本
gzip_http_version 1.1;

# 针对指定 MIME 类型进行压缩
gzip_types text/html text/css application/javascript;
```

### 参考资料与扩展阅读
1. [简单聊聊 GZIP 的压缩原理与日常应用](https://juejin.im/post/5b793126f265da43351d5125)
2. [你真的了解 gzip 吗？](https://zhuanlan.zhihu.com/p/24764131)
3. [MDN: Accept-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding)
4. [MDN: Content-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding)

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
1. [HTTP Method详细解读(`GET` `HEAD` `POST` `OPTIONS` `PUT` `DELETE` `TRACE` `CONNECT`)](https://www.cnblogs.com/machao/p/5788425.html)
2. [HTTP协议详解（一）](https://www.jianshu.com/p/d7a97cc74203)
3. [MDN: HTTP Methods](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods)

## HTTP 常见头部

### 通用
| HTTP 头    | 描述                     |
| ---------- | ------------------------ |
| Date       | 日期                     |
| Cookie     | -                        |
| Referer    | 页面来源                 |
| Host       | 当前请求针对的服务器域名 |
| User-Agent | 浏览器标识               |

### 安全相关

| HTTP 头                   | 描述                                           |
| ------------------------- | ---------------------------------------------- |
| Content-Security-Policy   | 控制当前的页面允许加载的资源来源               |
| Strict-Transport-Security | 即 HSTS，告诉浏览器只能通过 HTTPS 访问当前资源 |
| X-Frame-Options           | 当前页面是否允许被其它 iframe 嵌套             |

### 编码

| HTTP 头          | 描述                                          |
| ---------------- | --------------------------------------------- |
| Accept           | 浏览器期望的 MIME 类型                        |
| Accept-Encoding  | 浏览器支持的压缩方法(如 gzip)                 |
| Accept-Language  | 浏览器期望的语言                              |
| Content-Encoding | 与 Accept-Encoding 对应，响应的内容的压缩方式 |
| Content-Language | 与 Accept-Language 对应，响应的语言           |
| Content-Type     | 返回的 MIME 类型                              |

### 缓存相关

| HTTP 头           | 描述                                                                       |
| ----------------- | -------------------------------------------------------------------------- |
| Expires           | 缓存过期时间                                                               |
| Pragma            | 值只能是 no-cache，标识不使用缓存                                          |
| Cache-Control     | 缓存控制策略                                                               |
| Last-Modified     | 最后更改时间                                                               |
| If-Modified-Since | 和 Last-Modified 配合使用，服务器根据这个字段对比，判断是返回 200 还是 304 |
| ETag              | 文件签名                                                                   |
| If-None-Match     | 和 ETag 配合使用，服务器根据这个字段对比，判断是返回 200 还是 304          |

### CORS 相关

| 头                               | 描述                                                                   |
| -------------------------------- | ---------------------------------------------------------------------- |
| Origin                           | 源(协议+域名+端口)                                                     |
| Access-Control-Allow-Origin      | 服务器接受请求的域名，和 Origin 一致，或是`*`                          |
| Access-Control-Allow-Credentials | 服务器是否接受 Cookie                                                  |
| Access-Control-Expose-Headers    | 服务器可能返回的响应头列表                                             |
| Access-Control-Request-Method    | 预检请求时使用，标识浏览器的 CORS 用到的 HTTP 方法                     |
| Access-Control-Request-Headers   | 预检请求时使用，标识浏览器会额外带上的 HTTP 头                         |
| Access-Control-Allow-Methods     | 预检请求返回，服务器支持的**所有**支持的 HTTP 方法                     |
| Access-Control-Allow-Headers     | 预检请求返回，服务器支持的**所有** HTTP 头                             |
| Access-Control-Max-Age           | 预检请求返回，标识缓存有效期，类似于 Cache-Control 中的 max-age 的作用 |

### 参考资料与扩展阅读

1. [MDN: HTTP Headers](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)

## HTTP2

### 多路复用

在 HTTP 1.0 中，使用 HTTP 请求一个资源，要经历如下步骤：

1. DNS 查询和解析
2. 建立 TCP 连接
3. 发起 HTTP 请求
4. 收到 HTTP 响应
5. 断开 HTTP 连接

对于每一个 HTTP 请求，都会经历如上几个步骤。显然，这其中有一些重复的操作：每一个 HTTP 请求，都会重复的和服务器建立/销毁连接。因此在 HTTP 1.1 中，提出了“持久连接”(**keep-alive**)的概念：在一定的时间内，对于同一个域名进行多次请求，会复用原来的 TCP 连接，省去创建连接的过程（然而在实际实现中，现代浏览器为了高并发，基本会针对一个域名创建 6~8 个 TCP 连接）。它还同时解决了 TCP 慢启动的问题。

但是， Keep-Alive 仍然有两个无法解决的问题：

1. 对于单个连接，文件仍然是串行的传输。即文件 A 传输时，同一个通道的文件 B 只能处于等待状态，直至文件 A 回包。
2. 由于 keep-alive 是不会及时断开的，所以可能触达服务器最大连接数。

为解决这两个问题，HTTP/2 提出了**多路复用**的概念：引入**二进制数据帧**和**流**的概念，通过每一帧数据都有顺序标识，这样客户端和服务器之间就能通过流来**并行**传输数据了。同时，一个客户端和 server 之间只会存在一个连接(也仅需要一个连接)，服务器同样的连接数限制就可以支撑 6~8 倍的客户端。

### 头压缩

随着页面复杂度的增加，一个页面打开时，往往会发送几十上百个请求，每一个请求都带有一份 HTTP 头。很多时候，我们为了几十个字节的数据，就需要传输几百个字节的头，是一种很大的流量浪费。因此，HTTP2 提出了头压缩,它的实现原理简述如下：

1. 维护一份静态字典，字典中是一个映射表，包含常见的 HTTP 头和 HTTP 头和其对应值的组合。静态字典是协议规定的，映射关系可以参考[规范文档](https://httpwg.org/specs/rfc7541.html#static.table.definition)。
2. 服务器和客户端之间，以**连接**为维度，维护一份动态字典。
3. 基于这两个字典，使用哈夫曼编码进行头压缩：对于匹配到字典中的值或者键值对，可以直接用一个字符替代；同时在这个过程中，动态字典也会得到更新（例如保存整个 UA 串和 Cookie 到动态字典中），使得接下来的请求得到更大效率的压缩。需要注意的是， HTTP/2 的状态行（Method/Path/Status）也可以享受压缩。

### Server Push

在 HTTP/2 中，允许服务器主动推送文件给客户端，省去直接请求的过程。举个例子，我们可以在用户发起 HTML 请求之后，在返回给客户端 html 文件的同时，把 css/js 直接推送给客户端。具体需要推送什么文件，则需要业务 server 自行判断。可以参考 Github 上的这个[示例项目](https://github.com/RisingStack/http2-push-example)。

### 在 HTTP2 下的优化策略调整

1. 在 HTTP 1.x 时代，由于浏览器对单个域名下的请求数有限制（一般是 6~8 个），为了增大并发，我们经常会采用“域名发散”的策略，将静态文件、动态接口尽可能分散在不同的域名下。但是 HTTP2 下，不再存在并发问题，我们应当采用“域名收敛”的策略，尽可能将域名减少，以减少 DNS 解析和 TCP 连接的创建。
2. 在 HTTP 1.x 时代，为了减少请求数，我们往往会采取比较激进的文件合并策略。在 HTTP/2 中，因为有了头压缩和和多路复用，我们可以采取相对宽松的合并策略，以增加缓存的重用度。
3. 因为头压缩的存在，可以在一定程度上替代 Cookie-Free 域名。

### 参考资料与扩展阅读
1. [TCP 慢启动](https://zh.wikipedia.org/wiki/%E6%85%A2%E5%90%AF%E5%8A%A8)
2. [让面试官颤抖，HTTP2.0协议之你应该要准备的面试题](https://cloud.tencent.com/developer/article/1181549)
3. [HTTP/2 头部压缩技术介绍 | JerryQu 的小站](https://imququ.com/post/header-compression-in-http2.html)
4. [HTTP/2 与 WEB 性能优化（二） | JerryQu 的小站](https://imququ.com/post/http2-and-wpo-2.html)
5. [HTTP/2 服务器推送（Server Push）教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2018/03/http2_server_push.html)

## TCP 三次握手与四次挥手

### TCP 三次握手

1. 第一次握手（客户端请求建立连接）：客户端将标识位 SYN 置为 1，生成随机序列(Sequence Number) seq=i，发送给服务端。
2. 第二次握手（服务端确认连接建立）：服务端将标识位 SYN 置为 1，ACK 位置(Acknowledgment Number)为 i+1，生成新的 seq=j，发送给客户端。
3. 第三次握手（客户端确认连接建立）：客户端将标识位 ACK 置为 j+1，生成新的序列号 seq=k，发送给服务端。此次 TCP 传输，允许携带数据帧。

### TCP 四次挥手

TCP 连接的断开可以由服务端或客户端的另一方主动。

1. 第一次挥手（主动方表明自己不再发送数据）：设置 FIN=1，seq=i
2. 第二次挥手（被动方确认自己不会接收数据）：设置 Ack=i+1
3. 第三次挥手（被动方表明自己不再发送数据）：设置 FIN=1，Ack=i+1, seq=j
4. 第四次挥手（主动方确认自己不再接收数据，连接关闭）：Ack=j+1, seq=i+1

四次挥手的额外注意点：

1. 第二次挥手和第三次挥手之间，被动方仍然可能发送数据。
2. 在第三次挥手完成，第四次挥手请求发送后，主动方会等待 2MSL（2个 Maximum Segment Lifetime，即 TCP 连接的生存周期）再关闭连接。这样做是为了**确保被动方能够成功收到 ACK**：第三次挥手实质上是最后一次双向通信的机会，被动方发送 FIN+ACK 后，如果在 2 个 MSL 内没有收到最后一次挥手的信号，会认为之前的 FIN+ACK 可能发送失败了，会重新发送一次。这样，主动方就能在这 2MSL 内等到一个重传的 FIN+ACK，然后再次进行第四次挥手，并重新开始计时。

### 参考资料与扩展阅读
1. [TCP的三次握手与四次挥手（详解+动图）](https://blog.csdn.net/qzcsu/article/details/72861891)
1. [Maximum segment lifetime](https://en.wikipedia.org/wiki/Maximum_segment_lifetime)
