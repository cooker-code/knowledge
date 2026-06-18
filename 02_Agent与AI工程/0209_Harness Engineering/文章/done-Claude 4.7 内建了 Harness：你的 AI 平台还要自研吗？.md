> 已吸收至：[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/Harness边界与演进准则|Harness边界与演进准则]]
---
title: Claude 4.7 内建了 Harness：你的 AI 平台还要自研吗？
author: dolphin07
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg3NjgyODE2MQ==&mid=2247483812&idx=1&sn=10c5d20479e15133b2e9d0d77a8059db&chksm=ce161c757a006d21c73d500e753deb370b290bc83682735308c17f986d29e143942f6d4e935e&mpshare=1&scene=24&srcid=050889GNNz50ujnjWtJ2XP2Q&sharer_shareinfo=6ddf896766fe6863b008112e2b8de9b5&sharer_shareinfo_first=6ddf896766fe6863b008112e2b8de9b5#rd
---

# Claude 4.7 内建了 Harness：你的 AI 平台还要自研吗？

## 📌 本篇速览

* • **一句话**：Claude 4.7 随模型释出的"内建 Harness"让很多 AI 平台的自研规划一夜变成沉没成本。本文给你一条判定法和一张护城河地图，分清哪些该自建、哪些该买。
* • **适合谁**：AI 平台负责人 · 架构师 · CTO · 正在"自研 AI 基座"和"抱 SDK 大腿"之间摇摆的人
* • **阅读时长**：约 10 分钟
* • **读完你会带走**：

1. 1. **一条判定法** · 12 个月测试 —— 判断任何 AI 能力该自建还是该买
2. 2. **一张护城河地图** · 通用 Harness vs 私域 Harness 的完整边界
3. 3. **一份选型推荐表** · 通用层 7 大能力的主流方案
4. 4. **7 个私域组件的 MVP** · 每个配起步选型 + 演进路线
5. 5. **一个决策矩阵 + 10 个反模式** · 避免烧错钱

* • **如果你只读本系列一篇**：就读这篇。

---

## 一、Claude 4.7 发布的那个下午

2026 年 4 月，Anthropic 发布了 Claude 4.7。随模型一起释出的还有一个**升级版 Harness** —— Claude Code Skills 2.0、原生 context compaction、内建 memory、开箱即用的 sandbox。

三层架构里，**过去每家都要自搭的中间那一整层运行时，现在 Anthropic 默认给你** —— 一行代码不用写，跟着模型版本自动升级。这就是文章第五节要讲的"通用层 7 大能力"。

这意味着一件事：

> **你的 AI 平台团队，昨天还在自研的那些能力，今天就成了沉没成本。**

而且不只是 Anthropic。OpenAI Agents SDK、LangGraph、Cursor 同时在做同样的事。**整个行业正在经历快速的 commodity 化 —— 通用 Harness 正在被 SDK 厂商"吃掉"**。

但这不意味着 AI 平台团队没事干了。相反 —— 真正的机会才刚露头。

**问题是**：你怎么知道哪些能力该放手让 SDK 做，哪些能力必须自建、且永远不会被 SDK 吃掉？

答案是一条简单的判定法 —— **12 个月测试**。但要讲透它，先得搞清 Harness 正在经历的分化。

## 二、第一性原理：Harness 为什么会分化

两条曲线说明了一切：

* • **模型能力在收敛** · Claude 4.7 / GPT-5 / Gemini 2.5 Pro 的 benchmark 差距 ≤ 5pp，还在继续收窄
* • **Harness 能力在分化** · 同样的模型，裸 API 调用 vs 成熟 harness 能拉开 30pp 以上

但 "Harness 分化" 本身**又在二次分化**：

| 分化方向 | 经济学 | 结果 |
| --- | --- | --- |
| **通用能力 + 厂商规模经济** | 赢家通吃 | 一家做好，全世界用（OpenAI / Anthropic / LangChain 在做的事） |
| **业务耦合 + 私有知识** | 每家不同 | 永远不会有通用版本（只有你的团队知道） |

Claude 4.7 内建 Harness，是第一条路径的加速；而你的护城河，藏在第二条路径里。

## 三、判定法：12 个月测试

遇到任何"要不要自建"的决策，问一个问题：

> **"这个能力 12 个月后，OpenAI Agents SDK / Claude / LangGraph / Cursor 会不会做成默认实现？"**

