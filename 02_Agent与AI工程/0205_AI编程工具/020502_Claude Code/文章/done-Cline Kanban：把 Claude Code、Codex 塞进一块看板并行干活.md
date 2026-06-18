> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Cline Kanban：把 Claude Code、Codex 塞进一块看板并行干活
author: 向量猫
date: 向量猫向量猫
url: https://mp.weixin.qq.com/s?__biz=MzYzNzg4ODc5NQ==&mid=2247485808&idx=1&sn=1abffc1ec323423989aa46ac20a03ea2&chksm=f1ce448ea5fcb050d5e499e42397a53200623bde835713857af186be2e281dd72dc53f10d3ca&mpshare=1&scene=24&srcid=0527YEyPd7zAeJUDhcAuVCV0&sharer_shareinfo=9eac9394943c90cac16348a8c2959b32&sharer_shareinfo_first=9eac9394943c90cac16348a8c2959b32#rd
---

> 一块本地看板，把 Claude Code、Codex、Cline 全塞进去并行干活，每张卡自带 git worktree。

## 早上 10 点，你的屏幕上有 20 个终端窗口

Cline 团队在介绍 Kanban 的文章里描述过一个典型场景。

早上你心情不错，同时启动了几个 agent。到 10:30，屏幕上躺着 20 个终端窗口。一个 agent 在等你确认要不要 force push，你十分钟前忘了它。另一个 40 分钟前就跑完了，没人收。第三个三个 commit 之前已经走偏，你才发现。

每次 alt-tab 切窗口，要花 30 秒到 1 分钟"重新载入上下文"。有人把这种切窗口本身的脑力消耗叫作 **context reload tax**——不是写代码，但每天反复几十次就吃掉一两个小时。

2026 年的编码瓶颈早就不是 AI 能力，是开发者的注意力。模型已经能写出能跑的代码，问题在于一个人最多能并行 review 几个 agent 的输出。超过那个数，开始漏 review，开始误 merge，开始 revert，效率反而比单线程更低。

Cline Kanban 就是冲这个问题来的。一个本地 app，把每个 agent 任务变成看板上的一张卡。哪些在跑、哪些卡住、哪些跑完了，一眼扫完。

它不绑定 Cline 自家 agent。Claude Code、Codex、OpenCode、Cline CLI 都能跑，混着跑也行。按官方博客 Announcing Kanban 的说法，它的定位是**"在 agent 之上的协调层"**，不抢 agent 市场，而是做调度。

## 一行 npx 就能上手，但有几个坑

启动方式简单到没什么可说：

```
npx kanban
```

或者全局装一份：

```
npm i -g kanban kanban
```

进去以后做三件事就能跑起来。建任务卡，可以自己写，也可以让 sidebar 里的 agent 帮你拆。⌘+click 把卡片连起来，建立依赖链——前一张做完自动启动下一张。点 play。

点 play 这一刻，Cline 在底下做的事比文档里写的多。我去翻了 DeepWiki（Cline 仓库的 AI 生成源码索引站，可以按目录、文件直接读分析）上的 Task Worktree Lifecycle 一页，源码层的逻辑展开后比官方博客复杂不少。

它会用 `git worktree add` 给这个任务建一个独立目录。每个 agent 都在自己的分支里干活，互不污染。主仓还停在原本的分支上不动，worktree 里跑的 agent 想怎么折腾就怎么折腾。前置条件是主仓必须至少有一个 initial commit，否则 worktree 加不上——这是 git 本身的限制，Cline 也帮不了你。

加 worktree 这步是**串行的**。DeepWiki 索引把 task-worktree.ts 中的这段逻辑描述为"基于 .git 目录的 setup lock"（具体命名以源码为准）。即使你同时点 10 个 play，它们也会排队进入这个 setup 阶段，避免 git 内部数据结构被并发破坏。

为什么要这么做？git 的 worktree 命令本身不是并发安全的——同时跑两个 `git worktree add` 有概率把 `.git/worktrees/` 目录搞坏。Cline 用文件锁兜底，等于替你扫了一个常见的坑。

submodule 项目还有额外照顾。Cline 会自动跑 submodule 初始化，省你一步。对那种主仓 + 十几个 submodule 的大单体仓库来说，省了不少手动操作。

## node\_modules 不会重复装很多次

这是 Cline Kanban 区别于"普通 worktree 工作流"的关键细节，对 Node 项目重要到爆。

普通的 git worktree 是隔离的工作目录。每开一个 worktree，理论上 node\_modules、.next、.cache 这些 gitignored 的目录都要重新装一遍。你跑 10 个 agent，就要 npm install 十次。一个中型 Next.js 项目 node\_modules 两三个 GB，十个并行任务就是 30 GB 起步。磁盘秒满。

