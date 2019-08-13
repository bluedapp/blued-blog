---
uuid: 6ddc19d0-bd82-11e9-82d4-fb6005848ff0
author: ryli
email: lov.2014@gmail.com
github: https://github.com/ryli
avatar: https://avatars2.githubusercontent.com/u/4973519?v=4
title: 开发 node 命令行程序
date: 2019-08-13 12:25:49
tags:
---

> 简单开发一个命令行程序

## 演示

介绍两个 node 写的程序，可以通过 `npm i -g saikou-cli` 安装来感受一下。

- [saikou-cli](https://www.npmjs.com/package/saikou-cli)
- [carbon-now-cli](https://www.npmjs.com/package/carbon-now-cli)

## 开始

首先进入开发目录，开始创建项目.

### 0x00 创建项目

``` shell
# 创建目录
mkdir baka-cli

cd baka-cli

# 初始化项目
npm init -y
```

完成后会多一个 `package.json` 文件。

### 0x01 配置命令名称

修改 `package.json` 文件，添加 `bin` 属性，属性值为执行的 js 入口文件。

```js
// 如果和包名相同且只有一个命令可以直接写字符串
{
  "bin": "./index.js",
}

// 对象形式可以自定义名称，也可以一次定义多个命令
{
  "bin": {
    "baka": "./baka.js",
    "senkun": "./senkun.js"
  },
}

```

### 0x02 其他配置

`package.json` 文件里的 `description` 和 `version` 属性也是程序需要使用的。

### 0x03 开始编写脚本

创建 `index.js` 文件。
在文件的第一行添加以下代码，告诉命令行，找到本命令后用 `node` 来执行本文件。

```shell
#!/usr/bin/env node

```

然后再加入一些简单的代码。

```js
// 第一个命令
main()

function main() {
  console.log('hello world!')
}

```

### 0x04 本地测试

之后就可以开始本地测试啦。

```shell
# 使用如下命令让系统识别我们的程序
npm link

# 测试
baka

# --> hello world!
```

看到输出 `hello world!`，最简单的命令就完成了。

![image](https://user-images.githubusercontent.com/4973519/62866286-02dce400-bd43-11e9-92d8-8423aa3e0a7e.png)

### 0x05 完善程序

正常使用的命令都有一些支持的参数如 `-h`, `-v` 等，简单来实现一下。

```js
// 获取信息
const { name, version, description } = require('./package.json')

const appName = name.replace('-cli', '')

// app 支持的参数列表
const app = {
  'h': showHelp,
  'v': showVersion,
}

main()

// 修改 main 方法
function main() {
  // 获取参数
  // 有效参数从第三个开始，前两个是 node 和 index.js
  const args = process.argv.slice(2)

  // 获取操作符,去掉 '-'
  const operation = args[0].substr(1)

  // 执行程序
  app[operation]()
}

function showHelp() {
  const help = `${name} ${description}

usage: ${appName} [options] [arguments]

options:
  -h:     show help
  -v:     show version
`
  console.log(help)
}

function showVersion() {
  console.log(version)
}

```

保存代码后可以测试一下。

![image](https://user-images.githubusercontent.com/4973519/62866128-ab3e7880-bd42-11e9-8317-31e4376cff2d.png)

### 0x06 添加功能

简单完成一个 todoList 功能。

创建 `todo.json` 文件来保存数据。

再次修改代码。

```js
const fs = require('fs')
const path = require('path')
const { name, version, description } = require('./package.json')
const todoList = require('./todo.json')

const appName = name.replace('-cli', '')

const app = {
  'c': create,
  'd': done,
  'r': remove,
  's': reset,
  'l': listAll,
  'h': showHelp,
  'v': showVersion,
}

main()

function main() {
  // 有效参数从第三个开始
  const args = process.argv.slice(2)

  // 默认输出列表
  if (!args.length) {
    listAll()
    return
  }

  // 获取操作符，支持简写和单词
  const operation = args[0].length === 2 ? args[0].substr(1) : args[0].substr(2, 1)

  // 参数错误则展示帮助
  if (!args[0].startsWith('-') || !operation || !Object.keys(app).includes(operation)) {
    showHelp()
    return
  }

  app[operation](args[1])
}

function create(job) {
  const index = todoList.todo.indexOf(job)
  const doneIndex = todoList.done.indexOf(job)
  if (index === -1 && doneIndex === -1) todoList.todo.push(job)

  save()
}

function done(job) {
  let index = todoList.todo.indexOf(job)
  if (index > -1) todoList.todo.splice(index, 1)

  index = todoList.done.indexOf(job)
  if (index === -1) todoList.done.push(job)

  save()
}

function remove(job) {
  let index = todoList.todo.indexOf(job)
  if (index > -1) todoList.todo.splice(index, 1)

  index = todoList.done.indexOf(job)
  if (index > -1) todoList.done.splice(index, 1)

  save()
}

function reset() {
  todoList.todo = []
  todoList.done = []

  save()
}

function listAll() {
  const todo = todoList.todo.length ? '☐ ' + todoList.todo.join('\n  ☐ ') : ''
  const done = todoList.done.length ? '✔ ' + todoList.done.join('\n  ✔ ') : ''
  const info = `Todo:
----------
  ${todo}

Done:
----------
  ${done}
  `

  console.log(info)
}

function showHelp() {
  const help = `${name} ${description}

usage: ${appName} [options] [arguments]

options:
  -c, --create:   create job
  -d, --done:     mark that the job is done
  -r, --remove:   remove job
  -s, --reset:    reset the list
  -l, --list:     list all job
  -h, --help:     show help
  -v, --version:  show version
`
  console.log(help)
}

function showVersion() {
  console.log(version)
}

function save() {
  const file = path.resolve(__dirname, './todo.json')
  const data = JSON.stringify(todoList)
  fs.writeFile(file, data, 'utf8', (err) => {
    if (err) {
      console.error(err)
    } else {
      listAll()
    }
  })
}

```

看一下效果。

![image](https://user-images.githubusercontent.com/4973519/62866428-4f282400-bd43-11e9-8c05-5047d788d058.png)

![image](https://user-images.githubusercontent.com/4973519/62867548-18074200-bd46-11e9-905c-311c920f69f5.png)

![image](https://user-images.githubusercontent.com/4973519/62866470-66671180-bd43-11e9-88e0-bb5fe15830a4.png)


### 0x07 补全其他内容

建议添加一个 `README.md` 文件，来简单说明一下本程序。

```shell
touch README.md

echo '# baka-cli' > README.md
```

### ox08 其他

可以通过 npm 包来轻松的开发程序。

- 命令配置：[commander](https://www.npmjs.com/package/commander)
- 交互功能：[inquirer](https://www.npmjs.com/package/inquirer)
- 显示加载状态：[ora](https://www.npmjs.com/package/ora) / [node-progress](https://www.npmjs.com/package/node-progress)
- ...

完。
