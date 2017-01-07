---
title: Node.js 的 child_process 模块
tags: javascript
date: 2017-01-07 22:49:09
---


在 Node.js 中，我们可以使用 child_process 模块来创建一个子进程，并在这个子进程中执行一些其它任务（如执行 shell 脚本等）。这个模块包括`exec`,`execFile`,`fork`,`spawn`四个异步方法和`execSync`,`execFileSync`,`spawnSync`三个同步方法以及`ChildProcess`类。详细的使用说明，可以参照[官方文档](https://nodejs.org/api/child_process.html)，本文只会简单分析一下几个代码的执行过程。

<!-- more -->
## 函数接口
在 Github 上可以看到`child_process`模块的[源代码](https://github.com/nodejs/node/blob/master/lib/child_process.js)。从源代码不难看出，`child_process`模块的核心是`ChildProcess`这个对象，模块中所有的函数都是基于它实现（在这里我们只关注异步函数，下同）。在 exec/execFile/fork/spawn 几个函数中，最核心的函数是`spawn`，其它的函数都是对其不同程度的封装：
- `exec`: `execFile`在`shell != false`（默认为`true`）的特殊情况。
- `execFile`: 对`spawn`的 data/close/error 事件做了监听，组装`spawn`的输出数据。
- `fork`: `spawn`的`command`参数为`node`的特殊情况。其中`node`默认取`process.execPath`。
- `spawn`: 新建一个`ChildProcess`对象，调用其`spawn`方法，并返回该对象。

## ChildProcess 类
ChildProcess 的实现在 [lib/internal/child_process.js](https://github.com/nodejs/node/blob/master/lib/internal/child_process.js)，可以看到 ChildProcess 继承自 EventEmitter（[#L229](https://github.com/nodejs/node/blob/master/lib/internal/child_process.js#L229)），因此 ChildProcess 可以像 EventEmitter 一样绑定事件。从 ChildProcess 的[函数声明](https://github.com/nodejs/node/blob/master/lib/internal/child_process.js#L164-L228)可以看出，ChildProcess 的实现主要依赖`_handle`这个内部变量，它是一个`Process`类的实例，来自`process.binding('process_wrap')`。这里的`process.binding`是一个 Node.js 与 C++ 交互的函数，对于这里的`process.binding('process_wrap')`，对应的 C++ 函数调用在 [src/process_wrap.cc](https://github.com/nodejs/node/blob/master/src/process_wrap.cc)。通过函数名，可以直观的看出，node 中的`Process.prototype.spawn`对应 C++ 中的`Spawn`（见[#L109-#L222](https://github.com/nodejs/node/blob/master/src/process_wrap.cc#L109-#L222)）。Spawn 函数的结构大概如下：
```cpp
static void Spawn(const FunctionCallbackInfo<Value>& args) {
  // 变量初始化，options 初始化
  // ...

  int err = uv_spawn(env->event_loop(), &wrap->process_, &options);

  // 错误校验，释放内存
  // ...

  args.GetReturnValue().Set(err);
}
```
这里调用的`uv_spawn`，是 libuv 中封装的 spawn 函数，在 [Unix](https://github.com/libuv/libuv/blob/master/src/unix/process.c#L386) 下的实现和 [Windows](https://github.com/libuv/libuv/blob/master/src/win/process.c#L926) 有所不同。ChildProcess 的整个执行过程至此结束。

## 参考资料
- [Node.js 官方文档](https://nodejs.org/api/child_process.html)
- [Node.js 源码](https://github.com/nodejs/node)
- [libuv](https://github.com/libuv/libuv)
- [Stackoverflow \`process.binding\`](http://stackoverflow.com/questions/24042861/nodejs-what-does-process-binding-mean)
- [Process Bindings: How to do Node.js on the JVM](http://lanceball.com/process-bindings/#/)
