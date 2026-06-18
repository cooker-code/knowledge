> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/ChromeDevToolsMCP能力边界|ChromeDevToolsMCP能力边界]]
---
title: 🪢 [MCP] Chrome DevTools MCP 原理分析
author: 卤代烃实验室
date:
url: https://mp.weixin.qq.com/s?__biz=MzI4OTU0NTU1NA==&mid=2247485375&idx=1&sn=b6918dd4328adf9fb605a299dca7063f&chksm=edb720fdc4728257a2ba268de246884201633a1e814c745b13f1b6d1d3f9c1d06cda7e67522a&mpshare=1&scene=24&srcid=1127nfTRth0e7tzbrWtxOGLl&sharer_shareinfo=992e47aaa9d65ffaea275278126ebedf&sharer_shareinfo_first=992e47aaa9d65ffaea275278126ebedf#rd
---

2025.09.23 Google 官方发布了 Chrome DevTools MCP[1]，目前版本号还是 0.9.0，Google 官方对它的定位是协助 AI coding assistants 做调试和性能测试，提高网页编程的 debug 能力，一些使用案例可见官方 blog：Chrome DevTools (MCP) for your AI agent[2]。

本文主要是结合 源码 和 MCP output 内容，分析 chrome-devtools-mcp 的运行原理，探索它的能力边界。



## Tools 分类

从它的 MCP Tools[3] 可以看出，它的 tools 可以分为以下类别：

* **基础能力**：这部分能力主要是做网页操作，这个是所有的 Browser MCP 的基石，用户也可以不用任何网页调试的能力，只用这部分基础能力做网页自动化操作

+ **Navigation**：就是 createPage，closePage 等
+ **Input**：就是 page 内的 click，hover 等操作

* **调试能力**：这个其实是 chrome-devtools-mcp 主要定位的地方，提供基础的调试信息透出

+ **Emulation**：目前的功能还是比较弱，主要是弱网模拟，CPU 降速模拟，尺寸模拟，其实对应的是这几个功能:
+ **Performance**：用来收集性能 trace 数据，对标如下的功能：
+ **Network**：获取当前 page 的所有网络信息
+ **Debugging**：获取 console 信息，截图，还可以注入一些代码

## 技术细节

chrome-devtools-mcp 底层使用的还是 **puppeteer**，所以很多功能直接可以对标到 puppeteer 的操作。我们直接分析 Tools 的一些技术细节。

### Navigation

Navigation 这块儿包装的内容很少。

**首先是 Tabs 管理。**

其实就是用了 `browser.pages()` 这个 API，然后对每个 tab 做一个基础的编号，然后利用 `bringToFront()` 做 tab 的激活，被激活的 tab 会有一个 `[selected]` 的标记：

```
const parts = [`## Pages`];
let idx = 0;
for (const page of context.getPages()) {
  parts.push(
    `${idx}: ${page.url()}${idx === context.getSelectedPageIdx() ? ' [selected]' : ''}`,
  );
  idx++;
}
```

最后给模型的 response 结构如下：

```
## Pages
0: https://developer.chrome.com/?hl=zh-cn
1: https://www.baidu.com/ [selected]
```

chrome-devtools-mcp 这里封装非常浅，其实很多错误处理，超时处理都没有做。



**然后是页面导航内容。**

比如说 `new_page`，`navigate_page`，`close_page` 等，其实都比较简单：

```
// new_page
const page = await context.newPage();
await page.goto(request.params.url, {
timeout: request.params.timeout,
});

// navigate_page
const page = context.getSelectedPage();
await page.goto(request.params.url, {
timeout: request.params.timeout,
});

