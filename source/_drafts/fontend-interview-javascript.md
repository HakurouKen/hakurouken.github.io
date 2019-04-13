---
title: Web 前端常见的 Javascript 面试基础题
tags:
  - javascript
  - 面试
---

## == 的隐式转换

根据 ECMA 规范，非严格比较操作符 `==` 会做强制转换，他的表现在 [ECMA 规范中](http://www.ecma-international.org/ecma-262/7.0/index.html#sec-abstract-equality-comparison) 有一段详细的定义。它可以不严谨的抽象成几条规则：

1. `null` 和 `undefined` 相等
2. `Number` 和 `String` 都转换成 `Number` 比较
3. 如果比较两方有 `Boolean`，则转换为 `Number` 比较
4. 如果两边都是对象，转换为基本类型比较

转换为基本类型的过程，同样有详细的定义。对于 **非字符串和数字** 的对象来说，一般都是先调用 `.valueOf`；如果没有得到基本类型，再调用 `.toString`，如果还没有得到基本类型，抛出 `TypeError`。

### 几个常见的例子的比较过程

#### [] == 0

1. 尝试把 `[]` 转换为基本类型，调用 `[].valueOf`得到 `[]`，不满足要求，再调用 `[].toString()` 得到 `""`
2. 比较 `"" == 0`，尝试将 `""` 和 `0` 都转换为数字，结果为真

#### [0] == false

1. 尝试把 `[0]` 转换为基本类型，得到 `"0"`
2. 比较 `"0" == false`，尝试把二者都转为数字，比较 `0 == 0`，结果为真

#### [] == false

1. 同样的，把 `[]` 转为基本类型，得到空字符串 `""`
2. 比较 `"" == false`，把左右都转为数字，结果为真

#### !![] == true

1. 先计算 `!![]`，会尝试吧 `[]` 转为布尔值，得到 `true`，`!!true` 结果仍然是 true
2. `true == true`，结果为真

### 参考资料与扩展阅读

1. [ECMA 中对非严格相等的判定过程](http://www.ecma-international.org/ecma-262/7.0/index.html#sec-abstract-equality-comparison)
2. [ECMA 中的类型转换要求](http://www.ecma-international.org/ecma-262/7.0/index.html#sec-type-conversion)

## javascript 中如何判断类型

ECMA 规范中，共定义了 7 种数据类型，包括：

> 基本类型： String, Nubmer, Boolean, Undefined, Null, Symbol
> 引用类型： Object

### typeof

只能用于判断原始类型 (Primitive value)，且有有些情况下，并不会返回我们预期的结果，例如：

```javascript
typeof null; // => object
typeof []; // => object
typeof new String('123'); // => object
```

### instanceof

检测构造函数的原型是否在对象的原型链上，对于 `a instanceof b`，会判断 `a.__proto__ === b.prototype`，如果不相等，再递归查找 `a` 的原型链，即 `a.__proto__.__proto__`，直至相等或为空。它的缺点是无法判断基础类型，例如：

```javascript
[] instanceof Array; // => true
new Number(1) instanceof Number; // => true
1 instanceof Number; // => false
```

### constructor

constructor 属性，和字面标识的意思一样，会返回 **该对象的构造器的引用**。对于基础类型， `constructor` 指向我们期望的值，且使只读的。但是对于自定义对象，它的值使可以任意修改的。再使用时它判断类型时，需要注意的是：

1. constructor 是一个可被修改的属性，可能会导致不准确
2. null 和 undefined 没有 constructor，会抛出异常

### Object.prototype.toString.call

这种方法是最通用的，也是多数 js 工具库选用的方法。它返回的值是底层的 `[[Class]]` 属性的值，因此，它可以用来区分所有的内置方法，包括 `Date` 和 `HTMLElement` 等等。

```javascript
const toString = Object.prototype.toString;
toString.call(1); // => "[object Number]"
toString.call(new Number(1)); // => "[object Number]"
toString.call(new Date()); // => "[object Date]"
toString.call(document); // => "[object HTMLDocument]"
```

### 参考资料与扩展阅读

1. [JavaScript 中的类型判断，了解一下？](https://juejin.im/post/5b0554c86fb9a07acb3d3ddc)
2. [MDN 类型文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures)
3. [ECMA 中的类型规范定义](http://www.ecma-international.org/ecma-262/7.0/index.html#sec-ecmascript-language-types)

## 原型链

### 原型

以一个例子简单说明原型：

```javascript
function Person(gender) {
  this.gender = gender;
}
const p = new Person('male');

assert.equal(p.__proto__, Person.prototype); // 实例的 __proto__ 属性指向该实例的原型
assert。equal(Object.getPrototypeOf(p), Person.prototype) // 在 ES6 中，引入了 Object.getPrototypeOf 方法，与 __proto__ 一致
assert.equal(Person.prototype.constructor, Person); // 实例原型的 constructor 指向其构造器
```

### 原型链

有了原型之后，我们再来看原型链：`Person.prototype` 本身也是一个由 Object 构造函数生成的，即：

```javascript
assert.equal(Person.prototype, Object.prototype);
```

而 Object.prototype 是 `null`，这是所有原型链查找的结尾。

在了解了上述概念后，我们可以简单用代码模拟 new 的过程，大概是：

```javascript
function createClass(cls, ...args) {
  const o = Object.create(null);
  o.__proto__ = cls.prototype;
  cls.call(o, ...args);
  return o;
}
```

### 继承

在 javascript 中，继承有多种五花八门的实现方式，都各自有优缺点。这里仅描述提一种相对完善的解决方案，也是 MDN 给出的方案：

1. 完成父类的构造函数中的初始化
2. 继承父类原型的所有属性和方法
3. constructor 指向正确

```javascript
// 父类： Person
function Person(name) {
  this.name = name;
}

Person.prototype.say = function() {
  console.log(`Hello, I'm ${this.name}`);
};

// 子类 Teacher
function Teacher(name, subject) {
  // 调用父类的构造函数，完成父类的初始化
  Person.call(this, name);
  this.subject = subject;
}
// 继承原型方法
Teacher.prototype = Object.create(Person.prototype);
// 修复 constructor 的指向
Teacher.prototype.constructor = Teacher;
```

### 参考资料与扩展阅读

1. [JavaScript 深入之从原型到原型链](https://github.com/mqyqingfeng/Blog/issues/2)
2. [MDN 继承与原型链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
3. [JavaScript 中的继承](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects/Inheritance)

## 作用域链(scope chain)

详细请参考 [ECMA 规范的 NewDeclarativeEnvironment](https://www.ecma-international.org/ecma-262/5.1/#sec-10.2.2.2) 相关部分。

### 函数的作用域链

我们仅考虑函数的情况下，作用域链的描述可以简化为：

1. 函数在**定义**的时候，会将定义时的 scope chain 链接到函数对象的 `[[scope]]` 内部属性
2. 函数对象被调用时，创建一个保存所有形参的对象，将此对象作为 scope chain 的最前端，再将函数自身的 scope chain 加入 `[[scope]]`。

除去函数外，下列的情况也能够对作用域链产生影响：

1. try/catch
2. with
3. 块级作用域

### 变量提升

变量提升仅作用于 `var 声明`和 `function 定义`，javascript 在解析代码时，会将。用一段代码可以简单说明：

```javascript
(function() {
  console.log(foo); // => undefined
  var foo = 5;
  console.log(foo); // => 5;

  bar(); // => 'bar'
  function bar() {
    console.log('bar');
  }
})();
```

在变量提升中，函数声明具有最高优先级，它永远提升到当前的作用域最顶部：

```javascript
(function() {
  foo(); // => 2

  var foo = function() {
    console.log(1);
  };

  foo(); // => 1

  function foo() {
    console.log(2);
  }

  foo(); // => 1
})();
```

### 参考资料

1. [JavaScript 深入之词法作用域和动态作用域](https://github.com/mqyqingfeng/Blog/issues/3)

## function 的 call/apply/bind 方法

作用相似，改变 this 的指向，但是三者语法略微不同。

### call 和 apply

二者的作用一致，都是将对应的函数，在指定的 `this` 下执行，并返回结果。二者在 Ï 用法有些稍微的区别，call 接受单个的参数，apply 接受一个参数的数组(或类数组，如 arguments 对象)：

```javascript
function.call(thisArg, arg1, arg2, ...)
function.apply(thisArg, [argsArray])
```

### bind

bind()方法创建一个新的函数，在调用时设置 `this` 关键字为提供的值。并在调用新函数时，将给定参数列表作为原函数的参数序列的前若干项。与 `call` 和 `apply` 不同，它返回的是一个已绑定作用于的函数(偏函数)。

## bind 函数作为构造器使用

以 MDN 上的一段代码为例：

```javascript
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function() {
  return this.x + ',' + this.y;
};

// Point 作为构造器
var p1 = new Point(1, 2);
p1.toString(); // => '1,2'

// bind 之后的函数作为构造器
var YAxisPoint = Point.bind({}, 0);

var p2 = new YAxisPoint(5);
p2.toString(); // => '0,5'
```

## bind 的 polyfill

为了使 `new` 特性能使用，我们的 polyfill 要复杂一些：

```javascript
if (!Function.prototype.bind) {
  (function() {
    var noop = function() {};
    Function.prototype.bind = function(otherThis /* otherArgs */) {
      var func = this;
      var otherArgs = Array.prototype.slice.call(arguments, 1);

      var bound = function(/* restArgs */) {
        var restArgs = Array.prototype.slice.call(arguments);
        func.apply(
          // 对于普通函数和构造函数，采用不同的 this 绑定策略：
          // 普通函数直接绑定到给定的 context
          // 构造函数绑定到当前的 this，在后面单独修改原型链
          noop.prototype.isPrototypeOf(this) ? this : otherThis,
          otherArgs.concat(restArgs)
        );
      };

      // Function.prototype 没有 prototype 属性
      // 这里为了维持继承的原型链，需要额外判断
      if (this.prototype) {
        noop.prototype = this.prototype;
      }
      bound.prototype = new noop();
      return bound;
    };
  })();
}
```

### 参考资料与扩展阅读

1. [MDN Function..prototype.bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind)

## javascript 中的事件循环

### 执行栈

当我们调用一个 javascript 方法时，javascript 引擎都会为我们生成一个 context 上下文，这个上下文中，包括了当前函数代码所在的环境，包括作用域链，this 指向等等执行环境。由于**javascript 是单线程的**，同一时间只能执行一个函数，因此所有的方法和其上下文被存储到一个”执行栈“内排队执行。

当只考虑同步代码的情况下，javascript 会在解析的时候，按照执行顺序初始化当前的执行栈，然后依次执行代码。对于每一个方法，都遵循“初始化”/“执行”/“销毁”这三个步骤。当然，由于函数之间有相互调用，这个栈会在执行过程中不断的产生增减变化。当这个执行栈清空，代表所有代码已经执行完成。

对于异步函数（例如 ajax 请求的回调函数），在事件触发之后，并不是立即执行回调，而是将这个回调加入**事件队列(task queue)**中，等当前的执行栈清空之后，再从队列中取出对应回调，放入执行栈中，执行其中的同步代码。这一过程被称之为**事件循环(event loop)**。

### Task 与 MicroTask

先来看一个面试中经常出现的例子：

```javascript
setTimeout(() => {
  console.log('setTimeout callback');
}, 0);

new Promise((resolve, reject) => {
  console.log('promise callback');
  resolve('promise done');
}).then(val => {
  console.log(val);
});

console.log('sync');
```

它的执行结果是：

```
promise callback
sync
promise done
setTimeout callback
```

为了解释这个表现，我们需要引入一个微任务(microtask) 的概念。**当前执行栈执行完毕时，会优先清空微任务的队列，然后再从任务队列中取一个事件。**

注意：很多人在描述这个概念时，额外引入了一个**宏任务(micro task)** 的概念，这是没有必要的。宏任务就是我们上面提到的任务，只不过一个每一个任务执行前，有一个前置的微任务队列需要清空。

常见的任务：`setTimeout`, `setInterval`
常见的微任务： `promise`, `MutationObserver`

### 参考资料与扩展阅读

1. [详解 JavaScript 中的 Event Loop（事件循环）机制](https://zhuanlan.zhihu.com/p/33058983)
2. [nodejs 中的事件循环](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
3. [JavaScript 深入之执行上下文栈](https://github.com/mqyqingfeng/Blog/issues/4)

## 事件委托

简单的说，事件委托(event.delegate)只有一句话：利用事件冒泡机制，给父元素绑定事件监听，从而监听子元素的变化。当然，捕获(capture) 也可以实现事件委托，不过非常不常用，不展开讲。

### event.target/event.currentTarget

`event.target` 对应实际事件触发的对象。
`event.currentTarget` 对应事件当前冒泡到的节点对象，即事件监听绑定的元素。

### Element.matches

对应的元素是否满足对应的选择器，可以简单理解为“反向的 querySelector”。

### 一个简单的事件代理

有了上述两个概念，我们可以比较方便的实现一个简易的、相对通用的事件代理函数：

```javascript
function delegate(parent, selector, eventName, callback) {
  const actualCallback = event => {
    // 在这里的 currentTarget 和 parent 实际上等价
    const { target, currentTarget } = event;
    let current = target;
    while (current !== currentTarget) {
      if (current.matches(selector)) {
        callback.call(current, event);
      }

      current = current.parentNode;
    }
  };

  parent.addEventListener(eventName, actualCallback);

  // 返回一个“取消监听器”的方法
  return () => {
    parent.removeEventListener(eventName, actualCallback);
  };
}
```

当然，我们还可以继续扩展，包装成类似 `jQuery.delegate` 那样简单易用的方法，例如：

1. `parent` 参数可以考虑兼容 selector 和 HTMLElement
2. `eventName` 支持使用空格隔开的多个事件名
3. 做一些低版本浏览器的兼容，例如兼容 `addEventListener` 和 IE 下的 `attachEvent`；使用类似 sizzle 选择器的方式来替代 `Element.matches` 等等。

### 参考资料与扩展阅读

1. [JavaScript 事件委托详解](https://zhuanlan.zhihu.com/p/26536815)
2. [MDN Element.matches()](https://developer.mozilla.org/en/docs/Web/API/Element/matches)

## 生成器/迭代器

### 迭代器(iterator)

从字面意思，我们就能够知道“迭代器”的作用：用于迭代寻找一个集合中的下一个对象。javascript 中的迭代器有两个要求：

1. 具有 `.next()` 方法，用于返回迭代的下一项。
2. `.next()` 的返回值是 `{value?: any, done: boolean}` 形式，`value` 标识迭代的值；`done`标识是否为最后一个元素。

MDN 上，给出了一个简单的示例函数，下文的 `makeIterator` 的返回就是一个迭代器：

```javascript
function makeIterator(arr) {
  let nextIndex = 0;
  return {
    next() {
      return nextIndex < arr.length
        ? { value: arr[nextIndex++], done: false }
        : { done: true };
    }
  };
}
```

### 生成器

C G AM G F G C

由于手动管理迭代器比较容易出错，javascript 提供了生成器方法(generator function)来生成迭代器，生成器方法生成的结果我们将其称之为生成器(generator)。这个也就是 ES6 中的 `function\*` 和 `yield` 语法。将上面的 `makeIterator` 改写为生成器语法，即：

```javascript
function* makeIterable(arr) {
  for (let i = 0; i < arr.length; i++) {
    yield arr[i];
  }
}
```

### 可迭代对象

可迭代对象：必须实现 @@iterator 方法，这意味着这个对象（或其原型链中的一个对象）必须具有带 `Symbol.iterator` 键的属性。生成器可以被 for...of 语法遍历。javascript 内置的 `Array`, `String`, `Map`, `Set` 等方法，都是可迭代对象。

这里需要注意一个不同，可迭代对象需要满足**可迭代协议**，他和上文的提到的迭代器满足的**迭代器协议**不同。**生成器对象既是可迭代对象，又是迭代器**。

我们可以给任意一个值添加 `Symbol.iterator` 属性，使其变成一个生成器。

```javascript
const myIterable = {};
myIterable[Symbol.iterator] = function*() {
  yield 1;
  yield 2;
  yield 3;
};

for (let value of myIterable) {
  console.log(value);
}
// =>
// 1
// 2
// 3

console.log([...myIterable]); // [1, 2, 3]
```

### 高级生成器

迭代器的 `.next` 方法可以传入值，用于修改生成器的内部状态。这个值将作为 `yield` 的返回值。下面是 MDN 上的一个示例：

```javascript
function* fibonacci() {
  var fn1 = 0;
  var fn2 = 1;
  while (true) {
    var current = fn1;
    fn1 = fn2;
    fn2 = current + fn1;
    var reset = yield current;
    if (reset) {
      fn1 = 0;
      fn2 = 1;
    }
  }
}

var sequence = fibonacci();
console.log(sequence.next().value); // 0
console.log(sequence.next().value); // 1
console.log(sequence.next().value); // 1
console.log(sequence.next().value); // 2
console.log(sequence.next().value); // 3
console.log(sequence.next().value); // 5
console.log(sequence.next().value); // 8
console.log(sequence.next(true).value); // 0
console.log(sequence.next().value); // 1
console.log(sequence.next().value); // 1
console.log(sequence.next().value); // 2
```

### 参考资料与扩展阅读

1. [MDN 迭代器与生成器](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Iterators_and_Generators)
2. [ECMAScript 规范：迭代器相关](http://www.ecma-international.org/ecma-262/7.0/index.html#sec-control-abstraction-objects)
3. [MDN 可迭代协议与迭代器协议](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols)

## 节流与防抖

节流(throttle) 和 防抖(debounce) 是一对经常放在一起的概念，二者都是防止函数频繁触发的。

### 节流(throttle)

节流，我们可以从其英文字面意思理解为“**节流阀**”，即**限制一段时间内，函数只能触发一次**。

节流函数经常使用在限制频繁触发函数的频率的场景，例如无限滚动中，判断滚动条是否触达底部；或者在 resize 时，动态调整某个元素的大小等等。

下面是一个简单的节流函数的实现：

```javascript
function throttle(func, wait = 0) {
  let lastCalledTime = 0;

  return function(...args) {
    const now = Date.now();
    if (now > lastCalledTime + wait) {
      func.call(this, ...args);
      lastCalledTime = now;
    }
  };
}
```

### 防抖(debounce)

防抖函数的作用是“**取消连续触发的调用（抖动）**”。它可以形象的描述为**电梯的开门按钮**，在电梯关门之前（时间范围内），当有人按下按钮（函数调用）时，电梯门会重新打开，如果一段时间没有人按按钮（到达时间阈值），电梯才会上下楼（函数触发）。

防抖最常用的场景就是根据用户的输入，进行模糊搜索。使用防抖函数，可以做到当用户输入告一段落时，再发起搜索请求，避免不必要的请求。

```javascript
function debounce(func, wait) {
  let timer = null;

  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      action.call(this, ...args);
    }, delay);
  };
}
```
