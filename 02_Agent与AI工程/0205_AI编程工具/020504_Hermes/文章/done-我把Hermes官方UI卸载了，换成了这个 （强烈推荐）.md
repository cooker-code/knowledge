> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: 我把Hermes官方UI卸载了，换成了这个 （强烈推荐）
author: 未来基因派
date:
url: https://mp.weixin.qq.com/s?__biz=MzI2NjY5MjQzNQ==&mid=2247484066&idx=1&sn=ce0a80a4e845c299dfcd7657277dfcab&chksm=ebc9c1fa365a27e6b32cef19dde49920d680316b27d324912a4bea8cbc7705464ed8883206c3&mpshare=1&scene=24&srcid=0419pemH914KPw1jvXi9RcEa&sharer_shareinfo=fd901b4730035f123fc5622e136a8f6a&sharer_shareinfo_first=fd901b4730035f123fc5622e136a8f6a#rd
---

前几天 Hermes 官方出了 Web UI，我第一时间也体验了。虽然是官方出品，但说实话，**真的欣赏不来老外的审美**——页面丑字也丑，而且最关键的是**没法对话**，只能看看配置，然后就弃坑了。

后来经朋友推荐，看到了这款**国内大佬开发的 Hermes Web UI**。体验完发现页面效果和功能真的很棒！

先放一张图看看效果

**安装教程放前面**，两步就能跑起来：

---

## 两步安装

**方式一：npm 安装（推荐）**

```
npm install -g hermes-web-ui

hermes-web-ui start
```

然后浏览器打开：

```
http://127.0.0.1:8648/
```

**方式二：没装 Node.js 的一键脚本（支持 Debian/Ubuntu/macOS）**

```
bash <(curl -fsSL https://cdn.jsdelivr.net/gh/EKKOLearnAI/hermes-web-ui@main/scripts/setup.sh)
```

**WSL 用户：**

```
bash <(curl -fsSL https://cdn.jsdelivr.net/gh/EKKOLearnAI/hermes-web-ui@main/scripts/setup.sh) 

hermes-web-ui start
```

装完就能用，**不需要改配置文件，不需要记命令**，开箱即用。

---

## 为什么我会选这个 Web UI

### 1. 真的能对话，而且界面能打

官方的 Web UI 只能看不能聊，这个不一样——**支持完整对话**，而且 UI 是国人审美，简洁舒服。

左边菜单清清楚楚：对话、任务、模型、频道、技能、记忆、日志、用量，**一个页面管所有**，不用在终端和浏览器之间来回切。

### 2. 定时任务可视化

以前设置定时任务要手写 Cron 表达式，现在**鼠标点一点就能创建**。支持设置任务名称、调度时间、提示词、投递目标，还能限定重复次数。

再也不用背"0 0 \* \* \*"这种天书了，**有手就能配**。

### 3. 模型管理超方便

支持手动添加 Provider，Anthropic、Google AI、DeepSeek、智谱、Kimi、xAI、MiniMax 这些**主流厂商都内置了**，下拉框直接选。如果是自定义 API，也支持手动输入 Base URL。

> **注意**：如果是要加 OAuth 验证的，还是得回 Hermes CLI 操作，这个 Web UI 主要管基础配置。

### 4. 八大渠道一个页面搞定

Telegram、Discord、Slack、WhatsApp、Matrix、飞书、微信、企业微信，**全部可视化配置**。

最爽的是**微信支持扫码登录**！官方的 Web UI 都不支持这个，这里直接浏览器扫码，Token 自动填好，**不用抓接口不用改文件**。

每个渠道都能独立开关、设置是否需要@提及、配置自动线程，**运营小姐姐都能自己搞定**，不用麻烦开发改配置。

### 5. 技能管理一目了然

能看到所有 Hermes 自带的和你自己创建的 skills，支持搜索、查看详情、开关控制。

每个技能的说明、命令、依赖都展示得清清楚楚，**终于知道自己装了啥**。

### 6. 记忆直接改

SOUL.md、MEMORY.md、USER.md 这三个核心记忆文件，**直接网页上就能编辑**，不用 SSH 进去找文件位置。

写笔记、改用户画像、调灵魂设定，**指尖轻点就能完成**。

### 7. 用量统计直观

Token 消耗、会话数、预估费用、缓存命中率，**一屏看完**。还有模型分布和 30 天趋势图，**月底对账不再抓瞎**。

之前都是等账单来了才知道花了多少，现在**每天用了多少心里都有数**。

### 8. 多 Agent 切换

左下角支持多 Agent 切换，default、coder、planner、writer 这些角色**点一下就能换**，不用重启服务改配置。

---

## 总结

它不是那种只能当"聊天壳子"的 UI，而是**把对话、配置、任务、用量、技能、记忆全部整合**的完整驾驶舱。微信扫码、多 Agent 切换、定时任务可视化，这些**官方都不支持的功能**，这里全有。

**项目地址：**

https://github.com/EKKOLearnAI/hermes-web-ui

如果你觉得这篇文章对你有帮助，欢迎**点赞**、**收藏**、**转发**给身边的朋友一起学习！