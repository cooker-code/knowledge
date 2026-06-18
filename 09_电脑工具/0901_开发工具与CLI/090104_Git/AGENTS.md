# Git

## 知识点入口

- 本模块先看宏观流程，再看文章：[知识地图](090104_核心知识点/知识地图.md)。
- 新文章必须先归入流程节点，再判断是补充、冲突、不同层次还是降权。
- `文章/` 只保留原文锚点，长期知识必须沉淀到 `090104_核心知识点/`。

## 技术定位

| 项 | 内容 |
|---|---|
| 工具名 | Git / gh / Git Worktree / GitButler |
| 一级类目 | 电脑工具 |
| 二级类目 | 开发工具与 CLI |
| 技术本体 | 本机版本控制、分支管理、提交规范、远端同步、代码回退和多工作区并行开发工具链 |
| 全局位置 | 位于本地工作区、`.git` 元数据、远端仓库、分支、提交、工作树、代码评审和 CI/MR/PR 流程之间 |
| 主要使用者 | 开发者、AI 编程工具使用者、需要多任务并行开发和稳定代码收口的用户 |
| 主要产出 | commit、branch、tag、worktree、stash、remote sync、PR/MR、回滚记录、提交规范检查结果 |
| 不等同于 | GitHub/GitLab 平台全功能、工程质量体系全部、AI 编程工具本体、后端/前端工程机制 |

## 当前原文锚点

| 主题 | 原文 | 初始判断 |
|---|---|---|
| Git Worktree 与 AI 并行开发 | [Git Worktree，让你的 AI 并行 Coding](文章/Git Worktree，让你的 AI 并行 Coding.md)、[我是如何用 Git Worktree + Skill 实现多任务并行开发的](文章/我是如何用 Git Worktree + Skill 实现多任务并行开发的.md) | 精读候选，重点吸收多任务隔离、分支同步、收口和冲突边界 |
| worktree CLI 封装 | [我把 worktree 做成了一个顺手的 CLI](文章/我把 worktree 做成了一个顺手的 CLI.md) | 精读候选，重点判断封装是否降低误操作、路径混乱和清理成本 |
| GitButler 版本控制工作台 | [GitButler：专为 AI 时代设计的版本控制工具](文章/GitButler：专为 AI 时代设计的版本控制工具.md)、[GitButler：Git 的革命性开源神器！并行分支、无限撤销、AI 智能代理、零 rebase 烦恼，一文吃透全部功](文章/GitButler：Git 的革命性开源神器！并行分支、无限撤销、AI 智能代理、零 rebase 烦恼，一文吃透全部功.md) | 精读候选，重点吸收虚拟分支、撤销、GUI 工作台和 Git 原生命令边界；宣传性结论需降权 |
| 提交规范与校验 | [为什么很多团队开始上 commit 校验？](文章/为什么很多团队开始上 commit 校验？.md)、[把 Git 提交变成“可执行规范”：Commit 规范体系与 Husky_Commitlint_Commitizen_Lint-staged 全链路介绍](文章/把 Git 提交变成“可执行规范”：Commit 规范体系与 Husky_Commitlint_Commitizen_Lint-staged 全链路介绍.md) | 精读候选，重点吸收 commit message、hook、lint-staged 和团队协作边界 |
| gh 与平台 CLI | [有了 Git，为什么还要安装 gh？](文章/有了 Git，为什么还要安装 gh？.md) | 精读候选，重点区分 Git 本地版本控制与 GitHub 平台操作 |
| 已 push 代码撤回 | [面试官：Git 如何撤回已 Push 的代码？问倒一大片。。。](文章/面试官：Git 如何撤回已 Push 的代码？问倒一大片。。。.md) | 实践候选，必须写清 revert、reset、force push 的风险和团队协作边界 |

## 宏观流程

