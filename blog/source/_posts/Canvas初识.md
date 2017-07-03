---
uuid: 09332160-5d42-11e7-b81b-09d14a590ba2
author: mxx
email: mengxxself
github: https://github.com/mengxxSELF
avatar: https://avatars0.githubusercontent.com/u/20737114?v=3
title: Canvas初识
date: 2017-07-03 11:13:04
tags: canvas
banner: /img/mxx/20170630_canvas/banner_canvas.png
---

### 什么是canvas


> Canvas是一个可以使用脚本(通常为JavaScript)在其中绘制图形的 HTML 元素 这个HTML元素是为了客户端矢量图形而设计的。它自己没有行为，但却把一个绘图 API 展现给客户端 JavaScript 以使脚本能够把想绘制的东西都绘制到一块画布上

canvas 是HTML5新增的元素 可以将其看作为一块绘画的画板 而JavaScript就是画笔

浏览器支持情况
![canvas](/img/mxx/20170630_canvas/canvas_use.png)

### 何时使用canvas
何时使用此元素没有具体的场景限制 一般在需要展示多个实例的时候 可以考虑使用canvas

### 使用canvas

html页面写入canvas标签

```html
<canvas id='canvas'> 您的浏览器不支持canvas </canvas>
```
图像绘制是由JavaScript完成的
```javascript
let canvas = document.getElementById('canvas')
let ctx = canvas.getContext('2d')
```

## 基本API
这里只简单介绍几个基本的API 完整版请参见 [Canvas的MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Basic_usage)

* 1 ctx.beginPath() 开始绘制
* 2 ctx.closePath() 结束绘制

** 绘制一条线 **

```javascript
ctx.beginPath()
ctx.moveTo(50,50)
ctx.lineTo(200, 50)
ctx.stroke()
ctx.closePath()
```
* 3 ctx.rect() 绘制四边形
* 4 ctx.strokeStyle 定义绘制颜色
* 5 ctx.stroke()

stroke() 方法会实际地绘制出通过 moveTo() 和 lineTo() 方法定义的路径。默认颜色是黑色

** 绘制一个白色的四边形 **

```javascript
ctx.rect(10, 10, 100, 100)
ctx.strokeStyle = ‘#fff’
ctx.stroke()

```

* 6 ctx.drawImage() 绘制图片

其中的传递参数个数最少为3个 最多为9个

```javascript
 ctx.drawImage(image, dx, dy)
 ctx.drawImage(image, dx, dy, dWidth, dHeight)
 ctx.drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)
```
参数解析

| 参数 | 说明 |
| --- | --- | --- | --- | --- |
| image | 要进行绘制的图片
| dx | 目标左上角在画板X轴方向的位置
| dy | 目标左上角在画板Y轴方向的位置
|    | dx 与 dy 决定了图片在画板中出现的位置
| dWidth | 在目标画布上绘制图像的宽度
| dHeight | 在目标画布上绘制图像的高度
|    | dWidth 与 dHeight 决定了图片在画板中出现的大小
| sx | 需要绘制到目标上下文中的，源图像的矩形选择框的左上角 X 坐标
| sy | 需要绘制到目标上下文中的，源图像的矩形选择框的左上角 Y 坐标
|    | sx 与 sy 决定了资源图片出现在画板上的部分 *** 起始位置 ***
| sWidth | 需要绘制到目标上下文中的，源图像的矩形选择框的宽度。如果不说明，整个矩形从坐标的sx和sy开始，到图像的右下角结束
| sHeight | 需要绘制到目标上下文中的，源图像的矩形选择框的高度
|    | sWidth 与 sHeight 决定了资源图片出现在画板上的部分 *** 大小 ***

图片绘制实例

```javascript
let canvas = document.getElementById('canvas')
let ctx = canvas.getContext('2d')
let img = new Image()
img.src = './can.jpg'
img.onload = () => {
  ctx.drawImage(img, 50, 50, 200, 267)
  --  other code --
  ctx.drawImage(img, 500, 300, 200, 240, 300, 110, 200, 267)
  --  other code --
}
```
效果展示
![canvas](/img/mxx/20170630_canvas/canvas_img.png)

