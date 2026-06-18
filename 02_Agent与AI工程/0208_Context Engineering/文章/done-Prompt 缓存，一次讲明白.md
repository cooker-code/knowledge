> 已吸收至：[[02_Agent与AI工程/0208_Context Engineering/0208_核心知识点/上下文工程治理与边界|上下文工程治理与边界]]
---
title: Prompt 缓存，一次讲明白
author: 大迁世界
date: devdev
url: https://mp.weixin.qq.com/s?__biz=MzI0NDQ0ODU3MA==&mid=2247547625&idx=1&sn=7502d1617286fa8fc3491e8e3a18c4b1&chksm=e882db79c801bfe9cd5556bf62e5fb7ed1f7e3567191e4d3f4d3b4ec0bd10d2bb38ceb8d992d&mpshare=1&scene=24&srcid=0521wzsqJKwJjjcGdN8NFQbV&sharer_shareinfo=60f942031577fae38512aea340d830ed&sharer_shareinfo_first=60f942031577fae38512aea340d830ed#rd
---

每当一个 AI Agent 往前走一步，它其实都在交一笔税。

它会重新读取所有内容。

系统提示词。 工具定义。 项目上下文。 三轮前已经加载过的内容。

每一轮都重新读一遍。

这就是 **context tax**。

对长时间运行的 Agent 工作流来说，它往往是整个 AI 基础设施里最贵的一项开销。

算一下就很直观：一个 20,000 token 的 system prompt，如果跑 50 轮，就等于 100 万 token 的重复计算。你为它付了钱，但它没有创造任何新价值。

解决办法，就是 **prompt caching**。

但想真正用好它，你得先知道底层到底发生了什么。

## 先分清：什么会变，什么不会变

优化 prompt caching 之前，先要看懂一个 Agent 的上下文结构。

每次请求里，其实都有两部分。

第一部分是 **static prefix**。

它包括系统指令、工具定义、项目上下文、行为规范。这些内容在同一个 session 里基本不变。

第二部分是 **dynamic tail**。

它包括用户消息、工具输出、终端观察结果。这部分每次请求都不同，并且会随着对话推进不断增长。

这个区别非常关键。

真正被反复浪费计算的，是 static prefix。 真正需要重新计算的，只有 dynamic tail。

Prompt caching 的作用，就是把 static prefix 的数学状态缓存下来。后续请求如果前缀一样，就不用再重新计算，直接从缓存读取。

你只需要为这部分前缀处理付一次钱。

后面每一轮，都相当于从内存里读。

## 为什么它有效？

要理解 prompt caching，得先知道模型读 prompt 时做了什么。

一次 LLM 推理大致有两个阶段。

分享一个正版GPT5.5 目前 0.2 倍率,  https://api.aidazhi.com/，注册后把用户名发到私信里面，即可帮领 5 美元测试额度。

## 阶段1：Prefill

模型会处理完整输入 prompt。

这是最慢、最贵的部分。

因为它要对上下文里的每个 token 做大量矩阵计算，读取所有内容，并建立内部表示。

这一步是 compute-bound，也就是主要受计算量限制。

## 阶段2：Decode

模型开始一个 token 一个 token 地生成输出。

这一步更偏 memory-bound。因为模型主要是在读取之前算好的状态，而不是反复做重计算。

在 prefill 阶段，Transformer 会为每个 token 生成三个向量：Query、Key、Value。

注意关键点：

Key 和 Value 只依赖它们之前的 token。

也就是说，只要某个前缀内容不变，它对应的 Key-Value 张量就不需要重新算。

没有缓存时，请求结束后，这些 Key-Value 张量会被扔掉。下一次请求来了，又要把同样的 20,000 token 从头算一遍。

KV caching 就是把这些张量存起来。

基础设施会根据输入文本的加密哈希来索引它们。下一次请求如果前缀相同，哈希匹配，就能直接取回张量，跳过重复计算。

这会把重复前缀带来的计算成本大幅压下去。

对一个 20,000 token 前缀、重复 50 轮的工作流来说，节省非常可观。

## 经济账怎么算？

Prompt caching 真正重要，是因为它直接改变成本结构。

以 Anthropic 的定价逻辑为例，有三点要记住：

Cache read 大约是基础输入价格的 10%，相当于缓存读取 token 打 1 折。 Cache write 比基础输入价格贵 25%，因为要存储 KV 张量。 1 小时扩展缓存大约是基础价格的 2 倍。

所以，缓存不是永远自动划算。

它成立的前提是：cache hit rate 要足够高。

这也是 Claude Code 的重点。

## Claude Code：30分钟会话怎么省钱？

Claude Code 的设计目标很清楚：

让缓存一直保持热的。

看一个典型 30 分钟编码 session。

## 第0分钟：Session 开始

Claude Code 会加载 system prompt、工具定义，还会读取项目根目录里的 `CLAUDE.md`，了解代码库和约定。

这部分经常超过 20,000 token。

这是整个 session 最贵的一刻，因为所有 token 都是新的。

但好消息是：这笔钱只付一次。

## 第1到5分钟：第一次指令

你输入：

“看一下 auth 模块，给我一些改进建议。”

Claude Code 会派出 Explore Subagent。它会浏览代码库、打开文件、执行 grep、理解相关代码。

这些新内容会被追加到 dynamic tail。

但那 20,000 token 的静态基础上下文，已经进缓存了。后续每轮都可以按缓存读取价格来算。

## 第6到15分钟：深入工作

Plan Subagent 拿到 Explore Subagent 的发现。

Claude Code 不会把原始结果全量塞过去，因为那会让 dynamic tail 暴涨。它会传递简洁总结，让后缀保持可控。

