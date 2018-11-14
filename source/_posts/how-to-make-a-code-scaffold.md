---
title: 如何搭建一个项目脚手架
tags:
  - javascript
date: 2018-05-06 12:31:34
---

最近团队在推行组件的跨项目复用，需要将很多原有的代码单独抽离成包。在实际操作的过程中，我们需要创建很多的子项目，这一过程非常的繁琐，为了减小初始化项目的成本，我们决定写一个团队内部使用的脚手架。

<!-- more -->

## 基本选型

社区内有很多比较成熟的脚手架，比较老的通用脚手架`yeoman`，和用于特定的`vue-cli`和`create-react-app`等等。

以`vue-cli`为例，我们大概将整个脚手架的工作流程拆分成下面几个步骤：

1. 解析输入的命令行参数：这个已经有很多成熟的库，例如 [yargs](https://github.com/yargs/yargs) 和 [commander](https://github.com/commander-rb/commander) 。对于一个脚手架来说，这一部分的工作往往并不复杂，我们也可以不借助第三方库，通过直接分析 `process.argv` 来实现。

2. 获取用户指定的模板：为了方便扩展，脚手架工具往往允许我们将模板当作单独项目管理，这时，我们可以使用 [download-git-repo](https://github.com/flipxfx/download-git-repo) 来下载第三方的模板。对于自用脚手架，我们也可以将模板内置到脚手架内，直接读取本地文件系统即可。

3. 交互式命令行：这里推荐 [inquirer.js](https://github.com/SBoudrias/Inquirer.js/)，这个库几乎覆盖了所有的交互命令行应有的场景，同时还提供了强大的插件能力。需要注意的是，这里的交互式命令行的参数会放在**模板内**而非**脚手架**内（即每个模板会有不同的选项），我们需要根据项目的需求，对原始的 `inquirer.js` 参数进行一层封装。

4. 根据用户输入的参数，将模板渲染为输出：抽象的说，我们要实现一个 `render(template, input) => output` 的函数。模板引擎也有很多可选：尽管绝大多数模板的设计目的都是为了生成 HTML 模板，但是当中的绝大多数都是“字符串模板”，即核心实现方式是字符串拼接，并不会对内容进行过多的解析，因此可以应用于通用场景。这里的一个反例就是`jade`，它对于 HTML 的语法进行了抽象，我们写的所有模板内容都会通过解析生成 HTML，因此不适用。我们在这里选择使用 `ejs`，你也可以根据个人喜好来选择。

## 可能用到的帮助库

[fs-extra](https://github.com/jprichardson/node-fs-extra): 一个用于取代原生`fs`库的方法，它将原生的文件操作方法全部转化为`Promise`的方式，同时添加了一些实用方法（例如`ensureDir`和`copy`）等等。我们还可能用到类似 Python 中 `os.walk` 的递归遍历文件夹的方法，可以尝试采用 [klaw](https://github.com/jprichardson/node-klaw) 或其同步版本 [klaw-sync](https://github.com/manidlou/node-klaw-sync)。

[minimatch](https://github.com/isaacs/minimatch): 判断指定路径是否满足通配符。类似的，我们可能还会用到[globby](https://github.com/sindresorhus/globby)，

[chalk](https://github.com/chalk/chalk): 用于在终端中输出带颜色和格式的文本。

[shelljs](https://github.com/shelljs/shelljs): 用于在 Node.js 中，方便的执行一些 Unix 指令完成的功能。

## 核心代码

### render

我们代码的核心部件，用于“模板 + 数据”到“输出”的转化。借助现成的模板引擎 `ejs`，核心代码非常简单：

```javascript
const fs = require('fs');
const ejs = require('ejs');

/**
 * 模板渲染方法
 * @param {String} template 模板路径
 * @param {Object} data 数据
 * @param {Object} options 渲染选项
 * @returns {String} 模板渲染结果字符串
 */
function render(template, data = {}, options = {}) {
  const { encoding = 'utf-8' } = options;
  const file = fs.readFileSync(template, { encoding });
  return ejs.render(file, data);
}
```

### filter

除了生成文件，还有一步非常重要的操作是过滤。例如我们会通过用户选择“是否使用单元测试”，来忽略整个 `tests` 文件夹。我们会在模板中做类似如下的 map 配置：

```javascript
{
  filter: {
    // answers 参数是用户在交互式命令行中的回答
    'tests/**': answers => answers.test
  }
}
```

根据这个配置，我们可以生成一个自定义的过滤器：

```javascript
const minimatch = require('minimatch');

/**
 * 生成一个自定义过滤器的工厂方法
 * @param {Object} filterConfig 模板的 filter 配置
 * @param {Object} answers 用户在交互式命令行中的回答
 * @returns {Function} 过滤器方法
 */
function createFilter(filterConfig = {}, answers = {}) {
  const filters = Object.keys(filterConfig).map(key => {
    let result = false;
    // 当答案唯一确定之后，所有表达式/函数的结果都已确定，这里只计算一次
    const f = filterConfig[key];
    result = f.call(null, answers);

    return {
      test(filepath) {
        return minimatch(key, filepath);
      },
      result
    };
  });

  return function(filepath) {
    return !filters.some(filter => {
      // 命中一个 glob 匹配，且当前答案为 false 才会被过滤，否则放行
      return filter.test(filepath) && !filter.result;
    });
  };
};
```

如果你看过一些 `vue-cli` 的模版（例如[官方的 webpack 模板](https://github.com/vuejs-templates/webpack/blob/4ffcccb2e3b5c8f18ac5c28023ed983bb183ef96/meta.js#L160-L173)），会发现它支持简单的表达式解析。利用 `eval` 或者 `new Function` ，我们可以将这个功能简单的实现如下：

```javascript
/**
 * 表达式求值
 * @param {String} expression 需要求值的表达式
 * @param {Object} context 上下文
 */
function exec(expression, context = {}) {
  try {
    return (new Function('context', `with (context) { return ${expression}; }`))(context);
  } catch (e) {
    return null;
  }
}
```

我们对`createFilter`进行简单扩展：

```javascript
function createFilter(filterConfig = {}, answers = {}) {
  const filters = Object.keys(filterConfig).map(key => {
    let result = false;
    // 当答案唯一确定之后，所有表达式/函数的结果都已确定，这里只计算一次
    const f = filterConfig[key];
    if (typeof f === 'function') {
      result = f.call(null, answers);
    } else {
      result = exec(f, answers);
    }

    return {
      test(filepath) {
        return minimatch(key, filepath);
      },
      result
    };
  });

  return function(filepath) {
    return !filters.some(filter => {
      // 命中一个 glob 匹配，且当前答案为 false 才会被过滤，否则放行
      return filter.test(filepath) && !filter.result;
    });
  };
};
```

### generator

有了上述方法之后，脚手架生成代码的逻辑可以简化为：

1. 递归读取文件
2. 判断文件是否应当被过滤器过滤
3. 根据 answers 和模板，生成文件并写入

我们将这三个步骤，封装成为 `generator` 方法如下：

```javascript
const path = require('path');
const fsExtra = require('fs-extra');
const klawSync = require('klaw-sync');

function generator(src, dest, options = {}) {
  const {
    filters = {},
    answers = {},
    encoding = 'utf8'
  } = options;
  const filter = createFilter(filters, answers);
  const paths = klawSync(src, {
    nodir: true,
    filter: f => filter(path.relative(src, f.path))
  }).map(pathInfo => {
    const relPath = path.relative(src, pathInfo.path);
    const sourcePath = pathInfo.path;
    const destPath = path.join(dest, relPath);

    return {
      path: relPath,
      source: sourcePath,
      dest: destPath
    };
  });

  paths.forEach(info => {
    const content = render(info.source, answers);
    fsExtra.outputFileSync(info.dest, content, { encoding });
  });
}
```

### creator

在 `generator` 的基础上，我们需要一个更加上层的 `creator` 方法：它直接接受一个模板名，根据这个模板名读取响应的交互式命令，并根据交互式命令的结果生成代码。在此之前，我们需要约定：

1. 所有的模板位于我们脚手架工具的 `templates` 文件夹下。
2. 模板内，用`.template.config.js` 来配置问题和过滤器等参数(上文的`createFilter`函数也要进行改造，将此文件名强制过滤掉)。

```javascript
function creator(templateName) {
  let config;
  try {
    config = require(`../templates/${templateName}/.template.config`);
  } catch (e) {
    config = {};
  }
  const { questions = [], filters = {} } = config;

  return inquire.prompt(questions)
    .then(answers => {
      generator(
        path.join(__dirname, '..', 'templates', templateName),
        path.join(process.cwd(), answers.name),
        {
          answers,
          filters
        }
      );
      return answers;
    });
}
```

### bin

在所有逻辑完成后，我们需要一个简单的命令行封装即可：

```javascript
#!/usr/bin/env node
const fs = require('fs');
const path = require('path');
const yargs = require('yargs');
const chalk = require('chalk');
const creator = require('../lib/creator');

const templates = fs.readdirSync(path.join(__dirname, '..', 'templates'));
const argv = yargs.argv;
const name = argv._[0];
if (templates.indexOf(name) < 0) {
  console.warn(
    chalk.yellow(`${name} 不是一个合法的模版名称，当前仅支持下列模版：\n`),
    templates.join(', ')
  );
} else {
  creator(name)
}
```

至此，一个简单的脚手架就完成了。

## TODO

我们上文介绍的脚手架还很粗糙，只有一些基本功能。在此基础上，我们还可以做很多扩展：

1. DEBUG LOG：在 DEBUG 开关打开时，输出一些 DEBUG 日志，包括性能参数等。
2. 代码生成的运行时钩子：例如我们要实现在项目生成后，自动用 `yarn` 安装所有依赖，就需要一个后置钩子。
3. 支持远程模板：如果脚手架从自用变为需要推广给第三方使用，支持远程（自定义）模板就变得非常重要。可以借助 `download-git-repo` 或其它类似的库，来实现实时下载模板并使用。
