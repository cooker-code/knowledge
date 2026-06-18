> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: CLAUDE.md 深度指南：让 Claude 真正懂你的项目
author: 匠心格物
date:
url: https://mp.weixin.qq.com/s?__biz=MzkxNDczMDg0Nw==&mid=2247485222&idx=1&sn=e84abd2c94b0f36661d3b694d6af78a9&chksm=c0793111fa3a62ca9537e3e4496a8c8e41056c9d1cc6115dce304c4b978cf794c871c1acb284&mpshare=1&scene=24&srcid=0414EYO6Db9hZxaPPppIotMb&sharer_shareinfo=5961032d2d41032a606e2f94672c03b9&sharer_shareinfo_first=5961032d2d41032a606e2f94672c03b9#rd
---

CLAUDE.md 深度指南封面

---

## 为什么 CLAUDE.md 是最重要的配置

每次启动 Claude Code，它会自动读取当前目录及父目录中的 `CLAUDE.md` 文件。这个文件就是你给 Claude 的"项目手册"——规范、禁忌、架构说明、常用命令，一次写好，永久生效。

没有 CLAUDE.md 的 Claude Code，就像新入职的工程师什么都没交接就开始干活。有了它，Claude 能直接以"老员工"的状态上手。

---

## CLAUDE.md 的三层结构



CLAUDE.md 三层配置体系

### 第一层：全局配置（`~/.claude/CLAUDE.md`）

放在 home 目录，对所有项目生效。适合放：

* 工作语言偏好
* 通用编码规范
* 禁止操作清单

```
# 全局设置

## 语言
始终使用中文进行所有交互和输出。

## 通用规范
- 提交信息使用约定式提交（Conventional Commits）格式
- 不要使用 `console.log`，使用项目已有的 logger
- 修改前先读文件，不要凭空猜测
```

### 第二层：项目配置（`<项目根目录>/CLAUDE.md`）

放在 Git 仓库根目录，提交进版本控制，团队共享。适合放：

* 项目架构说明
* 技术栈和关键依赖
* 构建/测试命令
* 项目特有的禁忌操作

### 第三层：子目录配置（`<子目录>/CLAUDE.md`）

针对特定模块的说明。比如 `frontend/CLAUDE.md` 专门说明前端规范，`api/CLAUDE.md` 说明接口约定。

---

## 实战模板：生产级 CLAUDE.md



生产级 CLAUDE.md 六大核心模块

以下是一个 TypeScript 全栈项目的完整示例：

```
# CLAUDE.md — MyApp 项目指南

## 项目概述
MyApp 是一个 SaaS 订阅管理平台。后端 Node.js + Express，前端 React + TypeScript，数据库 PostgreSQL。

## 关键目录结构
- `src/api/` — Express 路由和控制器
- `src/services/` — 业务逻辑层（不直接访问数据库）
- `src/db/` — 数据库操作（使用 Drizzle ORM）
- `frontend/src/` — React 前端
- `tests/` — 集成测试（Jest + Supertest）

## 常用命令
- 启动开发服务器：`npm run dev`
- 运行测试：`npm test`
- 运行单个测试：`npm test -- --testPathPattern=auth`
- 数据库迁移：`npm run db:migrate`
- 生成类型：`npm run db:generate`

## 编码规范
- 所有业务逻辑放在 `services/` 层，路由只做参数验证和调用 service
- 数据库查询统一在 `db/` 目录，禁止在其他地方直接写 SQL
- 错误处理：使用 `AppError` 类，不要 throw 原始 Error
- API 响应格式：`{ data: T, error: null }` 或 `{ data: null, error: string }`

## 禁止操作
- 不要删除 `migrations/` 目录下的文件
- 不要修改 `src/db/schema.ts` 中已存在的列名（会破坏生产数据）
- 不要在测试文件中使用真实的外部 API，统一用 mock
- 不要提交 `.env` 文件

## 测试策略
- 新功能必须有集成测试
- 单元测试用于纯函数和工具类
- 测试数据库：使用 `TEST_DATABASE_URL` 环境变量指向测试库

## 当前已知问题
- `src/api/billing.ts` 的 webhook 处理逻辑待重构（技术债）
- 分页功能目前只有 offset 分页，cursor 分页正在开发中
```

---

## 七个让 CLAUDE.md 真正有效的原则



CLAUDE.md 七大黄金原则

### 1. 写"禁止"比写"规范"更重要

Claude 很聪明，大多数最佳实践它都知道。但项目特有的"坑"它不知道。直接写出来：

```
## 绝对禁止
- 不要修改 `config/feature-flags.json`，这个文件由 CI 自动生成
- 不要在 `useEffect` 里直接调用 API，使用 `useQuery` hook
```

### 2. 给出具体命令，别让 Claude 猜

```
# 不好的写法
运行测试后提交代码

# 好的写法
提交前运行：npm run lint && npm run test:unit
```

### 3. 架构约束要说清楚"为什么"

```
## 为什么不直接在组件里 fetch 数据？
我们用 React Query 统一管理服务端状态，直接 fetch 会导致缓存失效和重复请求问题。
```

### 4. 定期更新，别让它过时

CLAUDE.md 过时比没有更危险。把更新 CLAUDE.md 加入你的功能开发 checklist。

### 5. 利用层级结构避免信息过载

全局文件保持简洁，项目细节放项目文件，模块细节放子目录文件。

### 6. 包含常见错误案例

```
## 常见错误
- `prisma.user.findMany()` — 数据量大时会 OOM，必须加 `take` 参数
- `prisma.user.findMany({ take: 100, skip: offset })`
```

### 7. 说明测试环境特殊性

```
## 测试注意
- 测试使用内存 SQLite，不是 PostgreSQL，某些 JSON 查询行为不同
- 邮件发送在测试中被 mock，检查 `mockMailer.sent` 数组验证
```

---

## 进阶：动态 CLAUDE.md

你可以让 CLAUDE.md 引用外部信息：

```
## API 文档
参考 `docs/api-spec.yaml`（OpenAPI 3.0 格式）

## 数据库 Schema
参考 `src/db/schema.ts`，这是唯一真实来源
```

Claude Code 会自动去读取这些引用文件。

---

## 团队协作：共享 CLAUDE.md

把 CLAUDE.md 提交到 Git，让所有团队成员和 CI/CD 中的 Claude Code 使用相同的上下文：

```
git add CLAUDE.md
git commit -m "docs: 添加 Claude Code 项目配置指南"
```

---

## 检验你的 CLAUDE.md 是否有效



CLAUDE.md 有效性三问

一个简单的检验方法：让 Claude 在只看 CLAUDE.md 的情况下，回答这几个问题：

1. "如何运行测试？"
2. "我想加一个新 API 接口，应该改哪些文件？"
3. "有哪些操作是被明确禁止的？"

答不上来，就说明需要补充。

---

## 下一篇预告

配置好 CLAUDE.md 后，你需要掌握**高效对话的方法论**：如何管理上下文窗口、如何分解复杂任务、如何用 Claude Code 驱动完整的开发工作流。

[第一篇：Claude Code 入门指南](https://mp.weixin.qq.com/s?__biz=MzkxNDczMDg0Nw==&mid=2247485208&idx=1&sn=2ad2fda24f199ea179cfe9c54bb6b3c8&scene=21#wechat_redirect)