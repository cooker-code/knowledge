> 已吸收至：[[07_工程与架构/0702_前端工程/070206_前端工程化与质量/070206_核心知识点/前端工程化工具链与质量门禁准则|前端工程化工具链与质量门禁准则]]、[[07_工程与架构/0702_前端工程/070206_前端工程化与质量/070206_知识地图|070206_前端工程化与质量知识地图]]

---
title: Skills之Tailwind CSS v4 开发技能：GitHub Stars 5.3万，安装10万+
author: 玩转AI技能
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5ODkwMTA1OA==&mid=2247483922&idx=1&sn=0da32bc69b6db04e1b0ddfa0964a2c76&chksm=97c3bf1e9dd62d0e8c525e05ed5031181ff58a394cb67924ffaf363b1471892a98d31b408c46&mpshare=1&scene=24&srcid=0416nu7MOsO4nPWK8e13Ua8E&sharer_shareinfo=ef5668014a9b4b4070aa029a6afa81ea&sharer_shareinfo_first=ef5668014a9b4b4070aa029a6afa81ea#rd
---

**“** 发现值得安装的 AI Skills 与 MCP 工具，支持按分类浏览、查看社区安装量和 GitHub Stars 排行，并提供中英文安装与使用指南。**”**

⚠️重要前提

安装AI Skills的关键前提是：必须科学上网，且开启TUN模式，这一点至关重要，直接决定安装能否顺利完成，在此郑重提醒三遍：**科学上网，科学上网，科学上网。**

# Tailwind CSS 开发

## 适用场景

在以下情况激活此技能：

* 为组件或页面添加样式时
* 处理响应式设计时
* 实现深色模式时
* 将重复模式提取为组件时
* 调试间距或布局问题时

## 文档

使用 `search-docs` 获取详细的 Tailwind CSS v4 模式和文档。

## 基本用法

* 使用 Tailwind CSS 类来为 HTML 添加样式。在引入新模式之前，请检查并遵循项目中现有的 Tailwind 约定。
* 主动提出将重复模式提取为符合项目约定的组件（例如，Blade、JSX、Vue）。
* 考虑类的位置、顺序、优先级和默认值。移除冗余的类，谨慎地向父元素或子元素添加类以减少重复，并合理地对元素进行分组。

## Tailwind CSS v4 特性

* 始终使用 Tailwind CSS v4，并避免使用已弃用的工具类。
* Tailwind v4 不支持 `corePlugins`。

### CSS 优先配置

在 Tailwind v4 中，配置采用 CSS 优先的方式，使用 `@theme` 指令——无需单独的 `tailwind.config.js` 文件：

### 导入语法

在 Tailwind v4 中，使用常规的 CSS `@import` 语句导入 Tailwind，而不是 v3 中使用的 `@tailwind` 指令：

### 已替换的工具类

Tailwind v4 移除了已弃用的工具类。请使用下面显示的替换项。不透明度值保持为数字。

| 已弃用 | 替换项 |
| --- | --- |
| bg-opacity-\* | bg-black/\* |
| text-opacity-\* | text-black/\* |
| border-opacity-\* | border-black/\* |
| divide-opacity-\* | divide-black/\* |
| ring-opacity-\* | ring-black/\* |
| placeholder-opacity-\* | placeholder-black/\* |
| flex-shrink-\* | shrink-\* |
| flex-grow-\* | grow-\* |
| overflow-ellipsis | text-ellipsis |
| decoration-slice | box-decoration-slice |
| decoration-clone | box-decoration-clone |

## 间距

对于兄弟元素之间的间距，使用 `gap` 工具类代替边距：

## 深色模式

如果现有的页面和组件支持深色模式，则新的页面和组件必须以相同的方式支持它，通常使用 `dark:` 变体：

## 常见模式

### Flexbox 布局

### 网格布局

## 常见陷阱

