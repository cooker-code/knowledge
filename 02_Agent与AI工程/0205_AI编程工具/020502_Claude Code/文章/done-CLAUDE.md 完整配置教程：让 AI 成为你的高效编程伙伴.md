> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: CLAUDE.md 完整配置教程：让 AI 成为你的高效编程伙伴
author: Sam爱喝汤
date:
url: https://mp.weixin.qq.com/s?__biz=MjM5MjQ2MTg5NA==&mid=2247483743&idx=1&sn=627c3b48dff1d3ffa043626de0672b17&chksm=a7e20612455614c67638dce60c2cb6da3c1f3fc642e4985b1d7a15b277bebe94b7878be88464&mpshare=1&scene=24&srcid=0117p0MnwCpX5rKpezvQjVbV&sharer_shareinfo=7eb429bc8d03e5c3d63b593b85c4b13e&sharer_shareinfo_first=7eb429bc8d03e5c3d63b593b85c4b13e#rd
---

# 一、为什么需要 CLAUDE.md？

### 1.1 解决沟通成本问题

当你直接对 Claude 说"帮我写个登录页面"，它可能会：

* 使用你项目里不存在的技术栈（如 Vue 而不是 React）
* 忽略项目的代码规范（如不使用 TypeScript）
* 把代码放在错误的目录结构里
* 使用过时的 API 或已被弃用的写法

每次对话你都需要重复说明这些规则，**CLAUDE.md 就是解决这个问题的"项目说明书"**。

### 1.2 文件的重要性

* **一致性保证**：确保 AI 生成的代码符合团队规范，避免"风格污染"
* **效率提升**：减少 50% 以上的重复沟通，让 AI 直接"懂你"
* **知识沉淀**：将团队的编码共识文档化，新成员也能快速上手
* **错误预防**：明确禁止某些操作，防止 AI 误改关键代码

## 二、文件位置与优先级

### 2.1 配置文件的查找顺序

Claude 支持分层配置，会按照以下顺序查找 CLAUDE.md 文件，**优先级从高到低**：

1. **`./CLAUDE.local.md`**（本地配置）

* 仅对当前用户生效
* 不提交到 Git 版本控制
* 适合存放个人偏好或敏感配置

2. **`./CLAUDE.md`**（项目配置）

* 项目根目录下的主配置文件
* 必须提交到 Git，确保团队一致性
* 这是最常用的配置位置

3. **父目录的 CLAUDE.md**（适用于 Monorepo）

* 在 Monorepo 架构中，子项目会继承父目录的配置
* 子项目可以有自己的 CLAUDE.md 覆盖父级规则

4. **`~/.claude/CLAUDE.md`**（全局配置）

* 用户主目录下的全局配置
* 适用于所有项目
* 适合定义通用规范（如 Git 提交规范、通用工具链）

### 2.2 配置继承机制

**高优先级的配置会覆盖低优先级的配置**，而不是完全替换。这意味着：

* 如果 CLAUDE.local.md 和 CLAUDE.md 同时存在，会以 CLAUDE.local.md 为准
* 如果某个配置项只在低优先级文件中定义，高优先级文件没有定义，则低优先级的配置仍然生效
* 这种设计允许你在全局定义通用规则，在项目级定义特定规则，在本地定义个人偏好

### 2.3 实际应用场景

**场景 1：团队项目**

* 在项目根目录创建 CLAUDE.md，定义团队统一的代码规范
* 所有团队成员共享同一份配置
* 在 Code Review 时检查配置变更

**场景 2：个人项目**

* 在 `~/.claude/CLAUDE.md` 定义个人通用规范
* 在具体项目中使用 CLAUDE.md 定义项目特定规则
* 使用 CLAUDE.local.md 存放临时实验性配置

**场景 3：Monorepo**

* 在 monorepo 根目录定义通用规范
* 在子项目（如 `apps/web`、`apps/mobile`）中定义特定技术栈的配置
* 子项目配置会继承并覆盖父级配置

## 三、如何创建和配置 CLAUDE.md

### 3.1 创建文件

在项目根目录执行：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# 方法一：使用 /init 命令（推荐）cd your-projectclaude/init
# 方法二：手动创建touch CLAUDE.md
```

### 3.2 必须包含的 5 个核心模块

#### 模块 1：技术栈声明

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# 技术栈
## 前端- React 18.2 (Hooks 优先，避免 Class 组件)- TypeScript 5.2 (严格模式)- Next.js 14 (App Router)- Tailwind CSS 3.4
## 后端- Node.js 20 LTS- Express 4.18- Prisma ORM- PostgreSQL 15
```

**关键点**：必须写版本号，否则 AI 可能用旧 API。

#### 模块 2：项目结构说明

```
ounter(lineounter(lineounter(line# 项目结构
## 目录结构
```

src/ ├── app/              # Next.js 页面路由 ├── components/       # 可复用 UI 组件 │   ├── ui/          # 基础组件 (Button, Input) │   └── features/    # 业务组件 (UserCard, OrderList) ├── lib/             # 工具函数和 hooks ├── services/        # API 调用层 └── types/           # TypeScript 类型定义

