---
title: Anthropic《CLAUDE.MD编写完全指南(初始化 · 结构 · 最佳实践 · 实战样例)》
author: 金小牛柿
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzMzM4MDkwMA==&mid=2247483878&idx=1&sn=e7f81aeecf45be7fd7bbe97305fae6c8&chksm=c3817d728d8eef30e8bf59187c09d1c33e98dddaa3ef082618682c225b7b082670550cd69498&mpshare=1&scene=24&srcid=0327nbCCMVOoq6obTwM68ZsU&sharer_shareinfo=c8ed365118acda182dc68caa8d2ffa1c&sharer_shareinfo_first=c8ed365118acda182dc68caa8d2ffa1c#rd
---

CLAUDE.MD

编写完全指南

The Definitive Guide to Writing CLAUDE.md

 

初始化 · 结构 · 最佳实践 · 实战样例

2026.03 | 金小牛柿

# ■ 一、先写全局还是项目级？

答案是：先写项目级 CLAUDE.md，它是最重要的。全局 CLAUDE.md 是锦上添花。

原因很简单：项目级 CLAUDE.md 解决的是「这个项目怎么写代码」的问题，而全局 CLAUDE.md 解决的是「我个人喜好什么」的问题。前者的价值远大于后者。

|  |  |  |
| --- | --- | --- |
| 顺序 | 操作 | 说明 |
| ① | 项目根目录 /init | 自动生成项目级 CLAUDE.md，再根据实际情况精调 |
| ② | 创建 ~/.claude/CLAUDE.md | 添加个人偏好（语言、风格、全局规则） |
| ③ | 子目录 CLAUDE.md（可选） | Monorepo 或大型项目按模块拆分 |

💡 优先级：子目录 > 项目根目录 > 全局。更具体的覆盖更通用的。Claude 从当前工作目录向上递归读取。

────  ◆  ────

# ■ 二、如何初始化：/init 命令

在任意项目目录下启动 Claude Code，输入 /init 即可自动生成 CLAUDE.md：

cd your-project  
claude  
# 进入 Claude Code 后执行：  
/init

## ▸ /init 做了什么？

Claude 会自动分析你的代码库：

|  |  |
| --- | --- |
| 分析内容 | 具体操作 |
| 读取包管理文件 | package.json, requirements.txt, go.mod, Cargo.toml 等 |
| 扫描现有文档 | README.md, 现有配置文件 |
| 检测框架和工具 | Next.js, Flask, Jest, pytest, ESLint 等 |
| 分析目录结构 | 识别 src/, tests/, docs/ 等常见模式 |
| 推断编码规范 | 从现有代码中提取风格模式 |

## ▸ /init 之后做什么？

|  |  |
| --- | --- |
| # | 操作 |
| ① | 审查生成内容的准确性，修正不对的地方 |
| ② | 添加 Claude 无法推断的内容（分支策略、部署流程、踩坑记录） |
| ③ | 删除不适用的通用建议 |
| ④ | 提交到版本控制，让团队共享 |

 

💡 已有 CLAUDE.md 的项目也可以再次运行 /init，Claude 会审查现有文件并建议改进。

💡 工作中随时按 # 键可快速添加指令到 CLAUDE.md，随用随记。

────  ◆  ────

# ■ 三、CLAUDE.md 包含哪些部分？

根据官方文档和社区最佳实践，一个优秀的项目级 CLAUDE.md 通常包含以下七大模块：

|  |  |  |  |
| --- | --- | --- | --- |
| # | 模块名称 | 内容说明 | 为什么重要 |
| ① | 项目概述 | 一段话说清项目是什么、用什么技术栈 | 让 Claude 知道「我在哪个战场」 |
| ② | 关键目录结构 | 主要目录及其职责 | 让 Claude 知道「东西放在哪」 |
| ③ | 常用命令 | 开发/构建/测试/部署命令 | 让 Claude 能自己跑和验证 |
| ④ | 编码规范 | 代码风格、命名、测试要求 | 保证生成代码和项目一致 |
| ⑤ | 工作流程 | 任务执行步骤和审批要求 | 避免 Claude 乱来，按流程办事 |
| ⑥ | 注意事项/踩坑 | 关键规则、禁止事项、常见错误 | 避免重复犯错 |
| ⑦ | 工具集成 | 自定义工具、MCP 服务器、CI/CD | 让 Claude 能调用你的工具链 |

⚠️ CLAUDE.md 建议控制在 200 行以内。前沿模型稳定跟踪约 150-200 条指令，Claude Code 自身系统提示已占用约 50 条，留给你的「指令预算」约 100-150 条。太长会导致 Claude 均匀地忽略所有指令！

