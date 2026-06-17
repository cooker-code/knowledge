---
title: Muxy：Mac 上最容易上手、真正以项目为核心的终端工具
author: 如此才是
date: 小K小K
url: https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247485624&idx=1&sn=d71439177302c427a22dc8f5f1780139&chksm=f5482aca430bbe860923e6104cf523697a04f2b1b775310e076ee3b2dc23a19f74d49117d5a6&mpshare=1&scene=24&srcid=0601DmZnoqThMOdEGnr6MMNQ&sharer_shareinfo=ac90058598901e8662a50baae268b5d1&sharer_shareinfo_first=ac90058598901e8662a50baae268b5d1#rd
---

很多 Mac 开发者每天都在和「项目切换地狱」作斗争：

打开 iTerm2 + tmux，切项目就要重新开窗口、重新 cd、重新起服务……关掉重启后，上一秒的工作状态全没了，只能靠记忆一点点恢复。

**Muxy 就是专门解决这个痛点的。**

它不是单纯的终端模拟器，而是一个**以「项目（Project）」为核心的工作流多路复用器**。所有终端、分屏、编辑器、Git 状态、文件树、甚至窗口布局，都按项目永久保存。重启电脑、第二天打开，一键就能回到和昨天一模一样的工作空间。

用过之后，你会发现：**原来终端也可以这么丝滑和省心**。

### 一、Muxy 真正解决的日常痛点

●再也不用为「这个项目我当时开着几个终端、在哪个路径」而头疼

●切换项目就像切换文件夹一样简单，所有状态自动记住

●想同时看代码、跑终端、看 Git diff、预览 Markdown？全在一个窗口搞定

●出门在外还能用手机远程控制 Mac 上的终端

●不用再额外装一堆插件，Git、文件浏览、编辑器、AI 费用追踪全内置

**Muxy 把「终端 + 轻量 IDE + 项目管理器」三合一了，而且用起来比单独工具还简单。**

### 二、5 分钟上手 Muxy

**安装（推荐方式）：**

```
●●●bash

brew tap muxy-app/tap  
brew install --cask muxy
```

安装完直接打开就行，不需要任何复杂配置。

**第一次使用流程：**

1.打开 Muxy

2.点击左上角「+ Add Project」，选择你的项目文件夹

3.给项目设置一个好看的图标和颜色（强烈建议，视觉区分超爽）

4.搞定！现在这个项目的所有工作空间就永久保存了

以后再打开项目，直接在左侧项目列表点一下，就能瞬间恢复上次的所有标签、分屏和路径。

### 三、日常最常用、最实用的功能

**1. 垂直标签栏（超级推荐）**

●左侧是垂直标签，支持拖拽排序、固定常用标签、中键关闭

●比传统顶部标签好用太多，项目一大就特别清晰

**2. 自由分屏（水平/垂直都行）**

●拖拽标签到窗口边缘就能快速创建分屏

●支持键盘快捷键操作，几乎可以全程不用鼠标

**3. 内置轻量编辑器 + Markdown 实时预览**

●直接在 Muxy 里打开文件编辑，语法高亮

●Markdown 文件可以直接内联预览，所见即所得

●不想用大 IDE 时，这就够了

**4. Git 集成（重度用户福音）**

●侧边栏直接显示仓库状态、未提交文件

●支持 unified / split diff 查看

●一键提交、切换分支、管理 Git worktree

●甚至能通过 gh CLI 创建和管理 PR

**5. 文件树浏览器**

●带 Gitignore 过滤，干净清晰

●支持新建、删除、重命名、拖拽等常用操作

**6. 手机远程控制（真的太香了）**

下载官方 iOS / Android 伴侣 App，配对后就能用手机远程操作 Mac 上的终端，适合出门、躺床上、开会时临时处理紧急问题。

**7. AI 使用追踪面板**

实时显示你当前用 Claude、Cursor、Copilot、Kimi 等各种 AI 的 token 消耗和费用，再也不怕月底突然收到巨额账单。

### 四、几个让效率直接起飞的小技巧

●按 `Cmd + K` 打开命令面板，几乎所有操作都能在这里快速找到

