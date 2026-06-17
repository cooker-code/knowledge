---
title: Harness 驱动的企业级Agent工程交付方法论——GPT5.5赋能OpenSpec管规格，Ling-2.5赋能Worker干活
author: Ray的新范式
date: Ray的新范式Ray的新范式
url: https://mp.weixin.qq.com/s?__biz=MzI0Nzk1ODQyNw==&mid=2247484149&idx=1&sn=49338f207cabe6aecae5aa30bfaf7db0&chksm=e85865bd6323a496e340a8b80b22f4572b5903a43a0e466792922d8ff0548709bda41857a6d8&mpshare=1&scene=24&srcid=0509yXY0kHylHu9Z0a9AZcL6&sharer_shareinfo=8c01401f6ddd8fd573cff7c7ae9c23fc&sharer_shareinfo_first=8c01401f6ddd8fd573cff7c7ae9c23fc#rd
---

## 一句话定义

规格驱动的 AI 工程交付，是一套把需求转成规格、把规格转成可执行工单、把工单交给 AI Worker、再用独立 Judge 验收并归档知识的方法论。

它的目标不是让 AI 一次性变聪明，而是把 AI 放进一条可控、可验证、可重试、可复盘的工程生产线。

## 背景

AI 编程的瓶颈已经不只是“模型会不会写代码”，而是“模型能不能稳定交付”。在真实项目里，失败通常不是因为模型完全不会做，而是因为流程缺少约束：

* • 需求没有沉淀，下一次 AI 还要重新摸索。
* • 任务太大，Worker 容易自由发挥。
* • 验收标准不清，模型说完成但没人能快速判断。
* • 失败没有结构化反馈，下一轮继续重复犯错。
* • 代码变更没有证据链，无法复盘为什么通过或失败。

这套方法论的核心判断是：不要把所有“思考”都压在模型权重里，而要把一部分思考外置到工程流程中。规格负责约束方向，工单负责约束输入，Harness 负责约束执行，Judge 负责约束质量。

最终目标是从“让 AI 帮我写代码”，升级为“让 AI 进入一条工程交付生产线”。

## 工具组合

这套方法论可以由多种工具实现。推荐组合如下：

| 层级 | 推荐工具 | 负责什么 | 关键产物 |
| --- | --- | --- | --- |
| 规格层 | OpenSpec | 变更提案、规格沉淀、知识归档 | `proposal.md` 、`tasks.md`、`specs/` |
| 设计层 | Superpowers 方法 | 头脑风暴、拆任务、TDD、verification | `implementation-plan.md` 、`test-plan.md` |
| 编排层 | Oh My Codex | 任务分发、多代理协作、执行节奏 | agent 任务包、执行记录 |
| 执行层 | Codex | 读代码、改代码、跑测试、修复问题 | diff、测试结果、提交记录 |
| 控制层 | 自建 Harness | 调度、重试、状态流转、护栏 | `.fusion/` |
| 验收层 | Judge | 根据证据判断 PASS/FAIL | verdict、reason |
| 归档层 | OpenSpec | 把真实实现写回长期规格 | 更新后的 `specs/` |

推荐目录：

```
project/  
  specs/  
  openspec/  
    changes/<change-id>/  
      proposal.md  
      tasks.md  
  
  .fusion/  
    inbox/  
    running/  
    done/  
    failed/  
    runs/  
    traces/  
    reports/  
      verification-report.md
```

工具之间的边界必须清楚：

* • OpenSpec 是规格源，不负责执行代码。
* • Superpowers 是方法论，不抢调度主控。
* • Oh My Codex 是编排层，不发明需求。
* • Codex 是工程执行者，不做最终验收。
* • Harness 是状态机，不判断业务价值。
* • Judge 是质量门禁，不替 Worker 修代码。

最小可行组合：

| 阶段 | MVP 做法 |
| --- | --- |
| 规格 | 用 OpenSpec 或手写 `proposal.md`、`tasks.md` |
| 工单 | 手工或半自动生成 `.fusion/inbox/*.md` |
| 执行 | Codex 读取工单并修改代码 |
| 验收 | Judge 读取工单、规格、diff、测试输出 |
| 归档 | PASS 后更新 `specs/` |

## 核心信念

### 1. 规格是唯一事实源

AI 不能自己发明需求。所有实现必须从规格、任务和验收标准出发。

对应产物：

* • `proposal.md`
* • `tasks.md`
* • `specs/`

### 2. 工单是最小执行单元

一个可执行工单只做一件事，并且可以被独立验证。

好的工单必须包含：

* • 目标
* • 验收标准
* • 修改约束
* • 非目标
* • 必读上下文
* • 验证命令

### 3. Worker 只负责执行

Worker 不应该承担需求扩写、范围判断和最终验收。

Worker 的职责是：

* • 读取工单和上下文
* • 先补测试或确认测试
* • 做最小代码变更
* • 运行验证命令
* • 输出证据

