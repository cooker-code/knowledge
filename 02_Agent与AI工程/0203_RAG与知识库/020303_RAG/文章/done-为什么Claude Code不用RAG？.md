> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020303_RAG/020303_核心知识点/RAG生命周期与评估边界|RAG生命周期与评估边界]]
---
title: 为什么Claude Code不用RAG？
author: 我有一计
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg2ODI0MTg1Mw==&mid=2247490398&idx=1&sn=2bbe85856ab2b00c1326c8b78c6843d8&chksm=cfb6812ececf0ac9f773d8e8056f295d793837b7ac84ac902bafed7d763c710a9c4c2dcd091f&mpshare=1&scene=24&srcid=10287Ym2XgMjVTgWtx18mpfL&sharer_shareinfo=930dc41b7987e02b0b59b473bb155928&sharer_shareinfo_first=930dc41b7987e02b0b59b473bb155928#rd
---

# 引言

最近看到有一些文章提出了“RAG已死”的观点。

核心论据是 Claude Code 开发负责人 Boris Cherny 在一档播客节目[1]中，披露现在的 Claude Code已不再使用RAG，而改用 Agentic Search。

他的主要观点如下：

> 起初采用RAG路线，将整个代码库索引一遍，然后用 Voyage 这类检索器上，让模型用提示去查找信息，用的是标准模板。
>
> 但到最后，团队放弃了结构化检索，转而选择了最简单粗暴的办法：Agentic Search。
> 主要有以下三个原因：
> 第一，它比RAG强，不是说跑了什么标准测试，是纯“体感”，内部测了一圈，每个人一致觉得更聪明更顺滑；
> 第二，RAG要建索引，代码一变就过时，一天几次推送，索引没法实时更新；
> 第三，安全问题。索引如果被泄露，会出现安全性问题。

# 什么是 Agentic Search

Claude Code 用的 Agentic Search 指的是让AI自主去搜集相关信息。

主要方式很简单，用的就是`Glob`和`Grep`这样的传统工具，在源码文件中全文搜索匹配[2]：

* Glob：查文件路径：匹配符合通配符模式的文件名或路径
* Grep：查文本：搜索包含某字符串或正则的文本行

Claude Code 在使用时，会有一些`Read`的步骤就在做此操作。

# Coding 领域的路线之争

在 Coding Agent领域，其实存在以下三种技术路线[3]：

* 第一组：完全抛弃RAG的实时搜索派
  Cline 和 Claude Code
  相似点：都完全放弃了传统的RAG方法，采用实时动态搜索
* 第二组：基于图结构的智能索引派
  Aider
  使用AST+图结构，将代码文件作为节点，依赖关系作为边，通过自定义排名算法（不是PageRank）优化代码映射
* 第三组：混合RAG+高级技术派
  Cursor 和 Windsurf
  相似点：都使用embeddings向量化+RAG，但加入了很多高级技术
  区别：

+ Cursor：使用OpenAI的embedding模型+TurboPuffer向量数据库，实验DSI技术，支持云端分布式架构
+ Windsurf：使用”M-Query技术”+本地/远程混合索引，专注企业级部署，有Cascade代理系统

Agentic Search 在目前的效果上优于 RAG，但代价就是需要花更长的时间进行检索，以及花更多的Token。

# RAG本质

RAG本质是**空间换时间**，知识库和数据库有点异曲同工，比如在 MySQL中，会通过构建B+树索引的方式，去加速查询。

所谓索引，就像给一本书加一个目录，需要用额外的纸张去记录，但提升了查询效率。

再比如一些检索库，ES会采用倒排索引，就是花额外的时间去构建一个倒排表，统计哪些词出现在哪些文档之中：

| 词项（Term） | 出现在哪些文档中（Doc IDs） |
| --- | --- |
| 我 | [1] |
| 喜欢 | [1] |
| 吃 | [1] |
| 苹果 | [1, 2, 3] |

RAG的局限是必须花额外的空间进行存储，并且需要提前把材料准备好，但对于一些高频修改的场景，如代码库，会存在时效性问题。

# 总结

总而言之，到底采用 RAG 还是 Agentic Search 需要分场景讨论：

* 如果是问答类应用场景，知识基本固化，采用 RAG 的方式更好，不会让用户等待特别长的时间
* 对于高频动态的场景，比如代码编辑的场景中，信息更迭加快，用 Agentic Search 反而能取得更好的效果

| 特征 | RAG | Agentic Search |
| --- | --- | --- |
| 架构 | 检索+生成 | 推理+多轮搜索 |
| 智能水平 | 弱 | 强 |
| 成本 | 低 | 高 |
| 实时性 | 弱 | 强 |
| 可控性 | 强 | 弱 |
| 最佳用途 | 静态知识问答 | 动态探索与分析任务 |

这两者是可以有机结合的，参照[4]一些复杂Agent的理论，记忆层分成以下四种类型，而RAG可以作为外部记忆，让Agent像调用工具一样进行获取。

# 参考

[1] https://x.com/mubeitech/status/1926953711198163394
[2] https://blog.csdn.net/liuruiaaa/article/details/148877434
[3] https://x.com/BadUncleX/status/1933081602277716275
[4] https://www.bilibili.com/video/BV1Rz4iz1Exz