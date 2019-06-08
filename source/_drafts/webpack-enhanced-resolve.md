---
title: Webpack enhanced-resolve 详解
tags:
  - javascript
  - webpack
---

webpack 使用 [enhanced-resolve](https://github.com/webpack/enhanced-resolve) 来进行模块解析。它的作用类似于一个异步的 `require.resolve` 方法，将一个 `require/import` 的语句中的引入字符串，解析为引入文件的绝对路径。

本文以为 `enhanced-resolve` 的 4.1.0 版本的代码为例，

### 概要说明

`enhanced-resolve` 的官方文档中，将自己描述为”高度可配置的“，这得益于它完善的插件系统。事实上，`enhanced-resolve` 的所有内置功能，也基本是通过插件来实现的。从[源代码的目录](https://github.com/webpack/enhanced-resolve/tree/v4.1.0/lib) 也可以看出，所有 Plugin 结尾的文件，均为内置插件。webpack 配置中的 `resolve` 中的配置，和 `enhanced-resolve` 的 resolver options 是一一对应的，这些选项决定

函数的入口是 [lib/node.js](https://github.com/webpack/enhanced-resolve/blob/v4.1.0/lib/node.js)，它只是简单的调用了 `ResolverFactory.createResolver` 方法，生成了一个 `Resolver` 对象，并将它的 `resolve` 方法暴露给外部。

`ResolverFactory.createResolver`（源代码见[lib/ResolverFactory.js](https://github.com/webpack/enhanced-resolve/blob/v4.1.0/lib/ResolverFactory.js)），从名字就可以看出，它就是 `Resolver` 的工厂方法，它大致做了如下几个事情：

1. 生成一个 `Resolver` 对象
2. 定义一系列生命周期的钩子
3. 将所有外部传入的配置格式化，并根据配置加载不同的内置插件

`Resolver`（源代码见[lib/Resolver.js](https://github.com/webpack/enhanced-resolve/blob/v4.1.0/lib/Resolver.js)）本身是一个 `tapable` 的子类，
从[源代码](https://github.com/webpack/enhanced-resolve/blob/635c2c7e33910bb89845bbeb8ef2c4eda36527f2/lib/ResolverFactory.js#L154-L167) 可以看到所有外部插件可以使用的钩子。

而所有的内部插件的运行机制，就和 webpack 插件类似：首先通过 `resolver.getHooks` 声明挂载钩子（取得`tapable`对象），然后使用 `tap` 相关方法(例如 `tapAsync`) 注册钩子回调。为了方便的维持调用栈和提示报错信息，我们不直接调用 callback，而是通过 `resolver.doResolve` 来间接调用。可以参考 [AliasPlugin](https://github.com/webpack/enhanced-resolve/blob/v4.1.0/lib/AliasPlugin.js).

### 插件的基本结构

内部插件的结构大致如下：

```javascript
module.exports = class SomeInternalPlugin {
  constuctor(source, options, target) {
    this.source = source;
		this.options = options;
		this.target = target;
  }

  apply(resolver) {
    const target = resolver.ensureHook(this.target);
    resolver.getHook(this.source).tapAsync('SomeInternalPlugin', (request, resolveContent, callback) => {
      // 省略插件的具体逻辑
      // ...
      return resolver.doResolve(target, request, 'some resolve message', resolveContext, callback)
    })
  }
}
```

其中 `source` 和 `target` 是将零散的插件逻辑串联成流程的关键：插件逻辑通过 `resolver.getHook` 将自己的逻辑注册到 `source` 的钩子上，然后再使用 `resolver.doResolve` 调用 `target` 对应的钩子。为了这个目的，内部还实现了 [`NextPlugin`](https://github.com/webpack/enhanced-resolve/blob/v4.1.0/lib/NextPlugin.js) 和 [`TryNextPlugin`](https://github.com/webpack/enhanced-resolve/blob/v4.1.0/lib/NextPlugin.js) 两个”空插件“（这两个插件也是内部插件中的“最简结构”）。

### 第三方插件

第三方插件和内部插件的结构基本是一致的，除了我们不需要支持传入 `source` 和 `target` ，在插件内部控制即可。

所有可以使用的钩子名称，我们可以直接在[源代码](https://github.com/webpack/enhanced-resolve/blob/635c2c7e33910bb89845bbeb8ef2c4eda36527f2/lib/ResolverFactory.js#L154-L167)中找到，我们还可以通过给这些钩子添加 `before-` 或 `after-` 前缀来提高/降低当前插件的执行优先级。一个最简单的插件实现，我们可以参考 [directory-named-webpack-plugin](https://github.com/shaketbaby/directory-named-webpack-plugin)。

### 参考资料

1. [webpack/enhanced-resolve: Offers an async require.resolve function. It's highly configurable.](https://github.com/webpack/enhanced-resolve)
2. [Webpack 官方文档：Module Resolution](https://webpack.js.org/concepts/module-resolution/)
3. [Webpack 官方文档：Resolve](https://webpack.js.org/configuration/resolve/)
4. [Webpack 官方文档：Plugin](https://webpack.js.org/api/plugins/)
5. [Webpack 官方文档：Writing a Plugin](https://webpack.js.org/contribute/writing-a-plugin/)
6. [[webpack] enhanced-resolve · 语雀](https://www.yuque.com/bartonding/webdev/enhanced-resolve)
7. [webpack4.0源码分析之Tapable - 掘金](https://juejin.im/post/5abf33f16fb9a028e46ec352)
8. [shaketbaby/directory-named-webpack-plugin: A Webpack plugin that treats a file with the name of directory as the index file](https://github.com/shaketbaby/directory-named-webpack-plugin)
9. [webpack/tapable: Just a little module for plugins.](https://github.com/webpack/tapable)
