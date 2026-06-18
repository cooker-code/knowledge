> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: Superpowers 实测分析：从小游戏案例看其 14 个 Skills 如何协同
author: 阿南的技术手记
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg2OTI0MjIxNw==&mid=2247484935&idx=1&sn=16bc0927d4a75576eeaee3736ed8d61e&chksm=cfdb02562962d3423a432eb5e97e42ec9b9dbd47bd3f3b9ff1ebdeb30219d6f500fdc98343cd&mpshare=1&scene=24&srcid=04100XqiFoQMC3p7UIHmgRxw&sharer_shareinfo=d5dda0e743238262ef6419b083e24d41&sharer_shareinfo_first=d5dda0e743238262ef6419b083e24d41#rd
---

很多人接触 coding agent 时，有时都会有一种错觉：模型已经很强了，为什么它一到真实工程里，还是经常会跑偏？

它能很快写出函数，也能补几段代码，甚至能把一个页面搭出来。

但一旦任务稍微变复杂，问题就会集中出现：需求还没讲清楚，它先开始一顿写；bug 的根因还没找到，它先开始修；测试还没跑，它先说已经完成了有没有。

（先不说质量怎么，你就说快不快吧...）

这类问题，表面上看像模型不行，实际上很多时候不一定能力问题，而可能是流程问题。

今天要分享的是一个13.8万Star的项目`superpowers`，最近我认真研究了一遍 `superpowers` 当前仓库，发现它最有意思的地方不是发明了一套 prompt，而在于它试图给 coding agent 加上一层工程流程约束。

它要解决的不是怎么让 agent 多写一点代码或者写得更快，而是怎么让 agent 少做那些会让工程失控的动作。

我用一句话概括一下：

**Superpowers 不是提示词集合，而是一套把设计、计划、实现、审查、验证和收尾 串成闭环的 AI 编码工作流**

## 一、 代码写得飞快但缺少约束

coding agent 最常见的默认倾向，是收到任务后立刻行动。

这在简单任务里看起来很高效，但在真实项目里经常会变成问题。

因为工程工作并不是把代码尽快打出来这么简单。它需要一套顺序：

先澄清问题，再确定范围；

先设计，再实施；

先验证，再宣称完成；

先收尾，再进入下一个阶段。

而 agent 恰恰容易在这些地方失控。

比如在 `superpowers` 当前仓库里，`using-superpowers` 明确强调一条规则：

**如果哪怕只有 1% 的概率某个 skill 适用，也必须先调用 skill，再开始行动。**

这其实是在对抗 agent 的一种天然倾向: “快让我先做点什么再说”。

类似的约束在这个仓库里反复出现：

* `brainstorming` 强调，在任何创作型或实现型任务中，都不能跳过设计确认直接进入实现
* `systematic-debugging` 强调，不允许在没有根因分析的情况下直接修 bug
* `verification-before-completion` 强调，没有新鲜的验证证据，就不能宣称完成
* `test-driven-development` 强调，如果先写了代码再补测试，要删掉重来

把这些约束连起来看，你会发现 Superpowers 的设计意图非常明确：

**它不是在增强 agent 的创造力，而是在约束 agent 的冲动。**

这件事非常重要，因为在 AI 辅助研发里，最贵的成本往往不是模型没写出来，而是模型很快写出来了，但路径错了，效果不达预期。返工、误判、错误完成声明、无效修复，这些才是最伤协作效率的部分。

## 二、什么是 Superpowers



从项目官方的 README.md 来看，Superpowers 的定义非常直接：它是一套完整的软件开发工作流，建立在一组可组合的 `skills` 和一段确保 agent 会使用这些 skills 的初始指令之上。

这个定义有两个关键词很关键。

第一个是 **workflow**。

它不是一个库，不是一个 SDK，也不是一个给你导出若干 API 的运行时框架。当前仓库的主体也能证明这一点：你会看到大量 `skills/`、`commands/`、`hooks/` 和 `docs/`。

第二个是 **skills**。

