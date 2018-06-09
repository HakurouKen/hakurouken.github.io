---
title: eslint-for-beginner
tags:
- javascript
- eslint
---

ESLint 是当下最流行、最强大的静态代码检查工具。它不仅可以帮我们规范代码格式，同时还能够在帮助我们发现很多代码中潜在的错误。接入 ESLint 是提升项目代码质量的重要一环。

## 安装
```bash
# 使用 npm
npm install eslint --save-dev
# 或使用 yarn
yarn add eslint --dev
```

## eslint 配置
ESLint 的配置全部由 `.eslintrc.js` 或 `.eslintrc.json` 来管理。我们可以使用 `eslint --init` 来在项目根目录下生成一个配置（如果你在项目中局部安装的 ESLint，可以使用 `npx eslint --init`）。
下面是一个 `.eslintrc.js` 的示例：
```javascript
module.exports = {
  // eslint 的执行环境
  "env": {
    "browser": true,
    "commonjs": true,
    "es6": true
  },
  // 继承内置的 eslint:recommend 配置，它包含了一些常用的规则
  "extends": "eslint:recommended",
  // eslint 的语法解析规则
  "parserOptions": {
    "sourceType": "module"
  },
  // 具体的各个规则配置
  "rules": {
    // 缩进为 2 个空格，否则报错
    "indent": [ "error", 2 ],
    // 使用 Unix 的换行符（LF），否则报错
    "linebreak-style": [ 2, "unix" ],
    // 关闭 no-console 检查，可以自由使用 console
    "no-console": [ "off" ],
    // 使用单引号，否则告警
    "quotes": [ "warn", "single" ],
    // 使用行尾分号，否则报错
    "semi": [ "error", "always" ]
  }
};


```
补充说明：
1. 所有的规则(rules)都是一个数组：数组的第一项为等级，分为 `off`(关闭检查),`warn`(检查通过，但给予警告) 和 `error`(检查不通过) 三个等级，这三个等级也可以用 0/1/2 来标识；第二项是具体的配置参数，不同的属性的配置也不同，可以自行查看 eslint 官方文档。
2. ESLint 是一个“完全插件化”的工具，这意味着我们拥极大的自由度，但同时意味着我们需要的配置项也非常多。对于项目，我们一般使用 `extends` 字段，来基于社区现有的预设来进行更改。当下最流行的预设是 [Airbnb](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb-base) 、 [Google](https://github.com/google/eslint-config-google) 以及原生的 `eslint:recommend` 三个。
3. env 标识了 ESLint 执行环境，但是 ESLint 并不会真正执行你的代码，而是通过静态检查来对代码进行约束。例如在 `browser` 为 `false` 的情况（非浏览器环境）下，就不能使用全局的 `window` 变量；同理 `commonjs` 为 `false` 时，也不能使用 `require`。

## 使用
一般的项目脚手架都会帮我们生成配套的 ESLint 设施（包括 eslintrc 配置和相应的 webpack-loader），基本可以做到开箱即用。如果我们需要忽略 eslint 检查，可以在 `.eslintignore` 下添加对应的文件，匹配规则和 `.gitignore` 等是一致的。

在命令行中运行 eslint 也很简单，参照[官方文档](https://eslint.org/docs/user-guide/command-line-interface)，例如 `eslint src/**/*.{js,vue}`。需要特别指出，我们经常使用 `--fix` 参数来自动修正一些简单的错误。

有些时候，我们的 ESLint 规则中限制了一些不能使用的规则（例如 `no-new` 禁止使用 `new`），但是在一些特定的场景下，我们十分清楚它不会带来副作用（例如新建一个 Vue 实例）。这时我们就需要使用行内注释来临时禁止某些 ESLint 规则。下面的例子均来自官方文档：
```javascript
// 禁止所有的 eslint 规则
/* eslint-disable */
alert('foo');
/* eslint-enable */

/****************************************/

// 禁止指定的 eslint 规则
/* eslint-disable no-alert, no-console */
alert('foo');
console.log('bar');
/* eslint-enable no-alert, no-console */

/****************************************/

// 禁止 eslint 规则（单行）

alert('foo'); // eslint-disable-line no-alert, quotes, semi

// eslint-disable-next-line no-alert, quotes, semi
alert('foo');

alert('foo'); /* eslint-disable-line no-alert, quotes, semi */

/* eslint-disable-next-line no-alert, quotes, semi */
alert('foo');
```

## 配合 prettier 使用
上文的说明，基本都可以在 eslint 的官方文档中找到。但是实际使用的时候（尤其是当某个项目的规则不太符合你的个人风格的时候），我们经常会发现手动修改 lint 的报错是一件非常麻烦的事情。这时，我们可以使用 [prettier](https://github.com/prettier/prettier) 来帮助我们进行一系列的自动格式化。有关 prettier 相关的配置，我们不再展开讨论，这里简单介绍一下在主流编辑器中的插件集成。注意，prettier 仅能够帮你修正格式相关的错误，涉及到逻辑改变的（例如 `eqeqeq` 和 `no-console` 规则），仍然需要人工去干预。

### VSCode 下的配置
1. 安装 [vscode-eslint 插件](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
2. 安装 [prettier-vscode 插件](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
3. 在用户设置中，将 `prettier.eslintIntegration` 改为 `true`。
4. eslint 的报错，可以直接在控制台的“问题” TAB 看到；使用快捷键 `Alt + Shift + F`/`Cmd + Shift + F` ，即可使用 prettier 修复当前文件的 eslint 告警（大部分）。
5. 如果需要保存时自动格式化，可以打开 `editor.formatOnSave`。

### Atom 下的配置
1. 安装 [eslint 插件](https://atom.io/packages/eslint)
2. 安装 [prettier-atom 插件](https://atom.io/packages/prettier-atom)
3. 使用 `Cmd + Alt + l`/`Ctrl + Alt + l` 查看所有 eslint 错误；使用`ctrl + alt + f`/`ctrl + options + f` 来进行格式化。默认是勾选了保存时自动格式化的，可以在设置中去掉。

### SublimeText 3 下的配置
1. 安装 [SublimeLinter](http://www.sublimelinter.com/en/stable/)
2. 安装 [SublimeLinter-eslint](https://packagecontrol.io/packages/SublimeLinter-eslint)

需要说明的是，sublime 下的 prettier 插件[JsPrettier](https://packagecontrol.io/packages/JsPrettier)并不支持 eslint 相关配置，因此我们需要手动修正 eslint 的错误。

## 扩展阅读
1. [ESLint Getting Started - 官方文档](https://eslint.org/docs/user-guide/getting-started)
2. [ESLint 配置 - 官方文档](https://eslint.org/docs/user-guide/configuring)
3. [ESLint 在各主流编辑器中的插件集成 - 官方文档](https://eslint.org/docs/user-guide/integrations)
4. [Prettier](http://github.com/prettier/prettier)
