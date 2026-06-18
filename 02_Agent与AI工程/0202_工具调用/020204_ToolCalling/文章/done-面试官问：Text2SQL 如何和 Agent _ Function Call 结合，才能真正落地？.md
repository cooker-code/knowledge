> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020204_ToolCalling/020204_核心知识点/工具调用治理与协议边界|工具调用治理与协议边界]]
---
title: 面试官问：Text2SQL 如何和 Agent / Function Call 结合，才能真正落地？
author: 吴师兄学大模型
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489474&idx=1&sn=868b3d9eb1afcd421bc340413923d908&chksm=c306d93aeb8548eebc9b8adfc4a349205dfbd6b20f67b1142672d8d411c984c1fc2f5077b184&mpshare=1&scene=24&srcid=1219CBD6EWvx37o5RFwFA605&sharer_shareinfo=1b3979b868c8bb60c54bc66c097f952f&sharer_shareinfo_first=1b3979b868c8bb60c54bc66c097f952f#rd
---

大家好，我是吴师兄。

这两年面试候选人时，只要对方简历里写了 Text2SQL，面试官几乎都会追问一句：

> 你这个 Text2SQL，是一个 demo，还是一个能放进 Agent 系统里跑的工程？

这个问题，能直接把人分成两类。

一类停留在“能把自然语言翻译成 SQL”；另一类，已经开始思考 **系统如何稳定、可控、可扩展地运行**。

而 Text2SQL 真正的价值，从来不是“生成一条 SQL”，而是：

**作为 Agent 的一个核心工具节点，被调度、被约束、被验证、被复盘。**

今天这篇，我就从面试官视角，把这个问题完整拆开讲清楚。

## unsetunset一、为什么 Text2SQL 必须放进 Agent / Function Call 体系？unsetunset

很多同学做 Text2SQL，流程基本是这样：

```
用户问题
 → 拼 Prompt
 → LLM 生成 SQL
 → 数据库执行
 → 把结果再喂给 LLM
```

这个流程**在 demo 阶段是成立的**，但在真实系统里，有三个致命问题：

1. **无法判断什么时候该查数据库**
2. **无法处理歧义、补充条件**
3. **无法对 SQL 风险和结果正确性负责**

而 Agent + Function Call 的核心作用，就是把“查数据库”这件事：

**从一次 LLM 输出，升级成一次“被调度、被管理的行为”。**

一句话总结：

> Text2SQL 不是对话能力，而是 Agent 的一个只读工具。

## unsetunset二、在 Agent 里，Text2SQL 的真实身份是什么？unsetunset

在工程上，我通常会把 Text2SQL 定义成一个 **只负责查询、不负责决策的工具**。

它的职责非常明确：

* 输入：结构化后的用户查询意图
* 输出：可执行、可验证、受限的 SQL 查询结果

典型的 Function 定义长这样：

```
{
  "name": "text2sql",
"description": "将自然语言查询转换为只读 SQL 并执行",
"parameters": {
    "type": "object",
    "properties": {
      "question": {
        "type": "string",
        "description": "用户的查询问题"
      }
    },
    "required": ["question"]
  }
}
```

注意一个细节：

**Agent 决定“要不要调用 Text2SQL”，Text2SQL 不决定“要不要被调用”。**

这是边界。

## unsetunset三、Agent + Text2SQL 的标准调用流程unsetunset

一个工程级的调用流程，一定不是“用户一句话直接查库”。

而是下面这个结构：

1. Agent 接收用户问题
2. 判断是否涉及“结构化数据查询”
3. 如果存在歧义，先追问
4. 条件齐全后，再调用 Text2SQL
5. 校验 SQL
6. 校验结果
7. 生成最终自然语言回答

你可以把它理解成：

> Text2SQL 是 Agent 工作流中的 **第 N 步，而不是第 1 步**。

## unsetunset四、为什么 Schema 不能一次性塞给 LLM？unsetunset

这是面试里非常高频的一道追问。

如果数据库只有 4 张表，问题不大； 但一旦变成 50 张、200 张表，全量 Schema 会带来两个直接后果：

* Token 暴涨
* 语义噪声严重，准确率下降

工程上真正的做法，是 **动态 Schema 裁剪**。

核心思想只有一句话：

> 只把“可能相关的表”告诉模型。

实现思路也不复杂：

1. 给每张表生成 embedding
2. 用户问题生成 embedding
3. 相似度检索 top-k 表
4. 只把这几张表的结构拼进 Prompt

```
def _get_relevant_schema(self, question: str, top_k: int = 2) -> str:
    question_embedding = self.embedding.embed(question)
    relevant_tables = self._find_similar_tables(question_embedding, top_k)
    return self._format_schema(relevant_tables)
```

这一层，**是 Text2SQL 工程化的分水岭**。

## unsetunset五、歧义不是模型问题，是系统问题unsetunset

面试官如果继续追问，一定会问：

> 用户说“最近”“大涨”“低估值”，你怎么处理？

这里如果回答“让模型自己理解”，基本就结束了。

工程里，歧义必须显式消解。

做法只有两种：

1. **可定义的歧义，直接规则化**
2. **不可定义的歧义，必须追问用户**

例如：

```
BUSINESS_TERMS = {
    "最近": "最近30个自然日",
    "大涨": "涨跌幅 > 5%",
    "低估值": "PE < 15"
}
```

而像“最新”“业绩”“涨幅”这种，就必须进入澄清流程：

```
AMBIGUOUS_TERMS = {
    "最新": ["最新交易日", "最新报告期"],
    "业绩": ["营收", "净利润", "ROE"]
}
```

Agent 的职责，是在调用 Text2SQL **之前** 把问题变清楚。

