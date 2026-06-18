> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: Codex主创最新分享：把一次性提示变成可验证的工作循环
author: Vibe编码
date: VibeCoderVibeCoder
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODkzOTY3MA==&mid=2247485432&idx=1&sn=1acb907f715a1338194e7a637578c653&chksm=c4518a45203d341977254831d3efae5ce49bedc95173a0bc7d9e08258615d8de76f2b5e17946&mpshare=1&scene=24&srcid=0522gMVoZCHIpKP6ff9g1tt9&sharer_shareinfo=514e279389929950a1dbb56967479732&sharer_shareinfo_first=514e279389929950a1dbb56967479732#rd
---

Codex 的讨论最近有一个变化。大家一开始关心它能不能改代码、跑测试、开 PR，现在更值得写的是另一层：它能不能把电脑上的工作串起来。

Jason Liu 在 X 上发了一篇《Getting the most out of Codex》，主线很直接：代码仍然是 Codex 的中心，但很多电脑工作本来就绕不开 shell、浏览器、API、文档、消息、事件和自动化。Codex 一旦能碰到这些表面，它就不只是一个 coding assistant，更像一个可以持续运行的工作台。

Towards Data Science 也有一篇《How to Maximize OpenAI's Codex》，角度更偏工程实践：怎么配 reasoning，怎么接 Playwright MCP，怎么让 Codex 自己验证浏览器里的功能，怎么用 worktree alias 支撑并行任务。两篇放在一起看，能拼出一个更完整的使用方法。

真正决定 Codex 产出质量的，往往不是一句 prompt 写得多漂亮。更关键的是你有没有给它一个能持续推进、能被验证、能被人审查的工作循环。

## 从聊天窗口变成工作线程

很多人用 coding agent 的方式还停留在短对话：丢一个需求，等它改完，看 diff，跑测试。这个模式当然有用，但每次都要重新解释背景，重新告诉它项目习惯，重新强调哪些地方不要动。

Jason 提到的 durable thread 解决的是这个问题。一个长期线程对应一个长期工作流，比如 release、文档审查、开源项目维护、外部信息监控。它更像一个会积累上下文的工作空间。

这里有个成本问题。长线程不是免费的，隔很久回来可能没有缓存，继续跑会更贵。但对重要工作流来说，省下来的上下文重建成本通常更值钱。你不用每次从头解释这个项目为什么这么写、谁在等结果、上次做到了哪里。

这个思路对技术团队尤其有用。一个 release thread 可以记住当前版本的 blocker，一个 docs thread 可以记住术语翻译和风格约束，一个客服/运营 thread 可以记住哪些 Slack 和 Gmail 线程还没处理。

## 记忆要写进文件

长期线程会带来另一个问题：记忆如果只留在对话里，很难审查，也很难复用。

Jason 的做法是把长期工作流锚在一个 vault 里。这个 vault 可以是 Obsidian，也可以是普通文件夹，里面放 TODO、people、projects、agent、notes。顶部放 AGENTS.md，告诉 Codex 什么信息要保存，保存到哪里，什么时候不要制造无意义变更。

我觉得这个点比长上下文更重要。长上下文本身只是把历史塞进模型窗口。文件化记忆会强迫 agent 把经验压缩成可以读、可以改、可以 diff 的东西。

举个例子，AGENTS.md 里可以写：

```
Treat ~/vault as durable work memory.
Preserve decisions, blockers, owners, dates, and useful links.
Prefer canonical notes over note sprawl.
If nothing meaningful changed, do not churn the vault.
```

这类规则会改变 Codex 的行为。它不只是在当前线程里记得，还会把可复用信息沉淀到下一次能接上的地方。团队协作里这点更关键，因为记忆文件可以被代码审查一样审查。

## 让 Codex 碰到真实验收面

TDS 作者提到一个很实用的配置：给 Codex 接 Playwright MCP，让它能打开浏览器检查自己刚实现的功能。

这比让它写完代码再让我手动看有效得多。前端功能、登录流程、表单、图表、布局、按钮状态，这些东西只看代码很容易漏。Codex 能进浏览器，就可以跑一遍用户路径，截图，观察页面，发现文字溢出、按钮没反应、路由跳错、接口状态不对。

Jason 的文章把工具层拆得更细：`$browser` 适合 Codex App 侧边栏里的本地网页审查，`@chrome` 适合需要登录态和多标签的 Chrome 工作流，`@computer` 适合只能通过桌面 GUI 完成的任务。再往外是 MCP servers 和 connectors，比如 Slack、Gmail、Calendar。

这不是为了让 agent 到处乱点。真正的价值是把验证面拉近。写 UI 就让它看 UI，写脚本就让它跑脚本，写文档就让它打开渲染后的文档，做报表就让它检查表格和图表。

如果只能给 Codex 一条改进建议，我会选这个：让它接触最终产物，而不是只接触源文件。

## 权限边界要靠工程设计

