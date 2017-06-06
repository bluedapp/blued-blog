---
uuid: 1fabda00-40f9-11e7-b46f-5fe776c18c7e
title: 调用微信JS-SDK自定义分享内容
date: 2017-05-25 11:20:38
author: dujun
tags: JavaScript
---
调用微信SDK自定义分享内容
## 调用微信SDK自定义分享内容
### 1.本地开发测试
#### 1.1使用测试账号获取相关参数

* 使用个人微信登录公众平台接口测试号申请: [测试号申请](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)

* 登录后拿到appId和appsecret
<img src="/img/dujun/appID.jpeg" alt="appID" width="50%">

* 配置安全域名(以公司blued举栗子)
<img src="/img/dujun/安全域名.jpeg" alt="安全域名" width="50%">
<font color=#F08080>(注意:域名不能配置IP地址)</font>
#### 1.2 获取接口签名
* 用appID和appsecret获取token
``` javascript
https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appId={app_id}&secret={secret}
```
* 用token获取jsapi_ticket
```javascript
https://api.weixin.qq.com/cgi-bin/ticket/getticket?type=jsapi&access_token={token}
```
* 生成signature
```javascript
https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=jsapisign
```
微信 JS 接口签名校验工具
<img src="/img/dujun/signature.jpeg" alt="signature" width="50%">
noncestr: 随机字符串，由开发者随机生成
timestamp: 由开发者生成的当前时间戳 ```parseInt(new Date().getTime() / 1000)```
#### 1.3 设置反向代理
* 更改host，在最下方加入: 127.0.0.1 app.blued.cn
``` javascript
sudo vi /etc/hosts
```
* nginx反向代理
 安装nginx: ```brew install nginx```
 查看安装位置: ```brew list nginx```
 更改nginx配置: ```vi /usr/local/etc/nginx/nginx.conf```
 在最后一个大括号前添加 ```include ./conf.d/*.conf;```
 打开nginx/conf.d，创建并编辑: ```vi app.blued.cn-localhost.conf```

 添加:
 ``` javascript
 server {
  listen  80;
  server_name  app.blued.cn;

  location / {
   proxy_pass http://127.0.0.1:8000/;
  }
}
 ```
 在conf.d下启动nginx: ```sudo nginx```
 (可以使用```ping app.blued.cn```测试是否可用)

 * nginx常用命令:

  启动: ```sudo nginx```
  停止: ```sudo -s stop```
  重启: ```sudo -s reload```

#### 1.4 开始写代码
页面中引入微信js文件
```
<script src="//res.wx.qq.com/open/js/jweixin-1.2.0.js"></script>
```

将1.2步获得的config填入wx.config
```javascript
wx.config({
   debug: true,
   appId: 'wxe527f9f4ded086bf',
   timestamp: 'efbm2f95lcx7c3j',
   nonceStr:'1495707198',
   signature: '0520367302ec40bb8b2eb71384730fea187a8558',
   jsApiList: ['onMenuShareAppMessage', 'onMenuShareTimeline', 'onMenuShareQQ', 'onMenuShareQZone'] // 必填，需要使用的JS接口列表，所有JS接口列表见官方文档附录2
})
```
调用微信相关API
```javascript
wx.ready(function () {
   wx.onMenuShareAppMessage({
   title: 'title', // 分享标题
   desc: 'content', // 分享描述
   link: '', // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
   imgUrl: '', // 分享图标
   type: '', // 分享类型,music、video或link，不填默认为link
   dataUrl: '', // 如果type是music或video，则要提供数据链接，默认为空
   success: function () {
    // 用户确认分享后执行的回调函数
  },
   cancel: function () {
    // 用户取消分享后执行的回调函数
  }
})
```
 * 坑

  * 严格按照微信官方文档的书写方式,驼峰与下划线并存
  * imgUrl地址为绝对路径

#### 1.5 使用微信web开发者工具调试
现在微信的开发工具已经可以开发网页了，并且集成了Chrome的DevTool进行调试。
console里出现```errMsg: "config:ok"```就大功告成啦。
___

### 2.线上开发
* 出于安全考虑，必须在服务器端实现签名的逻辑，getWxSignature.js代码如下:

```javascript
const req = require('bd-require')
const urllib = req('./node_modules/urllib')
const JsSHA = require('jssha')

const appInfo = {
  appID: '',
  appsecret: ''
}

// 接口每日调用有限制,需做缓存
const cacheInfo = {}

// 时间戳
const timeStamp = () => parseInt(new Date().getTime() / 1000)

// 获取签名
const sign = (ticket, noncestr, timestamp, url) => {
  const str = `jsapi_ticket=${ticket}&noncestr=${noncestr}&timestamp=${timestamp}&url=${url}`
  const shaObj = new JsSHA(str, 'TEXT')

  return shaObj.getHash('SHA-1', 'HEX')
}

// 获取jsapi_ticket
function * getTicket (accessToken) {
  let result = yield urllib.request(`https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=${accessToken}&type=jsapi`, {
    dataType: 'json',
    timeout: 2000
  })

  if (Number(result.status) !== 200) {
    return null
  }

  return result.data.ticket
}

// 获取token
module.exports = function * (url) {
  let ticket
  // 如果第一次启动，或者该ticket已经存在了超过7200秒，则重新获取ticket
  if (!cacheInfo.startTime || !cacheInfo.ticket || (timeStamp() - 7200) > cacheInfo.startTime) {
    console.log('通过wxAPI获取token', new Date().valueOf(), url)
    let result = yield urllib.request(`https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=${appInfo.appID}&secret=${appInfo.appsecret}`, {
      dataType: 'json',
      timeout: 2000
    })

    if (Number(result.status) !== 200) {
      return null
    }
    let ticketInfo = yield getTicket(result.data.access_token)

    if (!ticketInfo) {
      return null
    }

    cacheInfo.startTime = timeStamp()
    ticket = cacheInfo.ticket = ticketInfo
  } else {
    ticket = cacheInfo.ticket
  }

	// 随机字符串
  let nonceStr = Math.random().toString(36).substr(2, 15)
  let timestamp = timeStamp()
  let signature = sign(ticket, nonceStr, timestamp, url)
  return { ticket, nonceStr, timestamp, url, signature, appId: appInfo.appID }
}
```
* 服务端routes.js (服务端用的是KOA)

```javascript
const getWxSignature = require('getWxSignature.js')
module.exports = () => {
router.get('/', function* (){
try {
  // 将http修改为https & 删除#后的所有部分
	let url = (this.header.referer || this.href).replace(/^http:/, 'https:').replace(/#.*$/, '')
	this.body = yield getWxSignature(url)
} catch (e) {
	console.log(e)
}
```
* 前端代码 (react + yarn)
前端发送请求后拿到相关参数，放入config

```javascript
wx.config({
   debug: false,
   appId,
   timestamp,
   nonceStr,
   signature,
   jsApiList: ['onMenuShareAppMessage', 'onMenuShareTimeline', 'onMenuShareQQ', 'onMenuShareQZone']
})
```
* 调用微信相关API

```javascript
wx.ready(function () {
   wx.onMenuShareAppMessage({
   title: 'title', // 分享标题
   desc: 'content', // 分享描述
   link: '', // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
   imgUrl: '', // 分享图标
   type: '', // 分享类型,music、video或link，不填默认为link
   dataUrl: '', // 如果type是music或video，则要提供数据链接，默认为空
   success: function () {
    // 用户确认分享后执行的回调函数
  },
   cancel: function () {
    // 用户取消分享后执行的回调函数
  }
})
```
