---
uuid: eb8979e0-dfcb-11e7-bd36-13e1ac221d6c
author: l19861225q
email: liuqian@blued.com
github: https://github.com/l19861225q
avatar: https://avatars1.githubusercontent.com/u/4251365?v=4
title: DOM 单触点旋转的探索
date: 2017-12-13 14:07:36
categories: 前端技术
tags:
 - js
---

由于 `hammerjs` 的 rotate 行为不支持单触点，所以只能另辟蹊径。搜索后发现了一个比较好用的 Library

> [Propeller](https://github.com/PixelsCommander/Propeller) - JavaScript library to rotate elements with mouse or touch gestures.

读了下源码，其实现原理是监听一些事件，然后通过各种数学函数（sin, tan...），根据位移差值计算旋转角度，最后再更新 DOM 的 CSS（transform: rotate）达到旋转的效果。根据浏览器的兼容性，渐进使用了 [RAF](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)
提升动画的帧数使其平滑展现。

### 旋转方向的问题
实际需求是要限制 DOM 的旋转方向，只可以顺时针旋转。官方尚未支持这个配置，但有提供部分事件钩子：
`onRotate`, `onDragStart` ...

期初，是想在 `onDragStart` 中根据 this.angle 判断是顺/逆时针旋转。发现这个事件是拖拽开始时才触发一次，此时并不能获取到 angle：即 this.angle = 0。又翻看了源码，只有在 `onRotate`时能获取到 angle（通过 RAF，每隔一段时间监听 evt.touches 的 X, Y 位移差值，通过数学公式计算旋转角度）。

源码中计算角度差并更新 DOM 的代码：
```javascript
if (
  Math.abs(this.lastAppliedAngle - this._angle) >= this.minimalAngleChange &&  this.transiting === false
) {
 this.updateCSS();
 ...
 this.lastAppliedAngle = this._angle; //  将本次的 angle 赋值给 lastAppliedAngle
```
看来是取了绝对值，没有办法通过正负数来判断旋转的方向了。

#### 两个关键的变量

- `this.angle` - 本次旋转的角度值
- `this.lastAppliedAngle` - 相邻最近旋转的一次的角度值（= this.angle），用来计算角度差值，默认是

```javascript
this.angle = options.angle || defaults.angle
this.lastAppliedAngle = this.virtualAngle = this._angle = options.angle || defaults.angle;
```

#### 逆时针旋转和复位
```javascript
onRotate: function () {
  const angleDiff = this.angle - this.lastAppliedAngle;

  if (angleDiff < 0 && angleDiff > -350) {
    // 逆时针旋转，停止旋转，让 DOM 复位
    this.stop();
    ...
  }
}
```
如果逆时针旋转，this.angle 取值 < 0，而 this.lastAppliedAngle > this.angle，所以 angleDiff < 0， 通过 angleDiff < 0 可以判断是否进行了逆时针旋转。

但是现在有个问题，当旋转了一圈再次回到0度时，DOM 不能再次旋转了

> 这是因为，旋转一圈再次划过0度触发 onRotate 时，this.lastAppliedAngle = 350（测试几次发现，这个范围和触发 onRotate 的频率有关，大概在 350 ~ 359 度之间），而 this.angle 大概在 0 ~ 10 度之间。所以 angleDiff = 0 - (350 ~ 359) = -350 ~ -359 度之间，进入了逆时针判断的逻辑。这时还需加上临界点，所以有了 `angleDiff > -350` 的边界。

#### 重置 propeller
> 用来重置 DOM 为初始角度，这里直接使用了 this，所以调用时注意 Scope

```javascript
const propellerReset = function () {
  this._angle = 0;
  this.updateCSS();
};
```

#### 完整的 onRotate
```javascript
...
onRotate: function () {
  const angleDiff =  this.angle - this.lastAppliedAngle;

  // 逆时针 > 复位
  if (angleDiff < 0 && angleDiff > -355) {
    this.rotateOrder = ANTI_CLOCKWISE;
    this.stop();

    setTimeout(() => {
      propellerReset.call(this);
    }, 200);
  }
}
...
```
rotateOrder 后续会用到。重置时注意设置一个超时（约 200 ms），保证 this.stop() 执行完再重置，否则动画会有跳帧抖动的现象。this.stop 会设置私有变量 this.active = false，当再次触发 this.update() 时会根据这个参数判断是否再次更新 DOM CSS。所以重置 propeller 的函数要手动调用一次 this.updateCSS()。

#### 旋转结束时的回调
```javascript
onDragStart: () => {
  this.rotateOrder = CLOCKWISE;
},
onDragStop: function () {
  setTimeout(function () {
    if (this.angle && this.rotateOrder === CLOCKWISE) {
      alert('stop');
      propellerReset.call(this);
    }
  }.bind(this), 200)
}
```
这个判断放在了 onDragStop 中，这里的超时是为了等待 onRotate 复位的超时，从而获得 DOM 最后真正的 angle。`rotateOrder === CLOCKWISE` 保证了只有顺时针才继续后面的逻辑。当旋转开始时，会触发一次 onDragStart 事件，此时设置 this.rotateOrder = CLOCKWISE（相当于重置 this.rotateOrder，默认视为是顺时针），当出现了逆时针旋转时，this.rotateOrder = ANTI_CLOCKWISE，此时触发的 onDragStop 中就能识别出旋转的方向了。

> Propeller 还支持更多的参数：
- `step` 每次转动一个指定的角度
- `inertia` 惯性旋转
- `stepTransitionEasing` 旋转动画的过度函数

感兴趣的可以自行探索哈
