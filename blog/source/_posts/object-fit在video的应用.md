---
uuid: 45b69340-bd61-11e7-a8e7-57fb058b06ae
author: mengxxSELF
email: 2473536678@qq.com
github: https://github.com/mengxxSELF
avatar: https://avatars3.githubusercontent.com/u/20737114?v=4
banner: https://user-gold-cdn.xitu.io/2018/9/7/165b29eb537d12cf?w=621&h=335&f=png&s=17010
title: object-fit在video的应用
date: 2018-09-07 16:14:04
tags: css3
---

这周在处理视频播放的问题，全屏播放，在css中将video宽高强制设置为100%，发现在某些手机中会有两侧黑边

![gap](https://user-gold-cdn.xitu.io/2018/9/5/165a863815e2a603?w=1026&h=274&f=png&s=147738)

并没有全屏，解决方案，使用一个CSS属性

```css
object-fit: cover;
```
但是为什么这个可以实现呢，今天来探究一下

## 替换元素

> 其内容不受CSS视觉格式化模型控制的元素,比如image,嵌入的文档(iframe之类),叫做替换元素

* 视觉格式化模型

可以理解为在视觉媒体上如何处理文档树。参考BFC,IFC

CSS渲染模型不考虑替换元素内容的渲染。这些替换元素的展现独立于CSS。

object,video,textarea,input也是替换元素,audio和canvas在某些特定情形下为替换元素。

使用CSS的content属性插入的对象是匿名替换元素。

* 替换元素

替换元素通常有其固有的尺寸:一个固有的宽度,一个固有的高度和一个固有的比率

比如图片和视频有宽度和高度,是有自身宽高比率的


## object-fit

| 参数 | 取值 |
| --- | --- | --- | --- |
| fill | 被替换的内容大小可以填充元素的内容框。 整个对象将**完全填充**此框。 如果对象的高宽比不匹配其框的宽高比，那么该对象将**被拉伸以适应** |
| contain | 被替换的内容将被缩放，以在填充元素的内容框时保持其宽高比。 整个对象在填充盒子的同时**保留其长宽比**，因此如果宽高比与框的宽高比不匹配，该对象**将被添加黑边** |
| cover | 被替换的内容大小**保持其宽高比**，同时填充元素的整个内容框。 如果对象的宽高比与盒子的宽高比不匹配，该对象将**被剪裁**以适应 |
| none | 被替换的内容大小**保持其宽高比**，而且被替换的内容**尺寸不会被改变** |
| scale-down |内容的尺寸就像是指定了none或contain，取决于哪一个将导致更小的对象尺寸|

各个浏览器支持情况

![use](https://user-gold-cdn.xitu.io/2018/9/5/165a9914fcb47d78?w=1339&h=373&f=png&s=58499)

object-fit 的默认值为fill，完全填充

### fill

```css
   img {
      border: 10px solid rgb(207, 154, 94);
      background: rgb(143, 131, 120);
      object-fit: fill;
      width: 300px;
      height: 300px;
      margin: 100px;
    }
    <img src="https://www.bldimg.com/eshop/photos/1535623413_43938.jpg?imageView2/0/w/250" alt="">
```


![fill](https://user-gold-cdn.xitu.io/2018/9/7/165b1e24971b416a?w=875&h=404&f=png&s=584852)

从左到右展示， img空元素  原始图片 img按照fill填充了原始图片之后

可以看到，图片被强制填充整个Img盒子，完全变形了

### contain

```css
   img {
      border: 10px solid rgb(207, 154, 94);
      background: rgb(143, 131, 120);
      object-fit: contain;
      width: 300px;
      height: 300px;
      margin: 100px;
    }
    <img src="https://www.bldimg.com/eshop/photos/1535623413_43938.jpg?imageView2/0/w/250" alt="">
```

![contain](https://user-gold-cdn.xitu.io/2018/9/7/165b1e5bb8a28800?w=928&h=418&f=png&s=646693)

从左到右展示， img空元素  原始图片 img按照contain填充了原始图片之后

图片保持了原始宽高比

我们找一个宽高比小于1的图片看看

```css
   img {
      border: 10px solid rgb(207, 154, 94);
      background: rgb(143, 131, 120);
      object-fit: contain;
      width: 300px;
      height: 300px;
      margin: 100px;
    }
    <img src="https://web.bldimg.com/cblued/static/img.1cmoujbgofhbhi.png?imageView2/0/w/250" alt="">
```

![contain](https://user-gold-cdn.xitu.io/2018/9/7/165b1ec4d3068935?w=863&h=416&f=png&s=617254)

contain属性 会选择边比较长的那一个为准，另外的一个边会保持居中

| 宽高 | 结果 |
| --- | --- | --- | --- |
| 宽度 > 高度 | 水平方向100% 垂直方向居中 |
| 宽度 < 高度 | 垂直方向100% 水平方向居中 |

所以这个属性也可以用来处理图片居中问题了

比如这个，商品的图片居中

![goods](https://user-gold-cdn.xitu.io/2018/9/5/165a9a6a1501d3cb?w=409&h=380&f=png&s=76418)

之前的css

```css
    img {
        width: 6.06rem;
        display: block;
        margin: 0 auto;
    }
```

限制宽度，高度自适应。

但是这样有一个问题，就是在做懒加载的过程中，由于没有固定盒子高度，导致在滑动页面的时候会有一种“跳”的感觉，用户体验不好。

但是如果固定高度，限制最大width，在遇到宽度比高度要大很多的长方形商品图，会变形

```css
    img {
        height: 6.06rem;
        max-width: 100%;
        display: block;
        margin: 0 auto;
    }
```

![css](https://user-gold-cdn.xitu.io/2018/9/5/165a9ad522668485?w=420&h=367&f=png&s=104854)

可以调整为

```css
img {
    width: 100%;
    height: 6.06rem;
    object-fit: contain;    
}
```
![end](https://user-gold-cdn.xitu.io/2018/9/5/165a9adeb4b4e525?w=446&h=369&f=png&s=81318)

## cover

```css
   img {
      border: 10px solid rgb(207, 154, 94);
      background: rgb(143, 131, 120);
      object-fit: cover;
      width: 300px;
      height: 300px;
      margin: 100px;
    }
```

![cover](https://user-gold-cdn.xitu.io/2018/9/7/165b1efc99dadc5d?w=880&h=415&f=png&s=625234)

从左到右展示， img空元素  原始图片 img按照cover填充了原始图片之后

图片的宽高比没变，但是多余的部分被裁剪掉了

## none


```css
   img {
      border: 10px solid rgb(207, 154, 94);
      background: rgb(143, 131, 120);
      object-fit: none;
      width: 300px;
      height: 300px;
      margin: 100px;
    }
    <img src="https://www.bldimg.com/eshop/photos/1535623413_43938.jpg?imageView2/0/w/250" alt="">
```

![none](https://user-gold-cdn.xitu.io/2018/9/7/165b1f220321414c?w=904&h=408&f=png&s=587489)

从左到右展示， img空元素  原始图片 img按照none填充了原始图片之后

> 被替换的内容大小**保持其宽高比** 并且尺寸不变

由于这里的图片源宽度(250px)是小于img的设置宽度(300px), 图片完整的展示到img盒子中

找一个图片源大于盒子本身大小的

![none](https://user-gold-cdn.xitu.io/2018/9/7/165b1fa00f269c13?w=926&h=490&f=png&s=871326)

从左到右展示， img空元素  原始图片 img按照none填充了原始图片之后

原始图片大小 576 x 636

none 好像是选取了原始图中一块 300 x 300的区域大小的资源块 填充到Img盒子

| 图片源和Img盒子大小对比 | 结果 |
| --- | --- | --- | --- |
| 图片源 >  Img盒子 | 选取图片源一块进行填充  |
| 图片源 <  Img盒子 | 完整展示 |

## scale-down

```css
   img {
      border: 10px solid rgb(207, 154, 94);
      background: rgb(143, 131, 120);
      object-fit: scale-down;
      width: 300px;
      height: 300px;
      margin: 100px;
    }
```

 scale-down的表现形式取决于 图片源 与 Img盒子 的大小

 如果  图片源 大于 Img盒子，其呈现结果和 contain 一样，保持图片源宽高比进行填充

![scale-down](https://user-gold-cdn.xitu.io/2018/9/7/165b202c210c3026?w=839&h=374&f=png&s=630571)

如果  图片源 小于 Img盒子, 其呈现结果和 none 一样，直接展现图片

![scale-down](https://user-gold-cdn.xitu.io/2018/9/7/165b206c01103413?w=815&h=377&f=png&s=479489)

## object-fit 的作用

在使用了object-fit的img元素，就像一个盒子，根据object-fit的属性值，填充真正的图片源

也就是设置了

```css
   img {
      border: 10px solid rgb(207, 154, 94);
      background: rgb(143, 131, 120);
      width: 300px;
      height: 300px;
      margin: 100px;
    }
```

这些大小的限制是对于 img盒子

```css
object-fit: xxx;
```
这个属性是用于处理真正展示的图片源，也就是控制最后呈现的图片源的样式



## Object-position

Object-position 可以按照 background-position 理解，用来决定替换资源的展示位置

[w3c 对于 background-position 的解释](http://www.w3school.com.cn/cssref/pr_background-position.asp)

可以简单理解为，你在Img盒子的哪个位置开始放置背景图 （相对于左上角）

![background-position](https://user-gold-cdn.xitu.io/2018/9/7/165b20acc536f0e1?w=831&h=489&f=png&s=74604)

```css
    img{
      border: 10px solid rgb(207, 154, 94);
      background: rgb(143, 131, 120);
      object-position: 150px 150px;
      width: 300px;
      height: 300px;
      margin: 100px 20px;
    }
```

![position](https://user-gold-cdn.xitu.io/2018/9/7/165b2160fbd4df91?w=922&h=389&f=png&s=343607)

object-position设置为img盒子的一半, 三个图片的展示样式

## 应用于video

在处理video元素播放的时候，我们经常能看到视频会自动补充黑边

```html
  <div>
    <video controls src="。。。"></video>
  </div>
  <div>
    <video controls src="。。。"></video>
  </div>
```

```css
    div {
      width: 350px;
      height: 500px;
      float: left;
      margin: 100px 20px;
      border: 10px solid rgb(207, 154, 94);
      background: rgb(143, 131, 120);
    }
     video {
      width: 100%;
      height: 100%;
    }
```

![PC](https://user-gold-cdn.xitu.io/2018/9/7/165b22b88f7d5ff9?w=704&h=444&f=png&s=350513)

虽然这里已经写了video元素设置为父级盒子大小，但是里面的视频源在播放的时候，并没有按照想要的占据全屏

加入object-fit属性处理一下

```css
video {
  object-fit: fill;
}
```
![pbject-fit](https://user-gold-cdn.xitu.io/2018/9/7/165b27ea8f376b76?w=572&h=376&f=png&s=322574)

## 参考文章

* 补充一些 video 的知识

[video 和 audio在移动端的表现](http://web.blued.cn/2017/10/30/video%E4%B8%8Eaudio%E5%9C%A8%E7%A7%BB%E5%8A%A8%E7%AB%AF%E7%9A%84%E8%A1%A8%E7%8E%B0/)

* [阻止视频在移动端页面中全屏播放](https://www.jianshu.com/p/8c17967adee7)

* [半深入理解CSS3 object-position/object-fit属性](https://www.zhangxinxu.com/wordpress/2015/03/css3-object-position-object-fit/)
