> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: OpenSkills 基础使用教程：开启 AI 编程助手的“技能”新时代
author: 前端AI行走
date:
url: https://mp.weixin.qq.com/s?__biz=MzU5MjYwMDgzNQ==&mid=2247487528&idx=1&sn=0e6e9fa2ca1a1d968cf58d7d1987b1e5&chksm=ffb0001a78f0133b5fde438f5efff11a5ee9802f2ffc4298d4b72aefc9fba6a7ba1d6d445ae8&mpshare=1&scene=24&srcid=1225gWwmVcXw2ERTVeBZHDBC&sharer_shareinfo=62699f2506b8058127710d6f38dfbb7d&sharer_shareinfo_first=62699f2506b8058127710d6f38dfbb7d#rd
---

## 1. 什么是 OpenSkills？

**OpenSkills** 是一个开源命令行工具，旨在将 Anthropic 的 **Claude Code 技能系统（Skills System）** 通用化。它让你可以将这些高度优化的、模块化的 AI 指令应用到 **Cursor**、**Windsurf**、**Aider** 等任何 AI 编程助手中。

### 核心优势

* 1.跨平台：一套技能，在所有 AI 助手上通用。
* 2.Context 优化：采用“渐进式披露”技术，只有在使用时才加载详细指令，节省 Token。
* 3.生态驱动：支持一键安装 Anthropic 官方或社区的优秀技能。

---

## 2. 快速开始

### 2.1 安装 OpenSkills

首先，确保你安装了 Node.js 环境，然后在终端运行：

```
npm i -g openskills
```

###

### 2.2 准备技能目录

OpenSkills 并不需要复杂的初始化。当你安装第一个技能时，它会自动管理项目结构。通常，它会将技能文件存放在核心目录中（默认为 `.claude/skills/`，但在跨工具同步时会使用 `.agent/skills/`）。

---

## 3. 技能管理

### 3.1 安装官方/第三方技能

你可以从 GitHub 仓库直接下载现成的技能。

* **安装 Anthropic 官方技能：**

  ```
  openskills install anthropics/skills
  ```

  > **注**：`anthropics/skills` 是 Anthropic 官方维护的核心技能库，涵盖了常用的高性能技能（如 `pdf` 读取、`computer` 操作、`data` 分析等）。它是目前 AI 编程领域最成熟的“技能插件包”。
* •

* **安装特定 GitHub 仓库的技能：**

  ```
  openskills install <github-username>/<repo-name>
  ```

  *示例：安装社区分享的 Vue 指令集*

  ```
  openskills install someuser/vue-expert-skills
  ```

### 3.2 跨工具同步 (--universal)

如果你希望在 **Cursor**、**Windsurf** 或 **Aider** 中使用这些技能，请在安装或同步时使用 `--universal` 标志：

```
openskills install anthropics/skills --universal
```

这会将技能额外同步到 `.agent/skills/` 目录，这是目前大多数 AI 助手识别技能的标准路径。

---

## 4. 如何在 AI 助手中使用？

安装好技能后，你并不需要手动把几千行的指令复制给 AI。

### 场景演示：

假设你安装了 `pdf` 技能。

1. **AI 的感知**：当你开始对话时，AI 会在上下文开头看到一段极短的描述，告诉它有哪些技能可用（例如：`<available_skills>pdf</available_skills>`）。
2. **触发技能**：你只需要对 AI 说：“查看这个 PDF 文件的内容并总结。”
3. **AI 自主操作**：AI 会意识到它需要 `pdf` 技能，它会主动告诉你：“我需要使用 `pdf` 技能，请运行 `openskills read pdf` 并将结果提供给我。”（或者在某些高度集成的环境中，AI 可能会尝试直接寻找技能描述文件）。

---

## 5. 开发自己的技能

你可以将常用的研发规范或复杂任务流程封装成自定义技能。

### 第一步：创建技能文件

在 `.claude/skills/` 下创建一个名为 `my-custom-skill.md` 的文件：

