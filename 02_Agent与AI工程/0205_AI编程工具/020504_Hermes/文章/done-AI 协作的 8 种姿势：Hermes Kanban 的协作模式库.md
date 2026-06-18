> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: AI 协作的 8 种姿势：Hermes Kanban 的协作模式库
author: 胖小天
date: 胖小天胖小天
url: https://mp.weixin.qq.com/s?__biz=MzA4OTI0MDM2Mg==&mid=2247484239&idx=1&sn=5359886e94ddd3fcc9f9836d10866b72&chksm=91ac8ac5e57c29eb401bffdf315d4090be3aa0202a65b06375d0f59551e6f4e6cc4f1cbd66b2&mpshare=1&scene=24&srcid=0604GOyHsC0pfoL81DaFZo3U&sharer_shareinfo=b0a94ba70cc1fb6cdc474ce7bd7dc1b2&sharer_shareinfo_first=b0a94ba70cc1fb6cdc474ce7bd7dc1b2#rd
---

引子：Kanban 是什么？

在看 8 种姿势之前,得先搞清楚一件事:**Kanban 是什么**。

你可能听过这个词。在工厂里,看板(Kanban)是丰田发明的——一张卡片,代表"一个工作"。

卡片从「待办」移动到「进行中」,再到「已完成」。

简单吧?

> **Kanban 看板系统 = 一张卡片 + 几个状态 + 谁能看见**。

那跟 AI Agent 协作有什么关系?

Hermes Agent(Nous Research 出品的开源 Agent 框架)把这一套搬到了 AI 世界。

它给 Agent 协作做了个 **"持久化任务板"**。

每个任务是一行,存在 SQLite 里。

每个 worker 是个独立 OS 进程,跑完写一行,挂了留一行,人可以随时看。

**Kanban 在 AI 协作里的角色**:

* ✅ **持久化** —— 任务写到 SQLite,断电不丢
* ✅ **状态机** —— 每个任务有 7 个状态(triage / todo / ready / running / blocked / done / archived)
* ✅ **可视化** —— `hermes dashboard` 打开浏览器,6 列一眼看完全局
* ✅ **可恢复** —— worker 崩了,dispatcher 自动 reclaim,任务不会丢
* ✅ **可审计** —— 每次心跳、每次评论、每次状态切换,都存
* ✅ **可协作** —— 多个 profile(角色)可以同时读/写同一个任务

**它解决的核心问题**:

单 Agent 干不完的事,交给多 Agent,但不能让协作变成"父进程阻塞等子进程"。

> Kanban = **多 Agent 协作的"共享白板"**。每个 Agent 在上面画一笔,所有人都看得见。

有了这个基础,下面讲 8 种姿势才不会悬空。

---

## 一、单 Agent 调用的天花板

你让 AI 帮你做研究分流。

3 个研究员并行。1 个分析师 fan-in。1 个写手输出。

第一次,跑通了。

直到第 4 次,你碰到新场景:用户想中途换角度。
第 5 次,你想让"法律顾问"插一脚,审一下合规。
第 6 次,你想跑 50 个 Instagram 账号,每个独立。
第 7 次,你想做 4 周的"每日简报",跨周累积。

`delegate_task` 全部 GG。

**不是框架弱。**

**是它从设计就只 cover 1 种姿势。**

---

软件工程里有个老办法。

> **把高频出现的解法抽象成"模式"**,让所有人共用同一套词汇。

GoF 23 种设计模式。
Cloud Design Patterns。
MapReduce / Saga / CQRS。

每种模式都有名字、有形状、有适用场景。

**AI Agent 协作也该有自己的"GoF 23 模式"**。

Hermes Kanban v1-spec 给出了答案:**8 种姿势 P1-P8**。

今天我把这 8 种姿势逐个拆给你。

---

## 二、8 种姿势速览

先看地图。

| # | 模式 | 形状 | 一句话 | 典型场景 |
| --- | --- | --- | --- | --- |
| **P1** | **Fan-out** | 1 → N | 1 个父任务拆 N 个子任务,并行执行 | 并行研究、并行编码、批量数据处理 |
| **P2** | **Pipeline** | A → B → C | 链式,角色管道 | 写 → 审 → 改 |
| **P3** | **Voting** | N → 1 | N 个 worker 投票,1 个 fan-in 收口 | 3 个 reviewer 评审 |
| **P4** | **Journal** | 时间累积 | 跨天跨周累积,每次结果是 journal 的一段 | 每天简报、每周周报 |
| **P5** | **Human-loop** | 阻塞 + 决策 | 关键决策点卡 human | 法律审查、战略决策 |
| **P6** | **@mention** | 主动 escalate | Worker 主动 @某 profile,触发跨域协作 | inbox-triage @legal |
| **P7** | **Thread ws** | 隔离 workspace | 按 thread 隔离,跨 thread 不污染 | 飞书多群 |
| **P8** | **Fleet** | 1 × N | 1 个 specialist 跑 N 个独立任务 | 50 账号 Instagram |