```
ounter(lineounter(lineounter(lineounter(lineounter(line
## 文件命名规范- 组件：PascalCase (UserProfile.tsx)- 工具函数：camelCase (formatDate.ts)- 常量：UPPER_SNAKE_CASE (API_BASE_URL)
```

有了这个，Claude 就知道新建用户卡片组件应该放在 `components/features/UserCard.tsx`，而不是随便找个地方。

#### 模块 3：常用命令

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# 常用命令
## 开发- `npm run dev`      # 启动开发服务器 (localhost:3000)- `npm run build`    # 生产构建- `npm run test`     # 运行 Jest 测试- `npm run lint`     # ESLint 检查- `npm run type-check` # TypeScript 类型检查
## 数据库- `npx prisma studio`        # 打开数据库 GUI- `npx prisma migrate dev`   # 运行数据库迁移
```

这部分经常被忽略，但超级有用，能避免很多"为什么测试不通过"的问题。

#### 模块 4：代码风格规范

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# 代码规范
## React 组件- 使用函数组件 + Hooks，禁止 Class 组件- Props 类型定义在组件上方，使用 interface 而非 type- 组件内部顺序：Props 定义 → 组件函数 → 导出- 单函数不超过 50 行，超过则拆分
## 状态管理- 本地状态：useState/useReducer- 服务器状态：TanStack Query- 全局状态：Zustand（避免使用 Context）
## 错误处理- API 调用必须 try-catch- 用户可见错误使用 toast 提示- 开发环境 console.error，生产环境上报 Sentry
```

**关键点**：避免模糊描述（如"保持代码简洁"），要具体到可验证的规则（如"单函数不超过 50 行"）。

#### 模块 5：工作流和限制

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# 工作流
## 新功能开发流程1. 先写类型定义2. 写组件3. 写测试
## 修 Bug 流程1. 先写复现测试2. 修复代码3. 验证测试通过
# 限制事项
## ❌ 禁止操作- 不要修改 `/prisma/schema.prisma`（需团队评审）- 不要安装新依赖包（需在 package.json review 时讨论）- 不要修改 `/lib/auth/*`（认证逻辑敏感）
## ✅ 允许操作- 可以自由修改 `/components` 和 `/app` 下的业务代码
```

这样能防止 AI 误改关键代码，同时明确它的"安全区"。

## 四、最佳实践和技巧

### 4.1 保持简洁（100 行以内）

CLAUDE.md 不是 README，不需要长篇大论。从技术角度讲，它占用 Claude 的上下文窗口，文件越长，留给实际代码分析的 token 就越少。100 行以内的配置文件效果最佳。

**错误示例**（冗长重复）：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# 项目简介这是一个基于 React 的前端项目。我们使用 React 来构建用户界面。React 是一个由 Facebook 开发的 JavaScript 库...（此处省略 200 字 React 介绍）
# 技术栈我们的技术栈包括以下内容：- React - 这是我们的 UI 框架，版本是 18.2...- TypeScript - 我们用它来做类型检查...
```

**正确示例**（精简直接）：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# 技术栈- React 18.2 (Hooks 优先，避免 Class 组件)- TypeScript (严格模式)- TailwindCSS (utilities 优先)
# 代码规范- 函数组件 + 自定义 Hooks- Props 解构赋值- 优先使用 const，避免 let
```

第二个版本只用了 60 个字符就传达了同样甚至更多的有效信息。

### 4.2 用 SHOULD/MUST 强调优先级

不是所有规则都同等重要，用关键词区分优先级：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# 规范优先级
## MUST（必须遵守）- MUST 使用 TypeScript 严格模式- MUST 为所有 API 添加错误处理
## SHOULD（推荐遵守）- SHOULD 组件不超过 200 行- SHOULD 提取重复逻辑为自定义 Hook
## COULD（可选）- COULD 添加 JSDoc 注释
```

这个技巧来自 RFC 规范文档的写法，用了之后 Claude 明显对"MUST"的规则执行得更严格。

### 4.3 示例代码胜过千言万语

与其描述规范，不如直接给示例：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# API 调用规范
参考示例：```typescript// src/services/user.tsexport async function getUser(id: string): Promise<User> {  try {    const response = await fetch(`/api/users/${id}`);    if (!response.ok) throw new Error('Failed to fetch user');    return await response.json();  } catch (error) {    console.error('getUser error:', error);    throw error;  }}
```

所有 API 函数都遵循此模式：类型返回值 + try-catch + 错误日志

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line
Claude 看到示例后，生成的代码会非常接近你的期望。
### 4.4 用 # 键快速更新在 Claude Code 里，按 `#` 键可以快速引用和编辑 CLAUDE.md。当你发现 AI 的行为不符合预期时，立刻改配置，不要拖延。
**真实场景**：如果 Claude 老是用 axios 生成代码，但团队已经统一用 fetch + 自定义封装，可以按 `#` 键添加：```markdown# HTTP 请求- 统一使用 `src/utils/request.ts` 封装的 fetch- 禁止直接使用 axios 或原生 fetch
```

