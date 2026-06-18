> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code Dynamic Workflows：把复杂长时任务交给一群 agent
author: Vibe编码
date: VibeCoderVibeCoder
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODkzOTY3MA==&mid=2247485573&idx=1&sn=ff630ae37b28eefa54e6c2780d041e9f&chksm=c4d77dd8199a1dc4971b946d9c4587c7dd8bfba9fcbcc023ae940c5b0a091e006e750a827ffa&xtrack=1&req_id=1780053432623749&scene=90&subscene=93&sessionid=1780054520&flutter_pos=1&clicktime=1780054531518&enterid=1780054531518&finder_biz_enter_id=4&ranksessionid=1780053432&key=daf9bdc5abc4e8d0c19089505aa01679ba17755e5b28340e128f521de7d6e9bfec12c0efe0f4bbd0762522f47b2e15eda892c39c49d60fbe2b032e5cdabe62d3da81f667ce80124b7f7a97ab0b942ea693c9245289e2feb44ce46faacc598c1bd073019b0664984ab9812237d8a413385ff99cdbdf5b89cf1e4ad20b361e4067&ascene=0&uin=MjAwNTI2NjEyMw%3D%3D&devicetype=OHOS-23&version=f3801127&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&countrycode=CN&exportkey=n_ChQIAhIQ2xGrgi9oG4OVDh7%2BloZbmxLQAQIE97dBBAEAAAAAAH0bOHo06rUAAAAOpnltbLcz9gKNyK89dVj0LOeUY6Q03t%2BWU1t6b9VnLoUVkFMvHdW800nRprmsRKfmKAhNRsYbr27mM1j5paWn9lYX8551AUterMGprv6GOn8%2FtrdVaPv7rFOsCMPHRkgt95fSBG1k9dtllZg01dvHhofzLHfoq64XXnIcdkp0RxKBavloKeznTAlIkZJTY48BwWlpH1m1dfZdqXj%2BUr29ezQmrmDvBKXFPs9UiqTBhU7GYEJpkYItheY%3D&pass_ticket=KIxHqQfZTpanGTXCQOlavYKWqdhoqDTEuuRSA8YtY55YA8%2FNSifIIOr6twP6RsEE&wx_header=3
---

Claude Code 在 2026 年 5 月 28 日发布了 Dynamic Workflows。这个功能跟 Opus 4.8 一起出现在 v2.1.154 changelog 里，官方把它标成 research preview。一天后，5 月 29 日，最新 changelog 已经到 v2.1.156，主要是修复 Opus 4.8 的 thinking block 问题。

这次更新真正有意思的地方，是 Claude Code 开始把复杂任务的编排逻辑写成脚本运行。以前我们让 Claude 派 subagent，本质上还是主会话在调度；现在 workflow runtime 可以在后台跑一段 Claude 写出来的 JavaScript 脚本，让几十到上百个 subagents 分阶段工作、互相复查，最后再把结果交回来。

## 它是什么

Dynamic Workflows 是 Claude Code 的多 agent 编排能力。你描述任务，Claude 生成一段 workflow 脚本，运行时在后台执行。脚本负责拆阶段、派 agents、收集中间结果、做验证和汇总。

官方文档里有个很关键的边界：workflow 脚本本身不直接访问文件系统，也不直接跑 shell。真正读文件、改文件、执行命令、调用 MCP 的还是 subagents。脚本像调度器，agent 才是 worker。

这解释了它为什么比普通 subagents 更适合大任务。普通 subagent 的结果会回到主对话，主会话要继续判断下一步；workflow 把循环、分支、复查逻辑留在脚本变量里，主会话只拿最后的报告。上下文压力小很多，执行也更稳定。

## 和 subagents、skills、agent teams 的区别

Claude Code 现在有好几套并行能力，刚看会有点乱。

Subagents 适合旁路任务。比如让一个 agent 去查日志、读一批文件、做一次局部 review，然后返回摘要。它省的是主会话上下文。

Skills 适合固定流程和能力注入。比如写文章、发小红书、生成图表、按某个团队规范做 review。它更像操作手册。

Agent teams 适合多个 Claude Code session 协作，带共享任务列表，队友之间能互相发消息。它更像一个小团队，适合跨层功能开发、不同假设并行调试。

Dynamic Workflows 适合把任务图固化进脚本。它不强调队友聊天，强调 fan-out、复核、聚合、恢复和复用。代码库级审计、几百个文件的迁移、多来源研究，都是它的主场。

## 怎么开启

先看版本：

```
claude --version
```

需要 Claude Code v2.1.154 或更高。

官方文档写的是所有 paid plans、Anthropic API、Amazon Bedrock、Google Cloud Vertex AI、Microsoft Foundry 都可用。Pro 用户需要在 `/config` 里找到 Dynamic workflows 并开启。官方发布博客补充说，Max、Team、API 默认开启；Enterprise 发布时默认关闭，需要管理员打开。

```
/config
```

大任务前也建议看一下当前模型：

```
/model
```

workflow 默认会让每个 agent 用当前会话模型，除非脚本把某些阶段路由到别的模型。你也可以在 prompt 里说清楚：低风险扫描用便宜模型，关键验证用强模型。

## 触发方式

最直接的方式是在 prompt 里写 `workflow`。

