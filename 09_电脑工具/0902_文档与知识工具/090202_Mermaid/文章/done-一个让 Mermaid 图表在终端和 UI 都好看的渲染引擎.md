---
title: 一个让 Mermaid 图表在终端和 UI 都好看的渲染引擎
author: 脑哩智
date:
url: https://mp.weixin.qq.com/s?__biz=MzUzMzE2ODc3NA==&mid=2247484177&idx=1&sn=55b70f5a0f59b015ebbfb28982d658ec&chksm=fb3ba89a03eab856d26af7a5418a2b04c21bcdcede4e13c19879e0bf8f6023216dbe3deaca41&mpshare=1&scene=24&srcid=0204Yk5bpthOocMcmZZzwp4v&sharer_shareinfo=2db13d10aa0cd3b8d95117c8b43faf71&sharer_shareinfo_first=2db13d10aa0cd3b8d95117c8b43faf71#rd
---
> 已吸收至：[[09_电脑工具/0902_文档与知识工具/090202_Mermaid/090202_核心知识点/Mermaid图表质量与渲染边界|Mermaid 图表质量与渲染边界]]


# 项目概览

---

# Beautiful Mermaid 是 Craft 团队为其 AI 编程助手开发的图表渲染工具。它解决了一个具体问题：在 AI 辅助编程场景中，开发者需要在终端、聊天界面等多种环境下快速可视化系统架构和数据流程。

#

传统 Mermaid 渲染器依赖 DOM、主题配置复杂、且无法输出 ASCII 格式。这个项目用纯 TypeScript 重写了渲染逻辑，同时支持 SVG 和 ASCII 两种输出格式，并将主题系统简化到只需两个颜色值。

目前支持 5 种图表类型：流程图、状态图、时序图、类图和 ER 图。

## 核心技术点

**双色派生系统**
通过 CSS `color-mix()` 函数，仅用背景色和前景色即可自动派生出文本、连接线、节点边框等所有视觉元素的颜色。这种设计让主题定义从数十行 CSS 简化为两个 Hex 值。

**零 DOM 依赖的渲染**
完全用 TypeScript 实现 SVG 生成逻辑，不依赖浏览器 DOM API。这使得它可以在 Node.js、Bun 等服务端环境运行，也方便集成到 CLI 工具中。

CSS 变量实现的热切换
所有颜色都映射为 SVG 根元素的 CSS 自定义属性。切换主题时只需修改这些变量，无需重新渲染整个图表，性能开销接近零。

## 适用场景

这个工具主要面向三类使用场景：

**AI 编程工具开发者**：需要在聊天界面或终端中实时展示架构图的场景（如 Cursor、Continue 等 AI 编辑器插件）。

**CLI 工具作者**：希望在命令行输出中嵌入流程图或状态图，但不想依赖图形界面的项目。

**文档系统维护者**：需要统一管理大量图表主题，且希望主题能与代码高亮方案（如 Shiki）保持一致的团队。

## 总结

从技术实现看，Beautiful Mermaid 的价值在于解决了 Mermaid 官方渲染器在特定场景下的工程问题：体积、依赖和输出格式的限制。双色派生系统是个聪明的设计，但也意味着对复杂配色需求的支持有限。

ASCII 渲染部分移植自 Go 语言项目 mermaid-ascii，目前支持的图表类型覆盖了常见需求。如果你正在构建需要在多种环境下展示图表的工具，且不想引入完整的 Mermaid.js，这是个值得尝试的方案。

演示地址：

https://agents.craft.do/mermaid

项目地址：

https://github.com/lukilabs/beautiful-mermaid
