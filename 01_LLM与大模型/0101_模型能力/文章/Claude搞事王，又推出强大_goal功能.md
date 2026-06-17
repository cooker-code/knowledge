---
title: Claude搞事王，又推出强大/goal功能
author: X Agentic
date: 丢钢笔的人丢钢笔的人
url: https://mp.weixin.qq.com/s?__biz=MzYzODA4ODUxOQ==&mid=2247484154&idx=1&sn=df4780d88997ce976f988d5a50cadd83&chksm=f198816362b2cc5869b3756d46f7f45fa6c2e2f29ebd55ca46f82a9f01326f29a53bf3232c73&mpshare=1&scene=24&srcid=0514lGF5XeSlply6o9eSEj77&sharer_shareinfo=cf698d5a53b4b1519367ad23be3bf980&sharer_shareinfo_first=cf698d5a53b4b1519367ad23be3bf980#rd
---

> Claude Code 推出 /goal 命令：让 AI 自己干活，直到"真的干完"，你只需要告诉它目标，剩下的它会自己来——反复迭代、自我纠错、直到条件满足。

---

昨天凌晨，Anthropic 官方开发者账号 @ClaudeDevs 发布了一条推文，正式介绍 Claude Code 最近上线的新功能：**`/goal`**。截至发稿，该推文已获得 **超过 1.1W 个赞**和 1091 次转发。

> "How do you keep Claude working until the job is done? Claude Code helps with this in a few ways, including one we shipped recently: /goal."

一句话概括：**`/goal` 让你设定一个"完成条件"，Claude 会自己跨轮次持续工作，直到条件满足才停下。**

---

## 什么是 /goal？

在 Claude Code 中，你以往每次都要手动输入 prompt、查看结果、再输入下一步。`/goal` 打破了这种"你来我往"的模式——它让 Claude 变成一个真正意义上的**自主代理（autonomous agent）**。

工作流程如下：

1. 你用 `/goal` 设定一个**可验证的完成条件**
2. Claude 开始干活，完成一个 turn 后不会把控制权还给你
3. 背后一个**小规模快速模型**（默认 Haiku）根据对话内容评估条件是否满足
4. 如果不满足，Claude 会带着评估建议进入下一轮，继续工作
5. 直到条件达成，goal 自动清除，控制权归还给你

核心设计思路很巧妙：**干活用一个强模型，判断"干没干完"用小模型**。这样既保证了执行质量，又控制了成本——评估阶段的 token 消耗相比主模型几乎可以忽略不计。

---

## 如何使用 /goal

### 基础用法

在 Claude Code 会话中直接输入：

```
/goal test/auth 目录下所有测试通过且 lint 检查无报错
```

执行后命令会立即启动，终端中会出现 `◎ /goal active` 状态指示器，显示 goal 已运行的时间、turn 数、token 消耗以及最近的评估结果。

### 查看进度

随时输入 `/goal`（不带参数）查看当前状态：

* 正在执行的 goal 显示运行时长、turn 数、token 消耗、评估原因
* 已完成 goal 显示达成条件、耗时和资源消耗

### 中途停止

如果觉得方向不对或目标已不需要继续：

```
/goal clear
```

`stop`、`off`、`reset`、`none`、`cancel` 均为等价别名。

### 跨会话恢复

如果一个 goal 在会话结束时尚未完成，使用 `--resume` 或 `--continue` 恢复会话时，goal 会自动恢复执行。

### 非交互模式

在 CI/CD 或脚本中使用：

```
claude -p "/goal CHANGELOG.md 包含本周所有已合并 PR 的记录"
```

Ctrl+C 可在条件达成前中断。

---

## 写好条件的技巧

`/goal` 的评估器（evaluator）不直接运行命令或读文件，它只能根据 Claude 在对话中呈现的内容做判断。所以条件必须是 Claude 的输出可以**展示**的。

好的条件通常包含三要素：

| 要素 | 说明 | 示例 |
| --- | --- | --- |
| 可度量的终态 | 一个明确的可验证结果 | "`npm test` exit code 为 0" |
| 明确的检查方式 | Claude 如何证明完成 | "运行 `npm test` 并展示输出" |
| 约束条件 | 过程中不能变的东西 | "不修改其他测试文件" |

几个典型的实战用例：

* **迁移 API**

  ：`/goal 所有 src/ 下的文件都引用了新的 API 且 npm run build 成功`
* **拆分大文件**

  ：`/goal src/handlers.ts 被拆分成多个模块，每个不超过 200 行，测试全部通过`
* **清理 issue 队列**

  ：`/goal docs/bugs.md 中标记的所有 bug 都已被修复，关联测试通过`

条件最长可达 4000 字符。如果担心无限执行，可以在条件中加上限制：`或者在 20 个 turn 后停止`。

---

## /goal vs. 其他自主模式

Claude Code 目前有三种"不间断工作"的方式，各有适用场景：

| 方式 | 触发条件 | 停止条件 |
| --- | --- | --- |
| `/goal` | 上一轮结束 | 模型判断条件满足 |
| `/loop` | 固定时间间隔 | 用户手动停止或 Claude 自行判断完成 |
| Stop Hook | 上一轮结束 | 自定义脚本或 prompt 决定 |

如果你想要"达到某个状态就停"，用 `/goal`；如果只是定期检查（比如每 10 分钟看一次部署状态），用 `/loop`；如果需要复杂的自定义判断逻辑，用 Stop Hook。

搭配 **Auto Mode**（自动批准工具调用），可以实现完全无人值守的编码工作——工具调用不用确认，turn 之间也不用你说"继续"。

---

## 用户反响

从社交媒体来看，开发者们对这功能的反应相当热烈。以下是一些典型评价：

> "给 Claude 设了一个 /goal，然后去煮咖啡……回来看到一整个仓库建好了。"

> "/goal 这个东西……有点疯狂。它自己分支、建项目结构、写代码，我就坐在旁边看着它。"

> "看着 Claude Code /goal 跑就像在看 Roomba 扫地机器人，不过是写代码版的……它就一直走一直走。"

> "okay yeah this changes things."（好吧，这会改变很多事情。）

也有部分用户提醒：如果目标设得太模糊或范围太大，可能会导致大量 token 消耗却偏离轨道。**写得越具体，效果越好。**

---

## 总结

`/goal` 是 Claude Code 从"对话式 AI 助手"迈向"自主编码代理"的关键一步。它把人类从"一步步指挥"中解放出来，让你从执行者变成监督者——定好目标，设定门槛，然后放手让它跑。

配合 Auto Mode 和 Headless 模式，Claude Code 已经可以在很多场景下真正做到"无人值守编码"。对于开发者来说，这意味着你可以同时推进多个任务，或者把耗时工作留到后台，自己专注于更需要创造力的部分。

感兴趣的话，打开终端，给 Claude 设个 goal 试试吧。

这里是钢笔的 AI 前沿速递栏目，持续关注 AI 开发工具的最新动态。欢迎点赞、关注、在评论区分享你使用 /goal 的体验！