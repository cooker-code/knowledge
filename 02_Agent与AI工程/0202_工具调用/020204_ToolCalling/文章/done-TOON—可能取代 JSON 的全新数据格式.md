> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020204_ToolCalling/020204_核心知识点/工具调用治理与协议边界|工具调用治理与协议边界]]
---
title: TOON—可能取代 JSON 的全新数据格式
author: 代码麻辣烫
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0NjY0MTc0NA==&mid=2247497940&idx=1&sn=62e94738a961bc11c3b6b0dec9588817&chksm=c23bd068aa927e16dfc5fdfdc3747a2a500cfdada25b3f97a03e2a1dcd663f0fabec7a691041&mpshare=1&scene=24&srcid=0101ftVxhdLVnYhjAIZWul7K&sharer_shareinfo=411ecb8176aa9b8f0f915a5669053831&sharer_shareinfo_first=411ecb8176aa9b8f0f915a5669053831#rd
---

每次打开一个 JSON 文件时，你都能看到它——那些无尽的大括号、引号和逗号。几十年来，它一直是我们的通用数据语言，从 API 到数据库再到前端配置，它都发挥着重要作用。但随着我们深入人工智能（AI）和大语言模型（LLM）驱动的世界，JSON 开始显得有些过时了。

现在，想象一个数据大小减半、更易于阅读、解析速度更快且专为大语言模型量身定制的世界。欢迎来到 Token-Oriented Object Notation（TOON）——这种新的数据格式旨在让人工智能通信更加简洁、经济和智能。

## JSON 成为瓶颈的原因

JSON 曾经为我们提供了很大的帮助，但它的结构曾经优雅，如今却在注重 token 的 AI 时代显得冗余。

以下是导致它变慢的原因：

* **冗长的语法**：到处都是大括号、逗号和引号。
* **重复**：每个对象都会反复使用相同的键名。
* **token 成本**：对于像 GPT 或 Claude 这样的 LLM，每个多余的符号都会产生费用。
* **解析开销**：大型 JSON 文件的解析速度较慢，尤其是对于统一数据。
* **未针对 AI 优化**：JSON 是为机器设计的，而不是为语义阅读 token 的模型设计的。

总之，JSON 说得太多了。TOON 学会用一半的词汇表达相同的内容，而这正是关键所在。

## TOON 的样子——快速了解

让我们看看 TOON 如何在不失去清晰度的情况下精简内容。

## 简单示例

```
{
  "name": "Alice",
  "age": 30,
  "city": "Bengaluru"
}
```

```
name: Alice
age: 30
city: Bengaluru
```

没有大括号、逗号和引号——只有有意义的数据。

## 数组

**JSON：**

```
{
  "colors": ["red", "green", "blue"]
}
```

**TOON：**

```
colors[3]: red,green,blue
```

`[3]` 表示数组长度。其他内容？都省略了。

## 对象数组

**JSON：**

```
{
  "users": [
    {
      "id": 1,
      "name": "Alice",
      "role": "admin"
    },
    {
      "id": 2,
      "name": "Bob",
      "role": "user"
    }
  ]
}
```

**TOON：**

```
users[2]{id,name,role}:
  1,Alice,admin
  2,Bob,user
```

在这里，`{id,name,role}` 定义了模式，每行都遵循该顺序，既优雅又紧凑。

## 嵌套对象

**JSON：**

```
{
  "user": {
    "id": 1,
    "profile": {
      "age": 30,
      "city": "Bengaluru"
    }
  }
}
```

**TOON：**

```
user:
  id: 1
  profile:
    age: 30
    city: Bengaluru
```

可读性强、结构化，但仍然简洁——TOON 像是 JSON 和 YAML 之间的桥梁，内置了 LLM 的效率。

## TOON 的重要性

## 1. Token 效率

在 JSON 中，每个大括号和逗号都会增加 token 数量。TOON 去除了多余的字符和重复的键名，在实际测试中可将 token 负载减少 40% - 60%。

例如：

> ❝
>
> JSON → 257 个 token
>
> TOON → 166 个 token

如果你使用结构化数据提示 LLM，这种差异可以降低你的成本并加快输出速度。

## 2. 模型理解

TOON 是为基于 token 的推理而设计的。它的结构与 LLM 解释信息的方式一致，使数据更易于被它们“读取”、理解和处理。

## 3. 对人类也简单

开发者能够迅速理解 TOON 的可读性。它保留了缩进和结构，但去除了标点符号的混乱。

## 4. 对统一数据紧凑

当你的数据集包含数千条类似的行（如交易、日志或事件）时，TOON 的优势就显现出来了。它的模式优先方法使其非常适合一致的记录。

## 实际案例：为 LLM 代理准备数据

假设你正在为一家电子商务公司构建一个**销售分析聊天机器人**。你的模型需要访问数千条日常交易记录。

## JSON 版本

```
{
  "transactions": [
    {
      "id": "T1",
      "user": "U1",
      "amount": 120.00,
      "date": "2025-11-15",
      "category": "Electronics"
    },
    {
      "id": "T2",
      "user": "U2",
      "amount": 45.50,
      "date": "2025-11-14",
      "category": "Books"
    }
  ]
}
```

## TOON 版本

```
transactions[500]{id,user,amount,date,category}:
  T1,U1,120.00,2025-11-15,Electronics
  T2,U2,45.50,2025-11-14,Books
```

**结果：**

* token 数量减少 40%
* 模型更容易解析的模式
* LLM 交互更快且成本更低

你的 AI 助手现在可以专注于洞察，而不是语法。

## 基准测试和 token 节省

| 格式 | 大小 (KB) | Tokens | 解析时间 (ms) |
| --- | --- | --- | --- |
| JSON | 240 | 2600 | 145 |
| TOON | 145 | 1650 | 103 |

这就是更少的文字、更少的 token 和更快的处理速度——正是现代 AI 工作流程所需要的。

## TOON 的未来

TOON 还很年轻，但它的潜力不容小觑。可以期待以下发展：

* 主要生态系统中出现像 `json2toon` 这样的**转换器**。
* 以 TOON 格式发布的**LLM 原生数据集**。
* **提示框架**（如 LangChain、LlamaIndex 等）采用 TOON 进行紧凑的数据交换。
* 在 IDE 和笔记本中进行**工具集成**以实现自动转换。

随着时间的推移，TOON 可能不会完全**取代** JSON，但它很可能会成为 AI 中心工作流程中人类与机器之间的**首选语言**。

## 哲学思考

JSON 反映了一个机器与机器交流的时代——冗长、僵硬、明确。

TOON 则更显人性化。它重视**清晰胜于混乱**，**意义胜于标记**。

在哲学上，笛卡尔曾说：“我思故我在。”在 AI 时代，TOON 可能会低语：“我结构化，所以我理解。”

这是一个小而深远的演变——从冗余走向优雅。

## 最后想法

TOON 是否会在所有地方取代 JSON？可能不会——但它不必如此。

它是新时代的新工具，旨在实现**效率、理解力和 token 意识**。

如果 JSON 定义了**数据时代**，TOON 可能会定义**AI 时代**。