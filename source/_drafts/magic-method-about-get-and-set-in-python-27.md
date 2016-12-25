---
title: Python 2.7 中 GET/SET 相关的魔术方法
tags: python
---

## 旧式类(经典类)和新式类
由于历史原因，Python 中的类分为旧式类(old-style class)和在 Python 2.2 中引入的新式类(new-style class),引入这个新式类的目的主要是为了统一类(class)和类型(type)。简单的说来，对于新式类中的 `x` 实例，有`type(x) == x.__class__`（在旧式类中二者不一致）。另外，新式类中为对象提供了一套完整的“元模型”（对纯 Python 代码而言，就是一系列的魔术方法），使新式类的功能更强大。**下文讨论的所有 magic-method 全部针对新式类。**

新旧式类的主要区别示例：

```python
# -*- coding: utf-8 -*-
class OldStyleClass:
    pass

old_instance = OldStyleClass()
# 旧式类中 type(o) 和 o.__class__ 的返回并不一样
type(old_instance) # type('instance')
old_instance.__class__ # <class __main__.OldStyleClass at 0x005F0A40>
# 旧式类的对象中，并不包含 __getattribute__ 这种 magic-method
dir(old_instance) # ['__doc__', '__module__']

# 继承自 object 的类就是新式类
class NewStyleClass(object):
    pass
# 新式类也可以使用元类创建，不过在实际编码中不推荐
class AnotherNewStyleClass(object):
    __metaclass__ = type

new_instance = NewStyleClass()
type(new_instance) # <class '__main__.NewStyleClass'>
new_instance.__class__ # <class '__main__.NewStyleClass'>
dir(new_instance) # ['__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__',...]
```

