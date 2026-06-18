# Hermes

## 知识点入口

- 本模块先看宏观流程，再看文章：[知识地图](020504_核心知识点/知识地图.md)。
- 新文章必须先归入 Hermes 的流程节点，再判断是补充、冲突、不同层次还是降权。
- `文章/` 只保留原文锚点，长期知识必须沉淀到 `020504_核心知识点/`。

## 技术定位

| 项 | 内容 |
|---|---|
| 技术名 | Hermes |
| 一级类目 | Agent 与 AI 工程 |
| 二级类目 | AI 编程工具 |
| 技术本体 | 面向本地 AI 编程和长任务协作的 Agent 工具链，覆盖 CLI、Web UI、Profile、Kanban、Skill、Workspace 和外部集成 |
| 全局架构位置 | 位于用户任务和模型/工具运行时之间，负责把任务、规则、工具、记忆和多 Agent 协作组织起来 |
| 主要使用者 | 开发者、技术负责人、个人自动化用户 |
| 主要产出 | 可执行任务、代码或文档产物、Kanban 协作状态、Skill、经验沉淀和运行记录 |

## 归类边界

| 相邻方向 | 归类规则 |
|---|---|
| Harness Engineering | 只讲通用运行时、质量门禁、环境、评估，不依赖 Hermes 产品特性时，归 Harness Engineering |
| Memory Management | 只讲 Agent 记忆抽象、压缩、检索、生命周期，不以 Hermes 为主角时，归 Memory Management |
| Prompt Engineering | 只讲系统提示词、角色、指令模板，不以 Hermes Profile/SOUL 为主角时，归 Prompt Engineering |
| 工作流编排 | 只讲 n8n、Dify、Coze 等低代码工作流，不归 Hermes |
| 电脑工具 | 只讲普通 CLI 安装和桌面软件使用，不涉及 Agent 工程能力时才归电脑工具 |

## 排重准则

问题指纹：

```text
Hermes 功能层 + 配置对象 + 运行机制 + 解决问题 + 稳定性/权限边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是安装教程 | 只保留一个入口，重复安装步骤降权 |
| 都是 Profile/SOUL | 比较是否新增角色边界、协作协议或失败案例 |
| 都是 Kanban | 比较是否讲任务状态、交接、冲突合并和质量门禁 |
| 都是 Skill 沉淀 | 比较是沉淀流程、触发机制、版本治理还是安全边界 |
| 只有“更聪明、更好用” | 没有机制和证据时降为略读 |

## 已覆盖能力

| 能力 | 文章目录 | 已覆盖问题 | 还缺什么 |
|---|---|---|---|
| CLI / Web UI / Desktop | [文章](文章) | 启动入口、Web UI、Desktop、命令清单 | 官方版本和配置补证 |
| Profile / SOUL | [文章](文章) | 人格装配、团队角色、Profile 协作 | 角色冲突和最小配置 |
| Kanban / 多 Agent | [文章](文章) | 多 Agent 看板、任务协作、状态管理 | 质量门禁和失败恢复 |
| Skill / Learning Loop | [文章](文章) | 自动沉淀经验、Skill 生成和使用 | 版本治理、安全审查 |
| 记忆 / 知识库 / Obsidian | [文章](文章) | 记忆、知识入库、自动整理 | 与 Memory Management 的边界 |
| 稳定性 / Checkpoint | [文章](文章) | 长任务运行、恢复、定时、Webhook | 实测日志和恢复指标 |

## 后续追查

- 关键词：Hermes Agent、Hermes Kanban、Hermes Profile、SOUL.md、Hermes Skill、Hermes Workspace、Hermes Web UI、Hermes Desktop。
- 待读资料：官方文档、版本发布记录、配置 schema、权限模型。
- 待补实验：最小 Profile、多 Agent Kanban、Skill 自动沉淀、Checkpoint 恢复、Webhook 自动化。
