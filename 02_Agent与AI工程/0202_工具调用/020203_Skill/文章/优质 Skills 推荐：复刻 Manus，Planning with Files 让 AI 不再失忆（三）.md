---
title: 优质 Skills 推荐：复刻 Manus，Planning with Files 让 AI 不再失忆（三）
author: 数据科学流水簿
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIzMDcwMDkzOA==&mid=2247488266&idx=3&sn=2988b1b4028d2474a3b61cd8cdf8d8de&chksm=e95c3c48c10a14074b3d60c68de5bab2c67f7766ee454782c2228a07fe0185da78db90005705&mpshare=1&scene=24&srcid=0420cDdhL3lV8TnisAMEWfCL&sharer_shareinfo=d6d3529fe1668c637cb8640644c08db6&sharer_shareinfo_first=d6d3529fe1668c637cb8640644c08db6#rd
---

# 优质 Skills 推荐：复刻 Manus，Planning with Files 让 AI 不再失忆（三）

前两篇介绍了 planning-with-files 的理念和工作流，这篇深度拆解 SKILL.md 的配置逻辑，以及 10 分钟上手指南。

## SKILL.md 深度拆解

很多人看到这个项目会以为它只是"提供了 3 个模板文件"。但真正起作用的是 `SKILL.md` 里的行为约束：它把"先计划、再执行、再验证"的流程，通过 hooks 和脚本变成可重复、可校验的机制。

### Frontmatter：这个 skill 先声明了什么能力边界

原版 `SKILL.md` 的头部包含这些关键信息：

```
---  
name: planning-with-files  
version: "2.10.0"  
description: Implements Manus-style file-based planning for complex tasks. Creates task_plan.md, findings.md, and progress.md. Use when starting complex multi-step tasks, research projects, or any task requiring >5 tool calls.  
user-invocable: true  
allowed-tools:  
  - Read  
  - Write  
  - Edit  
  - Bash  
  - Glob  
  - Grep  
  - WebFetch  
  - WebSearch  
hooks:  
  ...  
---
```

#### 为什么是 YAML 格式

这段 YAML 是所谓的 **frontmatter**（头部元数据）：用一段结构化配置告诉"skill 运行时"这份文件的身份、能力边界和触发机制。

YAML 在这里的优点主要有三点：

1. 1. **人读起来直观**：缩进 + 列表就能表达层级，比 JSON 更适合手写和审阅
2. 2. **机器解析稳定**：运行时可以明确读出 `name/description/allowed-tools/hooks` 等字段
3. 3. **配置与正文分离**：正文部分可以专注写"行为与流程"，配置部分专注写"元信息"

可以这么理解：YAML 是"skill 的说明书封面（元数据）"，下面的 Markdown 才是"正文（怎么做）"。

#### `allowed-tools` 这些动作是什么

这里的 `Read/Write/Edit/Bash/Glob/Grep/WebFetch/WebSearch` 对应的是 agent 环境里可用的"工具名"。

* • **它们是什么动作**：`Read` 代表读文件，`Edit/Write` 代表写入/修改文件，`Grep/Glob` 代表在项目内检索定位，`Bash` 代表执行命令
* • **动作背后发生什么**：skill 运行时会按"工具调用"来执行；`allowed-tools` 充当白名单，限制这份 skill 只能做这些类型的操作

**解读：**

* • `user-invocable: true`：意味着你可以手动触发它，而不是只能"被动命中"
* • `allowed-tools`：它把可用工具显式列出来，就是在做"能力白名单"
* • `description` 的触发条件写得很具体：`>5 tool calls`。这不是玄学，它是在避免把复杂流程用在简单任务上

### PreToolUse：每次重要动作前，先把目标塞回注意力窗口

原版配置：

```
PreToolUse:  
  - matcher: "Write|Edit|Bash|Read|Glob|Grep"  
    hooks:  
      - type: command  
        command: "cat task_plan.md 2>/dev/null | head -30 || true"
```

#### 这种 YAML 配置怎么"生效"

把 hooks 当成"生命周期回调"就行：在某些事件发生时自动跑一段动作。

* • **什么是 hooks**：在 `PreToolUse/PostToolUse/Stop` 这些时机触发的自动化动作
* • **matcher 是什么**：一个匹配规则（这里看起来像正则），用来判断"当前发生的工具调用是不是我关心的那类动作"
* • **如何触发**：当你准备执行一次工具调用，运行时先检查是否有 `PreToolUse` hook 的 matcher 命中；命中了，就先执行 `command`

这段配置的效果就是：每次你要做关键动作（写/改/跑命令/读/检索）之前，先把 `task_plan.md` 的前 30 行输出出来，强行把 Goal/当前阶段塞回注意力窗口。

**解读：**

* • `matcher` 覆盖了"会改变状态/会推动任务"的关键动作
* • hook 做的事很简单：每次先输出 `task_plan.md` 前 30 行
* • 这里的关键不是 30 行这个数字，而是"只读前面最关键的 Goal/Current Phase/前几个 phase"

普通用户怎么用这个思路：哪怕不用 hooks，也可以把它变成手动规则——"准备大改之前先读一遍 task\_plan 的 Goal"。

