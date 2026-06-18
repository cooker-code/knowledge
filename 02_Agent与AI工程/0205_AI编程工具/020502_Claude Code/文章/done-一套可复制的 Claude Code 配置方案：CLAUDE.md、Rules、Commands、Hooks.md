> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 一套可复制的 Claude Code 配置方案：CLAUDE.md、Rules、Commands、Hooks
author: 架构师
date:
url: https://mp.weixin.qq.com/s?__biz=MzAwNjQwNzU2NQ==&mid=2650408189&idx=1&sn=7d4f7a442a22af37f95c46ff1048a3df&chksm=8254e6e00606258f855bd296d9cdbfc85856ddbfb8c8ed0b884095a00ce9bb6eb58dfd90fe6e&mpshare=1&scene=24&srcid=0126Gw9UdxhnCOyV8weaGu9R&sharer_shareinfo=440f93f261c1d15d2bf5dcedfecb04ed&sharer_shareinfo_first=440f93f261c1d15d2bf5dcedfecb04ed#rd
---

架构师（JiaGouX）

我们都是架构师！
架构未来，你来不来？

---

很多团队用 Claude Code，会卡在一个尴尬区间：

* • 个人用起来很爽
* • 一换人就不稳定
* • 一换项目就复读
* • 产出质量时好时坏，返工还挺多

我这两个月反复观察后，得到一个结论：

**想把 Claude Code 用到“稳定可复制”，提示词只是入门。更关键的是把经验沉淀成一套可版本化、可评审、可迭代的配置体系。**

今天这篇，我想用一个开源仓库 `everything-claude-code` 当引子，讲清楚怎么把 Claude Code 的“用法”从个人技巧升级成团队资产，以及一套我认为最省心的落地顺序。

---

## TL;DR（太长不看版）

* • **先把“怎么工作”拆成 7 个构件**：`CLAUDE.md / rules / agents / commands / skills / hooks / MCP`。
* • **团队共享放 `.claude/` 并纳入版本控制**；个人偏好放 `~/.claude/`，不要混在一起。
* • **`rules` 固化底线，`commands` 固化流程，`hooks` 固化必做动作**。
* • **别照搬别人的整套配置**：先抽“结构”和“迁移方法”，再按你们的仓库与权限边界落地。
* • **落地顺序**：`CLAUDE.md` → `rules/` → `commands/` → `agents/` → `hooks/` → MCP。

---

## 先讲清楚：什么叫“把配置当代码”

“配置即代码”不只是在仓库里放一堆 prompt 文件。

它更像是给团队做了一套“AI 编程的工程化接口”：

* • 能被 review
* • 能被 diff
* • 能回滚
* • 能按项目演进
* • 能清晰分层（哪些是底线，哪些是偏好，哪些是流程入口）

你把它当成内部平台的一部分，会更容易做对。

---

## 一张图看全：配置体系的 7 个构件

我推荐用这 7 个“盒子”来拆 Claude Code 的用法，从强约束到弱约束：

1. 1. `CLAUDE.md`：项目级记忆与约定（构建/测试/目录结构/风格/禁区）。
2. 2. `rules/`：必须遵守的规则（安全、测试、代码风格、Git 工作流等）。
3. 3. `agents/`：专用子代理（规划、架构、代码审查、排障、E2E 等）。
4. 4. `commands/`：高频斜杠命令（把复杂流程“一键化”）。
5. 5. `skills/`：可复用的方法论与领域知识（TDD、后端/前端模式、写作规范等）。
6. 6. `hooks/`：在关键事件点自动化守卫（PreToolUse/PostToolUse/Stop/SessionStart…）。
7. 7. `.mcp.json` / MCP 配置：把外部工具接入到 Claude Code 的工具链里（MCP 可以理解成“外部工具接入的配置层”）。

一眼看懂它们的分工，可以用这句话：

**`CLAUDE.md` 讲“我们是谁”；`rules` 讲“必须守什么”；`commands` 讲“高频怎么做”；`hooks` 讲“必须发生什么”。**

如果你想把它画成一张内部培训图，可以用这段 Mermaid（渲染成图片再放进公众号更直观）：

---

## 推荐目录结构（团队最小可用）

把团队共享部分放进仓库，示例：

```
your-repo/
├─ CLAUDE.md
└─ .claude/
   ├─ rules/
   │  ├─ security.md
   │  ├─ testing.md
   │  └─ coding-style.md
   ├─ agents/
   │  ├─ planner.md
   │  ├─ code-reviewer.md
   │  └─ build-error-resolver.md
   ├─ commands/
   │  ├─ plan.md
   │  ├─ code-review.md
   │  └─ build-fix.md
   ├─ hooks/
   │  └─ hooks.json
   └─ settings.json
```

你也可以把个人层的东西放在 `~/.claude/`，但我建议团队先把“能共享的部分”收敛进项目里，避免人走配置也走。

---

## 把配置拆对：3 条判断规则

### 1) “所有项目都一样”放用户级，“项目独有”放项目级

