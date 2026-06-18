> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020602_VibeCode/020602_核心知识点/VibeCode边界与交付判断准则|VibeCode边界与交付判断准则]]
---
title: Vibe Engineering 实战第二弹:用 AI 设计 Dify Workflow 核心接口
author: 雷一言说
date:
url: https://mp.weixin.qq.com/s?__biz=MzU5NTkwMzg2Ng==&mid=2247485719&idx=1&sn=a0c50169f75c0e8b6cf9bf006fd99047&chksm=ff9c8831e19b0ea0d6fb311c0121bbb2a965b77da17ba7c6aaa90d91d24446cbbad9513d82e2&mpshare=1&scene=24&srcid=1231xUi921hTDAdx2gKZr0La&sharer_shareinfo=9d085a640e6744c9d8984ee02a0e9b39&sharer_shareinfo_first=9d085a640e6744c9d8984ee02a0e9b39#rd
---

# Vibe Engineering 实战第二弹:用 AI 设计 Dify Workflow 核心接口

大家好，我是 Radar哥。

上一篇文章我们完成了 Dify Workflow DSL 的分析，并定义了基础实体类（`Workflow`、`Graph`、`Node`、`Edge`）。

今天进入第二篇，**设计 Workflow 的执行器和枚举**。

这是整个项目的"骨架"，设计得好不好，直接决定了后续代码的可扩展性和可维护性。

---

## 01. 设计接口和枚举

但在 Vibe Engineering 的思路里，我们要先 **"定义契约"**，再 **"实现细节"**。

### **核心设计目标**

1. 1. **NodeType 枚举**：定义所有支持的节点类型（LLM、HTTP、IF-ELSE、CODE 等）
2. 2. **NodeExecutor 接口**：定义节点执行的标准契约（所有节点都要实现这个接口）
3. 3. **WorkflowContext 类**：定义工作流执行时的上下文（变量存储、节点间数据传递）

---

## 02. 应用 Vibe Engineering：给 AI 足够的上下文

我继续使用上一篇的 `plan.md`，让 AI 知道我们现在在哪个阶段。

### **我的 Prompt（第二版）**

```
我们现在进入第二阶段：设计核心接口和枚举。

【背景回顾】
上一篇我们已经完成了基础实体类的定义：
- Workflow：包含 graph、conversationVariables、environmentVariables
- Graph：包含 nodes 和 edges
- Node：包含 id、type、name、position、config（JsonNode）
- Edge：包含 id、source、target、type、sourceHandle、targetHandle

【当前任务】
现在需要设计三个核心组件：

1. **NodeType 枚举**
   - 定义所有支持的节点类型
   - 参考 Dify 的节点类型：llm、http-request、if-else、code、start、end、variable-aggregator、parameter-extractor
   - 每个枚举值要有对应的描述信息

2. **NodeExecutor 接口**
   - 定义节点执行的标准方法：execute(WorkflowContext context, Node node)
   - 返回值：ExecutionResult（包含是否成功、输出数据、错误信息）
   - 考虑异步执行的可能性（未来可能需要支持）

3. **WorkflowContext 类**
   - 存储工作流执行时的上下文信息
   - 管理变量（conversation_variables、environment_variables）
   - 管理节点输出（每个节点执行后的结果）
   - 提供变量解析方法（解析 {{#节点ID.字段名#}} 这种引用）

【技术要求】
- 使用 Java 21 的新特性（如果适用）
- 遵循 SOLID 原则
- 代码要有清晰的注释
- 考虑未来的扩展性

请先更新 plan.md，然后开始实现这三个组件。
```

---

## 03. AI 更新的 Plan.md

AI 很快更新了计划：

```
## 阶段 2：核心接口和枚举设计 (Core Interfaces & Enums)
- [ ] 定义 `NodeType` 枚举
  - [ ] 支持的节点类型：llm, http-request, if-else, code, start, end, variable-aggregator, parameter-extractor
  - [ ] 每个枚举值包含 code 和 description
- [ ] 定义 `NodeExecutor` 接口
  - [ ] 方法：execute(WorkflowContext context, Node node)
  - [ ] 返回值：ExecutionResult
- [ ] 定义 `WorkflowContext` 类
  - [ ] 字段：conversationVariables, environmentVariables, nodeOutputs
  - [ ] 方法：setNodeOutput(), getNodeOutput(), resolveVariable()
  - [ ] 变量解析逻辑（支持 {{#节点ID.字段名#}} 语法）
```

