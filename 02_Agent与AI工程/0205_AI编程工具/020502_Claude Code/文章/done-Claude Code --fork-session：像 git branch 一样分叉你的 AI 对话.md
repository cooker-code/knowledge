> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code --fork-session：像 git branch 一样分叉你的 AI 对话
author: 青岩码农兜底日记
date:
url: https://mp.weixin.qq.com/s?__biz=MzcwNjA1MTQ4OA==&mid=2247484547&idx=1&sn=3102454db966e3c46b90e4c2ae1f57b0&chksm=f5f4c02fb7bb8cc14e8d885f4c7a77cce800982510af555cea3778350d12dc304b05b6eacbd7&mpshare=1&scene=24&srcid=0415H8ivu6rb5NeIFd5eB777&sharer_shareinfo=ce0a3bcacb995398705450146a88fc8b&sharer_shareinfo_first=ce0a3bcacb995398705450146a88fc8b#rd
---

你加载了 40k token 的项目上下文，花了 20 分钟跟 Claude 对齐架构理解，确认了编码规范、数据库连接方式、API 约定。终于可以动手了。

然后你发现这个功能有两种实现方案，想都试试。

如果直接试方案 A，失败了——上下文已经被"污染"。Claude 记住了失败的尝试，后续推理会被误导。你想试方案 B，但起点已经不干净了。重新开一个会话？那 20 分钟的上下文对齐又要从头来。

这就是 `--fork-session` 解决的问题。

具体说：你跟 Claude 聊了 50 条消息，它已经理解了项目结构、编码风格、当前 bug 的排查进度。fork-session 把这 50 条消息完整复制一份，开出一个新会话——两边各聊各的，互不影响。

它类似 `git branch`，但分叉的不是代码，而是你和 AI 的整段对话。

但有一个关键区别必须先说清楚：**fork 只分叉，不能 merge**。没法把分支对话合并回主会话。这是它和 git branch 的核心差异——fork 是单向的探索工具，不是双向的协作工具。

2026 年 AI agent 能独立处理的任务越来越长，开发者的瓶颈已经从"AI 够不够强"转移到"我怎么同时管多个 AI 会话"。`--fork-session` 就是对这个瓶颈的直接回应。

---

## 一、fork-session 到底做了什么

### 机制

fork 复制完整会话历史，生成新的 session ID，原会话不受影响。两个会话从分叉点开始独立发展，互不干扰。

注意区分：fork 分叉的是**对话历史**（LLM 上下文），不是文件系统。两个 fork 出来的会话共享同一个工作目录——如果它们同时编辑同一个文件，会冲突。文件隔离需要另一个工具 `--worktree`，后面详述。

### 三种触发方式

**方式一：CLI 参数**

```
claude --continue --fork-session
```

从当前目录最近的会话 fork 出一个新会话。

**方式二：指定会话 fork**

```
claude --resume <session-id> --fork-session
```

从任意一个已保存的会话 fork。这是"基线会话"工作流的核心命令。

**方式三：会话内命令**

```
/branch
```

在当前会话中直接 fork（原命令 `/fork` 仍可用作别名）。fork 后 Claude 会显示原始 session ID，方便随时用 `--resume` 回到原会话。

补充一种相关操作：`Esc+Esc` 可以进入历史检查点回溯流程（rewind），回退到某个时间点重新开始。这主要用于撤销操作，不是 fork——如果目的是并行探索，优先用 `/branch` 或 `--fork-session`。

### 四个关键限制

**1. fork 不能 merge。** 没有"把分支对话合并回主会话"的能力。社区对此有大量需求（GitHub Issue #16276、#32631），但目前不支持。如果 fork 的分支里产出了好结果，需要手动把结论带回主会话，或者直接在 fork 分支里继续工作。

**2. 文件系统共享。** 两个 fork 会话编辑同一文件会冲突。解法是搭配 `--worktree`（后面详述）。

**3. 交互式权限不继承。** 主会话中通过弹窗临时 approve 的权限，fork 后需要重新授权。但通过 `settings.json` 中 `permissions.allow` 配置的权限规则不受影响，fork 会话自动继承。应对：把常用命令写入配置级权限规则，减少交互式授权依赖。

