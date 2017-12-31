---
title: 给非程序员的爬虫教程（二）：变量与类型
tags:
  - python
  - 非程序员的爬虫教程
date: 2017-12-31 21:17:14
---

变量(variable)是编程语言中最基础的一个概念，它一般用来储存我们的程序中的某个计算结果，或者是某个抽象概念。我们可以用任何一个表达式给变量赋值，例如：`a = 2`, `b = 99 * 5`, `c = "Hello"` 等等。变量都有一个 **类型(type)** ，比如一部手机的价格，我们一般用数字来表示；而手机品牌名称，则一般用字符串来描述。不同的类型的变量，后续所能进行的操作也不同。
在介绍类型之前，我们先介绍一个内置方法：`type`。type 方法接受一个参数，返回这个参数的类型。我们可以打开一个 REPL，输入`type(8)`，可以看到控制台的返回值是`<type 'int'>`，这里的`int`，就是数字`8`的类型。
和其它编程语言类似，Python 有很多 **内置类型(built-in types)**，我们不会一一进行介绍，这里挑选一些常用的类型展开。
  
<!-- more -->
## 数字
我们在之前，已经举了很多数字的例子，例如上文的`a = 2`, a 的类型就是`int` **整型(Integer)**，我们可以简单把它理解成整数。类似的，小数的类型是`float`，我们称之为 **浮点数(Float)** ，这就是我们最常用的数字类型。你可能会看到一些其它的教程中，还提到了 long 类型，这是一个 Python 2 时代用到的类型，在 Python3 中已经删掉，和 int 进行了统一。值得一提的是，Python 中内置支持了 **复数(complex)** ，不过这里不进行展开，需要的话，可以去查看官方文档的相关说明。

## 文本序列（即字符串）
字符串我们也已经见过，Hello World 程序中唯一的参数`"Hello World"`，就是一个字符串。字符串一般用单引号或双引号包裹，二者没有区别。
```python
greeting = 'hello'

# 我们可以对字符串做一些操作，python 的字符串内置了很多实用的方法
# 例如，我们想要调用首字母大写的方法
capitalized = greeting.capitalize()

# 注意，这里 greeting 变量的值并没有变，这是因为在 Python 中，所有的字符串类型为“不可变”
print(greeting) # 输出 “hello”
# 但是，`greeting.capitalize()`的计算结果`Hello`的值被保存到了 capitalized
print(capitalized) # 输出 "Hello"

# 字符串也可以相加
greeting = greeting + ', world' # 这里可以简写成 `greeting += ', world'`
# 输出 "hello, world"
print(greeting)
```
更多 str 的方法，可以参照[官方文档](https://docs.python.org/3/library/stdtypes.html#string-methods)。我们无需将这些全部记忆下来，只要在需要用到的时候查阅即可。

## 序列
### 列表(list)
最常用的序列(sequence)类型，应该非 **列表(list)** 莫属。下面的 DEMO 展示了一个简单的列表和其用法。
```python
# 键盘的品牌
keyboards = ['filco', 'cherry', 'logitech']
#
# 我们可以通过下标取得列表第 n 个元素，注意编程语言的计数都是从 0 开始
keyboard = keyboards[0]
# keyboard 此时的值为 'filco' 字符串
print(keyboard)
# 我们也可以直接更改 list 中的某个元素
keyboard[2] = 'razer'
# 同样的，列表也有一些实用方法，比如我想向列表末尾追加
keyboards.append('rapoo')
# 我们可以直接打印出整个列表，发现 "rapoo" 已经被追加到列表的末尾
print(keyboards)
```

### 元组(tuple)
与 list 类似，Python 中还有一种不可变的序列，叫做 **元组(tuple)**。这也是一个非常常用的数据类型，不过我们暂时只需要知道，它可以强制转化为 list 即可。
```python
# tuple 和 list 声明的方式类似，只是把中括号变成了小括号
keyborads = ('filco', 'cherry', 'logitech',)
# 通过 list() 方法，强制转化为 list
keyboards = list(keyboards)
# 现在 keyboards 已经是个 list 了
print(keyboards)
```

## 映射
### 字典(dict)
Python 中表达数学中的映射，一般用 **字典(dict)** 这个类型。字典用大括号`{}`包裹，一个典型的字典如下：
```python
person = {
  'name': 'HakurouKen',
  'age': 100,
  'gender': 'male',
  'is_programmer': True
}
```
我们把上面的`'name'`,`'age'`这些称之为 **键(key)** , `'HakurouKen'`, `100` 这些称为 **值(value)**。一个字典就是一系列的 **键值对(k-v pair)**，它们表达从 key 到 value 的映射。我们可以自由的存/取这些值，例如：
```python
name = person['name']
```

## 其它
### 布尔(Boolean)
我们可以简单把布尔值理解为开关，只有 True 和 False。

## 扩展阅读
1. [Python 官方文档：Built-in Types](https://docs.python.org/3/library/stdtypes.html)，或其[中文翻译版](http://python.usyiyi.cn/translate/python_352/library/stdtypes.html)。中文翻译的版本较老，而且有些部分为机翻，并不是非常准确。
2. [Python Programming/Data Types](https://en.wikibooks.org/wiki/Python_Programming/Data_Types)