这里的 skill 是行为约束和流程指导单元。每个 `SKILL.md` 都包含触发条件、执行原则、红线、流程图和常见反模式，它们就是用来塑造 agent 行为的。

从当前仓库来看，`skills/` 目录下共有 14 个核心 skill：

| 技能名称 | 功能 | 使用场景 |
| --- | --- | --- |
| `using-superpowers` | 调度器 | 任务前的总调度器 |
| `brainstorming` | 通过提问澄清需求 | 需求不明确或者需要补充设计时 |
| `writing-plans` | 任务拆解为客执行的步骤 | 复杂的任务开始前 |
| `using-git-worktrees` | 使用Git worktree隔离环境 | 同时开发多个功能时 |
| `subagent-driven-development` | 使用子代理执行任务 | 需要并行处理子任务时 |
| `executing-plans` | 按计划执行任务并设置检查点 | 按既定计划推进开发 |
| `dispatching-parallel-agents` | 并行代理调度 | 适用于需要处理两个或多个独立任务 |
| `finishing-a-development-branch` | 完成开发分支 | 当实现完成且测试均通过，并且需要决定如何集成工作时 |
| `test-driven-development` | 执行TDD | 在编写任何功能或错误修复的实现代码之前使用 |
| `systematic-debugging` | 系统性调试 | 在遇到任何错误、测试失败或意外行为时 |
| `requesting-code-review` | 请求代码审查 | 在完成任务、实现主要功能或合并代码之前使用 |
| `receiving-code-review` | 接收代码审查反馈 | 在接收代码审查反馈时、实施建议之前使用 |
| `verification-before-completion` | 完成前的验证 | 在准备声明工作已完成、已修复或已通过时使用 |
| `writing-skills` | 编写技能 | 在创建新技能、编辑现有技能或部署前 |

如果一定要给它一个更好理解的比喻，我会说：**Superpowers 更像给 coding agent 装上了一层流程操作系统。**

普通 prompt 在告诉 agent做什么，而 Superpowers 在规定 agent必须按什么顺序做。

## 三、Superpowers 的整体架构

如果把当前项目当成一个系统来看，它大致可以拆成四层

第一层是**平台接入层**。README 里已经明确写了支持的入口形态，包括 Codex、Claude Code、Gemini CLI、Cursor、Copilot CLI 等。这意味着 Superpowers 不是绑定单一产品的，而是试图做成一套跨 agent 运行环境的工作流插件。

第二层是**入口与路由层**。这里的核心就是 `using-superpowers`。它不是又一个 skill，而更像一个总调度器。它定义了最重要的一条上层规则：收到任务后，先判断有没有 skill 适用；有，就必须先用 skill。

第三层是**主工作流层**。这层负责把一个任务从模糊需求一路带到分支收尾。核心链路大致是：

`brainstorming -> writing-plans -> using-git-worktrees -> subagent-driven-development / executing-plans -> finishing-a-development-branch`

第四层是**质量与纠偏层**。这层不会单独完成一个功能，但它会在关键节点不断打断、校验和纠偏，包括：

* `test-driven-development`
* `systematic-debugging`
* `requesting-code-review`
* `receiving-code-review`
* `verification-before-completion`

这张图最重要的阅读方式，不是从左到右背 skill 名字，而是可以看出：**Superpowers 的核心不是 skill 数量，而是分层组织。**

翻译成白话，就是：平台只是入口，真正决定 agent 行为的，是中间那层路由规则，以及下面那两层工作流和质量约束

## 四、一次请求过来会发生什么

Superpowers 最有代表性的地方，是它把收到请求之后到底该怎么做拆成了一条顺序明确的链路。

第一步通常是 `using-superpowers`。

这个 skill 的作用，不是完成具体任务，而是阻止 agent 在没有做 skill 判断之前就开始行动。它几乎像一个“元规则”，要求所有任务都先经过 skill 检查。这是整个系统的入口控制器。

