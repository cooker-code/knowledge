---
title: Scheduled Tasks 与 Routines:让 Claude 在你下班后继续交付
author: 匠心格物
date: 何谓第一等事何谓第一等事
url: https://mp.weixin.qq.com/s?__biz=MzkxNDczMDg0Nw==&mid=2247485359&idx=1&sn=fcd8ba7239314544b6e5d052d05e05df&chksm=c0efd6945741ca82c598e600b67dd23a979c8c0b8ad8a1c4a78c1fa6c8c9d19f3ce98be3b1c2&mpshare=1&scene=24&srcid=0429CRQ0M054DEZr04RBvP7l&sharer_shareinfo=fabd05a4f1c9aab1d1286f0c9a588e2d&sharer_shareinfo_first=fabd05a4f1c9aab1d1286f0c9a588e2d#rd
---

关键词：Claude Code、Scheduled Tasks、Claude /loop、Claude /schedule、Claude Routines、AI 自动化工作流、Claude Code Skill、PR 自动分诊、AI Code Review、自动处理 PR、AI 编程助手、Claude 自动化、开发者效率工具、Agent 工作流、AI 工程效率

  

Scheduled Tasks 与 Routines 封面

---

昨晚 11 点关了电脑去睡觉。今早 7 点打开 GitHub，三个 PR 已经合并了。

不是同事加班，是 Claude。

它在我睡觉的时候自动处理了 code review 的修改意见，自动 rebase 了冲突的分支，跑完 CI 后自动合并。我什么都没做。

这不是演示，是 Boris Cherny 每天在用的工作方式。他在推特上说:

> ★
>
> /loop and /schedule — two of the most powerful features.

这篇文章讲两件事:怎么让 Claude 在你不在的时候持续干活，怎么把你的工作流变成一个不需要你的自动化流水线。

---

## /loop:本地定时循环

`/loop` 是 Claude Code 的本地定时器。它在你的终端里循环运行一个命令或 Skill，间隔时间你定。

最简单的用法:

```
/loop 5m /babysit
```

意思是:每 5 分钟运行一次 `/babysit` 这个 Skill。

`/babysit` 是一个典型的"保姆型" Skill，它会:

1. 检查你所有 open 的 PR 有没有新的 review 评论
2. 如果有修改意见，自动改代码、提交、推送
3. 如果 CI 挂了，自动看日志、修 bug、重新跑
4. 如果一切通过，自动合并

一个 Skill，一个 loop，你的 PR 就有了一个 24 小时值班的助手。

### /loop 的参数

```
/loop 5m /skill-name       # 每 5 分钟运行一次  
/loop 30m /skill-name      # 每 30 分钟  
/loop 1h /skill-name       # 每小时  
/loop 7d /skill-name       # 最长 7 天  
/loop /skill-name           # 不指定间隔,上一轮跑完立刻开始下一轮
```

不指定间隔的模式适合那种"做完一轮就接着做下一轮"的任务，比如处理一个队列里的 issue。

### 限制

`/loop` 跑在你的本地终端。电脑不能关，终端不能退。

你关了笔记本盖子，loop 就停了。这是它最大的限制，也是 `/schedule` 存在的原因。

---

## /schedule:云端 cron

`/schedule` 把任务放到 Anthropic 的云端基础设施上跑。你的电脑关了也没关系。

```
/schedule "0 9 * * 1-5" /daily-standup
```

这是标准的 cron 表达式:工作日每天早上 9 点，运行 `/daily-standup`。

Claude 在云端启动一个临时环境，clone 你的 repo，运行 Skill，把结果推回 GitHub 或发通知给你。

### /schedule vs /loop

| 特性 | /loop | /schedule |
| --- | --- | --- |
| 运行位置 | 本地终端 | Anthropic 云端 |
| 电脑关了 | 停止 | 继续运行 |
| 最长运行时间 | 7 天 | 无限制(cron) |
| 访问本地文件 | 可以 | 不可以(只能访问 repo) |
| 适合场景 | 短期密集任务 | 长期定时任务 |

简单说:/loop 是"这几天帮我盯着"，/schedule 是"从今天起每天帮我做这件事"。

---

## Boris 的真实使用案例

  

Boris 的四个生产级 /loop

Boris 在推特上分享过他日常在用的几个 loop，每个都是跑着的生产工作流。

### `/loop 5m /babysit`

每 5 分钟检查一次 open 的 PR。发现 review 评论就自动处理:改代码、提交、推送。CI 通过后自动合并。

这是 Boris 用得最多的一个。他说自己现在基本不手动处理 PR review 了。Claude 改，他只在关键决策点看一眼。

### `/loop 30m /slack-feedback`

每 30 分钟扫一次 Slack 的反馈频道。发现用户报的 bug 或功能请求，自动分析可行性，如果是小改动就直接提 PR。

流程:

1. 读 Slack 频道最近的消息
2. 过滤出 bug 报告和功能请求
3. 评估改动范围
4. 小改动(< 50 行):直接写代码、跑测试、提 PR
5. 大改动:创建 GitHub Issue，打标签，@相关人

### `/loop /post-merge-sweeper`

不设间隔，持续运行。每次有 PR 合并后，自动检查是否有遗漏的 review 评论没处理。

有时候 PR 合并得急，reviewer 的一些"nice-to-have"建议被跳过了。这个 loop 会把它们捞出来，逐条处理，提新的 PR 解决。

### `/loop 1h /pr-pruner`

每小时检查一次所有 open 的 PR，清理那些过期的、不再需要的、或者跟 main 冲突太严重的。

规则:

* 超过 14 天没活动的 PR:发评论提醒作者
* 超过 30 天没活动的 PR:自动关闭，打 `stale` 标签
* 跟 main 冲突超过 5 个文件的 PR:尝试 rebase，失败就通知作者

