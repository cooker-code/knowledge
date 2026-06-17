---
title: OpenSpec 实践指南：规范驱动开发的最佳实践
author: 和解AIAgent
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU0MzA4OTUyOQ==&mid=2247487871&idx=1&sn=f37660addf3be9da8ed62018cb81192e&chksm=facd688db1344639e459d7918719dea7f89dcc40ffc4b1c393d8b97ad9c595b6cc902fd18d0d&mpshare=1&scene=24&srcid=11104znXOFv9xhA0pSk4zru6&sharer_shareinfo=b4ee538a9198af7626670024eda70b44&sharer_shareinfo_first=b4ee538a9198af7626670024eda70b44#rd
---

01

概述

OpenSpec 是一个为 AI 编程助手打造的规范驱动开发工具，通过在工作开始前锁定意图，实现人类和 AI 之间的确定性协作。

02

核心理念

01

为什么需要 OpenSpec？

AI 编程助手虽然强大，但当需求仅存在于聊天历史中时会变得不可预测。OpenSpec 通过以下方式解决这个问题：

* 规范先行：在编写任何代码之前就确定要构建什么
* 结构化变更：提案、任务和规范更新保持范围明确且可审计
* 共享可见性：清晰了解什么是提议的、活跃的或已归档的
* 工具兼容：支持多种 AI 工具，无需 API 密钥

02

核心工作流

OpenSpec 采用三阶段工作流模式：

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8
* 9
* 10
* 11
* 12
* 13
* 14
* 15
* 16
* 17
* 18
* 19
* 20
* 21
* 22

```
┌────────────────────┐│ 起草变更提案        ││ (Proposal)         │└────────┬───────────┘         │ 与 AI 共享意图         ▼┌────────────────────┐│ 审查与对齐          ││ (编辑规范/任务)     │◀──── 反馈循环 ──────┐└────────┬───────────┘                          │         │ 批准计划                             │         ▼                                      │┌────────────────────┐                          ││ 实施任务            │──────────────────────────┘│ (AI 编写代码)       │└────────┬───────────┘         │ 发布变更         ▼┌────────────────────┐│ 归档并更新规范      ││ (真相来源)          │└────────────────────┘
```

03

快速开始

01

前置要求

* Node.js >= 20.19.0 - 使用 node --version 检查版本

02

安装步骤

#### 1. 全局安装 CLI

* 1

```
npm install -g @fission-ai/openspec@latest
```

验证安装：

* 1

```
openspec --version
```

#### 2. 在项目中初始化 OpenSpec

* 1
* 2

```
cd my-projectopenspec init
```

初始化期间：

* 系统会提示选择支持的 AI 工具（Claude Code、CodeBuddy、Cursor、OpenCode、Qoder 等）
* OpenSpec 自动配置斜杠命令
* 创建 openspec/ 目录结构
* 生成 AGENTS.md 交接文件

#### 3. 填充项目上下文（可选）

初始化完成后，使用以下提示填充项目上下文：

* 1

```
"请阅读 openspec/project.md 并帮我填写项目详情、技术栈和约定"
```

04

三阶段工作流详解

01

阶段 1：创建变更提案（Creating Changes）

#### 何时需要创建提案？

创建提案的触发条件：

* ✅ 添加新功能或能力
* ✅ 进行破坏性变更（API、架构）
* ✅ 改变架构或模式
* ✅ 性能优化（改变行为）
* ✅ 更新安全模式

**跳过提案的情况：**

* ❌ Bug 修复（恢复预期行为）
* ❌ 拼写错误、格式、注释
* ❌ 依赖更新（非破坏性）
* ❌ 配置更改
* ❌ 现有行为的测试

#### 提案创建流程

**步骤 1：探索当前状态**

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8

```
# 查看现有规范openspec spec list --long  
# 查看活跃变更openspec list  
# 全文搜索（可选）rg -n "Requirement:|Scenario:" openspec/specs
```

**步骤 2：选择唯一的 change-id**

命名规则：

* 使用 kebab-case
* 动词开头：add-, update-, remove-, refactor-
* 简短描述性：add-two-factor-auth
* 确保唯一性；如被占用，追加 -2, -3 等

**步骤 3：搭建变更结构**

* 1
* 2

