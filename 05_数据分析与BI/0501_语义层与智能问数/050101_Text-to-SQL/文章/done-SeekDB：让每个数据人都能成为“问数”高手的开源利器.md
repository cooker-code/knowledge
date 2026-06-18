---
title: SeekDB：让每个数据人都能成为“问数”高手的开源利器
author: 全栈数据
date:
url: https://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458355627&idx=1&sn=57f0832018be580b559a11398a80b763&chksm=819650aeafda9d229153345d8a19174ad5efafc1c793ef77142ffedf7ebdb57ba89a56c8edab&mpshare=1&scene=24&srcid=0116zQnGkPfOKtEcxYpb3U55&sharer_shareinfo=ec166d2b5199153b04ceb489d9d83ce1&sharer_shareinfo_first=ec166d2b5199153b04ceb489d9d83ce1#rd
---


> 已吸收至：[[05_数据分析与BI/0501_语义层与智能问数/050101_Text-to-SQL/050101_核心知识点/Text-to-SQL工程架构与SchemaLinking|Text-to-SQL工程架构与SchemaLinking]]

**引言：当数据洪流遇上“语言不通”的困境**

我们正处在一个前所未有的数据爆炸时代。无论是互联网巨头、金融科技公司，还是一个普通的创业团队，每天都在产生和积累海量的数据。这些数据是金矿，蕴藏着用户行为、业务趋势和市场机会。然而，这座金矿并非唾手可得。要挖掘其价值，传统的方式是写SQL（结构化查询语言）——一种强大但对非专业人士极不友好的数据库查询语言。

**想象一下这样的场景：**

* 市场部的同事想快速知道上个月哪个产品的销量最好，却不得不排队等待数据分析师；
* 运维工程师需要排查一个突发的性能问题，但复杂的表结构和关联关系让他无从下手；
* 甚至是你自己，一个熟练的开发者，在面对一个全新的、文档不全的数据库时，也需要花费大量时间去理解Schema，才能写出正确的查询语句。

这本质上是一个“语言不通”的问题。人类用自然语言思考和提问，而机器只认得冰冷的SQL。这个鸿沟，不仅降低了数据驱动决策的效率，也极大地限制了数据价值的普惠性。我们渴望一种工具，能像一个聪明的“翻译官”，将我们的日常语言瞬间转化为精准的数据库指令。

今天，我们要介绍的正是这样一位“翻译官”——由**蚂蚁集团开源的SeekDB**。它不仅仅是一个工具，更是一个旨在打破数据壁垒、赋能每一位数据使用者的开源智能问数平台。

SeekDB——揭开智能问数引擎的神秘面纱

SeekDB是什么？简单来说，它是一个**Text-to-SQL**（文本到SQL）的开源解决方案。它的核心使命非常明确：让用户通过自然语言提问，自动、准确地生成可执行的SQL查询，并返回直观的结果。

你可能会问，市面上不是已经有很多BI（商业智能）工具了吗？它们也能做图表、看板。但SeekDB的定位有所不同。它更底层、更灵活、更面向开发者和深度数据用户。它不是一个封闭的SaaS产品，而是一个你可以完全掌控、自由定制、深度集成的开源引擎。

**SeekDB的设计哲学源于蚂蚁集团内部庞大的数据应用场景**。在蚂蚁，每天有成千上万的员工需要与数据打交道，从风控专家到产品经理，从运营人员到普通客服。为了提升整个组织的数据生产力，蚂蚁的技术团队打造了SeekDB，并最终决定将其开源，回馈社区。

那么，SeekDB究竟有哪些过人之处，让它在众多同类项目中脱颖而出呢？

**产品架构**

SeekDB的三大核心能力——为何它值得你关注？

SeekDB的强大，并非空穴来风。它围绕着三个关键支柱构建了自己的护城河：**准确性、安全性与易用性。**

**1. 精准如“老中医”**：Schema Linking与多轮对话的魔法

Text-to-SQL最大的挑战是什么？是歧义。用户的提问往往是模糊、不完整的。比如，“看看销售额”这句话，系统需要知道：

* “销售额”对应数据库里的哪个字段？（可能是revenue, sales\_amount, total\_price...）
* 是哪个时间段的销售额？
* 需要按什么维度聚合？（按天、按产品、按地区？）

SeekDB解决这个问题的核心技术叫做Schema Linking（模式链接）。在用户提问的瞬间，SeekDB会启动一个智能匹配过程，将问题中的关键词（如“销售额”、“用户”、“订单”）与数据库的元数据（表名、列名、注释、甚至采样数据）进行深度关联。它能理解“销售额”最可能对应的是orders表里的amount字段，并且知道这个字段是数值类型，适合做SUM聚合。

**从搜索到推理，全栈赋能 AI 应用开发**

**1、混合搜索**

* 一条 SQL 查询中支持基于向量的语义召回和基于关键字的召回，优化的多路召回
* 查询重排序支持权重、RRF，也支持基于大模型的重排序
* 支持标量过滤下压存储的优化，还可以使用多表关联搜索相关数据

**2、向量 & 全文搜索**