记不住?

下面每个配 1 个真实场景 + 1 个 ASCII 形状图,看完就能用。

---

## 三、P1-P8 逐个拆解

### P1 — Fan-out(1 拆 N)

### 形状:1 个父任务 → 拆 N 个子任务 → 并行执行 → 父任务等所有子 done 才完成。

**真实场景**:研究分流。

```
# planner 拆 3 个角度
hermes kanban create "angle: cost"    --assignee researcher --parent T1
hermes kanban create "angle: latency" --assignee researcher --parent T1
hermes kanban create "angle: tools"   --assignee researcher --parent T1
```

3 个 researcher 并行 spawn。T1 等所有子任务 done 才完成。

**何时用**:批量独立工作,无依赖。

> P1 的本质:把"我一个人干 N 次"变成"N 个人各干 1 次"。

---

### P2 — Pipeline(链式)

### 形状:A → B → C,前一个的输出是后一个的输入。

**真实场景**:编码流水线。

* planner 拆任务
* backend-eng 实现
* reviewer 评审 → block → unblock
* backend-eng 修(再 spawn 一次,带历史 context)

**何时用**:角色专精不同,前一阶段产物是后一阶段输入。

> P2 的本质:把"全栈工程师"拆成"专精角色",每个角色只负责自己最擅长的一段。

---

### P3 — Voting(N 选 1)

**形状**:N 个 worker 投票 → 1 个 fan-in 节点收口。

**真实场景**:代码评审 + 多角度验证。

3 个 reviewer 同时评审,aggregator 收集 3 份意见,合并成 1 份报告。

**何时用**:需要多视角验证,或 N 选 1 的决策。

> P3 的本质:把"我相信 1 个 reviewer"变成"3 个 reviewer 互相校验"。

---

### P4 — Long-running Journal(跨天累积)

**形状**:跨天/跨周的任务,每次 run 的结果是 journal 的一段。

**真实场景**:每日 AI 简报。

```
# 每天 9 点跑一次(用 cron)
schedule: "0 9 * * *"
command: hermes kanban create "daily-brief" \
  --assignee scout --body "$(date +%Y-%m-%d)" \
  --workspace dir:~/Obsidian/AI-Funding/
```

30 天 = 30 个 task。每个 task 的 `metadata` 包含当日 findings。所有 `task_runs` 行拼成"30 天 journal"。

**何时用**:累积型任务,价值在历史里。

> P4 的本质:把"每天跑 1 次"变成"持续写 1 本书"。

---

### P5 — Human-in-the-loop Triage(阻塞 + 决策)

**形状**:Worker 跑到决策点 → `kanban_block` → human 评论决策 → `kanban_unblock` → worker respawn 带历史。

**真实场景**:法律审查。

```
# worker 碰到法律问题
kanban_comment(
    body="用户协议里有个 GDPR 风险条款,我看到两个解决方案..."
)
kanban_block(
    reason="GDPR 合规:选择 A(用户明确同意)还是 B(默认不收集 IP)?"
)

# 律师 review + 决策
hermes kanban comment T-xxx "选 A,这是理由..."
hermes kanban unblock T-xxx

# worker 自动 respawn,带完整评论历史
```

**何时用**:决策成本高、不能由 AI 自主决定。

> P5 的本质:把"AI 自主决策"变成"AI 提议 + Human 拍板"。

---

### P6 — @mention Delegation(主动 escalate)

**形状**:Worker 主动 @某个 profile 名,触发跨域协作。

**真实场景**:Inbox-triage 升级到律师。

```
# inbox-triage 收到一封可能涉及合同的邮件
kanban_comment(
    body="这封邮件涉及 NDA 修改,需要 legal 审一下"
)
# 在 Hermes 里 P6 是任务路由机制:
# inbox-triage 不能直接调 @legal
# 而是创建 1 个新 task,assignee=legal,parent=当前 task
hermes kanban create "review NDA" \
  --assignee legal \
  --parent T-current \
  --body "需要审一下用户协议修改"
```

**何时用**:跨域知识,当前 worker 能力不够。

> P6 的本质:把"AI 自己想办法"变成"AI 主动转给更专业的人"。

---

### P7 — Thread-scoped Workspace(隔离 workspace)

**形状**:每个 thread / 上下文有自己的 workspace,跨 thread 不污染。

**真实场景**:飞书群多线并行。

