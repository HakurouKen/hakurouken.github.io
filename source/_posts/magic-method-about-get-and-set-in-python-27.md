---
title: Python 2.7 中 GET/SET 相关的魔术方法
tags: python
date: 2016-12-27 21:35:40
---


Python 中有很多关于 get/set 的魔术方法和内置方法，在官方文档中描述的相对分散，这里做个简单的总结。

<!-- more -->

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

### `object.__getattribute__`
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

### `object.__getattr__` 和 `object.__setattr__`
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

### `getattr` 和 `setattr` (以及 `hasattr`)
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

### `__get__` ， `__set__` 和描述器(Descriptor)
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

### `__getitem__` , `__setitem__` 和 `__contain__`
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


## Python 中的点号调用

Python 中的点号调用，主要的逻辑集中在 `__getattribute__` ，在 `__getattribute__` 执行完毕后，只需要根据是否抛出 `AttributeError` 判断是否执行 `__getattr__`（如果有），之后即可得到最终结果。下文主要描述 `__getattribute__` 的内部实现。

### `__getattribute__` 的执行过程
先上结论：
下面讨论的过程，**只针对于非内建属性**，内建属性(例如`__doc__`)将会直接从 slot 中取，这里不列入考虑。
对于对象`object.__getattribute__` 和类`type.__getattribute__`处理方式稍微有些不同。

对于对象，Python 查找 `obj.attr_name` 的过程如下:
1. 沿着 MRO 寻找 `attr_name` 对应的值（并保存下结果`descr`），如果 `descr` 是资料描述器，执行`__get__(obj,type(obj))`并返回。
2. 在 `obj.__dict__` 中寻找对象属性，如果寻找到了，返回（这里找到的值就是实例变量）。
3. 如果在 步骤1 中 `descr` 为非资料描述器，执行`__get__(obj,type(obj))`并返回。
4. 在 步骤1 中 `descr` 不是个描述器，直接返回（这里就是类变量）。
5. 什么都没找到，抛出 `AttributeError`。

对于类，Python 查找 `Cls.attr_name` 和对象类似，具体的过程如下：
1. 沿着 MRO 寻找 `attr_name` 对应的值（并保存下这个结果 `meta_attribute`），如果是资料描述器，执行 `__get__(Cls,type(Cls))`并返回。
2. 在 `Cls.__dict__` 中寻找对象属性：
    1. 如果找到了属性，且属性为描述器，执行 `__get__(None,Cls)` 并返回结果。
    2. 如果找到了属性，且属性不是描述器，返回结果。
    3. 如果没找到属性，继续。
3. 如果在 步骤1 中的 `meta_attribute` 为非资料描述器，执行 `__get__(Cls, type(Cls))` 并返回。
4. 在 步骤1 `meta_attribute` 中找到的不是描述器，直接返回。
5. 什么都没找到，抛出 `AttributeError`。

可以看到，对象和类的不同处理方式主要在 `__dict__` 中找到的值如果是描述器，需不需要再做处理。

### `__getattribute__` 在 CPython 中的实现

