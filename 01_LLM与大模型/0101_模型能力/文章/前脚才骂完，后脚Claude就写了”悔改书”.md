---
title: 前脚才骂完，后脚Claude就写了”悔改书”
author: 字节笔记本
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIzMzQyMzUzNw==&mid=2247507993&idx=1&sn=386d7ee8444a0d5ec4345b3c70bdbd6c&chksm=e95dae9a1c49604a5cdbf678b72eafd5d355dbb90e09e11abc2bbadbea9eac0f2d7066c9f678&mpshare=1&scene=24&srcid=1107hOzwsorxwx6KeGoOCDma&sharer_shareinfo=30d166097b967b8b6cdf9db438cbd2b1&sharer_shareinfo_first=30d166097b967b8b6cdf9db438cbd2b1#rd
---

11月4日前脚才骂完 [为什么我的Claude Code选择“裸奔”？](https://mp.weixin.qq.com/s?__biz=MzIzMzQyMzUzNw==&mid=2247507895&idx=1&sn=654f8a9dd5a1f75f0f4ece339afde97a&scene=21#wechat_redirect)

没想到Claude官方当天就发布了一个Anthropic 工程实践文章。

官方原文链接：

https://link.bytenote.net/ZPr3VU

承认MCP工具定义会占用过多的上下文窗口，而且中间工具结果会消耗额外的标记。

和前面发布的文章的统计数据和体感也是完全一致。

而且官方在这个文章中特别提到，在处理每个中间结果都必须经过模型。官方举了一个"从 Google Drive 下载我的会议记录，并将其附加到 Salesforce 的潜在客户上"的例子。

在这个例子中，完整的通话记录会被传输两次。对于一场 2 小时的销售会议，这可能意味着需要额外处理 5 万个 token。更大的文档甚至可能超出上下文窗口限制，导致工作流程中断。

当然官方也不仅仅只是承认了MCP所带来的问题，还给出通过 MCP 执行代码提升上下文效率的方法。

其实也就是我上篇文章中所讲的完全一样的思路。

这样的收益有多少呢，原文如下：

"智能体通过探索文件系统来发现工具：首先列出 ./servers/ 目录以查找可用服务器（如 google-drive 和 salesforce ），然后读取所需的特定工具文件（如 getDocument.ts 和 updateRecord.ts ）来理解每个工具的接口。这使得智能体只需加载当前任务所需的定义，从而将令牌使用量从 150,000 个减少到 2,000 个——节省了 98.7%的时间和成本。"

没错，节省了 98.7%的时间和成本。

为什么能这么高效呢？官方引用了Cloudflare的观点：

“Cloudflare 也发布了类似的研究成果，将 MCP 代码执行称为"代码模式"。核心观点一致：LLMs 擅长编写代码，开发者应利用这一优势来构建能与 MCP 服务器更高效交互的智能体。”

其实总的思路就是一个叫“渐进式披露”的方法，模型非常擅长在文件系统中导航。将工具以代码形式呈现在文件系统上，可以让模型按需读取工具定义，而不必一次性全部读取。

比如：

可以在服务器上添加一个 search\_tools 工具来查找相关定义。例如，当使用上文假设的 Salesforce 服务器时，智能体会搜索"salesforce"并仅加载当前任务所需的工具。

在 search\_tools 工具中包含一个细节级别参数，允许智能体选择所需的细节程度（如仅名称、名称和描述，或包含模式的完整定义），也有助于智能体节省上下文并高效查找工具。

不过，还是有点难懂感觉无从下手是吧，所以当天我又Vibe Coding了一个小工具：[将任意MCP转为Claude Code Skill](https://mp.weixin.qq.com/s?__biz=MzIzMzQyMzUzNw==&mid=2247507914&idx=1&sn=0d54c73ee05f2bb3d1b5c3fca2bbcda8&scene=21#wechat_redirect)

这个小工具可以将任意MCP转为Claude Code Skill，使用Claude Code Skill以渐进式披露的方法来加载MCP Tool。

不仅可以对接既有的MCP丰富的生态，还能省去了Skill的编排书写过程，最主要是继承了Skill的灵活和按需加载，可谓一举多得。

[别学n8n了！用Claude Code Skills 复刻n8n工作流](https://mp.weixin.qq.com/s?__biz=MzIzMzQyMzUzNw==&mid=2247507976&idx=1&sn=4b9d6b1658295e24b22c713ddb83f576&payreadticket=HMWV1FZkmXG38fPJ1qspRwzhIshJ01kw6a_3kUP4CN0gY-GdW0kl1n-MyMRaj5GzEg-28B8&scene=21#wechat_redirect)

[为什么我的Claude Code选择“裸奔”？](https://mp.weixin.qq.com/s?__biz=MzIzMzQyMzUzNw==&mid=2247507895&idx=1&sn=654f8a9dd5a1f75f0f4ece339afde97a&scene=21#wechat_redirect)

[15分钟完成过去一周的工作量！Claude Code Skills 驱动的SDD开发流程规范](https://mp.weixin.qq.com/s?__biz=MzIzMzQyMzUzNw==&mid=2247507884&idx=1&sn=0e991f830a8835282e0cec9cda83a011&scene=21#wechat_redirect)