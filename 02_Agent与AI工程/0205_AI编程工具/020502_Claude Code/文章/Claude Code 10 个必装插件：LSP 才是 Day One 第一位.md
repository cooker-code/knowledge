---
title: Claude Code 10 个必装插件：LSP 才是 Day One 第一位
author: 向量猫
date: 向大猫向大猫
url: https://mp.weixin.qq.com/s?__biz=MzYzNzg4ODc5NQ==&mid=2247483655&idx=1&sn=9b1bfcd0d911b24c67f06ccdaf47df3e&chksm=f163a7ed16e8aa32147b38017853bcba5293f10b58b207e60504295e663de495c5114a31f86c&mpshare=1&scene=24&srcid=0428ORPRxPCT0yuW5OP8wufc&sharer_shareinfo=5d1382e2e31c4e053c6eac009d4b85aa&sharer_shareinfo_first=5d1382e2e31c4e053c6eac009d4b85aa#rd
---

> 大多数清单漏掉了真正该先装的那个——不是 feature-dev，是 LSP。

装完 Claude Code 之后，第一件事通常是去搜"必装插件"。然后大部分人会得到一个相似的清单：feature-dev、superpowers、各种花哨的 Skills 合集。

这份清单的问题不是推荐错了，是顺序错了。

官方文档里把 plugin 分成四大类，排在第一位的不是工作流，也不是外部集成，而是**代码智能（code intelligence）**——10 种语言对应的 LSP plugin。这个顺序不是随便排的。

下面这 10 个，按实际装机顺序给你排好。

## 为什么第一个必须装 LSP 而不是 feature-dev

先讲一个大多数推荐文章不会提的事实：Claude Code 在没有 LSP 的情况下，写 TypeScript 和 Python 基本靠猜类型。

它会根据上下文推断 `user.profile.name` 存不存在，推得准不准完全取决于它看到了多少代码。上下文窗口越小，猜得越离谱。

一旦装上 `typescript-lsp` 或 `pyright-lsp`，情况就变了。语言服务器（Language Server Protocol，IDE 背后提供类型检查和跳转的那一层）会在每次 edit 之后把诊断信息回送给 Claude——哪一行哪一列错了、类型期望是什么、实际是什么。Claude 看到错误会立刻改，不会等到你跑 `tsc` 才发现。

举个具体场景：你让 Claude 给一个返回 `Promise<User | null>` 的函数加调用，它写了 `.name` 访问。没装 LSP 的时候这段代码要跑起来才会爆；装了之后编辑器会立刻红线，Claude 下一轮自动加 null check。

官方内置的 marketplace `claude-plugins-official` 里提供了 10 种语言的 LSP plugin，命名规律一致：

```
/plugin install typescript-lsp@claude-plugins-official /plugin install pyright-lsp@claude-plugins-official /plugin install rust-analyzer-lsp@claude-plugins-official /plugin install clangd-lsp@claude-plugins-official
```

装哪个看你主力语言。这是 Day One 的第 1 个。任何 Skills 和工作流插件都替代不了这一层的基础能力。

## 第 2 个：feature-dev，把一次对话变成 7 阶段工作流

LSP 解决的是"写得对不对"，feature-dev 解决的是"该不该这么写"。

这是 Anthropic 官方推出的结构化开发 plugin，把一次完整的功能开发拆成 7 个阶段：discovery → codebase exploration → clarifying questions → architecture design → implementation → quality review → summary。

关键在阶段 2 和阶段 3。阶段 2 会启动专门的 subagent 去探索代码库，先画出相关模块的调用关系再动手，而不是一上来就写。阶段 3 会主动反过来问你边界情况——"如果用户没登录怎么处理""这个字段允许为空吗""要不要支持并发调用"。

不装它的时候，你得自己引导 Claude 思考这些。装了之后，它自己会问。

```
/plugin install feature-dev@claude-plugins-official
```

适合做新功能开发。不适合改 bug 或写一次性脚本——对这些场景来说 7 阶段太重了，你只是想改一行，它给你画一套流程图。

## 第 3 个：code-review，把审查从"聊天"变成"报告"

