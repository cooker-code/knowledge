---
title: Java Boy转型到Agent开发-大纲篇
author: 方丈的寺院
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI4NjI3MDc1NA==&mid=2247484282&idx=1&sn=0670b410217cb8cb8516f174fe433664&chksm=ea621d124ed79449da81a08a3de728acf73e1df43c9b50ee313fd2aeb99c785368fade943423&mpshare=1&scene=24&srcid=0106QSNGIujkvJsQY2kgl3mc&sharer_shareinfo=d0f4cd084fada3a838c78286e4a9b93b&sharer_shareinfo_first=d0f4cd084fada3a838c78286e4a9b93b#rd
---

一、 Agent开发整体大纲

主要包括6层

1. 用户交互层（包括web界面，cli,api)，没太多东西。

2. Agent 核心层

主要有控制器（ReAct）、推理引擎（Chain-of-Thought)、工具调用（function call, mcp)、记忆管理、prompt、上下文。

其实记忆管理、prompt、上下文都可以归类到context，只是做了进一步细分。

3. 工具层

各种原子化、外调的能力

4. LLM 服务层

调各种大模型

5. 数据存储层

比较常用的是向量数据库（RAG的存储）、缓存系统（输入缓存、结果缓存）

向量数据库对于java boy来说是个新知识点，要好好学。

6. agent平台能力保障

包括性能监控、告警、token调用成本等。

二、Agent开发工具

类Dify工具

Dify 是一个用于构建 AI 工作流的开源平台。通过在可视化画布上编排 AI 模型、连接数据源、数据存储层，定义处理流程，转化为可运行的软件。输出结果可以是现在通用的chat方式、也可以集成到各种企业软件，钉钉、飞书等。

https://docs.dify.ai/zh/use-dify/getting-started/introduction

这块一般由公司的AI工程基础团队搞定，部署一个内部版本的dify,这样其他的java boy就可以在这个基础上拖拖拽拽，搞起来了。

Agent核心层

把dify中的各个节点拆出来就是agent的核心层。主要控制节点有用户输入、LLM、知识检索、工具、输出。有些同学刚开始开发Agent时，发现能力都外调了，agent就是一个流程编排，和传统的流程编排也没什么区别啊？

# Dify vs 传统流程引擎编排工具对比分析

### **本质差异**

* **Dify**：AI 应用的**编排平台**，专注于 AI 能力组合
* **传统流程引擎**：业务流程的**执行引擎**，专注于任务调度

对于java boy来说，很容易把各个业务能力都封装到各个tool里面，然后用流程引擎串起来，而忽略了发挥AI能力。

另外一方面，对于非技术出身的搞大模型的同学呢，又全都依赖于AI能力，把一些结构化的数据处理、确定性的一些业务流程也用大模型来进行处理，导致上下文过长、输出结果不稳定。为了保障结果的确定性呢，开始疯狂加分支了。

两者需要结合一下。

#

# 传统流程引擎（如 Camunda、Activiti）主要用于**业务流程自动化**和**任务调度编排**。

### 1. **设计目标与定位**

| 维度 | Dify | 传统流程引擎 |
| --- | --- | --- |
| **核心目标** | AI 应用构建与编排 | 业务流程自动化 |
| **主要场景** | AI 对话、知识问答、内容生成 | 审批流程、数据处理、任务调度 |
| **技术栈** | LLM、向量数据库、Embedding | 关系数据库、消息队列、规则引擎 |

### 2. **节点类型差异**

#### Dify 的核心节点

* **用户输入节点**：接收用户查询、上下文信息
* **LLM 节点**：调用大语言模型（GPT、Claude 等）
* **知识检索节点**：RAG（检索增强生成）、向量搜索
* **工具节点**：API 调用、函数执行、外部服务集成
* **输出节点**：格式化输出、多模态输出
* **流程控制**：if-else、循环、条件分支

#### 传统流程引擎的核心节点

* **任务节点**：人工任务、自动任务、服务任务
* **网关节点**：并行网关、排他网关、包容网关
* **事件节点**：开始事件、结束事件、中间事件
* **数据节点**：数据对象、数据存储
* **规则引擎**：业务规则判断、条件分支
* **集成节点**：数据库操作、消息发送、HTTP 调用

### 3. **执行模式差异**

