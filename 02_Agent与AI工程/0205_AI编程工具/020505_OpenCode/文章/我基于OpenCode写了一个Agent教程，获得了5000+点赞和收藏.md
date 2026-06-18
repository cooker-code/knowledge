---
title: 我基于OpenCode写了一个Agent教程，获得了5000+点赞和收藏
author: Felix的上下文
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI2NzYxOTk4Ng==&mid=2247484777&idx=1&sn=fd4a6bd8a461f8c2907e5bcfad5577d5&chksm=eb3040c9c1567117d0d433e016e359eb09fe031be9d80aaebbec94f1579d43adf4f3ddd58918&mpshare=1&scene=24&srcid=0116UJC6L8Jsyn1hPm4u3bTC&sharer_shareinfo=7345ad9e40d921d70aff65e68c32ccff&sharer_shareinfo_first=7345ad9e40d921d70aff65e68c32ccff#rd
---

# learn-agents-from-opencode 教程导读

我之前介绍过 `opencode` 是一个非常好用的 coding agent，而且是开源的。

也正因为它是开源的，本身就成了一份质量很高的 **Agent 教程**。

基于它的源码结构，我整理了一套教程 **learn-agents-from-opencode**，仓库地址：

https://github.com/yexia553/learn-agents-from-opencode

前两天我在其他平台做过一个非常简短的视频介绍，获得了 **5000+ 的点赞和收藏**，感觉有很多人感兴趣，所以还是决定用文字再详细地写一下。

---

## 这不是从0开始的教程

首先这不是一套“从 0 开始教你写 Agent”的教程，而是一套**基于优秀开源Agent项目的拆解**，你至少具备以下两个条件：

1. 使用任何一款coding agent做过至少一个项目或者工具，哪怕是给自己使用的也行
2. 熟悉任何一门编程语言

这个教程更适合下面这些场景：

* 已经用过 Claude Code / Codex / OpenCode等coding agent，想知道它们**内部是怎么工作的**
* 想自己实现一个 **可控的 Coding Agent 或者其他类型的Agent**
* 对 Agent 的权限 / Tool / Session / Prompt 这些模块有真实工程需求

### 推荐阅读顺序（按目的）

如果你不打算从头到尾线性阅读，可以按目的选择：

* **只想理解 Agent 的整体工作方式**

+ 01 System Prompt
+ 03 Agent 系统
+ 05 迭代信息收集模式（Loop）

* **想自己实现一个简化版 OpenCode**

+ 01 → 02 → 04 → 05 → 07

* **想研究“Agent 如何在代码库中探索问题”**

+ 05 迭代信息收集模式
+ Explore Agent 相关部分

---

## 教程目录

| 序号 | 教程 | 说明 | 难度 |
| --- | --- | --- | --- |
| 01 | System Prompt 系统 | Prompt 构建、Provider 适配、环境注入 | ⭐⭐ |
| 02 | 权限审核系统 | 权限规则、请求流程、BashArity | ⭐⭐⭐ |
| 03 | Agent 系统 | 内置 Agent、配置系统、权限集成 | ⭐⭐ |
| 04 | Tool 系统 | 工具定义、注册、执行流程 | ⭐⭐⭐ |
| 05 | 迭代信息收集模式 | 核心循环、Tool Chaining、Explore Agent | ⭐⭐⭐⭐ |
| 06 | Skill 系统 | Skill 定义、发现机制、工具集成 | ⭐⭐ |
| 07 | Session 系统 | 会话管理、消息流转、上下文压缩 | ⭐⭐⭐ |
| 08 | Provider 系统 | 多模型适配、SDK 初始化、成本计算 | ⭐⭐⭐ |

> **说明：目前大部分内容由 AI（OpenCode / Claude Code / Codex）生成，我会逐步进行人工 review 和修正。**

---

## 1. System Prompt 系统（重点）

System Prompt 是 OpenCode 架构中**最容易被低估、但非常重要的部分**。

