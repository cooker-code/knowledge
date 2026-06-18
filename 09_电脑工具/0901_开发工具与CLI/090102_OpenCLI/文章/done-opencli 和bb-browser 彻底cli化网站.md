---
title: opencli 和bb-browser 彻底cli化网站
author: AI宝藏搬运站
date:
url: https://mp.weixin.qq.com/s?__biz=MzA4NDMyOTQ2NA==&mid=2650095467&idx=1&sn=f6b1bf6d5116b407f8ab7ea0235836aa&chksm=86e199460711563e30a74f3167aab54d7f7904fb9b917ecc25062dc2846754dc273c174c6092&mpshare=1&scene=24&srcid=0419qPr7wZqzMkd7po0cCgYW&sharer_shareinfo=8e7e912a9312b03d076f56cf23b6c828&sharer_shareinfo_first=8e7e912a9312b03d076f56cf23b6c828#rd
---
> 已吸收至：[[09_电脑工具/0901_开发工具与CLI/090102_OpenCLI/090102_核心知识点/OpenCLI网站CLI化与Agent接口边界|OpenCLI网站CLI化与Agent接口边界]]


## 当 AI Agent 学会用命令行，整个互联网都是它的 API

---

https://mcp.edgeone.site/share/evJAm00NJ\_jlc4NE0MpyU

你有没有想过，让 Claude Code 帮你刷微博、看B站、逛小红书？

不是用 Browser Use 打开浏览器慢慢爬，而是**一条命令，3秒返回结构化 JSON**。

```
$ opencli weibo hot
[{"rank": 1, "title": "#某明星官宣#", "reads": "2.3亿"}, ...]

$ bb-browser site bilibili/popular
{"videos": [{"title": "硬核拆解视频", "views": "520万"}, ...]}

$ bb-browser site xiaohongshu/search "Claude Code 教程"
{"notes": [{"title": "手把手教你用...", "likes": "2.1万"}, ...]}
```

这不是科幻，这是 2025 年正在发生的事。

---

## 01 为什么 CLI 是 AI Agent 的最佳界面？

大家都在讨论 MCP、Function Calling、Browser Use。但有一个被低估的事实：

> **对于 AI Agent 来说，一个结构化的 JSON 输出，比一整个网页的 DOM 树有用一万倍。**

想象一下：你让 Claude Code 帮你抓取微博热搜榜。

**方案 A：Browser Use** 启动 Playwright → 打开浏览器 → 等待页面渲染 → 解析 DOM → 提取数据。光微博首页的 DOM 就有上万个节点，消耗大量 token，还可能被反爬拦截。

**方案 B：一条 CLI 命令**`opencli weibo hot` → 直接返回结构化 JSON → 3 秒搞定 → token 消耗趋近于零。

这就是 **CLI-first** 的力量：

* ⚡ **极速响应** — 无需等待页面渲染，直接拿到结构化数据
* 💰 **Token 节省** — JSON 输出 vs HTML DOM，token 消耗降低 90%+
* 🔗 **管道友好** — jq 过滤、管道组合、脚本自动化，Unix 哲学
* 🤖 **Agent 原生** — Claude Code / Cursor 直接 Bash 调用，无需额外集成

---

## 02 OpenCLI：把网站变成命令行

**GitHub**: github.com/jackwener/opencli

| 指标 | 数据 |
| --- | --- |
| 命令数 | 45 条 |
| 覆盖网站 | 16 个 |
| 安装方式 | `npm install -g @jackwener/opencli` |

覆盖平台：Bilibili / Twitter(X) / Reddit / 微博 / 小红书 / V2EX / HackerNews / 携程 等

**核心原理**：通过 Playwright MCP Bridge 浏览器扩展，复用你已登录的浏览器会话，直接从页面中提取数据并返回 JSON。不需要申请 API Key，不需要处理 OAuth，你能看到的数据，CLI 就能拿到。

### 安装 3 步走

```
# 1. 安装
npm install -g @jackwener/opencli

# 2. 安装 Chrome 扩展 "Playwright MCP Bridge"
#    点击扩展图标，复制 Token

# 3. 设置环境变量
exportPLAYWRIGHT_MCP_EXTENSION_TOKEN="your_token"
```

