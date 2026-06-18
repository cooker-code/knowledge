> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/Claude Code上下文压缩与会话治理|Claude Code上下文压缩与会话治理]]、[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_知识地图|020502_知识地图]]、[[02_Agent与AI工程/0208_Context Engineering/0208_核心知识点/上下文工程治理与边界|上下文工程治理与边界]]
---
title: Claude Code 怎么压上下文？3 个机制，直接抄作业
author: 码农知道的事
date:
url: https://mp.weixin.qq.com/s?__biz=MzIxNzY5NDk5MA==&mid=2247485200&idx=1&sn=bd487cbea73cf6d23d0d5d571295cf66&chksm=9691f57866ecba592eff9f408af38fa3d8da88b7a798a15a47eaed38898ac021c023cf7990c9&mpshare=1&scene=24&srcid=0424M3krS54RO1kMHEZhIzmD&sharer_shareinfo=a74870d4ed92376f2fd2ef8f6a80cf73&sharer_shareinfo_first=a74870d4ed92376f2fd2ef8f6a80cf73#rd
---

同一份上下文，越聊越便宜。

你有没有这种时刻：明明只是“再读一下那个文件”“再跑一次命令”，结果上下文像雪球一样越滚越大，最后不是模型跑不动，就是账单跑不动。

Claude Code 这套实现很“工程脑”：它不试图用一个超级摘要解决一切，而是把上下文膨胀拆成三类问题，分别处理。

一句话结论：**先把“会爆炸的大块”变成“磁盘引用 + 小预览”，再清掉历史的工具噪音，最后才在快溢出时做摘要。**

下面按源码把它拆开讲，带你定位到关键文件与关键常量。

## 先看整体：三层治理，按顺序串起来

这三层分别对应：I/O 层、消息层、会话层。

1. **I/O 层：文件读取去重**

   （文件没变，就别再把全文塞进上下文）
2. **消息层：工具结果过大落盘**

   （只发预览 + 引用路径；并限制一条消息里总的 tool\_result 体积）
3. **会话层：micro-compact / auto-compact / session-memory**

   （旧工具结果先清理，真的逼近上限才摘要）

在 `query.ts` 里，顺序非常关键：

* 先 `applyToolResultBudget()`：把大 tool\_result 变成“引用 + 预览”
* 再 `snip`、`microcompact`：减少必须重写的历史体积（尤其缓存冷掉的时候）
* 之后再考虑 `autoCompact`：真的快爆了再上“摘要化”

这就像整理房间：先把大件收进柜子（落盘引用），再把桌面上的碎屑扫掉（micro-compact），最后才决定要不要把整屋打包搬家（auto-compact）。

## 1）文件读取去重：文件没变，就别“重复喂”

核心目标：减少重复 Read 导致的 cache\_creation 浪费。

### 1.1 去重怎么判？

代码位置：`tools/FileReadTool/FileReadTool.ts`

判定条件很“硬”：

* **同一个文件路径**
* **同一个读取范围**

  （offset/limit 一致）
* **磁盘 mtime 没变**
* 且必须是“真的 Read 过”的缓存（用 `offset !== undefined` 标识）

命中后不会回传正文，而是返回一个 stub：

`tools/FileReadTool/prompt.ts`

```
export const FILE_UNCHANGED_STUB =
  'File unchanged since last read. ... refer to that instead of re-reading.'
```

这一步的直觉很朴素：**你已经把全文放进上下文了，就别放第二份。**

### 1.2 为什么只对 Read 去重，不对 Edit/Write？

因为 Edit/Write 会让文件内容变成“新世界”。如果把“编辑后版本”的 readFileState 拿来去重，就可能把模型指回旧内容。

所以 FileReadTool 的实现里明确写了：只对 `offset !== undefined`（由 Read 写入的条目）做去重。

### 1.3 还有一层：底层 I/O 缓存

除了“少喂上下文”，Claude Code 还做了“少读磁盘”：

* `utils/fileReadCache.ts`

  ：按 mtime 失效的内存缓存
* `utils/file.ts`

  ：`readFileSyncCached()` 走这个缓存

```
if (cached && cached.mtime === stats.mtimeMs) {
  return cached
}
```

它解决的是性能问题（减少重复 I/O），而 Read 去重解决的是成本问题（减少重复上下文）。这两个层面互不冲突。

金句落点：**I/O 优化省时间，Read 去重省上下文，两者都要。**

## 2）工具结果过大：落盘，只留“预览 + 引用”

这一段是 Claude Code 里最值得抄作业的部分。

因为它不是简单 truncate，而是把“大块内容”变成“可引用资产”。

### 2.1 单个工具结果的落盘阈值

