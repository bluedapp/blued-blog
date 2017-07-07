---
uuid: f643d3a0-5cac-11e7-8afc-4d371fd0dd66
author: l19861225q
email: liuqian@blued.com
github: https://github.com/l19861225q
avatar: https://avatars2.githubusercontent.com/u/4251365?v=3
title: 交互式的 NPM Scripts
date: 2017-06-29 17:25:57
tags:
- node
- npm
---

尽管 Webpack 2 默认支持从终端传参数到配置文件中来实现定制：
> --env.x=xxx

但每次开发都需要在命令行输入冗长且难记的各种参数:
> npm run dev -- --env.p=xxx --env.s --env.r ...

希望可以屏蔽这些细节，提供一个交互式的终端用户界面。
先看一下最终效果：

![](/img/liuqian/interactive-npm-scripts/interactive.gif)

## 支持会话的终端界面 - Inquirer.js
> [Inquirer.js](https://github.com/SBoudrias/Inquirer.js) 可以预设一组问题来收集用户答案，支持过滤、验证等特性。

### 定义问题
这里我需要两个提问：

- 将要开发哪个项目（必填）
- 可选配置（是否生成雪碧图、是否使用 CSS Module 等）

```javascript
const questions = [
  {
    name: 'project',
    message: 'Please input the project name:',
    // 验证：这个答案必填才继续后面的问题
    validate: (str) => Boolean(str.length)
  },
  {
    name: 'conf',
    type: 'checkbox',
    message: 'Please make your choice:',
    choices: [
      'Sprite (是否生成雪碧图)',
      'Retina (是否支持 Retina 雪碧图)',
      'CSS Module (是否使用 CSS Module)',
      'Use React (是否使用 React, 默认 Preact)'
    ]
  }
]
```

### 收集答案
`--color` 来让终端始终高亮显示强调的内容。Inquirer 支持 Promise 特性，代码看起来也十分的简介、优雅。
```javascript
const args = ['--color']
inquirer.prompt(questions).then(function ({ project, conf }) {
  // 拼接成 Webpack 2 规定的参数格式
  args.push(`--env.p=${project}`)

  const [needSprite, needRetina, needCSSModule, needReact] = conf

  needSprite && args.push('--env.s')
  needRetina && args.push('--env.r')
  needCSSModule && args.push('--env.m')
  needReact && args.push('--env.R')
})
```

## 终端 Loading 动画 - ora
> [ora](https://github.com/sindresorhus/ora) 优雅的终端 Loading 动画

因为 webpack 的配置文件使用了 ES6，在启动前需要先经过 Babel 转码，需要一定的时间，这时如果能给终端 Loading 的反馈，用户体验将是美好的。

```javascript
console.log('\n') // 和上面 Inquirer 的问题保留一个空行更美观

const oraInstance = ora({
  color: 'cyan',
  text: 'Please waiting for the webpack start'
}).start()
```

这里返回了 ora 的实例，后面需要显示的调用 [ora 的 API](https://github.com/sindresorhus/ora#api) `stop` & `clear` 来停止并移除 ora .

## 用于 Node.js 的 shell 命令 - shellJS
> [ShellJS](https://github.com/shelljs/shelljs) 是 Node.js API 之上的 shell 命令的便携式（Windows / Linux / OS X）实现。

我需要在 js 文件里执行原先定义在 `package.json` 中的 `scripts` 命令。

```javascript
const child = shell.exec(`
  NODE_ENV=development webpack --hide-modules ${args}
`, { async: true }) // 异步执行速度更快哦
```

这里返回了子线程的实例 child，后面会用到他的 `stdout`、`stderr` 实现监听终端输出的功能。

```javascript
// npm run dev
// Stop and clear the ora when webpack started
child.stdout.on('data', function (data) {
  if (data.toLowerCase().includes('webpack')) {
    oraInstance.stop() // 停止 ora 动画
    oraInstance.clear() // 移除 ora
  }
})
```

因为 ora 的实现原理是不断刷新终端输出来达到动画的效果，如果不移除会造成 webpack stdout 的闪现，体验很差。

在 `NODE_ENV=production` 模式下，Webpack 只会产生标准错误输出 stderr，所以监听有些许改变：

```javascript
// npm run build
child.stderr.on('data', function (data) {
  if (data.toLowerCase().includes('compiling')) {
    oraInstance.stop()
    oraInstance.clear()
  }
})
```

OK~ 愉快地 coding 吧 ^_^