让 Claude 读一遍代码然后"给点意见"，是大多数人用 Claude Code 审代码的方式。问题是它经常纠结于无关紧要的东西——比如注释缺失——而漏掉 SQL 注入。

code-review 这个官方 plugin 的区别在于**结构化输出加置信度过滤**。它从安全、性能、可维护性、正确性四个维度独立评估，每条发现都会标上 `high/medium/low` 三档置信度，默认只显示 high 和 medium。

这个过滤机制的实际效果：一次审查从原来的几十条"建议"压缩成 3-5 条真问题。代码量大的时候差异尤其明显——过去需要你肉眼翻审查结果挑重点，现在列表里剩下的都是值得看的。

可以对当前 diff 跑，也可以传一个 PR 编号：

```
/plugin install code-review@claude-plugins-official
```

配合 feature-dev 使用——前者负责写出来，后者负责把关。

## 第 4 个：context7，解决训练数据过期的最省事方案

这是 Day One 清单里最容易被忽视的一个，也是装了之后立刻能感受到差异的一个。

Upstash 做的 context7 解决的是一个老问题：大模型训练数据是有截止日期的。它生成的 Next.js 代码可能在 v13 里完美运行，但你用的是 v15。App Router 的 API 改了，`fetch` 的缓存默认行为变了，`use client` 的位置规则也变了，Claude 不知道。

context7 的做法简单粗暴——按你指定的库名和版本，从源仓库拉取当前版本文档，切片后注入到上下文里：

```
/plugin install context7@claude-plugins-official /docs next.js /api-ref prisma findMany
```

你问它 `prisma.findMany` 的新参数时，它拉的是当前版本的文档，不是记忆里的旧版。对于版本迭代快的库（Next.js、LangChain、Pydantic v2），这个 plugin 基本是刚需。

## 第 5 个：commit-commands，把规范写进命令里

这是一个很小但每天都会用到的 plugin。它把 commit message 规范做成 slash 命令，自动读取 staged diff 并生成符合约定式提交（Conventional Commits）的信息。

```
/plugin install commit-commands@claude-plugins-official
```

对多人协作项目特别合适——团队的 commit 风格能瞬间统一。写 changelog 的时候也省事，因为格式规范了之后可以直接用脚本生成。

## 第 6 个：pr-review-toolkit，PR 级别的 code-review

code-review 是函数级别的，pr-review-toolkit 是 PR 级别的。它会把整个 PR 拆成逻辑块，分块审查然后汇总。

大 PR（超过 500 行 diff）必备。没它的时候 Claude 经常审到一半就丢上下文——前面看到的问题到后面就忘了，得让它从头再过一遍。分块策略解决了这个问题：每一块独立审查，最后由一个 summary 阶段做全局整合。

```
/plugin install pr-review-toolkit@claude-plugins-official
```

## 第 7 个：github 外部集成，让 Claude 能自己查 issue

官方 marketplace 里的外部集成这一类被严重低估。github、gitlab、atlassian、linear、notion、figma、vercel、firebase、supabase、slack、sentry——11 个 plugin 对应 11 个常用外部系统。

Day One 至少装 github。装了之后 Claude 能直接读 issue、查 PR、看 CI 日志，不用你来回复制粘贴：

```
/plugin install github@claude-plugins-official
```

场景举例：CI 挂了，你不用自己去 GitHub Actions 页面翻日志再贴回来，直接告诉 Claude "看一下最新一次 workflow 的失败原因"，它会用 MCP 自己拿数据。后端在用 Sentry 的加 sentry，用 Vercel 的加 vercel，按栈配。

## 第 8 个：document-skills，处理 Office 文档的瑞士军刀

这个来自 Anthropic 的官方 Skills 仓库（anthropics/skills，GitHub 上 Star 数已破 10 万）。它不在 `claude-plugins-official` 里，需要先把这个 marketplace 加进来：

```
/plugin marketplace add anthropics/skills /plugin install document-skills
```

装完之后 Claude 能直接生成和解析 docx、pdf、pptx、xlsx。处理合同、简历、表格报告时省掉大量样板代码——过去要先写一段 `python-docx` 的骨架代码，现在直接说"把这段内容导出成 Word 带目录"就行。

