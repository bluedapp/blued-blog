---
uuid: 45b69340-bd61-11e7-a8e7-57fb058b06ae
author: mengxxSELF
email: 2473536678@qq.com
github: https://github.com/mengxxSELF
avatar: https://avatars3.githubusercontent.com/u/20737114?v=4
title: video与audio在移动端的表现
date: 2017-10-30 18:58:29
tags: html5
banner: /img/mxx/20171030_video/video_banner.png
---

## video

video是HTML5提供的新标签 用于替代flash进行视频展示 然而video在移动端的表现情况却不如人意

<!--more-->

### 使用

```html
<video poster={`${source}?vframe/jpg/offset/0`} src={source}  controls="controls" />
```
或者是

```html
<video poster={`${source}?vframe/jpg/offset/0`} controls="controls">
  <source src={source} type="video/mp4">
</video>
```

浏览器支持情况

![use](/img/mxx/20171030_video/use.png)

在PC端表现还是不错的

### 常用属性

| 属性 | 属性值类型  | 含义 |
| --- | --- | --- | --- |
| autoplay | autoplay | 设置为自动播放 |
| controls | controls | 展示视频播放的控制条 |
| preload | preload | 视频预加载 如果使用 "autoplay"，则忽略该属性 |
| loop | loop | 开启循环播放 |
| src | string | 视频源 |
| muted | Boolean | 静音 |
| poster | string | 视频封面 |

#### controls

控制条包括：播放 暂停 定位 音量 全屏切换 字幕（如果可用）音轨（如果可用）

在iOS和Android中 控制条表现不同

左侧是Android中视频控制条的表现形式 右侧是iOS中视频控制条的表现形式

![control](/img/mxx/20171030_video/ios.png)

#### autoplay

此属性表示设置为默认播放 但实际上 这个效果在移动端体验非常不友好 并不能实现视频自动播放的功能

### 移动端实际样式

```html
<video>
  <source src={source} type="video/mp4" />
</video>
```

左侧是安卓系统表现 右侧是iOS系统表现

![actually](/img/mxx/20171030_video/control.png)

可以看到 移动端视频展现没有一个合适的封面图 实际应该展示的是视频的第一帧作为默认封面图

然后我们添加一个 poster 属性 用于展现封面

```html
<video poster={poster}>
  <source src={source} type="video/mp4" />
</video>
```

就可以正确展示视频封面了

#### 关于获取视频封面

项目中的视频是存储在七牛中的 七牛提供了一个图片地址 是截取视频第一帧

```javascript
let poster = `${source}?vframe/jpg/offset/0`
```

如果你无法拿到视频中的某一张图片 就需要另外处理了

* 视频设置自动播放 则默认使用视频第一帧作为封面图

如果视频有设置 autoplay 属性 则可以展现第一帧作为封面 但是移动端体验不好

* canvas截取视频封面

```javascript
  let canvas = document.createElement('canvas')
  let video = document.getElementById(`video`)
  let ctx = canvas.getContext('2d')
  canvas.width = video.offsetWidth
  canvas.height = video.offsetHeight
  ctx.drawImage(video, 0, 0)
  let img = document.createElement("img")
  img.src = canvas.toDataURL("image/png")
  document.body.appendChild(img)
```

这样是利用canvas截取视频 得到一个Img图片

** 视频跨域 则无法获取到图片 **

视频截取中 网页报错

![video](/img/mxx/20171030_video/crossDomain.png)

canvas中的图像进行操作时有跨域限制（canvas安全机制）,如在某一个域名项目中的canvas中加载非本域中的图像，在进行toDataURL或getImageData进行操作时抛出异常

所以如果你的项目中视频源不涉及到跨域问题 可以直接使用这种方式

如果视频不同源 则直接将canvas元素展现出来 配合CSS充当封面

```javascript
  let canvas = document.getElementById('canvas')
  let video = document.getElementById(`video`)
  let ctx = canvas.getContext('2d')
  canvas.width = video.offsetWidth
  canvas.height = video.offsetHeight
  ctx.drawImage(video, 0, 0)
```

