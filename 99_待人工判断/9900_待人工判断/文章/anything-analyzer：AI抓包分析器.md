---
title: anything-analyzer：AI抓包分析器
author: 攻防录
date: 攻防路攻防路
url: https://mp.weixin.qq.com/s?__biz=MzY5ODAyOTAwMg==&mid=2247484875&idx=1&sn=2cf7e2e15f83e0e6619d940b77b3253c&chksm=f53c43e7c92b18463860c00fbe51191b3368d84813c02135d0b3c586258f9025fe702a81b864&mpshare=1&scene=24&srcid=050848PiIkghx0GCtFzFDWFE&sharer_shareinfo=752bfd644223db5988f3146e18290192&sharer_shareinfo_first=752bfd644223db5988f3146e18290192#rd
---

`Anything Analyzer` 是一款桌面端协议分析工具。网页、桌面应用、终端命令、Python 脚本、手机 App 的 HTTP/HTTPS 流量，都可以汇入同一个 Session，再交给 AI 自动分析。

项目地址： https://github.com/Mouseww/anything-analyzer

Releases 下载： https://github.com/Mouseww/anything-analyzer/releases

它更适合处理这种问题：请求很多、来源很杂、签名和加密逻辑藏在前端或客户端里，人工翻包很慢。传统代理工具把包抓下来就结束了，`Anything Analyzer` 会继续往后走一步，把请求、Hook、Cookie、localStorage、sessionStorage 和 AI 分析报告串在一起。

## 技术原理

### 抓包入口分成两条线

`Anything Analyzer` 的抓包不是只靠浏览器，也不是只靠系统代理。它把两条线合在一个会话里：

| 流量来源 | 入口 | 适合场景 |
| --- | --- | --- |
| 内嵌浏览器 | Chrome DevTools Protocol / Fetch 拦截 | 网站 API、OAuth 登录、前端加密 |
| 外部程序 | 内置 MITM HTTPS 代理，默认端口 `8888` | 桌面应用、curl、脚本、手机 App |
| JS 调用 | 预注入 Hook 脚本 | fetch、XHR、crypto.subtle、CryptoJS、SM2/3/4 |
| 浏览器存储 | 定时快照 | Cookie、localStorage、sessionStorage |

对应到源码里，`src/main/cdp/cdp-manager.ts` 负责 CDP 拦截，`src/main/proxy/mitm-proxy-server.ts` 负责 MITM 代理，`src/preload/hook-script.ts` 负责 JS Hook。最后这些数据都会进入 `CaptureEngine`，再写入本地 SQLite。

这套结构的好处很实在。浏览器里触发的请求、手机代理过来的请求、脚本里跑出来的请求，不会散在不同工具里。它们会统一进入同一个 Session，后面 AI 分析时可以一起看。

### AI 分析

项目 README 里提到“两阶段分析”，源码里能看到具体阈值。

当请求数达到 `20` 条以上，且分析目的不是性能分析时，`AiAnalyzer` 会先进入 Phase 1。第一阶段只看请求摘要，选出更相关的请求；第二阶段再做深度分析。

| 阶段 | 做什么 | 源码里的关键点 |
| --- | --- | --- |
| Phase 1 | 过滤噪声请求 | `PRE_FILTER_THRESHOLD = 20` |
| Phase 2 | 深度生成报告 | 用过滤后的请求、Hook、存储快照构建 Prompt |
| 手动多选 | 跳过 Phase 1 | 只分析用户勾选的请求 |
| 性能分析 | 跳过过滤 | `SKIP_FILTER_PURPOSES = ["performance"]` |

这里的设计挺贴近日常逆向。真实页面经常混着静态资源、埋点、预检请求、WebSocket、SSE 和业务 API。直接把所有内容塞给模型，既浪费 token，也容易让报告跑偏。

更细一点看，Phase 1 如果选出的请求少于 `3` 条，会回退到全量分析。也就是说，它没有把过滤结果当成绝对判断，而是留了一个兜底。

### JS Hook 补的是“包里看不全”的部分

很多 API 逆向难点不在请求本身，而在请求发出去之前。

比如：

1. 参数在哪里拼出来。
2. token 从哪里读出来。
3. 签名函数怎么调用。
4. 请求体加密前是什么样。

`src/preload/hook-script.ts` 里做了不少拦截：`window.fetch`、`XMLHttpRequest`、`crypto.subtle`、`CryptoJS`、`JSEncrypt`、`forge`、`btoa/atob`、`document.cookie`。这些 Hook 记录会和网络请求一起进入分析上下文。

这就是它和普通代理工具的差别。代理看的是“已经发出去的包”，Hook 看的是“包发出去之前发生了什么”。两者放在一起，AI 才更容易还原签名、加密和鉴权流程。

### MCP 把抓包能力交给 Agent

`Anything Analyzer` 还有一层比较新的设计：MCP。

它既能作为 MCP Client 接外部 Server，也能启动内置 MCP Server，把会话、请求、Hook、报告这些能力暴露给 Claude Desktop、Cursor 等 Agent 工具。

