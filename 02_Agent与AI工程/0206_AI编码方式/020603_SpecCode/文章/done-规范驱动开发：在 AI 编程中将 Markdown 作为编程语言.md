> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020603_SpecCode/020603_核心知识点/SpecCode规格驱动与验收边界|SpecCode规格驱动与验收边界]]
---
title: 规范驱动开发：在 AI 编程中将 Markdown 作为编程语言
author: 几米宋
date:
url: https://mp.weixin.qq.com/s?__biz=MzIwNDIzODExOA==&mid=2650171080&idx=1&sn=34c067b426a004b69c7d06f07ce93eda&chksm=8fd8a4b48edc8c265c12cf9ed6f8cb96b643ee4adb44f6a0c9cf4a23a27cec76ddf4cd229496&mpshare=1&scene=24&srcid=1211j3kYANHC6XoY5OaPwUVA&sharer_shareinfo=52d84d15189f820820d641f92ba9f794&sharer_shareinfo_first=52d84d15189f820820d641f92ba9f794#rd
---

欢迎大家关注「几米宋」的微信公众号，公众号聚焦于 AI、云原生、开源软件、技术观察以及日常感悟等内容，更多精彩内容请访问个人网站 jimmysong.io。

---

> **📄 文章摘要**
> 本文介绍如何将 Markdown 作为 AI 编程的“规范驱动开发”语言，通过规范文档与代码生成的深度融合，提升协作效率与代码一致性，为 AI 辅助开发带来全新范式。

> 本文翻译自 Spec-Driven Development: Using Markdown as a Programming Language When Building with AI，版权归 GitHub 所有，原文作者 @wham。

## 引言

在 AI 编程助手（如 GitHub Copilot）日益普及的今天，如何高效、可控地驱动 AI 生成代码成为开发者关注的焦点。本文以 GitHub 官方博客为蓝本，介绍一种创新的“规范驱动开发”方法：将 Markdown 作为编程语言，驱动 AI 生成和维护应用代码，实现文档与实现的高度同步。

## 背景与问题

传统的 AI 编程助手工作流通常是“写一个做 X 的应用 A”，然后不断迭代：“添加功能 Y”、“修复 bug Z”。这种方式虽然高效，但随着项目复杂度提升，AI 助手容易遗忘上下文、重复询问、甚至忽略先前决策，影响开发体验和产出质量。

为了解决这一问题，一些 AI 编程助手支持自定义指令文件（如 GitHub Copilot 的 `copilot-instructions.md`），用于记录应用目标和设计决策。然而，手动同步文档和实现往往繁琐且易被忽略。

## 规范驱动开发理念

作者提出了一种全新的思路：**将整个应用的规范和实现都写在 Markdown 文件中，让 AI 编程助手“编译”成实际代码。** 以 GitHub Brain MCP Server 项目为例，开发者几乎不再直接编辑 Go 代码，而是通过维护 Markdown 规范文件驱动整个开发流程。

这种方法不仅适用于 Go，也适用于任何支持 AI 编程助手的语言和平台。

## 关键文件与目录结构

在该工作流中，主要涉及以下四个文件：

```
.
├── .github/
│   └── prompts/
│       └── compile.prompt.md
├── main.go
├── main.md
└── README.md
```

整体流程为：开发者编辑 `README.md` 或 `main.md`，通过 `compile.prompt.md` 让 AI 编程助手生成 `main.go`，再像普通 Go 应用一样构建和运行。

## 主要文件说明

在规范驱动开发中，每个文件承担着不同的角色。以下分别介绍其作用和内容。

### 用户文档：README.md

`README.md` 主要面向用户，提供安装和使用说明。如果是库项目，则包含 API 文档。以下为示例应用的精简片段：

```
# GitHub Brain MCP Server

**GitHub Brain** 是一个用于汇总 GitHub 讨论、Issue 和 PR 的实验性 MCP 服务器。

## 用法

```sh
go run main.go <command> [<args>]
```

**工作流：**

1. 使用 `pull` 命令填充本地数据库。
1. 使用 `mcp` 命令启动 MCP 服务器。

### __STRING_PLACEHOLDER_5__

拉取 GitHub 数据填充本地数据库。

示例：

```sh
go run main.go pull -o my-org
```

参数说明：
- `-t`：你的 GitHub 个人访问令牌。**必填。**
- `-o`：要拉取数据的 GitHub 组织。**必填。**
- `-db`：SQLite 数据库目录路径。默认：当前目录下的 `db` 文件夹。

### __STRING_PLACEHOLDER_13__

使用本地数据库启动 MCP 服务器。

...README.md 其余内容...
```

通过将 `README.md` 内容嵌入 `main.md`，实现文档与实现的同步更新。

### 规范说明：main.md

`main.md` 是应用的“源代码”规范文件。每次添加功能或修复 bug，均在此文件中描述。示例片段如下：

```
# GitHub Brain MCP Server

AI 编程助手规范说明。面向用户的文档见 [README.md](agents/历程/Hermes_Agent_记忆架构/README.md)。

## CLI

根据 [Usage](agents/历程/Hermes_Agent_记忆架构/README.md#usage) 部分实现 CLI。严格遵循参数/变量命名。仅支持 __STRING_PLACEHOLDER_0__ 和 __STRING_PLACEHOLDER_1__ 命令。

## pull
- 将 CLI 参数和环境变量解析为 `Config` 结构体：
- `Organization`：组织名（必填）
- `GithubToken`：GitHub API Token（必填）
- `DBDir`：SQLite 数据库路径（默认：`./db`）
- 始终使用 `Config` 结构体，避免多次读取环境变量
- 拉取内容：仓库、讨论、Issue、PR、团队
- 使用 `log/slog` 自定义日志器，控制台输出最近 5 条带时间戳的日志

...main.md 其余内容...
```