```
CHANGE=add-profile-filtersmkdir -p openspec/changes/$CHANGE/specs/profile
```

创建以下文件：

* proposal.md - 为什么、什么变更、影响
* tasks.md - 实施清单
* design.md - 技术决策（仅在需要时）
* specs/[capability]/spec.md - 规范增量

#### 提案文件模板

**proposal.md 模板：**

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8
* 9
* 10

```
## Why[1-2 句话说明问题/机会]  
## What Changes- [变更项列表]- [标记破坏性变更为 **BREAKING**]  
## Impact- Affected specs: [列出受影响的能力]- Affected code: [关键文件/系统]
```

**规范增量模板（specs/[capability]/spec.md）：**

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8
* 9
* 10
* 11
* 12
* 13
* 14
* 15
* 16

```
## ADDED Requirements### Requirement: 新功能名称系统 SHALL 提供...  
#### Scenario: 成功场景- **WHEN** 用户执行操作- **THEN** 预期结果  
## MODIFIED Requirements### Requirement: 现有功能名称[完整的修改后需求内容]  
## REMOVED Requirements### Requirement: 旧功能名称**Reason**: [移除原因]**Migration**: [如何处理]
```

**关键格式要求：**

⚠️ **场景格式（最常见错误）：**

✅ **正确**（使用 #### 标题）：

* 1
* 2
* 3

```
#### Scenario: 用户登录成功- **WHEN** 提供有效凭证- **THEN** 返回 JWT token
```

❌ **错误**（不要使用列表或粗体）：

* 1
* 2
* 3

```
- **Scenario: 用户登录**  ❌**Scenario**: 用户登录     ❌### Scenario: 用户登录      ❌
```

**tasks.md 模板：**

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8
* 9
* 10
* 11
* 12

```
## 1. 数据库设置- [ ] 1.1 创建用户表 OTP 密钥列- [ ] 1.2 创建 OTP 验证日志表  
## 2. 后端实施- [ ] 2.1 添加 OTP 生成端点- [ ] 2.2 修改登录流程以要求 OTP- [ ] 2.3 添加 OTP 验证端点  
## 3. 前端更新- [ ] 3.1 创建 OTP 输入组件- [ ] 3.2 更新登录流程 UI
```

**design.md 创建条件：**

仅在以下情况创建 `design.md`：

* 跨领域变更（多个服务/模块）或新架构模式
* 新外部依赖或重大数据模型变更
* 安全、性能或迁移复杂性
* 需要在编码前做出技术决策的模糊性

**design.md 最小模板：**

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8
* 9
* 10
* 11
* 12
* 13
* 14
* 15
* 16
* 17
* 18
* 19

```
## Context[背景、约束、利益相关者]  
## Goals / Non-Goals- Goals: [...]- Non-Goals: [...]  
## Decisions- Decision: [什么以及为什么]- Alternatives considered: [选项 + 理由]  
## Risks / Trade-offs- [风险] → 缓解措施  
## Migration Plan[步骤、回滚]  
## Open Questions- [...]
```

**步骤 4：验证提案**

* 1

```
openspec validate <change-id> --strict
```

修复所有验证错误后再分享提案。

02

阶段 2：实施变更（Implementing Changes）

#### 实施前检查清单

在开始任何任务前：

* 阅读相关规范：specs/[capability]/spec.md
* 检查待处理变更：changes/ 中的冲突
* 阅读项目约定：openspec/project.md
* 运行 openspec list 查看活跃变更
* 运行 openspec list --specs 查看现有能力

#### 实施步骤

**严格按照以下顺序执行：**

1. 阅读 proposal.md - 理解要构建什么
2. 阅读 design.md（如果存在）- 审查技术决策
3. 阅读 tasks.md - 获取实施清单
4. 按顺序实施任务 - 逐个完成
5. 确认完成 - 确保 tasks.md 中的每个项目都已完成
6. 更新清单 - 所有工作完成后，将每个任务设置为 - [x] 以反映实际情况
7. 批准门控 - 在提案审查和批准之前不要开始实施

**重要提示：**

* 不要跳过文档阅读直接编码
* 不要批量完成任务标记
* 完成每个任务后立即标记
* 保持顺序，不要提前跳转

#### 使用 AI 工具实施

