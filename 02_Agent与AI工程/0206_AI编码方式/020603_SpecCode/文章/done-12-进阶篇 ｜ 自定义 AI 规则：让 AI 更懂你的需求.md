> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020603_SpecCode/020603_核心知识点/SpecCode规格驱动与验收边界|SpecCode规格驱动与验收边界]]
---
title: 12-进阶篇 ｜  自定义 AI 规则：让 AI 更懂你的需求
author: 爱学习的蝌蚪
date:
url: https://mp.weixin.qq.com/s?__biz=MzkyODM0MTI0MA==&mid=2247493440&idx=1&sn=5b7aa5492256a31ab14eb7bc634398a5&chksm=c39003f23086ca825a18e5bef245f2e7cd5b76d286161864d33d7668c9e97ee17512d5164018&mpshare=1&scene=24&srcid=12109Psk3akgsQxkd4JgnwyK&sharer_shareinfo=6a17d552b79009791000f7fe7721cd05&sharer_shareinfo_first=6a17d552b79009791000f7fe7721cd05#rd
---

本文介绍如何设置 Cursor 的 AI 规则。这些规则可以让 AI 更好地理解你的工作方式。通过简单的设置，你可以让 AI 助手提供更有用的帮助。

## 什么是 AI 规则？

AI 规则是 Cursor 的一个功能。你可以用它来控制 AI 的行为。这些规则会在每次对话时自动使用。这样 AI 就能按照你想要的方式工作。

主要用途有：

* 设定编码风格
* 定制文档格式
* 统一团队流程（团队协作特别适用）
* 个性化代码审查

## 三种规则设置方式

Cursor 目前支持三种规则配置方式，项目级规则优先级最高，推荐使用项目级规则，下文会详细介绍。

* 项目级规则（推荐，0.45 版本开始有）
* 全局规则
* 规则文件（将来会停用，不推荐）

image.png

## 项目级规则（推荐）

你需要在项目根目录的 `.cursor/rules` 文件夹中创建 `.mdc` 文件。mdc 是 Cursor 的规则文件格式。它基于 Markdown 格式。它支持四种规则类型:

1. Always：自动附加到所有聊天和命令请求（command + k），适用于全局规则。
2. Auto Attached：基于文件模式匹配自动触发，例如: .tsx, .json, Test.cpp。
3. Agent Requested：针对特定任务场景，需要提供任务描述。帮助 AI Agent 更好地理解和执行特定任务
4. Manual：需要手动提及才会被包含，适用于特殊场景的临时规则。

image.png

支持的功能：

* **规则内容**：用 Markdown 格式写规则
* **规则引用**：用 @file 引用其他规则文件

创建方法一：

1. 打开命令面板（⌘+⇧+P）
2. 选择 "New Cursor Rule"

 3. 输入规则文件名，如 `react-components`

image.png

4. 在打开的 `.mdc` 文件中写规则

image.png

创建方法二：

1. 在 Cursor 设置中，找到 Rules 选项，点击 "+ Add new rule" 按钮

image.png

## 全局规则

全局规则对所有项目都有效。设置步骤：

1. 打开 Cursor 设置
2. 进入 "Rules" → "User Rules"
3. 添加全局规则（如语言设置）

如下所示，期望所有响应使用中文。

image.png

## 迁移指南（从 .cursorrules 迁移）

为了兼容性，旧版使用的 `.cursorrules` 文件，目前仍被支持，但 Cursor 官方文档明确提到 `.cursorrules` 文件将来会移除。

建议迁移到新系统，迁移步骤如下：

1. 将现有规则按功能拆分为多个文件
2. 在 `.cursor/rules` 目录创建对应规则
3. 在 Globs 字段指定适用范围
4. 通过 @file 引用保持规则链式调用

## 项目示例

下面是一个 React + TypeScript 项目的规则设置例子。我们来看看如何设置这些规则。

### 基础规则

```
# 项目基础开发规范

## 基础开发规则
- 使用 ESLint 和 Prettier 进行代码格式化
- 文件命名采用 kebab-case
- 导入语句按类型分组并排序
- 避免魔法数字，使用常量定义
- 函数长度不超过 50 行

## 项目目录结构

├── .cursor/
│   └── rules/
│       ├── base.mdc           # 基础规则
│       ├── react.mdc          # React 相关规则
│       ├── typescript.mdc     # TypeScript 规则
│       ├── testing.mdc        # 测试规则
│       └── docs.mdc           # 文档规则
└── src/
    ├── components/
    ├── features/
    └── tests/
```

Cursor 编辑器是这样的：

image.png

### React 开发规则

```
Description:
React 组件开发规范，确保组件的可维护性和性能

Globs:
src/components/**/*.tsx, src/features/**/*.tsx

# React 组件开发规范
- 优先使用函数组件和 Hooks
- 组件文件结构：
  1. 类型定义
  2. 常量声明
  3. 组件实现
  4. 样式定义
- Props 必须使用 interface 定义类型
- 使用 React.memo() 优化渲染性能
- 样式使用 CSS Modules 或 styled-components
- @base.mdc
```

Cursor 编辑器是这样的：

image.png

下面的配置也是基于这个编辑器来配置的，大家也可以参考上面编辑器配置格式直接复制文档内容来配置自己的规则。

### TypeScript 规则

```
Description:
TypeScript 开发规范，确保类型安全和代码质量

Globs:
**/*.ts, **/*.tsx

# TypeScript 规则
- 禁用 any 类型，使用 unknown 替代
- 启用 strict 模式
- 使用类型推导减少冗余类型声明
- 公共函数必须包含 JSDoc 注释
- 使用 type 而不是 interface（除了 Props）
```

### 测试规则

```
Description:
单元测试和集成测试规范

Globs:
src/**/*.test.ts, src/**/*.test.tsx

# 测试规则
- 使用 React Testing Library
- 测试文件与源文件同目录
- 测试用例结构：
  1. 准备测试数据
  2. 执行被测试代码
  3. 断言结果
- Mock 外部依赖使用 MSW
- 测试覆盖率要求：
  * 语句覆盖率 > 80%
  * 分支覆盖率 > 70%
  * 函数覆盖率 > 90%
```

### 文档规则

```
Description:
项目文档编写规范

Globs:
**/*.md

# 文档规则
- 使用中文编写
- 包含以下部分：
  1. 功能说明
  2. 使用示例
  3. API 文档
  4. 注意事项
- 代码块标注语言类型
- 配置说明使用表格格式
- 重要信息使用引用块标注
```

### 使用效果

我们通过实际例子看看这些规则如何工作：

#### 1. 创建新的 React 组件

打开 Chat 窗口选择 Agent 模式，输入 `创建一个用户列表组件` 并按回车，AI 会按照我们的项目规则生成代码。

image.png

#### 2. 编写测试文件

0.46 版本会提示匹配的规则文件，而 0.47 版本没有提示匹配文件，只是提示 “根据规范”。新版有的功能还不如旧版体验好。

image.png

image.png

#### 3. 编写文档

文档都是按照我们设定的规范生成的。

image.png

## 总结

Cursor 的 AI 规则功能很强大。它可以让 AI 助手更好地理解你的项目需求。

无论是生成代码、测试还是文档，Cursor 都会先读取规则设置，然后按规则工作。

建议优先使用项目级规则。如果有所有项目都适用的规则，可以设置全局规则。

通过设置这些规则，你可以得到一个更智能的编程助手。特别是在团队协作中，规则可以确保所有成员遵循相同的开发标准，从而提高代码质量和团队效率。