```
Run a workflow to audit every API endpoint under src/routes for missing auth checks.
Map endpoints first, assign independent reviewers by route group, then verify each finding
with a second agent before reporting. Do not edit files; return file paths and evidence.
```

Claude Code 会高亮 `workflow` 这个词，然后 Claude 会为任务写 workflow 脚本。假如你只是顺手写了这个词，不想触发，按 `alt+w` 可以忽略。

另一种方式是开 ultracode：

```
/effort ultracode
```

ultracode 是 xhigh reasoning effort 加自动 workflow 编排。打开以后，Claude 会自己判断哪些任务值得跑 workflow。一个请求可能拆成几个连续 workflow：先理解代码，再实施，再验证。

它只在当前 session 生效。普通活儿做完后可以切回：

```
/effort high
```

官方还内置了一个 workflow：

```
/deep-research What changed in the Node.js permission model between v20 and v22?
```

这个命令会从多个角度搜索资料，抓取来源，交叉验证 claim，再生成带引用的报告。前提是 WebSearch tool 可用。

## 运行时怎么管

workflow 运行前会让你审批。CLI 会显示 planned phases，可以选择运行、以后同项目不再询问、查看 raw script、取消。`Ctrl+G` 能把脚本打开到编辑器里，`Tab` 可以在启动前调整 prompt。

Desktop app 里会出现 approval card，显示 workflow 名称、阶段列表和 token 使用提醒。你可以选 Once、Always 或 Deny。

跑起来后，用：

```
/workflows
```

这里能看到每个 phase 的 agent 数、token 总量、耗时。常用按键是 `Enter` 进入详情，`p` 暂停或恢复，`x` 停止，`r` 重启选中的 agent，`s` 保存成可复用命令。

保存时有两个位置：

```
.claude/workflows/
~/.claude/workflows/
```

项目级 workflow 适合进仓库，团队一起用；个人级 workflow 适合跨项目复用。保存后它会变成 slash command，后面直接 `/<name>` 就能跑。如果项目级和个人级同名，项目级优先。

## 权限和成本要认真看

官方文档有几个限制很实际。

workflow 中途不能等你输入业务决策。只有 agent 权限提示能暂停运行。如果某个阶段必须人工 sign-off，就拆成多个 workflow。

workflow 脚本不能直接读写文件或跑 shell。它只协调 agents，真实操作由 agents 执行。

并发最多 16 个 agents，低 CPU 机器可能更少。单次 run 最多 1,000 个 agents。

还有一个容易忽略的权限点：workflow 派出来的 subagents 总是在 `acceptEdits` 模式下运行，并继承你的 tool allowlist。文件编辑会自动批准。shell、web fetch、MCP 如果不在 allowlist 里，还是可能中途要权限。

所以第一次跑，我会建议只读审计：

```
Run a workflow to audit src/routes for missing auth checks.
Do not edit files. Return only verified findings with file paths and evidence.
```

等你确认结果质量，再跑修复型 workflow。

## 几个好用的 prompt

审计类可以这样写：

```
Run a workflow to audit [scope] for [risk].
Phase 1: map all relevant files and entry points.
Phase 2: assign independent agents by module; each agent must cite exact files and lines.
Phase 3: run verifier agents that try to disprove each finding.
Return only findings that pass verification, with severity, evidence, and recommended fix.
Do not edit files.
```

迁移类可以这样写：

```
Create a workflow to migrate [old API/framework] to [new API/framework] in [scope].
Start with an inventory and risk map. Split implementation by ownership boundary.
Use isolated agents per module. After each batch, run the relevant tests.
Stop and report if the same blocker appears in three modules.
Keep behavior unchanged unless the migration requires otherwise.
```

研究类直接用：

```
/deep-research [your question]
```

如果问题比较严肃，就补一句：

```
Prioritize primary sources and recent official docs.
Separate confirmed facts, source-backed inferences, and open questions.
```

## 什么时候别用

小任务别上 workflow。改一个按钮、修一个测试、解释一个函数，普通对话或 subagent 就够了。

需要频繁人工判断的任务也别硬跑。比如每一步都要产品确认、每个文件都要你选方案，这种任务拆成几个小 workflow 更稳。

同一批 agents 会抢同一片文件时也要谨慎。大迁移最好按目录、模块或 ownership boundary 拆清楚，prompt 里写明每个 agent 的编辑边界。

## 总结

Dynamic Workflows 真正改变的是 Claude Code 的工作形态：它开始把大任务写成可以运行的编排程序。这个能力会让代码库审计、迁移、研究、方案评审更像工程流水线。

用好它的关键不是喊一句“帮我跑 workflow”，而是把任务边界说清楚：范围在哪里，能不能改文件，证据格式是什么，怎么验证，什么时候停止。

我的判断是，团队里重复跑的高价值流程最适合被沉淀成 `.claude/workflows/*.js`。比如安全审计、迁移预检、发布前多角度 review、技术调研报告。跑通一次，保存下来，后面它就变成团队自己的 slash command。

这类工具最怕被当成炫技按钮。真正该交给 workflow 的，是那些并行探索能显著提高判断质量的任务。

## 参考来源

* Claude Code Dynamic Workflows 文档：https://code.claude.com/docs/en/workflows
* 官方发布博客：https://claude.com/blog/introducing-dynamic-workflows-in-claude-code
* Claude Code changelog：https://code.claude.com/docs/en/changelog
* 并行 agents 对比：https://code.claude.com/docs/en/agents