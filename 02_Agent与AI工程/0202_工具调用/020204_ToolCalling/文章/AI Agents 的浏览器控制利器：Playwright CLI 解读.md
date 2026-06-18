---
title: AI Agents 的浏览器控制利器：Playwright CLI 解读
author: AI秋得狠
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYzMTM0ODkxOQ==&mid=2247487806&idx=1&sn=5393f82c7dea143dc31af88656309df4&chksm=f19ce2694372299438869a91a4f54889b7c2e34fec2c5dbe3d7f3b6b5bbd09a3607842a51a35&mpshare=1&scene=24&srcid=0424OEnZ17cHDGElKb8puxZI&sharer_shareinfo=07d811157934979329f561ae71741ae6&sharer_shareinfo_first=07d811157934979329f561ae71741ae6#rd
---

让 AI Agents 自动操控浏览器完成网页操作，是个常见需求。

怎么让 Agent 知道页面上有哪些按钮、输入框可以操作？操作完了怎么拿到结果继续下一步？

这背后有一整套信息传递的逻辑，Playwright CLI 把这件事做得很漂亮。今天展开聊聊它的设计思路。

## Playwright 简史：从人类用到 AI 用

Playwright 是微软 2019 年推出的浏览器自动化工具，和 Puppeteer 是"表亲"——都是通过 CDP 控制浏览器。

**人类用法**是这样的：写 JavaScript/TypeScript 代码，import Playwright 库，操作元素。

```
const { chromium } = require('playwright');  
const browser = await chromium.launch();  
const page = await browser.newPage();  
await page.goto('https://example.com');  
  
// 用 CSS Selector 找到按钮然后点击  
await page.click('#submit-btn');  
await page.fill('input[name="email"]', 'test@example.com');  
await browser.close();
```

但交给 AI Agent 就有问题了：每次操作之前，Agent 需要知道当前页面有哪些可交互元素。用 CSS Selector 传给 Agent？`#submit-btn` 这样的东西对 Agent 来说既不语义化，也不稳定——页面改一改 selector 就失效了。

于是微软给 Playwright 增加了 CLI 模式，专门针对 AI Agent 使用场景优化。这就是 **Playwright CLI**——**不是替代原有功能，而是适配 AI Agent 的工作方式。**

## CLI + Skill：AI Native 应用的主流模式

先说个大背景。

**CLI + Skill 的组合正在成为 AI Native 应用的主流架构。**

传统 MCP（Model Context Protocol）模式，是 Agent 通过持久化的协议连接维持上下文：

```
Agent → MCP Server → 浏览器 → 返回完整的 accessibility tree → Agent  
↑ 每次操作都要把整个页面结构传回来，Token 爆炸
```

CLI + Skill 模式不一样。Agent 每次需要什么能力，就调用对应的 Skill——一个独立的 CLI 工具：

```
Agent → CLI 命令（如 click e15）→ 返回精简结果 → Agent  
↑ 只传递必要的引用（ref）和结果，不传完整页面结构
```

**命令 → 结果，完事走人，不拖泥带水。**

Playwright CLI 就是这种模式的典型代表。它有几个具体机制保证 Token 高效：

* **Snapshot 按需调用**：Agent 需要时才调用 `snapshot`，不需要每次操作都返回完整页面树
* **Ref 引用机制**：页面元素用短引用（如 `e15`），不传完整 CSS selector
* **`--depth=N` 限制深度**：可以只取部分 DOM 结构，不抓全树
* **工具 schema 更小**：CLI 命令参数简单，而 MCP 的工具 schema 非常庞大要占用大量 context
* **不强制 LLM 读页面**：原文："Does not force page data into LLM"

典型例子：

* **Claude Code**：工具封装成 CLI，Agent 按需调用
* **Playwright CLI**：微软出的浏览器自动化方案
* **Vercel Agent Browser**：Vercel 出的同类型方案

为什么这种模式更受欢迎？

**1. 易于构建。** CLI 开发成本低，不需要维护持久化协议服务器。

**2. 易于分发。** 一个 CLI 工具可以被任何 AI 框架调用，不挑框架。

**3. 易于扩展。** 想加新功能？写个新的 CLI 就行。

**4. 能力强但资源占用低。** 按需调用，不维持持久上下文，Token 消耗可控。

## Playwright CLI 核心流程

Playwright CLI 把相同能力封装成终端命令，Agent 直接敲命令操控浏览器：

```
# 打开浏览器访问页面  
playwright-cli open https://example.com  
  
# 获取页面快照（只看可交互元素）  
playwright-cli snapshot -i  
  
# 用 Ref 引用点击按钮  
playwright-cli click e3  
  
# 填一个输入框  
playwright-cli fill e1 "hello world"
```

这里的 `e1`、`e3` 就是 Ref——Playwright CLI 给页面元素分配的短引用。Agent 不需要知道 CSS Selector 或 XPath，只需要记住「e3 是那个登录按钮」就够了。

## 核心技术：Accessibility Tree 到 Ref 的映射

为什么 Agent 直接用 CSS Selector 或 XPath 不行？

**因为页面的 DOM 结构对 LLM 来说太臃肿了。**

Playwright CLI 的解决方案是：**把 DOM 转换成 Accessibility Tree，再提取可交互元素，分配 Ref 引用。**

用一个具体例子说明整个过程。

### 测试页面

