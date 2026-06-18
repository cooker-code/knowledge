> 已吸收至：[[02_Agent与AI工程/0211_Memory Management/0211_核心知识点/记忆管理分层与注入边界|记忆管理分层与注入边界]]
---
title: OpenChronicle：给 AI Agent 装一个本地记忆层
author: Git Trend
date: wsleepybearwsleepybear
url: https://mp.weixin.qq.com/s?__biz=MzkwMzY1Mjg5MQ==&mid=2247486883&idx=1&sn=0999dc96bd163676500eb46aa56ce2db&chksm=c16d31ee0170b1224c98670b82edee86cff5fac09627fb6138442d324f3d219d1aa9660a7134&mpshare=1&scene=24&srcid=05019JeKl23eNVJin5k8ml1z&sharer_shareinfo=ef8dd3bdb767ac2da178427c2efb2fd6&sharer_shareinfo_first=ef8dd3bdb767ac2da178427c2efb2fd6#rd
---

> **项目卡片**
>
> * **项目**：OpenChronicle[1]
> * **状态**：v0.1.0 / 1.5k Stars / 2026-04-21 发布，一周内快速增长
> * **一句话判断**：目前最完整的开源 AI Agent 本地记忆方案，macOS 独占，工程完成度远超 alpha 标签暗示的水平。

OpenAI 最近发布了 Chronicle，指向一个明确需求：AI Agent 需要记住你的工作上下文。OpenChronicle 是这件事的开源替代——代码在 GitHub 上，记忆文件在本地磁盘上，你可以打开任何一个 Markdown 文件看它记了什么。

它做了什么

一个后台守护进程持续捕获你 Mac 上的屏幕上下文，压缩为结构化记忆，再通过 MCP 协议暴露给任何支持工具调用的 AI Agent。

核心流程是一条四级处理管道：

1. **Capture**：通过 macOS 辅助功能（AX）树捕获当前窗口的焦点元素、可见文本、URL
2. **Timeline**：每 60 秒将捕获窗口归一化为一个时间线块，保留用户原文、去掉 UI 噪声
3. **Reducer**：按会话边界（空闲切割、应用切换、超时）将时间线压缩为每日事件条目
4. **Classifier**：从事件条目中提取持久事实，写入 `user-`、`project-`、`person-` 等分类文件

最终产物是人可读的 Markdown 文件加 SQLite FTS5 全文索引。Agent 通过 MCP 工具查询这些记忆，回答"我在做什么"、"上次和 Alice 聊了什么"、"这个项目的背景是什么"之类的问题。

AX-first：为什么不用截图

这是我读完整份代码后觉得最有信息量的设计取舍。

OpenAI Chronicle 以截图 + OCR 为主信号。OpenChronicle 反其道行之，优先使用 macOS 的 AX Tree（Accessibility Tree）——操作系统维护的结构化 UI 元素树。

理由很务实：

* **成本低**。结构化文本的 LLM 处理成本远低于截图的视觉理解管线
* **意图更准**。AX Tree 直接暴露当前焦点元素、可编辑区域的用户输入、URL、窗口标题——这些是理解用户在做什么的精确信号
* **记忆更干净**。结构化文本容易去重、归一化、索引，长期积累后质量可控

仓库文档中有一个很说明问题的诊断：在 Claude Desktop 中一段 90 秒的对话，用户输入的 "18:00" 出现在 AX Tree 的第 5639 个字符处。AX 深度设为 8 时只拿到窗口边框和侧栏标题；设为 100 才能拿到完整对话内容。默认深度 100 就是基于这个发现选定的。

截图作为辅助信号保留——捕获时同时截屏，但 24 小时后自动剥离 base64 数据（占每个捕获 77% 的体积），只保留 AX 文本。这个分层保留策略让 7 天的默认保留窗口在存储上可承受。

会话驱动的记忆写入

v1 版本按每次捕获写入记忆，导致了两个生产问题：长会话被去重逻辑吃掉尾部，以及事件文件按周归拢无法回答"今天做了什么"。v2 彻底改为会话驱动。

会话切割有三条规则：

* **硬切**：5 分钟无任何捕获事件 → 在最后一个事件时间点结束
* **软切**：单个不相关应用聚焦 3 分钟，且最近 2 分钟没有频繁切换 → 结束当前会话
* **超时**：会话超过 2 小时强制结束

频繁多应用切换时软切不触发——这是对"IDE + 终端 + 浏览器参考"这种真实工作模式的适配。

写入节奏上，活跃会话中 reducer 每 5 分钟增量 flush 一次（标记为 `[flush]`），classifier 每 30 分钟提取一次持久事实。会话结束时做最终 reduce + 终端 classifier 补跑。每条记录有唯一 ID 和时间戳，每条写入都会推进会话行的书签位，确保不重不漏。

记忆格式：Supersede，不删除