* 支持稠密向量、稀疏向量，支持曼哈顿距离、欧式距离、内积、余弦距离等多种类型向量距离的计算
* 向量索引支持基于内存的 HNSW、HNSW\_SQ、HNSW\_BQ 和基于磁盘的 IVF、IVF\_PQ 等索引类型，优化向量存储成本
* 全文搜索支持关键字、短语及布尔表达式等多种匹配模式，支持 BM25 排序

**3、AI 函数**

* DBMS\_AI\_SERVICE 包用 SQL 对内置大模型服务进行管理，并支持注册外部大模型服务
* 通过 AI\_EMBED 函数在 SQL 中把文本转换为向量嵌入
* 通过 AI\_COMPLETE 函数在 SQL 中执行文本生成，支持用提示词模板化复用
* 通过 AI\_RERANK 函数在 SQL 中使用重排序大模型对文本进行排序

但这还不够。真正的交互是动态的。SeekDB支持多轮对话。如果初次生成的SQL不够准确，或者用户想进一步钻取细节，系统会主动追问：“您是指最近7天的销售额吗？”或者“需要按产品类别分组展示吗？”。这种类似真人对话的体验，极大地提升了查询的准确率和用户的满意度。它不是一次性的问答机，而是一个能与你共同探索数据的智能伙伴。

**2. 安全如“保险柜”：企业级数据安全的坚实保障**

对于任何涉及生产数据库的工具，安全是生命线。很多开源的Text-to-SQL项目在Demo阶段很惊艳，但一旦接入真实环境，就可能因为权限失控而带来巨大风险。SeekDB从设计之初就将安全放在首位。

它实现了精细化的权限控制。你可以为不同的用户或角色配置不同的数据库访问权限。比如，财务人员只能看到与财务相关的表，而无法访问用户隐私信息表。SeekDB在生成SQL之前，会严格校验当前用户是否有权访问所涉及的表和字段。这意味着，即使一个不懂SQL的实习生通过SeekDB提问，他也绝对无法越权查询到他不该看到的数据。

此外，SeekDB还内置了SQL注入防护和查询资源限制机制。它可以识别并拒绝恶意构造的查询语句，同时可以限制单次查询返回的最大行数或执行时间，防止因一个不当的“全表扫描”请求拖垮整个数据库。这些企业级的安全特性，让SeekDB不仅能用于个人学习和探索，更能放心地部署在真实的生产环境中。

**3. 易用如“乐高”：开箱即用与深度定制的完美平衡**

一个好的开源项目，不仅要功能强大，更要易于上手和集成。SeekDB在这点上做得非常出色。

首先，它提供了开箱即用的Web界面。你只需按照简单的文档，配置好你的数据库连接信息，启动服务，就能立刻拥有一个功能完备的智能问数平台。界面简洁直观，左侧是数据库Schema树，中间是聊天窗口，右侧是查询结果和生成的SQL。你可以清晰地看到系统是如何理解你的问题，并将其转化为代码的，这对于学习SQL和理解数据模型也非常有帮助。

其次，SeekDB的架构高度模块化。它的核心能力被封装成独立的服务，这意味着你可以轻松地将其集成到你自己的应用中。无论你是在开发一个新的BI工具，还是想为你的内部管理系统增加一个“数据问答”功能，SeekDB都能作为一个强大的后端引擎为你提供支持。这种灵活性，正是开发者们所珍视的。

动手实践——五分钟体验SeekDB的魅力

**代码片段：可切换语言，无法单独设置文字格式**

理论说得再好，不如亲自上手试试。让我们用一个简单的例子，感受SeekDB带来的效率革命。

文档地址：https://www.oceanbase.ai/docs/zh-CN/seekdb-overview/

**python pip安装**

```
pip install pyseekdb
```

安装 pyseekdb 的同时也会安装嵌入式模式的 seekdb，使您可以直接连接到嵌入式 seekdb 执行创建数据库等操作。

如果您的 pip 版本比较低，请先升级 pip 后再安装。

```
pip install --upgrade pip
```

连接数据库

```
import pyseekdb
# Create embedded admin clientadmin = pyseekdb.AdminClient(path="./seekdb.db")# Create databaseadmin.create_database("hybrid_search_test")
```

插入数据