TDS 作者还提到自己会在限定目录里给 Codex 更宽的执行权限。他的观点是，真正危险的权限应该在基础设施层隔离。开发者和 agent 都不该拥有一键删除生产数据库这类能力。

这个判断我认同一半。宽权限确实能减少交互摩擦。Codex 如果每跑一个测试、装一个依赖、生成一个文件都要等确认，长任务会被切碎。可权限放宽的前提是边界清楚：工作目录在哪里，哪些命令可以跑，哪些环境变量不能碰，生产凭证有没有隔离，破坏性操作有没有二次保护。

更稳的方式是把权限写进项目约束。比如在 AGENTS.md 里明确：默认只改当前任务相关文件；运行测试前说明范围；不碰生产配置；遇到迁移、删除、重写历史这类动作要停下来确认。

Codex 的听话是优点，也会放大坏指令。你让它严格做一件事，它通常会少动无关代码；你漏说了上下游验证，它也可能不会主动补齐。

## Worktree 解决并行任务

TDS 作者认为 Codex 当时的 worktree 体验不够顺手，于是自己做了 `codex-wt <worktree-name>` 这样的 alias。这个点要看当前入口。OpenAI 现在的 Codex 产品页已经把 built-in worktrees 写进能力里，所以 alias 更适合作为本地 CLI 或旧工作流的 fallback。

这个小技巧很实用。coding agent 的一个常见问题是并行任务互相污染：A 任务改了一半，B 任务也要动同一个 repo，结果 diff 混在一起。worktree 可以把任务隔开，每个线程拥有自己的目录、分支和验证结果。

实际用 Codex 时，我建议给不同类型任务配不同工作目录。小 bug 可以直接在当前分支做；大重构、依赖升级、迁移、UI 实验，最好开 worktree。这样失败成本低很多，回滚也清楚。

## Goal 要有验收信号

Jason 讲 Goals 时有个很好的判断：弱目标是实现这个 Markdown 计划，强目标会带一个真实完成线。

比如迁移一个库，目标不该只写把 Python 版本迁到 Rust。更好的写法是：迁移到 Rust，并且通过原 Python 项目的单元测试集合。这里的测试就是 verifier。

这个原则可以套到大多数 Codex 任务里。修 bug 要有复现脚本。做性能优化要有 benchmark。改前端要有 Playwright 路径和截图检查。写文章要有来源清单、字数、禁用词、图片占位检查。做数据分析要有可复算的 notebook 或 CSV 输出。

没有 verifier，长任务很容易跑成漂亮的过程记录。看起来做了很多，真正能不能用没人知道。

## Steering 比完美 prompt 更实用

很多人想把 prompt 一次写到完美。Codex App 的协作方式其实更像边做边修。

Jason 区分了 steering 和 queuing。steering 是任务还在跑时打断并纠偏，比如这个 copy 错了、这里间距太大、先别开 PR。queuing 是不打断当前动作，把下一步排进去，比如做完后把 preview link 发到 Slack 给评审人。

这两个动作会让人从等待模型完成，变成跟模型共同操作同一个产物。尤其是 side panel 出现以后，用户看到页面、文档、表格、幻灯片，直接在旁边补充意见，Codex 再回去改。

这也是为什么 `index.html`、Storybook、Remotion Studio、Slidev、Streamlit 这类可视化表面会变重要。它们既是输出，也是控制面。Codex 可以生成它、打开它、检查它，再根据人的标注继续改它。

## 和 Claude Code 怎么取舍

TDS 作者的体验是：Codex 在边界明确的任务上更快，也更少做额外动作。Claude Code 的功能栈更完整，比如 worktree 和 agents view。这个判断比较接近我对这类工具的理解。

如果任务很明确，比如修一个具体 bug、补一个测试、改一个局部 UI、验证一个浏览器路径，Codex 的范围控制会很舒服。它通常会盯着你说的那件事做。

如果任务本身需要 agent 主动拆解很多方向，或者你已经有一套成熟的 Claude Code 多 agent 工作流，Claude Code 可能更顺手。这里没有绝对赢家，关键看任务边界。

有个容易忽略的点：Codex 越严格执行指令，用户越要把验收写完整。你不能只说改好登录页，然后期待它自动猜到所有设备、权限、异常状态、无障碍检查和回归测试。

## 总结

用好 Codex 的关键，是把任务设计成一个小型工作系统。

上下文放在长期线程和文件化记忆里。工具接到真实验收面。产物在 side panel 里被人审查。长任务用 Goal 表达，并带上 verifier。权限给得足够让它工作，但边界要靠目录、凭证、基础设施和 AGENTS.md 约束清楚。

prompt 当然重要，但 prompt 只是入口。真正拉开差距的是后面的循环：Codex 能不能看见上下文，能不能动真实工具，能不能检查自己的产物，能不能把经验写回下次还用得上的地方。

如果你现在只把 Codex 当成代码生成器，可以先改一个习惯：每次给任务时顺手补上验收信号。让它知道怎么证明自己做对了。这个动作很小，但会立刻改变结果质量。