#### 全屏播放

iOS系统中 默认是全屏播放 当你点击视频 iOS会自动全屏

Android 系统就是在展现的位置播放 当点击全屏按钮才会显示全屏

* 禁止iOS全屏播放

如果不想在iOS展示全屏效果 那么可以添加属性 playsinline 来禁止iOS的默认全屏

```javascript
<video ref='videoEl' src={source} {...videoProps} playsinline />
```

这样视频会在指定位置处播放 [和Android表现相同]

* Android 无法实现自动全屏 需要再处理

如果有设置 control 属性 则可以通过点击控制条中的全屏按钮实现全屏播放

还可以通过一个虚假的蒙层实现 全屏播放 当点击按钮的是 传入视频源 然后强制蒙层中的video元素100% 播放


###  常用方法

| 方法 | 含义 |
| --- | --- | --- | --- |
| play | 播放视频 |
| pause | 暂停视频 |

#### play

```javascript
componentDidMount () {
  // 尝试播放视频
  let target = findDOMNode(this.refs.videoEl)
  target.play()
}
```

然而 并没有什么用  PC端一切正常 但是在移动端是无法实现的

移动端的视频播放 必须通过用户执行点击动作 否则是无法实现播放的

```javascript
document.addEventListener('touchstart', function () {
  let video = document.querySelector('video')
  video.play()
})
```

项目开始的时候提出一个需求 类似于微博客户端效果 当视频滑动到用户可视区 即可自动播放此视频 当此视频滑离可视区 则视频停止播放

```javascript
window.addEventListener('scroll', () => {
  // 自动播放过一次之后 就禁止第二次自动播放了
  let {played} = this.state
  if (played) return
  let scrollTop = document.documentElement.scrollTop || document.body.scrollTop
  // 滑动距离 + 屏幕距离  > 盒子距离顶部 + 200
  if (scrollTop + screenH > boxTop + 200) {
    this.refs.videoEl.play()
    this.setState({played: true})
  }
})
```

可以看到在PC端 可以实现这种效果  然而 由于移动端 ** 必须通过用户点击屏幕的动作来触发播放事件 **  所以无法实现这个效果  

添加一个事件: 视频滑入可视区的时候 监听用户的 touch 事件 用户触发视频的播放

```javascript
window.addEventListener('scroll', () => {
  let {played} = this.state
  if (played) return
  let scrollTop = document.documentElement.scrollTop || document.body.scrollTop
  if (scrollTop + screenH > boxTop + 200) {
    document.addEventListener('touchstart', () => {
      this.refs.videoEl.play()
      this.setState({played: true})
    })
  }
})
```

功能上是可以实现的 但是这种方式非常不友好 体验的时候就能感觉到 当用户把视频滑动到可视区域 并不一定发生touch事件


## audio

audio 是HTML5提供的新标签 用于进行音频文件的播放

### 基本使用

```javascript
let audioProps = {
 src: '../music.mp3',
 loop: 'loop'
}
<audio {...audioProps} />
```

### 常用属性

| 属性 | 属性值类型  | 含义 |
| --- | --- | --- | --- |
| autoplay | autoplay | 设置为自动播放 |
| controls | controls | 展示音频播放的控制条 |
| preload | preload | 音频预加载 如果使用 "autoplay"，则忽略该属性 |
| loop | loop | 开启循环播放 |
| src | string | 音频源 |

#### autoplay 自动播放

音频文件在ios系统中无法实现自动播放的这个效果

> User Control of Downloads Over Cellular Networks
> In Safari on iOS (for all devices, including iPad), where the user may be on a cellular network and be charged per data unit, preload and autoplay are disabled. No data is loaded until the user initiates it. This means the JavaScript play() and load() methods are also inactive until the user initiates playback, unless the play() or load() method is triggered by user action. In other words, a user-initiated Play button works, but an onLoad="play()" event does not.

简而言之 ios认为自动播放音频视频是不友好的行为 不想用户浪费过多的流量，禁止了自动播放的功能

所以即使你在标签中添加了 autoplay属性 在移动端也不会触发视频或者音频的播放