它并不是一个单一的 Prompt，而是一套 **分层拼装的系统输入**。

### System Prompt 的整体结构

从结构上看，System Prompt 可以拆成四个层次：

**Provider Prompt** + **环境上下文** + **项目 / 用户规则** + **Agent 角色约束**

OpenCode 在实现上把这四层拆得非常清晰。

### System Prompt 构建流程

`packages/opencode/src/session/prompt.ts`

```
System Prompt 组成:  
├── Provider 特定 prompt (provider/model 相关)  
│   ├── anthropic.txt  
│   ├── beast.txt  
│   ├── gemini.txt  
│   ├── codex.txt  
│   └── qwen.txt  
│  
├── 环境信息 (SystemPrompt.environment)  
│   ├── 工作目录  
│   ├── Git 仓库状态  
│   ├── 平台信息  
│   └── 当前日期  
│  
├── 自定义规则 (SystemPrompt.custom)  
│   ├── 项目级: AGENTS.md, CLAUDE.md, CONTEXT.md  
│   ├── 全局级: ~/.claude/CLAUDE.md  
│   ├── config.instructions  
│   └── URL 远程规则  
│  
└── Agent 特定 prompt  
    ├── build  
    ├── plan  
    ├── explore  
    ├── general  
    └── 自定义 agent
```

从这里可以看出，OpenCode 并不是“把所有规则塞进一个 prompt”， 而是明确区分了：

* **模型对齐（Provider Prompt）**
* **现实世界状态（环境）**
* **约束与规范（规则）**
* **行为角色（Agent）**

### 关键文件位置

| 文件 | 路径 | 用途 |
| --- | --- | --- |
| Provider Prompt | `packages/opencode/src/session/prompt/anthropic.txt` | 模型级系统提示 |
| Agent 定义 | `packages/opencode/src/agent/agent.ts` | Agent 配置 |
| Agent Prompt | `packages/opencode/src/agent/prompt/*.txt` | Agent 专用提示 |
| System Prompt 构建 | `packages/opencode/src/session/system.ts` | 组装逻辑 |
| Prompt 注入 | `packages/opencode/src/session/prompt.ts` | LLM 调用入口 |

---

## 2. 权限审核系统

**核心文件:**

| 文件 | 路径 | 说明 |
| --- | --- | --- |
| **权限核心** | `packages/opencode/src/permission/next.ts` | PermissionNext 命名空间 |
| **旧权限** | `packages/opencode/src/permission/index.ts` | Permission 命名空间 (旧版，这个说法不一定正确吗，只是为了与PermissionNext区分开) |
| **Arity 计算** | `packages/opencode/src/permission/arity.ts` | bash 命令参数分组 |
| **权限 UI** | `packages/opencode/src/cli/cmd/tui/routes/session/permission.tsx` | TUI 权限对话框 |
| **Bus 事件** | `packages/opencode/src/cli/cmd/tui/context/sync.tsx` | 事件同步 |

**权限类型:**

OpenCode 支持的权限类型覆盖了文件操作、代码搜索、命令执行等核心功能(实际上就是预置的tools)：

| 权限类型 | 功能说明 |
| --- | --- |
| edit | 文件编辑（创建和修改） |
| read | 文件读取 |
| glob | 文件路径匹配 |
| grep | 代码内容搜索 |
| list | 目录列表 |
| bash | 命令执行 |
| task | 子 Agent 调用 |
| todoread | 待办事项读取 |
| todowrite | 待办事项写入 |
| question | 向用户提问 |
| webfetch | 网页内容抓取 |
| websearch | 网络搜索 |
| codesearch | 代码库搜索 |
| external\_directory | 外部目录访问 |
| doom\_loop | 死循环检测 |

**权限动作:**

权限动作定义了用户对权限请求的响应方式：

* **allow** - 允许执行
* **deny** - 拒绝执行
* **ask** - 每次询问用户

用户响应选项（Reply）包括：Once（本次允许）、Always（始终允许）、Reject（拒绝）

