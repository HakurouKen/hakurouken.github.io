---
title: 为 Vue 组件提供 js-api
tags:
  - javascript
  - vue
date: 2017-07-16 15:17:10
---

在 Vue 单文件组件模式下，我们在引用组件时，需要局部注册 component 来使用。例如，我们需要一个类似`window.alert`的提示弹框组件，核心代码大致如下（使用 jsx 语法进行描述）：
```jsx
export default {
  name: 'MyAlert',
  props: ['text'],
  methods: {
    close() {
      this.$emit('close')
    }
  },
  render(h) {
    return (
      <div class="popbox-container">
        <section class="popbox-header">
          <span class="close" onClick={close}></span>
        </section>
        <section class="popbox-body">
          <p>{text}</p>
        </section>
        <section class="popbox-footer">
          <button onClick={close}>确定</button>
        </section>
      </div>
    );
  }
}
```

在使用的时候，我们需要在父级引入：
```html
<template>
  <div>
    <button @click="popup = true" />
    <my-alert :text="'test'" v-if="popup" @close="popup = true" />
  </div>
</template>
<script>
import MyAlert from './my-alert';
export default {
  data () {
    return { popup: false }
  },
  components: { MyAlert }
}
</script>
```
这样的使用方式，从“组件化”的思路上看，并没有任何问题，但是从实际工程的角度来看，它有这样几个瑕疵：
1. 每次使用都要然后注册到 component，比较麻烦。
2. 类似 alert 这种弹窗组件，和父组件只有逻辑上的属于关系，但是实际的 DOM 结构并不适合放在父组件内的任何位置，从直观感受上，应该直接是 body 的子元素。
3. 每个需要用到 alert 的父组件中，都要初始化一个独立的 alert 组件，不能做到像一些 jQuery 插件一样，复用现有 DOM 结构。
4. 在父组件中，需要一个额外的变量来控制 alert 组件初始化。

在实际使用的时候，我们更希望像原生`alert`一样，使用`myAlert("test")`就能够初始化组件，同时将 DOM 结构直接放到 body 下。

<!-- more -->
## js-api 封装
在 [Element](https://github.com/ElemeFE/element) 和 [iView](https://github.com/iview/iview) 库中，大量用到了类似的做法，例如 Element 的 [MessageBox](https://github.com/ElemeFE/element/blob/dev/packages/message-box/src/main.js) 和 iView 的 [Notification](https://github.com/iview/iview/blob/1b785ff08bbf00c96f4b1cdca4d83aef72dd30ef/src/components/base/notification/index.js)，其核心原理就是**创建一个 Vue 示例并 mount 到一个自定义空 div 上，再手动添加到 dom 树中**。在这之后，我们只需要对外暴露一套 API ，把创建 Vue 和 mount 过程隐藏在函数内部即可。

## 事件封装
虽然我们可以直接通过 props 把外部的回调传递给组件，不过我们实际开发组件时一般不习惯这样做。为了保持组件自身的完整性，我们还是通过`on`方法来监听。当我们需要从外部手动销毁示例时，要手动销毁实例，并将插入的 dom 元素移除。

按照上面两条的思路，上文的 MyAlert 组件可以做一层简单的包装，如下：
```javascript
import MyAlert from './my-alert'

export default function (text, onClose) {
  function close() {
    instance.$destroy();
    document.body.removeChild(instance.$el);
    onClose && onClose()
  }

  const instance = new Vue({
    render(h) {
      return h(MyAlert, {
        props: {
          text: text
        },
        on: {
          close: close
        }
      })
    },
    el: document.createElement('div')
  });

  document.body.appendChild(instance.$el);

  const component = instance.$children[0];

  return {
    _component: component,
    close
  }
}
```

## 适用场景
对于绝大多数情况，这种 js-api 初始化的方式都不适用。但是，对于浮层/弹窗/通知等等控件来讲，使用 js-api 进行一层封装，会大大提高开发的体验。

## 参考资料
1. [iView 源代码](https://github.com/iview/iview)
2. [Element 源代码](https://github.com/ElemeFE/element)
