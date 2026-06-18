---
title: 本体（Ontology）与知识图谱如何通过标注防止大模型幻觉
author: 算子之心
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI5NjU0MzY2Nw==&mid=2247486893&idx=1&sn=3225500a6fe8c483f04d4a6fb3bc0112&chksm=ed38def149fce4073f01b45fcef82c3ad30e9dbf8b4a38459a05d1739bd95a23517b7037be4e&mpshare=1&scene=24&srcid=0408KLrxg4MctBkT6QGEDGJv&sharer_shareinfo=500490958347b81a88e3f77ed04cc23b&sharer_shareinfo_first=500490958347b81a88e3f77ed04cc23b#rd
---

# 

# 

> 用标注驱动生成打通语义与语法

大语言模型（LLM）创造力强、表达流畅，但常常出现**幻觉**——在不知道答案时编造事实。而另一方面，知识图谱（如 Neo4j）与 RDF 数据 100% 基于事实，却对非专业人员极不友好。

本文将介绍如何借助 **LangGraph**、**Neo4j** 和**本体标注**，把这两者结合起来。我们会搭建一套系统，让 LLM 充当**翻译器**，并严格遵循本体中定义的规则。

### 相关工作（WebNLG、Graph2Text、RDF2Text）

要理解我们的方向，必须先回顾结构化数据转文本生成的发展历程：

1

**WebNLG 挑战赛**  
该基准任务要求模型将 RDF 三元组转换为自然语言文本。早期方法使用序列到序列（Seq2Seq）模型，虽然文本流畅，但在处理未见过的实体时表现很差，还经常丢失具体数据点。

2

**Graph2Text**  
研究者利用图神经网络（GNN），尝试将图的结构信息直接编码到神经网络隐状态中。这提升了关系编码能力，但在扩展到复杂、嵌套的本体时仍然脆弱。

3

**RDF2Text 与预训练大模型**  
随着 LLM 出现，研究重心转向将 RDF 线性化（把三元组转成文本字符串）并通过提示词引导模型。这种方式生成的文本流畅度很高，但引入了**幻觉**：模型会优先追求统计合理性，而非事实正确性。

我们的方案在此基础上更进一步：使用 LangGraph 实现**状态机架构**，确保 LLM 只做翻译，不做创造。

---

### 本体驱动的语言标注模型（ODLAM）

我们方案的核心是**本体驱动的语言标注模型（Ontology-Driven Linguistic Annotation Model, ODLAM）**。  
在 LLM 接触任何数据之前，本体就已经作为**语义框架的 schema** 存在。

•

**RDF 的作用**：一条 RDF 三元组  
`(主语, 谓词, 宾语)`  
是事实，但表述生硬枯燥。

•

**本体的作用**：OWL 定义的本体描述**语义**。它会告诉我们：  
`hasParent` 是逆函数属性，或者 `Person` 是 `Agent` 的子类。

•

**语言标注**：我们用本体将谓词映射到自然语言模板或“框架”。例如：

•

RDF：  
`ex:Earth ex:radius "6371 km"`

•

本体约束：  
`ex:radius` 是定义域为 `Planet` 的数据属性

•

语言框架：  
“[主语] 的半径为 [宾语]。”

这种映射存储在知识图谱中，供智能体调用，成为**严格的 RDF 结构**与**流畅自然语言**之间的桥梁。

---

### 符号—神经混合架构

我们使用 **LangGraph** 实现这套系统——一个用于构建有状态、多智能体 LLM 应用的库。  
与简单的链（chain）不同，LangGraph 支持**循环（loop）**，这对**事实校验**至关重要。

---

### 智能体工作流

整个架构在 LangGraph 状态机中包含三个核心节点：

1

**规划（Plan）**（神经）  
该 LLM 智能体接收 RDF 图，查询本体以理解关系。**它不直接写最终文本**，而是生成一份**规划**（按顺序排列的待表述事实清单）。

2

**校验（Verify）**（符号/规则驱动）  
该节点**非神经**。它用 Python 函数将规划结果与原始 RDF 比对。  
如果规划想写“火星半径 6371 千米”，但 RDF 里是“地球”，校验器会直接中断流程。

3

**生成（Generate）**（神经）  
规划通过校验后，该智能体根据合法规划与本体映射，生成流畅文本。

---

###

### 1. 数据层：从 RDF 到 Neo4j

从一个简单例子开始：关于一个人 Alice 的数据。

RDF 三元组（主语, 谓词, 宾语）：

```
ex:Alice  ex:worksAt   ex:TechCorp .  
ex:Alice  ex:hasSkill  "Python" .
```

在 Neo4j 中：RDF 被存储为节点与关系。

•

节点 A：`Person {name: "Alice"}`

•

节点 B：`Company {name: "TechCorp"}`

•

关系：`[:WORKS_AT]` 连接 A → B

---

### 2. 核心秘诀：本体标注

这是最关键的部分。  
普通 LLM 看到 `ex:worksAt` 可能会猜测含义，但有时会猜错（比如“Alice works hard at TechCorp” 而不是 “Alice is employed by TechCorp”）。

我们的解决方法：**给本体添加语言标注**，精确定义关系该如何表述。

本体（Turtle 格式）：我们定义一个自定义属性 `ex:verbalizationTemplate`。