---

## 04. 实战演示：设计 NodeType 枚举

### **AI 生成的代码（第一版）**

```
@Getter
public enum NodeType {

    /**
     * 开始节点 - 工作流的入口点
     */
    START("start", "开始节点"),

    /**
     * 结束节点 - 工作流的出口点
     */
    END("end", "结束节点"),

    /**
     * LLM 节点 - 调用大语言模型
     */
    LLM("llm", "LLM节点"),

    /**
     * 代码执行节点 - 执行自定义代码
     */
    CODE("code", "代码节点"),

    /**
     * 条件分支节点 - 根据条件选择执行路径
     */
    IF_ELSE("if-else", "条件分支节点"),

    /**
     * 知识库检索节点 - 从知识库检索相关信息
     */
    KNOWLEDGE_RETRIEVAL("knowledge-retrieval", "知识库检索节点"),

    /**
     * 问答节点 - 基于上下文的问答
     */
    QUESTION_CLASSIFIER("question-classifier", "问题分类节点"),

    /**
     * HTTP 请求节点 - 发起 HTTP 请求
     */
    HTTP_REQUEST("http-request", "HTTP请求节点"),

    /**
     * 模板转换节点 - 使用模板转换数据
     */
    TEMPLATE_TRANSFORM("template-transform", "模板转换节点"),

    /**
     * 变量赋值节点 - 设置或更新变量
     */
    VARIABLE_ASSIGNER("variable-assigner", "变量赋值节点"),

    /**
     * 变量聚合节点 - 聚合多个变量
     */
    VARIABLE_AGGREGATOR("variable-aggregator", "变量聚合节点");
    
}
```

这就是 Vibe Engineering 的价值：AI 快速生成代码，我负责审查和优化。

---

## 05. 实战演示：设计 NodeExecutor 接口

### **我的 Prompt**

```
现在设计 NodeExecutor 接口。

要求：
1. 定义 execute() 方法，接收 WorkflowContext 和 Node 作为参数
2. 返回 ExecutionResult
3. 考虑未来可能需要支持异步执行，所以方法签名要留有扩展空间
4. 添加一个 supports() 方法，用于判断该执行器是否支持某个节点类型
```

### **AI 生成的代码**

```
public interface NodeExecutor {

    /**
     * 执行节点
     *
     * @param input 节点执行输入参数
     * @return 节点执行结果
     * @throws Exception 执行过程中的异常
     */
    NodeExecutionResult execute(NodeExecutionInput input) throws Exception;

    /**
     * 验证节点配置是否有效
     *
     * @param node 节点定义
     * @return 如果配置有效返回 true，否则返回 false
     */
    boolean validate(Node node);

    /**
     * 获取支持的节点类型
     *
     * @return 节点类型字符串
     */
    String getSupportedType();

    /**
     * 从 JsonNode 配置创建类型安全的节点数据对象
     *
     * @param config JsonNode 配置
     * @return 类型化的节点数据对象
     * @throws Exception 解析失败时抛出异常
     */
    BaseNodeData createNodeData(JsonNode config) throws Exception;
}
```

### **反馈**

我需要一个 **"执行器注册表"**，让系统能够根据节点类型自动找到对应的执行器。

我让 AI 补充了一个 `NodeExecutorRegistry` 类：