### 实战演示

```
# B站热门
$ opencli bilibili hot
1. 某UP主的硬核拆解视频 | 播放 520万
2. 2026年最值得学的技术栈 | 播放 380万

# 微博热搜
$ opencli weibo hot
[{"rank": 1, "title": "#热搜话题#", "reads": "2.3亿"}, ...]

# 小红书搜索
$ opencli xiaohongshu search "家居好物"
Found 30 notes matching query

# Twitter 趋势
$ opencli twitter trending
1. #AIAgent  2. #ClaudeCode  3. #MCP ...

# V2EX 热门
$ opencli v2ex hot
1. 有没有人用 Claude Code 开发过完整项目？
2. 2026 年了，你们还在用什么编辑器？
```

> **Pro Tip**: 如果你用的不是 Chrome，比如 Comet、Arc 等 Chromium 内核浏览器，可以设置 `OPENCLI_BROWSER_EXECUTABLE_PATH` 环境变量指向你的浏览器路径。

---

## 03 bb-browser：104 个 Adapter 的全能战士

**GitHub**: github.com/epiral/bb-browser

| 指标 | 数据 |
| --- | --- |
| Adapter 数 | 104 个 |
| 覆盖平台 | 36 个 |
| 安装方式 | `pip install bb-browser` |

覆盖平台：Bilibili / Twitter / Reddit / 微博 / 小红书 / YouTube / GitHub / 雪球 / 知乎 / Google / 豆瓣 / 东方财富 / Product Hunt / V2EX / LinkedIn / npm / PyPI / Wikipedia / 36Kr 等

bb-browser 采用 **Extension + Daemon** 架构：浏览器扩展捕获会话，Daemon 进程管理连接，Adapter 定义数据提取逻辑。

### 实战演示

```
# 安装
pip install bb-browser
bb-browser site update  # 拉取社区 adapter 库

# B站热门视频
$ bb-browser site bilibili/popular
{"videos": [{"title": "...", "views": "520万",
  "author": "某UP主", "bvid": "BV1xx..."}]}

# 微博热搜
$ bb-browser site weibo/hot
{"trends": [{"title": "#热搜第一#", "reads": "3.2亿"}...]}

# 小红书笔记搜索
$ bb-browser site xiaohongshu/search "咖啡推荐"
{"notes": [{"title": "这杯咖啡绝了", "likes": 8200}...]}

# 豆瓣 Top 250
$ bb-browser site douban/top250

# 雪球实时行情
$ bb-browser site xueqiu/stock SH000001

# 知乎热榜
$ bb-browser site zhihu/hot

# YouTube 视频搜索
$ bb-browser site youtube/search "Claude Code tutorial"
```

---

## 04 两个工具怎么选？

| 维度 | OpenCLI | bb-browser |
| --- | --- | --- |
| 语言 | TypeScript / Node.js | Python |
| 安装 | `npm install -g` | `pip install` |
| 覆盖 | 16 个网站 / 45 命令 | 36 个平台 / 104 adapter |
| 浏览器连接 | Playwright MCP Bridge | 自有 Extension + Daemon |
| Adapter 模型 | 内置 TS 模块 | 社区 YAML/JS，热更新 |
| 优势 | Node.js 生态、轻量启动 | 覆盖广、社区驱动 |
| 适合 | 前端/全栈开发者 | 数据分析/Python 用户 |

**我的建议**：两个都装。日常用 bb-browser（覆盖广），特定场景用 opencli（如小红书 Feed 拦截、B站字幕提取）。让 AI Agent 根据任务自动选择最优工具。

---

## 05 实战：AI Agent 自动生成热点日报

用 Claude Code + opencli + bb-browser 实现的真实场景：**每天自动聚合全网热点，生成一份结构化日报**。

### 工作流

**Step 1** — 抓取微博热搜`opencli weibo hot` 获取微博热搜 Top 50

**Step 2** — 拉取 B 站热门`bb-browser site bilibili/popular` 获取热门视频列表

**Step 3** — 搜索小红书笔记`bb-browser site xiaohongshu/search "今日热点"` 获取热门笔记