| 答案 | 策略 | 为什么 |
| --- | --- | --- |
| ✅ **会** | 不要自建 | 跟全世界最强的团队卷，他们更多资源 / 数据 / 用户反馈。你的自建版本会在下次 SDK 升级时变成沉没成本 |
| ❌ **不会** | 必须自建 | 只有你知道你们公司的业务、资损、故障、玩法、代码、运维约束。SDK 永远吃不掉 |

### 通用层的 commodity 化进程

**过去 18 个月，通用 Harness 商品化已完成 62%，剩余 38% 在 2026 年内**：

| 时间 | 里程碑 | 影响 |
| --- | --- | --- |
| 2025 Q1 | LangGraph GA | orchestration 层标准化 |
| 2025 Q2 | OpenAI Agents SDK 发布 | 替代 Assistants API，Responses 成形 |
| 2025 Q3 | Claude Code Skills | skill 发现机制进入通用层 |
| 2025 Q4 | OTel GenAI v1.37 稳定 | 埋点标准 · APM 原生支持 |
| **2026 Q1 ⭐ 现在** | **Claude 4.7 内建 Harness** | **memory / sandbox / compaction 一等公民** |
| 2026 Q2 | 预计 MCP v1.0 稳定 | 工具互操作成事实标准 |
| 2026 Q3-Q4 | 通用 Harness commodity 完成 | 自建 = 重造轮子 |
| 2027+ | 差异化只剩私域层 | skill + invariant + MCP |

**现在还在启动自建通用层的团队，几乎一定会在 2027 年之前被迫迁移**。

## 四、边界图：冰山之下才是护城河

这张图是**全文最值得截图保存**的一张：

* • **冰山尖 · 水面以上 ~20%** · 通用 Harness —— 你看得见，但 SDK 会吃掉
* • **冰山体 · 水面以下 ~80%** · 私域 Harness —— 支撑你的、必须自建的

## 五、通用层 · 7 大能力的选型推荐

这 7 项过去 12 个月里**已经或即将被 SDK 吃掉**。不要自建，只做选型 + 少量适配。下表给你直接可用的选型建议：

| 能力 | 首选方案 | 备选 | 成本模型 | 自建代价 | 建议 |
| --- | --- | --- | --- | --- | --- |
| **Context 压缩** | Claude Code 原生 · OpenAI Agents SDK | LangGraph 自带 | 按模型计费 | 算法快速演进，12 个月过时 | 直接跟随 SDK 版本 |
| **基础代码沙箱** | E2B · Daytona | Modal · Firecracker · 自建 gVisor | $0.01-0.05 / 运行 | K8s + gVisor 自建成本巨大，且安全性不如专业厂商 | 选托管 SaaS，少量场景自部署 |
| **Memory（跨会话）** | Letta · Mem0 | OpenAI Memory API · Zep | 开源免费 / SaaS 订阅 | 记忆层算法每季度更新 | 买 SaaS 或 OSS 自部署 |
| **Tool dispatch / Orchestration** | LangGraph（开源） | OpenAI Agents SDK · Autogen | 开源 · 自部署 | 重造轮子，且失去生态 | LangGraph 做基座 |
| **OTel GenAI 埋点** | OpenTelemetry v1.37+ | Langfuse · Arize | 开源 + APM 费用 | 私有埋点方案注定被标准淘汰 | 直接上 OTel，选 Datadog/Grafana 消费 |
| **Skill / Subagent 加载** | Claude Code Skills | OpenAI Custom GPT | 跟随模型计费 | 加载机制易变，难追上上游 | 跟随事实标准 |
| **基础 Eval Harness** | OpenAI Evals · Ragas | Braintrust · Langfuse Eval | 开源 / SaaS 订阅 | runner 本身不是差异化 | 选托管或 OSS 起步 |

> **判定窍门**：如果这个东西在 Crunchbase 上已经有 10 家公司在卷，你自建就没机会。

### 反直觉提醒

很多团队 CTO 会把这些能力包装成 "我们自研的 XX 平台"。12 个月后 SDK 出齐，这些自研系统会以"为了不影响现有业务"的名义长期维护下去 —— **变成最沉重的技术债**。

## 六、私域层 · 7 个必须自建的组件

这 7 项**永远不会被商品化** —— 因为它们全部来自企业内部。每个组件下面给出**定义 + 为什么不会被吃掉 + MVP 选型**。

### 6.1 业务 Invariant 引擎 · 最硬核

