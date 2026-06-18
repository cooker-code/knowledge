> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020114_长任务Agent运行时/020114_核心知识点/Ralph Loop持续迭代边界|Ralph Loop持续迭代边界]]
---
title: Ralph Loop：让 AI Agent 自己干到完的秘密武器
author: tc9011
date:
url: https://mp.weixin.qq.com/s?__biz=MzU0OTE1ODE5NA==&mid=2247484608&idx=1&sn=80acf7c71dae651b6553c0d5d3b4995c&chksm=fadcc5800d7b595e4e89cc1800df95e4e1021a1db128e59c2152581765dc106aa7ef0d8aa51e&mpshare=1&scene=24&srcid=04190HlbggW59bRu3m4U822O&sharer_shareinfo=41f92d8ad60c2b0a2faac1af05ecd1c4&sharer_shareinfo_first=41f92d8ad60c2b0a2faac1af05ecd1c4#rd
---

你有没有经历过这种场景：让 AI Agent 帮你实现一个功能，它干到一半，上下文窗口满了，然后客客气气地告诉你"我已经完成了主要部分，剩下的你可以手动调整"？或者更常见的——你给了它一个复杂任务，它做了 70%，漏掉了边界情况，你不得不重新开一轮对话、重新喂上下文、重新解释需求？

Ralph Loop 就是为了解决这个问题而生的。

## 一句话解释

Ralph Loop 是一个**自引用开发循环**：把 AI Agent 的输出重新喂回输入，让它持续工作，直到任务真正完成。不是你说完成就完成——是系统验证完成才算完成。

## 名字的来历

2025 年 7 月，澳大利亚开发者 Geoffrey Huntley[1] 发了一篇博客，标题是 *"Ralph Wiggum as a software engineer"*。Ralph Wiggum 是《辛普森一家》里那个看起来傻乎乎的小孩——他经常走错路、说错话，但如果你在他的环境里放上正确的指示牌（signs），他最终总能到达目的地。

这个比喻精准地描述了 AI Agent 的行为模式：单次对话可能会犯错、会遗漏，但如果你**让它在一个循环里反复尝试**，通过文件系统和 git 历史保持状态，通过测试结果提供反馈，它最终能把事情做完。

这个概念一经提出就在开发者社区炸开了锅。VentureBeat 专门写了一篇报道：How Ralph Wiggum went from The Simpsons to the biggest name in AI right now[2]。Twitter 上有人用 Ralph Loop 一晚上把一个 45,000 行的 Tauri 桌面应用转成了 SaaS Web 应用。

## 为什么传统 Agent 对话模式不够用

先搞清楚问题在哪。

### 上下文窗口是一堵墙

目前的大模型都有上下文窗口限制。即便 Claude 已经支持 1M tokens，一个真正复杂的任务——比如重构一个中型项目的认证系统——很容易就会耗尽上下文。对话进行到后半段，模型开始"忘记"前面的内容，输出质量断崖式下降。

### 单轮对话缺乏纠错机制

传统工作流是这样的：

```
1



你描述需求 → Agent 一口气干完 → 你检查结果 → 发现问题 → 重新开一轮
```

问题在于"重新开一轮"的成本极高。你需要重新描述上下文、重新加载文件、重新解释之前做了什么。每次重启都是一次信息损失。

### Agent 倾向于"提前宣布完成"

这是一个被广泛观察到的行为模式：AI Agent 在完成 80% 的工作后，倾向于宣布任务完成。它不会主动跑测试来验证，不会检查边界情况，不会确认所有 TODO 都被处理。它的"完成"更像是"我写完代码了"，而不是"这个功能可以交付了"。

## Ralph Loop 的核心设计

Ralph Loop 的设计哲学可以用一句话概括：**每一轮都是全新的开始，但进度永远不会丢失。**

### 状态不在对话里，在文件系统里

这是 Ralph Loop 和普通"多轮对话"的本质区别。传统多轮对话依赖聊天历史来保持上下文，一旦历史太长，信息就会被截断或遗忘。

Ralph Loop 反其道而行之：每一轮循环都是一个**全新的 Agent 会话**。没有聊天历史。Agent 需要的一切信息都来自文件系统——代码文件、git 提交历史、测试输出、TODO 清单。

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14