---

## Routines(Beta):云端自动化编排

Routines 是 Claude Code 正在测试的云端自动化功能。你可以把它理解成"高级版 /schedule"，支持更多触发方式。

三种触发方式:

### 1. 定时触发(跟 /schedule 一样)

```
trigger:  
  schedule: "0 9 * * 1-5"  
action:  
  skill: /daily-standup
```

### 2. API 触发

```
trigger:  
  webhook: true  
action:  
  skill: /process-order
```

给你一个 webhook URL，任何系统都可以调用它来触发 Claude 运行指定 Skill。你的后端服务收到订单，调一下这个 URL，Claude 自动处理后续流程。

### 3. GitHub 事件触发

```
trigger:  
  github:  
    event: pull_request  
    action: opened  
action:  
  skill: /auto-review
```

有人开了 PR，Claude 自动跑 code review。不需要配 GitHub Actions，不需要写 CI 脚本。

Routines 还在 Beta，API 可能会变。但方向很清楚:Claude 正在从一个"你坐在电脑前跟它对话的工具"，变成一个可以嵌入整个工作流的自动化节点。

---

## 实战:搭建一个 PR 自动分诊的 /loop

  

3 步搭建 PR 自动分诊流水线

把前面说的串起来，搭一个真实的自动化流程。

### 第一步:写 Skill

创建 `.claude/skills/pr-triage/SKILL.md`:

```
---  
name:pr-triage  
description:自动分诊新PR。当用户说"分诊PR"、"triagePRs"时使用,  
或由/loop定时调用。  
allowed-tools:Bash,Read,Grep,Glob  
hooks:  
PreToolUse:  
    -matcher:Bash  
      command:|  
        echo "$CLAUDE_TOOL_INPUT_COMMAND" | grep -qiE '(merge|push|delete)' \  
        && echo "BLOCKED: 分诊模式下禁止合并、推送或删除操作" && exit 1 || true  
---  
  
# PR 自动分诊  
  
## 任务  
检查所有open的PR,为每个PR:  
1.读取diff,判断改动类型(feature/bugfix/refactor/docs/test)  
2.评估风险(high/medium/low)  
3.添加对应的label  
4.如果是low-risk+docs/test类型,自动approve  
5.如果是high-risk,@reviewer并留评论说明风险点  
  
## 约束  
-不要合并任何PR,只做分类和打标签  
-high-risk的判断标准:改了超过300行、涉及数据库migration、改了认证/支付模块  
-每个PR最多花2分钟分析,超时跳过
```

### 第二步:启动 loop

```
/loop 15m /pr-triage
```

每 15 分钟跑一次。新 PR 进来，15 分钟内就被自动分类、打标签、分配 reviewer。

注意 Skill 里的 on-demand Hook:分诊模式下，Claude 不允许合并、推送或删除。它只能读和打标签。这就是 13 篇讲的安全护栏在实战中的用法。

### 第三步:升级到 /schedule

跑了几天觉得稳定了，不想让电脑一直开着?

```
/schedule "*/15 * * * *" /pr-triage
```

迁移到云端，每 15 分钟自动运行。你关电脑、去度假，PR 照样被分诊。

---

## /loop + Skill:封装工作流的正确姿势

Boris 在同一条推文里还说了一句:

> ★
>
> experiment with turning workflows into skills + loops.

这句话是整个二期系列的一个缩影。

回头看一下:

* 08 篇讲了怎么把重复操作封装成 Skill
* 09 篇讲了怎么把常用提示封装成 Slash Command
* 13 篇讲了怎么用 Hooks 给 Skill 加安全护栏
* 这篇讲了怎么用 /loop 让 Skill 自动运行

串起来就是一条完整的自动化路线:

```
手动重复 → 封装成 Skill → 加上安全 Hook → 放进 /loop 自动运行
```

你从"每次手动告诉 Claude 怎么做"，变成"Claude 按 SOP 做、按护栏拦、按时间表跑"。你不在电脑前，它也在干活。

---

## 你的自动化搭对了吗

三个问题:

1. 你有没有把至少一个每天重复做的事变成 Skill + /loop? 如果没有，你还在人肉驱动 Claude。
2. 你的 /loop Skill 里有没有 on-demand Hook 防止它做超出范围的事? 如果没有，自动化就是自动闯祸。
3. 你有没有评估过哪些 loop 应该迁移到 /schedule? 如果你的笔记本每天开着只是为了跑 loop，那你在用最贵的服务器(你自己的电脑)干最便宜的活。

全答对了，你已经把 Claude Code 从"工具"用成了"团队成员"。

---

## 二期收尾

  

二期路线图：从手动到自动化

二期八篇，从 Plan Mode 到 Scheduled Tasks，讲的其实就一件事:让 Claude 从一个"你说什么它做什么"的助手，变成一个"你不说它也知道该做什么"的同事。

Plan Mode 让它先想再做。Skill 让它记住怎么做。Hooks 让它不敢乱做。/loop 让它在你不在的时候继续做。

给它越多结构，它回报越多产出。就这么简单。

---

## 第三期预告

二期讲的是"怎么让一个 Claude 变强"。三期要讲的是"怎么让多个 Claude 协作"。

Multi-agent 编排、Agent 之间的通信协议、用 Claude Code SDK 构建生产级 agent 系统，以及 Boris 团队内部正在探索的"agent mesh"架构。

三期第 15 篇:Multi-agent 编排入门，让你的 Claude 们学会分工。

---

## 本文引用的推文

* /loop and /schedule — two of the most powerful features · @bcherny
* experiment with turning workflows into skills + loops · @bcherny