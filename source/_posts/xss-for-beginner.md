---
title: 浅析 XSS
tags:
  - javascript
  - xss
date: 2019-03-18 01:44:14
---

本文主要简单介绍 XSS 的一些基本注入场景和如何防御 XSS，主要面向对 Web 安全不太了解的同学。

<!-- more -->

## 一个简单的 XSS 实例

考虑这样一个产品需求：我们需要做一个帖子（纯文本）的搜索的结果页面，需要在前端高亮所有"完全匹配" 的字符。我们 Vue 为例，写一段 DEMO 代码：

```html
<template>
  <div class="content" v-html="highlighted"></div>
</template>

<script>
  export default {
    name: 'FeedsContent',
    props: ['content', 'keyword'],
    computed: {
      highlighted() {
        return this.content
          .split(keyword)
          .join(`<span style="color:red">${keyword}</span>`);
      }
    }
  };
</script>
```

如果用户输入的 keyword 为`<script>alert(1)</script>`，我们就可以在浏览器内看到弹出的 alert 对话框。这意味着我们的代码有 XSS 注入漏洞。实际场景下，黑客可以把我们的 alert 替换成一些危险代码，例如“将当前用户的 Cookie 发送到指定的服务器”等等。

## XSS 的分类

习惯上，大家一般将 XSS 分为 `DOM-based XSS`，`反射型（非持久型） XSS` 和`存储型（持久型） XSS`。但是这个分类有些不严谨：DOM-based XSS 是指浏览器端的 Javascript 操作 HTML 不当产生的注入；而反射型/存储型对应着后台是否有落地存储，二者的维度并不一致，因此分类有重叠。我们不必去过分深究一个 XSS 到底属于哪种类型，只要知道他们的原理以及如何防御即可。

### DOM-based XSS

我们开头举例的搜索的 XSS 问题，就是一个典型的 dom xss。它的产生和 SQL 注入类似：SQL 语句和 HTML 文档都是 **使用字符串表示的结构化文本**，在字符串拼接的过程中，黑客可以通过一些特殊的文本注入，来干扰原来的结构，从而使逻辑发生变化。狭隘的说，DOM-based XSS 就是 innerHTML 操作时，没有正确过滤用户输入。这里的输入，不仅仅包括，甚至还包括 `document.referrer` 和 `window.name` 等等这些可以从外部干预的值。事实上，在目前流行的“前后端分离”的开发模式下，绝大部分的 XSS 都是 DOM-based XSS。

### 反射型 XSS

反射型 XSS 是一个很形象的说法，它的成因一般是 **服务端未将用户输入进行转义/过滤（或转义有漏洞），直接“反射”到输出内**。例如下面这个 DEMO：

```php
<?php
$name = $_GET["name"];
?>
<p>名称为<?php echo $name; ?>的搜索结果：</p>
<ul>
  <li>...</li>
  <!-- 省略无关代码 -->
</ul>
```

反射型 XSS 的数据，多数是包含在 URL 的 GET 参数内的。攻击者一般会使用“诱导点击”的方式，来引诱用户去点击对应的链接。所有注入的 XSS 代码，只会出现在当前的 Query 请求下，不会影响到其余正常访问的用户。

### 存储型 XSS

顾名思义，存储型 XSS 就是将 XSS 注入的数据存储到了数据库等持久存储中。一般情况下，发帖/评论回复等需要用户提交信息的地方，都是存储型 XSS 的重灾区，需要重点关注。

另外，DOM-based XSS 和反射型 XSS 都有相对成熟的工具来进行扫描，而存储型 XSS 不同，它比较难通过自动化扫描工具发现的，因为注入点和输出点往往不在一起，且与业务逻辑相关（对应不同的后台接口/页面，例如发表评论和查看评论）。同时，它也是危害最大、能够波及用户最多的。

## XSS 的防御

### CSP

XSS 的目的无非是为了窃取用户信息，为了达到这个目的，必须和第三方通信。基于这个特性，我们可以通过 CSP（Content-Security-Policy）来限制被信任的域，这样可以直接限制了外部通信，用户无法加载第三方脚本，也无法向第三方的域名传递信息，这样就起到了防御 XSS 的作用。有关 CSP 的详细配置，[MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) 上已经有很详细的说明和示例，这里不再展开细说。

**注意，CSP 并没有解决 XSS 注入，但是可以在一定程度上减少 XSS 注入的危害。** 如果黑客只是单纯的想搞破坏，而不是盗取用户的信息，例如让对应的用户“调用内部接口发送一条微博”这种行为，CSP 就无能为力了。

### http-only

将 Cookie 设置为 http-only，也能够防止 Cookie 的明文被 javascript 读取到，从而避免了黑客直接拿到 cookie 的明文信息。但是同样的，它也没有解决“注入脚本”这个根本问题。

### 转义

转义是解决 XSS 的根本方案，唯一

1. 前端对于使用 javascript 插入的信息，如果是非富文本的场景，一律使用 HTML 实体。即在 Vue/React 中，不使用 `v-html`/`dangerouslySetInnerHTML`。
2. 对于富文本场景，后台使用成熟的 XSS 过滤库，进行 html 标签和 html 属性的过滤，仅放行白名单标签和属性。一般情况下，白名单策略要优于黑名单，因为 HTML 。另外，由于 HTML 解析有非常多的 edge case，应当使用成熟的 XSS 过滤库，尽量不要自造轮子。
3. 对于后端模板输出的 HTML，要根据具体的输出位置进行过滤，尽量采用成熟的模板引擎，避免直接的字符串拼接，尤其是涉及到用户产生的内容。

## 扩展阅读

1. [维基百科的 XSS 词条](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%B6%B2%E7%AB%99%E6%8C%87%E4%BB%A4%E7%A2%BC)
2. [MDN 的 CSP 文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
3. [OWASP: XSS 注入文本的 cheatsheet](https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet)
4. [Github: Awesome XSS](https://github.com/s0md3v/AwesomeXSS)
