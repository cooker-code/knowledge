---
title: 一文讲透 Claude Code 的 /goal 功能
author: 码读时光
date: 耐心耐心
url: https://mp.weixin.qq.com/s?__biz=Mzk2NDQwNDAxMQ==&mid=2247484391&idx=1&sn=98e94d23b1cd3e6ce2dae53036548d71&chksm=c5ba2398f8a062bdbf00801ee3c4a560b43f2fe7e0b80de7228612aadb5f2f1621c9a1288c79&mpshare=1&scene=24&srcid=0513Rdbinz1NOKu8KReyBdfr&sharer_shareinfo=4d660a3d96d87db30e8fa882e3768814&sharer_shareinfo_first=4d660a3d96d87db30e8fa882e3768814#rd
---

# 一文讲透 Claude Code 的 /goal 功能

Claude Code v2.1.139 新加了一个 `/goal` 命令。一句话说清作用：「设个完成条件，AI 跑到达成为止。」

今天我们先来扒一段此功能的来龙去脉，再讲讲实现机制，明白后就知道："什么任务该挂、什么任务别挂"。

## /goal 的三代演进：从社区到原生

`/goal` 不是凭空发明的，它是社区里一个 pattern 走完三代演进后被官方收编的成果。

**第一代：Ralph Wiggum（2025 社区）**。Geoffrey Huntley 在 2025 年中提出 「Ralph the Wiggum」 模式，名字取自《辛普森一家》里那个一句 「I'm helping!」 不肯停的胖男孩。本质是 bash while 循环：让 Claude 干活，干完之后用 hook 拦住它的退出动作，把原 prompt 一字不改再灌一遍。Claude 看到 git diff 自己琢磨下一步，反复直到输出指定的 magic string 才算完。后来这套模式被打包成 marketplace 插件 `ralph-wiggum@claude-code-plugins`，装好之后用 `/ralph-wiggum:ralph-loop` 一行命令调用，加上 `--max-iterations` 硬上限和 `--completion-promise` 完成口令两个参数。

**第二代：prompt-based Stop hooks（v2.0.41 底层抽象）**。Anthropic 看到 Ralph 这种 pattern 在社区流行后，把底层机制抽象为正式 API：每当 Claude 想停止当前任务时，可以触发一段 prompt 拦住它。Ralph Wiggum 插件就是基于这个机制打包的，但底层 API 暴露给普通用户太裸了，得懂 hooks 系统、写配置、起 magic string。

**第三代：`/goal`（v2.1.139 原生化）**。v2.1.139 这版 Anthropic 把整个 pattern 包装成一行命令：`/goal <完成条件>`。背后还是 prompt-based Stop hook，但引入了独立的 Haiku 模型当评估器，替代字面字符串匹配。

社区先发明、底层抽象、官方收编，这是 Claude Code 又一个招安案例。

## /goal 内部是怎么工作的

挂上 `/goal` 后，背后有三件事在转。

**第一件，给 Claude 套个 Stop hook**。Claude 本来干完一轮任务想停，但 Stop hook 把它的退出动作拦住，递到评估器手里。这一步用户看不见，但所有"继续"动作都靠这个机制串起来。

**第二件，评估器跑判断**。Anthropic 让一个 Haiku 模型读完整段对话（**不是文件，是对话**），问它：「condition 满足了没？」Haiku 回 yes 或 no，附一段 reason。yes 就放行，浮层显示 Goal achieved；no 就把 reason 当下一轮的 directive 灌回 Claude，让它接着干。Haiku 比 Opus 便宜约十几倍，所以评估器开销几乎可以忽略。

**第三件，condition 必须写得让 Haiku 能判**。这是最容易踩的坑。因为评估器只看对话不看文件，所以 condition 必须告诉 Claude 「用什么方式留下证据」。官方文档原话是 「**condition 本身就是 directive**」 —— 它不只是终点描述，还是给 Claude 看的指令。官方推荐 condition 包含三件事：

* • **end state**：结束状态长什么样（「`npm test` 退出码 0」）
* • **stated check**：怎么证明已经到了那个状态（「贴出完整测试输出」）
* • **constraints**：过程里不能动什么（「不许改测试文件本身」）

比如让 Claude 跑测试，condition 里只写「让测试全过」并不保险。得进一步要求 Claude「跑 `node --test` 并把完整的 `# pass / # fail` 计数贴在对话里」。

Haiku 看不到测试退出码、也看不到文件。对话里没有这串数字，它就无从推断条件达成。

## 智能裁判 vs 字面裁判：跟 Ralph 真正的差异

讲到这儿，`/goal` 跟自家 Ralph 真正的差异就能看清了。

底层机制两者**完全一样**：都是 prompt-based Stop hook 包装。区别在 「谁来判完成」 和 「下一轮灌什么」 这两件事。

| 维度 | Ralph Loop | `/goal` |
| --- | --- | --- |
| 完成判定 | **字面裁判** ：找指定 magic string | **智能裁判** ：Haiku 自然语言判 |
| 下一轮指令 | 原 prompt 一字不改重灌 | 评估器 reason（「还差什么」） |
| 失败时的信息 | 无（Claude 看 git diff 自己琢磨） | 语义反馈（「fib(-2) 应返回 -1n」） |
| 完成口令 | 必须程序员写 magic string | 自然语言中文描述就行 |

类比一下：

