---
title: 优化 Node.js 中的深层 require 路径
tags:
---

随着 Node.js 应用规模的增长，我们经常会将应用划分成多个模块。随着模块变多与路径变深，我们可能不得不在`require`时，使用一长串的相对路径，比如：
```javascript
const utils = require('../../../libs/utils')
```
下面提供一些解决方案。

## 使用 require 的替代方法

## 使用第三方库

## 设置 NODE_PATH

## 利用 node_modules

## 利用软连接

## Webpack

## 参考资料