个人偏好适合放 `~/.claude/CLAUDE.md`，比如“不用 emoji”“默认更偏向先写测试”等。

项目独有必须放仓库里，比如“这个项目用 pnpm”“这个仓库测试命令是什么”“哪些目录绝对不能改”。

### 2) 用 `rules` 固化底线，用 `commands` 固化流程

`rules/` 解决“必须遵守什么”，它应该短而硬。

`commands/` 解决“高频怎么做”，它应该可执行，可复用，并且带验收。

命令最值钱的地方在于减少个人差异，让团队输出趋同。

### 3) 把“需要换脑子”的任务交给 `agents`

典型要交给专用 agent 的任务包括：

* • 方案规划与权衡（planner/architect）
* • 代码审查（code-reviewer）
* • 安全审计（security-reviewer）
* • 构建排障（build-error-resolver）
* • E2E 测试设计（e2e-runner）

把这些任务从主对话里拆出去，能显著降低上下文污染。

---

## 参考仓库：为什么我建议你看看 everything-claude-code

`everything-claude-code` 是一个“配置集合型仓库”：它不卖概念，直接给你一套生产可用的 agents、commands、rules、hooks、skills、MCP 配置骨架。

我认为它的核心价值主要在两点：

1. 1. **让你看到一个成熟配置体系长什么样**：哪些东西应该写成 rule，哪些应该写成 command，哪些应该交给 hook。
2. 2. **让你少踩坑**：比如 `hooks.json` 里就包含了“提醒用 tmux 跑长命令”“push 前暂停让你复查”等很实战的护栏思路。

我自己的建议是按“先学结构，再抄局部”的方式用它：

* • 第一步：只抄目录结构和最小骨架
* • 第二步：挑 1 个 command（比如 `/plan`）落进你们的流程
* • 第三步：挑 1 个 hook（比如 `console.log` 提醒）做一致性守卫
* • 第四步：再考虑 MCP 和更强自动化

---

## 一个最值钱的习惯：把高风险动作变成 `/plan`

仓库里对 `/plan` 的定义很克制：
先复述需求，评估风险，列步骤，然后**等待你确认**，再开始改代码。

这件事看起来“慢”，但它解决的其实是团队里最贵的成本：返工与信任损耗。

你可以把它当成团队规范的一部分：遇到这些情况，必须先 `/plan`：

* • 多文件改动
* • 代码结构调整、重构
* • 引入新依赖
* • 安全/权限相关改动
* • 需求本身还不清晰

我更喜欢把它写成一句可执行的规则：

**如果你没法用一句话说清楚 diff，那就先写 Plan。**

---

## Hooks 的价值：把“经验教训”变成自动化护栏

团队里最常见的幻觉是：把规则写进 `CLAUDE.md` 之后，就会自动被遵守。

现实通常相反。越忙的时候，越容易忘；越急的时候，越容易破例。

所以我建议你用一个很硬的分工：

* • `CLAUDE.md / rules` 负责“解释和约束”
* • `hooks` 负责“确保发生”

`everything-claude-code` 的 `hooks.json` 里有一些非常典型的思路，我挑 3 类给你做参考：

1. 1. **提醒型**：跑 `npm install / test / cargo build / pytest` 这类耗时命令时，提醒用 tmux（终端会话管理工具）跑，避免会话断了丢日志。
2. 2. **阻断型**：某些行为直接 `exit 1` 拦住，比如“禁止在非 tmux 环境启动 dev server”。
3. 3. **一致性型**：编辑 JS/TS 文件后自动格式化、做类型检查，或者提醒 `console.log`。

这类 Hooks 的价值在于把团队共识落成确定性行为。

---

## 你应该怎样“抄”一个开源配置仓库

先说一句边界：
**不要把别人的完整配置当成你团队的标准答案。**

原因很现实：

* • 他们的语言栈、测试框架、Git 流程未必和你一致
* • 他们的权限边界和安全策略可能更激进或更保守
* • 他们长期迭代出的取舍，放到你们的业务里可能不适用

更稳的做法是把“抄”拆成三层：

### 第一层：抄结构

把 7 个构件的目录结构抄下来，先让团队有一个共同的放置位置。

### 第二层：抄模式

挑 2～3 个“可迁移模式”，比如：

* • `/plan` 强制先计划再实现
* • `rules/security.md` 约束密钥与高风险操作
* • 一个“提醒型 hook”，提高团队体验但不改变行为

### 第三层：抄实现

最后才是具体内容，比如某个 agent 的提示词、某个 hook 的脚本。

这时候你已经有了边界，也更知道该删什么。

---

## Plugins：把能力打包成“可分发组件”

很多人把 plugins 理解成“装几个工具”。我更愿意把它当成团队推广的手段：

**当你把 commands、agents、hooks、skills 打包成插件，新项目落地就更接近一次安装，推广成本会从“重新教”降到“装完就能跑”。**

在 `everything-claude-code` 的插件文档里，它给了一个很直接的安装路径（不同版本以你本机 Claude Code 为准）：

