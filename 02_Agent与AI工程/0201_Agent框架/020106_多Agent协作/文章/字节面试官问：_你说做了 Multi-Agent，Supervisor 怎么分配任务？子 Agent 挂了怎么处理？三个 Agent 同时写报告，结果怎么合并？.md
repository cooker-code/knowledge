---
title: 字节面试官问："你说做了 Multi-Agent，Supervisor 怎么分配任务？子 Agent 挂了怎么处理？三个 Agent 同时写报告，结果怎么合并？"
author: 吴师兄学大模型
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247490011&idx=1&sn=b0833c131ac585a0fb06b7dd6fba5a28&chksm=c3042df311acd8b8d3ccb9391d6348dac895c72fe6ce9261d1de48bd2a7545c6b7621aef786c&mpshare=1&scene=24&srcid=0328vyq60gyZOtKZAoccbxH8&sharer_shareinfo=a81531ec664eba81331b4f0fffd64a06&sharer_shareinfo_first=a81531ec664eba81331b4f0fffd64a06#rd
---

# 大家好，我是吴师兄。

上周有个学员找我复盘字节的大模型面试，说面试官一开口就把他问懵了。

面试官问："你简历写了做过 Multi-Agent 系统，讲一下你们的 Supervisor 是怎么分配任务的？"

他答："就是 Supervisor 把任务拆分成几个子任务，分别交给不同的 Agent 去做。"

面试官点头，继续追："那子 Agent 挂了怎么处理？比如网络超时，或者 LLM 调用失败？"

他答："……会重试。"

面试官又追："重试几次？等待多久？重试还失败了整个任务就失败了吗？"

他开始语塞。

面试官最后一个问题："三个 Agent 同时在分析同一家公司，ResearchAgent 说这家公司营收增长，AnalysisAgent 说财务数据显示营收下滑，两个结论矛盾，WriterAgent 写报告的时候怎么合并？"

沉默。

面试官放下笔，说了一句话："你说的 Multi-Agent，只是把几个 LLM 调用摆在一起，不是真正的 Multi-Agent 系统。"

这句话，值得我们认真展开。今天把 Multi-Agent 从设计模式到工程实现全部拆开讲。

## 为什么需要 Multi-Agent——单 Agent 的根本局限

在讲 Multi-Agent 之前，我们得先诚实地承认：**单 Agent 不是万能的**。

我在"拓业智询"项目（一个银行对公客户智能咨询助手）中遇到的第一个真实问题，就是这个。

任务是这样的：客户经理输入一家中小企业的名称，系统需要生成一份完整的贷款风险评估报告，内容包括企业工商信息、财务状况、行业竞争格局、宏观市场趋势、历史贷款记录五个维度。

最开始我用单 Agent 实现：一个 ReAct Agent，配上搜索工具、SQL 查询工具、财务数据接口，让它自己规划步骤，顺序执行。

**第一个问题：上下文过长，精度下降。**

顺序执行五个维度的分析，每个维度检索出来的内容都要放进上下文。跑到第四个维度的时候，上下文已经超过 30000 token。在这种长上下文下，LLM 开始出现"中间遗忘"现象：报告的前两个维度分析得很详细，后两个维度分析越来越粗糙，甚至出现前后矛盾的结论。

这不是模型能力不够，而是长上下文本身的注意力分散问题。学术上叫"lost in the middle"现象——LLM 对位于上下文中间位置的信息，注意力显著低于开头和结尾。

**第二个问题：任务天然可以并行，但单 Agent 只能顺序执行。**

企业财务分析、竞争对手分析、市场趋势分析，这三件事之间没有强依赖关系，完全可以同时进行。但单 Agent 的 ReAct 循环是顺序的：先做 A，再做 B，再做 C。五个维度串行下来，整个任务耗时约 45 秒。用户等待体验极差。

**第三个问题：一个 Agent 很难在所有任务上都表现最优。**

检索任务需要 Agent 懂得怎么拆解查询词、构造召回策略；数据分析任务需要 Agent 理解财务指标、识别异常；报告生成任务需要 Agent 有良好的文字组织能力和结构化输出能力。这三种能力在 Prompt 层面的要求是不同的，甚至是相互冲突的——一个 Prompt 不可能同时把三件事都优化到极致。

这就是 Multi-Agent 的出发点：**分工、并行、独立上下文**。

