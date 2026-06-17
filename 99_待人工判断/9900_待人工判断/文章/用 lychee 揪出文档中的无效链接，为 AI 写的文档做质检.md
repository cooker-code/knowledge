---
title: 用 lychee 揪出文档中的无效链接，为 AI 写的文档做质检
author: oh my x
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzk4MTc0Ng==&mid=2247494079&idx=1&sn=53b6e09bbe8cb6e1103ed99792b1824c&chksm=c19cc5809eed0a56faa64f535f648c8fda9133b66f03e83efc2c791d0fc63969506ebe73de9d&mpshare=1&scene=24&srcid=0321nc5UPNcisxIWSW8IBdbQ&sharer_shareinfo=92674cea6cd54e504185577ba2440bf8&sharer_shareinfo_first=92674cea6cd54e504185577ba2440bf8#rd
---

让 AI 帮忙收集资料、撰写文档。花了半小时，看起来内容充实、逻辑通顺。但有些链接要么点不开，要么点进去显示"页面不存在"。没办法确认资料的来源，无法验证内容是否真实可靠，搞了半天反而更花时间。

这不是小概率事件。AI 生成内容时，不仅会捏造事实，也会捏造链接：看起来格式正确、域名合理，但指向的内容根本不存在。

### lychee 是什么？

lychee 是一个用 Rust 编写的异步链接检查器，支持 Markdown、HTML、reStructuredText、网站等多种输入源。它能快速发现无效链接和邮件地址。

它的定位很清晰：不是普通的死链检查工具，而是文档质量基础设施的一部分。

### 快速上手

用 x-cmd 安装 lychee 非常简单，只需在终端中运行以下命令即可：

```
x install lychee
```

最基本用法，检查当前目录所有文件或指定文件：

```
lychee .  
lychee README.md test.html info.txt
```

输出示例（发现断链时）：

```
✗ https://example.com/not-found [404]  
✓ https://github.com/lycheeverse/lychee [200]
```

缓存检查结果，避免重复请求：

```
lychee --cache .
```

检查指定网站链接：

```
lychee https://endler.dev
```

### 实际工作流：AI 生成 + lychee 验证

**独立检查流程**

AI 生成文档后，运行 lychee 检查，人工只处理报告的断链：

```
AI 生成文档 → lychee 检查 → 修复断链 → 发布
```

这个流程把"检查断链"这件机械的事交给工具，人可以聚焦在内容质量本身。

### 核心特性

#### 异步并发，速度够快

AI 批量生成的文档量大，链接检查必须跟上效率。lychee 基于 Tokio 异步运行时，默认 128 并发，能在几秒内检查完一个大型代码库的所有链接。这不是工具快不快的问题，而是 AI 工作流能否闭环的问题。

#### 缓存机制

重复检查时直接读缓存，避免不必要的网络请求。缓存默认存储在 `.lycheecache`，可以用 `--max-cache-age` 控制过期时间。这个设计对 CI 环境很友好——同一个 PR 的两次检查，第二次几乎瞬间完成。

#### 多种输出格式

`--format` 支持 compact、detailed、json、markdown、raw 五种格式。json 格式方便后续脚本处理，markdown 格式适合直接嵌入 CI 报告。工具的灵活性往往体现在这些细节上。

### 为什么 AI 会捏造链接？

大语言模型的核心目标是预测"最可能的 token"，而非"正确的 token"。当它生成链接时，并不是在"检索"一个真实存在的 URL，而是在根据训练数据中的 URL 模式"拼凑"一个看起来合理的字符串。

这两种行为有本质区别：

* **知识检索：**

  在知识库中找到正确答案
* **概率拼接：**

  根据 token 序列的统计规律生成下一个 token

URL 的特殊性在于：正确答案是**极其稀疏**的。`https://api.example.com/v2/users` 和 `https://api.example.com/v3/users` 看起来同样合理，但 v2 可能从未发布过。模型学会了 URL 的表面模式，却无法验证它指向的内容是否真实存在。

更根本的矛盾在于：模型的训练目标是**流畅性**（fluency），不是**事实性**（factuality）。一篇文档即使满是无效链接，只要语法正确、读起来通顺，在训练时就不会受到惩罚。

### 总结

lychee 的价值在 AI 时代被重新定义了：它不只是"检查断链的工具"，而是文档质量闭环中的关键一环。对于 AI Agent 批量生成的文档，lychee 提供了一个自动化的链接验证层，让人肉复核可以聚焦在真正重要的事情上——内容质量本身。

**来源：**  
https://github.com/lycheeverse/lychee

---

🎉 **欢迎****加入我们的用户交流群** 🎉

在这里，你可以：  
📢 第一时间了解最新动态和活动信息  
🤝 结识同行，共享实战经验  
📚 探讨行业趋势，拓展人脉圈子

📌 **如何加入？**📷 扫描添加小助手，完成验证即可入群！

期待你的加入，一起交流成长！🚀

请点击**下方**的 阅读原文 ，将获得更详尽的介绍和相关阅读推荐。

转载请标明 **原文** 链接 ：

https://www.x-cmd.com/install/lychee