如果任务是创作型、功能型、需要构建或修改行为的，那么下一步通常会进入 `brainstorming`。这个 skill 的核心思想是：不要因为任务看起来简单，就跳过设计过程。哪怕是一个小功能，也要先理解目标、约束和成功标准，再提出方案，再展示设计，再让用户确认。

设计确认后，链路会进入 `writing-plans`。这个 skill 要求把实现计划拆成足够细的步骤，细到一个“上下文很少、判断力一般的工程师”也能照着做。它会要求明确文件路径、代码片段、验证命令和预期结果。

真正开始实施之前，还会经过 `using-git-worktrees`。这一步的意义在于给执行创造隔离空间，避免直接在主工作区或主分支上野蛮操作。

接下来才进入执行阶段。这里有两个主分支：

* `subagent-driven-development`：适合在当前会话里，用子代理逐任务推进，并在每个任务后做两阶段 review
* `executing-plans`：适合执行已有实现计划，按任务推进并在关键处停下来确认

最后，完成开发并不等于完成交付。系统还会要求进入 `finishing-a-development-branch`，在这里决定是本地合并、发 PR、保留分支还是丢弃工作，并先验证测试状态。

它想表达的重点只有一个：

**Superpowers 试图把 agent 从“收到任务就开写”，改造成“先定规则、再按阶段推进”的执行者。**

## 五、14 个 Skills 怎么分层理解

如果按文件名平铺介绍 14 个 skills，文章会很像说明书，不利于理解。更好的办法，是按职责给它们分层。

### 1. 入口与路由层

这里只有一个核心 skill：`using-superpowers`。

它负责定义“所有事开始前先检查 skill”这条总规则。可以把它理解成系统的入口守门员。

### 2. 设计与计划层

这一层包含两个 skill：

* `brainstorming`
* `writing-plans`

前者负责把模糊想法打磨成经过确认的设计，后者负责把设计变成可执行的工程计划。一个偏设计收敛，一个偏实现拆解。

### 3. 执行与协作层

这一层负责把计划真正变成代码与工程动作：

* `using-git-worktrees`
* `subagent-driven-development`
* `executing-plans`
* `dispatching-parallel-agents`
* `finishing-a-development-branch`

这里你能看到 Superpowers 的另一个特点：它并不把“写代码”看成单线程过程，而是把隔离工作区、并行代理、任务审查、开发收尾都纳入执行范畴。

### 4. 质量与纠偏层

这一层是整个系统的“刹车和校准系统”：

* `test-driven-development`
* `systematic-debugging`
* `requesting-code-review`
* `receiving-code-review`
* `verification-before-completion`
* `writing-skills`

严格来说，`writing-skills` 有一点元能力意味，因为它是拿来扩展和构建新 skill 的，但从读者理解上，把它放在质量与纠偏层最合适。因为它强调的仍然是：**就算写文档型 skill，也要测试、也要压测、也要验证。**

Superpowers 不是 14 个孤立的小工具，而是 14 个分工明确的行为单元。

## 六、这些 Skills 是怎么配合

理解 Superpowers，最重要的不是记住 skill 名字，而是看清它们之间的依赖关系。

我认为当前仓库里至少存在三条主要链路。

### 第一条：主开发链

这是从“有任务”到“开发完成”的主路径：

`using-superpowers -> brainstorming -> writing-plans -> using-git-worktrees -> subagent-driven-development | executing-plans -> finishing-a-development-branch`

这条链负责把一个功能型任务从模糊状态带到完成状态，是系统最核心的骨架。

### 第二条：质量保障链

这是为了防止开发过程失控而设置的保障链：

`test-driven-development -> requesting-code-review -> receiving-code-review -> verification-before-completion`

它的意义在于，不让 agent 因为“已经写完了”就自动进入“应该没问题了”的心理状态。写完不等于通过，review 完不等于完成，必须要有新鲜验证证据。

### 第三条：问题修复链

这是面向 bug 和异常行为的修复链：

`systematic-debugging -> test-driven-development -> verification-before-completion`

