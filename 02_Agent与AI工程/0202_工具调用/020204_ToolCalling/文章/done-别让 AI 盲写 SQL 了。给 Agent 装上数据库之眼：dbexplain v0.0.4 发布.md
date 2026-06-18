> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020204_ToolCalling/020204_核心知识点/工具调用治理与协议边界|工具调用治理与协议边界]]
---
title: 别让 AI 盲写 SQL 了。给 Agent 装上数据库之眼：dbexplain v0.0.4 发布
author: 在场札记
date: 交易菜鸟训练交易菜鸟训练
url: https://mp.weixin.qq.com/s?__biz=Mzk0NTE5NjI4Nw==&mid=2247483981&idx=1&sn=834a689f8da99101c8a11a8a45b46272&chksm=c27af0244a5861f047d3bc54c536e73d53e19a0d1bacb593bb7aad963e8f8d99d962f0a031cb&mpshare=1&scene=24&srcid=0526t1dvPSRtRmrYqFTgWt6B&sharer_shareinfo=3350b3fed18fc8f5ae345d2d148cc97c&sharer_shareinfo_first=3350b3fed18fc8f5ae345d2d148cc97c#rd
---

```
别让 AI 盲写 SQL 了。给 Agent 装上数据库之眼：dbexplain v0.0.4 发布
```

## 一句话：这工具干什么的

你用 Claude Code、Cursor 写 SQL，Agent 不知道你的表结构——它要么盲猜，要么你手动把 schema 贴进 prompt。

**dbexplain** 就是解决这个问题的。一个 40MB 的零依赖二进制文件，给它数据库连接串，5 秒拉出全部表结构、索引、外键关系、风险诊断，输出结构化数据直接喂给 AI Agent。

```
$ ./dbexplain -env -include mysql
> Instances (1)  aiops-mysql                     mysql  1 db(s), 2 tables
> aiops-mysql / testdb  iplist [InnoDB] ~12 rows 16.0KB     核心设备清单表  port   [InnoDB] ~30 rows 16.0KB     端口服务表
> Issues (2)  [!] iplist  no timestamp column -- audit trail gap  [!] port    no timestamp column -- audit trail gap
```

不做 AI 推理，只输出可证实的确定性事实。支持 MySQL / PostgreSQL / GaussDB / ClickHouse / SQLite / Redis / MongoDB / Elasticsearch / Qdrant — 9 种异构数据库，一个二进制全部搞定。

---

## 为什么 AI 编程时代需要它

2026 年，用 AI 写代码已经是常态。但让 AI 操作数据库，始终有个问题没解决：

* **Agent 不知道你的库里有啥。** 你把 DSN 给它，它不知道有哪些表、什么字段、索引在哪。
* **手动贴 DDL 太占上下文窗口。** 一个中等项目的 schema 轻易上万 token，留给业务逻辑的空间就少了。
* **Agent 写的 SQL 可能不走索引。** 它不知道 `user_id` 上有索引还是没索引，全表扫描和索引查询的写法完全不同。

dbexplain 做的事很简单：**在 AI Agent 和数据库之间，充当一层确定性的 Schema 真值层。**

```
你的 9 种数据库 → dbexplain（5s）→ 结构化 Schema 文件 → AI Agent 精准决策
```

不分析你的数据内容，不执行 Agent 生成的 SQL。只采元数据。**它不会把你的数据喂给 LLM，也不会执行 Agent 脑补的危险 SQL。** 跟 MCP Server 配合使用效果更佳——MCP 负责协议层，dbexplain 负责提供准确的 Schema 上下文。

---

## v0.0.4 重磅更新：AI Agent 原生化

### 核心变革：IR v1 通用图模型

之前的版本每种数据库各自为政。v0.0.4 引入了 **IR (Intermediate Representation) 图模型**——Node（表）、Column（列）、Edge（关系）——所有数据库类型统一表达。新增数据库只需声明能力，算法自动适配。这是从"工具"到"编译器"的架构跃迁。

### AI Agent 三件套

v0.0.4 为 AI Agent 量身打造了三层输出：

| 输出层 | 文件 | 用途 |
| --- | --- | --- |
| **全局摘要** | `summary.json` | 实例列表、表排行（含 importance\_score）、操作统计 |
| **关系拓扑** | `topology.json` | 跨库外键图、表聚类、数据血缘 |
| **问题诊断** | `diagnostics.json` | MissingPK、NoTTL、无索引外键等安全隐患 |
| **检索分块** | `chunks/*.md` | 每表独立 Markdown，适合 RAG 向量化检索 |

一键生成：`./dbexplain -env --context ./ai-context`

### 重要性排序：让 Agent 知道先看什么

多因子加权评分（图度 x0.35 + 外键中心性 x0.35 + 行数 x0.20 + 索引密度 x0.10），自动识别核心表。Agent 拿到 context 后，注意力天然聚焦在最重要的表上——**不用在 200 张配置表里大海捞针。**

### Schema 指纹：数据库的 "Git Diff"

给数据库 schema 拍快照，下次跑就能发现谁改了什么。

**实战三步走：**

```
# 第一步：生成基准快照（比如上线前一天跑一次）./dbexplain -env --cache schema_cache.json# → 生成 schema_cache.json，包含每张表的列/索引/外键的 SHA-256 指纹
# 第二步：日常巡检（配合 cron，每天凌晨跑一次）./dbexplain -env --cache schema_cache.json --context ./ctx# → 如果 schema 没变：无额外输出，schema_cache.json 维持不变# → 如果 schema 变了：输出 schema_cache_delta.json，精确到：#      "added":   ["业务库.orders.new_field"]    ← 谁加了字段#      "removed": ["业务库.users.old_index"]      ← 谁删了索引#      "changed": ["业务库.products"]             ← 哪张表结构变了
# 第三步：发现变更后，diff 对比cat schema_cache_delta.json  # 一目了然
```