每一个 skill 就是一个带 SKILL.md（YAML frontmatter + 指令）的文件夹，按需触发，不会常驻消耗上下文。这是 Skills 系统和 Hooks 最大的区别——不调用不占 token。

## 第 9 个：输出风格 plugin，把 Claude 切换到"讲课模式"

这是个冷门但对学习场景极其有用的类别。官方提供了两个：

* `explanatory-output-style`

  ：每次生成代码都附带详细解释，说清楚这行为什么这么写、为什么不用另一种方案
* `learning-output-style`

  ：Socratic 式追问，帮你理解而不是替你做，会反过来问你"这里你觉得应该用 Map 还是 Object"

```
/plugin install explanatory-output-style@claude-plugins-official
```

适合新人或者在学陌生领域（比如你是前端但要写一段 Rust）。不适合赶进度的场景——会显著变慢，生成同样代码的 token 消耗大约是 1.5-2 倍。

## 第 10 个：隐藏 marketplace 里的 knowledge-work-plugins

这是本文最后一个独家发现。

除了内置的 claude-plugins-official，Anthropic 还有两个官方 marketplace 需要手动添加：life-sciences 和 knowledge-work-plugins。后者里有 14 个角色型 plugin，覆盖 Sales、Marketing、Finance、Legal、HR、Brand Voice 等场景。

Claude.com 的网页端给其中 11 个打了 Cowork-only 的标签，让人以为开发者用不上。**实际上这些 plugin 在 Claude Code 里完全能装能用**，Brand Voice 和 Marketing 实测有效：

```
/plugin marketplace add anthropics/knowledge-work-plugins
```

这对做产品或市场方向的技术人是一个隐藏资源——你不用再写一长串 prompt 教 Claude 怎么按品牌调性说话，一个 plugin 搞定。写落地页文案、改招聘邮件、出季度总结，调性稳定性比自己维护 prompt 好得多。

## 装 plugin 之前最该知道的一件事

最后说个大部分教程不会讲、但会影响会话体验的事——**Plugin 在 Claude Code 里是一个由 4 种构件组成的容器**：

| 构件 | 触发方式 | 常驻成本 |
| --- | --- | --- |
| Skills | slash 命令按需 | 零 |
| Commands | 自定义斜杠命令 | 轻量 |
| Hooks | 会话后台自动触发 | 有 token 开销 |
| MCP servers | 按查询次数 | 按调用计 |

新人常犯的错误是把"装 plugin"当成零成本的事，装了一堆带 hook 的插件，然后发现会话变卡、token 消耗爆了——不知道哪里在偷偷吃 context。一个带 hook 的 plugin 可能在你每次 edit 时都往上下文里塞几百 token 的检查结果，装 5 个就是几千 token 常驻开销。

Day One 清单的取舍逻辑就是从这里来的：**优先选 Skill 和 Command 为主的 plugin**（feature-dev、context7、commit-commands 都属于这类），避开过多 hook 型的插件。等摸清楚了自己的工作流，再按需加重度 plugin。

## 一句话适合人群

* **全栈/后端工程师**

  ：LSP（选一种）+ feature-dev + code-review + context7 + github，这 5 个必装，其他按需
* **AI 应用开发者**

  ：额外加 agent-sdk-dev 和 plugin-dev，直接命中开发 Agent 的日常
* **做 B 端产品/市场的技术人**

  ：knowledge-work-plugins 的 Brand Voice 值得试
* **刚入行的新人**

  ：加 learning-output-style，前期慢一点但知识沉淀更扎实

别一次装 10 个。按顺序往里加，装一个用两天再加下一个——你会更清楚每个 plugin 到底给你带来了什么，也更清楚哪些是你根本用不上但在后台偷偷吃 token 的。

---

> 这里是 AI 工具站，帮你从 100 个工具里挑出 3 个真正值得用的。

参考来源：

* Anthropic Skills 官方仓库：https://github.com/anthropics/skills
* Claude Code 官方文档 - Discover and install prebuilt plugins：https://code.claude.com/docs/en/discover-plugins
* Claude Plugins 官方页面：https://claude.com/plugins