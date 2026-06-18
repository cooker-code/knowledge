---
title: 当 AI 遇见 Obsidian，第二大脑开始管项目：Hermes Agent-kanban
author: DNOPC
date: DNOPCDNOPC
url: https://mp.weixin.qq.com/s?__biz=MzY4ODE5Mjc0MQ==&mid=2247483878&idx=2&sn=3f947145111f55417d07a8efc4c3c338&chksm=f21a34a19528df2db909ef1cdd62689851711818ec6f383ee721ea2e8dfe96e940c98c68ff10&mpshare=1&scene=24&srcid=0602FZmjvx8H3BLfiqvJrWNC&sharer_shareinfo=fe831739024de955b5b76f3abfccc3d2&sharer_shareinfo_first=fe831739024de955b5b76f3abfccc3d2#rd
---

hermes-kanban 通过 REST API 将 Hermes Agent 与 Obsidian 打通，实现 AI 对看板的直接操作。本文从技术架构、产品设计、使用场景三个维度深度解析这一工具的价值。当前星标214，fork13个。

---

## 背景

Obsidian 的核心价值是**本地优先**和**双向链接**。

过去几年，越来越多人把 Obsidian 当作「第二大脑」——所有笔记、文档、会议记录都存在这里。

但项目管理的部分，始终没有很好地融入这个生态。

要么用独立的工具（Notion、Linear、飞书），要么在 Obsidian 里手写任务列表。

前者造成信息孤岛。后者缺乏可视化。

**hermes-kanban 试图解决这个问题。**

---

## 技术架构

hermes-kanban 的架构可以分成四层：

### 第一层：Hermes Agent

用户通过自然语言与 Hermes 对话。Hermes 理解用户意图，生成对应的操作指令。

核心交互示例：

```
用户：「帮我把 Q3 产品上线拆成看板」  
Hermes：调用 skill，生成看板结构
```

### 第二层：hermes-kanban-bridge

一个跑在 Obsidian 里的插件，同时充当 **REST API Server**。

* • 监听本地端口（默认 27124）
* • 接收 Hermes 的 API 请求
* • 调用 Obsidian Vault API 操作文件
* • 返回操作结果

```
Hermes → HTTP POST → localhost:27124 → hermes-kanban-bridge
```

### 第三层：Obsidian Vault

所有看板数据以 Markdown 文件形式存在 Vault 里。

每张卡的本质是 Markdown 列表项，包含：

* • 卡片标题
* • 描述
* • 状态
* • 元数据

这意味着：

* • 数据完全归用户所有
* • 不依赖任何云服务
* • 可以用 Git 做版本管理

### 第四层：obsidian-kanban

社区插件，负责把 Markdown 渲染成可视化看板。

用户看到的是图形化界面，底层还是 Markdown 文件。

---

## 产品设计亮点

### 1. AI 与本地工具的解耦

很多 AI + 笔记工具的组合是深度绑定的——AI 必须用它的云服务。

hermes-kanban 的设计是**解耦的**：

* • Hermes 可以跑在任何地方（本地机器、服务器）
* • Obsidian 必须跑在本地
* • 两者通过 HTTP 通信

这意味着：

| 场景 | 支持情况 |
| --- | --- |
| Hermes + Obsidian 同机器 | ✅ 默认支持 |
| Hermes 云端 + Obsidian 本地 | ✅ 通过 IP 访问 |
| 跨设备协作 | ✅ 需要网络互通 |

### 2. 隐私优先

整个链路没有云端中转。

用户数据：

* • 存在 Obsidian Vault（本地硬盘）
* • Hermes 在本地处理（如果本地部署）
* • 两个服务之间的通信是局域网内直连

对于注重隐私的用户，这是重要考量。

### 3. Skill 驱动的可扩展性

Hermes 通过 skill 文件定义操作逻辑。

用户可以：