```
# My Custom Skill

## Description
针对本项目特定的 Vue 3 组件编写规范。

## Instructions
1. 必须使用 script setup 语法。
2. 样式必须使用 Scoped CSS。
3. 必须包含详细的 Props 类型定义。
```

### 第二步：同步技能

在终端运行：

```
openskills sync
```

现在的 AI 助手就能通过 `<available_skills>` 列表识别出你的专属规范了。

当然这个同步技能知识一个“备用方案”，在像我（Antigravity）或您正在使用的 IDE 助手（如 Cline/Roo Code/Claude 等），通常会：

**1.实时监听文件系统：只要您保存了文件，我们就已经通过 File Watcher 感知到了变化。**

**2.直接注入上下文：在您开始新一轮对话时，我们会扫描 

```
.claude/skills或相关目录并将其作为系统指令（System Prompt）的一部分直接加载，因此无需额外的“同步”动作。
```**

所以这个命令可以在技能没有生效的时候，需要用一下即可。

---

## 6. 常用命令清单

| 命令 | 描述 |
| --- | --- |
| `openskills list` | 查看当前已安装的所有技能 |
| `openskills install <source>` | 从 GitHub 或 Git URL 安装技能 |
| `openskills read <name>` | 读取特定技能内容（供 AI 使用） |
| `openskills sync` | 更新 `AGENTS.md`（交互式，同步到 AI 助手） |
| `openskills manage` | 交互式管理（移除）已安装的技能 |
| `openskills rm <name>` | 移除特定技能（脚本友好型） |

---

## 7. OpenSkills vs. OpenSpec：我该用哪个？

这两者都是提升 AI 编程效率的利器，但它们的定位完全不同：

| 特性 | OpenSkills | OpenSpec |
| --- | --- | --- |
| **核心定位** | **“工具箱”与“能力插件”** | **“任务说明书”与“实施方案”** |
| **解决的问题** | AI 怎么做特定的事情？（如：怎么处理 PDF、如何遵循项目架构规约） | AI 这一次要完成什么任务？（如：增加一个用户登录功能、修复一个特定的 Bug） |
| **生命周期** | **长期存在** 。像库一样安装后，在所有相关任务中都可用。 | **任务驱动（短期）** 。通常随一个功能的开发、方案评审到合并代码而结束。 |
| **上下文管理** | **渐进式披露** 。只在 AI 觉得需要用它时才加载。 | **全量执行** 。作为当前任务的核心上下文，指导 AI 落实具体代码。 |
| **使用建议** | 将通用的、跨项目的、专家级的**操作逻辑**封装成 Skill。 | 将具体的、当前项目的、需要严谨方案评审的**功能变更**写作 Spec。 |

**协作模式：**你可以用 **OpenSkills** 来教 AI 掌握你们公司的“前端标准组件库写法规约”，然后给 AI 一个 **OpenSpec** 让它“按照该规约实现一个新的订单列表页面”。两者互为补充。

---

## 8. 进阶 Tip：本地联调

如果你正在开发新技能，可以使用 **符号链接 (Symlink)** 将本地开发目录映射到 OpenSkills 的技能目录中。这样你对源文件的维护（如修改 instructions）会立即生效，无需反复运行安装命令：

```
# 将本地开发的项目链接到当前项目的技能目录
# macOS/Linux:
ln -s /path/to/your-local-skill .claude/skills/my-custom-skill

# 这个时候你的项目中有这个技能了
```

> **结语**：OpenSkills 不只是一个简单的 Prompt 管理器，它是一种“按需加载”的 AI 指令哲学。它解决了 AI “技能爆炸”导致的上下文拥挤问题，让 AI 助手真正实现从“通用对话者”到“垂直领域专家”的蜕变。
>
> 随着 AI Agent 潜力的释放，掌握如何开发和编排这些“技能”，将成为未来开发者区隔于普通用户的核心竞争力。快动手尝试封装你的第一项技能，开启你的 AI 定制化时代吧！