注意，面向用户的文档内容直接嵌入了规范说明。这保证了文档和实现同步。如果我想给 `-o` 参数加个别名，只需更新 `README.md`，无需额外操作。

再看一段 `main.md` 的片段：

```
### Discussions
- 针对每个仓库（`has_discussions_enabled: true`）查询讨论
- 在首次拉取前，从数据库记录最近的仓库讨论 `updated_at` 时间戳

```graphql
{
  repository(owner: "<organization>", name: "<repository>") {
    discussions(first: 100, orderBy: { field: UPDATED_AT, direction: DESC }) {
      nodes {
        url
        title
        body
        createdAt
        updatedAt
        author {
          login
        }
      }
    }
  }
}
```
- 如果仓库不存在，删除该仓库及其所有关联数据后继续
- 按 `updatedAt` 倒序查询讨论
- 遇到 `updatedAt` 早于记录时间戳的讨论时停止拉取
- 以 `url` 为主键保存或更新
- 保留讨论的 Markdown 正文

...main.md 其余内容...
```

这实际上就是用 Markdown 和自然语言编程：描述变量、循环和逻辑条件。你可以用 `if`、`foreach`、`continue` 等常用关键字，还能用 Markdown 链接 `[]()` 作为“导入”。

数据库结构同样用 Markdown 描述：

```
## 数据库

SQLite 数据库存储于 `{Config.DbDir}/{Config.Organization}.db`（如需自动创建文件夹）。避免使用事务，每个 GraphQL 项目即时保存。

### 表结构

#### table:repositories
- 主键：`name`
- 索引：`updated_at`
- `name`：仓库名（如 `repo`），不含组织前缀
- `has_discussions_enabled`：布尔值，是否启用讨论功能
- `has_issues_enabled`：布尔值，是否启用 Issue 功能
- `updated_at`：最后更新时间戳

...main.md 其余内容...
```

### AI 编程助手提示文件：compile.prompt.md

`compile.prompt.md` 采用 GitHub Copilot 的 prompt file 格式，指示 AI 将 `main.md` 编译为 `main.go`。示例内容如下：

```
---
mode: agent
---
- 按照 [the specification](../../main.md) 更新应用
- 用 VS Code 任务构建代码。不要让我手动运行 `go build` 或 `go test`。
- 对每个用到的库，获取其 GitHub 主页文档和示例。
```

所有关键信息均在 `main.md`，提示文件只需简单指令，便于迁移和复用。

## 工作流与实践

规范驱动开发的循环流程如下：

1. 编辑 `main.md` 或 `README.md`，更新规范说明。

2. 让 AI 编程助手编译生成 Go 代码。

3. 运行并测试应用，发现问题后回到规范文件修正。

4. 重复以上步骤，持续迭代。

在 VS Code 的 GitHub Copilot 中，可通过 `/` 命令调用提示，自动完成规范到代码的转换。

在 VS Code 的 GitHub Copilot 中使用 / 命令调用 AI 编程助手提示。

如规范较大，可在提示中加入 `focus on <the-change>` 引导 AI 关注具体变更。

演示如何用 / 命令让 Copilot 关注特定变更。

### 编码体验

在 `main.md` 里编程要求开发者清晰描述需求，这也是软件开发的核心难点。AI 助手不仅能自动补全，还能推荐最佳实践。例如，为所有 MCP 工具添加分页功能时，Copilot 会自动推荐合适的参数和风格。

Copilot 推荐分页风格和参数名。

### 规范整理（Linting）

随着规范文件增大，结构可能变乱。可通过 lint.prompt.md 让 AI 帮助整理规范，提升清晰度和一致性。示例内容如下：

```
---
mode: agent
---
- 优化 [the app specification](../../main.md) 的清晰度和简洁性
- 把英语当作编程语言处理
- 统一术语，避免 pull/get/fetch 等同义词混用
- 删除重复内容
- 保留所有重要细节
- 不要修改 Go 代码，只优化 Markdown 文件
- 不要修改本提示内容
```

通过 `/` 命令调用后，AI 会自动整理 `main.md`，再用 `compile.prompt.md` 编译为代码。

Copilot 整理和 lint 规范说明，提升清晰度和简洁性。

## 实践体会与展望

经过数月实践，作者总结如下：

• **方法可行且高效。** Copilot 升级后体验持续提升。

• **规范文件变大后编译速度变慢。** 可尝试将规范拆分为多个模块。

• **测试依然重要。** 规范描述预期行为，测试验证实际效果。

未来还可探索直接用规范驱动生成其他语言的应用，进一步提升自动化和一致性。

## 总结

规范驱动开发通过将 Markdown 作为编程语言，极大提升了 AI 编程助手的协作效率和代码一致性。开发者只需维护规范文档，AI 即可自动生成和维护代码，实现文档与实现的高度同步。这一方法为 AI 辅助开发提供了全新范式，值得在更多场景中推广实践。

## 参考文献

1. Spec-Driven Development: Using Markdown as a Programming Language When Building with AI - github.blog

2. GitHub Brain MCP Server - github.com

3. GitHub Copilot 指令文件官方文档 - docs.github.com

---

**🔗 更多精彩内容**

• 🌐 个人网站：jimmysong.io

• 🎥 Bilibili：space.bilibili.com/31004924

> 💫 **如果这篇文章对你有帮助，欢迎点赞、分享给更多朋友！**