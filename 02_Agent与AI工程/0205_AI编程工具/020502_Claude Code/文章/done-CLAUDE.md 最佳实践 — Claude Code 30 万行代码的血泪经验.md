> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: CLAUDE.md 最佳实践 — Claude Code 30 万行代码的血泪经验
author: 链熵工坊
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5ODA1NTAwNQ==&mid=2247484444&idx=1&sn=0551bf453810029d8dc5d18817290d51&chksm=971a09fe8dc3ab72a80f0bcbcf45576380ae35a8a40743621242fbc1a4079ad9adebcaed30de&mpshare=1&scene=24&srcid=1123OZAgpn7LGdB15ioPO9Ce&sharer_shareinfo=59b3d398dfadae76fa49cee783764d15&sharer_shareinfo_first=59b3d398dfadae76fa49cee783764d15#rd
---

# CLAUDE.md 最佳实践 — Claude Code 30 万行代码的血泪经验

CLAUDE.md 文件从 100 行膨胀到 1400 行？文档越来越臃肿，Claude 反而不怎么看了？本文介绍如何通过**职责分离**，让 CLAUDE.md 从 1400 行瘦身到 200 行，并建立清晰可扩展的文档架构。

## 一、文档膨胀的痛苦

### 1.1 初始状态

**6 个月前的我**：

* • 创建了 `CLAUDE.md` 文件（约 100 行）
* • 包含项目基本信息、快速命令等
* • 感觉一切都很好

**3 个月后**：

* • `CLAUDE.md` 增长到 400 行
* • 新增了 `BEST_PRACTICES.md`（800 行）
* • 开始感觉有点混乱

**6 个月后**：

* • `CLAUDE.md` 达到 600 行
* • `BEST_PRACTICES.md` 膨胀到 1400+ 行
* • Claude 有时读取，有时完全忽略
* • 我自己都不想维护了

### 1.2 BEST\_PRACTICES.md 的内容

**包含了所有东西**：

* • TypeScript 标准（100 行）
* • React 模式（200 行）

+ • Hooks 使用指南
+ • 组件组织规范
+ • Suspense 模式

* • 后端 API 模式（300 行）

+ • 路由设计
+ • 控制器模式
+ • 服务层设计

* • 错误处理（150 行）

+ • Sentry 集成
+ • 错误边界

* • 数据库模式（200 行）

+ • Prisma 最佳实践
+ • 查询优化

* • 测试指南（200 行）
* • 性能优化（150 行）
* • 还有其他各种...

**总计**：1400+ 行

### 1.3 问题所在

**Claude 的表现**：

* • ❌ 有时完全忽略 BEST\_PRACTICES.md
* • ❌ 即使读取，也记不住所有内容
* • ❌ 经常使用旧模式，尽管文档中已更新

**我的困扰**：

* • 😰 维护困难：修改一个规范需要在多处更新
* • 😰 查找困难：想找某个规范要翻很久
* • 😰 职责不清：不知道该放在 CLAUDE.md 还是 BEST\_PRACTICES.md

**根本问题**：
文档试图做太多事情，没有清晰的职责边界。

---

## 二、顿悟时刻：职责分离

### 2.1 核心洞察

**我意识到**：

* • 技能系统（第 1 篇）已经解决了"如何编写代码"的问题
* • CLAUDE.md 应该专注于"这个项目如何工作"

**职责分离原则**：

```
┌─────────────────────────────────────────┐
│ Skills 系统                              │
│ 职责：如何编写代码                        │
│ - 编程语言规范                            │
│ - 框架最佳实践                            │
│ - 设计模式                                │
│ - 通用开发规范                            │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ CLAUDE.md                                │
│ 职责：这个项目如何工作                    │
│ - 项目特定命令                            │
│ - 服务配置                                │
│ - 工作流说明                              │
│ - 项目特有的怪癖                          │
└─────────────────────────────────────────┘
```

**比喻**：

* • **Skills**：就像编程语言的官方文档
* • **CLAUDE.md**：就像项目的 README 和快速开始指南

### 2.2 迁移策略

**迁移到 Skills 的内容**：

```
BEST_PRACTICES.md (1400 行)
       ↓
迁移到 Skills 系统
       ↓
- backend-dev-guidelines (304 行主文件 + 11 个资源文件)
- frontend-dev-guidelines (398 行主文件 + 10 个资源文件)
- database-verification (200 行)
- workflow-developer (250 行)
- notification-developer (180 行)
```

**保留在 CLAUDE.md 的内容**：