**Agent 权限配置示例** (`packages/opencode/src/agent/agent.ts`):

Agent 权限配置采用简洁的声明式语法，支持通配符匹配和优先级规则：

* `*: "allow"` - 默认允许所有操作
* `doom_loop: "ask"` - 死循环检测需询问用户
* 特殊文件保护：通过详细规则禁止读取 `.env` 等敏感文件，同时允许读取示例文件

**工具调用权限请求示例** (`packages/opencode/src/tool/bash.ts`):

工具在执行敏感操作前通过 `ctx.ask()` 方法发起权限请求，每个请求包含：

* **permission** - 权限类型（如 `external_directory`、`bash`）
* **patterns** - 本次请求匹配的文件路径模式
* **always** - 记忆的"始终允许"路径模式
* **metadata** - 附加元数据

---

## 3. Agent 系统

`packages/opencode/src/agent/`

OpenCode 内置了三类核心 Agent，每一类都对应不同的权限与行为边界：

| Agent | 模式 | 用途 |
| --- | --- | --- |
| build | primary | 默认开发 |
| plan | primary | 只读规划 |
| explore | subagent | 代码探索 |

---

## 4. Tool 系统

**工具定义模式:**

OpenCode 的工具通过 `Tool.define()` 方法注册，每个工具包含三个核心要素：

| 要素 | 说明 |
| --- | --- |
| name | 工具唯一标识符 |
| description | 工具功能描述，供 LLM 理解使用场景 |
| parameters | 使用 Zod 定义参数Schema |
| execute | 工具执行逻辑 |

---

## 5. 迭代信息收集模式（核心）

这一部分是 OpenCode **与大量“简单 Agent 实现”拉开差距的地方**。

很多 Agent 的问题并不在模型能力，而在于：

* 过早修改代码
* 一次性读取大量上下文
* 缺少“探索 → 判断 → 行动”的分离

OpenCode 的做法是，把“信息收集”本身当成一等公民。

### 迭代信息收集流程

`packages/opencode/src/session/processor.ts`

```
用户提问  
  ↓  
LLM 判断：信息是否足够？  
  ↓  
不够 → 调用工具 → 评估结果 → 继续  
  ↓  
足够 → 执行最终行动
```

### Tool Chaining 示例

```
glob("**/*.ts")  
grep("API|export")  
read("file.ts")  
bash("npm test")
```

这是一个典型的“探索链”，而不是“执行链”。

### Explore Agent

`explore` 是一个专门为代码探索设计的子 Agent，权限采用白名单模式，只允许读取和搜索。

从我目前的理解开看，这个agent比较适合用于学习某一个repo。

---

## 6. Skill 系统

**Skill 定义格式** (`.opencode/skill/test-skill/SKILL.md`):

Skill 以 Markdown 文件形式定义，文件头部使用 YAML frontmatter 声明元数据：

| 字段 | 用途 |
| --- | --- |
| name | Skill 唯一标识符 |
| description | 功能描述，供 Agent 判断何时使用 |

文件主体为详细的使用文档，Agent 在需要时会调用对应的 Skill。

---

## 7. Session 管理

`packages/opencode/src/session/`

* 会话状态
* 消息流
* 上下文压缩
* 权限继承

---

## 8. Provider 系统

`packages/opencode/src/provider/`

* 多模型适配
* Tool Schema 转换
* 成本与调用管理

---

## 最后

这套教程的目标不是复刻 OpenCode，而是学习OpenCode这种工程级的Agent实现，所以教程的重点会放在(sub)Agent、Tools，Loop，Skills等方面，TUI很少涉及。

如果你正在做 Agent / AI 应用工程化，希望它能提供一些可复用的结构和思路。

前面提到，这个教程，目前大部分内容都是由OpenCode、Claude Code和Codex这些AI Coding Agent生成的，需要人工review，让其变得更好、更准确，如果有小伙伴感兴趣，欢迎参与开源协作！

最后，致敬 OpenCode：https://github.com/anomalyco/opencode