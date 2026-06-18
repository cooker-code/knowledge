> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor Rules与Plan工作流边界|Cursor Rules与Plan工作流边界]]、[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor工程使用与上下文边界|Cursor工程使用与上下文边界]]
---
title: Cursor 官方出品：AI 编程 Agent 最佳实践指南
author: 十二AI编程
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzMzg5OTQyMg==&mid=2247485354&idx=1&sn=86fa69f17d1dd8df523225254dc60163&chksm=c3a6fc95c975104bcf85b5d22537c3d566b7cf3b20c4b5b8543ab950c0b6da39ffb72788412d&mpshare=1&scene=24&srcid=0227C6QKSqWXd9nDOSyPaG15&sharer_shareinfo=2811e26cdaf5a870bda65f5ffca672c1&sharer_shareinfo_first=2811e26cdaf5a870bda65f5ffca672c1#rd
---

大家好，我是十二。专注于分享AI编程方面的内容，欢迎关注。另有 Cursor、Claude Code、Codex 的优惠渠道，欢迎私信。

AI 编程 Agent（智能体）正在彻底改变软件构建的方式。现在的模型不仅能运行数小时，完成庞大的多文件重构，还能不断迭代直到测试通过。

但要充分发挥 Agent 的潜力，不仅需要了解它们的工作原理，还需要掌握新的协作模式。

无论你是 AI 编程的新手，还是希望像 Cursor 团队一样高效工作的开发者，这份指南都将为你揭示与 Agent 结对编程的最佳实践。

01 谋定而后动：善用“计划模式”

Cursor 团队发现，经验丰富的开发者在写代码前往往会先做计划。为了让 Agent 更像资深开发者，Cursor 引入了 Plan Mode（计划模式）。

在 Agent 输入框中按 Shift+Tab 即可切换到计划模式。Agent 不会立即写代码，而是执行以下步骤：

1. 调研：搜索你的代码库，找到相关文件。

2. 提问：针对你的需求提出澄清性问题。

3. 规划：创建一个包含文件路径和代码引用的详细实施计划。

4. 待命：等待你的批准后再开始构建。

注意事项：

保存计划：点击 "Save to workspace" 将计划保存到 .cursor/plans/。这不仅是团队文档，也能帮助你在工作中断后快速恢复上下文。

从计划重来：如果 Agent 写偏了，不要试图通过对话修修补补。不如回滚更改，修改 Plan 文件，然后让 Agent 重新执行。这通常比修正“正在进行中”的 Agent 更快且代码更干净。

02 上下文管理：少即是多

随着你对 Agent 越来越熟悉，你的核心工作将转变为提供上下文。

不要手动 Tag 所有文件：Cursor 的 Agent 拥有强大的搜索工具。当你问“认证流程”时，它能通过 grep 和语义搜索找到相关文件，哪怕你的提示词里没有精确的文件名。

善用 @ 符号：使用 @Branch 让 Agent 了解当前分支的变动；在开启新对话时，使用 @Past Chats 引用过去的工作，而不是复制粘贴整个聊天记录，这样能减少上下文中的“噪音”。

什么时候该开启新对话？当 Agent 开始感到困惑、重复犯错，或者你已经完成了一个逻辑单元的工作时，果断开启新对话。过长的对话会积累干扰信息，导致 Agent 注意力分散。

03 定制你的 Agent：Rules 与 Skills

Cursor 提供了两种方式来定制 Agent 的行为：

1. Rules（规则）：静态上下文

这是适用于所有对话的“常驻指令”。你可以将规则保存为 .cursor/rules/ 下的 Markdown 文件。

内容建议：常用的构建命令、代码风格偏好（如“使用 ES modules”）、项目特定的目录结构规范。

避免什么：不要复制整个风格指南（用 Linter 代替），也不要通过 Rules 记录所有边缘情况。

2. Skills（技能）：动态能力

Agent Skills 扩展了 Agent 能做的事情，定义在 SKILL.md 文件中。与始终加载的 Rules 不同，Skills 是按需加载的。

自定义命令：例如定义一个 /pr 命令，让 Agent 自动执行 git diff、写 Commit 信息、Push 代码并创建 Pull Request。

Hook 脚本：例如创建一个循环脚本，让 Agent 持续修改代码直到所有测试通过。

04 高级工作流模式

测试驱动开发 (TDD)

Agent 在有明确目标时表现最佳。你可以尝试以下 TDD 流程：

1. 让 Agent 编写测试（明确说明是 TDD，避免它生成 Mock 实现）。

2. 运行测试并确认失败。

3. 让 Agent 编写代码通过测试，并指示其不断迭代直到成功。

并行开发与云端 Agent

并行工作：Cursor 自动为并行运行的 Agent 创建 Git worktrees（工作树）。这意味着你可以同时运行多个 Agent，它们在各自隔离的环境中修改、构建和测试，互不干扰。

云端委托：对于重构、修 Bug 或写文档等后台任务，可以委托给 Cloud Agents。它们在远程沙箱中运行，即使你合上电脑也会继续工作，并在完成后通知你。

调试模式 (Debug Mode)

遇到棘手的 Bug 时，使用 Debug Mode。它不仅仅是猜测修复方案，还会：

1. 提出多个假设。

2. 在代码中插入日志（Instrumentation）。

3. 分析运行时数据以定位根因。

4. 基于证据进行修复。

05 像专家一样 Review

AI 生成的代码看起来正确，但可能包含细微错误。仔细审查是高效使用 Agent 的关键一环。

生成时审查：在 Diff 视图中看着 Agent 写代码。如果发现方向不对，按 Escape 键及时打断。

Agent Review：代码生成后，点击 Review -> Find Issues，让 Agent 对自己的改动进行逐行分析和查错。

Bugbot：提交 PR 后，Cursor 的 Bugbot 会自动进行高级分析，在合并前拦截潜在问题。

06 总结

使用 AI 编程不仅仅是输入 Prompt，更是一种新的协作艺术。

那些从 Agent 中获益最多的开发者，往往具备以下特质：编写具体的提示词、提供可验证的目标（如测试）、并且像对待有能力的同事一样，要求 Agent 提供计划和解释。

Agent 的能力正在飞速进化，希望这篇指南能帮助你在这个 AI 编程的新时代中游刃有余。

AI编程交流群

欢迎进群交流，关注公众号，点击【进交流群】，加我好友，我拉你进群。

感谢阅读，如果觉得不错，随手点个赞、在看、转发三连吧~