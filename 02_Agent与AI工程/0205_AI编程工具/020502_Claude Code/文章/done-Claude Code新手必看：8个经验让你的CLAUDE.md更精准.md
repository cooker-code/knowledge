> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code新手必看：8个经验让你的CLAUDE.md更精准
author: 代码麻辣烫
date: 一叶扁舟一叶扁舟
url: https://mp.weixin.qq.com/s?__biz=Mzk0NjY0MTc0NA==&mid=2247500033&idx=1&sn=48abc50a354ff78b40837f3acbb6f295&chksm=c2589e043b137cf26011b72900fffcd378238c6b53930ff9b4590f10dca3307fc4905956b7aa&mpshare=1&scene=24&srcid=0509ywjlS7yhWhRjhRyGTyhS&sharer_shareinfo=fc80198a1ef23d24bc54da69fa840284&sharer_shareinfo_first=fc80198a1ef23d24bc54da69fa840284#rd
---

# 精简至上，CLAUDE.md 应控制在200行以内

**「反直觉点：」** 许多人认为，提供给Claude的上下文信息越丰富，它就能越好地理解项目需求。然而，过多的信息反而会导致Claude在大量数据中迷失方向，无法抓住关键点。

**「专家建议：」** claude-code-best-practice 的作者 Boris Cherny 建议，CLAUDE.md 的内容不应超过200行。这是因为Claude在每次会话中都会加载CLAUDE.md，过多的内容会占用上下文窗口，影响其对代码的理解。

**「实战标准：」**

```
# ❌ 不要这样
## 项目历史
2023年，我们的CTO在hackathon上提出了这个想法...
（300行的公司叙事+营销文案）

# ✅ 要这样
## Project Overview
B2B分析仪表盘，面向运营经理。
核心目标：缩短「从数据到洞察」的时间。
优化优先级：加载速度>交互丰富度>视觉花哨。
```

**「验证标准：」** 一个未接触过项目的新人，在阅读CLAUDE.md后能否在30秒内回答出产品是什么、技术栈有哪些、新代码应该放在哪里？

# 明确“禁止”和“允许”的技术栈

**「反直觉点：」** 即使列出了技术栈，Claude也可能因为不了解项目的历史和技术债务，而引入与项目不兼容的技术方案。

**「技术栈明确性：」** 没有“禁止清单”的CLAUDE.md是危险的。Claude可能会出于好意引入它“知道”的最优方案，但这可能与项目的需求完全冲突。

**「示例：」**

```
## Tech Stack
- Next.js 15 App Router + TypeScript
- Tailwind CSS + shadcn/ui
- Supabase（认证+数据）

Do NOT introduce unless explicitly requested:
- Redux（项目已迁移到React Context+Zustand）
- styled-components（全站Tailwind，不接受CSS-in-JS）
- Material UI（与shadcn/ui样式冲突）
- MongoDB（数据层已锁定PostgreSQL）
```

这条规则的价值在于，它不仅节省了纠正错误的时间，还避免了Claude在未被察觉的情况下引入不兼容的依赖，导致后续会话中不断修复兼容性问题。

# 规则需要具体可执行，而非抽象感受

**「反直觉点：」** “写出干净的代码”听起来是一个好规则，但对于AI来说，这样的规则太过抽象，无法执行。

**「具体规则：」** Claude理解的是具体的编程规则，如“使用named export而不是default export”、“组件不超过200行”、“async/await不使用then链”。

**「对比：」**

```
# ❌ 模糊——Claude无法执行
## Coding Rules
- 写出干净的代码
- 保持简洁
- 注重性能

# ✅ 具体——Claude可以直接执行
## Coding Rules
- 使用named export（路由文件除外）
- 禁止any类型，用泛型或接口替代
- 单个组件不超过200行（有充分理由可超）
- async/await替代Promise链
- 变量名全拼，不缩写（除id/url/ctx）
- 只在意图不明显时写注释
- 不留注释掉的代码块或console.log
```

**「测试方法：」** 读完规则后，能否在5秒内判断一段代码是否符合规则？如果可以，规则就是合格的；如果不可以，需要重新编写。

# CLAUDE.md 作为信息指针，非存储库

**「反直觉点：」** 许多人试图将所有架构文档塞进CLAUDE.md，但CLAUDE.md的职责并不是存储信息，而是告诉Claude去哪里找信息。

**「信息指针：」** 这是区分顶级用户和普通用户的一个标准。普通用户的CLAUDE.md是知识梳理，而顶级用户的CLAUDE.md则是一个router。

**「示例：」**

```
## Project Context
- 架构总览：`docs/architecture.md`
- 工程设计决策记录：`docs/adrs/`
- API文档：`docs/api.md`
- 部署流程：`docs/deploy.md`
```

Claude不需要在CLAUDE.md里读完所有架构文档，它只需要知道“我需要架构信息时，打开docs/architecture.md”。

**「渐进式上下文（Progressive Disclosure）：」**

```
## Context Tiers
Tier 1（每次加载）：CLAUDE.md — 项目是什么+怎么工作
Tier 2（按需加载）：docs/architecture.md, docs/api.md — Claude工作时自动读取
Tier 3（忽略）：docs/archive/ — 除非明确要求，不碰
```