**4. 基线会话会老化。** 项目代码持续变更，基线会话中的架构理解会逐渐过时。长期项目需要定期重建基线（比如每个 sprint 结束后）。

### 一个类比

`--fork-session` ≈ `git branch`（但没有 merge），`--worktree` ≈ `git worktree`。两者组合 ≈ 完整的并行开发环境——fork 复制"思考历史"，worktree 隔离"代码现场"。

---

## 二、四个实战场景

### 场景 1：A/B 测试实现方案

需要在 React 组件中选择状态管理方案——Context 还是 Zustand。与其让 Claude 口头分析利弊，不如让两个会话各实现一个，对比代码质量和复杂度。

**操作步骤**：

1. 1. 主会话加载项目上下文（CLAUDE.md、架构文档、现有代码结构），用 Plan Mode（规划模式，按 `Shift+Tab` 切换进入，让 Claude 先输出方案再执行）对齐需求理解
2. 2. `claude --continue --fork-session` 创建分支 A
3. 3. 再从主会话 fork 一次，创建分支 B
4. 4. 分别给出实现指令，两个方案并行推进

```
# 主会话对齐需求后
claude --continue --fork-session           # 分支 A：Context 方案
claude --continue --fork-session           # 分支 B：Zustand 方案

# 强烈建议搭配 worktree 做文件隔离
git worktree add -b feature/context-approach ../worktree-branch-a
git worktree add -b feature/zustand-approach ../worktree-branch-b
```

这里有一个踩坑点：两个分支都要改 `page.tsx` 之类的同路径文件时，必须搭配 `--worktree` 做文件隔离，否则会冲突。只做只读分析或各自改不同文件，则不需要。

fork 让两个方案共享同一份需求理解，worktree 让两个方案的代码互不干扰——这是两者组合使用的典型场景。

### 场景 2：预热基线会话 + 批量特性开发

每次开新任务都要花 10 分钟给 Claude 喂项目架构、编码规范、API 文档。维护多个项目时，这个成本乘以 N。

基线会话模式的做法是：先创建一个只加载上下文、不做实际开发的"基线会话"，后续每个新任务都从这个基线 fork。

**操作步骤**：

1. 1. 创建基线会话：加载 CLAUDE.md + 架构文档 + 编码规范 + 常见陷阱
2. 2. 用 Plan Mode 确认 Claude 正确理解了项目架构（这一步很关键，确保基线质量）
3. 3. 从基线 fork 出多个特性会话：一个处理 bug 修复，一个开发新功能，一个写文档
4. 4. 每个 fork 继承完整上下文，无需重复解释项目技术栈和约束

```
# 启动新会话时命名（--name 是启动参数，不是事后改名）
claude --name "project-baseline"

# 如果要给已有会话改名，在会话内用 /rename
# /rename project-baseline

# 每个新任务从基线 fork
claude --resume project-baseline --fork-session
```

根据 BSWEN 的实测案例（docs.bswen.com），不使用 fork 时相当一部分 token 花在每次任务的上下文重建上。基线会话 + fork 将这部分开销降至接近零。

一个加载了 CLAUDE.md + 架构文档的基线会话约 30-40k token。fork 5 次后，每个分支产生 10-20k 新对话，总消耗约 200-300k token。但省下的是每个分支重复加载上下文的时间和 token。

**注意事项**：

* • session 是本地的，不能跨团队成员共享。如果团队想共享"起点上下文"，正确做法是维护好 CLAUDE.md 和 `.claude/commands/`，每人各自建基线。共享的是文档，不是 session
* • 基线会话需要定期重建。项目代码变更后，基线中的架构理解会过时。sprint 结束后重新加载最新架构变更，旧基线归档

### 场景 3：调试时保存现场

调试一个棘手 bug，已经做了大量日志分析和假设推理——排除了 A、B、C 三个假设，定位了根因，但想尝试一个激进修复，不确定是否正确。

**操作步骤**：

