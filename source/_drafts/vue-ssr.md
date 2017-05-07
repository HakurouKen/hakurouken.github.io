---
title: Vue 服务器渲染
date: 2017-04-22 23:24:04
tags:
  - javascript
  - vue
---

Vue 2.0 之后引入了服务端渲染（Server-Side Render，简称 SSR）的支持。我们可以采用直出的方式，对一些对首屏时间和 SEO 有要求的页面进行优化。

## 预渲染与服务器渲染
官方文档中在某些情况下，我们可以利用 Webpack 的插件 [prerender-spa-plugin](https://github.com/chrisvfritz/prerender-spa-plugin)来进行预渲染，从而覆盖一部分的 SSR 需求。这个插件的配置很简单，只要在 webpack 中引入这个插件，并指定需要预渲染的页面路径即可。它的原理更是简单粗暴，就是用 phantomjs 加载对应的页面然后将生成的对应 HTML 保存下来。如果页面用到了 Ajax 获取数据，我们可以指定在 DOM 事件触发后或指定的 DOM 元素出现后再抓取页面。当然我们也可以简单的指定一个延时，但如果网络波动导致请求返回超时，生成的页面可能有所不同，所以不推荐。

然而，预渲染实际生成的是**静态页面**，无法适用于动态数据生成的页面。换句话说，**能使用预渲染解决的问题，几乎都可以使用直接写静态页面的方式解决**。个人认为，多数情况下，预渲染都是个比较鸡肋的功能。

## vue-server-renderer

## 服务器端 Webpack

## Nuxt.js

## 参考资料
1. [Vue.js Server-Side Rendering Guide](https://ssr.vuejs.org/)
2. [Vue 服务端渲染](https://cn.vuejs.org/v2/guide/ssr.html)
3. [vue-hackernews-2.0](https://github.com/vuejs/vue-hackernews-2.0)
4. [Vue prerenders](https://github.com/chrisvfritz/prerender-spa-plugin)