**真实场景**：有人在 `orders` 表偷偷加了个 `priority` 字段没通知你，delta 里直接显示 `"added": ["shop-db/mydb.orders"]`。去线上查一看，还真是。配合 cron，每周甚至每天自动巡检。

这个功能很轻——只做 SHA-256 哈希对比，不存任何实际数据。缓存文件就是一个 JSON，可以直接猫看。

### 操作统计：零配置的"慢查询替代"

从 MySQL/PostgreSQL 系统表自动采集每表查询频率和写入强度（`query_frequency` / `write_intensity`）。不需要额外的监控 Agent，不需要 PL/SQL 脚本——工具自己搞定。不可用时自动降级，不影响核心采集。

---

## 运维体验升级

### 输出不再被"噪音"污染

```
# 之前：满屏 skipping 日志$ ./dbexplain -env -include mysql
skipping clickhouse://... (did not match include filter) ← 噪音skipping redis://... (did not match include filter)    ← 噪音[采集中] aiops-mysql[完成] aiops-mysql (2 表)
# 现在：干干净净，跳过记录静默落入 logs/filter.log
```

### `--manual` 完整中文手册

600+ 行按数据库分类的详细文档，支持 `--language en` 中英切换，`--filter` 关键字查找。

```
./dbexplain --manual --filter "Redis 集群"
```

### `-h` 帮助信息重构

不再是字母序参数罗列。按**功能场景**分为 7 组：数据源 / 过滤 / 输出控制 / 显示格式 / AI 上下文 / 性能 / 帮助。中英双语。

### Windows 支持

`-o` 文件输出会检测系统代码页：中文 Windows 自动输出 GBK 编码（CMD `type` 和记事本都正确），英文/其他语言 Windows 输出 UTF-8 BOM（现代编辑器通用）。Unicode 制表符全部替换为 ASCII，PowerShell 里也能正常渲染。不会再看到 `锘?` 和乱码。

## 安全第一

* **全量只读**：MySQL/PostgreSQL 仅 `SELECT`/`SHOW`/`PRAGMA`，Redis 仅 `SCAN`，绝不做写操作
* **密码脱敏**：所有输出和日志中密码替换为 `***`
* **连接隔离**：每 DSN 独立日志（`logs/<label>.log`），故障不互相影响
* **采样上限**：Redis 最多扫描 2000 key、5 字段、512 字节，不会打崩生产环境
* **敏感文件不暴露**：算法细节文档不上传远程仓库

---

## 5 秒上手

```
# 1. 下载wget https://github.com/IamWWT/understand_dbs_skills/releases/download/v0.0.4/dbexplain-linux-amd64chmod +x dbexplain-linux-amd64
# 2. 配置 .envcat > .env << 'EOF'DB1=mysql://root:pass@127.0.0.1:3306/mydb?label=业务库DB2=postgres://user:pass@127.0.0.1:5432/mydb?label=分析库&sslmode=disableDB3=redis://:pass@10.0.0.1:7000/0?cluster=true&label=缓存集群EOF
# 3. 运行./dbexplain-linux-amd64 -env              # 终端看报告./dbexplain-linux-amd64 -env --human      # 人类友好模式./dbexplain-linux-amd64 -env --context ./ctx  # 生成 AI 上下文./dbexplain-linux-amd64 -env --cache fingerprint.json  # 增量监控
```

##

---

## 性能

v0.0.4 vs v0.0.3 对标测试（9 异构数据源）：

| 指标 | v0.0.3 | v0.0.4 | 结论 |
| --- | --- | --- | --- |
| 总耗时 | ~32ms | ~40ms | 网络噪声内，无退化 |
| JSON 大小 | 135,524 B | 135,527 B | +3B（仅 BOM） |

新增的 IR 转换、重要性评分、操作统计、指纹计算等代码路径开销 <5ms，完全被网络 I/O 掩盖。

---

## 项目信息

* **GitHub**: https://github.com/IamWWT/understand\_dbs\_skills
* **Release**: https://github.com/IamWWT/understand\_dbs\_skills/releases/tag/v0.0.4
* **平台**: Linux (amd64/arm64) / macOS (amd64/arm64) / Windows (amd64)
* **License**: Apache 2.0
* **语言**: Go 1.26+，纯标准库，零第三方算法依赖

零依赖、只读安全、AI Agent 原生。欢迎 Star，欢迎集成到你的 AI 工作流中。

===历史发布===

[1] v0.0.3: [一个二进制文件搞定 9 种数据库：dbexplain v0.0.3 发布](https://mp.weixin.qq.com/s?__biz=Mzk0NTE5NjI4Nw==&mid=2247483974&idx=1&sn=d0d7a3665bcb531cd7538aa908eb4973&scene=21#wechat_redirect)

[2] v0.0.2: [DevOps 必入：只读安全的数据库“X 光机”，一键生成关系图谱与健康报告](https://mp.weixin.qq.com/s?__biz=Mzk0NTE5NjI4Nw==&mid=2247483941&idx=1&sn=ca78b524f8be55a2730420975ff16ca9&scene=21#wechat_redirect)