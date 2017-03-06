---
title: JavaScript 中的 void 关键字（译）
tags: javascript
date: 2017-03-06 22:21:19
---

> 本文翻译自： http://cmichel.io/javascript-void-keyword/

这是什么语言？
```javascript
void function foo()
{
  // ...
}
```
是的，这是 JavaScript ，我知道标题里写了。但是，我最近才在 ESLint 的[返回值一致性](http://eslint.org/docs/rules/consistent-return)<sup>注①</sup>中知道`void`这个关键字。

<!-- more -->
## JS 中的 void 是干什么的？
`void` 接受一个表达式，执行并返回`undefined`。我认为确实没什么用，但是你可以通过它节省一些代码的书写。

让我们再看看[返回值一致性](http://eslint.org/docs/rules/consistent-return)这个 ESLint 规则。它规定了函数要么 *总是* 返回值，要么永远不返回值（在 JavaScript 中意味着永远返回 undefined）。举个例子，如果你在一个 **Express** 中间件中使用`next()`调用下一个中间件，同时你有不想在此后执行任何代码，你可能会写`return next()`。
```javascript
function middleware(next) {
  somethingAsync((err, value) => {
    // 发生错误时，调用 next 并且从这个回调中返回
    if (err) return next(err)

    // 否则，使用 value 做一些事情
    database.save(value)

    // 这里会隐式返回 undefined
    // => 违反了返回一致性原则，因为这里没有显式的 return ，
    // 但是在错误的分支中，返回了（未知的） next() 的返回值
  })
}
```
然而，错误分支现在（可能）返回一个值，正常回调的执行却不会。由于回调的调用者可能依赖于这个返回值，导致函数丧失了一致性，不是一个好的实践。为了避免这个混乱，我们可以这样写：
```javascript
if (err) {
  next(err)
  return
}
```
或者用`void`关键字，把代码写的短一些：
```javascript
if (err) return void next(err)
```
这样，函数就可以确定返回`undefined`。

## 其它用法
`void`还有一些其它用法，其中一个有趣的应用是与 IIFE (立即执行函数表达式)<sup>注②</sup> 结合，来避免在作用域内泄漏一些不必要的变量。你可以把`function`关键字当作一个表达式而非一个声明，同时加上`()`来立即执行它。
```javascript
var foo = 1
void function() { foo = 2 } {}
console.log(foo) // 2
```
比下面这个好读一些：
```javascript
var foo = 1
(function() { foo = 2}) ()
console.log(foo) // 2
```

## 我们应该使用它么？
在我看来，不应该。这确实是个没什么用的关键字。它可以在某些时候缩短代码，但对于没听说过`void`关键字的人，大大降低了代码的可读性<sup>注③</sup>。

## 译注
1. 原文为：`consistent return`，对应的 ESLint 说明的标题为`require return statements to either always or never specify values (consistent-return)`。
2. 立即执行函数表达式：immediately-invoked function expressions
3. 作者在原文评论中，对是否在开发中使用一个特性，有这样一段评述（同时也是我非常赞同的）:
>（新特性是否应该被使用）取决于：1) 一个特性是否有用，2) 它是否广泛被人接受。我支持使用 ES6 的 Promise，生成器 和 async/await ，尽管他们很复杂（译注：async/await 实际上并不属于 ES6 规范），因为他们确实有用，可以大大简化你的代码，并且没有真正的替代品。但是`void`的应用是在优先，而且几乎不为人所知。同时它可以被函数执行后加一个`return undefined`简单的替换掉。
