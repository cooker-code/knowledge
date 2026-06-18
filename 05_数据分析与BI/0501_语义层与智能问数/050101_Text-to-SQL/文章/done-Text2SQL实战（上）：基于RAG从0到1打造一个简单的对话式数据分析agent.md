---
title: Text2SQL实战（上）：基于RAG从0到1打造一个简单的对话式数据分析agent
author: 反时钟效率笔记
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzNTk3NTYyNA==&mid=2247483868&idx=1&sn=3e067de1a31e19dfe44d14a3913f01ad&chksm=c31a956d3129ed2b628fafc6bd7f6a30540fae43753a920283aaa2b49dc1ca541edb026b8272&mpshare=1&scene=24&srcid=11149bIGLLgVu2RoFXdaOh4h&sharer_shareinfo=f982b1b592a4a2e8df84d2be27ccb2c2&sharer_shareinfo_first=f982b1b592a4a2e8df84d2be27ccb2c2#rd
---


> 已吸收至：[[05_数据分析与BI/0501_语义层与智能问数/050101_Text-to-SQL/050101_核心知识点/Text-to-SQL工程架构与SchemaLinking|Text-to-SQL工程架构与SchemaLinking]]

LLM正以前所未有的速度渗透到各个行业，而让模型直接与企业最核心的数据资产——数据库——进行对话，无疑是其中最激动人心的应用场景之一。

我们期待这样一个未来：

任何业务人员都能用自然语言提问，AI则瞬间返回精准的数据洞察。

这就是Text2SQL的魅力。

然而，理想与现实之间，横亘着一道难以逾越的障碍：大模型固有的“幻觉” 问题。

在无数次实践中，我们发现，即便是最先进的模型，在面对复杂的数据库结构时，也常常会捏造不存在的表名或字段，生成看似合理却无法执行的SQL语句。

这种结果的不可靠性，极大地阻碍了Text2SQL在严肃业务场景中的落地。

那么，除了投入巨大的成本对模型进行微调，我们是否还有另一条路可走？

本文将分享自己结合RAG与Agentic Workflow的agent实践。

接下来，我们将从0到1构建这个Text2SQL智能体的demo，并最终见证它在处理复杂查询时，如何通过自我修正，达成非常不错的准确性和鲁棒性。

## 1.原生Text2SQL的局限

### **1.1 Schema幻觉**

这是最直接的障碍。当数据库包含数十个表、数百个字段时，LLM很难在单次交互中完全记忆和理解整个数据库的结构。其结果是，模型常常会“自信地”生成包含不存在的表名或字段名的SQL语句。

### **1.2 语义鸿沟**

用户的提问方式是业务化、口语化的，而数据库的字段命名则是工程化的。这之间存在着巨大的语义鸿沟。

比如，当用户询问“客单价最高的城市”，LLM需要理解“客单价”并非一个现成的数据库字段，而是一个需要通过 `SUM(total_price) / COUNT(DISTINCT customer_id)` 这样复杂计算得出的业务指标。

缺乏这种深度的领域知识，模型生成的SQL往往“词不达意”。

### **1.3 逻辑脆弱性**

真实的业务分析往往需要跨越多张表的复杂`JOIN`操作。我们发现，随着`JOIN`数量的增加，LLM生成正确关联逻辑的成功率会显著下降。

更关键的是，原生的Text2SQL流程是一个“一次性”的过程。一旦生成的SQL因为逻辑错误（例如错误的关联键、不恰当的聚合方式）而执行失败，整个任务便宣告终止。

系统缺乏一个反馈机制，来告知模型“你错了，再试一次”，从而不具备从失败中学习和修正的能力。

正是这些局限，促使我们必须跳出单纯依赖LLM自身智能的框架，转而探索一种更具结构性、更可靠的系统级解决方案。

## 2.设计思想与核心架构

面对原生Text2SQL的诸多局限，一种常见的思路是投入大量资源对模型进行微调。但这条路径成本高昂，且并非总能奏效。

我的探索方向不同：**不依赖于模型本身能力的跃升，而是通过设计一个更严谨、更具韧性的外部工作流(Workflow) 来“驾驭”模型**。

### 2.1 RAG增强

对抗“Schema幻觉”的第一步，是确保LLM在生成SQL时，其决策依据不是其内部的静态知识，而是我们提供的、与目标数据库实时同步的精确信息。为此，我引入了检索增强生成（RAG）技术，其在本系统中的唯一职责，就是为SQL生成环节**动态构建一个高质量、高相关的上下文。**