### PostToolUse：写完文件就提醒你更新阶段状态

原版配置：

```
PostToolUse:  
  - matcher: "Write|Edit"  
    hooks:  
      - type: command  
        command: "echo '[planning-with-files] File updated. If this completes a phase, update task_plan.md status.'"
```

**解读：**

* • 这不是"无意义的提示"，它是在对抗两个常见问题：

1. 1. 你写完了东西，但忘了把 phase 从 `in_progress` 改成 `complete`
2. 2. 你以为自己会记得更新，结果下一轮又忘

* • 这个 hook 不强制你更新，但把"提醒"变成每次写入后的默认行为

普通用户怎么借鉴：如果你不用 Claude Code，也可以在自己的流程里加一个"写完就更新状态"的固定动作。

### Stop Hook：用脚本把"是否完成"变成可计算的闸门

原版 Stop hook 会调用 `check-complete` 脚本。而 `check-complete.sh` 的核心逻辑很直接：

```
TOTAL=$(grep -c "### Phase" "$PLAN_FILE" || true)  
COMPLETE=$(grep -cF "**Status:** complete" "$PLAN_FILE" || true)  
  
if [ "$COMPLETE" -eq "$TOTAL" ] && [ "$TOTAL" -gt 0 ]; then  
  exit 0  
else  
  exit 1  
fi
```

**解读：**

* • 这一步是在对抗"80% 完成就宣布结束"。只要 phase 没全部 `complete`，就判定未完成
* • 它把"完成"从主观感受，变成了可验证规则
* • 对普通用户最有价值的启发：你也可以给任何交付设一个"硬闸门"，比如写文章必须满足：表格 + Mermaid + 代码块 + 参考文献齐全，才允许发布

### Session Recovery：`/clear` 后它怎么把"丢失上下文"捞回来

这个项目在 `SKILL.md` 里专门写了"先检查上一会话"的流程，并提供脚本：

* • `scripts/session-catchup.py`：生成 catchup 报告
* • 建议流程：先看 `git diff --stat`，再读 planning files，再把 catchup 要点同步回三个文件

**解读：**

* • 它承认"上下文一定会丢"，所以把恢复流程工程化：用脚本告诉你"丢了什么"
* • 这背后其实是一个更通用的思路：把重要状态写到可恢复介质里（文件/数据库），把聊天上下文当缓存

## 10 分钟上手指南

如果你只想最小成本体验一次这套 workflow，不要从"大项目"开始。挑一个 10–30 分钟能结束的小任务（写一段说明文、修一个小 bug、做一次调研）。

### 步骤清单

1. 1. **安装并触发 `/plan`**（或 `/planning-with-files:start`）
2. 2. **在项目根目录生成三文件**：`task_plan.md/findings.md/progress.md`
3. 3. **在 `task_plan.md` 写一句话 Goal**，并拆 3 个 phase（理解/执行/验证）
4. 4. **做两次阅读/搜索后**，立刻把要点写进 `findings.md`（2-Action Rule）
5. 5. **每做完一个 phase**：更新 `task_plan.md` 状态 + 在 `progress.md` 记录你做了什么
6. 6. **收尾时跑 `check-complete.sh`**，直到 phases 全部 complete

### 最小循环模板

```
Loop N:  
1) Read task_plan.md      # 把目标/当前阶段塞回注意力窗口  
2) 做一件事（搜索/读文件/写实现/写总结）  
3) Write/Edit files       # 把新发现/新状态写回文件系统
```

给新手的解读：

* • 这里的 `Read/Write/Edit/WebSearch` 就是在说"读计划/查资料/写笔记/更新状态"
* • 最关键的不是你用哪个文件名，而是保持职责稳定：计划（plan）负责阶段状态机，发现（notes/findings）负责证据与结论，过程（progress）负责可追溯日志

### 失败恢复示例

```
Action: Read config.json  
Error: File not found  
  
# Update task_plan.md:  
## Errors Encountered  
- config.json not found → Will create default config
```

**解读：** 失败不只是"需要修复的 bug"，它还是给 agent 的关键信号。你把失败写进文件，就等于把"新的世界模型"写进了可持久化记忆里。

## 总结

你会得到的不是"又一个模板"，而是一个可复用的习惯：以后任何复杂任务，你都能在三文件里 30 秒恢复状态并继续推进。

核心价值总结：

| 价值点 | 说明 |
| --- | --- |
| **持久化记忆** | 三文件作为外部存储，避免上下文丢失 |
| **目标对齐** | PreToolUse 强制回读，对抗目标漂移 |
| **可追溯过程** | progress.md 记录所有操作和错误 |
| **可验证完成** | Stop hook 用脚本校验，避免提前宣布完成 |
| **会话恢复** | /clear 后可通过脚本捞回丢失信息 |

---

**参考文献：**

1. 1. Planning with Files（项目仓库）
2. 2. planning-with-files SKILL.md
3. 3. Examples (skill-level): Planning with Files in Action
4. 4. Planning-with-Files 完全指南：复刻 Manus（源码七号站）