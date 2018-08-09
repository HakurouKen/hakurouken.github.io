---
title: 如何检查项目中的硬编码
tags:
  - javascript
  - eslint
  - vue
date: 2018-08-09 23:03:24
---


最近在对项目做国际化的适配，其中很重要的一步就是将代码中硬编码的中文提取成多语言的配置。这是一个比较繁琐的工作，一不小心就会有些遗漏。因此，我们准备开发一个简单的工具来辅助我们检查代码中的硬编码。

检查中文并不是一个很复杂的事情，复杂的是如何对注释进行过滤，以及保留每个关键词的信息用于输出。我们决定整合现有的一些现有的工具来辅助我们完成这些事情。

## 利用 babel

在开始之前，首先推荐大家阅读 babel 的[官方插件手册](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md)，它介绍了一些关于 babel 插件开发和 babel 编译相关的基础知识。下文将从这篇文章中，提取一些我们需要了解的概念，进行一个大致介绍。

### AST

AST(Abstract Syntax Tree)即抽象语法树，和我们认知中的树一样，它由若干个节点（Node）构成。Babel 的语法树详细说明可以参照 babel 的[官方文档](https://github.com/babel/babel/blob/master/packages/babel-parser/ast/spec.md)。AST 是一个相对复杂的概念，由于篇幅原因，这里不再对其进行过多的讲解。如果你想对 AST 有一个感性的认识，可以在 [http://astexplorer.net/](http://astexplorer.net/) 输入一些代码来查看真实的 AST 输出结果。babel 默认使用的 语法解析器是 `babylon` （后续将会更名为 `@babel/parser`），是从 `acorn` fork 出来的项目。对比 `acorn`，babylon 的解析结果中多了很多实用的信息（例如每个 token 的行列信息）。

### babel 解析的过程

如果你大概看过 babel 的文档，就可以知道 babel 处理代码分为三个主要步骤：解析(parse), 转换(transform) 和生成(generate)。我们分别用一句话来解释这三步：

1. 解析：通过词法分析，将和语法分析，将我们的源代码解析为 AST 树。
2. 转换：对生成的 AST 进行操作。这一步也是整个 babel 的核心也是最复杂的部分，同时也是各个插件的主要工作范围。
3. 生成：将 AST 再次生成代码。

### 几个会用到的库

1. babylon: babel 的解析引擎。在 babel@7.x 中，更名为 [@babel/parser](https://github.com/babel/babel/tree/master/packages/babel-parser)。
2. babel-types: 一个用于处理 AST 节点的工具库。在 babel@7.x 中，更名为 [@babel/types](https://github.com/babel/babel/tree/master/packages/babel-types)。
3. babel-traverse: 遍历 AST 树的方法。在 babel@7.x 中，更名为 [@babel/traverse](https://github.com/babel/babel/tree/master/packages/babel-traverse)。

### 使用 Visitor 模式访问树的节点

访问 AST 具体的节点，最常用的办法是定义一个“访问者(Visitor)”，这个词的来源是[访问者模式](https://zh.wikipedia.org/zh-hans/%E8%AE%BF%E9%97%AE%E8%80%85%E6%A8%A1%E5%BC%8F)。babel-ast 的一个简单的访问者大概定义如下：

```javascript
const Visitor = {
  Identifier: {
    enter(path) {
      console.log('Entered node: ', path.node);
    },
    exit(path) {
      console.log('Exited from node: ', path.node);
    }
  }
};
```

上述的访问者会在所有的类型为 `Identifier` 的节点被访问的时候被调用。所有的访问者在执行的时候，都可以拿到当前的 `path` ，它标识这当前节点和整个 AST 树中整个节点的路径的关系。我们可以简单把它理解成“上下文”的概念：我们可以通过这个 path 拿到当前节点自身、临近节点、根节点、作用域等等所有的信息。

### 代码

有了上述的基础知识，我们可以很容易有一个大致思路：

1. 使用 babylon 将代码转换成 AST 树
2. 定义这个 AST 树的一个所有标识符（Identifier）的访问者，取所有的字符串/数字类型，然后根据我们指定的条件过滤，将结果 push 到一个外部的数组。注意这里还要简单过滤一些特殊的情况，例如 `import` 和 `require`。
3. 输出所有的结果

下面是一段简单的 DEMO 代码：

```javascript
import babylon from 'babylon';
import traverse from 'babel-traverse';
import t from 'babel-types';

function check(code, isHardCode = () => false, plugins = []) {
  const hardcodes = [];
  const ast = babylon.parse(code, { sourceType: 'module', plugins });
  traverse(ast, {
    enter(path) {
      const node = path.node;
      // 在基本类型中，只有 文本/数字 才算硬编码
      if (t.isStringLiteral(node) || t.isNumericLiteral(node)) {
        if (isHardCode(node.value, path)) {
          const loc = node.loc;
          hardcodes.push({
            value: node.value,
            start: loc.start,
            end: loc.end
          });
        }
      } else if (t.isTemplateElement(node)) {
        if (isHardCode(node.value.cooked, path)) {
          const loc = node.loc;
          hardcodes.push({
            value: node.value.cooked,
            start: loc.start,
            end: loc.end
          });
        }
      }
    }
  });

  return hardcodes;
};

/**
 * 调用：
 *
 *  check(`const cn = "中文";`, value => /[\u4e00-\u9fa5]/.test(value));
 *
 * 输出：
 *
 * [
 *  {
 *    value: '中文',
 *    start: { line: 1, column: 11 },
 *    end: { line: 1, column: 15 }
 *  }
 * ]
 */

```

这里只处理了基本类型，如果你需要处理 jsx 等等，需要自己传入对应的 babel-plugin，之后去对于 JSX 的相关类型进行判断即可。思路完全一致，不再赘述。

### 需要注意的问题

1. babel 的 7.x 版本仍处于 beta 版。7.x 版本的很多包明都发生了变化，将所有的 npm 包都加上了 @babel 的 scope，例如 `babylon` 对应 `@babel/parser`、`babel-core` 对应 `@babel/core` 等。建议我们的项目还维持在 6.x 的版本，待 7.x 稳定后再考虑升级。
2. 如果你确定要使用 babel 的 7.x 版本，且使用了 `stage-2` 以上的特性，可能会出现 [#7786](https://github.com/babel/babel/issues/7786) 类似的报错。我们需要稍微修改一下我们的 `.babelrc` 文件。
3. 上述的 DEMO 代码只用于过滤一个 js 文件，我们需要对项目下的所有 js(x) 进行一个遍历，同时过滤掉我们不需要的文件（例如 i18n 的语言包目录等）。

### 未解决的问题

1. 这个脚本比较难与我们现有的工作流集成，只能作为一个旁路脚本，在构建/发布前进行检查。
2. 过滤只能基于文件的维度增加特例，在某些时候我们确实需要临时打破规则 ，就不得不把定义都放到外部文件然后再引入，不但繁琐，而且不利于维护。
3. babel 本身无法处理 `.vue` 文件。我们当然可以参照 [vue-loader 的源代码](https://github.com/vuejs/vue-loader/blob/master/lib/loaders/templateLoader.js)，使用 [@vue/component-compiler-utils](https://github.com/vuejs/component-compiler-utils) 来对 .vue 文件进行一个预处理，然后再使用 babel 解析，但是这样做会丢失掉所有 template 中的行数信息（同时 script 中的行数信息也需要自己修正）。

## eslint 插件

提到静态检查，我们应该会马上想到 eslint，eslint 的配置化也使得我们通过自定义插件来完成一些校验成为可能。

### eslint 配置组成部分

我们以 Vue 的 eslint 的推荐配置 [plugin:vue/base](https://github.com/vuejs/eslint-plugin-vue/blob/master/lib/configs/base.js) 为例，描述一个完整的 eslintrc 的配置大致需要那些部分：

```javascript
module.exports = {
  root: true,
  // 解析器相关配置
  parser: 'vue-eslint-parser',
  parserOptions: {
    ecmaVersion: 2015,
    sourceType: 'module',
    ecmaFeatures: {
      jsx: true,
      experimentalObjectRestSpread: true
    }
  },
  // 环境配置
  env: {
    browser: true,
    es6: true
  },
  // 插件配置
  plugins: [
    'vue'
  ],
  // 具体的规则配置
  rules: {
    'vue/comment-directive': 'error',
    'vue/jsx-uses-vars': 'error'
  }
}
```

#### 解析器（Parser）

解析器作用是将我们的 javascript 代码解析为 AST 树，并为我们的自定义规则提供一些作用域管理的机制以及一些帮助方法。需要注意的是，一份 eslint 配置只有一个 parser。写一个 parser 是一个非常繁琐的工作，但是 **绝大多数情况下，我们都无需自己手动编写一个 parser**。 对于绝大多数项目，使用 `babel-eslint` 或 `vue-eslint-parser`(仅针对 Vue 项目)都足以满足需求。更多有关 parser 的详细信息，可以参考官方文档[Working with Custom Parsers](https://eslint.org/docs/developer-guide/working-with-custom-parsers)。

#### 规则（Rule）

使用 eslint 时，我们大部分的关注点都在如何配置各种规则。`.eslintrc`中的 rules 只是一些简单的选项，具体的 rules 逻辑由 eslint 的插件（以及内置插件）来支持。

#### 插件（Plugin）

eslint 强大的插件机制保证了它的扩展性，一个 eslint 插件的主入口大概结构如下：

```javascript
module.exports = {
  // 所有支持的规则，规则的逻辑是我们最重点关注的
  rules: {
    // ...
  },
  // 可以导出任意的环境，例如 jQuery 可以将 $ 设置为 global 等等
  // 非必须项
  environments: {
    // ...
  },
  // configs 字段导出的是一系列已经配好的 eslint 配置
  // 我们可以在 eslint 中使用 extend 来“继承”这些配置进行二次修改
  // 它的用途类似 babel 的 presets，不是必须项
  configs: {
    // ...
  },
  // 我们可以对指定后缀的文件进行预处理/后处理，不是必须项
  processors: {
    // ...
  }
};
```

### 开发一个 eslint 插件

在 eslint 插件的开发中，`environments` 和 `configs` 只是一些简单的配置，`processors` 的逻辑往往也不复杂，我们最终大部分逻辑复杂度集中在 `rules` 上。可以说 eslint 插件的开发，就是一系列 eslint 规则的开发。

在开发 eslint 插件之前，我们需要先阅读 eslint 的开发者指南。对于插件开发最有用的两篇文档是 [Working with Plugins](https://eslint.org/docs/developer-guide/working-with-plugins) 和 [Working with Rules](https://eslint.org/docs/developer-guide/working-with-rules)。当我们写测试用例时，还可以参考 eslint 的 [Node.js API 的 RuleTester](https://eslint.org/docs/developer-guide/nodejs-api#ruletester) 部分。

eslint 插件相关的第三方资源比较少，配合官方文档，这里推荐几个 eslint 插件的源码阅读，有助于我们快速上手：

1. [eslint 的内置规则](https://github.com/eslint/eslint/tree/master/lib/rules)：官方的规则，我们
2. [eslint-plugin-markdown](https://github.com/eslint/eslint-plugin-markdown) ：有关 processor（尤其是 preprocessor）的应用。
3. [eslint-plugin-vue](https://github.com/vuejs/eslint-plugin-vue)：如果你需要开发 Vue 相关的 eslint 规则，一定要参考 vue 官方插件的功能。尤其是它会大量使用到 [vue-eslint-parser](https://github.com/mysticatea/vue-eslint-parser) 注入到 `parserServices` 的帮助方法，我们可以用这个 parser 服务帮我们完成很多麻烦的语法解析（例如定义 template 的 Vistor 等等）。这部分细节并没有文档化，你也可以选择去直接阅读 vue-eslint-parser 的相关源码。
4. [eslint-plugin-html](https://github.com/BenoitZugmeyer/eslint-plugin-html)： 这个插件的实现非常的另类，它并不是一个标准的 eslint 插件，而是通过劫持 eslint 原生的`verify` 方法来实现。相关的 Github Issues 有 [#3422](https://github.com/eslint/eslint/issues/3422) 和 [#4153](https://github.com/eslint/eslint/issues/4153)。一般情况下不建议像这种方式处理，但如果你确实有无法实现的需求，这也可以作为一个参考。

### 核心源代码

在有了遍历 Babel 生成的 AST 树的代码经验之后，我们的规则这里做的事情非常类似：

1. 遍历生成 AST 树，这一步 eslint 已经帮我们完成了。
2. 定义一系列 Visitor，根据我们的规则拿到所有的硬编码信息。这里额外说明一点，我们这里的场景是针对 Vue 来实现的，因此解析器是 `vue-eslint-parser`，它会在 parserServices 注入一个 `defineTemplateBodyVisitor` 的方法。我们需要用它来定义 `template` 和 js 代码的 Vistor 并返回。如果你不是使用的 Vue，则按照 eslint 插件的规则，直接返回一系列的 Vistor 即可。
3. 过滤掉一些场景：例如 `import` 或 `console.log` 等。
4. 输出：这一步我们也无需关心，我们只需调用 eslint 的 `context.report` 方法，所有的格式化输出都交由 eslint 即可。

```javascript
function buildStringHardcodeChecker(str) {
  if (str.charAt(0) !== '/' && str.charAt(str.length - 1) !== '/') {
    return function(value) {
      return value === str;
    };
  }
  const tester = new RegExp(str.slice(1, -1));
  return function(value) {
    return tester.test(value);
  };
}

function buildNumberHardcodeChecker(v) {
  const checkNum = !!v;
  return () => checkNum;
}

function isRequireOrImport(node) {
  const parent = node.parent;
  if (!parent) return false;
  if (parent.type === 'ImportDeclaration') {
    return true;
  }
  if (
    parent.type === 'CallExpression' &&
    parent.callee.type === 'Identifier' &&
    parent.callee.name === 'require'
  ) {
    return true;
  }
  return false;
}

module.exports = {
  meta: {
    docs: {
      description: 'Check hardcodes in your js/vue files.',
      category: 'strongly-recommended',
      url: ''
    },
    schema: [
      {
        type: 'object',
        properties: {
          number: { type: 'boolean' },
          string: { type: 'string' }
        },
        additionalProperties: false
      }
    ]
  },
  create(context) {
    const defineTemplateBodyVisitor =
      context.parserServices.defineTemplateBodyVisitor;
    // 解析器不是 vue-eslint-parser，直接返回
    if (defineTemplateBodyVisitor == null) {
      context.report({
        loc: { line: 1, column: 0 },
        message:
          'Use the latest vue-eslint-parser. See also https://github.com/vuejs/eslint-plugin-vue#what-is-the-use-the-latest-vue-eslint-parser-error'
      });
      return {};
    }
    // 所有硬编码都从外部定义
    const options = context.options[0] || {};
    const isStringHardcode = buildStringHardcodeChecker(
      options.string || '/[^\\w\\s-_@:.#$%\\/()\\[\\]{}]/'
    );
    const isNumberHardcode = buildNumberHardcodeChecker(
      options.number || false
    );
    const isHardcode = value => {
      if (typeof value === 'number') {
        return isNumberHardcode(value);
      } else if (typeof value === 'string') {
        return isStringHardcode(value);
      }
      return false;
    };

    function visitor(node) {
      // require('package') 或 import 'package' 的情况，不做检测
      if (isRequireOrImport(node)) return;
      let value;
      if (node.type === 'TemplateElement') {
        value = node.value.raw;
      } else {
        value = node.value;
      }

      if (isHardcode(value)) {
        context.report({
          message: `Hardcode detected: "${value}"`,
          node,
          loc: node.loc
        });
      }
    }

    // vue-eslint-parser 的 AST 类型参照
    // HTML 节点: https://github.com/mysticatea/vue-eslint-parser/blob/master/src/html/tokenizer.ts
    // VNode: https://github.com/mysticatea/vue-eslint-parser/blob/master/src/ast/nodes.ts
    // JSX: https://github.com/babel/babel-eslint/blob/master/lib/babylon-to-espree/toToken.js
    return defineTemplateBodyVisitor(
      {
        VText: visitor,
        HTMLText: visitor,
        Literal: visitor,
        TemplateElement: visitor
      },
      {
        JSXText: visitor,
        Literal: visitor,
        TemplateElement: visitor
      }
    );
  }
};
```

PS：在本文的写作过程中，我发现一个 [eslint-plugin-i18n-text](https://github.com/dgraham/eslint-plugin-i18n-text/) 的插件，它完成的事情和我们类似，但是有些具体实现有些不同，在此列出供参考。