* 使用已弃用的 v3 工具类（bg-opacity-*、flex-shrink-* 等）
* 使用 `@tailwind` 指令而不是 `@import "tailwindcss"`
* 尝试使用 `tailwind.config.js` 而不是 CSS `@theme` 指令
* 使用边距来处理兄弟元素间距而不是 gap 工具类
* 当项目使用深色模式时，忘记添加深色模式变体

安装命令

```
npx skills add https://github.com/coollabsio/coolify --skill tailwindcss-development
```

每周安装量

161

代码仓库

https://github.com/coollabsio/coolify

GitHub 星标数

51.5K

首次出现

2026年2月12日

安全审计

Gen Agent Trust HubPass SocketPass SnykPass

安装于

opencode158

gemini-cli158

codex158

kimi-cli158

amp158

github-copilot158

更多技能>>>

[怎么安装AI Skills](https://mp.weixin.qq.com/s?__biz=MzE5ODkwMTA1OA==&mid=2247483887&idx=1&sn=e2e9b05533f1a417f0af92f0d3108626&scene=21#wechat_redirect)

[find-skills 技能搜索工具 - 让AI更智能的skill](https://mp.weixin.qq.com/s?__biz=MzE5ODkwMTA1OA==&mid=2247483772&idx=1&sn=5ceca663bf7662fb8d4c6e2c3a77b572&scene=21#wechat_redirect)

[Skills之OpenAPI 3.1 规范生成与验证工具 - 从代码生成、设计优先到SDK生成全流程](https://mp.weixin.qq.com/s?__biz=MzE5ODkwMTA1OA==&mid=2247483911&idx=1&sn=931b3d697a7b5fca85da05fac2a36eba&scene=21#wechat_redirect)

[Skills之Laravel安全最佳实践指南 | 防范CSRF、SQL注入、XSS攻击](https://mp.weixin.qq.com/s?__biz=MzE5ODkwMTA1OA==&mid=2247483898&idx=1&sn=c7dfa5a48aa0f8834634f1827fd3f8a0&scene=21#wechat_redirect)

[Skills之Nuxt 4 模式指南：SSR、混合渲染、路由规则与数据获取最佳实践](https://mp.weixin.qq.com/s?__biz=MzE5ODkwMTA1OA==&mid=2247483893&idx=1&sn=033aa296207dddd1a2c3519257cc13c5&scene=21#wechat_redirect)

[Skills之Claude技能创建器指南：构建模块化AI技能包，优化工作流与工具集成](https://mp.weixin.qq.com/s?__biz=MzE5ODkwMTA1OA==&mid=2247483888&idx=1&sn=57a14e9b2731b4f09b19d30d1a4d62a1&scene=21#wechat_redirect)

[Skills之Figma自动化工具：通过Rube MCP实现设计文件管理、组件提取与图像导出](https://mp.weixin.qq.com/s?__biz=MzE5ODkwMTA1OA==&mid=2247483874&idx=1&sn=cc8cd42382fb91a44e38d14aa135aeac&scene=21#wechat_redirect)

[Skills之Azure 部署准备官方指南 | 权威应用部署与基础设施设置步骤](https://mp.weixin.qq.com/s?__biz=MzE5ODkwMTA1OA==&mid=2247483836&idx=1&sn=967b872042448ffcd03c787df8015f4a&scene=21#wechat_redirect)

[Skills之agent-browser 浏览器自动化工具 - Vercel Labs 命令行网页操作与测试](https://mp.weixin.qq.com/s?__biz=MzE5ODkwMTA1OA==&mid=2247483825&idx=1&sn=a618ba4e9dfc90a281d9f8c81bbff715&scene=21#wechat_redirect)

[Skill Creator 技能创建工具 - Anthropic Claude 技能开发指南](https://mp.weixin.qq.com/s?__biz=MzE5ODkwMTA1OA==&mid=2247483820&idx=1&sn=0547facb305e0e8f112b36c073f77c75&scene=21#wechat_redirect)