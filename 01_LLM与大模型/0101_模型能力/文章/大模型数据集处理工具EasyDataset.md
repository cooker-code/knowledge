---
title: 大模型数据集处理工具EasyDataset
author: CloudNative云原生技术
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU1OTgyMDEzOQ==&mid=2247484090&idx=1&sn=0d9b3e170378e6204b2aa1fcbc026b35&chksm=fd93db8f2d1f932806b6a93bb7aedbd06318961c9f3cfce5b6772be6fca80b815fcdb6d73441&mpshare=1&scene=24&srcid=0425IlxOPKT067rMcOMj6ftA&sharer_shareinfo=3f36030580e36368ccd767c2b2683239&sharer_shareinfo_first=3f36030580e36368ccd767c2b2683239#rd
---

## 大模型数据集处理工具EasyDataset

> Easy Dataset 是一个专为创建大型语言模型（LLM）微调数据集而设计的应用程序。它提供了直观的界面，用于上传特定领域的文件，智能分割内容，生成问题，并为模型微调生成高质量的训练数据  
> 通过 Easy Dataset，您可以将领域知识转化为结构化数据集，兼容所有遵循 OpenAI 格式的 LLM API，使微调过程变得简单高效

---

* •Easy-Dataset官方说明

---

### EasyDataset支持的数据集格式

1. 1. Alpaca 格式

* •**Github仓库:**`https://github.com/tatsu-lab/stanford_alpaca`
  > Alpaca 由斯坦福团队发布，常见于各类 LLaMA 系列模型的指令微调（instruction‑tuning）。其核心思想是用三元组 (instruction, input, output) 来表示一条样本

```
[  
  {  
    "instruction":"将下面英文翻译成中文：I love you.",  
    "input":"",  
    "output":"我爱你。"  
},  
{  
    "instruction":"计算长为 10cm、宽为 5cm 的矩形面积。",  
    "input":"",  
    "output":"矩形的面积是 50 平方厘米。"  
},  
{  
    "instruction":"总结下面这篇文章：",  
    "input":"（一大段文章内容）",  
    "output":"文章主要讲述了……"  
}  
]
```

* •`instruction（必需）`：任务描述
* •`input（可选）`：提供上下文或额外信息，若无可置空字符串
* •`output（必需）`：模型应生成的回答

1. 2. ShareGPT 格式
   > ShareGPT 格式源自 ChatGPT 聊天记录的导出，特点是保留多轮对话，适合训练聊天机器人或多轮对话模型。典型的 JSON 数组，每个元素表示一次完整会话

```
[  
  {  
    "system":"You are a helpful assistant.",            // （可选）系统提示  
    "conversations":[                                   // 对话列表  
      {"from":"human","value":"你好！"},  
      {"from":"gpt",   "value":"你好！有什么可以帮您？"},  
      {"from":"human","value":"解释一下量子力学。"},  
      {"from":"gpt",   "value":"量子力学是一门……"}  
    ],  
    "tools":"可选的工具描述，比如检索或计算工具"              // （可选）额外信息  
},  
{  
    "system":"",  
    "conversations":[  
      {"from":"human","value":"今天天气如何？"},  
      {"from":"gpt",   "value":"请提供您的城市信息。"}  
    ]  
}  
]
```

* •`system（可选）`：对话开始的系统提示
* •`conversations：`消息列表，每条消息为

+ •`from:`"human" 或 "gpt"（或 "user"/"assistant"）
+ •`value:`消息内容

* •`tools（可选）`：当对话涉及外部工具（检索、计算等）时，可在此描述
* •**ms‑swift文档对ShareGPT格式说明**:`https://swift.readthedocs.io/en/latest/Customization/Custom-dataset.html?utm_source=chatgpt.com`

### 使用场景

| **场景** | **推荐格式** | **理由** |
| --- | --- | --- |
| 单轮指令微调 | `Alpaca` | 结构简单，专注指令—响应 |
| 多轮对话/聊天机器人训练 | `ShareGPT` | 原生保留多轮上下文，可学习对话流 |
| 需混合单轮 & 多轮场景 | `双格式混用` | 预处理时统一映射到内部标准结构（如 messages 列表） |

### Easy-Dataset功能特点

* •**智能文档处理**：上传 Markdown 文件并自动将其分割为有意义的片段
* •**智能问题生成**：从每个文本片段中提取相关问题
* •**答案生成**：使用 LLM API 为每个问题生成全面的答案
* •**灵活编辑**：在流程的任何阶段编辑问题、答案和数据集
* •**多种导出格式**：以各种格式（Alpaca、ShareGPT）和文件类型（JSON、JSONL）导出数据集
* •**广泛的模型支持**：兼容所有遵循 OpenAI 格式的 LLM API
* •**用户友好界面**：为技术和非技术用户设计的直观 UI
* •**自定义系统提示**：添加自定义系统提示以引导模型响应

> **如何安装使用参考官方详细说明即可**`https://github.com/ConardLi/easy-dataset/blob/main/README.zh-CN.md`