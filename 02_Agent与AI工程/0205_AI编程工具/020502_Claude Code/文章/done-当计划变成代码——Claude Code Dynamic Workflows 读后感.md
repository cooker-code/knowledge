> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 当计划变成代码——Claude Code Dynamic Workflows 读后感
author: Lx私房菜
date: NTLxNTLx
url: https://mp.weixin.qq.com/s?__biz=MzI3MzIwNDk3Nw==&mid=2649470842&idx=1&sn=0726017c074160af72b0277a6884d1b9&chksm=f2d5a66839f56c06f3c9d6ad285964c760ef6c00f5c5760a0d34e81d805e9b4aff820dd975c0&mpshare=1&scene=24&srcid=0602yYYqvQoSiGSm0bzJL2AQ&sharer_shareinfo=774a325f0aff66f61c5dc341ca3f76b5&sharer_shareinfo_first=774a325f0aff66f61c5dc341ca3f76b5#rd
---

5 月 28 号，Anthropic 在发布 Opus 4.8 的同时，悄悄放出了一个叫 Dynamic Workflows 的功能。研究预览，没有大肆宣传。

紧接着，Bun 的作者 Jarred Sumner 披露了一组数字：用 dynamic workflows，**11 天**，把 Bun 从 Zig 重写成了 Rust。**75 万行代码，99.8% 的测试通过**。

多数人看到的第一反应是"快"。我觉得这组数字指向速度之外——**AI 开始写"指挥其他 AI 写代码"的代码**。

Claude Code 的官方文档里有一张表，比较了 Subagents、Skills 和 Workflows。乍一看三者都是"让 AI 做多步任务"，区别只是调度方式不同。

但有一行值得盯着看：**Where intermediate results live**。

Subagents 的中间结果在 Claude 的上下文窗口里。Skills 也是。唯独 Workflow，中间结果在**脚本变量**里。

一行表格的差异，其实是整个功能的内核。

以前用 Claude Code，所有计划都住在对话里。Claude 逐轮决定下一步做什么，推理过程塞进上下文窗口。窗口满了就压缩，会话断了就丢失。**计划存在于一个不断蒸发的容器中**。

Workflow 把计划搬到了 JavaScript 脚本里。`phase()` 定义阶段，`agent()` 派生 worker，`schema` 约束输出格式，`return` 聚合结果。编排逻辑变成了可阅读、可复用的代码。

熟悉吗？十五年前 DevOps 走过同一条路。

那时候运维工程师 SSH 到服务器手动敲命令，部署计划住在脑子里和 shell history 里。然后 Ansible 来了，把操作变成 playbook；Terraform 来了，把基础设施变成声明式配置。每一步演进，计划都从一个隐式载体搬到了一个显式的、可审计的载体。

> **Dynamic Workflows 对 AI agent 做的事，和 Terraform 对基础设施做的事，是同构的。** 从手动 SSH 到 Infrastructure as Code 花了十年。从"跟 AI 对话"到"Orchestration as Code"，压缩到了几年。

## 从管 Agent 到造编排器

徐靖峰画过一个 **AI 编程的 8 阶段模型**。前 7 个阶段描述的是人和 agent 之间越来越紧密的耦合：

1. 1. 不用 AI
2. 2. IDE 里有个 Agent
3. 3. YOLO 模式关掉权限确认
4. 4. IDE 内 Agent 全屏
5. 5. CLI 单 Agent
6. 6. CLI 多 Agent 并行（日常 3-5 个实例）
7. 7. 10+ Agent 手动管理——触及人类极限

第 8 阶段只有一句话：**构建自己的编排器**。

第 7 到第 8 之间有一条断层线。前面所有阶段，人还是调度者，区别只是管的 agent 越来越多；到第 8 阶段，人退出了调度循环，转而设计调度循环本身。量变积累到这里，变成了质变。

**Dynamic Workflows 就是 Anthropic 给这条断层线填上的答案。**

有意思的是，王巍（onevcat）反编译 Claude Code CLI 后发现，`WORKFLOW_SCRIPTS` 这个 feature flag 早就藏在二进制里了。在 v2.1.147 中短暂出现又被移除，直到 v2.1.154 才正式发布。背后是 30 多个 feature flag，由 GrowthBook 做 A/B 灰度控制。

一个功能在代码里躺了这么久，经历了出现、撤回、再发布的完整循环。说明 Workflow 不是临时起意。Anthropic 很早就判断出单 agent 对话会触及天花板，一直在等模型能力跟上来。

## 75 万行的证据

Bun 是一个 JavaScript 运行时，速度极快，最初用 Zig 写。Sumner 决定把它移植到 Rust。

用 dynamic workflows 做这件事的方式，值得拆开看：