1. 1. 当前会话已积累了 30-50 轮对话，包含完整的排查链路（哪些假设已排除、哪些日志已分析、环境状态确认）
2. 2. `/branch` 创建分支会话
3. 3. 分支中尝试激进修复
4. 4. 如果修复失败——原会话完好无损，继续从上次排查点出发
5. 5. 如果修复成功——用原会话的 session ID 归档完整调试过程，作为知识沉淀

不 fork 的话，试了激进修复失败后，上下文已经被"污染"——Claude 记住了失败的尝试，后续推理容易被误导。fork 保证原始排查上下文干净保留，可以平行尝试多条修复路径。

这个场景是否需要 worktree 取决于修复方式：如果两个分支都可能改同一路径的文件（比如同时改同一个 handler），应该加 worktree；如果只是纯分析或明确不会改同文件（比如一个改代码、一个只看日志），可以省略。

**`/btw` 和 `/branch` 的配合**

调试过程中经常需要临时查一个旁路问题（比如"这个 retry 逻辑是怎么工作的？"）。如果直接在主会话中问，会产生上下文污染（context pollution）——无关对话内容挤占上下文窗口。

两个命令的分工清晰：

* • `/btw`（by the way 的缩写）：旁路提问，问答内容不进入主上下文。适合临时插问一个不相关的小问题，问完即弃
* • `/branch`：创建持久的分支会话，各自独立发展。用于并行探索不同方案

### 场景 4：多模块并行开发

场景 2 讲的是如何建基线，这个场景侧重 fork 多了之后怎么管。

同时维护认证模块、Kafka 消费者、API 文档三个任务，每个都从基线 fork 出来，问题是：五六个会话同时开着，怎么不搞混？

靠命名。`--name` 给会话起有意义的名字，`--resume` 按名称恢复：

```
# 命名方式管理
claude --name "feature-auth"
claude --name "bugfix-kafka"
claude --name "docs-api-v2"

# 恢复指定会话
claude --resume feature-auth
```

fork 多了之后，session ID 是一长串随机字符，靠名称管理比翻 ID 高效很多。已完成的 fork 会话及时清理，避免列表膨胀。

---

## 三、fork-session vs worktree——什么时候用哪个

这两个概念经常被混淆，但它们解决的是完全不同层面的问题。

| 维度 | --fork-session | --worktree |
| --- | --- | --- |
| 隔离层 | 对话历史（LLM 上下文） | 文件系统（git 工作树） |
| 适用场景 | 不同思路探索、A/B 对比 | 不同特性并行编码 |
| 文件冲突 | 有风险（共享工作目录） | 无（独立目录） |
| 成本 | 仅上下文 token | 磁盘空间 + 上下文 token |
| 组合使用 | 推荐搭配 worktree | 推荐搭配 fork-session |

单独用都有短板，组合才完整：

* • **单用 fork**：多个会话共享工作目录，编辑同一文件会冲突
* • **单用 worktree**：每次新会话需要从零重建上下文
* • **fork + worktree**：fork 复制"思考历史"，worktree 隔离"代码现场"，两层隔离叠加才是完整方案

判断标准：worktree 隔离代码，fork-session 隔离思路。需要改同一路径的文件时加 worktree，需要保留相同对话上下文时加 fork。

组合使用的命令：

```
# 从基线 fork + 在 worktree 中工作
claude --resume <baseline-id> --fork-session --worktree feature/auth
```

关于 worktree 的创建和隔离机制，详见本系列 #21 文章《25 分钟完成 5 套 UI 风格：AgentTeam + Git Worktree 并行开发实战》，这里不重复展开。

**避坑提示**：

* • `git worktree add` 的语法是 `git worktree add -b <branch> <path>`，注意参数顺序不要搞反
* • 新建 worktree 不自动安装依赖，fork 会话进入新 worktree 后第一步应该是 `npm install`（或对应的包管理器命令）
* • fork 出的会话容易继承主会话的工作目录路径，需在 fork 后明确告知 Claude "你的工作目录是 xxx"
* • 从混乱的调试 session 分叉会继承噪音——确保基线会话是"干净"的

---

## 四、Boris Cherny 的并行工作流与 fork-session 的乘法效应