| 特性 | Dify | 传统流程引擎 |
| --- | --- | --- |
| **执行确定性** | **非确定性** （AI 输出可能不同） | **确定性** （相同输入相同输出） |
| **执行方式** | **流式响应** 异步处理 | **同步/异步** 、批量处理 |
| **状态管理** | **对话状态** 上下文记忆 | **流程实例状态** 任务状态 |
| **性能优化** | **Token 优化** 缓存策略 | **并发控制** 资源调度 |

### 4. **数据处理方式**

#### Dify 的数据流

```
用户输入 → 上下文理解 → 知识检索 → LLM 推理 → 工具调用 → 结果生成
```

* **非结构化数据**：文本、图像、语音
* **语义理解**：向量化、相似度搜索
* **动态生成**：内容生成、摘要提取
* **流式输出**：Token 流式返回

#### 传统流程引擎的数据流

```
输入数据 → 数据验证 → 规则判断 → 任务执行 → 数据转换 → 结果输出
```

* **结构化数据**：数据库记录、JSON、XML
* **数据验证**：格式校验、业务规则校验
* **确定性处理**：数据转换、计算、存储
* **批量处理**：批量导入、批量计算

### 5. **集成对象差异**

| 集成类型 | Dify | 传统流程引擎 |
| --- | --- | --- |
| **AI 服务** | ✅ LLM API、Embedding API | ❌ 通常不涉及 |
| **知识库** | ✅ 向量数据库、文档库 | ❌ 关系数据库为主 |
| **业务系统** | ⚠️ 通过 API 集成 | ✅ 深度集成（ERP、CRM） |

### 6. **状态管理与持久化**

#### Dify

* **对话状态**：多轮对话上下文
* **记忆管理**：短期记忆、长期记忆
* **向量存储**：文档索引、语义检索
* **缓存机制**：Token 缓存、结果缓存

#### 传统流程引擎

* **流程实例状态**：运行中、已完成、已终止
* **任务状态**：待处理、处理中、已完成
* **数据持久化**：流程数据、业务数据
* **历史记录**：流程历史、审计日志

### 7. **错误处理与容错**

#### Dify

* **LLM 调用失败**：重试、降级到备用模型
* **Token 超限**：上下文压缩、分块处理
* **知识检索失败**：降级到直接回答
* **工具调用失败**：错误提示、建议替代方案

#### 传统流程引擎

* **任务执行失败**：重试、补偿事务
* **数据异常**：回滚、数据修复
* **系统故障**：流程恢复、状态恢复
* **超时处理**：超时告警、自动终止

### 8. **监控与可观测性**

| 监控指标 | Dify | 传统流程引擎 |
| --- | --- | --- |
| **性能指标** | Token 使用量、响应时间、成本 | 任务执行时间、吞吐量 |
| **质量指标** | 回答准确性、用户满意度 | 流程完成率、错误率 |
| **业务指标** | 对话轮次、知识检索命中率 | 流程耗时、任务积压 |
| **成本指标** | API 调用成本、Token 成本 | 资源消耗、系统负载 |

---

## 适用场景对比

### Dify 适合的场景

1. **AI 对话应用**：智能客服、AI 助手
2. **知识问答系统**：企业知识库、文档问答
3. **内容生成**：文章生成、代码生成、摘要提取
4. **数据分析助手**：自然语言查询、数据洞察
5. **多模态应用**：图像理解、语音交互

### 传统流程引擎适合的场景

1. **审批流程**：请假审批、报销审批、合同审批
2. **数据处理**：ETL 流程、数据清洗、报表生成
3. **任务调度**：定时任务、批量作业、工作流调度
4. **系统集成**：系统间数据同步、业务流程打通
5. **规则引擎**：业务规则自动化、决策流程

三、工具层

# Agent核心层架构总览

```
Agent核心层├── Agent控制器 (Agent Controller)│   └── 类似：Spring MVC的DispatcherServlet + 策略模式├── 推理引擎 (Reasoning Engine)│   └── 类似：规则引擎 + 决策树├── 工具调用器 (Tool Executor)│   └── 类似：策略模式 + 工厂模式 + 责任链模式├── 记忆管理器 (Memory Manager)│   └── 类似：缓存系统 + 会话管理├── Prompt管理器 (Prompt Manager)│   └── 类似：模板引擎 + 配置管理└── 上下文管理器 (Context Manager)    └── 类似：ThreadLocal + 请求上下文管理
```

---

## 1. Agent控制器 (Agent Controller)