**Step 4** — AI 智能分析分类 Claude Code 自动分析 JSON 数据，按科技/娱乐/社会/财经分类

**Step 5** — 生成日报存入 Obsidian 输出格式化日报，一键存入笔记

### 实际执行

```
# 你只需要说一句话:
# "帮我看看今天全网在聊什么，生成日报存到笔记"

# Claude Code 自动执行:
$ opencli weibo hot | jq '.[:10]'
$ bb-browser site bilibili/popular
$ bb-browser site bilibili/trending
$ bb-browser site xiaohongshu/search "今日热点"
$ bb-browser site zhihu/hot
$ opencli v2ex hot

# 输出:
Generating daily report...
Saved to Obsidian: 日报/2026-03-15 全网热点日报.md
```

> 整个过程我只说了一句话。Claude Code 自动调用 6 条 CLI 命令，聚合微博 + B站 + 小红书 + 知乎 + V2EX 五个平台的数据，分析、分类、写报告、存笔记 --- 全程零浏览器操作。

---

## 06 CLI520：AI Agent 的工具注册表

**网站**: cli.cypggs.com**GitHub**: github.com/cypggs/cli

当 CLI 工具越来越多，AI Agent 怎么知道该用哪个？**CLI520** 解决的就是这个问题 --- 它是一个为 AI Agent 优化的 CLI 工具注册表。

### 核心特性

* 🤖 **双重可读** — 每个工具页面同时嵌入 HTML（人看）和 JSON metadata（AI 读）
* 🔗 **JSON API** — `/api/cli-index.json` 全量工具索引，Agent 可直接查询
* ✓ **Ground Truth** — 解决 AI 幻觉：工具名、命令语法、参数全部真实可验证
* 📦 **开源贡献** — Astro 6 构建，添加工具只需 Markdown + JSON 两个文件

### 为什么需要它？

AI Agent 经常会"幻觉"出不存在的 CLI 工具或错误的命令参数。CLI520 提供了一个 **ground-truth source**：Agent 先查询注册表确认工具是否存在、语法是否正确，再执行命令。

就像 npm registry 之于 Node.js，CLI520 之于 AI Agent。

```
# Agent 查询可用工具
$ curl-s https://cli.cypggs.com/api/cli-index.json | jq '.[].name'
"opencli"
"bb-browser"
"gh"
"obsidian-cli"
...
```

CLI520 完全开源，欢迎提交你发现的好用 CLI 工具。只需在 `src/content/cli/` 下创建一个目录，包含 `index.md` 和 `cli.json` 即可。

---

## 07 CLI-first Agent 的未来

回头看 AI Agent 的发展，有一个有趣的规律：

* **2023** — Agent 学会调 API（Function Calling）
* **2024** — Agent 学会用浏览器（Browser Use / Computer Use）
* **2025** — Agent 学会用命令行（CLI is All You Need）

命令行不是倒退，而是一种**更高效的抽象**。就像人类从图形界面回到终端不是因为 GUI 不好，而是因为终端更**可组合、可自动化、可编程**。

想象这样的未来：

* 💪 **万物皆 CLI** — 每个网站、每个 SaaS 都有对应的 CLI adapter
* 🔬 **工具发现** — Agent 自动查询注册表，找到最合适的工具
* 🔁 **工作流编排** — 多个 CLI 通过管道组合，形成复杂自动化
* 🌟 **社区驱动** — 开源社区贡献 adapter，覆盖长尾场景

> **The best interface for AI isn't a chatbox or a browser. It's a terminal. Always has been.**

---

## 开始你的 CLI Agent 之旅

* 📚 **CLI520** — cli.cypggs.com （AI-Native CLI 工具注册表）
* ⚙ **OpenCLI** — `npm install -g @jackwener/opencli`
* 🌐 **bb-browser** — `pip install bb-browser`

举个例子：如何用 CLI520安装 cli 工具，很简单一句话即可

`有了 weibo  cli 就可以很方便让 claw 去定时任务抓想看的信息了，下面这位 tk 圣手就是我很喜欢的博主，观点十分具有启发性。`

---

*Written with Claude Code | 2026.03*

```
echo "CLI is All Agents Need" | agent --execute
```
