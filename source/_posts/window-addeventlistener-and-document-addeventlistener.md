---
title: window.addEventListener 和 document.addEventListener 的区别
tags:
  - javascript
date: 2017-10-05 19:42:33
---


当我们需要监听一个"全局" DOM 事件时，往往会用到`window`和`document`这两个对象的`addEventListener`方法。那么，它们两个之间有什么区别与关联呢？

<!-- more -->

## 区别
**在冒泡事件机制中，window 处于 document 的上层。** 这就意味着，对于冒泡事件(例如 click)，`document`上的事件触发后，`window`上相应的事件也会被触发（除非手动阻止冒泡）。比如，如果我们想在点击屏幕任何位置时，触发某个特定事件（例如收起所有已展开的菜单），使用`window.addEventListener`和`document.addEventListener`都可以。一般的习惯是冒泡层级越少越好，因此使用后者的情况居多（当然，实际使用中，可能用`document.body`的情况更多一些）。

![事件流](https://www.w3.org/TR/DOM-Level-3-Events/images/eventflow.svg)

**Window 和 Document 对象本身就是两个不同的事件目标，二者拥有不同的事件。** 在 MDN [EventTarget.addEventListener()](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)上，有相关的描述：
> 事件目标可以是 Document 中的元素、Document 自身、Window 或者任何支持事件的对象（例如 XMLHttpRequest）。

举例来说，`window`上有`resize`和`load`事件，而`DomContentLoaded`事件则在`document`上。更多详细的列表，可以参照[MDN 的 Event Reference](https://developer.mozilla.org/en-US/docs/Web/Events)文档。

## 参考资料
1. [Stackoverflow: Difference between document.addEventListener and window.addEventListener?](https://stackoverflow.com/questions/12045440/difference-between-document-addeventlistener-and-window-addeventlistener)
2. [MDN EventTarget.addEventListener()](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
3. [MDN: Event Reference](https://developer.mozilla.org/en-US/docs/Web/Events)
4. [W3 : DOM-Level-2-Events](https://www.w3.org/TR/DOM-Level-2-Events/events.html)
5. [W3 : DOM-Level-3-Events](https://www.w3.org/TR/DOM-Level-3-Events/#dom-event-architecture)
