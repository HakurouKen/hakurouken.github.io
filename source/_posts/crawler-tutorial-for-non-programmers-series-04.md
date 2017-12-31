---
title: 给非程序员的爬虫教程（四）：函数/类与封装
tags:
  - python
  - 非程序员的爬虫教程
date: 2017-12-31 21:18:58
---

在这章之前，我们已经接触过函数这一概念，我们使用过很多次的`print`就是一个函数。我们称这种 Python 原生提供给我们的函数为 **内置函数(built-in functions)**。我们从 Python 的[官方文档](https://docs.python.org/3/library/functions.html)中，找到所有的内置函数。除了这些内置函数，我们还可以自定义一些函数。

<!-- more -->
## 函数的定义
在 Python 中，我们想要定义一个函数，需要通过`def`这个关键字(对应英文的 define)，大致格式如下：
```
def 函数名(参数1, 参数2, 参数3, ..., 参数n):
  函数具体逻辑
  return 返回值
```
我们来看一个例子:
```python
def statistics(name, elements):
  length = len(elements)
  s = ','.join(elements)
  print('{} has elements, which are {}'.format(length, s))
  return [length, s]

statistics('fruits', ['apple', 'banana', 'peach'])
statistics('keyboards', ['cherry', 'razer', 'filco', 'logitech'])
statistics('vegetables', ['carrot', 'potato', 'tomato'])
```
和数学中的函数`y=f(x)`类似，这里的`name`和`elements`是函数`statistics`的参数，用于指代传进来的值。
函数的参数也可以带默认值：
```python
def sum(num1, num2=0):
  return num1 + num2

sum(1, 5) # 结果为 6
sum(1) # 结果为 1
```
需要注意的是，**有默认值的参数，必须放在没有默认值参数之后。**

## 为什么要有函数
和前面的条件/循环不一样，函数是一个完全抽象的概念，即使没有函数，我们也可以将同样的代码复制若干次来完成逻辑，例如上文的代码，我们可写成：
```python
fruits = ['apple', 'banana', 'peach']
length_fruits = len(fruits)
s_fruits = ','.join(fruits)
print ('{} has elements, which are {}'.format(length_fruits, s_fruits))

keyboards = ['apple', 'banana', 'peach']
length_keyboards = len(keyboards)
s_keyboards = ','.join(keyboards)
print ('{} has elements, which are {}'.format(length_keyboards, s_keyboards))

vegetables = ['apple', 'banana', 'peach']
length_vegetables = len(vegetables)
s_vegetables = ','.join(vegetables)
print ('{} has elements, which are {}'.format(length_vegetables, s_vegetables))
```
对比一下两段代码，我们可以明显看出函数的优点：
1. 简洁易读。代码最终还是要人来维护，因此可读性非常重要。
2. 易修改/扩展。如果我们需要改一下输出的文字（比如文末加个句号），只需要改动一次函数内部逻辑。

具体什么样的逻辑需要封装成函数，这个没有明确的标准，每个人的习惯也都不一样。但是有一条通用的原则，**一旦你开始复制粘贴代码的时候，往往意味着这部分代码需要封装**。

## 类（class)和实例(instance)
对于初学者来讲，类往往是一个很难理解的抽象概念。不过没有关系，我们暂时可以不需要完全理解类的定义。多数简单的爬虫也无需封装成类，但是，我们应当看懂一个类的定义，以及如何使用一个别人封装好的类。我们可以直接使用一个类，但更多的情况下，我们是使用这个类创建的实例(instance)。我们直接从一个例子入手来说明：
```python
# class 是类的标志，下面这段代码的作用，就是定义了一个名为 Person 的类
# 我们习惯将类的首字母大写，用来和函数做一些区分
class Person(object):
  # 几乎所有的类都有 __init__ 方法，这个方法会在对象初始化时，由 Python 内部自动调用，一般不手动调用
  def __init__(self, name, age):
    # 第一个参数默认为 self，代表对象自身
    # self 不是我们手动传入的，是 Python 自身传入的
    # 把传入的 name 和 age 值赋给实例
    self.name = name
    self.age = age

  # 所有的类方法，第一个参数都默认是 self，代表实例自己
  # 同样的，self 是由 Python 自身传入，后面我们调用的时候
  # 后续我们实例上调用 laugh 方法的时候，用 `.laugh()` 来使用
  def laugh(self):
    print('hahaha')

  # 另一个类方法，后续使用类似 `.greeting('name')` 来调用
  def greeting(self, to):
    print('Hello, {}'.format(to))

# 初始化一个 Person 类的实例
person = Person('Jack', 18)
# person.name 的值为 'Jack'
print(person.name)
# 调用实例上的 laugh 方法
# laugh 方法接收 0 个参数(self 参数是 Python 自身传入的)
person.laugh()
# 调用实例上的 greeting 方法
# greeting 方法接收 1 个参数 to
person.greeting('Mark')
```