---
title: LangGraph 子图功能详解：进阶技巧与实战经验（下篇）
author: X Agentic
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYzODA4ODUxOQ==&mid=2247483760&idx=1&sn=eb4a9be73866e6b00ac8ad1e24adbb1c&chksm=f1b83e5b255d939f9df11816b2c512846855faad2ad3e247ac59bf39c970995056d7bd271da4&mpshare=1&scene=24&srcid=1230GBmzK3eYHCHjTlAEJv90&sharer_shareinfo=750323029ed4610b0f69bbc49c85b450&sharer_shareinfo_first=750323029ed4610b0f69bbc49c85b450#rd
---

> 欢迎回来！在上篇中，我们了解了子图的基本概念和"扇入扇出"模式。今天，我们要深入探讨子图的进阶用法，包括嵌套子图、条件子图、错误处理等高级技巧。

## 子图的嵌套：俄罗斯套娃式的优雅

"子图里还能有子图吗？"

答案是：**当然可以！** 这就是子图最强大的地方——它可以无限嵌套。

假设一下这样的场景：你的主图执行到某个阶段，需要并行执行多个子任务。而每个子任务本身又是一个复杂的流程，需要多个步骤并行执行。

在 AgoraAI 项目中，我们虽然没有实现嵌套子图，但完全可以这样做：

```
def execute_stage_with_nested_subgraph(state: ProjectState, stage_agent_ids: List[str]):  
    """执行一个阶段，使用嵌套子图"""  
      
    # 第一层子图：并行执行多个智能体  
    outer_workflow = StateGraph(ProjectState)  
      
    for agent_id in stage_agent_ids:  
        # 为每个智能体创建一个节点  
        # 但这个节点内部又是一个子图！  
        outer_workflow.add_node(  
            f"agent_{agent_id}",  
            create_agent_subgraph(agent_id)  # 返回一个子图节点  
        )  
      
    # ... 设置入口和汇聚节点  
      
    return outer_workflow.compile()
```

嵌套子图的优势：

* **模块化**

  ：每个子图都是一个独立的、可复用的模块
* **清晰的结构**

  ：复杂的流程被分解成多个层次，易于理解和维护
* **灵活的组合**

  ：可以根据需要组合不同的子图

但要注意：**不要嵌套太深**，否则调试会变成噩梦。建议最多 2-3 层嵌套。

## 条件子图：根据状态动态创建

有时候，你需要根据当前状态动态决定创建什么样的子图。这就是**条件子图**的用武之地。

在我们的项目中，虽然 `execute_parallel_agents` 函数会根据 `agent_ids` 的数量决定是否创建子图，但我们可以做得更智能：

```
def execute_parallel_agents(state: ProjectState, agent_ids: List[str]) -> ProjectState:  
    """并行执行多个智能体的任务 - 增强版"""  
      
    # 根据智能体数量选择策略  
    iflen(agent_ids) == 0:  
        return state  
    eliflen(agent_ids) == 1:  
        # 单个智能体：直接执行，不需要子图  
        agent_id = agent_ids[0]  
        execute_func = execute_agent_node(agent_id)  
        update = execute_func(state)  
        return state  
    eliflen(agent_ids) <= 3:  
        # 少量智能体：使用简单并行子图  
        return create_simple_parallel_subgraph(state, agent_ids)  
    else:  
        # 大量智能体：使用分批并行子图（避免资源耗尽）  
        return create_batched_parallel_subgraph(state, agent_ids)
```

更高级的用法是根据智能体的类型或依赖关系创建不同的子图结构：

```
def create_adaptive_subgraph(state: ProjectState, agent_ids: List[str]):  
    """根据智能体特性创建自适应子图"""  
      
    # 分析智能体的执行时间  
    fast_agents = [aid for aid in agent_ids if is_fast_agent(aid)]  
    slow_agents = [aid for aid in agent_ids ifnot is_fast_agent(aid)]  
      
    workflow = StateGraph(ProjectState)  
      
    # 快速智能体：直接并行  
    for agent_id in fast_agents:  
        workflow.add_node(f"fast_{agent_id}", execute_agent_node(agent_id))  
      
    # 慢速智能体：分批执行，避免资源竞争  
    for i, agent_id inenumerate(slow_agents):  
        workflow.add_node(f"slow_{agent_id}", execute_agent_node(agent_id))  
      
    # 根据实际情况设置不同的执行策略  
    # ...  
      
    return workflow.compile()
```