* 解决方案1

如果就是想要用户开启播放 可以为页面绑定一个 touchstart 点击事件  这样在页面点击的时候 触发此音频播放

```javascript
document.addEventListener('touchstart', function () {
  let audio = document.getElementById('audio')
  audio.play()
})
```

* 解决方案2

虽然在浏览器中有此限制 但是在微信中却可以做到自动播放

Android 中可以使用 [ios 无效]

```javascript
document.addEventListener('DOMContentLoaded', function () {
  let audio = document.getElementById('audio')
  audio.play()
})
```
iOS 可以使用 [Android 也可以]

```javascript
document.addEventListener("WeixinJSBridgeReady", function () {
  let audio = document.getElementById('audio')
  audio.play()
}, false)
```

#### src

测试的时候使用了一首来自5sing的歌曲链接

```javascript
let src = 'http://data.5sing.kgimg.com/G106/M00/0A/0F/SpQEAFljNBqATRL-ADks3ezSCI8032.m4a'
```
然后发现无法播放  是由于audio不支持此格式的音频文件

audio只支持三种格式的音频文件

![video](/img/mxx/20171030_video/mime.png)

#### 创建实例

其实还可以通过new 来创建一个 audio 实例  就类似于 Image 来创建 img 一样

```javascript
let src = 'https://os4ty6tab.qnssl.com/web/static/AllProjects/music-0c0d070d.mp3'
this.audio = new Audio(src)
this.audio.setAttribute('loop', 'loop')
```
这样就不必讲元素放入页面中也可以播放音乐

### 第三方视频插件

其实原生的video标签有的时候无法满足UI需求 比如一般需要另行处理播放或者全屏按钮

![video](/img/mxx/20171030_video/video4.png)

可以看到页面元素中 下面控制条部分并没有被解析为标签元素展示 整个视频源和控制条都在一个video元素中展示 所以无法处理这些按钮 比如调整位置或者样式

#### video.js

> Video.js 是一个通用的在网页上嵌入视频播放器的 JS 库，Video.js 自动检测浏览器对 HTML5 的支持情况，如果不支持 HTML5 则自动使用 Flash 播放器

[官网](http://videojs.com/)

[npm](https://www.npmjs.com/package/video.js)

[Github](https://github.com/videojs/video.js)

* 使用

```javascript
videojs(videoID, {}, function(){
  // console.log(this)
  this.play()
})
```
可以实现视频自动播放功能

传入的参数是  

| 属性 | 类型 | 含义 |
| --- | --- | --- | --- |
| videoID | string | video标签元素的id |
| option | {} | 标签参数 |
| cb | function | 回调函数  其中的this表示为当前播放源 |

去看了源码 有这么一段

![video](/img/mxx/20171030_video/code.png)

简化版

```javascript
function videojs(id, options, ready) {
  let tag;

  if (typeof string) {
    tag = $(`#${id}`)
  } else {
    tag = id
  }
  ......
}  
```

所以第一个参数那里 不只是可以传递 id  也可以直接传递 DOM节点 比如

```javascript
videojs(document.querySelector('video'), {}, function(){
  console.log(this)
  this.play()
})
```

查看页面元素 可以看到 每一个部分都被解析为元素节点了

![video](/img/mxx/20171030_video/video5.png) 这样就可以调整按钮样式了

** video.js 提供了在React中的使用方式  

[react-video](http://docs.videojs.com/tutorial-react.html)

```javascript
import videojs from 'video.js'

export default class VideoPlayer extends React.Component {
  componentDidMount() {
    this.player = videojs(this.videoNode, this.props, function onPlayerReady() {
      console.log('onPlayerReady', this)
    });
  }

  componentWillUnmount() {
    if (this.player) {
      this.player.dispose()
    }
  }

  render() {
    return (
      <div data-vjs-player>
        <video ref={ node => this.videoNode = node } className="video-js"></video>
      </div>
    )
  }
}
```

有一些坑请看程总文章 [video.js文档笔记(videojs)](https://github.com/GodEngine/videojs_hls)
