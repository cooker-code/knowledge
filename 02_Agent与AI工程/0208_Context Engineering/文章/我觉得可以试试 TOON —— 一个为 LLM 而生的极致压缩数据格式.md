---
title: 我觉得可以试试 TOON —— 一个为 LLM 而生的极致压缩数据格式
author: 老码小张
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkxNzY0OTA4Mg==&mid=2247491308&idx=1&sn=e0fa543e8c97c493776252b9bee5d86a&chksm=c0e783ede40711d45ad57e7e9bb1e2abe25c4e1fee8dcbd611af0820e4c7cf42908e929b76fb&mpshare=1&scene=24&srcid=1129wR3CnYE3BGy6IRwu6wwE&sharer_shareinfo=33e0fbfe9f92c56aac2137e2e831863b&sharer_shareinfo_first=33e0fbfe9f92c56aac2137e2e831863b#rd
---

朋友们，有时候我真觉得，agent开发这行越来越像“数据运输业”。  
我们每天都在运数据：前端到后端、后端到数据库、数据库再到模型。  
只是现在的运输成本，最大的不再是网络带宽，反而，潜移默化的变成了——**LLM 的 token 费**。你认同吗？我猜你早就在想怎么在这里省钱了，这不，枕头来了

几毛钱一次 API，看着不心疼。  
可一旦项目跑到几百上千次调用，就开始肉疼。  
于是，有人动了脑筋：**能不能在不损失语义的前提下，把 JSON 压得更紧？**

这，就是 TOON（Token-Oriented Object Notation） 想解决的问题。

---

### 一、JSON 的问题不在“好不好”，而在“贵不贵”

JSON 是我们这些程序员的老朋友了。  
它直观、标准、到处都能解析。  
但问题在于——**它太啰嗦了。**

看下面这段数据：

```
{  
  "users":[  
    {"id":1,"name":"Alice","role":"admin"},  
    {"id":2,"name":"Bob","role":"user"}  
]  
}
```

每一行都在重复 `id`, `name`, `role`，这在 LLM 里就意味着重复的 token。  
你可能没意识到：LLM 看到 `"role": "admin"`，是要一个个词元分开的。  
比如 `{`、`"role"`, `":"`, `"admin"`——全算钱。

如果你每天要给模型塞几百条这样的结构数据，那真是白花花的钱往外流。

---

### 二、TOON 是个什么玩意？

一句话：  
**TOON 是一种为大模型优化的“压缩版 JSON”。**

举个例子，它会把上面的 JSON 写成这样：

```
users[2]{id,name,role}:  
  1,Alice,admin  
  2,Bob,user
```

看到没？没有花哨的引号，没有重复的 key。  
看起来像 CSV，又像 YAML，但其实都不是。

它的几个特点特别有意思：

* • **结构简洁**：像 YAML 一样靠缩进，不靠大括号。
* • **数组声明清晰**：`users[2]{...}` 一眼就能看出长度。
* • **字段只声明一次**：下面直接数据行，干净利落。
* • **LLM 解析更友好**：模型一看就知道“哦，这是一张表”。

---

### 三、这玩意真的省钱吗？

官方的基准测试挺狠。  
他们用 GPT-5 的 tokenizer 测了一堆结构数据，结果如下：

* • 对大数组（上百条结构相同的对象），**token 减少 30%–60%**
* • 对半结构化数据（部分字段不统一），**减少约 15%–30%**
* • 对纯表格类数据（类似 CSV），**略大 5% 左右，但更可靠**

意思是：  
如果你是批量喂模型的，比如知识库 embedding、RAG 文档、批量问答输入，TOON 能让你的调用成本直接少一半。  
这在预算紧张的公司里，简直是省命的优化。

---

### 四、但别太激动，TOON 也不是万能药

我喜欢它，但我也要实话实说。

TOON 有几个“坑点”：

1. 1. **结构必须统一**：  
   也就是说，数组里的对象字段要完全一样，否则格式就乱了。
2. 2. **深层嵌套结构不划算**：  
   如果你的 JSON 是那种嵌套好几层的配置树，那 TOON 的缩进反而更占 token。
3. 3. **生态还在起步**：  
   TOON 目前有 TypeScript SDK 和 CLI 工具，但生态远不如 JSON 成熟。  
   想用在 Python 或 Java 里？要么自己封装，要么等等。
4. 4. **可读性不是人人喜欢**：  
   有些同事第一次看到会问：“这 CSV 谁写的？”😂

---

### 五、来点实战：怎么用？

假设你装了包：

```
pnpm add @toon-format/toon
```

简单一行代码就能转：

```
import { encode } from '@toon-format/toon'  
  
const data = {  
  users: [  
    { id: 1, name: 'Alice', role: 'admin' },  
    { id: 2, name: 'Bob', role: 'user' },  
  ]  
}  
  
console.log(encode(data))
```

输出就是：

```
users[2]{id,name,role}:  
  1,Alice,admin  
  2,Bob,user
```

还可以反向解码回来：

```
npx @toon-format/cli data.toon --decode
```

如果想看 token 节省情况，加个 `--stats`：

```
npx @toon-format/cli input.json --stats
```

它会告诉你：“节省 48% token”，非常爽。

---

### 六、我个人的看法：TOON 是个信号

我不认为 TOON 会取代 JSON。  
但我认为它代表了一个趋势——  
**开发者开始认真对待 token 成本和模型输入结构了。**

过去几年，我们讨论优化时，都是“算法优化”、“前端性能优化”；  
而未来几年，“**提示结构优化（Prompt Structuring）**”会变成一种新工程能力。

你看，过去我们在传输层压缩数据；  
现在，我们在**语义层压缩思维**。  
TOON 正好踩在这个分界点上。

---

### 七、习惯性的总结

TOON 适合：

* • 给大模型批量输入结构化数据（表格类、日志类）
* • 想减少 token 成本的 AI 应用
* • 构建 RAG、Agent、自动化系统的开发者

不适合：

* • 配置文件、树状嵌套结构
* • 小型一次性输入（节省不明显）
* • 要求通用生态兼容的生产系统

一句话：  
**TOON 更像是 LLM 时代的“二级语言”——介于 JSON 和 Prompt 之间的那层中间语。**

---

我知道这类格式层出不穷，很多人会觉得“这又是个玩票的项目”。  
但说实话，我挺喜欢这种“工程师式浪漫”——  
当别人在追求更大的模型、更强的算力，总有人悄悄在研究：  
**怎么用更少的词，让机器更懂人话。**

---

> 🧩 项目地址：https://github.com/toon-format/toon  
> 推荐朋友们收藏一下，  
> 这可能是你未来写 LLM 工具时，用得最多的一种“小众神器”。