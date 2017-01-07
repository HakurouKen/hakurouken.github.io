---
title: Node.js 的 child_process 模块
tags: javascript
---

在 Node.js 中，我们可以使用 child_process 模块来创建一个子进程，并在这个子进程中执行一些其它任务（如执行 shell 脚本等）。这个模块包括`exec`,`execFile`,`fork`,`spawn`四个异步方法和`execSync`,`execFileSync`,`spawnSync`三个同步方法以及`ChildProcess`类。详细的使用说明，可以参照[官方文档](https://nodejs.org/api/child_process.html)，本文只会简单分析一下几个代码的执行过程。

## 函数接口
在 Github 上可以看到`child_process`模块的[源代码](https://github.com/nodejs/node/blob/master/lib/child_process.js)。从源代码不难看出，`child_process`模块的核心是`ChildProcess`这个对象，模块中所有的函数都是基于它实现（在这里我们只关注异步函数，下同）。在 exec/execFile/fork/spawn 几个函数中，最核心的函数是`spawn`，其它的函数都是对其不同程度的封装：
- `exec`: `execFile`在`shell != false`（默认为`true`）的特殊情况。
- `execFile`: 对`spawn`的 data/close/error 事件做了监听，组装`spawn`的输出数据。
- `fork`: `spawn`的`command`参数为`node`的特殊情况。其中`node`默认取`process.execPath`。
- `spawn`: 新建一个`ChildProcess`对象，调用其`spawn`方法，并返回该对象。