记忆文件是带 YAML frontmatter 的 Markdown，按实体类型分文件：`user-profile.md`、`project-openchronicle.md`、`person-alice.md` 等。

事实更新时不是删除旧记录，而是用 `supersede` 操作：旧条目正文加删除线、标记 `#superseded-by:{new_id}`，新条目追加在末尾。FTS 索引中旧条目标记为已替代，默认搜索不出现，但带 `include_superseded=true` 可以追溯完整变更链。

这个设计的好处是：人类可以直接打开 Markdown 文件阅读、grep、diff，数据始终透明。SQLite 索引是派生视图，随时可以通过 `openchronicle rebuild-index` 从文件重建。

文件膨胀时有 compaction 机制：LLM 重写压缩，但用基于名词短语的保持率检查（>5% 丢失则拒绝）。压缩不会静默丢信息。

模型无关与本地运行

四个 LLM 阶段（timeline、reducer、classifier、compact）各自可配置不同模型，全部通过 litellm[2] 路由。这意味着 Ollama、LM Studio、OpenAI、Anthropic、DeepSeek——任何 litellm 兼容的 provider 都能用。

完全本地的配置示例：

```
[models.default]
model = "ollama/qwen2.5:7b"
base_url = "http://localhost:11434"
api_key_env = ""

[models.classifier]
model = "ollama/qwen2.5:14b"
```

仓库文档明确标注了本地部署的注意事项：classifier 必须支持 tool-calling（驱动 append/create/supersede 的函数调用循环），timeline 和 reducer 需要 JSON mode，context window 建议 ≥32k。小模型（如 Llama-3.2 和 Phi 变体）在 tool-calling 上不可靠。timeline prompt 要求用户原文必须加引号保留不得转述；classifier 提取偏好类事实需要 ≥2 个独立会话的交叉验证——这些反幻觉措施直接写在了 prompt 里。

`openchronicle status` 命令会探测每个阶段配置的模型可用性并显示响应时间，第一次就能发现配置问题而不是等到运行几小时后。

MCP 集成：一个 URL 接入所有 Agent

MCP 服务运行在守护进程内部（`http://127.0.0.1:8742/mcp`），提供 8 个只读工具：`list_memories`、`read_memory`、`search`、`recent_activity`、`current_context`、`search_captures`、`read_recent_capture`、`get_schema`。

注意一个关键设计：MCP 端点只暴露读操作，记忆写入完全由内部 writer 管道负责。这是架构上的硬保证，不是约定。Agent 查记忆时有两层可 drill-down：压缩记忆（Markdown 文件）找不到，事件条目内嵌了 `read_recent_capture(at="14:30", app_name="Cursor")` 这样的面包屑可以直接跳到原始屏幕内容。

集成方式很省心：

```
openchronicle install claude-code    # 一条命令完成
openchronicle install codex
openchronicle install opencode
```

支持 Claude Code、Claude Desktop、Codex、Cursor、opencode 等。ChatGPT Desktop 因为 MCP 客户端在 OpenAI 云端运行，需要通过 ngrok/Cloudflare Tunnel 暴露本地端点——仓库文档诚实标注了这意味着数据离开本机的安全代价。

一个容易被忽略的可靠性设计：本地时间 23:55 自动强制结束当前会话并回收所有 `ended`/`failed` 状态的未处理会话。守护进程崩溃或跨午夜都不会导致数据丢失。

局限与风险

* **macOS 独占**。依赖 macOS 辅助功能 API 和 Swift 二进制，没有 Linux/Windows 支持
* **早期 alpha**。v0.1.0，发布仅一周，API 和配置格式可能变化
* **需授予辅助功能权限**。用户必须在系统设置中手动授权，对非技术用户有门槛
* **LLM 成本不可忽视**。timeline 每分钟运行一次，长工作日可能产生数百次 LLM 调用；本地模型有硬件门槛
* **无认证机制**。MCP 端点默认无鉴权，任何本地进程可查询你的记忆

总结

AX-first 路线在成本和精度上有结构性优势，会话驱动解决了 v1 的实际生产问题，supersede-not-delete 的语义让人类始终拥有数据的完整视图。配套的 8 个 MCP 工具覆盖了从"我现在在干什么"到"上周三和 Alice 聊了什么"的常见查询场景。

它目前还不是一个开箱即用的产品——macOS 独占、需要手动授予辅助功能权限、LLM 调用有持续成本。但作为发布仅一周的项目，它的架构完成度和文档质量已经超出 alpha 标签给人的预期。如果你在用 Claude Code 或 Cursor，并且觉得每次对话都从零开始是浪费，可以试一下。

---

如果这篇对你有用，建议点个关注。我会持续把 GitHub 上值得用的 AI 工具拆成「最短上手闭环 + 坑点清单 + 可复用配置」，让你少走弯路。

### 引用链接

[1]OpenChronicle: *https://github.com/Einsia/OpenChronicle*

[2]litellm: *https://github.com/BerriAI/litellm*