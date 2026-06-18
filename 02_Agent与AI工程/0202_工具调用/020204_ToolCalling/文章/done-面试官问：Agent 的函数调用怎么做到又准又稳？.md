> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020204_ToolCalling/020204_核心知识点/工具调用治理与协议边界|工具调用治理与协议边界]]
---
title: 面试官问：Agent 的函数调用怎么做到又准又稳？
author: 吴师兄学大模型
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489115&idx=1&sn=4d5b70eb8cc5fd9e2f6de0039553f57a&chksm=c3624b65a48580833dd23ac55f606bb779a7e099bf54c96b314b913455427090deea22569de6&mpshare=1&scene=24&srcid=1125gEbhEQSwVwak03H2uBV2&sharer_shareinfo=0179d3f2a2a7a99588640beaa2050f9a&sharer_shareinfo_first=0179d3f2a2a7a99588640beaa2050f9a#rd
---

大家好，我是吴师兄。

最近面试里，一个问题出现得特别频繁：

> “你做的 Agent 怎么保证 Function Call 的准确率？工具多了会不会乱调？参数会不会填错？”

这个问题乍一看是“实现细节”，但实际上，面试官真正想听的是：**有没有完整的端到端体系，而不是停留在框架 demo 那种层面。**

很多人回答，“我们用了 OpenAI 的 function calling，让模型自己选工具”， 这种回答最多算 40 分。

真正能拿 90 分的，是能把这四件事讲明白：

1. 怎么量化正确率
2. 工具调用为什么会出错
3. 怎么系统化优化
4. 用项目里的案例说明“确实做过”

接下来这篇，就按这个思路来拆。

## unsetunset一、为什么 Agent 的函数调用，必须“工程化”？unsetunset

只要工具一多，模型就会出错，这是行业共识。

比如我们在训练营项目里做一个智能旅行助手： 有航班查询、酒店查询、订票、订酒店、天气查询、用户偏好等等六种工具。

如果不做限制，模型只要选错一个函数、填错一个参数、生成一个无效 JSON， 整个链路都能直接挂掉。

换句话说就是**Function Call 的准确率，就是 Agent 系统的生命线。**

这也是为什么我们在训练营里的几套 Agent 项目，基本都绕不开：

* 量化指标
* badcase 收敛
* Prompt 迭代
* 参数校验
* 路由机制
* 失败重试
* 记忆注入

手上没有这套体系的人，面试一定答不深。

## unsetunset二、工具调用为什么会出错？（四大原因）unsetunset

这里和 RAG 一样，得先从“本质问题”开始讲。

影响 Function Call 准确率的因素，主要分四类👇

### **① Schema 设计不清晰（最常见）**

函数名模糊、参数名相似、含糊不清。 比如：

* `queryWeather(city)`
* `getWeather(cityName)`

这俩一起给模型，它十有八九会混。

### **② Prompt 或上下文有歧义**

工具描述太长、系统说明不够明确、User 输入含糊。

比如“查一下明天的天气”，模型不知道是默认城市还是用户之前的城市。

### **③ 模型采样策略太高**

温度设 0.8，那就是让它“自由发挥”。

结果自然就是：

* 函数名拼错
* JSON 拼坏
* 参数缺失

### **④ 没有防御机制**

即使调用错了、JSON 格式不对、工具返回异常， 系统也没有：

* schema 校验
* retry
* reflection
* fallback

那就是“错一次，挂一次”。

## unsetunset三、准确率要提升？得有一套体系而不是几个技巧unsetunset

这部分要用“系统化能力”去回答，面试官最吃这一套。

在训练营的 Agent 项目里，我们总结出五件最重要的事情👇

### **① 动态函数路由（减少歧义，减少 token）**

继续用旅行助手举例。

用户问：

> “明天北京的天气怎么样？”

如果把 6 个工具都给模型：

* 航班查询？
* 订机票？
* 酒店？
* 订酒店？
* 用户信息？
* 天气？

很多模型就开始乱想了。

所以我们会做：

> “Query → 意图分类器 → tool\_subset → 再交给 LLM”

比如这个例子：

意图识别 → 「查天气」 那么工具子集就是：

```
[get_weather]
```

调用正确率直接提升一截。

### **② CoT + Plan-Execute（强制模型按步骤来）**

复杂任务不拆解＝必挂。

例子：

> “帮我订明天从上海到北京的机票，然后订机场附近的酒店。”

如果不做 CoT 推理，模型常见的坏行为：

* 先订酒店
* 忘记出发城市
* 日期填“明天后天”而不是具体日期
* 航班没选就直接预订

我们会让模型先生成 Plan：

```
1. search_flights()
2. 用户选择航班
3. book_flight()
4. search_hotels()
5. book_hotel()
```

再逐步执行，每一步都带观察值（Observation）。

错了就 ReAct 自我纠正。

这个机制，用在哪个 Agent 项目都好使。

### **③ 结果校验层（Result Validation Layer）**

LLM 幻觉无法完全消灭，但可以完全“封住”。

三类校验：

#### ✔ 参数是否完整？

缺失参数 → 反馈错误 → Retry

#### ✔ JSON 格式是否符合 schema？

字段类型不对 → 清洗 / 重新生成

#### ✔ API 返回是否可用？

常见错误：

* 429 → Retry（指数退避）
* 401 → 重新认证
* 数据格式异常 → fallback

RAG 系统要“检索校验”， Agent 系统要“调用校验”。

这两者逻辑几乎是一致的。

### **④ Memory + 变量注入（多轮稳定性）**

继续旅行助手例子：

Turn1：

> “想去北京玩几天。”

Turn2：

> “下周一出发。”

Turn3：

> “帮我查机票。”

如果没有记忆机制，模型不知道从哪出发、不知道目的地、不知道日期。

