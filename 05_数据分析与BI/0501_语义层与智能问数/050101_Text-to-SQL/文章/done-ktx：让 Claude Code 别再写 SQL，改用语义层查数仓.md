> 已吸收至：[[05_数据分析与BI/0501_语义层与智能问数/050101_Text-to-SQL/050101_核心知识点/Text-to-SQL工程架构与SchemaLinking|Text-to-SQL工程架构与SchemaLinking]]
---
title: ktx：让 Claude Code 别再写 SQL，改用语义层查数仓
author: 向量猫
date: 向量猫向量猫
url: https://mp.weixin.qq.com/s?__biz=MzYzNzg4ODc5NQ==&mid=2247486424&idx=1&sn=3bab7bf16c3287173bfebab83e180d33&chksm=f1ae44001621acb83daf4a1b6f54132e35ec7aed7652bcefdce5af2186e0c26079bf18e122c9&mpshare=1&scene=24&srcid=0604halB7jGZmbO5hH5sCdri&sharer_shareinfo=c4a0b7cd8dcd244a8eed208797a01c92&sharer_shareinfo_first=c4a0b7cd8dcd244a8eed208797a01c92#rd
---

> Claude Code 写仓库 SQL 永远是「合法但不正确」，ktx 想把 SQL 生成权从 LLM 手里收回来。

## 让 Claude Code 跑数仓的三种翻车现场

你让 Claude Code 去公司数仓里跑一句「按客户分群算一下 ARR」。

它给你写了一句结构上完全没毛病的 SQL。`SELECT` 没错，`JOIN` 没错，`GROUP BY` 没错。

数字也是错的。

ktx 作者 lucamrtl 在 Show HN 帖子里直接列了三个最常见的翻车现场，每一个我都见过同事踩过：

第一种叫**过期列加隐藏业务规则**。`accounts.industry` 这列还在表里，但内部早就改用了别的字段，而且所有历史报表都默认要排除 suspended subscriptions。Agent 不知道这两件事，跑出来的数和 BI 报表对不上。麻烦的是，数字看起来还挺「合理」——只是悄悄差了 8%，没人会去复核。

第二种叫 **join fanout**。Agent 把 `orders` join 上 `order_items`，然后对 `orders.total_amount_cents` 求和。一个订单有三个行项目，金额就被算了三遍。这在数据仓库里叫 fan trap，是经典坑，但 LLM 不懂表的 grain（行粒度），它只看 schema 名字猜关系。

第三种是**归因逻辑缺失**。`marketing_touches → users → orders` 一通 join，但 first-touch、last-touch、multi-touch 到底用哪个？Agent 直接默认你想要什么都给你 join 上，结果是个谁也认不出的怪物。

这三种错误的共同点是：SQL 本身没语法问题，跑得出来，看起来对。但业务层面上完全错。它们的危险在于「静默」——和那种 `column does not exist` 报错完全不同，错误数据会被直接复制进 Slack、决策会议、季度汇报。

## ktx 不是让 Agent 写更好的 SQL，是让 Agent 别写 SQL

这是我读完 README 和 HN 帖子后最大的发现。

市面上绝大多数 text-to-SQL 工具的思路是：让 LLM 看更多 schema、更多注释、更多示例，期望它写出更靠谱的 SQL。本质还是在赌概率。Vanna、Cursor 内置的数据库 mode、各家「AI 数据分析师」产品都是这条路。

ktx 反过来。它的 planner 是确定性的——agent 不再「写 SQL」，而是发一个声明式请求：我要这个 measure、按这些 dimension 切、加这些 filter。剩下的事 ktx 来：从语义层里查这个 measure 的定义，沿着 join graph 找路径，用 grain 元数据自动检测 fan trap 和 chasm trap，然后编译出仓库能跑的 SQL。

作者原话是：「agent fetches metrics declaratively instead of rewriting canonical SQL each time」。

把「可能正确」换成「定义层正确」。这是和 Cursor 那种 text-to-SQL 完全不一样的路线。换个说法：LLM 负责理解用户问题、选 measure、选 dimension；编译器负责生成 SQL。各做各擅长的事。

