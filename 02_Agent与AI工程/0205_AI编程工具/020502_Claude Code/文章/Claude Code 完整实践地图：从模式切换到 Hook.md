---
title: Claude Code 完整实践地图：从模式切换到 Hook
author: AI智友记
date: 维小灿维小灿
url: https://mp.weixin.qq.com/s?__biz=Mzk3NTA1NDQ2OA==&mid=2247483983&idx=1&sn=d0ccac549eb473cedc220f6be4ff75b4&chksm=c54c9ed90ccb498d9e2cd5b412b82e01ffcca384ae1e64f5e29db2647c1e20f9fc0242a5a8e9&mpshare=1&scene=24&srcid=0602ky0AA5eR54dNTYjTWjDi&sharer_shareinfo=0f849a6a5c1c1bb73a9047105f01ce93&sharer_shareinfo_first=0f849a6a5c1c1bb73a9047105f01ce93#rd
---

这是我看完 Anthropic 官方 Claude Code 101 课程后整理的一份实践地图。

之前写过的 learn-claude-code 系列是从代码逻辑层面拆解了 Claude Code 的内部机制，回答的是"它内部怎么运转"。这篇换一个角度——**从使用者出发，介绍 Claude Code 有哪些功能值得关注，这些功能什么时候该用、怎么用。**

## 一、模式与工作流

### 四种权限模式

Claude Code 有四种权限模式，决定了模型在改文件、跑命令前要不要寻求同意。按 `Shift+Tab` 可以在不同模式之间循环切换。

* Default（默认）：写文件、跑命令都弹确认，只读操作不弹。安全但啰嗦。
* Accept Edits（接受编辑）：文件编辑免确认，Bash 命令仍然弹。文件放开，shell 不放开。
* Plan Mode（计划模式）：只读。只能看代码搜信息，不能改文件不能执行命令，专门用来先出方案再动手。
* Auto Mode（自动模式）：所有操作免确认。不过有一个独立的分类器模型会在每个动作前审查，拦截超出请求范围或高危操作（如 curl | bash、force push、生产环境部署等）。Auto 不在默认循环里，需要单独启用才会出现。

循环顺序：Default → Accept Edits → Plan →（Auto，如已启用）→ 回到 Default。

什么时候用哪个？我的判断标准是**任务复杂度**和**操作风险**：

* Default：不熟的项目、敏感操作、第一次跑某个新流程
* Accept Edits：方向已确认，让 Claude 批量改文件，事后用 git diff 复盘
* Plan Mode：复杂任务开始前，先让它出方案
* Auto：完全信任的循环——跑测试、装依赖、批量生成

### 一种最佳实践：Explore → Plan → Code → Commit

课程给出了一套推荐的工作流：**Explore**（探索代码库）→ **Plan**（出执行计划）→ **Code**（实现）→ **Commit**（审查提交）。

四步里，**前两步最容易被跳过，但非常重要**。

如果跳过前两步直接让 Claude 写代码，很容易出现方向不对、改了三轮越改越乱的情况。原因很简单：**Claude 不了解项目，按自己的假设动手，假设错了后面全是纠偏成本。**

前两步有几种做法：用 Plan Mode 让 Claude 只读地翻代码、出方案；也可以用 Explore subagent 先做一轮代码库调研，再进入规划。不管哪种方式，核心是一样的——**先摸清情况再动手**。

计划阶段改一句话，比代码阶段回滚五个文件省十倍时间。

**一个实践小tip**：面对更复杂的系统，可以在 Plan 之前再加一层需求讨论——把做什么、给谁用、解决什么问题梳理清楚后再进入开发。

## 二、上下文管理与 Subagent

### Context 是什么

Context 就是模型当前能看到的全部内容——对话历史、读过的文件、工具调用的返回结果，全部装在一个有限的窗口里。每发一条消息、每读一个文件、每跑一次工具调用，都在往这个窗口里堆。塞满之后 Claude Code 会自动压缩——把重要信息归纳成摘要、删掉无用的工具返回结果来腾空间。**压缩不是无损的，细节可能丢失。**

三个管理命令：

* /context：查看当前上下文占用情况——总量多大、哪些类别占得最多
* /compact：手动触发压缩。保留当前工作的摘要，释放空间。适合在同一个任务中上下文快满了但还要继续的时候用
* /clear：完全清空，从零开始。适合切换到新任务，不想让上一轮对话的内容影响新任务的判断

`/compact` 和 `/clear` 的核心区别：**一个保留记忆继续干，一个切断历史重新来**。

一个反直觉的点：**详细的 prompt 比模糊的 prompt 更省上下文**。模糊的 prompt 看起来字数少，但 Claude 不得不自己探索代码库、自己推理补全缺失信息——这些探索过程全部消耗上下文，远比一段写清楚的 prompt 占得多。

### Subagent：用隔离的上下文分担工作

