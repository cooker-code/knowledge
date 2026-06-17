---
title: 再聊聊 Claude Code 工作流：这次让 AI 自己组队跑
author: sanvi.dev
date: 卢灿伟同学卢灿伟同学
url: https://mp.weixin.qq.com/s?__biz=MzA3MjQ2NDM3MA==&mid=2648914526&idx=1&sn=812f06cfc8c8f7e35f038fb9d90e2f06&chksm=861dd70b989ae7008cd27205f236f52a9426ebd7bf5cba477816cf383457a6de49bfa40b1715&mpshare=1&scene=24&srcid=0516Z1GdtW9qPG4Kxx50U59J&sharer_shareinfo=06d8b2d14239bf7c3d3146fd05841176&sharer_shareinfo_first=06d8b2d14239bf7c3d3146fd05841176#rd
---

上一篇讲那套「不丢上下文」的工作流[我用 Claude Code CLI 搭了一套「不丢上下文」的工作流](https://mp.weixin.qq.com/s?__biz=MzA3MjQ2NDM3MA==&mid=2648914407&idx=1&sn=58d017ce7aa25c1fc3ff2b08309672a9&scene=21#wechat_redirect)（checkpoint / recap / postmortem / init-project），用了大半年，主体没大改。但最近又踩了几个新坑，所以又加了三个新命令——`/handoff`、`/cross-review`、`/kickoff`。

按时间顺序讲。

## 前情提要：Opus 1M 上下文带来的反转

先交代一个背景，不然后面的逻辑接不上。

前阵子 Anthropic 把 Opus 的上下文窗口从默认拉到了 1M，我第一时间就升级了。一升级就感觉到差别——以前一个深度讨论 4 小时要被 compact 打断 2-3 次，现在开 8 小时都不带触发的。

看起来上下文问题被彻底解决了。

但用了几天发现不对——一个比 compact 还难受的问题被暴露出来：**失真**。

之前 compact 触发的时候，Claude 会主动拉一遍最近的对话和文件做摘要，多少能"重新整理一下思路"。1M 上下文不触发 compact 了，反而越聊越漂——内容全在 context 里，但权重已经偏了。

什么叫失真？举个例子：你跟它讨论一个大功能，开 4 个小时，中间否决过方案 A、确认了方案 B、又补充了三个边界条件。然后它突然给你建议方案 A，理由头头是道。

它没忘。但注意力权重已经偏向最近的对话——离当前越近权重越高，远的虽然还在 context 里，但它"用不到"。1M 上下文不仅没救这个问题，反而让你能聊得更长，从而更容易触发失真。

## 坑 1：checkpoint 救不了失真

我一开始的本能反应是开新 session，反正有 `/recap`。但 `/recap` 能恢复 progress 和工作日志，**救不了"这个任务做到一半，思路是什么"**——这种半成品状态，progress.md 里没有（还没完成不会写），日志里也没有（checkpoint 只记完成项和决策）。

你在新 session 重新讲一遍？讲不清楚——你自己也忘了你在中间想到的那个细节。

所以问题被拆成了两个：

* **当前 session 内**防上下文丢失，靠 `/checkpoint`
* **跨 session 接棒**，需要一个完全不同的东西

## /handoff —— 自包含的交接包

`/handoff` 和 `/checkpoint` 长得像，但本质完全不同：

**`/checkpoint`**

* 给谁看：当前 session 自己
* 内容形态：增量记录
* 触发时机：任务途中、随手存
* 输出文件：追加到 `docs/memory/YYYY-MM-DD.md`

**`/handoff`**

* 给谁看：下一个 session（可能换了模型 / 换了机器）
* 内容形态：完整自包含
* 触发时机：感觉要失真、要换 session 之前
* 输出文件：覆盖写入 `docs/HANDOFF.md`

关键词是 **"自包含"**。意思是接棒人不需要任何对话历史、不需要看 git log、不需要问你，光读这一份文件就能直接动手。

七个必填板块（少一个都接不上）：

1. **任务（Mission）**：1-2 句话讲在做什么、为什么做。让接棒人 30 秒读懂目标
2. **进度快照**：完成的、进行中的、未开始的——分别用 ✅🟡⚪ 标
3. **关键决策**：路上做的取舍，每条「选了 X 而不是 Y，因为 Z」
4. **失败的尝试**：试过但走不通的方案 + 失败原因（避免接棒人重蹈覆辙）
5. **下一步**：3 个以内具体动作，每条要可立即执行
6. **陷阱与约束**：用户修正过的偏好、项目特定约束、不能动的部分
7. **打开的问题**：需要决策、需要外部输入的

最关键的是第 2 条里的 **"进行中"**。这是 handoff 成败的命门——写得粗就是失败：

* ❌ "在改 auth 流程"
* ✅ "正在 `src/auth.ts:42` 修复 session 续期。已确认 bug 在 cookie maxAge 计算，思路是改用服务端时间。代码写到一半，未保存"

第二种写法接棒人能直接进去接着写。第一种就是「啊？你在改啥？」。

**`/handoff` 实际跑的时候发生了什么：**

1. 主对话先整理那 7 个板块的内容——这一步在你眼前发生，你可以打断、补充、纠正
2. 整理好之后，spawn 一个后台 subagent 同时写两个文件：覆盖写入 `docs/HANDOFF.md`（完整交接包）+ 追加到 `docs/memory/YYYY-MM-DD.md`（一行摘要 + 指针）
3. 主对话只输出一句「已生成交接包，新 session 输入 /recap 接棒」就结束，不等 subagent 完成

后台写文件这个设计是从 `/checkpoint` 学来的——主对话不被文件 IO 卡住，工作节奏不被打断。

## 让 /recap 自己接棒

写好 `docs/HANDOFF.md` 只是一半。新 session 怎么知道有这份文件？

最早设计是 handoff 末尾打印「在新 session 输入 `/resume` 接棒」。但用了两次就嫌烦——多一个命令要记。

后来直接改了 `/recap` 的逻辑：开头先看 `docs/HANDOFF.md` 存不存在。

* 存在 → 进入「接棒模式」：读完 HANDOFF，输出「上一棒做到 X，下一步是 Y，要继续吗？」，跳过标准 recap 流程
* 不存在 → 走原来的 recap 流程

这样无论是接棒还是普通新 session，都只记一个命令 `/recap`。

唯一要记的反向操作：**接棒任务完成后，手动删掉 `docs/HANDOFF.md`**。否则下次 recap 会误把已完成的旧任务又当成"上一棒"接进来。这个我目前靠自觉，后面打算让 `/kickoff` 完成阶段自动清。

## 坑 2：让 AI review 自己写的代码，没用

第二个坑，说出来大家都懂。

我让 Claude 写完一段代码，再让它自己 review。它说「逻辑没问题」。我点确认。跑起来——边界条件炸了。

回头看 review 报告，没漏看，是真的没看出来。

其实想想也合理。一个人写完作文让他自己批改，能挑出几个错？写代码尤其是这样——边界条件、竞态、安全漏洞，是「写的时候没想到，所以 review 时也想不到」的同源盲区。靠同一个模型自己审自己，本质就是双标。

之前我有个糙办法：写完之后我手动调几个 review agent，pr-review-toolkit 里的 code-reviewer、silent-failure-hunter、code-simplifier 各跑一遍，再让 Codex 看一遍，再让 Gemini headless 跑一遍。每次手打 5 行命令，懒得做就跳过，跳过的那一次往往就出事。

所以这事必须固化成命令。

## /cross-review —— 4 路并行，跨家挑刺

`/cross-review` 一次启动 4 路并行 review，分两个阵营：

**阵营 A：外部模型**

* **Codex（MCP）**：跑 GPT-5.5，重点看逻辑错误、边界条件、安全漏洞、竞态
* **Gemini（CLI headless）**：另起一家挑刺

**阵营 B：Claude 自家工具栈**

* **pr-review-toolkit** 三个 agent 并发：

+ `code-reviewer`：代码规范、项目约定
+ `silent-failure-hunter`：错误处理、静默失败
+ `code-simplifier`：简化建议

* **`/simplify`** Skill：复用、质量、效率

**`/cross-review` 实际跑的时候发生了什么：**

1. 解析参数。无参就跑 `git diff main..HEAD`（branch vs main 的全部 diff），也可以指定 commit 范围（`HEAD~3`）、文件路径、或者 PR 号
2. 拿到 diff 内容，确认非空（空就提示换范围）
3. **同一个 message 内同时发出 6 个 tool call**：Codex MCP、Gemini Bash、3 个 pr-review-toolkit Agent、1 个 /simplify Skill。这点很关键——同 message 多 tool call 才会真的并发，分多次发就变成串行，时间翻 6 倍
4. 等所有路返回，跑**共识算法**：至少两路指向同一文件:行（±5 行容差）→ 标记为"共识问题"
5. 如果总输出超过 8K token，spawn 一个 aggregator subagent（用 haiku 模型，便宜）整理成报告；否则主对话直接整理

整个过程大概 2-4 分钟，主要时间花在 Codex 和 Gemini 上（外部模型有网络延迟）。

报告分两块：

```
共识问题（多路同时挑到，高置信度建议修）  1. [Codex + Gemini] src/auth.ts:42     登录 cookie 过期时间算错了，跨时区会少一小时  
单路问题（只有一路挑到，自己判断）  2. [code-simplifier] src/utils.ts:88     这段格式化代码有重复，可以合并成一个公共函数
```

共识问题是高置信度的——不同家的模型同时挑出来，你自己看不挑都难受。单路问题是参考——可能是真问题也可能是某个 reviewer 自己的洁癖，你扫一眼自己定。

一个原则：**`/cross-review` 只产出报告，不自动修代码**。修复要走 `/kickoff` 派 Developer 改。原因是 review 和修复是两件事，自动修很容易越改越乱（比如 simplifier 觉得这段可以提取，结果一提取破坏了原来的语义）。

## 坑 3：多 agent 协作没有统一编排

前两个命令都在管「记忆」和「review」，但开发流程本身没有抓手。

需求和 plan 这些我自己一直都在写，问题不在这里。问题在于：以前我得手动把每一步串起来——

* 跟 Claude 说「看一下 `docs/requirements/042` 的需求」
* 让它出方案，我看完确认它才能写
* 想加一道方案 review 还得另起一段对话让 Codex 反审一下
* 写完我手动跑一遍功能验收
* 验收过了让 pr-review-toolkit 再扫一遍代码
* 全过了我手动更新 `docs/features/042`

每一步都要我打字告诉 AI「下一步做什么」。一开始还行，做到第三步我就在想「这玩意儿不如我自己写」。

而且随着 AI 能干的事越来越多——agent-browser 能跑浏览器验收、Maestro 能在真机跑 iOS 测试、Doc Engineer 能同步文档、Codex 能反向 review 方案——这些步骤之间手动串联只会越来越累，越来越容易漏掉某一步。

需要一个东西把整套流程编排起来，让 AI 自己接力跑完。

## /kickoff —— 把流程编排成门禁

`/kickoff` 就是那个编排器。打 `/kickoff <需求描述 | REQ-XXX | 文件引用>` 之后会经历五个阶段，每个阶段做完自动进入下一个阶段，AI 接力跑：

**1. Plan 阶段（强制 Plan Mode，不能写文件）**

按顺序走：

* 理解需求（读 `docs/requirements/` 或用户描述）
* 查 Linear 看有没有相关 ticket
* 审视：这真的是要做的吗？有没有更简单的方案？10 分版本长什么样？
* **提取需求 R#**：每个独立行为变更一行，包括边界情况和隐含需求
* **R# 提取完丢给 Codex 反向审一遍**：Claude 自己拆需求自己审，盲区是同源的——让 Codex 这个外人来看一眼，往往能挑出 Claude 系全注意不到的遗漏。这步是跨家的，跟 cross-review 一个思路
* **拆解任务 T#**：按需求切片，每个 T 配 file ownership 和执行者（Opus / Sonnet / Codex）
* 生成验收标准 AC# + 测试骨架
* Plan 输出 → **等用户在 Plan UI 里确认/修改/拒绝**

未确认禁止写任何实现代码——这一条是硬规则。后面所有 agent 都基于这份 plan 接力，plan 不准会一路传下去。

**2. Developer 阶段**

按 T# 派 teammate 干活，编码力量分配原则：

* **Codex（GPT-5.5）拿最大独立块**：独立模块给它交付质量最好——它没有项目全局上下文，所以越独立、越自包含的任务越适合它
* **Opus 拿难点**：模糊需求、深度推理、紧密协调的部分
* **Sonnet 拿胶水**：轻量协调、配置类、不需要深度推理的代码

每个 teammate 在自己的 file ownership 范围内工作，避免互相覆盖。

**3. QA 阶段**

像用户一样操作验收，**不是跑测试套件**：

* Web 项目：用 Glance MCP 做视觉验收（看页面有没有按预期渲染）
* iOS 项目：用 Maestro + assertWithAI（在真机/模拟器上跑用户路径）
* 验收账号从环境变量读

**4. Review 阶段**

自动调 `/cross-review`，把上面那一套四路并行 review 跑一遍。共识问题修了再进下一步。

**5. Doc Engineer 阶段**

sub-agent（用 haiku）执行文档审计：代码 vs `docs/features/` 一致性检查、术语统一、更新对应需求文档。

整个流程的精髓是**多 agent 围绕统一锚点（R# / T# / AC#）协作**。Lead 拆出来的 R# 不光是给我看的——后面 Developer 按 T# 写代码、QA 按 AC# 跑验收、Doc Engineer 按 R# 更新文档，所有 agent 都靠这套编号对齐。任何一步出问题都能反查到具体的 R# 上，而不是"哪里改坏了不知道"。

为了让 plan 阶段不啰嗦，每个阶段的详细指令分文件存在 `~/.claude/kickoff/` 下，进入哪个阶段才 Read 哪个文件——这样上下文不会一上来就被全流程的指令塞满。

## 多模型联动的逻辑

写完上面三个命令，回头看一眼整个工作流，发现已经不止一个模型在干活了。Claude 之外还有 Codex（GPT-5.5）和 Gemini，各做各的。这个分工是慢慢长出来的，不是一开始设计好的。

**Claude：Lead 和最后的整合**

理解模糊需求、协调子任务、关键决策、最后把所有结果拼起来。Anthropic 这家模型最擅长处理"边界不清"的场景，让它做拍板的角色。也是最贵的，所以让它把 token 花在这种地方最划算。

**Codex（GPT-5.5）：主力实现 + 方案外审**

独立的、边界清楚的大块任务交给 Codex。它没有项目全局上下文，所以越独立、越自包含的任务越适合它——一次吃下一个完整模块的代码 + 测试。Lead 派任务给它的时候 prompt 必须自包含：关键代码片段、风格参考、输出格式都要贴进去。

除了写代码，Codex 在 kickoff 的 plan 阶段还有一个角色——**反向 review Claude 拆出来的 R# 列表**。Claude 自己拆需求自己审，盲区是同源的；让 Codex 这个外人看一眼，能挑出 Claude 系全注意不到的遗漏。

我现在还会顺便把 icon、占位图、示意图、OG 图这种小图片资源也打包给 Codex（gpt-image-2），让它一次交付代码 + 图片，不用单独切个"做图"任务。

**Gemini：i18n 翻译为主，review 为辅**

我没用 Gemini 写代码。它在我工作流里主要做两件事：

一是 **i18n 翻译**。StudyThai 要做泰语、英语、中文等多语言文案，翻译这种「准确性 + 自然度」要求高、但创造性低的任务，给 Gemini 跑批最划算——Google 的模型在多语言上一直有优势（特别是中文和东南亚语言），价格也便宜，一次能吃下整个 messages JSON 文件。

二是 **cross-review 里顺手调一路**。它跨家，正好挑 Claude 系挑不出来的盲区。但不是 Gemini 的主业——主业是翻译，挑刺只是兼职。

**Claude 自家的多 agent（pr-review-toolkit / /simplify）**

这部分是 Claude 内部的不同视角——code-reviewer 看规范、silent-failure-hunter 看错误处理、simplifier 看简化机会。它们是 Claude 系，但被预设了不同的"看代码角度"，多少能拓宽视野。覆盖度比真正的"跨家"差，但便宜（同一个 subscription）。

合在一起的分工大概是：

**Lead 层**：Claude（理解需求 + 协调 + 整合）

**Developer 层**（Lead 派活，并行）：

* Codex —— 主力实现 + 图片资源
* Claude Devs（Sonnet / Opus）—— 胶水 / 小任务
* Gemini —— i18n 翻译 + 兼职外审

**Review 层**（`/cross-review` 4 路并行 → 共识算法 → 报告）：

* 阵营 A：Codex + Gemini（外部模型挑刺）
* 阵营 B：pr-review-toolkit 三个 agent（code-reviewer / silent-failure-hunter / code-simplifier） + `/simplify` Skill

一个原则：**写代码归一家，review 归另一家**。同一家自己 review 自己永远是双标。

## 命令清单（从 4 升到 7）

上一篇结尾列了 4 个命令。这次更新一下：

**会话生命周期**

* `/recap` —— 新 session 恢复上下文（自动检测 HANDOFF，存在则进入接棒模式）
* `/checkpoint` —— 当前 session 内随时存档
* `/handoff` —— 跨 session 接棒包（自包含，给下一个 session 看）
* `/postmortem` —— 踩坑变记忆

**任务流程**

* `/kickoff` —— 启动开发任务，强制 plan + 派 teammate + QA + review + doc 全流程
* `/cross-review` —— 四路并行 code review，跨家挑刺

**项目初始化**

* `/init-project` —— 新项目搭骨架，自带规范

加起来 7 个命令。把一个开发任务的完整生命周期管完了：开工（kickoff）→ 推进（checkpoint）→ 中途交接（handoff）→ 接棒（recap）→ 审查（cross-review）→ 复盘（postmortem）。新项目用 init-project 起步。

## 总结

上一篇的结论是「让 AI 不要每次都从零开始」（解决**记忆**问题）。

这一篇加两句：**让 AI 不要既当运动员又当裁判**（解决**自审失真**），**让多个 AI 接力跑完整条流水线**（解决**多 agent 协作没有编排**）。

三件事都是工程团队的成熟做法——

软件工程团队管 session 接棒，是用 standup 和 handoff doc。

管多人 review，是 PR 强制 reviewer 不能是作者本人。

管开发流程，是 PRD → Plan → Implement → QA → Review → Ship 这一整条流水线，每个工种接好上一棒。

现在我把这套东西搬到 AI 身上：handoff 文件让 session 之间能接棒，cross-review 让"作者"和"审稿人"必须不是同一个模型，kickoff 把整条流水线编排起来，每个 AI agent 接好自己那一棒。

再升一层，本质是把人类协作里的角色分层（Lead / Developer / Reviewer / Doc Engineer）整套搬到 AI 身上。一人公司从 1 个人 → 1 个人 + N 个 AI 角色，每个角色给最擅长的那家模型。Claude 当 Lead 和 Reviewer，Codex 当主力 Developer，Gemini 做翻译和外审，Claude 自家 agent 当胶水和文档。

每加一个角色，就让我从一些重复劳动里脱身。我现在写代码越来越少，更多是在做需求拆解、决策、和审报告——这其实已经是个小团队 leader 在干的事，只不过团队成员都是 AI。

下一步打算把 Linear 的 ticket 流转也接进来，让 AI 自己创建子任务、自己分配、自己关闭。还在调试，等跑通了再说。

文件还是不复杂——三个新命令各一个 markdown，加上 `/recap` 加几行检测逻辑。一个晚上能搭完。但搭完之后 AI 协作的体感又往前走了一截：从「记得住事的同事」变成了「会按流程协作的团队」。