---
title: 使用更小的 Lodash
tags:
  - webpack
  - javascript
date: 2017-04-10 21:38:23
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
一般介绍 lodash 打包优化的文章，介绍到这里就算结束了。但是，lodash-webpack-plugin 还有一系列的 feature 配置，它们的作用又是什么？实际上，经过 lodash-webpack-plugin 转化的 lodash 函数，已经和文档中对应的函数不完全等价，因此，如果我们用到了一些特定的特性，我们需要通过调整这些配置，保证我们的代码正常运行。

要关注这些配置的作用，只需要关注[mapping.js](https://github.com/lodash/lodash-webpack-plugin/blob/master/src/mapping.js)这个文件，其中配置了`features`和`overrides`两个变量。`features`中每一项对应一个开关，每个开关则对应着若干组替换规则。以`cloning`为例，当对应配置关闭（默认即为关闭）时，插件将把内部的`_._baseClone`（暂且这样表示）方法替换为`_.identity`方法。而`overrides`变量中的每一项表示：只要我们引用了对应方法，则打开对应配置开关。

了解了这一点，我们再谈谈这个配置是怎样影响对应方法的。以`_.sortBy`方法为例，在它的[源代码](https://github.com/lodash/lodash/blob/es/sortBy.js)中（注：由于我们使用 npm 安装，因此需要查看 es 分支的源代码），我们可以看到它使用了四个内部库：
```javascript
import baseFlatten from './_baseFlatten.js';
import baseOrderBy from './_baseOrderBy.js';
import baseRest from './_baseRest.js';
import isIterateeCall from './_isIterateeCall.js';
```
在`flattening`规则中，内部`_baseFlatten`将会被替换为`_.head`方法。本来根据`_.sortBy`[文档](https://lodash.com/docs/4.17.4#sortBy)，下列代码可以正常运行：
```javascript
var users = [
  { 'user': 'fred',   'age': 48 },
  { 'user': 'barney', 'age': 36 },
  { 'user': 'fred',   'age': 40 },
  { 'user': 'barney', 'age': 34 }
];

_.sortBy(users, function(o) { return o.user; });
```
经过插件转化后，这种写法将会抛出异常。我们只能使用下列写法：
```javascript
_.sortBy(users, [function(o) { return o.user; }]);
```
收到类似影响的函数还有`_.overArgs`,`_.flow`等几个方法。同理，`_.get`，`_.has`等方法，也会`_baseGet`方法被替换受到影响。

要解决这个问题，我们需要校验自己的代码中，有没有用到官方描述的[Feature Sets](https://github.com/lodash/lodash-webpack-plugin#feature-sets)中类似的特性，然后更改自己的代码，或者开启对应的特性配置。**一般情况下，开启 flattening 和 paths 配置可以解决大部分问题。**

## 参考资料
1. [lodash 官方文档](https://lodash.com)
2. [babel-plugin-lodash 项目文档](https://github.com/lodash/babel-plugin-lodash)
3. [lodash-webpack-plugin 项目](https://github.com/lodash/lodash-webpack-plugin)