┌─────────────────────────────────────┐
│          文件系统 / Git             │
│  (PROMPT.md, progress.md, 测试结果) │
└─────────┬───────────────────────────┘
          │ 读取
          ▼
┌─────────────────────┐
│   AI Agent 第 N 轮   │──→ 修改代码 → git commit
└─────────────────────┘
          │ 退出
          ▼
┌─────────────────────┐
│   AI Agent 第 N+1 轮 │──→ 读取最新状态 → 继续工作
└─────────────────────┘
```

### 自愈反馈循环

每一轮循环中，Agent 能看到上一轮的结果——包括失败的测试、编译错误、lint 警告。这意味着上一轮犯的错误，在下一轮会被自动纠正。

这就像人类开发者的工作方式：写完代码跑测试，测试挂了就修，修完再跑，直到全部通过。只不过 Ralph Loop 把这个过程自动化了。

### 显式完成信号

Agent 不能自己宣布完成。它必须输出一个明确的完成标记（completion promise），例如 `<promise>DONE</promise>`。如果它不输出这个标记，循环就会自动继续——系统会注入一条新的提示，让 Agent 继续工作。

这个设计解决了"Agent 提前宣布完成"的问题。你可以在提示中定义"完成"意味着什么：所有测试通过、lint 无警告、功能可以端到端运行。Agent 必须达到这些条件才能输出完成标记。

## 最简实现：一行 Bash

Geoffrey Huntley 的原始实现简单到令人发指：

```
1



while :; do cat PROMPT.md | claude -p --dangerously-skip-permissions; done
```

就这么一行。`while :` 是无限循环，每次循环把 `PROMPT.md` 的内容通过管道喂给 Claude Code，然后 Agent 工作、退出（上下文耗尽或它认为做完了），循环重新开始。

`PROMPT.md` 是整个循环的灵魂。一个典型的 `PROMPT.md` 长这样：

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21



# 你的任务
 
把 src/auth 模块从 session-based 迁移到 JWT-based 认证。
 
## 完成标准
 
- [ ] 所有现有测试通过
- [ ] 新增 JWT token 生成和验证的测试
- [ ] 登录、注册、密码重置流程端到端可用
- [ ] 没有硬编码的 secret
 
## 规则
 
- 每次只做一件事
- 做完一件事就 git commit
- 跑完测试再继续下一件
- 如果卡住了，换一种方法
 
## 当前进度
 
查看 git log 了解已完成的工作。
```

关键洞察：**PROMPT.md 不是一次性的指令，而是 Agent 每轮循环都会重新读取的"操作手册"**。你可以在循环运行期间随时编辑它——加新规则、调整优先级、修正方向。Agent 下一轮就会读到更新后的内容。

## 进阶实现：Ralphify

Ralphify[3] 是目前最成熟的 Ralph Loop 独立工具。它在原始 Bash 循环的基础上增加了几个关键能力：

### 动态占位符

RALPH.md（Ralphify 版的 PROMPT.md）支持 `{{ }}` 占位符，循环每次迭代时会自动填充：

```
1

2

3

4

5

6

7



## 当前测试状态
 
{{ commands.tests }}
 
## 最近的 Git 日志
 
{{ commands.git_log }}
```

每轮循环开始前，Ralphify 会执行这些命令，把实际输出注入到提示中。Agent 不需要自己去跑 `git log` 或 `pytest`——它直接在提示里就能看到最新状态。

### 六步循环

Ralphify 的每次迭代遵循固定流程：

1. 1. **重新读取 RALPH.md**（支持运行时热编辑）
2. 2. **执行占位符命令**（测试、git log、lint 等）
3. 3. **解析占位符**，注入命令输出
4. 4. **组装完整提示**
5. 5. **发送给 Agent**
6. 6. **Agent 退出 → 循环回第 1 步**

安装和使用很直接：

```
1

2

3



uv tool install ralphify
ralphify init          # 生成 RALPH.md 模板
ralphify start         # 启动循环
```

## Oh My OpenCode 内置的 Ralph Loop