────  ◆  ────

# ■ 四、每一部分的最佳实践与样例

## ▸ ① 项目概述 —— 一段话说清「我是谁」

一段话说明项目是什么、用什么技术栈、服务于什么场景。不要写太长。

最佳实践：

• 控制在 3-5 行以内

• 包含核心技术栈（语言 + 框架 + 数据库 + 部署）

• 说明项目的业务场景

# 项目概述  
  
内部运营数据平台，前后端分离架构。  
后端: Python 3.12 + FastAPI + PostgreSQL + SQLAlchemy 2.0 (async)  
前端: SoybeanAdmin (Vue3 + Naive UI + TypeScript)  
部署: Docker + Docker Compose，Alibaba Cloud ECS

## ▸ ② 关键目录结构 —— 给 Claude 一张地图

最佳实践：

• 只列关键目录（不要全量 tree）

• 每个目录加一句说明职责

• Monorepo 特别重要：说清哪个应用是什么

## 关键目录  
  
- `app/api/v1/`       — FastAPI 路由 (APIRouter)  
- `app/models/`       — SQLAlchemy ORM 模型  
- `app/schemas/`      — Pydantic 请求/响应模型  
- `app/services/`     — 业务逻辑层  
- `app/tasks/`        — APScheduler 定时任务  
- `app/utils/`        — 通用工具（企微推送、邮件解析）  
- `tests/`            — pytest + httpx AsyncClient

## ▸ ③ 常用命令 —— 让 Claude 能自己动手

最佳实践：

• 包含开发/构建/测试/lint 的完整命令

• 如果 hook 已自动执行某些操作，明确标注「不要手动做」

• 优先写单个测试命令（而非全量套件）

## 常用命令  
  
```bash  
uvicorn app.main:app --reload       # 启动开发服务器  
pytest tests/ -v                    # 运行全部测试  
pytest tests/test\_fund.py -k sync   # 单个测试（优先用这个）  
ruff check app/                     # 代码检查  
alembic upgrade head                # 数据库迁移  
docker compose up -d                # 启动所有服务  
```

## ▸ ④ 编码规范 —— 保证风格一致

最佳实践：

• 只写 Claude 无法从代码推断的规则

• 能用 linter/formatter 解决的不要写进 CLAUDE.md（用 hook 代替）

• 用指向而非复制：引用文件:line 而非贴代码片段

• 重要规则用 IMPORTANT 或 MUST 标记以提高遵守率

## 编码规范  
  
- IMPORTANT: 全局使用 async/await，数据库用 AsyncSession  
- 所有函数必须有 type hints  
- 日志用 loguru，禁止 print()  
- MUST: 数据库操作用 SQLAlchemy ORM，禁止裸 SQL  
- 外部 HTTP 用 httpx.AsyncClient，禁止 requests  
- 新接口必须写 pytest 测试

⚠️ 注意：不要用 CLAUDE.md 做 linter 的工作。LLM 比 linter 慢且贵。代码格式化应该交给 PostToolUse hook 自动执行。

## ▸ ⑤ 工作流程 —— 让 Claude 先想再做

最佳实践：

• 定义「做事前四问」：是否需要调研、是否需要计划、缺啥信息、怎么验证

• 为不同任务类型定义不同流程

• 包含 Git 分支、提交消息规范

## 工作流程  
  
### 新功能开发  
1. 创建分支: feat/功能名  
2. 先写测试（TDD）  
3. 实现功能  
4. 确保测试通过 + ruff check 无警告  
5. commit message 格式: feat: 一句话说明  
  
### 修 Bug  
1. 先复现问题（写失败测试）  
2. 修复代码让测试通过  
3. commit message 格式: fix: 一句话说明

## ▸ ⑥ 注意事项/踩坑 —— 每条都是血泛教训

最佳实践：

• 每次 Claude 犯错，加一条规则（复利工程）

• 用粗体或 IMPORTANT/MUST 标记关键规则

• 说明「不要做什么」往往比「要做什么」更有效

## 注意事项  
  
- IMPORTANT: 所有 API 路由用 /api/v1 前缀，不要省略  
- MUST: 修改 model 后必须生成 alembic migration，不要手动改表  
- 禁止直接修改 app/tasks/ 下的生产 APScheduler 调度配置  
- .env 文件不要提交到 git  
- Docker 环境下数据库连接用服务名 db，非 localhost

✅ 「复利工程」核心思想：每次 Claude 犯错 → 加一条 CLAUDE.md 规则 → 同类错误永远不再发生。随时间累积，价值复利增长。

## ▸ ⑦ 工具集成 —— 让 Claude 调用你的工具链

