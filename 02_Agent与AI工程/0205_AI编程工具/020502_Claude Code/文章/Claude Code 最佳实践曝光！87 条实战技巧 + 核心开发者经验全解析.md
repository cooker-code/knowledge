---
title: Claude Code 最佳实践曝光！87 条实战技巧 + 核心开发者经验全解析
author: 多智能体实验室
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkyMzIzOTYyMA==&mid=2247483753&idx=1&sn=e961a90b8c0a1fbbefb7abaf0e6d078e&chksm=c0f4ca23b62eb6f2327ec5cc9df71d33a75ec51a8264be1b45335869e9fd3c49aa6c47d1176c&mpshare=1&scene=24&srcid=0409w73qGR5FGDPmpp1rY7Uf&sharer_shareinfo=ed0278d77359dce43167fde70ca96550&sharer_shareinfo_first=ed0278d77359dce43167fde70ca96550#rd
---

这是一份来自 Claude Code 社区的实战指南，Boris Cherny 等核心开发者贡献了 87 条经验。如果你在用 Claude Code，或者想搞清楚子智能体、命令、技能这些东西到底怎么回事，往下看。

## 三个基本概念

Claude Code 有三种扩展方式，很多人搞混。先说清楚：

**子智能体（Agent）**`.claude/agents/<name>.md`  
全新的独立上下文，可以自定义工具、权限、模型。适合需要大量算力、需要隔离的复杂任务。

**命令（Command）**`.claude/commands/<name>.md`  
注入到当前上下文的提示模板，用来编排工作流。轻量，适合每天重复多次的操作。

**技能（Skill）**`.claude/skills/<name>/SKILL.md`  
可配置、可预加载、可自动发现。支持 context:fork 隔离运行，也可以在当前上下文执行。

简单说：

* 命令 = 给当前对话加点知识（轻量）

* 技能 = 给当前对话加点知识（可配置，能隔离）

* 子智能体 = 开个新对话（重量级）

### 怎么组合使用

```
命令 → 子智能体 → 技能
```

典型工作流：**研究 → 规划 → 执行 → 审查 → 发布**

### 其他机制

* **钩子（Hooks）**

  — 在特定事件触发时运行自定义处理器

* **MCP 服务器**

  — 连接外部工具、数据库、API

* **插件**

  — 打包技能、子智能体、钩子、MCP 服务器

* **检查点**

  — 基于 git 的自动回退，`Esc Esc` 或 `/rewind`

* **记忆**

  — 通过 CLAUDE.md 和 `.claude/rules/` 持久化上下文

## 什么时候用哪个？

**每天重复多次的操作** → 用命令  
比如格式化代码、生成文档、运行测试。

**需要隔离上下文、投入更多算力** → 用子智能体  
比如复杂的代码重构、大规模搜索、独立的研究任务。

**需要可配置、可复用** → 用技能  
比如代码审查、部署流程、QA 测试。

对于简单任务，直接用 Claude Code 就行，别搞复杂了。

## 技能设计的几个坑

**技能是文件夹，不是文件**  
用 references/、scripts/、examples/ 子目录组织内容。

**description 字段是触发器，不是摘要**  
别写"这个技能用来做代码审查"，要写"当用户说'review this PR'、'code review'、'检查代码'时触发"。

**构建 Gotchas 部分**  
记录 Claude 的失败点，这是信噪比最高的部分。比如"Claude 经常忘记检查边界条件"、"Claude 倾向于过度抽象"。

**给目标和约束，别写逐步指令**  
别写"第一步做 X，第二步做 Y"，写"目标是 Z，约束是不能破坏现有 API"。

**用 context: fork 隔离运行**  
复杂技能在隔离的子智能体中运行，主上下文只看最终结果。

## CLAUDE.md 怎么写

**每个文件控制在 200 行以内**  
太长了 Claude 会忽略。

**用 `<important if="...">` 标签**  
包裹特定领域的规则，防止被忽略。比如：

```
<important if="working with database"> 永远不要在生产环境直接运行 DROP TABLE </important>
```

