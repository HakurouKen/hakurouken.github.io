---
title: Vue 中的驼峰和连字符命名
tags:
  - javascript
  - vue
date: 2017-10-18 21:31:47
---


由于 HTML 的大小写不敏感，当我们使用 in-dom 的模板时，`myProp＝"123"`会被转换成`myprop="123"`。为了解决这个问题，Vue 引入了模板中驼峰/Pascal/连字符混用的机制，官方推荐开发者在所有的模板中，使用连字符的格式来描述组件/属性。

<!-- more -->
## 解析规则
事实上，Vue 底层处理模板字符串的方法相对比较复杂，做了很多兼容性处理。
1. 在模板 html 解析(parse)的过程中，配对标签(tag)名时，是忽略大小写的，形如`<HelloWorld>text</helloworld>`这样的模板，也可以正常配对匹配。这样处理的原因，主要是为了模拟原生 HTML 的 tag 解析方式。但是这里解析出的 AST 树中，**仍然保留着大小写信息，以起始的标签为准**。详细的匹配细节，可以参照 Vue 源码中的 [src/compiler/parser/html-parser.js](https://github.com/vuejs/vue/blob/08bc7595fd57f8f52db83ae1b6bc9b7a33cdd4f9/src/compiler/parser/html-parser.js)。
2. 自定义组件和属性，在模板编译时做了 Pascal/驼峰/连字符 三种方式的兼容（其中属性会在解析时，内部统一标准化为驼峰）。
3. 对于事件名，Vue 不会做处理，$emit 的事件字符串要和模板中的字符串**完全匹配**(keep it as it is)。
4. IN-DOM 的模板会全部转小写(HelloWorld => helloworld)，但这并不是 Vue 的限制，而是 HTML 的限制。

由于第4条的原因，我们习惯在模板内，全部使用连字符的格式（包括组件名/属性/事件名），这样可以保持形式上的统一。因此，事件名也统一使用形如`custom-event`的格式。

## JSX
在 Vue 对于 jsx 的支持，一般使用官方插件 [babel-plugin-transform-vue-jsx](https://github.com/vuejs/babel-plugin-transform-vue-jsx)。和 React JSX 不同，这个插件对 JSXAttribute 中的连字符的情况做了特殊处理，我们可以在模板中(一定程度上)使用连字符格式来代替驼峰：
```jsx
<div on-click={this.click}></div>
```
但是，这个兼容做的并不完全，比如使用`nativeOn`时，我们不能使用形如`native-on-event`的格式，而必须从下列两种方式中选择一种：
```jsx
<CustomComponent nativeOn-click={this.click} />
<CustomComponent nativeOnClick={this.click} />
```
对于自定义事件，**JSX 无法支持连字符方式的事件名**，只能采用驼峰（`on-`前缀倒是可以兼容），可以参照[这个 issue](https://github.com/vuejs/babel-plugin-transform-vue-jsx/issues/20)。

## 引入JSX后的事件命名
这里便出现了一个争论点，在用项目用到 vue + jsx 的前提下，事件名到底有没有必要统一为驼峰？经过一系列的实践，这里的答案是不需要统一，尽量保持原有的连字符事件，只有在必要的情况下，再做修改。大致原因有以下几点：
1. 驼峰和下划线混用，并不会导致什么逻辑上的问题，已有的组件和开发习惯，多数情况下无需变更。
2. 使用 template 的方式还是要比 jsx 更加简洁，因此绝大多数场景下，我们还是采用 template 的方式来书写组件模板，这时事件名也使用连字符格式保持相对统一。使用 JSX 的场景，多是作为最外层的组合业务组件复杂逻辑组件，或是作为内部组件的函数式组件，这两种情况不涉及暴露外部接口，驼峰和连字符混用的情况只出现在少量的内部组件中，可以接受。
3. 如果确实是多业务公共组件遇到此类问题，可以考虑通过传入参数来控制事件名是驼峰还是连字符格式。有必要的话，可以进一步将这一系列逻辑封装成插件，取代原生的`$emit`方法。
4. 对于单一业务场景，尽量把事件名抽象为形如原生事件中的`touchstart`和`transitionend`这种“行为/控件 + 状态”的格式，例如`commentstart`,`dataready`等等，然后全部用小写，也可以避免这个问题。
5. 我们可以采用 render function 和 jsx 混用的方式，来应付连字符的事件名。

当然，这里有一个前提，就是**避免使用 IN-DOM 的模板**。事实上，Vue 官方脚手架生成的代码中，也刻意避开了 IN-DOM 模版的使用，因为 HTML 的大小写不敏感的特性，确实不利于目前以 JS 为主体的业务开发。

## 参考资料
1. [Vue 官方文档： camelCase vs kebab-case](https://vuejs.org/v2/guide/components.html#camelCase-vs-kebab-case)
2. [Vue Issue: HTML case sensitivity workaround](https://github.com/vuejs/vue/issues/2308)
3. [JSX 官方文档](https://facebook.github.io/jsx/)
4. [babel-plugin-transform-vue-jsx issuse: Compile error kebabcase event name](https://github.com/vuejs/babel-plugin-transform-vue-jsx/issues/20)