// close_page
await context.closePage(request.params.pageIdx);
```

这里的封装也是比较浅的，很多健壮性工作都没有做。

### Input

chrome-devtools-mcp 目前还是基于 DOM 的 snapshot 去做 click/drag/hover 等操作的。这里以百度首页为例，从一个简单的 click 场景分析一下它的运行流程。

1. 首先，我们通过 pptr 的可访问性 API 拿到 DOM 的 JSON 树：

```
const rootNode = await page.accessibility.snapshot({
 includeIframes: true,
});
```

2. 然后，保存 JSON 树的每个节点，对每个节点做好编号方便后续格式化：

```
let idCounter = 0;
const idToNode = new Map<string, TextSnapshotNode>();
const assignIds = (node: SerializedAXNode): TextSnapshotNode => {
 const nodeWithId: TextSnapshotNode = {
   ...node,
   id: `${snapshotId}_${idCounter++}`, // 编号
   children: node.children
     ? node.children.map(child => assignIds(child)) // 递归处理 dom 树节点
     : [],
 };
 idToNode.set(nodeWithId.id, nodeWithId);
 return nodeWithId;
};
```

| 处理前的 snapshot | 处理后的 snapshot |
| --- | --- |
|  |  |

3. 最后，执行相关 action 操作，比如说我要 click「新闻」这个按钮，会定位到 `uid=1_1 link "新闻"` 这条数据，然后参数传入 `1_1`，执行如下的代码：

```
const node = this.#textSnapshot?.idToNode.get(uid); // uid="1_1"
// pptr 可以根据 node.elementHandle 拿到相关 dom 的 handle
const handle = await node.elementHandle(); 

await handle.asLocator().click({ count: 1 });
```



其余 action 操作都大差不差，都是 **拿到 snapshot** -> **分析意图拿到 uid** -> **通过 uid 拿到 DOM 执行 action** 的操作。

这一套方案的优势是可以直接通过 pptr 的 accessibility API 拿到 DOM 的信息（而不是 browser-use 注入 JS 代码然后做自定义格式化），封装更浅更通用；劣势是可能最终 snapshot 会构建的比较大，可能会占用不少上下文。

### Emulation

这个内容比较简单，就是通过 pptr 的 API 调用用来模拟慢速网络，慢速 CPU，调整网页大小。只是把 API MCP 化了，不涉及很多的逻辑和上下文处理，所以略过。

### Performance

这个是 MCP 主打的一个功能，做性能度量和性能测试。我们先看一下它是怎么做的。



目前 MCP 的实现很简单，其实就是通过 pptr 提供的 API，对相关网页做性能测试的 start/stop 两个操作：

但是它不会把完整的 trace 交给模型分析（完整的 trace 数据量很大，可能是 10～100 MB的 json 数据，交给模型分析也不现实），而是输出为一份**性能报告模板**：

```
## Summary of Performance trace findings:
URL: https://developer.chrome.com/?hl=zh-cn
Bounds: {min: 771625656677, max: 771629299667}
CPU throttling: none
Network throttling: none
Metrics (lab / observed):
  - LCP: 907 ms, event: (eventKey: r-6881, ts: 771626610159), nodeId: 170
  - LCP breakdown:
    - TTFB: 808 ms, bounds: {min: 771625702955, max: 771626510489}
    - Render delay: 100 ms, bounds: {min: 771626510489, max: 771626610159}
  - CLS: 0.03, event: (eventKey: s--1, ts: 771626595695)
