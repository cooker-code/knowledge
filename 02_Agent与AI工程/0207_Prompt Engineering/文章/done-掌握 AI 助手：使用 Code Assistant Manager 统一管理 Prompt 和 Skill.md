> 已吸收至：[[02_Agent与AI工程/0207_Prompt Engineering/0207_核心知识点/Prompt任务契约与评估闭环|Prompt任务契约与评估闭环]]
---
title: 掌握 AI 助手：使用 Code Assistant Manager 统一管理 Prompt 和 Skill
author: JZ AI与云计算
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg4MDA2NzUzNQ==&mid=2247483852&idx=1&sn=c57acb783a5094138a791c10b22b0712&chksm=ce4ef454d82753a39e0cecc075ce83e73d59be2b48289591992107899433862ff1927de5ca94&mpshare=1&scene=24&srcid=1126qRKjUDPtZRb8oLmKycPx&sharer_shareinfo=598981449581c408cee36ea006ca6cc3&sharer_shareinfo_first=598981449581c408cee36ea006ca6cc3#rd
---

# 你是否厌倦了为每个不同的 AI 助手手动编辑 `.md` 配置文件？你是否发现自己在 Claude、Gemini 和 Codex 之间复制粘贴相同的系统指令？

**Code Assistant Manager (CAM)** 旨在简化你的工作流程。它提供了一个统一的命令行界面，用于跨多个 AI 助手管理你的 **Prompts (提示词)** 和 **Skills (技能)**。

## 为什么选择 CAM？

* **集中管理**

  ：将所有的 prompt 和 skill 保存在一个地方。
* **多应用支持**

  ：无缝支持 Claude、Gemini 和 Codex (OpenAI)。
* **上下文切换**

  ：在不同的“角色”之间即时切换（例如，从“代码审查员”切换到“技术文档撰写人”）。
* **技能发现**

  ：轻松从社区仓库中查找并安装新功能。

---

## 1. 管理 Prompts (提示词)

在 CAM 中，Prompt 是可重用的文本模板，定义了 AI 助手的行为方式。你不再需要直接编辑 `CLAUDE.md` 或 `GEMINI.md`，而是管理“Prompt 对象”并根据需要激活它们。

### 创建 Prompt

假设你需要一个专门用于 Python 开发的 prompt。

```
# 交互式创建一个新的 prompt
cam prompt create python-dev --name "Python Expert" --description "Strict Python 3.10+ coding standards"

# 或者从现有文件创建
cam prompt create python-dev --name "Python Expert" --file ./my-prompts/python-expert.md
```

### 激活 Prompt

创建后，你可以为任何支持的助手“激活”此 prompt。这会自动更新底层的配置文件（例如 `~/.claude/CLAUDE.md`）。

```
# 让 Claude 成为你的 Python 专家
cam prompt activate python-dev --app claude

# 让 Gemini 也成为你的 Python 专家
cam prompt activate python-dev --app gemini
```

现在，两个助手都配置了相同的高质量指令。

### 实时导入 (Live Import)

已经在 `CLAUDE.md` 中有了很好的配置？将其导入 CAM 以防丢失：

```
cam prompt import-live --app claude --name "My Legacy Config"
```

---

## 2. 利用 Skills (技能) 增强功能

Skill 是赋予 AI 助手额外能力的插件或工具。CAM 允许你从 GitHub 仓库获取 skill 并将其直接安装到助手的环境中。

### 发现 Skills

首先，从配置的仓库（如 `anthropics/skills` 或 `ComposioHQ/awesome-claude-skills`）获取最新的 skill：

```
cam skill fetch
cam skill list
```

### 安装 Skill

发现了很酷的 skill？为你正在使用的特定助手安装它。

```
# 为 Claude 安装 "Google Search" skill
cam skill install "owner/repo:google-search" --app claude

# 为 Gemini 安装 "File System" skill
cam skill install "owner/repo:fs-tools" --app gemini
```

CAM 会处理 skill 文件的下载，并将其放置到正确的目录中（例如 `~/.claude/skills/`）。

### 管理仓库

你可以添加自己的 skill 仓库：

```
cam skill add-repo --owner my-org --name internal-skills --branch main
```

---

## 3. 实际工作流示例

想象一下，你正在从 **后端开发** 任务切换到 **技术文档编写** 任务。

**第一步：创建你的角色**

```
cam prompt create backend --name "Backend Dev" --file prompts/backend.md
cam prompt create tech-writer --name "Tech Writer" --file prompts/writer.md
```

**第二步：开始编码**

```
cam prompt activate backend --app claude
cam skill install "awesome-skills:git-tools" --app claude
# 现在 Claude 准备好写代码了！
```

**第三步：切换到写作**

```
cam prompt activate tech-writer --app claude
# 你的 CLAUDE.md 瞬间更新为写作指南。
```

---

## 总结

Code Assistant Manager 将配置 AI 工具的手动繁琐工作转化为强大的脚本化工作流。通过将 prompt 和 skill 视为托管资产，你可以构建一个一致且强大的 AI 环境，以适应你的需求。

立即尝试，掌控你的 AI 上下文！