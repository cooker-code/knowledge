---
title: OpenSpec：AI辅助开发的规范驱动框架
author: JZ AI与云计算
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg4MDA2NzUzNQ==&mid=2247484199&idx=1&sn=ae5f18135ee9bef2f4633a9f4f32a6fd&chksm=ce2bb2e6a6ad0f6cdd4ab12a4564595f2a902cdaeb9626e125eb6426bcc434204289b410cf6b&mpshare=1&scene=24&srcid=0206ic0ovX7xKofPc8wWFkJh&sharer_shareinfo=57494b01dd50883e4659e0cbe53ba51b&sharer_shareinfo_first=57494b01dd50883e4659e0cbe53ba51b#rd
---

## 项目简介

OpenSpec 是一个革命性的工具，让你和 AI 编程助手在编写代码前就达成共识。它通过规范驱动的方式，确保开发过程清晰、有据可循。✨

**核心特性：**

* 📋 规范先行：先定义需求，再动手编码
* 🔄 增量更新：使用 Delta Specs 管理变更
* 🤝 人机协作：AI 助手参与整个开发流程
* 📁 结构化管理：自动生成项目结构和文档

## 架构概览

OpenSpec 的工作流程遵循简单而强大的模式：

### 核心概念

**制品（Artifacts）** 是 OpenSpec 的核心组件：

* `proposal.md`

  ：为什么要做这个变更
* `specs/`

  ：增量规范（Delta Specs）
* `design.md`

  ：技术实现方案
* `tasks.md`

  ：实施清单

**Delta Specs** 使用特殊格式标记变更类型：

* `## ADDED Requirements`

  ：新增需求
* `## MODIFIED Requirements`

  ：修改需求
* `## REMOVED Requirements`

  ：删除需求

## 快速开始

### 安装 OpenSpec

首先安装 OpenSpec CLI：

```
# 从源码安装  
git clone https://github.com/Fission-AI/OpenSpec.git  
cd OpenSpec  
pip install-e.  
  
# 或使用 pip  
pip install openspec
```

### 初始化项目

在你的项目根目录运行：

```
openspec init
```

这会在项目中创建 `openspec/` 目录结构：

```
openspec/  
├── specs/              # 源规范  
│   └── <domain>/  
│       └── spec.md  
├── changes/            # 变更提案  
│   └── <change-name>/  
│       ├── proposal.md  
│       ├── design.md  
│       ├── tasks.md  
│       └── specs/      # 增量规范  
└── config.yaml         # 配置（可选）
```

## 使用示例

让我们通过一个实际例子——为应用添加暗色模式，来展示 OpenSpec 的完整流程。

### 1. 开始变更

```
/opsx:new add-dark-mode
```

AI 助手会创建变更目录并准备生成提案。

### 2. 创建制品

使用快速前进命令生成所有制品：

```
/opsx:ff
```

这会创建：

* `proposal.md`

  ：变更意图和范围
* `specs/ui/spec.md`

  ：UI 相关的增量规范
* `design.md`

  ：技术实现方案
* `tasks.md`

  ：实施任务清单

**生成的制品示例：**

*proposal.md* - 捕获意图：

```
# Proposal: Add Dark Mode  
  
## Intent  
Users have requested a dark mode option to reduce eye strain  
during nighttime usage.  
  
## Scope  
- Add theme toggle in settings  
- Support system preference detection  
- Persist preference in localStorage  
  
## Approach  
Use CSS custom properties for theming with a React context  
for state management.
```

*specs/ui/spec.md* - 增量规范：

```
# Delta for UI  
  
## ADDED Requirements  
  
### Requirement: Theme Selection  
The system SHALL allow users to choose between light and dark themes.  
  
#### Scenario: Manual toggle  
- GIVEN a user on any page  
- WHEN the user clicks the theme toggle  
- THEN the theme switches immediately  
- AND the preference persists across sessions  
  
#### Scenario: System preference  
- GIVEN a user with no saved preference  
- WHEN the application loads  
- THEN the system's preferred color scheme is used
```

*tasks.md* - 实施清单：

```
# Tasks  
  
## 1. Theme Infrastructure  
- [ ] 1.1 Create ThemeContext with light/dark state  
- [ ] 1.2 Add CSS custom properties for colors  
- [ ] 1.3 Implement localStorage persistence  
  
## 2. UI Components  
- [ ] 2.1 Create ThemeToggle component  
- [ ] 2.2 Add toggle to settings page  
- [ ] 2.3 Update Header to include quick toggle  
  
## 3. Styling  
- [ ] 3.1 Define dark theme color palette  
- [ ] 3.2 Update components to use CSS variables
```

### 3. 实现任务

```
/opsx:apply
```

AI 助手会逐个完成任务清单中的项目，包括：

* 创建主题上下文
* 添加 CSS 自定义属性
* 实现本地存储持久化
* 创建主题切换组件

### 4. 归档合并

```
/opsx:archive
```

变更完成！规范会合并到主 specs 目录，变更文件夹移至归档。

### 5. 验证和审查

使用 CLI 命令检查变更状态：

```
# 列出活跃变更  
openspec list  
  
# 查看变更详情  
openspec show add-dark-mode  
  
# 验证规范格式  
openspec validate add-dark-mode  
  
# 交互式仪表板  
openspec view
```

## 配置

OpenSpec 支持通过 `config.yaml` 自定义行为：

```
# 项目配置示例  
domains:  
- auth  
- ui  
- api  
  
templates:  
proposal: custom_proposal_template.md  
spec: custom_spec_template.md
```

## 常见问题

OpenSpec 支持通过 `config.yaml` 自定义行为：

```
# 项目配置示例  
domains:  
- auth  
- ui  
- api  
  
templates:  
proposal: custom_proposal_template.md  
spec: custom_spec_template.md
```

## 常见问题

**Q: 如何处理规范冲突？**  
A: 使用 `/opsx:validate` 检查格式，然后手动调整冲突的规范。

**Q: 可以跳过某些制品吗？**  
A: 不推荐，但对于简单变更可以直接从设计开始。

**Q: 如何与现有项目集成？**  
A: 运行 `openspec init` 后，手动迁移现有文档到 specs 目录。

## 总结

OpenSpec 重新定义了 AI 辅助开发的流程，让规范成为开发的基石。通过结构化的制品管理，确保每一次变更都有据可循，大大提升开发效率和代码质量。

**关键优势：**

* 🔍 需求明确，避免返工
* 📝 文档自动生成
* 🤖 AI 深度参与
* 📊 变更可追溯