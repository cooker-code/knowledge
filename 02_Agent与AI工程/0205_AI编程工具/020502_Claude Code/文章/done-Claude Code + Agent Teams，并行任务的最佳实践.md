> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code + Agent Teams，并行任务的最佳实践
author: 机器学习实验室
date:
url: https://mp.weixin.qq.com/s?__biz=MzI4ODY2NjYzMQ==&mid=2247502142&idx=1&sn=aace22852ad064c44454aab4ea70b2f1&chksm=ed813a273bc2e2c6555983bf9f503a812a935feb27e877ca1dd103552a7c542d35bf87dafa9d&mpshare=1&scene=24&srcid=0428hmIQUk9LDzO08RbTWNSm&sharer_shareinfo=6da87205c87ae032776653447e30aaa6&sharer_shareinfo_first=6da87205c87ae032776653447e30aaa6#rd
---

大家好，我是鲁工。

Agent Teams不算啥新功能了，我很早就在Claude Code里面用上了。之前有Agent swarm、Agent group等叫法，但本质上都是并行驱动多个Agent来完成任务，其实就是开团。

我上周末把Claude Code中有关Agent teams的文档仔细研读了下，发现这个东西能挖的东西挺多。

这篇文章按官方文档的逻辑顺序串起来写一下：先讲清楚和subagents的区别、什么场景该用、怎么启动、怎么控制、怎么算才算最佳实践，再用三个我自己跑过的实测案例落到具体场景。看完这篇基本能从零上手。

## Agent Teams vs. Subagents

Claude Code里做多任务并行，之前主要靠Subagents（子智能体）或者git worktrees。这块我之前写过一些文章，具体可参考：

