---
title: UI/UX Pro Max — 让 AI 帮你生成专业级 UI/UX 设计
author: 汇Ui
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5ODA4MzQ0OA==&mid=2247483786&idx=1&sn=e1572572ac4d5961197bc44b24b91702&chksm=9767f2353ea0514cad6221709e20119390a25c02053f95066d0ba78fa7361ee910650c72732a&mpshare=1&scene=24&srcid=0111kiaUqYPFRgHhTk3tUZRf&sharer_shareinfo=2a55daa4ef3c2bb3b24ecfafbe3d40e3&sharer_shareinfo_first=2a55daa4ef3c2bb3b24ecfafbe3d40e3#rd
---

哈喽，大家好，我是小新，最近大家都在忙些什么呢？

# 前言

在Ai编程越来越普及的今天，很多开发者都会遇到一个共同的问题，就是AI生成的Ui界面跟💩一样，功能是对的，但审美和平衡感总差点意思。

所以今天给大家带来一个很有意思的项目 —— UI/UX Pro Max。

# 什么是 UI/UX Pro Max？

UI/UX Pro Max是一个开放源码的设计智能技能（Skill），其实本质上就是一个可搜索的设计知识库。它是一个把设计规范作为结构化决策依据，让 AI 生成的前端代码更符合 UI/UX 的标准和最佳实践。

# 安装与使用教程

1.全局安装 CLI 工具

```
npm install -g uipro-cli
```

2.为指定 AI 助手安装技能

```
uipro init --ai claude      # Claude Codeuipro init --ai cursor      # Cursoruipro init --ai windsurf    # Windsurfuipro init --ai antigravity # Antigravity (.agent + .shared)uipro init --ai copilot     # GitHub Copilotuipro init --ai kiro        # Kirouipro init --ai all         # 所有 AI 助手
```

那么这里小新以kiro为例，通过 uipro init --ai kiro 将项目添加到我的工作目录中。

接着通过/ui-ux-pro-max命令来创建一个项目

最后我们来看一下kiro生成的页面效果图

# 总结：它值不值得用？

如果你希望 AI 不只是“生成功能代码”，而是真正能产出看起来像设计师写的 UI 界面代码——UI/UX Pro Max 是一个非常值得尝试的工具。尤其是对于那些缺乏设计背景的开发者来说，它能节省大量时间，同时提升产品交付质量。