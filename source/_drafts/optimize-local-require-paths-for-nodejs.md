---
title: 优化 Node.js 中的深层 require 路径
tags:
---

## 问题
随着 Node.js 应用规模的增长，我们经常会将应用划分成多个模块。随着模块变多与路径变深，我们可能不得不在`require`时，使用一长串的相对路径，比如：
```javascript
const utils = require('../../../libs/utils')
```
这种写法不但丑陋，而且会给代码定位造成麻烦。下面针对这一问题，提供一些解决方案。

## 解决方案
### 使用 require 的替代方法
在 CommonJS 规范中，模块不仅可以通过相对路径加载，也可以通过绝对路径加载。因此，我们可以在项目的入口处，写入一个全局方法：
```javascript
const path = require('path')
global.requireLocal = function (name) {
  return require(path.join(__dirname, name))
}
```
在项目中，我们将所有
```javascript
require('../../../libs/utils/time')
```
替换为
```javascript
requireLocal('libs/utils/time')
```
即可。这里选择直接写成全局方法是为了避免出现需要使用引入这个方法时，出现类似
```javascript
require('../../../lib/requireLocal')
```
的代码。这种解决方案实施起来简单粗暴，缺点也很明显：侵入性较强，需要对源代码进行修改才可以使用，同时无法支持 es6 的 import/export 语法。

### 使用第三方库
这里和上面的方法没什么本质区别，但是利用了成熟的第三方库来替代我们上文简陋的方法。例如，使用 [require.js](http://requirejs.org/docs/node.html)，可以将 require 相关的代码改为：
```javascript
var requirejs = require('requirejs');

requirejs.config({
  baseUrl: __dirname,
  nodeRequire: require
});

var foo = requirejs('foo');
```

### 设置 NODE_PATH
由于`NODE_PATH`的包可以直接引用，我们可以将`NODE_PATH`环境变量指向对应的应用路径。假设我们想暴露到全局的文件夹是`src/lib`，

在 Windows 下：`export NODE_PATH=./src/lib`

在 Linux/Mac 下：`set NODE_PATH=./src/lib`

我们也可以用 npm 脚本，将设置环境变量和执行脚本放在一起。这里我们用 [cross-env](https://github.com/kentcdodds/cross-env) 统一 Windows 和 Linux 下环境变量的设置：
```javascript
{
  "name": "test",
  "scripts": {
    "start": "cross-env NODE_PATH=lib node index.js"
  }
}
```
有不少库用到了这种方法来设置环境变量，但这种做法也有一定的局限性：环境变量的设置是全局的，可能会影响到同一机器上的其它应用。因此我们可以在 node 代码中，用下列的 hack 手法(需要放在所有的 require 之前):
```javascript
process.env.NODE_PATH = __dirname + '/src/lib'
require('module').Module._initPaths()
```
这里的作用就显得不那么直观，需要对 node 的 Module 模块有一定的了解，才能理解代码的含义。

### 利用 node_modules
将需要的`lib`放在`node_modules/lib`即可解决。需要注意的是，如果你的`.gitignore`里有
```
node_modules
```
则需要将其改为：
```
node_modules/*
!node_modules/lib
```
这里的做法有些反常识，因为通常`node_modules`是完全由第三方组件构成的文件夹。将自己的业务代码混入 node_modules ，会使得管理上有些混乱。

### 利用 node_modules（软连接）
由于将代码 hardcode 到 node_modules 中存在一些问题，我们可以将代码写在外部，然后在 node_modules 中建立软连接，也可达成一样的效果。我们还可以把这个过程自动化，写成 npm 的 install/postinstall 的 script，这样在执行`npm install`后，这一步骤将会自动完成; 如果使用 git 或 svn 管理代码，也可以将这个写成钩子，在代码更新时执行。由于 Linux/Windows 下的“软连接”的命令有差异，可以用一个 node 的脚本：
```javascript
const fs = require('fs')
const path = require('path')

function linkToNodeModule(src, dst){
  dst = path.join(__dirname, 'node_modules', dst || src)
  src = path.join(__dirname, 'dist', src)

  fs.lstat(dst, (err, stat) => {
    if (!stat) {
      fs.symlink(src, dst, 'dir', err => {
        if (err) throw err
      });
    }
  });
}

linkToNodeModule('lib');
```
之后，我们的代码中，就可以使用
```javascript
const misc = require('lib/misc')
```
这样的代码了。这也是我**个人比较推荐**的一种无副作用的方式，而且相对直观，在大多数场景下都适用。

### Webpack
如果你的代码主要在浏览器中运行，用到了 Webpack 打包，那么问题就简单了, Webpack 的 [alias 参数](https://webpack.github.io/docs/configuration.html#resolve-alias)已经帮你解决了这一问题。

## 总结
虽然上述方法可以缓解相对路径过长的问题，但我们仍然应该意识到，过深相对路径往往意味着在设计时过早将一个完整的 app 打散成了多个部分，导致不同的模块间依赖较严重。多数情况下，我们可以将公共的模块抽象到一个文件夹下（通常叫 lib 或 common 之类的），然后再用上述技巧来消除过多的`../../`路径。

## 参考资料
