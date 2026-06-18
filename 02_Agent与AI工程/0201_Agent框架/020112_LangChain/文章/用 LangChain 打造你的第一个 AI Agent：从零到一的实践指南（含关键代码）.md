---
title: 用 LangChain 打造你的第一个 AI Agent：从零到一的实践指南（含关键代码）
author: 弓长先生的杂货铺
date: 
url: https://mp.weixin.qq.com/s/FMnYdmo9pkKtE_51Ir4a2Q
---

想开发一个 AI Agent，有三项核心功能是你绕不开的：**大模型接入、工具决策和任务管理**。

目前市面上主流的 Agent 开源框架，比如 **LangChain、AutoGen 和 Google ADK**，对这三项功能都有不错的支持，而且各有特色：

* **LangChain**：推出时间较早，内置工具包最丰富，学习资源也多。GitHub 上超过 11 万的 star，可见它的受欢迎程度。
* **AutoGen**：微软的开源项目，对 Azure 生态支持有天然优势。
* **Google ADK**：Google 在最近推出的 SDK，对多 Agent 交互和 Google Cloud 原生应用支持较强，开发语言支持 Python 和 Java。目前 ADK 的工具包仍在持续完善中。

本文将基于 **LangChain**框架来开发 Agent。

### 一、LangChain 对 Agent 核心能力的支持

### 

针对 Agent 的三项关键能力，LangChain 框架提供了如下支持：

1. **能力接入**：LangChain 自身的工具库非常齐全，提供了丰富的接入工具包。大模型方面，除了 OpenAI、Google，也支持国内的通义千问、智谱等模型。同时还支持 LiteLLM、Ollama 等第三方扩展包，为更多模型的接入提供了可能。数据库方面，支持 SQLite、Redis 等。
2. **工具决策**：LangChain 框架原生支持工具决策能力，同时也支持 LLM 原生的 Function Calling。对于复杂工具的选择，LangChain建议先使用embedding进行Top K的预筛选，再将预筛选的结果送入 LLM 进行最终决策。
3. **任务管理**：框架支持 **ReAct (Reasoning + Action) 循环和会话管理**。这个特性能够驱动大模型实现理解、推理、执行、观察的循环任务链条，并支持多轮推理，促使大模型更高效、更精准地完成任务。从实现的功能来看，Agent 和 MCP 有点像，都是对一些现有能力通过API接口进行了整合，建立了 Tool 和 LLM 之间的连接，解决了 LLM 调用 Tool 的问题。但区别在于，Agent 可以独立完成任务，MCP 更像一个聚合接口（USB Hub），所以任务管理能力是两者本质的区别。

在 LangChain 框架中，要实现以上三个能力，具体的代码实现步骤如下：

1. **能力接入**：借助 LangChain 内置的工具包实现 LLM 接入、数据库连接等基础能力。
2. **工具决策**：实现 API 封装、Tool 注册、Tool 查找等功能。
3. **任务管理**：实现 Prompt Template 定义、LLM 交互、LLM Output 处理、Agent 调用等功能。

下面，我将通过一个实际例子来具体说明关键代码实现。

### 二、动手实践：构建一个自然语言访问 SQLite 数据库的 Agent

### 

这个例子中，我将实现一个可以通过自然语言访问 SQLite 数据库的 Agent。用户可以向 Agent 发送自然语言指令，例如“显示数据库中所有的数据库表”、“显示订单表中价格最高的前 5 个订单”，Agent 能够识别这些自然语言，并访问 SQLite 数据库，最终展示查询结果。

项目概述如下：

1. 核心特性

```
## 核心特性### 智能推理与行动- **ReAct循环**: 思考 → 行动 → 观察 → 思考的智能循环- **自然语言理解**: 准确理解复杂的中文查询意图- **多步推理**: 支持需要多个步骤完成的复杂任务- **透明推理**: 完整展示AI的思考和决策过程### 智能工具系统- **向量检索**: 基于语义相似度的智能工具选择- **专业化工具**: 6个专门的数据库操作工具- **动态工具选择**: 根据查询内容自动筛选最相关工具- **工具协作**: 多个工具协同完成复杂任务### 安全与可靠- **多层安全防护**: 代码检查 + 提示规则 + 工具验证- **危险操作拦截**: 自动拒绝DELETE、UPDATE等修改操作- **只读查询**: 确保数据库安全，只支持SELECT查询- **错误处理**: 完善的异常处理和用户友好的错误提示### 高度可配置- **多LLM支持**: DeepSeek、OpenAI等多种大语言模型- **嵌入模型选择**: 支持本地和云端嵌入模型- **灵活配置**: 通过环境变量轻松配置各种参数- **扩展性**: 模块化设计，易于扩展新功能
```

