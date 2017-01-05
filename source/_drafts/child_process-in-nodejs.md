---
title: Node.js 的 child_process 模块
tags: javascript
---
`execFile`,`exec`,`fork` 都是对 `spawn` 的进一步封装，`spawn` 的实现核心使用了 `ChildProcess`

ChildProcess：
1. new Process() 创建新的 process runtime, Process 来自 process.binding('process_wrap').Process ,调用内部(C++)源码实现
2. 绑定事件，继承自 EventEmitter