| 节点 | 要回答的问题 |
|---|---|
| G00 场景判断 | 当前问题是本地版本控制、远端协作、并行 worktree、提交规范、平台 CLI、回滚恢复，还是工具推荐 |
| G01 初始化与身份配置 | 仓库、remote、user、签名、认证、默认分支、忽略文件和本机凭证如何确认 |
| G02 工作区状态判断 | `status`、`diff`、`log`、`branch`、`remote`、未跟踪文件和脏状态如何作为动作前置证据 |
| G03 分支与 worktree | 分支、工作树、多任务隔离、路径命名、同步基线、清理和合并如何治理 |
| G04 提交与规范 | commit 粒度、message、hook、commitlint、lint-staged、签名和可追溯性如何落地 |
| G05 远端同步与平台 CLI | fetch、pull、rebase、push、PR/MR、gh/glab 等平台 CLI 如何进入工作流 |
| G06 回退与恢复 | restore、checkout、reset、revert、stash、reflog、force push 分别适合什么风险等级 |
| G07 自动化与 AI 协作 | AI 编程、多工作区、提交收口、冲突处理、权限边界和人为确认点如何设计 |
| G08 维护与排障 | 大仓库、二进制、LFS、子模块、换行、权限、历史污染和冲突反复如何定位 |

## 横向对标

| 对标对象 | 对比重点 | 使用判断 |
|---|---|---|
| Git 原生命令 | 最通用、透明、可脚本化，但误操作成本高 | 关键动作优先理解原生命令语义，再决定是否封装 |
| gh / glab | GitHub/GitLab 平台操作、PR/MR、issue、workflow、release | 本地版本控制归 Git；平台协作动作可用 gh/glab 作为上层接口 |
| GitButler | GUI/工作台、虚拟分支、撤销、并行分支体验 | 可作为复杂本地工作流辅助，但不能替代对 Git 状态和历史风险的判断 |
| worktree CLI 封装 | 降低多工作区创建、切换和清理成本 | 适合 AI 并行任务；必须有命名、基线同步和删除保护 |
| IDE Git 面板 | 可视化强，但隐藏命令细节 | 日常查看可用；回滚、rebase、force push 等高风险动作要回到命令证据 |

## 排重准则

问题指纹：

```text
Git + 工作流节点 + 操作对象 + 命令/工具接口 + 回滚/验证边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是 worktree | 按创建、同步、清理、合并、AI 并行任务、路径管理拆分 |
| 都是提交规范 | 区分 message 规范、hook 校验、lint-staged、自动生成 commit、团队可追溯性 |
| 都是回退代码 | 按本地未提交、已提交未 push、已 push、共享分支、历史重写风险拆分 |
| 都是 Git GUI | 只有当影响分支模型、撤销模型、并行工作流或风险控制时才沉淀 |
| 都是 GitHub/GitLab | 如果主问题是平台项目管理、CI、权限或发布，转到对应工程实践或平台工具目录 |

## 认知校准优先级

| 优先级 | 要校准的问题 |
|---|---|
| P0 | 所有破坏性 Git 动作前必须先看真实状态：branch、status、diff、log、remote 和未跟踪文件 |
| P0 | `revert`、`reset`、`restore`、`checkout`、`reflog`、`force push` 不能混用，必须按是否已 push 和是否共享分支分级 |
| P1 | worktree 对 AI 并行开发有价值，但真正风险在基线同步、分支命名、产物清理和最终合并 |
| P1 | commit 规范不是格式洁癖，价值在审计、自动化、变更理解和发布回溯 |
| P2 | GitButler、gh、worktree CLI 都是 Git 工作流上层工具，不能把它们的体验宣传写成 Git 本体能力 |

## 后续追查

- 从当前 9 篇原文维护文章路由表，先区分 worktree、commit 规范、gh、GitButler、回退恢复。
- 抽一篇核心知识点：`Git破坏性操作前置检查与恢复边界`。
- 抽一篇核心知识点：`Git Worktree支撑AI并行开发的使用边界`。
- 抽一篇核心知识点：`Commit规范从格式到可执行质量门`。
- 用本机真实命令补证常用状态检查、worktree 创建/清理、reflog 恢复和 gh 工作流。
