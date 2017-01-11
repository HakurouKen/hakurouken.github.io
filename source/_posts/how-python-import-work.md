---
title: python 的 import 是如何工作的
tags: python
date: 2017-01-10 17:15:11
---

本文主要探讨 Python 中 import 的运作机制。在 Python [官方文档](https://docs.python.org/3/reference/simple_stmts.html#import)中，将 Python import 的过程简单分成**查找/加载**和**赋值**两步。

<!-- more -->
对于 import 语法（不含有 from）:
1. 查找对应的模块，加载并(在有必要的情况下)初始化（一般在初次加载时初始化）
2. 在当前作用域定义相应变量

对于 from ... import 语法：
1. 查找对应的模块，加载并(在有必要的情况下)初始化
2. 对于每个导入的声明：
    1. 检查导入的模块中有没有对应名称的属性
    2. 如果没有对应的属性，尝试导入具有该名称的子模块
    3. 如果没有找到，引发 ImportError
    4. 如果找到了对应属性/子模块，将对应的引用储存在当前命名空间中

上述两个赋值部分，还要额外考虑 as 子句的情况：如果有 as 子句，则赋值时采用 as 后的名称，否则保持原样。
赋值部分相对比较简单，只需要把已经加载的模块绑定到对应变量即可，下文主要关注模块查找和加载。另外，**这里的探讨只针对 Python3, 在 Python2 下会有一些不同**。

## Finder/Loader
在进行整个查找过程之前，首先要引入几个概念：查找器（finder），加载器（loader），导入器（impoter）和 ModuleSpec。需要注意的是，在 Python 3.4 中，对这些概念涉及到的类进行了一些调整，下文会针对不同版本分别说明。
### Finder
Finder 的功能主要是通过名字查找对应 loader，在 Python 中存在两种 finder ：在`sys.meta_path`中使用的“元路径查找器”(meta path finder)和`sys.path_hooks`中使用的“路径入口查找器”(path entry finder ，于 3.3 后引入)。
在 3.3 之前，只有`importlib.abc.Finder`一种加载器，在 3.3 版本之后作为`MetaPathFinder`和`PathEntryFinder`的父类，不被直接使用。
Meta path finder 一般继承自`importlib.abc.MetaPathFinder`，需要实现`find_module(fullname, path)`(3.4之前)或`find_spec(fullname, path, target=None)`(3.4及之后)。
Path entry finder 一般继承自`importlib.abc.PathEntryFinder`，需要实现`find_loader(fullname)`或(3.4之前)`find_spec(fullname, target=None)`(3.4及之后)。
如果对应的 finder 找到了对应模块，则返回一个对应的 loader(3.4之前)或 module-spec(3.4之后)。
### Loader
Loader 负责加载模块，一般继承自`importlib.abc.Loader`，需要实现`load_module(fullname)`(3.4之前)或`exec_module(fullname)`(3.4及之后)。Loader主要可以做下列事情：
1. 执行一些（module 相关）验证
2. 创建一个 module 对象
3. 在模块上设置导入相关属性
4. 将模块“注册”到 sys.modules
5. 执行模块
6. 在加载模块时发生错误时清理现场

### Importer
同时实现了查找和加载的功能（即同时是 finder 和 loader 的对象）。
### ModuleSpec
上文提到在 Python 3.4 之后 finder 返回的不是一个 loader，而是 ModuleSpec，它一般是`importlib.machinery.ModuleSpec`的实例或其它结构相同的对象。详细的定义可以参照[官方文档](https://docs.python.org/3/library/importlib.html#importlib.machinery.ModuleSpec)或[PEP-0451](https://www.python.org/dev/peps/pep-0451/#modulespec)的相关说明。

## Python 中的模块查找/加载
有了上文的 finder/loader 的概念，我们来看 Python 中是如何查找和加载一个模块的。
### sys.modules
在 import 搜索第一步会检查`sys.modules`这个 dict 对象，这个变量中缓存了所有曾经引入过的模块。如果这里搜索得到了结果，则直接结束。注意，这个对象是**可写的**，手动添加/修改可能会改变下一次 import 的结果。
### sys.meta_path
如果 sys.modules 中没有对应的 module, 将会搜索 sys.meta_path 变量，这是一个 meta path finder 的 list，通过调用每一个 finder 的`find_spec`方法判断是否找到对应的 module-spec。python3 中默认有三个 finder： BuiltinImporter， FrozenImporter， PathFinder，分别用来导入内建模块，[freeze的模块](https://wiki.python.org/moin/Freeze)，和通用的路径模块，在 Windows 上额外还有一个 WindowsRegistryFinder（特别说明一下，在 Python2 中`sys.meta_path`变量是空数组）。
这里单独说一下 PathFinder，这个是我们默认导入非内建模块时用到的 finder。在[这里](https://hg.python.org/cpython/file/3.6/Lib/importlib/_bootstrap_external.py#11055)我们可以看到它的源码，这里简单说下它的主体执行思路：
1. 遍历`sys.path`（或者指定的 path），对于每一个路径：
    1.1 从`sys.path_importer_cache`取路径对应的 finder，如果没有，进入步骤 1.2
    1.2 按顺序尝试每一个 sys.path_hooks 中的 hooks，直到成功返回（没有抛出 ImportError），将结果缓存在`sys.path_importer_cache`。这些 hooks 要求也返回一个 finder。
    1.3 判断 finder 的合法性，并调用 finder 的 find_spec 方法
    1.4 如果得到一个合法的 spec，跳出循环，否则继续尝试下一个路径
2. 如果 spec 为 None，直接返回
3. 如果 spec 没有对应的 loader，设置 spec.submodule_search_locations ，返回对应的 spec
4. 其他的状况下，直接返回 spec

需要注意的是，一个 import 可能会遍历多次 meta_path 变量，例如`import foo.bar.baz`时，对每个meta path finder(下文以 mpf 指代一个实例)，会先调用`mpf.find_spec("foo",None,None)`，foo 成功导入后，调用`mpf.find_spec("foo.bar",foo.__path__,None)`，之后是 `mpf.find_spec("foo.bar.barz",foo.bar.__path__,None)`。
### 完成查找过程
如果上述都没找到，抛出异常 ImportError(ModuleNotFoundError)。
### 加载
官网给了一段**伪代码**，很好的说明了（在 Python 3.4 之后）整个 Loading 的大致过程，其中 spec 变量指的是 loader 返回的 module-spec：
```python
module = None
if spec.loader is not None and hasattr(spec.loader, 'create_module'):
    # 如果 loader 有 `create_module` 方法，则默认它也有 `exec_module`
    module = spec.loader.create_module(spec)
if module is None:
    module = ModuleType(spec.name)
# The import-related module attributes get set here:
# 设置 import 相关的模块属性，包括 __name__, __loader__ , __package__, __file__ 等等
# 详细可以可以参照官网： https://docs.python.org/3/reference/import.html#import-related-module-attributes
_init_module_attrs(spec, module)

if spec.loader is None:
    if spec.submodule_search_locations is not None:
        # 包命名空间
        sys.modules[spec.name] = module
    else:
        # 不支持
        raise ImportError
elif not hasattr(spec.loader, 'exec_module'):
    module = spec.loader.load_module(spec.name)
    # 如果没有 __loader__ 和 __package__ 属性，重新设置
else:
    sys.modules[spec.name] = module
    try:
        spec.loader.exec_module(module)
    except BaseException:
        try:
            del sys.modules[spec.name]
        except KeyError:
            pass
        raise
return sys.modules[spec.name]
```

## Import Hooks 示例
对于开发者来说，我们用到最多的钩子是`sys.meta_path`和 PathFinder 用到的`sys.path_hook`(或直接修改 `sys.path`)。下面给出一个例子，在引入下列代码之后，在 import 没有找到包时，会提示出与 import 相似的包名。
```python
# -*- coding: utf-8 -*-
from difflib import SequenceMatcher
from collections import namedtuple
import pkgutil
import importlib
import sys

Similarity = namedtuple('Similarity',['a','b','ratio'])

modules = [m.name for m in pkgutil.iter_modules()]

class SimilarFinder(object):
    @classmethod
    def find_spec(self, name, path, target=None):
        # 额外判断，sys.meta_path 中应当有且仅有一个 SimilarFinder 的实例，且在队尾
        index = next((index
            for index,spec in enumerate(sys.meta_path)
                if isinstance(spec,self)),
        None)
        if index != len(sys.meta_path) - 1:
            raise ValueError('There should be exactly 1 SimilarFinder instance at the end of sys.meta_path')

        similarity = Similarity(None,None,0)
        # 在所有比较中，取相似性最大且大于 0.6 的值
        # 0.6 这个经验值来自 Python 的官方文档推荐
        for module in modules:
            ratio = SequenceMatcher(None, module, name).ratio()
            if ratio > similarity.ratio and ratio > 0.6:
                similarity = Similarity(module,name,ratio)
        # 上文已保证这个是最后一个 hook，因此到达这里已经确定是没找到对应的包，直接抛异常即可
        if similarity and similarity.ratio > 0.6:
            raise ModuleNotFoundError('{} is not exists, do you mean {} ?'.format(name, similarity.a))

    @classmethod
    def find_module(self, name, path):
        # 这里做一个 backup，支持 Python 3.4- 的版本
        return self.find_spec(name, path)

sys.meta_path.append(SimilarFinder())
```
在 [liuchang 的 PyCon2015](https://github.com/Liuchang0812/slides/tree/master/pycon2015cn#一些hook示例) 的演示中，有更多详细的示例参考，这里不再举更多的示例。

## 参考资料
官方文档：
[importlib 文档](https://docs.python.org/3/library/importlib.html)
[import 文档](https://docs.python.org/3/reference/import.html)
[sys 模块](https://docs.python.org/3/library/sys.html)
[import 语法声明](https://docs.python.org/3/reference/simple_stmts.html#import)

PEP 文章
[PEP 451 -- A ModuleSpec Type for the Import System](https://www.python.org/dev/peps/pep-0451/)
[PEP 420 -- Implicit Namespace Packages](https://www.python.org/dev/peps/pep-0420/)
[PEP 302 -- New Import Hooks](https://www.python.org/dev/peps/pep-0302/)
[PEP 338 -- Executing modules as scripts](https://www.python.org/dev/peps/pep-0338/)

其它：
[刘畅的 Pycon2015cn 演讲文章](https://github.com/Liuchang0812/slides/tree/master/pycon2015cn)