单 Agent 的局限清楚了，接下来看 Multi-Agent 的 Supervisor 架构是怎么设计的。

## Supervisor 模式：Multi-Agent 的核心架构

Multi-Agent 有多种架构模式，但在生产环境中用得最多的是 **Supervisor 模式**。

基本结构是这样的：一个 Supervisor Agent 负责任务分解和协调，多个子 Agent 各司其职执行具体任务，最后由 WriterAgent 汇总生成报告。

Multi-Agent Supervisor模式架构

我们在"拓业智询"项目中使用 LangGraph 实现这个架构。先看核心状态定义和 Supervisor 节点：

```
from langgraph.graph import StateGraph, END from typing import TypedDict, Literal  class SupervisorState(TypedDict):     task: str                    # 总任务     subtasks: list[str]          # 分解的子任务     agent_results: dict          # 各Agent的结果     final_report: str            # 最终报告  def supervisor_node(state: SupervisorState) -> SupervisorState:     """Supervisor：任务分解与分配"""     task = state["task"]     # LLM分解任务为子任务     subtasks = decompose_task(task)     # 返回决策：下一步执行哪个Agent     return {**state, "subtasks": subtasks}  def route_to_agents(state: SupervisorState) -> Literal["research", "analysis", "writer", "end"]:     """根据状态决定下一个执行的Agent"""     if not state.get("agent_results", {}).get("research"):         return "research"     elif not state.get("agent_results", {}).get("analysis"):         return "analysis"     elif not state.get("final_report"):         return "writer"     else:         return "end"
```

这里有几个关键设计决策需要解释。

**为什么用 TypedDict 定义状态？**

LangGraph 的 Graph 是围绕状态流转设计的。所有节点（Agent）共享同一个状态对象，每个节点读取自己需要的字段，更新自己负责的字段，然后把更新后的状态传给下一个节点。用 TypedDict 定义状态，有两个好处：类型检查（在开发阶段就能发现字段拼错这类低级错误），以及文档化（state 的结构即文档，新成员接手时一眼就能理解数据流）。

**`route_to_agents` 函数的作用是什么？**

这是 LangGraph 中的"条件边"。Graph 中的节点通过这个路由函数决定下一步走哪条边。它的返回值对应不同的下游节点名称。这样 Supervisor 就实现了动态调度：根据当前状态（哪些 Agent 已经完成），决定下一步执行哪个 Agent。

完整的 Graph 构建如下：

```
def build_supervisor_graph():     graph = StateGraph(SupervisorState)      # 添加所有节点     graph.add_node("supervisor", supervisor_node)     graph.add_node("research", research_node)     graph.add_node("analysis", analysis_node)     graph.add_node("writer", writer_node)      # 设置入口     graph.set_entry_point("supervisor")      # Supervisor根据状态路由到子Agent     graph.add_conditional_edges(         "supervisor",         route_to_agents,         {             "research": "research",             "analysis": "analysis",             "writer": "writer",             "end": END         }     )      # 子Agent执行完毕后回到Supervisor进行下一步决策     graph.add_edge("research", "supervisor")     graph.add_edge("analysis", "supervisor")     graph.add_edge("writer", "supervisor")      return graph.compile()
```

注意这里的关键结构：每个子 Agent 执行完之后，都回到 Supervisor，由 Supervisor 再次判断下一步。这是 Supervisor 模式的核心——**决策权始终在 Supervisor 手里**，子 Agent 只负责执行，不负责决定下一步做什么。

架构搭好了，并行执行里有一个容易踩的坑，我们来看具体实现。

## 并行执行：让 ResearchAgent 同时检索多个维度

顺序执行的问题已经很清楚了。现在来看并行执行的实现。

在"拓业智询"的场景中，对一家企业的贷款风险评估，ResearchAgent 需要同时检索三个维度：企业工商信息、行业竞争格局、宏观政策环境。这三个检索任务之间没有依赖，天然适合并行。

```
import asyncio  async def parallel_research(queries: list[str]) -> list[dict]:     """并行执行多个Research子任务"""     tasks = [research_agent.ainvoke({"query": q}) for q in queries]     results = await asyncio.gather(*tasks, return_exceptions=True)      # 处理失败的子任务     final_results = []     for i, result in enumerate(results):         if isinstance(result, Exception):             print(f"子任务{i}失败: {result}，使用降级方案")             final_results.append({"query": queries[i], "result": "检索失败，跳过此部分"})         else:             final_results.append(result)     return final_results
```

