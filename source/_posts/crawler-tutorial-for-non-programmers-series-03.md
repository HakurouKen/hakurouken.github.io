---
title: 给非程序员的爬虫教程（三）：条件与循环
tags:
  - python
  - 非程序员的爬虫教程
date: 2017-12-31 21:18:33
---

程序一般是代替人来执行特定的计算的一个工具，编程的本质也就是告诉计算机，应当用什么计算流程来处理我们的数据。这一计算过程，也就是程序的主要逻辑，我们称之为算法。
<!-- more -->

## 流程图(flowchat)
流程图是用来表达算法的一种最简单且直观的形式。我们平时在工作中，为了理清思路，也经常用一些抽象的、非标准的图来描述思考过程，这其实就是流程图的核心。标准流程图的要求相对严格，不过一般情况下，如果只是为了说明或辅助思考，我们没有必要严格按照要求来画这些图。无论是不是标准流程图，对实际结果影响的一般有两个关键点，就是**条件**和**循环**。

## 条件判断
条件判断是个非常自然的逻辑，我们用中文来描述一下条件判断，大致就像这样：
```
如果(if) 某个条件：
  做一些事情
否则(else if) 某个条件：
  做一些事情
否则：
  做一些事情
```

我们来看看一个实际 Python 代码中，条件判断的写法：
```python
# input 是一个 Python 内置的函数，用于接受用户在控制台的输入
# 例如这里，我们会在控制台输出“请输入：”的字样，然后等待用户输入，并按下回车
# 之后将用户的输入，作为字符串，赋值给变量 s
s = input("Laugh：")

if s == "haha":
  print("哈哈")
elif s == "hehe":
  print("呵呵")
elif s == "heihei":
  print("嘿嘿")
else:
  print("吼吼")
```
可以看到，这里的 Python 代码和我们中文的描述基本类似，这就是一个 if 语句的基本用法。
在实际使用中，可以有零个或多个`elif`的部分（"elif" 就是英文单词 "else if" 的简写），同时最后的`else`语句也是可选的 。

## 循环
在实际的编程中，除了条件判断，我们经常需要重复执行某个相同/类似的功能，这时我们就需要用到循环(loop)。
### while 循环
while 循环将在某个表达式为真时，一直重复执行，直到表达式不为真。语法大致如下：
```
while 某个条件:
  执行循环
```
注意，一般在循环执行的时候，条件会不断的进行变化。否则会程序会陷入死循环(infinite loop)。一个简单的循环示例如下：
```python
i = 0
while i < 10:
  print(i)
  i = i + 1
```
程序将会不断输出 i 的值，直到 i >= 10 为止。

### for 循环
理论上，使用 while 循环已经可以处理所有的需要循环的场景，但是对于序列(列表/元组/字符串)，Python 提供了更加便捷的遍历方式，就是 for 循环。
```python
fruits = ['apple', 'banana', 'peach']
for f in fruits:
  print(f)
```

### continue/break
如果我们需要在循环中间跳出循环，就需要用到 continue 和 break 两个语句。break 语句的作用是跳出整个循环，continue 是跳过当次循环。
```python
i = 0
while i < 5:
  i += 1
  if i == 3:
    continue
  print(i)
# 上面这个循环，会输出：1, 2, 4, 5 因为当 i == 3 时，在输出前循环就被跳过

fruits = ['apple', 'banana', 'peach', 'pear']
for f in fruits:
  if f == 'peach':
    break
  print(f)

# 上面这个循环，会输出 apple, banana ,当 f 为 'peach' 时，我们用 break 强制终止了循环
```

## 扩展阅读
[维基百科：流程图](https://en.wikipedia.org/wiki/Flowchart)