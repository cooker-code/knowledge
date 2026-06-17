---
title: 开源发布 - Skill-Lib AI Agent 技能库
author: 橘子冰的技术栈
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIwMDk0ODI3Nw==&mid=2247483936&idx=1&sn=b31bfd1a262669ca0a40905506c0081d&chksm=97209a38657d68c8c9028554ab538762270a1b0ff369956a1f4b086d4f84b1f4ecea25b2d95b&mpshare=1&scene=24&srcid=0310h0K2umKO6bHlShjE6JjC&sharer_shareinfo=946654e00ce528f80eb9312aee29996e&sharer_shareinfo_first=946654e00ce528f80eb9312aee29996e#rd
---

## 前言

大家好呀，我是橘子冰，很长时间没有发微信公众号了。

上周六到现在我一直在体验OpenClaw这个项目，它的出现颠覆了我过往的开发工作流，只是听我描述你可以无法感知到这种颠覆性，因此我建议看到这篇文章的技术开发人员都应该将 OpenClaw投入你的工作流当中。

为了更好地优化工作流，我将最近打包的Skill封装了一个Github仓库，下面我将介绍这个仓库。

## 什么是 Skill-Lib？

Skill-Lib 是一个面向 AI Agent 的开源技能仓库，收集整理了可复用的标准化工作流程和最佳实践。每个 Skill 都经过实际项目验证，可以直接应用到你的 Agent 中，帮助 Agent 开发者避免重复造轮子，专注于创造价值。

🌐 **项目地址**: https://github.com/Dqz00116/skill-lib

## 为什么创建 Skill-Lib？

随着 AI Agent 的普及，我们发现许多 Agent 在重复解决相同的问题：

* 如何系统性地分析代码？
* 如何根据设计文档生成代码？
* 如何高效地管理知识库？
* 如何安全地提交代码？

**Skill-Lib** 通过提供以下特性来解决这些挑战：

* ✅ **开箱即用的工作流程** - 预构建、经过验证的解决方案
* ✅ **最佳实践** - 从实际项目中总结的经验教训
* ✅ **标准化格式** - 易于理解和应用
* ✅ **社区驱动** - 开放贡献，共同成长

## 现有 Skills

目前 Skill-Lib 包含 **7 个精心设计的 Skills**：

| Skill | 描述 | 适用场景 |
| --- | --- | --- |
| **code-analysis** | 4步结构化代码分析流程 | 理解新代码、架构评审 |
| **code-generator** | 分阶段代码生成 | 根据设计文档生成代码 |
| **daily-log** | 结构化每日操作日志 | 工作记录、知识沉淀 |
| **git-workflow** | 安全的 Git 提交流程 | 代码提交、版本控制 |
| **knowledge-base-cache** | 三层知识库管理 | 大规模知识、降低成本 |
| **msvc-build** | MSVC C++ 编译指南 | 编译项目、排查错误 |
| **mvp-design** | MVP 设计规范 | 快速原型、架构设计 |

## 快速开始

### 使用 Skill

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# 克隆仓库git clone https://github.com/Dqz00116/skill-lib.git  
# 阅读 Skill 文档cat skill-lib/code-analysis/SKILL.md  
# 应用到你的 Agent# Agent 读取 Skill 并按照工作流程执行
```

### 安装到本地工作空间

详细安装指南请参考 INSTALL.md，支持一键安装和手动安装。

## 欢迎提交你的 Skill

我们热忱欢迎社区贡献！无论你是 AI Agent 开发者、工作流程设计师，还是有好点子的人，我们都希望你的 Skill 能加入我们的库。

### 为什么贡献？

* 🌍 **帮助他人** - 与社区分享你验证过的工作流程
* 📝 **知识沉淀** - 以结构化格式保存你的专业知识
* 🔄 **获得反馈** - 通过社区反馈改进你的工作流程
* 🏆 **建立声誉** - 向 AI 社区展示你的专业能力

### 如何贡献

1. **创建你的 Skill**按照模板创建 `SKILL.md` 文件，包含：

* 使用场景
* 前置条件
* 详细工作流程
* 最佳实践

2. **遵循规范**

* ✅ 脱敏处理（移除个人信息）
* ✅ 命名规范：小写+连字符，如 `code-analysis`
* ✅ 包含清晰的示例
* ✅ 经过实际项目验证

3. **提交你的 Skill**

* Fork 仓库
* 添加 Skill 到 `skill-name/SKILL.md`
* 创建 Pull Request

### 征集的 Skill 类型

我们特别希望获得以下类型的 Skills：

* 数据库设计和迁移
* API 开发工作流程
* 测试策略
* 文档生成
* 部署自动化
* 代码审查流程
* 性能优化

## 加入社区

Skill-Lib 不仅是一个仓库，更是一个 AI 从业者分享知识和最佳实践的社区。

⭐ **在 GitHub 上 Star 我们**: https://github.com/Dqz00116/skill-lib

🤝 **贡献代码**: 我们欢迎 Pull Requests！

💬 **分享传播**: 告诉其他 AI Agent 开发者

## 未来路线图

* [ ] 更多语言支持（目前已支持中英文）
* [ ] Skill 验证工具
* [ ] 社区投票精选 Skill
* [ ] 流行 Agent 框架集成指南
* [ ] 复杂 Skill 的视频教程

## 许可证

Skill-Lib 基于 **MIT License** 开源 - 可免费用于个人和商业用途。

---

**由 Agents 构建，为 Agents 服务 🤖**

*让我们一起让 AI Agents 更智能！*