> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: 我从Github提取了1000个Claude Code的用户配置文件，分析了一下程序员用什么MCP
author: 魔轮智能说
date:
url: https://mp.weixin.qq.com/s?__biz=MzkwNjg2NDE3MQ==&mid=2247484721&idx=1&sn=e2ae093da0daba5300fc475fa730914b&chksm=c1017fb4669f64584e17e99e6d5212e2d644c76d57fc41713b83431f5d2ddf2fedfa1fc977d2&mpshare=1&scene=24&srcid=1115weas4O27yRjnz2WXGghw&sharer_shareinfo=80e96b9ce9e1c489f4e96c279940563f&sharer_shareinfo_first=80e96b9ce9e1c489f4e96c279940563f#rd
---

# 哎嘿

最近有点疯魔，一直在考虑什么MCP可以让自己可以更好、更快的完成工作。

然后这个问题，大概率是很难有答案。

然后灵机动了再动，最后还是打算掏出本人最喜欢的，数据分析来解决。

分析啥，自然是Github上的的大大大哥 无意中泄露出来的，项目的 Claude Code设置文件，里面其实有项目的allow、deny这些信息。这里就可以捞到MCP的信息

至于怎么获取到的这些信息，就不说了，懂的自然懂，不懂的也不需要去了解，但是方式绝对是合法合规的。

# 直接说结果吧

## 概要数据

* • 提取了1000个文件
* • 合并去重后，提取出来 MCP 规则 576 条
* • 按MCP软件合并，涉及MCP工具 125 个
* • 各种MCP，出现在超过（>=）10个项目的，仅有23个

这里有点点必要的说明：

1. 1. 一个MCP，如果它有很多函数，在统计里面就会出现很多次，的确有点不公平，但是，不纠结，反正我就是用来大海摸鱼的。
2. 2. 因为MCP是按大家配置的信息配进去的，文字有点儿混乱，虽然我已经整理了一下，但是也可能会出错
3. 3. 因为存在大量的开源MCP，一个服务可能有很多个不同的MCP，其实都是用对应的最后的服务，，虽然我已经整理了一下，但是也可能会出错

## 概要的看法

* • Serena很重要，虽然上手有点门槛，但是群众已经点赞，社区的说法是，如果只能有一个MCP，只用他
* • 自动化测试是MCP应用的重要方式，上榜的不少是自动化测试的
* • 大模型信息跟不上最新情况，给大模型提供最新的技术知识，是刚需，譬如contenx7这些，也是大幅度使用的
* • 大模型虽然可以做计划，然后去按顺序开展工作，各种cli也有对应的todo工具。但是受限于大模型实在太多，能力不一，不少人还是选择用 mcp 来做计划编排工具

其他，就自我见解了

## 下面是个数据清单

| MCP名称 | 数量 |
| --- | --- |
| serena | 166 |
| playwright | 162 |
| context7 | 64 |
| ide | 52 |
| github | 49 |
| filesystem | 45 |
| puppeteer | 38 |
| desktop-commander | 32 |
| archon | 29 |
| sequential-thinking | 28 |
| memory | 27 |
| chrome-devtools | 25 |
| supabase | 24 |
| laravel-boost | 21 |
| git | 20 |
| taskmaster-server | 19 |
| code-analysis-server | 18 |
| godot | 13 |
| zen | 13 |
| rust-docs | 12 |
| browserbrowser | 11 |
| docker-gateway | 11 |
| kapture | 11 |
| render | 10 |

# 下面用超简单的方式，介绍一下这些MCP

### Serena

提供类似IDE的语义级代码检索和编辑工具，非常强大，但是要用好有点麻烦，建议大家去了解一下

### Playwright

前端自动化测试必备，当前后端联调也是可以的

### Context7

提供最新、最准确的代码文档和示例，避免大模型的写代码知识陈旧

### IDE

这个就没搞清是什么，无法介绍。，蛤蛤蛤蛤

### GitHub

不用介绍了吧，，，

### Filesystem

本地文件管理工具

### Puppeteer

也是一个自动化前端的工具，和playwright一个用途。

### Desktop Commander

电脑桌面级MCP，功能非常强大，特别涉及大项目的文件查找之类，非常好用。但是很多时候会用来代替基本的文件管理（filesystem）

我强力推荐

### Archon

Archon这个东西，我也是第一次知道，不好解释，给个地址：https://github.com/coleam00/Archon

看介绍很牛逼，翻译过来的介绍：

Archon是AI编程助手的指挥中心。对您而言，它是一个用于管理项目知识、上下文与任务的优雅界面；对AI编程助手而言，它则是通过模型上下文协议（MCP）实现协同处理相同知识、上下文与任务的服务器。通过连接Claude Code、Kiro、Cursor、Windsurf等工具，让您的AI助手能够调用：

• 项目文档（爬取的网页内容、上传的PDF/文档）
• 采用先进RAG策略的智能搜索功能
• 与知识库联动的任务管理系统
• 实时同步更新——当您新增内容或与编程助手协同处理任务时
更多功能即将上线，Archon将持续演进为全方位的上下文工程集成环境

此次Archon的全新定位取代了旧版本（原agenteer）。过去Archon是用于构建AI智能体的母体，而现在您不仅能实现智能体构建，还能解锁更广阔的应用场景。

### Sequential Thinking

鼎鼎有名的将事情分解为系列步骤，然后按顺序执行的mcp。

### Memory

将关键信息保存下来作为短期/长期记忆，以便后续使用的mcp，人话就是‘避免失忆’。

### Chrome DevTools

也是前端自动化测试的MCP了

### Supabase

Supabase，开源的Firebase替代方案，我也不懂如何介绍了，需要用到supebase的自然懂了。

### Laravel Boost

为Laravel 开发准备的MCP，集成海量文档和对应的功能

### Git

不解释

### Taskmaster Server

找不到信息，太多类似的了，请见谅

### Code Analysis Server

应该是这个 https://github.com/johannhartmann/mcpcodeanalysis 。静态代码分析和搜索的工具

### Godot

Godot 游戏开发mcp

### Zen

应该就是 zen-mcp-server 这个。核心是让这个MCP协调多个AI模型/工具来干活。

https://github.com/BeehiveInnovations/zen-mcp-server

### Rust-docs

应该是这个 https://github.com/Govcraft/rust-docs-mcp-server

为特定Rust crate提供精准更新的知识源，也是解决大模型的信息跟不上时代的

### Browserbrowser

也是一个浏览器自动化测试的工具

### Docker Gateway

Docker的MCP，不需要解释了

### Kapture

应该是 这个 https://github.com/williamkapke/kapture
也是一个浏览器自动化测试的工具

### Render

Render云平台的MCP，是用来配合云资源管理的。

# 完结

本稿水完，结论就是

1. 1. 我准备去调整下我自己的MCP组合了
2. 2. 各位用git的，记得还是做好 .gitignore 文件的管理