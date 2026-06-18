> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020603_SpecCode/020603_核心知识点/SpecCode规格驱动与验收边界|SpecCode规格驱动与验收边界]]
---
title: 告别混乱，用 AGENTS.md 统一你的 AI 开发工具规则
author: Simon Wong 的非线性漫游
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk3NTc2MTc1NA==&mid=2247484132&idx=1&sn=23cf6ad543b1cd92d5cdafa1a73b6554&chksm=c547f543741e22345780801e344799d35a2a156be858d714ca2592e0ad3b91c5fe5c13454f56&mpshare=1&scene=24&srcid=1028pq0OL4IQTKxzmkH799Ns&sharer_shareinfo=fac5d73cba123cab4e4a080cfc420849&sharer_shareinfo_first=fac5d73cba123cab4e4a080cfc420849#rd
---

## 前言

目前 AI 开发工具越来越多，各种基于 VScode 的 AI 编辑器、VSCode 插件、还有 CLI 工具等等。使用这些工具通常会创建一个或多个给 AI 看的 Agent 行为规则文件（后面我都叫 Agent 描述文件）。但是**它们普遍存在一个问题：各自的 Agent 描述文件都放在不同的地方**。

Cursor 放在 `./.cursor/rules/` 下，
Claude Code 放在 `CLAUDE.md` 中，
Gemini CLI 放在 `GEMINI.md` 中，
RooCode 放在 `.roo/rules/` 下。

看了都头大，不论是在团队协作中各自使用 Coding 工具，还是自己个人切换不同的工具，都会导致那项目中生成各种的 Agent 描述文件，要让他们统一会比较困难。

AGENTS.md[1] 解决了这类问题，它是一种开放性倡议。使用一个 AGENTS.md 可以跨多种 Agent 运行。

## AGENTS.md 介绍

AGENTS.md 是一个专门的、规范的文件，**用来给 AI 提供上下文信息和操作指南**，协助 AI coding Agent 处理你的项目。

区别于 README.md 是服务于人类的，而 AGENTS.md 服务于 AI 的。

OpenAI 并没有命名为 `OPENAI.md` ，也不是 `./openai/rules/`，而是一个通用的名称。这样就可以让各家 AI Coding Agent 兼容进来。**一个 AGENTS.md 可以跨多种 Agent 运行**。以下是主流的兼容 AGENTS.md 的开发工具

主流的兼容 AGENTS.md 的开发工具

AGENTS.md 背后不仅仅只有 OpenAI，还有谷歌的 Jules、Cursor、Amp、Factory 等的参与。这个标准被明确设计为厂商中立的，会使得整个开发者社区受益。

## AGENTS.md 使用

AGENTS.md 只是一个标准的 Markdown 文件，并没有特定的格式，可以直接创建文件。内容通常包括：

* • 项目概述
* • 构建和测试命令
* • 代码风格指南
* • 测试说明
* • 安全注意事项

### 示例

这里给你看一个简单的 AGENTS.md 的示例来感受下

```
## 项目概述

这是一个使用 React、TypeScript 和 Tailwind CSS 构建网站...

## 项目指令

- 使用 `pnpm run test --filter <project_name>` 来对指定包运行测试用例
- 在提交前，必须运行 `pnpm lint` 和 `pnpm test` 并确保全部通过
- ...

## 代码风格

- 优先使用函数式组件和 Hooks，避免使用类组件
- ...

## 注意事项

- 绝不能在前端代码中硬编码任何 API 密钥或敏感凭证
- ...

其他内容...
```

当然，实际项目我更建议让 AI 来编写 AGENTS.md ，并让 AI 实时调整。

### 复杂的规则扩展

AGENTS.md 只是一个单独的文件，没法像 Cursor 的 `.cursor/rules` 一样支持非常多的独立的规则。

我的建议是，你可以和团队商定一个存放各类规则的公共目录，比如 `.ai/rules/`。

然后你可以在 AGENTS.md 中补充 rules 信息内容。如下

```
### 开发规则

项目的规则存放于目录 `.ai/rules/` 下。

- 项目架构设计请看 [.ai/rules/project-architecture.md](project-architecture.md)
- 代码风格规范请看 [.ai/rules/code-style.md](code-style.md)
- ...
```

以 AGENTS.md 为入口，让 AI 自己识别需要到哪些文件中查看规则。

## 各家 AI Coding 如何支持

我这里放了部分主流的 AI Coding 工具的支持方式供你参考，如果没有找到你也可以去查看官方文档。

### Cursor

Cursor 默认支持 `AGENTS.md`，并把它视为 `.cursor/rules` 的简化代替方案。

### Copilot

Copilot 目前试验性的支持，你可以通过修改 VSCode 的设置 `"chat.useAgentsMdFile": true` 来支持。

### Gemini CLI

你可以在项目的 `.gemini/settings.json` 设置 `"contextFileName": "AGENTS.md"`。

以此让 Gemini CLI 使用 AGENTS.md。

### RooCode

RooCode 默认会自动加载项目中的 `AGENTS.md` 文件，你也可以显式的在 VSCode 的设置中加上 `"roo-cline.useAgentRules": true`。

### Claude Code 和其他不支持 `AGENTS.md` 的工具

Claude Code 本身是不支持自动读取 `AGENTS.md` 的，但是你可以在 `CLAUDE.md` 中加上一段提示词 `项目概况、规则相关内容请在 @AGENTS.md 中维护和查看`。

其他不支持的工具，你也可以使用类似的方式。

## 总结

AGENTS.md 的出现，为我们提供了一个优雅的解决方案，用以应对 AI 工具多样化带来的配置文件混乱问题。我倡导团队协同的项目中，都去维护一份标准化的 AGENTS.md ，并与项目代码一起版本控制。

**如果你觉得这篇文章对你有帮助，欢迎点赞、分享，你的支持是我持续创作的最大动力！**

#### 引用链接

`[1]` AGENTS.md: *https://agents.md*