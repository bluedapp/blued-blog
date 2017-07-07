---
uuid: ac229330-50b1-11e7-9ca3-0ba3e7ebedc7
author: l19861225q
email: liuqian@blued.com
title: 前端项目架构时遇到的问题
banner: /img/liuqian/framework-qa/banner.jpg
date: 2017-06-14 11:29:28
tags:
  - JavaScript
  - Framework
  - Webpack
  - React
  - Antd
  - Test
github: https://github.com/l19861225q
avatar: https://avatars0.githubusercontent.com/u/4251365?v=3
---

**本文涉及到的相关技术点不会讲解基础用法（请参考官方文档），只会针对开发中遇到的痛点展开讨论。**

## 技术概览

主要使用了以下技术栈，从环境、编译、开发，再到测试，较为全面的涵盖了目前前端开发中常用的一套流程，可作为脚手架使用。

- {% link Yarn https://yarnpkg.com %} - <small>Node 包管理器</small>
- {% link ES6 http://es6.ruanyifeng.com %} - <small>ECMAScript6，下一代 JS 语法</small>
- {% link Babel https://babeljs.io %} - <small>ES6 编译器</small>
- {% link Webpack https://webpack.js.org %} - <small>前端开发构建工具</small>
- Lint 代码检查
  - {% link ESLint http://eslint.org %} - <small>JavaScript 代码检查</small>
  - {% link SASS Lint https://github.com/sasstools/sass-lint %} - <small>SASS 代码检查</small>
  - {% link HTML Lint http://htmlhint.com %} - <small>HTML 代码检查</small>
- React 相关
  - {% link React https://facebook.github.io/react %} - <small>React 库</small>
  - {% link React DOM https://github.com/facebook/react/tree/master/packages/react-dom %} - <small>React 插件，封装了和 DOM 相关的操作</small>
  - {% link React Router DOM https://reacttraining.com/react-router %} - <small>前端路由</small>
- Style 相关
  - {% link SASS http://sass-lang.com %} - <small>强大的 CSS 预处理器</small>
    - {% link Bemify https://github.com/franzheidl/bemify %} - <small>帮助写出 {% link BEM http://getbem.com %} 风格的 SASS 混入</small>
    - {% link Bourbon http://bourbon.io %} - <small>轻量的 SASS 工具集</small>
    - {% link Post CSS http://postcss.org %} - <small>强大的 CSS 后处理器</small>
- Unit Testing {% link 单元测试 http://baike.baidu.com/item/%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95 %}
  - {% link Karma https://karma-runner.github.io/1.0/index.html %} - <small>单元测试执行器</small>
  - {% link Jasmine https://jasmine.github.io %} - <small>行为驱动测试框架</small>
  - {% link Enzyme http://airbnb.io/enzyme %} - <small>React 单元测试套件</small>
  - {% link PhantomJS http://phantomjs.org %} - <small>基于 WebKit 的服务器端 JavaScript API，无需浏览器的 Web 测试</small>
- E2E Testing {% link 端到端测试 https://zhidao.baidu.com/question/29346350.html %}
  - <a href="http://nightwatchjs.org/" target="_blank">Nightwatch</a> - <small>基于 Node 的验收测试框架，使用 <a href="http://www.seleniumhq.org/" target="_blank">Selenium WebDriver API</a> 以将 Web 应用测试自动化</small>

## 一次配置，多处使用

项目结构如下：

```javascript
/hd
 - /components // 基于组件化思想，把活动页面通用的部分提出来作为公共组件，开发时可按需引入
 - /constants // 定义全局使用的常量
 - /coverage // 单元测试覆盖率报告
 - /libs // 常用库函数（获取用户详情、handlebars helper 等）
 - /public // Webpack 打包编译后文件存放目录
 - /routes // Koa 路由
 - /SCSS // 通用 SCSS 样式
 - /src // 开发目录
 - /test // 测试（unit、e2e）
 - /utils // 工具函数
 - /views // Koa 模板
 - /webpack // 拆分的 Webpack 任务（若全部写在 webpack.config.js 中，会导致文件臃肿、不便阅读）
 - .babelrc // Babel 配置
 - .eslintrc // ESLint 配置
 - .gitignore // Git 忽略定义
 - CHANGELOG.md // 更新日志
 - karma.conf.js // Karma 配置
 - nightwatch.conf.js // Nightwatch 配置
 - package.json // NPM Package JSON
 - postcss.config.js // Post CSS 配置
 - README.md // 说明文档
 - webpack.config.babel.js // Webpack 配置
 - yarn.lock // Yarn 锁（锁定 NPM 依赖包的版本）
```

为了设计出一个通用的开发环境，满足不同的活动业务需求（可能来自运营、市场、公益等部门），需要做到彼此的项目隔离、互不影响。因此在 `/src` 目录下的第一子级是各个活动项目的源代码（命名采取：年份-{Project Name}，便于日后检索也可防止冲突）。

```javascript
/src
 - /2017-lovers // 2017 情人节活动
 - /2017-trees // 2017 植树节活动
 ...
```

因为使用了 webpack，开发时在终端输入指定的参数，定位到具体的某个活动项目，webpack 会根据当前的项目目录执行编译任务。为了适配这种通用性，会要求目录结构、文件名等要满足特定的规则：

```javascript
// 举个栗子
/2017-nightlive
 - /components // 该项目用到的 React 组件
 - /img // 图片文件
 - /SCSS // 样式文件
 - /sprite // 将要被生成雪碧图的图标文件
 - index.hbs // 项目模板
 - index.js // Webpack 编译时的 entry 文件
```

## Babel (6.x)

> 可以将 ES6 语法转换成 ES5 语法，让我们在使用 ES6 新特性编写代码的同时，不需要考虑各大浏览器具体的兼容性情况。

这里选择了 Babel，主要有以下几个原因：
- Babel 对 ES6 的支持程度比其它同类更高或相当
- Babel 拥有完善的文档和较好体验的在线编译环境
- Babel 使用广泛，用户基础好

关于第一点原因的主要数据支持可以在 {% link Bebel 官网 https://babeljs.io %}，我们可以看到不同版本 Babel 对 ES6 跟进和支持的情况，另外，关于在线编译平台，可以访问官网进行体验，这对于研究 Babel 编译结果十分方便。

关于 Babel 的接入和使用方法，社区上的资料很多，这里配合构建工具 webpack，只需要安装插件 <a href="https://github.com/babel/babel-loader" target="_blank">babel-loader</a> 并在 <a href="https://webpack.js.org/configuration/module" target="_blank">webpack.module.rules</a> 中进行相关配置即可使用。

### {% link babel-polyfill https://babeljs.io/docs/usage/polyfill %}

> Babel 默认只转换新的 JavaScript 语法，而不转换新的 API。

Babel 可以编译 `let`、`const` 等特性，但是诸如 Iterator、Generator、Reflect、Promise 等全局对象，或者数组实例的 find 这些新的方法并不会得到编译。如果想让这个方法运行，必须使用 babel-polyfill，**同时要保证这个 polyfill 在你的所有其他脚本之前就要加载执行。因为编译产出为 ES5 代码，所以又要处在 ES5 垫片 {% link es5-shim https://github.com/es-shims/es5-shim %}、ES6 垫片 {% link es6-shim https://github.com/paulmillr/es6-shim %} 之后。**（垫片就是在低级环境中使用高级语法时，手动实现高级功能，模拟高级环境）

实际情况中，在公共组件 `<App />` 中开头引入：

```javascript
// Babel Polyfill
// Error: only one instance of babel-polyfill is allowed
// https://github.com/stylelint/stylelint/issues/1316
if (!global._babelPolyfill) { // 为了解决重复引入的问题
  require('babel-polyfill')
}
```

项目开发时将这个组件作为最外层容器使用即可。

### Babel 配置 - Presets（转码规则）

#### {% link babel-preset-env http://babeljs.io/docs/plugins/preset-env %}

随着浏览器和 Node.js 的版本迭代，对新语法的支持也越来越好。但非常尴尬的是，我们总是使用 Babel 把所有代码一股脑转换成 ES5。这意味着我们抛弃了性能优秀的 `let`、`const` 关键字，放弃了简短的代码，而选择了又长又丑像坨屎的经过变换后的代码。

即使仅仅将代码跑在对 ES5 支持度在 99% 的 Node 6 上，一旦使用了 `import` 关键字，你就得用 Babel 对代码进行转换，一般还是全部转换为 ES5，辣鸡 Node.js 竟然还不支持 `import` 和 `export。`

> 那么有没有什么工具能智能识别当前运行环境，并且进行适当的转换，以及填充适当的 polyfill 呢？

还真有，而且是 Babel 官方提供的，一个名为 babel-preset-env 的插件。它不需要你自行添加任何 preset，比如我们最常用的 es2015，它能根据设置智能转换代码。

#### {% link babel-preset-react http://babeljs.io/docs/plugins/preset-react/ %}

React 转码规则，支持编译 .jsx 文件。

#### {% link babel-preset-stage-0 https://babeljs.io/docs/plugins/preset-stage-0 %}

ES7 不同阶段语法提案的转码规则，涵盖了 {% link stage-1 https://babeljs.io/docs/plugins/preset-stage-1 %}、{% link stage-2 https://babeljs.io/docs/plugins/preset-stage-2 %}、{% link stage-3 https://babeljs.io/docs/plugins/preset-stage-3 %}。

### Babel 配置 - Plugins（插件）

#### {% link babel-plugin-transform-react-remove-prop-types https://github.com/oliviertassinari/babel-plugin-transform-react-remove-prop-types %}

在 webpack production 编译模式下移除 React propTypes 定义来减小编译后的 js 文件体积。

#### {% link babel-plugin-import https://github.com/ant-design/babel-plugin-import %}

实现 {% link antd-mobile https://mobile.ant.design %} 的按需加载，另外此插件配合 style 属性可以做到模块样式的按需自动加载。

### Babel 配置文件

Babel 的配置文件是 `.babelrc`，存放在项目的根目录下，用来设置上述转码规则和插件。

```javascript
{
  "presets": [
    "env",
    "react",
    "stage-0"
  ],
  "env": {
    "production": {
      "plugins": [
        ["transform-react-remove-prop-types", {
          "mode": "wrap",
          "ignoreFilenames": ["node_modules"]
        }]
      ]
    }
  },
  "plugins": [
    ["import", {
      "style": "css",
      "libraryName": "antd-mobile"
    }]
  ]
}
```

### Babel ESLint

```javascript
// .eslintrc
parser: babel-eslint // ESLint 解析器
```

ESLint 允许自定义解析器。但是 ESLint 不支持 Babel 支持的一些语法节点。使用 {% link babel-eslint https://github.com/babel/babel-eslint %} 时，ESLint 将被修改，代码将转换为 ESLint 可以理解的代码。所有位置信息（如行号，列）也保留，以便轻松跟踪错误。

## Webpack (2.x)

### 配置 & ES6

用 `webpack.config.babel.js` 命名 Webpack 的配置文件，会先经过 Babel 的转码，So 可以使用 ES6 语法咯。

### 按需配置

webpack 2.x 默认支持从命令行传参到配置文件实现按需配置。

```shell
// Command line
webpack -- --env.x=xxx
```

### 忽略解析

```javascript
noParse: vendor.map(v => new RegExp(`${v}$`))
```

webpack.module.noParse 可防止 webpack 解析那些任何与给定正则表达式相匹配的文件，忽略大型的 library 可以提高构建性能。

### babel-loader 之 cacheDirectory

```javascript
use: 'babel-loader?cacheDirectory'
```
`cacheDirectory` 可以缓存处理过的模块，对于没有修改过的文件不会再重新编译，有着2倍以上的速度提升，这对于 rebuild 有着非常大的性能提升。

### 动态匹配路径

```javascript
use: (() => {
  const loaders = [
    {
      loader: 'file-loader',
      options: {
        regExp: /components\/(.*)\//,
        name: `[1]/[name]-[hash:8].[ext]`
      }
    }
  ]

  if (isProd) {
    loaders.push(imageWebpackLoaderRule)
  }

  return loaders
})()
```
regExp 中匹配到的分组内容将会对应到下面的 [1] 处。

### Webpack Hash 的困扰

由于 Webpack 的{%link 一个问题 https://github.com/webpack/webpack/issues/1315 %}，生成哈希值的方法并不是确定的。为了保证哈希值是根据文件内容生成的，需要使用 {% link webpack-md5-hash https://github.com/erm0l0v/webpack-md5-hash %} 插件

当更改了代码的任何一部分，即使剩下的文件内容没有被修改，入口也会被更新以放入新的清单。这样反过来也就导致新的哈希值，影响了长期缓存。为了修复这个问题，使用插件 {% link chunk-manifest-webpack-plugin https://github.com/soundcloud/chunk-manifest-webpack-plugin %} 来把清单导出到单独的 JSON 文件中。

> 延伸阅读：{% link 用 webpack 实现持久化缓存 https://sebastianblade.com/using-webpack-to-achieve-long-term-cache %}

### Webpack Clean 父级目录的权限问题

在任务开始时，经常会使用插件 {% link clean-webpack-plugin https://github.com/johnagan/clean-webpack-plugin %} 来清除上次打包生成的文件，以保证目录的干净。但是，如果你想要清除父级目录，会遇到一个错误提示：

> xxx/xxx must be inside the project root...

解决办法是设置 `root` 参数，指向 webpack 编译时的当前目录。

```javascript
new CleanPlugin(somePath, {
  root: process.cwd()
})
```

### 提升 Webpack 压缩 JS 文件的速度

Webpack 提供的 UglifyJS 插件由于采用单线程压缩，速度很慢。使用 {% link webpack-parallel-uglify-plugin https://github.com/gdborton/webpack-parallel-uglify-plugin %} 可以并行运行 UglifyJS 插件，可有效减少构建时间。

## 前端测试

有关单元测试、端到端测试的内容，可以参考@闲总的文章{% post_link 单元测试 《React--单元测试》 %}。
