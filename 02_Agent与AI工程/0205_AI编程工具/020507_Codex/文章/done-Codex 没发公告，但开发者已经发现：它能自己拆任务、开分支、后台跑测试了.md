> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020507_Codex/020507_核心知识点/Codex工程使用与沙箱边界|Codex工程使用与沙箱边界]]
---
title: Codex 没发公告，但开发者已经发现：它能自己拆任务、开分支、后台跑测试了
author: WaytoAGI 实用指北
date: AGIAGI
url: https://mp.weixin.qq.com/s?__biz=MzYzOTEzODM2MA==&mid=2247489188&idx=1&sn=f655c46236eff9602558a5d30ff4e1fe&chksm=f1ddae1e979ee7ee8e4c98bd1ee1a08121928381608019c1ac0beb3f8d865349546cdb48df5b&mpshare=1&scene=24&srcid=06057LVAzltcRUo5fCLjx0Xn&sharer_shareinfo=81427b23219e0c9a1b6efbaa9e5fee7e&sharer_shareinfo_first=81427b23219e0c9a1b6efbaa9e5fee7e#rd
---

导读
开发者 Dan McAteer 在 X 上爆料：OpenAI Codex 悄悄上线了多 thread 协调能力——它可以围绕本地项目和 git worktree 同时管理多条工作线，包括独立的后台线程。更关键的是，官方 app-server 源码已经公开了完整的 Thread/Turn/Item 编排模型、subagent 父子线程、按目录过滤和权限隔离等 API。Codex 正在从"一个聊天窗口"变成"一个能调度自己工作区的 meta-agent"。

## 一条没有官宣的「杀手级功能」

6 月 2 日，开发者 Dan McAteer 在 X 上发了一条帖子，454 个点赞，4.3 万人看过：

> "Killer new Codex feature that went unannounced: Codex can now coordinate threads for local projects/worktrees. Includes separate background threads. Transforms Codex into a meta-agent that can orchestrate its own workspace."

「Codex 有个没公告的杀手级新功能：它现在可以为本地项目和 worktree 协调多个 thread，还带独立的后台 thread。这让 Codex 变成了一个能编排自己工作区的 meta-agent。」

▲ Dan McAteer 发布于 6 月 2 日，引发开发者社区讨论

注意他的用词——**went unannounced**。OpenAI 没有发博客，没有开发布会，没有推送更新日志。一个开发者在日常使用中发现了它，然后告诉了所有人。

## 先搞清楚 Codex 是什么

很多人对 Codex 的印象还停留在"网页里的 AI 编程助手"。但现在的 Codex CLI 早就跑在你的本地机器上了。

▲ npm 上的 @openai/codex 包，描述明确写着"runs locally on your computer"

npm 上 `@openai/codex` 包的描述写得很明确：**Codex CLI is a coding agent from OpenAI that runs locally on your computer。**采集时 npm latest 版本为 0.136.0，最近修改时间是 2026 年 6 月 1 日——和 Dan 发帖几乎同一天。

▲ openai/codex GitHub 仓库，Rust + TypeScript 混合架构

这意味着 Codex 能直接访问你的文件系统、终端、git 仓库——它拿到的权限和一个真人开发者坐在你电脑前差不多。这也是为什么"多线程协调"在本地场景下会特别有意义。

## 硬证据在 app-server 源码里

Dan 的帖子只是一个开发者的体验描述。但真正让这件事站得住脚的，是 OpenAI 自己开源的 `codex-rs/app-server` 代码和文档。

▲ codex-rs/app-server 的 README，定义了 Thread / Turn / Item 三层抽象

app-server 的 API 围绕三个核心概念构建：

* **Thread**

  ：用户与 Codex agent 的一次对话，每个 thread 包含多个 turn
* **Turn**

  ：一轮交互，通常从用户消息开始，以 agent 消息结束
* **Item**

  ：turn 中的具体内容——用户输入、agent 推理、shell 命令、文件编辑等

传统的 CLI agent 像什么？像一个终端里的连续会话，从头聊到尾，中间断不了线。但 app-server 把会话拆成了**可以创建、列出、读取、恢复、fork、订阅事件的独立 thread**。

这个区别很大。

## 关键能力一：每个 thread 可以绑定不同的工作目录

`turn/start` API 允许对目标 thread 发送用户输入，同时可以覆盖**model、cwd、sandbox policy、permissions、approval policy**等配置。

这里的 `cwd` 就是连接"本地项目/worktree"的关键。一个 thread 的 turn 可以在特定目录下运行；多个 thread 如果分别指向不同的本地目录或不同的 git worktree，那每条工作线就天然隔离了。

更直接的证据：`thread/list` API 支持按 `cwd` 过滤 thread，文档示例里的 cwd 数组包含两个路径——

``` /Users/me/project /Users/me/project-worktree ```

这已经把 worktree 写进了 API 的使用场景。

还有 `runtimeWorkspaceRoots`——`thread/start`、`thread/resume`、`thread/fork` 的返回值中都包含这个字段，文档说明它代表 thread-scoped 的运行时 workspace 根目录，`turn/start` 还可以用它来替换当前 thread 的 workspace roots。

**workspace 从"进程启动时的固定目录"变成了"可以随 thread 和 turn 动态配置的参数"。**

## 关键能力二：subagent 线程，父子关系已经建模