```
项目特定信息
       ↓
- 快速命令（pnpm pm2:start, pnpm build 等）
- 服务特定配置（7 个微服务的端口、路径等）
- 任务管理工作流（Dev Docs 系统说明）
- 测试已认证路由（项目特有的认证流程）
- 工作流试运行模式（项目特有功能）
- 浏览器工具配置（开发环境设置）
```

---

## 三、新的文档结构

### 3.1 根目录 CLAUDE.md（100 行）

**结构**：

```
# CLAUDE.md

## 项目概述
这是一个多服务架构的内部工具，包含 1 个前端 + 7 个后端微服务。

## 关键通用规则
- 永远使用简体中文进行思考和对话
- 代码内容使用英文，代码注释使用中文
- 遵循各仓库的 Skills 指南

## 仓库结构
- `/frontend` - React 19 + TypeScript 前端
  - 详见：frontend/claude.md
- `/auth-service` - 认证服务
  - 详见：auth-service/claude.md
- `/user-service` - 用户服务
  - 详见：user-service/claude.md
- 其他 5 个微服务...

## Skills 指南引用
- 前端开发：参考 frontend-dev-guidelines 技能
- 后端开发：参考 backend-dev-guidelines 技能
- 数据库操作：参考 database-verification 技能
- 工作流开发：参考 workflow-developer 技能

## 快速命令
- 启动所有服务：`pnpm pm2:start`
- 停止所有服务：`pnpm pm2:stop`
- 查看日志：`pm2 logs [service-name]`
- 构建所有服务：`pnpm build:all`

## Dev Docs 工作流
详见第 2 篇《三个文件治好 AI 失忆症》
```

**特点**：

* • ✅ 简洁明了（100 行以内）
* • ✅ 指向具体的 Skills 和子文档
* • ✅ 只包含项目层面的信息

### 3.2 各仓库的 claude.md（50-100 行）

**frontend/claude.md 示例**：

```
# Frontend 仓库

## 快速开始
1. 安装依赖：`pnpm install`
2. 启动开发服务器：`pnpm dev`
3. 运行测试：`pnpm test`
4. 构建生产版本：`pnpm build`

## 技术栈
- React 19 + TypeScript
- MUI v7（主题系统）
- TanStack Query v5（数据获取）
- TanStack Router（文件路由）
- Vite（构建工具）

## 开发指南
- 遵循 frontend-dev-guidelines 技能
- 所有组件使用函数式组件 + Hooks
- 使用 MUI v7 的主题系统
- API 调用统一使用 TanStack Query

## 项目结构
- `/src/features` - 功能模块（按业务领域组织）
- `/src/components` - 共享组件
- `/src/layouts` - 页面布局
- `/src/routes` - TanStack Router 路由定义
- `/src/theme` - MUI 主题配置

## 重要文档
- [项目知识库](./PROJECT_KNOWLEDGE.md) - 架构和集成说明
- [故障排除](./TROUBLESHOOTING.md) - 常见问题解决
- [API 文档](./docs/api/) - 自动生成的 API 文档

## 特定注意事项
- 使用 Suspense 处理异步加载
- 错误边界必须包裹所有路由
- 表单验证使用 Zod + React Hook Form
- 图标统一使用 MUI Icons

## 浏览器工具
- React DevTools
- TanStack Query DevTools（开发模式自动启用）
- Redux DevTools（如果使用状态管理）
```

**auth-service/claude.md 示例**：

```
# Auth Service 仓库

## 快速开始
1. 安装依赖：`pnpm install`
2. 配置环境变量：复制 `.env.example` 到 `.env`
3. 运行数据库迁移：`pnpm prisma:migrate`
4. 启动服务：`pnpm dev`（开发模式）或 `pm2 start` (PM2)

## 技术栈
- Node.js + Express
- TypeScript
- Prisma ORM
- Keycloak（外部认证）
- Redis（会话存储）

## 开发指南
- 遵循 backend-dev-guidelines 技能
- 路由 → 控制器 → 服务 → 仓库 架构
- 所有错误捕获到 Sentry
- 使用 repository pattern 访问数据库

## 服务配置
- 端口：3001
- 健康检查：http://localhost:3001/health
- Swagger 文档：http://localhost:3001/api-docs

## 重要文档
- [认证流程](./docs/authentication-flow.md)
- [Token 管理](./docs/token-management.md)
- [Keycloak 集成](./docs/keycloak-integration.md)

## 特定注意事项
- 使用 test-auth-route.js 脚本测试已认证路由
- Token 刷新逻辑在 TokenService.ts
- 会话存储在 Redis，过期时间 24 小时
```

### 3.3 分层文档架构

**层级 1：根目录 CLAUDE.md**

* • 项目全局信息
* • 指向各仓库的 claude.md

**层级 2：各仓库 claude.md**

* • 仓库特定信息
* • 指向详细的技术文档