3 个 thread 各自有 kanban board + workspace,worker 物理上不知道其他 thread 存在。

**何时用**:多线并发,且线与线之间数据要严格隔离。

> P7 的本质:把"1 个大杂烩"变成"多个隔离的房间"。

---

### P8 — Fleet Farming(1 specialist × N subjects)

**形状**:1 个 profile 跑 N 个独立任务,各管各的。

**真实场景**:50 账号 Instagram。

```
# dispatch-fleet user-space helper
for acct in $(seq 1 50); do
  hermes kanban create "engage $acct" \
    --assignee insta-manager \
    --workspace dir:~/insta/$acct/ \
    --tenant $acct
done

# dispatcher 自动起 50 个 worker(实际限并发,比如 5 个一批)
```

**17 号被风控?只 block 17 号 task**。其他 49 照常。

**何时用**:N 个相同性质的对象,要"被同样地管"。

> P8 的本质:把"1 个人管 50 个账号"变成"50 个员工各管 1 个账号"。

---

## 四、模式的组合 + 决策

看完 8 种姿势,你可能想:

> "我到底该用哪个?"

答案是:**大多数项目 = 几种模式组合**,不是单选。

### 4.1 4 个真实 user story 的模式组合

| Story | 用到的模式 | 组合方式 |
| --- | --- | --- |
| **Story 1:研究分流** | P1 + P2 + P3 + P5 | planner(P1 拆 4 角度)→ 4 个 researcher 并行 → analyst(P3 fan-in)→ writer(P2)→ P5 关键决策卡人 |
| **Story 2:每日简报** | P2 + P4 | scout → editor → writer(P2 链式)+ 30 天累积(P4 journal) |
| **Story 3:数字孪生** | P4 + P5 + P6 | inbox-triage 长期跑(P4)+ 合同问题 P5 卡律师 + P6 escalate 到 legal |
| **Story 4:编码流水线** | P1 + P2 + P5 | planner(P1 拆 5 模块)→ 5 个 eng 并行 worktree → reviewer(P2 串行)+ P5 模糊决策 |

**关键洞察**:

> **大多数项目 = P1 + P2 + P5 就能 cover**。其他 5 种模式是"专业化场景"的补充。

### 4.2 模式选择决策树

### 4.3 决策清单(更直白)

| 场景 | 推荐模式 |
| --- | --- |
| 批量独立工作(数据处理、并行研究) | **P1** |
| 角色专精 + 链式(写-审-改) | **P2** |
| 多视角验证(代码评审、3 选 1) | **P3** |
| 累积型(每天简报、每周周报) | **P4** |
| 关键决策(法律、伦理、战略) | **P5** |
| 跨域协作(转给律师/医生/财务) | **P6** |
| 多线并行(飞书多群) | **P7** |
| 集群工作(50 账号、12 监控) | **P8** |

---

## 五、为什么"模式库"比"框架"更有生命力

看完上面,你应该感觉:

> "P1-P8 这套姿势库,有点眼熟。"

没错。

**这套思想,跟软件工程的 GoF 23 设计模式一脉相承**。

### 启示 1:模式是知识,框架是工具

* GoF 23 模式:25 年了还在用
* 某个前端框架:3 年就过时

**AI Agent 协作也该有自己的"GoF 23 模式"**。

### 启示 2:模式可以跨框架迁移

* P1 Fan-out 在 LangGraph 里叫 "Map"
* P2 Pipeline 在 AutoGen 里叫 "Sequential Chat"
* P3 Voting 在 CrewAI 里叫 "Hierarchical"

**名字不同,形状一样**。

学会 P1-P8,跨框架 0 成本。

### 启示 3:模式库让"教学"成为可能

* 框架文档:看一遍不会用
* 模式库:看一遍"知道有这么回事",用时翻

**Hermes Kanban 的 P1-P8 是一份"地图",不是"说明书"**。

---

## 六、未来 1 年会出现什么

* **短期(2026 内)**

  :其他 Agent 框架开始"补"持久化协作层(AutoGen Kanban / LangGraph Kanban)
* **中期(2027)**

  :"模式库"成为 AI Agent 协作的标准教学内容(类比 GoF 23)
* **长期**

  :每个 multi-agent 框架都会内置 P1-P8 模式支持

> 框架会过时,模式会留下。
> 学会 8 种姿势,比学 8 个框架更有用。

---

## 引用源

### 官方文档

* Hermes Kanban 官方文档
* Kanban tutorial - 4 个 user stories
* Kanban worker lanes
* kanban-worker SKILL.md

### 设计文档

* hermes-kanban-v1-spec.pdf - 内含 P1-P8 模式库定义(章节 5)

### 仓库

* NousResearch/hermes-agent

###