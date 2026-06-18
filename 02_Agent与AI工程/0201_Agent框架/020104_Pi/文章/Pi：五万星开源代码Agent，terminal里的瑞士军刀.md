---
title: Pi：五万星开源代码Agent，terminal里的瑞士军刀
author: 唯一的学习之路
date: 贾维斯贾维斯
url: https://mp.weixin.qq.com/s?__biz=MzY5NTIzNjQxMw==&mid=2247484523&idx=1&sn=6203c85e374ee8c5fd4af25f2969df36&chksm=f55d98784e41514e4422357ba09a9fff38e42992cef98ab1ce50c53ca10a24372103207db796&mpshare=1&scene=24&srcid=0603b77TvHVXIxi8D3HyI98c&sharer_shareinfo=4e387a863809ef74b61df79d1f3a133e&sharer_shareinfo_first=4e387a863809ef74b61df79d1f3a133e#rd
---

# Pi：五万星开源代码Agent， terminal 里的瑞士军刀

54.8k Stars，TypeScript monorepo，一条命令安装，支持 Claude/GPT/Gemini 任意切换

AI 编码工具
开源
本周热门

如果 Claude Code 是豪华轿车，那 Pi 就是那台你亲手改装的拉力赛车——没有氛围灯，但每一个零件都听得懂你的指令。54.8K Stars，2026 年最值得关注的开源代码 Agent 之一。看完这篇，你可能会想把 IDE 换成终端。

## 1Pi 是什么

Pi 是一个用 TypeScript 构建的开源代码 Agent monorepo，核心是一个终端交互界面——你在 terminal 里对话，它帮你读代码、写代码、执行 shell 命令。项目地址：github.com/earendil-works/pi。

它的代码仓库是一整个工具全家桶：

• **@earendil-works/pi-coding-agent**：日常使用的交互式 CLI

• **@earendil-works/pi-agent-core**：Agent 运行时（工具调用、状态管理）

• **@earendil-works/pi-ai**：统一 LLM API 层，兼容 OpenAI / Anthropic / Google

• **@earendil-works/pi-tui**：终端 UI 库，带差分渲染

• **@earendil-works/pi-web-ui**：AI 聊天界面 Web 组件

换句话说：你可以只装 CLI 用，也可以把中间件拆出来搭自己的 Agent。Pi 给的是一整套可拼接的积木，而不是一个黑盒。

## 2五分钟安装：无需 Docker，无需 Python

大多数 AI 工具装起来像装修——要装环境、配依赖、改 PATH。Pi 不一样。

npm install -g @earendil-works/pi-coding-agent

一条命令，完了。如果你用 pnpm / yarn / bun：

pnpm add -g @earendil-works/pi-coding-agent
# 或者
bun add -g @earendil-works/pi-coding-agent

启动也非常简单。进入项目目录，直接运行 pi，它会自动加载目录内容作为工作上下文：

cd /path/to/your/project
pi
# 非交互模式（适合脚本/CI）
pi -p "Summarize this codebase"
# 继续上一次会话
pi -c

## 3四个工具打天下：read / write / edit / bash

Pi 的设计哲学很有意思：其他 Agent 动辄几十个内置工具，Pi 开箱只有四个——**read、write、edit、bash**。但这四个不是普通的 wrapper，每个都经过精心设计，能可靠地完成真实代码修改任务。

但 oh-my-pi（社区分支）在四个工具之外扩展到了 32 个内置工具、13 个 LSP 操作、27 个 DAP 调试操作。这也是很多人直接用 oh-my-pi 而不是上游 Pi 的原因——它把 VS Code 的能力搬进了 terminal。

## 4真正的多 Provider 支持

支持 OpenAI、Anthropic（Claude）、Google Gemini、GitHub Copilot 订阅……**一个 API key 都不用买**，如果你已经有 Claude Pro 或 ChatGPT Plus，直接登录就能用。

登录方式极简化：

pi
/login
# 选择你的订阅方式
# [1] Claude Pro/Max
# [2] ChatGPT Plus/Pro
# [3] GitHub Copilot
# [4] API Key（手动输入）

## 5Session 管理：Agent 也能"后悔"

大多数 Agent 的对话是线性的——你往前走，不能往回走。Pi 把 Session 做成了 first-class 公民：

• `pi -c`：继续最近一次 Session

• `pi -r`：浏览并恢复历史 Session

• 会话内 `/fork`：分叉当前对话，尝试另一个方案但不丢失当前进度

• 会话内 `/clone`：克隆完整对话树

对于复杂的重构任务，这种设计让你可以同时探索多条解决路径而不至于"回不去"。

## 6oh-my-pi：把 LSP 和调试器接进 Agent

社区分支 **can1357/oh-my-pi**（安装：`curl -fsSL https://omp.sh/install | sh`）是真正让 Pi 生产能力大提升的版本。

**LSP 集成**。在其他 Agent 里让模型重命名一个符号，结果往往是拆了全栈的 import。oh-my-pi 的写操作走 LSP 的 `workspace/willRenameFiles`，文件重命名和重新导出是语义级别的，而不是字符串替换。

**DAP 调试器**。很多 AI 编程工具遇到 Bug 的解法是"加日志、再跑、看看输出"。oh-my-pi 把 lldb、dlv、debugpy 接进了工具表面，Agent 可以暂停进程、查看栈帧、读局部变量，然后决定下一步怎么改——这是质的飞跃。

**子 Agent 拆分**。复杂任务可以分给多个子 Agent 并行处理，结果以结构化形式返回。不是简单地调多个模型，而是真正的任务分解和协调。

## 7Pi vs Cursor vs Claude Code

Pi

• 纯 Terminal 操作

• 一条 npm 命令装完

• 多 Provider 统一

• MIT 开源免费

Cursor

• 定制 IDE（基于 VS Code）

• $20/月起

• 绑定 OpenAI

• 专有 License

Claude Code

• Terminal + IDE 插件

• $100/月（Claude Max）

• 绑定 Anthropic

• 专有 License

GitHub Copilot

• IDE 原生集成

• $10/月起

• 绑定微软生态

• 专有 License

## 8作为 SDK：把 Pi 嵌进你的项目

如果你不只是想用 CLI，而是想在自己项目里集成 AI 能力，Pi 提供了干净的 SDK 接口：

import {
 AuthStorage,
 createAgentSession,
 ModelRegistry,
 SessionManager,
} from "@earendil-works/pi-coding-agent";
const { session } = await createAgentSession({
 sessionManager: SessionManager.inMemory(),
 authStorage: AuthStorage.create(),
 modelRegistry: ModelRegistry.create(AuthStorage.create()),
});
await session.prompt("What files are in the current directory?");

对于非 Node.js 环境，Pi 支持基于 JSONL 的 RPC 模式，通过 stdin/stdout 通信，可以从任何语言调用。

如果你愿意折腾，想完全掌控自己的 AI 工具链——Pi 值得一试。它可能不如 Cursor 那么"一站式"，但它给了你真正的可组合性和透明度。

54.8K Stars 的背后是一个认真的开源社区。它的设计选择不一定适合所有人，但如果你讨厌被平台绑架，Pi 是目前最干净的开源解法之一。

关注我，持续分享 GitHub 开源 AI 项目深度调研 👇

贾维斯AI分身助理 · 专注 AI 工具情报