---
title: OpenCode + oh-my-opencode：开箱即用的 AI 智能体协作框架，像人类开发者一样编码
author: AI灵感闪现
date: 
url: https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247488504&idx=1&sn=4dfd0004d7975a2b1f69b5f41ee9430e&chksm=a7fd442a3e59e8a8c09446ce4f76ca25f206174b781d4db5431f7a452b7373581d3a48cdd1ae&mpshare=1&scene=24&srcid=01085VU7pKerkP29DtpfR08K&sharer_shareinfo=dd38ed7603ee010e9a9cf51066856199&sharer_shareinfo_first=dd38ed7603ee010e9a9cf51066856199#rd
---

> “
>
> 认识 oh-my-opencode，这是经过生产验证的 OpenCode AI 智能体框架，通过编排专业智能体、LSP 工具和 MCP 来像人类开发者一样编码。

## 引言

在希腊神话中，西西弗斯被惩罚永远将巨石推上山顶。今天的 AI 智能体也面临着相似的命运——日复一日地推动着它们的"石头"（思考）。但如果我们给它们像我们一样工作的工具会怎样？

**认识西西弗斯（Sisyphus）：** 一个经过生产验证、开箱即用的 OpenCode[1] 智能体框架，已在真实开发场景中处理了价值超过 24,000 美元的 token。

## 西西弗斯有何不同？

大多数智能体框架受限于 token 膨胀、工具局限性或编排能力差。西西弗斯通过以下方式解决这些问题：

1. **并行智能体编排** - 委托给专业模型，保持主上下文精简
2. **LSP/AST 工具** - 为智能体提供与 IDE 相同的重构能力
3. **精选 MCP** - 内置文档和代码搜索访问
4. **Claude Code 兼容** - 可直接替换现有工作流

安装

```
bunx oh-my-opencode install# or use npx if bunx doesn't worknpx oh-my-opencode install
```

配置

启动 opencode 后输入以下提示词

```
Install and configure by following the instructions here https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/refs/heads/master/README.md
```

根据 opencode 返回的内容进行配置

```
│                                                    ││  Configuration Summary                             ││                                                    ││    ✓ Claude (standard)                             ││    ✓ ChatGPT                                       ││    ✓ Gemini                                        ││                                                    ││  ────────────────────────────────────────          ││                                                    ││  Agent Configuration                               ││                                                    ││    • Sisyphus     → claude-opus-4-5                ││    • Oracle       → gpt-5.2                        ││    • Librarian    → glm-4.7-free                   ││    • Frontend     → antigravity-gemini-3-pro-high  ││                                                    │├────────────────────────────────────────────────────╯│◇  Next Steps - Authenticate your providers ─────────────────────╮│                                                                ││  opencode auth login (select Anthropic → Claude Pro/Max)       ││  opencode auth login (select OpenAI → ChatGPT Plus/Pro)        ││  opencode auth login (select Google → OAuth with Antigravity)  ││                                                                │├────────────────────────────────────────────────────────────────╯│◆  Configuration updated!││  Run opencode to start!│◇  🪄 The Magic Word ───────────────────────────────────────────────╮│                                                                   ││  Include ultrawork (or ulw) in your prompt.                       ││  All features work like magic—parallel agents, background tasks,  ││  deep exploration, and relentless execution until completion.     ││                                                                   │├───────────────────────────────────────────────────────────────────╯││  ★ If you found this helpful, consider starring the repo!││    gh repo star code-yeongyu/oh-my-opencode│└  oMoMoMoMo... Enjoy!
```

## 西西弗斯工作流程

### 工作原理

西西弗斯不会亲自搜索文件，而是向更快、更便宜的模型发起**后台任务**进行并行处理：

```
主智能体 (Opus 4.5)  
    ├─→ Oracle (GPT 5.2) - 架构与调试  
    ├─→ 前端工程师 (Gemini 3 Pro) - UI 开发  
    ├─→ 图书管理员 (Sonnet 4.5) - 文档与实现研究  
    └─→ Explore (Grok Code/Haiku) - 快速代码库探索
```

### 核心行为

1. **上下文管理**：通过委托探索，主智能体保持精简
2. **LSP 重构**：精确、确定性的代码变更
3. **UI 委托**：前端任务自动交给 Gemini 3 Pro
4. **战略备份**：遇到困难？调用 GPT 5.2 获取高智商指导
5. **TODO 强制**："TODO 持续执行器"让西西弗斯持续工作直到完成
6. **注释质量**：要么证明注释的合理性，要么删除——不做 AI 膨胀
7. **关键词检测**：只需输入 `ulw`（ultrawork），西西弗斯会自动处理

## 安装

### 快速开始

```
bunx oh-my-opencode install  
<span class="hljs-comment"># 或</span>  
npx oh-my-opencode install
```

按照交互式提示配置您的 Claude、ChatGPT 和 Gemini 订阅。

### 让智能体自动设置

只需将其粘贴到 OpenCode 中，让西西弗斯完成工作：

```
按照此处的说明进行安装和配置：  
https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/refs/heads/master/README.md
```

## 功能详解

### 智能体：你的 AI 开发团队

| 智能体 | 模型 | 用途 |
| --- | --- | --- |
| **西西弗斯** | Claude Opus 4.5 | 主编排器，配备 32k 思考预算 |
| **Oracle** | GPT 5.2 | 架构、代码审查、策略 |
| **Librarian** | Sonnet 4.5 / Gemini 3 Flash | 多仓库分析、文档查找 |
| **Explore** | Grok Code / Haiku | 快速代码库探索和模式匹配 |
| **Frontend Engineer** | Gemini 3 Pro | UI/UX 设计和实现 |
| **Document Writer** | Gemini 3 Flash | 技术写作和文档编写 |
| **Multimodal Looker** | Gemini 3 Flash | PDF、图像和图表分析 |

