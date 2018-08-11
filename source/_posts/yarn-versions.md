---
title: 使用 yarn 管理版本的注意事项
tags:
  - javascript
  - yarn
  - semver
date: 2018-08-05 17:32:42
---


项目背景：我们的业务场景是一个典型的多页应用，我们将其拆散成了多个子项目，由不同的开发团队来维护。对于跨项目之间公用的代码，我们使用发布到内部 npm 源的包来进行管理。

由于我们的项目需要大量的使用到外部依赖，因此对依赖以及深层依赖之间的版本管理非常敏感。为了安装速度和 CI 构建速度，我们选用了 yarn 来进行了版本管理。下面简单介绍我们在版本管理中遇到的两个小问题，以及对应的解决方案。

## ^0.0.x 版本更新问题

我们的业务最近在进行域名切换/收敛以及 https 升级，由于涉及到的域名和场景较多，我们写了一个小的公共 adaptor 来适配各种场景，并将其作为一个独立的 npm 包来进行发布。由于我们业务涉及非常多的域名，我们在完成了核心功能之后，随意发布了一个 0.0.1 版本，以供一些业务先行试点，准备在后续版本迭代中，不断的更新域名配置。但是我们发现，**在 yarn.lock 中，被间接依赖到的`^0.0.2` 和业务中直接用到的 `^0.0.4` 版本并不会 resolve 到同一个版本，而是会分别 resolve 到 0.0.2 和 0.0.4 两个版本**。

在解释这个问题之前，我们需要简单介绍一下 semver 规则。我们在 npm 包中常见的形如 `2.5.12` 和 `16.0.3-rc.1` 这样的版本号，都是满足 semver 标准的版本号。

简单的说，semver 将版本号定义为 `主版本号.次版本号.修订版本号` 的格式。在 semver 的 nodejs 实现 [node-semver](https://github.com/npm/node-semver) 中，额外定义了一个 **版本范围(Version Range)** 的概念：无论是 yarn 还是 npm ，对于依赖是否满足某个范围的判断的判断都是根据这个版本范围的定义（即 `semver.satisfies` 方法）来完成的。

在日常开发中，我们用到最多的版本范围是 **Caret Ranges**（即`^`）和 **Tilde Ranges** 即(`~`)。

`~` 依赖在有次版本号的情况下，仅允许修订版本号升级。否则允许次版本号升级。事实上，在实际应用中，我们几乎不会用到不指定次版本号的情况（形如`~1`这种依赖），多数情况下，我们就可以简单将其理解为允许修订版本升级。
`^` 依赖允许 **不更改最靠左的非零的版本号升级**。即在大于 1.0.0 的版本中，允许次版本号和修订版本号升级；在 0.x.y 的版本中，允许修订版本号升级；在 0.0.x 的版本中，不允许升级。**我们的错误出现在，将其误认为简单的允许次版本号和修订版本号升级。**

为了解决这个问题，我们对后续的内部的 npm 包发布的版本进行了一系列规范：

1. 一个模块在早期实验版本时，可以使用 0.0.x 的格式发布。我们对与 0.0.x 的版本的预期是 **仍然处于实验性质，除特殊场景外，一般不会被外部引用**。
2. 一个模块如果功能就绪之后，应当发布 0.1.0 的版本。我们对于 0.x.y 的版本的预期是 **可用但仍处于开发版本，随时可能变更**。
3. 如果一个包将会被用于生产环境，则请发布 1.0.0 的版本。我们对 x.y.z 的版本的预期是 **稳定且可用**。
4. CLI 工具由于不用考虑被其他的模块间接引用，限制可以相对宽松一些。

## 重复依赖问题

yarn upgrade 在有些情况下，会引入一些重复依赖。假设我们有如下的依赖树：

```
project
 - shared-dependency@^1.0.0
 - other-dependency@^2.0.0
  - shared-dependency@^1.1.0
```

yarn.lock 如下：

```
shared-dependency@^1.0.0, shared-dependency@^1.1.0
  version "1.1.0"
  resolved "..."
```

当 shared-dependency 更新到 1.2.0 之后，我们使用 `yarn upgrade shared-dependency`进行升级，会发现 yarn.lock 变为：

```
shared-dependency@^1.1.0
  version "1.1.0"
  resolved "..."

shared-dependency@^1.2.0
  version "1.2.0"
  resolved ".."
```

我们的项目便引入了两个版本的 `shared-dependency`，同样在打包的时候也会产生两份。

这实际上是 yarn 的一个 BUG（[#3967](https://github.com/yarnpkg/yarn/issues/3967)）。为了解决这个问题，我们不得不 **手动修改 yarn.lock**。如果你的业务能够将变化控制在可控范围内（例如已经将所有不可控的第三方依赖的版本写死，如`vue: "2.4.2"`），可以选择 **将 yarn.lock 和 node_modules 删掉然后重新生成**。当然，在完成这一步操作之后，需要 **人肉确认所有的更改处于可控范围内**，并让测试团队进行上层表现的检查。

## yarn resolutions

yarn 通过识别 `package.json` 中的 `resolutions` 字段，来 **强制指定子依赖的版本**。利用这个规则，我们也能够解决上述两个问题。例如我们在“重复依赖问题”中提到的场景，我们可以通过在 `package.json` 中添加下列字段来解决：

```json
{
  "resolutions": {
    "shared-dependency": "1.2.0"
  }
}
```

resolutions 的使用方法和场景，我们可以参照[官方文档](https://yarnpkg.com/zh-Hans/docs/selective-version-resolutions) 和 [yarn rfcs](https://github.com/yarnpkg/rfcs/blob/master/implemented/0000-selective-versions-resolutions.md)，此处不再赘述。

## 参考资料

1. [语义化版本 2.0.0](https://semver.org/lang/zh-CN/)
2. [Github: node-semver](httpsF://github.com/npm/node-semver)
3. [yarn 选择性依赖解决 官方文档](https://yarnpkg.com/zh-Hans/docs/selective-version-resolutions)
4. [yarn 选择性依赖解决 标准详细说明](https://github.com/yarnpkg/rfcs/blob/master/implemented/0000-selective-versions-resolutions.md)