Planner 生成实施计划，你审核后批准，然后 Claude Code 开始修改代码。

这个循环里的每一轮，都会从缓存读取那 20,000 token 的前缀。

而每一次 cache hit，都会刷新 TTL，让缓存继续保持热状态。

## 第16到25分钟：迭代修改

你要求调整。Claude Code 修改方案。更多工具调用，更多终端输出。

dynamic tail 在增长，但它代表的是这个 session 里真正新增的内容。

此时，总处理 token 可能已经达到几十万。

但基础的 20,000 token，一直是在缓存里反复读取。

## 第28分钟：查看成本

如果没有缓存，这种 session 很容易超过 200 万 token。

按 Sonnet 4.5 价格，大概会到 6 美元左右。

有缓存后，大量 token 都以低价 cache read 计费，只有新的 dynamic tail 需要新计算。

实际中，单个任务能看到 80% 以上成本下降。

如果再乘以每天所有用户、所有 session，这就是巨大的基础设施成本差距。

## 最容易踩坑的规则

Prompt caching 最反直觉的地方是：

`1 + 2 = 3`。

但 `2 + 1` 是 cache miss。

为什么？

因为缓存匹配靠 prompt 的哈希。

只要顺序变了，哪怕内容一样，哈希也会变。哈希一变，缓存就对不上，整个前缀要重新计算。

所以要记住三条规则。

第一，不要在 session 中途增删工具。

工具定义属于缓存前缀。你改了工具，后面的缓存基本就废了。

第二，不要中途切换模型。

缓存是和模型绑定的。你换成更便宜的模型，也要重建整段缓存。

第三，不要通过修改 prefix 来改变状态。

Claude Code 的做法是，把状态提醒加到下一条用户消息里，而不是改系统前缀。这样 prefix 不变，缓存还能继续命中。

## 你自己做 Agent 时怎么用？

如果你在做自己的 Agent，结构可以这样安排：

最顶部放 system instructions 和规则。中途不要改。 提前加载所有需要用到的 tools，不要临时增删。 然后放检索到的上下文和文档，在 session 内尽量保持稳定。 底部放对话历史和工具输出。

开启 auto-caching 后，缓存断点会随着对话推进自动前移。

Anthropic 已经在 API 里加入 auto-caching，所以你也可以为自己的 Agent 使用类似方式。

没有 auto-caching 时，你需要自己记住 token 边界。边界错了，就吃不到缓存。

如果需要为了上下文限制做压缩，也要用 cache-safe forking。

也就是保持相同 system prompt、tools 和 conversation，然后把 compaction 作为一条新消息追加进去。

这样压缩请求看起来几乎和上一轮一样，缓存前缀还能继续复用。真正按新 token 计费的，只有那条压缩指令。

## 怎么判断缓存有没有生效？

看 API 响应里的三个字段：

`cache_creation_input_tokens`：写入缓存的 token。`cache_read_input_tokens`：从缓存读取的 token。`input_tokens`：正常处理的输入 token。

你的缓存效率，可以看 read tokens 和 creation tokens 的比例。

这个指标应该像 uptime 一样持续监控。

因为它直接影响成本。

## 关键结论

Prompt caching 不是一个“打开就完事”的功能。

它是一种架构纪律。

Claude Code 是一个很好的例子：通过让前缀稳定、工具稳定、上下文结构稳定，它能把 cache hit rate 做到 92%，成本降低 81%。

如果你在做 Agent，这就是蓝图。

Context tax 一定存在。

区别只在于：

你是一直为它付钱，还是从架构上把它消掉。

**最后：**

**[精通 React 面试：从零到中高级(针对面试回答)](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NDQ0ODU3MA==&action=getalbum&album_id=4438329314299920385&from_itemidx=1&from_msgid=2247546427&sessionid=#wechat_redirect)**

**[CSS终极指南](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NDQ0ODU3MA==&action=getalbum&album_id=4274694215210369041&from_itemidx=1&from_msgid=2247538974&sessionid=#wechat_redirect)**

**[Vue 设计模式实战指南](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NDQ0ODU3MA==&action=getalbum&album_id=4198498164833402888&from_itemidx=1&from_msgid=2247535710#wechat_redirect)**

[20个前端开发者必备的响应式布局](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NDQ0ODU3MA==&action=getalbum&album_id=4143266676156547085&from_itemidx=1&from_msgid=2247533575&sessionid=#wechat_redirect)

**[深入React:从基础到最佳实践完整攻略](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NDQ0ODU3MA==&action=getalbum&album_id=4062982567140671496&from_itemidx=1&from_msgid=2247531462#wechat_redirect)**

**[python 技巧精讲](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NDQ0ODU3MA==&action=getalbum&album_id=3973477600504201220&from_itemidx=1&from_msgid=2247529168&sessionid=#wechat_redirect)**

**[React Hook 深入浅出](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NDQ0ODU3MA==&action=getalbum&album_id=3660558698339991553&from_itemidx=1&from_msgid=2247523919&scene=173&subscene=91&sessionid=1728003498&enterid=1728004916&count=3&nolastread=1#wechat_redirect)**

**[CSS技巧与案例详解](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&__biz=MzI0NDQ0ODU3MA==&scene=2&album_id=3545535769677725703&count=3&uin=&key=&devicetype=iMac+MacBookPro18%2C3+OSX+OSX+14.3+build(23D56)&version=13080812&lang=zh_CN&nettype=WIFI&ascene=2&fontScale=100)**

**[vue2与vue3技巧合集](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&__biz=MzI0NDQ0ODU3MA==&scene=1&album_id=2509459125236416515&count=3#wechat_redirect)**