## 错误处理：当并行任务失败时

这是子图使用中最容易出现的问题：**如果并行执行的多个任务中，有一个失败了怎么办？**

在我们的当前实现中，每个 `execute_agent_node` 都会捕获异常并返回失败状态：

```
def execute_agent_node(agent_id: str):  
    def_execute(state: ProjectState) -> ProjectState:  
        try:  
            # ... 执行逻辑  
            return {"agent_tasks": {agent_id: success_task}}  
        except Exception as e:  
            logger.error(f"执行智能体 {agent_id} 任务失败: {e}")  
            # 返回失败状态  
            return {  
                "agent_tasks": {  
                    agent_id: AgentTask(  
                        agent_id=agent_id,  
                        status="failed",  
                        result=f"执行失败: {str(e)}",  
                        dependencies_met=False  
                    )  
                }  
            }  
    return _execute
```

但这样还不够！我们需要在子图的汇聚节点中检查是否有任务失败：

```
def create_robust_parallel_subgraph(agent_ids: List[str]):  
    """创建带错误处理的并行子图"""  
    workflow = StateGraph(ProjectState)  
      
    # ... 添加执行节点 ...  
      
    # 增强的汇聚节点：检查执行结果  
    defmerge_with_error_check(state: ProjectState) -> ProjectState:  
        agent_tasks = state.get('agent_tasks', {})  
          
        # 检查是否有失败的任务  
        failed_agents = [  
            agent_id for agent_id in agent_ids  
            if agent_tasks.get(agent_id, {}).status == 'failed'  
        ]  
          
        if failed_agents:  
            logger.warning(f"以下智能体执行失败: {failed_agents}")  
            # 可以选择：  
            # 1. 继续执行（部分成功）  
            # 2. 标记整个阶段失败  
            # 3. 重试失败的智能体  
            state['partial_failure'] = True  
            state['failed_agents'] = failed_agents  
          
        return state  
      
    workflow.add_node("merge_results", merge_with_error_check)  
    # ...  
      
    return workflow.compile()
```

## 性能优化技巧

子图功能虽然好用，但不要盲目使用，要根据项目因地制宜，以下是子图功能的使用tip。

### 1. 避免不必要的子图创建

如果只有一个任务，直接执行，不要创建子图：

```
if len(agent_ids) == 1:  
    # 直接执行，避免子图开销  
    return execute_single_agent(state, agent_ids[0])  
else:  
    # 多个任务才创建子图  
    return create_parallel_subgraph(state, agent_ids)
```

### 2. 合理控制并行度

不要无限制地并行执行，可能会耗尽资源：

```
MAX_PARALLEL_AGENTS = 5  
  
def execute_with_limit(state: ProjectState, agent_ids: List[str]):  
    """限制并行度的执行"""  
    iflen(agent_ids) <= MAX_PARALLEL_AGENTS:  
        # 直接并行执行  
        return execute_parallel_agents(state, agent_ids)  
    else:  
        # 分批执行  
        batches = [agent_ids[i:i+MAX_PARALLEL_AGENTS]   
                   for i inrange(0, len(agent_ids), MAX_PARALLEL_AGENTS)]  
          
        for batch in batches:  
            state = execute_parallel_agents(state, batch)  
            if state.get('error'):  
                break  
          
        return state
```

### 3. 使用异步执行（如果支持）

虽然 LangGraph 本身是同步的，但你可以让每个节点内部使用异步：

```
import asyncio  
  
async def async_execute_agent(agent_id: str, state: ProjectState):  
    """异步执行智能体任务"""  
    # 使用异步 HTTP 客户端、异步数据库操作等  
    result = await some_async_operation(state)  
    return result  
  
defexecute_agent_node(agent_id: str):  
    def_execute(state: ProjectState) -> ProjectState:  
        # 在同步节点中运行异步代码  
        loop = asyncio.new_event_loop()  
        asyncio.set_event_loop(loop)  
        try:  
            result = loop.run_until_complete(async_execute_agent(agent_id, state))  
            return {"agent_tasks": {agent_id: result}}  
        finally:  
            loop.close()  
    return _execute
```

