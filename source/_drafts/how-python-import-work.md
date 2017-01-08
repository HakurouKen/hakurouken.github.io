---
title: python 的 import 是如何工作的
tags: python
---

本文主要探讨 Python 中 import 的运作机制。在 Python [官方文档](https://docs.python.org/3/reference/simple_stmts.html#import)中，将 Python import 的过程简单分成**查找**和**加载**两步。

对于 import 语法（不含有 from）:
1. 查找对应的模块，加载并在有必要的情况下初始化（一般在初次加载时初始化）
2. 在当前作用域定义相应变量

对于 from ... import 语法：
1. 查找对应的模块，加载并在有必要的情况下初始化
2. 对于每个导入的声明：
    1. 检查导入的模块中有没有对应名称的属性
    2. 如果没有对应的属性，尝试导入具有该名称的子模块
    3. 如果没有找到，引发 ImportError
    4. 如果找到了对应属性/子模块，将对应的引用储存在当前命名空间中
