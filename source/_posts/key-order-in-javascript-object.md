---
title: javascript 中 Object 的键值顺序
tags: javascript
date: 2017-01-14 14:06:35
---

在 javascript 中，我们通常想要遍历一个对象，通常会有下列几种方式：
    1. Object.keys / Object.values / Object.entries
    1. for-in / for-of
    3. Object.getOwnPropertyNames(obj)
    4. Reflect.ownKeys(obj)
    5. Object.getOwnPropertySymbols(obj)

需要注意的是，在 ES2015 标准之后，keys 除了是 String，还可以是 Symbol（之前只可以是 String）。除此之外其它的 key 值，在设置的时候，会隐式调用 toString 转化为字符串。
针对上述一系列情况，遍历 key 的顺序是怎样的？下文将对这个问题进行一些探究。

<!-- more -->
## ECMAScript 标准
这里的标准主要参考[最新标准（截至目前是 ES2017）](https://tc39.github.io/ecma262/)。
以 Object.keys 为例，Object.keys 执行时，内部的顺序由`EnumerableOwnProperties`方法决定，而这个方法的遍历顺序和[EnumerateObjectProperties](https://tc39.github.io/ecma262/#sec-enumerate-object-properties)保持一致。这个规定中有一句说明：
> EnumerateObjectProperties must obtain the own property keys of the target object by calling its \[\[OwnPropertyKeys\]\] internal method.

对于 Object.getOwnPropertyNames，它对应内部方法`GetOwnPropertyKeys(O, String)`，内部规定了遍历顺序是按照`[[OwnPropertyKeys]]`的顺序。同样的，其它所有的遍历方式中，最后决定顺序的也都是这个`[[OwnPropertyKeys]]`方法，在 ES2016/ES2017 的标准中，这个方法内部指向[OrdinaryOwnPropertyKeys](https://tc39.github.io/ecma262/#sec-ordinaryownpropertykeys)，这里贴出标准的原文：

> When the abstract operation OrdinaryOwnPropertyKeys is called with Object O, the following steps are taken:
> 1. Let keys be a new empty List.
> 2. For each own property key P of O that is an integer index, in ascending numeric index order
>     a. Add P as the last element of keys.
> 3. For each own property key P of O that is a String but is not an integer index, in ascending chronological order of property creation
>     a. Add P as the last element of keys.
> 4. For each own property key P of O that is a Symbol, in ascending chronological order of property creation
>     a. Add P as the last element of keys.
> 5. Return keys.

简单的说就是：
1. 所有可以数字化的 key ，按照数字升序排列
2. 其它字符串的 key 按照插入顺序排列
3. Symbol 按照插入顺序排列

在 ES2015/ES2016 标准中，提到的内部方法名称和调用流程都和上文描述的不尽相同，但最后得到的结论是一致的。同时在 ES5 以及之前的标准中，并没有规定任何遍历的顺序。

以上，是我个人针对标准做出的解读，然而**有关 EnumerateObjectProperties 的参数顺序，标准中描述的比较模糊**。因此，内部调用这个方法获取 keys 的 for-in 的遍历顺序，会有一些不同的说法。下面附上全部原文：
> When the abstract operation EnumerateObjectProperties is called with argument O, the following steps are taken:
>
>    1. Assert: Type(O) is Object.
>    2. Return an Iterator object (25.1.1.2) whose next method iterates over all the String-valued keys of enumerable properties of O. The iterator object is never directly accessible to ECMAScript code. The mechanics and order of enumerating the properties is not specified but must conform to the rules specified below.
>
> The iterator's throw and return methods are null and are never invoked. The iterator's next method processes object properties to determine whether the property key should be returned as an iterator value. Returned property keys do not include keys that are Symbols. Properties of the target object may be deleted during enumeration. A property that is deleted before it is processed by the iterator's next method is ignored. If new properties are added to the target object during enumeration, the newly added properties are not guaranteed to be processed in the active enumeration. A property name will be returned by the iterator's next method at most once in any enumeration.
>
> Enumerating the properties of the target object includes enumerating properties of its prototype, and the prototype of the prototype, and so on, recursively; but a property of a prototype is not processed if it has the same name as a property that has already been processed by the iterator's next method. The values of \[\[Enumerable\]\] attributes are not considered when determining if a property of a prototype object has already been processed. The enumerable property names of prototype objects must be obtained by invoking EnumerateObjectProperties passing the prototype object as the argument. EnumerateObjectProperties must obtain the own property keys of the target object by calling its \[\[OwnPropertyKeys\]\] internal method. Property attributes of the target object must be obtained by calling its \[\[GetOwnProperty\]\] internal method.

标准中特意说明了`The mechanics and order of enumerating the properties is not specified but must conform to the rules specified below.`，即 **参数遍历顺序不指定，只需遵守下述规则**。而“下述规则”中又提到了，获取内部属性时，**必须使用\[\[OwnPropertyKeys\]\]这个方法**。而使用这个方法，得到的参数顺序又是固定的。在 Stackoverflow 和 chromium 的 issuse 中，有很多关于这个规则的探讨，但似乎得到的结论都不一样。这里附上几个对标准的不同解读，供参考。
1. [Stackoverflow: Does JavaScript Guarantee Object Property Order? 中 ftor 的回答](http://stackoverflow.com/questions/5525795/does-javascript-guarantee-object-property-order#answer-38218582)
2. [Stackoverflow: Does JavaScript Guarantee Object Property Order? 中 JMM 的回答](http://stackoverflow.com/questions/5525795/does-javascript-guarantee-object-property-order#answer-32149345)
3. [Stackoverflow: Does ES6 introduce a well-defined order of enumeration for object properties? ](http://stackoverflow.com/questions/30076219/does-es6-introduce-a-well-defined-order-of-enumeration-for-object-properties)
4. [Chromium issuse 中 bret...@gmail.com 的回复](https://bugs.chromium.org/p/v8/issues/detail?id=164#hc148)

## 各个浏览器的实现
ECMAScript 是一个工业先行的标准，最终我们使用时，还要看浏览器是如何实现。这里针对不同的浏览器做下测试，测试代码如下：
```javascript
function forIn(o) {
    var key, keys = [];
    for (key in o) {
        keys.push(key);
    }
    return keys;
}

function objectKeys(o) {
    return Object.keys(o);
}

function getOwnPropertyNames(o) {
    return Object.getOwnPropertyNames(o);
}

function reflectOwnKeys(o) {
    try {
        return Reflect.ownKeys(o);
    } catch(e) {
        return null;
    }
}

var obj = {};
['100', 'a', 2, 'c', {}, '6', 'b'].forEach(function(key){
    obj[key] = 'value';
});

var order = {
    'for-in': forIn(obj),
    'Object.keys': objectKeys(obj),
    getOwnPropertyNames: getOwnPropertyNames(obj),
    'Reflect.ownKeys': reflectOwnKeys(obj)
};

console.log(order);
```
注：
1. 测试平台 Windows 7
2. 这里没有测试 key 为 Symbol 的值，一时因为在实际开发中，使用 Symbol 作为 key 的情况比较少，二是 for-in / Object.keys / getOwnPropertyNames 遍历时也不会获取 Symbol 的 key<del>，三是 IE 还不支持 Symbol</del>。
3. 按照标准中所描述，遍历的预期结果为`["2","6","100","a","c","[object Object]","b",7]`，为了使表格更清晰，结果中用 ✓ 标识这个结果。

| 浏览器 | for-in | Object.keys | getOwnPropertyNames | Reflect.ownKeys |
| --- | :---: | :---: | :---: | :---: |
| Chrome 55.0 | ✓ | ✓ | ✓ | ✓ |
| Firefox 45.0.2 | ✓ | ✓ | ✓ | ✓ |
| IE Edge | ✓ | ✓ | ✓ | 不支持 |
| IE 10 | ✓ | ✓ | ✓ | 不支持 |

可以看到，几乎所有浏览器都按照同样的标准来定义这一规范（甚至包括一些老浏览器）。这里不涉及一些复杂的 case （例如删除 keys 再添加等等），如果需要的话，读者可以自行测试。

## 应用
如果只需要兼容新式主流浏览器，这个顺序基本可以作为可信。如果需要兼容老浏览器，为了确保顺序可信，建议还是采用单属性数组或键值对数组，如：`[{key:'value'},{key2:'value2'}]` 或 `[['key','value'],['key2','value2']]`。
另外给一个参考：在`React.createFragement`方法中，已经依赖了 for-in 遍历的顺序，详见 React [文档](https://facebook.github.io/react/docs/create-fragment.html)和[相关源码](https://github.com/facebook/react/blob/master/src/addons/ReactFragment.js#L65-L82)。

## 参考资料以及扩展阅读
1. [ECMA2015 标准文档](http://www.ecma-international.org/ecma-262/6.0/)
2. [ECMA2016 标准文档](http://www.ecma-international.org/ecma-262/7.0/)
3. [最新ECMA 标准文档](https://tc39.github.io/ecma262/)
4. [Stackoverflow: Does JavaScript Guarantee Object Property Order?](http://stackoverflow.com/questions/5525795/does-javascript-guarantee-object-property-order)
5. [Stackoverflow: Does ES6 introduce a well-defined order of enumeration for object properties? ](http://stackoverflow.com/questions/30076219/does-es6-introduce-a-well-defined-order-of-enumeration-for-object-properties)
6. [Chromium 的一个相关 issuse](https://bugs.chromium.org/p/v8/issues/detail?id=164)