### 4. Judge 只相信证据

Judge 不相信 Worker 说“我完成了”。Judge 只看：

* • 工单
* • 规格
* • git diff
* • 测试输出
* • 变更文件
* • 执行记录

### 5. Harness 承担流程思考

模型可以不稳定，但流程必须稳定。

Harness 负责：

* • 任务调度
* • 状态流转
* • 失败重试
* • 循环检测
* • 超时限制
* • 证据记录
* • 结果归档

## 术语表

| 术语 | 中文理解 | 说明 |
| --- | --- | --- |
| Spec | 规格 | 长期项目知识和能力定义 |
| Proposal | 提案 | 一次变更的目标、范围、验收标准 |
| Task | 任务 | 提案下的任务清单 |
| Story | 可执行工单 | AI Worker 可直接执行的最小任务单 |
| Worker | 执行者 | 负责写代码、改文件、跑命令 |
| Judge | 验收者 | 根据证据判断 PASS/FAIL |
| Dispatcher | 调度器 | 控制工单进入执行、重试和完成 |
| Harness | 执行框架 | Dispatcher、Worker、Judge、状态和护栏的总称 |
| Archive | 归档 | 把真实实现同步回规格和知识库 |

## 生命周期

```
需求  
-> 提案  
-> 人工审查  
-> 设计深化  
-> 拆分可执行工单  
-> Dispatcher 调度  
-> Worker 执行  
-> Judge 验收  
-> 失败反馈重试  
-> 通过交付  
-> 规格归档
```

## 阶段一：提案

目标：把模糊需求转成可审查的变更提案。

输入：

* • 用户需求
* • 当前代码和产品状态
* • 现有规格

动作：

* • 明确目标
* • 明确非目标
* • 明确验收标准
* • 明确风险
* • 明确影响范围

输出：

* • `proposal.md`
* • `tasks.md`
* • 规格草案

完成标准：

* • 人能判断是否该做
* • AI 能知道不能做什么
* • 验收标准能被验证

## 阶段二：人工审查

目标：防止 AI 在错误方向上高效执行。

动作：

* • 确认需求方向
* • 删除范围外内容
* • 补充业务约束
* • 锁定验收标准

规则：

* • 未审查不进入实现
* • 审查只确认方向，不急着写代码
* • 有争议的需求回到提案阶段

## 阶段三：设计深化

目标：把已批准提案转成可执行计划。

方法：

* • `brainstorming`：找边界、风险、方案分歧
* • `writing-plans`：拆分为原子任务
* • TDD 设计：先定义测试和验证方式

输出：

* • `implementation-plan.md`
* • `test-plan.md`
* • 工单拆分方案

完成标准：

* • 每个工单能独立完成
* • 每个工单能独立验收
* • 工单之间依赖清楚
* • 每个工单有明确写入范围

## 阶段四：生成可执行工单

目标：把计划转成 Worker 可直接执行的输入。

工单模板：

```
---  
id: us-<change-id>-<task-id>  
title: <工单标题>  
priority: normal  
max_retries: 3  
---  
  
## Goal  
  
<本工单只做一件事>  
  
## Acceptance Criteria  
  
- [ ] <可验证条件>  
- [ ] <可验证条件>  
- [ ] <验证命令通过>  
  
## Constraints  
  
- Touch only <允许修改范围>  
- Do not modify <禁止修改范围>  
  
## Out of Scope  
  
- <本工单不做的事情>  
  
## Required Context  
  
- `specs/<相关规格>`  
- `<相关源码>`  
  
## Verification  
  
- `<命令>`
```

拆分规则：

* • 一个工单只解决一个目标
* • 一个工单最好只涉及少量文件
* • 一个工单必须能在一次或少数几次重试内完成
* • 一个工单不能同时包含设计、实现、迁移、文档、测试所有内容

需要拆分的信号：

* • 出现“顺便”
* • 出现“同时”
* • 出现多个模块
* • 出现多个验证命令
* • 修改范围无法提前声明
* • 一个 Worker 无法独立完成

## 阶段五：执行

目标：让 Worker 在受控范围内完成工单。

Worker 执行纪律：

* • 先读工单
* • 再读规格
* • 再读代码
* • 先补测试或确认测试
* • 做最小实现
* • 跑验证命令
* • 记录结果

Worker 禁止事项：

* • 不得扩大需求
* • 不得修改未授权文件
* • 不得跳过测试
* • 不得用口头声明代替证据
* • 不得在失败后无限循环

## 阶段六：验收

目标：用独立 Judge 判断工单是否完成。

Judge 输入：

* • 工单内容
* • 相关规格
* • git diff
* • 测试输出
* • 变更文件列表
* • Worker 执行记录

Judge 输出：

```
{  
  "verdict": "PASS",  
  "reason": "All acceptance criteria are met."  
}
```

或：