上图两个实例展示的其实是同一张资源图 通过更改canvas中设置参数 即可显示不同的结果


利用此方法不断变化图片位置 可以做一个人物来回运动的轨迹
```javascript
-- other code --
++index;
ctx.clearRect(0, 0, canvas.width, canvas.height) // 清除画布
// 判断是否达到边缘
if( ctx.flag == 'right'){ // 向右跑
   -- other code --
   ctx.flag = 'left'
   ctx.drawImage(img, 96*(index%4), 96*2,96, 96, ctx.x, 10, 96*2, 96*2) //图片位置
} else if( ctx.flag == 'left'){  // 向左跑
   -- other code --
   ctx.flag = 'right'
   ctx.drawImage(img, 96*(index%4), 96, 96, 96, ctx.x, 10, 96*2, 96*2) //图片位置
}
-- other code --
```

![canvas](/img/mxx/20170630_canvas/girl.gif)

#### 案例 Dense aroma

实现思路  

* 1 随机位置产生气泡  
* 2 气泡实例不断变动运动轨迹


核心代码 1

```javascript
class Bubble {
  constructor () {
    this.x = Math.random() * width // x 方向
    this.y = Math.random() * height // y 方向
    this.cX = this.cY = Math.random() * step // 每次实例位置变动
    this.size = Math.random() * size
    ctx.ary.push(this)
  }
  erase () {
    this.x += this.cX
    this.y += this.cY
    return this
  }
  draw () {
    ctx.drawImage(img, this.x, this.y, this.size, this.size) // 绘制图案
  }
}
```

创建一个Bubble类 为其添加属性以及方法

核心代码 2

```javascript
setInterval(() => new Bubble(), startTime)
```
开启定时器 不断产生新的实例

核心代码 3

```javascript
setInterval(() => {
  ctx.clearRect(0, 0, canvas.width, canvas.height)
  ctx.ary.map((item) => item.erase().draw())
}, 80)
```

开启定时器 循环实例数组 数组中每一个实例执行erase以及draw方法
** 同时记得canvas画板的擦除 ** 不然会保留上一次的痕迹 出现叠加效果

[完整版核心代码GitHub地址](https://github.com/mengxxSELF/Bubble/blob/master/canvas.js)

效果展示
![canvas](/img/mxx/20170630_canvas/bubble.gif)



## Other

这里使用了setInterval来不断处理的位置变动 会发现有卡顿现象
因为如果使用 setTimeout  或者 setInterval 来处理动画  那必须使画面的更新频率要达到每秒60次 这样肉眼才能看到流畅的动画效果

经过@程总的指点 尝试使用 [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) 来改善卡顿问题

### window.requestAnimationFrame()

> The window.requestAnimationFrame() method tells the browser that you wish to perform an animation and requests that the browser call a specified function to update an animation before the next repaint. The method takes as an argument a callback to be invoked before the repaint

这个方法原理其实也就跟setTimeout/setInterval差不多，通过递归调用同一方法来不断更新画面以达到动起来的效果

但它优于setTimeout/setInterval的地方在于 它是由浏览器专门为动画提供的API，在运行时浏览器会自动优化方法的调用

requestAnimationFrame不需要使用者指定循环间隔时间，浏览器会基于当前页面是否可见、CPU的负荷情况等来自行决定最佳的帧速率，从而更合理地使用CPU

浏览器支持情况
![requestAnimationFrame](/img/mxx/20170630_canvas/canvas_timer.png)

关于此方法的资料 参见张鑫旭大神的[《CSS3动画那么强，requestAnimationFrame还有毛线用》](http://www.zhangxinxu.com/wordpress/2013/09/css3-animation-requestanimationframe-tween-%E5%8A%A8%E7%94%BB%E7%AE%97%E6%B3%95/)

[第三方插件 raf](https://www.npmjs.com/package/raf) 对其进行了封装

因此引入此模块来代替定时器处理这个问题

```javascript
let raf = require('raf')
-- other code --
raf(function tick () {
  ctx.clearRect(0, 0, canvas.width, canvas.height)
  ctx.ary.map((item) => item.erase().draw())
  raf(tick)
})
```
