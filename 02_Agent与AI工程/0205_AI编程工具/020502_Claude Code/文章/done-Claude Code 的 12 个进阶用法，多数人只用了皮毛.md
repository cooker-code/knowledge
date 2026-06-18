> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 的 12 个进阶用法，多数人只用了皮毛
author: 萝卜要加油
date:
url: https://mp.weixin.qq.com/s?__biz=MzU2NjU1MTk1MQ==&mid=2247495533&idx=1&sn=d8c6285a76e656d7f559b1fda9ea9cac&chksm=fd69fc78a6099706b0d665a985ef614f640f467e739601878375753cf5c38bb971d008579fcd&mpshare=1&scene=24&srcid=0422VMmT0xyrvrdPwD3FlTTD&sharer_shareinfo=cd40e7c5e76214b1a18f8f82ff41f6c0&sharer_shareinfo_first=cd40e7c5e76214b1a18f8f82ff41f6c0#rd
---

---

# 刚开始用 Claude Code 的时候，我的用法大概是这样的：

打开终端，输需求，等回答，逐条批准权限申请，出错了重新描述，它问要不要继续，我说要，它又问，我又说要。

我以为这就是正常用法。

直到看到 `shanraisshan/claude-code-best-practice`——今年 3 月冲上 GitHub 日榜第一的仓库，收录了 Claude Code 创始工程师 Boris Cherny 和 Anthropic 团队整理的 69 条实战经验。

仓库的副标题只有一句话：

**from vibe coding to agentic engineering**。

看完之后才明白，我之前那套，就是他们说的 vibe coding——随机的、不可复现的、完全依赖当下提示词的用法。下面是我认为最值得立刻去试的 12 条。

## 一、CLAUDE.md 写越长，Claude 越不听

大多数人写 `CLAUDE.md` 的本能是：写得越详细越好。

我最开始写了大概 500 行，代码风格、提交规范、测试要求、变量命名……什么都往里放。后来发现 Claude 基本不遵守后半段。不是它不想，是它顾不过来。

**Claude 推荐的最优长度是 60 行，上限是 200 行。** 超出这个范围，靠后的规则会被悄悄忽略。

解决方法不是删规则，而是让规则按需出现。

`<important if="language=go">` 这个语法可以做条件激活——只有在处理 Go 文件时，这段才对 Claude 可见。

Monorepo 项目可以按子目录分别放 `CLAUDE.md`，比如这样：

```
my-service/
├── CLAUDE.md           # 根级：通用规则，约 30 行
├── api/
│   └── CLAUDE.md       # REST 规范、错误码约定
├── internal/
│   └── CLAUDE.md       # 包命名、interface 设计规范
└── cmd/
    └── CLAUDE.md       # flag 解析、日志初始化约定
```

`internal/CLAUDE.md` 里的 Go 规则这样写：

```
<important if="language=go">
- 禁止裸 goroutine，必须通过 errgroup 或 WaitGroup 管理生命周期
- channel 容量大于 0 时必须注释说明原因
- context 参数必须是函数第一个参数，命名为 ctx
</important>
```

Claude 处理 Go 文件时读这段，写 Python 时完全忽略。总行数不变，但每段规则只在需要的时候出现。

## 二、别把 Claude Code 当聊天框用

用 code review 举个例子。

以前每次让 Claude review PR，都要在提示词里加一句"只分析，不要改文件"。因为不加的话，它真的可能顺手改掉什么。

现在的做法是在 `.claude/agents/` 里放一个 `reviewer.md`，只开 `Read` 权限，写文件的工具直接不给。这个 Agent 物理上没有写权限，不管你怎么说它也改不了。

`reviewer.md` 长这样：

```
---
name: reviewer
description: Code review specialist. Reads code and provides feedback. Never modifies files.
tools: Read, Glob, Grep
model: claude-sonnet-4-6
---

你是一个严格的 code reviewer。

只读代码，不修改任何文件。Review 时关注：
- 错误处理是否完整
- 是否有潜在的并发问题
- 接口设计是否合理
- 测试覆盖是否足够

每条 review 意见注明文件路径和行号，给出具体的改进建议。
```

