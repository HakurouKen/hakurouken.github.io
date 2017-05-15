---
title: Vue 服务器渲染
tags:
  - javascript
  - vue
date: 2017-04-22 23:24:04
---

Vue 2.0 之后引入了服务端渲染（Server-Side Render，简称 SSR）的支持。我们可以采用直出的方式，对一些对首屏时间和 SEO 有要求的页面进行优化。

<!-- more -->

## 预渲染与服务器渲染
官方文档中在某些情况下，我们可以利用 Webpack 的插件 [prerender-spa-plugin](https://github.com/chrisvfritz/prerender-spa-plugin)来进行预渲染，从而覆盖一部分的 SSR 需求。这个插件的配置很简单，只要在 webpack 中引入这个插件，并指定需要预渲染的页面路径即可。它的原理更是简单粗暴，就是用 phantomjs 加载对应的页面然后将生成的对应 HTML 保存下来。如果页面用到了 Ajax 获取数据，我们可以指定在 DOM 事件触发后或指定的 DOM 元素出现后再抓取页面。当然我们也可以简单的指定一个延时，但如果网络波动导致请求返回超时，生成的页面可能有所不同，所以不推荐。

然而，预渲染实际生成的是**静态页面**，无法适用于动态数据生成的页面。换句话说，**能使用预渲染解决的问题，几乎都可以使用直接写静态页面的方式解决**。个人认为，多数情况下，预渲染都是个比较鸡肋的功能。

## vue-server-renderer
有关服务器渲染，官网已经给了[一个简单的例子](https://cn.vuejs.org/v2/guide/ssr.html#通过Express-Web服务器实现简单的服务端渲染)，总结下来大致分为3步：
1. 创建一个 Vue 实例。
这一步和在浏览器中使用 Vue 没什么区别，只是在服务端，我们不需要`$mount`方法绑定到 DOM 元素上。

2. 创建一个渲染器(renderer)。
使用`createRenderer`或`createBundleRenderer`创建一个渲染器。一般推荐使用`createBundleRenderer`，因为它支持在开发环境中的热重载。

3. 将 Vue 组件渲染成 HTML。
renderer 提供了两个 API，我们可以选择用`renderToString`渲染成字符串，或用`renderToStream`处理成流。剩下的就交由 Web Server 处理即可。一般情况下，我们会进行一些封装，把这一步放在中间件处理。

## 服务器端 Webpack
上述例子中，避开了一些 webpack 打包的细节，如果我们使用了单文件组件(.vue 文件)，那么，服务端打包是一个不得不谈的话题。具体的配置，我们可以参照 [vue-hackernews-2.0](https://github.com/vuejs/vue-hackernews-2.0/blob/master/build/webpack.server.config.js) 和 [Nuxt.js](https://github.com/nuxt/nuxt.js/blob/master/lib/webpack/server.config.js) 的服务端配置，这里只简单介绍一些注意点。

1. `vue-server-renderer/server-plugin` 和 `vue-server-renderer/client-plugin` 两个插件，可以自动帮我们生成`createBundleRenderer`时用到的 serverBundle 和 clientManifest 两个配置，因此必须要引入。
2. 服务端**不要**使用 CommonsChunkPlugin，我们不需要提取公共部分。
3. 我们一般不想把 node_modules 里的库也打包进来，因此把这些库全部用`externals`引入。一般采用的方式是直接遍历`package.json`里的`dependencies`(和`devDependencies`) 或 node_modules 文件夹来生成 externals 配置。当然，也可以用第三方的插件`webpack-node-externals`，但是原理也类似。
4. 还有不少细节，比如`target`要改成`node`，`output`的`libraryTarget`一般配成`commonjs2`等等，就不再一一举了。

## 可能会遇到的问题
### window/document 对象未定义
在浏览器代码中，我们经常需要操作 BOM/DOM ，这里会用到只在浏览器中才有的 window/document 对象，在 Node.js 中运行就会出现异常。

如果这个问题出现在我们自己的组件中，解决方案相对简单：多数情况下我们需要操作 BOM/DOM 时，元素都已经成功 mount (比如用到`vm.$el`或者`vm.$refs`)，而服务器渲染的生命周期只会进行到 created，因此只需要将相关的逻辑直接丢到 mount 钩子即可。

需要注意的是，如果你用到了一些第三方库，它们可能是专为浏览器设计的，因此可能会在库初始化时就会用到 window/document 变量（比如[flatpickr](https://github.com/chmln/flatpickr)）。使用这种第三方库，可以考虑采用下列步骤：
- 在 webpack 配置中，用一个`DefinePlugin`变量来区分服务端和客户端。
- 自己用一个 wrapper 包装这个库，在浏览器端返回原本的库，在服务端，返回一个**保证代码能够正常运行的 polyfill**。还以`flatpickr`为例，我们的 wrapper 大概长这样（假设我们用`process.IS_BROWSER_BUILD`来标识在浏览器中的打包）：

```javascript
let Flatpickr = {}
if (process.IS_BROWSER_BUILD) {
  Flatpickr = import('flatpickr')
  require('flatpickr/dist/flatpickr.css')
  require('flatpickr/dist/themes/airbnb.css')
  const {zh} = import('flatpickr/dist/l10n/zh')
  Flatpickr.localize(zh)
}

export default Flatpickr
```
- 所有用到这个第三方库的地方，都用我们自己这个 wrapper 替代。

### DOM 操作相关
我在使用 Vue 的 SSR 功能时，遇到最多的就是这个报错：
```
[Vue warn]: The client-side rendered virtual DOM tree is not matching server-rendered content. This is likely caused by incorrect HTML markup, for example nesting block-level elements inside <p>, or missing <tbody>. Bailing hydration and performing full client-side render.
```
这个报错的原因是因为服务器渲染出的 html 和客户端 Vue 生成的 DOM 树（VNode 树）结构不一致，开发版本的 Vue 会给出报错。这个错误导致的原因除了 warning 消息中提到的浏览器自动修正不规范 DOM 之外，更多的是一些修改 DOM 的 directive 导致的。官网对此给出了两个修改意见：
1. 修改 directive ，使其工作以 Virtual-DOM 的方式工作。但是这个改造成本较高，而且对于第三方依赖库，改造难度也相对较高。
2. 额外开发一个服务端用的 directive 版本，然后按照类似上文的处理方式引入。改造方式可以参照 [v-show](https://github.com/vuejs/vue/blob/dev/src/platforms/web/server/directives/show.js) 的服务端版本。原则上，这个服务器版本应当尽量简单，甚至不需要覆盖原有 directive 原有逻辑，只需要保证在初始化时渲染结果和原来一致即可。

## Nuxt.js
如果是一个全新的项目，可以考虑使用 [Nuxt.js](https://github.com/nuxt/nuxt.js)。这是一个类似 react 的 [next.js](https://github.com/zeit/next.js) 的项目，也收到 Vue 官方的推荐。这里简单说说它的优点：
1. 完善的脚手架，减少了项目初始化的成本。它同时还有搭配各个服务器使用的中间件版本。
2. 预置一些实用的 mixin ，例如`asyncData`,`fetch`,`head`,`transition`等等，可以简化开发时的繁琐操作。
3. 基于路径的路由系统的设计，使得路由在多数情况下非常直观，而且易于配置。
4. 在保证了易用性的同时，扩展性也不错，我们可以方便的通过 extend 方法，来扩展自己的 webpack 配置。

## 参考资料
1. [Vue prerenders](https://github.com/chrisvfritz/prerender-spa-plugin)
2. [Vue.js Server-Side Rendering Guide](https://ssr.vuejs.org/en/)
3. [Vue 服务端渲染](https://cn.vuejs.org/v2/guide/ssr.html)
4. [vue-hackernews-2.0](https://github.com/vuejs/vue-hackernews-2.0)
5. [Nuxt.js](https://nuxtjs.org)
