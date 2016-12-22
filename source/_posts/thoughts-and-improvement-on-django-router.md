---
title: 对 Django URL 路由机制的思考与改进
tags:
  - python
  - Django
date: 2016-12-15 02:36:58
---

在 Django 中配置路由看上去是个很简单的事，就是在 urlpatterns 里加一个 url 对象而已。但是，在项目中有上百个路由的时候，维护这个 urlpatterns 就变得分外蛋疼。而且涉及到在多人在同一台服务器协作的时候，又会出现相互覆盖/一个人的代码导致整个服务器崩溃的问题。本文即是讨论这种问题在特定场景下的解决方案。

<!-- more -->

## Django 中的路由

对于大多数项目，Django 的 URL 匹配，基本遵循这个流程：
1. 读取 `<project>/settings.py` 中配置的 `ROOT_URLCONF`, 作为总入口。
2. 读取 `urlpatterns`。这是一个 `django.conf.urls.url()` 的 `list/tuple`.
3. 按顺序匹配 `urlpatterns` 中的每一个 url,匹配成功则停止。如果没有匹配到，进行错误处理。

至此，Django 就成功将一个 URL 对应到了一个 view-function(或 view-class) + 参数 的组合。当然，这里有一些特例，比如 `Request` 对象被中间件设置了 `urlconf` 属性，Django 就会采用这个值做入口。更为详细的说明，可以参照[Django官方文档](https://docs.djangoproject.com/en/1.10/topics/http/urls/#how-django-processes-a-request)。


## Django 路由实际使用中的一些问题

### 开发环境说明

在说明问题之前，首先同步下我们团队采用的开发模式：
1. 远程开发，本地使用脚手架生成/监控代码，并自动上传同步到开发环境。
2. 在开发环境的 uwsgi 下设置自动重启。

选择这样的开发模式，主要是由于开发时使用的 DB 在本机没有连接权限，由于涉及到和不同的开发对接，也不能把 DB 放在本地开发。另外，团队之前不是 python 开发背景，每个人本地 Python 版本也不一致，很多人也没有采用 pip 和 virtualenv 来管理依赖/环境，在服务器可以保证开发环境统一，避免开发完成后提测出现些奇怪的问题。

### 问题

在这种开发模式下，**所有人都工作在同一台服务器上并行开发**。随着项目增大 && 并行进行的任务增多，逐渐出现一些问题：

1. 路由配置经常会相互覆盖。这个问题，多数路由管理的解决方案是一种类似“切片”的方法：对于 Django 来说，就是把 url 配置分配到不同的地方，然后再 include 进来。但是问题并没有彻底解决：如果分片的粒度太大（比如以 APP 为单位），覆盖问题还是会经常出现；分片粒度太小，又难以管理，度比较难把握。

2. 几个模块并行开发时，一个人在自己的 view 文件里写了些错误代码导致 import 时产生异常（比如写了个 SyntaxError），就会导致整个系统都崩掉，影响其他人的开发。

3. 不少 view 的逻辑都很轻（比如只是直接读个 DB 返回），还要配置个 URL，群众表示好麻烦...


## 多人协同模式下的 Django 路由

为了解决这几个问题，经过不断的摸索，我们引入了一套动态加载 VIEW 的机制。大概思路就是根据实际文件路径，通过特定的变换规则，映射到对应的 view_func. 这里需要注意的是，这个变换规则最好不要太复杂，否则后续会变得越来越难维护。

函数具体流程大概分几步：
1. 遍历得到所有 views 文件的路径。
2. 引入所有 views—pack 。
3. inspect 得到所有的 view-func , 根据路径生成了路由。

### 遍历 views 文件
首先用 `os.listdir` 得到的文件夹和 `INSTALLED_APPS` 取一个交集，获取所有需要遍历的 APP。之后 `os.walk` 得到所有的 `.py` 文件（这里要做一些额外的判断，比如子文件夹是不是含有 `__init__.py`）。在我们的项目中，所有的对外视图文件都以 views.py 结尾，这里需要多做一步过滤。另外为了兼容老代码，这里做了一些特例判断，比如如果该 app 下有 `urls.py`,就直接 include, 跳过所有这些逻辑。

### package 引入
将相对于项目跟路径，转化成 dot-style 的包名 ,用 `importlib` 或 `__import__` 引入，在引入过程时用 try/catch 包裹，如果 import 时产生错误，我们会包装一个特定的 “会抛出错误的VIEW”，加到所有正常 URL 匹配的队尾。

```python
# -*- coding: utf-8 -*-

# 注意这段代码只能应用于 Python 2, Python 3 中去掉了 raise 的这个语法，这里需要稍微改写一下。
# 详见： https://docs.python.org/3/reference/simple_stmts.html#the-raise-statement
def reraise(err_type,err_value,err_traceback):
    ''' wrapper the error and reraise in specific view '''
    def wrapper(*args, **kwargs):
        raise err_type, err_value, err_traceback
    return wrapper

# 这里省略了一些必要的初始化代码
try:
    pack = importlib.import_module(view)
except:
    # 这里的路由绑定在包对应的名称
    # 比如这个 view 的路径是 app/something/wow.py
    # 路由就绑定 /app/something/wow
    error_patterns.append(
        url(regex,reraise(*sys.exc_info()))
    )

# 在所有的 VIEWS 遍历结束后，将错误路由放在最后
urlpatterns += error_patterns
```

用 `inspect` 遍历 import 的包，筛选出所有的 view. 我们业务中的 view，为了方便格式化输出数据，都被自定义的一套 `render` 装饰器装饰。我们只要改动下这个装饰器，为被装饰函数加一个标识，即可方便的筛选出需要绑定路由的函数/类。这样的做法，可以做到几乎对业务代码无侵入。

最简单的装饰器，大概长这样：
```python
def route(func):
    @wraps(func)
    def wrapper(*args,**kwargs):
        return func(*args,**kwargs)
    wrapper._auto_urlconf = True
    return wrapper
```
之后我们只需要判断 `getattr(func,'_auto_urlconf',None)` 的值，就知道这个是不是我们需要让用户直接执行的 view 了。
有了 view-func 和对应的路径，我们就可以生成 url 了。然后添加到 urlpatterns, 大功告成！


## 通用性的改进
上面的方法虽然解决了我们项目中的问题，但还不是很通用（比如对 RESTFUL 完全不能支持）。对此我们需要做一些改进。

### 支持自定义子路由与路由名称
把我们的装饰器升级一下，改成这样(省略部分 import 代码)：
```python
UrlConf = namedtuple('UrlConf',['url','name'])
def route(url=None,name=None):
    def resolver(func):
        @wraps(func)
        def wrapper(*args,**kwargs):
            return func(*args,**kwargs)
        wrapper._auto_urlconf = UrlConf(url,name)
        return wrapper
    return resolver
```
让我们的内部变量携带更多信息（子路由/路由名称），之后遍历时，将 `_auto_urlconf` 的路由信息作为一个 **子路由** 追加。这里的子路由的概念大概相当于 django 中的 `include`：比如针对 `app/demo.py` 文件下的 `func`, 指定匹配子路由 `/func/(\d+)/$`，最终将会匹配 `/app/demo/func/(\d+)/$`。

### 支持连字符/下划线/驼峰等多重格式
在绑定 URL 时，不能直接用文件路径，需要进行一步转换。这里为了方便，可以假定所有 函数名都是满足 PEP8 的 snake_style 。

### 支持截断与自定义转化
比如项目所有的 views 定义在 `views.py` 下，又不想让路由中出现 `view` 这种字样，这时就需要在绑定前指定一个 transform 方法，根据统一的 transform 转换路由。

按照上述思路，大致整理了一个 `django-auto-route` 的库。欢迎来 [Github](https://github.com/HakurouKen/django-auto-route/) Fork/Star。

## 结语
注意，这里讨论的对于 Django 路由的 “优化” 是基于我们特定的需求/开发模式下的，并不是银弹。比如，如果需要的 URL 全是较为复杂的 RESTFUL 接口，这种方式就完全不适合。在实际使用中，需要根据自己业务的需求选择。
