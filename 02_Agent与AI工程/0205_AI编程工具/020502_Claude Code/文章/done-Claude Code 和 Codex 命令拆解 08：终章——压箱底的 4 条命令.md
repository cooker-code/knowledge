> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 和 Codex 命令拆解 08：终章——压箱底的 4 条命令
author: 阿锦AI
date: 阿锦AI阿锦AI
url: https://mp.weixin.qq.com/s?__biz=MzU4ODkxNzY3Ng==&mid=2247484547&idx=1&sn=f627b2e1bd8b33b165e5e666cfcf421e&chksm=fce2d3161e2c2b9b1cb244e0abed4d3ca6c87710b89ecc1445391231e0761bb441a3dc6ce992&mpshare=1&scene=24&srcid=0517jBAnEmikecmvXBEz5dUB&sharer_shareinfo=5bd69c31b0614954cd07ce065654efb7&sharer_shareinfo_first=5bd69c31b0614954cd07ce065654efb7#rd
---

这是「Claude Code 和 Codex 命令拆解」系列第 8 篇，也是终章。前 7 篇从入门走到进阶——AI 怎么读项目、怎么管上下文、怎么找回现场、怎么记规矩、怎么走流程、怎么动手碰你的数据、怎么用 skill 把重复活变成肌肉记忆：

* 01｜第一步先跑 /init，让 Agent 读懂项目
* 02｜/clear、/compact、新开会话，到底该用哪个
* 03｜别急着开新会话，先用 resume 找回现场
* 04｜让 AI 记住你和你工作的规矩 —— CLAUDE.md / /memory
* 05｜装个插件让 AI 协作再升一档 —— helloagents
* 06｜让 AI 真的连上数据库 —— MCP
* 07｜你早就在用 skill 了，只是不知道

到这一篇，常用命令你都见过了——`/init`、`/clear`、`/compact`、`/resume`、`CLAUDE.md`、`/agents`、`/mcp`、`/skills`。

今天不再罗列。挑 4 条**最值钱但最容易被忽略**的命令——前 3 条是斜杠命令，第 4 条不在 `/` 菜单里，但你迟早会用上。

---

## 一、`/plan`——下手前先看路线图（两边都有）

一个普通的场景：

> 你跟 AI 说：「帮我把这 5 个文件重构一下，把鉴权抽成独立模块。」
>
> AI 二话不说开干，10 分钟后你发现它的方向错了——抽得太碎、新模块绕开了你现有的中间件。
>
> 现在你有两个选择：让它返工，或者自己手动收拾。

`/plan` 就是给这种情况设的：

```
/plan 帮我把这 5 个文件重构一下，把鉴权抽成独立模块
```

AI 进入「**只想不动**」模式，把改动方案给你看——哪几个文件改、新结构长啥样、风险点在哪。**你确认后它才动手**。

类比一下：你出差前是先打开导航看一眼路线，还是直接踩油门？

两边都有 `/plan`，**但 90% 的人想起用它的时候已经返工过一次了**。养成这个肌肉记忆。

### `/plan` 也有局限：需求本身模糊时不够用

`/plan` 适合**你大致知道要做什么**的场景——它能帮你把"怎么做"理清楚。

但如果你给的需求本身就模糊（"帮我优化下这个项目"、"用户体验做好点"），或者你自己也没想清楚要哪种方案——`/plan` 只会基于猜测给你一份计划，**它不会反过来追问你**。

这时候 05 篇讲过的 **helloagents** 更对路：装上以后，它会先**给你的需求打分**（需求范围、成果规格、实施条件、验收标准 4 个维度），不到 8 分就**主动追问**你，直到把需求搞清楚再动手。

简单分工记一下：

* **你想清楚了**（明确改哪几个文件、改成什么样）→ `/plan` 看一眼方案就开干
* **你也没想清楚**（只有方向、没有具体方案）→ 让 helloagents 先帮你想清楚，再用 `/plan`

详见 05 篇 helloagents 那篇。

---

## 二、`/rewind`——AI 改乱了，退回 30 分钟前（Claude 独家）

另一个场景：

> AI 改着改着越改越偏。你看着它在第 4 个文件里写出一段莫名其妙的逻辑，意识到——**前面三个文件可能也有问题了**。
>
> 旧办法：一个个 `git diff`、`git checkout`、手动挑回来。痛苦。

Claude Code 给你一颗"后悔药"：`/rewind`（别名 `/checkpoint`、`/undo`）。

### 怎么触发

两种方式，**第二种比第一种快**：

1. 输入 `/rewind`
2. **连按两下 `Esc`**——直接弹出回退菜单。这个快捷键官方文档藏在小字里，但日常用最顺手

### 它给你 4 个动作选项

选完一个历史节点后，会问你想退多少：