首先看对于对象的实现，在 Python 的官方提供的 [C-API](https://docs.python.org/2/c-api/object.html#c.PyObject_GetAttr) 中，我们可以看到上层的 Python 代码`o.attr_name` 对应 CPython 中底层 C 代码的 `PyObject* PyObject_GetAttr(PyObject *o, PyObject *attr_name)`。

`PyObject_GetAttr` 的完整代码如下(在 `Objects/object.c`)：
```c
PyObject *
PyObject_GetAttr(PyObject *v, PyObject *name)
{
    PyTypeObject *tp = Py_TYPE(v);
    /* 省略了一些字符串/unicode 校验代码 */
    if (tp->tp_getattro != NULL)
        return (*tp->tp_getattro)(v, name);
    if (tp->tp_getattr != NULL)
        return (*tp->tp_getattr)(v, PyString_AS_STRING(name));
    PyErr_Format(PyExc_AttributeError,
                 "'%.50s' object has no attribute '%.400s'",
                 tp->tp_name, PyString_AS_STRING(name));
    return NULL;
}
```

`Py_TYPE` 是一个宏定义，实质是将对象转 `PyObject` 后取 `ob_type`（参见 `Include/object.h`）。这里的 `ob_type` 被称作元类，是一个 `PyTypeObject` 的指针。对于一个 `object` 对象，可以见到它的完整定义：

```c
PyTypeObject PyBaseObject_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "object",                                   /* tp_name */
    sizeof(PyObject),                           /* tp_basicsize */
    0,                                          /* tp_itemsize */
    object_dealloc,                             /* tp_dealloc */
    0,                                          /* tp_print */
    0,                                          /* tp_getattr */
    0,                                          /* tp_setattr */
    0,                                          /* tp_compare */
    object_repr,                                /* tp_repr */
    0,                                          /* tp_as_number */
    0,                                          /* tp_as_sequence */
    0,                                          /* tp_as_mapping */
    (hashfunc)_Py_HashPointer,                  /* tp_hash */
    0,                                          /* tp_call */
    object_str,                                 /* tp_str */
    PyObject_GenericGetAttr,                    /* tp_getattro */
    PyObject_GenericSetAttr,                    /* tp_setattro */
    0,                                          /* tp_as_buffer */
    Py_TPFLAGS_DEFAULT | Py_TPFLAGS_BASETYPE, /* tp_flags */
    PyDoc_STR("The most base type"),            /* tp_doc */
    0,                                          /* tp_traverse */
    0,                                          /* tp_clear */
    0,                                          /* tp_richcompare */
    0,                                          /* tp_weaklistoffset */
    0,                                          /* tp_iter */
    0,                                          /* tp_iternext */
    object_methods,                             /* tp_methods */
    0,                                          /* tp_members */
    object_getsets,                             /* tp_getset */
    0,                                          /* tp_base */
    0,                                          /* tp_dict */
    0,                                          /* tp_descr_get */
    0,                                          /* tp_descr_set */
    0,                                          /* tp_dictoffset */
    object_init,                                /* tp_init */
    PyType_GenericAlloc,                        /* tp_alloc */
    object_new,                                 /* tp_new */
    PyObject_Del,                               /* tp_free */
};
```

在这里，我们需要关注的就是 `PyObject_GenericGetAttr` 这个函数。`__getattribute__` 的核心逻辑就是在这个函数中完成的。在 Python 的 [C-API](https://docs.python.org/2/c-api/object.html#c.PyObject_GenericGetAttr) 中也解释了它的功能：在对象的 MRO 中类的字典中查询对应的描述符，并在 `__dict__` 中查询对应属性（如果存在的话），然后按照 资料描述器 -> 实例属性 -> 非示例属性的顺序获取属性。如果都没有，则会引发 `AttributeError`。

下文的代码演示在 Python 2.7.13 源代码的基础上做了改动，省略掉了一些校验和 `PyObject_GenericGetAttr` 调用 `_PyObject_GenericGetAttrWithDict` 中不会走到的代码流程。
```c
/* Generic GetAttr functions - put these in your tp_[gs]etattro slot */
PyObject *
PyObject_GenericGetAttr(PyObject *obj, PyObject *name)
{
    return _PyObject_GenericGetAttrWithDict(obj, name, NULL);
}

PyObject *
_PyObject_GenericGetAttrWithDict(PyObject *obj, PyObject *name, PyObject *dict)
{
    PyTypeObject *tp = Py_TYPE(obj);
    PyObject *descr = NULL;
    PyObject *res = NULL;
    descrgetfunc f;
    Py_ssize_t dictoffset;
    PyObject **dictptr;

    /* 这里省略掉了 PyString_Check 相关的校验代码 */

    Py_INCREF(name);

    /* 如果 __dict__ 不存在，判断是否初始化，如果初始化失败，直接抛异常 */
    if (tp->tp_dict == NULL) {
        if (PyType_Ready(tp) < 0)
            goto done;
    }

    /*
     * _PyType_Lookup 是一个沿着 MRO 搜索对应名称的内部 API，源代码说明如下：
     * Internal API to look for a name through the MRO.
     * This returns a borrowed reference, and doesn't set an exception!
     */
    descr = _PyType_Lookup(tp, name);
    Py_XINCREF(descr);

    f = NULL;
    /* 如果找到了对应的描述器， 且描述器是资料描述器的话， 执行该描述器的 __get__，结束 */
    if (descr != NULL &&
        PyType_HasFeature(descr->ob_type, Py_TPFLAGS_HAVE_CLASS)) {
        f = descr->ob_type->tp_descr_get;
        if (f != NULL && PyDescr_IsData(descr)) {
            res = f(descr, obj, (PyObject *)obj->ob_type);
            Py_DECREF(descr);
            goto done;
        }
    }

    /*
     * 由于传入的 dict 为 NULL，这里一定会进入这个分支。
     * 可以看出，这里的操作就是把指针移到 __dict__ 区域对应的区域
     */
     if (dict == NULL) {
         /* Inline _PyObject_GetDictPtr */
         dictoffset = tp->tp_dictoffset;
         if (dictoffset != 0) {
             if (dictoffset < 0) {
                 Py_ssize_t tsize;
                 size_t size;

                 tsize = ((PyVarObject *)obj)->ob_size;
                 if (tsize < 0)
                     tsize = -tsize;
                 size = _PyObject_VAR_SIZE(tp, tsize);

                 dictoffset += (long)size;
                 assert(dictoffset > 0);
                 assert(dictoffset % SIZEOF_VOID_P == 0);
             }
             dictptr = (PyObject **) ((char *)obj + dictoffset);
             dict = *dictptr;
         }
     }


     /*
      * 如果有 __dict__ ，直接取对应的 name ，取到就返回。
      */
     if (dict != NULL) {
         Py_INCREF(dict);
         res = PyDict_GetItem(dict, name);
         if (res != NULL) {
             Py_INCREF(res);
             Py_XDECREF(descr);
             Py_DECREF(dict);
             goto done;
         }
         Py_DECREF(dict);
     }


    /* 上文已经对资料描述器做了处理，这里处理非资料描述器 */
    if (f != NULL) {
        res = f(descr, obj, (PyObject *)Py_TYPE(obj));
        Py_DECREF(descr);
        goto done;
    }

    /* f == NULL , 说明沿 MRO 搜索到的是普通属性，直接把它返回 */
    if (descr != NULL) {
        res = descr;
        /* descr was already increfed above */
        goto done;
    }

    /* 上述查找全都失败，抛出 AttributeError */
    PyErr_Format(PyExc_AttributeError,
                 "'%.50s' object has no attribute '%.400s'",
                 tp->tp_name, PyString_AS_STRING(name));
  done:
    Py_DECREF(name);
    return res;
}
```
至此，`__getattribute__` 的过程结束。

对于类，大体和对象是一致的，只不过 `PyObject_GenericGetAttr` 变成了 `type_getattro`，如下：
```c
/* 官方注释也说，它们两个差不多... */
/* This is similar to PyObject_GenericGetAttr(),
   but uses _PyType_Lookup() instead of just looking in type->tp_dict. */
static PyObject *
type_getattro(PyTypeObject *type, PyObject *name)
{
    PyTypeObject *metatype = Py_TYPE(type);
    PyObject *meta_attribute, *attribute;
    descrgetfunc meta_get;

    if (!PyString_Check(name)) {
        PyErr_Format(PyExc_TypeError,
                     "attribute name must be string, not '%.200s'",
                     name->ob_type->tp_name);
        return NULL;
    }

    /* Initialize this type (we'll assume the metatype is initialized) */
    if (type->tp_dict == NULL) {
        if (PyType_Ready(type) < 0)
            return NULL;
    }

    /* No readable descriptor found yet */
    meta_get = NULL;

    /* Look for the attribute in the metatype */
    meta_attribute = _PyType_Lookup(metatype, name);

    if (meta_attribute != NULL) {
        meta_get = Py_TYPE(meta_attribute)->tp_descr_get;

        if (meta_get != NULL && PyDescr_IsData(meta_attribute)) {
            /* Data descriptors implement tp_descr_set to intercept
             * writes. Assume the attribute is not overridden in
             * type's tp_dict (and bases): call the descriptor now.
             */
            return meta_get(meta_attribute, (PyObject *)type,
                            (PyObject *)metatype);
        }
        Py_INCREF(meta_attribute);
    }

    /* No data descriptor found on metatype. Look in tp_dict of this
     * type and its bases */
    attribute = _PyType_Lookup(type, name);
    if (attribute != NULL) {
        /* Implement descriptor functionality, if any */
        descrgetfunc local_get = Py_TYPE(attribute)->tp_descr_get;

        Py_XDECREF(meta_attribute);

        if (local_get != NULL) {
            /* NULL 2nd argument indicates the descriptor was
             * found on the target object itself (or a base)  */
            return local_get(attribute, (PyObject *)NULL,
                             (PyObject *)type);
        }

        Py_INCREF(attribute);
        return attribute;
    }

    /* No attribute found in local __dict__ (or bases): use the
     * descriptor from the metatype, if any */
    if (meta_get != NULL) {
        PyObject *res;
        res = meta_get(meta_attribute, (PyObject *)type,
                       (PyObject *)metatype);
        Py_DECREF(meta_attribute);
        return res;
    }

    /* If an ordinary attribute was found on the metatype, return it now */
    if (meta_attribute != NULL) {
        return meta_attribute;
    }

    /* Give up */
    PyErr_Format(PyExc_AttributeError,
                     "type object '%.50s' has no attribute '%.400s'",
                     type->tp_name, PyString_AS_STRING(name));
    return NULL;
}
```

## 参考资料

官方文档 / HOWTO：
- [Python 内建函数](https://docs.python.org/2/library/functions.html) `getattr` 和 `setattr`
- [Python 中的数据模型](https://docs.python.org/2/reference/datamodel.html)
- [Python 中的内建类型](https://docs.python.org/2/library/stdtypes.html#internal-objects) `Internal-Objects`
- [Python 中 in 操作符](https://docs.python.org/2/reference/expressions.html#membership-test-details)
- [CPython-API 类型对象(PyTypeObject)](https://docs.python.org/2/c-api/type.html)
- [CPython 对象协议](https://docs.python.org/2/c-api/object.html)
- [Python 描述器 HowTo](https://docs.python.org/2/howto/descriptor.html)

Stackoverflow:
- [Python getattribute and setattribute ](http://stackoverflow.com/questions/14787334/confused-with-getattribute-and-setattribute-in-python#answer-14787522)
- [Python `__getattribute__` method and descriptors](http://stackoverflow.com/questions/24863787/python-the-getattribute-method-and-descriptors#answer-24863898)
- [Python `__getattr__` 与 `getattr`](http://stackoverflow.com/questions/1944625/what-is-the-relationship-between-getattr-and-getattr)

其它：
- [Python 描述器 HowTo 翻译](http://pyzh.readthedocs.io/en/latest/Descriptor-HOW-TO-Guide.html)
- [Python 魔术方法指南](http://pycoders-weekly-chinese.readthedocs.io/en/latest/issue6/a-guide-to-pythons-magic-methods.html)
- [python 属性查找](http://blog.jidanke.com/2015/04/21/python-attribute-lookup/)
