---
title: 为什么 npm --version 卡住了？
tags:
  - npm
---

最近有开发同学反应，在自己本地升级 node 版本之后，我们的构建工具（基于 webpack 的一个上层封装）运行特别慢。通过测试，我们发现是在检查 npm 版本的时候卡住了，示例代码如下：

```javascript
const childProcess = require('child_process');
const semver = require('semver');
const packageJson = require('../package.json');

function exec(cmd) {
  return childProcess.execSync(cmd).toString().trim();
}

semver.satisfies(exec('npm --version'), packageJson.engines.npm);
```

## 源码
