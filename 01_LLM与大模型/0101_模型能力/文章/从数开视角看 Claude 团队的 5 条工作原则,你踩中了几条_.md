---
title: 从数开视角看 Claude 团队的 5 条工作原则,你踩中了几条?
author: 小友Data+AI
date: 小友Data+AI小友Data+AI
url: https://mp.weixin.qq.com/s?__biz=MzYyMzMzNzk0MQ==&mid=2247484166&idx=1&sn=7204021364f770c33e845692d12e8ca6&chksm=fe1bf63ffc9da62d8bc45ddcbaee7b559d78cf78e2431db3b3157b559f300c2a297e545be6af&mpshare=1&scene=24&srcid=0604DX45lDm1GtEyMYoMrCVx&sharer_shareinfo=13d1e0e439869baccbf2121131d5dff6&sharer_shareinfo_first=13d1e0e439869baccbf2121131d5dff6#rd
---

> 刷到 Claude Code 工程总监 Fiona Fung 的分享。关于**她分享的相关内容，我们团队几乎在以同样的逻辑探索**。

> 5 条做成 5 道选择题。**先猜，看答案，再看数开场景怎么落**。

---

# 

# 题 1:规划

你上次按计划完整跑完一个季度，是什么时候?

A. 上个季度,严格执行B. 一半按计划C. 头一个月就被打乱D。写完就再没看过

Claude 答案:C/D。Fiona 入职时写的"六个月路线图"，3 个月就过时。改用 JIT 规划 — 直接在 PR / 原型讨论，不写设计文档。

## 数开视角

数据团队的"年度规划"过期更快:

| 变化源 | 频率 |
| --- | --- |
| 上游业务方重构 | 半年 1-2 次,字段全变 |
| 治理团队改指标定义 | 一年 3-5 次 |
| 新业务线接入 | 一年 2-3 次 |
| 监管口径变化 | 不定期 |

我做项目mvp：不写架构白皮书，做 3 层 vs 5 层两版 demo 跑通对比。10 分钟胜过 2 小时讨论。

---

# 

# 题 2:技术分歧

对建模方案吵不下来(3 层 vs 5 层 / Kuzu vs Dgraph),第一反应?

A. 开会 + PPTB. 拉群讨论C. 让 Claude 各跑一版 + 评测D. Leader 拍板

Claude 答案:C。

> **Building is cheap. Arguing is expensive.**

## 数开视角

以前对着架构图争 1.5 小时 ODS 该不该带 partition。现在 Claude 各做一版跑评测：

3 层 vs 5 层 → 9500 query 跑 P95，20 分钟决策

Kuzu vs Dgraph → schema 翻译 + 经典查询，1 小时决策

BGE vs OpenAI → Hard Negative 跑 NDCG@10，半天决策

---

# 

# 题 3:重复活

今天你重复做了 3 次以上的事,怎么处理?

A. 习惯了B. 写 shell 脚本C.追问:能不能自动化?Claude 十分钟搞定D. 让实习生做

Claude 答案:C。重点不是这一件,是养成肌肉记忆。

## 数开视角

| 重复活 | 老做法 | 新做法 |
| --- | --- | --- |
| 看 Airflow | 手工开 UI | hook + Slack 自动汇总失败 |
| ETL 失败排查 | 35 分钟人工 | Skill + 哨兵 → 一句话报告 |
| BI 问字段口径 | 翻 Confluence | MDL YAML + `/dw-explain` Skill |
| SQL Review | 人工逐条 | sqlglot 静态 + 沙箱 dry-run |
| 字段血缘 | 2 人 1 周 | sqlglot.lineage() CI 跑 |

我团队规矩:重复 3 次以上必须自动化。

以前自动化要 1 天,只有高频高价值才值得。现在 Claude 十分钟搞定,ROI公式被推翻— 几乎所有重复都该自动化。

---

# 

# 题 4:人才

招新数开,优先约谁?

A. 一年写 200+ ETL 的产出王B. 大厂出身有光环C.业务规则提炼强 + 能识别"看着对其实错"的数据D. 算法功底深

Claude 答案:C。

> **Taste is scarce. Typing is not.**

> 不在乎你一小时写多少代码。在乎**你选择做什么**,**怎么知道它对**。

## 数开视角

写SQL不稀缺,设计ADS schema 才稀缺。

我踩过的坑:86 张 ADS 表全用英文缩写(ts / ret\_1y / mdd),BI 习惯了。接 Data Agent 后,LLM 把「今年回撤」选成「3 月回撤」。根因:schema 没语义。

用 MDL YAML 重写,每字段加 description + business\_rule + sample:

- name: mdd\_ytd

description: 近一年最大回撤

business\_rule: 月初最高点到本日最低点的最大跌幅 %

sample: [-12.34, -5.67, -8.90]

NL2SQL 准确率 76% → 89%,一半涨幅来自 schema 重构。换更强的 LLM 拉不动,schema 加语义就能拉。

数开的"品味":schema 让下游一眼读懂、business\_rule 能 cover 边界、看到异常 5 分钟说出根因。不写代码,价值高 10 倍。

---

# 

# 题 5:流程

团队有个开了 2 年的「DA周会」,8 人 1 小时,大家都低头看电脑。你?

A. 继续,是传统B. 改议程C.站出来问:为什么还在开?D. 不是我组织的

Claude 答案:C。Fiona 之前团队也这样,她问"为什么还在开?",所有人才意识到 — 这会根本不需要。

> **人不会主动删流程,只会叠新流程。**

## 数开视角

| 流程 | 当年合理 | 真实状态 |
| --- | --- | --- |
| 每周一 DA 周会 | 同步指标变化 | Grafana 自动告警,周报没人看 |
| 每周三治理评审 | 评审上线 | datacontract CI 卡住,变冗余 |
| 每月 DQ 月报 PDF | 让管理层看到 DQ | PSI + Slack 推,月报没人翻 |
| ETL 上线 3 人 Review | 防事故 | sqlglot + 沙箱 + 灰度,人工 Review 形同虚设 |

大家心里都知道这些会死了,但没人说。

我最近做的:周会缩 15 分钟 standup;月报砍;治理评审 PR 自动化。没有人反对。

---

你踩中了几条?

数你选 C 的次数:

0-1 条:工作模式可能停在 5 年前。今天找一件重复的事自动化掉

2-3 条:已在 AI 转型路上。下一步:把"转"变成"系统性重设计"

4-5 条:你和 Fiona 想得一样。这种人稀缺

---

# 

# 最后

底层思维就三句:

> **遇到重复,自动化。遇到没用的流程,删掉。遇到不需要人判断的事,交给 AI。**

数开过去 10 年堆叠了太多"当年合理但现在多余"的流程。AI 让"写 SQL"变便宜，也给了我们重新设计工作模式的机会。

不是买几个 Claude 会员就叫"AI 原生"。真正的 AI 原生数开,是规划、评审、人才、知识管理每层都重新设计过的。

---

> **Pick your noisiest workflow.Ask if it still earns its place.**

**找到你最繁琐的工作流，问它，**是不是还配占着这位置**。**

**Data + AI 项目获取：**

**公众号后台回复【项目】**