保存后，Claude 就再也不会犯这个错误了。

### 4.5 团队共享：纳入版本控制

CLAUDE.md 必须提交到 Git 仓库，和 `.gitignore` 一样重要。因为你的配置代表了团队的编码共识，如果每个人本地都有不同版本，AI 给不同成员生成的代码风格会完全不同。

**正确做法**：

```
ounter(lineounter(lineounter(lineounter(line# .gitignore 中不要忽略 CLAUDE.md*.md!CLAUDE.md!README.md
```

而且要在 Code Review 时检查 CLAUDE.md 的变更，如果有人改了配置，整个团队都应该知道。

### 4.6 分层配置（Monorepo 必备）

如果你的项目是 Monorepo 架构，一定要用分层配置：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(linemonorepo-root/├── CLAUDE.md           # 全局：通用规范、Git 工作流├── apps/│   ├── web/│   │   └── .claude/CLAUDE.md   # Web 应用：React + Next.js│   └── mobile/│       └── .claude/CLAUDE.md   # 移动端：React Native└── packages/    └── shared/        └── .claude/CLAUDE.md   # 共享包：纯 TS 工具库
```

根目录 CLAUDE.md 定义通用规范，子目录定义特定规则，实现"继承"思想。

## 五、完整示例

下面是一个假设的全栈项目 `TaskFlow`（任务管理系统）的完整 CLAUDE.md 配置：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line# TaskFlow - 任务管理系统
## 技术栈
### 前端- Next.js 14 (App Router)- React 18 (Server Components 优先)- TypeScript 5.2 (严格模式)- Tailwind CSS 3.4- TanStack Query (数据获取)- Zustand (状态管理)
### 后端- Node.js 20 LTS- Express 4.18- Prisma ORM- PostgreSQL 15- JWT 认证
## 项目结构
```

src/ ├── app/              # Next.js 页面路由 ├── components/       # 可复用 UI 组件 │   ├── ui/          # 基础组件 (Button, Input) │   └── features/    # 业务组件 (TaskCard, TaskList) ├── lib/             # 工具函数和 hooks ├── services/        # API 调用层 ├── types/           # TypeScript 类型定义 └── pages/           # 旧版页面路由 (逐步迁移到 app/)

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line
## 常用命令
### 开发- `npm run dev`      # 启动开发服务器 (localhost:3000)- `npm run build`    # 生产构建- `npm run test`     # 运行 Jest 测试- `npm run lint`     # ESLint 检查- `npm run type-check` # TypeScript 类型检查
### 数据库- `npx prisma studio`        # 打开数据库 GUI- `npx prisma migrate dev`   # 运行数据库迁移- `npx prisma generate`      # 生成 Prisma 客户端
## 代码规范
### React 组件- 使用函数组件 + Hooks，禁止 Class 组件- Props 类型定义在组件上方，使用 interface 而非 type- 组件内部顺序：Props 定义 → 组件函数 → 导出- 单函数不超过 50 行，超过则拆分- 列表渲染必须添加 key 属性，用 ID 而非 index- 避免嵌套三元表达式，改用 if/else 或早期 return
### 状态管理- 本地状态：useState/useReducer- 服务器状态：TanStack Query- 全局状态：Zustand（避免使用 Context）
### 错误处理- API 调用必须 try-catch- 用户可见错误使用 toast 提示- 开发环境 console.error，生产环境上报 Sentry
### 类型定义- 所有函数必须有类型注解- 优先使用 interface 定义对象类型- 避免使用 any，必要时用 unknown
## 工作流
### 新功能开发1. 先写类型定义2. 写组件3. 写测试4. 提交前运行测试和 lint
### 修 Bug1. 先写复现测试2. 修复代码3. 验证测试通过
### 代码审查- 通过所有测试- 符合代码规范- 有适当的文档- 无安全问题- 性能考虑
## 限制事项
### ❌ 禁止操作- 不要修改 `/prisma/schema.prisma`（需团队评审）- 不要安装新依赖包（需在 package.json review 时讨论）- 不要修改 `/lib/auth/*`（认证逻辑敏感）- 不要修改数据库迁移历史文件
### ✅ 允许操作- 可以自由修改 `/components` 和 `/app` 下的业务代码- 可以修改 `/lib` 下的工具函数- 可以修改 `/services` 下的 API 调用层
```

## 六、总结

配置 CLAUDE.md 的核心原则：

1. **从简单开始**：先用 `/init` 生成基础模板，再逐步完善
2. **保持简洁**：控制在 100 行以内，避免冗长
3. **具体可执行**：每条规则都要明确、可验证
4. **随时更新**：用 `#` 键快速添加新规则
5. **团队共享**：提交到版本控制，在 Code Review 中检查变更

记住：CLAUDE.md 是你与 Claude 协作的基础，值得投入时间精心打磨。根据实际测试，配置良好的 CLAUDE.md 能让 AI 的编码准确率提升 5%-10%，同时减少大量无效建议。