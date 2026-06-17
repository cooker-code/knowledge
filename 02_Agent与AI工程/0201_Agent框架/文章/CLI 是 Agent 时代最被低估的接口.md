---
title: CLI 是 Agent 时代最被低估的接口
author: 硅基旅人
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg4MjcwNDkxOQ==&mid=2247485497&idx=1&sn=01e597872b8c2fb506d9c8b9ca36c7f4&chksm=ce712a983b462205593c50884d4580fddac34c556cca49aa7619cac63d6c64fd7e64c415215d&mpshare=1&scene=24&srcid=0227THUDWrjQHI4dUwmqWTLh&sharer_shareinfo=9aab7c443609760cd05fec5d4efee31b&sharer_shareinfo_first=9aab7c443609760cd05fec5d4efee31b#rd
---

**导读：** Karpathy 昨天发了一条推文，核心观点只有一句话：CLI 是 AI Agent 时代最被低估的接口。他用 Polymarket 刚开源的 Rust CLI 做了个演示——让 Claude 3 分钟搭了一个终端预测市场仪表盘。这件事背后的逻辑，比看上去要深得多。

## 一个 3 分钟的演示

Polymarket 是全球最大的去中心化预测市场，你可以在上面用真金白银押注各种事件的走向——从美国大选到比特币价格，从 AI 监管到体育赛事。

2 月 24 日，Polymarket 开发者 Suhail Kakar 宣布开源了 polymarket-cli，一个用 Rust 写的命令行工具。不需要钱包就能浏览市场数据，装上就能用：

```
# 不需要钱包，装完直接查
polymarket markets list --limit 5
polymarket markets search "election"

# JSON 输出，方便脚本处理
polymarket -o json markets list --limit 3
```

几个小时后，Karpathy 转发了这条推文，配了一张截图：他让 Claude 用这个 CLI 搭了一个终端仪表盘，展示交易量最高的预测市场和 24 小时价格变化。整个过程大约 3 分钟。

3 分钟，从零到一个可用的数据仪表盘。不是因为 Karpathy 写代码快，而是因为他根本没写代码——他只是告诉 AI Agent「装这个 CLI，然后给我做个仪表盘」。

## 为什么 CLI 是 Agent 的最佳接口

Karpathy 的原话是：CLI 之所以让人兴奋，恰恰因为它是一种「遗留」技术。

这话听着反直觉，但仔细想想完全说得通。CLI 的特点是什么？

* 纯文本输入输出，没有 GUI 的视觉复杂度
* 标准化的参数格式（`--flag value`），机器解析零成本
* 可以通过管道（pipe）组合，`A | B | C` 就是一条数据流水线
* 有 `--help` 和 man page，Agent 自己就能读懂怎么用

这些特点在人类用户看来可能是「不够友好」，但对 AI Agent 来说，这就是天然的 API。不需要点击按钮，不需要解析 HTML，不需要处理 JavaScript 渲染——直接读文本、写文本、组合工具。

对比一下其他接口方式：

| 接口类型 | Agent 友好度 | 上手成本 | 组合能力 |
| --- | --- | --- | --- |
| CLI | 极高 | 装完即用 | 管道组合，天然支持 |
| REST API | 高 | 需要读文档、处理认证 | 需要写胶水代码 |
| MCP | 高 | 需要配置 server | 协议层面支持 |
| Web UI | 低 | 需要浏览器自动化 | 几乎不可组合 |

CLI 的优势在于：它已经存在了几十年，生态成熟到不行。`git`、`curl`、`jq`、`ffmpeg`、`gh`（GitHub CLI）……这些工具本来就是给人用的，但 Agent 拿过来直接就能用，不需要任何适配。

## Polymarket CLI 到底能干什么

说回 Polymarket 这个 CLI 本身。它不只是一个查价格的工具，功能覆盖了 Polymarket 平台的几乎所有操作：

**不需要钱包就能用的：**

* 浏览和搜索市场（按交易量、标签、状态筛选）
* 查看订单簿、价格、价差
* 查看价格历史（支持 1 分钟到全量的时间粒度）
* 查看排行榜、持仓分布、交易活动

**需要钱包的：**

* 下单（限价单、市价单、批量下单）
* 管理订单（取消、查看状态）
* 链上操作（拆分/合并/赎回条件代币）
* 跨链充值

关键是每个命令都支持 `--output json`，这意味着任何脚本或 Agent 都能直接解析输出结果，不用去 scrape 表格格式的文本。

Rust 写的，性能不用担心。MIT 协议开源，想怎么改怎么改。

## 不只是 CLI：Build. For. Agents.

Karpathy 推文最后那句话才是重点：

> It's 2026. Build. For. Agents.

他给出了一个产品/服务的 Agent 友好度自检清单：

* 你的文档能导出 Markdown 吗？
* 你为产品写过 Skills 吗？
* 你的产品能通过 CLI 使用吗？或者 MCP？

这不是在说 CLI 是唯一答案。CLI、MCP、REST API，这些都是 Agent 可以使用的接口。Karpathy 的意思是：如果你做产品，你得开始认真想「AI Agent 怎么用我的东西」这个问题了。

💡 PROMPT

提示词参考：  
Install the polymarket CLI. Then build me a terminal dashboard showing the top 10 highest volume markets with their 24h price change.

这条 prompt 就是 Karpathy 演示中大概率用的指令。Agent 收到后会自己去装 CLI、读 `--help`、写脚本、调试运行。整个过程人类只需要描述想要什么。

## 这对开发者意味着什么

如果你在做一个 SaaS 产品、一个开源工具、甚至一个内部系统，现在就该想想：

**最低成本的 Agent 适配方案就是 CLI。** 你不需要从零搭一个 MCP server，不需要设计复杂的 API 认证流程。写一个 CLI，支持 JSON 输出，加上清晰的 `--help`，Agent 就能用了。

Polymarket 这个案例特别有说服力，因为预测市场的数据天然适合被 Agent 消费——实时价格、历史趋势、市场情绪，这些都是 Agent 做决策时需要的输入。但同样的逻辑适用于任何领域：

* 监控工具出个 CLI → Agent 可以自动巡检
* 数据库出个 CLI → Agent 可以自动查询和分析
* 部署工具出个 CLI → Agent 可以自动化运维

Karpathy 这条推文的信息量其实很大。表面上是在安利一个预测市场的 CLI 工具，实际上是在说：2026 年了，产品的用户不再只是人类。  
  
CLI 不是什么新技术，但它可能是 Agent 时代最实用的「旧」接口。不需要花哨的协议，不需要复杂的集成——一个命令行工具，加上 JSON 输出，就够了。  
  
Build. For. Agents. 这不是口号，是产品策略。

## 参考链接

* Polymarket CLI GitHub 仓库：https://github.com/Polymarket/polymarket-cli
* Polymarket 官网：https://polymarket.com

---

— **完** —