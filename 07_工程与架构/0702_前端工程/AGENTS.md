# 前端工程

> 分类规则、一二级类目定位、跨域归类边界以根 [目录划分.md](../../目录划分.md) 为准。
> 文章理解的四步法、整理流程以根 [AGENTS.md](../../AGENTS.md) 为准。
> 本文件只维护本领域的认知重点、排重指纹、已覆盖三级节点和待补缺口。

## 三级节点入口

| 节点 | 用途 |
|---|---|
| [070201_Nuxt](070201_Nuxt/AGENTS.md) | Nuxt 目录、Layers、Nitro 与 SSR 路线 |
| [070202_React](070202_React/AGENTS.md) | React 组件、Hook、Next/TanStack Start |
| [070203_TypeScript](070203_TypeScript/AGENTS.md) | 类型系统与前端类型边界 |
| [070204_Vue](070204_Vue/AGENTS.md) | Vue3、Pinia、组件集成与高交互应用 |
| [070205_前端AI应用](070205_前端AI应用/AGENTS.md) | 生成式 UI、前端 AI SDK 与工具调用 |
| [070206_前端工程化与质量](070206_前端工程化与质量/AGENTS.md) | 构建、质量门禁、测试与发布闭环 |

## 用户认知重点

| 项 | 内容 |
|---|---|
| 已知基础 | 用户熟悉常见框架与组件模式；不需重复讲入门 |
| 待补边界 | React/Vue/Nuxt 生产架构差异、TypeScript 运行时边界、AI UI 权限边界 |
| 易偏差点 | 教程混入资讯与工具推荐；演示能力误当作生产落地能力 |
| 优先抽取 | 框架边界、状态/数据/渲染流程节点、可迁移工程准则 |

## 排重指纹

```text
前端路线 + 流程节点（场景/框架/组件/数据/渲染/构建/AI）+ 核心机制 + 解决问题 + 生产边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是 React | 按组件组合、数据获取、Next/TanStack、质量与非浏览器渲染拆分 |
| 都是 Nuxt | 按目录、Layers、Nitro、SSR/CSR、认证、MDC 拆分 |
| 都是 AI UI | 按生成式 UI、Provider、Tool Calling、可操作页面拆分 |
| 只是趋势资讯 | 没有可迁移准则时不新建 |
| 跨框架对比 | 同一流程节点差异沉淀到对应路线，不混到单一路线 |

## 已覆盖问题（按三级节点）

| 节点 | 已覆盖 | 还缺 |
|---|---|---|
| React | 组件组合、Next/TanStack Start、React Query、质量扫描、非浏览器渲染 | 真实项目分层与测试 |
| Vue | Pinia、样式工程、高交互组件、数字孪生、BI 组件、Vue3 + FastAPI 全栈案例 | 组合式函数、路由、测试、部署 |
| Nuxt | Nuxt 4、Layers、Nitro、认证、服务端/客户端组件、MDC | 生产部署与缓存策略 |
| TypeScript | 类型基础梳理、函数接口 | 运行时校验与外部输入边界 |
| 前端 AI 应用 | Tambo、TanStack AI、AI Elements Vue、AI 版 Chrome、Ant Design AI、json-render、流式 Markdown、生成式 UI 白名单/Schema/回滚 | 权限确认、观测评估和生产可控性证据 |
| 前端工程化与质量 | 前端趋势、ESLint、Flutter/Web Install 资讯、CLI E2E 测试、Tailwind Skill、Web 终端组件 | 构建、质量门禁、性能、可访问性、发布闭环 |

## 待补缺口

| 优先级 | 项 | 为什么补 |
|---|---|---|
| P0 | React 与 Nuxt 生产架构边界 | 资讯/教程/工具混杂，需抽稳定工程准则 |
| P0 | 前端 AI 应用权限与工具调用边界 | 已补生成式 UI 控制边界，仍缺真实权限、审计和回滚证据 |
| P1 | Vue 工程化主线 | 现有文章偏组件集成，缺项目级流程 |
| P1 | TypeScript 运行时边界 | 仅类型基础不能支撑外部输入安全 |
| P1 | 前端质量门禁 | 缺测试、性能、可访问性、安全与发布回归 |
