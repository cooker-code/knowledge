---
title: 融合了andrej-karpathy以及 Claude Code 建议的 AI Coding 技巧，我把他做成了一个 plugin，用上后效率自己体会有多强
author: 老码小张
date: 小张小张
url: https://mp.weixin.qq.com/s?__biz=MzkxNzY0OTA4Mg==&mid=2247493429&idx=1&sn=96121748463e6547d9242753df9b3c12&chksm=c0718fe0b76598abc0434ec1db8482f3dd24beda8648aa20ad2e0297fd0f35f445d2fda78b9a&mpshare=1&scene=24&srcid=0505SptP3dH0ppGkTZslhA0E&sharer_shareinfo=7ae436d7eb28ffd3bc795d44fa6f8739&sharer_shareinfo_first=7ae436d7eb28ffd3bc795d44fa6f8739#rd
---

[这个插件其实来源于昨天的一篇文章的思考](https://mp.weixin.qq.com/s?__biz=MzkxNzY0OTA4Mg==&mid=2247493415&idx=1&sn=6d08b8721b5841be805a06ea229e562f&scene=21#wechat_redirect)，我不用想都知道有人会说，能不能做成一个插件啊，这样我就直接可以在我的 XX （Claude code）中使用上这些技能，所以就有了今天这篇文章，确实，plugin 方便很多，一键安装并使用。

注意，一般来说，对于新项目，直接执行 /acp-init 即可，Claude code 会自主判断技能调用，很省心。下面说说我们为啥需要这样一份固化为一堆 skills 的插件来提升我们 AI Coding 的效率吧。

先说一个场景，我想你应该见过这种场景。

你让 AI 改一个小 bug：

"profile 页面在没有头像的时候会崩，帮我修一下。"

它答应得很爽，几秒之后 diff 出来：

* • 改了 `UserProfile.tsx`（合理）
* • 顺手"优化"了 `Avatar.tsx` 的命名（你没让它改）
* • 把 `utils/format.ts` 整个重排了一遍（你也没让它改）
* • 新建了 `useAvatarFallback.ts`，里面只有一个 hook，只被一个地方调用（？）
* • 最后告诉你："已完成，所有测试通过。"

你问它："你跑测试了吗？"

它说："没跑，但根据代码逻辑，应该没问题。"

这大概就是很多朋友面临的 AI Coding 的真实体验：**模型不是不会写代码，是没有工程纪律。**

它会猜需求、扩大范围、过度设计、动无关代码、用一段自信的总结掩盖没做的验证。

这些毛病，资深工程师不会犯，不是因为他们更聪明，大概率是因为他们脑子里有一套"工程脾气"，一套明确知道"什么不能干"的判断。

我做的这个项目，就是想把这套脾气，**沉淀成 Agent 能执行的规则、Skill、检查清单**，然后封装成一个 Claude Code 插件，让所有人一行命令就能用。

仓库地址：

> https://github.com/bravekingzhang/agent-coding-playbook

使用后的效果：

你看看，废话没那么多，该干的活都干了，不该干得活不干，哪里没干好和你说一下，完全就是按照咱们昨天讲的七条原则来写代码，你说这能不规范吗？

下面看，马上帮你回顾一下哪 7 个原则。

### 我的核心思想：不是 prompt 大全，是工作手册

我必须把话说在前面：**这不是又一个"神奇咒语集合"。**

市面上 prompt 仓库已经够多了，但 prompt 是临场发挥，没法形成团队资产。一个工程师写得好不代表团队都用得好；今天 prompt 有效不代表换个模型还有效。

真正能沉淀的是**判断**：

* • 改代码之前，要不要先复述一遍需求？
* • 一个 bug 的修复 diff 应该有多大？
* • 多大的改动需要先写测试？
* • 哪些模块属于"高风险"，必须人审？
* • "完成"两个字到底意味着什么——代码编译过 vs 测试跑过 vs 行为验证过？

这些不是 prompt，是工程纪律。

我用 7 条原则把它写下来：

1. 1. **Think Before Coding** — 先讲清楚理解，再动手
2. 2. **Simplicity First** — 优先最简单直接的方案
3. 3. **Surgical Changes Only** — 只改必须改的
4. 4. **Goal-Driven Execution** — 没验证不算完成
5. 5. **Context Discipline** — 不要乱加上下文
6. 6. **Human Review Awareness** — 高风险必须人审
7. 7. **Honest Communication** — 不掩饰不确定性

听起来都是常识，但**让 AI 持续遵守这些常识，本身就是工程问题**。

### 从 Playbook 到 Plugin：为什么要做插件

最早这个项目只是一堆 markdown 文件，用法是手动复制 `CLAUDE.md` 到自己项目根目录。能用，但门槛高，更新麻烦，也没法在多个项目间统一复用。

直到 Claude Code 把 plugin 系统打磨得足够干净，机会才出现：

```
agent-coding-playbook/  
├── .claude-plugin/  
│   └── plugin.json          # 插件清单  
├── commands/                 # Slash 命令  
│   └── acp-init.md  
├── skills/                   # 自动触发的技能  
│   ├── acp-bug-fix/SKILL.md  
│   ├── acp-code-review/SKILL.md  
│   ├── acp-refactor/SKILL.md  
│   ├── acp-release-check/SKILL.md  
│   ├── acp-test-first/SKILL.md  
│   ├── acp-feature-add/SKILL.md  
│   ├── acp-investigate/SKILL.md  
│   └── acp-plan-task/SKILL.md  
├── checklists/  
└── docs/
```

只要在 Claude Code 里执行：

```
# 1. 把本仓库注册为 marketplace（marketplace 名字叫 acp）  
/plugin marketplace add bravekingzhang/agent-coding-playbook  
  
# 2. 从 acp marketplace 安装 agent-coding-playbook 插件  
/plugin install agent-coding-playbook@acp
```

8 个 Skill 就会**根据用户当前的请求自动触发**——不需要记任何咒语，不需要手动 `@skill xxx`。你说"帮我修这个 bug"，`acp-bug-fix` skill 就被加载；你说"我要 review 这个 PR"，`acp-code-review` 就上场。

> 所有 skill 名都加了 `acp-` 命名空间前缀，避免和你已有的同类 skill 撞名（这是 plugin 化最容易踩的坑）。

这就是 plugin 化最大的意义：**把工程经验从"手册"变成"环境"**。手册要人记得去翻，环境是默认在那里。

### Skill 是怎么自动触发的：一个被低估的细节

很多人没注意到，Claude Code 的 Skill 有一个关键字段叫 `description`。**它不是给人看的注释，是给模型看的触发条件。**

举个例子，看 `acp-bug-fix` 这个 skill 的 frontmatter：

```
---  
name: acp-bug-fix  
description: Use when the user asks to fix a bug, investigate  
  an error, debug a crash, explain a stack trace, or make a  
  failing test pass. Enforces reproduce → isolate → smallest  
  fix → verify, instead of guessing a cause and patching code.  
version: 1.0.0  
---
```

模型在每一轮对话里都会扫一眼所有可用 Skill 的 `description`，自己决定哪个 skill 和当前请求匹配。

所以**写好 description 是 skill 设计的关键能力**。它要做到：

* • 列举具体的触发短语（"fix a bug"、"debug a crash"、"explain a stack trace"）
* • 写清楚边界（不只是"修 bug"，而是"reproduce → isolate → fix → verify"）
* • 排除歧义（如果有同类 skill，要说清自己跟它们的区别）

我写 `acp-investigate` skill 的时候特别强调：

> Use when the user asks to understand, explain, trace, audit, or explore code **WITHOUT modifying it**.

为什么强调 WITHOUT？因为如果不强调，模型很可能在用户问"这段代码是怎么跑的？"时，顺手就给你"修"了。**Skill description 是模型行为边界的唯一规约。**

### 如果是在 Claude code 中谁用，那何不把Claude Code 新机制编织进去

只把老内容换皮放进 plugin 是不够的。Claude Code 迭代速度也是恐怖，harness 无人能及，为了充分使用上他的能力，咱们的Skill 必须跟上：

#### Plan Mode（计划模式）

通过 `ExitPlanMode` 工具，模型可以**先把计划摆出来给用户批准，再开始改代码**。这是"Think Before Coding"原则最直接的工程落地。

我在 `acp-feature-add`、`acp-plan-task`、`acp-refactor` skill 里都加了对它的引用：

> For anything beyond a trivial 1-file change, enter Plan Mode (ExitPlanMode) before editing.

#### Subagents（子代理）

`Explore` subagent 是专门做代码搜索的轻量级 agent，它读完一堆文件之后**只把摘要返回给主对话**，主上下文不会被一堆 grep 结果污染。

这是"Context Discipline"原则的关键工具。我在 `acp-bug-fix`、`acp-code-review`、`acp-investigate`、`acp-plan-task` 里都引导它优先用 Explore 而不是直接 Read：

> For non-trivial bugs that span multiple files, use the Explore subagent to locate suspect call sites without polluting the main session.

#### TodoWrite（任务分解）

多步任务的进度跟踪现在是 Claude Code 一等公民。`refactor`、`feature-add`、`plan-task` 这些 skill 在动手前都会创建 Todo 列表，**让每一步都有可验证的终点**。

这三个机制的引入只在每个 Skill 里加了 1–2 行，但效果完全不一样——它们把抽象原则变成了具体可执行的工具调用。

### `/acp-init`：一键给项目装上"工程脾气"

光有 Skill 还不够，每个项目都需要一个 `CLAUDE.md` 来定义项目级行为规则。

但写 `CLAUDE.md` 这件事，95% 的人会拖延到永远。

所以我做了 `/acp-init` 这个 slash command。在你的项目里执行一下，它会：

1. 1. 自动检测 `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod`，识别技术栈
2. 2. 自动填写 `install` / `test` / `typecheck` / `lint` / `build` 命令占位符
3. 3. 写入一份带 7 大原则的 `CLAUDE.md` 模板
4. 4. 如果文件已存在，**不会粗暴覆盖**——会先停下来问用户：跳过、追加、还是备份后替换
5. 5. 最后报告哪些占位符还需要人工填写

这是把"工程纪律"从一份倡议变成 30 秒就能装进项目的能力。

### 关于 Skill 设计的几个心得

写这 8 个 Skill 的过程，让我对 "Agent 能力设计"这件事有了几个更具体的认知：

#### 1. Skill 的边界比 Skill 的内容更重要

`refactor` 和 `feature-add` 都涉及改代码，但前者**绝对不能改行为**，后者**就是要改行为**。把这条边界写死，比写多少工作流细节都重要。

#### 2. Output Format 是纪律的容器

每个 skill 最后都要求模型按固定格式收尾：

```
Changed:  
- ...  
  
Verified:  
- ...  
  
Not verified:  
- ...  
  
Risks:  
- ...
```

`Not verified` 这一行是关键。**有这一行，模型就被迫坦白哪些没跑；没这一行，模型会本能地用 "Verified" 兜底所有东西。**

#### 3. Guardrails 是给"未来的自己"的护栏

每个 skill 都有一个 "Stop and ask for confirmation if..." 的兜底列表——auth、payment、migration、public API。这些是当模型跑得太顺、忘了风险时，最后一道刹车。

我宁可让 skill 啰嗦一点，也不要让它在跑得开心的时候撞墙。

### 关于这个项目"是什么 / 不是什么"

最后必须把丑话说在前面，因为我自己也讨厌过度营销：

**它是什么：**

* • 一份把工程判断写成 Agent 可执行规则的工作手册
* • 一个 Claude Code v0.1.0 插件，可以一键安装
* • 一份能让团队共享的 AI Coding 工程纪律基线

**它不是什么：**

* • 不是 prompt 大全
* • 不是神奇咒语集合
* • 不是"装上就能让 AI 写出完美代码"的银弹
* • 不是已经在大型企业实战过的成熟方案（v0.1.0，欢迎来踩）

它能解决的真实问题是：**让你的 AI Coding agent 不再"看起来很努力但其实在乱搞"，而是按一个靠谱工程师的方式干活。**

### 写在最后

如果你只能从这篇文章带走一句话，请带走这句：

> 不要训练 AI 会写代码，要训练 AI 按工程规矩写代码。

写代码本身已经是 AI 的强项。**工程规矩才是稀缺资源。**

仓库地址：**https://github.com/bravekingzhang/agent-coding-playbook**

安装方式：

```
# 1. 把本仓库注册为 marketplace（marketplace 名字叫 acp）  
/plugin marketplace add bravekingzhang/agent-coding-playbook  
  
# 2. 从 acp marketplace 安装 agent-coding-playbook 插件  
/plugin install agent-coding-playbook@acp
```

欢迎 star、欢迎 issue、欢迎 PR，更欢迎在评论区吐槽你被 AI Coding 坑过的经历。

我们一起，把工匠精神留在代码里。