---
title: Claude 开发者平台高级工具使用 (Advanced Tool Use) 功能解读
author: 星河智旅Voyage
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODg0MzA0NA==&mid=2247483878&idx=1&sn=37ea5ca5e276fa3af29b68fb87801488&chksm=c489ff871b17660cc51685ebbd6ee7c9bfc25a01868e66efba43db8f33bf40525146a221a5bb&mpshare=1&scene=24&srcid=12152zG5fazMjeCtEqBQ5X9h&sharer_shareinfo=b96276917b9fc44000c393900f285079&sharer_shareinfo_first=b96276917b9fc44000c393900f285079#rd
---

---

# Claude 开发者平台高级工具使用 (Advanced Tool Use) 功能解读

随着 Agent 系统日益复杂，开发者面临着上下文（Context）限制、推理成本高昂以及工具调用准确性不足的三大挑战。Claude 开发者平台近期推出了三项高级功能，旨在解决这些瓶颈。

---

## 1. Tool Search Tool：从“全量加载”到“按需检索”

### 传统方法的瓶颈

传统方法将所有工具的定义（Schema）一次性加载到上下文中。当工具数量增多或变得复杂时，会产生以下问题：

* **浪费上下文资源**：大量 Token 被工具描述占据。
* **效率低下**：在一个具体任务中，实际上只有极少数工具会被使用。

### 解决方案：引入检索思想 (RAG)

借鉴 RAG（检索增强生成）的思想，Claude 引入了 **Tool Search Tool**。类似于文档检索，Agent 可以根据当前的 Query 检索并只加载相关的工具。

#### 核心机制

* **检索方式**：Claude 提供了基于 Regex（正则表达式）或 BM25 的检索方法，同时也支持开发者自定义（如 Embedding）。
* **延迟加载配置 (`defer_loading`)**：

+ 为工具定义增加 `defer_loading` 字段。
+ **High-use tools (高频工具)**：设置为 `false`，直接载入上下文，保证响应速度。
+ **Low-use tools (低频工具)**：设置为 `true`，仅在通过 Tool Search Tool 检索到时才载入。

> **💡 技术洞察与思考**
>
> * **检索精度的挑战**：这种方式将“检索研究”引入了工具调用领域。如果工具的定义和描述不够全面，检索精度下降会导致 Agent 找不到正确的工具。
> * **Trade-off**：使用 Tool Search Tool 引入了额外的交互步骤（检索->加载->调用）。开发者需要在“时间消耗”和“Token 消耗”之间做平衡。这种方法最适合工具繁多且大部分不会被用到的场景。

---

## 2. Programmatic Tool Calling：程序化工具调用

### 传统 Tool Calling 的局限

传统模式下，Agent 一步一步进行工具调用（Step-by-step），存在两大痛点：

1. **上下文污染**：例如处理大文件分析时，所有中间结果（如读取的CSV内容）都会被读入上下文，导致重要信息被挤出窗口。
2. **推理开销与错误**：工具连接过程完全由 LLM 推理决定，不仅消耗 Token，且每一步都可能产生幻觉或逻辑错误。

### 解决方案：以代码为核心的编排

**Programmatic Tool Calling** 允许 Agent 编写 Python 代码来调用和编排多个工具，具有更严谨的控制逻辑。

#### 执行流程

1. **环境配置**：为 Agent 设置一个 `code_execution` 工具，并将其他工具的 `allowed_caller` 权限开放给它。
2. **代码生成**：Agent 编写代码作为参数调用 `code_execution`。
3. **沙箱执行**：代码在独立环境中运行，发送 Tool Request 并处理 Output。
4. **结果返回**：**仅将最终结果**返回给 Agent 上下文，中间过程数据不进入上下文。

> **❗ 核心优势**
>
> * **净化上下文**：避免了大量中间数据对上下文窗口的占用。
> * **逻辑严密与并行**：通过代码（而非 LLM 的自然语言推理）处理多步依赖或并行调用，增加了准确性并优化了延迟。
> * **本质**：这是一种 Code-based function calling，在处理复杂逻辑时优于传统的 JSON/自然语言模式。

---

## 3. Tool Use Examples：用示例消除歧义

### Schema 定义的不足

仅靠 JSON 格式的 structure-types 和 required fields 往往无法清晰表达使用细节。例如：何时使用可选参数？具体的日期格式是什么？

### 解决方案：Few-Shot Prompting

在工具定义中增加 `input_examples` 字段，通过具体示例指导模型。

```
{  
    "name":"create_ticket",  
    "input_schema":{/* ... */},  
    "input_examples":[  
      {  
        "title":"Login page returns 500 error",  
        "priority":"critical",  
        "labels":["bug","authentication","production"],  
        "reporter":{  
          "id":"USR-12345",// 明确ID格式  
          "name":"Jane Smith"  
        }  
        // ...  
      }  
    ]  
}
```

> **⚠️ 注意事项**
>
> 虽然这会增加 Token 消耗，但在以下场景极具价值：
>
> * 复杂的嵌套结构。
> * 参数具有特定领域约束且未在 Schema 中体现。
> * 存在多个功能相似的工具，需要区分使用场景。

---

## 总结：最佳实践指南

如何结合这三种方式进行优化？我们可以从**瓶颈分析**的角度出发：

| 遇到的瓶颈 | 推荐解决方案 | 原理 |
| --- | --- | --- |
| **工具定义导致上下文冗余** | **Tool Search Tool** | 引入检索，按需加载。 |
| **中间结果导致上下文污染** | **Programmatic Tool Calling** | 代码执行，只取最终结果。 |
| **参数/格式错误调用** | **Tool Use Examples** | 提供示例，消除歧义。 |

**研究启示：**在实践中，往往应该从最朴素的实现开始，发现具体的瓶颈（是上下文不够用了？还是逻辑太复杂出错了？），再针对性地引入上述解决方案。这种“发现瓶颈 -> 解决问题”的过程本身就是一个有意义的研究课题。