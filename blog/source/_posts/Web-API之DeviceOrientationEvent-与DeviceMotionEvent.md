---
uuid: 1e96e2d0-51a8-11e7-be83-b5923ff0aabb
author: GodEngine
email: chenggong@blued.com
avatar: https://avatars0.githubusercontent.com/u/16115512?v=3
title: Web-API之DeviceOrientationEvent-与DeviceMotionEvent
date: 2017-06-15 16:53:35
tags:
---
w3c提供了关于提供主机设备物理方向的DOM事件标准。

凡实现了该标准的User agent必须提供名为deviceorientation和devicemotion 的Dom事件，且事件响应的类型为DeviceOrientationEvent和DeviceMotionEvent，且该两个事件必须在window Object上触发。


## DeviceOrientationEvent
DeviceOrientationEvent提供与运行当前页面的设备有关的方向信息

### DeviceOrientationEvent 主要属性
| 属性 | 说明 | 取值范围 | 默认值 | 趋势 |
| --- | --- | --- | --- | --- |
| alpha | 0~360 | 绕z轴转，屏幕长度方向距y轴的偏移角度 | 360/0 附近 | 顺时针至0，逆时针至360 |
| beta | -180~0~180 |绕x轴转，屏幕垂直方向距z轴的偏移角度 | 0附近 | 顺时针至180，逆时针至-180 |
| gamma | -90~0~90 |绕y轴转，屏幕宽度方向距x轴的偏移角度 | 0附近 | 顺时针至90，逆时针至-90|

### 关于alpha、beta、gamma三个角度的示意图
![alpha角](/img/godengine/alpha.png)
![beta角](/img/godengine/beta.png)
![gamma角](/img/godengine/gamma.png)

## DeviceMotionEvent
DeviceMotionEvent提供与当前设备位置和方向的速度变化先关的信息

### DeviceMotionEvent 主要属性
| 属性 | 说明 | 单位 |
| --- | --- | --- | --- | --- |
| acceleration | 包含x,y,z方向上的加速度 | m/s^2|
| accelerationIncludingGravity  | 含x,y,z方向上的加速度且带地球重力加速度g的影响 |m/s^2|
| rotationRate | 绕三个轴每秒的旋转速度 | deg/s |

## 引入方式

```javascript
// 判断当前window对象是否有DeviceMotionEvent属性，有则监听devicemotion事件:

if (window.DeviceMotionEvent) {
	window.addEventListener("devicemotion", motionHandler, false);
}


// 判断当前window对象是否有DeviceOrientationEvent属性，有则监听deviceorientation事件:

if (window.DeviceOrientationEvent) {
	window.addEventListener("deviceorientation", orientationHandler, false);
}

```

## 兼容性
移动端支持大部分浏览器（不支持的为IE phone和Opera Mobile）

## 相关库 [parallax.js](https://github.com/wagerfield/parallax)

[parallax.js](https://github.com/wagerfield/parallax)为支持监听pc端鼠标hover移动事件或监听移动端重力感应事件的库，并对相关的层做视差位移的效果。

### 示例demo
``` html
<!--class和data-depth是必须要的-->
<div id="container" class="container">
	<div id="scene" class="scene">
		<div class="layer" data-depth="1.00"><img src="./img/parallax/layer1.png"></div>
		<div class="layer" data-depth="0.80"><img src="./img/parallax/layer2.png"></div>
		<div class="layer" data-depth="0.60"><img src="./img/parallax/layer3.png"></div>
		<div class="layer" data-depth="0.40"><img src="./img/parallax/layer4.png"></div>
		<div class="layer" data-depth="0.20"><img src="./img/parallax/layer5.png"></div>
		<div class="layer" data-depth="0.00"><img src="./img/parallax/layer6.png"></div>
	</div>
</div>
```

```javascript
	import Parallax form './parallax.js'

	let scene = document.querySelector('#scene');
	let parallax = new Parallax(scene, {
	  relativeInput: true,
	  clipRelativeInput: false,
	  hoverOnly: true,
	  calibrateX: false,
	  calibrateY: true,
	  invertX: false,
	  invertY: true,
	  limitX: false,
	  limitY: 10,
	  scalarX: 2,
	  scalarY: 8,
	  frictionX: 0.2,
	  frictionY: 0.8,
	  originX: 0.0,
	  originY: 1.0,
	  precision: 1,
	  pointerEvents: false
	});

  /*
    xMotion = parentElement.width  * (scalarX / 100) * layerDepth
    yMotion = parentElement.height * (scalarY / 100) * layerDepth

    layer层在x,y方向上的位移是由父元素scene的尺寸以及scalarX，scalarY以及layerDepth属性计算而成的
  */

```

### parallax.js 主要的API
| Behaviour           | Values              | Default       | Description                                                                                                                                          |
| ------------------- | ------------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `relativeInput`     | `true` or `false`   | `false`       | Specifies whether or not to use the coordinate system of the scene. **Mouse input only.**                                                            |
| `clipRelativeInput` | `true` or `false`   | `false`       | Specifies whether or not to clip the mouse input to the scene bounds. No effect in combination with `hoverOnly`. **Mouse input only.**               |
| `hoverOnly`         | `true` or `false`   | `false`       | Apply the parallax effect only while the cursor is over the scene. Best together with `relativeInput` set to true. **Mouse input only.**             |
| `calibrate-x`       | `true` or `false`   | `false`       | 是否储存或者计算初始化的x坐标值，默认false                                   |
| `calibrate-y`       | `true` or `false`   | `true`        | 是否储存或者计算初始化的y坐标值，默认false                        |
| `invert-x`          | `true` or `false`   | `true`        | 设置层相当于设备方向的移动方向，true为反向，false为正向，默认为true                                                                    |
| `invert-y`          | `true` or `false`   | `true`        | 设置层相当于设备方向的移动方向，true为反向，false为正向，默认为true                                                                    |
| `limit-x`           | `number` or `false` | `false`       | number限制x轴的值或者false不限制                                        |
| `limit-y`           | `number` or `false` | `false`       | number限制y轴的值或者false不限制                                        |
| `scalar-x`          | `number`            | `10.0`        | x方向的位移计算中的一个与灵敏度有关的乘积因子，默认10                                             |
| `scalar-y`          | `number`            | `10.0`        | y方向的位移计算中的一个与灵敏度有关的乘积因子，默认10                                             |
| `friction-x`        | `number` `0 - 1`    | `0.1`         | 影响位移平滑度的一个参数，0~1之间，默认0.1                                                 |
| `friction-y`        | `number` `0 - 1`    | `0.1`         | 影响位移平滑度的一个参数，0~1之间，默认0.1                                                |
| `origin-x`          | `number`            | `0.5`         | x轴方向上的初始化鼠标位置，默认0.5位于中间，0 位于左侧，1位于右侧 |
| `origin-y`          | `number`            | `0.5`         | y轴方向上的初始化鼠标位置，默认0.5位于中间，0 位于左侧，1位于右侧 |
| `precision`         | `integer`           | `1`           | Decimals the element positions should be rounded to. Changing this value should not be necessary anytime soon.                                       |
| `pointerEvents`     | `true` or `false`   | `false`       | 设为false可以屏蔽一些点击事件，提高视差的性能，默认false   |


### 友情链接:

[w3c文档](http://w3c.github.io/deviceorientation/spec-source-orientation.html#devicemotion)

[html5rocks orientation](https://www.html5rocks.com/en/tutorials/device/orientation/)
