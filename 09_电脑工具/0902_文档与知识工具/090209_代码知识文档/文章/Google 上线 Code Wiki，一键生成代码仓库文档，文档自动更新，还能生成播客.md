---
title: Google 上线 Code Wiki，一键生成代码仓库文档，文档自动更新，还能生成播客
author: AI工具派
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651428561&idx=1&sn=58d269dc070793b153908d883933ed94&chksm=8ad8dfc1dcd019abbdbdc149e3f5621b7cf0d9ee4170f13efe913535f7a1bbbe3468c15ecf3f&mpshare=1&scene=24&srcid=1117t5Fi20sCaVUDDjqAy1c1&sharer_shareinfo=bc65618386ce6cfa813e3e17a3870bf3&sharer_shareinfo_first=bc65618386ce6cfa813e3e17a3870bf3#rd
---

关注 “**AI 工具派”**

探索最新 AI 工具，发现 AI 带来的无限可能性！

嗨，我是Chris，一个专注于探索各类 AI工具的博主，与大家一起发掘 AI 的潜力。我正在开发[WiseMindA](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651426924&idx=1&sn=403e903af2256c3dc18d3ec3c29fa8f1&scene=21#wechat_redirect)，期待它能成为提升学习效率的宝藏。

Chris 作为一名老程序员，经常遇到的痛点：**代码更新频繁，而文档却没有及时更新**，容易导致“接手新项目时没有文档”或“文档与代码实际逻辑不符”的问题。

往期产品还有 DeepWiki、豆包编程和 Chris 分享过的：[智谱 AI 的 Zread](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651427815&idx=1&sn=66e180b9f0e27e8d5a02e8cfc393c328&scene=21#wechat_redirect)，但还是有点不同。

今天来看看 Google 推出的 Code Wiki，可以很有效的改善这个问题。

> 🌟 工具名称：Code Wiki
>
> 🔗 工具地址：https://codewiki.google/

## 一、工具介绍 🛠️

Google Code Wiki 是谷歌新推出的一款 **AI 代码文档生成工具**，能够**自动分析 GitHub 代码仓库**，并生成结构化的 Wiki 文档，包含系统概览、模块说明、类与函数详解等。

比较特别的是，它能**自动绘制调用关系和组件关系图**，并在**代码更新时自动同步更新文档**，**确保文档始终与代码保持一致**。

与传统文档工具不同，Code Wiki **集成了 Google Gemini 智能助手**，用户可以**直接在页面上提问**，如“这个服务入口在哪？”或“这段代码会影响哪些模块？”，系统会基于深度代码分析给出精准回答，就像与一个真正懂项目的人对话一样。

## 二、快速上手 🚀

Code Wiki 使用起来也很简单，只要在首页输入框中，**输入 Github 仓库地址即可**，比如 Chris 将 gemini-cli 项目仓库的地址填取，Code Wiki 会自动找到已解析过的 Github 项目出来。

点击项目后，跳转到文档详情页面中。

Code Wiki 的详情页分为：

* **左侧目录**：即文档的目录，点击直接跳转对应章节；
* **中间文档内容**：既有 NotebookLM 同款视频播客，还有非常详细的文档介绍；
* **右侧 AI 对话**：可以与当前仓库进行 AI 对话，方便更好的了解这个项目。

## 三、核心功能 🔍

### 1.自动化更新文档

Code Wiki 的文档并不是静态文件，而是为每个仓库维护了一个持续更细的结构化的 Wiki 文档。

在扫描整个代码库后，会在**每次更改后重新生成文档**，文档会随着代码的修改而更新，**减少手动写更新文档的负担**。

### 2.自动生成 AI 播客

Code Wiki 还会根据文档内容，智能**生成一个 NotebookLM 同款的双人对话视频播客**，值得注意的是，不是每个项目都会生成视频，比如 Chris 测试的 https://codewiki.google/github.com/qwenlm/qwen 就没有生成。

也不清楚是什么情况才有。

### 3.智能 AI 文档对话

Code Wiki 提供一个**基于 Gemini 的对话接口**，你可以像聊天机器人那样问：“这个模块是干什么的？”、“某函数为什么抛错？”等。

聊天接口会利用**整个代码库知识库作为上下文**，实现**更精准的回答**，而不是泛泛的 AI 模型。帮助开发者更快定位问题、理解模块代码。

### 4.搜索作者名称

Code Wiki 也支持直接输入 Github 作者名称，搜索相关的 Github 项目，比如输入 openai，可以找到其账号下的所有仓库，还是很方便：

## 四、收费情况 💰

目前完全免费使用。

## 五、总结 📝

Code Wiki 是**谷歌推出的 AI 驱动代码文档生成工具**。它能自动分析你的 GitHub 仓库，一键生成结构化 Wiki 文档，包含系统概览、函数详解，甚至能自动绘制调用关系图。

你觉得在你的项目中，“谁写文档、谁懂代码”这个问题严重吗？如果用了像 Code Wiki 这样的工具，你最希望它解决哪一个痛点？欢迎在评论里交流！

AI 工具派交流群，扫码加入⬇️⬇️

**近期推荐**

* 👉 [2024 热门 AI 工具](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651426374&idx=1&sn=2a53f9b605a37f61124048bd7f51f42c&scene=21#wechat_redirect)
* 👉 [2023 热门 AI 工具](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651422281&idx=1&sn=2d86a38e6e6364f1342e41efb686152d&scene=21#wechat_redirect)
* 👉 [WiseMindAI：本地化 AI 智能知识库，数据本地化](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651426924&idx=1&sn=403e903af2256c3dc18d3ec3c29fa8f1&scene=21#wechat_redirect)
* 👉 [AI视频生成工具](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651424504&idx=1&sn=c0b6e6b194adaee21c5eb5fab6f854a5&scene=21#wechat_redirect)｜[AIPPT工具](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651422707&idx=1&sn=edfd33df45d39b3d65417d891b782733&scene=21#wechat_redirect)｜[AI代码编辑器](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651426229&idx=1&sn=e0a09c72fe00ba44d3da7219923b4ba0&scene=21#wechat_redirect)
* 👉 [AI文本转语音](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651424131&idx=1&sn=0b8b830b3f35b1e1e3a4d40004e07b31&scene=21#wechat_redirect)｜[AI音视频转录](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651423111&idx=1&sn=8693bb13fb6601dd5d8cf295a4e6d50c&scene=21#wechat_redirect)｜[AI翻译插件](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651424550&idx=1&sn=7c3935d4b9c70516dd976c85bf8eeb10&scene=21#wechat_redirect)
* [VideoTutor：一款 AI 教育辅助工具，一键生成 K12 动画讲解视频](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651427453&idx=1&sn=5930fac4f5f1a4358b88c30ba713a77a&scene=21#wechat_redirect)
* [PixelX：自媒体创作者、电商运营、设计师的效率神器！](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651427423&idx=1&sn=b2deb88fbd4bb02b99c2f36e6ca6d908&scene=21#wechat_redirect)
* [ListenHub：一键将文本、网页、文档生成播客音频的 AI 工具](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651427420&idx=1&sn=948f92e81df545652505c59bbce2d185&scene=21#wechat_redirect)
* [WiseMindAI 五月份新功能汇总！](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651427387&idx=1&sn=21bd24880e3f3cd50e130ed5ab431e82&scene=21#wechat_redirect)
* [Pemo：一款 AI 驱动的文档管理神器！](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651427368&idx=1&sn=9b2afb06490eb3ec345738a45e6a9dcb&scene=21#wechat_redirect)
* [ChatWise：一款本地化 AI 聊天客户端，支持多种模型](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651427340&idx=1&sn=314156a1e9c058dbced0e95e7210ceda&scene=21#wechat_redirect)
* [观猹：一个汇集 AI 好产品的社区](https://mp.weixin.qq.com/s?__biz=MzA5MjU0NzQ3Ng==&mid=2651427976&idx=1&sn=78512cd3b7cdcd8153774a959329916d&scene=21#wechat_redirect)