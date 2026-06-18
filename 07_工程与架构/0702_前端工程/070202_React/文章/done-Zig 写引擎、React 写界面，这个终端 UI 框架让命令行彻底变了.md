> 已吸收至：[[07_工程与架构/0702_前端工程/070202_React/070202_核心知识点/React架构与质量边界准则|React架构与质量边界准则]]、[[07_工程与架构/0702_前端工程/070202_React/070202_知识地图|070202_React知识地图]]

---
title: Zig 写引擎、React 写界面，这个终端 UI 框架让命令行彻底变了
author: 何三笔记
date: 何三何三
url: https://mp.weixin.qq.com/s?__biz=MzA4NTI3OTcyMA==&mid=2649632636&idx=1&sn=ee0257d893661b5c7e40e0add396cb00&chksm=862bef5b9855693910df8446a87d4411ea0ced3467bcc6cadc5a9fe007684be7324b197fc716&mpshare=1&scene=24&srcid=0602k9WR3WDyeuhdFGC8jqBg&sharer_shareinfo=0c3269f9ec4a7e5c17e808006df6f2b5&sharer_shareinfo_first=0c3269f9ec4a7e5c17e808006df6f2b5#rd
---

大家好，我是何三，独立开发者

我第一次看到 OpenTUI 的时候，整个人是懵的。

一个用 **Zig** 写的终端 UI 框架，竟然支持 **React** 和 **SolidJS** 组件，还能在终端里跑 **Three.js WebGPU 渲染**。

你没看错，终端，命令行那个终端，跑 Three.js。

这不是什么玩具项目。OpenCode（一个 AI 代码编辑器）已经在生产环境用它了，terminal.shop 也在用。一个 Zig 写的底层引擎，加了层 TypeScript 绑定，然后前端框架的组件可以直接扔进终端渲染——这玩意儿说实话，我一开始以为是个整活项目。

结果发现人家是认真的。

### 为什么说这个项目很"不对劲"

先说底层。

终端 UI 框架其实不少。Node.js 那边有 Ink（用 React 写终端），Python 有 Rich，Go 有 Bubble Tea。但它们都有一个共同问题——**性能天花板**。

因为都是解释型语言写的，想在终端里搞复杂的布局、动画、异步渲染，很快就会碰到瓶颈。帧率上不去，内存下不来。

OpenTUI 的做法就有点不讲道理了。

它用 **Zig** 写了一个原生核心，通过 C ABI 暴露接口，上层再封装 TypeScript 绑定。Zig 是什么？它是一门系统级编程语言，比 C 更现代，比 Rust 更简洁，编译出来的二进制小得离谱——但这个语言在国内知道的人真不多。

终端UI进化史

等等，说到 Zig，我突然想到一个事。

去年有个叫 Ghostty 的终端模拟器，也是 Zig 写的，火得一塌糊涂。当时我还在想，Zig 这语言是不是要起飞了。结果现在又冒出来一个 OpenTUI，还是 Zig 写的。这俩项目要是联个动，一个 Zig 写的终端跑一个 Zig 写的 TUI 框架——画面太美我不敢看。

说回正题。

OpenTUI 最骚的操作是什么？它搞了 **React reconciler** 和 **SolidJS reconciler**。

什么意思？

就是你可以用 React 的 JSX 语法直接写终端界面。状态管理、组件化、响应式更新——这些前端概念，全都能用在命令行里。

```
import { createCliRenderer, Text, Box } from "@opentui/core"

const renderer = await createCliRenderer({
  exitOnCtrlC: true,
})

renderer.root.add(
  Box({
    direction: "col",
    children: [
      Text({ content: "你好，OpenTUI！", fg: "#00FF00", bold: true }),
      Text({ content: "按 Ctrl+C 退出", fg: "#888888" }),
    ],
  })
)
```

说白了，以前你想写一个好看的终端工具，得用 ncurses 或者直接跟 ANSI 转义码死磕。现在你写终端界面，就跟写网页一样——JSX 拼组件，CSS 类似的属性调样式。

为什么这么设计？别问我，问作者去。反正我个人觉得，这个思路确实让 TUI 开发的门槛降了一大截。

### 组件多到离谱

OpenTUI 内置的组件阵容，看得我有点想吐槽——一个终端框架，你搞这么多东西干嘛？