**支持原生斜杠命令的工具：**

| 工具 | 命令 |
| --- | --- |
| Claude Code | `/openspec:proposal` , `/openspec:apply`, `/openspec:archive` |
| Cursor | `/openspec-proposal` , `/openspec-apply`, `/openspec-archive` |
| CodeBuddy | `/openspec:proposal` , `/openspec:apply`, `/openspec:archive` |
| OpenCode | `/openspec-proposal` , `/openspec-apply`, `/openspec-archive` |

**示例对话：**

* 1
* 2
* 3
* 4
* 5
* 6

```
你：规范看起来不错。让我们实施这个变更。    （支持斜杠命令的工具快捷方式：/openspec:apply add-profile-filters）  
AI：我将完成 add-profile-filters 变更中的任务。    *实施 openspec/changes/add-profile-filters/tasks.md 中的任务*    *标记任务完成：任务 1.1 ✓，任务 1.2 ✓，任务 2.1 ✓...*
```

03

阶段 3：归档变更（Archiving Changes）

#### 归档时机

在以下情况归档变更：

* ✅ 实施已完成
* ✅ 代码已部署
* ✅ 所有任务已完成并标记

#### 归档步骤

**方法 1：使用 AI 工具**

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8

```
AI：所有任务已完成。实施已就绪。  
你：请归档变更    （支持斜杠命令的工具快捷方式：/openspec:archive add-profile-filters）  
AI：我将归档 add-profile-filters 变更。    *运行：openspec archive add-profile-filters --yes*    ✓ 变更已成功归档。规范已更新。准备好进行下一个功能！
```

**方法 2：使用命令行**

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8

```
# 交互式归档openspec archive add-profile-filters  
# 非交互式归档（自动化场景）openspec archive add-profile-filters --yes  
# 工具类变更归档（不更新规范）openspec archive <change-id> --skip-specs --yes
```

**归档后验证：**

* 1

```
openspec validate --strict
```

05

规范文件格式详解

01

增量操作类型

* ## ADDED Requirements - 新能力
* ## MODIFIED Requirements - 变更的行为（包含完整的更新文本）
* ## REMOVED Requirements - 已弃用的功能
* ## RENAMED Requirements - 名称变更

02

ADDED vs MODIFIED 使用指南

**使用 ADDED 当：**

* 引入新的独立能力或子能力
* 变更是正交的（例如，添加"斜杠命令配置"）
* 不改变现有需求的语义

**使用 MODIFIED 当：**

* 改变现有需求的行为、范围或验收标准
* 必须粘贴完整的更新后需求内容（标题 + 所有场景）
* 归档器会用你提供的内容替换整个需求

**常见陷阱：**  
使用 MODIFIED 添加新关注点而不包含先前文本。这会导致归档时丢失细节。如果你没有明确更改现有需求，请在 ADDED 下添加新需求。

**正确编写 MODIFIED 需求：**

1. 在 openspec/specs/<capability>/spec.md 中找到现有需求
2. 复制整个需求块（从 ### Requirement: ... 到其场景）
3. 粘贴到 ## MODIFIED Requirements 下并编辑以反映新行为
4. 确保标题文本完全匹配（忽略空格）并保留至少一个 #### Scenario:

**RENAMED 示例：**

* 1
* 2
* 3