所以我们做：

> 把 origin、destination、date、duration 记录下来

> 在 System prompt 中自动注入变量

像这样

```
当前会话信息：
- 出发地：{{origin}}
- 目的地：{{destination}}
- 出发日期：{{date}}
- 行程天数：{{duration}}
```

模型一看到这个上下文，自然就不会乱填参数。

### **⑤ 日志驱动优化（Log-driven）**

我们会把每次失败记录成这样的四元组：

```
(query, tool_chosen, params, error_type)
```

再按 error\_type 聚类：

* 参数缺失？ → schema 不明确
* 选错函数？ → 路由需要改
* JSON 拼错？ → 思考链不完整
* API 格式问题？ → 加清洗层

这是每个 Agent 项目的“进化根基”。

## unsetunset四、用案例解释 Agent 是如何一步步变稳的unsetunset

这是面试中最能“拉开差距”的部分。

下面贴两个你可以在面试中复述的场景（完全来自训练营项目，真实、可讲清楚）

### **案例：动态路由如何把准确率提高 25%？**

用户问：

> “帮我查一下明天北京的天气。”

没有路由：给 LLM 6 个工具 → 工具选择错误率 17%（真实统计）

加了路由： → 工具选择错误率掉到 2% → Token 消耗减少 70%

这是面试官最喜欢的那种数据化叙述。

### **案例：Plan-Execute 如何让复杂任务“不会乱”**

用户任务：

> “订机票 + 订酒店”

未拆解：

* 调用顺序混乱
* 参数缺失
* 两步混在一起

加入 CoT + Plan：

* 工具调用成功率从 62% → 92%
* 失败后能正确自我纠正（ReAct）

这些数字面试官一听就知道你是真的做过。

## unsetunset五、总结：Agent 的稳健性不靠“玄学”，靠体系unsetunset

“Function Call 准确率怎么保证？” 这题不是考你记不记得某个框架函数名， 而是考你有没有：

* 动态路由
* 思考链
* 参数校验
* 多轮记忆
* 失败重试
* 日志调优
* badcase 收敛

换句话说，它考的是：**能不能把一个 Agent 系统从 60 分调到 95 分。**

而这些方法，也都是我们在训练营的多个 Agent 实战项目里，从一次次 badcase 里总结出来的。

## unsetunset最后说一句unsetunset

这段时间，我陆续写了二十几篇关于 RAG（检索增强生成）的面试答题文章，阅读量和反馈都非常好。

很多同学说，看完之后不仅知道“怎么答”，还知道“为什么这么答”，甚至能把思路直接用到自己的项目里。

其实，这些文章并不是凭空写出来的，也不是简单整理网络资料，而是**来自我在[大模型训练营](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489051&idx=1&sn=504dfdab92351854b4f097d79e01c847&scene=21#wechat_redirect)里的真实项目沉淀**。

训练营里有多个从零到落地的实战项目。

1、企业培训问答 Agent（含多轮理解与记忆模块）

2、金融研报 RAG 系统（混合检索、重排序、多模态解析）

3、行业深研助手 DeepResearch（实时检索 + 知识沉淀链路）

4、深学 AI 学习助手（上下文结构化与生成链路可解释）

这些实战项目不是“照着文档做一遍”那种，而是会带着同学一步步拆逻辑、跑代码、调权重、对指标，最终能说清楚“为什么这么设计、哪里容易踩坑、怎么迭代优化”。

这些内容最终沉淀成训练营内部的**体系化笔记、方法论文档、Badcase 修复记录和面试表达模板**，而我近期写的那一系列文章，就是从这些文档中衍生出来的。

所以你会看到：

不是只讲概念，而是讲**落地**。

不是只讲方案，而是讲**取舍**。

不是只讲原理，而是告诉你**面试官到底在听什么**。

如果你正在准备[大模型](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489051&idx=1&sn=504dfdab92351854b4f097d79e01c847&scene=21#wechat_redirect)方向的求职，或希望真正把 RAG 从“知道”变成“能做、能讲、能复盘”，那[大模型训练营](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489051&idx=1&sn=504dfdab92351854b4f097d79e01c847&scene=21#wechat_redirect)可能会非常适合你。

真正能拉开差距的，从来不是知识点，而是**体系与思考方式**。

在过去的几个月中，我们已经有超过 **80 个** 同学（战绩真实可查）反馈拿到了**心仪的 offer ，包含腾讯、阿里、字节、华为、快手、智谱、月之暗面、minimax、小红书等各家大厂**以及**传统开发 / 0 基础转行的同学**在短时间内拿到了各类大中小厂的 offer。

如果你近期准备转向[大模型](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489051&idx=1&sn=504dfdab92351854b4f097d79e01c847&scene=21#wechat_redirect)、想拿下一个能讲清楚、能上简历的实战项目，[大模型训练营](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489051&idx=1&sn=504dfdab92351854b4f097d79e01c847&scene=21#wechat_redirect)这可能是你最值得的选择。

往期推荐

[面试官问：Agent 项目经历该怎么写进简历？](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489106&idx=1&sn=ff1f72bcf83ffd0f5c57a9350c26e5e8&scene=21#wechat_redirect)

[面试官问：Agent 的记忆模块是怎么实现的？](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489099&idx=1&sn=a4e98a2f1e18e6b40f72f3055f689541&scene=21#wechat_redirect)

[面试官问：Agent 的规划模块是怎么实现的？](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489091&idx=1&sn=e02839292e5ae86bec4d9483cde6f4da&scene=21#wechat_redirect)

[面试官问：为什么你的 RAG 能做得比别人好？](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489077&idx=1&sn=64e887a5e0143e7f935f1e4027cf1c0c&scene=21#wechat_redirect)