* • **Ralph 是 「考试只看最后那道选择题填了啥」**。你做的过程对不对它判不了，只能看你最后那张写有「我做完了」的纸条。这就是 「字面裁判」。
* • **`/goal` 是 「考试请了个真人阅卷老师」**。它能读懂你过程里的每个步骤，告诉你「还差第 3 题没答」。这就是 「智能裁判」。

Anthropic 把社区 Ralph 原生化为 `/goal` 时，最大的升级不是机制本身，是**把字面字符串裁判换成自然语言裁判**。

这一升级让「完成条件」从程序员写的 magic string 变成普通用户能写的中文句子。这才是 `/goal` 的"民主化"。

## 跨厂商对照：Codex 的 /goal 选了另一条路

OpenAI 在 Codex CLI 在前几天也推了 `/goal`，命令名一字不差，但思路完全不同。

* • **Claude 走「外部裁判」路线**：用独立 Haiku 模型判完成。优点是工作模型自欺欺人骗不过去；缺点是裁判看不到文件，得在 condition 里要求 Claude 贴铁证。
* • **Codex 走「自我审判 + 制度约束」路线**：工作模型自己用 `update_goal` 工具标 complete，但加 SQL 持久化、no-tool 静默（一轮没工具调用就停）、token budget 硬上限三层防护。

两家选了两种「防 AI 失控」的工程哲学：Anthropic 怕模型自欺欺人、OpenAI 怕模型空转或超支。

没绝对优劣，但 Claude 这条路径更适合不信任 LLM 自评的用户。

顺便提一句限制：Codex 的 `/goal`  好像只向pro订阅用户开放，plus都不行，更别说API key了。Claude 这边正式发布、无门槛。

## 适合挂 /goal 的 5 类任务

机制讲到这儿，可以聊聊什么场景挂上去最值了。

**第一类，长任务的全测试套件修复**。比如十几个测试挂得到处都是、根因不在一个文件、改一遍跑一遍才知道还差什么。这是评估器最有可能"开口"指路的场景。每轮 reason 会点名「还差 testFoo / testBar」，下一轮 Claude 拿着这个反馈精准补漏。

**第二类，性能优化达标**。完成条件里写「`bench.test.mjs` 显示 `elapsed < 100ms`」，Claude 改一版跑一版，elapsed 没到阈值评估器就让它接着改。这种实测数据驱动迭代的任务，挂 `/goal` 比你守着屏幕按 Enter 强太多。

**第三类，大型重构或迁移**。跨几十个文件改 API 签名、改完跑一遍测试套件、有遗漏继续改。工作量大、有客观验证手段、可能跨数轮，正是 Anthropic 把 `/goal` 原生化的本意。

**第四类，长跑无人值守任务**。挂上去走开睡觉、第二天看结果的场景。比如「把这 50 个 PR 的 review comment 全处理完」「把这个 codebase 的 lint 错误降到 0」。没有 `/goal` 你得每完成一个手动点继续，挂上它能省一晚上的人工。

**第五类，有外部诊断成分的任务**。比如「让 CI 在 main 分支跑绿」「修这个内存泄漏直到 profiler 显示稳定」。关键是问题在哪不预先知道，需要 Claude 跑工具、看输出、再改。

## 不适合挂 /goal 的 4 类任务

反过来看，这四类挂上去就纯属浪费。

**第一类，单文件小修改**。改个变量名、加段注释、补一个 if 分支，Opus 4.7 半秒就出活。挂 `/goal` 等于多调用一次评估器 Haiku，纯加开销。

**第二类，3 文件以内的合约升级**。我实测的就是这种：三个互相 import 的文件、按合约改参数风格、有完整测试套件。`/goal` 跑 medium effort 一回合 1 分钟拿下、low effort 反而更快 37 秒。评估器只在终态盖个章，全程没指过路。

**第三类，创造性任务**。写一篇文章、设计一个 UI、起一个文案标题，「完成」没有客观定义，评估器判不了。这种活该用 `/loop` 或者干脆手动跑。

**第四类，强依赖人工判断的任务**。比如「让代码风格更好看」「让函数名更优雅」。美感、口味这种事 Haiku 评估器没能力替你看。

## 实操技巧 + 5 条用前要留意的边界

5 类适合 + 4 类别挂的判断框架有了，剩下就是把 condition 写到位。

写 condition 时把三要素全写上：终态 + 验证方式 + 约束。比如「`npm test` 退出码 0、贴出完整 pass/fail 计数、不许改测试文件本身」。

最常踩的坑：condition 里只写终态、忘写「证据格式」。

评估器看不到文件，Claude 跑完测试如果没把 `# pass / # fail` 完整贴在对话里，评估器没铁证，永远判不了 yes，挂死在那。

最后列几条用前要留意的边界：

1. 1. 评估器是独立 Haiku 模型，**本身不调用任何工具**，所有验证都要能从对话推断（这是跟 Codex 自评式 `/goal` 的核心差异）
2. 2. 每个会话同一时刻只能挂 1 个 `/goal`，不能并行跟踪多个目标
3. 3. `/goal` 走的是 hooks 系统，未 trust 的工作目录跑不起来；企业策略 `disableAllHooks` 下同样不可用（同源原因，prompt-based Stop hook 被禁就连带禁了 `/goal`）
4. 4. condition 字段长度大致以 4000 字符为上限，复杂目标得精炼写

## 写在最后

`/goal` 不是给「让 AI 反复试错」用的，是给「让 AI 跑完不用你盯着」用的。

觉得这篇文章值得让朋友也看看，麻烦**点个「在看」**。