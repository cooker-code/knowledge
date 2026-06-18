> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: Hermes Kanban 看板 v2.0 升级笔记
author: 傲来说
date: TrevanZhangTrevanZhang
url: https://mp.weixin.qq.com/s?__biz=Mzk0NjQ5MTY0OA==&mid=2247484089&idx=1&sn=5baa09bcb64b8a57b109c2dd55ad26f3&chksm=c22b2e1108d3c0d124efd19052c60a709f5be9b8c6ea079462184a094675f4b813cacb4872c3&mpshare=1&scene=24&srcid=06016urxacjWv33bTUU2e5rZ&sharer_shareinfo=4c5c579ceda745117a06b58e7fcdf6b5&sharer_shareinfo_first=4c5c579ceda745117a06b58e7fcdf6b5#rd
---

# Hermes Kanban 看板 v2.0 升级笔记

> 版本：Hermes Agent v0.15.1 · 技能 v2.0.0
> 整理时间：2026-05-29

Hermes Kanban 是基于 SQLite 的持久化任务看板，用于协调多个 AI Agent 协作。本次升级是**架构级变化**，从简单任务队列进化为完整的多 Agent 协作平台。

---

## 一、CLI 命令变化

新增 swarm（蜂群拓扑）、specify/decompose（自动分解）、schedule/unblock/promote（状态管理）、notify-subscribe（通知推送）、stats/heartbeat/assignees（监控）、gc（垃圾回收）、boards（多看板）等命令。daemon 命令已废弃，调度器内置在 gateway 中。

---

## 二、Swarm v1 —— 蜂群拓扑

多个专家并行工作，然后统一验证和合成。

```
planning root（共享黑板）
    ├─ 并行专家 worker 1、2、N
    └─ verifier（等所有 worker 完成）
         └─ synthesizer（等验证通过后合成）
```

**核心机制**：共享黑板（结构化 JSON 评论实现跨 worker 通信）、验证门控（verifier 必须通过）、幂等创建。

---

## 三、Triage 列 + 自动分解

从"手动创建任务"进化为"丢一句话，系统自动规划"。

**流程**：往 Triage 列丢一个粗略想法 → 调度器自动调用辅助 LLM → LLM 看你的 profile 名册生成任务图 → 系统自动创建子任务、链接依赖。

支持 Auto（默认，每个调度周期自动分解）和 Manual（手动触发）两种模式。Decompose 是 Specify 的严格超集——当 LLM 判断不需要扇出时自动回退为单任务推进。

---

## 四、Dashboard 可视化看板

全新 Web GUI，Linear/Fusion 风格。六列看板（Triage→Todo→Ready→Running→Blocked→Done），支持拖放、泳道分组、WebSocket 实时更新、内联创建、多选批量操作、Markdown 渲染。通过 `hermes dashboard` 打开。

---

## 五、通知订阅系统

任务完成后自动推送通知到飞书/Telegram 等平台。从 gateway 创建任务时自动订阅创建者，任务到达终态（completed/blocked/crashed）时推送 worker summary 的第一行。

---

## 六、多看板 Boards

不同项目的工作流隔离。每个 board 有独立的 SQLite DB 和工作空间，Worker 只能看到所在 board 的任务。

---

## 七、Worker 交互变化

Worker 不再通过 shell 执行 CLI，而是通过专用 `kanban_*` 工具集直接操作数据库（kanban\_show/complete/block/heartbeat/create 等）。优势：后端可移植、无 shell 脆弱性、更好的错误处理。

**结构化交接**：下游 worker 自动看到上游的 summary + metadata，重试 worker 看到之前的失败原因，不会重蹈覆辙。

---

## 八、熔断器与崩溃恢复

**熔断器**：连续 N 次失败（默认 2 次）后自动阻塞，防止无限抖动。
**崩溃检测**：调度器轮询 PID，发现进程死亡后自动回收并重试。

---

## 九、八种协作模式

P1 扇出（N 个同级同角色）、P2 流水线（角色链）、P3 投票（N 个同级+聚合器）、P4 长期日志（共享目录+cron）、P5 人工介入（阻塞→评论→解除）、P8 批量（一个 profile N 个对象）、P9 分诊规格器（一句话变完整任务）。

---

## 十、Kanban vs delegate\_task

delegate\_task 是 RPC 调用（fork→join），失败即失败，不支持人工介入；Kanban 是持久化消息队列 + 状态机，支持阻塞/重试/崩溃恢复/人工介入/永久审计追踪。

---

## 十一、Profile 与 SOUL.md

Kanban 的 assignee 是预先创建的 Hermes profile。真正区分角色的是 SOUL.md（灵魂文件）——定义身份、风格、行为准则和禁区。分解器依赖 profile 描述来决定任务路由。

---

## 配置速查

dispatch\_in\_gateway: true（调度器内置 gateway）、dispatch\_interval\_seconds: 60（调度间隔）、failure\_limit: 2（熔断阈值）、auto\_decompose: true（自动分解）、auto\_decompose\_per\_tick: 3（每周期分解上限）。

---

## 总结

三个最有价值的新功能：Triage + Auto-Decompose（丢一句话自动规划任务图）、Swarm 拓扑（标准化并行→验证→合成模式）、Dashboard + 通知（完整可视化和实时推送）。