据 METR 2026 年初发布的基准测试报告（Time Horizon TH1.1），AI agent 能独立完成的任务时间范围在持续增长——整体趋势约每 196 天翻一倍，近期加速到约 131 天翻倍。最新一代模型已经能够独立完成人类专家需要数小时才能做完的任务。当 agent 的能力持续增长，同时管理多个这样的 agent 就不再是可选项。

以下工作流整理自 Boris Cherny（Claude Code 开发者工具负责人）2026 年 1-3 月的多次公开分享，来源包括 InfoQ 报道、个人站 howborisusesclaudecode.com、海外社区帖文，不是单一采访。

### 核心理念转变

Boris 的核心观点是一个思维模式的转变——从"深度专注单线程编码"转向"管理多个并行 Agent + 快速上下文切换"。

用他自己的话说："It's not so much about deep work, it's about how good I am at context switching and jumping across multiple different contexts very quickly."

### 他的日常并行规模

* • **终端**：5 个独立 checkout 同时运行 Claude Code，iTerm2 标签编号管理，系统通知提醒需要输入
* • **Web**：额外 5-10 个 claude.ai/code 会话
* • **移动端**：通过 `--teleport` 在本地和云端之间无缝切换
* • 约 10-20% 的会话因意外情况被放弃——这是正常损耗，不是失败

### 与 fork-session 相关的关键习惯

**Plan Mode + fork 的乘法效应**

先在主会话用 Plan Mode 对齐方案，然后 fork 出执行会话。Plan Mode 确保 fork 出去的每个分支方向正确，避免浪费 token。

这里不展开 Plan Mode 的方法论（详见本系列 #15），只强调一点：Plan Mode 和 fork 是乘法关系。Plan Mode 提升单会话质量，fork 增加并行数量，两者叠加的效果远大于各自单独使用。

用他的原话说："Pour your energy into the plan so Claude can 1-shot the implementation." fork 则让这个 1-shot 可以并行地在多个方向上同时进行。

**会话命名与管理**

Boris 用 `--name` 给每个会话命名，用 shell 别名（za/zb/zc）快速切换不同 worktree 会话。他还有一个专用的 "analysis" worktree，只做只读分析——看日志、查数据，不写代码。这种"只读分支"的思路对避免上下文污染很有帮助。

**上下文压缩钩子**

通过 PostCompact hook，在上下文压缩后自动重新注入关键指令。这对长时间运行的 fork 会话尤其有用——压缩不会丢失核心约束。

### 对普通开发者的渐进路径

不需要一开始就 5 个并行。先从 2 个会话起步——一个写代码，一个查文档或调 bug。适应了再加到 3-4 个，比如主开发加文档加测试。超过 5 个时，人脑的上下文切换成本开始反噬，并行收益反而下降。

多终端管理的一个实用做法：

```
# iTerm2 标签命名示例
Tab 1: baseline-context       # 基线会话，只加载不操作
Tab 2: feature-auth-impl      # fork 出来的认证功能实现
Tab 3: bugfix-kafka-retry     # fork 出来的 Kafka 重试修复
```

这种工作方式下，编码能力本身让位于上下文切换和任务调度能力——你不再是写代码的人，而是决定哪个 Agent 该往哪个方向走的人。

### 横向对比：并行能力不只 Claude Code 有

截至 2026 年 3 月，几乎所有主流 AI 编码工具都在解决并行问题。Claude Code 在"会话分叉"维度上提供了 `/branch` 和 `--fork-session` 这组明确入口，其他工具（Cursor、Copilot、Windsurf）更侧重多 Agent 并行执行而非会话历史分叉。但 Claude Code 的短板也明显：没有 merge back，没有分支树导航，分支管理全靠手动命名。

---

## 五、配额与成本现实

并行会话意味着 token 消耗成倍增加。这不是免费午餐，需要算账。

### 订阅 vs API：成本结构决定并行策略

| 计划 | 月费 | 适合 |
| --- | --- | --- |
| Max 5x | $100/月 | 中度并行（2-3 个会话） |
| Max 20x | $200/月 | 重度并行（5+ 个会话） |
| API | 按量付费 | 可控但成本敏感 |