Cline 的做法是：扫描主仓的 `.gitignore`，把里面声明的路径用 symlink 链回主仓库。每个 worktree 里看到的 node\_modules，实际上是主仓 node\_modules 的一个软链接。**装一次，所有并行任务共享。**这个机制不只对 node\_modules 有效，凡是写在 .gitignore 里的目录（.next、.turbo、.cache、dist 之类）都会被同样处理。

> Gitignored files like node\_modules are symlinked from your main repo into the worktree, avoiding slow reinstalls for each task. 
> —— Cline 官方文档

图：N 个 worktree 共享主仓 node\_modules，N 取决于你同时启动的并行任务数

但这里有个工程细节，藏在源码里。DeepWiki 索引中提到 task-worktree.ts 同目录下还有一个专门处理 Turbopack 项目的文件（DeepWiki 标注为 task-worktree-turbopack.ts，具体文件名以仓库源码为准）。检测到 Turbopack 时，**部分 node\_modules 路径会被主动跳过，不做 symlink**。

为什么要单独处理？Turbopack 自己有一套 file-watching 和 caching 机制，DeepWiki 的措辞是"避免与 Turbopack 的 native file-watching 和 caching 冲突"。如果几个 worktree 都 symlink 到同一份 node\_modules，watcher 行为可能互相影响。Cline 宁可让这些路径在 worktree 里走原生流程，也不让 watcher 出问题。

Windows 用户有个 corner case：symlink 在 Windows 上需要管理员权限或开发者模式。DeepWiki 描述 Cline 检测到 symlink 失败时不会让任务死掉，而是走"best-effort 跳过"——任务继续跑，但 Windows 用户拿不到 node\_modules 共享的好处，每个 worktree 还是要重新装一遍。

这是工程团队在意细节的样子。能把 Turbopack 和 Windows 这两个 corner case 都处理掉，意味着他们真的拿这个工具跑过自己的项目。

## 删任务之前，Trash 里有一份完整的 patch

读 DeepWiki 那一页时，我发现了一个所有营销文案都没提的安全网。

任务被你拖进 Trash 不会立刻消失。DeepWiki 索引把删除流程描述为"任务删除前的现场快照"（内部函数名以源码为准）：先跑一次 `git diff --binary HEAD` 抓所有 tracked 的改动，再遍历 untracked 文件、和空 blob 做 diff，把两部分合并成一个完整的 `.patch` 文件，存到本地：

```
~/.cline/kanban/trashed-task-patches/
```

文件名带 task ID 和当时的 HEAD commit hash，方便后续定位恢复。想要恢复就是普通的 `git apply` 这个 patch 文件，几秒钟的事。

对实际跑过几十个并行 agent 的人来说，这个机制太关键了。Agent 偶尔会做出蠢事——可能改了不该改的文件，可能误删了什么。你随手把这个任务扔进 Trash，又突然意识到里面有半小时前一个有用的小修改。没有这个机制，那半小时白干。

`--binary` 这个标志也不是随手加的。即使你这个 worktree 里改了图片、字体、二进制资源文件，patch 也能完整还原，不是只能恢复文本改动。

营销文案不提，是因为说出来等于承认"agent 经常出错"。但跑过的人都知道，**这才是真正能让你放心放手的地方**。一个并行编排工具如果没有这种"撤回机制"，你永远不敢真的并行——总是要先 review 完一个再开下一个，那并行就没意义了。

## 完成之后，commit 还是 PR 你自己选

任务跑完，三条路。

**第一条，点卡片打开 diff review。**Cline 把 agent 的 TUI 和 git diff 并排放着，你可以在 diff 的某一行内留 comment，这条 comment 会被作为新指令喂给 agent，让它修。这个 inline review loop（在 diff 里直接评论、agent 直接迭代）是它替代 IDE 的核心场景之一——你不需要先 merge 才能给反馈，看到不对的地方直接在 diff 里圈出来说"这里改成 X"，agent 就在同一个 worktree 里继续迭代。

**第二条，手动 commit 或 Open PR。**看完没问题就推。Cline 内部直接调 GitHub API，不用你切到浏览器。

**第三条，去 settings 里把 auto-commit、auto-PR 打开。**Agent 跑完直接 commit 甚至直接开 PR。激进派的玩法，适合那种已经对某个 agent + 某种任务类型建立了信任度的场景，比如重复性的 dependency bump、文档同步、test case 补全。

还有一个被忽略的小功能：Cline Kanban 内置了一个 git interface。能看 commit history，能切 branch，不用切回 terminal 或者打开 GitKraken 之类的工具。对纯 CLI 流的开发者来说，这是少有的"图形界面带来的爽点"——你本来就要看 history 决定 merge 哪个分支，现在不用离开看板。

## 这条赛道已经挤进了好几家