```
{  
  "verdict": "FAIL",  
  "reason": "Missing test for Asia/Tokyo timezone."  
}
```

PASS 条件：

* • 所有验收标准满足
* • verification 通过
* • 未越权修改
* • 未引入明显回归
* • 实现和规格一致

FAIL 条件：

* • 任一验收标准缺失
* • 测试失败
* • 修改范围越界
* • Worker 只声称完成但无证据
* • 实现偏离规格

## 阶段七：失败反馈与重试

目标：让失败变成下一轮明确输入。

失败时：

* • Judge 写出具体原因
* • 原因追加到工单
* • Dispatcher 开启下一轮
* • Worker 基于反馈修复

规则：

* • 每个工单有 `max_retries`
* • 超过重试上限则进入 `failed/`
* • failed 工单不得归档为完成
* • 失败原因必须可行动

## 阶段八：交付与归档

目标：把真实实现沉淀回项目知识。

交付前检查：

* • 所有工单 PASS
* • 所有测试通过
* • 残留风险已记录
* • 变更和提案一致

归档动作：

* • 更新 `specs/`
* • 记录最终实现
* • 删除临时设计假设
* • 保留有长期价值的约束

归档原则：

* • 只归档真实实现
* • 不归档未完成能力
* • 不把临时 workaround 写成长期规范

## 推荐目录

```
project/  
  specs/  
  openspec/  
    changes/  
      <change-id>/  
        proposal.md  
        tasks.md  
  
  .fusion/  
    inbox/  
    running/  
    done/  
    failed/  
    feedback/  
    runs/  
    traces/  
    reports/  
      verification-report.md  
  
  docs/  
    260504-spec-driven-ai-delivery-methodology.md
```

## 状态机

```
draft  
-> reviewed  
-> planned  
-> inbox  
-> running  
-> judging  
-> done  
-> archived
```

失败分支：

```
judging  
-> feedback  
-> inbox
```

失败终态：

```
feedback  
-> failed
```

## 角色分工

| 角色 | 负责什么 | 不负责什么 |
| --- | --- | --- |
| 人 | 审查方向、确认取舍 | 逐行指导实现 |
| OpenSpec | 规格、提案、归档 | 执行代码 |
| Planner | 设计深化、拆工单 | 直接写大段代码 |
| Dispatcher | 调度、重试、状态 | 判断业务价值 |
| Worker | 实现工单 | 扩大需求和最终验收 |
| Judge | 验收证据 | 替 Worker 修代码 |
| Reporter | 汇总结果 | 改变验收结论 |

## 模型分工

| 工作 | 推荐模型 |
| --- | --- |
| 需求澄清 | 强模型 |
| 规格提炼 | 强模型 |
| 工单拆分 | 强模型 |
| 明确 bugfix | 快模型 |
| 测试补齐 | 快模型 |
| 小范围重构 | 快模型 |
| 新功能实现 | 强模型或中强模型 |
| Judge | 稳定审查模型 |

原则：

* • 强模型做判断
* • 快模型做执行
* • Harness 做流程控制
* • Judge 做质量门禁

## 方法论检查清单

进入执行前：

* • [ ] 是否有明确 proposal
* • [ ] 是否有人工审查
* • [ ] 是否有验收标准
* • [ ] 是否已拆成可执行工单
* • [ ] 每个工单是否有修改范围
* • [ ] 每个工单是否有验证命令

执行中：

* • [ ] Worker 是否读取规格
* • [ ] 是否先补测试或确认测试
* • [ ] 是否只改授权范围
* • [ ] 是否记录 diff 和测试输出
* • [ ] 是否触发了 retry 或护栏

交付前：

* • [ ] 所有工单是否 PASS
* • [ ] verification 是否通过
* • [ ] 是否有失败工单
* • [ ] 是否有未说明风险
* • [ ] 是否可以 archive

## 最小可行版本

第一版只需要做到：

* • 手工或半自动生成 `.fusion/inbox/*.md`
* • Codex 作为 Worker 执行
* • 用 git diff 和测试输出做 Judge 输入
* • 生成 `.fusion/reports/verification-report.md`
* • PASS 后再 `/opsx:archive`

第一版不需要做到：

* • 自动多模型调度
* • 自动并行执行
* • 完整 UI
* • 自动归档
* • 替代 OpenSpec

## 方法论边界

适合：

* • 代码功能开发
* • bug 修复
* • 测试补齐
* • 小中型重构
* • 有明确验收标准的工程任务

不适合：

* • 需求本身还不清楚
* • 验收标准无法定义
* • 高度探索性研究
* • 一次性大爆炸重写
* • 没有测试或验证手段的关键系统变更

## 最终原则

```
先规格，后执行。  
先工单，后 Worker。  
先测试，后交付。  
先证据，后判断。  
先归档，后遗忘。
```

我是Ray觉得有启发？请点个「在看」或转发给同样在做 AI 自动化的朋友。喜欢请关注我，谢谢。