| 维度 | 内容 |
| --- | --- |
| **定义** | 把企业业务的"不变量"翻译成可被 agent 运行时断言的规则引擎 |
| **为什么不会被吃掉** | 资金守恒在支付场景是 invariant，在广告扣量场景完全不同 —— SDK 不知道你的不变量 |
| **MVP（1Q 起步）** | 10-20 条核心 invariant（从过去 3 年资损事件反推）· Python / CEL 表达式引擎 · 接入 trace 作为数据源 · agent 产出前强制断言 |
| **演进（4Q 后）** | v1 规则 DSL 化 + 异步断言 · v2 从 post-mortem 自动抽取候选规则 |
| **不要做** | 不要直接接入 LLM 做"智能判断" · invariant 必须是确定性规则 |

### 6.2 企业 MCP 网关 · 身份映射是灵魂

| 维度 | 内容 |
| --- | --- |
| **定义** | 企业所有工具、数据、配置、日志统一走 MCP 协议，网关层做身份映射 + 权限下钻 + 审计 + PII 脱敏 |
| **为什么不会被吃掉** | 身份映射是企业独有 —— agent 借用"调用它的工程师"的权限，每家内部契约都不同 |
| **MVP（1Q 起步）** | 基于 Kong / Spring Cloud Gateway 套一层 MCP 协议适配 · 初期只读 · 身份从 SSO 取 · 审计复用 ELK |
| **关键原则** | ① Agent 能力边界由网关定义，不由 agent 自觉 ② 敏感字段脱敏必须在网关层 ③ 所有调用必须带 `trace_id` + `caller_user_id` ④ 写操作走 "生成 diff → 人工审批 → 网关 apply" |
| **不要做** | 不要让 agent 直接握有服务账号 · 不要搞平行于既有 RBAC 的权限系统 |

### 6.3 链路集成沙箱 · 业界还没标准

| 维度 | 内容 |
| --- | --- |
| **定义** | 不是单服务沙箱（E2B 已搞定），而是多服务 + 依赖基础设施的沙箱 |
| **为什么不会被吃掉** | 依赖企业自己的微服务拓扑、数据分布、消息契约。SDK 厂商不知道你有多少 Kafka topic |
| **MVP · 三条路径按规模选** | **中型团队** · 按需拉起子链路（K8s namespace + 3-10 个相关服务 · 其他 mock） · **大厂** · 流量染色 + 复用 pre · **超大厂** · 全量影子环境 |
| **硬核实战** | 详见 **06-资金链路沙箱的最佳实践** |

### 6.4 四大私有资产库 · 真正的护城河

| 资产 | 定义 | MVP 起点 |
| --- | --- | --- |
| **玩法模板库** | 营销/活动代码骨架 + invariant 清单 + 强制项 | 资深 leader 手工沉淀 20-30 个模板 |
| **红线规则库** | 资损教训 + 改动对比规则 | 资深架构师写"我 CR 时最关心的 30 件事" |
| **历史故障库** | P0/P1 post-mortem 结构化档案 | 从今天起每次故障强制结构化 schema |
| **配置语义标签** | 关键配置项的语义 + 风险等级 | 选 100-200 个关键配置人工打标 |

**为什么不会被吃掉**：这四项**全部来自企业的历史事件、资深工程师的脑子、过去 3-5 年的教训**。Cursor / Copilot / Devin 拿不到这些数据。

### 6.5 代码结构理解三层图

| 维度 | 内容 |
| --- | --- |
| **定义** | AST 符号图（精确调用关系）+ 依赖图（编译期）+ 运行时图（流量 + 接口调用） |
| **为什么不会被吃掉** | 企业代码是独有的 —— 把 AST × 依赖 × 运行时 × 业务语义叠在一起必须企业自建这层 fusion |
| **MVP 选型** | Java · Eclipse JDT · Go · gopls · 统一灌入 Neo4j · 依赖层解析 maven/gradle/proto · 运行时层从 SkyWalking / Zipkin 采样提取 |
| **关键判断** | 不要只做向量化 —— 向量检索查不准精确调用关系 / 不知道编译期依赖 / 看不到运行时流量 |

### 6.6 Skill / Subagent 注册表 · 内容非机制

**关键区分**：Skill 的**加载机制**属于通用层（SDK 会吃）；**注册表的内容**属于私域层，必须企业自建。