## unsetunset六、为什么 SQL 安全校验是 P0？unsetunset

我见过太多 Text2SQL demo，直接执行模型生成的 SQL。

这是非常危险的。

在工程里，**SQL 安全校验是绝对的底线**：

* 禁止 DELETE / DROP
* 强制 SELECT
* 强制 LIMIT
* 限制子查询深度

```
FORBIDDEN_KEYWORDS = {
    'DELETE', 'DROP', 'UPDATE', 'INSERT', 'ALTER'
}
```

并且即便模型生成了 LIMIT，也要二次校验：

```
if limit_value > MAX_LIMIT:
    sql = replace_limit(sql, MAX_LIMIT)
```

这一步，不是为了“提高准确率”，而是为了：

> 防止一条 SQL 把整个服务拖死。

## unsetunset七、Text2SQL 的结果也需要“验证”unsetunset

很多人忽略的一点是：

**SQL 语法正确 ≠ 语义正确**

比如：

* 结果为空
* 数值明显异常
* 市盈率 1000+
* ROE 超过 50%

这些都不是模型的错，而是**系统没有做结果校验**。

工程里通常会做三层验证：

1. 返回行数是否合理
2. 数值范围是否合理
3. 让 LLM 自检一次结果是否符合问题

```
if result["row_count"] == 0:
    warnings.append("查询结果为空")
```

最终这些 warning 会被带回给 Agent，用于：

* 重新生成 SQL
* 或提示用户调整条件

## unsetunset八、为什么要做语义缓存？unsetunset

这是一个非常工程的问题。

如果用户反复问：

* “市值最大的银行股”
* “银行里市值最大的是谁”

没有缓存，就会重复：

* embedding
* LLM 调用
* SQL 执行

语义缓存的本质，是：

> 把“问题 → SQL → 结果”当成一个可复用单元。

```
if similarity > threshold:
    return cached_result
```

这一步对 **成本、延迟、稳定性** 都是实打实的收益。

## unsetunset九、Text2SQL 为什么一定要有日志和 Badcase 闭环？unsetunset

最后一个，也是面试官最喜欢问的：

> 你这个系统，怎么持续优化？

如果没有日志，这个问题没法答。

工程里我们会记录：

* 原始问题
* 预处理问题
* 生成 SQL
* 是否执行成功
* 返回行数
* 用户反馈

```
class Text2SQLLog:
    question
    processed_question
    generated_sql
    execution_success
    result_count
```

然后定期跑脚本分析：

* 哪类问题失败最多
* 是 schema 错，还是语义错
* 哪些可以进入 few-shot 示例库

这才是一个 **能长期跑的系统**。

## unsetunset十、面试官最想听到的总结回答unsetunset

如果让我帮你浓缩成一段面试用标准回答，我会这样说：

> 在工程中，我把 Text2SQL 作为 Agent 的一个只读工具来设计。
>
> Agent 负责意图判断、歧义澄清和流程调度，Text2SQL 只在条件齐全时被调用。
>
> 在实现上，我通过动态 Schema 裁剪降低 token 和歧义，通过业务术语词典和澄清机制提升理解准确率，并在执行前加入 SQL 安全校验和 LIMIT 约束，防止风险查询。
>
> 执行后，我会对结果做合理性验证，并结合日志和用户反馈持续优化 few-shot 示例，从而形成稳定可迭代的闭环系统。

面试官听到这里，基本就知道：

**你不是在玩 demo，而是在做工程。**

## unsetunset最后说一句unsetunset

真正能拉开差距的，从来不是知识点，而是**体系与思考方式**。

在过去的几个月中，我们已经有超过 **80 个** 同学（战绩真实可查）反馈拿到了**心仪的 offer ，包含腾讯、阿里、字节、华为、快手、智谱、月之暗面、minimax、小红书等各家大厂**以及**传统开发 / 0 基础转行的同学**在短时间内拿到了各类大中小厂的 offer。

如果你近期准备转向[大模型](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489051&idx=1&sn=504dfdab92351854b4f097d79e01c847&scene=21#wechat_redirect)、想拿下一个能讲清楚、能上简历的实战项目，[大模型训练营](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489051&idx=1&sn=504dfdab92351854b4f097d79e01c847&scene=21#wechat_redirect)这可能是你最值得的选择。

点击了解：[6 周打造工业级 AI Agent：从零到上线的全链路实战](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489269&idx=1&sn=3f33c848c24c1416e7d26b0c52cba0b1&scene=21#wechat_redirect)

我希望，学完、跟着做完这个项目， 让每个学员都有一条能写进简历、能过面试、能转岗/跳槽的硬项目。

这套体系你在网上绝对找不到，因为：

❌ 没人会把「数据采集 + RAG + Backend + ReAct + MCP + Text2SQL + Code Interpreter」全部串起来教

❌ 没人会带你做成一个可部署、可监控、可扩展的完整生产系统

❌ 也没几个人真的把 Agent 做到能独立跑一个行业分析任务

但这次，训练营会手把手带你。

[6 周打造工业级 AI Agent：从零到上线的全链路实战](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489269&idx=1&sn=3f33c848c24c1416e7d26b0c52cba0b1&scene=21#wechat_redirect)正在直播上课！

往期推荐

[从高考落榜到裸辞创业，我走过的所有路都有一个共同点](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489440&idx=1&sn=59788158ae8cd8c68d1260daafad38ef&scene=21#wechat_redirect)

[面试官问：今年大模型应用开发招聘到底看什么？](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489390&idx=1&sn=cfcba94bb0c6d7e8b8fb2ba0781031af&scene=21#wechat_redirect)