---
title: 给非程序员的爬虫教程（一）：编程基础概念
tags:
  - python
  - 非程序员的爬虫教程
---

Python 是一种[解释型语言(Interpreted language)](https://zh.wikipedia.org/wiki/%E7%9B%B4%E8%AD%AF%E8%AA%9E%E8%A8%80)，解释型语言的代码可以直接由机器执行，而无需编译(compile)成机器码。完成一个 Python 程序，一般只需要两步：
1. 在编辑器内写好代码，保存成`.py`后缀的文件。
2. 打开命令行，使用`python xxx.py`的方式执行。如果你配置了 py launcher，也可以双击执行，不过还是建议采用前者的方式，因为这样比较方便在控制台查看我们自定义输出的信息。

下面我们先用几个简单示例程序，来简单介绍一下在编程中，我们用到的一些基础概念。

## Hello World！
当我们接触一门新的编程语言的时候，写的第一个程序往往是输出“Hello World”，这已经成为了一种程序员文化。下面就是一个 Python 的 Hello World 程序：
```python
print("Hello World!")
```
按下回车执行，我们即可得到 "Hello World!" 的输出。

## 函数
上文的 Hello World 程序中，`print`我们称之为 **函数(function)** ，函数接收 n(n>= 0)个用逗号隔开的参数，得到一个计算结果。有时候，我们也称之为 **方法(method)** 。例如这里，`print`接受了一个参数`"Hello World!"`，结果是在控制台中输出了这行对应的文字。

## 表达式
上文的程序片段中，`"Hello World!"`也有一个名字，叫做 **表达式**。和数学中的表达式类似，表达式是一个可以计算的运算组合，这里的计算结果就是`Hello World!`这个字符串，`1+2`,`3 * 5 + 7`等等，都是表达式。

## = 与 ==
在绝大多数编程语言中，= 都是用来进行 **赋值** 的符号。例如代码`x = 2`，表示把`2`的值赋给 x，等号的右边叫做右值(R-value)，多数情况下都是表达式。
如果我们想要判断二者是否相等，一般使用两个等号的`==`，例如`a == 2`。在 Python 中，还有一个更加优雅的语法`is`，我们也可以写成 `a is 2`。不过 is 和 `==` 有些细微的区别，这里不展开细说。

## 缩进与代码块
很少有人给出代码块(code block)的严格定义，不过它的表现非常直观，我们一眼就可以看出来。
```python
result = 1 + 1

if result == 2:
  print('1 + 1 = 2')
  print('Answer is corrent')
else:
  print('Answer is wrong, what happend?')
```
即使你还不会 Python 编程，你也能够大致猜到上文的代码的作用：如果`result`值为2，输出正确的提示文字，否则输出错误的文字。上文第4行和第5行，就构成了一个代码块，这里表示它们分别在上文的`if`作用范围之下。在很多语言中，是采用`{}`来划分的代码块的，但是在 Python 中，代码块是通过 **缩进(indent)** 来标识的。一般情况下，我们应当在所有代码中统一使用 2空格/4空格/8空格/Tab 中的一种进行缩进。注意，Tab 和空格混用也会出现各种问题，所以请 **严格遵守** 这一规范。本文为了排版简洁，统一采用2空格进行缩进，你可以自行选择自己习惯的一种。现代的编辑器，可以帮你智能处理很多缩进的问题，你也可以通过一些编辑器的高级设置来自行定义一些缩进的表现形式（例如将 tab 强制替换为空格等等）。

## 注释
在 Python 中，我们以`#`作为注释的起始，类似 C 语言中的`//`和`/* */`，所有的注释只是给人来阅读的，在实际执行的时候，并不会对其它的代码产生影响。
```python
# 在“#”后的都是注释
print('Hello!') # 注释也可以在一行的末尾

# 写注释是一个好习惯，这样可以方便别人理解你的代码逻辑
# Python 中没有多行注释的概念，不过这并不影响我们手动对注释分行
# 例如这里就是一个多行注释
print('Hello Python!')
```

## 几个有用的方法
### help
我们可以对任意的变量使用`help`方法，python 会自动在控制台打印这个变量的相关文档。例如`help("hello")`，可以输出一长串的帮助文档。
### dir
dir 函数会列出对应参数上所有可用的属性，例如，我们创建了一个`"hello"`的字符串，我们想看可以对它使用哪些方法：
```python
dir("hello")
# 我们可以在 REPL 得到下列输出：
# ['__add__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'capitalize', 'casefold', 'center', 'count', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'format_map', 'index', 'isalnum', 'isalpha', 'isdecimal', 'isdigit', 'isidentifier', 'islower', 'isnumeric', 'isprintable', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']
```
我们得到了一个很长的列表，其中带`__`前后缀的都是内部方法，一般不直接使用，我们一般称之为 **魔术方法(magic method)**。Python 中的方法命名一般浅显易懂，我们可以一眼看出它们的用途，例如：`lower`方法会将字符串的所有字母变成小写；`capitalize`方法会将首字母大写。

## 扩展阅读
1. [解释型语言 - 维基百科](https://zh.wikipedia.org/wiki/%E7%9B%B4%E8%AD%AF%E8%AA%9E%E8%A8%80) 或者 [解释型语言_百度百科](https://baike.baidu.com/item/%E8%A7%A3%E9%87%8A%E5%9E%8B%E8%AF%AD%E8%A8%80)
2. [learnpython.org](http://learnpython.org/)：一个在线的交互式学习 Python 网站
