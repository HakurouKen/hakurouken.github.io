---
title: 使用更小的 Lodash
tags:
  - webpack
  - javascript
---

Lodash 是个十分有用的工具库，但我们往往只需要其中的一小部分函数。这时，整个 lodash 库就显得十分庞大，我们需要减小 lodash 的体积。

## cherry-pick 方法
Lodash 官方在 npm 上为每个方法维护了[独立的包](https://github.com/lodash/lodash/tree/npm-packages)，便于我们直接引用。举个例子，如果我们需要`lodash.chunk`方法，可以直接安装`lodash.chunk`这个 npm 包，在代码中 import 即可：
```javascript
import _chunk from 'lodash.chunk';
const result = _chunk(['a', 'b', 'c', 'd'], 2);
console.log(result);
```
这样的弊端很明显，我们如果需要使用一个新方法，就要重新 install 一个新包并 require 进来。另外，每个包之间会有一部分重复（公共）方法会被重复打包，也会增大最终的打包体积。更好的解决办法，是直接从 lodash 完整库内导入方法：
```javascript
import flatten from 'lodash/flatten';
import flatMap from 'lodash/flatMap';
```
由于 lodash 库中，每个方法都以独立模块封装，这样的引入方法，在 webpack 打包时，只会按需取指定的两个模块进行打包。从[flatten](https://github.com/lodash/lodash/blob/master/flatten.js) 和 [flatMap]([flatten](https://github.com/lodash/lodash/blob/master/flatten.js)) 的代码可以看出，二者都用到了`./.internal/baseFlatten.js`这个模块，在 webpack 打包时，这个模块就变成了公共模块，这就避免了引入两个独立方法的包带来的重复。这种引入方式，官方叫做 “cherry-pick methods”，就像捡樱桃一样，捡自己需要的方法。

## babel-plugin-lodash
然而，cherry-pick 方法的书写习惯，和我们平时使用第三方库的书写习惯并不一致。我们更习惯于直接 import 整个库：
```javascript
import _ from 'lodash';
_.flatten([1, [2, 3], [4]])
_.flatMap([1, 2], n => [n, n])
```
这时，就是[babel-plugin-lodash](https://github.com/lodash/babel-plugin-lodash)库发挥作用的时候。在 babel 编译代码时，这个插件可以自动把这种代码
```javascript
import _ from 'lodash';
import { add } from 'lodash/fp';

const addOne = add(1);
_.map([1, 2, 3], addOne);
```
转化为
```javascript
import _add from 'lodash/fp/add';
import _map from 'lodash/map';

const addOne = _add(1);
_map([1, 2, 3], addOne);
```
不过，这里有一些[额外限制](https://github.com/lodash/babel-plugin-lodash#limitations)：必须要引入 babel（6以上版本）；必须要使用 ES2015 的方式引入包（即 import 的方式）。

## lodash-webpack-plugin
在这一步的基础上，我们还可以使用 [lodash-webpack-plugin](https://github.com/lodash/lodash-webpack-plugin) 进一步减包。它的原理是将我们用到的方法（和间接用到的内部方法）使用更简单的方法替代。具体的方法对应关系，参照源代码中的[mapping.js](https://github.com/lodash/lodash-webpack-plugin/blob/master/src/mapping.js)。[使用方法](https://github.com/lodash/lodash-webpack-plugin#usage)也非常简单，只需要在 webpack 配置中引入该 plugin ，即可将所有的 **cheery-pick 引入的方法**自动进行转换。

## 需要注意的问题

## 参考资料