我将知识库解构为三种不同类型的知识源，并通过向量检索进行动态调用：

* **DDL****(数据定义语言):** 这是最基础的结构化信息，直接从数据库中提取。它为LLM提供了关于表、字段、数据类型及主外键的精确“物理蓝图”。

```
    -- 示例: customers 表的DDL
    CREATE TABLE "customers" (
    "customer_id" TEXT,
    "customer_name" TEXT,
    "city" TEXT,
    "age" INTEGER,
    "gender" TEXT
)
```

* **业务描述 (Descriptions):** DDL解决了“是什么”的问题，但无法解释“为什么”。我为关键的表和字段手动撰写了业务描述。这些描述用自然语言解释了数据的商业含义，用于弥合工程命名与业务术语之间的语义鸿沟。

```
    // 示例: order_items 表的描述
{
    "content": "order_items表是订单的关键详情表...计算总销售额需要将quantity和unit_price相乘。",
    "type": "description"
}
```

* **查询示例 (Q-SQL Pairs):** 对于一些具有固定模式的复杂查询（尤其是多表`JOIN`），我预置了高质量的“问题-SQL”配对。当用户的提问在语义上与某个示例相似时，这个示例就会被检索出来，为LLM提供一个可以直接参考的、经过验证的查询结构。

```
    // 示例: 复杂JOIN的Q-SQL对
{
    "content": "问题：'手机'品类的总销售额是多少？\nSQL：SELECT SUM(T2.quantity * T2.unit_price) FROM products AS T1 INNER JOIN order_items AS T2 ...",
    "type": "q-sql"
}
```

通过在生成SQL前，先用用户的提问去向量检索这三类知识，我们确保了LLM的每一次执行都有据可依。

### 2.2 Agentic工作流：引入“执行-验证-修复”的反馈闭环

RAG提供了高质量的输入，但并不能100%保证LLM输出的SQL在逻辑上完全正确。为了解决“逻辑脆弱性”，我设计的核心是一个具备反馈能力的工作流。

我的整体实现逻辑如下：

1. **检索:** Agent接收用户问题，并首先向我们构建的知识库发起向量检索，获取与问题最相关的DDL、描述和Q-SQL示例，形成一个临时的、高度聚焦的上下文。
2. **生成 :** Agent将用户问题和检索到的上下文，通过一个精心设计的Prompt，提交给LLM，指令其生成初始的SQL查询语句。
3. **执行:** Agent拿到生成的SQL后，**立即**在一个真实的数据库连接上尝试执行它。这是关键的验证步骤。
4. **验证与修复:**

1. **若执行成功，** 则证明SQL有效，工作流将结果返回，流程结束。
2. **若执行失败，** 数据库必然会返回一个具体的错误信息（例如 `no such column: name`）。Agent会捕获这个错误信息，然后启动**修复循环**： 它将**原始问题、失败的SQL、数据库错误信息**以及**原始的知识库上下文**重新组合，再次调用LLM，并下达一个全新的指令——“请根据这个错误，修复这条SQL”。

## **3.代码实现**

在本章节中，我们将把上一章设计的架构图，转化为可运行、可维护的Python代码。为了保证项目的清晰度和扩展性，我将整个系统解耦为三个核心类：`KnowledgeBase`, `SQLGenerator`, 和 `Text2SQLAgent`。

**整个项目的最终代码结构如下：**

下面，我们将重点剖析`SQLGenerator`和`Text2SQLAgent`这两个最能体现技术细节的类的实现。

### **3.1**`SQLGenerator`

`SQLGenerator`是系统的“大脑”，它不存储知识，只负责推理。其核心是利用精心设计的提示工程，引导LLM完成“生成”和“修复”两个关键动作。

* **步骤1：创建**`sql_generator.py`**文件**

在你的项目文件夹中，创建`sql_generator.py`文件，并引入必要的库。`import os
from typing import List, Dict, Any
from langchain_deepseek import ChatDeepSeek`

* **步骤2：实现****SQL****生成功能 (**`generate_sql`**)**

