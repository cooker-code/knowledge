---
title: gnhf:让 AI Agent 替你打工一整夜的极简编排器
author: 大鱼读书
date: 阿畅阿畅
url: https://mp.weixin.qq.com/s?__biz=MjM5MDQ1NzMwMg==&mid=2257486263&idx=1&sn=d63933c037cd0b97a385604078c09be2&chksm=a4a5dd339a4828f608194828ace7b3b93c43c72f62fe5547d017e42d398024bf9003a3f55fb0&mpshare=1&scene=24&srcid=0530b4isCmW5XZ45fRmifHYu&sharer_shareinfo=1fa21594851727fb1c382fd938740d77&sharer_shareinfo_first=1fa21594851727fb1c382fd938740d77#rd
---

## 它是什么

gnhf 全称 **"good night, have fun"**——作者 kunchenguid 睡前对自己 agent 说的话。

它做的事情用一句话就能说清:**你睡觉前给它一个目标,它让你的 agent(Claude Code / Codex / OpenCode 等)循环迭代,每次只做一个被 commit、被记录的小改动,直到你 Ctrl+C 或跑到上限**。

它不是又一个 agent,而是给现有 agent 套了一层"长跑外壳"。早上醒来,你会看到一条干净的 git 分支和一份完整日志。

```
$ gnhf "reduce complexity of the codebase without changing functionality"  
# have a good sleep
```

---

## 用在什么场景

最适合那种**目标明确、工作量大、可以拆成无数小步**的任务:

* • 降低代码库复杂度
* • 批量重构
* • 补测试
* • 整理 TODO
* • 批量更新依赖、迁移 API
* • 文档同步、跨文件命名规范统一

**不适合**:

* • 需要人类每步判断的任务(架构设计、产品决策)
* • 强依赖外部副作用且不可回滚的任务(发邮件、转账、扣费)
* • 不能用 git 管理的工作(纯文档协同、Notion、Figma)

一句话定位:**"这事我有空就能做完,但我不想做"——交给 gnhf**。

---

## 核心特点与原理

gnhf 最特别的不是它做了什么,而是它**没做什么**——几乎没引入新概念,完全靠 Git 做状态管理。这种克制是它区别于一切重型 agent 框架的根本。

### 1. 用 Git 做状态机

**特点**:每次迭代成功就 `git commit`,失败就 `git reset --hard`,整体跑完所有 commit 都在一条 `gnhf/` 分支上。

**原理**:把"agent 在跑长任务"这件原本需要数据库、状态文件、事务管理的复杂事情,完全用 **git 的原子提交特性**来表达。git 本来就是为"一系列原子修改"而设计的——成功的迭代就是一次正常 commit,失败的迭代等于这次 commit 没发生过。**没有锁,没有持久化层,没有任何额外状态**。

### 2. 每次迭代 = 一次独立 commit

**特点**:早上醒来,你看到的不是一个巨大的 diff,而是一串语义清晰、独立可逆的 commit。

**原理**:**commit 粒度 = review 粒度 = 回滚粒度**。你可以 cherry-pick 喜欢的、revert 不喜欢的、squash 合并相关的——所有 git 工具链(GitHub PR、SourceTree、lazygit)直接可用。这是"整段时间一次性 commit"给不了的控制力。

### 3. notes.md:跨迭代的"工作记忆"

**特点**:每次迭代成功后,agent 把"这一轮做了什么、下一轮该做什么"追加到 `notes.md`,下一次迭代会把这份 notes 注入到 prompt 里。

**原理**:模型本身的 context 窗口在长任务里靠不住——既会溢出,也会被无关信息污染。gnhf 不依赖模型的内部记忆,而是**让 agent 自己管理一份外部的、可见的、可手动编辑的"工作日记"**。这是从"短时记忆"升级为"工作记忆"的关键,也是 gnhf 区别于"裸 while 循环"的核心增量。

### 4. Agent 无关

**特点**:`--agent claude / codex / rovodev / opencode` 一行切换底层 agent。

**原理**:gnhf 只调用 agent 的 CLI 非交互模式,**把 agent 视为黑盒**。你的 prompt、notes、commit 历史完全独立于 agent 实现——今天 Claude 跑,明天 quota 用完了换 Codex,后天接 OpenCode 自托管模型,**所有进度无缝继承**。

### 5. 失败重试 + 自动熔断

