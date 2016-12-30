---
title: Webpack 使用技巧
tags:
    - webpack
    - javascript
---

本文翻译自 Github [rstacruz/webpack-tricks](https://github.com/rstacruz/webpack-tricks)。

这里列举了一些我学到的 Webpack 技巧，所有下列技巧仅适用于 Webpack 1， Webpack 2 的 API 有些变动，因此有些技巧可能无法正常在 Webpack 2 中工作。[这篇文章](http://javascriptplayground.com/blog/2016/10/moving-to-webpack-2/)详细介绍了如何迁移到 Webpack 2。

## 显示进度
执行 Webpack 时添加参数：
```
---progress --colors
```

## 压缩
添加 `-p` 参数执行生产环境构建。
```
webpack -p
```

## 导出多个包
将 output 选项改为`[name].js`，可以导出多个多个包。下面的例子输出`a.js`和`b.js`。
```javascript
module.exports = {
    extry: {
        a: './a',
        b: './b'
    },
    output: { filename: '[name].js' }
}
```
担心公共包重复导出？使用 [CommonsChunkPlugin](https://webpack.github.io/docs/list-of-plugins.html#commonschunkplugin) 将公共部分提取到新的文件。
```javascript
plugins: [ new webpack.optimize.CommonsChunkPlugin('init.js') ]
```
```html
<script src='init.js'></script>
<script src='a.js'></script>
```

## 分割应用代码和库代码
使用 CommonsChunkPlugin 把库代码统一到 `vendor.js`。
```javascript
var webpack = require('webpack')

module.exports = {
  entry: {
    app: './app.js',
    vendor: ['jquery', 'underscore', ...]
  },

  output: {
    filename: '[name].js'
  },

  plugins: [
    new webpack.optimize.CommonsChunkPlugin('vendor')
  ]
}
```
上面的代码做了这些事：
1. 把 `vendor` 作为一个入口，加载一些库文件
2. CommonsChunkPlugin 将会把这些库文件从 app.js 中移除（因为它们现在出现在两个包中）
3. CommonsChunkPlugin 把 Webpack runtime 移入 `vendor.js`
> 相关：[Code splitting](https://webpack.github.io/docs/code-splitting.html#split-app-and-vendor-code)

## Source maps
最好用的 Source Map 插件是 `cheap-module-eval-source-map`，它可以在 Chrome/Firefox 开发工具中现实源文件，而且比 `source-map`和`eval-source-map`。
```javascript
const DEBUG = process.env.NODE_ENV !== 'production'

module.exports = {
  debug: DEBUG ? true : false,
  devtool: DEBUG ? 'cheap-module-eval-source-map' : 'hidden-source-map'
}
```
文件在 Chrome 开发工具中显示为 `webpack:///fool.js?a93h`，我们想让它显示成`webpack://path/to/foo.js`，这样更加清晰易读。
```javascript
output: {
  devtoolModuleFilenameTemplate: 'webpack:///[absolute-resource-path]'
}
```
> 引用：[devtool documentation](https://webpack.github.io/docs/configuration.html#devtool)

## CSS
这个很麻烦。待续。

## 开发模式
想要只在开发环境下使用一些选项？
```javascript
const DEBUG = process.env.NODE_ENV !== 'production'

module.exports = {
  debug: DEBUG ? true : false,
  devtool: DEBUG ? 'cheap-module-eval-source-map' : 'hidden-source-map'
}
```
当构建生产环境组件时,需要这样调用 Webpack：`env NODE_ENV=production webpack -p`。
