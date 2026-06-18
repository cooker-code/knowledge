> 已吸收至：[[05_数据分析与BI/0501_语义层与智能问数/050101_Text-to-SQL/050101_核心知识点/Text-to-SQL工程架构与SchemaLinking|Text-to-SQL工程架构与SchemaLinking]]
---
title: 复刻 OpenAI Data Agent 的语义层：Metadata Pipeline 设计与实践
author: 小皮大杰瑞
date: 小皮大杰瑞小皮大杰瑞
url: https://mp.weixin.qq.com/s?__biz=MzcwMDE2OTAyOA==&mid=2247483752&idx=1&sn=3909c3ed1872e705b81a7b5991ce5556&chksm=f5552bc158d95f8d9fb41135e895356f17ab14d3c8c3be197d8d4b22bae7659359c2467fe8f6&mpshare=1&scene=24&srcid=0519Kpisqkq38gYeTymSsOfi&sharer_shareinfo=f2ad3b4b5a985c16ca37822edc62bf28&sharer_shareinfo_first=f2ad3b4b5a985c16ca37822edc62bf28#rd
---

#

上篇文章[OpenAI 内部 Data Agent 的构建原理分析](https://mp.weixin.qq.com/s?__biz=MzcwMDE2OTAyOA==&mid=2247483731&idx=1&sn=efa99fef09694eb8928b932170437011&scene=21#wechat_redirect)分析 OpenAI 内部 Data Agent 的设计时，结尾提出了 Metadata Pipeline 这一概念：

> 在 AI for Data 时代，构建 Metadata Pipeline 与 Data Pipeline 同等重要。需要大量的元数据工程：包括物理元数据、血缘、数据特征的采集和抽样；元数据的语义修正和增强；业务语义的搜集和处理。这些都需要构建对应的 Metadata Pipeline，能够及时地处理和更新企业内部的元数据系统。

OpenAI 的六层上下文体系——schema 元数据、人工标注、Codex Enrichment、机构知识——并非凭空生成，背后依赖一套持续运行的机制完成采集、修正与维护。这套机制，就是 Metadata Pipeline。本文围绕其本质与所解决的问题，借助 Gravitino 与大模型，探索如何在实践中构建它。

---

# 一、什么是 Metadata Pipeline？定义、结构与核心三层架构

在展开之前，有必要先做一个澄清。业界对「Metadata Pipeline」这个词的理解并不统一，本文讨论的是其中一种含义：**将元数据本身作为被处理的对象**，而非用元数据来驱动数据管道的执行。

Metadata Pipeline 的核心，是构建一套持续运行的系统，对元数据进行采集、修正、增强与激活，以保障 AI Agent 始终在正确的业务语义下操作数据。

这一概念在业界尚处于成型阶段，各厂商的命名不尽相同：Atlan 称之为 Active Metadata Pipeline，侧重元数据的持续流动与自动更新；McKinsey 称之为 Context Enrichment Pipeline，侧重为 AI Agent 构建可消费的业务上下文；Collate/OpenMetadata 称之为 Semantic Intelligence Pipeline，侧重语义与业务含义的关联。本文统一采用 **Metadata Pipeline** 这一表述。

理解它，最直观的方式是与 Data Pipeline 做对比：

| 维度 | Data Pipeline | Metadata Pipeline |
| --- | --- | --- |
| 流动的是什么 | 业务数据 | 关于数据的描述和语义 |
| 输出给谁 | 数仓、BI、业务应用 | AI Agent、数据目录 |
| 触发方式 | 定时调度 / 数据到达 | Schema 变更 / 新资产上线 / 定时 |
| 腐烂风险 | 数据过期，有监控告警 | 语义注释过期，通常没有告警 |
| 工程成熟度 | 非常成熟 | 仍在建立规范 |

表中倒数第二行值得特别注意：Data Pipeline 有完善的监控体系，数据延迟会报警、质量下降会告警；但语义注释过期了，什么都不会发生，直到 AI Agent 给出错误答案才被发现。**这正是 Metadata Pipeline 作为独立工程对象存在的核心理由。**

在内部结构上，一个 Metadata Pipeline 大致分三层：

**采集（Collect）**：从各类数据源自动提取物理元数据——表结构、字段类型、数据血缘、使用频率，汇聚到统一的地方。

**修正与增强（Enrich）**：把业务含义注入物理元数据。技术元数据只告诉你字段叫 `amount`、类型是 `DECIMAL`；Enrich 层的工作是把”这是平台收入字段，必须排除 `channel='test'` 的内部测试订单”这类知识写进去。

**激活（Activate）**：通过 API 将增强后的元数据暴露给下游 AI Agent，让 Agent 在生成查询之前就能拿到正确的语义上下文。

有了结构上的定义，下一个问题自然是：这件事为什么值得被当作独立的工程对象来对待？

---

# 二、为什么 Data Agent 需要 Metadata Pipeline：五大根本问题

## 第一层问题：有 Schema 还不够

**语义鸿沟**

数据库存储的是物理世界：字段名、类型、数值。AI Agent 需要理解的是业务世界：这个字段是什么含义、用在什么场景、有什么限制。两者之间存在一道鸿沟。

对人工分析师而言，这道鸿沟通常依靠经验传承来弥合——资深人员告知新成员某字段的口径内涵。当 AI Agent 取代人工分析师后，这一机制失去了作用。Agent 没有经验积累的过程，只能依据被显式记录的信息做出判断。Schema 确定了字段的存在，却未说明其业务含义，也未说明使用限制。这一差距，就是语义鸿沟。

**多源语义割裂**

语义鸿沟在单一数据源场景下尚属可控，但企业数据常常分布于多个异构系统。同一业务概念——以“收入”为例——在交易系统、用户系统与数仓中可能分别对应不同口径的字段定义。缺少跨源统一的语义管理层，AI Agent 在跨源查询时就难以确定该用哪张表、哪个字段。

**隐性知识的显式化**

上篇文章在介绍 OpenAI Data Agent 的 Memory 系统时提到：查询某实验数据时，需要过滤一个特定字符串，而该字符串仅存在于配置文件中，从未进入任何 schema 或文档。此类使用约束与已知陷阱，往往散落于即时消息、代码注释、历史事故记录和相关人员的经验之中，既未纳入 schema，也缺乏系统性归档。Metadata Pipeline 的职责之一，就是主动采集并结构化此类非正式知识，将其转化为机器可消费的标注，供 Agent 在决策时加以利用。

以上三点，是 schema 本身无法解决的问题。但即便完成了一次完整的语义标注，挑战并未结束。

## 第二层问题：标注一次还不够

**语义腐烂**

语义标注会腐烂。业务口径在调整，表结构在演进，新的数据资产不断上线，一次性的标注会迅速过时——而且过时是无声的。Data Pipeline 挂了会有报警，语义注释失效了却没有任何提示。这就要求 Metadata Pipeline 必须持续运行，而不能只是一次性的项目。

**可信度与时效性**

即便语义标注仍然存在，AI Agent 也无从判断该描述形成于何时、由何人撰写、是否已经过验证。引用一条已过期或错误的描述，其潜在危害不亚于没有任何描述。因此，Metadata Pipeline 需要在每条元数据上维护验证状态与时效信号，使 Agent 及人工均能评估其可信程度。

**可发现性**

元数据再完整，在 OpenAI 这种规模——7 万张表——AI Agent 如何从中快速定位最相关的几张，仍然是独立的挑战，需要向量化和语义检索能力的支撑。这超出了本文的讨论范围，留待后续展开。

以上五个问题，共同指向同一个结论：**元数据需要被当作与业务数据同等重要的工程对象来对待，而不是一次性的人工整理任务。** 下面通过一个具体的 Demo，来看这件事在实践中是什么样的。

# 三、实战：用 Gravitino + LLM 构建 Metadata Pipeline（完整 Demo）

## 场景：ShopFlow 的收入之谜

ShopFlow 是一家电商平台，数据分散在三个系统。Data Agent 刚上线，产品总监就抛来了一个看似简单的问题：**「上个月平台的收入总额是多少？」**

三个系统里都有名为 `amount` 的字段：

| 数据源 | 表 | 字段 | 真实口径 |
| --- | --- | --- | --- |
| 交易系统（MySQL） | `orders` | `amount` | 订单金额，含测试订单 |
| 用户系统（PostgreSQL） | `members` | `amount` | 用户累计消费额，非当期收入 |
| 数仓（Iceberg） | `dw_orders` | `amount` | 平台收入，但必须排除 `channel='test'` 的内部测试订单 |

三张表，三个同名字段，Gravitino catalog 里没有任何描述——都是 `DECIMAL` 类型，comment 全部为空。Agent 不知道该选哪个，更不知道数仓里还藏着一个必须过滤的隐性规则。

这条规则的来源是一次真实事故：2024 年 11 月，有人查平台收入时漏掉了 `channel='test'` 的过滤，导致一份财务报告的数字虚高了约 15%。事后教训被记录在 dbt 的 meta 字段里，但从未被正式化到任何字段描述或文档中——它就这样静静地躺在那里，等着下一次被人遗忘。

同一个问题，是否经过 Metadata Pipeline 的元数据增强，会得到截然不同的答案。

## 输出范式：四段式 Comment

在动手构建之前，需要先确定一件事：经过 Pipeline 处理后的元数据，应该长什么样？

原始物理元数据需要经历两类操作。**修正**，是补全或纠正那些模糊、缺失的描述——三张表的 `amount` 字段描述全部为空，口径各不相同，必须逐一说清楚。**增强**，是补充那些压根不存在于任何 schema 里的信息——比如 `dw_orders.amount` 必须排除 `channel='test'` 的内部测试订单，这条规则只存在于事故记录中，不在字段名或表名里。

本 demo 约定了一套四段式 comment 格式来承载这些信息：

```
【口径】字段的业务含义和计算方式（一到两句话）
【适用】适合用在哪类查询场景，不适合用在哪里
【注意】使用限制、必要过滤条件、已知陷阱（没有则省略）
【来源】LLM推断 | dbt描述 | dbt事故记录 | 迁移继承
```

四段式的设计对应了第三章提到的几个问题：【口径】填补语义鸿沟，【适用】帮助 Agent 判断字段匹配度，【注意】显式化隐性约束，【来源】提供可信度信号。有了统一结构，Pipeline 的输出才是可验证的，Agent 也能更可靠地从中提取约束条件。

## 工具链

有了范式，下一个问题是：用什么工具来实现它，各自承担什么角色？

**Gravitino** 是存储层和统一接口。它把 MySQL、PostgreSQL、Iceberg 等异构数据源统一纳入一个 Metalake，提供跨源一致的 REST API。所有 catalog 的注册、column comment 的读写都通过它完成。对 Pipeline 来说，Gravitino 屏蔽了底层数据源的差异，使得整套逻辑只需面对一套 API。

**dbt manifest** 是初始知识的来源。真实的 dbt 项目编译后会产出 `manifest.json`，里面有字段描述（质量参差不齐）、数据血缘和自定义的 meta 字段。本 demo 用静态文件模拟，其中三张表的 `amount` 字段描述全部为空，`dw_orders` 的 meta 字段里藏着那条关于 `channel` 过滤的事故记录——这是 Pipeline 的原始输入。

**LLM（Claude API）** 是语义增强引擎。它将字段名、类型、血缘、dbt 残缺描述和事故记录一并处理，生成符合四段式格式的完整 comment。这是 Pipeline 中唯一需要「理解」的环节，也是将隐性知识转化为结构化标注的核心步骤。

**APScheduler** 提供调度和触发机制。它让 Pipeline 不只是一个一次性脚本，而是能持续运行、感知变化的工程系统，支持定时触发和 schema 变更事件触发两种模式。T3 幕中 Pipeline 的自动修复，正是由它驱动的。

四者的协作关系如下图所示：

## 四幕演示

前面用了较多篇幅解释为什么需要 Metadata Pipeline，以及用什么工具来实现它。这一节换一种方式——用一个连续的故事，把所有问题和工具串起来，看它们在真实场景里是怎么发生的。

贯穿四幕的问题只有一个：**「查询上个月平台的收入总额」**

---

### T0：混乱现场

Metadata Pipeline 尚未运行。Gravitino catalog 里三张表都有 `amount` 字段，都是 `DECIMAL` 类型，comment 全部为空：

```
transaction_db.ods.orders     amount   DECIMAL   comment: ""
user_db.public.members        amount   DECIMAL   comment: ""
data_warehouse.dwd.dw_orders  amount   DECIMAL   comment: ""
                              channel  STRING    comment: ""
```

那条 2024 年 11 月的事故记录——“漏掉 `channel='test'` 过滤，收入数字虚高 15%”——仍然静静地躺在 dbt 的 meta 字段里，从未被正式化：

```
nodes:
  dw_orders:
        columns:
      amount:
        description:""
        meta:
          known_issues:
            -date:"2024-11-03"
             description:"未排除 channel='test' 的内部测试订单，导致收入数据虚高约 15%"
             filter_required:"channel != 'test'"
      channel:
        description:""    # ← 空白，无任何业务暗示
      dt:
        description:"日期分区"
```

Agent 选了看起来最合理的 `dw_orders`，生成了这条 SQL：

```
SELECT SUM(amount) AS total_revenue
FROM data_warehouse.dwd.dw_orders
WHERE dt >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1' MONTH)
  AND dt < DATE_TRUNC('month', CURRENT_DATE)
```

没有 `WHERE channel != 'test'`。内部测试订单被计入，结果虚高约 15%。这不是模型的问题——那条过滤规则不在任何字段名或表名里，Agent 根本无从知晓。

---

### T1：Metadata Pipeline 第一次运行

Metadata Pipeline 被手动触发，四个阶段依次执行：

Pipeline Run Report（实际运行结果）：

```
══════════════════════════════════════════════════════
  PipelineRun Report
  Run ID   : run-e9b22d4d  |  Trigger: manual
  Status   : ✅ SUCCESS    |  Duration: 28115ms

  Stage                  Status       In  Out      ms
  ---------------------- ---------- ---- ---- -------
  Stage-1 Collect        ✅ SUCCESS     3   15       0
  Stage-2 Enrich         ✅ SUCCESS    15   15   27848
  Stage-3 Annotate       ✅ SUCCESS    15   15     267
  Stage-4 Validate       ✅ SUCCESS    15   15       0

  语义覆盖率 : 15/15 (100%)
  隐性知识   : 1 条事故记录 → 转化为【注意】段
══════════════════════════════════════════════════════
```

LLM 读到了 dbt meta 字段里的事故记录，将其转化为结构化 comment，写回 Gravitino catalog：

```
data_warehouse.dwd.dw_orders.amount:
【口径】订单金额，包含订单原始金额、退款和结算调整后的最终金额数据。
【适用】适合用于收入统计、财务分析和业务指标计算。
【注意】⚠️ 必须配合 WHERE channel != 'test' 使用，排除内部测试订单，
        否则数据虚高约 15%（见 2024-11-03 事故记录）。
【来源】LLM推断 + dbt事故记录（2024-11-03）

data_warehouse.dwd.dw_orders.channel:
【口径】订单来源渠道标识，区分不同业务渠道的订单。
【适用】适合用于渠道维度分析；在收入统计时必须用于过滤测试数据。
【注意】'test' 对应内部测试订单，占总量约 15%，收入统计必须排除。
【来源】LLM推断 + dbt事故记录
```

这一次，Agent 从【注意】段提取到了过滤条件，生成的 SQL 是正确的：

```
SELECT SUM(amount) AS total_revenue
FROM data_warehouse.dwd.dw_orders
WHERE dt >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1' MONTH)
  AND dt < DATE_TRUNC('month', CURRENT_DATE)
  AND channel != 'test'
```

那条在事故记录里躺了半年的隐性知识，第一次进入了 Agent 的视野。Pipeline 首次运行的价值，到这里已经体现出来了。但故事还没结束。

---

### T2：Schema 变更，语义腐烂

三个月后，数仓团队将 `dw_orders` 按地区拆分为两张表，`amount` 同时改名为 `net_amount`。这是完全正常的技术决策，没有人意识到语义层会受影响：

```
SCHEMA_CHANGE_EVENT = {
    "type": "TABLE_SPLIT",
    "timestamp": "2025-04-01T09:00:00Z",
    "changes": {
        "deprecated":    ["data_warehouse.dwd.dw_orders"],
        "created":       ["data_warehouse.dwd.dw_orders_cn",
                          "data_warehouse.dwd.dw_orders_global"],
        "field_renames": {"amount": "net_amount"}
    }
}
```

Gravitino catalog 的状态随之变成了这样：

```
data_warehouse.dwd.dw_orders（旧表，已废弃）
  amount   DECIMAL  comment: "【口径】..."  ← comment 还在，但字段已废弃

data_warehouse.dwd.dw_orders_cn（新表）
  net_amount  DECIMAL  comment: ""   ← 语义腐烂
  channel     STRING   comment: ""   ← 语义腐烂

data_warehouse.dwd.dw_orders_global（新表）
  net_amount  DECIMAL  comment: ""   ← 语义腐烂
  channel     STRING   comment: ""   ← 语义腐烂
```

新表没有任何 comment，旧表的 comment 还在但表已经废弃。Agent 无法判断哪张表是可用的，退而选择了有语义描述的旧表——结果是查询报错或拿到空数据。

这正是语义腐烂的典型形态：Data Pipeline 挂了会报警，但语义注释失效是无声的，什么都不会发生，直到 Agent 给出错误答案才被发现。

---

### T3：Metadata Pipeline 感知变更，自动修复

APScheduler 检测到 schema 变更事件，立即触发新一轮 Pipeline 运行：

这一次无需人工介入，Pipeline 运行结果如下：

```
════════════════════════════════════════════════════
  PipelineRun Report
  Run ID   : run-155c9998  |  Trigger: schema_change_event
  Status   : ✅ SUCCESS    |  Duration: 52ms

  变更处理摘要：
  字段                              状态        说明
  ─────────────────────────────────────────────────
  dw_orders_cn.net_amount       MIGRATED    ⚠️ 建议人工复核口径
  dw_orders_global.net_amount   MIGRATED    ⚠️ 建议人工复核口径
  dw_orders.amount              DEPRECATED  ❌ 已废弃
════════════════════════════════════════════════════
```

Pipeline 将旧表的语义自动迁移到两张新表，并标注 `MIGRATED` 状态——这是系统的边界声明：能做到自动迁移，但不能替代人对口径变化的判断，因此留下了需要人工复核的信号。

Agent 读到新表的 comment，生成了正确的 SQL：

```
-- Agent 找到两张新表，都有 MIGRATED 状态的 comment
-- 从【注意】段提取过滤条件：channel != 'test'
SELECT SUM(net_amount) AS total_revenue
FROM (
  SELECT net_amount
  FROM data_warehouse.dwd.dw_orders_cn
  WHERE dt >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1' MONTH)
    AND dt < DATE_TRUNC('month', CURRENT_DATE)
    AND channel != 'test'
  UNION ALL
  SELECT net_amount
  FROM data_warehouse.dwd.dw_orders_global
  WHERE dt >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1' MONTH)
    AND dt < DATE_TRUNC('month', CURRENT_DATE)
    AND channel != 'test'
) combined;
```

---

### 四幕回顾

| 时间点 | 状态 | Agent 回答 | 暴露的问题 |
| --- | --- | --- | --- |
| T0 | 无语义层 | ❌ 缺少 channel 过滤，虚高 ~15% | 隐性业务规则无法推断 |
| T1 | Pipeline 首次运行后 | ✅ 正确，自动加了过滤条件 | 隐性知识注入 |
| T2 | Schema 变更后 | ⚠️ 回退使用已废弃旧表 | 语义腐烂 |
| T3 | Pipeline 自动修复后 | ✅ 拆分表合并查询，正确恢复 | Pipeline 持续运行的价值 |

这四幕不是四个独立的问题，而是同一件事在时间轴上的展开：语义鸿沟、隐性知识、语义腐烂、可信度——它们都是元数据未被当作工程对象持续维护所导致的结果。Metadata Pipeline 要解决的，正是这整条时间轴。

完整的 demo 可以使用 Gravitino playground 来运行，代码示例见 https://github.com/jerryshao/metadata-pipeline-demo。启动 Gravitino playground，配置 Anthropic API key 即可运行。

---

# 总结

上篇分析了 OpenAI 如何搭建 Data Agent，本文则从 Metadata Pipeline 的角度，尝试在实践中复现其语义层的核心机制。不过，真正落地时，Metadata Pipeline 不该被期待成”自动生成正确答案”的万能层。它更现实的价值，是把原本分散在表结构、文档、事故复盘和人脑里的语义，尽可能稳定地沉淀下来，并让这些信息以可追踪、可验证、可更新的方式流动起来。

进一步看，这套系统的难点往往不在第一次写入，而在持续维护：哪些需要人工复核，哪些规则应该随业务变更自动失效，哪些历史事故值得升级成强约束，哪些字段只适合在特定场景下使用。这些判断一旦纳入流程，Metadata Pipeline 才真正从”注释工具”变成”语义基础设施”。

这件事不应被理解为一次性的治理项目，而应被看作一套需要长期运行的基础设施。随着 schema 变化、业务口径调整、事故经验沉淀，元数据本身也在持续演化。真正有价值的，不是某一次注释写得多完整，而是这套注释能否持续被采集、修正、验证和激活，能否在变化发生时及时更新，能否让 Agent 始终站在最新的业务语义上工作。

如果你的团队现在就想迈出第一步，不必一开始就把整套方案搭完。先从最关键的几张表入手，把高频使用、最容易出错的字段标注清楚，再把那些隐藏在事故记录和历史经验里的约束补进来，往往就已经能带来明显的提升。Metadata Pipeline 的起点，不是”大而全”，而是把最重要的那一小块先做对。