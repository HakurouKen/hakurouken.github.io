---
title: python 的 import 是如何工作的
tags: python
---

本文主要探讨 Python 中 import 的运作机制。在 Python [官方文档](https://docs.python.org/3/reference/simple_stmts.html#import)中，将 Python import 的过程简单分成**查找/加载**和**赋值**两步。

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

上述两个赋值部分，还要额外考虑 as 子句的情况：如果有 as 子句，则赋值时采用 as 后的名称，否则保持原样。
赋值部分相对比较简单，只需要把已经加载的模块绑定到对应变量即可，下文主要关注模块查找和加载。

## Finder/Loader
在进行整个查找过程之前，首先要引入几个概念：查找器（finder），加载器（loader），导入器（impoter）。需要注意的是，在 Python 3.4 中，对这些概念涉及到的类进行了一些调整，下文会针对不同版本分别说明。
### Finder
Finder 的功能主要是通过名字查找对应 loader，在 Python 中存在两种 finder ：在`sys.meta_path`中使用的“元路径查找器”(meta path finder)和`sys.path_hooks`中使用的“路径入口查找器”(path entry finder,于 3.3 后引入)。
在 3.3 之前，只有`importlib.abc.Finder`一种加载器，在 3.3 版本之后作为`MetaPathFinder`和`PathEntryFinder`的父类，不被直接使用。
Meta path finder 多继承自`importlib.abc.MetaPathFinder`，需要实现`find_module(fullname, path)`，(3.4之前)或`find_spec(fullname, path, target=None)`(3.4及之后)。
Path entry finder 多继承自`importlib.abc.PathEntryFinder`，需要实现`find_loader(fullname)`或(3.4之前)`find_spec(fullname, target=None)`(3.4及之后)。
如果对应的 finder 找到了对应模块，则返回一个对应的 loader(3.4之前)或 module-spec(3.4之后)。
### Loader
Loader 负责加载模块，多继承自`importlib.abc.Loader`，需要实现`load_module(fullname)`(3.4之前)或`exec_module(fullname)`(3.4及之后)。Loader主要可以做下列事情：
1. 执行一些（module 相关）验证
2. 创建一个 module 对象
3. 在模块上设置导入相关属性
4. 将模块“注册”到 sys.modules
5. 执行模块
6. 在加载模块时发生错误时清理现场
### Importer
同时实现了查找和加载的功能（即同时是 finder 和 loader 的对象）。