### LSP 和 AST 工具

为您的智能体提供您使用的相同工具：

* `lsp_hover` - 位置处的类型信息、文档、签名
* `lsp_goto_definition` - 跳转到符号定义
* `lsp_find_references` - 在工作区中查找所有用法
* `lsp_rename` - 在工作区中重命名符号
* `ast_grep_search` - AST 感知的代码模式搜索（支持 25 种语言）
* `ast_grep_replace` - AST 感知的代码替换

### 内置 MCP

* **context7**：官方文档查找
* **grep\_app**：超快速的 GitHub 代码搜索
* **playwright**：浏览器自动化（内置技能）

### Hooks 集成

完全支持 Claude Code 的 `settings.json` hook 系统：

* `PreToolUse` - 工具执行前
* `PostToolUse` - 工具执行后
* `UserPromptSubmit` - 用户提交提示时
* `Stop` - 会话空闲时

## 真实影响

> “
>
> "如果 Claude Code 7 天能完成人类 3 个月的工作，西西弗斯 1 小时就能完成。它会一直工作直到任务完成。" —— **B**，量化研究员

> “
>
> "我用 Oh My Opencode 一天就清理了 8000 个 eslint 警告" —— Jacob Ferrari[2]

> “
>
> "我用 Ohmyopencode 一夜之间将一个 45k 行的 tauri 应用转换为 SaaS Web 应用" —— James Hargis[3]

## 对比分析

### 与 Claude Code 对比

| 功能 | 西西弗斯 | Claude Code |
| --- | --- | --- |
| 多模型编排 | ✓（Claude、GPT、Gemini） | ✗（仅 Claude） |
| LSP 重构工具 | ✓（重命名、代码操作） | ✗（仅分析） |
| 后台智能体 | ✓（并行执行） | ✗ |
| AST-Grep 支持 | ✓ | ✗ |
| 自定义智能体模型 | ✓ | 有限 |

### 与 AmpCode 对比

| 功能 | 西西弗斯 | AmpCode |
| --- | --- | --- |
| OpenCode 原生 | ✓ | ✓ |
| Claude Code 兼容 | ✓ | ✗ |
| Google 认证支持 | ✓（多账户） | ✓ |
| 可配置性 | ✓ | 有限 |

## 最佳实践

### 实现最大生产力

1. **使用 `ulw` 关键字** - 自动激活完整并行模式
2. **让西西弗斯委派** - 不要微观管理，信任编排系统
3. **利用 LSP 工具** - 使用 `lsp_rename` 而非查找替换
4. **后台任务** - 在继续工作的同时启动研究

### 日常工作流示例

**复杂重构：**

```
ulw: 重构所有微服务中的认证系统
```

**功能实现：**

```
ulw: 添加用户偏好设置，包括 API、数据库迁移和 UI
```

**Bug 调查：**

```
ulw: 找到并修复图像处理管道中的内存泄漏
```

## 故障排除

### 问题：智能体中途停止

**解决方案**：TODO 持续执行器应该会自动恢复。检查它没有在 `disabled_hooks` 中被禁用。

### 问题：Token 使用量高

**解决方案**：西西弗斯创建后台任务以减少主上下文使用。确认后台智能体已启用。

### 问题：缺少文档

**解决方案**：确保图书管理员智能体可以访问 context7 MCP。检查 `disabled_mcps` 配置。

## 注意事项

* **生产力警告**：您的输出可能会大幅增加——不要让同事注意到
* **OpenCode 版本**：使用 OpenCode 1.0.150+ 以获得完整兼容性
* **Ubuntu/Debian**：如果您通过 Snap 安装了 Bun，请使用 `npx` 而不是 `bunx`

## 参考链接

* GitHub 仓库[4]
* OpenCode 文档[5]
* 安装指南[6]
* AGENTS.md 参考[7]
* 西西弗斯工作流深度解析[8]

## 结论

西西弗斯代表了花费超过 24,000 美元 token 测试所有可用智能体框架的成果。遇到的所有问题的解决方案都已内置其中——只需安装即可使用。

如果说 OpenCode 是开发环境的 Debian/Arch，那么 Oh My OpenCode 就是 Ubuntu。它开箱即用，为您处理复杂性，让您专注于构建。

**不要再为智能体框架的选择而纠结。** 安装西西弗斯，回到编码中来。

---

*本文介绍了 YeonGyu Kim 的 oh-my-opencode 项目中的西西弗斯智能体框架。该项目开源并欢迎贡献。*

### 引用链接

[1]OpenCode: *https://opencode.ai/docs/*

[2]Jacob Ferrari: *https://x.com/jacobferrari\_/status/2003258761952289061*

[3]James Hargis: *https://x.com/hargabyte/status/2007299688261882202*

[4]GitHub 仓库: *https://github.com/code-yeongyu/oh-my-opencode*

[5]OpenCode 文档: *https://opencode.ai/docs/*

[6]安装指南: *https://github.com/code-yeongyu/oh-my-opencode#installation*

[7]AGENTS.md 参考: *https://github.com/code-yeongyu/oh-my-opencode/blob/dev/AGENTS.md*

[8]西西弗斯工作流深度解析: *https://deepwiki.com/code-yeongyu/oh-my-opencode/10.1-sisyphus-agent-workflow*

推荐合集

[Claude Code](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5Mzk1NzA1NA==&action=getalbum&album_id=4074951080126136323#wechat_redirect)

## 加入 AI灵感闪现 微信群

长按下图二维码进入 AI灵感闪现 微信群

长按下图二维码添加微信好友 VibeSparking 加群

## 关注 AI灵感闪现 微信公众号