此方法的核心在于Prompt模板的设计。一个好的模板，是约束LLM行为、确保输出质量的关键。```` class SimpleSQLGenerator:
    def __init__(self, api_key: str = None):
        self.llm = ChatDeepSeek(
            model="deepseek-chat",
            temperature=0,
            api_key=api_key or os.getenv("DEEPSEEK_API_KEY")
        )

    # ... _build_context 方法省略，与前文一致 ...

    def generate_sql(self, user_query: str, knowledge_results: List[Dict[str, Any]]) -> str:
        context = self._build_context(knowledge_results)
        
        # --- Prompt模板是核心 ---
        prompt = f"""你是一个SQL专家，你的任务是根据下面提供的数据库信息和用户问题，生成一句可以在SQLite数据库上执行的SQL查询语句。

        ### 数据库信息:
        {context}
        
        ### 用户问题:
        {user_query}
        
        ### 要求:
        1.  **只返回SQL语句**，不要包含任何额外的解释、注释或格式化（如 ```sql ... ```）。
        2.  确保SQL语法与 **SQLite** 兼容。
        3.  严格使用数据库信息中提供的表名和字段名。不要捏造不存在的字段。
        4.  如果需要进行多表连接（JOIN），请根据表结构中的主外键关系进行。
        5.  理解用户问题中的日期和数值，并正确地应用在WHERE子句中。
        
        SQL语句:
        """
        response = self.llm.invoke(prompt)
        sql_query = response.content.strip()
        
        return sql_query ````

* **步骤3：实现****SQL****修复功能 (**`fix_sql`**)**

这是体现Agent“智能”的关键。此方法使系统具备了从错误中学习的能力。```` def fix_sql(self, user_query: str, original_sql: str, error_message: str, knowledge_results: List[Dict[str, Any]]) -> str:
        context = self._build_context(knowledge_results)
        
        prompt = f"""你是一个SQL调试专家。你之前生成的SQL语句在执行时出错了，请根据错误信息和原始的数据库上下文来修正它。

        ### 数据库信息:
        {context}
        
        ### 用户问题:
        {user_query}
        
        ### 执行失败的SQL:```sql
        {original_sql}
        
        ### 数据库返回的错误信息:
        {error_message}
        
        要求:
        仔细分析错误原因，并生成一句修正后的、正确的SQLite SQL查询语句。
        只返回修复后的SQL语句，不要包含任何解释或道歉。
        
        修复后的SQL语句:
        """
        response = self.llm.invoke(prompt)
        fixed_sql = response.content.strip()

        return fixed_sql ````

### 3.2 Text2SQLAgent：编排核心工作流

Text2SQLAgent是系统的“总指挥”，它不处理具体的推理，只负责编排“检索->生成->执行->修复”这个核心工作流。

* **步骤1：创建`text2sql\_agent.py`文件** 在同一目录下创建该文件，并引入我们刚刚完成的模块。

```
import sqlite3
from typing import Dict, Any, List, Tuple
from .knowledge_base import SimpleKnowledgeBase
from .sql_generator import SimpleSQLGenerator
```

* **步骤2：实现**`run`**方法中的核心工作流**

`run`方法是整个Agent的入口和主循环，它的代码直接反映了我们在上一章设计的架构图。`class SimpleText2SQLAgent:
    def __init__(self, db_path: str, api_key: str = None):
        self.db_path = db_path
        self.knowledge_base = SimpleKnowledgeBase()
        self.sql_generator = SimpleSQLGenerator(api_key)
        self.max_retry_count = 3

    # ... _execute_sql 方法省略 ...

    def run(self, user_question: str) -> Dict[str, Any]:
        print(f"\n{'='*20} 开始处理新查询 {'='*20}")
        print(f"用户问题: {user_question}")

        # --- 步骤 1: 知识库检索 ---
        knowledge_results = self.knowledge_base.search(user_question)

        # --- 步骤 2: 生成初始SQL ---
        sql = self.sql_generator.generate_sql(user_question, knowledge_results)

        # --- 步骤 3 & 4: 执行与重试修复循环 ---
        retry_count = 0
        while retry_count < self.max_retry_count:
            print(f"\n--- 正在执行SQL (第 {retry_count + 1} 次尝试) ---")
            success, result = self._execute_sql(sql)

            if success:
                print("--- SQL执行成功！ ---")
                # ... 返回成功结果 ...
                return {"success": True, "sql": sql, "results": result}
            
            # 如果执行失败，进入修复逻辑
            error_message = result
            print(f"--- SQL执行失败: {error_message} ---")
            
            # 如果还有重试机会
            if retry_count < self.max_retry_count - 1:
                sql = self.sql_generator.fix_sql(user_question, sql, error_message, knowledge_results)
            
            retry_count += 1
        
        # ... 返回最终失败结果 ...
        return {"success": False, "error": f"重试{self.max_retry_count}次后仍然失败。"}`

通过以上步骤，我们便将抽象的架构思想，转化为了具体的、模块化的代码。这种解耦的设计，使得每个部分都可以独立进行测试和优化，为后续的维护和功能扩展奠定了坚实的基础。