这里有两点值得单独讲。

**`asyncio.gather(*tasks, return_exceptions=True)` 的用法。**

如果不加 `return_exceptions=True`，任何一个子任务抛出异常，`gather` 会立刻终止所有子任务并把异常往上抛。这在生产环境中是不可接受的——一个子任务失败，不能让所有任务都失败。

加上 `return_exceptions=True` 后，异常会被当作正常返回值放进 results 列表，我们可以在后续逐一检查每个结果是不是 Exception 实例，分别处理。

**`ainvoke` 而不是 `invoke`。**

LangGraph 和 LangChain 的 Agent 默认都提供了异步版本的调用接口 `ainvoke`。只有使用异步接口，`asyncio.gather` 才能真正并发地执行多个任务。如果用同步的 `invoke`，即使放进 `gather`，实际上还是顺序执行的。

实测数据：在"拓业智询"项目中，三个检索维度的任务，顺序执行约需 24 秒，并行执行约需 9 秒，提速约 2.7 倍（不是 3 倍，因为有协调开销和资源竞争）。

并行执行解决了速度问题，但子 Agent 一旦失败，整个流程怎么办——这是面试官问的第二个核心问题。

## 子 Agent 错误处理：重试 + 降级，缺一不可

这是面试官问的第二个核心问题。很多同学说"失败了就重试"，但重试本身也有设计问题。

我们的错误处理策略分两层：**重试**和**降级**。

子Agent错误处理：重试+降级策略

完整实现如下：

```
import asyncio from typing import Optional  async def execute_agent_with_retry(     agent,     input_data: dict,     agent_name: str,     max_retries: int = 2,     retry_delay: float = 1.0 ) -> dict:     """     带重试和降级的Agent执行器     - max_retries: 最多重试次数（不含第一次执行）     - retry_delay: 每次重试前等待的秒数     """     last_exception: Optional[Exception] = None      for attempt in range(max_retries + 1):  # 0, 1, 2 共三次机会         try:             result = await agent.ainvoke(input_data)             return {"status": "success", "data": result, "agent": agent_name}         except Exception as e:             last_exception = e             if attempt < max_retries:                 print(f"[{agent_name}] 第{attempt + 1}次失败: {e}，{retry_delay}秒后重试...")                 await asyncio.sleep(retry_delay)             else:                 print(f"[{agent_name}] 已重试{max_retries}次，全部失败，触发降级")      # 降级处理：标记数据缺失，不抛出异常，允许流程继续     return {         "status": "degraded",         "data": None,         "agent": agent_name,         "error": str(last_exception),         "message": f"{agent_name}数据获取失败，该部分分析结果缺失"     }
```

在 Supervisor 层，需要感知哪些 Agent 发生了降级：

```
def supervisor_node(state: SupervisorState) -> SupervisorState:     agent_results = state.get("agent_results", {})      # 检查是否有降级的Agent     degraded_agents = [         name for name, result in agent_results.items()         if isinstance(result, dict) and result.get("status") == "degraded"     ]      if degraded_agents:         print(f"警告：以下Agent数据缺失，将继续生成报告但需标注: {degraded_agents}")         # 在state中记录缺失信息，WriterAgent生成报告时需要标注         return {**state, "data_gaps": degraded_agents}      return state
```

这里最重要的设计原则是：**一个子 Agent 的失败，不能阻塞整个任务**。在银行的业务场景中，宁可生成一份带有"部分数据缺失"标注的报告，也不能让整个系统因为某一个数据源的超时而返回错误。客户经理拿到带注释的报告，还能参考其他维度做判断；拿到一个系统错误，什么都做不了。

这也是降级策略的业务逻辑根据——**降级不是妥协，是对用户体验的保护**。

错误处理有了兜底，还有一个更难的问题：多个 Agent 并行分析时得出了相互矛盾的结论，WriterAgent 该怎么处理？

## 结果合并：WriterAgent 怎么处理矛盾的结论

这是面试官最后那个问题，也是最容易被忽视的一个细节。

三个 Agent 并行分析同一家企业，有可能得出矛盾的结论。在"拓业智询"中确实出现过这种情况：ResearchAgent 通过新闻检索发现某企业最近签了几个大合同，判断营收向好；AnalysisAgent 通过分析企业提交的财务报表，发现应收账款周转率下降，实际现金流紧张。两个结论方向相反。