**层级 3：详细技术文档**

* • PROJECT\_KNOWLEDGE.md：架构和集成
* • TROUBLESHOOTING.md：常见问题
* • docs/ 目录：API 文档、设计文档等

**导航路径示例**：

```
用户想了解"如何创建一个新的 API 端点"
       ↓
1. 阅读根目录 CLAUDE.md
   → 发现应该参考 backend-dev-guidelines 技能
       ↓
2. Claude 自动加载 backend-dev-guidelines 技能
   → 获取通用的 API 开发最佳实践
       ↓
3. 阅读 auth-service/claude.md
   → 获取该服务的特定配置和注意事项
       ↓
4. 如果需要，查看 auth-service/docs/
   → 获取更详细的技术细节
```

---

## 四、迁移实战

### 4.1 TypeScript 标准

**迁移前**（BEST\_PRACTICES.md）：

```
## TypeScript Standards

### Type Definitions
- Always define explicit types for function parameters
- Use interfaces for objects with fixed shape
- Use type aliases for unions and intersections
- Avoid using `any`, prefer `unknown`

### File Organization
- One type definition per file for complex types
- Export types from a central types/ directory
- Use barrel exports (index.ts)

### Naming Conventions
- Interfaces: PascalCase, prefix with I (e.g., IUser)
- Type aliases: PascalCase (e.g., UserId)
- Generics: Single capital letter or descriptive PascalCase
```

**迁移后**（frontend-dev-guidelines 技能）：

```
# Frontend Development Guidelines

## TypeScript 规范
详见：[@typescript-standards.md](@typescript-standards.md)
```

**资源文件**（typescript-standards.md）：

```
# TypeScript 标准

## 类型定义
- 函数参数必须有显式类型
- 对象使用 interface
- 联合类型使用 type
- 禁止 any，使用 unknown

## 文件组织
- 复杂类型单独文件
- 集中导出到 types/ 目录
- 使用 barrel exports

## 命名规范
- Interface: PascalCase, I 前缀（如 IUser）
- Type: PascalCase（如 UserId）
- Generic: 单字母或描述性 PascalCase
```

**好处**：

* • ✅ 从 100 行减少到主文件中的 3 行引用
* • ✅ 详细内容在资源文件中，按需加载
* • ✅ 易于更新和维护

### 4.2 React 模式

**迁移前**（BEST\_PRACTICES.md）：

```
## React Patterns (200 lines)
- Component organization
- Hooks guidelines
- State management
- Performance optimization
- ...
```

**迁移后**（frontend-dev-guidelines 技能）：

```
## React 开发
- [组件模式](@react-component-patterns.md)
- [Hooks 使用](@react-hooks.md)
- [状态管理](@state-management.md)
- [性能优化](@react-performance.md)
```

**资源文件示例**（react-hooks.md）：

```
# React Hooks 使用指南

## useState
- 简单状态使用 useState
- 复杂对象考虑 useReducer

## useEffect
- 依赖数组必须完整
- 清理函数处理订阅
- 避免在 useEffect 中设置状态

## 自定义 Hooks
- 命名以 use 开头
- 返回数组或对象
- 提供 TypeScript 类型
```

---

## 五、850+ 文档的管理策略

### 5.1 文档分类

**我的 850+ 文档分类**：

```
docs/
├── architecture/           # 架构设计（50 个）
│   ├── system-overview.md
│   ├── microservices-communication.md
│   └── data-flow.md
│
├── api/                   # API 文档（200 个，自动生成）
│   ├── auth-service/
│   ├── user-service/
│   └── ...
│
├── features/              # 功能说明（300 个）
│   ├── authentication/
│   ├── notifications/
│   ├── workflow-engine/
│   └── ...
│
├── development/           # 开发指南（100 个）
│   ├── setup/
│   ├── testing/
│   └── deployment/
│
├── troubleshooting/       # 故障排除（50 个）
│   ├── common-errors.md
│   ├── performance-issues.md
│   └── ...
│
└── decisions/             # 架构决策记录（150 个）
    ├── adr-001-tech-stack.md
    ├── adr-002-database-choice.md
    └── ...
```

### 5.2 文档索引系统

**PROJECT\_KNOWLEDGE.md（主索引）**：

