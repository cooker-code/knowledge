---
title: Hermes Agent 多代理Kanban看板实战：让AI团队像流水线一样协作
author: 悟果AI
date: 悟果AI悟果AI
url: https://mp.weixin.qq.com/s?__biz=MzY4NjI4MDQwMg==&mid=2247484148&idx=1&sn=95cd92fd06a9567b453b2b9dbe95fcd4&chksm=f2dfd36968d6f73c20de3c372508fd825b57f6c80cfd545622249a6838756b8e3a19d8863ef0&mpshare=1&scene=24&srcid=06022HnRgwgEoollgPtE91A8&sharer_shareinfo=141e098a29fbba532e6dda10955ab2ec&sharer_shareinfo_first=141e098a29fbba532e6dda10955ab2ec#rd
---

## 核心要点

* Kanban 系统将多代理任务协作可视化，六列看板实时追踪任务状态
* 父子依赖关系自动管理，子任务在父任务完成前保持等待
* 结构化交接机制：summary + metadata 传递上下文，下游 worker 不用重复读文档
* 熔断器 + 崩溃恢复双保险，防止坏任务占用调度资源
* 一个命令启动网关，调度器自动拾取就绪任务，真正"启动后不管"

## 背景：为什么需要多代理看板

用过 Hermes 的同学可能有过这种体验：让 AI 做一个复杂项目——比如搭建用户认证系统——你会怎么分任务？

通常的做法是：一条一条 prompt 发，让 AI 依次完成"设计数据库"、"写 API"、"写测试"。问题是：这些任务之间有依赖关系（API 依赖数据库设计），但 AI 不知道上一个任务产出了什么，结果往往是"各干各的"，最后还得人工缝合。

v0.13.0 发布的 Multi-agent Kanban 就是来解决这个问题的。它把一个项目的多步骤任务做成可视化的看板，每个步骤有明确的状态（待办、进行中、已完成），下游任务自动读取上游产出物，实现真正的流水线协作。

**一句话总结：Kanban 让你的多个 AI Worker 像工厂流水线一样协作，上下游无缝交接。**

## 环境准备

### 前置条件

| 要求 | 说明 |
| --- | --- |
| Hermes Agent | v0.13.0 或更新版本 |
| 依赖项 | SQLite3（系统自带）、watch（Unix）/sleep（跨平台） |
| 仪表盘端口 | 9119（如被占用需配置） |
| 磁盘空间 | 看板数据库 `~/.hermes/kanban.db` 增长缓慢，1GB 足够 |

### 安装/升级 Hermes Agent

```
# 升级到最新版（确保包含 Kanban 功能）  
pip install hermes-agent --upgrade  
  
# 验证版本  
hermes --version  
# 应输出: hermes-agent 0.13.0 或更高  
  
# 首次使用前初始化看板数据库（可选）  
hermes kanban init
```

### 启动看板仪表盘

```
# 在浏览器中打开看板界面  
hermes dashboard  
  
# 仪表盘地址：http://127.0.0.1:9119  
# 左侧导航栏点击 "Kanban" 进入
```

> 💡 **提示**：仪表盘与 CLI 共享同一个 SQLite 数据库，两个界面操作实时同步。用 CLI 或浏览器操作都行。

## 看板六列状态

| 状态 | 说明 | 触发条件 |
| --- | --- | --- |
| **Triage** | 待分类 | 任务刚创建，规格待完善 |
| **Todo** | 待办 | 已创建但依赖未满足或未分配 |
| **Ready** | 就绪 | 已分配，等待调度器认领 |
| **In progress** | 进行中 | 工作者正在执行 |
| **Blocked** | 受阻 | 需要人工输入，或熔断器触发 |
| **Done** | 已完成 | 任务成功完成 |

## 场景一：单人开发者的功能交付（父子依赖）

这是最常用的场景：一个大任务拆成多个子任务，按依赖顺序执行。

**目标**：完成用户认证模块，拆成"设计数据库 → 实现 API → 编写测试"三步。

### 步骤一：创建数据库设计任务

```
# 创建父任务（数据库设计）  
SCHEMA_ID=$(hermes kanban create "Design auth schema" \  
    --assignee backend-dev \  
    --tenant auth-project \  
    --priority 2 \  
    --body "设计认证模块的数据模型：用户表、会话表、令牌表" \  
    --json | jq -r .id)  
  
echo "Schema任务ID: $SCHEMA_ID"
```