**monorepo 用多个 CLAUDE.md**  
祖先目录的 CLAUDE.md 会被加载，后代目录的也会被加载。

**用 `.claude/rules/` 拆分大型指令**  
把不同领域的规则拆到不同文件。

**用 settings.json 强制工具行为**  
别在 CLAUDE.md 里写"绝不使用 rm -rf"，在 settings.json 里配置工具权限。

## 上下文管理是关键

**50% 时手动 /compact**  
上下文用到 50% 时，手动执行 `/compact` 压缩，避免进入"智能体愚蠢区"。

**切换任务时 /clear**  
开始新任务时用 `/clear` 重置上下文。

**说"使用子智能体"**  
保持主上下文干净专注。

**用 1M token 模型**  
配合 `/compact` 解决压缩期间的错误。

## 并行开发提效

**tmux 智能体团队 + git 工作树**  
多个智能体并行开发，独立上下文窗口。一个智能体引入 bug，另一个可能发现。

**周期性任务**

* `/loop`

  — 本地，最长 3 天

* `/schedule`

  — 云端

## 规划技巧

**始终从计划模式开始**  
别直接开始写代码。

**让 Claude 用 AskUserQuestion 采访你**  
搞清楚需求，再新建会话执行。

**启动第二个 Claude 审查计划**  
以高级工程师身份审查，或者用跨模型审查。

**原型 > PRD**  
构建 20-30 个版本，别写规格。构建成本低，多试试。

## Git / PR 规范

**PR 中位数 118 行**  
每个 PR 一个功能。

**始终压缩合并**  
干净的线性历史，易于 git revert 和 git bisect。

**频繁提交**  
尽量每小时至少提交一次。

## 调试技巧

**截图分享给 Claude**  
遇到问题直接截图。

**用 MCP 让 Claude 查看 Chrome 控制台**  
Claude in Chrome、Playwright。

**智能体搜索胜过 RAG**  
glob + grep 比向量数据库好用。Claude Code 试过向量数据库，后来放弃了。

## 常用功能

* **自动模式**

  — `claude --enable-auto-mode` 或 `Shift+Tab`

* **沙箱隔离**

  — `/sandbox`

* **语音输入**

  — `/voice`

* **远程控制**

  — `/remote-control` 或 `/rc`

* **定时任务**

  — `/loop`（本地）、`/schedule`（云端）

* **代码审查**

  — `/code-review`

* **回退**

  — `Esc Esc` 或 `/rewind`

## 社区工作流

* **Superpowers**

  — TDD 优先、铁律约束、全计划审查

* **Everything Claude Code**

  — 本能评分、AgentShield 安全、多语言规则

* **Spec Kit**

  — 规格驱动开发、宪法约束

* **gstack**

  — 角色人设、/codex 审查、并行冲刺

* **BMAD-METHOD**

  — 完整 SDLC、智能体人设、22+ 平台支持

* **HumanLayer**

  — RPI、上下文工程、300k+ 代码行实战

## 几个没有答案的问题

1. CLAUDE.md 里到底应该放什么，又该留下什么？
2. 什么时候用命令 vs 智能体 vs 技能？
3. 给子智能体详细人设能提高质量吗？
4. 能把现有代码库转换成规格，删除代码，然后让 AI 重新生成完全相同的代码吗？
5. 为什么 Claude 仍然忽略 CLAUDE.md 中的指令——即使全大写写着 MUST？

## 我的看法

这份指南最有价值的是**三个基本概念的区分**和**技能设计原则**。

几个值得试试的：

**技能的 description 字段**  
我现在的技能描述更像摘要，不像触发器。需要改。

**Gotchas 部分**  
在技能中记录失败点，这个习惯很好。可以逐步完善现有技能。

**context: fork**  
隔离运行技能，保持主上下文干净。复杂技能应该用这个。

**50% 时手动 /compact**  
这个阈值很实用，避免上下文退化。

这和 Claude Code 工具设计的底层哲学一致：核心是让开发者精细控制上下文和执行边界。