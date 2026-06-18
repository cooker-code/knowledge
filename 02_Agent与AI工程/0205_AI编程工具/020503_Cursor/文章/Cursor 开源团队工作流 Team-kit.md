---
title: Cursor 开源团队工作流 Team-kit
author: Vibe编码
date: VibeCoderVibeCoder
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODkzOTY3MA==&mid=2247485112&idx=1&sn=ddb37f1d13f0cb76bad3be98f5da9e62&chksm=c4fcfae368426ad60a2bb146c74de8c43a01b8aa1b915b0eb6a78b80e65d3e21dec014a3776e&mpshare=1&scene=24&srcid=0509vHvrQQ4ffYMc8jOINYN7&sharer_shareinfo=91409012255ebf696722ec8b6e67d500&sharer_shareinfo_first=91409012255ebf696722ec8b6e67d500#rd
---

Cursor 官方插件仓库里多了一个很有意思的东西：`cursor-team-kit`。它把 Cursor 团队内部用于 CI、Code Review、发 PR、测试、验证、代码清理、周报的工作流，打包成了一个可以直接安装的插件。

安装方式很短：

```
/add-plugin cursor-team-kit
```

## 它是什么

这个插件当前是 `1.1.0`，manifest 里写得很直白：Internal workflows used by Cursor developers。它包含 17 个 Skills、1 个 Sub Agent、2 条 Rules。

官方 README 也强调了一点：plug and play without requiring third-party service integrations。翻成工程语言就是，它不强绑 Linear、Jira、Slack、Notion 这类系统，主要靠 Git、GitHub CLI、本地测试、浏览器自动化、终端 harness 这些基础能力工作。

这点挺关键。很多插件的价值是把某个 SaaS 接进 IDE。Team Kit 的价值更像把一个团队怎么收尾、怎么验证、怎么 review，写成 agent 能读懂的操作手册。

## 技术原理

Cursor 插件可以打包很多东西：Skills、Subagents、MCP servers、Hooks、Rules。Team Kit 这次只用了三类：skills、agents、rules。

目录结构大概是这样：

```
cursor-team-kit/  
├── .cursor-plugin/plugin.json  
├── agents/ci-watcher.md  
├── rules/  
│   ├── no-inline-imports.mdc  
│   └── typescript-exhaustive-switch.mdc  
└── skills/  
    ├── loop-on-ci/  
    ├── verify-this/  
    ├── control-cli/  
    ├── control-ui/  
    ├── pr-review-canvas/  
    └── ...
```

最明显的设计是：Skills 很多，Sub Agent 很少。

整个 kit 只有一个后台 agent：`ci-watcher`，负责盯当前分支 PR 的 checks。其他复杂动作都拆成独立 skill，比如修 CI、跑 smoke tests、生成 PR walkthrough、验证一个 claim、驱动 UI、清理 AI 生成代码。

这是一种挺务实的拆法。长期常驻的 agent 越多，状态越复杂，行为越难预测。按需触发的 skill 更像 checklist，触发条件、步骤、guardrails、输出都写在文件里，出了问题也更容易读源码定位。

## 先看 verify-this

`verify-this` 是我觉得最有价值的 skill。

它的要求很硬：先把用户的 claim 改写成可证伪命题，再采集 baseline 和 treatment 两组 artifact，用相同命令、相同数据、相同环境比较，最后只允许三种结论：

```
VERIFIED  
NOT VERIFIED  
INCONCLUSIVE
```

这个设计直击 agent 写代码最大的毛病：太容易凭感觉宣布完成。

测试没跑完，UI 没打开，性能没量过，日志没看清楚，模型也可能直接说已修复。`verify-this` 把这件事拦住了。你要说修好了，就拿证据。证据不足，就给 `INCONCLUSIVE`。没修好，就给 `NOT VERIFIED`。

这个 skill 完全可以迁移到 Claude Code、Codex、OpenCode 或任何本地 coding agent 里。它不依赖 Cursor 特有能力，依赖的是工程纪律。

## control-cli 和 control-ui 是执行层

`control-cli` 和 `control-ui` 也很值得单独看。

`control-cli` 的目标是给交互式 CLI/TUI 搭一个可重复 harness。它会优先找 repo 自己的测试或 demo 工具，没有的话就用 tmux、PTY、Expect、Node inspector 这类本地工具来驱动输入、捕获屏幕、记录 transcript、做 profile。