这种分工还有个隐性收益：可审计。每条 query 都能追溯到具体的 measure 定义，定义又在 git 里。出问题不是「让 LLM 再跑一遍看看」，而是去 PR history 里翻是谁、什么时候改了什么定义。

## 它做了一件被忽视的事：把语义层和「部落知识」合在一起

数据领域过去几年其实分成了两派工具。

一派是**语义层**：Cube、dbt MetricFlow、Wren（社区也叫 WrenAI）。能编译出正确的 SQL，但需要工程师手写 MDL/语义模型。门槛高，维护贵，小团队压根用不起来。语义层只懂「结构化的定义」，不懂「为什么这么定义」——比如「客户活跃」为什么用 30 天而不是 7 天，文档里通常没写。

另一派是**公司大脑**：类似 Glean、Guru 这种企业知识搜索工具，本质是把 wiki、Notion 全文索引一下。能告诉你「退款政策写在哪」，但没法安全地查仓库。

lucamrtl 在 HN 评论里点名和 Wren、Cube、dbt MetricFlow 做了对比：ktx 是同时做这两半。

具体怎么做：

**业务上下文**进 Markdown wiki，分 global（共享业务定义）和 user（个人笔记）。ingestion agent 自动从 Notion、内部 wiki 把内容抓进来，去重、标矛盾。比如「ARR」在三个文档里出现，定义略有差异，ingestion 会标出来等人确认，而不是默默挑一个用。

**可查询定义**进 YAML，每个语义源描述一张表的 grain（行粒度）、joins、measures、dimensions、filters、filter groups。一个最小的 YAML 大概长这样：

```
name: orders grain: [order_id] measures:   revenue:     expression: sum(total_amount_cents) / 100   order_count:     expression: count(distinct order_id) joins:   - to: customers     on: orders.customer_id = customers.id     relationship: many_to_one filters:   exclude_test:     expression: is_test = false
```

这部分可以从已有的 dbt、Looker、Metabase、MetricFlow 定义里反向 ingest 出来，不用从零写。

两边交叉链接。Agent 通过 lexical + semantic 双路检索（用 RRF 融合，也就是 Reciprocal Rank Fusion）找入口，再沿链接遍历收集上下文。一次问答可能从 wiki 起步（「退款怎么算」），跳到 YAML 的 measure 定义（`refund_amount`），再跳到另一篇 wiki（解释为什么 chargeback 单独算）。

对数据团队来说这意味着：你已经在 dbt 里写好的 model、在 Looker 里建好的 explore，ktx 直接吃进来变成语义层。不用再为 AI 单独维护一份。这一点对中型数据团队特别关键——重复维护一套定义的成本，比直接放弃 AI 还高。

## 三条命令上手，CLI + MCP 两种用法

支持的数仓：PostgreSQL、Snowflake、BigQuery、ClickHouse、MySQL、SQL Server、SQLite。集成对象：dbt、MetricFlow、LookML、Looker、Metabase、Notion。覆盖面在同类开源项目里算齐的。

安装两条路径，看你是不是 agent 重度用户：

```
# 路径一：自己装 npm install -g @kaelio/ktx ktx setup  # 路径二：让 agent 替你装 npx skills add Kaelio/ktx --skill ktx
```

后者直接把 ktx 注册成 Claude Code / Codex 的 skill，agent 在需要时自己调用。如果你已经在用 Claude Code 的 skill 体系，这一行就够了。

核心命令就六个：

| 命令 | 作用 |
| --- | --- |
| `ktx setup` | 初始化或更新项目 |
| `ktx status` | 检查项目就绪状态 |
| `ktx ingest` | 为所有数据连接构建上下文 |
| `ktx sl 'revenue'` | 在语义层里搜 |
| `ktx wiki 'refund policy'` | 在 wiki 里搜 |
| `ktx mcp start` | 启动 MCP server 给 agent 调用 |

实际工作流大概是这样：开 Claude Code，问「上个月 enterprise 客户的净留存是多少」。Claude 通过 MCP 调 ktx 的 search 工具找 measure 定义，发现 `net_retention` 在 `subscriptions.yaml` 里，grain 是 `customer_id × month`，default filter 是 `exclude_test = true`。Claude 再调 ktx 的 query 工具，传声明式参数，拿到结果。整个过程它没碰过一行 SQL。

