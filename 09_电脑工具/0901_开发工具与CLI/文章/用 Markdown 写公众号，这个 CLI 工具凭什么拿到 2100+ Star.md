---
title: 用 Markdown 写公众号，这个 CLI 工具凭什么拿到 2100+ Star
author: 极客BIM设计工坊
date: 朗朗晴空朗朗晴空
url: https://mp.weixin.qq.com/s?__biz=MzI2MjA3ODk0OQ==&mid=2648116995&idx=1&sn=dd5bddf4e7cc96a95eddb40615a0e723&chksm=f3656f72cfef9b740a3eaf5bf078fac9550de306c42fec3e1850b5eec1e3d58121dffd8ef092&mpshare=1&scene=24&srcid=0510RUvX1kkLpbWQTpJG105M&sharer_shareinfo=e730c04947016534b213455d1404042d&sharer_shareinfo_first=e730c04947016534b213455d1404042d#rd
---

# 

▌ 核心结论

md2wechat 是一个 Go 语言写的 CLI 工具，把 Markdown 直接转换成微信公众号排版并上传草稿箱。GitHub 2100+ Star，支持 40+ 主题、43 个排版模块，可接入 Claude Code / Codex / OpenClaw 等 AI 助手。免费模式够用，API 模式解锁完整能力。

## 小白先看懂：这项目是干什么的

如果你运营公众号，应该经历过这种场景：在微信编辑器里排两小时版，换个手机预览全崩。或者你在用 Typora / Obsidian 写稿，但发布时得把 Markdown 复制到编辑器里一点点调样式。

md2wechat 就是解决这个痛点的。你继续用 Markdown 写作，它负责把 Markdown 转成带排版的公众号文章，然后直接上传到你的微信草稿箱。全程命令行，不需要打开浏览器。

适合谁

长期运营公众号的创作者、技术团队的内容运营、用 AI 辅助写稿但被排版困扰的人。不适合只在手机上写作、对排版没有要求的用户。

## 一个 CLI，包揽公众号全流程

安装很简单，macOS 用户一行命令：brew install geekjourneyx/tap/md2wechat。其他平台也提供了 npm、go install、install.sh 等方式。

基本工作流：用 Markdown 写完文章，运行 md2wechat 转换，排版自动生成，上传到微信草稿箱。整个过程在终端完成，不需要打开公众号后台编辑器。

和市面其他转换工具相比，md2wechat 的差异在于：API 模式下同样 Markdown 永远输出同样排版（确定性），不是每次让 AI 重新生成。这对需要批量发布、团队协作的场景很关键。

## 43 个排版模块意味着什么

项目提供了 43 个结构化排版模块，通过 :::block hero、:::block callout、:::block timeline 这类语法在 Markdown 里直接调用。想加个醒目引用块，写一行 :::block callout 就行，不用手动调样式。

40+ 主题分为 Minimal、Focus、Elegant、Bold 四个系列，每个都在微信渲染环境里精调过。可以在官网预览效果再决定用哪个。

## 免费够用吗

项目分两种模式。AI 模式免费，由你的 Claude / Codex 处理排版，提供 3 个基础主题。适合个人创作者、偶尔发文的场景。

API 模式是付费服务，解锁完整 40+ 主题和 43 个排版模块，秒级响应，确定性输出。适合团队协作、自动化发布、对排版一致性有高要求的场景。

对大多数个人创作者来说，免费 AI 模式已经能大幅提升效率。只有当你需要批量发布、团队协作、或对排版有极致追求时，才需要考虑 API 模式。

## 值不值得试

如果你符合下面任意一条，值得花十分钟试试：你在用 Markdown 写公众号文章，每次排版耗时超过写稿时间；你在用 AI 辅助写稿，想把写和排彻底分离；你运营多个公众号，需要批量发布模板化的内容。

如果只是偶尔发一篇个人随笔，排版直接用微信编辑器默认样式就够了，不需要额外装工具。

上手建议：先 brew install，用 AI 模式跑一篇文章看看效果。觉得好用再决定要不要申请 API。项目作者是极客杰尼，GitHub 上搜 md2wechat-skill 就能找到，2100+ Star 的项目社区活跃度不错。