| 选项 | 干啥 | 什么时候选 |
| --- | --- | --- |
| **Restore code and conversation** | 代码 + 对话**都**退回去 | 最常用，完整后悔药 |
| **Restore conversation** | 只退对话、**代码保留** | 想换种说法重问，但已经改对的代码别丢 |
| **Restore code** | 只退代码、**对话保留** | 想保留 AI 的思路记录，但回收代码改动 |
| **Summarize from here** | 不回退，把这之后压缩成摘要 | 上下文塞满了想腾空间，代码不动 |

90% 时候你会选第一个。但中间两个特别值钱——比如 AI 改了 3 个文件，前 2 个对、最后 1 个翻车，你想**只退最后那一步对话**，前 2 个文件的成果不能丢。

### 和 03 篇讲过的 `/resume` 怎么搭

03 篇讲过 `/resume`——找回上一次会话。这两条命令的关系是：

* `/resume` = **跨会话**穿越（回到昨天那个项目的聊天里）
* `/rewind` = **会话内**穿越（在当前聊天里退到 30 分钟前）

而且 checkpoint **跨会话保留**——你今天 `/resume` 进昨天的会话，里面的 `/rewind` 节点照样能用。两条命令组合起来，等于「时间机器 + 时光机」都有了。

### 注意

* **bash 命令改的文件不在 checkpoint 范围**——`rm`、`mv`、`cp` 改的文件 `/rewind` 救不了你（这种还是得靠 git）
* 它不是 git 替代品，是「会话级 Ctrl+Z」

Codex 现在还没有这个能力——只能靠 git 自救。**这条单拎出来就足以构成选 Claude Code 的理由。**

---

## 三、`/batch`——让 AI 同时开 5 台车（Claude 独家）

再来一个：

> 你要把项目从 Solid 迁到 React，20 个组件、每个差不多。
>
> 普通做法：让 AI 一个一个改，喝着咖啡等半天。

`/batch` 是 Claude Code 的「**集团军作战**」：

```
/batch migrate src/ from Solid to React
```

它会先研究代码，然后**把改动自动拆成 5-30 个独立任务**——每个任务**开一个独立的 git worktree**，并行往下跑，跑完每个自动开一个 PR。

注意是 `worktree`，不是「子代理同时聊天」——是真·5 个独立工作区在硬盘上并行干活。你晚上挂着，第二天起来收 20 个 PR 验收。

跟 05 篇讲的子代理（`/agents`）不是一回事：

* 子代理 → 同一个会话里分工协作
* `/batch` → 完全独立的几个克隆体，互不知道彼此存在

Codex 现在没有等价的命令。

---

## 四、那条不在菜单里的命令——「放飞开关」

前面三条都是斜杠命令。**第四条不在 `/` 菜单里**，但实战里你早晚会查它：

bash

```
# Claude Code
claude --dangerously-skip-permissions

# Codex
codex --dangerously-bypass-approvals-and-sandbox
```

启动时加这个 flag，**整个会话不再弹任何权限/审批对话**——AI 想读啥读啥、想写啥写啥、想跑啥跑啥，全程不打扰你。

什么时候用？

* **挂机跑长任务**——晚上让它自己改自己测自己提 PR，没人盯着确认按钮
* **Docker / VM / 独立项目目录**——反正这个环境炸了你也无所谓
* **流水线脚本里调 AI**——非交互场景没人能点确认

什么时候**绝对别用**？

* **主目录** `~/` **下直接开**——AI 误删了你的照片、文档、密码本，找谁哭
* **生产服务器**——一句 `rm -rf` 你就解释一年
* **关键代码仓库的 main 分支**——先 checkout 一个分支再说

实战 SOP：

bash

```
# 1. 单独建一个 worktree 或者 clone 一份
# 2. cd 进去
# 3. claude --dangerously-skip-permissions 或者
#    codex --dangerously-bypass-approvals-and-sandbox
# 4. 让 AI 放飞，自己去喝咖啡
```

用人话总结：**你愿意 100% 把方向盘交给 AI 时，才开。但记得给它装个保险——开在一个炸了也不疼的环境里。**

---

## 附：剩下命令的对照表

下面是「**前 7 篇没单独讲过、但你可能用得到**」的命令，按类型分。先看一眼速查海报，下面是完整版：

（命令会随版本变，以你装的版本为准。）

---

参考资料：

* Claude Code commands reference: https://code.claude.com/docs/en/commands
* Claude Code checkpointing（/rewind）：https://code.claude.com/docs/en/checkpointing
* Claude Code CLI reference: https://docs.anthropic.com/en/docs/claude-code/cli-reference
* OpenAI Codex CLI slash commands: https://developers.openai.com/codex/cli/slash-commands

---

我建了一个「阿锦AI交流群」，对 AI 和 AI Coding 感兴趣的朋友欢迎进群交流。扫码添加我的微信，备注 **AI** 即可进群。群里不定期赠送大模型 API 体验额度。