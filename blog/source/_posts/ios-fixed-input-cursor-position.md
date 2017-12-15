---
uuid: f0d0ff10-e163-11e7-a8ab-cd089080c9c2
author: l19861225q
email: liuqian@blued.com
github: https://github.com/l19861225q
avatar: https://avatars1.githubusercontent.com/u/4251365?v=4
title: iOS 上，fixed 元素内的输入元素，获取焦点时的光标错位问题
date: 2017-12-15 14:48:20
categories: 前端技术
tags:
 - html
 - js
 - iOS
---

RT，如果一个输入元素（input, textarea ...）的父容器设置了 `position: fixed`，当这个元素获取焦点时，会触发底部键盘的弹起。这时在输入框内打字的时候，会发现其光标错位了，一般会跑到下方。

<center>
  <img src="/img/liuqian/ios-fixed-input-cursor-position/demo.gif" width="200" alt="demo">
</center>

当你专注于一个输入时，浏览器会自动向下滚动，以便将焦点输入突出显示给用户，这就造成了页面内高度浮动，导致光标位移。
> 遗憾的是，截至目前，iOS 11.x 上也有这个问题。

### 曾经尝试过的方案
#### 当元素获取焦点时，改变父容器的定位方式：`fixed` > `absolute`

```js
for (let evt of ['focus', 'blur']) {
  const isFocus = evt === 'focus'
  const fn = isFocus ? 'add' : 'remove'

  inputDOMNode.addEventListener(evt, () => {
    parentDOMNode.classList[fn]('input-focus')
    htmlDOMNode.classList[fn]('no-scroll')
    bodyDOMNode.classList[fn]('no-scroll')

    isFocus && setScrollTop(0)
  })
}
```

```css
.input-focus {
  position: absolute;
  ...
}

.no-scroll {
  height: 100%;
  overflow: hidden;
}
```

监听了输入元素 `focus` 和 `blur` 事件，为父元素添加或移除某些样式。
> 当 `position: absolute` 时，输入框的定位方式需要手动设置（这里采取了顶部对齐）；`.no-scroll` 是为了禁止 <body /> 的滑动，保证输入框可见。

但是这个方案在部分 Android 设备上，当键盘收起时并不会触发输入元素的 `blur` 事件，往往还需要用户主动点击页面的其他区域，算是一点小遗憾吧。

### 终极解决方案（推荐）
#### 直接给 `html`, `body` 元素设置样式

```css
html,
body {
  -webkit-overflow-scroll: touch !important;
  overflow: auto !important;
  height: 100% !important;
}
```

[`-webkit-overflow-scrolling`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/-webkit-overflow-scrolling) 控制元素在移动设备上是否使用滚动回弹效果。

```css
-webkit-overflow-scrolling: touch; /* 当手指从触摸屏上移开，会保持一段时间的滚动 */
overflow: auto; /* 由浏览器定夺，如果内容被修剪，就会显示滚动条 */
```

当输入元素获取焦点时，键盘弹起，输入元素被顶到了键盘的上方，此时用户的手指会从触摸屏上移开，输入元素会保持一段时间的滚动，从而光标的位置可以被正确计算。

> `!important` 在这里是为了防止这些属性会因为浏览器优先级过高而发生变化。
> 有点小遗憾的是，`!important` 侵入性有些高。