## 4.效果评估

把整个程序的框架搭建好后，我还加入了意图分类、智能解读和数据可视化的模块，做成了一个简单的界面，这里就不详细说明了。之后运行程序。可以看到终端的输出，已经可以完整展现我们的整个流程了：

为此，我还设计了一系列覆盖不同难度和业务场景的测试问题。

比如：

> * "统计一下我们顾客的复购率是多少？"

这类需要AI理解并计算复合的业务指标的问题；

> * "对比一下'北京'和'上海'这两个城市的总销售额、平均客单价和顾客数量。"

这类需要执行多次查询或一个复杂的`GROUP BY`查询并整合结果的问题；

> * "哪个顾客最帅？"“今天天气如何？”

这类与业务问题无关或涉及主观判断的问题。

根据反馈结果我进行了一些小的补充和优化，发现大部分问题的回答都是相当不错的。

可以看看下面的演示：

至此，一个简单的对话式数据分析智能体的核心构建工作就完成了。通过模块化的设计和Agentic工作流的引入，我们成功地将一个容易产生“幻觉”的大语言模型，约束成了一个相对可靠、可验证的数据库查询引擎。

这个v1.0版本的实践证明，通过精巧的架构设计，我们可以在不进行模型微调的前提下，显著提升Text2SQL系统在真实业务场景中的鲁棒性和准确性。

然而，当前的实现仅仅是一个起点。在测试过程中，我同样观察到了当前架构存在的三个明显局限，这也让我对于后续的迭代有了一些构想：

1. **架构局限性：从“单步执行”到“多步规划”**

1. **现状：** 当前Agent是一个“单线程”执行模型，面对包含多个分析任务的复合型问题，它只能选择其一进行处理，无法完成多步推理。
2. **演进方向：** 下一步的核心是引入“规划 (Planning)”层。在生成SQL前，先由一个“规划师LLM”将复杂问题拆解为一系列逻辑步骤，再由agent按序调用Text2SQL等工具，完成整个分析链路。

2. **知识库的静态性：从“一次性加载”到“动态自适应”**

1. **现状：** 系统的“知识”（DDL、查询示例等）是静态的，一次性加载后便不再更新，无法适应数据库结构的变化和新查询模式的出现。
2. **演进方向：** 赋予Agent**动态更新知识库**的能力。例如，设计一个反馈机制，将成功的复杂查询案例自动沉淀回知识库，实现系统的“自学习”和持续进化。

3. **交互的单一性：从“无状态查询”到“有记忆对话”**

1. **现状：** 系统目前是“一问一答”的无状态模式，无法理解涉及上下文指代的多轮追问。
2. **演进方向：** 引入对话历史管理机制。通过在每次交互中注入短期记忆，将Agent从一个“查询工具”升级为真正的“对话式分析助手 (Conversational BI)”。

这三个方向，不仅是我们这个项目从v1.0走向v2.0的路线图，也代表了当前Agent技术从“工具执行”迈向“问题解决”的核心演进趋势。

在接下来的系列文章中，我将分享一些更详细的实践过程和技术细节，以及关于构建更强大、更智能的DataAgent的更多思考。

—The End—

如果你看到这里，**或许这篇内容对你来说是有价值的。**

点亮 **「赞」 和 「推荐」**，让更多人也看到它❤️
你的认可是我继续创作下去的动力~

**▍往期文章推荐**

* [n8n实战 | 结合飞书的对话式工作流（技术实现与关键步骤）](https://mp.weixin.qq.com/s?__biz=MzkzNTk3NTYyNA==&mid=2247483838&idx=1&sn=4a816786605de65bbea388fbdeadfa1c&scene=21#wechat_redirect)
* [RPA + AI 实战：我搭建了一个自动化收集招聘数据的工作流（附教程+prompt）](https://mp.weixin.qq.com/s?__biz=MzkzNTk3NTYyNA==&mid=2247483810&idx=1&sn=1ce0efb11d4a4513f8984e7141ff6fd2&scene=21#wechat_redirect)
* [环境配置从入门到进阶：原理、实践与可复用SOP](https://mp.weixin.qq.com/s?__biz=MzkzNTk3NTYyNA==&mid=2247483849&idx=1&sn=d326a14d0f84281c918321f2cc824258&scene=21#wechat_redirect)

**关于作者 · Aren**

06intj | 211计算机本科在读

秩序重建 | 效率提升 | 个人成长 | 技能实践

📬 **链接我：**F00056165（VX）