写到这里得给一个对比视角，不然推荐就不完整。

"多 agent 编排"已经是 2026 年开发者工具里的一个细分赛道。最常被拿来跟 Cline Kanban 对标的是 **BloopAI 的 Vibe Kanban**——本地 orchestration + git worktree 的玩法跟 Cline 几乎是镜像。

另外两家走的是不同路线。**Conductor OSS** 主打 repo-native、self-hosted，定位在企业内代码隔离场景。**auto-coder.chat** 自带的 Kanban 走的是 cloud control + local execution 的混合架构，跟纯本地的 Cline Kanban 有架构级差异。再往外，**OpenHands** 这种 Docker-sandboxed 平台也算同一赛道，但它把每个 agent 关进容器里，权重更偏"沙箱安全"。

Cline Kanban 在几个点上跟它们拉开了距离。

| 工具 | 架构 | agent 兼容性 | 特点 |
| --- | --- | --- | --- |
| Cline Kanban | 纯本地 | Claude Code / Codex / OpenCode / Cline CLI（4+） | inline diff review + 内置 git 视图 |
| Vibe Kanban | 纯本地 | 多 agent 兼容 | 玩法与 Cline 镜像 |
| Conductor OSS | self-hosted | repo-native | 面向企业内代码隔离 |
| auto-coder.chat | cloud control + local exec | 混合架构 | 控制面在云端 |

**Agent 兼容性更广**：Claude Code、Codex、OpenCode、Cline CLI 都能在同一块板上跑。Vibe Kanban 也做兼容，但目前 Cline 兼容的 runtime 列表（4 个并持续扩）在已开源的几家里偏长。

**数据完全不出本机**：看板、worktree、patch 文件都落在你机器上，不需要登录、不需要同步。这对企业内部代码是硬要求——auto-coder.chat 那种 cloud control 模式对合规团队就是个负担，OpenHands 的 Docker 沙箱虽然安全但要先配 Docker 环境。

**Inline diff review + 内置 git 视图**：这是它对标 IDE 的核心打法。其他几家偏 orchestration——分发任务可以，但看 diff 通常还要切到别的工具。Cline 把 review 这一环也吃下来了，等于在 agent 时代重新做了一遍 IDE 的核心交互。

截至 2026 年 5 月，cline/kanban 仓库还在 **Research Preview** 阶段（仓库 README 自标）。这个阶段适合早期采用者帮忙试错，不适合放进 critical path。Research Preview 跟稳定版的区别官方没明说，但从用语和 v0.x 版本号能看出来：API 不保证向后兼容，breaking change 随时可能来。

## 谁该装，谁该等等

**适合现在装：**

* 每天要并行跑 3 个以上 agent 的人
* 在写 Node/Next.js 项目，被 node\_modules 重复安装搞烦的人
* 同时混用 Claude Code 和 Codex 的人——Kanban 让两个 agent 在同一块板上跑，对比哪个更靠谱很方便，相当于一个免费的 agent 横评工具
* 重视本地化、不愿意把代码上云的人

**先等一等：**

* 只用一个 agent、一次跑一个任务——这种场景 Kanban 没收益，徒增复杂度，直接用原生 CLI 反而清爽
* Windows 用户且没法启用开发者模式——symlink 跳过虽然有 best-effort 处理，但 node\_modules 共享的好处基本拿不到
* 必须等正式版的团队——v0.x 阶段意味着 breaking change 随时可能来
* 非 git 项目——整套机制都建立在 git worktree 之上，不用 git 这工具一行都跑不起来

最后一个小提醒：用之前先确保主仓有 initial commit，不然 worktree 创不起来。DeepWiki 索引提到源码会返回一条友好提示，但新手第一次跑容易撞上。建一个空仓库直接试，第一反应肯定是"为什么报错"——答案就是这个。

---

> 这里是向量猫，帮你从 100 个工具里挑出 3 个真正值得用的。

参考来源：

* cline/kanban GitHub 仓库：https://github.com/cline/kanban
* Cline 官方博客 Announcing Kanban（Sidd Sant，2026-03-26）：https://cline.ghost.io/announcing-kanban/
* Cline 官方文档 Kanban Core Workflow：https://docs.cline.bot/kanban/core-workflow
* Cline 官方文档 Kanban 用法：https://docs.cline.bot/usage/kanban
* DeepWiki Task Worktree Lifecycle（最后索引 2026-05）：https://deepwiki.com/cline/kanban/3.1-task-worktree-lifecycle
* Conductor OSS 对 Vibe Kanban 的对比：https://conductross.com/blog/vibe-kanban-alternative
* auto-coder.chat Kanban 与 Vibe Kanban 架构对比：https://zhuhailin.com/en/blog/kanban-vs-vibekanban