```
import pyseekdb
# Create embedded clientclient = pyseekdb.Client(path="./seekdb.db", database="hybrid_search_test")# get collectioncollection = client.get_collection("hybrid_search_demo")
# Define documentsdocuments = [    "Machine learning is revolutionizing artificial intelligence and data science",    "Python programming language is essential for machine learning developers",    "Deep learning neural networks enable advanced AI applications",    "Data science combines statistics, programming, and domain expertise",    "Natural language processing uses machine learning to understand text",    "Computer vision algorithms process images using deep learning techniques",    "Reinforcement learning trains agents through reward-based feedback",    "Python libraries like TensorFlow and PyTorch simplify machine learning",    "Artificial intelligence systems can learn from large datasets",    "Neural networks mimic the structure of biological brain connections"]# Define metadatasmetadatas = [    {"category": "AI", "topic": "machine learning", "year": 2023, "popularity": 95},    {"category": "Programming", "topic": "python", "year": 2023, "popularity": 88},    {"category": "AI", "topic": "deep learning", "year": 2024, "popularity": 92},    {"category": "Data Science", "topic": "data analysis", "year": 2023, "popularity": 85},    {"category": "AI", "topic": "nlp", "year": 2024, "popularity": 90},    {"category": "AI", "topic": "computer vision", "year": 2023, "popularity": 87},    {"category": "AI", "topic": "reinforcement learning", "year": 2024, "popularity": 89},    {"category": "Programming", "topic": "python", "year": 2023, "popularity": 91},    {"category": "AI", "topic": "general ai", "year": 2023, "popularity": 93},    {"category": "AI", "topic": "neural networks", "year": 2024, "popularity": 94}]
ids = [f"doc_{i+1}"for i in range(len(documents))]# Insert datacollection.add(ids=ids, documents=documents, metadatas=metadatas)
```

执行混合搜索并打印查询结果

```
import pyseekdb
# Create embedded clientclient = pyseekdb.Client(path="./seekdb.db", database="hybrid_search_test")# get collectioncollection = client.get_collection("hybrid_search_demo")
# Perform hybrid searchhybrid_result = collection.hybrid_search(    query={"where_document": {"$contains": "machine learning"}, "n_results": 10},    knn={"query_texts": ["AI research"], "n_results": 10},    rank={"rrf": {}},    n_results=5)
# Print resultsprint("\nhybrid_search() Results:")print(f"  ids: {hybrid_result ['ids'][0]}")print(f"  Document: {hybrid_result ['documents'][0]}")
```

场景：你有一个电商数据库，包含users（用户表）、products（产品表）和orders（订单表）。你想知道“上个月北京地区购买iPhone的用户有多少？”

**传统方式：**

1. 打开数据库客户端。
2. 回忆或查找相关表的结构。
3. 编写复杂的JOIN和WHERE条件SQL。
4. 执行，如果出错，调试语法或逻辑。
5. （假设一切顺利）得到结果。

**SeekDB方式：**

1. 在SeekDB的聊天框里，直接输入：“上个月北京地区购买iPhone的用户有多少？”
2. SeekDB瞬间完成以下工作：
3. 理解意图：这是一个COUNT查询，目标是“用户”。
4. Schema Linking：识别“北京”对应users.city，“iPhone”对应products.name，“上个月”需要处理日期。
5. 生成SQL：自动生成类似SELECT COUNT(DISTINCT u.user\_id) FROM users u JOIN orders o ON u.user\_id = o.user\_id JOIN products p ON o.product\_id = p.product\_id WHERE u.city = '北京' AND p.name LIKE '%iPhone%' AND o.order\_date >= DATE\_SUB(CURDATE(), INTERVAL 1 MONTH);的语句。
6. 执行并返回：以表格或数字的形式，直接告诉你答案。

整个过程，从提问到获得洞察，可能只需要几秒钟。更重要的是，你无需关心背后的SQL语法细节，可以将全部精力集中在业务问题本身。

不止于查询——SeekDB开启的数据新范式

SeekDB的价值远不止于简化一个查询动作。它正在悄然改变我们与数据交互的方式，开启一种全新的数据工作范式。

**1. 降低数据使用门槛，实现“全民数据化”**

当业务、产品、运营等非技术岗位的同事也能自助获取数据时，整个组织的决策链条将变得前所未有的敏捷。他们不再需要等待，可以随时验证自己的想法，快速迭代策略。数据不再是少数人的特权，而是每个人手中的利器。

**2. 提升开发者和分析师的生产力**

对于专业的数据工程师和分析师而言，SeekDB同样是一个强大的助手。在**探索性数据分析（EDA）**阶段，他们可以用自然语言快速验证假设、发现异常，将宝贵的时间从繁琐的SQL编写中解放出来，投入到更高价值的建模和洞察工作中。

**3. 构建下一代智能数据应用的基石**

SeekDB的开源，为整个社区提供了一个高质量的Text-to-SQL基础组件。开发者可以基于它，构建出无数创新的应用场景：嵌入到企业微信/钉钉里的数据机器人、结合大模型的智能数据Agent、甚至是面向消费者的个性化数据报告生成器。SeekDB就像一块坚实的乐高积木，等待着有创造力的你去搭建更宏伟的建筑。

★★★★★五星推荐★★★★★

数据的价值在于流动和使用。SeekDB所做的，就是移除横亘在人与数据之间的那堵墙，让信息的流动变得如呼吸般自然。它代表了一种趋势：未来的数据工具，将不再是冰冷的命令行或复杂的配置面板，而是能够理解我们、与我们对话的智能伙伴。

作为一款由蚂蚁集团贡献给社区的开源项目，SeekDB不仅技术先进，而且经过了大规模生产环境的严苛考验。无论你是希望提升个人效率的数据爱好者，还是寻求为企业构建智能数据能力的开发者，SeekDB都值得你投入时间去了解、去尝试、去贡献。

更多数据科学和技术，请扫码关注：**全栈数据**