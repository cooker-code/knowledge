---
title: Mac 上跑容器，原来可以这么顺滑！
author: 开源先锋
date: 开源君开源君
url: https://mp.weixin.qq.com/s?__biz=MzkwNzU4NTMyMA==&mid=2247505431&idx=1&sn=6c7b9054acb75f94cbb8d5272290d196&chksm=c1b542903a66f0c0d441c091590d70ae25b715e8a1b4e8cbfa84153bc713ed09b00208baa8f1&mpshare=1&scene=24&srcid=0522nJhP6GViJfm7OdEbDbpP&sharer_shareinfo=dd6da94c06d319f9b7e1f5434417961a&sharer_shareinfo_first=dd6da94c06d319f9b7e1f5434417961a#rd
---

\* 戳上方蓝字“开源先锋”关注我

大家好，我是开源君！

各位在 Mac 上搞开发的铁子门，肯定在本地环境用过 Docker Desktop。

说实话，确实好用，但它对资源的占用……懂的都懂。

今天给大家分享一个完美替代 Docker Desktop的神器 - `OrbStack`，不仅快，还特别轻，可以说是macOS 上的“超级 WSL”。

## 项目简介

`OrbStack` 是一个专为 macOS 打造的极速、轻量且简单的应用。它不仅能跑 Docker 容器，还能跑 Kubernetes 和完整的 Linux 机器。

它直接用 Swift 写了原生应用，而不是像 Docker Desktop 那样套个 Electron 壳子，所以占用资源极低。

最爽的是，它是 Docker Desktop 的“即插即用”替代品，你原来的 `docker` 命令、`docker-compose` 文件，一行都不用改，直接无缝迁移。

## 功能特性

OrbStack 主打**快、轻、简、强**四个字——启动快、资源省、上手简单、能力全面。

但这些都是基操，下面才是它真正让我觉得「哇塞」的地方。

### 01 容器体验堪比原生

构建镜像速度飞快，网络和文件共享稳定不卡顿，绑定挂载、端口转发不用额外配置，点开就能用。

调试容器也方便，直接就能访问卷文件，不用绕弯子；

就算是x86架构的容器，靠Rosetta模拟也能流畅运行，Docker Compose等常用工具也都自带，不用额外安装。

### 02 Linux虚拟机免折腾

想用啥发行版就用啥，不用手动配置复杂环境。

最贴心的是，能和VS Code（或你常用的编辑器）无缝衔接，SSH代理转发也支持，远程开发和本地操作没区别。

要是用的是Apple Silicon芯片的Mac，还能靠Rosetta运行Intel架构的Linux机器。

### 03 终端党狂喜的命令行集成

在终端里就能轻松执行各种命令，还能在Mac和Linux之间快速复制文件，不用借助第三方工具。

更方便的是，能从Linux系统里发送通知、打开文件和链接，和Mac系统联动超顺畅，完全不割裂。

### 04 极致轻量不添乱

这应该也是Mac用户最爱的一点！

在Apple Silicon上，后台CPU占用还不到0.1%，几乎感觉不到它在运行；

初始磁盘占用也才10MB左右，不会臃肿占空间。

## 快速安装、使用

`OrbStack` 安装方式非常简单：

**方式一：Homebrew（推荐）**直接在终端敲：

```
brew install orbstack
```

**方式二：官网下载**去 https://orbstack.dev/ 下载 `.dmg` 文件，拖进 Applications 就行。

### 基础使用

* 切换Docker上下文：安装后一键接管Docker，执行`docker context use orbstack`即可切换，想切回原环境可执行`docker context use desktop-linux`。
* 常用命令：

+ `orb list`：查看容器和虚拟机列表
+ `orb stats`：查看资源占用状态
+ `orb restart docker`：重启Docker引擎
+ `ssh 用户名@orb`：免密登录Linux虚拟机

* 图形化操作：菜单栏打开OrbStack面板，可视化管理容器、镜像、虚拟机，支持一键启动、停止、删除，还能直接访问容器文件。

## 小结

`OrbStack`是目前 macOS 上体验非常不错的的容器与虚拟机管理工具之一。它用原生的 macOS 技术，换来了更快的速度、更低的资源占用，同时还保留了你习惯的所有功能，切换成本几乎为零

如果你正在为 Docker Desktop 的卡顿、费电、占内存而烦恼，OrbStack 绝对值得你花 10 分钟试一试。

更多细节功能，感兴趣的可以到项目地址查看：

```
https://github.com/orbstack/orbstack
```

推荐阅读：

[硬刚Cloudflare、reCAPTCHA，源码级伪装](https://mp.weixin.qq.com/s?__biz=MzkwNzU4NTMyMA==&mid=2247505251&idx=1&sn=b3284fb75e5d4c64e570c35dc9268811&scene=21#wechat_redirect)

[这是我见过最全的免费 LLM API 指南，正规合法、开箱即用！](https://mp.weixin.qq.com/s?__biz=MzkwNzU4NTMyMA==&mid=2247505298&idx=1&sn=e9b087eba378368b1d36c3eb34259bf8&scene=21#wechat_redirect)

[代码“裸奔”还不知道？这款27K star神器建议都装一下](https://mp.weixin.qq.com/s?__biz=MzkwNzU4NTMyMA==&mid=2247505325&idx=1&sn=1cf49f742c53ed6a3f926db4d0a8ac88&scene=21#wechat_redirect)

[90+工具，还带工作流自动化，这款开源PDF新秀火了!](https://mp.weixin.qq.com/s?__biz=MzkwNzU4NTMyMA==&mid=2247505369&idx=1&sn=940af20ae4c43584a788115d64812d36&scene=21#wechat_redirect)

[仅 11MB 大小，Win转Mac必装，爽了~](https://mp.weixin.qq.com/s?__biz=MzkwNzU4NTMyMA==&mid=2247505405&idx=1&sn=38e397d4f9b7d719515a7624fd5fdcb3&scene=21#wechat_redirect)