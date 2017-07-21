---
uuid: 3484d0f0-683e-11e7-a43f-f3644c513724
author: mxx
email: mengxxself
title: ReactNative--项目初探
date: 2017-07-14 10:43:22
tags: ReactNative
github: https://github.com/mengxxSELF
avatar: https://avatars0.githubusercontent.com/u/20737114?v=3
---


## RN 项目初探

* 1 RN组件
* 2 项目分析

<!-- More -->


## 1 RN 组件

### 第三方模块

#### 1 面板切换

三个主面板的切换以及分类页面有子级面板 使用的是模块 [react-native-scrollable-tab-view](https://github.com/skv-headless/react-native-scrollable-tab-view)


```html
<ScrollableTabView {...ScrollProps} renderTabBar={() => <ScrollTabBar />} >
  <HotList {...HotListProps} />
  <VarietyGroup {...VarietyGroupProps} />
  <FollowList {...FollowListProps}  />
</ScrollableTabView>
```

主面板以及子级面板都没有使用组件提供的默认tab样式 而是自定义了tab组件

```javascript
renderTabBar={() => <ScrollTabBar /> }
```

组件的面板的切换并没有使用默认处理的方法 而是在用户点击Tab的时候调用goToPage方法 进行面板的切换

```html
-- other code --

<TouchableOpacity  onPress={handlerPress} >
  <Text>
      {item.name}
  </Text>
</TouchableOpacity>

-- other code --
handlerPress (index) {
  this.props.goToPage(index)
}

-- other code --
```


#### 2 Banner 滑动

首页的Banner部分 使用第三方模块 [react-native-swiper](https://github.com/leecade/react-native-swiper)

#### 3 主面板内容部分

面板的内容部分使用的是第三方组件 [PullToRefreshListView](https://github.com/react-native-component/react-native-smart-pull-to-refresh-listview/)

这个组件提供了下拉刷新以及上滑加载的功能

```javascript
<PullToRefreshListView
  viewType={PullToRefreshListView.constants.viewType.listView}
  dataSource={this.state.dataSource}
  renderRow={this._renderRow}
  renderHeader={this._renderHeader}
  renderFooter={this._renderFooter}
  onRefresh={this._onRefresh}
  onLoadMore={this._onLoadMore}
  -- other props --
/>
```

组件还提供了一个方法beginRefresh  用于强制执行一次下拉刷新

```javascript
this.props.beginRefresh()
```


### ReactNative 内部组件

#### 1 关注的推荐列表

关注面板的推荐部分 使用RN内部提供的滑动组件 [ScrollView](http://reactnative.cn/docs/0.46/scrollview.html#content)

此模块在默认状态下是垂直方向滑动的

#### 2 触摸点击

[TouchableOpacity](http://reactnative.cn/docs/0.46/touchableopacity.html#content)组件用于封装视图，使其可以正确响应触摸操作

```javascript
<TouchableOpacity onPress={this._onPressButton}>
  <Image  source={require('image!myButton')} />
</TouchableOpacity>
```


## 2 项目分析

1 核心组件 List

项目核心组件 List 处理页面的渲染

使用上文中提到的组件[PullToRefreshListView](https://github.com/react-native-component/react-native-smart-pull-to-refresh-listview/)

三个主面板的数据渲染都是基于List类

List中 定义state对象结构

```javascript
const ds = new ListView.DataSource({
  rowHasChanged (r1, r2) {
    return r1.lid !== r2.lid
  }
})
this.state = {
  ds,
  dataSource: ds.cloneWithRows([]),
  data: []
}
```

这里的dataSource 就是下文进行页面渲染的数据来源

```javascript
dataSource={state.dataSource}
```

所以页面在处理上拉获取更多数据 或者 下滑获取最新数据时 只需要重置state就可以了

2 数据组件 xxxListBase.js

** 此组件继承自List组件 **

```javascript
export default class RecommendListBase extends List {
  ......
}
```
此组件只是定义了几个获取数据的方法

```javascript
 async _getData() {......}
 async _getLoadMoreData() {......}
 async _getRefreshData() {......}
```

3 状态组件 xxxList.js

** 此组件继承自同组的xxxListBase组件 **

```javascript
export default class RecommendList extends RecommendListBase {
  ......
}  
```

因为继承自同组base类  所以可以调用其对应父类的方法 从而用来处理状态state

```javascript
  async _onLoadMore () {
    -- other code ---
    let getData = await this._getData()
    this.setState({data: getData})
    -- other code ---
  }
  async _onRefresh () {......}
```

4 项目入口

项目的入口文件 APP/index.js  是上文中提到的主面板部分

```html
<ScrollableTabView {...ScrollProps} >
  <HotList {...HotListProps} />
  <VarietyGroup {...VarietyGroupProps} />
  <FollowList {...FollowListProps}  />
</ScrollableTabView>
```
可以看到 每一个子元素都是一个状态组件 每一个状态组件可以追溯到最终的父类 List 从而具有刷新与获取更多数据的功能

5 祖宗组件

项目入口文件 index 继承自父类 libs/MethodBase

```javascript
export default class Live extends MethodBase {
  ......
}
```
MethodBase 组件中主要封装一些与客户端交互的方法 和 一些基本的方法 比如 获取数据 获取语言等


### 组件是如何实现下拉刷新的

[下拉刷新展示](https://www.processon.com/view/link/59702617e4b0b3c2da59db8d)
![rn](/img/mxx/20170714_RN/rn_refresh.png)

5.1 当第一次进入页面 或者说 通过点击Tab第一次进入几个主面板的时候

触发对应List组件生命周期函数 componentDidMount

```javascript
componentDidMount () {
  -- Hide Code --
  this._pullToRefreshListView.beginRefresh()
}
```

实现一次强制下拉刷新 去执行对应状态组件中的方法
```javascript
async _onRefresh () {......}
```
从而引起state变化 -> 变动核心组件List 中dataSource属性值 -> 页面数据初次渲染


当面板已经完成初次加载之后 数据的刷新就需要手动将页面下滑一定距离 从而实现数据刷新


** 分类列表的子级面板原理相同 **


5.2 tab切换 如果间隔时间超过五分钟 切换时自动刷新

5.2.1 主面板 三个tab之间切换 如何实现五分钟之后自动刷新

主面板的生命周期函数 在面板切换时候

```javascript
componentWillUpdate () {
  -- other code --
  this.props.xxxListRefresh()
}
```

然后在父级组件 index.js 找到此函数

```javascript
xxxListRefresh () {
  this._xxxList.autoRefresh()
}
```

然后在 List中 定义有定义autoRefresh方法

```javascript
autoRefresh () {
  if (!this.enableAutoRefresh) return
  this.enableAutoRefresh = false
  this._pullToRefreshListView && this._pullToRefreshListView.beginRefresh()
}
```

这里有一个标志位 enableAutoRefresh 用来判断此面板是否可以利用beginRefresh进行强制刷新


而此标志位就是五分钟刷新的关键:
  如果组件展示时间已经大于五分钟 则此标志位为TRUE -> 强制刷新
  如果组件展示时间已经小于五分钟 则此标志位为FALSE -> 直接return  无法强制刷新页面

** 组件在哪里改变此标志位 **

在 List 组件中 生命周期 componentDidMount 中 将此标志位设置为 TRUE

```javascript
componentDidMount () {
  -- hide code --
  this.enableAutoRefresh = false
  -- hide code --
}
```
所以所有的面板在首次渲染之后 此标志位都已经为FALSE了 无法再去执行 autoRefresh 方法

** 组件在哪里改变此标志位 为TRUE **

在祖宗级组件 MethodBase 中

```javascript
if (ifEnough) {
  this.shouldAutoRefresh()
}
```
当状态足够五分钟  执行 shouldAutoRefresh 方法 此方法为

```javascript
shouldAutoRefresh (flag) {
  this._hotList.shouldAutoRefresh()
  this._varietyGroup.shouldAutoRefresh()

  -- other code --
  this._hotList.autoRefresh()
  -- other code --
  this._varietyGroup.autoRefresh()

  -- other code --
}
```
这里执行组件的 shouldAutoRefresh 方法 此方法在 List 中

```javascript
shouldAutoRefresh () {
  this.enableAutoRefresh = true
}
```
用以将标志位置为 TRUE 从而可以执行 autoRefresh 方法 页面实现强制刷新



[项目流程图展示](https://www.processon.com/view/link/597015f4e4b0b3c2da59c8d2)
![rn](/img/mxx/20170714_RN/rn.png)