最佳实践：

• 说明自定义工具的用法和场景

• 提示工具有 --help 可查

• MCP 服务器的使用约束

## 工具集成  
  
### 外部数据获取  
- `app/services/wind\_client.py` — Wind API 封装，httpx.AsyncClient  
- `app/services/akshare\_client.py` — AKShare 数据拉取  
- 数据存入 PostgreSQL，通过 SQLAlchemy AsyncSession  
  
### 定时任务 (APScheduler)  
- 任务定义在 app/tasks/\*.py  
- 调度配置在 app/core/scheduler.py  
- 任务失败自动通过企微告警  
  
### 企业微信推送  
- 仅用于日报、异常告警、重要通知  
- 调用 app/utils/wechat.py 中的 send\_message()

────  ◆  ────

# ■ 五、全局 CLAUDE.md 怎么写？

全局 CLAUDE.md 位于 ~/.claude/CLAUDE.md，对所有项目生效。它应该只包含「真正全局」的个人偏好，而非项目特定内容。

适合放在全局 CLAUDE.md 的内容：

• 回复语言偏好（如「优先中文回复」）

• 代码注释语言（如「注释用英文」）

• 通用编程风格（如「Python 必须用 type hints」）

• 个人工作习惯（如「修改代码后必须跑测试」）

# ~/.claude/CLAUDE.md  
  
## 语言偏好  
- 优先使用中文回复  
- 代码注释和 commit message 用英文  
  
## 通用编码规则  
- Python: 全局 async/await，type hints，loguru 日志  
- HTTP 请求用 httpx.AsyncClient，禁止 requests  
- 禁止 print() 调试，用 logger.debug()  
- 修改代码后必须跑测试验证  
  
## 个人习惯  
- 复杂任务先制定计划再动手  
- Git: conventional commits (feat/fix/chore)

⚠️ 全局 CLAUDE.md 要尽量精简（10-30 行即可）。它每次会话都会加载，占用宝贵的指令预算。

────  ◆  ────

# ■ 六、完整实战样例

## ▸ 样例 A：项目级 CLAUDE.md（后端）

适用于：FastAPI + PostgreSQL + SoybeanAdmin 前后端分离项目的后端仓库。

# 内部数据平台 — 后端  
  
内部运营数据平台，涵盖日报、投研、交易、客服、路演等 14+ 业务模块。  
前后端分离架构，前端为 SoybeanAdmin (Vue3 + Naive UI)。  
  
## 技术栈  
- Python 3.12+ / FastAPI / Uvicorn  
- PostgreSQL 15+ / SQLAlchemy 2.0 (async)  
- APScheduler (定时任务)  
- httpx (异步 HTTP 请求)  
- Docker + Docker Compose (部署)  
- Alembic (数据库迁移)  
  
## 关键目录  
- `app/api/v1/`       — FastAPI 路由 (APIRouter)  
- `app/models/`       — SQLAlchemy ORM 模型  
- `app/schemas/`      — Pydantic 请求/响应模型  
- `app/services/`     — 业务逻辑层  
- `app/core/`         — 配置、安全、依赖注入  
- `app/tasks/`        — APScheduler 定时任务  
- `app/utils/`        — 通用工具 (企微推送、邮件解析等)  
- `alembic/versions/` — 数据库迁移文件  
- `tests/`            — pytest + httpx AsyncClient  
  
## 命令  
```bash  
uvicorn app.main:app --reload           # 开发服务器 (localhost:8000)  
pytest tests/ -v                        # 全部测试  
pytest tests/test\_fund.py -k test\_nav   # 单个测试（优先用这个）  
alembic upgrade head                    # 数据库迁移  
alembic revision --autogenerate -m "desc"  # 生成迁移  
docker compose up -d                    # 启动所有服务  
docker compose logs -f backend          # 查看后端日志  
ruff check app/                         # 代码检查  
```  
  
## 编码规范  
- IMPORTANT: 全局使用 async/await，数据库操作用 AsyncSession  
- 所有函数必须 type hints，接口必须 Pydantic schema 验证  
- 日志: loguru，禁止 print()，错误必须 logger.error() + 上下文  
- MUST: 数据库操作用 SQLAlchemy ORM，禁止裸 SQL  
- 外部 HTTP 调用统一用 httpx.AsyncClient，禁止 requests  
- API 响应统一格式: {code, msg, data}  
- 新接口必须写 pytest 测试 (httpx AsyncClient)  
  
## 工作流程  
1. 复杂任务先制定计划，说清改什么、影响哪些模块  
2. 创建分支: feat/xxx 或 fix/xxx  
3. 实现功能 → 写测试 → ruff check → 确认通过  
4. commit message: feat: xxx 或 fix: xxx  
5. 涉及数据库变更时必须同步生成 alembic migration  
  
