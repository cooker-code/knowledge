> 已吸收至：[[02_Agent与AI工程/0211_Memory Management/0211_核心知识点/记忆管理分层与注入边界|记忆管理分层与注入边界]]
---
title: 把 Obsidian 变成 Claude 的长期记忆：4 个组件搞定
author: 向量猫
date: 向量猫向量猫
url: https://mp.weixin.qq.com/s?__biz=MzYzNzg4ODc5NQ==&mid=2247484108&idx=1&sn=f95bdd0dec77740c0e7a50ccc83e755f&chksm=f199ee962f6615013339b54a6cbcecdac56f2703496df54e86f452540b1c1f1a01436f79f2c9&mpshare=1&scene=24&srcid=0428TEukihKH8ZsR7Pz4zURE&sharer_shareinfo=1d99758944f06506d9b0c126cd89ec3a&sharer_shareinfo_first=1d99758944f06506d9b0c126cd89ec3a#rd
---

把 Claude 的对话沉淀进 Obsidian vault，索引交给 Ollama 跑——这条链路比想象中短。

## Claude 的"金鱼脑"问题，最便宜的解法在你硬盘里

聊到第三轮，Claude 已经忘了你前天讨论过的项目架构。这是所有 Claude 重度用户都在踩的坑。

云端方案有 ChatGPT 的 Memory，有各家厂商的"持久化记忆"。但这些方案都有同一个问题：你的笔记交给了别人。换一个工具，记忆就归零；某个厂商关停服务，过去半年的对话沉淀全部作废。

最近翻 GitHub 的时候，发现一条更有意思的路径——把 Obsidian vault 当 Claude 的外置硬盘，Ollama 跑本地 embedding，记忆完全留在自己机器上。整条链路涉及四个组件，没有一个是重量级框架，每个都是 markdown 友好的小工具。

这篇文章把这套组合拆开讲清楚。包括两条互不冲突的记忆线、一条比常见教程更短的路径、Ollama 在里面真正的位置，以及最小上手时该按什么顺序装。

## 整条链路的最小拼图：四个组件就够了

先把骨架画清楚，避免一上来就陷进配置细节。

要让 Claude Desktop 能"读+写"你的本地笔记，需要一座桥。

第一层是 **Obsidian Local REST API**。这是 coddingtonbear 写的社区插件，开启后 Obsidian 在本机 27124 端口暴露一组 HTTPS 接口，外部程序可以拿 API key 读写 vault。整套方案里它最不起眼，但缺了它后面什么都跑不起来。注意它默认是 HTTPS 自签证书，第一次连接会有信任问题，必须手动信任或者改成 HTTP 模式（仅限本机）。这一步卡住的人不少。

第二层是 **mcp-obsidian**。MarkusPfundstein 维护，3.5k star。它是一个 MCP server，把 REST API 包装成 Claude Desktop 能调用的 7 个工具：list\_files\_in\_vault、list\_files\_in\_dir、get\_file\_contents、search、patch\_content、append\_content、delete\_file。

这 7 个工具值得逐个看一眼，因为它决定了 Claude 在 vault 里能干什么：

* `list_files_in_vault`

  / `list_files_in_dir`：让 Claude 知道你都有什么笔记，相当于 `ls`
* `get_file_contents`

  ：读单个 markdown 文件
* `search`

  ：调 Obsidian 内置的关键词搜索（注意是关键词，不是语义）
* `append_content`

  ：往现有笔记追加内容，最常用
* `patch_content`

  ：定位到具体 heading/段落做局部修改
* `delete_file`

  ：删笔记，少用

安装一行命令：`uvx mcp-obsidian`，把 API key 写进 `claude_desktop_config.json` 的 env 字段就行。