### 概念理解

**Java类比**：

* 类似 `DispatcherServlet`：统一接收请求，分发到不同的处理器
* 类似 `Controller`：协调各个服务组件
* 类似 `策略模式`：根据任务类型选择不同的执行策略

### 核心功能

1. **任务分解**：将复杂任务分解为子任务
2. **决策协调**：决定执行顺序和策略
3. **流程控制**：管理整个Agent的执行流程
4. **错误处理**：处理执行过程中的异常

### 

#### ReAct模式（Reasoning + Acting）while循环执行

```
思考 → 行动 → 观察 → 思考 → 行动 → ...
```

### 💻 代码示例

#### Java类比代码（伪代码）

```
@RestControllerpublic class AgentController {  
    @Autowired    private ReasoningEngine reasoningEngine;  
    @Autowired    private ToolExecutor toolExecutor;  
    @Autowired    private MemoryManager memoryManager;  
    // 类似@RequestMapping    @PostMapping("/agent/execute")    public AgentResponse execute(@RequestBody AgentRequest request) {        // 1. 任务分解（类似任务调度）        List<Task> tasks = decomposeTask(request.getInput());  
        // 2. 决策协调（类似策略模式）        ExecutionStrategy strategy = selectStrategy(tasks);  
        // 3. 执行流程（类似责任链）        AgentResponse response = strategy.execute(tasks);  
        return response;    }  
    private List<Task> decomposeTask(String input) {        // 任务分解逻辑        // 类似：将复杂业务拆分为多个Service调用    }}
```

## 2. 推理引擎 (Reasoning Engine)

### 概念理解

**Java类比**：

* 类似 `规则引擎`（Drools、Easy Rules）：根据规则进行推理
* 类似 `决策树`：根据条件进行分支判断
* 类似 `状态机`：管理推理状态转换

### 核心功能

1. **思考过程**：分析问题、生成推理步骤
2. **决策制定**：决定下一步行动
3. **逻辑推理**：进行逻辑判断和推理
4. **自我反思**：评估当前状态，调整策略

### 🏗️ 推理模式

#### Chain-of-Thought (CoT)

```
问题 → 思考步骤1 → 思考步骤2 → ... → 结论
```

### 代码示例

