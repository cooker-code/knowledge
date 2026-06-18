---
title: x weixin - 让命令行的所有 Agent 任务进度和自动化流程皆可发微信
author: oh my x
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzk4MTc0Ng==&mid=2247494146&idx=1&sn=8a5804cc75dff61bfc40aa688ffa22b9&chksm=c10100bd875d6a59ac90f5b014f50a9e35eb019ec0bfb2d2d4d49411bafaaf986b916abd8274&mpshare=1&scene=24&srcid=0331aSGYoWqR03Es77bop4Ex&sharer_shareinfo=99d78c8052720bea0d8c9e3743ed9e7e&sharer_shareinfo_first=99d78c8052720bea0d8c9e3743ed9e7e#rd
---

作为一个开发者，最消磨意志的时刻，不是修 Bug，而是等。

等 Agent 跑完任务，等一个几百 GB 的数据库迁移，等一个极其缓慢的编译打包。时不时就要看一眼，生怕进程在中途某个节点报错断掉，摸鱼都不方便。

`x weixin` 专为 Agent 和 自动化流程设计，让你的 Agent、自动化操作在任务结束时，把执行报告、生成的图表、甚至是异常截图，直接推送到你的手机微信。

### 典型场景

当你的 Claude Code、Codex 等 Agent 跑完任务时，直接发个微信通知自己，再也不用盯着屏幕。

```
# 任务执行成功后通知  
x weixin send text "✅ 任务已完成"
```

集成到 CI/CD 流水线，代码部署、测试结果实时推送，团队协作更高效。

```
# 部署成功后通知  
x weixin send text "🚀 项目已部署成功，版本：v1.2.3"
```

### 初始化配置

在使用 `x weixin` 之前，需要先完成机器人账号的授权。  
运行以下命令，使用微信扫描弹出的二维码即可。

```
1x weixin login
```

登录成功后，凭证会自动保存到本地，后续使用无需重复登录。

### 发送多媒体文件

除了文本消息外，`x weixin` 还支持发送图片、文件、视频。

```
x weixin send image ./assets/logo.png

  

x weixin send file ./reports/weekly_summary.pdf
```

### 消息监听：实现自动化交互

`x weixin` 还实现了消息监听功能。通过长轮询方式，你可以实时获取发送给微信机器人的消息，从而实现各种自动化交互。

```
1x weixin listen poll
```

当你启动消息监听后，机器人会持续监听收到的消息。你可以根据接收到的消息内容，编写脚本实现自动回复、触发其他操作等功能。

连接 Claude Code

### 更多探索

`x weixin` 还提供了其他实用功能：

* 配置管理 (`--cfg`)  
  管理机器人的配置信息，包括 token、默认参数等
* Profile 管理 (`--cur`)  
  为不同的使用场景创建独立的配置Profile
* 快捷发送 (`send`)  
  `x weixin send` 是 `x weixin bot send` 的简写形式

官网：

https://weixin.x-cmd.com/

---

---

🎉 **欢迎****加入我们的用户交流群** 🎉

在这里，你可以：  
📢 第一时间了解最新动态和活动信息  
🤝 结识同行，共享实战经验  
📚 探讨行业趋势，拓展人脉圈子

📌 **如何加入？**📷 扫描添加小助手，完成验证即可入群！

期待你的加入，一起交流成长！🚀