`control-ui` 则面向 Web、IDE、Electron。它会用 Playwright 或 CDP 去连接真实页面，截图、读 accessibility tree、抓 console/network、做性能和内存分析。

这两个 skill 说明 Cursor 内部对 agent 的要求已经不止是读代码和写代码。代码写完之后，agent 要真的去操作它，点页面，跑命令，截证据，再回来修。

这也是我认为 Team Kit 对普通团队最有启发的地方。现在很多 AI 编程流程还停在生成代码加跑测试。但大量 bug 出在交互层：按钮 focus、终端 resize、弹窗遮挡、滚动区域、快捷键、浏览器 console error。这些靠读代码很难看全。

## PR 被当成阅读体验来设计

Team Kit 里还有一组 PR 生命周期工具：

```
new-branch-and-pr  
review-and-ship  
make-pr-easy-to-review  
get-pr-comments  
pr-review-canvas
```

`make-pr-easy-to-review` 这个名字很直接。它专门整理 noisy history、改 PR 描述、补风险说明、给 reviewer 标出入口文件和注意点。

更夸张一点的是 `pr-review-canvas`。它会抓 PR 数据，把 diff 渲染成一个交互式 HTML 走读页面。核心文件展开，机械改动折叠，还能加伪代码、流程图、review checklist。

AI 让代码产出速度变快后，review 压力会明显上升。一个 PR 横跨多个模块时，GitHub 默认 diff 视图的信息组织能力不够。Cursor 这个 skill 其实是在承认一个现实：审查代码也要做信息设计。

## 两条 Rules 暴露了代码品味

Team Kit 只有两条 always-on rules：

```
typescript-exhaustive-switch  
no-inline-imports
```

第一条要求 TypeScript union/enum 的 switch 做穷尽处理，通常用 `never` 兜住。第二条要求 import 放文件顶部，不在函数体或类型注解里随便 inline import。

这两条都不花哨，但它们共同指向一个偏好：代码要容易被静态分析，也要容易被人扫读。

我不建议把这两条当成完整规范。它们更像种子。真正落地时，团队应该在这个基础上补自己的规则，比如错误处理、数据迁移、feature flag、埋点、权限校验、设计系统组件使用方式。

## workflow-from-chats 是长期价值

`workflow-from-chats` 很有意思，因为它是一个元技能。

它会从最近对话里提取团队偏好：触发条件、工作步骤、质量标准、停止条件、证据和置信度。然后判断这些偏好该写成 skill、rule、workflow doc，还是不落地。

这解决的是一个长期问题：人会不断纠正 agent。

比如：别乱加 fallback。别改无关文件。UI 改完要截图验。PR 描述要写风险和测试。

如果这些纠正只留在单次对话里，下一次还要重新说。`workflow-from-chats` 的方向是把纠正沉淀成组织知识。团队真正用 agent 用得久，最后拼的一定是这个能力。

## 怎么用

如果只是想试，可以直接装：

```
/add-plugin cursor-team-kit
```

但我更建议先读源码，再分批启用。

第一批可以看 `verify-this`、`control-ui`、`control-cli`、`deslop`。这几个最容易提升质量，也不太依赖 GitHub 工作流。

第二批再看 `fix-ci`、`loop-on-ci`、`ci-watcher`、`run-smoke-tests`。这些适合已经有稳定 CI、稳定测试命令的团队。

第三批看 `make-pr-easy-to-review`、`pr-review-canvas`、`workflow-from-chats`。这部分和团队 review 文化、隐私边界、聊天记录治理关系更大，要谨慎改。

如果你的团队不用 GitHub Actions，很多命令需要替换。比如 `gh pr checks` 要换成 GitLab pipeline、Buildkite build、Jenkins job 或内部 CI 的查询命令。Team Kit 的文件结构值得学，命令层不用原封不动搬。

## 总结

Cursor Team Kit 最有价值的地方，是它给了一个很具体的答案：团队该怎么把 AI agent 放进日常工程流程。

它没有把重点放在更快生成代码。它关心的是 CI 红了怎么回绿，PR 怎么让人容易看，claim 怎么拿证据验证，UI/CLI 怎么真实操作，反复纠正的偏好怎么沉淀。

如果团队已经开始大量用 AI 写代码，这套东西值得花时间通读。它开源的是一套工程收尾习惯，工具能力只是载体。