2. 项目架构

```
## 项目架构### 目录结构```langchain-agent/├── langchain_agent/              # 核心代码包│   ├── __init__.py              # 包初始化和公共接口│   ├── agents/                  # Agent实现│   │   ├── __init__.py│   │   └── sql_agent.py         # EnhancedSQLAgent核心实现│   ├── tools/                   # 工具系统│   │   ├── __init__.py│   │   ├── sql_toolkit.py       # 增强SQL工具包│   │   └── database_tools.py    # 6个专业数据库工具│   ├── config/                  # 配置管理│   │   ├── __init__.py│   │   ├── settings.py          # 统一设置管理│   │   ├── llm_config.py        # LLM配置和创建│   │   └── database_config.py   # 数据库连接配置│   └── utils/                   # 工具函数│       ├── __init__.py│       ├── embeddings.py        # 嵌入模型管理│       └── retrieval.py         # 向量检索系统├── examples/                    # 使用示例│   ├── basic_usage.py           # 基础交互式示例│   └── advanced_usage.py        # 高级功能演示├── tests/                       # 测试套件│   ├── test_agents/             # Agent功能测试│   ├── test_tools/              # 工具功能测试│   └── test_config/             # 配置系统测试├── docs/                        # 项目文档├── pyproject.toml              # 项目配置├── requirements.txt            # 生产依赖├── requirements-dev.txt        # 开发依赖└── .env                        # 环境配置```### 核心组件#### EnhancedSQLAgent- **ReAct架构**: 实现思考-行动-观察循环- **安全检查**: 多层防护机制- **工具检索**: 智能选择相关工具- **错误处理**: 完善的异常处理#### 专业工具集1. **database_info** - 数据库概览信息2. **table_list** - 表名列表查询3. **table_schema** - 表结构详细信息4. **sample_data** - 示例数据展示5. **record_count** - 记录数量统计6. **sql_query** - SQL查询执行#### 智能检索系统- **向量检索**: 基于FAISS的语义检索- **嵌入模型**: 支持本地和云端模型- **缓存优化**: 智能模型缓存管理
```

具体实现关键步骤和代码如下：

**1. 实现 LLM 接入**

`llm_config.py`

```
"""LLM配置模块提供统一的LLM配置和创建接口，支持多种LLM提供商。"""from typing import Optional, Dict, Anyfrom dataclasses import dataclassfrom langchain_core.language_models import BaseLanguageModelfrom langchain_deepseek import ChatDeepSeekfrom langchain_agent.config.settings import get_settings
```

```
def create_deepseek_llm(config: Optional[LLMConfig] = None) -> ChatDeepSeek:    """    创建DeepSeek语言模型实例  
    Args:        config: LLM配置，如果为None则使用默认设置  
    Returns:        ChatDeepSeek: DeepSeek聊天模型实例  
    Raises:        ValueError: 当API密钥无效时        Exception: 当创建模型失败时    """    if config is None:        settings = get_settings()        config = LLMConfig(            api_key=settings.deepseek_api_key,            model=settings.llm_model,            temperature=settings.llm_temperature,            max_tokens=settings.llm_max_tokens,            timeout=settings.llm_timeout        )  
    if not config.api_key:        raise ValueError(            "DeepSeek API key is required. Please set DEEPSEEK_API_KEY in .env file.\n"            "Get your API key from: https://platform.deepseek.com/api_keys"        )  
    try:        llm = ChatDeepSeek(            api_key=config.api_key,            model=config.model,            temperature=config.temperature,            max_tokens=config.max_tokens,            timeout=config.timeout,            **config.extra_params        )  
        return llm
