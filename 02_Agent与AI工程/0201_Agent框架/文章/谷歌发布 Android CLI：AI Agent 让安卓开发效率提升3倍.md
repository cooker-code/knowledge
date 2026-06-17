---
title: 谷歌发布 Android CLI：AI Agent 让安卓开发效率提升3倍
author: 技术不倒翁
date: 
url: https://mp.weixin.qq.com/s?__biz=MzY5MjE2OTU5Mg==&mid=2247484199&idx=1&sn=49f59299eb05f82b2bef759154daf0e8&chksm=f5241ae029e0a301a9793c74725296b5ede103fb55bc310a1a32efc79a7c96ed3736e6451e4a&mpshare=1&scene=24&srcid=0419KrC9P5tZb1Gh1fddh9Dm&sharer_shareinfo=9bbe2a137dc1c930bf982ce40408477f&sharer_shareinfo_first=9bbe2a137dc1c930bf982ce40408477f#rd
---

GOOGLE · ANDROID CLI · 2026

# 谷歌发布 Android CLI：AI Agent 让安卓开发效率提升 3 倍

支持任意 AI 编程工具 · Skills 机制让模型更懂 Android · 开发者的新生产力范式

2026 年 4 月 17 日，Google 正式发布了面向 AI Agent 时代的 Android 开发工具套件——**Android CLI**，配套推出了 **Android Skills 官方技能库**和 **Android Knowledge Base**。这套工具的核心使命只有一个：无论你用哪款 AI 编程助手，都能让它真正懂 Android、写对代码、跑起来。

## 一、Android CLI 是什么？

简单说：**Android CLI 是为 AI Agent 量身打造的命令行工具**。过去，AI Agent 在做 Android 项目创建、SDK 管理、设备调试时，往往要靠"猜"——因为它只有文档，没有精准的操作接口。Android CLI 的出现，直接把这条路打通了。

▲ Android CLI 终端界面（官方截图）

Google 内部实验数据显示，使用 Android CLI 之后：

📉
   LLM Token 消耗降低 **70% 以上**（环境初始化和项目创建阶段）

⚡
   任务完成速度 **提升 3 倍**（vs. 仅靠标准工具集的 Agent）

🤝
   **兼容所有主流 AI 工具**：Gemini、Claude Code、Codex、Antigravity……

## 二、四大核心命令，覆盖开发全流程

SDK 管理
     `android sdk install`

按需下载 SDK 组件，告别"全量安装"，保持精简的开发环境。

项目创建
     `android create`

基于官方模板秒速生成项目，自动应用推荐架构和最佳实践，第一行代码就走在正确的路上。

设备管理
     `android emulator`

快速创建和管理虚拟设备，配合 `android run` 一键部署，Agent 可以自主完成整个 UI 导航测试。

自动更新
     `android update`

随时拉取最新功能，工具链始终保持在最新状态。

## 三、Android Skills：让 AI 真正懂 Android 的秘密武器

这是整套工具里最值得关注的设计理念。传统文档是写给人看的，概念多、层级深、语言模糊——LLM 读了不一定知道该怎么操作。**Android Skills 是写给 AI 看的文档**，基于开放标准的 SKILL.md 格式，每个 Skill 都是一份精准的任务执行规范。

📦 首批开放的 Android Skills 包括：

✓
   **Navigation 3 设置与迁移** — 新一代导航库的正确用法和升级路径

✓
   **Edge-to-Edge 全面屏支持** — 正确处理系统栏、手势区域的完整方案

✓
   **AGP 9 升级与 XML→Compose 迁移** — 最新 Gradle 插件适配和 UI 现代化

✓
   **R8 配置分析** — 代码混淆、缩减规则的最佳实践

安装方式极简，一条命令搞定：

`android skills add --all
# 或者只安装特定技能
android skills add --skill=navigation3
android skills add --skill=compose-migration`

Skills 会自动安装到 Gemini、Antigravity 等 Agent 的目录，Claude Code 和 Codex 等第三方工具也完全兼容。更重要的是，这个仓库开放给社区贡献——未来开发者可以提交自己的 Skill，形成一个持续生长的 Android AI 知识生态。

## 四、Android Knowledge Base：解决"模型知识过期"难题

大模型有一个根本性痛点：训练数据有截止日期，而 Android 生态每年都在高速演进。一个训练截止于2024年底的模型，可能完全不知道2025年新出的 API 和最佳实践。

Android Knowledge Base 的解法：

 通过 `android docs` 命令，Agent 可以实时检索并拉取来自 **developer.android.com、Firebase 文档、Google Developers、Kotlin 官方文档**的最新内容作为上下文。不管模型训练时间多久，它拿到的都是当下最新的官方指南。

这套机制已经内置到最新版 Android Studio，无需额外配置。命令行开发者通过 `android docs` 同样可以访问。

## 五、这对 Android 开发者意味着什么？

**一个重要的信号：**Google 在主动解决"AI 写的 Android 代码质量不可靠"的问题，不是靠训练更大的模型，而是通过工具层和知识层的精准供给来提升 AI 的输出质量。

对不同阶段的开发者来说，这套工具的价值各有侧重：

初学者
   一条 `android create` 命令就能得到一个架构正确的项目骨架，不用担心踩坑。

资深开发者
   把繁琐的环境配置、设备管理、SDK 更新交给 CLI + Agent，精力聚焦在业务逻辑。

跨平台团队
   可以用任意 AI 工具快速原型，随时切换到 Android Studio 做精调和发布，工具链之间无缝衔接。

📌 总结

 Android CLI + Android Skills + Android Knowledge Base，三件套联手解决了 AI Agent 做安卓开发时的三大痛点：**操作接口混乱**、**最佳实践缺失**、**文档知识过期**。

 这不只是一套开发工具，更是 Google 给整个 AI 辅助编程行业提供的一套范本：**与其期待 LLM 自己学会每个垂直领域，不如直接给它一把经过精心打磨的专属工具。**

🚀 立即上手

→
   官方下载页：**d.android.com/tools/agents**

→
   Android Skills GitHub：**github.com/android/skills**

→
   Android Skills 文档：**developer.android.com/tools/agents/android-skills**

来源：Android Developers Blog · GitHub android/skills  
   整理日期：2026年4月17日