```
ex:worksAt  
    rdf:type        rdf:Property ;  
    rdfs:label      "works at" ;  
    rdfs:comment    "Indicates employment relationship" ;  
    # 下面这条就是给 LLM 智能体看的标注：  
    ex:verbalizationTemplate "[Subject] is employed by [Object]" .
```

为什么重要：  
`ex:verbalizationTemplate` 相当于**严格指令**，告诉 AI：  
“看到这个关系，就用这个句式。”  
这消除歧义，大幅减少幻觉。

---

### 3. 架构实现：LangGraph 与多智能体

我们用 LangGraph 管理整个工作流。  
不再简单让 LLM“写一段关于 Alice 的内容”，而是构建**状态机**，强制 AI 在说话前先查询本体。

### 工作流程

1

检索节点：从 Neo4j 获取数据

2

增强节点：获取该数据对应的本体标注

3

生成节点：用模板生成句子

4

校验节点：确保输出与输入一致

---

### 4. 端到端示例

下面是系统内部完整执行步骤：

#### 步骤 1：用户请求

用户：“介绍一下 Alice。”

#### 步骤 2：Neo4j 检索（Fetch 智能体）

LangGraph 中第一个智能体查询 Neo4j。

•

Cypher 查询：  
`MATCH (p:Person {name: 'Alice'})-[r]->(o) RETURN p, r, o`

•

结果：

•

关系：`WORKS_AT`，宾语：`TechCorp`

•

关系：`HAS_SKILL`，宾语：`Python`

#### 步骤 3：本体查询（Enrich 智能体）

系统看到关系 `WORKS_AT`，去本体库中查询标注。

•

输入：`ex:worksAt`

•

本体返回：

```
{  
  "label": "works at",  
  "template": "[Subject] is employed by [Object]"  
}
```

#### 步骤 4：文本生成（Writer 智能体）

LLM 同时收到**数据**和**模板**，不需要“猜”句式。

•

给 LLM 的提示：

> 你有一条事实：  
> 主语='Alice'，关系='worksAt'，宾语='TechCorp'。  
> 本体规则：使用模板 "[Subject] is employed by [Object]"。  
> 转换为自然语言。

•

LLM 输出：  
“Alice is employed by TechCorp.”

#### 步骤 5：事实校验（Guardrails 护栏）

最后用**符号逻辑**（Python）校验句子中的实体与数据库一致。

•

检查：“Alice” 在库中吗？是。

•

检查：“TechCorp” 在库中吗？是。

•

结果：**通过**

---

### 5. LangGraph 实现细节

下面是用 LangGraph 定义循环的概念版 Python 代码：

```
from langgraph.graph import StateGraph, END  
from typing import TypedDict  
  
# 1. 定义状态（在智能体之间传递的数据）  
class AgentState(TypedDict):  
    user_query: str  
    graph_data: dict       # 来自 Neo4j  
    templates: dict        # 来自本体  
    final_text: str  
  
# 2. 定义智能体节点  
  
def neo4j_retriever(state: AgentState):  
    # 模拟从 Neo4j 获取数据  
    print("--- Fetching from Neo4j ---")  
    # 实际中：driver.session().run("MATCH ...")  
    data = {  
        "subject": "Alice",  
        "predicate": "worksAt",  
        "object": "TechCorp"  
    }  
    return {"graph_data": data}  
  
def ontology_enricher(state: AgentState):  
    # 模拟获取前面定义的标注  
    print("--- Fetching Ontology Templates ---")  
    predicate = state['graph_data']['predicate']  
  
    # 获取 ex:verbalizationTemplate 的逻辑  
    # 如果谓词是 'worksAt'，使用模板：  
    template = "[Subject] is employed by [Object]"  
  
    return {"templates": {"worksAt": template}}  
  
def generator_agent(state: AgentState):  
    print("--- Generating Text ---")  
    data = state['graph_data']  
    template = state['templates']['worksAt']  
  
    # 简单符号替换（也可用 LLM 润色）  
    # 这里直接符号替换，确保 0% 幻觉  
    text = template.replace("[Subject]", data['subject'])  
    text = text.replace("[Object]", data['object'])  
  
    return {"final_text": text}  
  
# 3. 构建图  
workflow = StateGraph(AgentState)  
  
workflow.add_node("retriever", neo4j_retriever)  
workflow.add_node("enricher", ontology_enricher)  
workflow.add_node("generator", generator_agent)  
  
# 定义流程  
workflow.set_entry_point("retriever")  
workflow.add_edge("retriever", "enricher")  
workflow.add_edge("enricher", "generator")  
workflow.add_edge("generator", END)  
  
# 编译并运行  
app = workflow.compile()  
result = app.invoke({"user_query": "Tell me about Alice"})  
  
print("\nFinal Output:", result['final_text'])
```

---

### 结论

通过使用**本体标注**（如 `verbalizationTemplate`），我们把 LLM 从“创意作家”变成了“严谨记者”。

1

Neo4j 保存**事实**

2

本体保存**语言规则**（标注）

3

LangGraph 确保智能体严格遵循顺序：  
**检索 → 增强 → 生成**

这种**混合架构**极大减少幻觉，因为 AI 不再编造关系，而只是**按照知识图谱给出的模板填空**。

-------------------------------------------------------------

希望这篇文章能为您带来一些帮助。如果有任何疑问或建议，请在评论区留言，我们将尽力回答！

让我们一起探索并推动前沿技术发展！🚀💻 

祝好运！😊✍️