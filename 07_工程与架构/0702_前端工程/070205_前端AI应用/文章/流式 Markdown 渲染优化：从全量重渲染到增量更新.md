---
title: 流式 Markdown 渲染优化：从全量重渲染到增量更新
author: 前端C罗
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkxNDMwMzUzNQ==&mid=2247483804&idx=1&sn=846fbe335bf5ff4a5dc9dce33f552181&chksm=c0c3b0f9bbbbba57e1f2bca5c9568d57184e0fd303bdde0f9d86d74b5e24ac2311b24407db95&mpshare=1&scene=24&srcid=0106HotUyImQkVcUVR6gdBMm&sharer_shareinfo=3586bb08ce87e891091bf52be2d04fb5&sharer_shareinfo_first=3586bb08ce87e891091bf52be2d04fb5#rd
---

> 本文介绍如何在 LLM 流式输出场景下，实现高性能的 Markdown 渲染方案。

# 一、问题背景

## 1.1 业务场景

LLM 采用流式输出（逐 token 返回 Markdown 文本），降低用户感知延迟。典型场景：用户发送问题后，AI 响应内容逐字显示，而非等待完整响应后一次性渲染。

## 1.2 现有方案的问题

基于react-markdown的传统方案，在流式场景下暴露出严重问题：

#### 问题一：全量重渲染

现象：每次 content 更新，整个文档重新解析和渲染，长文档明显卡顿。

原因分析：react-markdown设计为静态渲染，接收完整 Markdown 字符串后：

调用unified解析为 AST

1. 遍历 AST 生成 React 元素
2. React 执行 reconciliation

流式场景下，每新增一个 token 就触发完整流程，复杂度 O(n) 随内容增长线性上升。

#### 问题二：代码块闪烁

现象：已渲染完成的代码块被反复重新高亮，产生视觉闪烁。

原因分析：react-syntax-highlighter每次接收新 code 时重新计算高亮。即使代码内容未变，父组件重渲染也会触发子组件更新。

#### 问题三：未闭合语法错误

现象：流式输入\*\*bold时，显示为原始文本而非加粗样式。

原因分析：Markdown 解析器要求语法闭合。流式输入过程中，\*\*尚未配对，解析器将其视为普通文本。

#### 问题四：LaTeX 解析错误

现象：未完成的$...公式触发 KaTeX 红色报错信息闪现。

原因分析：KaTeX 对语法错误零容忍，未闭合的$或不完整的公式语法直接抛出异常，异常信息被渲染到页面。

#### 问题五：DOM 抖动

现象：滚动位置丢失，布局频繁跳动。

原因分析：每帧全量替换 DOM 节点，浏览器无法保持滚动状态。React key 不稳定时更为严重。

# 二、方案选型

## 2.1 候选方案对比

| 方案 | 原理 | 优点 | 缺点 |
| --- | --- | --- | --- |
| 虚拟滚动 | 只渲染可视区域 | 减少 DOM 节点 | 无法解决解析开销 |
| Web Worker | 后台线程解析 | 不阻塞主线程 | 通信开销，复杂度高 |
| 增量 DOM | 对比更新 DOM | 减少 DOM 操作 | react-markdown 不支持 |
| **分块渲染** | 拆分为独立块，memo 优化 | 只更新变化块 | 需要精确分块 |

## 2.2 选型决策

最终方案：分块渲染 + React.memo

决策依据：

根因定位：性能瓶颈在于"全量重渲染"，而非 DOM 数量或主线程阻塞

最小改动：复用react-markdown生态，无需重写解析器

可验证性：分块后可通过 React DevTools 直接观察重渲染范围

核心思路：

```
ounter(lineounter(lineounter(lineounter(line流式输入 → remend 补全 → marked.Lexer 分块 → Block[] → memo 渲染                                                    ↓                                    只有最后一个 Block 更新                                    前面的 Block 保持稳定
```

## 2.3 核心依赖

| 库 | 版本 | 选型理由 |
| --- | --- | --- |
| `marked` | 17.x | Lexer 可独立使用，无需完整渲染 |
| `remend` | 1.x | 专为流式 Markdown 设计的补全库 |
| `react-markdown` | 8.x | 成熟的 React Markdown 渲染器 |
| `rehype-katex` | - | KaTeX 官方 rehype 插件 |
| `mermaid` | 11.x | 图表渲染，支持懒加载 |

