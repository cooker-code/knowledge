---
title: 3 步搞定！让 Claude Code 显示进度和状态
author: TecNote技术思维
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU1NTU3MzE2Mw==&mid=2247483904&idx=1&sn=9d046bb409d4ec81ef21bc93b282af1d&chksm=faaf75ca0d14248b2b3ea8d8646ae6363a1629e4d8fbef33f2874c1bebcd55edbf2424b6f899&mpshare=1&scene=24&srcid=0417hE4u4cIhWudgRn4bFP1E&sharer_shareinfo=df8ccb28af22701a895786d1f5d83058&sharer_shareinfo_first=df8ccb28af22701a895786d1f5d83058#rd
---

在使用 Claude Code 时，你是否曾好奇：上下文用了多少？有哪些工具在运行？子 Agent 在做什么？今天介绍的神器将让一切一目了然。

## 一、什么是 Claude HUD？

**Claude HUD** 是一个为 Claude Code 打造的实时状态栏插件。它就像汽车的仪表盘，让你时刻掌握 AI 编程助手的"运行状态"。

### 核心功能一览

| 监控内容 | 价值所在 |
| --- | --- |
| **项目路径** | 清楚知道当前在哪个项目工作（可配置 1-3 级目录） |
| **上下文健康度** | 精确掌握 token 使用情况，避免上下文溢出 |
| **工具活动** | 实时查看 Claude 正在读写、搜索哪些文件 |
| **Agent 追踪** | 看到哪些子 Agent 正在运行，它们在做什么 |
| **Todo 进度** | 跟踪任务完成情况，了解整体进度 |

## 二、它能显示什么？

### 默认显示

**第一行**：模型名称、计划名称（或 Bedrock）、项目路径、Git 分支

**第二行**：

* 上下文进度条（绿色→黄色→红色 颜色分级）
* API 使用配额（Pro/Max/Team 用户可见）

展示效果

### 三、三步快速安装

### 第一步：添加插件市场

在 Claude Code 中运行：

```
/plugin marketplace add jarrodwatts/claude-hud
```

### 第二步：安装插件

```
/plugin install claude-hud
```

### 第三步：配置状态栏

```
/claude-hud:setup
```

完成！重启 Claude Code，HUD 就会出现。

## 四、工作原理

Claude HUD 利用 Claude Code 原生的 **statusline API** 工作：

**核心特点：**

* 使用 Claude Code 提供的原始 token 数据（非估算）
* 自动适配新的 1M 上下文会话
* 解析 transcript 文件获取工具/Agent 活动
* 每 ~300ms 更新一次

## 五、灵活配置

运行以下命令随时自定义：

```
/claude-hud:configure
```

### 三种预设模式

| 预设 | 显示内容 |
| --- | --- |
| **Full** | 全部功能 — 工具、Agent、Todos、Git、配额、时长 |
| **Essential** | 活力行 + Git 状态，精简信息 |
| **Minimal** | 仅核心 — 模型名和上下文进度条 |

### 高级配置选项

编辑 `~/.claude/plugins/claude-hud/config.json` 可自定义：

* `lineLayout`

  : 布局模式（`expanded` 多行 / `compact` 单行）
* `pathLevels`

  : 显示 1-3 级目录
* `colors.*`

  : 自定义颜色（context、usage、warning、critical）
* `gitStatus`

  : Git 状态显示选项
* `display`

  : 各种显示元素的开关

## 六、技术架构浅析

### 数据流向

### 数据来源

**1. stdin JSON**

1）来自 Claude Code 的原生准确数据

2）模型名称

3）Token 使用量

4） 上下文窗口大小

5）Transcript 文件路径

**2. transcript JSONL**

1）从会话记录中解析

2）工具使用状态（运行中/已完成）

3）Agent 活动信息

4）Todo 列表

**3. 配置文件**

1)读取项目配置

2）CLAUDE.md 文件数量

3） MCP 服务器数量

4） Hooks 数量

### 文件结构

```
src/  
├── index.ts          # 入口，协调整体数据流  
├── stdin.ts          # 解析 stdin JSON  
├── transcript.ts     # 解析会话记录  
├── config-reader.ts  # 读取配置项  
├── git.ts            # Git 状态  
├── usage-api.ts      # API 使用配额  
└── render/           # 渲染引擎  
    ├── session-line.ts    # 第一行：会话信息  
    ├── tools-line.ts      # 第二行：工具活动  
    ├── agents-line.ts     # 第三行：Agent 状态  
    ├── todos-line.ts      # 第四行：Todo 进度  
    └── colors.ts          # ANSI 颜色处理
```

### 上下文阈值分级

| 使用率 | 颜色 | 含义 |
| --- | --- | --- |
| 0-70% | 🟢 绿色 | 健康 |
| 70-85% | 🟡 黄色 | 警告 |
| 85%+ | 🔴 红色 | 危险，显示详细 token 分解 |

## 七、适用场景

### 1. 上下文管理

当你在处理大型项目时，能精确知道还剩多少上下文空间，避免关键时刻被截断。

### 2. 调试监控

实时查看 Claude 正在使用哪些工具、访问哪些文件，更好地理解其工作方式。

### 3. 多 Agent 协作

当 Claude 派出多个子 Agent 时，HUD 让你清楚看到每个 Agent 的任务和进度。

### 4. 配额管理

Pro/Max/Team 用户可以实时监控 API 使用配额，合理规划使用。

## 八、开源贡献

Claude HUD 由 **Jarrod Watts** 开发，采用 MIT 开源协议。

* GitHub: https://github.com/jarrodwatts/claude-hud
* 要求: Claude Code v1.0.80+ / Node.js 18+

## 九、常见问题

**Q: 配置没有生效？**

A: 检查 JSON 语法错误，确保路径级别为 1-3，布局模式为 expanded 或 compact。

**Q: Git 状态不显示？**

A: 确认你在 Git 仓库中，且 `gitStatus.enabled` 未设为 false。

**Q: 工具/Agent 行不显示？**

A: 这些行默认隐藏，需要在配置中开启 `showTools`、`showAgents`、`showTodos`。

**Q: HUD 不出现？**

A: 重启 Claude Code 以加载新的 statusLine 配置。

## 十、总结

Claude HUD 是一个简单但强大的工具，它让 Claude Code 的"黑盒"操作变得透明可见。无论是日常开发还是调试学习，它都能显著提升你的使用体验。

**如果你是 Claude Code 的重度用户，这个插件绝对值得一试！**

> 欢迎关注我的公众号，获取更多 AI 编程工具和技巧分享！