## 注意事项  
- MUST: 修改 model 后必须 alembic revision --autogenerate  
- MUST: 新增/修改 API 路由后检查 app/api/v1/\_\_init\_\_.py 的 router 注册  
- 所有 API 路由前缀 /api/v1，不要省略  
- APScheduler 任务在 app/tasks/ 下，禁止直接修改生产环境调度配置  
- .env 不提交 git，参考 .env.example  
- Docker 部署时数据库连接用服务名（db）而非 localhost  
- 前端 SoybeanAdmin 在单独仓库，通过 /api/v1 接口通信

## ▸ 样例 B：项目级 CLAUDE.md（前端 SoybeanAdmin）

适用于：基于 SoybeanAdmin 的前端仓库。

# 内部数据平台 — 前端  
  
基于 SoybeanAdmin 的管理后台前端，对接后端 FastAPI 接口。  
Vue 3 + TypeScript + Naive UI + UnoCSS + Pinia  
  
## 关键目录  
- `src/views/`        — 页面组件（按模块分文件夹）  
- `src/service/api/`  — API 请求层 (axios 封装)  
- `src/store/`        — Pinia 状态管理  
- `src/router/`       — 路由配置  
- `src/components/`   — 公共组件  
- `src/typings/`      — TypeScript 类型定义  
  
## 命令  
```bash  
pnpm dev              # 开发服务器  
pnpm build            # 生产构建  
pnpm lint             # ESLint 检查  
pnpm type-check       # TypeScript 类型检查  
```  
  
## 编码规范  
- 组件用 <script setup lang="ts">，禁止 Options API  
- 样式用 UnoCSS 原子化类，禁止 <style scoped> 写自定义 CSS  
- TypeScript strict mode，禁止 any  
- API 调用统一走 src/service/api/，禁止在组件里直接 axios  
- 新页面必须在 src/router/ 注册路由  
  
## 注意事项  
- IMPORTANT: 遵循 SoybeanAdmin 的目录规范和命名规则  
- 后端 API 基址在 .env 中配置，不要硬编码  
- 表格组件统一用 Naive UI NDataTable  
- 国际化文件在 src/locales/，新增文案同步更新中英文

## ▸ 样例 C：全局 ~/.claude/CLAUDE.md

个人偏好，跨所有项目生效。控制在 20 行以内。

# ~/.claude/CLAUDE.md  
  
## 语言偏好  
- 优先使用中文回复  
- 代码注释和 commit message 用英文  
  
## 通用规则  
- Python: 全局 async/await，type hints，loguru 日志  
- 禁止 print() 调试，用 logger.debug()  
- 修改代码后必须跑测试验证  
- 复杂任务先计划再动手  
- Git: conventional commits (feat/fix/chore)

────  ◆  ────

# ■ 七、维护技巧与常见问题

## ▸ 日常维护习惯

|  |  |
| --- | --- |
| 场景 | 做法 |
| Claude 犯了一个错误 | 立即加一条规则到 CLAUDE.md（# 键快捷添加） |
| CLAUDE.md 超过 200 行 | 拆分到 .claude/rules/\*.md 按路径组织 |
| 技术栈变更 | 更新 CLAUDE.md，否则 Claude 会用旧技术写代码 |
| Claude 忽略规则 | 检查是否太长；用 IMPORTANT/MUST 强调 |
| Claude 问已答的问题 | 文件中表述可能模糊，重写更清晰 |
| 切换任务时 | 用 /clear 重置上下文，CLAUDE.md 仍保留 |

## ▸ 绝对不要放在 CLAUDE.md 里的内容

• API 密钥、凭据、数据库连接串

• 详细安全漏洞信息

• 大段代码片段（用文件引用代替）

• 很少用到的领域知识（用 Skills 代替）

## ▸ 高级技巧：@ 导入语法

CLAUDE.md 支持用 @path/to/file 语法导入其他文件，实现模块化：

# CLAUDE.md  
  
## 参考文档  
See @README.md for project overview  
See @package.json for available npm commands  
  
## 额外指令  
- Git 工作流: @docs/git-instructions.md  
- 个人覆盖: @~/.claude/my-project-rules.md

💡 导入语法仅在代码块外生效。可以用这个功能把个人设置排除在版本控制之外。

 

 

参考：claude.com/blog/using-claude-md-files | code.claude.com/docs/en/best-practices

──  全文完  ──

愿每一个读到这里的人，都能在自己的奋斗路上，找到一个好搭档。