---
title: 如何打包一个 Vue 组件库
tags:
  - javascript
  - vue
  - webpack
---

最近对一个公司内部的 Vue UI 组件库做了打包优化，

## 组件库打包与项目打包的异同

1. 组件库的产出结果，并不直接面向用户，而是面向开发者。**你的代码多数情况下是被二次打包使用**。因此，不提升接入成本的情况下，我们可以将我们组件的输出结果使用第三方工具链来进行二次处理，例如使用自定义的 webpack-loader 或 babel 插件等等。以一个具体的场景为例，很多组件库的“按需引入”的功能都是通过 `babel-plugin-import` 插件来实现的；
2. 因为需要满足不同的使用场景，组件库可能需要面向不同环境的打包结果，例如需要 es/commonjs 两种格式的输出结果，以及单个 UI 组件也需要产出输出结果等等；
3. 组件库对于包大小更加敏感，因为组件库往往是基础库，影响范围广。

因此，对于组件库的打包，在工具选型上往往有很多需要抉择的点。尤其是“按需加载”的功能几乎成为标配后，我们不但需要考虑将整个库作为一个 bundle，还要将每个组件作为独立入口打包，打包

1. 是否使用 webpack 打包：webpack 会每个独立入口都会引入 webpack 的 runtime 代码（~900 bytes minified）。
2. 是否使用 `.vue` 的单文件组件：使用 .vue 组件，无论是使用 `vue-loader` 还是 `rollup-plugin-vue`，都会自动注入 `normalizeComponent` （~750 bytes minified）用于格式化单文件组件。（对于新版本的`rollup-plugin-vue`，我们可以把 `vue-runtime-helpers` 作为 `external`）。
3. 多入口之间复用代码: 例如我们有两个组件 `Button` 和 `Dialog`，`Dialog` 是依赖 `Button` 组件的，那么我们需要在引入 `Button` 再引入 `Dialog` 后，不重复引入 `Button` 。这就要求我们将所有的独立入口和可能的公共部分，均不能进行代码合并，而是需要保留“引用”的状态。
4. 样式：为了业务的二次定制换肤，我们的样式往往不和业务逻辑放在一起，而是独立出去单独打包。

## 流行组件库的做法

即使同为 UI 组件库，对于打包的选择也各不相同。下面列举一些主流的 UI 库和打包相关的关键设置，

### element-ui

1. 使用 webpack 打包完整包
2. 为了按需引入，将每一个独立的组件都作为 webpack 的入口进行独立打包。打包的配置中，将除此组件外所有的文件都作为 external 进行打包
3. 样式独立打包，甚至独立发布

### iview

1. 使用 webpack 打包
2. 没有为按需引入专门打包（按需引入直接指向了源代码）
3. 样式独立打包

### muse-ui

1. 项目中不使用单文件组件和 jsx 语法
2. 使用 rollup 进行打包完整包（无需 `rollup-plugin-vue` 等相关配置）
3. 直接使用 babel-cli 编译 src 目录作为按需引入的打包结果

### ant-design-vue

