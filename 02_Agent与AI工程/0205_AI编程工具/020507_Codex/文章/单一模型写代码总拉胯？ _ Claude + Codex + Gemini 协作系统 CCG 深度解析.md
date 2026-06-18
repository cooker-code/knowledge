---
title: 单一模型写代码总拉胯？ | Claude + Codex + Gemini 协作系统 CCG 深度解析
author: 技术杨工
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI0ODA2NzYwNw==&mid=2674937218&idx=1&sn=062882bea87d036845a12001096d8481&chksm=f251f717d6b169db1592afb3bbe563b3e0d31b8f10aba0e5c44551182eb2df8e2e788f7e32ae&mpshare=1&scene=24&srcid=0117Lm1FfhnxUtgEP0Sxae3a&sharer_shareinfo=7a979ab9fe374f07a67d2d4689a35ff7&sharer_shareinfo_first=7a979ab9fe374f07a67d2d4689a35ff7#rd
---

# 单一模型写代码总拉胯？ | Claude + Codex + Gemini 协作系统 CCG 深度解析

作为一名开发者，你是否经历过这样的痛点：让通用的 AI 大模型写前端，它经常在最新的 CSS 语法上“翻车”；让它写复杂的后端逻辑，又容易在数据库连接上卡壳。单一模型往往很难在所有领域都保持顶尖水平。

如果能把“专精前端”的模型和“专精后端”的模型结合起来，让一个“最强大脑”来指挥它们，那开发效率岂不是直接起飞？

今天要介绍的 **CCG (Claude + Codex + Gemini)** 就是这样一个让各路神仙“各司其职”的多模型协作开发系统。它让 `Claude Code` 担任项目经理和代码审核，`Codex` 专职后端，`Gemini` 专职前端，通过智能路由实现极致的开发效率。

---

### 核心功能深挖：为何 CCG 如此硬核？

CCG 不仅仅是一个简单的脚本集合，它在架构设计上解决了多模型协作中的安全与效率问题。以下是其最核心的三大亮点：

#### 1. 智能任务路由：专业的人做专业的事

系统会根据任务类型，自动将其分发给最擅长的模型，避免了通用模型“博而不精”的问题。

* **前端任务**：自动路由至 `Gemini`（擅长 UI/UX 和现代前端框架）。
* **后端任务**：自动路由至 `Codex`（擅长逻辑处理和 API 开发）。
* **全局编排**：由 `Claude Code` 负责整体决策、代码审查和最终合并。

#### 2. 规划与执行分离（v1.7.39 新特性）

很多时候 AI 一口气生成代码容易失控。CCG 引入了“先规划，后执行”的机制，让人类可以在代码生成前进行干预。

* **多阶段解耦**：将庞大的工作流拆分为规划（Phase 1-2）和执行（Phase 3-5）。
* **计划持久化**：生成的计划保存为 Markdown 文件，支持跨会话执行。
* **人工干预**：在执行前，开发者可以直接修改生成的计划，确保方向正确。

#### 3. 严格的安全架构：只看 Patch，不碰代码

为了防止外部模型“胡乱修改”代码库，CCG 设计了一套极其实用的安全机制：

* **无写入权限**：外部模型（Codex/Gemini）没有直接的文件写入权限。
* **Unified Patch**：外部模型仅返回代码差异。
* **Claude 审核制**：由 Claude Code 审查 Patch 确认无误后，才会应用到项目中。

#### 核心架构图

```
Claude Code (编排)  
   │  
   ┌───┴───┐  
   ↓       ↓  
Codex   Gemini  
(后端)   (前端)  
   │       │  
   └───┬───┘  
       ↓  
  Unified Patch
```

---

### 实战演示：从安装到高效开发

#### 1. 快速安装

CCG 的安装非常简单，只需 Node.js 18+ 环境即可。

```
npx ccg-workflow
```

**环境要求**：

* 必须安装：Claude Code CLI、Node.js 18+
* 可选安装：Codex CLI（后端专用）、Gemini CLI（前端专用）

#### 2. 核心配置详解

安装完成后，CCG 会在 `~/.claude/` 下生成严格的目录结构和配置文件，掌握这些配置是你灵活调度的关键。

**目录结构：**

```
~/.claude/  
├── commands/ccg/       # 斜杠命令定义  
├── agents/ccg/         # 子智能体配置  
├── bin/codeagent-wrapper  
└── .ccg/  
    ├── config.toml  
    └── prompts/{codex,gemini}/
```

**关键环境变量配置**：

为了避免长时间运行任务导致超时，建议根据项目大小调整以下超时参数。在 `~/.claude/settings.json` 中配置：

| 变量 | 说明 | 默认值 |
| --- | --- | --- |
| `CODEAGENT_POST_MESSAGE_DELAY` | Codex 完成后等待时间（秒） | 5 |
| `CODEX_TIMEOUT` | codeagent-wrapper 执行超时（秒） | 7200 |
| `BASH_DEFAULT_TIMEOUT_MS` | Claude Code Bash 默认超时（毫秒） | 120000 |
| `BASH_MAX_TIMEOUT_MS` | Claude Code Bash 最大超时（毫秒） | 600000 |

**配置示例代码：**

```
{  
  "env": {  
    "CODEAGENT_POST_MESSAGE_DELAY": "1",  
    "CODEX_TIMEOUT": "7200",  
    "BASH_DEFAULT_TIMEOUT_MS": "600000",  
    "BASH_MAX_TIMEOUT_MS": "3600000"  
  }  
}
```

#### 3. 常用工作流命令

CCG 提供了丰富的斜杠命令来覆盖开发全流程，以下是几个高频命令的使用方式：

**场景一：开发新功能（规划与执行分离）**

这是 v1.7.39 推荐的最佳实践，先让 AI 出方案，你确认后再干活。

```
# 1. 生成实施计划  
/ccg:plan 实现用户认证功能  
  
# 2. 审查计划（可修改）  
# 计划保存至 .claude/plan/user-auth.md  
  
# 3. 执行计划（新会话也可执行）  
/ccg:execute .claude/plan/user-auth.md
```

**场景二：全流程自动化**

如果你对 AI 充分信任，可以直接使用 6 阶段完整工作流。

```
/ccg:workflow
```

更多专用命令包括：

* `/ccg:frontend`：仅处理前端任务
* `/ccg:backend`：仅处理后端任务
* `/ccg:debug`：智能问题诊断
* `/ccg:review`：代码审查

---

### 避坑指南与总结

#### ⚠️ 已知问题与避坑

在使用 Codex CLI 的过程中，社区反馈了一个常见问题：

* **问题**：Codex CLI 0.80.0 进程不退出（在 `--json` 模式下，Codex 完成输出后进程不会自动退出）。
* **解决**：务必在环境变量中设置 `CODEAGENT_POST_MESSAGE_DELAY=1`（如上文配置示例所示）。

#### 🎯 适用人群

* 使用 **Claude Code** 作为主力 AI 编码工具的开发者。
* 希望利用 **Gemini** 处理前端、**Codex** 处理后端以发挥模型特长的技术团队。
* 需要严格控制 AI 代码生成权限，追求代码安全的工程化项目。

**总结**：CCG 通过巧妙的编排层设计，将不同 AI 模型的优势最大化。它不仅是效率工具，更是未来“多模型协作开发”范式的一次优秀实践。目前该项目在 GitHub 上也备受关注，Star 增长迅速。

> GitHub开源地址：https://github.com/fengshao1227/ccg-workflow