第三层是 **Claude Desktop 本体**。它通过 MCP 协议调用上面那 7 个工具，本质上把 vault 当成可读可追加的工作区。配置文件位置 macOS 在 `~/Library/Application Support/Claude/claude_desktop_config.json`，Windows 在 `%APPDATA%\Claude\` 下。改完必须完全退出 Claude Desktop（不是关窗口，是 Quit）才能生效。

第四层是可选的 **Ollama**。这一层之后再说，先记住——它不是用来替代 Claude 的。

> 整条链路的本质是：Claude 还是 Claude，Obsidian 还是 Obsidian，中间用 MCP 把两边接起来。所有数据都是本地 markdown，没有任何专有格式。

## 一份 vault，两条记忆线，可以同时跑

这是翻完几个仓库 README 之后才看明白的事，大多数教程没说清楚。

同一个 Obsidian vault 里，你其实可以并行两条记忆线，互不冲突。

**线 A：Claude 视角的记忆。** 走 Local REST API → mcp-obsidian → Claude Desktop。Claude 通过 MCP 工具读 vault、追加新笔记、改已有内容。它把这堆 markdown 当成自己的"外置长期记忆"。每次新对话开始，你只要一句"看一下 vault 里和 X 项目相关的笔记"，Claude 就会自己调 search 和 get\_file\_contents。

**线 B：本地模型视角的记忆。** Smart Connections（4.9k star）和 obsidian-copilot（6.8k star）这两个插件，在 Obsidian 内部直接用 Ollama 跑 embedding 和对话模型，把同一份 markdown 当成语义检索的索引源。你在 Obsidian 侧边栏就能问"这份会议纪要和上个月的产品规划有什么关系"。

关键在于：两条线写入和读取的都是 `.md` 文件。

没有数据库迁移，没有格式锁定。Claude 写的笔记，Smart Connections 第二天就能索引到；你在 Obsidian 里手动整理过的内容，Claude 下一轮对话立刻能搜到。

这件事的实操含义是——你不用在"给 Claude 建记忆"和"给本地模型建知识库"之间二选一。一份 vault，两边都喂。

obsidian-copilot 在 v3.1.0 把 long-term memory 做成 Agent 自动调用的工具，意味着本地 Agent 那条线也开始有"主动记"的能力。两条线在同一份 vault 上各自演化，但底层文件是同一份。Smart Connections 和 obsidian-copilot 的差异也值得一提：前者更侧重"被动连接"，在你浏览某篇笔记时自动推荐相关内容；后者更接近"主动对话"，给你一个聊天侧栏。两个可以一起装，索引会重复一份但不冲突。

## 一条比常见教程更短的路径：basic-memory v0.19.0

如果只是为了给 Claude 加语义记忆，其实有比 mcp-obsidian 更直接的选择。

basicmachines-co/basic-memory，2.9k star。它是一个专门给 LLM 做长期记忆的 MCP server，把 Claude 的对话沉淀成 `~/basic-memory/` 目录下的 markdown，结构跟 Obsidian vault 完全兼容——同样的 frontmatter、同样的 `[[wiki link]]` 风格、同样的标签语法。

这里有个被严重低估的更新——v0.19.0。

之前用 mcp-obsidian 的 search 工具，本质是调 Obsidian 的关键字搜索 API，没有 embedding。搜"用户认证相关的笔记"经常什么都搜不到，因为关键词没匹配上。你笔记里写的是"OAuth 流程"，Claude 搜"用户认证"就空手而归。

v0.19.0 给 basic-memory 内建了 **FastEmbed hybrid search**：本地向量搜索 + 全文检索混合排序。FastEmbed 整个跑在本地，不调任何云端服务。Hybrid 的意思是关键词匹配和语义匹配各占一半权重，避免纯向量检索把字面命中给挤掉。

这意味着什么？

把 basic-memory 的目录直接指向你现有的 Obsidian vault，Claude 立刻就有了语义检索能力。**不用装 Local REST API 插件，不用装 Smart Connections，不用 mcp-obsidian。** 一个 MCP server 全搞定。

安装也是一行：

```
uv tool install basic-memory
```

然后在 `claude_desktop_config.json` 加：

```
"basic-memory": {   "command": "uvx",   "args": ["basic-memory", "mcp"] }
```

v0.19.0 同时还塞了几个值得用的小特性：edit\_note 自动建文件、write\_note 的 overwrite guard（防止 Claude 不小心覆盖你手写的笔记）、Schema 推断/校验、Per-Project Cloud Routing、FastMCP 3.0、auto-update。最后那几个对长期使用很重要——意味着 Claude 写出来的内容会按你 vault 的现有 frontmatter 风格来，不会一夜之间格式打架。

overwrite guard 这个细节特别值得拎出来。Claude 在长对话里偶尔会"自信地"覆盖一篇你精心整理过的笔记，v0.19.0 之后它会检测目标文件是否已有大量内容，要求 LLM 显式声明"我确定要覆盖"才放行。这是从用户血泪反馈里加出来的设计。

如果你只想要"Claude 记得住事"，这条路径比 mcp-obsidian 那条短一半。两条路径并不互斥——很多人最后是 basic-memory 做语义检索 + mcp-obsidian 做精确文件操作，分工合作。

## Ollama 真正的位置：embedding 提供方，不是对话模型

很多人第一次看到"Obsidian + Ollama + Claude"的组合，第一反应是：Ollama 是不是用来替代 Claude 的本地大脑？

不是。

在长期记忆这个场景里，Claude（Sonnet / Opus）仍然是推理大脑，Ollama 干的是另一件事——**给 vault 跑 embedding 模型**。

具体来说，Smart Connections 和 obsidian-copilot 都支持把 embedding 模型切到本地 Ollama，常见选择是 nomic-embed-text 或 bge-m3。两者特性对比：

| 模型 | 体积 | 维度 | 适合场景 |
| --- | --- | --- | --- |
| nomic-embed-text | 约 274MB | 768 维 | 英文为主 |
| bge-m3 | 约 1.2GB | 1024 维 | 中英混杂 |

如果你 vault 里中英混杂，bge-m3 几乎是默认选择。

普通笔记本就能跑，M 系列 Mac 上 nomic-embed-text 索引几千篇笔记大概十几分钟，bge-m3 慢一些但质量更好。索引一次后增量更新就很快了。

这样的好处是：

* 推理质量靠 Claude，不降级
* 索引和向量完全离线，不会泄露给任何 embedding 服务（OpenAI 的 text-embedding-3-small 虽然便宜，但等于把你所有笔记内容都喂给了 OpenAI）
* vault 里有多少笔记，Ollama 就帮你索引多少，不花一分钱

> obsidian-copilot 在 README 里写过一句很到位的话：no provider lock-in，因为记忆就是 markdown。换模型不丢记忆，这是这套架构最大的价值。

把 Ollama 摆在 embedding 这个位置上，整套方案的成本和隐私边界就都清晰了：推理走 Claude API，按 token 付费，质量在线；索引走本地 Ollama，零成本零外发，速度够用。两边各自做擅长的事。

## 适合谁、不适合谁、最小上手路径

**适合的场景：**

* 你已经有一份用了一段时间的 Obsidian vault，至少几百篇笔记
* Claude 是你日常主力，每次对话都要重新喂上下文很烦
* 介意把笔记内容发给云端 embedding 服务（咨询、医疗、法律行业尤其敏感）
* 不希望被某个厂商的 Memory 功能锁定

**不太适合的：**

* 完全没用过 Obsidian 的人。先把 vault 建起来再谈记忆，否则只是给 Claude 一个空文件夹
* 移动端为主的用户。整条链路依赖桌面端 Claude Desktop + 本地 REST API，手机上跑不起来
* 期望"开箱即用"的人。MCP 配置文件、插件设置、Ollama 模型下载，至少要折腾一个晚上
* 想用 Claude.ai 网页版的人。MCP 目前主要在 Desktop 客户端生效

**最小上手路径建议按这个顺序：**

1. **先装 Claude Desktop + basic-memory**

   ，跑通基础语义记忆。这一步不依赖 Obsidian，几分钟搞定，先确认 MCP 链路本身能用
2. **把 basic-memory 的目录指向你的 Obsidian vault**

   ，让 Claude 直接读你已有的笔记
3. **再加 Obsidian 端的 Ollama 索引**

   （Smart Connections 或 obsidian-copilot 二选一），让你在 Obsidian 里也有语义搜索
4. **最后才是 mcp-obsidian + Local REST API**

   ，如果你需要 Claude 做精确的 patch\_content 这种局部修改才上

倒过来装很容易在 REST API 那步卡住放弃，那是整条链路里最脆弱的一环。

记忆这件事最怕的就是格式锁定。这套组合最让人安心的一点是——哪天 Claude 不用了，所有记忆还是一堆 markdown，照样能给下一个工具用。这是 ChatGPT Memory 和各家厂商方案给不了的东西。

---

> 这里是向量猫，帮你从 100 个工具里挑出 3 个真正值得用的。

参考来源：

* mcp-obsidian：https://github.com/MarkusPfundstein/mcp-obsidian
* basic-memory：https://github.com/basicmachines-co/basic-memory
* obsidian-smart-connections：https://github.com/brianpetro/obsidian-smart-connections
* obsidian-copilot：https://github.com/logancyang/obsidian-copilot
* obsidian-local-rest-api：https://github.com/coddingtonbear/obsidian-local-rest-api