它解决的是另一类常见失控：在没有搞清根因时就连续试错。

除此之外，还有两类横向支撑关系。

第一类是 `dispatching-parallel-agents`。它不一定每次都在主链路中出现，但当任务之间彼此独立时，它会显著提高并行处理能力。

第二类是 `writing-skills`。它的作用不是完成业务任务，而是让 Superpowers 这套系统能够继续扩展自己。某种意义上，这是一个“写系统自身规则”的元技能。

如下的skills 依赖关系图，你会更容易把这些关系放到脑海中：**有主链，有质量链，有修复链，还有横向支撑。**

这就是为什么我说它更像“工作流操作系统”，而不是“skill 集合”。

## 七、它和 OpenSpec 有什么区别

如果你之前了解过 OpenSpec，读到这里大概率会有一个问题：它们不是都在强调流程、规范和阶段推进吗？差别到底在哪里？

这个问题必须讲清楚。因为如果这里不展开，Superpowers 很容易被误读成另一套流程文档方法论。

我的观点：**OpenSpec 和 Superpowers 都反对无约束地直接开写，但它们切入问题的层级并不一样。**

OpenSpec 更像是一套**规格驱动的方法论**。它强调把需求、设计、计划这些产物先沉淀出来，让团队在实现前对要做什么形成共识。它的核心抓手更偏文档、规格和阶段性产物。

而 Superpowers 更像是一套**直接作用在 agent 行为上的执行约束系统**。它不只是要求应该先写 spec，而是通过 `using-superpowers`、`brainstorming`、`verification-before-completion` 这种 skill 组合，直接限制 agent 在不同阶段能做什么、不能做什么。

如果用一句更直白的话说：

* **OpenSpec 更像是在定义项目过程应该产出什么**
* **Superpowers 更像是在约束 agent 在过程里应该怎么行动**

所以两者的重点并不相同。

OpenSpec 更偏规格治理，Superpowers 更偏行为治理。

OpenSpec 要解决的问题是：团队在开始做之前，是否已经把目标、边界、方案和计划说清楚了。

Superpowers 要解决的问题是：哪怕这些东西还没说清楚，agent 也不能跳过它们直接往下做；哪怕代码已经写了，agent 也不能不验证就宣布完成；哪怕 bug 看起来很明显，agent 也不能不找根因就直接修。

这也是为什么我会把它们看成**可以叠加，而不是互斥**的两种东西。

如果你的团队已经有 OpenSpec 这类规格流程，那么 Superpowers 很适合放在执行层，作为 agent 的行为护栏。

你甚至可以把它们理解成上下两层：

* 上层用 OpenSpec 约束项目该怎么被定义
* 下层用 Superpowers 约束agent 该怎么执行这些定义

所以，Superpowers 不是 OpenSpec 的替代品，更准确地说，它更像是 OpenSpec 这类方法论在 agent 执行侧的一层强化器。

## 八、开发一个小游戏案例

我选择了一个比较经典的小游戏："弹球打砖块"，提示词也比较简单，就一句话需求："帮我开发一个弹球打砖块的游戏"。

首先我选择不启用superpowers插件来进行开发

开发的非常快，当然效果也还不错，总耗时1分39秒就完成了

我觉得效果还不错，至少一次性能玩起来，70分

如下是游玩交互效果

接下来，我再开启Superpowers插件，再用同样的提示词，看看效果

开始首先它开启头脑风暴模式，进行需求讨论与分析

开启UI模式，居然让我在线选择交互的UI稿（有点惊了）

有点强，然后可以进行UI交互方案选择

接下来进行产品布局设计与功能，UI设计稿直出，可以提前看到成品UI稿与功能列表

还有道具规则，游戏规则设计的相当全面

整体产品设计方案确认完毕后，才开始干活

最后每项任务完成后，有相关状态进行功能验证

如下是交付结果，效果相当不错并且能直接玩

通过下面的动图感受下，提升至少两个档次（有道具、有关卡、有生命），互动感提升明显

