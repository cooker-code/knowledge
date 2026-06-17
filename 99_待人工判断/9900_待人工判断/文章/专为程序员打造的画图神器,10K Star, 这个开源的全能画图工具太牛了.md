---
title: 专为程序员打造的画图神器,10K Star, 这个开源的全能画图工具太牛了
author: 韩数同学
date: 韩数韩数
url: https://mp.weixin.qq.com/s?__biz=Mzg3ODIwNDg5OA==&mid=2247487036&idx=1&sn=3cb3b0676d1abf619a6b042b178e9123&chksm=ce59123dfdc98c9af633098008834cf7e02b98fea727add2afdfda2fb450d54fe246c1d10ae0&mpshare=1&scene=24&srcid=0605EXFCMgUc2xmFdB2EYLcv&sharer_shareinfo=bf150f168cc3ae7f376d6725f257bf62&sharer_shareinfo_first=bf150f168cc3ae7f376d6725f257bf62#rd
---

大家好，我是韩数同学。致力于分享 Github 上那些  好玩  有趣  免费  实用  的高质量项目。我们致力于发现和体验那些好玩的开源项目, 也愿意和你一起去探索那些开源项目背后的故事。

#### 正文

2014 年, 一个 Knut Sveidqvist 的瑞典工程师不小心丢失了自己画了很久的 Microsoft Visio 文件,  这意味着他辛辛苦苦画的架构图都需要重画一遍了，我经历过这样的时刻，确实很难受。

那个时候 Makrdown 才刚刚开始流行，于是他诞生了一个想法,  能不能把图表也用纯文本的方式描述，就像 markdown 一样,  写的时候只需要遵循简单的语法，渲染的时候可以把文本渲染成图表出来？

他希望语法足够简单，不需要查文档就能记住，一个流程图可以用简单的语法就能描述。

2014 年他选择把这个项目开源出来，MIT 协议，任何人都可以使用，这个开源项目就是 Mermaid

不过 Mermaid 刚开始并没有什么热度,  2022 年,  GitHub 宣布原生支持 Mermaid 渲染, 到现在包括 Notion, Obsidian 等这些工具都已经内置了 Mermaid,  不需要任何插件，直接在笔记里面写 ```` ```mermaid ```` 代码块就能渲染出图表。

到今天 Mermaid 已经在 Github 上收获了 88.3K 的 star 数,  基本上大多数程序员都有用过它。Mermaid 确实很方便，但是一直以来都有几个问题,  那就是功能虽然强大，但是画出来的图不是很好看，而且在终端里面几乎没法用。

今年一月份, 有一个叫 beautiful-mermaid 的项目开源了，并在一天内收获了 2.2k 的 star 数。

beautiful-mermaid  做的事情很简单，它不改 Mermaid 语法，只改长相，你的语法不需要做任何改动，但是渲染出来则变成了更干净，更现代的流程图，而且还可以支持把图表渲染为 SVG 或者 ASCII 在终端展示。

而且默认内置了包括流程图, 状态图、序列图、类图、ER 图 等六种图表类型，以及 15 套内置主题,  最关键的是，beautiful-mermaid 使用 TypeScript 开发，零 DOM 依赖，图表渲染也很快，不到 500 毫秒就可以渲染 100+ 图表。

相较于 Mermaid,  beautiful-mermaid 渲染的图确实好看了许多。

图表类型,  确实看起来更现代化了。

当然我觉得 beautiful-mermaid 这个库的意义还不在于此，毕竟现在程序员文档都不愿意手写了，更别说去写 mermaid 语法了。但是 AI 很适合干这个事情，beautiful-mermaid  也提供了在线的渲染器，可以直接渲染，支持调整颜色，字体和背景色，而且可以导出为图片，如果需要配图的话，还是挺方便的。

唯一不足的地方是，beautiful-mermaid 刚开源几个月,  还有一些图表类型没有支持到，例如饼图这些，而且obsidian 这些工具还不支持用 beautiful-mermaid 渲染。不过 beautiful-mermaid 是一个前端库，也可以自己设计主题，集成起来也很方便，现在 AI 编程这么厉害，其实完全可以基于 beautiful-mermaid 做一个自己用着趁手的画图工具。目前 beautiful-mermaid  已经在 github 收获了 10.2k 的 star 数，而且还在持续维护中~

开源地址:

https://github.com/lukilabs/beautiful-mermaid

项目评价

|  |
| --- |
| 结合 实用性  上手门槛  有趣好玩  三个维度  本期给到 beautiful-mermaid 的评分为: 🌟🌟🌟🌟 |

往期文章

|  |
| --- |
| [为了能卸载掉 IDE，我花一个月开源了一个轻量级 AI 编程工具](https://mp.weixin.qq.com/s?__biz=Mzg3ODIwNDg5OA==&mid=2247486776&idx=1&sn=c9ea1f53017d168f9008ce2c004bd501&scene=21#wechat_redirect)  [Github 历史最快登顶的项目, 也是凉的最快的..](https://mp.weixin.qq.com/s?__biz=Mzg3ODIwNDg5OA==&mid=2247487009&idx=1&sn=ac3e6409feb74f4baabb9f5e013023fd&scene=21#wechat_redirect)  [比 grep 快 10 倍! 64K Star, 这个用 Rust 开发的命令行搜索神器太香了](https://mp.weixin.qq.com/s?__biz=Mzg3ODIwNDg5OA==&mid=2247486998&idx=1&sn=9b8533eb4a4e2badca9a3bdf3e84e8a0&scene=21#wechat_redirect)  [68K Star, 跨平台可自托管, 近五年 Notion 最佳开源平替](https://mp.weixin.qq.com/s?__biz=Mzg3ODIwNDg5OA==&mid=2247486985&idx=1&sn=32e31cc0d0e5a28cf68197f41e979af7&scene=21#wechat_redirect)  [78k Star! 这个开源的 Git 效率神器, 让代码管理效率提升 100%](https://mp.weixin.qq.com/s?__biz=Mzg3ODIwNDg5OA==&mid=2247486972&idx=1&sn=05b5769b2f1b73d52c4db9883a566ccd&scene=21#wechat_redirect) |

历史开源项目

韩数同学历史发布过很多有趣的开源项目，如果你懒得翻文章一个个找，你可以关注韩数同学。 我们整理了一个合集，希望可以帮助到你。

我是韩数同学, 致力于发现和体验那些好玩的开源项目, 也愿意去探索和分享那些开源项目背后的故事，我们下期再见～