重度使用场景下，API 按量计费的月成本可达数千美元，而 Max 订阅是固定月费。对于日常高频使用 Claude Code 的开发者，Max 计划通常比 API 按量计费更经济。

这意味着 Max 用户并行 fork 的边际成本远低于 API 用户。fork-session 作为日常实践，在 Max 计划下才真正可持续。

### fork 的 token 经济学

fork 不需要重新手工喂上下文，但后续每个分支的推理仍受上下文大小影响。省与花要分开看。

**省的部分**：前面提到过，相当一部分 token 花在重复加载上下文上。fork 直接继承已有历史，省下了重复读文件、重复分析架构、重复对齐需求的开销。

**花的部分**：每个 fork 分支后续的对话独立计费。多个并行会话的总 token 消耗会成倍增长，因为每个会话维护独立的 context window。extended thinking（深度思考）是隐性成本驱动因素——多数编码任务不需要最高思考深度，可通过 `/effort`、`/model` 或 `/config` 下调思考强度来控费。

**实际消耗**：据 Anthropic 官方成本文档（code.claude.com/docs/en/costs），平均开发者日均消耗 $6，90% 的开发者低于 $12/天。密集并行时这个数字会翻倍甚至更多。如果每个 fork 只做短任务（10-20 轮），fork 比从零开始明显更经济；但如果 fork 出去的分支都跑成了长对话，总消耗会快速膨胀。

### 成本控制实践

1. 1. **基线会话精简**：只加载必要信息，不要把所有文档都塞进去。一个精简基线 30k token，一个臃肿基线可能 80k+
2. 2. **Plan Mode 先行**：确认方案再 fork，避免 fork 出去后发现方向错了白费 token
3. 3. **已完成的 fork 及时结束**：不要让废弃会话持续占用上下文
4. 4. **CLAUDE.md 设置输出约束**：如 `Be concise in output but thorough in reasoning`，全局控制输出 token
5. 5. **监控消耗**：订阅用户用 `/stats` 查看用量，API 用户用 `/cost` 或账单页面
6. 6. **混合模型策略**：复杂推理任务用高端模型，重复性工作用轻量模型，通过 effort level 分级控制

---

## 从单线程到多线程

不 fork 的代价是：试错一次，上下文就脏了，要么带着错误继续，要么花 20 分钟重建。

`--fork-session` 把试错成本降到趋近于零。一条命令，当前会话一分为二。方案 A 和方案 B 并行推进，哪个好用哪个。调试时想试激进修复，fork 一个出去，原始现场毫发无损。

起步只需要三步：

1. 1. 找一个已有的长会话（积累了项目上下文的那种）
2. 2. 执行 `/branch` 或 `claude --continue --fork-session`
3. 3. 给新会话命名（`/rename experiment-zustand`），只在这个分支里测试一个备选方案

```
# fork 出新会话，尝试不同方案
claude --continue --fork-session

# 随时回到原会话继续（指定会话名或 ID）
claude --resume <原会话名或session-id>
```

---

**参考资料**

* • Claude Code 官方文档 Sessions 管理：https://code.claude.com/docs/en/how-claude-code-works
* • Claude Code 官方成本文档：https://code.claude.com/docs/en/costs
* • Boris Cherny 完整工作流：https://howborisusesclaudecode.com
* • Boris Cherny InfoQ 报道（2026.01）：https://www.infoq.com/news/2026/01/claude-code-creator-workflow
* • BSWEN Session Forking 实战指南：https://docs.bswen.com/blog/2026-03-22-claude-code-session-forking
* • Trigger.dev 10 Claude Code Tips：https://trigger.dev/blog/10-claude-code-tips-you-did-not-know
* • Richard Hightower Context Hygiene Toolkit：https://medium.com/@richardhightower
* • METR Time Horizon 基准测试（2026.01）：https://metr.org/blog/2026-1-29-time-horizon-1-1
* • Claude Code 成本分析：https://www.ksred.com/claude-code-pricing-guide
* • AI Engineering Report Token 成本分析：https://www.aiengineering.report/p/the-hidden-costs-of-claude-code-token
* • 本系列 #15：先计划，再编码
* • 本系列 #21：AgentTeam + Git Worktree 并行开发实战