更多有关旧式类和新式类的区别，可以参照 [Python 的官方文档](https://docs.python.org/2/reference/datamodel.html#new-style-and-classic-classes)。
在实际开发中，建议全部使用新式类。另外，在 Python3 中，已经去掉了旧式类的概念，全部都是新式类。


## 魔术方法

1. `object.__getattribute__`
在获取对象的 **任意** 属性时 **无条件** 执行。
```python
# -*- coding: utf-8 -*-
class Person(object):
    ANCIENT = 'ape'

    def __init__(self,sex,age=0):
        self.sex = sex
        self.age = age

    def growup(self):
        self.age += 1

    def __getattribute__(self,name):
        print 'Try to get attribute: `{}`.'.format(name)
        # 这里不能使用 self.name, 否则会导致循环递归，抛出 RuntimeError
        return object.__getattribute__(self,name)

if __name__ == '__main__':
    person = Person('male')
    assert(person.sex == 'male')
    # 这里直接取了 sex 变量，输出:
    #   Try to get attribute: `sex`.
    person.growup()
    # 这里直接取了 growup 变量，在 growup 内又取了 age 变量，因此输出：
    #   Try to get attribute: `growup`.
    #   Try to get attribute `age`.
    try:
        person.haha
    except AttributeError as err:
        print 'Got an AttributeError'
    # 取了不存在的变量 haha，也会触发 __getattribute__
    # 输出：
    #   Try to get attribute `haha`.
    #   Got an AttributeError
    assert(person.ANCIENT == 'ape')
    # 获取类变量时，__getattribute__ 也会触发
    # 输出：
    #   Try to get attribute `ANCIENT`
    assert(Person.ANCIENT == 'ape')
    # __getattribute__ 是对象的属性，从类直接获取类变量，不会调用，没有输出
    assert(person.__class__ == Person)
    # 取内部变量时，__getattribute__ 也会触发
    # 输出:
    #   Try to get attribute `__class__`
```

`__getattribute__` 是相对底层的方法，在实际开发中，很少涉及到(并且也 **不建议** )对其进行重载。如果修改默认的 `__getattribute__`，也会对 `__getattr__` 和描述符的调用产生影响。另外需要注意的是，`__getattribute__` 没有与之对应的 set 方法。

2. `object.__getattr__` 和 `object.__setattr__`
`__getattr__` 在获取对象的属性，且该属性不存在时调用（`__getattribute__`抛出 `AttributeError` 时调用）。
例如，我们打算创建一个 “用点号访问的字典类型” 的类：
```python
# -*- coding: utf-8 -*-
class DotDict(object):
    def __init__(self,**data):
        self._data = data

    def __getattr__(self,name):
        print "__getattr__ called, name: `{}`".format(name)
        return self._data.get(name)

obj = DotDict(key='value')

assert(obj.kwargs == {'key':'value'})
# 对于对象已有的属性，不会执行 __getattr__ ,这里没有任何输出
assert(obj.key == 'value')
# key 不是 obj的一个属性，会调用 __getattr__ ,输出：
#   __getattr__ called, name: `key`
```
另外，如果重写了 `__getattribute__` 方法，导致其不抛出 `AttributeError`(正常返回，或抛出其它错误)，都会导致 `__getattr__` 不执行。

`__setattr__` 是在为对象设定属性值时即会被调用。需要注意的是，这个触发时机和 `__getattr__` 并 **不对称**（却和 `__getattribute__`类似）。
与 `__getattribute__`类似，我们不能在 `__setattr__` 中使用类似 `self.name = value` 这样的语句，否则会导致死循环。另外
我们为上面的 `DotDict` 完善一个 set 方法如下：
```python
# -*- coding: utf-8 -*-
class DotDict(object):
    def __init__(self,**data):
        # 这里如果写 self._data = data , 会导致死循环
        # 需要改成调用 object 的 __setattr__ 方法
        super(DotDict,self).__setattr__('_data', data)

    def __getattr__(self,name):
        print "__getattr__ : {}".format(name)
        return self._data.get(name)

    def __setattr__(self,name,value):
        print "__setattr__: {} = {}".format(name,value)
        self._data[name] = value

obj = DotDict()
# obj.key 触发 __getattr__ 调用，输出：
#   __getattr__ : key
assert(obj.key == None)
# 为 obj 对象设置值，触发 __setattr__, 输出：
#   __setattr__: key = value
obj.key = 'value'
```

3. `getattr` 和 `setattr` (以及 `hasattr`)
`getattr` 、`setattr` 和 `hastattr` 是 Python 的内置函数，他们的表现比较直观:

在不给定 default 时，`getattr(object, name[, default])` 表现和 `object.name` 一致。
给定 default 后，则会将 default 当作属性不存在（即抛出 `AttributeError`）时的默认值。

`setattr(object, name, value)` 和 `object.name = value` 表现一致。

`hasattr(object, name)` 的表现可以用下列的 Python 代码描述：
```python
def hasattr_(obj, name):
    try:
        getattr(obj, name)
        return True
    except AttributeError:
        return False
```

4. `__get__` ， `__set__` 和描述器(Descriptor)
在 Python 中，描述器是指一个访问控制(可以简单理解为getter/setter)被对应的协议方法重写的 **对象属性**。这里的“访问控制”对应的魔术方法有 `__get__(self, obj, type=None)`, `__set__(self, obj, value)` 和 `__delete__(self, obj)`。其中定义了 `__get__` 和 `__set__` 的方法叫做 **资料描述器(data descriptor)**, 只定义了 `__get__` 的方法叫做 **非资料描述器(non-data descriptor)**。
对于 `obj.attr`，如果寻找到的 `attr` 属性定义了 `__get__` 方法，则会按照 **资料描述器 -> 实例变量 -> 非资料描述器 -> __getattr__ （如果有）** 的顺序来查找执行。
描述器的概念相对抽象，这里看一个官方给的描述器的例子：
```python
class RevealAccess(object):
    """A data descriptor that sets and returns values
       normally and prints a message logging their access.
    """

    def __init__(self, initval=None, name='var'):
        self.val = initval
        self.name = name

    def __get__(self, obj, objtype):
        print 'Retrieving', self.name
        return self.val

    def __set__(self, obj, val):
        print 'Updating', self.name
        self.val = val

class MyClass(object):
    x = RevealAccess(10, 'var "x"')
    y = 5

m = MyClass()
assert(m.x == 10) # 输出： Retrieving var "x"
m.x = 20 # 输出： Updating var "x"
assert(m.x == 20) # 输出： Retrieving var "x"
assert(m.y == 5) # y 不是一个描述器，没有输出
```

描述器在 Python 内应用相当广泛，比如常用的 property/staticmethod/classmethod 内部都使用了描述器。更多关于描述器的详细介绍，可以参照[官方文档](https://docs.python.org/2/howto/descriptor.html) ，或其[中文译本](http://pyzh.readthedocs.io/en/latest/Descriptor-HOW-TO-Guide.html)。

5. `__getitem__` , `__setitem__` 和 `__contain__`
这两个魔术方法和其它的魔术方法比较好区分，因为它们是作用于容器类型（例如 `dict/tuple/list`）,在 `self[key]` 时调用。比如 `list` 中的负索引的实现，就是使用了 `__getitem__` 的方法。这里需要注意，内置的 `dict/tuple/list` 等的这个方法不能被直接修改。

```python
class Language(object):
    def __init__(self):
        self.languages = {}

    def __getitem__(self, language):
        ver = self.languages.get(language)
        if not ver:
            print 'There is no language named {}.'.format(language)
        return self.languages.get(language)

    def __setitem__(self, language, version):
        old_version = self.languages.get(language)
        print 'Language update from version {} to {}'.format(old_version or 0,version)
        self.languages[language] = version

lang = Language()
assert(lang['python'] == None)
# 输出: There is no language named python.
lang['python'] = '2.7'
# 输出： Language update from version None to 0 to 2.7
lang['python'] = '3.6'
# 输出： Language update from version None to 2.7 to 3.6
assert(lang['python'] == '3.6')
# 没有输出
```

当我们使用 `in` 时，会优先调用 `__contain__` 方法，如果没有则使用 `__iter__`，最后尝试 `__getitem__`。