Metrics (field / real users): n/a – no data for this page in CrUX
Available insights:
  - insight name: DocumentLatency
    description: Your first network request is the most important.  Reduce its latency by avoiding redirects, ensuring a fast server response, and enabling text compression.
    relevant trace bounds: {min: 294, max: 771626579945}
    estimated metric savings: FCP 706 ms, LCP 706 ms
    example question: How do I decrease the initial loading time of my page?
    example question: Did anything slow down the request for this document?
  - insight name: RenderBlocking
    description: Requests are blocking the page's initial render, which may delay LCP. [Deferring or inlining](https://web.dev/learn/performance/understanding-the-critical-path#render-blocking_resources "Deferring or inlining") can move these network requests out of the critical path.
    relevant trace bounds: {min: 771626513396, max: 771626531288}
    estimated metric savings: FCP 90 ms, LCP 90 ms
    example question: Show me the most impactful render blocking requests that I should focus on
    example question: How can I reduce the number of render blocking requests?
  - insight name: LCPBreakdown
    description: Each [subpart has specific improvement strategies](https://web.dev/articles/optimize-lcp#lcp-breakdown "subpart has specific improvement strategies"). Ideally, most of the LCP time should be spent on loading the resources, not within delays.
    relevant trace bounds: {min: 771625702955, max: 771626610159}
    example question: Help me optimize my LCP score
    example question: Which LCP phase was most problematic?
    example question: What can I do to reduce the LCP time for this page load?
  - insight name: CLSCulprits
    description: Layout shifts occur when elements move absent any user interaction. [Investigate the causes of layout shifts](https://web.dev/articles/optimize-cls "Investigate the causes of layout shifts"), such as elements being added, removed, or their fonts changing as the page loads.
    relevant trace bounds: {min: 771626595695, max: 771628821131}
    example question: Help me optimize my CLS score
    example question: How can I prevent layout shifts on this page?
  - insight name: NetworkDependencyTree
    description: [Avoid chaining critical requests](https://developer.chrome.com/docs/lighthouse/performance/critical-request-chains "Avoid chaining critical requests") by reducing the length of chains, reducing the download size of resources, or deferring the download of unnecessary resources to improve page load.
    relevant trace bounds: {min: 771625703654, max: 771626579945}
    example question: How do I optimize my network dependency tree?
  - insight name: ThirdParties
    description: 3rd party code can significantly impact load performance. [Reduce and defer loading of 3rd party code](https://web.dev/articles/optimizing-content-efficiency-loading-third-party-javascript/ "Reduce and defer loading of 3rd party code") to prioritize your page's content.
    relevant trace bounds: {min: 771626513396, max: 771628820176}
    example question: Which third parties are having the largest impact on my page performance?

## Details on call tree & network request formats:
Information on performance traces may contain main thread activity represented as call frames and network requests.

Each call frame is presented in the following format:

'id;eventKey;name;duration;selfTime;urlIndex;childRange;[S]'

Key definitions:

* id: A unique numerical identifier for the call frame. Never mention this id in the output to the user.
* eventKey: String that uniquely identifies this event in the flame chart.
* name: A concise string describing the call frame (e.g., 'Evaluate Script', 'render', 'fetchData').
* duration: The total execution time of the call frame, including its children.
* selfTime: The time spent directly within the call frame, excluding its children's execution.
* urlIndex: Index referencing the \"All URLs\" list. Empty if no specific script URL is associated.
* childRange: Specifies the direct children of this node using their IDs. If empty ('' or 'S' at the end), the node has no children. If a single number (e.g., '4'), the node has one child with that ID. If in the format 'firstId-lastId' (e.g., '4-5'), it indicates a consecutive range of child IDs from 'firstId' to 'lastId', inclusive.
* S: _Optional_. The letter 'S' terminates the line if that call frame was selected by the user.

Example Call Tree:

1;r-123;main;500;100;;
2;r-124;update;200;50;;3
3;p-49575-15428179-2834-374;animate;150;20;0;4-5;S
4;p-49575-15428179-3505-1162;calculatePosition;80;80;;
5;p-49575-15428179-5391-2767;applyStyles;50;50;;

Network requests are formatted like this:
`urlIndex;eventKey;queuedTime;requestSentTime;downloadCompleteTime;processingCompleteTime;totalDuration;downloadDuration;mainThreadProcessingDuration;statusCode;mimeType;priority;initialPriority;finalPriority;renderBlocking;protocol;fromServiceWorker;initiators;redirects:[[redirectUrlIndex|startTime|duration]];responseHeaders:[header1Value|header2Value|...]`

- `urlIndex`: Numerical index for the request's URL, referencing the \"All URLs\" list.
- `eventKey`: String that uniquely identifies this request's trace event.
Timings (all in milliseconds, relative to navigation start):
- `queuedTime`: When the request was queued.
- `requestSentTime`: When the request was sent.
- `downloadCompleteTime`: When the download completed.
- `processingCompleteTime`: When main thread processing finished.
Durations (all in milliseconds):
- `totalDuration`: Total time from the request being queued until its main thread processing completed.
- `downloadDuration`: Time spent actively downloading the resource.
- `mainThreadProcessingDuration`: Time spent on the main thread after the download completed.
- `statusCode`: The HTTP status code of the response (e.g., 200, 404).
- `mimeType`: The MIME type of the resource (e.g., \"text/html\", \"application/javascript\").
- `priority`: The final network request priority (e.g., \"VeryHigh\", \"Low\").
- `initialPriority`: The initial network request priority.
- `finalPriority`: The final network request priority (redundant if `priority` is always final, but kept for clarity if `initialPriority` and `priority` differ).
- `renderBlocking`: 't' if the request was render-blocking, 'f' otherwise.
- `protocol`: The network protocol used (e.g., \"h2\", \"http/1.1\").
- `fromServiceWorker`: 't' if the request was served from a service worker, 'f' otherwise.
- `initiators`: A list (separated by ,) of URL indices for the initiator chain of this request. Listed in order starting from the root request to the request that directly loaded this one. This represents the network dependencies necessary to load this request. If there is no initiator, this is empty.
- `redirects`: A comma-separated list of redirects, enclosed in square brackets. Each redirect is formatted as
`[redirectUrlIndex|startTime|duration]`, where: `redirectUrlIndex`: Numerical index for the redirect's URL. `startTime`: The start time of the redirect in milliseconds, relative to navigation start. `duration`: The duration of the redirect in milliseconds.
- `responseHeaders`: A list (separated by '|') of values for specific, pre-defined response headers, enclosed in square brackets.
The order of headers corresponds to an internal fixed list. If a header is not present, its value will be empty.
```

从上面这个 Summary 模板可以看出主要是暴露了一些常用的性能数据，可以拿到一个较为完整的性能报告。



另外一个用法是通过 `performance_analyze_insight` API 拿到具体的指标内容，比如说我想看一下 LCP 的数据，它就会从 trace 数据中拿 LCP 的内容，其实内容也是**预设好的模板**：

```
# performance_analyze_insight response

## Insight Title: LCP breakdown

## Insight Summary:
This insight is used to analyze the time spent that contributed to the final LCP time and identify which of the 4 phases (or 2 if there was no LCP resource) are contributing most to the delay in rendering the LCP element.

## Detailed analysis:
The Largest Contentful Paint (LCP) time for this navigation was 4,412 ms.
The LCP element (IMG, nodeId: 6815) is an image fetched from https://contentcms-bj.cdn.bcebos.com/cmspic/dc39f2b27f6e7b4ce3d76a5019276cde.jpeg?x-bce-process=image/crop,x_0,y_0,w_665,h_362 (eventKey: s-4377, ts: 779075655743).
## LCP resource network request: https://contentcms-bj.cdn.bcebos.com/cmspic/dc39f2b27f6e7b4ce3d76a5019276cde.jpeg?x-bce-process=image/crop,x_0,y_0,w_665,h_362
eventKey: s-4377
Timings:
- Queued at: 457 ms
- Request sent at: 457 ms
- Download complete at: 457 ms
- Main thread processing completed at: 457 ms
Durations:
- Download time: 0.2 ms
- Main thread processing time: 0 μs
- Total duration: 0.2 ms
Initiator: https://mbdp02.bdstatic.com/pcnews/static/fisp_static/common/module_static_include/module_static_include_5d6af88.js
Redirects: no redirects
Status code: 200
MIME Type: image/jpeg
Protocol: h2
Priority: Low
Render blocking: No
From a service worker: No
Initiators (root request to the request that directly loaded this one): https://news.baidu.com/, https://mbdp02.bdstatic.com/pcnews/static/fisp_static/common/module_static_include/module_static_include_5d6af88.js
Response headers
- content-md5: <redacted>
- x-bce-flow-control-type: <redacted>
- x-bce-image-info: <redacted>
- age: 10675
- ohc-cache-hit: <redacted>
- expires: Fri, 17 Oct 2025 04:17:05 GMT
- date: Tue, 14 Oct 2025 07:16:54 GMT
- content-type: image/jpeg
- last-modified: Tue, 14 Oct 2025 04:17:04 GMT
- ohc-file-size: <redacted>
- x-cache-status: <redacted>
- x-bce-debug-id: <redacted>
- x-bce-request-id: <redacted>
- accept-ranges: bytes
- ohc-global-saved-time: <redacted>
- content-length: <redacted>
- x-bce-is-transition: <redacted>
- server: nginx
- x-bce-storage-class: <redacted>

We can break this time down into the 4 phases that combine to make the LCP time:

- Time to first byte: 207 ms (4.7% of total LCP time)
- Resource load delay: 250 ms (5.7% of total LCP time)
- Resource load duration: 0.2 ms (0.0% of total LCP time)
- Element render delay: 3,955 ms (89.6% of total LCP time)

## Estimated savings: none

## External resources:
- https://web.dev/articles/lcp
- https://web.dev/articles/optimize-lcp
```

值得注意的是，上面的模板内容其实都不在 MCP 代码里，而是直接复用的 devtools 里的代码：

综上所述，MCP 做性能测试时的流程如下：

* 先全量获取一份 trace 数据
* 然后通过全量的 trace 数据给一份 Summary 内容
* 如果想看具体的性能指标（比如说 LCP），可以直接指定相关指标拿到一份相关模板数据



从我个人角度看，chrome-devtools-mcp 的性能测试其实就是**预制菜**。目前它的能力上限就是 Lighthouse。

因为我做过较长时间的性能优化，所以对 devtool 这套 perfs 的使用还是比较有心得的。MCP 提供的这些性能优化，其实都是复用 web.dev[4] 中的性能优化建议（这也是 Google 官方的好处，可以复用相同的资产），如果开发中接入 MCP，可能会修复解决一些初级的性能优化问题。

如果想解决一些非常复杂的性能问题，目前 MCP 提供的能力还是远远不够的，可以这样理解，一个 100MB 的 trace 数据，现在的 MCP 只提供了几个切面（Summary/LCP/FCP 等），从几个切面是无法获取全局理解的。

### Network

提供了两个 API，`list_network_requests`，列出网络请求列表，还有一个 `get_network_request` 查看具体请求的细节信息。

其实 network 能力核心就 1 行代码，pptr page 监听并收集所有的网络数据：

```
page.on('request', request => collect(request));
```

拿到所有的网络数据了，那么 全量/筛选/细节 其实都能拿到，然后做格式化即可。



下面是百度首页的全量 network response summary 信息：

```
# list_network_requests response
## Network requests
Showing 1-63 of 63 (Page 1 of 1).
https://news.baidu.com/ GET [success - 200]
https://mbdp02.bdstatic.com/pcnews/static/fisp_static/common/resource/js/usermonitor_88a158c.js?v=1.2 GET [failed - 304]
https://gss0.bdstatic.com/5foIcy0a2gI2n2jgoY3K/static/fisp_static/wza/aria.js?appid=c890648bf4dd00d05eb9751dd0548c30 GET [success - 200]
https://mbdp02.bdstatic.com/pcnews/static/fisp_static/news/js/jquery-1.8.3.min_a6ffa58.js GET [failed - 304]
https://efe-h2.cdn.bcebos.com/cliresource/ubc-report-sdk/2.0.8/ubc-web-sdk.umd.min.js GET [success - 200]
https://mbdp02.bdstatic.com/pcnews/static/fisp_static/common/module_static_include/module_static_include_326f211.css GET [success - 200]
https://mbdp02.bdstatic.com/pcnews/static/fisp_static/news/focustop/focustop_2701266.css GET [success - 200]
https://mbdp02.bdstatic.com/pcnews/static/fisp_static/common/img/sidebar/newErweima_9fa03e0.png GET [failed - 304]
https://news-bos.cdn.bcebos.com/mvideo/log-news.png GET [success - 200]
https://mbdp02.bdstatic.com/pcnews/static/fisp_static/common/img/footer/newErweima_9fa03e0.png GET [failed - 304]
https://mbdp02.bdstatic.com/pcnews/static/fisp_static/common/lib/mod_b818356.js GET [failed - 304]
https://mbdp02.bdstatic.com/pcnews/static/fisp_static/news/focustop/focustop_b924ecb.js GET [success - 200]
```



下面是某个 HTTP 请求的 request/response 细节：

```
# get_network_request response
## Request https://mbdp02.bdstatic.com/pcnews/static/fisp_static/news/js/jquery-1.8.3.min_a6ffa58.js
Status:  [failed - 304]
### Request Headers
- sec-ch-ua-platform:\"macOS\"
- if-none-match:\"a6ffa586a4bd1dc6ac369de9a08c18ef\"
- referer:https://news.baidu.com/
- user-agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/141.0.0.0 Safari/537.36
- sec-ch-ua:\"Google Chrome\";v=\"141\", \"Not?A_Brand\";v=\"8\", \"Chromium\";v=\"141\"
- if-modified-since:Wed, 12 Mar 2025 06:44:26 GMT
- sec-ch-ua-mobile:?0
### Response Headers
- accept-ranges:bytes
- access-control-allow-origin:*
- age:165476
- connection:keep-alive
- content-md5:pv+lhqS9HcasNp3poIwY7w==
- content-type:text/javascript; charset=utf-8
- date:Tue, 14 Oct 2025 07:52:52 GMT
- etag:\"a6ffa586a4bd1dc6ac369de9a08c18ef\"
- expires:Wed, 15 Oct 2025 09:54:56 GMT
- last-modified:Wed, 12 Mar 2025 06:44:26 GMT
- ohc-cache-hit:wzix57 [2]
- ohc-file-size:93010
- ohc-global-saved-time:Sun, 12 Oct 2025 09:54:56 GMT
- server:JSP3/2.0.14
- x-cache-status:HIT
- x-bce-content-crc32:2237621194
- x-bce-debug-id:Rm9nn/ehbuTaGCz6KrvSy6cqmggWUatfykgluv61oqpFxkZl6/I9a7WaPOkKP9v5zCrujYrhMbyf9b2K0sYiUA==
- x-bce-flow-control-type:-1
- x-bce-is-transition:false
- x-bce-request-id:bf31960c-378d-46b8-a13d-95dbbb3741fa
- x-bce-storage-class:STANDARD
### Response Body
!function(e,t){function n(e){var t...
```



从上面两段 response 可以看出，这部分数据不复杂，也没有做更多的预制处理。基本就是把所有网络请求列出来，把 HTTP 的 Header/Body 打印出来。

### Debugging

Debugging 的内容比较杂，可以分类说一下。

#### evaluate\_script

首先是 `evaluate_script` 这个 tool，就是向 page 里注入一段 JS 代码做执行，然后拿到结果。

#### list\_console\_messages

另一个是 `list_console_messages`，核心两行代码，监听页面的 console 和 error 事件，然后全部收集起来：

```
page.on('console', event => collect(event));
page.on('pageerror', event => collect(event));
```

比如说这里我收集到的百度首页信息，从 devtools 控制台和 MCP 的 response 对比，可见内容是一致的：

```
# list_console_messages response
## Console messages
A parser-blocking, cross site (i.e. different eTLD+1) script, https://news-bos.cdn.bcebos.com/mvideo/pcconf_2019.js?1760428369873, is invoked via document.write. The network request for this script MAY be blocked by the browser in this or a future page load due to poor network connectivity. If blocked in this page load, it will be confirmed in a subsequent console message. See https://www.chromestatus.com/feature/5718547946799104 for more details.
A parser-blocking, cross site (i.e. different eTLD+1) script, https://news-bos.cdn.bcebos.com/mvideo/pcconf_2019.js?1760428369873, is invoked via document.write. The network request for this script MAY be blocked by the browser in this or a future page load due to poor network connectivity. If blocked in this page load, it will be confirmed in a subsequent console message. See https://www.chromestatus.com/feature/5718547946799104 for more details.
Error> Failed to load resource: the server responded with a status of 500 (Internal Server Error)
b.jpg?cmd=1&class=technnews&cy=0&0.4963333576628812:undefined:undefined
Error> Failed to load resource: the server responded with a status of 500 (Internal Server Error)
&refer_hostname=news.baidu.com&byInputUrl=1&ra=3078942552352606:undefined:undefined
```

#### take\_screenshot

就是对当前页面或某个 DOM 做截图。直接套用 pptr 的 API，没什么特别的。

## 总结

从「**基础能力**」的角度看，如果只是想用 chrome-devtools-mcp 去控制浏览器做一些自动化操作，它其实还是非常基础的，很多能力都是属于入门级别，只能说能用，和其他的 Browser MCP 相比谈不上有什么优势（优势可能是 Google 官方出品？）。但是这些能力如果只是为了方便开发者调试网页，可能也够用了。

从「**调试能力**」的角度看，chrome-devtools-mcp 可以复用 ChromeDevTools 团队的一些工作，在调试能力上可以保证长期维护的稳定性。但目前看调试能力提供的还是**比较少的**：

* 可以感知到 console 控制台的一些  warning/error log，然后反向去优化修改代码
* 可以感知一些 network 信息，可以做一些网络问题的分析
* 可以做一些基础的性能优化，目前看分析的上限就是 Lighthouse。但性能优化本身是一个很复杂的事情，目前的形态可能帮助不大



其余的基础调试能力其实都没有涉及，比如说 Element 调试，分析一下各种奇形怪状的 CSS，这个还是做不到的，即使是借助 screenshot，也拿不到真正的细节只能纯瞎蒙。

更高级的调试能力（比如说 Animations，Rendering 等能力）预期后续也很难加入，因为基础能力还没有打磨好，还需要大量的时间去完善和优化。

参考资料

[1] 

Chrome DevTools MCP: *https://github.com/ChromeDevTools/chrome-devtools-mcp*

[2] 

Chrome DevTools (MCP) for your AI agent: *https://developer.chrome.com/blog/chrome-devtools-mcp?hl=en*

[3] 

MCP Tools: *https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/tool-reference.md*

[4] 

web.dev: *https://web.dev*