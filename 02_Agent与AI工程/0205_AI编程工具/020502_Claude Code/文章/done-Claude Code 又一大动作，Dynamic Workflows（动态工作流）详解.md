> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 又一大动作，Dynamic Workflows（动态工作流）详解
author: Ai学习的老章
date: 老章很忙老章很忙
url: https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2649014838&idx=1&sn=87580bd32f362ade129d355ee2e0f889&chksm=869f67af95a05198f8c655cba01b1c8135ee8915bd0ab52b85b81ef042e41f157102a36e6004&mpshare=1&scene=24&srcid=0601RWWCdX11XeHQVR4QW0Jb&sharer_shareinfo=33ff6d83fd19611c39359dc102178071&sharer_shareinfo_first=33ff6d83fd19611c39359dc102178071#rd
---

大家好，我是 Ai 学习的老章

Claude Code 又搞了个大动作——Dynamic Workflows（动态工作流）正式进入 Research Preview 阶段，这东西简单说就是让你一句话指挥上百个 AI Agent 干活，而且干活的过程中你的终端完全不卡，该聊天聊天，该喝茶喝茶

### 简介

先说个问题：你用 Claude Code 写代码的时候，有没有遇到过这种场景——让它审查整个项目的 API 安全性，它翻了几个文件就开始总结了；让它做个大规模迁移，搞了十几个文件就停了，因为上下文窗口塞满了？

这就是 Subagent 模式的天花板。Subagent 是 Claude 在对话里一个一个派出去的"小弟"，每个小弟干完活汇报一次，所有结果都得塞回 Claude 的脑子里。脑子就那么大，装不了太多东西

Dynamic Workflows 的思路完全不同：**它把编排计划写成一段 JavaScript 脚本，由专门的运行时来执行，中间结果存在脚本变量里，不占 Claude 的上下文窗口**。最终只有一个结果返回给你

换句话说，以前 Claude 是既当指挥官又当士兵，现在它只写作战计划，执行交给运行时引擎。这一下子就把规模打开了——单次运行最多能同时跑 16 个 Agent，总共可以调度多达 1000 个 Agent

**核心功能与特点：**

* **编排即代码**：工作流是一段 JS 脚本，Claude 根据你的需求自动生成。有循环、有条件分支、有并行控制，不是靠 Claude 一轮一轮猜该干什么
* **后台运行不阻塞**：工作流在后台跑，你的会话完全空闲，随时可以用 `/workflows` 看进度、暂停、恢复、甚至停掉某个 Agent
* **质量模式内置**：不只是"多派几个人干活"，而是可以让多个 Agent 互相对抗审查——A 找 bug，B 来验证 A 找的是不是真 bug，这种对抗验证模式是 Workflow 独有的杀手锏
* **可保存可复用**：跑完一次觉得好，按 `s` 就能保存成命令，下次直接 `/你的命令名` 就能重跑。团队共享也行，扔到 `.claude/workflows/` 目录就完事

下面这张图展示了 Workflow 从用户输入到最终输出的完整工作原理，重点看中间的"对抗验证"阶段：

Dynamic Workflow 工作原理：用户输入 → Claude 生成脚本 → 运行时后台执行多阶段 Agent 编排（扫描→对抗验证→综合输出）→ 最终结果返回主会话

### Workflow vs Subagent vs Skill vs Agent Team

这几个概念容易搞混，我给大家拉个对比表一目了然：

|  | Subagent | Skill | Workflow | Agent Team |
| --- | --- | --- | --- | --- |
| 本质 | Claude 临时派的工人 | Claude 遵循的指令 | 运行时执行的脚本 | 多个独立 Claude 会话 |
| 谁决定下一步 | Claude 逐轮判断 | Claude 按指令走 | 脚本程序控制 | 共享任务列表 + 互相通信 |
| 中间结果存在哪 | Claude 上下文窗口 | Claude 上下文窗口 | 脚本变量 | 各自独立上下文 |
| 规模 | 每轮几个 | 同 Subagent | 几十到上百个 Agent | 3-5 个协作者 |
| 能否中断恢复 | 不能 | 不能 | 同一会话内可恢复 | 有限支持 |

一张图看清四种方式的核心差异：

四种并行方式对比：Subagent 帮跑腿、Skill 按 SOP、Workflow 自动执行作战方案（规模最大、支持断点恢复）、Agent Team 组队讨论

**我的理解**：Subagent 是"帮我跑个腿"，Skill 是"按这个 SOP 来"，Workflow 是"这是整套作战方案，自动执行"，Agent Team 是"组个项目组互相讨论着干"

大多数日常任务 Subagent 就够了。但当你需要全代码库扫描、大规模迁移、多源交叉验证这种"重活"的时候，Workflow 就是唯一的选择

Subagent 和 Agent Team 架构对比图：Subagent 由主 Agent 派出去干活再汇报，Agent Team 通过共享任务列表协作，成员之间可以直接通信

### 三种触发方式

#### 方式一：关键词触发

在提示词里带上 `workflow` 这个词，Claude 就自动写脚本帮你跑

```
Run a workflow to audit every API endpoint under src/routes/ for missing auth checks
```

Claude Code 会高亮 `workflow` 这个词，然后自动生成一段工作流脚本。如果不小心触发了，按 `alt+w` 忽略就行

#### 方式二：Ultracode 模式

这是暴力模式——开了之后 Claude 会自动判断每个任务需不需要工作流，觉得需要就自动安排

```
/effort ultracode
```

Ultracode 本质上是 `xhigh` 推理等级 + 自动工作流编排。一个请求可能变成好几个 Workflow 串行：先理解代码，再改代码，最后验证。当然代价是 token 消耗大幅增加，适合那种"这次一定要搞对"的场景。用完记得 `/effort high` 切回去