WriterAgent 的设计需要处理这类矛盾。核心代码：

```
def writer_node(state: SupervisorState) -> SupervisorState:     """WriterAgent：合并各Agent结果生成最终报告"""     research_result = state["agent_results"].get("research", {})     analysis_result = state["agent_results"].get("analysis", {})     data_gaps = state.get("data_gaps", [])      # 构建降级说明     gap_notice = ""     if data_gaps:         gap_notice = f"\n\n注意：以下分析模块数据缺失，相关结论请谨慎参考：{', '.join(data_gaps)}"      merge_prompt = f"""     你是报告生成专家。请基于以下各分析模块的结果，生成一份结构化的综合报告：      ## 研究发现     {research_result.get('data', '数据缺失')}      ## 数据分析     {analysis_result.get('data', '数据缺失')}     {gap_notice}      要求：     1. 合并重复内容，保留关键信息     2. 如有数据矛盾，优先采用数据分析模块的结论（财务数据比新闻检索更客观）     3. 对矛盾点必须明确标注："研究发现与数据分析存在分歧，建议进一步核实"     4. 生成执行摘要（3-5条核心结论）     5. 如有数据缺失，在对应章节标注"[数据缺失，仅供参考]"     """     final_report = llm.invoke(merge_prompt).content     return {**state, "final_report": final_report}
```

这里有一个关键规则：**数据分析模块的结论优先于检索模块**。这是我们基于业务场景制定的优先级规则——企业提交的财务报表（数据分析的来源）比公开新闻（检索的来源）更具法律效力，在贷款风险评估场景下应该赋予更高权重。

不同业务场景下，这个优先级规则可能不同。关键在于：**优先级规则要显式地写进 WriterAgent 的 Prompt，而不是让 LLM 自己决定**。LLM 自己决定意味着每次合并的结果可能不一致，这在金融场景中是不可接受的。

架构细节讲完了，用真实数据来看一下单 Agent 和 Multi-Agent 的差距究竟有多大。

## 单 Agent vs Multi-Agent：拓业智询的真实数据

单Agent vs Multi-Agent 性能对比

在"拓业智询"项目中，我们对同一批 50 个企业评估任务做了 A/B 对比测试，结果如下：

* **总耗时**

  ：单 Agent 顺序执行平均 45 秒，Multi-Agent 并行执行平均 18 秒，提速 60%
* **上下文长度**

  ：单 Agent 到后期分析维度时上下文超过 32000 token，Multi-Agent 各子 Agent 独立上下文，最长不超过 8000 token
* **报告质量评分**

  ：我们邀请 5 名有经验的客户经理对报告进行盲评（1-100 分），单 Agent 平均 72 分，Multi-Agent 平均 89 分，提升 23.6%
* **适用场景**

  ：单 Agent 在简单的单维度查询任务（如"查一下这家公司的注册资本"）上反而更快，Multi-Agent 的优势在多维度复杂分析任务中才能体现

**但并行不是越多越好。** 这一点值得单独强调。

在实验中，我们尝试了 2、3、4、5、6 个子 Agent 并行的方案：

* 2-3 个子 Agent 并行：性能提升最明显，协调开销小
* 4-5 个子 Agent 并行：性能提升趋于平缓，开始出现 LLM API 的并发限速问题
* 6 个子 Agent 并行：总耗时反而比 3 个并行的方案更长，原因是 API 限速导致多个请求排队等待

另外，**任务有强顺序依赖时，不能并行**。在"拓业智询"中，WriterAgent 必须在所有分析 Agent 完成之后才能运行，因为它需要所有分析结果作为输入——这是典型的顺序依赖，强行并行没有意义。

实战建议：**2-3 个子 Agent 并行是大多数场景下的最佳平衡点**，既能获得明显的提速效果，又不会因协调开销和 API 限速把收益全部抵消。

数据和实现都清楚了，最后来看面试时怎么把这些内容组织成一个有层次的回答。

## 面试怎么答 Multi-Agent 架构？

这是这篇文章最实用的部分。

面试官问"你们的 Multi-Agent 架构是怎么设计的"，不要上来就讲技术细节。先给一个整体框架，再按层次展开：

**第一层：为什么用 Multi-Agent（动机）**