* • 修改现有 skill 定制行为
* • 添加新 skill 扩展功能
* • 完全掌控 AI 的操作边界

---

## 使用场景分析

### 场景一：目标拆解

**传统方式**：用户自己在脑子里想结构，然后手建任务列表

**hermes-kanban 方式**：

```
用户：「帮我把 Q3 产品上线拆成看板」  
→ Hermes 理解目标，拆解成可执行的任务  
→ 生成看板结构  
→ 用户确认和调整
```

**行业观察**：这个场景的价值在于「想法到结构」的转化。AI 帮你做这个转化，减少了从思维到可操作步骤的摩擦。

### 场景二：每日站会

**传统方式**：

* • 翻笔记回顾昨天
* • 想今天要干什么
* • 找阻塞项

**hermes-kanban 方式**：

```
用户：「帮我过一下站会」  
→ Hermes 读看板数据  
→ 自动生成：昨日完成 / 今日计划 / 阻塞项  
→ 用户确认
```

**行业观察**：站会的核心价值是信息同步。AI 帮你聚合看板数据，让人只需要做判断而不是做收集。

### 场景三：周复盘

**传统方式**：翻一周的文档和笔记，自己统计，自己写

**hermes-kanban 方式**：

```
用户：「做个周复盘」  
→ Hermes 统计本周看板数据  
→ 生成复盘报告
```

**行业观察**：复盘的前提是有数据。看板本身就是数据源。AI 让这个数据源直接变成可读的报告。

### 场景四：卡片操作

**传统方式**：打开看板，手动拖动

**hermes-kanban 方式**：

```
用户：「把那张卡移到进行中」  
→ Hermes 调用 API  
→ 卡片状态更新
```

**行业观察**：最小的一个场景，但也是最体现「AI 接管操作」本质的场景。说话就能动卡，这才是 AI 协作者的正确姿势。

---

## 局限与适用性

### 局限

1. 1. **Obsidian 必须是桌面版**：移动端不支持（依赖 Node.js http 模块）
2. 2. **需要配置网络访问**：Hermes 和 Obsidian 不同机器时需要额外配置
3. 3. **看板插件依赖**：需要 obsidian-kanban 插件才能渲染可视化界面
4. 4. **Skill 学习成本**：深度定制需要了解 Hermes skill 的编写方式

### 适用人群

* • Obsidian 重度用户（有大量笔记在 Obsidian 里）
* • Hermes 用户（已经在用 Hermes 做 AI 助手）
* • 本地优先用户（不希望数据上云）
* • 追求工作流统一的用户（希望笔记和项目管理在同一个工具里）

---

## 安装与配置

### 环境要求

* • Obsidian 1.4.0+（桌面版）
* • Node.js 18+
* • Hermes Agent（需要 skill 支持）

### 快速安装

```
# 1. 配置 Vault 路径并运行安装脚本  
bash hermes-kanban-install.sh  
  
# 2. 验证服务  
curl http://localhost:27124/health  
# {"ok":true,"status":"running","port":27124,"version":"1.0.0"}
```

### 跨机器配置

如果 Hermes 和 Obsidian 在不同机器：

1. 1. 获取 Obsidian 机器的 LAN IP 或 Tailscale IP
2. 2. 确保防火墙允许 27124 端口入站
3. 3. Hermes 端配置使用该 IP 而非 localhost

---

## 行业趋势

hermes-kanban 反映了一个趋势：**AI 协作工具正在向本地化、私有化演进**。

过去一年，很多 AI 工具是云优先的——数据上云，AI 处理上云，用户对数据的控制权很少。

hermes-kanban 的设计是：**AI 是接口，本地工具是底座**。

用户通过 AI 与 Obsidian 交互，但数据始终留在 Obsidian。

这个方向值得持续关注。

**GitHub**：https://github.com/GumbyEnder/hermes-kanban

---

*当 AI 开始管项目，你的第二大脑终于不只是存储——它开始工作了。*