[用Subagents打造Claude Code专业开发团队](https://mp.weixin.qq.com/s?__biz=MzE5ODY5MDU4Mw==&mid=2247483687&idx=1&sn=7e14d5bd5dafc9e8db0181687d3da5bc&scene=21#wechat_redirect)

[Claude Code + Git Worktrees，并行开发最正确的打开方式](https://mp.weixin.qq.com/s?__biz=MzE5ODY5MDU4Mw==&mid=2247484869&idx=1&sn=1d723494a0cef93f4a21d1d1e5913f8f&scene=21#wechat_redirect)

Subagents能有独立的context window、配专用的系统提示、选不同的工具和模型。但它有个天花板：只能向主代理汇报结果，彼此之间没有直接对话的通道。

Agent Teams把这个限制打掉了。

三个维度看差别：

| 维度 | Subagents | Agent Teams |
| --- | --- | --- |
| 通信 | 只向主代理汇报 | 队友之间直接收发消息 |
| 协调 | 主代理统一管理 | 共享任务列表+自协调 |
| Context | 自己的窗口，结果回主会话 | 自己的窗口，完全独立 |
| Token成本 | 较低（结果汇总） | 较高（每个队友是独立Claude实例） |
| 适用场景 | 结果导向的专注任务 | 需要讨论和质疑的复杂工作 |

总结下就是，Subagents是实习生各自给领导交作业，Agent Teams是小组同事坐一起能互相讨论。

## Agent Teams什么时候启用

不是所有任务都适合开团。因为开团的token成本是真的高，一个3人团队的token消耗大概是单会话的3到4倍，盲目滥用会很疼。

官方文档列了几个最强用例：

**研究和审查类任务**。多个队友可以同时调查问题的不同方面，然后互相分享和质疑彼此的发现。比如审一个PR，三个队友分别负责安全、性能、测试覆盖三个角度。

**新模块或功能开发**。每个队友拥有一个独立的部分，不会相互干扰。前提是任务可拆，文件不冲突。

**带竞争假设的Bug调试**。多个队友各拿一个理论并行测试，能更快收敛到根因。这个对抗"找到一个看似合理的解释就停下来"的锚定偏见特别有用。

**跨层协调**。前端、后端、测试改动同时进行，每个队友负责一个方面。

反过来不适合的场景也很明确：

顺序依赖强的任务（B必须等A完成）、同一个文件多人编辑（容易覆盖）、依赖关系特别多的工作。这几种场景下要么用单会话顺序处理，要么用subagents就行，强行开团反而是负担。

## 如何启用Agent Teams

Agent Teams功能在Claude Code中并不是默认开启的，开团有两个前置条件。

一是Claude Code v2.1.32以上（这个大家基本都满足，Native安装方法会自动更新Claude Code版本）。

二是在 `~/.claude/settings.json` 里加一个环境变量：

```
{  "env": {    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"  }}
```

启动方式很简单：直接用自然语言告诉Claude你想要什么团队、做什么任务。Claude自己负责创建team lead、生成teammates、协调工作。

官方文档里给的标准例子：

> I'm designing a CLI tool that helps developers track TODO comments across their codebase. Create an agent team to explore this from different angles: one teammate on UX, one on technical architecture, one playing devil's advocate.

这条prompt官方原话是"This example works well"，三个角色独立、不需要互相等待，开团效果最好。

**触发关键词很关键**。prompt里必须显式出现`agent team` / `teammate` / `team lead` / `shared task list`这类词，Claude才会走Agent Teams分支。中文口语里说的"开两个agent并行"、"派两个子代理"这种写法有概率会被识别成subagents，可能不会创建team。

**指定队友数和模型**。Claude会根据任务自己决定开几个队友，但你也可以直接指定：

> Create a team with 4 teammates to refactor these modules in parallel. Use Sonnet for each teammate.

也可以让不同队友用不同模型，比如lead用 Opus、teammate用Sonnet控制成本。

**Claude主动提议团队**。如果你给的任务确实受益于并行，Claude会主动建议开团，但不会未经你同意就创建。最终决定权在你手里。

判断team是否真的被创建有个硬标准：执行后看 `~/.claude/teams/` 目录下有没有新生成的team目录（带`config.json`）。有目录才是真的开团了；没有就是走的subagents。

## 如何控制Teammates

团队跑起来之后，你不是只能等结果，整个过程都可以介入。先看显示模式，Agent Teams功能支持in-process和split panes两种显示模式：

**In-process**：所有队友都跑在主终端里，用`Shift+Down`循环切换。任何终端都能用，不用额外装东西。

**Split panes**：每个队友一个独立窗格，输出全程可见，点击哪个窗格就直接进到对应队友的会话。需要tmux或iTerm2支持。

启用方式三种，从大到小：

全局config，在 `~/.claude.json` 里设：

```
{  "teammateMode": "tmux"}
```

```
单次启动flag：
```

```
claude --teammate-mode tmux
```

默认`auto`模式：我们已经在tmux session里打开Claude Code，Claude会自动切split panes，否则回到in-process。

split panes的两种入口：tmux（macOS下最稳定）；iTerm2，需要安装it2 CLI，并在Settings → General → Magic开Enable Python API）。

每个队友都是完整独立的Claude Code会话，可以直接给它们发消息：

In-process模式：`Shift+Down`切到目标队友，输入消息直接发；`Enter`进入队友会话查看详情，`Escape`中断当前轮次；`Ctrl+T`调出共享任务列表。

Split-pane模式：直接点击窗格就能进对应队友的会话，每个队友有完整终端视图。

共享任务列表协调整个团队的工作。任务有三种状态：pending、in progress、completed，还可以设置依赖关系（一个任务blocked by另一个时，依赖任务不解除就不能认领）。

两种分配方式：lead显式指派（"把这个任务给researcher队友"），或者队友完成自己的任务后自动认领下一个未分配的。

复杂或风险高的任务可以要求队友先做planning，lead审过才动手：

> Spawn an architect teammate to add structured logging middleware. Require plan approval before any changes. Reject any plan that does not include rollback strategy and test coverage.

关键：在prompt里给lead写清楚批准准则（"必须包含测试覆盖"、"必须有回滚方案"），不然lead会按自己的判断批。被拒绝的队友会留在plan模式，按反馈修订重新提交。

### 关闭和清理方面，可以选择关闭单个队友以及任务完成后清理团队。关闭单个队友：

> Ask the researcher teammate to shut down

队友收到关闭请求可以批准或拒绝。

整个团队跑完，跟lead说一句：

> Clean up the team

这会删除共享团队资源。如果还有队友在跑，清理会失败，先关闭它们再清。这一步必须由lead执行，队友自己跑clean会出问题。

### 另外还可以使用hooks强制质量门。Agent Teams配套了三个新的hook事件：`TeammateIdle`（队友空闲时）、`TaskCreated`（任务创建时）、`TaskCompleted`（任务完成时）。在hook里以exit 2退出可以阻断对应行为，stderr内容会作为反馈传给teammate。

###

适合把团队规范硬编码进去，比如"任务标记完成前必须跑过测试"、"不允许创建修改数据库schema的任务"。比prompt里反复叮嘱可靠得多，prompt是软约束模型可能忘，hook是硬约束直接拦住。

## 两个实测玩法

下面给两个Agent Teams的玩法，一个是常规启用玩法，一个是tmux启用玩法。

### 案例一：一句话调研开团

最常规的玩法，给一个调研任务+大致角色分工，剩下交给Claude。

```
比如：
```

> 启用Agent Teams帮我调研Claude Code中MCP、Skills、Plugins、Hooks、Agent teams的核心区别与联系。

```
跑下来Claude会自动创建team lead加5个teammates，各跑各的方向，最后lead综合输出。
```

案例二：使用tmux体验分窗并行

我们可以把上述常规的Agent teams启用方法在tmux下重试一遍，感受下tmux对多个teammates的自动分窗并行效果。

## 并行工作的最佳实践

跑过几个团队之后总结的几条经验，结合官方建议。

**给队友足够的context**。队友自动加载项目级的CLAUDE.md、MCP servers和skills，但不会继承lead的对话历史。在生成prompt里要把任务相关的关键信息写清楚，特别是文件路径、模块约定、技术栈细节。

**团队规模3到5个最合适**。再多协调开销和token成本就指数上升，3到5个队友做5到6个任务每人是经过验证的甜蜜点。如果有15个独立小任务，3个队友是一个好起点，让队友各自轮转处理。teammates多了，即使是Max 20x订阅，也很容易触发五小时限额：

**任务粒度要恰到好处**。太小协调开销超过收益；太大队友长时间不check浪费风险高。理想粒度是一个自包含的可交付单元（一个函数、一个测试文件、一份审查）。

**等队友完成再继续**。lead有时会自己开始干活而不是等队友。如果发现这种情况，直接告诉它 "wait for your teammates to complete their tasks before proceeding"。

**从研究和审查类任务入手**。新手第一次开团，建议从不需要写代码的任务开始（PR审查、库调研、Bug调查），这种任务边界清楚、不涉及并行实施的协调挑战，能直观感受到Agent Teams的价值。

**避免文件冲突**。两个队友编辑同一个文件会互相覆盖。任务拆分时保证每个队友拥有独立的文件集。

**主动监控和指导**。让团队无人值守跑太久会浪费成本。中途用 `Ctrl+T` 看任务列表，发现方向跑偏直接Shift+Down过去重定向。

最容易碰上的问题：**team怎么都开不起来**。prompt跑下去仍然是`Running N agents...`这种subagents形态。那么请按下列顺序自查：

1. prompt里有没有显式用 `agent team` / `teammate` 关键词（启动章节强调过的）
2. `echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`

   确认下输出是不是`1`
3. 改完settings.json有没有完全退出Claude Code重启
4. 跑完prompt后 `ls ~/.claude/teams/` 看下目录有没有生成新team。有目录才是真触发了，这是判断的最硬标准

Agent Teams本质上是Claude Code工具层在多智能体协作上的一套工程化实现。3到5人的小团队规模、开发者能看懂每一步的透明度、本地直接跑不依赖云端，这些是它的核心定位。

如果你的任务到了几百个sub-agent并发那种量级（比如一次性生成30个落地页、批量匹配100个岗位生成简历），可以去看看Kimi K2.6刚发的Agent Swarm，走的是另一条云端大规模分解的路。两者定位不重合，按场景选就是了。

如果觉得有用，点个赞或者在看，也方便更多朋友看到。

我是鲁工，九年AI算法老兵，AI全栈开发者，深耕AI编程赛道。欢迎关注，感兴趣的朋友也可以加我微信（louwill26\_）交个朋友。

>/ 作者：鲁工