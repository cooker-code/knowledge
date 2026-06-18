> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: Trellis：统一 13 个 AI 编程平台的工作流框架
author: 比特财商
date:
url: https://mp.weixin.qq.com/s?__biz=MjM5MDQ1NjIyNA==&mid=2247490113&idx=1&sn=1a79e1d6b6e7fa83f19b0b8f12369e27&chksm=a7dbc04e54b630adb2385d23bb07b5eb393fd6f77606fa859d7787087280c098e5a555cf2102&mpshare=1&scene=24&srcid=0416nNgJdrZLicgNDGGENgq4&sharer_shareinfo=5008f5e1ae053f1936895a0865e4388e&sharer_shareinfo_first=5008f5e1ae053f1936895a0865e4388e#rd
---

你是否有过这样的困惑？

> 同时用 Cursor、Claude Code、Windsurf、OpenCode...每个工具都要重新配置一遍规范？
> 团队里有人用 Cursor，有人用 Copilot，代码规范根本统一不起来？
> 每次打开新会话，都要重复解释项目结构和开发流程？

**Trellis** 就是来解决这些问题的。

---

## 什么是 Trellis？

**Trellis** 是一个多平台 AI 编程框架，它可以：

* • 把一套规范复用到 **13 个不同的 AI 编程工具**
* • 让团队的代码规范和 workflow **统一管理**
* • 让 AI 会话之间 **记住上次的工作**

简单来说：一次配置，处处运行。

---

## 核心能力

### 1. 自动注入规范

把编码规范、文件结构规则、review 习惯写进 `.trellis/spec/`，Trellis 会自动把相关内容注入每个会话。

不再需要每次都重新解释项目。

### 2. 任务中心化工作流

在 `.trellis/tasks/` 目录下管理：

* • PRD（需求文档）
* • 实现上下文
* • 审查状态
* • 任务进度

AI 工作全程有迹可循。

### 3. 并行 Agent 执行

用 git worktrees + Trellis 任务结构，多个任务同时推进，不会互相踩脚。

### 4. 项目记忆

`.trellis/workspace/` 中的 journal 记录每次会话发生了什么，下一次启动时 AI 直接继承上下文。

### 5. 团队共享规范

规范放在仓库里，一个人的最佳实践可以造福整个团队。

---

## 支持的平台

目前支持 **13 个平台**：

| 平台 | 说明 |
| --- | --- |
| Claude Code | Anthropic 官方 |
| Cursor | 最火的 AI IDE |
| OpenCode | 开源替代 |
| iFlow | 字节跳动 |
| Codex | OpenAI |
| Kilo | //TODO |
| Kiro | //TODO |
| Gemini CLI | Google |
| Antigravity | //TODO |
| Windsurf | //TODO |
| Qoder | //TODO |
| CodeBuddy | //TODO |
| GitHub Copilot | //TODO |

---

## 目录结构

```
  .trellis/
├── spec/           # 项目规范、模式、指南
├── tasks/          # 任务 PRD、上下文、状态
├── workspace/     # 个人日志和会话连续性
├── workflow.md    # 共享工作流规则
└── scripts/       # 工作流工具
```

---

## 快速开始

```
  # 1. 安装
npm install -g @mindfoldhq/trellis@latest

# 2. 初始化
trellis init -u your-name

# 3. 或指定平台
trellis init --cursor --opencode --codex -u your-name
```

---

## 对比其他方案

| 方案 | 优点 | 缺点 |
| --- | --- | --- |
| CLAUDE.md | 简单 | 容易变成一个大文件 |
| AGENTS.md | 可团队共享 | 缺乏结构化 |
| .cursorrules | 针对单个工具 | 无法跨平台 |
| **Trellis** | 分层规范+任务+记忆+多平台 | 需要学习成本 |

---

## 为什么值得用？

1. 1. **不再重复输入**：一次编写，处处注入
2. 2. **团队统一**：规范在仓库里，团队成员都能用
3. 3. **会话连续**：AI 记住上次做了什么
4. 4. **多平台兼容**：换一个工具不用重新配置

---

## 项目地址

**GitHub**: https://github.com/mindfold-ai/Trellis

**文档**: https://docs.trytrellis.app/

**快速开始**: https://docs.trytrellis.app/guide/ch02-quick-start

---

*如果你管理过多个 AI 编程工具，或者带团队用 AI 编程，Trellis 值得试试。*