### 步骤二：创建API任务（依赖数据库设计）

```
# 创建API任务，--parent 指定依赖  
API_ID=$(hermes kanban create "Implement auth API endpoints" \  
    --assignee backend-dev \  
    --tenant auth-project \  
    --priority 2 \  
    --parent $SCHEMA_ID \  
    --body "实现注册、登录、刷新令牌、登出接口" \  
    --json | jq -r .id)  
  
echo "API任务ID: $API_ID"
```

### 步骤三：创建测试任务（依赖API）

```
# 创建测试任务，依赖API  
hermes kanban create "Write auth integration tests" \  
    --assignee qa-dev \  
    --tenant auth-project \  
    --priority 2 \  
    --parent $API_ID \  
    --body "覆盖正常流程、密码错误、令牌过期、并发刷新等场景"
```

### 步骤四：完成任务并交接

```
# 认领并完成数据库设计任务  
hermes kanban claim $SCHEMA_ID  
# ... AI 执行数据库设计 ...  
  
hermes kanban complete $SCHEMA_ID \  
    --summary "users(id, email, pw_hash), sessions(id, user_id, jti, expires_at); refresh tokens stored as sessions with type='refresh'" \  
    --metadata '{  
        "changed_files": ["migrations/001_users.sql", "migrations/002_sessions.sql"],  
        "decisions": ["bcrypt for hashing", "JWT for session tokens", "7-day refresh, 15-min access"]  
    }'
```

### 关键效果

执行 `hermes kanban complete` 后会发生两件事：

1. `SCHEMA_ID` 任务进入 **Done** 状态
2. `API_ID` 任务自动从 **Todo** → **Ready**，调度器开始拾取

下游的 API worker 打开任务时，会自动看到：

* 父任务的 summary（数据库设计产出）
* 父任务的 metadata（变更文件列表、设计决策）

**不需要再去翻文档，AI 直接拿到上一步的产出物。**

## 场景二：批量任务处理（舰队视图）

多个 worker 并行处理独立任务池，最简单的看板用例。

### 创建批量任务

```
# 创建翻译任务池  
for lang in Spanish French German Japanese Korean; do  
    hermes kanban create "Translate homepage to $lang" \  
        --assignee translator \  
        --tenant content-ops  
done  
  
# 创建转录任务池  
for i in $(seq 1 10); do  
    hermes kanban create "Transcribe Q3 customer call #$i" \  
        --assignee transcriber \  
        --tenant content-ops  
done
```

### 启动网关调度

```
# 一键启动网关，调度器自动拾取所有 Ready 任务  
hermes gateway start
```

**实际效果**：

* 三个 translator worker 并行处理 5 个翻译任务
* 一个 transcriber worker 按顺序处理 10 个转录任务
* "进行中"列按角色分组（"按角色分道"）
* 任务完成后自动调度下一个就绪任务

### 查看看板状态

```
# 实时监控任务状态  
hermes kanban watch --kinds completed,gave_up,timed_out  
  
# 查看具体任务详情  
hermes kanban show <任务ID>  
  
# 查看任务运行历史（包含每次尝试的摘要）  
hermes kanban runs <任务ID>
```

## 场景三：多角色流水线（阻塞与重试）

产品经理 → 工程师 → 审查者的标准流水线，支持任务阻塞与迭代重试。

### 步骤一：PM 完成规格

```
# PM 完成任务，写入验收标准  
hermes kanban complete $SPEC_ID \  
    --summary "POST /forgot-password 发送邮件，GET /reset/:token 渲染表单，POST /reset 应用新密码" \  
    --metadata '{"acceptance": [  
        "过期token返回410",  
        "重复使用最近3个密码返回400",  
        "成功重置会失效所有活跃会话"  
    ]}'
```

### 步骤二：工程师认领并实现

```
hermes kanban claim $IMPL_ID  
# ... AI 实现功能 ...
```

### 步骤三：审查者发现缺陷，阻塞任务

```
hermes kanban block $IMPL_ID "审查意见：缺少密码强度检查，重置链接不是一次性使用"
```

### 步骤四：工程师迭代修复