```
@Slf4j
@Component
public class NodeExecutorRegistry {

    /**
     * 执行器存储
     * key: 节点类型
     * value: 对应的执行器
     */
    private final Map<String, NodeExecutor> executors = new HashMap<>();

    /**
     * 注册节点执行器
     *
     * @param executor 要注册的执行器
     */
    public void register(NodeExecutor executor) {
        String type = executor.getSupportedType();
        if (executors.containsKey(type)) {
            log.warn("Executor for type '{}' already exists, will be replaced", type);
        }
        executors.put(type, executor);
        log.info("Registered executor for node type: {}", type);
    }

    /**
     * 根据节点类型获取执行器
     *
     * @param type 节点类型
     * @return 对应的执行器，如果不存在返回 Optional.empty()
     */
    public Optional<NodeExecutor> getExecutor(String type) {
        return Optional.ofNullable(executors.get(type));
    }

    /**
     * 检查是否存在指定类型的执行器
     *
     * @param type 节点类型
     * @return 如果存在返回 true，否则返回 false
     */
    public boolean hasExecutor(String type) {
        return executors.containsKey(type);
    }

    /**
     * 获取所有已注册的节点类型
     *
     * @return 已注册的节点类型集合
     */
    public java.util.Set<String> getRegisteredTypes() {
        return executors.keySet();
    }

    /**
     * 清空所有已注册的执行器
     */
    public void clear() {
        executors.clear();
        log.info("Cleared all registered executors");
    }
}
```

**这就是 Vibe Engineering 的"Best of N"思维**：不要只接受 AI 的第一个答案，要主动思考"还缺什么"。

---

## 06. 实战演示：设计 WorkflowContext 类

这是三个组件中最复杂的一个，因为它要负责：

1. 1. 存储变量（conversation\_variables、environment\_variables）
2. 2. 存储节点输出（每个节点执行后的结果）
3. 3. 解析变量引用（`{{#节点ID.字段名#}}`）

### **我的 Prompt**

```
现在设计 WorkflowContext 类。

要求：
1. 字段：conversationVariables (Map<String, Object>)
2. 字段：environmentVariables (Map<String, Object>)
3. 字段：nodeOutputs (Map<String, JsonNode>) - 存储每个节点的输出
4. 方法：setNodeOutput(String nodeId, JsonNode output)
5. 方法：getNodeOutput(String nodeId)
6. 方法：resolveVariable(String expression) - 解析 {{#节点ID.字段名#}} 这种引用
7. 使用 Jackson 的 JsonNode 来处理动态 JSON 数据
```

---

## 08. 更新 Plan.md：打勾 ✅

完成这一阶段后，我让 AI 更新 `plan.md`：

```
## 阶段 2：核心接口和枚举设计 (Core Interfaces & Enums)
- [x] 定义 `NodeType` 枚举
  - [x] 支持的节点类型：llm, http-request, if-else, code, start, end, variable-aggregator, parameter-extractor
  - [x] 每个枚举值包含 code 和 description
  - [x] 优化 fromCode() 方法（使用 Map 缓存）
- [x] 定义 `ExecutionResult` 类
  - [x] 字段：success (boolean), output (JsonNode), error (String)
  - [x] 提供静态工厂方法：success(), failure()
- [x] 定义 `NodeExecutor` 接口
  - [x] 方法：execute(WorkflowContext context, Node node)
  - [x] 方法：supports(NodeType nodeType)
  - [x] 补充 `NodeExecutorRegistry` 类（执行器注册表）
- [x] 定义 `WorkflowContext` 类
  - [x] 字段：conversationVariables, environmentVariables, nodeOutputs
  - [x] 方法：setNodeOutput(), getNodeOutput(), resolveVariable()
  - [x] 变量解析逻辑（支持 {{#节点ID.字段名#}} 语法）
  - [x] 补充 resolveVariableAsJson() 方法
```

---

## 09. 本篇总结

今天我们完成了：

**设计了 `NodeType` 枚举**（支持 8 种节点类型，优化了查找性能）
**设计了 `NodeExecutor` 接口**（定义了节点执行的标准契约）
**设计了 `WorkflowContext` 类**（管理变量和节点输出，支持变量引用解析）
**补充了 `NodeExecutorRegistry` 类**（执行器注册表）
**定义了 `ExecutionResult` 类**（统一的执行结果封装）

### **关键收获**

1. 1. **先定义契约，再实现细节**：接口和枚举是整个系统的"骨架"，设计好了后续开发会很顺畅
2. 2. **主动思考"还缺什么"**：不要只接受 AI 的第一个答案，要思考完整性（比如 `NodeExecutorRegistry`）

---