---
title: Python 中的 __future__ 模块
tags: python
date: 2016-12-29 21:26:49
---

`__future__` 库是 Python 内置的 “未来模块”，模块中记录了 Python 中引入的一些不向前兼容的修改的特性、引入版本以及合入主干的版本等信息。`__future__` 库从 Python 2.1 开始引入。
有关这个库的探讨，网上有不少相关文章。不过相比较起来，还是官方文档对它的细节描述的最清楚详尽（针对每一个修改都有一篇对应的 PEP 文章说明），这里只对其进行简单的总结，特性的详细说明，可以参照对应的 PEP 文章。

<!--more-->
## 语法与解析过程
先贴出官方的对于 future 语句的语法说明：
```
future_statement ::=  "from" "__future__" "import" feature ["as" name]
                      ("," feature ["as" name])*
                      | "from" "__future__" "import" "(" feature ["as" name]
                      ("," feature ["as" name])* [","] ")"
feature          ::=  identifier
name             ::=  identifier
```
简单的说，就是只支持下列几种格式：
```python
# 普通的方式引入
from __future__ import absolute_import
# 带括号方式引入
from __future__ import (absolute_import,print_function)
# import as 方式
from __future__ import absolute_import as absolute_import_info
```
而对于下列情况不支持：
```
# 这种情况下，什么都不会发生
import __future__
# 下面的情况会报错
# from __future__ import *
```
同时，`__future__` 的引入还必须在模块的（接近）顶部。在 future 语句前，只允许存在：** docstring， 注释，空行 以及其他 future 语句 **。
从上述不同，我们可以发现，`__future__` 模块和普通的 `import` 的工作机制并不一样。`__future__` 模块的[源代码](https://hg.python.org/cpython/file/3.6/Lib/__future__.py)很简单，里面只有一些声明。这些声明对应的语言解析特性，是硬编码在语言解析器中的。详细可以参照 [`Python/future.c`](https://hg.python.org/cpython/file/3.6/Python/future.c)。我们的每个 future 语句的 feature ，在 Python 解析器内部会被转换成对应的 flag，解析器根据有没有设置 flag 进行不同的判断。后续版本将某个特性置为默认后，会直接 PASS 掉对应的 flag ，并去掉相关的判断。
另外对于 CPython 2.7 版本及之前有些特别，由于有一些 future 语句的引入会影响到后续语法树的解析，在 [`Parser/parser.c`](https://hg.python.org/cpython/file/2.7/Parser/parser.c) 解析的过程中，在 `future_hack` 中对一些会对 tokenizer 产生影响的部分（例如 `print_function`）做了特殊处理（设置一些特殊的 flag 作为后续 token 解析的条件）。后续在 3.0 中去掉了相关的部分（由于后续的 future 特性不会影响 token 分析）。

## `__future__` 特性
`__future__` 包是一个“自解释的包”，当我们 import 进来的模块中，包含了特性引入的版本，和将要强制生效的版本，举个例子：
```python
from __future__ import absolute_import
absolute_import
# 输出 _Feature((2, 5, 0, 'alpha', 1), (3, 0, 0, 'alpha', 0), 16384)
```
这说明了 `absolute_import` 是在 2.5.0.a1 版本被引入，将在 3.0 版本合入主干，内部标志位为 16384 (这个我们可以不关注)。
截至 Python 3.6.0, 所有的 `__future__` 中的特性如下：

| 特性名称 | 引入版本 | 强制版本 | 效果简介 | 详细的 PEP 文章 |
| --- | --- | --- | --- | --- | --- |
| nested_scopes | 2.1.0b1 | 2.2 | 嵌套作用域 | [PEP-0227](https://www.python.org/dev/peps/pep-0227/) |
| generators | 2.2.0a1 | 2.3 | 生成器语法 | [PEP-0255](https://www.python.org/dev/peps/pep-0255/) |
| division | 2.2.0a2 | 3.0 | 强制浮点数除法 | [PEP-0238](https://www.python.org/dev/peps/pep-0238/) |
| absolute_import | 2.5.0a1 | 3.0 | 绝对引入 | [PEP-0328](https://www.python.org/dev/peps/pep-0328/) |
| with_statement | 2.5.0.a1 | 2.6 | with 声明 | [PEP-0343](https://www.python.org/dev/peps/pep-343/) |
| print_function | 2.6.0a2 | 3.0 | 强制 print 为函数 | [PEP-3105](https://www.python.org/dev/peps/pep-3105/) |
| unicode_literals | 2.6.0a2 | 3.0 | 默认为 unicode | [PEP-3112](https://www.python.org/dev/peps/pep-3112/) |
| generator_stop | 3.5.0b1 | 3.7 | 终止生成器 | [PEP-0479](https://www.python.org/dev/peps/pep-0479/) |
| barry_as_FLUFL | 3.1.0a2 | 3.9 | `<>` 转为 `!=` | [PEP-0401](https://www.python.org/dev/peps/pep-0401/) |

`barry_as_FLUFL` 这个并没有出现在 Python 的官方文档中（因为是个愚人节彩蛋），不过它确实添加了 `<>` 的语法。另外还有一个彩蛋：如果你不习惯使用缩进，可以用大括号的方式来替代缩进，只需要 `from __future__ import braces` 即可 （然而并不会被解析，抛出 `SyntaxError: not a chance`）。

下面对几个常用的 `__future__` 特性做一些简要说明。

### division
在 Python2 中，两个`int`相除，得到的还是`int`；两个`float`/`int`和`float`相除，得到的都是浮点数。引入`division`，会把所有的除法结果统一为浮点数。如下：
```
In [1]: 5/3
Out[1]: 1

In [2]: from __future__ import division

In [3]: 5/3
Out[3]: 1.6666666666666667
```
### absolute_import
`absolute_import` 看上去是绝对导入，然而它的实际的功能是 **禁用隐式相对导入**。在 PEP0328 中，也强烈建议不要使用隐式相对导入。网上相关的介绍也有很多，下文将用一个具体的例子，说明下隐式相对导入可能带来的一些问题。假设一个这样的目录结构：
```python
module
│  common.py
│  __init__.py
│  __main__.py
|
├─client
│      common.py
│      __init__.py
|      logic.py
│
└─server
        common.py
        __init__.py
        logic.py
```
我们将整个 module 文件夹作为一个模块，执行 `python -m module`，那么对于 `client/logic.py` 文件：

| 隐式相对导入 | 开启 | 关闭 |
| --- | --- | --- |
| `from . import common` | 导入`client.common`模块 | 导入`client.common`模块 |
| `from server import common` | 导入`server.common`模块 | 导入`server.common` 模块 |
| `import server` | 导入`server`模块 | 导入`server`模块 |
| `import common` | 导入`client.common`模块 | 导入`common` 模块 |

可以看出，在开启相对导入后，`import` 产生了一些二义性（`import server`和`import common`表现不同），这也是为什么对于 python2，**强烈建议开启`absolute_import`**的一个原因（当然，升级到 Py3 就没有这个烦恼了）。

### print_function
在 python2 中，`print`既是一个语句又是一个内置函数，引入`print_function`后，将会移除`print`语句。

### unicode_literals
简单的讲，开启`unicode_literals`后，会将 Python2 中的 `s="测试"` 变成 `s=u"测试"`，`unicode`和`str`两种字符串类型也是 Python2 中著名的坑，不再赘述（在 Python3 中已解决此问题）。** 建议开启 **。不过对于有些只接受`str`作为参数的函数（例如`datetime.datetime.strftime`），如果包含超出 ascii 范围的字符，在使用前要先`encode`一下。

## 参考资料
- [Python 官方文档](https://docs.python.org/3/library/__future__.html)
- [Python `__future__` Statement](https://docs.python.org/3/reference/simple_stmts.html#future)