| 角色 | 作用 | 典型用法 |
| --- | --- | --- |
| MCP Client | 让分析过程调用外部工具 | 扩展 AI 分析能力 |
| 内置 MCP Server | 把抓包数据暴露出去 | 让 IDE / Agent 读取 Session 和请求详情 |
| 内置请求详情工具 | AI 可按序号拉完整请求 | Phase 1 过滤后仍可回看被过滤请求 |

这点很适合做自动化工作流。比如你在 IDE 里让 Agent 分析一个登录流程，Agent 不需要你手动导出 JSON，再复制到聊天框里。它可以通过 MCP 去拿当前会话的数据。

## 快速上手

### 1. 下载并安装

从 Releases 页面下载对应平台安装包：

```
Windows: Anything-Analyzer-Setup-x.x.x.exe  
macOS Apple Silicon: Anything-Analyzer-x.x.x-arm64.dmg  
macOS Intel: Anything-Analyzer-x.x.x-x64.dmg  
Linux: Anything-Analyzer-x.x.x.AppImage
```

下载地址： https://github.com/Mouseww/anything-analyzer/releases

截至 2026 年 5 月 6 日，GitHub 最新 release API 返回的是 `v3.6.6`，仓库内 `package.json` 和 `RELEASE_NOTES.md` 已经写到 `3.6.7`。实际安装时以 Releases 页面展示为准。

### 2. 配置 LLM

打开 Settings → LLM，填入模型服务配置。

README 写到它支持 OpenAI、Anthropic，以及任何兼容 OpenAI API 的服务。源码里 LLM 侧由 `src/main/ai/llm-router.ts` 统一处理。

```
Provider: OpenAI / Anthropic / Custom  
API Base URL: 按你的模型服务填写  
API Key: 自己的 Key  
Model: 选择要用于分析的模型
```

### 3. 抓网页流量

新建 Session，填入目标 URL，然后在内嵌浏览器里操作页面。

```
新建 Session  
输入目标 URL  
点击 Start Capture  
正常操作网站  
点击 Stop  
切到 Inspector 或 Report
```

这种方式适合网站 API、OAuth 跳转、前端加密和登录流程。因为浏览器侧同时有 CDP、JS Hook 和存储快照，分析上下文会比较完整。

### 4. 抓外部应用、终端和手机

先在 Settings → MITM 代理里安装 CA 证书，再启用代理。默认端口是 `8888`。

终端命令可以这样走代理：

```
curl -x http://127.0.0.1:8888 https://api.example.com/data
```

Python 脚本可以这样配置：

```
import requests  
  
proxies = {  
    "http": "http://127.0.0.1:8888",  
    "https": "http://127.0.0.1:8888",  
}  
  
requests.get("https://api.example.com/data", proxies=proxies)
```

手机 App 则在 Wi-Fi 里设置 HTTP 代理，填电脑 IP 和端口 `8888`。`RELEASE_NOTES.md` 里提到，移动端证书下载页支持 `http://cert.anything.test` 和 `http://cert.anything.local`。

## 使用场景

### 1. 网站 API 逆向

**任务示例**：分析一个登录后才能访问的管理后台，整理 API 端点、鉴权头、分页参数和关键业务请求。

**技术要点**：用内嵌浏览器跑完整操作路径。CDP 抓请求，JS Hook 看签名和加密调用，Storage 快照补 Cookie 和 localStorage。

### 2. App 协议分析

**任务示例**：手机连同一局域网，Wi-Fi 代理指到电脑，抓一个 App 的 HTTP/HTTPS 请求。

**技术要点**：先安装 CA 证书，再看 App 是否做证书绑定。工具适合授权测试和自有 App 调试，不适合绕过第三方风控。

### 3. JS 加密逆向

**任务示例**：定位一个请求体加密流程，确认参数是在哪里拼接、签名函数调用了哪些输入。

**技术要点**：重点看 Hook 记录。单看最终请求，很多时候只会看到密文；Hook 能补上调用栈、函数名、参数和返回值。

### 4. Agent 自动分析流量

**任务示例**：让 Cursor 或 Claude Desktop 通过 MCP 读取抓包会话，继续追问某个请求为什么重要。

**技术要点**：内置 MCP Server 默认配置里能看到端口 `23816`。开启后，外部 Agent 可以围绕当前会话做持续分析，不用来回导出文件。

往期推荐 📚

[让 JADX 接上 AI](https://mp.weixin.qq.com/s?__biz=MzY5ODAyOTAwMg==&mid=2247484837&idx=1&sn=499b084201ef757ec85728e7f157bca2&scene=21#wechat_redirect)

[taste-skill：AI前端审美外挂](https://mp.weixin.qq.com/s?__biz=MzY5ODAyOTAwMg==&mid=2247484810&idx=1&sn=26bdcc11d8d2221d3d6d7694a78d2118&scene=21#wechat_redirect)

[从 OpenClaw 到 Hermes](https://mp.weixin.qq.com/s?__biz=MzY5ODAyOTAwMg==&mid=2247484743&idx=1&sn=5fe9c0d969fadf792b8e1c0bf678218a&scene=21#wechat_redirect)

欢迎关注“攻防录”✨