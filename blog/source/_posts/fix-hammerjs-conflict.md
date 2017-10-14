---
uuid: 79120770-ae79-11e7-9b1b-eda24dc05fdc
author: l19861225q
email: liuqian@blued.com
github: https://github.com/l19861225q
avatar: https://avatars1.githubusercontent.com/u/4251365?v=4
title: 解决 HammerJS 拖拽、缩放和旋转的手势问题
date: 2017-10-11 19:43:55
tags:
  - hammerjs
  - react
  - hoc
---

近期基于 HammerJS 开发了一个贴纸的 React 高阶组件，但是拖拽、缩放和旋转事件一起监听时会有手势问题。。。

开启了 hammerjs 的拖拽、缩放和旋转后，当两个手指触碰屏幕的瞬间，元素会立即旋转180度，但此时并未开始执行旋转，这是由于两个手指的位移差导致触发了旋转事件。另外拖拽和缩放、旋转事件也有冲突，执行缩放或旋转时会触发拖拽从而导致发生位移，元素会出现位置跳跃、抖动的问题。

所以决定自行实现这些事件的处理，每次旋转开始时，记录当前旋转的角度和缩放比率（因为设置了缩放和旋转可以同时触发）；旋转过程中实时计算旋转的差值来实现元素的旋转。旋转结束后再保存当前旋转的角度和缩放比率，一遍下次旋转开始时可以获取正确的值。通过 event.pointers.length 来区分拖拽还是缩放、旋转，从而避免手势的冲突。

```javascript
...
componentDidMount () {
  this.DOMNode = findDOMNode(this)

  const mc = new Hammer.Manager(element, {
    tap: { enable: false } // 禁用 tap 事件
  })

  // 拖拽、缩放和旋转，pointers 指定触发该事件的手指数，后面用来区分事件
  const pan = new Hammer.Pan({ pointers: 1 })
  const pinch = new Hammer.Pinch({ pointers: 2 })
  const rotate = new Hammer.Rotate({ pointers: 2 })

  // 缩放和旋转可以同时进行
  pinch.recognizeWith(rotate)

  mc.add([pan, pinch, rotate])

  // 定义一些公共的变量，缓存旋转角度、缩放比率、位移
  let startRotation = 0
  let rotation = 0
  let lastRotation
  let scale = 1
  let lastScale = this.DOMNode.clientWidth
  let lastPosX = 0
  let lastPosY = 0
  let posX = 0
  let posY = 0

  // 监听如下的事件
  mc.on(`
    pinchstart pinch pinchend
    rotatestart rotate rotateend
    pan panend
    tap, multitap
  `, (e) => {
    e.preventDefault()

    switch (e.type) {
      case 'pinchstart':
        lastScale = scale
        break

      case 'pinch':
        scale = Math.max(0, Math.min(lastScale * e.scale, 10))
        break

      case 'pinchend':
        lastScale = scale
        break

      case 'rotatestart':
        lastScale = scale
        lastRotation = rotation
        startRotation = e.rotation
        break

      case 'rotate':
        const diff = startRotation - e.rotation
        rotation = lastRotation - diff
        break

      case 'rotateend':
        lastScale = scale
        lastRotation = rotation
        break

      case 'pan':
        posX = e.deltaX + lastPosX
        posY = e.deltaY + lastPosY
        break

      case 'panend':
        lastPosX = posX
        lastPosY = posY
        break
    }

    if (e.pointers.length < 2) {
      // 当触屏的手指数量小于2个时，执行拖拽
      // translate3d(`${posX}px, ${posY}px, 0`)
    } else {
      // 否则执行旋转或缩放
      // rotate(`${rotation}deg`)
      // scale3d(`${scale}, ${scale}, 1`)
    }

    // 更新 element.style.transform 实现拖拽、缩放或旋转，注意浏览器兼容性
    // element.style['transform/-webkit-transform/-ms-transform/...'] = translate3d rotate scale3d
  })
}
...
```