这样Claude就不会在无关请求时浪费上下文去读历史文档，但在需要时知道去哪里找。

# 为敏感模块设置“本地CLAUDE.md”

**「反直觉点：」** CLAUDE.md只有一个，放在根目录。但某些模块的风险比其他模块高得多。

**「本地CLAUDE.md：」** 在src/auth/、src/payments/、infra/下面各放一个本地CLAUDE.md，Claude在操作这些目录时会自动加载。这就像给危险区域装上护栏。

**「示例：」**

```
# src/auth/CLAUDE.md

## 安全红线
- 绝不修改token验证逻辑，除非明确要求且经过review
- 绝不引入新的认证方式而不更新测试
- 所有认证相关变更必须通过`pnpm test src/auth`全部测试

## 已知陷阱
- Magic link生成依赖`crypto.randomUUID()`，不要换成其他随机方法
- Session存储在Redis，不是内存——重启不会丢失
```

# 利用CLAUDE.md驱动Hook，而非依赖记忆

**「反直觉点：」** 你写了测试规则，但Claude写完代码从来不跑测试——因为它忘了。

**「Hook的可靠性：」** Claude的记忆不可靠，但Hook是可靠的。把CLAUDE.md里的规则变成Hook的触发条件。

**「示例：」**

```
## Hooks & Quality Gates
以下规则由`.claude/hooks/`强制执行，不是提醒：
- 每次编辑后自动格式化（PreToolUse hook → prettier）
- 核心模块变更后自动跑测试（PostToolUse hook → vitest related）
- 禁止直接编辑`src/auth/`、`src/billing/`、`prisma/migrations/`而不先确认
```

**「对应Hook示例：」**

```
// .claude/hooks/pre-tool-use.json
{
  "hooks": [
    {
      "matcher": "Edit|Write",
      "command": "npx prettier --write ${CLAUDE_FILES}",
      "on_failure": "warn"
    }
  ]
}
```

Hook是CLAUDE.md规则的**「强制执行层」**。写在CLAUDE.md里的规则是“请记住”；配了Hook的规则是“你必须”。

# 利用CLAUDE.md建立长期记忆回路

**「反直觉点：」** 每次新会话，Claude像失忆一样重新认识你的项目。但你不需要一个复杂的向量数据库来解决这个问题。

**「MEMORY.md：」** 在CLAUDE.md里加一条指令，让Claude自己维护一个MEMORY.md：

```
# CLAUDE.md中加入

## Memory
`MEMORY.md`记录了之前任务中发现的关键洞察、最佳实践和已知陷阱。
每次新任务开始前，先读取MEMORY.md。
每次任务结束后，如果有新的发现
```

这比任何“长期记忆MCP”都简单、可控、可Git追踪。成本：一个文件。收益：Claude在跨会话时保留下文中最有价值的那5%。

# 用CLAUDE.md代替会话的“开场白”

**「反直觉点：」** 你应该训练Claude，不是每次问它“你能帮我做X吗”。你应该让CLAUDE.md承载你的工作风格，让Claude在第一次对话时就知道你讨厌什么。

**「实战总结：」** 一个优秀的CLAUDE.md里应该有“你是谁”和“你讨厌什么”：

```
## My Working Style
- 先给方案，不要直接写代码
- 不确定时列出选项，不要猜测
- 重大变更前先问，小优化可以直接执行
- 不要用“Great question!”、“I'd be happy to help!”这类废话
- 回复用中文，代码注释用英文
- 文件路径用绝对路径，不要相对路径
```

这6行省掉了你每次新会话的前5条消息。Claude从第一句就知道你在乎什么、讨厌什么、期望什么交互节奏。

# 一张表总结

| 序号 | 经验点 | 描述 |
| --- | --- | --- |
| 1 | 精简至上 | CLAUDE.md应控制在200行以内 |
| 2 | 明确技术栈 | 明确“禁止”和“允许”的技术栈 |
| 3 | 具体规则 | 规则需要具体可执行，而非抽象感受 |
| 4 | 信息指针 | CLAUDE.md作为信息指针，非存储库 |
| 5 | 本地CLAUDE.md | 为敏感模块设置“本地CLAUDE.md” |
| 6 | 驱动Hook | 利用CLAUDE.md驱动Hook，而非依赖记忆 |
| 7 | 长期记忆回路 | 利用CLAUDE.md建立长期记忆回路 |
| 8 | 会话开场白 | 用CLAUDE.md代替会话的“开场白” |

# 行动指南

1. 打开你的CLAUDE.md，删到200行以内——不删的，不值得留。
2. 加一个“Do NOT introduce”区块，列出至少3个禁用的库。
3. 把每一条模糊规则改成具体可验证的指令。
4. 给最敏感的模块（auth/billing/infra）各加一个本地CLAUDE.md。
5. CLAUDE.md不是一次写完就放那的文件。它是活的——你每发现一个Claude反复踩的坑、每总结一条有效的规则，都应该更新进去。一个月后回头看，你会发现Claude从一个菜鸟实习生，变成了真正懂你项目的高级工程师。