## 实战：AgoraAI 项目中的子图应用

让我们看看 AgoraAI 项目中子图的完整应用场景：

```
def execute_all_tasks_node(state: ProjectState) -> ProjectState:  
    """执行所有任务的主节点"""  
    execution_plan = state.get('execution_plan')  
    stages = execution_plan.stages  
      
    # 按阶段顺序执行（阶段内并行，阶段间串行）  
    for stage_index, stage_agent_ids inenumerate(stages, 1):  
        logger.info(f"执行第 {stage_index}/{len(stages)} 阶段，包含 {len(stage_agent_ids)} 个智能体")  
          
        # 使用子图并行执行该阶段的所有智能体  
        state = execute_parallel_agents(state, stage_agent_ids)  
          
        if state.get('error'):  
            logger.error(f"阶段 {stage_index} 执行失败")  
            break  
      
    return state
```

这个设计的精妙之处在于：

1. **阶段间串行**

   ：确保依赖关系得到满足
2. **阶段内并行**

   ：同一阶段的任务可以同时执行，提高效率
3. **子图封装**

   ：每个阶段的并行执行都被封装在子图中，主图保持简洁

## 调试子图的技巧

子图调试确实比普通节点复杂，这里有几个技巧：

### 1. 添加详细的日志

```
def execute_parallel_agents(state: ProjectState, agent_ids: List[str]):  
    logger.info(f"🔵 创建并行子图，智能体数量: {len(agent_ids)}")  
    logger.info(f"🔵 智能体列表: {agent_ids}")  
      
    # ... 创建子图 ...  
      
    logger.info(f"🟢 开始执行子图")  
    state = subgraph.invoke(state)  
    logger.info(f"🟢 子图执行完成")  
      
    # 检查结果  
    agent_tasks = state.get('agent_tasks', {})  
    for agent_id in agent_ids:  
        task = agent_tasks.get(agent_id)  
        status = task.status if task else"unknown"  
        logger.info(f"📊 智能体 {agent_id} 状态: {status}")  
      
    return state
```

### 2. 可视化子图结构

```
def visualize_subgraph(agent_ids: List[str]):  
    """可视化子图结构（用于调试）"""  
    print(f"子图结构（{len(agent_ids)} 个智能体）:")  
    print("  start_parallel")  
    for agent_id in agent_ids:  
        print(f"    -> execute_{agent_id}")  
    print("  -> merge_results")  
    print("  -> END")
```

### 3. 测试单个子图

在开发时，可以单独测试子图：

```
# 测试代码  
test_state = {  
    "topic": "测试项目",  
    "agents": [...],  
    "agent_tasks": {}  
}  
  
# 只测试子图部分  
result_state = execute_parallel_agents(test_state, ["agent1", "agent2"])  
print(result_state)
```

## 常见陷阱和解决方案

### 陷阱 1：状态合并冲突

**问题**：多个并行节点更新了状态的同一个字段，导致覆盖。

**解决**：使用 `Annotated` 类型和合并函数，确保状态正确合并。

### 陷阱 2：子图执行时间过长

**问题**：子图中的某个节点执行时间很长，阻塞了整个流程。

**解决**：添加超时机制，或者将长时间运行的任务拆分成多个步骤。

### 陷阱 3：资源竞争

**问题**：多个并行任务同时访问共享资源（如数据库、API），导致竞争。

**解决**：使用连接池、限流器，或者分批执行。

## 总结

子图是 LangGraph 中一个强大但容易被忽视的功能。它让你能够：

* ✅ 实现真正的并行执行
* ✅ 优雅地管理复杂的状态合并
* ✅ 构建模块化和可复用的工作流
* ✅ 处理复杂的多任务协作场景

在 AgoraAI 项目中，子图帮助我们实现了多智能体的高效协作，让整个系统既灵活又高效。

记住：**子图并非适用于所有场景，但在需要并行执行和复杂状态管理的场景下，它是优雅且高效的解决方案。**

---

如果你在使用子图时遇到了问题，欢迎在评论区交流。也欢迎查看 AgoraAI 项目的完整代码，了解更多实现细节。

**项目地址**：https://github.com/senga07/AgoraAI.git

我是丢钢笔的人，下期见~