**特点**:失败后 `git reset --hard` → 指数退避 → 重试;连续失败 3 次自动 abort。

**原理**:网络抖动、模型偶发性失败是常态,不能因为一次失败就放弃整夜任务。但也不能无限重试——agent 真卡进死循环时,3 次基本能确认。**退避 + 熔断是分布式系统里非常成熟的模式**,gnhf 把它原样搬到了 agent 编排上。

### 6. 双重 runtime cap

**特点**:`--max-iterations` 卡迭代次数(下一轮开始前检查);`--max-tokens` 卡 token 总量(**迭代中途检查,超限立刻停,未提交工作回滚**)。

**原理**:夜间无人值守最大的恐惧是"早上起来发现信用卡爆了"。两道闸门一个防跑太久、一个防烧太多 token,而且 token cap 不是事后对账,是中途即停 + 自动回滚——哪怕你把 max-tokens 设小了,最坏情况也只是没出活儿,**而不是出了一半烂尾**。

### 7. 断点续跑 + 元数据隔离

**特点**:运行元数据放在 `.gnhf/runs/` 下并自动 ignore,在已有 `gnhf/` 分支上再跑一次 `gnhf` 自动续上。

**原理**:**工作产物(commit)和过程元数据(prompt、notes、resume 信息)物理分离**。前者是你要保留的最终成果,可以直接提 PR;后者是过程性的本地缓存,丢了也不影响成果(除非你想续跑)。这种分离是工程上非常成熟的做法,但很多 agent 框架反而做不到——它们的状态、缓存、产物经常混在一起,导致工作分支根本没法直接拿出来用。

---

## 工作流程一图看懂

```
                    ┌─────────────┐  
                    │  gnhf start │  
                    └──────┬──────┘  
                           ▼  
                ┌──────────────────────┐  
                │  validate clean git  │  
                │  create gnhf/ branch │  
                └──────────┬───────────┘  
                           ▼  
              ┌────────────────────────────┐  
              │  build iteration prompt    │◄──────────────┐  
              │  (inject notes.md context) │               │  
              └────────────┬───────────────┘               │  
                           ▼                               │  
              ┌────────────────────────────┐               │  
              │  invoke your agent         │               │  
              └────────────┬───────────────┘               │  
                           ▼                               │  
                    ┌─────────────┐                        │  
                    │  success?   │                        │  
                    └──┬──────┬───┘                        │  
                  yes  │      │  no                        │  
                       ▼      ▼                            │  
              ┌──────────┐  ┌───────────┐                  │  
              │  commit  │  │ git reset │                  │  
              │  append  │  │  --hard   │                  │  
              │ notes.md │  │  backoff  │                  │  
              └────┬─────┘  └─────┬─────┘                  │  
                   │              │                        │  
                   ▼              ▼                        │  
              ┌────────────┐    yes   ┌──────────┐         │  
              │ 3 consec.  ├─────────►│  abort   │         │  
              │ failures?  │          └──────────┘         │  
              └─────┬──────┘                               │  
                 no │                                      │  
                    └──────────────────────────────────────┘
```

整个流程**没有一个非 git 的状态点**——这是 gnhf 在设计上最克制也最巧妙的地方。

---

## 总结:克制是一种力量

agent 编排领域过去一年涌现了大量重型框架——LangGraph、AutoGen、AutoAgent、CrewAI——它们都很强,但学习曲线陡峭,概念繁多。

gnhf 走的是相反的路。它不告诉你"agent 应该怎么协作",它只回答一个具体问题:**怎么让一个 agent 安全地跑一整夜**。

它的所有设计都围绕这一个问题:

| 设计 | 解决的问题 |
| --- | --- |
| 用 Git 做状态机 | 过程透明、可回滚 |
| commit 粒度 = 迭代粒度 | 结果可 review、可挑选 |
| notes.md | 长任务的记忆持续性 |
| 失败熔断 + 双重 cap | 夜间无人值守的安全感 |
| Agent 无关 | 工具栈快速迭代下的可移植性 |
| 元数据隔离 | 工作分支保持纯净,可直接 PR |

最终用最少的概念,解决了 agent 长跑里最痛的几个问题。

> **有时候你只需要一个 agent、一个 git 分支、和一句晚安。**

🌙 Good night, have fun.

---

**项目地址**:https://github.com/kunchenguid/gnhf