* • **映射**：一个 workflow 扫描 Zig 代码库里每个 struct 的每个字段，确定它在 Rust 中正确的生命周期语义。干的不是写代码的活，是架构决策。
* • **执行**：几百个 agent 并行工作，每个把一个 `.zig` 文件翻译成行为等价的 `.rs` 文件。每个文件配两个独立的 reviewer agent，交叉检查。
* • **修复循环**：驱动构建和测试，发现失败就修，修了再跑，直到全部通过。

产出：**75 万行 Rust，99.8% 测试通过，首次提交到合并 11 天**。

一个 AI 系统同时调度数百个专职 agent，各自负责一小块，互相审查，迭代修复，最终合力完成了一次完整的语言移植。说"AI 写代码快"把这个过程说小了。

历史上，大规模代码移植是出了名的苦差。90 年代 COBOL 到 Java，2000 年代 VB6 到 .NET，每一次都耗时数年、耗资数亿。Dynamic workflows 给出了一个新范式：**移植的不是代码，是意图**。旧代码描述"做什么"，workflow 理解意图后用新语言重新表达。

## 花钱买信任

官方文档反复出现一个警告：*workflows can use substantially more tokens*。几乎每提到一次好处，就配一次成本提醒。

这种张力本身就值得琢磨。

如果 workflow 只是更快地产出，多花 token 就是单纯的资源消耗，没什么好讨论的。但官方说结果**更可信**。这就有意思了。

可信的机制是**对抗性审查**。一个 workflow 里，多个独立 agent 从不同角度解决同一问题，同时还有专门的 agent 试图推翻其他 agent 的结论。运行迭代直到答案收敛，只有经过交叉验证的结论才呈现给用户。

并行是同时做多件事。对抗性审查是同时做多件事，然后让它们互相拆台，活下来的才是可信的。两件事看起来像，实际上差了很远。

传统代码审查是一个人的视角加一个大脑。Workflow 的审查是多个独立视角加互相质疑。AI 超元域在实测中发现了这种模式的边界：用 Workflow 做代码审查时，它能快速发现很多高价值问题，但也可能把某些问题严重级别判断过高。一个看似 SQL 注入的 finding，人工复核后发现上游已经有约束。Workflow 能提升覆盖面，但不能替代工程判断。

所以 token 消耗和结果可信度之间，其实没有张力。正是因为花了更多 token 做交叉验证，结果才更可信。高风险场景下，这个溢价值得付。

话说回来，这也暗示了一个不那么舒服的推论：**便宜的工作流，可能就是不可信的工作流**。

## 然后呢

当编排逻辑变成代码，可以版本控制、可以共享，共享的东西就变了。以前大家共享 prompt。现在可以共享完整的多 agent 工作流脚本。

AI 超元域的实测文章末尾列了一张清单：未来可能出现的 workflow 类型——代码审查、安全审计、文档生成、Prompt eval、Release gate。每一项都让我想到 Docker Hub 刚上线时的场景：有人把多年积累的运维经验打包成镜像，其他人 pull 下来就能用。

> **Workflow 脚本就是 AI agent 时代的 Dockerfile。**

但这也意味着一个新的门槛。Dockerfile 要求你理解容器化；workflow 脚本要求你理解多 agent 编排。阶段怎么划分，agent 之间怎么分工，输出 schema 怎么设计，每一个都是工程决策。AI 超元域的实测里提到，schema 字段命名不当（比如叫 `ok`）就能导致 critic agent 误判，让整个脚本断言失败。编排逻辑变成代码的好处是可复用，代价是你得真的会写编排逻辑。

当然，现在还是早期。研究预览，token 消耗高，feature flag 灰度控制。Anthropic 的谨慎说明他们知道这个能力有多强，强到需要慢慢放出来。

对大多数开发者来说，短期内不会天天写 workflow 脚本。但有一个变化已经在发生：你开始习惯把任务拆成可以并行的独立子任务，开始思考哪些步骤该让 agent 互相检查而不是只跑一遍。这种思维方式本身，就是 workflow 思维的前奏。

但方向已经清楚了。**AI 编程工具的竞争，正在从"谁的单次回答更聪明"转向"谁能更可靠地编排大规模执行"。计划变成代码之后，一切都会改变。**

> *你在用 Claude Code 时遇到过"单 agent 搞不定"的场景吗？你觉得 dynamic workflows 最先能解决你手头的什么问题？*

## 原文参考

> Anthropic Blog — "Introducing Dynamic Workflows in Claude Code"
> https://claude.com/blog/introducing-dynamic-workflows-in-claude-code
>
> Claude Code Documentation — "Workflows"
> https://code.claude.com/docs/en/workflows