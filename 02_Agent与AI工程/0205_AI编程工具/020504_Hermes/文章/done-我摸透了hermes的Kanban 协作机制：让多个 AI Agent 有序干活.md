> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: 我摸透了hermes的Kanban 协作机制：让多个 AI Agent 有序干活
author: 花摇响铃铛
date: 云云云间云云云间
url: https://mp.weixin.qq.com/s?__biz=MzI3NjYxMDkzMg==&mid=2247484871&idx=1&sn=9d8b03c5a110e43a3c8287e6751b2726&chksm=eacd148857e9d675f9572d489f3711757b2161f2c72a49960f6c4effa002071347d27e25a946&mpshare=1&scene=24&srcid=0602f8Ox7Wg698yi1sEj1tdw&sharer_shareinfo=68cd59bc4d4b5b0ce83cd5607f6407e7&sharer_shareinfo_first=68cd59bc4d4b5b0ce83cd5607f6407e7#rd
---

> 我有6个Agent，想象一下：一个搜资料、一个写文章、一个出图、一个审查……所有 Agent 各司其职、自动接力，无需你反复催单。这就是 Kanban 协作机制。

Kanban 协作机制整体架构图

---

## 1. 什么是 Kanban 协作

Kanban 协作是 Hermes Agent 框架内置的多 Agent 调度系统。通过一块"共享白板"（SQLite 数据库），多个专业 Agent 有序配合，完成复杂工作流。

**核心能力：**

* **多 Agent 协作**： coder / writer / designer / researcher 各司其职
* **任务持久化**：存储在数据库，不受系统重启影响
* **依赖管理**：用 Parent/Child 链控制执行顺序
* **审计追踪**：所有操作记录在案，随时可追溯

---

## 2. 3 分钟快速上手

### 第一步：创建任务

```
hermes kanban create "写产品介绍" \
  --assignee writer \
  --body "保存到 workspace/files/文档/产品介绍.md" \
  --priority 50
```

### 第二步：派发任务

```
hermes kanban dispatch
```

调度器自动扫描所有就绪任务，按优先级排序启动对应 Agent。

### 第三步：查看进度

```
hermes kanban list          # 查看所有任务
hermes kanban tail <id>     # 实时跟踪
```

---

## 3. 三种协作模式

### 串联：写 → 审 → 改

适合文章、报告等需要多轮迭代的工作。

```
T1 墨客写初稿
 ↓
T2 包拯审查
 ↓
T3 墨客修改
```

```
# T1: 写初稿
T1=$(hermes kanban create "墨客写初稿" --assignee writer --priority 50 --json | python3 -c "import sys,json; print(json.load(sys.stdin)['id'])")

# T2: 审查（依赖 T1）
T2=$(hermes kanban create "包拯审查" --assignee reviewer --parent $T1 --priority 50 --json | python3 -c "import sys,json; print(json.load(sys.stdin)['id'])")

# T3: 修改（依赖 T2）
T3=$(hermes kanban create "墨客修改" --assignee writer --parent $T2 --priority 50 --json | python3 -c "import sys,json; print(json.load(sys.stdin)['id'])")
```

### 并联：多个 Agent 同时干

适合调研、出图、文案等独立任务。

```
hermes kanban create "探子调研" --assignee researcher --priority 50
hermes kanban create "画师出图" --assignee designer --priority 50
hermes kanban create "墨客写文案" --assignee writer --priority 50
hermes kanban dispatch  # 三任务同时运行
```

### Fan-out/Fan-in：并发 + 汇总

适合：调研 → 并行创作 → 最终汇总。

```
T1 探子搜资料
  ↓
T2 画师出图  ←┐
T3 墨客写文案←┤ 并行
              ↓
         T4 铁匠组装
```

三大协作模式示意图

---

## 4. Agent 角色介绍

| Profile | 角色 | 典型任务 |
| --- | --- | --- |
| researcher | 探子 | 调研、竞品分析、数据收集 |
| writer | 墨客 | 写文章、文档、脚本 |
| coder | 铁匠 | 开发功能、修复 Bug |
| designer | 画师 | PPT、插图、UI 设计 |
| reviewer | 包拯 | 代码审查、内容审核 |

---

## 5. 通信分层

| 层级 | 工具 | 用途 |
| --- | --- | --- |
| 飞书群 | `send_message` | 最终交付、用户可见 |
| Kanban Comment | `kanban_comment` | 审查意见、版本记录 |
| Mailbox | 内部机制 | Agent 间即时协调 |

三层通信架构图

---

## 6. 审查门机制

包拯（reviewer）的职责：**发现问题直接修复，不要只报告不改。**

**审查铁律：**

1. 发现问题直接修复文件
2. 包拯说通过才能交付
3. 驳回后重新派任务改
4. 审核意见写 Comment，方便追溯

审查门机制流程图

---

## 7. 常见陷阱

### 优先级形同虚设

代码按 priority 降序排列，但大多数任务 priority=0。

**解决**：创建任务时必须显式设置 `--priority 50`。

### Workspace 目录从不自动清理

缓存190 个目录共 286MB 永久累积。

**解决**：重要文件必须手动归档到 `workspace/files/`。

### 评论区几乎为空

176/185 done 任务零评论。审查反馈应写入 `kanban_comment`。

### Body 必须在 create 时写入

`hermes kanban edit --body` 不生效，需重新创建任务。

### Shell 中文 JSON 编码失败

`--body "中文..."` 报 `surrogates not allowed`。

**解决**：用 Python + shlex.quote() 代替。

任务生命周期状态图

---

## 8. 总结

Kanban 协作机制让多个 AI Agent 像流水线工人一样有序配合：

* **串联**：写 → 审 → 改，适合迭代型工作
* **并联**：多 Agent 同时干，适合独立任务
* **Fan-out/Fan-in**：并发 + 汇总，适合调研+创作场景

记住三个关键：

1. **显式设置 priority**（默认 50）
2. **审查意见写 Comment**（方便追溯）
3. **重要文件归档到 workspace/files/**（目录不自动清理）

---