跟 Claude 协作中经常会遇到这种情况：在一个大任务中需要完成一个小任务（比如查个接口在哪、确认一下某个依赖的用法），这些小任务的探索过程会给主任务带来大量关系不大的上下文，造成上下文污染。

Subagent 解决的就是这个问题。Claude 可以创建一个或多个独立的子 agent，让它们在各自的上下文窗口里干活。干完之后只把摘要返回给主 agent，中间过程不污染主上下文。

有两种使用方式：

**临时创建**：直接在对话中说"用一个 subagent 帮我查一下认证接口在哪个文件"，Claude 会现场创建一个一次性的子 agent，用完即弃。适合一次性的探索、查询。

**自定义配置**：通过 `/agents` 命令创建持久化的 subagent——给它一个名字、一段系统 prompt、指定它能用哪些工具。适合反复要做的专业化任务，比如 code review、安全审查。

Claude Code 内置了 5 个 subagent，在 `/agents` 的 Library 标签里能看到：

* Explore（用 Haiku）：只读探索代码库
* Plan（继承主对话模型）：Plan Mode 下做代码库调研
* General-purpose（继承主对话模型）：需要同时探索 + 修改的复杂任务
* claude-code-guide（用 Haiku）：回答关于 Claude Code 本身的问题
* statusline-setup（用 Sonnet）：配置状态栏

我试着建了一个 code-reviewer subagent，过程很直接：`/agents` → Create new agent → 选 scope → 描述用途 → 选工具权限 → 保存。**一个关键设置：reviewer 的工具权限应该限制为只读——它的工作是指出问题，不是顺手改代码。**

## 三、经验沉淀的三种机制

跟 Claude 协作几次之后会发现，有些话在反复说——比如"这个项目用 Python 3.11"、"测试命令是 pytest""commit 信息用中文"。这些重复的内容，该有个地方固定下来。

Claude Code 提供了三种沉淀机制，各自解决不同层面的问题。

### CLAUDE.md：项目说明书

CLAUDE.md 是一个 markdown 文件，Claude Code 每次开 session 都会自动读取，把内容加到上下文的最前面。

它的作用是**让 Claude 每次开工都知道这个项目的基本情况**——用什么技术栈、怎么跑测试、代码风格有什么约定。没有这个文件，Claude 每次都得重新摸索，浪费上下文也容易猜错。

一个典型的 CLAUDE.md 大概长这样：

```
```
# Project  
Python 3.11 数据分析项目，pandas + pyarrow  
  
# Commands  
- 跑测试：pytest  
- 跑 pipeline：python -m pipeline.run  
  
# Code Style  
- 路径用 pathlib，不用字符串拼接  
- 数据假设要显式校验，不要静默通过 NaN
```
```

**CLAUDE.md 有两个层级**：项目根目录的是项目级，跟团队共享、签入 git；`~/.claude/CLAUDE.md` 是用户级，跨所有项目生效，放个人偏好。两者叠加生效。

一个实用的建议：**不要一开始就写一份完美的 CLAUDE.md**。先空着跑几次，看 Claude 在哪些地方反复出错、需要反复纠正，把那些点写进去。先有问题，再写规则——这样写出来的每一条都是有用的。准备好了之后跑 `/init`，Claude 会扫描项目帮生成一个起步版本，在上面改就行。

### Auto Memory：Claude 自己写的笔记

CLAUDE.md 是用户写给 Claude 看的规则。Auto Memory 反过来——**是 Claude 在协作过程中自己记下来的笔记**。

比如我纠正了一次"入口命令不是 `python main.py`，是 `python -m pipeline.run`"，Claude 会判断这条信息未来有用，自动写到一个叫 MEMORY.md 的文件里。下次开 session，这条笔记会被加载，不用再纠正。

用 `/memory` 命令可以查看当前的记忆系统状态：

这个界面里能看到四样东西：

* Auto-memory 开关：是否开启自动记忆
* Project memory：项目级 CLAUDE.md（我写的规则）
* User memory：用户级 CLAUDE.md（我的个人偏好）
* Open auto-memory folder：Claude 自己记的笔记存放目录

### Skill：把方法论封装成可触发的能力

CLAUDE.md 和 Auto Memory 解决的是"背景信息"——项目是什么、有什么约定。Skill 解决的是另一个问题：**怎么做某件事**。

Skill 的核心是一个 SKILL.md 文件，但它可以是一个完整的目录——包含多个说明文件、示例、甚至可执行脚本。Claude 根据 SKILL.md 里的 description 判断"现在该用这个 Skill 了"，自动加载；不用的时候不占上下文。

**这是 Skill 和 CLAUDE.md 最关键的区别**：CLAUDE.md 每次 session 都全量加载，常驻上下文；Skill 按需触发，不调用就不占空间。所以 CLAUDE.md 要短——因为它每次都在；Skill 可以详细——因为只在需要时才出现。