* **Text** / **Box** — 基础组件
* **Input** / **Textarea** — 表单组件
* **Select** / **TabSelect** — 选择器
* **ScrollBox** / **ScrollBar** — 滚动组件
* **Slider** — 滑块
* **Code** — 代码渲染（带语法高亮）
* **Markdown** — Markdown 渲染
* **FrameBuffer** — 帧缓冲
* **ASCIIFont** — ASCII 艺术字生成
* **Diff** — 代码对比
* **QR Code** — 二维码生成

对，你没看错，一个终端框架内置了 **二维码生成** 和 **Diff 对比**。

这个 QR Code 组件让我想了很多。比如你可以写个终端工具，跑完任务直接输出一个二维码，扫码跳到某个页面。这太适合 DevOps 场景了——部署完 CI/CD，终端里直接蹦个二维码，扫码看结果。

还有那个 **FrameBuffer** 组件，它其实是 Three.js WebGPU 渲染器的底层支持。OpenTUI 有个叫 `@opentui/three` 的包，可以在终端里用 Three.js 做 3D 渲染。

这个压缩率——算了先不说这个，你先装上看效果。

### 一分钟上手

装 OpenTUI 很简单，用 Bun 装就行：

```
bun create tui
```

或者手动装：

```
mkdir my-tui && cd my-tui
bun init -y
bun add @opentui/core
```

然后写个 `index.ts`：

```
import { createCliRenderer, Text } from "@opentui/core"

const renderer = await createCliRenderer({
  exitOnCtrlC: true,
})

renderer.root.add(
  Text({
    content: "Hello, OpenTUI!",
    fg: "#00FF00",
  })
)
```

跑起来：

```
bun run index.ts
```

终端里就出现了一行绿色的 "Hello, OpenTUI!"。

说实话，这块我也没完全搞懂它的渲染管线是怎么跟 Zig 原生核心交互的。大致流程就是 TypeScript 层把组件树 diff 一下，然后通过 C ABI 把渲染指令发给 Zig 核心，Zig 那边直接用原生方式写终端帧缓冲。细节可能有出入——有懂的大佬欢迎指正。

目前 OpenTUI 是 **Bun 独占**的，Deno 和 Node 的支持正在开发中。所以想尝鲜的话，得先装个 Bun。

### 同类项目跟它有什么不一样

终端 UI 框架这块，市面上其实有几条路线：

| 项目 | 语言 | 特点 |
| --- | --- | --- |
| **Ink** | JavaScript | React 写终端，但性能受限于 Node.js |
| **Bubble Tea** | Go | 基于 Elm 架构，生态成熟 |
| **Rich** | Python | 纯文本渲染，不适合复杂交互 |
| **Ratatui** | Rust | Rust 生态，性能好但上手门槛高 |
| **OpenTUI** | Zig + TS | 原生性能 + 前端框架绑定，性能与开发的平衡点 |

OpenTUI 的定位其实很特别——它不是跟 Ink 抢饭碗，而是给那些觉得 Ink 性能不够、又不想跳进 Rust 深坑的人一个中间选择。

底层用 Zig 保证性能，上层用 TypeScript 保证开发体验。这个组合拳打得挺聪明。

如果你对 Zig 生态感兴趣，我上次还写过一篇关于 Zig 语言生态的文章（公众号回复「Zig」可以看），里面提到了 Ghostty 和 TigerBeetle 等项目，它们跟 OpenTUI 算是同一个技术浪潮。

### 几点吐槽和期待

文档写得有点抽象。很多概念对刚从网页开发转过来的人不太友好，比如 "Renderables vs Constructs" 那块，我看了三遍才大概明白区别。

不过好在它们有丰富的 **Examples** 包，可以直接跑起来看效果：

```
curl -fsSL https://raw.githubusercontent.com/anomalyco/opentui/main/packages/examples/install.sh | sh
```

还有一件事我觉得挺酷的——OpenTUI 支持 **AI Agent Skill**。你可以用 `npx skills add anomalyco/opentui` 把它的 API 知识库装到你的 AI 编码助手里。说白了，你写代码的时候 AI 可以直接帮你生成 OpenTUI 的组件代码。

这个功能我真的吹爆。

### 总结

OpenTUI 不是那种"装不装都行"的项目。如果你写 CLI 工具、做 DevOps 面板、搞终端交互式应用，它值得你花半小时折腾一下。

核心就一句话：**Zig 保性能，React 保体验，一个让你用前端思维写终端的框架。**

装不装都行，看你自己。

> 项目地址：https://github.com/anomalyco/opentui

*本文使用 MGO 编辑并发布*

> 关注"何三笔记"，回复"mgo" 免费下载使用