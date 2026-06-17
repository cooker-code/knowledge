---
title: AGENTS.md和CLAUDE.md — 企业级AI编程经验分享(1)
author: 君哥聊编程
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU1ODcwMjE2OQ==&mid=2247483964&idx=1&sn=c08d2e5930cbbe58bda8e95934f94fc1&chksm=fd408c494fbae15d2cb832cf6f901a8f864f992d2014e76e9ae6c8988c382ff64ded190f7ea3&mpshare=1&scene=24&srcid=04212vo2OCImKSR8Y3fc0TpV&sharer_shareinfo=c887308337c10bc68c2e309c3d2bd0fd&sharer_shareinfo_first=c887308337c10bc68c2e309c3d2bd0fd#rd
---

> "
>
> 用AI编程2年了，一直在学习，却很少分享总结，这是我的AI实战经验文章，关注我一起学习
>
> 今天要讲的是AGENTS.md和CLAUDE.md，这里我称之为总则文件

# 总则文件

## 如何统一总则文件

**总则文件是一种“广泛适用”的规则。建议在项目根目录下统一使用`AGENTS.md`， 把所有人都要长期遵守、几乎每次会话都相关的内容都放进来，且不超过200行，**

---

以下是关于claude.md和agents.md的介绍和兼容性讨论

1. 1. AGENTS.md： AI规范文件，用来给 AI 提供上下文信息和操作指南，是一种开放标准，协助 AI coding Agent处理你的项目。README.md服务于人，而 AGENTS.md 服务于 AI。
2. 2. CLAUDE.md：claude code的总则文件，建议通过`include`方式引入`AGENTS.md`，保证项目下的总则文件只维护一份。

> "
>
> claude code官方对 CLAUDE.md 的定位：它是一个持久指令文件，适合承载项目架构、编码标准、构建和测试命令、命名约定以及常见工作流。官方在最佳实践中还特别提到，可以把 Bash commands、code style 和 workflow rules 放进 CLAUDE.md。
>
> 适合放“广泛适用”的规则，也就是那些无论 Claude 正在改哪个模块、读哪个文件，几乎都应该知道的内容。

下面这些内容就很适合放入总则文件：

* • 项目的构建、测试和启动命令
* • 团队统一的代码风格约定
* • 关键目录的职责说明
* • 单元测试规则
* • 常见工作流要求，例如改动后先跑哪类测试
* • Git规范简述
* • 架构层面的硬性约定，例如 API 统一放在哪个目录

## AGENTS.md与CLAUDE.md差异

AGENTS.md 驻扎在项目中，每个不同层级的目录都可以有，并且采用"就近原则"： 优先级从高到低：

1. 1. 用户在聊天中的直接提示词（最高优先级）
2. 2. 距离当前编辑文件最近的 AGENTS.md
3. 3. 根目录的 AGENTS.md

```
mds/  
├── AGENTS.md          # 根目录（通用规则）  
├── com.it235.mds/  
│   ├── api/  
│   │   └── AGENTS.md  # api 模块（REST规则）  
│   ├── service/  
│   │   └── AGENTS.md  # service 实现模块（逻辑层规则）  
│   └── repository/  
│       └── AGENTS.md   # repository（持久层开发规则）
```

已实现`AGENTS.md`的IDE

* • cursor
* • qoder
* • trae
* • codebuddy
* • opencode
* • codex
* • gemini cli
* • 等等主流的IDE

官方文档：https://agents.md/

CLAUDE.md 除了项目根目录存在，也可以存在以下位置，不同位置代表着不同的优先级。Claude Code 从当前目录开始，逐级向上遍历到文件系统根目录，在每一级检查以下文件

```
每一级目录检查：  
├── CLAUDE.md              → Project 类型（可版本控制）  
├── .claude/CLAUDE.md      → Project 类型（可版本控制）  
├── .claude/rules/*.md     → Project 类型（可版本控制，递归扫描子目录）  
└── CLAUDE.local.md        → Local 类型（不应提交到版本控制）  
  
加上两个全局位置：  
~/.claude/CLAUDE.md        → User 类型（个人全局指令）  
~/.claude/rules/*.md       → User 类型（个人全局规则）  
/etc/claude-code/CLAUDE.md → Managed 类型（企业管理员策略）  
/etc/claude-code/.claude/rules/*.md → Managed 类型
```

## 总则文件示例

