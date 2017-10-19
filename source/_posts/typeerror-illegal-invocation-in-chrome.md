---
title: Chrome 中的 Illegal invocation 错误
tags:
  - javascript
date: 2017-09-22 23:18:09
---

## 问题描述

在使用 `requestAnimationFrame` 方法时，为了封装/兼容上古浏览器，可能会写出如下的代码：
```javascript
const animation = {
  raf: window.requestAnimationFrame ||
        window.mozRequestAnimationFrame ||
        window.webkitRequestAnimationFrame ||
        window.msRequestAnimationFrame ||
        window.oRequestAnimationFrame ||
        function () {
          // requestAnimationFrame polyfill
        }
}

animation.raf(function () {})
```

这段代码，在 chrome 上会给出下列报错：
```
Uncaught TypeError: Illegal invocation
```

<!-- more -->
## 解决方案
对于 BOM 方法（如 setTimeout/alert/scrollTo 等），浏览器要求其执行上下文必须是`window`对象。而上文的代码中，requestAnimationFrame 执行的上下文变成了`animation`对象，因此会报错。解决方案也很简单，我们只需用 call/apply 手动在将执行上下文切换到 window，或者将这些原生方法绑定到`window`上即可：
```javascript
const _nativeRaf = window.requestAnimationFrame ||
      window.mozRequestAnimationFrame ||
      window.webkitRequestAnimationFrame ||
      window.msRequestAnimationFrame ||
      window.oRequestAnimationFrame || null;

const animation = {
  raf: _nativeRaf ? _nativeRaf.bind(window) : function () {}
};
```

## 更多
1. 如果你采用 Firefox 作为开发工具，控制台会给出清晰的报错提示：
```
TypeError: 'requestAnimationFrame' called on an object that does not implement interface Window.
```

2. 在老版本的 Chrome 中，`console`对象下的方法，也要求上下文必须是`console`对象，类似`console.log.apply(null, [args])`的代码也会有同样的报错。

## 参考资料
1. [Chromium Bugs](https://bugs.chromium.org/p/chromium/issues/detail?id=179628)
2. [“Uncaught TypeError: Illegal invocation” in Chrome](https://stackoverflow.com/questions/9677985/uncaught-typeerror-illegal-invocation-in-chrome)
