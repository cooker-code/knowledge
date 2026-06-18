---
title: Spec Kit：GitHub 官方出品，规范即代码
author: 赛博虾条
date: 1RMF1RMF
url: https://mp.weixin.qq.com/s?__biz=MzA3ODkxMTI5Mg==&mid=2247483848&idx=1&sn=a46965ba9c6de28ca39bc5bd2ab0766f&chksm=9e62126c885de9c88ae09fa02c9e271b60e1863785ff3437b5d27338bba40200e4df9cdec2c2&mpshare=1&scene=24&srcid=0531utwJCAte2oJiIG77j65Q&sharer_shareinfo=f7d5382886a941846a34aa85e3e8efe0&sharer_shareinfo_first=f7d5382886a941846a34aa85e3e8efe0#rd
---

前面聊了两款规范驱动开发工具——Superpowers 重纪律，OpenSpec 重灵活。这次聊一个有"官方背景"的：

**Spec Kit，GitHub 自己做的。**

---

## 它是什么？

Spec Kit 是 GitHub 开源的规范驱动开发工具包。它的核心理念一句话就能说清：

**规范不是代码的附属品，规范才是主体，代码只是规范的产出物。**

这不是我瞎总结的。Spec Kit 的官方文档里写得很直白：

> 几十年来，代码一直是王。规范服务于代码——是我们搭的脚手架，等"真正的"代码写完就拆掉。规范驱动开发（SDD）颠覆了这个权力结构。规范不服务代码，代码服务规范。

说大白话就是：PRD 不是写给开发看的参考文档，它是**直接生成代码的源头**。你维护的是规范，代码只是规范的"编译结果"。

这个思路很大胆。

---

## 一个关键词：Constitution（宪法）

Spec Kit 有一个别的框架没有的东西——**项目宪法**。

你用 `/speckit.constitution` 命令创建一份宪法，里面写的是项目的基本原则：代码质量标准、测试要求、用户体验一致性、性能要求……所有后续开发都要遵守这些原则。

它像什么？像一个国家的宪法。后续的每个功能规范、每个实施计划，都不能和宪法冲突。

这是个很"企业级"的设计。Superpowers 的纪律是"先设计再实现"，OpenSpec 的纪律是"先对齐规范"，Spec Kit 的纪律是"先定原则，再按原则做一切"。

---

## 五步工作流

Spec Kit 的标准流程是五步：

**第一步：定宪法**（constitution）

```
  /speckit.constitution 创建以代码质量、测试标准、用户体验一致性为核心的项目原则
```

**第二步：写需求**（specify）

```
  /speckit.specify 做一个照片相册应用，按日期分组，支持拖拽排序，照片用瓦片式预览
```

注意这个命令只关注"要什么"和"为什么"，不说"怎么做"。Spec Kit 的模板会强制把"做什么"和"怎么做"分开。

**第三步：做计划**（plan）

```
  /speckit.plan 用 Vite 构建，尽量用原生 HTML/CSS/JavaScript，图片不上传，元数据存本地 SQLite
```

这一步才涉及技术选型。AI 会分析需求，生成实施方案、数据模型、API 合约、测试场景，还有一个快速验证指南。

**第四步：拆任务**（tasks）

```
  /speckit.tasks
```

AI 把计划拆成可执行的任务列表。独立任务标记为 `[P]`，可以并行执行。

**第五步：实现**（implement）

```
  /speckit.implement
```

AI 按任务列表逐项实现。

这五步看起来和 OpenSpec 的 propose → apply → archive 差不多？表面上是。但 Spec Kit 更重——每一步都有模板约束、一致性检查、宪法合规验证。

---

## 15 分钟 vs 12 小时

Spec Kit 官方给了一个对比：

**传统开发流程：**

* • 写 PRD：2-3 小时
* • 写设计文档：2-3 小时
* • 手动搭项目结构：30 分钟
* • 写技术规范：3-4 小时
* • 写测试计划：2 小时
* • **总计：约 12 小时的文档工作**

**Spec Kit 流程：**

* • /speckit.specify：5 分钟
* • /speckit.plan：5 分钟
* • /speckit.tasks：5 分钟
* • **总计：15 分钟**

15 分钟内你得到：完整的功能规范（含用户故事和验收标准）、详细的实施计划（含技术选型和理由）、API 合约和数据模型、测试场景、版本化在功能分支里的所有文档。

这个对比当然是理想情况，但方向是对的——AI 把文档工作从"人写"变成"人审"。

---

## 支持多少 AI 工具？

30+。是目前三个框架里最多的。

Claude Code、Cursor、GitHub Copilot、Codex CLI、Gemini CLI、Windsurf、Roo Code、Devin、JetBrains Junie、阿里通义灵码（Lingma）、Kimi Code、Trae……基本你能想到的都支持。

而且它还支持多工具共存——同一个项目里，你可以同时装 Claude Code 和 Cursor 的集成，团队成员各用各的工具。这在企业场景里很实用。

---

## 社区生态

Spec Kit 有一个社区扩展（Extensions）机制。社区可以贡献新的命令、钩子和能力。也有社区预设（Presets），可以自定义 Spec Kit 的行为模板和术语。

这和 OpenSpec 的第三方 Schema Bundle 类似，但 Spec Kit 背靠 GitHub，社区规模和曝光度天然占优势。

---

## 和 OpenSpec 的区别

两者理念相近，但风格不同：

**OpenSpec** 的 Delta Spec 只关注变化——加了什么、改了什么、删了什么。轻、迭代、不搞瀑布流。适合"我在已有项目上改改东西"。

**Spec Kit** 的规范是完整的——从宪法到需求到计划到任务，每一步都有严格的模板和验证。适合"我要从零开始建一个系统，每一步都要可追溯"。

**OpenSpec 更轻更灵活，Spec Kit 更重更严谨。**

---

## 谁适合用？

**适合：**

* • GitHub 生态用户（Copilot、Actions、Issues 都能联动）
* • 企业团队，需要严格的过程追溯
* • 从零开始的大项目，需要完整的规范体系
* • 团队成员使用不同 AI 工具、但需要统一流程的场景

**可能不太适合：**

* • 个人开发者的小项目（太重）
* • 只想快速迭代、不想写规范的人
* • 不在 GitHub 生态里的团队

---

**🔗 项目地址：** https://github.com/github/spec-kit

**📖 文档站：** https://github.github.io/spec-kit/

**💡 下期预告：** 《Superpowers vs OpenSpec vs Spec Kit：该选哪个？》——三个框架聊完了，最后一篇来个硬碰硬的对比。