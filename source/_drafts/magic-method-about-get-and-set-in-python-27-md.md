---
title: Python 2.7 中 GET/SET 相关的魔术方法
tags: python
---
## Python 2.7 中 GET/SET 相关的魔术方法

### 旧式类(经典类)和新式类
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

### 魔术方法
1. `object.__getattribute__(self,name)`
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
    person.sex
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
    person.ANCIENT
    # 获取类变量时，__getattribute__ 也会触发
    # 输出：
    #   Try to get attribute `ANCIENT`
    Person.ANCIENT
    # __getattribute__ 是对象的属性，从类直接获取类变量，不会调用，没有输出
    person.__class__
    # 取内部变量时，__getattribute__ 也会触发
    # 输出:
    #   Try to get attribute `__class__`
```

`__getattribute__` 是相对底层的方法，在实际开发中，很少涉及到对其进行重载。而且如果修改默认的 `__getattribute__`，会对 `__getattr__` 和描述符的调用产生影响。