```
# 项目知识库

## 架构文档
- [系统概览](./architecture/system-overview.md)
- [微服务通信](./architecture/microservices-communication.md)
- [数据流](./architecture/data-flow.md)

## 功能文档（按模块）

### 认证系统
- [认证流程](./features/authentication/flow.md)
- [Token 管理](./features/authentication/token-management.md)
- [Keycloak 集成](./features/authentication/keycloak.md)

### 通知系统
- [通知架构](./features/notifications/architecture.md)
- [邮件模板](./features/notifications/email-templates.md)
- [推送通知](./features/notifications/push-notifications.md)

## API 文档
- [认证服务 API](./api/auth-service/)
- [用户服务 API](./api/user-service/)
- ...

## 故障排除
- [常见错误](./troubleshooting/common-errors.md)
- [性能问题](./troubleshooting/performance-issues.md)

## 架构决策记录（ADR）
- [ADR 001：技术栈选择](./decisions/adr-001-tech-stack.md)
- [ADR 002：数据库选择](./decisions/adr-002-database-choice.md)
- ...
```

### 5.3 文档维护原则

**自动生成 vs 手动编写**：

* • ✅ API 文档：自动生成（Swagger/OpenAPI）
* • ✅ 代码文档：自动生成（TypeDoc/JSDoc）
* • ✅ 架构文档：手动编写
* • ✅ 功能说明：手动编写
* • ✅ 决策记录：手动编写

**更新频率**：

* • 🔄 每日：API 文档（自动生成）
* • 🔄 每周：功能文档（新功能添加）
* • 🔄 每月：架构文档（架构变更）
* • 🔄 按需：决策记录（重大决策）

**文档质量检查**：

* • 每个文档都有"最后更新"时间戳
* • 定期审查 6 个月未更新的文档
* • 过时文档添加"已过时"标记或删除

---

## 六、实际效果

### 6.1 CLAUDE.md 大小对比

**迁移前**：

```
CLAUDE.md: 600 行
BEST_PRACTICES.md: 1400 行
总计：2000 行

Claude 的表现：
- 有时读取，有时忽略
- 记不住所有内容
- 经常违反规范
```

**迁移后**：

```
CLAUDE.md: 100 行（根目录）
各仓库 claude.md: 50-100 行 × 8 = 400-800 行
Skills 系统: 自动激活，按需加载
总计：500-900 行（但分散在合理的位置）

Claude 的表现：
- Skills 自动激活（第 1 篇的钩子系统）
- 只读取需要的部分
- 始终遵循规范
```

### 6.2 查找效率提升

**场景：我想更新"React Hooks 使用规范"**

**迁移前**：

```
1. 打开 BEST_PRACTICES.md（1400 行）
2. 搜索 "Hooks"
3. 找到第 453 行
4. 修改
5. 保存
6. 希望 Claude 下次会读取（不保证）
```

**迁移后**：

```
1. 打开 frontend-dev-guidelines/resources/react-hooks.md
2. 直接看到完整的 Hooks 指南（单独文件）
3. 修改
4. 保存
5. 钩子系统确保下次自动激活（100% 保证）
```

**时间节省**：从 5 分钟减少到 1 分钟

### 6.3 维护负担减轻

**迁移前**：

* • 😰 修改一个规范可能需要在多处更新
* • 😰 不确定 Claude 是否会应用
* • 😰 经常需要手动提醒 Claude 检查文档

**迁移后**：

* • 😊 每个规范在一个地方
* • 😊 Skills 系统确保自动应用
* • 😊 从不需要手动提醒

---

## 七、核心要点总结

### 7.1 职责分离原则

**明确界限**：

* • **Skills**：语言和框架的通用最佳实践
* • **CLAUDE.md**：项目特定的配置和流程
* • **技术文档**：详细的架构和设计说明

**不要混淆**：

* • ❌ 不要把 React 通用模式放在 CLAUDE.md
* • ❌ 不要把项目特定命令放在 Skills
* • ❌ 不要把所有东西都塞在一个文件里

### 7.2 实施建议

**迁移步骤**：

1. 1. 识别 CLAUDE.md 中的通用规范
2. 2. 将它们迁移到相应的 Skills
3. 3. 保留项目特定信息在 CLAUDE.md
4. 4. 创建分层文档索引
5. 5. 测试 Skills 自动激活

**从小开始**：

* • 先迁移一个领域（如 TypeScript 规范）
* • 验证 Skills 自动激活是否工作
* • 逐步迁移其他领域

### 7.3 文档管理经验

**大型项目（850+ 文档）**：

* • ✅ 建立清晰的分类系统
* • ✅ 使用主索引文件（PROJECT\_KNOWLEDGE.md）
* • ✅ 尽可能自动生成（API、代码文档）
* • ✅ 定期审查和清理过时文档
* • ✅ 每个文档都有最后更新时间戳

**中小型项目（< 100 文档）**：

* • ✅ 简化的分类（architecture、features、api）
* • ✅ 单个索引文件
* • ✅ 手动维护为主

---

**相关资源**：

* • GitHub 仓库
* • 文档架构最佳实践

**最后更新**：2025-11-16