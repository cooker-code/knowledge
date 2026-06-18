---
title: Context Hub：为 AI 编程助手打造的文档基础设施
author: 老肖说两句
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA4ODgwNTk1NQ==&mid=2649952287&idx=1&sn=80019f81439ad545619b9343dc9c6554&chksm=89b481773ed03269dbf60ec0168f75ce388e231d222fb7959f7d6ac02778895ff815144feca8&mpshare=1&scene=24&srcid=0318fgeG9HRboLKV73jI6WiX&sharer_shareinfo=de467ad5c614b62e75bfa1e3201cfd1c&sharer_shareinfo_first=de467ad5c614b62e75bfa1e3201cfd1c#rd
---

# Context Hub：为 AI 编程助手打造的文档基础设施

> 一个开源项目，解决了 LLM 在代码生成中最常见的痛点——过时的 API 知识。

---

## 背景：LLM 的知识截止问题

大语言模型的训练数据有截止日期。当你让 Claude 或 GPT 帮你写 Stripe 支付代码时，它给出的 API 调用方式可能已经在三个版本前被废弃了。这不是模型的错，而是架构上的固有局限。

Context Hub（`@aisuite/chub`）是 Andrew Ng 团队开源的一个工具，思路很直接：**与其让模型猜，不如在运行时把正确的文档喂给它**。

---

## 整体架构

项目分为三层：

```
```
content/          ← 600+ 个库的 LLM 优化文档（Markdown）  
cli/src/lib/      ← 核心逻辑：注册表、缓存、搜索、标注  
cli/src/mcp/      ← MCP 服务器，暴露给 AI Agent 调用
```
```

用户侧有两个入口：

* chub — 给人用的命令行工具
* chub-mcp — 给 AI Agent 用的 MCP 服务器

---

## 内容格式：为 LLM 优化的 Markdown

`content/` 目录下每个库都遵循固定的路径约定：