直观来看对比，产品交付效果提升非常的明显。

## 九、Superpowers 使用场景

如果读到这里，一个自然的问题就是：既然这套系统这么强调流程，那是不是所有任务都该用它？

答案恰恰相反。

Superpowers 并不是要把所有对话都变成重型工程流程。它真正适合的是那些**有阶段、有协作、有验证要求**的任务。

比如下面这些场景，就很适合：

* 你在做一个需求还不够清晰的新功能
* 你需要实现一组多步骤、多文件、多阶段的改动
* 你希望 agent 不是“帮你打一段代码”，而是“按过程推进一个任务”
* 你要在多个子任务之间做并行协作
* 你对“最后是否真的完成”这件事有明确验证要求

但如果只是下面这些任务，Superpowers 就不一定是最佳选择：

* 一次性问答
* 轻量信息查询
* 不涉及设计、实现、验证、收尾的小任务
* 你已经非常确定做法、只是想让 agent 帮你补一段局部代码
* 你处在探索初期，只想快速试错，不想引入完整流程负担

换句话说，**如果任务本身不需要“阶段化协作”，Superpowers 的价值就会下降。**

这一点必须说清楚，因为它也是这套系统最明显的代价来源。

它的第一个缺点，是**流程偏重**。

你会明显感觉到，任务开始前的澄清、设计、计划、拆解、验证都更严格了。对复杂任务来说，这是收益；对简单任务来说，这可能就是额外摩擦。

第二个缺点，是**对平台能力有依赖**。

当前仓库的设计明显依赖 skill discovery、hook、甚至 subagent 这类能力。平台能力越强，Superpowers 的效果越完整；平台支持越弱，它的很多优势就发挥不出来。

第三个缺点，是**它要求使用者接受“先慢一点”**。

这套系统的核心逻辑是：前期多一点约束，后期少一点返工。可如果你的当下目标就是“尽快试一个想法”，那它不一定符合你的节奏。

所以更准确的结论不是“Superpowers 值不值得用”，而是：

**你的任务值不值得流程化。**

它的价值在于提醒读者：**Superpowers 不是默认全开，而是应该在值得流程化的时候启用。**

这点很关键。因为一个真正成熟的工作流系统，不是把所有事情都复杂化，而是知道什么时候该上流程，什么时候不该。

## 十、如何接入你 Coding Agent

### Claude Code 官方插件市场

**Superpowers 插件**已在 Claude 官方插件市场上线。

#### 安装方式（Claude 官方市场）

直接执行：

```
/plugin install superpowers@claude-plugins-official
```

### Claude Code（通过自定义插件市场）

在 Claude Code 中，需要先注册插件市场：

#### 1. 添加市场

```
/plugin marketplace add obra/superpowers-marketplace
```

#### 2. 安装插件

```
/plugin install superpowers@superpowers-marketplace
```

### Codex

对 Codex 输入以下指令：

```
Fetch and follow instructions from:
https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md
```

### Gemini CLI

#### 安装插件

```
gemini extensions install https://github.com/obra/superpowers
```

### Cursor（通过插件市场）

在 Cursor 的 Agent 聊天中执行：

```
/add-plugin superpowers
```

或者在插件市场中搜索 **“superpowers”** 并安装。

### OpenCode

对 OpenCode 输入以下指令：

```
Fetch and follow instructions from:
https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

## 最后：它最重要的是它的系统观

所以如果你已经在使用 Codex、Claude Code 或 Gemini CLI，我觉得 Superpowers 给你的最大收益不是装一个插件试试，而是能收获：**普通 prompt 在告诉 agent 做什么，Superpowers 在规定 agent 必须按什么顺序做。**

而这件事，恰恰是 AI 编码从能用走向可协作、可验证、可交付时，最缺的一层东西。

总结一下，**Superpowers 解决的不是代码生成问题，而是工程行为失控问题。**

来吧，赶快来安装试一下吧~  

下期分析OpenSpec原理与实践案例