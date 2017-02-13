---
title: Vue 中的双向绑定
tags:
  - javascript
  - vue
---

在 Vue 中构建组件的时候，父子组件采用的通信方式可以简单总结为 **Props down, Events up**。即：父组件通过 props 向下对子组件传递数据，子组件通过 events 给父组件发消息。

![父子组件通信示意图](https://vuejs.org/images/props-events.png)

可以看到，子组件是无法对 props 进行修改的，这一特性，保证了数据流是单向的，解耦了父子组件之间的数据，便于维护和管理。但是，在一些特殊的场景，我们还是希望有些局部性数据是双向绑定的，下文给一个这种场景的实例，并给出特定场景下的解决方案。

**注意：** 如果你使用了 vuex 这种状态管理的组件，下文讲述的方法将不再适用。请使用 action 和 mutation 来处理。

## 场景
以一个对话框组件为例，我们这里考虑两个基本功能：
1. 可以用从外部（父组件）对其进行开关
2. 有关闭按钮，可以点击关闭按钮可以关闭

因此，我们需要从父组件传入一个 props，来控制整个对话框显示和隐藏。同时，点击关闭按钮时，需要向父组件发送一个事件，来通知父组件关闭该对话框。思路很简单，写出的代码也很简单：
```html
<template>
  <div class="dialog-container" v-show="show">
    <div class="dialog-header">
      <slot name="header"></slot>
      <!-- 这是个关闭按钮，点它将会关闭 Dialog -->
      <a href="javascript:void(0);">x</a>
    </div>
    <div class="modal-body"><slot></slot></div>
  </div>
</template>
<script>
  export default {
    name: 'MyDialog',
    props: ['show'],
    methods: {
      onClose () {
        this.$emit('on-close')
      }
    }
  }
</script>
```

父组件调用时，也很简单：
```html
<div id="dialog-example">
  <button type="button" @click="show = true"></button>
  <my-dialog :show="show" @on-close="close"></my-dialog>
</div>
```
```javascript
new Vue({
  el: '#dialog-example',
  data: {
    show: false
  },
  methods: {
    close () {
      this.show = false
      console.log('Dialog is closed!')
    }
  }
})
```

需求似乎已经解决，不过有一点小瑕疵：这个 on-close 的事件，并不是当按钮关闭时的回调事件，而是通知父组件来关闭自己的事件（父组件还要通过修改 props 来手动关闭）。这也是单向数据流的设计理念之一：子组件不能修改父组件的属性。为了解决这个问题，我们想到了 Vue 中唯一的 “双向绑定” 指令：`v-model`。

## v-model 的解决方案

## 方案的不足

## 参考资料