```
<!DOCTYPE html>  
<html>  
<head><title>Token 对比演示</title></head>  
<body>  
<header>  
  <h1>我的博客</h1>  
  <nav>  
    <a href="/">首页</a>  
    <a href="/about">关于</a>  
    <a href="/contact">联系</a>  
  </nav>  
  <button id="login">登录</button>  
</header>  
<main>  
  <form>  
    <label for="search">搜索</label>  
    <input type="text" id="search" placeholder="输入搜索内容...">  
    <button type="submit">搜索</button>  
  </form>  
  <div class="articles">  
    <article>  
      <h3>文章标题 1</h3>  
      <p>第一篇文章描述。</p>  
      <button class="read-more">阅读更多</button>  
    </article>  
    <article>  
      <h3>文章标题 2</h3>  
      <p>第二篇文章描述。</p>  
      <button class="read-more">阅读更多</button>  
    </article>  
    <article>  
      <h3>文章标题 3</h3>  
      <p>第三篇文章描述。</p>  
      <button class="read-more">阅读更多</button>  
    </article>  
  </div>  
</main>  
</body>  
</html>
```

这个页面有 9 个可交互元素：3 个链接、2 个按钮、1 个输入框、3 个"阅读更多"按钮。

### 第一步：CDP 获取原始 Accessibility Tree

浏览器内部维护着一棵 Accessibility Tree，是给屏幕阅读器用的——告诉视障用户页面上有哪些按钮、输入框、链接。

通过 Chrome DevTools Protocol，Playwright CLI 调用 `Accessibility.getFullAXTree`，拿到原始树。返回的是**扁平节点数组**，每个节点结构如下：

```
{  
  nodeId: "5",  
  role: { type: "internalRole", value: "banner" },  
  name: "",  
  ignored: false,  
  childIds: ["6", "13"],  
  properties: [...],  
  backendDOMNodeId: 6,  
  chromeRole: 94  
}
```

注意每个属性都是 `{type, value}` 的序列化格式，需要解析成标准值。

同一个页面，CDP 返回了 **72 个节点**，总大小 12,141 chars——比原始 DOM 大了 14 倍，因为每个节点有大量元数据。

### 第二步：扁平数组组装成树

用 `nodeId` 做 key 建立映射，再递归用 `childIds` 组装成树。

### 第三步：遍历树，分配 Ref

遍历过程中：

* 跳过 `ignored=true` 的节点（装饰性元素，如纯样式的 div）
* 遇到可交互角色（button、textbox、checkbox、link 等），分配 Ref

**完整提取流程：**

```
72 个 CDP 节点  
    ↓ getValue() 解析 {type, value} 格式  
扁平对象数组  
    ↓ 建立 nodeId → node 映射  
    ↓ 递归用 childIds 组装成树  
树形结构  
    ↓ 遍历，遇到可交互 role (button/textbox/link...)  
    ↓ 分配 ref: e1, e2, e3...  
    ↓ ignored=true 跳过，但仍递归其子节点
```

### 最终输出

```
- e1 [link] "首页"  
- e2 [link] "关于"  
- e3 [link] "联系"  
- e4 [button] "登录"  
- e5 [textbox] "搜索"  
- e6 [button] "搜索"  
- e7 [button] "阅读更多"  
- e8 [button] "阅读更多"  
- e9 [button] "阅读更多"
```

从 12,141 chars 的 CDP 原始数据，压缩到只有 9 行。简洁多了吧？

## Token 节省：4 倍的实质

同一页面，各方案数据对比：

| 方案 | 大小 | 相对 CLI |
| --- | --- | --- |
| DOM (outerHTML) | 855 chars | **3.9x 更大** |
| 完整 AX Tree (CDP) | 12,141 chars | **55x 更大** |
| **CLI Ref 输出** | **220 chars** | **基准** |

关键洞察：

**DOM 本身并不小**。HTML 标签、属性、class 加起来已经有 855 chars。

**CDP 返回的 AX Tree 反而更大**。因为每个节点有 nodeId、childIds、properties、chromeRole、backendDOMNodeId 等元数据，72 个节点撑到了 12k chars。

**Ref 输出只有 220 chars**。只保留 role + name，Agent 拿到的是最精简的信息。

55 倍听起来夸张，但实际场景中：

* 复杂页面有大量 div/span 等无语义标签，CDP 节点数量会爆炸
* DOM 里充斥着 class、style、data-\* 属性
* Ref 只保留"需要操作的元素"，过滤掉了 90% 的页面内容

**省 Token 的本质：只传"需要操作的元素"，不传整个页面的结构。**

对于中等复杂度的页面（几十个可交互元素 + 几百个装饰性元素），Token 节省轻松超过 4 倍。

## 踩坑：ignored 节点的递归陷阱

最后说一个容易踩的坑。

在提取 Ref 时，最初我这样想：**遇到 ignored 节点就停止递归**。

但问题是：**如果一个父节点 ignored=true，不代表它的子节点也要被跳过。**

举例：

```
<div role="none">  
  <button>点击我</button>  
</div>
```

这个 div 的 role 是 "none"，可能会被标记为 ignored。但它的子元素 button 是可交互的，应该被提取。

正确做法：**跳过 ignored 节点的 Ref 分配，但仍然递归它的子节点。**

这个细节很容易被忽略。

## 总结

今天聊了 Playwright CLI 的设计思路，三个核心结论：

**1. CLI + Skill 是 AI Native 应用的主流模式。** 按需调用，易构建易分发易扩展，Token 消耗可控。

**2. Accessibility Tree 是省 Token 的关键。** 通过 CDP 获取原始树，过滤 ignored 节点，提取可交互元素，分配 Ref 引用——最终输出只有原始 DOM 的 1/4。

**3. Ref 引用让 Agent 操作更简单。** 不需要 CSS Selector 或 XPath，只需要记住 `e3 是登录按钮` 就能完成点击。

完整代码和调研过程在这里：playwright-cli[1]

### 引用链接

[1]github.com/microsoft/playwright-cli: *https://github.com/microsoft/playwright-cli*