"我们做的是银行对公客户风险评估，单 Agent 顺序分析五个维度，上下文过长精度下降，而且45秒的响应时间用户体验很差。Multi-Agent 让各维度分析并行执行，各 Agent 独立上下文，总耗时降到 18 秒。"

**第二层：架构是什么样的（设计）**

"我们用 LangGraph 实现 Supervisor 模式。Supervisor 负责任务分解和状态管理，三个子 Agent（ResearchAgent、AnalysisAgent、SQLAgent）并行执行，最后由 WriterAgent 汇总生成报告。每个子 Agent 执行完回到 Supervisor，由 Supervisor 决定下一步——决策权始终在 Supervisor 手里。"

**第三层：错误处理怎么做的（健壮性）**

"每个子 Agent 有独立的重试机制：失败后最多重试 2 次，每次等待 1 秒。重试全部失败触发降级——标记该模块数据缺失，继续执行其他 Agent。WriterAgent 生成报告时会在缺失部分标注说明。核心原则是一个子 Agent 的失败不能阻塞整个任务。"

**第四层：结果合并怎么处理矛盾（细节）**

"如果检索结果和数据分析结论矛盾，我们规定数据分析模块优先，因为财务报表比公开新闻更有法律效力。矛盾点会在报告中明确标注，建议人工核实。这个优先级规则写在 WriterAgent 的 Prompt 里，不让 LLM 自行决定，保证结果一致性。"

**第五层：有什么局限（自我批判，加分项）**

"并行不是越多越好。我们测试发现超过 5 个子 Agent 并行时，因为 API 限速，总耗时反而比 3 个并行更长。另外强顺序依赖的任务不能并行，WriterAgent 必须等所有分析完成才能运行。实战下来，2-3 个子 Agent 并行是最佳平衡点。"

能说到第五层，这道题基本稳了。

大多数候选人只能说到第一层或第二层，能说到错误处理和结果合并细节的，面试官会认为你真正在生产环境里跑过这个系统，而不只是看过几篇论文。

## 总结

Multi-Agent 不是把几个 LLM 调用摆在一起。它需要解决的是：任务如何分解、子任务如何调度、子 Agent 失败如何处理、并行结果如何合并、矛盾结论如何裁定。

Supervisor 模式是生产环境中最主流的 Multi-Agent 架构：一个中心 Supervisor 掌控决策，子 Agent 只负责执行，决策权和执行权分离。

错误处理的核心是"不阻塞"原则：重试 + 降级，让一个子 Agent 的失败止步于自己，不传染给整个系统。

结果合并的核心是"规则显式化"：数据矛盾时谁优先、矛盾怎么标注、缺失怎么处理，这些规则要写进 Prompt，不能让 LLM 自己决定。

并行是有上限的：2-3 个子 Agent 并行是大多数场景的最优解，更多的并行带来的是协调开销和 API 限速，不是更快的速度。

我是吴师兄，我们下篇文章见。

*本文内容基于吴师兄大模型训练营 RAG 实战系列课程整理。系列往期文章可在主页查看。*

往期推荐

[阿里面试官问："HotpotQA 只有 2 跳推理，你的 Deep Research Agent 线上要搜 20 步，这模型训出来能用吗？"](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489850&idx=1&sn=3cb83b85a96f8eb41e9e987339483172&scene=21#wechat_redirect)

[字节面试官问："你说做过 Deep Research 系统，那它跟传统 RAG 的核心区别在哪？搜了 20 个网页之后怎么防止规划路径跑偏？"](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489776&idx=1&sn=fb69de639e40071d30cbd3e2aede07c8&scene=21#wechat_redirect)

[蚂蚁面试官怒怼："你说用了混合检索加 Rerank 精排，那检索函数的核心代码你能手撕出来吗？连 retrieval 的调用流程都讲不清？"](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489763&idx=1&sn=51537559674e0f8857b770088c091ae4&scene=21#wechat_redirect)

[阿里面试官怒了："MinerU 你就只会调 API？它的表格跨页截断、公式识别失败这些短板你怎么解决的？"](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489725&idx=1&sn=b853a2636ae8912dbf3f967b11c97945&scene=21#wechat_redirect)

[从零到拿OFFER，专注于落地实战的大模型训练营来了！](https://mp.weixin.qq.com/s?__biz=MzkzMDIwMzg1Mw==&mid=2247489823&idx=1&sn=ebe86cb11141855cf94b6ceb6badbe40&scene=21#wechat_redirect)