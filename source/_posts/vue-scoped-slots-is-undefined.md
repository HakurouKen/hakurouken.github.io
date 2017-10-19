---
title: Vue 升级引发的 $scopedSlots 失效问题
tags:
  - javascript
  - vue
date: 2017-06-20 23:44:35
---


## 问题描述
项目的 Vue 的版本从 2.3.2 升级到了 2.3.3，所有作用域插槽失效，vm.$scopedSlots 的值为 `{undefined: undefined}`。

<!-- more -->
## 原因
搜索了一下，发现很少有人遇到类似的问题，发现有一个一样情况的[issue](https://github.com/vuejs/vue/issues/5807)，不过完全没有反映问题根源所在。唯一的共同点是，这个问题都是由 2.3.2 升级到 2.3.3 导致的。
对比下 2.3.2 和 2.3.3 的源代码，果然有关作用域插槽的部分[有所变更](https://github.com/vuejs/vue/compare/v2.3.2...v2.3.3#diff-c93dcf5fbf55768d2f3aed1685d8f34c)。这也就意味着，一定是 template 转化为 render 函数的过程中出现了偏差。对于用 webpack 管理的项目，所有的 .vue 文件都是通过`vue-loader`解析的，而`vue-loader`内部的模板解析，又间接的用到了`vue-template-compiler`。分别取两个版本的 vue 跑个对比，对于下列模板：
```html
<hello>
  <template scope="props">{{props.msg}}</template>
</hello>
```

使用 vue-template-compiler 2.3.2 版本下，会编译为：
```javascript
function (){var _vm=this;var _h=_vm.$createElement;var _c=_vm._self._c||_h;
  return _c('hello', {
    scopedSlots: _vm._u([
      ["default", function(props) {
        return [_vm._v(_vm._s(props.msg))]
      }]
    ])
  })
}
```

在 2.3.3 版本下，则是：
```javascript
function (){var _vm=this;var _h=_vm.$createElement;var _c=_vm._self._c||_h;
  return _c('hello', {
    scopedSlots: _vm._u([{
      key: "default",
      fn: function(props) {
        return [_vm._v(_vm._s(props.msg))]
      }
    }])
  })
}
```

看来已经破案了，编译模板使用的`vue-template-compiler` 的 2.3.2 版本，而使用的 vue 版本是 2.3.3 ，因此导致了这个编译问题，直接把`package.json`的`vue-template-compile`改成`=2.3.2`，应该就可以复现这个 bug。但是实际操作的时候，却出现了另一个错误：
```
Vue packages version mismatch:

- vue@2.3.3
- vue-template-compiler@2.3.2

This may cause things to work incorrectly. Make sure to use the same version for both.
If you are using vue-loader@>=10.0, simply update vue-template-compiler.
If you are using vue-loader@<10.0 or vueify, re-installing vue-loader/vueify should bump vue-template-compiler to the latest.
```
仔细检查了`vue-template-compiler`的代码，发现其中有如下校验:
```javascript
try {
  var vueVersion = require('vue').version
} catch (e) {}

var packageName = require('./package.json').name
var packageVersion = require('./package.json').version
if (vueVersion && vueVersion !== packageVersion) {
  throw new Error(
    '\n\nVue packages version mismatch:\n\n' +
    '- vue@' + vueVersion + '\n' +
    '- ' + packageName + '@' + packageVersion + '\n\n' +
    'This may cause things to work incorrectly. Make sure to use the same version for both.\n' +
    'If you are using vue-loader@>=10.0, simply update vue-template-compiler.\n' +
    'If you are using vue-loader@<10.0 or vueify, re-installing vue-loader/vueify should bump ' + packageName + ' to the latest.\n'
  )
}
```
这段代码实际保证了 vue 的版本的正确性，而我们的项目是使用的 external 方式引入的 vue，为了兼容老项目，在 webpack 中 resolve 为了 `common/vue`，导致最后没有抛出这个异常。不过问题的核心还是在这里，我们直接升级一下对应的包即可解决这个问题。

## 反思
1. 能用 npm 管理的包，尽量统一用 npm 管理。现有的构建工具和各种脚手架都是基于 npm 生态构建，如果自己管理包，会遇到很多不必要的麻烦。事实上，我们将 vue resove 为 `common/vue` 的方式，就在 vue 的 hot-reload 上踩了坑。
2. 如果出现 CI 构建和本地不一致的情况，多数情况下是因为 npm 包版本不一致导致的，`npm install`可以解决大部分的问题。如果不行，删掉 lock 文件再试一次。

## 参考资料
1. [Vue issue#3941](https://github.com/vuejs/vue/issues/3941)
2. [Vue issue#]