```
claude plugin marketplace add https://github.com/anthropics/claude-plugins-official
claude plugin marketplace add https://github.com/mixedbread-ai/mgrep
```

然后用 `/plugins` 打开插件浏览器，或直接安装：

```
claude plugin install typescript-lsp@claude-plugins-official
```

我对团队的建议是循序渐进：

1. 1. 先装“只读类”或“体验类”的插件（搜索、LSP（语言服务器协议）、review 辅助）。
2. 2. 再装会影响行为的插件或 hooks（格式化、lint、阻断）。
3. 3. 最后再做外部系统接入（MCP）。

---

## MCP：先把外部工具接进来，再谈自动化

当你希望 Claude Code 能直接查 issue、读 PR、拉日志、查监控时，MCP 才真正派上用场。

但我不建议一上来就接一堆。原因很朴素：启用的工具越多，上下文越贵，权限面也越大。

比较稳的落地顺序是：

* • 先从 CLI（命令行）工具开始（比如 `gh`），把流程跑通
* • 再把“稳定、可重复”的外部能力接成 MCP
* • 密钥和凭证用环境变量占位符注入，别进仓库

---

## 落地路线图（7 天版）

如果你要在团队里推进“配置即代码”，我建议按这个顺序来，阻力最小：

1. 1. Day 1：写一个够短的 `CLAUDE.md`，只放“Claude 读代码推不出来的信息”。
2. 2. Day 2：加 `rules/security.md` 与 `rules/testing.md`，把底线写硬。
3. 3. Day 3：落地 1 个命令：`/plan`，要求输出“改哪些文件 + 如何验收”。
4. 4. Day 4：再落地 1 个命令：`/code-review`（或等价流程），把 review 维度写死。
5. 5. Day 5：加 1 个提醒型 hook（不阻断），先让团队接受。
6. 6. Day 6：加 1 个一致性型 hook（格式化或类型检查），让质量变得可预期。
7. 7. Day 7：引入 1 个 subagent（code-reviewer 或 security-reviewer），把“查”和“审”拆出主会话。

到这一步，你会明显感觉到：Claude Code 不再是“靠个人发挥”，它开始像一套团队工具链。

---

## 常见翻车点（提前避开）

1. 1. **`CLAUDE.md` 写太长**：规则淹没规则，等于没有规则。
   修法：狠删，把“必须发生”的动作迁到 hooks。
2. 2. **把偏好写成底线**：团队里一定有人不买账，最后落地失败。
   修法：`rules` 只写底线，偏好放 `CLAUDE.md` 或命令模板里。
3. 3. **Hook 过度阻断**：一上来就拦各种命令，团队体验直接崩。
   修法：先提醒型，再一致性型，最后才是阻断型。
4. 4. **密钥进仓库**：这是最危险也最常见的事故之一。
   修法：用环境变量占位符，仓库里只放示例与说明。
5. 5. **照搬别人的流程**：你的项目结构和他们不一样，最后变成“每次都要解释为什么不适用”。
   修法：先抄结构，再抄模式，最后才抄实现。

---

## 最后一句话

把 Claude Code 用到团队层面，本质是做一件很传统的事：
**把隐性经验变成显性资产，把口头约定变成可执行系统。**

你不需要一上来就把 7 个构件全部做满。
从 `CLAUDE.md + rules + 一个 /plan 命令` 开始就够了。

---

## 参考

* • `everything-claude-code`：github.com/affaan-m/everything-claude-code
* • 本文参考的中文拆解文档：`https://claudecn.com/docs/claude-code/advanced/config-as-code`

```
```
```
```
```
```
相关阅读：

```
---
```
```
```
```
```

+ Claude Code 最佳实践：把上下文变成生产力（团队可落地版）
+ 把 AI 当成新同事：Agent Coding 的上下文与验证体系
+ 一周写百万行的背后：Cursor长时间运行 Agent 的工程方法论
+ 2026年生活重启指南
+ 我真不敢相信，AI 先加速的是工程师。
+ 扒一扒 Claude Cowork 系统提示词：Anthropic 如何打造数字同事
+ Cowork 安全架构深度解析：从 Claude Code 到 Cowork，Anthropic 如何把“可控”做成产品
+ Anthropic官方万字长文：AI Agent评估的系统化方法论
+ 银弹还是枷锁？Claude Agent SDK 的架构真相
+ Claude Code创始人亲授13条使用技巧
+ Claude Code 内部工具开源 code-simplifier：终结 AI 屎山代码的终极方案
```
```

> 版权申明：内容来源网络，仅供学习研究，版权归原创者所有。如有侵权烦请告知，我们会立即删除并表示歉意。谢谢!

**架构师**

我们都是架构师！

****关注**架构师(JiaGouX)，添加“星标”**

**获取每天技术干货，一起成为牛逼架构师**

**技术群请****加若飞：****1321113940****进架构师群**

投稿、合作、版权等邮箱：**admin@137x.com**