如果你已经在用 Oh My OpenCode[4]（OpenCode 的增强插件），Ralph Loop 是开箱即用的。不需要额外安装任何东西。

### 基础版：`/ralph-loop`

在 OpenCode 中直接输入：

```
1



/ralph-loop "把 src/auth 模块迁移到 JWT 认证"
```

系统会启动一个自引用循环。Agent 持续工作，直到它输出 `<promise>DONE</promise>` 完成标记，或者达到最大迭代次数（默认 100 次）。

完整的参数格式：

```
1



/ralph-loop "任务描述" [--completion-promise=TEXT] [--max-iterations=N] [--strategy=reset|continue]
```

* • `--completion-promise`：自定义完成标记，默认是 `DONE`
* • `--max-iterations`：最大迭代次数，默认 100
* • `--strategy`：`reset` 每轮重置上下文，`continue` 保持上下文延续

### 加强版：`/ulw-loop`（Ultrawork Loop）

这是 Ralph Loop 的升级版。区别在于：Agent 宣布完成后，系统不会直接结束循环，而是**要求 Oracle（一个独立的高推理能力 Agent）进行验证**。只有 Oracle 确认结果符合要求，循环才会真正结束。

```
1



/ulw-loop "重构整个支付模块"
```

Ultrawork Loop 的最大迭代次数是 500 次（普通 Ralph Loop 是 100 次），适合真正大型的任务。

### 底层机制

Oh My OpenCode 的 Ralph Loop 通过 Hook 系统实现：

1. 1. **状态持久化**：循环状态保存在 `.sisyphus/ralph-loop.local.md` 文件中，包括当前迭代次数、任务描述、策略等
2. 2. **完成检测**：Hook 在每轮 Agent 输出中检测 `<promise>DONE</promise>` 标签
3. 3. **自动继续**：如果没有检测到完成标记，系统自动注入一条新的提示，让 Agent 继续工作
4. 4. **迭代计数**：每轮自动递增，达到上限后强制停止
5. 5. **取消机制**：随时可以用 `/cancel-ralph` 终止循环

### 和 Todo Enforcer 的配合

Oh My OpenCode 还有一个 Todo Enforcer 机制——如果 Agent 在循环中试图"摸鱼"（停止工作但不输出完成标记），系统会检测到未完成的 TODO 项，自动把 Agent 拉回来继续干。Ralph Loop + Todo Enforcer 的组合，基本堵死了 Agent 偷懒的所有路径。

## 其他 Ralph Loop 实现

生态里还有不少其他实现，各有侧重：

| 工具 | 特点 | 适用场景 |
| --- | --- | --- |
| snarktank/ralph[5] | 最早的 PoC 之一，Bash 脚本 + PRD 驱动 | 想要最简单的入门方式 |
| PageAI ralph-loop[6] | Docker 沙箱隔离，PRD 驱动 | 需要安全隔离的环境 |
| Chief[7] | 完整 TUI，Worktree 隔离，用户故事驱动 | 想要可视化管理的团队 |
| Agent Utils[8] | OpenCode 插件，轻量 | 已经在用 OpenCode |
| Monitored Ralph Loop[9] | 事件驱动，支持推送通知 | 需要离开电脑后远程监控 |

## 实战建议

### 1. 任务粒度要合适

Ralph Loop 不是万能的。它最适合**边界清晰、可以通过测试验证的任务**：

* • ✅ "把项目从 ESLint 8 迁移到 ESLint 9"
* • ✅ "给所有 API 接口加上输入验证"
* • ✅ "修复所有 TypeScript 严格模式下的类型错误"
* • ❌ "设计一个新的产品架构"（太开放）
* • ❌ "优化用户体验"（没有明确的完成标准）

### 2. 写好完成标准

PROMPT.md / RALPH.md 里最重要的部分不是任务描述，而是**完成标准**。它必须是可验证的：

```
1

2

3

4

5

6



## 完成标准
 
1. `pnpm test` 全部通过
2. `pnpm lint` 零警告
3. `pnpm build` 成功
4. 所有新增的 API 都有对应的测试用例
```

模糊的标准（"代码质量要好"）会让 Agent 要么过早宣布完成，要么永远不敢说完成。