# 三、核心实现

## 3.1 分块解析算法

目标：将 Markdown 拆分为独立块，每块可独立渲染。

难点：某些语法跨多个 token，不能简单按 token 拆分。

实现策略：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineexport const parseMarkdownIntoBlocks = (markdown: string): string[] => {  // 脚注需要全局上下文，无法分块  if (hasFootnotes(markdown)) return [markdown];  
  const tokens = Lexer.lex(markdown, { gfm: true });  const mergedBlocks: string[] = [];  const htmlStack: string[] = []; // 追踪未闭合 HTML 标签  
  for (const token of tokens) {    // 1. HTML 块追踪：<div> 未闭合时，合并后续内容    // 2. LaTeX 块合并：$ 开始到 $ 结束合并为单块    // 3. 其他块：独立成块  }  return mergedBlocks;};
```

跨块场景处理：

| 场景 | 处理策略 |
| --- | --- |
| `$$` 块级公式 | 追踪 `$$` 计数，奇数时合并后续块 |
| HTML 嵌套 | 使用栈追踪标签开闭状态 |
| 脚注引用 | 放弃分块，整体渲染 |

## 3.2 未闭合语法补全

问题分析：流式输入必然产生未闭合语法，需要"预测性补全"。

解决方案：使用remend库自动补全：

| 输入 | 补全结果 | 原理 |
| --- | --- | --- |
| `**bold` | `**bold**` | 检测未配对 `**` |
| `` `code `` | `` `code` `` | 检测未配对反引号 |  |
| `[link](url` | `[link](streamdown:incomplete-link)` | 补全无效 URL |
| `$$formula` | `$$formula$$` | 检测未配对 `$$` |

## 3.3 Block memo 优化

优化原理：React.memo 浅比较 props，content 未变则跳过渲染。

```
ounter(lineounter(lineounter(lineconst Block = memo(({ content, components }: BlockProps) => (  <Markdown components={components}>{content}</Markdown>), (prev, next) => prev.content === next.content);
```

效果：流式输入时，只有最后一个 Block 触发重渲染，前面的 Block 完全跳过。

## 3.4 Block Key 稳定性

问题分析：流式输入时块数量变化，若使用数组 index 作为 key，会导致：

* 新块插入时，后续块 key 全部变化
* React 误判为节点替换，触发不必要的 DOM 操作

解决方案：使用messageId + index生成稳定 key：

```
ounter(lineconst blockKeys = blocks.map((_, idx) => `${stableId}-block-${idx}`);
```

# 四、LaTeX 流式处理

## 4.1 问题深入分析

KaTeX 解析失败的场景：

| 输入状态 | KaTeX 行为 |
| --- | --- |
| `$$\frac{1}{2}$$` | 正常渲染 |
| `$$\frac{1}{2}` | 抛出 ParseError |
| `$$\frac{1}{` | 抛出 ParseError |
| `$$` | 抛出 ParseError |

根因：KaTeX 设计为一次性解析完整公式，无流式支持。

## 4.2 解决方案

采用"移除 + 补全 + 验证"三步策略：

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineconst processedContent = useMemo(() => {  if (enableLatex && isStreaming) {    // Step 1: 移除未闭合的 $ 块（避免 KaTeX 报错）    const cleaned = removeUnclosedLatexBlock(content);  
    // Step 2: 补全其他语法（不补全 katex，由 Step 1 处理）    const result = remend(cleaned, { katex: false });  
    // Step 3: 验证所有公式有效性    if (!areAllLatexBlocksValid(result)) {      // 无效则使用上次有效内容，避免闪烁      return lastValidContentRef.current;    }  
    lastValidContentRef.current = result;    return result;  }  return content;}, [content, isStreaming, enableLatex]);
```

关键设计：lastValidContentRef缓存上次有效内容，公式输入过程中保持稳定显示。

# 五、Mermaid 流式渲染

## 5.1 问题分析

Mermaid 图表渲染面临多重挑战：

| 问题 | 触发条件 | 表现 |
| --- | --- | --- |
| 语言标识不完整 | ```` ```m ```` → ```` ```me ```` → ```` ```mermaid ```` | 组件类型反复切换 |
| 代码语法错误 | 流式输入的不完整代码 | mermaid.render 抛出异常 |
| 渲染闪烁 | 每次代码变化触发重渲染 | 图表反复重绘 |

## 5.2 多层防护策略

#### 第一层：语言前缀检测

问题：语言标识逐字符到达，m→me→mer→mermaid

方案：检测前缀匹配，显示骨架屏等待完整标识

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line// 流式渲染时，代码内容为空说明语言标识可能还在输入中const isLanguageIncomplete = isStreaming && code.trim() === "";  
// 检测是否为 mermaid 前缀const isMermaidPrefix = isLanguageIncomplete && "mermaid".startsWith(language);  
if (isMermaidPrefix) {  return <MermaidSkeleton />; // 显示骨架屏}
```

#### 第二层：静默失败 + 缓存

问题：不完整代码触发 mermaid.render 异常

方案：捕获异常，保留上次成功的 SVG

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(linetry {  const { svg } = await mermaid.render(uniqueId, code);  lastValidSvgRef.current = svg; // 缓存成功结果  setSvgContent(svg);} catch (err) {  // 流式中静默失败，不显示错误  // 非流式且无缓存时才显示错误  if (!streaming && !lastValidSvgRef.current) {    setError(errorMessage);  }}
```

#### 第三层：显示优先级

```
ounter(lineounter(line// 当前 SVG > 上次成功的 SVG > 骨架屏const displaySvg = svgContent || lastValidSvgRef.current;
```

## 5.3 主题跟随（选看）

需求：Mermaid 图表需跟随 内部Tea 设计系统的 light/dark 主题。

实现：监听 body 属性变化

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineexport function detectTeaThemeMode(): 'light' | 'dark' {  const body = document.body;  const themeMode = body.getAttribute('theme-mode');  const themeEnable = body.getAttribute('theme-enable');  
  // 只有 theme-mode="dark" 且 theme-enable="true" 时才是 dark 模式  return (themeMode === 'dark' && themeEnable === 'true') ? 'dark' : 'light';}  
// MutationObserver 监听属性变化const observer = new MutationObserver(() => {  callback(detectTeaThemeMode());});  
observer.observe(document.body, {  attributes: true,  attributeFilter: ['theme-mode', 'theme-enable'],});
```

# 六、Mermaid 交互功能

## 6.1 PanZoom 实现

需求：支持滚轮缩放和拖拽平移。

技术选型：使用 PointerEvent 实现跨平台支持（鼠标 + 触控）。

这里未使用浏览器自带的溢出滚动，当前体验会更友好。

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineconst ZOOM_CONFIG = { min: 0.25, max: 3, step: 0.25, default: 1 };  
// 滚轮缩放const handleWheel = useCallback((e: WheelEvent) => {  e.preventDefault();  const delta = e.deltaY > 0 ? -zoomStep : zoomStep;  const newZoom = Math.max(minZoom, Math.min(maxZoom, zoom + delta));  onZoomChange(newZoom);}, [zoom, minZoom, maxZoom, zoomStep]);  
// 拖拽平移const handlePointerDown = useCallback((e: React.PointerEvent) => {  if (e.button !== 0) return; // 仅响应左键  setIsPanning(true);  panStartRef.current = { x: e.clientX, y: e.clientY };  panStartPositionRef.current = pan;  (e.target as HTMLElement).setPointerCapture(e.pointerId);}, [pan]);
```

Reset 功能：同时重置 zoom 和 pan

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineconst [resetKey, setResetKey] = useState(0);  
const handleZoomReset = useCallback(() => {  setZoomLevel(ZOOM_CONFIG.default);  setResetKey((prev) => prev + 1); // 触发 pan 重置}, []);
```

## 6.2 PNG 导出

#### 问题一：画布跨域污染

现象：canvas.toBlob()抛出 SecurityError

原因分析：使用URL.createObjectURL(blob)创建的 blob URL，被浏览器视为跨域资源。当 img 加载后绘制到 canvas，canvas 被标记为"tainted"（污染），禁止导出。

解决方案：使用 base64 data URL 替代 blob URL

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line// 将 SVG 转换为 base64 data URLconst svgBase64 = btoa(  encodeURIComponent(svgContent).replace(/%([0-9A-F]{2})/g, (_, p1) =>    String.fromCharCode(parseInt(p1, 16))  ));const dataUrl = `data:image/svg+xml;base64,${svgBase64}`;  
// 加载图片const img = new Image();img.src = dataUrl; // 同源，不会污染 canvas
```

#### 问题二：SVG 尺寸获取

现象：导出的 PNG 只有部分内容

原因分析：SVG 使用 viewBox 定义尺寸，img.width/height返回的是默认值（通常为 0 或 300x150），而非实际内容尺寸。

解决方案：从 SVG 内容解析真实尺寸

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(linefunction getSvgDimensions(svgContent: string): { width: number; height: number } {  const parser = new DOMParser();  const doc = parser.parseFromString(svgContent, "image/svg+xml");  const svgElement = doc.querySelector("svg");  
  // 优先级 1：viewBox  const viewBox = svgElement?.getAttribute("viewBox");  if (viewBox) {    const parts = viewBox.split(/[\s,]+/).map(Number);    if (parts.length === 4) {      return { width: parts[2], height: parts[3] };    }  }  
  // 优先级 2：width/height 属性  const width = parseFloat(svgElement?.getAttribute("width") || "0");  const height = parseFloat(svgElement?.getAttribute("height") || "0");  if (width && height) {    return { width, height };  }  
  // 优先级 3：style 属性  // ...fallback 逻辑}  
// 绑定到 canvas 时明确指定尺寸const { width, height } = getSvgDimensions(svgContent);canvas.width = width;canvas.height = height;ctx.drawImage(img, 0, 0, width, height);
```

# 七、性能对比

## 7.1 测试环境

内容：复杂 AI 响应（代码块 + 表格 + LaTeX + 列表）

流式模拟：20 次增量更新

迭代次数：1000 次

## 7.2 量化指标

| 场景 | 原方案 | 新方案 | 提升 |
| --- | --- | --- | --- |
| 简单内容渲染 | 2.1ms | 2.0ms | ~5% |
| 中等内容渲染 | 8.5ms | 3.2ms | **62%** |
| 复杂内容渲染 | 15.3ms | 4.8ms | **69%** |
| 流式 20 步更新 | 306ms | 48ms | **84%** |
| 大文档 (50 块) | 89ms | 12ms | **87%** |

## 7.3 优化点分析

| 优化项 | 原理 | 效果 |
| --- | --- | --- |
| 分块 memo | 只更新变化的块 | 减少 80%+ 重渲染 |
| 增量 DOM | React 只 patch 最后一个块 | DOM 操作减少 90% |
| 代码高亮复用 | 已完成块不重新高亮 | 避免重复计算 |
| LaTeX 缓存 | 有效公式缓存 | 避免重复解析 |

## 7.4 内存占用

| 指标 | 原方案 | 新方案 |
| --- | --- | --- |
| 峰值内存 | 45MB | 32MB |
| GC 频率 | 高 | 低 |

# 八、后续优化方向

| 优先级 | 方向 | 说明 | 预期收益 |
| --- | --- | --- | --- |
| 高 | CSS `:has()` 降级 | 为旧版浏览器提供 JS fallback | 兼容性 |
| 中 | 代码高亮懒加载 | 按语言按需加载 highlight 规则 | 首屏体积 |
| 中 | 虚拟滚动 | 超长文档只渲染可视区域 | 极端场景性能 |
| 低 | Web Worker 分块 | 后台线程执行 parseMarkdownIntoBlocks | 主线程零阻塞 |
| 低 | WASM marked | 使用 WASM 版本的 Lexer | 分块速度 2-3x |

# 九、总结

本文针对 LLM 流式输出场景下的 Markdown 渲染性能问题，通过分块渲染 + React.memo策略实现了：

性能提升 84%：流式场景下渲染耗时从 306ms 降至 48ms

渲染稳定：通过 remend 补全和 LaTeX 验证，消除闪烁和错误

功能完整：支持 GFM、LaTeX、Mermaid 等丰富语法

交互增强：Mermaid 支持 PanZoom、PNG 导出、主题跟随