#### Java类比代码（伪代码）

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(line// 类似规则引擎public class ReasoningEngine {  
    private RuleEngine ruleEngine;  // 类似Drools  
    public ReasoningResult reason(String question, Context context) {        // 1. 构建推理规则（类似规则定义）        List<Rule> rules = buildReasoningRules(question);  
        // 2. 执行推理（类似fireRules）        ReasoningResult result = ruleEngine.execute(rules, context);  
        // 3. 记录推理过程        logReasoningSteps(result);  
        return result;    }  
    private List<Rule> buildReasoningRules(String question) {        // 根据问题构建推理规则        // 类似：根据业务场景构建Drools规则    }}
```

### 

---

## 3. 工具调用器 (Tool Executor)

### 概念理解

**Java类比**：

* 类似 `策略模式`：不同工具使用不同策略
* 类似 `工厂模式`：根据工具类型创建执行器
* 类似 `责任链模式`：工具调用的链式处理
* 类似 `适配器模式`：统一不同工具的接口

### 🎯 核心功能

1. **工具注册**：注册和管理可用工具
2. **工具选择**：根据任务选择合适的工具
3. **工具执行**：执行工具并获取结果
4. **错误处理**：处理工具执行失败的情况

### 🏗️ 工具类型

* **计算工具**：数学运算、统计分析
* **搜索工具**：Web搜索、API调用
* **文件工具**：文件读写、数据处理
* **数据库工具**：SQL查询、数据操作
* **代码工具**：代码执行、代码生成

### 💻 代码示例

#### Java类比代码（伪代码）

```
public interface Tool {    String execute(String input);    String getName();    String getDescription();}  
// 工具实现（类似Service实现）@Componentpublic class CalculatorTool implements Tool {  
    @Override    public String execute(String input) {        // 执行计算逻辑        return calculate(input);    }  
    @Override    public String getName() {        return "calculator";    }}  
// 工具执行器（类似策略管理器）@Servicepublic class ToolExecutor {  
    @Autowired    private List<Tool> tools;  // 自动注入所有工具  
    private Map<String, Tool> toolMap;  
    @PostConstruct    public void init() {        // 构建工具映射（类似工厂模式）        toolMap = tools.stream()            .collect(Collectors.toMap(                Tool::getName,                tool -> tool            ));    }  
    public ToolResult execute(String toolName, String input) {        Tool tool = toolMap.get(toolName);        if (tool == null) {            throw new ToolNotFoundException(toolName);        }  
        try {            String result = tool.execute(input);            return ToolResult.success(result);        } catch (Exception e) {            return ToolResult.failure(e.getMessage());        }    }}
```

### 4. 记忆管理器 (Memory Manager)

### 概念理解

**Java类比**：

* 类似 `Session管理`：管理用户会话状态
* 类似 `缓存系统`（Redis、Caffeine）：存储和检索数据
* 类似 `ThreadLocal`：管理线程本地变量
* 类似 `状态模式`：管理不同状态的记忆

### 🎯 核心功能

1. **短期记忆**：存储当前对话的上下文
2. **长期记忆**：存储重要的历史信息
3. **记忆检索**：根据上下文检索相关记忆
4. **记忆更新**：更新和删除记忆

### 记忆类型

#### 短期记忆（Short-term Memory）

* **ConversationBufferMemory**：保存完整对话历史
* **ConversationSummaryMemory**：保存对话摘要
* **ConversationBufferWindowMemory**：保存最近N轮对话

#### 长期记忆（Long-term Memory）

* **向量数据库记忆**：使用向量搜索检索相关记忆
* **数据库记忆**：存储在关系数据库中
* **文件记忆**：存储在文件中

### 💻 代码示例

#### Python实现（LangChain）

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(linefrom langchain.memory import ConversationBufferMemoryfrom langchain.memory import ConversationSummaryMemoryfrom langchain.memory import VectorStoreRetrieverMemoryfrom langchain.vectorstores import Chromafrom langchain.embeddings import OpenAIEmbeddings  
class MemoryManager:    """记忆管理器 - 类似Java的Session管理 + 缓存系统"""  
    def __init__(self):        # 短期记忆（类似Session）        self.short_term_memory = ConversationBufferMemory(            return_messages=True,            memory_key="chat_history"        )  
        # 长期记忆（类似Redis缓存）        self.long_term_memory = None        self.vector_store = None  
    def initialize_long_term_memory(self):        """初始化长期记忆 - 类似初始化缓存系统"""        embeddings = OpenAIEmbeddings()        self.vector_store = Chroma(            embedding_function=embeddings,            persist_directory="./memory_db"        )  
        self.long_term_memory = VectorStoreRetrieverMemory(            retriever=self.vector_store.as_retriever(),            memory_key="long_term_history"        )  
    def save_short_term(self, user_input: str, ai_response: str):        """保存短期记忆 - 类似Session.setAttribute"""        self.short_term_memory.save_context(            {"input": user_input},            {"output": ai_response}        )  
    def save_long_term(self, key: str, value: str):        """保存长期记忆 - 类似Redis.set"""        if self.long_term_memory:            self.long_term_memory.save_context(                {"input": key},                {"output": value}            )  
    def retrieve_memories(self, query: str, k: int = 5):        """检索记忆 - 类似缓存查询"""        memories = []  
        # 检索短期记忆（类似从Session获取）        short_term = self.short_term_memory.load_memory_variables({})        if short_term:            memories.extend(short_term.get("chat_history", []))  
        # 检索长期记忆（类似从Redis查询）        if self.long_term_memory:            long_term = self.long_term_memory.load_memory_variables({"input": query})            if long_term:                memories.extend(long_term.get("long_term_history", []))  
        return memories[:k]  
    def clear_short_term(self):        """清空短期记忆 - 类似Session.invalidate"""        self.short_term_memory.clear()  
    def update_memory(self, key: str, value: str):        """更新记忆 - 类似缓存更新"""        if self.long_term_memory:            self.long_term_memory.save_context(                {"input": key},                {"output": value}            )  
    def delete_memory(self, key: str):        """删除记忆 - 类似缓存删除"""        if self.vector_store:            # 删除向量数据库中的记忆            # 实现删除逻辑            pass  
# 使用示例memory_manager = MemoryManager()memory_manager.initialize_long_term_memory()  
# 保存短期记忆memory_manager.save_short_term("用户：你好", "AI：你好，有什么可以帮助你的？")  
# 保存长期记忆memory_manager.save_long_term("用户偏好", "喜欢简洁的回答")  
# 检索记忆memories = memory_manager.retrieve_memories("用户偏好")print(memories)
```

### 

## 5. Prompt管理器 (Prompt Manager)

### 概念理解

**Java类比**：

* 类似 `模板引擎`（Thymeleaf、FreeMarker）：管理模板
* 类似 `配置管理`（Spring Config）：管理配置
* 类似 `策略模式`：不同场景使用不同Prompt
* 类似 `建造者模式`：构建复杂的Prompt

### 核心功能

1. **Prompt模板管理**：存储和管理Prompt模板
2. **Prompt构建**：根据上下文构建Prompt
3. **Prompt优化**：优化Prompt以提高效果
4. **Prompt版本管理**：管理不同版本的Prompt

### Prompt类型

* **Zero-shot Prompt**：零样本提示
* **Few-shot Prompt**：少样本提示
* **Chain-of-Thought Prompt**：思维链提示
* **Role-based Prompt**：角色扮演提示

### 代码示例

#### Python实现（LangChain）

```
ounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(lineounter(linefrom langchain.prompts import PromptTemplatefrom langchain.prompts import ChatPromptTemplatefrom typing import Dict, Anyimport json  
class PromptManager:    """Prompt管理器 - 类似Java的模板引擎 + 配置管理"""  
    def __init__(self):        self.templates: Dict[str, PromptTemplate] = {}  # 模板注册表        self.prompt_configs: Dict[str, Dict] = {}  # Prompt配置  
    def register_template(self, name: str, template: str, input_variables: list):        """注册Prompt模板 - 类似注册Thymeleaf模板"""        prompt_template = PromptTemplate(            input_variables=input_variables,            template=template        )        self.templates[name] = prompt_template        print(f"Prompt模板已注册: {name}")  
    def build_prompt(self, template_name: str, variables: Dict[str, Any]) -> str:        """构建Prompt - 类似模板引擎渲染"""        if template_name not in self.templates:            raise ValueError(f"模板不存在: {template_name}")  
        template = self.templates[template_name]        return template.format(**variables)  
    def load_prompt_config(self, config_path: str):        """加载Prompt配置 - 类似加载Spring配置"""        with open(config_path, 'r', encoding='utf-8') as f:            configs = json.load(f)  
        for name, config in configs.items():            self.prompt_configs[name] = config            self.register_template(                name=name,                template=config['template'],                input_variables=config['input_variables']            )  
    def get_prompt(self, scenario: str, context: Dict[str, Any]) -> str:        """获取Prompt - 类似根据场景获取模板"""        # 根据场景选择模板        template_name = self._select_template(scenario)  
        # 构建Prompt        prompt = self.build_prompt(template_name, context)  
        # 优化Prompt（可选）        optimized_prompt = self._optimize_prompt(prompt, context)  
        return optimized_prompt  
    def _select_template(self, scenario: str) -> str:        """选择模板 - 类似策略选择"""        template_mapping = {            "qa": "qa_template",            "summarization": "summary_template",            "code_generation": "code_template",            "analysis": "analysis_template"        }        return template_mapping.get(scenario, "default_template")  
    def _optimize_prompt(self, prompt: str, context: Dict) -> str:        """优化Prompt - 类似模板优化"""        # 可以添加Prompt优化逻辑        # 例如：压缩、添加示例、调整格式等        return prompt  
# Prompt模板定义示例qa_template = """你是一个专业的AI助手。请根据以下上下文回答问题。  
上下文：{context}  
问题：{question}  
请提供准确、详细的回答："""  
few_shot_template = """你是一个数据分析专家。请根据以下示例进行分析。  
示例1：输入：{example1_input}输出：{example1_output}  
示例2：输入：{example2_input}输出：{example2_output}  
现在请分析：输入：{user_input}输出："""  
# 使用示例prompt_manager = PromptManager()  
# 注册模板（类似配置模板）prompt_manager.register_template(    name="qa_template",    template=qa_template,    input_variables=["context", "question"])  
prompt_manager.register_template(    name="few_shot_template",    template=few_shot_template,    input_variables=["example1_input", "example1_output",                      "example2_input", "example2_output", "user_input"])  
# 构建Prompt（类似模板渲染）prompt = prompt_manager.build_prompt(    "qa_template",    {        "context": "Python是一种编程语言",        "question": "Python是什么？"    })print(prompt)
```

### 6. 上下文管理器 (Context Manager)

### 概念理解

**Java类比**：

* 类似 `ThreadLocal`：管理线程本地变量
* 类似 `RequestContext`：管理请求上下文
* 类似 `Spring的@Scope`：管理作用域
* 类似 `缓存管理`：管理Token使用和成本

### 核心功能

1. **上下文收集**：收集对话上下文信息
2. **Token管理**：管理Token使用和限制
3. **上下文压缩**：压缩过长的上下文
4. **成本优化**：优化API调用成本

### 上下文类型

* **对话上下文**：当前对话的历史
* **知识上下文**：从知识库检索的相关信息
* **工具上下文**：工具执行的结果
* **系统上下文**：系统配置和状态

### 代码示例

#### Python实现

```
from langchain.schema import BaseMessagefrom typing import List, Dictimport tiktoken  
class ContextManager:    """上下文管理器 - 类似Java的ThreadLocal + 请求上下文"""  
    def __init__(self, max_tokens: int = 4000, model: str = "gpt-3.5-turbo"):        self.max_tokens = max_tokens        self.model = model        self.encoding = tiktoken.encoding_for_model(model)        self.context_history: List[BaseMessage] = []  
    def add_context(self, message: BaseMessage):        """添加上下文 - 类似ThreadLocal.set"""        self.context_history.append(message)  
    def get_context(self) -> List[BaseMessage]:        """获取上下文 - 类似ThreadLocal.get"""        return self.context_history  
    def count_tokens(self, text: str) -> int:        """计算Token数量 - 类似计算资源消耗"""        return len(self.encoding.encode(text))  
    def manage_context(self, new_message: str) -> List[BaseMessage]:        """管理上下文 - 类似上下文压缩和优化"""        # 1. 添加新消息        new_tokens = self.count_tokens(new_message)  
        # 2. 计算当前上下文Token数        current_tokens = sum(            self.count_tokens(msg.content)             for msg in self.context_history        )  
        # 3. 如果超过限制，进行压缩        if current_tokens + new_tokens > self.max_tokens:            self.context_history = self._compress_context()  
        return self.context_history  
    def _compress_context(self) -> List[BaseMessage]:        """压缩上下文 - 类似缓存淘汰策略"""        # 策略1：保留最近的N条消息        # 策略2：保留最重要的消息        # 策略3：生成摘要  
        # 简单实现：保留最近的消息        max_messages = 10        if len(self.context_history) > max_messages:            # 保留最近的消息            return self.context_history[-max_messages:]  
        return self.context_history  
    def clear_context(self):        """清空上下文 - 类似ThreadLocal.remove"""        self.context_history.clear()  
    def get_context_summary(self) -> Dict:        """获取上下文摘要 - 类似获取上下文统计"""        total_tokens = sum(            self.count_tokens(msg.content)             for msg in self.context_history        )  
        return {            "message_count": len(self.context_history),            "total_tokens": total_tokens,            "remaining_tokens": self.max_tokens - total_tokens,            "utilization": total_tokens / self.max_tokens        }  
# 使用示例context_manager = ContextManager(max_tokens=4000)  
# 添加上下文context_manager.add_context(    BaseMessage(content="用户：你好", type="human"))  
# 管理上下文context = context_manager.manage_context("AI：你好，有什么可以帮助你的？")  
# 获取摘要summary = context_manager.get_context_summary()print(summary)
```

### 上下文管理流程

```
上下文收集（Context Collection）  ├─ 对话历史  ├─ 知识检索结果  └─ 工具执行结果  ↓Token计算（Token Counting）  ├─ 计算当前Token数  ├─ 预测新增Token数  └─ 检查是否超限  ↓上下文管理（Context Management）  ├─ 如果超限：压缩上下文  ├─ 保留重要信息  └─ 删除冗余信息  ↓返回优化后的上下文
```

---

## 模块间协作

```
用户输入  ↓Agent控制器（任务分解、决策协调）  ↓推理引擎（思考、推理）  ↓Prompt管理器（构建Prompt）  ↓上下文管理器（管理上下文、Token优化）  ↓LLM API调用  ↓记忆管理器（保存对话历史）  ↓工具调用器（执行工具）  ↓结果整合  ↓返回用户                                                          
```

## 四、向量数据库

## RAG&向量数据库非常重要，下篇单独学习。