---
title: <label> 标签的点击事件
tags: javascript
---

在 Web 中，我们经常使用 label 标签来为 input 标签做标记。用法有两种，一种是用`for`属性：
```html
<label for="price">价格</label><input type="number" id="price">
```
或这直接用`label`包裹对应的`input`:
```html
<label>开关<input type="checkbox"></label>
```
label 的这个特性，可以方便的扩大 input 的点击范围（尤其是对于 checkbox/radio 等点击范围比较小的元素）。但是这一特性，会使得在监听点击事件时需要额外注意一些事情。

## 问题
考虑下列 HTML 结构：
```html
<div id="parent">
  <label for="input">标题</label>
  <input type="checkbox" id="input">
</div>
<script>
  document.getElementById('parent').addEventListener('click', function (evt) {
    console.log(evt.target)
  })
</script>
```
此时，当我们点击 label 标签时，可以看到父元素`#parent`的 click 事件被触发了两次，一次由 label 触发，一次由 input 触发。直接点击 checkbox 则只会触发一次。类似的事情，同样适用于 label 包裹的 input 标签：
```html
<label>
  <span>标题</span>
  <input type="checkbox">
</label>
<script>
  document.querySelector('label').addEventListener('click', evt => {
    console.log(evt.target)
  })
</script>
```

## W3C 标准

## 解决方案

## 参考资料
