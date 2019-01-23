---
uuid: 6fc72e80-1eb1-11e9-b08d-ed9794ea6bdb
author: l19861225q
email: liuqian@blued.com
github: https://github.com/l19861225q
avatar: https://avatars1.githubusercontent.com/u/4251365?v=4
banner: /img/liuqian/how-to-run-qconf-on-osx/qconf.png
title: 如何在 OSX 上运行 QConf
date: 2019-01-23 09:51:43
tags: qconf
---

最近，由于项目需要，需要在 Mac 上安装 <a href="https://github.com/Qihoo360/QConf" target="_blank">QConf</a>，其中的过程真是山路十八弯...

废话不多说，直接上菜！

<!-- more -->

### 1.安装 QConf
- `git clone https://github.com/Qihoo360/QConf.git`
- `cd QConf && mkdir build && cd build && cmake ..`
  <i>这里 `cmake ..` 第一次可能会报错，直接无视，再 `cmake ..` 一遍就可以了。</i>
- `make`
- `sudo make install`
  <i>安装完成以后，默认应该是在 `/usr/local/qconf` 这个路径下面</i>

### 2.<a href="https://github.com/Qihoo360/QConf/wiki/FAQ" target="_blank">问题处理</a>
#### 使用 qconf 需要调整共享内存限制
**通过 sysctl -a | grep shm 查看当前的共享内存上限的大小，如果不足2G，则进行如下操作：**
> 修改共享内存上限，使当前正在运行的系统生效 `需要 sudo -s 下执行`
> Mac 执行：
```bash
  sysctl kern.sysv.shmmax=2048000000
  sysctl kern.sysv.shmall=1073741824
```
> 修改共享内存上限，使机器重启时生效，需要在 /etc/sysctl.conf 添加：
```bash
  kern.sysv.shmmax=2048000000
  kern.sysv.shmall=1073741824
```

### 3.安装 flock
> `/usr/local/qconf/bin/agent-cmd.sh:355` 用到了 `flock` 命令，但 OSX 并不支持，需要手动安装

```bash
brew tap discoteq/discoteq
brew install flock
```

装好以后，修改（需要 sudo） `/usr/local/qconf/bin/agent-cmd.sh:355` 去掉 `-e` 参数，否则会报错。

### 4.启动 QConf
- `cd /usr/local/qconf/bin && sh agent-cmd.sh start`

### 5.安装 QConf 的 Node 驱动 <a href="https://www.npmjs.com/package/node-qconf" target="_blank">node-qconf</a>
当然也有 C++ PHP 等 <a href="https://github.com/Qihoo360/QConf/tree/master/driver" target="_blank">其他驱动</a> 咯

- 设置环境变量 .bashrc/.zshrc/...
  `export QCONF_INSTALL=/usr/local/qconf`
- `npm install node-qconf`

### 6.愉快的玩耍吧

```js
var qconf = require('node-qconf');
  qconf.getAllHost();
  ...
```
<br />
## A Big Hole (一个大坑！)

##### 运行时，如果提示找不到 `libqconf.dylib` 这货怎么办？

多半是因为 `libqconf.dylib` 这货的路径设置问题。
其实这货就在 `/usr/local/qconf/lib` 下，所以我也尝试过设置环境变量

```bash
  export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:$QCONF_INSTALL/lib
```

确实，这样设置以后在 Node 命令行模式下加载 `node-qconf` 已经可以了

```bash
node
> require('node-qconf')
> // No error, so happy ~
```

但是！如果用了类似 <a href="http://nodemon.io/" target="_blank">nodemon</a> 来启动项目，还是会提示找不到这货的诡异错误。原因呢...

Presumably, you are running El Capitan (OS X 10.11). It's a side effect of System Integrity Protection. From the <a href="https://developer.apple.com/library/prerelease/mac/documentation/Security/Conceptual/System_Integrity_Protection_Guide/RuntimeProtections/RuntimeProtections.html" target="_blank">System Integrity Protection Guide: Runtime Protections<a/> article:
<br />
> When a process is started, the kernel checks to see whether the main executable is protected on disk or is signed with an special system entitlement. If either is true, then a flag is set to denote that it is protected against modification. …
<br /><br />
… Any dynamic linker (dyld) environment variables, such as DYLD_LIBRARY_PATH, are purged when launching protected processes.

<i>参考</i>
- [Why isn't DYLD_LIBRARY_PATH being propagated here?](http://stackoverflow.com/questions/35568122/why-isnt-dyld-library-path-being-propagated-here)

额，好吧，我们设置的 DYLD_* 这些环境变量都在启动一个受保护的进程时被净化掉了！

后来仔细发现，报错的文件是

```bash
./node_modules/node-qconf/build/Release/qconf.node
```

这货还是个二进制文件！>_<
那么有什么办法可以查看它在启动时引用的 Shared library paths？

##### otool

```bash
cd ./node_modules/node-qconf/build/Release/
otool -L qconf.node

// qconf.node:
//   libqconf.dylib (compatibility version 0.0.0, current version 0.0.0)
//   ...
```

看来，的确是路径问题，它从当前目录引用了 `libqconf.dylib`
好吧，那么有什么办法可以来修改这个路径？

##### install_name_tool

```bash
install_name_tool -change libqconf.dylib /usr/local/qconf/lib/libqconf.dylib ./qconf.node
```
这时再执行 `otool -L qconf.node` 会发现已经被设置成为正确的路径
```bash
qconf.node:
  /usr/local/qconf/lib/libqconf.dylib (compatibility version 0.0.0, current version 0.0.0)
```
大功告成！再也不会看到烦人的找不到 .dylib 的错误了。但这个方法只能算是一种 hack，如果重新安装了 `node-qconf`，还是需要重新 hack 的。

<i>参考</i>
- [两个 Xcode 的实用工具： otool 和 install_name_tool](http://www.jianshu.com/p/193ba07dadcf)
-  [Where do I set DYLD_LIBRARY_PATH on Mac OS X, and is it a good idea?](http://superuser.com/questions/282450/where-do-i-set-dyld-library-path-on-mac-os-x-and-is-it-a-good-idea)