| 维度 | 内容 |
| --- | --- |
| **定义** | 企业私域所有 skill / subagent 的清单 + 版本 + 权限 + 适用场景 |
| **MVP 选型** | Git 仓 · 每个 skill 一个目录 · `metadata.yaml` + prompt + 示例 · metadata 含：`id / version / required_tools / safety_tier / owner / eval_suite` |
| **CI 集成** | skill 更新触发其 golden dataset 的 eval · 不通过不能合并 |

### 6.7 Agent I/O Envelope 契约

| 维度 | 内容 |
| --- | --- |
| **定义** | 跨 agent / 跨团队 / 跨系统流通的标准信封 —— Task / Context / Result 三件套 |
| **为什么不会被吃掉** | 业界会出标准（类似 Open Agent Schema），但**字段集合和业务风险等级的映射**必须企业自己定 · `risk_tier` 在 C 端娱乐和金融严监管完全不同 |
| **MVP 选型** | JSON Schema 写 3 个 envelope 定义 · 所有 agent 入口强制校验 Task · 出口强制校验 Result · 接入 L2 trace 和 L0 审计 |

## 七、决策矩阵 · 自建、买、还是混合

**决策口诀**：气泡越靠右下越不能买，70% 投入砸在 **Q4**。

## 八、10 个反模式 · 让你烧错钱的典型场景

挨个打勾，命中 3 条以上就要警觉：

| # | 反模式 | 为什么错 | 替代方案 |
| --- | --- | --- | --- |
| 1 | 自研 Agent 平台 | 多半在重造 LangGraph / OpenAI Agents SDK | 基于 LangGraph 或 Agents SDK 加私域扩展 |
| 2 | 自研向量数据库 | pgvector / Qdrant 成熟得可怕 | 直接用，自建 = 浪费 |
| 3 | 自研 context 压缩算法 | 研究机构在做，追不上 | 跟随 SDK |
| 4 | 自研 code execution sandbox | E2B / Daytona 几十万用户打磨过 | 选托管 |
| 5 | 自研 LLM 网关（限流/计费/路由） | LiteLLM / Portkey 早就 commodity | 选开源部署 |
| 6 | 自研 prompt 版本管理系统 | 纯配置管理，复用 Git 就够 | Git + 简单 UI |
| 7 | 自研 trace 采集 SDK | OTel 已经接管 | 直接上 OTel |
| 8 | 自研 eval runner | 只有 eval criteria（业务知识）是私域 | runner 用开源，criteria 自写 |
| 9 | 自研 agent 观察面板 | Langfuse / Braintrust / Arize 成熟 | 选一家托管 |
| 10 | 自研 MCP 协议替代品 | 2025 年试过的都关了 | 直接用 MCP |

## 九、投入怎么分 · 70% 私域 · 30% 场景

**整体分布**：私域 Harness 70% · 场景 / UI / 业务接入 30%。

### 私域层内部分配 · 70% 投入怎么分 + Q1 里程碑

1 格 `█` = 5%，一眼看出钱砸在哪：

| 组件 | 占比 | 分布 | Q1 起步里程碑 |
| --- | --- | --- | --- |
| MCP 网关 + 身份映射 | 15% | `███` | 只读工具 100% 覆盖 |
| 代码理解三层图 | 15% | `███` | 覆盖核心 10 个 repo |
| 链路集成沙箱 | 15% | `███` | 核心链路 3-5 条跑通 |
| 四大资产库 | 15% | `███` | 每库 v1 落地 50-200 条 entry |
| Invariant 引擎 | 5% | `█` | 20 条核心规则 · CI 强制 |
| Skill 注册 + Envelope | 5% | `█` | JSON Schema + 5 个 skill |

## 十、带走这一句

> **企业 AI Native 能力 = 通用 Harness（选型）+ 私域 Harness（自建）。**
>
> Claude 4.7 这类 SDK 级 harness 的每一次更新，都是通用层在加速 commodity 化 —— 跟随就好。
>
> 真正的护城河只有一侧 —— 冰山之下的**私域 Harness**。
>
> **70% 的投入砸在那里，100% 的差异化从那里来。**

## 继续读

* • 上一篇 · **01-AI Native 技术基建的核心**
* • 下一篇 · **02-Harness Engineering 深度拆解**（学科入门）
* • 架构映射 · **03-六层技术体系架构**
* • 场景落地 · **04-研发全生命周期的 Agent 落地**
* • 硬核实战 · **06-资金链路沙箱的最佳实践**