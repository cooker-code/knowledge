---
title: Skills之Vercel React 最佳实践指南 | 58条Next.js性能优化规则与代码重构
author: 玩转AI技能
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5ODkwMTA1OA==&mid=2247483784&idx=1&sn=4e51e480fb695b1cddca4db83fef773c&chksm=9717358e6ee47ff4b12d51c68514be84e54ee751f063dd70541ad8f41a285b51e9f851de74b7&mpshare=1&scene=24&srcid=0415nIRl16uBca5tjtMIB9DQ&sharer_shareinfo=9f453785984d94f9197ad8f1e18b0c4f&sharer_shareinfo_first=9f453785984d94f9197ad8f1e18b0c4f#rd
---

# Vercel React 最佳实践

由 Vercel 维护的 React 和 Next.js 应用程序综合性能优化指南。包含 8 个类别共 58 条规则，按影响优先级排序，以指导自动化重构和代码生成。

# 何时应用

在以下情况下参考这些指南：

编写新的 React 组件或 Next.js 页面时

实现数据获取（客户端或服务器端）时

审查代码以查找性能问题时

重构现有 React/Next.js 代码时

优化包大小或加载时间时

# 按优先级划分的规则类别

| 优先级 | 类别 | 影响 | 前缀 |
| --- | --- | --- | --- |
| 1 | 消除瀑布流 | 关键 | `async-` |
| 2 | 包大小优化 | 关键 | `bundle-` |
| 3 | 服务器端性能 | 高 | `server-` |
| 4 | 客户端数据获取 | 中高 | `client-` |
| 5 | 重新渲染优化 | 中 | `rerender-` |
| 6 | 渲染性能 | 中 | `rendering-` |
| 7 | JavaScript 性能 | 低中 | `js-` |
| 8 | 高级模式 | 低 | `advanced-` |

# 快速参考

# 1. 消除瀑布流（关键）

## async-defer-await

- 将 await 移动到实际使用的分支中

## async-parallel

- 对独立操作使用 Promise.all()

## async-dependencies

- 对部分依赖项使用 better-all

## async-api-routes

- 在 API 路由中尽早启动 Promise，稍后 await

## async-suspense-boundaries

- 使用 Suspense 流式传输内容

# 2. 包大小优化（关键）

## bundle-barrel-imports

- 直接导入，避免使用桶文件

## bundle-dynamic-imports

- 对重型组件使用 next/dynamic

## bundle-defer-third-party

- 在 hydration 后加载分析/日志记录

## bundle-conditional

- 仅在功能激活时加载模块

## bundle-preload

- 在悬停/聚焦时预加载以提高感知速度

# 3. 服务器端性能（高）

## server-auth-actions

- 像 API 路由一样验证服务器操作

## server-cache-react

- 使用 React.cache() 进行请求级去重

## server-cache-lru

- 使用 LRU 缓存进行跨请求缓存

## server-dedup-props

- 避免在 RSC props 中重复序列化

## server-hoist-static-io

- 将静态 I/O（字体、徽标）提升到模块级别

## server-serialization

- 最小化传递给客户端组件的数据

## server-parallel-fetching

- 重构组件以实现并行获取

## server-after-nonblocking

- 对非阻塞操作使用 after()

# 4. 客户端数据获取（中高）

## client-swr-dedup

- 使用 SWR 实现自动请求去重

## client-event-listeners

- 对全局事件监听器进行去重

## client-passive-event-listeners

- 对滚动使用被动监听器

## client-localstorage-schema

- 对 localStorage 数据进行版本控制和最小化

# 5. 重新渲染优化（中）

## rerender-defer-reads

- 不要订阅仅在回调中使用的状态

## rerender-memo

- 将昂贵的工作提取到记忆化组件中

## rerender-memo-with-default-value

- 提升默认的非原始 props

## rerender-dependencies

- 在 effects 中使用原始依赖项

## rerender-derived-state

- 订阅派生的布尔值，而非原始值

## rerender-derived-state-no-effect

- 在渲染期间派生状态，而非在 effects 中

## rerender-functional-setstate

- 使用函数式 setState 以获得稳定的回调

## rerender-lazy-state-init

- 将函数传递给 useState 以处理昂贵的值

## rerender-simple-expression-in-memo

- 避免对简单原始值使用 memo

## rerender-move-effect-to-event

- 将交互逻辑放在事件处理程序中

## rerender-transitions

- 对非紧急更新使用 startTransition

## rerender-use-ref-transient-values

- 使用 refs 处理瞬态频繁变化的值

# 6. 渲染性能（中）

## rendering-animate-svg-wrapper

- 对 div 包装器进行动画处理，而非 SVG 元素

## rendering-content-visibility

- 对长列表使用 content-visibility

## rendering-hoist-jsx

- 将静态 JSX 提取到组件外部

## rendering-svg-precision

- 降低 SVG 坐标精度

## rendering-hydration-no-flicker

- 对仅客户端数据使用内联脚本

## rendering-hydration-suppress-warning

- 抑制预期的失配警告

## rendering-activity

- 使用 Activity 组件进行显示/隐藏

## rendering-conditional-render

- 使用三元运算符，而非 && 进行条件渲染

## rendering-usetransition-loading

- 对加载状态优先使用 useTransition

# 7. JavaScript 性能（低中）

## js-batch-dom-css

- 通过类或 cssText 分组 CSS 更改

## js-index-maps

- 为重复查找构建 Map

## js-cache-property-access

- 在循环中缓存对象属性

## js-cache-function-results

- 在模块级 Map 中缓存函数结果

## js-cache-storage

- 缓存 localStorage/sessionStorage 读取

## js-combine-iterations

- 将多个 filter/map 合并到一个循环中

## js-length-check-first

- 在进行昂贵比较之前检查数组长度

## js-early-exit

- 尽早从函数返回

## js-hoist-regexp

- 将 RegExp 创建提升到循环外部

## js-min-max-loop

- 使用循环求最小/最大值，而非 sort

## js-set-map-lookups

- 使用 Set/Map 进行 O(1) 查找

## js-tosorted-immutable

- 使用 toSorted() 实现不可变性

# 8. 高级模式（低）

## advanced-event-handler-refs

- 将事件处理程序存储在 refs 中

## advanced-init-once

- 每次应用加载时初始化一次应用

## advanced-use-latest

- 使用 useLatest 获得稳定的回调 refs

# 如何使用

阅读单个规则文件以获取详细解释和代码示例：

rules/async-parallel.md

rules/bundle-barrel-imports.md

每个规则文件包含：

简要解释为何重要

错误的代码示例及解释

正确的代码示例及解释

其他上下文和参考

完整编译文档

包含所有规则详解的完整指南：AGENTS.md

每周安装量

186.9K

仓库

安装命令

```
npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-react-best-practices
```

GitHub 星标数

22.4K

首次出现

2026 年 1 月 19 日

安全审计

安装于

opencode127.9K

gemini-cli125.7K

codex124.5K

github-copilot114.5K

claude-code108.9K

cursor107.8K