```
示例来源：  
https://blog.csdn.net/a18792721831/article/details/156729996  
  
# TaskFlow 项目配置  
  
## 项目概述  
TaskFlow 是一个基于 React + Node.js 的任务管理系统，支持团队协作、任务分配、进度追踪等功能。  
  
## 技术栈  
- 前端：React 18 + TypeScript + Vite + TailwindCSS  
- 后端：Node.js + Express + TypeScript  
- 数据库：PostgreSQL + Prisma ORM  
- 测试：Jest + React Testing Library  
  
## 常用命令  
  
### 开发  
-`pnpm dev` - 同时启动前后端开发服务器  
-`pnpm dev:client` - 仅启动前端（端口 5173）  
-`pnpm dev:server` - 仅启动后端（端口 3000）  
  
### 构建  
-`pnpm build` - 构建前后端  
-`pnpm build:client` - 仅构建前端  
-`pnpm build:server` - 仅构建后端  
  
### 测试  
-`pnpm test` - 运行所有测试  
-`pnpm test:client` - 运行前端测试  
-`pnpm test:server` - 运行后端测试  
-`pnpm test:e2e` - 运行端到端测试  
  
### 数据库  
-`pnpm db:migrate` - 运行数据库迁移  
-`pnpm db:seed` - 填充测试数据  
-`pnpm db:studio` - 打开 Prisma Studio  
  
### 代码质量  
-`pnpm lint` - 运行 ESLint  
-`pnpm typecheck` - 运行类型检查  
-`pnpm format` - 运行 Prettier 格式化  
  
## 代码风格  
  
### TypeScript  
- 严格模式开启，不允许 any 类型  
- 优先使用 interface 定义对象类型  
- 使用 type 定义联合类型和工具类型  
- 函数返回值必须显式声明类型  
  
### React  
- 使用函数组件 + Hooks，不使用类组件  
- 组件 props 使用 interface 定义  
- 状态管理使用 Zustand  
- 表单处理使用 React Hook Form + Zod  
  
### API 设计  
- RESTful 风格  
- 响应格式：`{ code: number, data: T, message: string }`  
- 错误码：200 成功，400 参数错误，401 未授权，500 服务器错误  
  
## 目录结构  
  
taskflow/  
├── client/                 # 前端代码  
│   ├── src/  
│   │   ├── components/     # 通用组件  
│   │   ├── features/       # 功能模块（按业务划分）  
│   │   ├── hooks/          # 自定义 Hooks  
│   │   ├── lib/            # 第三方库封装  
│   │   ├── stores/         # Zustand stores  
│   │   └── types/          # 类型定义  
│   └── tests/              # 前端测试  
├── server/                 # 后端代码  
│   ├── src/  
│   │   ├── controllers/    # 控制器  
│   │   ├── services/       # 业务逻辑  
│   │   ├── middlewares/    # 中间件  
│   │   ├── routes/         # 路由定义  
│   │   └── utils/          # 工具函数  
│   └── tests/              # 后端测试  
├── prisma/                 # Prisma 配置和迁移  
└── e2e/                    # 端到端测试  
  
## 开发规范  
  
### Git 工作流  
- 主分支：main（生产）、develop（开发）  
- 功能分支：feature/xxx  
- 修复分支：fix/xxx  
- 提交前必须通过 lint 和 typecheck  
  
### 提交信息格式  
<type>(<scope>): <subject>  
  
<body>  
  
<footer>  
  
type 可选值：  
- feat: 新功能  
- fix: 修复 bug  
- docs: 文档更新  
- style: 代码格式（不影响功能）  
- refactor: 重构  
- test: 测试相关  
- chore: 构建/工具相关  
  
### Code Review 要求  
- 每个 PR 至少需要一个 approve  
- PR 标题格式与提交信息一致  
- 必须关联对应的 Issue  
  
## 注意事项  
  
### 环境变量  
- 前端环境变量以 `VITE_` 开头  
- 后端环境变量在 `.env` 文件中配置  
- 敏感信息不要提交到代码库  
  
### 数据库操作  
- 所有数据库操作通过 Prisma 进行  
- 复杂查询使用 Prisma 的 raw query  
- 迁移文件不要手动修改  
  
### 测试要求  
- 新功能必须有对应的单元测试  
- 核心业务逻辑测试覆盖率 > 80%  
- API 接口必须有集成测试  
  
## 常见问题  
  
### Q: 启动报错 "Cannot find module"  
A: 运行 `pnpm install` 重新安装依赖  
  
### Q: 数据库连接失败  
A: 检查 `.env` 中的 `DATABASE_URL` 配置，确保 PostgreSQL 服务已启动  
  
### Q: TypeScript 类型错误  
A: 运行 `pnpm typecheck` 查看详细错误，确保类型定义正确
```