入口在 `utils/toolResultStorage.ts`：

* `processToolResultBlock()`

  / `processPreMappedToolResultBlock()`
* 里面统一走 `maybePersistLargeToolResult()` 判断是否落盘

阈值来自两部分：

1. 工具自身：`Tool.maxResultSizeChars`
2. 系统全局钳制：`constants/toolLimits.ts` 的 `DEFAULT_MAX_RESULT_SIZE_CHARS = 50_000`

也就是说：哪怕某个工具说自己能输出 100K，也会被系统强行压到 50K 的“落盘门槛”。

### 2.2 落盘后，模型到底看到了什么？

它会把全文写到会话目录的 `tool-results/`，再生成一个给模型看的包装消息：

```
<persisted-output>
Output too large (N KB). Full output saved to: <path>

Preview (first 2.0KB):
<preview...>
...
</persisted-output>
```

预览大小是固定的：`PREVIEW_SIZE_BYTES = 2000`，并且优先在换行处截断，避免半截句子把人逼疯。

### 2.3 还有更狠的：单条消息聚合预算

真实事故往往不是“一个工具输出 100K”，而是：

> 你并行跑了 8 个工具，每个输出 30K。
> 单个都不过线，但总和 240K，照样炸。

所以 Claude Code 又加了一层：**按一条 user message 的 tool\_result 总量做预算**。

常量：`MAX_TOOL_RESULTS_PER_MESSAGE_CHARS = 200_000`

实现：`utils/toolResultStorage.ts` 的 `enforceToolResultBudget()`

它会从“本轮新增 tool\_result”里挑最大的几个落盘，直到预算回到 200K 以内。

关键点：它不是每轮重新挑，而是用 `ContentReplacementState` 把“tool\_use\_id → replacement 内容”记录下来，确保多轮请求前缀稳定，尽量命中 prompt cache。

金句落点：**预算控制不是为了省内存，是为了让缓存命中变得可控。**

## 3）长上下文：先清噪音，再摘要

Claude Code 对“历史越来越长”不是上来就总结，而是先把最容易膨胀的部分拿掉：旧工具结果。

### 3.1 micro-compact：只清工具结果（优先不破坏缓存）

文件：`services/compact/microCompact.ts`

它只针对 `COMPACTABLE_TOOLS`（Read/Shell/Grep/Glob/WebFetch/WebSearch/Edit/Write 等）。

两条路：

* **cached micro-compact**

  ：通过 cache edits 在 API 层删除旧 tool\_result，不改本地消息内容 → 更利于缓存
* **time-based micro-compact**

  ：当缓存已经冷了（隔太久），直接本地清掉旧 tool\_result 内容

### 3.2 auto-compact：真的要爆了才做摘要

文件：`services/compact/autoCompact.ts`、`services/compact/compact.ts`

触发逻辑是“估算 token 使用 >= 阈值”，阈值会留 buffer（例如 13K tokens）防止贴边。

摘要生成过程中，优先走 forked agent 共享主会话的 prompt cache（`streamCompactSummary()` → `runForkedAgent()`）。失败才 fallback 到常规流式请求。

### 3.3 session-memory compact：摘要也要防膨胀

文件：`services/compact/sessionMemoryCompact.ts`

如果 session memory 很长，它也会做截断，避免出现“摘要占满摘要预算”的尴尬。

## 一张表把关键常量捋清楚

| 类别 | 常量/字段 | 默认值 | 作用 |
| --- | --- | --- | --- |
| 单工具落盘门槛 | `DEFAULT_MAX_RESULT_SIZE_CHARS` | 50,000 | 超过则写磁盘 + 只回传预览 |
| 单条消息预算 | `MAX_TOOL_RESULTS_PER_MESSAGE_CHARS` | 200,000 | 一轮并行工具结果的总量上限 |
| 预览大小 | `PREVIEW_SIZE_BYTES` | 2000 bytes | 落盘后给模型的首段预览 |
| Read 特例 | `maxResultSizeChars = Infinity` | ∞ | Read 永不落盘（避免循环） |

## 最后：抄作业建议（按收益排序）

1. **先做“落盘 + 预览 + 引用”**

   ：这一步最立竿见影，尤其是 bash/webfetch 这种容易喷大输出的工具。
2. **加“单条消息聚合预算”**

   ：不然并行工具会把你打穿。
3. **对 Read 做 mtime + range 去重**

   ：别让同一份文件在上下文里出现两次。
4. **最后才是摘要**

   ：摘要是“不得已”，不是“默认方案”。

结尾 CTA：你更想抄哪一段？是“Read 去重”，还是“工具结果落盘”？评论区告诉我，我可以把对应实现再画一张更细的流程图。