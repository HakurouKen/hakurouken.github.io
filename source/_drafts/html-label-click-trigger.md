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
### 阻止 label 的默认行为
```javascript
document.querySelector('label').addEventListener('click', e => {
  e.preventDefault()
})
```
阻止默认行为固然可以解决触发两次的问题，但这也失去了使用 label 的意义，因此不推荐此方案。

### 计时器
基本是万能解法。在每次事件触发后，都和上一次触发事件的时间做对比，如果时间差异过小，就舍弃掉。
```javascript
let lastEvtTime = 0
element.addEventListener('click', e => {
  if (e.timeStamp - lastEvtTime < 16) {
    return
  }
  lastEvtTime = e.timeStamp
  // 一些逻辑代码...
  console.log(e)
})
```
这里用了`event.timeStamp`作为时间来源。如果需要考虑不支持`event.timeStamp`的低版本浏览器，可以用`+new Date()`替代。另外，为了不污染全局环境，可以将`lastEvtTime`放在闭包内：
```javascript
element.addEventListener('click', (function (e) {
  let lastEvtTime = 0
  return function () {
    if (e.timeStamp - lastEvtTime < 16) {
      return
    }
    lastEvtTime = e.timeStamp
    // 一些逻辑代码...
    console.log(e)
  }
})(e))
```
这种做法 hack 的意味很重，并没有在本质上解决这一问题，只是在实际应用中规避了异常情况的发生，但由于泛用性较强，不失为一种解决方案。

### 过滤掉 label 的事件
```javascript
element.addEventListener('click', (e) => {
  if (e.target.tagName === 'label') return
  // 一些逻辑代码
  console.log(e)
})
```
这是严格按照需求执行的过滤，如果我确实需要点击 label 时做一些额外处理，这种方案也就不适用了。

### 只将对应事件绑定在 input 上
这个方案在多数应用场景下都适用，唯一可能出现的问题是，在改造已有代码的时候，需要多做一些调整。

## 参考资料
[MDN <label>](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label)

[W3C 标准](https://www.w3.org/TR/html51/sec-forms.html#the-label-element)