```
# 解除阻塞  
hermes kanban unblock $IMPL_ID  
  
# 重新认领  
hermes kanban claim $IMPL_ID  
  
# 完成第二次迭代  
hermes kanban complete $IMPL_ID \  
    --summary "添加了 zxcvbn 强度检查，重置 token 现在是一次性使用" \  
    --metadata '{  
        "changed_files": ["auth/reset.py", "auth/tests/test_reset.py"],  
        "tests_run": 11,  
        "review_iteration": 2  
    }'
```

**关键价值**：每次运行都记录在 `task_runs` 表中，审查者可以完整看到：之前尝试了什么、为什么被阻塞、这次改了什么。

## 场景四：熔断器与崩溃恢复

### 熔断器（永久性失败）

场景：部署任务因缺少 AWS 凭据连续失败。

```
hermes kanban create "Deploy to staging" \  
    --assignee deploy-bot \  
    --tenant ops
```

**熔断流程**：

| 次数 | 状态 | 说明 |
| --- | --- | --- |
| 第1次 | ready → in\_progress → blocked | spawn 失败，释放认领 |
| 第2次 | ready → in\_progress → blocked | 再次失败，失败计数+1 |
| 第3次 | gave\_up | 连续3次失败，进入熔断，不再重试 |

**配置熔断阈值**（可选）：

```
# 默认 failure_limit=3，可自定义  
hermes kanban create "Task name" \  
    --assignee worker \  
    --failure-limit 5
```

### 崩溃恢复（Worker 中途死亡）

场景：Worker 因 OOM 被杀死，自动回收任务。

```
# 运行1 - crashed  
错误：OOM kill at row 2.3M (process 99999 gone)  
任务状态：ready（等待重新调度）  
  
# 运行2 - completed  
元数据："strategy": "chunked with LIMIT + WHERE id > last_id"
```

**关键点**：

* 调度器通过 `kill(pid, 0)` 检测死掉的 worker
* 死掉的任务回到 `ready` 状态
* 重试 worker 能看到崩溃原因，自动选择更安全的策略（如分批处理）

## 核心配置

### 看板数据库位置

```
# 默认位置  
~/.hermes/kanban.db  
  
# 自定义路径  
export HERMES_KANBAN_DB=/path/to/kanban.db
```

### 通知配置（可选）

```
# 订阅任务完成通知  
hermes kanban notify-subscribe <任务ID> \  
    --platform telegram \  
    --chat-id <你的TelegramChatID>
```

### 常用命令速查

| 命令 | 作用 |
| --- | --- |
| `hermes kanban create "任务" --assignee xxx` | 创建任务 |
| `hermes kanban claim <ID>` | 认领任务 |
| `hermes kanban complete <ID> --summary xxx` | 完成任务 |
| `hermes kanban block <ID> "原因"` | 阻塞任务 |
| `hermes kanban unblock <ID>` | 解除阻塞 |
| `hermes kanban show <ID>` | 查看详情 |
| `hermes kanban runs <ID>` | 查看运行历史 |
| `hermes kanban watch` | 实时监控 |
| `hermes gateway start` | 启动调度器 |

## 总结与行动建议

**本文要点回顾**：

1. **父子依赖自动管理**：创建任务时用 `--parent` 指定依赖，调度器自动控制子任务的 Ready 时机
2. **结构化交接是核心**：`--summary` 和 `--metadata` 让下游 worker 直接拿到上游产出，不用重复读文档
3. **熔断器防止资源浪费**：连续失败的任务自动熔断，不再占用调度资源
4. **崩溃恢复无缝衔接**：Worker 死亡后任务自动回收，重试时能读取之前的失败原因
5. **一个命令启动不管**：`hermes gateway start` 后调度器自动拾取任务，真正的"启动后不管"

**下一步行动**：

1. **立即体验**：升级到 v0.13.0+，运行 `hermes kanban init && hermes dashboard`
2. **小规模试用**：选一个小项目，用父子依赖拆成 2-3 个任务，体验交接流程
3. **批量任务实战**：用舰队视图处理一批翻译/转录/数据清洗任务
4. **配置通知**：接入 Telegram/Slack，任务完成/阻塞时收到通知

**相关资源链接**：

| 资源 | 链接 |
| --- | --- |
| Hermes Agent GitHub | https://github.com/NousResearch/hermes-agent |
| Kanban 概览文档 | https://hermes-doc.aigc.green/user-guide/features/kanban |
| v0.13.0 Release Notes | https://github.com/NousResearch/hermes-agent/releases |
| 持久目标 (/goal) | https://hermes-doc.aigc.green/user-guide/features/goals |