●每个项目可以自定义专属布局（支持 YAML 声明式定义，进阶用户可用）

●富输入面板：写超长命令或 AI Prompt 时特别好用，支持多行、保存草稿、发给所有分屏

●拖拽文件路径到终端 = 自动 cd + 粘贴路径

Muxy 的核心理念就是**「让工具适应你的工作流，而不是你去适应工具」**。

足够轻量（内存占用低、启动快），又足够强大（把你需要的功能都合理集成在一起），最重要的是——**真的容易上手**。

如果你：

●每天要在多个项目之间频繁切换

●讨厌每次重启都要重新布置工作空间

●希望终端、代码编辑、Git 操作更一体化

●偶尔需要手机远程操作 Mac

那我强烈建议你现在就去试试 Muxy。

**安装命令再贴一次：**

```
●●●bash

brew tap muxy-app/tap  
brew install --cask muxy
```

**—— 如此才是**

**把复杂的技术，讲成你真正能用上的生产力**

**[零基础也能玩转卫星！开源Ground Station + SDR 打造个人地面站全攻略](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484408&idx=1&sn=fa96368ff3647cd53bad3ee9391103ee&scene=21#wechat_redirect)**

**[OpenClaw & Hermes刷屏后，GitHub  Mercury Agent如何打动用户？ 灵魂驱动+权限铁闸+24/7永动 vs 两大竞品](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484903&idx=1&sn=8ea193b342d5f61ed5ed30de9ebb32b9&scene=21#wechat_redirect)**

**[苹果M系列芯片的福音！无需H100、无需云GPU，本地MacBook就能微调Gemma 4多模态模型](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484529&idx=1&sn=63e2f7f4ac65540fd05ef9d05f0d28a8&scene=21#wechat_redirect)**

**[开源Minecraft终极杀手！12.7K星GitHub神器Luanti（原Minetest）完整中文攻略：零基础安装、2800+模组随便玩、服务器+源码编译](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484515&idx=1&sn=f7cac21fab871c06cee81d780a1e37af&scene=21#wechat_redirect)**

**[AI 直接操控 Unity/Godot/Unreal 编辑器！用 OpenClaw + TomLeeLive 插件，聊天就能把你的游戏梦想变成现实](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484263&idx=1&sn=49474c84d4c0c6a1dd7925d821680aca&scene=21#wechat_redirect)**

[开源项目Paseo，AI编码代理跨设备统一指挥中心：统管Claude Code、Codex、OpenCode（以及Copilot、Pi等）](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484927&idx=1&sn=b4d7d4aed5a5ad7263bff54b50c395a5&scene=21#wechat_redirect)

[老婆/女朋友每天早上纠结45分钟穿什么？GitHub 开源AI衣柜神器 Wardrowbe 彻底解放！完整自托管安装+使用教程](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484594&idx=1&sn=2a9832be4fd2b3d423f9c62fbae5b0a3&scene=21#wechat_redirect)

[Notebook LM平替，开源Open Notebook：隐私零泄露、18+AI模型随意切、1-4人定制播客秒生成](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247484913&idx=1&sn=a3307c1fb6b981881b22ca1c1ca407e2&scene=21#wechat_redirect)

[30MB Rust无头浏览器Obscura：击败Chrome、V8真实JS+CDP全兼容，AI Agent与爬虫的隐形核武器](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247485078&idx=1&sn=84152e9774e0eab3d16839db3a7657de&scene=21#wechat_redirect)

[Rust重写的jcode：性能碾压Cursor Claude Code 139倍的下一代Coding Agent Harness，人类级内存图谱+多会话Swarm](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247485066&idx=1&sn=2a563ec1e199af1807b6541f91d0842b&scene=21#wechat_redirect)

[Warp开源震撼发布！5年Rust GPU终端+Oz Agentic开发环境完整拆解：功能全览、源码编译教程、核心架构深度解析](https://mp.weixin.qq.com/s?__biz=MzY5MTAxODQ1MQ==&mid=2247485052&idx=1&sn=f612497afd348acd327221233af635c2&scene=21#wechat_redirect)