```
```
content/{package}/docs/{category}/{language}/DOC.md  
content/{package}/docs/{category}/{language}/references/*.md
```
```

每个 `DOC.md` 文件头部是 YAML frontmatter，携带结构化元数据：

```
```
---  
name: package  
description: "aiohttp package guide for Python - async HTTP client/server"  
metadata:  
  languages: "python"  
  versions: "3.13.3"  
  revision: 1  
  updated-on: "2026-03-11"  
  source: maintainer  
  tags: "aiohttp,python,asyncio,http,web,websocket"  
---
```
```

文档正文不是普通的 API 参考，而是面向 LLM 消费设计的：有"黄金法则"（Golden Rule）、最小可运行示例、常见陷阱，以及明确的"不要这样做"警告。这种格式让模型能快速提取关键约束，而不是在大段文字里找答案。

---

## 注册表与构建流程

`chub build` 命令扫描 `content/` 目录，调用 `discoverAuthor()` 遍历每个包，解析 frontmatter，生成两个产物：

1. registry.json — 所有条目的元数据索引
2. search-index.json — 预构建的 BM25 搜索索引

构建时还会计算每个文档目录的大小，决定是否需要打包成 tar 归档分发。

---

## 搜索引擎：BM25 的工程实现

项目没有用向量数据库，而是自己实现了 BM25 全文检索（`cli/src/lib/bm25.js`）。

BM25 的核心公式对每个字段独立打分，再加权求和：

```
```
score(q, d) = Σ IDF(t) × (tf × (k1+1)) / (tf + k1 × (1 - b + b × dl/avgdl))
```
```

实现上有几个值得注意的设计决策：

**字段权重差异化**：名称、标签、描述三个字段的权重分别是 3.0、2.0、1.0。搜索 "stripe" 时，名称匹配的权重是描述匹配的三倍，符合用户意图。

```
```
const FIELD_WEIGHTS = {  
  name: 3.0,  
  tags: 2.0,  
  description: 1.0,  
};
```
```

**索引在构建期预计算，搜索期只做评分**：IDF（逆文档频率）在 `chub build` 时就算好存入索引，运行时搜索只需要遍历文档做 TF 计算，避免了每次搜索都重建索引的开销。

**分词器在构建和搜索两端保持一致**：这是 BM25 实现中容易踩的坑。`tokenize()` 函数被 build 和 search 两个路径共享，确保索引词和查询词经过相同的处理（小写、去停用词、按空格和连字符分割）。

---

## 缓存层：多源注册表管理

`cache.js` 管理本地缓存，支持多个内容源（官方 CDN + 私有内部文档）。

```
```
# ~/.chub/config.yaml  
sources:  
  - name: community  
    url: https://cdn.aichub.org/v1  
  - name: internal  
    path: /path/to/local/docs
```
```

缓存有新鲜度检查（`isSourceCacheFresh()`），避免每次都重新下载。当 ID 在多个源中冲突时，用 `source:id` 前缀消歧，例如 `chub get internal:openai/chat`。

文档内容按需拉取（`fetchDoc()`），也支持一次性拉取完整包（`fetchDocFull()`），后者用于需要 `references/` 子目录的场景。

---

## 标注系统：跨会话的知识积累

`annotations.js` 实现了一个轻量的本地持久化层，让 Agent 能把使用文档时发现的关键信息保存下来：

```
```
chub annotate stripe/api "Webhook 验签必须用原始 body，不能用解析后的 JSON"
```
```

实现上非常简单：每条标注是 `~/.chub/annotations/{id}.json`，ID 中的 `/` 替换为 `--` 作为文件名。

```
```
function annotationPath(entryId) {  
  const safe = entryId.replace(/\//g, '--');  
  return join(getAnnotationsDir(), `${safe}.json`);  
}
```
```

这个设计的价值在于：Agent 下次调用 `chub_get` 时，标注会随文档一起返回，相当于给模型注入了上一次使用时积累的经验。

---

## MCP 服务器：与 AI Agent 的集成接口

`chub-mcp` 基于 `@modelcontextprotocol/sdk` 实现，暴露 5 个工具：

| 工具 | 用途 |
| --- | --- |
| `chub_search` | 按关键词/标签/语言搜索 |
| `chub_get` | 按 ID 获取文档内容 |
| `chub_list` | 列出所有可用条目 |
| `chub_annotate` | 读写本地标注 |
| `chub_feedback` | 向作者发送质量反馈 |

有一个细节处理得很干净：MCP 服务器通过 stdio 传输 JSON-RPC，任何写到 stdout 的内容都会破坏协议。项目把所有 `console.log/warn/info/debug` 重定向到 stderr：

```
```
const _stderr = process.stderr;  
console.log = (...args) => _stderr.write(args.join(' ') + '\n');
```
```

这样即使依赖库（比如 posthog-node）偷偷调用 `console.log`，也不会污染 MCP 通信流。

---

## Agent Skill：教会 Agent 主动查文档

项目还附带了一个 `skills/get-api-docs/SKILL.md`，这是一段给 Agent 的元指令，告诉它"当需要使用某个库时，先用 `chub_search` 查一下有没有对应文档，再用 `chub_get` 获取内容，而不是依赖训练数据"。

把这个文件复制到 Claude Code 或 Cursor 的规则目录，Agent 就会在编码时自动触发文档查询流程。

---

## 小结

Context Hub 的设计哲学是：**不改变 LLM，改变 LLM 拿到的上下文**。

技术选型上没有过度工程化——BM25 而非向量检索，本地 JSON 文件而非数据库，Markdown 而非专有格式。每个选择都指向同一个目标：让正确的文档在正确的时机出现在模型的上下文窗口里。

对于维护内部 SDK 或私有 API 文档的团队来说，这套工具链提供了一个低成本的接入路径：按照 `content/` 的目录约定写好 DOC.md，配置一个本地源，AI 编程助手就能实时获取最新的内部文档，而不是凭空猜测。