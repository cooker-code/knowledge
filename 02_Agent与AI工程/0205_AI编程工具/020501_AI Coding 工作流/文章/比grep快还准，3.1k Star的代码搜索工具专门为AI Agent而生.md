---
title: 比grep快还准，3.1k Star的代码搜索工具专门为AI Agent而生
author: 晴天的码场
date: 晴天的码场晴天的码场
url: https://mp.weixin.qq.com/s?__biz=MzYzNjczNDI1Mw==&mid=2247484885&idx=1&sn=cb188c9089432e123fe732d6b05e3ee7&chksm=f17c07cbf6712a8c62e2788e1f0cc278ffed17e6be13ee4572d899076bcf95e9d9a69a401ae0&mpshare=1&scene=24&srcid=0521QSPLSeTMYIeoUhBO6W5B&sharer_shareinfo=3fe6a35ef61afe0a456cfd555d301a73&sharer_shareinfo_first=3fe6a35ef61afe0a456cfd555d301a73#rd
---

# 比grep快还准，3.1k Star的代码搜索工具专门为AI Agent而生

前段时间在逛 GitHub 时发现了这个项目，发布没多久就收到了 3.1k+ ⭐，专门做给 AI 编程 Agent 用的代码搜索工具——semble。

**01**

## 它在解决什么问题

用过 Claude Code、Cursor 这类编程 Agent 的人多少都遇到过一个问题：Agent 想找某段代码，要么用 grep 搜关键词，要么直接把整个文件读进来，token 哗哗地烧。semble 要干的事就是：Agent 用自然语言问一句"这个项目怎么处理鉴权的"，semble 直接把相关代码片段精准返回，不读多余的东西。官方数据是比 grep+read 方式少用约 98% 的 token。

**02**

## 跑得有多快

semble 走的是轻量嵌入路线，全程跑在 CPU 上，不需要 API key，不依赖任何外部服务。按项目给出的数据：索引一个普通规模的代码库约 250ms，单次查询约 1.5ms。和专门做代码理解的 transformer 模型比，索引速度快约 200 倍，查询速度快约 10 倍，检索质量（NDCG@10）保持在 0.854，相当于专用模型的 99%。

**03**

## 接入方式

两条路：MCP Server 或者 Bash/AGENTS.md。用 Claude Code 的话，一行命令搞定：

```
claude mcp add semble -s user -- uvx --from "semble[mcp]" semble
```

Cursor、Codex、VS Code、Windsurf、Zed 等也都支持，各自往对应的配置文件里加一段 JSON 就行。子 Agent 不能直接调 MCP 工具，这种场景就用 Bash 方式，把 semble 的使用说明写进 `AGENTS.md` 或 `CLAUDE.md`，主流 Agent 都能识别。

**04**

## token 节省到底有没有那么夸张

semble 自带一个 `semble savings` 命令，会记录每次搜索节省了多少 token。计算方式是：用包含返回代码片段的文件总字符数减去实际返回的片段字符数，除以 4 估算 token 数。这是偏保守的估算，因为 Agent 探索陌生代码库时通常是整文件读入的。

**05**

## CLI 直接用也行

不想走 Agent，脚本里直接调也没问题：

```
# 自然语言搜索本地项目  
semble search "authentication flow" ./my-project  
  
# 搜远程仓库（自动 clone）  
semble search "save model to disk" https://github.com/MinishLab/model2vec  
  
# 找跟某行代码相似的代码  
semble find-related src/auth.py 42 ./my-project
```

**06**

## 适合谁用，门槛在哪

本质上是个开发者工具，目标用户是重度使用 AI 编程 Agent 的工程师。如果你的工作流里已经在用 Claude Code 或 Cursor 日常写代码，接入成本很低，一条命令或一段配置的事。如果你对 Agent 本身还不熟悉，semble 对你帮助有限。

---

觉得这个项目有意思的话，去 GitHub 给它点个 Star 支持一下。有在用 Claude Code 或 Cursor 的朋友，欢迎留言聊聊你现在的代码搜索工作流——是不是也遇到过 token 烧得莫名其妙的情况？

```
# GitHub 项目地址  
https://github.com/MinishLab/semble
```

谢谢你这么优秀还关注我✨