#### 方式三：内置命令 /deep-research

最简单的上手方式，直接体验工作流的威力：

```
/deep-research What changed in the Node.js permission model between v20 and v22?
```

这个命令会自动多角度搜索、获取源文档、交叉验证、投票筛选，最后输出一份带引用的研究报告。没经过交叉验证的结论直接过滤掉

### 运行过程详解

工作流启动后在后台运行，你的会话完全空闲。想看进度：

```
/workflows
```

进入进度视图后，你能看到每个阶段（Phase）的 Agent 数量、token 用量和耗时。常用操作：

| 快捷键 | 功能 |
| --- | --- |
| ↑ / ↓ | 选择阶段或 Agent |
| Enter 或 → | 进入阶段详情，再进入 Agent 看它的 prompt、工具调用和结果 |
| p | 暂停/恢复运行 |
| x | 停掉选中的 Agent 或整个工作流 |
| r | 重启选中的 Agent |
| s | 保存脚本为可复用命令 |

**权限模型值得注意**：工作流里的 Agent 统一以 `acceptEdits` 模式运行，文件编辑自动批准。但 Shell 命令、WebFetch、MCP 工具如果不在你的白名单里，还是会弹权限确认。跑长任务之前，建议先把 Agent 需要用的命令加到白名单里，免得半夜跑到一半卡住等你点确认

### 保存和复用

跑完一个满意的 Workflow，按 `s` 保存：

* 保存到 `.claude/workflows/`：项目级，clone 仓库的人都能用
* 保存到 `~/.claude/workflows/`：个人级，你所有项目都能用

保存后就变成了一个命令，比如你保存了一个叫 `security-audit` 的工作流，以后直接 `/security-audit` 就能跑。如果项目级和个人级重名，项目级优先

这意味着你可以把 Code Review 的最佳实践编排成 Workflow，团队里每个人跑出来的审查标准是一样的，不再依赖 Claude 的临场发挥

### 中断和恢复

这是 Workflow 相比 Subagent 的一个硬核优势——**可以断点续跑**

如果你暂停了一个运行，已经完成的 Agent 会缓存结果，恢复后直接从断点继续，不重跑已完成的部分。在 `/workflows` 界面选中暂停的运行按 `p` 就能恢复

不过有个限制：恢复只在同一个 Claude Code 会话内有效。退出 Claude Code 再打开，上次的运行就没了，得重新来

### 约束和限制

| 约束 | 说明 |
| --- | --- |
| 运行中不能交互输入 | 除了权限确认弹窗，不能中途给 Agent 额外指令。如果需要阶段间确认，就拆成多个 Workflow |
| 脚本本身不能直接操作文件系统 | Agent 可以读写文件和跑命令，但脚本只负责编排 Agent |
| 最多 16 个并发 Agent | CPU 核心少的机器会更少 |
| 单次最多 1000 个 Agent | 防止死循环跑飞 |

### 关于费用

坦白说，Workflow 是个费 token 的功能

一次运行几十上百个 Agent，token 消耗是正常对话的很多倍

控制成本的几个建议：

* 跑之前先 `/model` 看看当前用的什么模型，不需要 Opus 级别推理的阶段可以降级到 Sonnet 甚至 Haiku
* 可以在描述任务时告诉 Claude "对于探索阶段用 Haiku，对于最终验证用 Opus"
* 随时可以从 `/workflows` 界面停掉运行，已完成的工作不会丢
* 不用了记得关，`/config` 里可以关闭 Dynamic Workflows

### 适用场景

说了这么多，什么时候该用 Workflow：

**强烈推荐：**

* 全代码库安全审计 / Bug 扫描——几百个文件逐个扫，单靠 Subagent 上下文根本装不下
* 大规模代码迁移——500 个文件的框架升级，Workflow 自动分批并行
* 需要交叉验证的研究——多角度搜索、互相验证，过滤掉不靠谱的结论
* 需要多角度评审的方案设计——让多个 Agent 各自出方案，互相对比打分

**没必要用：**

* 改个函数、修个 bug——杀鸡用牛刀
* 需要你频繁介入的迭代开发——Workflow 不支持中途交互
* 团队协作讨论型任务——这种用 Agent Team 更合适（虽然还是实验性功能）

### 如何关闭

如果你觉得这功能太费钱或者不需要，关闭方式：

* `/config` 里关闭 Dynamic Workflows 开关
* 或者在 `~/.claude/settings.json` 里加 `"disableWorkflows": true`
* 或者设环境变量 `CLAUDE_CODE_DISABLE_WORKFLOWS=1`
* 企业管理员可以通过 managed settings 全组织关闭

### 总结

Dynamic Workflows 是 Claude Code 从"AI 编程助手"向"AI 编程指挥官"进化的关键一步。它解决了 Subagent 的上下文瓶颈问题，把编排逻辑从 Claude 的脑子里搬到了可执行的脚本里，规模上限从"几个"提到了"上千个"

**优点**：规模大、可复用、支持对抗验证、后台不阻塞、可断点续跑**缺点**：费 token、不支持中途交互、仅限同一会话恢复、还是 Research Preview 阶段

我的建议是：先用 `/deep-research` 感受一下 Workflow 的威力，再根据自己的需求决定要不要在日常开发中引入。对于那些"每次 Code Review 都走同一套流程"的团队，这东西保存成命令后的复用价值是很大的

**制作不易，如果这篇文章觉得对你有用，可否点个关注。给我个三连击：点赞、转发和在看。若可以再给我加个🌟，谢谢你看我的文章，我们下篇再见！**