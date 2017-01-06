---
title: Node.js 的 child_process 模块
tags: javascript
---
`execFile`,`exec`,`fork` 都是对 `spawn` 的进一步封装，`spawn` 的实现核心使用了 `ChildProcess`

ChildProcess：
1. new Process() 创建新的 process runtime, Process 来自 process.binding('process_wrap').Process ,调用内部(C++)源码实现
2. 绑定事件，继承自 EventEmitter

`exec`:
`execFile`在 `shell != false`（默认为`true`）的特殊情况。

`execFile`:
对`spawn`的 data/close/error 事件做了监听，组装`spawn`的输出数据。

`fork`:
`spawn`的`command`参数为`node`的特殊情况。其中`node`默认取`process.execPath`。

`spawn`:
新建一个`ChildProcess`对象，调用其`spawn`方法，并返回该对象。