```
## RENAMED Requirements- FROM: `### Requirement: Login`- TO: `### Requirement: User Authentication`
```

03

需求措辞规范

* 使用 SHALL/MUST 表示规范性需求
* 避免 should/may（除非有意非规范性）
* 每个需求必须至少有一个场景

06

故障排除

01

常见错误及解决方案

**错误："Change must have at least one delta"**

解决方案：

* 检查 changes/[name]/specs/ 是否存在 .md 文件
* 验证文件具有操作前缀（## ADDED Requirements）

**错误："Requirement must have at least one scenario"**

解决方案：

* 检查场景使用 #### Scenario: 格式（4 个井号）
* 不要对场景标题使用列表项或粗体

**错误：静默场景解析失败**

解决方案：

* 需要精确格式：#### Scenario: Name
* 使用调试命令：openspec show [change] --json --deltas-only

02

验证技巧

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8

```
# 始终使用严格模式进行全面检查openspec validate [change] --strict  
# 调试增量解析openspec show [change] --json | jq '.deltas'  
# 检查特定需求openspec show [spec] --json -r 1
```

03

变更冲突处理

1. 运行 openspec list 查看活跃变更
2. 检查重叠的规范
3. 与变更所有者协调
4. 考虑合并提案

04

验证失败处理

1. 使用 --strict 标志运行
2. 检查 JSON 输出以获取详细信息
3. 验证规范文件格式
4. 确保场景格式正确

05

缺少上下文处理

1. 首先阅读 project.md
2. 检查相关规范
3. 审查最近的归档
4. 请求澄清

07

最佳实践

01

简单性优先

* 默认新代码 <100 行
* 单文件实现直到证明不足
* 避免没有明确理由的框架
* 选择经过验证的模式

02

复杂性触发条件

仅在以下情况添加复杂性：

* 性能数据表明当前解决方案太慢
* 具体的规模要求（>1000 用户，>100MB 数据）
* 多个已证明的用例需要抽象

03

清晰的引用

* 使用 file.ts:42 格式表示代码位置
* 将规范引用为 specs/auth/spec.md
* 链接相关变更和 PR

04

能力命名

* 使用动词-名词：user-auth, payment-capture
* 每个能力单一目的
* 10 分钟可理解性规则
* 如果描述需要"AND"，则拆分

05

变更 ID 命名

* 使用 kebab-case，简短描述性：add-two-factor-auth
* 偏好动词开头前缀：add-, update-, remove-, refactor-
* 确保唯一性；如被占用，追加 -2, -3 等

08

多能力变更示例

当变更影响多个能力时，为每个能力创建单独的增量文件：

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8

```
openspec/changes/add-2fa-notify/├── proposal.md├── tasks.md└── specs/    ├── auth/    │   └── spec.md   # ADDED: Two-Factor Authentication    └── notifications/        └── spec.md   # ADDED: OTP email notification
```

**auth/spec.md:**

* 1
* 2
* 3

```
## ADDED Requirements### Requirement: Two-Factor Authentication...
```

**notifications/spec.md:**

* 1
* 2
* 3

```
## ADDED Requirements### Requirement: OTP Email Notification...
```

09

CLI 命令参考

01

基本命令

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8
* 9
* 10
* 11
* 12
* 13
* 14

```
# 查看活跃变更openspec list  
# 查看规范openspec list --specs  
# 显示变更或规范详情openspec show [item]  
# 验证变更或规范openspec validate [item]  
# 归档已完成的变更openspec archive <change-id> [--yes|-y]
```

02

项目管理命令

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8

```
# 初始化 OpenSpecopenspec init [path]  
# 更新指令文件openspec update [path]  
# 交互式仪表板openspec view
```

03

命令标志

* --json - 机器可读输出
* --type change|spec - 消除歧义
* --strict - 全面验证
* --no-interactive - 禁用提示
* --skip-specs - 归档时不更新规范
* --yes/-y - 跳过确认提示（非交互式归档）

10

目录结构

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8
* 9
* 10
* 11
* 12
* 13
* 14
* 15
* 16

```
openspec/├── project.md              # 项目约定├── AGENTS.md               # AI 助手指令├── specs/                  # 当前真相 - 已构建的内容│   └── [capability]/       # 单一聚焦的能力│       ├── spec.md         # 需求和场景│       └── design.md       # 技术模式└── changes/                # 提案 - 应该变更的内容    ├── [change-name]/    │   ├── proposal.md     # 为什么、什么、影响    │   ├── tasks.md         # 实施清单    │   ├── design.md        # 技术决策（可选）    │   └── specs/          # 增量变更    │       └── [capability]/    │           └── spec.md # ADDED/MODIFIED/REMOVED    └── archive/            # 已完成的变更
```

11

工具选择指南

| 任务 | 工具 | 原因 |
| --- | --- | --- |
| 按模式查找文件 | Glob | 快速模式匹配 |
| 搜索代码内容 | Grep | 优化的正则搜索 |
| 读取特定文件 | Read | 直接文件访问 |
| 探索未知范围 | Task | 多步骤调查 |

12

团队采用建议

01

采用步骤

1. 初始化 OpenSpec - 在仓库中运行 openspec init
2. 从新功能开始 - 要求 AI 将即将进行的工作捕获为变更提案
3. 增量增长 - 每个变更归档到活的规范中，记录系统
4. 保持灵活 - 不同团队成员可以使用不同的 AI 工具，同时共享相同的规范

02

工具切换

当有人切换工具时运行 `openspec update`，这样代理就能获取最新的指令和斜杠命令绑定。

03

更新 OpenSpec

1. 升级包npm install -g @fission-ai/openspec@latest

* 1

2. 刷新代理指令

* 在每个项目内运行 openspec update 以重新生成 AI 指南并确保最新的斜杠命令处于活跃状态

13

关键要点总结

01

阶段指示器

* changes/ - 提议的，尚未构建
* specs/ - 已构建并部署
* archive/ - 已完成的变更

02

文件用途

* proposal.md - 为什么和什么
* tasks.md - 实施步骤
* design.md - 技术决策
* spec.md - 需求和行为

03

核心原则

**记住：规范是真相。变更是提案。保持它们同步。**

* 规范（specs/）代表当前系统的真实状态
* 变更（changes/）代表提议的更新
* 归档时将批准的更新合并回规范
* 始终在实施前验证提案
* 使用严格验证模式确保质量

14

完整示例：添加双因素认证

01

* 1
* 2
* 3
* 4
* 5

```
你：创建一个 OpenSpec 变更提案，用于添加双因素认证    （支持斜杠命令的工具快捷方式：/openspec:proposal 添加双因素认证）  
AI：我将为双因素认证创建一个 OpenSpec 变更提案。    *生成 openspec/changes/add-2fa/ 以及 proposal.md、tasks.md、spec 增量。*
```

02

* 1
* 2
* 3

```
$ openspec list                             # 确认变更文件夹存在$ openspec validate add-2fa                 # 验证规范格式$ openspec show add-2fa                     # 审查提案、任务和规范增量
```

03

* 1
* 2
* 3
* 4

```
你：你能为 OTP 验证添加验收标准吗？  
AI：我将用 OTP 验证的场景更新规范增量。    *编辑 openspec/changes/add-2fa/specs/auth/spec.md 和 tasks.md。*
```

04

* 1
* 2
* 3
* 4
* 5
* 6

```
你：规范看起来不错。让我们实施这个变更。    （支持斜杠命令的工具快捷方式：/openspec:apply add-2fa）  
AI：我将完成 add-2fa 变更中的任务。    *实施 openspec/changes/add-2fa/tasks.md 中的任务*    *标记任务完成：任务 1.1 ✓，任务 1.2 ✓，任务 2.1 ✓...*
```

05

* 1
* 2
* 3
* 4
* 5
* 6
* 7
* 8

```
AI：所有任务已完成。实施已就绪。  
你：请归档变更    （支持斜杠命令的工具快捷方式：/openspec:archive add-2fa）  
AI：我将归档 add-2fa 变更。    *运行：openspec archive add-2fa --yes*    ✓ 变更已成功归档。规范已更新。准备好进行下一个功能！
```

15

结论

OpenSpec 通过规范驱动开发，为 AI 编程助手提供了确定性和可审查性。通过遵循本文档中的三阶段工作流和最佳实践，团队可以实现：

* 更好的协作：人类和 AI 在开始工作前就规范达成一致
* 可审计的变更：结构化的变更文件夹保持范围明确
* 共享可见性：清晰了解提议、活跃或已归档的内容
* 工具灵活性：支持多种 AI 工具，无需 API 密钥

开始使用 OpenSpec，体验规范驱动开发的强大力量！

---

文章推荐

[➤](https://mp.weixin.qq.com/s?__biz=MzU0MzA4OTUyOQ==&mid=2247487806&idx=1&sn=76a55b5a5d0d80f95f70829754ba5675&scene=21#wechat_redirect)[那个让我熬夜的Speckit + Claude Code平台，教会了我什么? 👈️](https://mp.weixin.qq.com/s?__biz=MzU0MzA4OTUyOQ==&mid=2247487806&idx=1&sn=76a55b5a5d0d80f95f70829754ba5675&scene=21#wechat_redirect)

专注于知识管理与数据分析。为你的第二大脑注入 AI 引擎。- **和解AI说**