### 3. 善用 Git 作为状态

每完成一步就 commit 是 Ralph Loop 的黄金法则。这样即使某一轮循环搞砸了，下一轮可以通过 `git log` 和 `git diff` 看到发生了什么，甚至可以 `git revert` 回退。

在 PROMPT.md 里加上这条规则：

```
1

2

3

4

5



## 规则
 
- 每完成一个独立的改动就 git commit
- commit message 要说明做了什么、为什么
- 如果改动引入了测试失败，立即回退并换一种方式
```

### 4. 监控和介入

Ralph Loop 不意味着完全放手。建议：

* • **定期检查 git log**，确认 Agent 的方向是对的
* • **随时编辑 PROMPT.md**，加入新的约束或修正方向
* • **关注 API 成本**，设置合理的迭代上限
* • **有条件的话用 Ultrawork Loop**，让 Oracle 自动验证

### 5. 成本意识

每轮循环都会消耗 API tokens。一个 100 轮的 Ralph Loop，如果每轮平均消耗 50K tokens（输入 + 输出），总计就是 5M tokens。按照 Claude Opus 的定价，这不是一笔小数目。

控制成本的策略：

* • 给大型任务设置迭代上限
* • 用更便宜的模型做初始迭代，用更强的模型做最终验证
* • 在 Oh My OpenCode 中，Ultrawork 模式已经内置了多模型编排——用 Claude/Kimi 做编排，用 GPT 做推理

## 局限性

Ralph Loop 不是银弹。需要注意的坑：

1. 1. **发散风险**：如果完成标准不明确，Agent 可能在循环中反复做无用功——加了又删、改了又改。设置合理的迭代上限是安全网。
2. 2. **累积错误**：Agent 在第 5 轮做了一个错误的设计决策，后面 20 轮都在这个错误的基础上继续建设。定期人工审查 git log 可以尽早发现这类问题。
3. 3. **不适合创造性任务**：Ralph Loop 适合"把明确的事做完"，不适合"想清楚应该做什么"。需要架构设计、产品决策的部分，应该在进入循环之前由人类完成。
4. 4. **成本失控**：没有迭代上限的 Ralph Loop 可能会跑上几百轮，尤其是遇到无法通过的测试时。始终设置 `--max-iterations`。

## 总结

Ralph Loop 的核心洞察很简单：**AI Agent 不可靠，但足够的迭代可以让它变得可靠。**

就像 Ralph Wiggum——他可能会走错路，但只要环境里有足够的指示牌，他总会到达目的地。关键不在于让 Agent 一次做对，而在于建立一个系统，让它在反复尝试中趋向正确。

这和软件工程里很多经典思想是一脉相承的：最终一致性、重试机制、自愈系统。只不过现在，我们把这些思想应用到了 AI Agent 的编排上。

如果你已经在用 OpenCode + Oh My OpenCode，直接试试 `/ralph-loop`。如果没有，一行 Bash 就能开始：

```
1



while :; do cat PROMPT.md | claude -p --dangerously-skip-permissions; done
```

先在一个小任务上试试。感受一下 Agent 在循环中自我纠正、逐步推进的过程。然后你会理解，为什么这个以动画角色命名的技术，正在改变开发者和 AI Agent 协作的方式。

#### 引用链接

`[1]` Geoffrey Huntley: *https://ghuntley.com/ralph/*
`[2]` How Ralph Wiggum went from The Simpsons to the biggest name in AI right now: *https://venturebeat.com/technology/how-ralph-wiggum-went-from-the-simpsons-to-the-biggest-name-in-ai-right-now*
`[3]` Ralphify: *https://ralphify.co*
`[4]` Oh My OpenCode: *https://github.com/code-yeongyu/oh-my-openagent*
`[5]` snarktank/ralph: *https://github.com/snarktank/ralph*
`[6]` PageAI ralph-loop: *https://github.com/PageAI-Pro/ralph-loop*
`[7]` Chief: *https://chiefloop.com*
`[8]` Agent Utils: *https://brennanmceachran.github.io/agent-utils/*
`[9]` Monitored Ralph Loop: *https://github.com/openclaw/skills*