1. 只使用 jsx，不使用 vue 单文件组件，样式也单独处理
2. 构建工具基于 [antd-tools](https://github.com/ant-design/antd-tools) 修改。本质上是基于 gulp 进行上层开发，使用 gulp-babel 编译单个组件，使用 webpack 做完整包构建。

### vuetify

1. 使用 ts/js，不使用单文件组件和 jsx
2. 样式分开在单独的 .styl 写，在 js 中引入，并使用独立的 babel 插件 [babel-transform-stylus-paths.js](https://github.com/vuetifyjs/vuetify/blob/master/packages/vuetify/build/babel-transform-stylus-paths.js) 来将 .styl 路径替换为对应的 .css 路径
3. 使用 webpack 构建完整包

### vant

1. 使用 js/ts/tsx，不使用单文件组件
2. 样式在独立的文件写，直接使用 less/postcss 工具包编译
3. 使用 webpack 编译完整包，使用 babel 编译单个组件入口

### vux

1. 使用带样式和模板的 .vue 单文件组件
2. 发布源代码，使用 vux-loader 进行编译/按需引入

### vue-material

1. 使用带样式和模板的 .vue 单文件组件
2. 使用 webpack 打包单组件入口和完整包，使用 ExtractTextPlugin 提取样式

## 打包工具的选择

尽管不同的 UI 库的打包配置不尽相同，但是我们可以得到下列的共识：

1. 如果使用单文件组件，第一选择是使用 webpack 打包（因为其余的打包工具对 .vue 支持都不太完善，有一些坑要踩）。具体的配置和项目的打包差异不大，因此好理解和维护，但同时不得不接收 webpack 和 vue 的 runtime 代码。
2. 如果类似 vux 同样交由外部打包，拥有最高的灵活度，但同时也限制了外部的使用方式，同时增加了编译时间。而且并不是所有的使用方都使用 webpack 来进行打包（或者使用 webpack3/4 的版本差别)，我们可能会面临维护多个打包插件的麻烦。
3. 如果放弃单文件组件，可以使用 rollup 或者 babel-cli 直接打包。这是一个相对理想的方式，而且因为 jsx 的存在，放弃 template 并不会导致编码过于繁琐。

基于上述的结论综合一些业务特性，我们做出了如下选择：

1. 因为业务需要，UI 库需要内置多套皮肤，因此选择单独写样式
2. 放弃 .vue 单文件组件，使用 `@babel/cli` 直接**转义**源代码。因为这一步并不需要合并文件，所以不需要使用 rollup。
3. 使用 webpack 打包完整包（commonjs/umd）。因为整个 bundle 相对较大，webpack 的 runtime 代码在这里的影响可以忽略不计。当然，这一步也可以使用 rollup 进行打包，但考虑到本身组件库开发就已经需要依赖 webpack，

## 更多细节

### @babel/polyfill

由于 polyfill 本身是有副作用的，再加上 UI 库自身无法预知外部的使用环境（例如是否只需要兼容现代浏览器，或者是还需要支持 IE9+，甚至使用方本身就是一个基于当前 UI 库二次开发的组件库），无法做到细粒度的控制，因此**不推荐在业务中引入 polyfill**。当然，为了外部引用的简便，我们的组件库应当尽量避免需要 polyfill 的语法，例如 `Array.prototype.includes` 等。

### @babel/plugin-transform-runtime

两点建议：

1. 根据你业务的自身情况，决定是否启用 `@babel/plugin-transform-runtime`。如果只用到了很少的帮助办法，完全可以不使用这个插件，将用到的方法 inline。如果大量的用到了需要转换的语法（如对象结构、class、taggedTemplate 等），则使用 `@babel/plugin-transform-runtime`，并将 `@babel/runtime` 加入 `dependencies`。
2. 即使你启用了 `@babel/plugin-transform-runtime`，也不要使用可能引入 regenerator-runtime 的语法（即 `async/await` 和 `generator`），因为这个包太大了。对于 UI 组件库，我们并不需要大量的异步操作，使用 `async/await` 并不会对我们的编码效率带来太大的提升。

### 样式

因为我们的开发模式比较特殊，逻辑和样式往往由不同的人负责，因此我们很自然的把 style 抽离到单独的文件夹来管理。不过即使由一个人负责，分开到独立的文件夹也有合理性：你可以更方便的处理在相同的 DOM 结构下的“换肤”的需求。这里的换肤不仅仅简单的 less 变量替换，可能还有一些不同使用场景下的定制。不过我们可能需要添加一些静态检查的脚本，例如强制要求所有的组件都有一个对应的 css 入口等等。

### jsx

如果使用了 jsx，按照[vue/jsx 官方文档](https://github.com/vuejs/jsx)配置即可，注意`@vue/babel-helper-vue-jsx-merge-props` 需要加到 `dependencies`。

## 参考资料

1. [组件库 rollup 打包体积优化 - 掘金](https://juejin.im/post/5b25f324f265da59921a02d5)
2. [如何开发一个基于 Vue 的 ui 组件库（一） - 知乎](https://zhuanlan.zhihu.com/p/54290649)
3. [javascript - babel 的 polyfill 和 runtime 的区别 - SegmentFault 思否](https://segmentfault.com/q/1010000005596587)
4. [@babel/polyfill 与 @babel/plugin-transform-runtime 详解 · Issue #4 · Weiyu-Chen/blog](https://github.com/Weiyu-Chen/blog/issues/4)