```

注：temperature参数强烈建议配置为0。

**2. 实现数据库管理**

**database\_config.py**

```
"""数据库配置模块提供数据库连接和配置管理。"""from typing import Optional, Dict, Anyfrom dataclasses import dataclassfrom pathlib import Pathfrom langchain_community.utilities.sql_database import SQLDatabasefrom langchain_agent.config.settings import get_settings
```

```
def create_database_connection(config: Optional[DatabaseConfig] = None) -> SQLDatabase:    """    创建数据库连接  
    Args:        config: 数据库配置，如果为None则使用默认设置  
    Returns:        SQLDatabase: LangChain数据库连接实例  
    Raises:        ValueError: 当配置无效时        Exception: 当连接失败时    """    if config is None:        settings = get_settings()        config = DatabaseConfig(            database_path=settings.database_path,            database_type=settings.database_type        )  
    config.validate()  
    try:        db = SQLDatabase.from_uri(            config.database_uri,            **config.connection_params        )        return db    except Exception as e:        raise Exception(f"Failed to create database connection: {str(e)}")
```

**3. 实现 API 封装****database\_tools.py**

```
"""数据库工具提供各种数据库操作工具，遵循LangChain工具设计模式。"""from typing import Optional, Dict, Anyfrom langchain_core.tools import BaseToolfrom langchain_community.utilities.sql_database import SQLDatabasefrom pydantic import Field
```

```
class TableSchemaTool(BaseTool):    """表结构查询工具"""    name: str = "table_schema"    description: str = (        "获取指定表的结构信息，包括列名、数据类型等，输入表名"    )  
    db: SQLDatabase = Field(exclude=True)  
    def _run(self, table_name: str) -> str:        """查询表结构"""        try:            # 验证表是否存在            available_tables = self.db.get_usable_table_names()            if table_name not in available_tables:                return f"表 '{table_name}' 不存在。可用的表: {', '.join(available_tables)}"  
            # 获取表结构            table_info = self.db.get_table_info_no_throw([table_name])            return f"表 '{table_name}' 的结构信息:\n{table_info}"  
        except Exception as e:            return f"查询表结构时出错: {str(e)}"
```

**4. 实现 Tool 注册**

通过自然语言描述 API 的功能，并将 API 注册到 Tool 中。

sql\_toolkit.py

```
"""增强的SQL工具包基于LangChain的SQLDatabaseToolkit，提供增强的SQL查询功能。"""from typing import List, Optionalfrom langchain_core.tools import BaseToolfrom langchain_core.language_models import BaseLanguageModelfrom langchain_community.utilities.sql_database import SQLDatabasefrom langchain_community.agent_toolkits.sql.toolkit import SQLDatabaseToolkit
```

```
    def _create_enhanced_tools(self) -> List[BaseTool]:        """创建增强的工具集合"""        tools = [            DatabaseInfoTool(db=self.db),            TableListTool(db=self.db),            TableSchemaTool(db=self.db),            SampleDataTool(db=self.db),            RecordCountTool(db=self.db),            QueryExecutorTool(db=self.db),        ]
```

**5. 实现 Tool 查找**

使用 Vector Store 和 Embedding 对 Tool 的 description 信息进行存储和查找。这里我使用了 **FAISS**作为 Vector Store，**sentence-transformers/all-MiniLM-L6-v2**作为 Embedding 模型，并强制使用了离线模式。Agent 第一次启动时会拉取 Embedding，之后会使用本地缓存。

**注意**：在 LangChain 官方示例中，此步骤使用了 `OpenAIEmbeddings`。该包会实时与 OpenAI 交互，需要 OpenAI API Key 才能工作。本文未采用此方案。

retrieval.py

```
"""检索工具提供工具检索功能，用于根据查询内容选择最相关的工具。"""from typing import List, Callablefrom langchain_core.tools import BaseToolfrom langchain_core.documents import Documentfrom langchain_community.vectorstores import FAISSfrom langchain_agent.utils.embeddings import create_embeddings_with_fallback, check_model_cache
```

```
        # 为每个工具创建文档        docs = [            Document(                page_content=tool.description,                metadata={"index": i, "tool_name": tool.name}            )            for i, tool in enumerate(tools)        ]  
        # 创建FAISS向量存储        vectorstore = FAISS.from_documents(docs, embeddings)  
        def retrieve_tools(query: str) -> List[BaseTool]:            """根据查询使用向量检索选择相关工具"""            try:                # 使用向量检索                relevant_docs = vectorstore.similarity_search(query, k=top_k)                # 根据检索结果返回工具                retrieved_tools = []                for doc in relevant_docs:                    tool_index = doc.metadata["index"]                    if 0 <= tool_index < len(tools):                        retrieved_tools.append(tools[tool_index])                if settings.verbose:                    print(f"🔍 使用向量检索选择工具: {[t.name for t in retrieved_tools]}")                return retrieved_tools if retrieved_tools else tools            except Exception as e:                print(f"Warning: Tool retrieval failed: {e}, returning all tools")                return tools  
        return retrieve_tools