frontmatter 里的 `tools: Read, Glob, Grep` 就是权限边界——这三个工具只能读，不能写。正文是这个 Agent 的行为指令，只对它自己生效，不占用主上下文。

用的时候在对话里输 `/reviewer`，它就启动了。提示词里那句"不要改文件"，从此可以删掉了。

这是 Agent 机制的核心用途：**不是给 Claude 加角色扮演，而是从权限层面约束它能做什么**。

Claude Code 的 `.claude/` 目录支持三种配置：`commands/` 放工作流快捷指令，`agents/` 放权限受限的专用 Agent，`skills/` 放可复用的知识模块。三个目录组合起来，能搭出一套不依赖提示词的自动化流水线。

## 三、`context: fork`：一行配置，保持上下文干净

Skill 默认和主 Agent 共享上下文。

这意味着什么？假设你有一个分析代码质量的 Skill，它要读几十个文件、生成报告。跑完之后，那些内容全留在主上下文里，后续每个任务都要带着这段历史。时间长了，上下文越来越重，Claude 的质量会悄悄变差。

解决方法就一行，在 Skill 的 frontmatter 里加：

```
context: fork
```

这个 Skill 就会在独立的 subagent 里运行，用完即销毁，主上下文不受影响。不需要手动清理历史，也不需要频繁开新对话。

我之前用一个 Skill 扫描了整个 repo 的依赖关系，读了一百多个文件。跑完之后后续对话明显变慢，Claude 回答一个简单问题也要想半天。加了 `context: fork` 之后这个问题就解决了。

## 四、Hooks：在 Claude 不知道的地方做事

Claude Code 有三个 Hook 节点：`PreToolUse`（工具执行前）、`PostToolUse`（工具执行后）、`Stop`（任务完成后）。

这三个 Hook 在 Claude 的主循环外面运行，**不占 token，不打断任务**，是真正意义上的后台自动化。

几个直接能用的场景：

**写完自动格式化**：`PostToolUse` 里挂上 `gofmt`，Claude 每次写完 Go 文件，格式化自动触发，不需要提醒。

**危险操作拦截**：`/careful` 模式激活后，任何删除、覆盖、不可逆的操作都会暂停，等你手动确认再继续。上次做数据库 migration 脚本，我在开始前打了 `/careful`，Claude 在要执行 `DROP TABLE` 之前停下来等我确认——这一步如果在生产环境跑错了，数据就没了。

**任务完成验证**：这个放到第十一条专门说。

## 五、`/sandbox`：把权限确认从十几次降到两三次

Claude Code 执行 shell 命令时，默认每一步都要弹窗确认。设计上很安全，但快速迭代时体验很差——刚进入状态，又跳出来批准一个 `ls`。

`/sandbox` 让命令在隔离沙箱里运行，影响范围受限，确认提示随之减少 **84%**。

手动批准少了，代码还是跑在受控环境里。这两件事不冲突。

写一个新接口，Claude 要跑 `go build`、`go test`、再 `curl` 测几个端点验证结果——没有 `/sandbox` 的话，这一套下来要手动批准六七次。开了之后，中间那些确认框基本消失了，Claude 一口气跑完，我只在最后看结果。

## 六、十几个 Claude 同时工作

Boris Cherny 描述他自己的工作方式用了这句话：

> "dozens of Claudes running at all times"

实现方式是 `claude -w`——基于 git worktree 启动独立工作区，配合 tmux 多窗格同时运行多个 Claude 实例。

Agent A 在做认证模块重构，Agent B 在修数据库查询，Agent C 在给两者写测试。它们不共享上下文，不互相干扰，改动在各自的分支上，最后合并。

从等一个 Claude 回复，到同时跑十几个——这大概是 agentic engineering 和 vibe coding 最直观的区别。

## 七、`ultrathink`：没人告诉你的关键词

在任务里加上 `ultrathink` 这个词，Claude 会进入更高强度的推理模式，等价于手动触发 extended thinking。

不需要改配置也不需要切换模型，直接在提示词里写就行，超级方便。

