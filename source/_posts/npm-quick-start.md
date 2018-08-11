---
title: npm 快速上手
tags:
  - javascript
date: 2018-06-09 22:27:34
---

本文主要目的是为**还在使用传统项目管理方式的前端开发者/团队**，提供使用 npm 来进行组织项目的上手指南。

npm(Node Package Manager) 是 Node.js 官方的包管理工具。随着 JS 的不断发展，npm 几乎成了前端项目的管理工具，我们可以借助 npm 来进行一些前端的代码库和项目的管理。

<!-- more -->

## 基本指令

我们可以在 [npm 的官方文档](https://docs.npmjs.com/#cli) 中查看所有指令的详细使用方式，我们这里只对最常用的指令做一个列表。

* `npm install`： 安装该项目的所有依赖。
* `npm install chalk --save-dev`：安装第三方依赖`chalk`，并保存到开发时依赖(devDependencies)。
* `npm install lodash --save`：安装第三方`lodash`，并保存到运行时依赖(dependencies)。
* `npm uninstall chalk --save-dev`：移除第三方`chalk`，并从开发时依赖中移除(devDependencies)。
* `npm uninstall lodash --dev`：移除第三方`lodash`，并从运行时依赖中移除(dependencies)。
* `npm update lodash`：升级项目中的`lodash`。
* `npm publish`：将当前项目当作一个 npm 包发布。
* `npm init`：初始化一个基于 npm 管理的项目（生成一个 `package.json` 文件）。

## package.json

package.json 文件是 npm 包默认的配置管理方案，它也是一个 npm 项目必须有的文件。

1. 列出项目依赖的所有包。
2. 指定该项目依赖的所有包的版本。
3. 指定开发构建脚本。

### name

项目对应的 npm 包名。包名必须是全小写，支持连字符/下划线，例如 `lodash`。当你的项目发布到 npm 上后，别的开发者可以使用这个名字来安装你的包（使用`install`命令）。如果你的项目不需要发布到 npm 源，可以不填。

### version

对应的项目版本，需要满足[semver](https://semver.org/lang/zh-CN/)的标准，多数情况下，我们只需要知道是**主版本号.次版本号.修订版本号**（即 x.x.x）的格式即可。同样的，如果你不需要发布，可以不填。

### private

如果你的项目不需要发布，请务必设置 `private: true`，确保不会将你的项目误发布到 npm 源上。

### description

项目的描述。

### scripts

scripts 是一个 hash 对象，我们一般用它来汇总整个项目用到的所有开发、构建、部署等命令。

阮一峰之前写过一篇[介绍 npm script 的文章](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)，很适合入门者，我们不再过多赘述。不过要注意的是，这篇文章写作时间较早，现在有一些钩子已经进行了更新。我们可以参照官方文档的 [npm-scripts](https://docs.npmjs.com/cli/run-script) 和 [npm-run-script](https://docs.npmjs.com/cli/run-script) 进行学习。

虽然没有成文的规定，但是有些命令基本是我们约定俗称的，比如：

* `start`：启动服务器，多用在服务端的项目。
* `dev`: 开发指令，多数情况下都是启动一个开发用的服务器，或者是构建一个中间产物用于开发。
* `build` 或 `dist`：构建结果代码。
* `lint`: 使用 lint 工具检查源代码。
* `test`: 单元测试/E2E 测试。

### dependencies

项目的依赖管理，我们使用 `npm install xxx --save` 的时候，npm 会自动把我们安装的包存在这个字段。一般情况下，请不要手动编辑。

### devDependencies

项目的开发依赖管理，我们使用 `npm install xxx --save-dev` 的时候，npm 会自动把我们安装的包存在这个字段。一般情况下，请不要手动编辑。

devDependencies 和 dependencies 的区别是：devDependencies 是开发代码时的依赖，而 dependencies 是代码的运行时的依赖。
例如：如果项目开发中用到了 webpack 打包，但是在打包完成后，我们的代码运行时并不依赖 webpack，所以 webpack 应当被添加在 devDependencies。

### main/module

main 用于指定入口文件，当我们使用 `require('module-name')`的时候，就会默认引入这个文件。默认是`index.js`。

它的作用是标识 ES6 模块的入口。当我们使用 `import 'module-name'`的时候，默认导入的即为这个文件。module 目前不是一个标准的字段，但是打包工具 webpack 和 rollup 都可以识别这个字段。更多的关于这个字段的详细信息，可以在这个 StackOverflow 的[回答](https://stackoverflow.com/questions/42708484/what-is-the-module-package-json-field-for#answer-42817320)下查看。

## 全局与本地

在使用 `install` 指令安装包的时候，我们可以使用 `-g` 参数将包装到 npm 全局，这对一些命令行工具非常实用。但是对于我们的项目的开发依赖（例如 webpack/rollup），建议还是全部装到项目本地，这样有利于每个项目独立管理依赖版本。
在 node 8+ 之后的版本，会自动安装 `npx`，它可以方便的执行项目内的二进制包，可以在一定程度上替代全局安装。

## 锁

如果你注意观察，会发现 npm 的依赖中，默认所有的包的版本都是类似`^1.0.4`的这种格式：这在 semver 规范中代表着，我们只严格限制主版本号为`1`、次版本号为`0`，只要修订版本大于等于`4`，即可满足当前要求。在理想状况下，`1.0.5` 和 `1.0.6` 的版本发布，并不会引入新的问题，我们业务可以放心的升级。但是现实中，一个依赖的小版本升级有可能会引入一些额外的问题，上层没有任何的保护机制来处理这种第三方库引入的 BUG。

在新版本的 node 中，`npm install` 会默认生成一个锁文件 `package-lock.json`。这个 `package-lock.json` 中，记录了我们直接和间接引入的所有包的版本、安装源以及 hash。有了这个文件，就可以保证在所有的机器上使用 `npm install` 会得到相同的结果。因此，我们应当**将 package-lock.json 纳入版本管理中**。

## yarn

yarn 是一个 Facebook 提出的替代 npm 的包管理器。比起 npm 来讲，它最大的优势就是安装速度快。尤其是用于 CI 时，安装速度占了整个集成时间的一大部分，优化安装速度可以显著提高整个 CI 的集成速度。

再的详细使用方法和 npm 类似，这里不再赘述，可以参考[官方的快速上手指南](https://yarnpkg.com/zh-Hans/docs/usage)。
**注意：**一个项目中，请仅使用一个工具来管理包，用了 yarn 就不要用 npm，反之亦然。如果你适用了 yarn 管理包，务必把 yarn.lock （yarn 的版本锁文件，效果和 `package-lock.json` 类似）这个文件纳入版本管理，并且可以把 package-lock.json 纳入 ignore 列表，反之则相反。