```

embeddings.py

```
        embeddings = HuggingFaceEmbeddings(            model_name=model_name,            model_kwargs=model_kwargs,            encode_kwargs={                'normalize_embeddings': True,                'batch_size': 32,  # 批处理大小            },            cache_folder=cache_dir,  # 指定缓存目录        )
```

**6. 实现 Prompt Template 定义和 LLM 交互**

根据官方示例，编写提示词如下：

```
        # 定义提示模板        template = """你是一个专业的SQL数据库查询助手。你可以使用以下工具来帮助用户查询数据库:{tools}⚠️ 严格安全规则 - 必须遵守:1. 绝对禁止执行或提供任何修改数据的SQL语句2. 禁止的操作包括: DELETE, UPDATE, INSERT, DROP, ALTER, CREATE, TRUNCATE3. 如果用户要求这些操作，必须立即拒绝，不提供任何相关SQL语句4. 只能帮助用户查询数据(SELECT)，不能修改数据5. 当遇到危险请求时，回答格式: "抱歉，我不能执行[操作类型]操作，因为这会修改数据库。我只能帮助您查询数据。"使用以下格式回答:问题: 用户的输入问题思考: 你应该思考要做什么动作: 要采取的动作，应该是 [{tool_names}] 中的一个动作输入: 动作的输入观察结果: 动作的结果... (这个思考/动作/动作输入/观察结果可以重复N次)思考: 我现在知道最终答案了最终答案: 对原始问题的最终答案开始!问题: {input}思考: {agent_scratchpad}"""        # 创建提示模板（使用官方推荐的tools_getter模式）        prompt = CustomPromptTemplate(            template=template,            tools_getter=tools_getter,  # 动态工具检索            input_variables=["input", "intermediate_steps"]        )
```

```

```

**7.实现 LLM Output 处理**

```
        # 检查是否包含最终答案        if "最终答案:" in llm_output:            return AgentFinish(                return_values={"output": llm_output.split("最终答案:")[-1].strip()},                log=llm_output,            )  
        # 解析动作        regex = r"动作:\s*(.*?)\n动作输入:\s*(.*)"        match = re.search(regex, llm_output, re.DOTALL)
```

**8. 实现 Agent 调用**

Agent的调用代码比较简单，但其内部执行流程比较复杂。

```
        # 创建Agent执行器        return AgentExecutor.from_agent_and_tools(            agent=agent,            tools=all_tools,  # 提供所有工具，但通过tools_getter动态选择            verbose=verbose,            max_iterations=self.settings.max_iterations        )  
 
```

```
                   # 执行查询（工具选择在提示模板中动态进行）            result = self.agent_executor.run(question)
```

Agent 启动后，在其`run`方法中，LangChain 框架的执行引擎实现了 LLM 调用、工具执行管理和状态管理等功能。

`run()`方法调用后，会启动一个完整的 **ReAct (Reasoning + Action) 循环**，具体步骤为：理解用户问题、检索工具、推理下一步行动、执行工具、观察结果、重复上述操作。最大推理次数由参数 max\_iterations 决定。

执行流程图如下：

```
graph TD    A[query方法调用] --> B[AgentExecutor.run启动]    B --> C[初始化intermediate_steps = []]    C --> D[开始推理循环]    D --> E[工具检索阶段]    E --> F[向量检索相关工具]    F --> G[构建提示模板]    G --> H[LLM推理阶段]    H --> I[发送提示给DeepSeek]    I --> J[解析LLM输出]    J --> K{输出类型判断}    K -->|AgentFinish| L[返回最终答案]    K -->|AgentAction| M[执行工具]    M --> N[调用SQL工具]    N --> O[更新intermediate_steps]    O --> D    L --> P[返回给用户]
```

```
假设用户的问题是“显示orders表中最贵的订单记录”，那么 LangChain 框架的执行步骤如下：
```

```

```

**# 第 1 轮循环**

**1. 动态工具检索**

```
# 在CustomPromptTemplate.format()中调用tools = self.tools_getter("显示orders表中最贵的订单记录")# 输出: 使用向量检索选择工具: ['table_schema', 'sql_query', 'sample_data', 'database_info', 'record_count', 'table_list']
```

**2. 生成提示词模板**

```
prompt = """你是一个专业的SQL数据库查询助手。你可以使用以下工具来帮助用户查询数据库:  
table_schema: 获取指定表的结构信息，包括列名、数据类型等，输入表名sql_query: 执行SQL查询获取具体数据记录，如查找最贵订单、价格最高商品、特定条件的记录等。用于复杂查询：最大值、最小值、排序、筛选、聚合统计、条件查询。输入完整的SQL SELECT语句。sample_data: 获取指定表的前5条示例数据，用于了解数据内容，输入表名database_info: 获取数据库的基本信息，包括表列表等record_count: 统计指定表中的记录总数，输入表名table_list: 列出数据库中的所有表名，用于了解数据库结构  
⚠️ 严格安全规则 - 必须遵守:1. 绝对禁止执行或提供任何修改数据的SQL语句2. 禁止的操作包括: DELETE, UPDATE, INSERT, DROP, ALTER, CREATE, TRUNCATE3. 如果用户要求这些操作，必须立即拒绝，不提供任何相关SQL语句4. 只能帮助用户查询数据(SELECT)，不能修改数据5. 当遇到危险请求时，回答格式: "抱歉，我不能执行[操作类型]操作，因为这会修改数据库。我只能帮助您查询数据。"  
使用以下格式回答:  
问题: 用户的输入问题思考: 你应该思考要做什么动作: 要采取的动作，应该是 [table_schema, sql_query, sample_data, database_info, record_count, table_list] 中的一个动作输入: 动作的输入观察结果: 动作的结果... (这个思考/动作/动作输入/观察结果可以重复N次)思考: 我现在知道最终答案了最终答案: 对原始问题的最终答案  
开始!  
问题: 显示orders表中最贵的订单记录思考: """
```

**3. LLM 推理**

```
# LLM分析提示并生成回应llm_output = """用户想要找到orders表中最贵的订单记录。我需要先了解orders表的结构，特别是金额相关的字段，然后执行查询找出最贵的记录。  
动作: table_schema动作输入: orders"""
```

**4. 解析输出**

```
# CustomOutputParser.parse()解析LLM输出action = AgentAction(    tool="table_schema",    tool_input="orders",    log="用户想要找到orders表中最贵的订单记录...")
```

**5. 执行工具**

```
# 查找table_schema工具并执行tool = self.get_tool("table_schema")observation = tool.run("orders")  
# 工具返回结果observation = """表 'orders' 的结构信息:  
CREATE TABLE orders (        id INTEGER,         coupon_code VARCHAR(255),         create_time TIMESTAMP NOT NULL,         logistics_company VARCHAR(255),         note VARCHAR(255),         pay_time TIMESTAMP,         payment_method VARCHAR(255),         payment_transaction_id VARCHAR(255),         product_id BIGINT NOT NULL,         quantity INTEGER NOT NULL,         receiver_address VARCHAR(255) NOT NULL,         receiver_name VARCHAR(255) NOT NULL,         receiver_phone VARCHAR(255) NOT NULL,         specification VARCHAR(255),         status VARCHAR(255) NOT NULL,         total_amount NUMERIC(38, 2) NOT NULL,         tracking_number VARCHAR(255),         update_time TIMESTAMP NOT NULL,         user_id BIGINT NOT NULL,         PRIMARY KEY (id),         CHECK (status in ('PENDING_PAYMENT','PENDING_SHIPMENT','PENDING_RECEIPT','COMPLETED','CANCELLED')))  
/*3 rows from orders table:id      coupon_code     create_time     logistics_company       note    pay_time        payment_method       payment_transaction_id  product_id      quantity        receiver_address        receiver_name receiver_phone  specification   status  total_amount    tracking_number update_time     user_id2       None    2024-11-26 07:57:55     None    None    None    None    None    2       4       上海市浦东新区张江高科技园区5号     张先生6 13812348152     None    CANCELLED       716.00  None    2024-04-06 15:37:55  73       None    2024-06-03 22:06:55     None    None    None    None    None    3       1       上海市浦东新区张江高科技园区48号    张先生15        13812340741     None    CANCELLED       413.00  None2024-04-08 12:41:55      894       None    2024-10-13 20:49:55     None    None    None    None    None    4       2       上海市浦东新区张江高科技园区37号    张先生4 13812342197     None    PENDING_RECEIPT 384.00  None    2024-04-18 16:51:55  33*/"""
```

**6. 更新状态**

```
intermediate_steps = [(action, observation)]
```

# 第 2 轮循环

**1. 再次动态检索工具**

```
# 再次调用工具检索器（基于相同的查询）tools = self.tools_getter("显示orders表中最贵的订单记录")# 输出: 使用向量检索选择工具: ['table_schema', 'sql_query', 'sample_data', 'database_info', 'record_count', 'table_list']
```

**2. 生成新的提示词模板**

```
prompt = """你是一个专业的SQL数据库查询助手。你可以使用以下工具来帮助用户查询数据库:  
[工具列表相同...]  
问题: 显示orders表中最贵的订单记录思考: 用户想要找到orders表中最贵的订单记录。我需要先了解orders表的结构，特别是金额相关的字段，然后执行查询找出最贵的记录。  
动作: table_schema动作输入: orders  
观察结果: 表 'orders' 的结构信息:[完整的表结构信息...]  
思考: """
```

**3. LLM 推理（基于新信息）**

```
llm_output = """现在我已经了解了orders表的结构，看到有total_amount字段表示订单总金额。要查询最贵的订单记录，我需要按total_amount降序排序并取第一条记录。  
动作: sql_query动作输入: SELECT * FROM orders ORDER BY total_amount DESC LIMIT 1"""
```

**4. 解析输出**

```
action = AgentAction(    tool="sql_query",    tool_input="SELECT * FROM orders ORDER BY total_amount DESC LIMIT 1",    log="现在我已经了解了orders表的结构...")
```

**5. 执行 SQL 查询工具**

```
tool = self.get_tool("sql_query")observation = tool.run("SELECT * FROM orders ORDER BY total_amount DESC LIMIT 1")  
# SQL查询结果observation = """[(744, None, datetime.datetime(2024, 4, 15, 9, 32, 55), None, None, None, None, None, 744, 1, '上海市浦东新区张江高科技园区744号', '张先生744', '13812340744', None, 'PENDING_RECEIPT', Decimal('997.00'), None, datetime.datetime(2024, 4, 15, 9, 32, 55), 744)]"""
```

**6. 更新状态**

```
intermediate_steps = [    (action1, observation1),  # table_schema的结果    (action2, observation2)   # sql_query的结果]
```

******# 第 3 轮循环******

**1. 再次动态工具检索**

```
tools = self.tools_getter("显示orders表中最贵的订单记录")# 输出: 使用向量检索选择工具: [相同的工具列表]
```

**2. 生成最终提示词模板**

```
prompt = """[工具列表...]  
问题: 显示orders表中最贵的订单记录思考: [第一次的思考和行动...]观察结果: [表结构信息...]思考: [第二次的思考和行动...]观察结果: [SQL查询结果...]思考: """
```

**3. LLM 最终推理**

```
llm_output = """我现在已经获得了orders表中最贵的订单记录。查询结果显示ID为744的订单，总金额为997元，是最贵的订单。我现在知道最终答案了。  
最终答案: orders表中最贵的订单记录是ID为744的订单，总金额为997元。详细信息如下：- 订单ID: 744- 总金额: 997.00元- 创建时间: 2024-04-15 09:32:55- 收货人: 张先生744- 收货地址: 上海市浦东新区张江高科技园区744号- 收货电话: 13812340744- 订单状态: PENDING_RECEIPT (待收货)- 商品ID: 744- 数量: 1"""
```

**4. 解析为最终答案**

```
final_result = AgentFinish(    return_values={"output": "orders表中最贵的订单记录是ID为744的订单..."},    log="我现在已经获得了orders表中最贵的订单记录...")
```

**5. 返回最终结果**

```
# AgentExecutor检测到AgentFinish，结束循环return final_result.return_values["output"]
```

**以上为一个完整的ReAct执行过程，包括了思考、行动、观察三个关键动作，并由Agent控制迭代次数，可以动态进行Tool检索。**

### **三、Agent执行效果**

### 

这个Agent基于langchain framework开发，实现了自然语言操作sqlite数据库的功能。

项目代码将近3000行，90%由Augment贡献，10%由我编写。

写完我才发现，项目规模有点超过了我的预期。langchain framework用起来不太难，但其中涉及到的知识点比较多，大家可以借助AI的代码解释能力加深理解。

后续我可能会挑几个知识点进行深入剖析，比如ReAct、embedding等。