Dan 说 Codex 变成了 meta-agent。这个词背后的技术支撑是什么？

`thread/list` 文档写明：每个返回的 thread 包含 status 字段；subagent threads 还包含 `parentThreadId`（当直接控制/产生关系已知时）。`thread/read` 返回的 thread 中还可以包含 `agentNickname`、`agentRole` 等字段。

这意味着 app-server 的数据模型已经能表达"某个 thread 由另一个上级 thread 产生或管理"的关系——**线程之间有层级结构**。

想象一下这个工作流：

1. 主 thread 收到一个大任务，比如"重构认证模块" 2. 它拆出三个子 thread：一个改 API 层，一个改数据库迁移，一个写测试 3. 每个子 thread 绑定自己的 git worktree，在独立分支上工作 4. 主 thread 监控进度，等三个子 thread 都完成后汇总

这就是 Dan 说的**orchestrate its own workspace**。

## 关键能力三：后台终端和长任务

Dan 提到了**separate background threads**。在 app-server 文档中，可以找到对应的概念：`thread/backgroundTerminals/clean`——一个 experimental API，用于终止某个 thread 下所有运行中的后台终端。

此外还有 `process/spawn`、`process/outputDelta`、`process/exited` 等进程管理 API，以及 `thread/shellCommand` 用于运行 shell 命令，进度通过事件流推送。

为什么后台终端重要？因为真实的软件工程任务需要这些：

* 跑测试套件，可能要几分钟
* 启动 dev server，保持运行
* 等构建完成，观察日志
* 并行检查 lint、类型检查、安全扫描

一个只会同步执行命令的 agent 会被长任务卡住。有了 thread-scoped 的后台终端，agent 可以把长任务扔到后台，同时推进其他工作。

## 社区反应：有人早就知道，有人才刚意识到

Dan 的帖子下面，社区的反应分成了两派。

一派说这个能力其实已经存在一段时间了：

> "it's been able to do this bc of the codex app server" —— @jand\_\_

「这个能力来自 codex app server，已经有了。」

> "It was in the app-server for months." —— @metalagman\_dev

「app-server 里有好几个月了。」

另一派关心的是实际应用和边界问题：

@leetllm 说自己几个月来一直通过 worktrunk 跑并行 agent 循环，因为手动管理多个目录太麻烦了，原生 worktree 协调能省很多事。

@noxfield405 认为 thread-per-worktree 是正确的 primitive，但真正的难点在于——contexts 何时共享状态、何时隔离？git 能处理文件合并，但 agent 仍然需要追踪每条分支在做什么。

@SynabunAI 提了一个很实际的担忧：meta-agent 管理自己的子线程很有用，**但如果其中一个子线程悄悄挂了，编排器可能不知道自己在等谁。**

## 这个变化为什么值得关注

**第一，coding agent 的工作单位变了。**

过去，用户把任务交给一个 agent，一条上下文从头做到尾。想并行？自己开多个终端、多个目录、多个分支。thread/worktree 协调把工作单位从"一个任务"变成了"一个小队"：主 thread 拆任务，子 thread 在各自 workspace 里跑测试、改模块、查原因，最后汇总。

**第二，git worktree 这个老工具被重新激活了。**

worktree 本来就是为了让同一 repo 下多个工作目录并行处理不同分支而设计的。对人类开发者，它解决"我想同时改两个 feature"的问题；对 agent，它解决"多个子任务同时改代码但不会互相踩文件"的问题。Codex 把 thread 和 cwd/worktree 绑定后，就能利用 git 原生的隔离与合并机制。

**第三，"后台工作"成了 agent 体验的标配。**

同步执行命令的 agent 做不了真正的软件工程。测试要跑、server 要起、构建要等。有了后台终端，agent 可以同时推进多条线。但这也带来新问题：后台任务是否还活着？输出是否异常？谁负责读日志、谁负责 kill？

**第四，权限和安全模型被推到了台前。**

多 thread、多 worktree、多后台进程意味着更大的副作用面。app-server 文档中 `turn/start` 可以覆盖 sandbox policy、permissions、approval policy——说明 OpenAI 也把运行时边界当成 API 参数来设计。meta-agent 化不意味着"放开权限让它乱跑"，而是更需要清晰的 workspace roots、sandbox profile、approval reviewer 与审计记录。

## Codex 的方向越来越清楚了

它不只是想做一个"更聪明的聊天窗口"。从 app-server 暴露出来的 API 设计看，Codex 的方向是**本地 agent runtime + workspace 编排器**。

Thread/Turn/Item 给了对话可管理的结构；cwd 和 runtimeWorkspaceRoots 给了 workspace 可编排的锚点；subagent 和 parentThreadId 给了任务可拆分的层级；background terminals 给了长任务可运行的基础。

这些 primitives 单独看都不算"杀手级"。但当它们被组合在一起，放在一个跑在你本地机器上的 coding agent 里——Dan McAteer 说得对，这更像一个能编排自己工作区的 meta-agent。

当然，目前这还没有变成 OpenAI 的官方产品公告。app-server 的部分 API 仍标记为 experimental，全自动的 git worktree 生命周期管理（创建、合并、冲突解决）也还没有被文档完整覆盖。但方向已经摆在那里了——**coding agent 的竞争焦点，正在从"模型聪不聪明"转向"运行时能不能把项目、分支、进程和权限编排成一个可控的工作台"。**

---

— END —

— END —