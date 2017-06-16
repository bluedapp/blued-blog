---
uuid: 5d29f830-41e4-11e7-823d-d1797b475f45
title: CSS--项目中那些居中问题
date: 2017-06-10 15:24:32
author: mxx
tags: css
github: https://github.com/mengxxSELF
avatar: https://avatars0.githubusercontent.com/u/20737114?v=3
banner: /img/mxx/20170526_Gleaning_css/banner_css.png
---

元素的居中问题可以从以下方面实现

* 自身属性设置
* flex布局
* 定位

### 垂直居中

#### 1 自身属性设置

* 1.1 height && line-height

使用案例 [基友词典](http://app.blued.cn/hd/2017-dictionary)  选项设置

* 1.2 padding || margin

使用案例 [白色情人节](http://app.blued.cn/hd/2017-Valentines)  排行榜组件

组件高度是因为padding或者margin而产生

* 1.3 display && vertical-align

```html
<div> <img  src={...}/>  </div>

--style--
div{
  display: table-cell;
  vertical-align: middle;
}

```
效果如下
![css](/img/mxx/20170526_Gleaning_css/css-table.png)

这种垂直居中方式利用了行内元素属性 vertical-align 所以如果子级是块元素 则不会有垂直居中效果

* 1.4 position && margin-top
将已知大小的元素定位在最高层级位置并实现垂直方向居中效果

```css
div{
  height:50px;
  position:absolute;
  margin-top:25px;
  ...
}
```
使用案例 [泼水节](http://app.blued.cn/hd/2017-songkran) 播放按钮

#### 2 flex  align-item属性

``` css
display: flex;
align-items:Center;

```

使用案例 [春光乍泄旅游季](http://app.blued.cn/hd/2017-spring)  排行榜组件

#### 3 借助 position

```html
<div> <p>BLUED</p> </div>

--style--

p{
  position: absolute;
  left:0;
  right: 0;
  top:0;
  bottom: 0;
  margin:auto;
  width:100px;
  height:100px;
}
```
这种方式需要知道元素的具体大小

使用案例 [基友词典](http://app.blued.cn/hd/2017-dictionary?id=75111#questions#questions)  模态框组件


### 水平居中

#### 1 自身属性设置

* 1.1 display && text-align

```scss
display:inline-block;
text-align:center;
```

将元素设置为inline-block 然后使用text-align:center 可实现元素水平居中

使用案例 [基友词典](http://app.blued.cn/hd/2017-dictionary)  上一题按钮处理

适用于 元素宽度未知

* 1.2 width && margin

使用案例 [Photo Lab](http://app.blued.cn/hd/2017-photolab)  Header部分头像处理

适用于 元素的宽度大小已知

* 定位 left && margin-left

使用案例 [缘定三生](http://app.blued.cn/hd/2017-lovers) 用户排名

适用于 需要使用定位的固定宽度大小元素

#### 2 flex

```html
<div>
  <span>spot</span>
</div>

--style--

div{
  display:flex;
  justify-content:center;
}

```

#### 3 对于需要定位的 未知宽度的元素 如何实现水平居中

* 3.1 行内元素

考虑使用text-align属性

```html
<div>
 <span>BLUED</span>
</div>

--style--
div{
  ...
  position:relative;
  text-align:center;
}
span{
  position:absolute;
  width:100px;
}

```

效果如下
![css](/img/mxx/20170526_Gleaning_css/position-inline.png)

图中白色竖线是处于水平居中位置

当元素使用定位之后，就具有了display的属性 所以可以给span元素添加width属性

然而span元素并未像我们预期中处于居中分布的状态 而是与父级容器的中垂线左侧对齐显示


如果我们在前面添加一行文字

```html
<div>
  CENTER <span>BLUED</span>
</div>
```
效果如下
![css](/img/mxx/20170526_Gleaning_css/position-inline2.png)

可以看到 文字实现了居中效果

这是因为：text-align属性作用的不是absolute元素， 而是absolute元素前面的文字 [或者可以说是具有inline或者inline-block属性的元素]

所以span元素位于中垂线左侧对齐的原因是：标签内有不占据任何空间的匿名文字节点元素 ，导致span元素就紧挨着此匿名文字节点元素显示


处理方案

* 方案1 属性设置

```html
<div> <img src={...}/> </div>

--style--

img{
  position: absolute;
  left:0;
  right: 0;
  top:0;
  bottom: 0;
  margin:0 auto;
}

```
效果如下
![css](/img/mxx/20170526_Gleaning_css/Can.png)


* 方案2 外面再包装一层盒子

```html
<div>
  <p> <img src={...} />  </p>
</div>

--style--
div{
  position:relative;
  ...
}
p{
  position:absolute;
  bottom:10px;
  width:100%;
  text-align:center;
}

```
将定位属性放在盒子元素上  居中目标元素受到text-align影响 水平方向为居中位置


* 3.2 块级元素

虽然inline元素设置position为absolute可以使其具有块级元素的特性  但是如果是块级元素直接absolute 却无法利用text-align进行水平方向的处理


未知高度的块级元素实现层级定位并且水平方向保持居中处理方案同行内元素相同
不过需要注意的是如果使用的是第一种处理方式 当此元素未设置高度 其高度会被撑到父级高度大小

```html
<div> <p>BLUED</p> </div>

--style--

p{
  position: absolute;
  left:0;
  right: 0;
  top:0;
  bottom: 0;
  margin:0 auto;
}
```

不设置元素具体高度大小 显示如下

效果如下
![css](/img/mxx/20170526_Gleaning_css/position-display.png)

可以看到 块级元素p的高度 被撑开了
所以这种方式需要处理高度


#### Tips
* flex 各浏览器使用情况

![flex](/img/mxx/20170526_Gleaning_css/support_flex.png)

* 当元素具有absolute 属性的时候 此元素会具有块级元素的特性

```css
display:inline-block;
```
* bourbon 中提供的 ellipsis 方法
```scss
@include ellipsis()
```
在Android和iOS中表现不同

```html
<a> <span>所爱隔山海 山海皆可平</span> </a>

-- style --
a{
  background: #fff;
  ...
  span{
    @include ellipsis(3rem)
  }
}

```

Android中 效果如下
![Android](/img/mxx/20170526_Gleaning_css/Android.png)
iOS 效果如下
![iOS](/img/mxx/20170526_Gleaning_css/ios.png)
可以看到文字位置有偏差


其源码为
```scss
@mixin ellipsis(
  $width: 100%,
  $display: inline-block
) {
  display: $display;
  max-width: $width;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  word-wrap: normal;
}
```
可以看到元素默认display属性是inline-block  对于具有此属性的元素 其默认的对齐方式为基线对齐
```css
vertical-align:baseline    
```
这个属性导致Android和iOS中表现不同

解决方案:
为元素添加属性
```css
display:block
```
或者
```css
vertical-align:middle  
```