设计上 read-only，不会动你的数仓。运行用你自己的 LLM API key 或 Claude Pro/Max 订阅，ktx 本身不另外收钱，Apache-2.0 协议。这一点对企业部署友好——数据不出网，定义在自己 git 里，工具开源可审计。

## 一个少有人讲的技术选型：file-first 而非 graph DB

同类工具基本都用向量库或者图数据库存上下文。ktx 反其道而行——所有东西都是 md + yaml + git。

这个选择在 README 里没怎么强调，但维护者 andreybavt 在 HN 评论里点明了。我觉得这是工程团队评估 ktx 时最该看的一点。

三个好处：

**1. 可读、可 PR review、可 diff。**语义层的每次修改都能 code review。新人加入直接看 repo，不用学新工具的 UI。

**2. 天然兼容现有工作流。**你的 dbt repo 怎么管，ktx 项目就怎么管。CI/CD、branch protection、code owner 全套都能复用。

**3. git history 就是业务知识演化轨迹。**哪一天哪个 measure 改了定义，谁改的，一目了然。出现报表口径变化时，能 blame 到具体 commit。

代价当然有：检索性能不如专门的向量库快，超大规模（几千个 model）下可能要做分片。但对绝大多数中小数据团队，瓶颈根本不在这。

ingestion 自动去重合并简单冲突，复杂矛盾留人类处理。这个分工很现实——别指望 LLM 帮你做架构决策。比如同一个指标在不同部门有不同口径，ktx 会把两个定义都保留并标矛盾，等数据负责人来决断。

据维护者在 HN 评论区透露，他们正在准备 Spider 2 benchmark（text-to-SQL 业界基准）的提交，围绕 link detection 做对比开发。这个 benchmark 比传统的 Spider 难得多，是真实企业级数仓 schema 上的复杂多表查询。结果出来值得追一下，因为它能正面回答「确定性 planner 路线到底比纯 LLM 路线强多少」。

## 适合谁，不适合谁

**适合**：已经在用 dbt / Looker / Metabase 的中小数据团队，想让团队里的工程师用 Claude Code / Cursor 直接问业务问题；正在被「AI 给的数和报表对不上」反复折磨的数据负责人；想给数据 agent 加确定性 guardrail 的 infra 团队。

**不适合**：完全没有语义层基础、表都没注释、连 dbt 都还没用的小团队——ktx 能从已有定义反向 ingest，但不能凭空造定义，你得先有「定义」这个东西；只想试个 demo 看 agent 写 SQL「酷不酷」的人——ktx 的价值要接到真实的 fan trap、chasm trap、口径冲突场景才看得出来；纯 BI 用户，不接 agent 客户端——ktx 的核心交付是 MCP 接口，不是给人用的查询 UI。

550+ stars、400+ commits、Show HN 83 分，作为一个刚被广泛讨论的项目，节奏可以。重点是它指了一个明确方向：**让 AI 做声明式查询，让确定性 planner 做 SQL 编译**。这个分工比「把 schema 全塞给 LLM 然后求它写对」靠谱得多。

如果你最近正在评估「怎么让数据 agent 在生产环境跑得安全」，ktx 至少是个该看一眼的参考实现。哪怕不用它，里面 file-first + 双层检索 + 声明式 query 接口的设计思路，搬到自家系统里也能少踩几个坑。

---

> 这里是向量猫，帮你从 100 个工具里挑出 3 个真正值得用的。

参考来源

[1] ktx GitHub 仓库：https://github.com/Kaelio/ktx-ai-data-agents-mcp-context-skills

[2] Show HN: Ktx – Open-source executable context layer for data agents：https://news.ycombinator.com/item?id=48309986

[3] lucamrtl 在 HN 评论里关于 Wren / Cube / dbt MetricFlow 的对比说明：https://news.ycombinator.com/item?id=48313227

[4] 维护者 andreybavt 在 HN 评论里关于 file-first 选型与 Spider 2 benchmark 准备工作的说明：https://news.ycombinator.com/item?id=48316780