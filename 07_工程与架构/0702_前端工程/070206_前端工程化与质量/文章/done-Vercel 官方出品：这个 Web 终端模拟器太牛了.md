> 已吸收至：[[07_工程与架构/0702_前端工程/070206_前端工程化与质量/070206_核心知识点/前端工程化工具链与质量门禁准则|前端工程化工具链与质量门禁准则]]、[[07_工程与架构/0702_前端工程/070206_前端工程化与质量/070206_知识地图|070206_前端工程化与质量知识地图]]

---
title: Vercel 官方出品：这个 Web 终端模拟器太牛了
author: 极客Labs
date:
url: https://mp.weixin.qq.com/s?__biz=MzIzNjIyNTE2Nw==&mid=2651139984&idx=2&sn=9e7fef06d69fc7ba23c1b80f94be14be&chksm=f2ac2d8b4bde43d73c987604827136633d3e0e48c962f1d825fe1cc52a60604aada873b7eeec&mpshare=1&scene=24&srcid=0420W4dizP0hxil703OtQKfY&sharer_shareinfo=2e4a6fde787efe8467a28f987700b3a6&sharer_shareinfo_first=2e4a6fde787efe8467a28f987700b3a6#rd
---

##

## 做 Web 应用开发的时候，经常会遇到需要嵌入终端的场景。

比如给 AI 助手加个命令行界面，或者做在线代码编辑器/Cloud IDE，又或者做技术文档网站需要演示终端操作——以前这种需求要么用笨重的 xterm.js，要么就得绕一大圈。

今天发现 Vercel 官方开源了一个项目 **wterm**，专门解决这个痛点。

GitHub：https://github.com/vercel-labs/wterm

### 为什么值得关注

**1. 技术栈很硬核**

核心是用 Zig 写的 WebAssembly 模块，编译成 WASM 后在浏览器里跑。Zig 代码直接编译成 WASM，不需要任何 Runtime，整个终端模拟逻辑全在 WASM 里。

这样做的好处是性能很好，体积也很小——整个 WASM 模块只有 12KB 左右。

**2. DOM 渲染，不是 Canvas**

和传统的终端模拟器用 Canvas 不同，wterm 用 DOM 来渲染文字。这意味着：

* 原生支持文本选择和复制
* 原生支持剪贴板操作
* 可以用 CSS 自定义样式，包括自定义字体、光标样式、背景等

**3. 集成门槛低**

官方提供了清晰的 API，文档里直接给出了嵌入到 React 的示例。如果你在用 Next.js 或者 Vercel 平台，集成起来会更方便。

### 适合谁用

* **AI 应用开发者：给 AI 助手加个命令行界面，让用户可以直接在 Web 里跑命令**
* **在线 IDE/代码编辑器：做 Cloud IDE 时需要一个轻量好用的终端组件**
* **技术文档网站：演示命令行操作，原生文本选择比 Canvas 流畅多了**

### 怎么用

官方 examples 里有两个主要用法示例，一个是基础的终端渲染，一个是支持多终端实例的管理。

```
bash

# 安装
npm install @vercel/wterm
```

核心思路就是创建一个 WASM 实例，然后往里面写 PTY 的输出数据。官方 examples 里有完整的代码可以参考。

### 小结

如果你在做一个需要嵌入终端的 Web 应用，wterm 值得看看。Vercel 官方出品，代码质量和维护应该不用担心。体积小、DOM 渲染、集成方便这几个特点，对需要这类功能的开发者来说挺实用的。

项目地址：https://github.com/vercel-labs/wterm