大多数任务不值得用——写单元测试不需要，整理 import 不需要。但遇到真正复杂的问题，用一次，输出会明显不同。

我用它做过一次服务拆分的方案设计。普通模式给了三个方案，都还不错。加了 `ultrathink` 之后，它多分析出了两个我没考虑到的约束条件——一个是跨服务的事务边界，一个是现有监控系统的接入成本。最终选的方案和普通模式推荐的完全不同。

## 八、`/loop`：让 Claude 自己跑起来

```
/loop 30m /code-review
```

这一行，每 30 分钟自动触发一次 code review，没有人在旁边操作。支持 `5m`、`30m`、`1h`，最长可以持续 3 天。

CI/CD 解决的是代码合并后的自动化。`/loop` 解决的是本地开发过程中的自动化——它知道你当前的工作状态，可以做 CI 做不了的事：基于上下文的实时代码建议、持续的测试监控、自动生成的 PR 摘要。

很多人不知道这个能力已经内置在 Claude Code 里了。

## 九、`/btw`：不要为一个小问题打断整个任务

Claude 在跑一个长任务，如果中途你有一个问题想问，但问了又会打断长任务的进度。

`/btw` 就是为了解决这个问题诞生的。它把你的问题排进队列，Claude 继续干当前的活，它会在自然停顿点来回答你的任务，然后接着跑长任务。

用过一次就会觉得，以前那种"先停下来问、问完再重启任务"的方式真的很低效。

## 十、Skill 文档里最值钱的部分

如果你在维护团队的 Skill 库，这个仓库 Skills 领域的核心贡献者 Thariq 有一条建议：

> "The highest-signal content in any skill is the Gotchas section."

Gotchas 记的是 Claude 在实际执行中踩过的坑——是真实的失败案例，随着你的使用持续积累。

大段的使用说明 Claude 往往读一遍就忘。Gotchas 里的内容，每次任务跑失败时都会被翻出来参考。先建 Gotchas，再写其他文档。

一条 Gotchas 长这样：

```
## Gotchas

- 调用 GitHub API 时不要用 `octokit`，这个项目已经换成了直接 `fetch`，
  上次用 octokit 导致编译报错，排查了 20 分钟才发现。
- 生成 PR 描述时必须先读 CONTRIBUTING.md，否则格式不符合要求会被 CI 拒绝。
```

这两条都是真实踩过的坑，写下来之后 Claude 下次不会再犯同样的错。

## 十一、"完成了"不等于真的完成了

Claude 说任务完成，和任务真的完成，是两件事。

Stop Hook 可以在 Claude 宣告完成时，自动触发验证脚本——跑测试、检查文件是否存在、验证 API 响应。验证失败，Claude 继续工作，直到通过再停。

之前让 Claude 写一组 API handler，它说完成了。我去看代码，发现三个 handler 里有两个错误处理直接是空的——`err != nil` 之后什么都没写。如果在 Stop Hook 里挂了一个检查测试覆盖率的脚本，会直接触发这类问题并回退，让 Claude 补完再重新跑。现在"完成"这个词对我来说才真的有意义。

## 十二、SDK 调用慢？加一个 Flag

如果你用 Claude Agent SDK 做程序化调用时，遇到加载速度比较慢的情况，不妨加上一个 `--bare` 标志。

`--bare` 的作用是跳过上下文发现过程（找 CLAUDE.md、加载配置、检查环境），根据官方的介绍，启动速度最高可以提升十倍。

可能在跑单个任务感觉差别不大。但是如果要批量跑几百个 Agent 的时候，差距就出来了。

我们有一个每天跑的代码分析 pipeline，大概 300 个文件，每个文件起一个 Agent 独立处理。加 `--bare` 之前跑完要将近 40 分钟，加了之后降到 18 分钟左右。

---

这 12 条来自 `shanraisshan/claude-code-best-practice` 整理的 69 条实战经验，仓库仍在持续更新。

工具是同一个工具，大多数人只能用到了 20% 的功能，剩下的功能放在文档里没人看。你现在用到第几层？