三种沉淀机制的关系：

* CLAUDE.md：用户的规则，常驻加载，要短
* Auto Memory：Claude 写的笔记，自动积累，要定期检查
* Skill：封装好的方法论，按需触发，可以详细

## 四、MCP：接入外部工具

项目相关的信息不只在代码里——还散落在数据库、Issue 系统、文档站、笔记工具等代码库之外的位置。**MCP（Model Context Protocol）是 Claude Code 接入这些外部工具和数据源的方式。**

比如团队用 Linear 管项目，加一个 Linear MCP server，Claude 就能直接查 issue 详情；需要查某个依赖的最新文档，接一个文档类 MCP server 就行。

### 添加和管理

用 `claude mcp add` 命令添加 MCP server，用 `/mcp` 查看当前连接了哪些、各自状态如何。

MCP server 有三种作用域：

* Local：只在当前项目生效，只有自己能用
* User：跨所有项目生效，只有自己能用
* Project：写在 .mcp.json 文件里签入 git，团队所有人自动共享

### MCP 的上下文代价

MCP 有一个重要的代价：**MCP server 的工具定义会占用上下文空间**。MCP server 越多，占用越大，不是装得越多越好。

一个实用的判断标准：**如果某件事有 CLI 工具能做（比如 `gh` 做 GitHub 操作、`aws` 做 AWS 操作），优先用 CLI 而不是装 MCP**。CLI 通过 Bash 调用，不往上下文里塞工具定义，更省空间。

同样的逻辑，如果需求是"教 Claude 怎么做某件事"而不是"接入外部数据"，用 Skill 比用 MCP 更合适——Skill 按需加载，MCP 常驻。

## 五、Hook：确定性保证

前面介绍的所有功能——CLAUDE.md、Skill、Subagent——都是在"告诉 Claude 该怎么做"。Claude 大部分时候会遵守，但**不是百分百的**。

课程里举了一个很直观的例子：在 CLAUDE.md 里写"每次改完文件跑一下 Prettier 格式化"，大部分时候 Claude 会照做，但偶尔就是会忘。

Hook 解决的就是这个问题。**Hook 是 Claude Code 中的一种确定性机制——不依赖 Claude 的判断，机制保证每次都执行。**

### 怎么工作

Hook 挂在 Claude Code 生命周期的特定事件上，事件发生就自动跑。Claude Code 的生命周期事件有很多个且还在持续增加，常用的有以下几个：

* PreToolUse：工具调用之前。可以用来拦截危险操作——比如阻止写入生产目录、阻止 rm -rf。返回 exit code 2 时阻止该工具调用，并把 stderr 信息反馈给 Claude
* PostToolUse：工具调用之后。可以用来自动善后（比如改完文件自动跑格式化），也可以反馈信息给 Claude（比如告诉它"刚改的文件 lint 没过，请修一下"）
* UserPromptSubmit：提交 prompt 后、Claude 处理前
* Stop：Claude 准备结束响应时。可以用 exit code 2 强制 Claude 继续工作而不是结束
* Notification：Claude 生成通知时触发，常用来把通知路由到 Slack、桌面提醒或日志系统

### 什么时候该用 Hook

判断标准有两个：

**漏一次的代价大吗？** 漏一次格式化——下次补上就行，CLAUDE.md 提醒够了。漏一次提交了 `.env`——代价大，需要机制保证。

**能用静态规则描述吗？** 如果只是"禁止写某个路径"、"禁止跑某个命令"，`settings.json` 里加一条 `permissions.deny` 就够了，不用写 Hook。Hook 的真正价值是做带上下文的判断——比如"Bash 命令里包含 `rm -rf` 或 `DROP TABLE` 就拒绝"、"改完 schema 文件就自动跑迁移测试"，这些用 deny 规则的静态匹配做不到。

一句话：**代价小用 CLAUDE.md，代价大且能写死规则用 deny，代价大且要看上下文用 Hook。**

Hook 通过 `/hooks` 命令配置，也可以直接编辑 `settings.json`。写在项目的 `.claude/settings.json` 里可以签入 git，团队共享。

## 核心功能概览

**模式与工作流**：

* Shift+Tab 切换四种权限模式，复杂任务先 Plan 再动手
* 推荐流程：Explore → Plan → Code → Commit，需求写清楚再开始

**上下文管理**：

* /context 看占用，/compact 压缩继续干，/clear 切任务时清空
* 需要探索但不想污染主上下文的任务，丢给 Subagent

**沉淀经验**：

* CLAUDE.md = 项目规则，常驻加载
* Auto Memory = Claude 自己记的笔记，定期检查
* Skill = 封装好的方法论，按需触发，可以详细

**扩展与兜底**：

* MCP 接入外部工具，但会占上下文；能用 CLI 就别装 MCP
* 必须 100% 执行的事，不写 prompt，写 Hook