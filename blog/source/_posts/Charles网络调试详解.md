---
uuid: 9c13ea50-ec15-11e9-9429-cbf76a73d4b7
author: menglingyu
email: 1770413875@qq.com
github: https://github.com/menglingyu
avatar: https://avatars3.githubusercontent.com/u/19648678?v=4
title: Charles网络调试详解
date: 2019-10-11 18:55:15
tags:
---


### Charles网络调试详解
Charles是一个HTTP代理/ HTTP监视器/反向代理，使开发人员能够查看其机器和Internet之间的所有HTTP和SSL / HTTPS流量。这包括请求，响应和HTTP标头（包含cookie和缓存信息）。

#### 为什么要用它
chrome network 已经足够好用了，为什么要用它，简单一句话，chrome 只是让你看到请求的过程，而不能调试这个过程。而charles是可以对网络这一层进行比如断点、限速、修改等调试功能，并且charles不局限于浏览器，可以代理任何平台的请求、比如后端、postman等。

#### 主要功能
1. 带宽限制
2. 请求、响应断点，请求、响应重写
3. 自动配置浏览器和系统代理设置
4. **无缓存工具:** 它可以防止客户端应用程序（如Web浏览器）缓存任何资源
5. **映射、本地远程工具:** 比如你可以把线上资源映射到本地某个文件夹下，这样可以方便的处理一些特殊情况下的bug和线上调试
6. 阻止Cookies
7. DNS欺骗工具
8. 支持手机等其他终端也网络调试
9. 支持SOCKS

下载地址：[https://www.charlesproxy.com/download/](https://www.charlesproxy.com/download/)

#### 破解安装
1. [点我下载破解文件](https://www.zzzmode.com/mytools/charles/)
2. 替换charles.jar文件
macOS: /Applications/Charles.app/Contents/Java/charles.jar
Windows: C:\Program Files\Charles\lib\charles.jar

#### 常规使用
##### 抓包浏览器
浏览器如 chrome 的http请求默认走的是系统代理，打开 Proxy -> 勾选 macOS proxy 即可抓包
#### 抓https
默认不支持抓https，看到的会是unkonw,需要提供证书，步骤如下。

1. Proxy->SSL Proxy Settings，弹出一个ssl代理设置界面
2. 勾选 Enable SSL Proxying
3. 添加你想要的设置代理的域名,端口默认443,设置为 \*,抓所有域名的包
4. Help->SSL Proxying ->Install Charles Root Certificate ... ，根据要调试的平台选择对应的安装方式
##### 电脑端
选择 Install Charles Root Certificate，然后会弹出"添加证书提示"，选择系统，自动安装后，打开"钥匙串访问"，找到对应的证书选择始终信任，重启Charles即可
<img src="/img/mly/2bd94eb935bbef95214c1c71a19598f2.png" alt="appID" width="50%">

##### 移动端
首先确保连接同一个wifi，在同一个网段。选择 Install Charles Root Certificate on a Mobile Device or Remote Browser...，然后设置设备代理，无限局域网 -> 配置代理 -> 手动 -> 配置上提示的ip地址和端口，如果成功了，电脑会提示是否允许访问，同意后，用手机打开提示的 chls.pro/ssl会自动下载证书，然后：设置->通用->描述文件。启用描述文件，然后关于本机->证书信任设置、打开根据根证书启用完全信任，之后即可抓到包。
<img src="/img/mly/db0d1c7ad3f295252028c4e16da3f1dc.png" alt="appID" width="50%">

#### 网速模拟
用于模拟不同情况下的网速

打开Proxy -> Start/Stop Throttling，然后选择Throttle Settings对模拟网速进行配置

<img src="/img/mly/0c5dd7f5773ece6a1a75dd0c6f9138d7.png" alt="appID" width="50%">

#### 断点设置
抓包届的debugger，常用两种方式

1. 打开Proxy -> Enable/Disable Breakpoints，开启断点调试，然后通过 Breakpoint Settings 配置哪些请求需要断点
2. 直接在请求记录上右键 勾选 Breakpoints, 之后请求就在请求的响应的过程停住，并且通过 edit request/responent，进行编辑

<img src="/img/mly/c6541008778ff9cc0012ead04e770a21.png" alt="appID" width="50%">

##### tools

tools也是常用的一系列功能
Tools 菜单包含以下功能：

1. No Caching Settings：禁用缓存设置。
2. Block Cookies Settings：禁用 Cookie设置。
3. Map Remote Settings：远程映射设置。
4. Map Local Settings：本地映射设置。
5. Rewrite Settings：重写设置。
6. Black List Settings：黑名单设置。
7. White List Settings：白名单设置。
8. DNS Spoofing Settings：DNS 欺骗设置。
9. Mirror Settings：镜像设置。
10. Auto Save Settings：自动保存设置。
11. Client Process Settings：客户端进程设置。
12. Compose：编辑修改。
13. Repeat：重复发包。
14. Repeat Advanced：高级重复发包。
15. Validate：验证。
16. Publish Gist：发布要点。
17. Import/Export Settings：导入/导出设置。
18. Profiles：配置文件。
19. Publish Gist Settings：发布要点设置。

#### 常见问题
##### 代理冲突
当想抓电脑包时需要勾选 Proxy -> MacOs Proxy，此时会设置系统代理，而其他代理冲突比如科学上网软件shadowsocks，也是直接修改系统代理，相当于，
```
export http_proxy=http://127.0.0.1:****;export https_proxy=http://127.0.0.1:****;
```

<img src="/img/mly/54e7c04b908c80bd666bca3bae4e140e.png" alt="appID" width="50%">


解决办法：配置外部代理，Proxy -> External Proxy Settings -> Use external proxy servers

<img src="/img/mly/243a9a207476f410ab83b33290cd1924.png" alt="appID" width="50%">

##### 代理不到项目怎么办？
解决问题的通用逻辑是，在想要抓包的平台上配置代理到charles的代理服务器上(默认为:127.0.0.1:8888)
比如node中 通常请求库都会有一个proxy的api配置项、chrome默认不走代理，需要设置成charles代理，或者勾选macOS proxy 的情况下设置为系统代理。
以及 postman都可以设置代理。

##### 开启charles后上不去网
1. 是不是是开启了白名单，但网站没有添加到白名单里，解决方式：关闭White List；Tools-->White List
2. 是不是开启了外部代理，但此时外部代理没有启动，解决方式：Proxy -> External Proxy Settings -> Use external proxy servers，关掉选项


### 更多高级用法

官方文档：https://charlesproxy.com/documentation/configuration/browser-and-system-configuration/