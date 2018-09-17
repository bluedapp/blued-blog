---
uuid: a23d4ec0-5d6f-11e8-b090-0fc9bfa03ecf
author: jiandandkl
email: emolingzhu@126.com
github: https://github.com/jiandandkl
avatar: https://avatars1.githubusercontent.com/u/16009933?v=3
title: wx.getUserInfo废弃之后 -- 弹窗获取信息
date: 2018-05-22 11:24:27
tags: 小程序
---

  > 先吐槽下微信,这么基础的API晚上11点说废就废,程序猿何苦为难程序猿.(不过最近好像又恢复了...)

  官方是推荐使用button点击获取用户信息,但有时候页面多这么一个按钮很突兀啊、或者tabbar切换的时候想要先获取用户信息再展示页面就很尴尬了。
  目前碰到的情况就是第二种,既然必须要点击button来获取,就采用模态框点击登录的方法了。
  但小程序的模态框实在鸡肋(尤其很难理解字符限制七个字),所以自定义模态框。

  * loading模态框

  目前小程序是不支持自定义的loading图,只有`success`、`loading`、`none`,所以写个公共组件,可以传入自定义的图片及文字。
  其中的关键方法为: `getCurrentPages()[getCurrentPages().length - 1]`,获取当前页实例。拿到当前页实例后便可以将自定义内容传到data中。

  showToast.js:

  ```javascript
  function showToast(obj) {
    // 获取当前page实例
    const that = getCurrentPages()[getCurrentPages().length - 1];
    that.setData({
      showToast: obj
    })
  }

  ```
  showToast.wxml:
  ```javascript
  <view class="middle" wx:if="{{showToast.icon}}">
    <image class="toast-icon" src="{{showToast.icon}}" mode="scaleToFill" wx:if="{{showToast.icon}}" />
    <text wx:if="{{showToast.content}}" class="toast-content">{{showToast.content}}</text>
  </view>
  ```

  loading.js:
  ```javascript
  const feedbackApi = require('../../components/showToast/showToast')
  feedbackApi.showToast({
    content: '正在获取数据',
    icon: '../../images/loading.svg',
    duration: 3000
  })
  ```

  效果图:

  ![](/img/dujun/loading.gif)

  > 推荐个loading图网站 [https://loading.io](https://loading.io/)

  * 登录模态框

  有自定义模态框在实现登录就比较简单了,在wxml绑定两个按钮就可以了
  
  ```javascript
  <text wx:if="{{showToast.title}}" class="toast-title">{{showToast.title}}</text>
  <view wx:if="{{showToast.btnCancel && showToast.btnUserInfo}}">
    <text class='toast-text'>请先登录~</text>
    <button class='toast-btn' catchtap='click_cancel'>{{showToast.btnCancel}}</button>
    <button class='toast-btn toast-btn2' open-type="getUserInfo" bindgetuserinfo="click_user_info">{{showToast.btnUserInfo}}</button>
  </view>
  ```

  my.js:
  ```javascript
  feedbackApi.showToast({
    title: '尚未登录',
    btnCancel: '知道了',
    btnUserInfo: '登录',
    bg: '#fff',
    duration: 10000
  }),
  // 点击取消
  click_cancel(e) {
    feedbackApi.hideToast()
    wx.switchTab({
      url: '../index/index'
    });
  },
  // 点击登录
  click_user_info(e) {
    const data = e.detail
    if (data.userInfo) {
      this.setData({
        userInfo: data.userInfo
      })
      wx.setStorageSync('userInfo', data.userInfo)
      feedbackApi.hideToast()
    } else {
      wx.switchTab({
        url: '../index/index'
      })
    }
  }
  ```

  效果图:
  
  ![](/img/dujun/login.gif)

  详细代码可见 [https://github.com/jiandandkl/toast-demo](https://github.com/jiandandkl/toast-demo)

  