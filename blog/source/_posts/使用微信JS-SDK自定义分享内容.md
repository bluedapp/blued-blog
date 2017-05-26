---
uuid: 1fabda00-40f9-11e7-b46f-5fe776c18c7e
title: 使用微信JS-SDK自定义分享内容
date: 2017-05-25 11:20:38
tags:
---
## 如何调用微信SDK
###1.本地开发测试
#### 1.1使用测试账号获取相关数据

* 使用个人微信登录公众平台接口测试号申请: [测试号申请](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)

* 登录后拿到appId和appsecret
![appID](media/14951079874043/appID.jpeg)￼

* 配置安全域名(以公司blued举栗子)
![安全域名](media/14951079874043/安全域名.jpeg)￼
<font color=#F08080>(注意:域名不能配置IP地址)</font>
####1.2 获取接口签名
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
![signature](media/14951079874043/signature.jpeg)￼
noncestr:随机字符串，由开发者随机生成
timestamp: 由开发者生成的当前时间戳 ```parseInt(new Date().getTime() / 1000)```
####1.3 设置反向代理
* 更改host, 在最下方加入: 127.0.0.1 app.blued.cn
``` javascript
sudo vi /etc/hosts
```
* nginx反向代理
 安装nginx: brew install nginx
 查看安装位置: brew list nginx
 打开: cd /usr/local/Cellar/nginx/

 
