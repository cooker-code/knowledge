---
title: Agent实战教程：如何从头开始使用LangGraph构建自己的DeepResearch(下)
author: ChallengeHub
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247495258&idx=1&sn=f3c288f7765814727dffc160d50f52f6&chksm=9a6f8c064b8658af1dd6e82632f743bbeb0e0b3e50eeb044df30d35490dd289094d45ff4132b&mpshare=1&scene=24&srcid=09255RCCiVf4gTt3KJrbVZWe&sharer_shareinfo=1a77bdd7cdaa192ae0402e728b4ed42f&sharer_shareinfo_first=1a77bdd7cdaa192ae0402e728b4ed42f#rd
---

**[Agent实战教程：如何从头开始使用LangGraph构建自己的DeepResearch(上)](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247495233&idx=1&sn=3672c81708f22ca1ae5b9aec563f71c3&scene=21#wechat_redirect)**

> 本项目完整代码地址：https://github.com/yanqiangmiffy/Agent-Tutorials-ZH/tree/main/deep\_research\_from\_scratch

上一篇内容我们讲解了怎么构建DeepResearch两个比较核心的组件：确定用户研究范围以及研究，这两个组件可以大致满足用户输入研究主题或者查询，然后输出一个研究结果，不过这个对于系统来说大概率会遇到当用户查询相对复杂，或者我们拆分成多个子查询子主题的时候会造成上下文窗口累计导致上下文冲突，这个时候我们需要一个监督者，LangGraph中叫做**Supervisor**。

# **DeepResearch中的监督者**

监督者的工作很简单：将研究任务委派给适当数量的子智能体。

下面是我们加上监督者的整体研究流程：

我们之前构建了一个绑定到自定义工具或MCP服务器的研究智能体。现在，如果请求很复杂并且有几个子主题会怎样？单个智能体在处理多个子主题时响应质量可能会下降（例如，比较A子主题、B子主题、C子主题），因为单个上下文窗口需要存储和推理所有子主题的工具反馈。

当上下文窗口在许多不同子主题之间累积工具调用时，许多失效模式（如上下文冲突）变得普遍。正如Anthropic博客文章中**How we built our multi-agent research system**讨论的，多智能体系统可以将子主题分配给具有隔离上下文窗口的子智能体。我们将构建一个系统，其中监督者确定研究简报是否可以分解为独立的子主题，并委派给具有隔离上下文窗口的子智能体。

## **提示词设计**

现在，让我们为监督者设计一个遵循我们讨论的原则的提示词，并包含一些来自研究文献的见解。

### **1. 像智能体一样思考**

您会给新工作同事什么指示？

* 仔细阅读问题 - 用户需要什么具体信息？
* 决定如何委派研究 - 仔细考虑问题并决定如何委派研究。是否有多个独立的方向可以同时探索？
* 在每次调用ConductResearch后，暂停并评估 - 我是否有足够的信息来回答？还缺少什么？

### **2. 具体启发式规则（用于任务委派）**

使用硬限制来防止研究智能体过度调用工具：

* 偏向单个智能体 - 除非用户请求有明确的并行化机会，否则为了简单起见使用单个智能体。
* 能够自信回答时停止 - 不要为了完美而继续委派研究。
* 限制工具调用 - 如果找不到合适的来源，在3次ConductResearch工具调用后总是停止。

### **3. 展示你的思考过程**

在调用ConductResearch工具之前，使用think\_tool规划您的方法：

* 任务是否可以分解为更小的子任务？

在每次ConductResearch工具调用后，使用think\_tool分析结果：

* 我发现了什么关键信息？
* 还缺少什么？
* 我是否有足够的信息来全面回答问题？
* 我应该搜索更多还是提供我的答案？

### **4. 扩展规则**

简单的事实查找、列表和排名可以使用单个子智能体。

例如：列出旧金山前10名咖啡店 -> 使用1个子智能体

用户请求中呈现的比较可以为每个比较元素使用一个子智能体。

例如：比较OpenAI vs. Anthropic vs. DeepMind的AI安全方法 -> 使用3个子智能体。

委派清晰、不同、不重叠的子主题

```
from utils import show_prompt  
from deep_research_from_scratch.prompts import lead_researcher_prompt  
show_prompt(lead_researcher_prompt, "lead_researcher_prompt")
```

```
lead_researcher_prompt = """你是一名研究主管。你的工作是通过调用"ConductResearch"工具来进行研究。为了提供上下文，今天的日期是{date}。  
  
<任务>  
你的重点是调用"ConductResearch"工具，针对用户传入的整体研究问题进行研究。  
当你对工具调用返回的研究结果完全满意时，你应该调用"ResearchComplete"工具来表明你已完成研究。  
</任务>  
  
<可用工具>  
你可以使用三个主要工具：  
1. **ConductResearch**：将研究任务委托给专门的子智能体  
2. **ResearchComplete**：表明研究已完成  
3. **think_tool**：用于研究过程中的反思和战略规划  
  
**关键：在调用ConductResearch之前使用think_tool来规划你的方法，并在每次ConductResearch之后使用think_tool评估进展**  
**并行研究**：当你识别出可以同时探索的多个独立子主题时，在单个响应中进行多个ConductResearch工具调用，以启用并行研究执行。对于比较性或多方面的问题，这比顺序研究更有效率。每次迭代最多使用{max_concurrent_research_units}个并行智能体。  
</可用工具>  
  
<指导>  
像一个时间和资源有限的研究经理一样思考。遵循以下步骤：  
  
1. **仔细阅读问题** - 用户需要什么具体信息？  
2. **决定如何委派研究** - 仔细考虑问题并决定如何委派研究。是否有多个可以同时探索的独立方向？  
3. **每次调用ConductResearch后，暂停并评估** - 我是否有足够信息来回答？还缺少什么？  
</指导>  
  
<硬性限制>  
**任务委派预算**（防止过度委派）：  
- **偏向单一智能体** - 除非用户请求有明确的并行化机会，否则为简单起见使用单一智能体  
- **当你能够自信回答时停止** - 不要为了完美而继续委派研究  
- **限制工具调用** - 如果你找不到正确的信息源，始终在{max_researcher_iterations}次对think_tool和ConductResearch的工具调用后停止  
</硬性限制>  
  
<展示你的思考>  
在调用ConductResearch工具之前，使用think_tool来规划你的方法：  
- 任务是否可以分解为更小的子任务？  
  
在每次ConductResearch工具调用后，使用think_tool分析结果：  
- 我找到了哪些关键信息？  
- 还缺少什么？  
- 我是否有足够信息来全面回答问题？  
- 我应该委派更多研究还是调用ResearchComplete？  
</展示你的思考>  
  
<扩展规则>  
**简单的事实查找、列表和排名**可以使用单个子智能体：  
- *示例*：列出旧金山排名前10的咖啡店 → 使用1个子智能体  
  
**用户请求中提出的比较**可以为比较的每个元素使用一个子智能体：  
- *示例*：比较OpenAI、Anthropic和DeepMind在AI安全方面的方法 → 使用3个子智能体  
- 委派明确、独特、不重叠的子主题  
  
**重要提醒：**  
- 每次ConductResearch调用都会为该特定主题生成一个专门的Research智能体  
- 将有一个单独的智能体撰写最终报告 - 你只需要收集信息  
- 调用ConductResearch时，提供完整的独立指令 - 子智能体无法看到其他智能体的工作  
- 在你的研究问题中不要使用首字母缩写或缩略语，要非常清晰和具体  
</扩展规则>"""
```

## **状态管理**

监督者状态管理整体研究协调，而研究员状态处理个别研究任务。

```
"""  
多智能体研究监督者的状态定义  
  
此模块定义了用于多智能体研究监督者工作流的状态对象和工具，  
包括协调状态和研究工具。  
"""  
  
import operator  
from typing_extensions import Annotated, TypedDict, Sequence  
  
from langchain_core.messages import BaseMessage  
from langchain_core.tools import tool  
from langgraph.graph.message import add_messages  
from pydantic import BaseModel, Field  
  
class SupervisorState(TypedDict):  
    """  
    多智能体研究监督者的状态。  
      
    管理监督者和研究智能体之间的协调，跟踪研究进度  
    并累积来自多个子智能体的发现。  
    """  
      
    # 与监督者交换的消息，用于协调和决策  
    supervisor_messages: Annotated[Sequence[BaseMessage], add_messages]  
    # 指导整体研究方向的详细研究简报  
    research_brief: str  
    # 为最终报告生成准备的已处理和结构化笔记  
    notes: Annotated[list[str], operator.add] = []  
    # 跟踪已执行研究迭代次数的计数器  
    research_iterations: int = 0  
    # 从子智能体研究中收集的原始未处理研究笔记  
    raw_notes: Annotated[list[str], operator.add] = []  
  
@tool  
class ConductResearch(BaseModel):  
    """将研究任务委派给专门子智能体的工具。"""  
    research_topic: str = Field(  
        description="要研究的主题。应该是单一主题，并应详细描述（至少一个段落）。",  
    )  
  
@tool  
class ResearchComplete(BaseModel):  
    """表示研究过程已完成的工具。"""  
    pass
```

## **多智能体系统**

现在，我们将定义我们的智能体。多智能体系统是由多个智能体协作完成任务的系统。主要好处是上下文隔离，正如《智能体的上下文工程》中讨论的那样。

```
"""用于协调多个专门智能体研究的多智能体监督者。  
  
此模块实现监督者模式，其中：  
1. 监督者智能体协调研究活动并委派任务  
2. 多个研究智能体独立处理特定子主题  
3. 结果被聚合和压缩用于最终报告  
  
监督者使用并行研究执行来提高效率，同时  
为每个研究主题维护隔离的上下文窗口。  
"""  
  
import asyncio  
  
from typing_extensions import Literal  
  
from langchain.chat_models import init_chat_model  
from langchain_core.messages import (  
    HumanMessage,   
    BaseMessage,   
    SystemMessage,   
    ToolMessage,  
    filter_messages  
)  
from langgraph.graph import StateGraph, START, END  
from langgraph.types import Command  
  
from deep_research_from_scratch.prompts import lead_researcher_prompt  
from deep_research_from_scratch.research_agent import researcher_agent  
from deep_research_from_scratch.state_multi_agent_supervisor import (  
    SupervisorState,   
    ConductResearch,   
    ResearchComplete  
)  
from deep_research_from_scratch.utils import get_today_str, think_tool  
  
def get_notes_from_tool_calls(messages: list[BaseMessage]) -> list[str]:  
    """从监督者消息历史中的ToolMessage对象中提取研究笔记。  
      
    此函数检索子智能体作为ToolMessage内容返回的压缩研究发现。  
    当监督者通过ConductResearch工具调用将研究委派给子智能体时，  
    每个子智能体将其压缩的发现作为ToolMessage的内容返回。  
    此函数提取所有此类ToolMessage内容以编制最终的研究笔记。  
      
    参数:  
        messages: 来自监督者对话历史的消息列表  
          
    返回:  
        从ToolMessage对象中提取的研究笔记字符串列表  
    """  
    return [tool_msg.content for tool_msg in filter_messages(messages, include_types="tool")]  
  
# 确保Jupyter环境的异步兼容性  
try:  
    import nest_asyncio  
    # 仅在Jupyter/IPython环境中运行时应用  
    try:  
        from IPython import get_ipython  
        if get_ipython() isnotNone:  
            nest_asyncio.apply()  
    except ImportError:  
        pass# 不在Jupyter中，不需要nest_asyncio  
except ImportError:  
    pass# nest_asyncio不可用，无需继续  
  
  
# ===== 配置 =====  
  
supervisor_tools = [ConductResearch, ResearchComplete, think_tool]  
supervisor_model = init_chat_model(model="anthropic:claude-sonnet-4-20250514")  
supervisor_model_with_tools = supervisor_model.bind_tools(supervisor_tools)  
  
# 系统常量  
# 个别研究智能体的最大工具调用迭代次数  
# 这可以防止无限循环并控制每个主题的研究深度  
max_researcher_iterations = 6# 对think_tool + ConductResearch的调用  
  
# 监督者可以启动的最大并发研究智能体数量  
# 这被传递给lead_researcher_prompt以限制并行研究任务  
max_concurrent_researchers = 3  
  
# ===== 监督者节点 =====  
  
asyncdef supervisor(state: SupervisorState) -> Command[Literal["supervisor_tools"]]:  
    """协调研究活动。  
      
    分析研究简报和当前进度以决定：  
    - 需要调查什么研究主题  
    - 是否进行并行研究  
    - 研究何时完成  
      
    参数:  
        state: 当前监督者状态，包含消息和研究进度  
          
    返回:  
        继续到supervisor_tools节点的命令，并更新状态  
    """  
    supervisor_messages = state.get("supervisor_messages", [])  
      
    # 准备带有当前日期和约束的系统消息  
    system_message = lead_researcher_prompt.format(  
        date=get_today_str(),   
        max_concurrent_research_units=max_concurrent_researchers,  
        max_researcher_iterations=max_researcher_iterations  
    )  
    messages = [SystemMessage(content=system_message)] + supervisor_messages  
      
    # 对下一步研究步骤做出决定  
    response = await supervisor_model_with_tools.ainvoke(messages)  
      
    return Command(  
        goto="supervisor_tools",  
        update={  
            "supervisor_messages": [response],  
            "research_iterations": state.get("research_iterations", 0) + 1  
        }  
    )  
  
asyncdef supervisor_tools(state: SupervisorState) -> Command[Literal["supervisor", "__end__"]]:  
    """执行监督者决策 - 要么进行研究要么结束过程。  
      
    处理：  
    - 执行think_tool调用进行战略反思  
    - 为不同主题启动并行研究智能体  
    - 聚合研究结果  
    - 确定研究何时完成  
      
    参数:  
        state: 当前监督者状态，包含消息和迭代计数  
          
    返回:  
        继续监督、结束过程或处理错误的命令  
    """  
    supervisor_messages = state.get("supervisor_messages", [])  
    research_iterations = state.get("research_iterations", 0)  
    most_recent_message = supervisor_messages[-1]  
      
    # 为单一返回模式初始化变量  
    tool_messages = []  
    all_raw_notes = []  
    next_step = "supervisor"# 默认下一步  
    should_end = False  
      
    # 首先检查退出条件  
    exceeded_iterations = research_iterations >= max_researcher_iterations  
    no_tool_calls = not most_recent_message.tool_calls  
    research_complete = any(  
        tool_call["name"] == "ResearchComplete"  
        for tool_call in most_recent_message.tool_calls  
    )  
      
    if exceeded_iterations or no_tool_calls or research_complete:  
        should_end = True  
        next_step = END  
      
    else:  
        # 在决定下一步之前执行所有工具调用  
        try:  
            # 将think_tool调用与ConductResearch调用分开  
            think_tool_calls = [  
                tool_call for tool_call in most_recent_message.tool_calls   
                if tool_call["name"] == "think_tool"  
            ]  
              
            conduct_research_calls = [  
                tool_call for tool_call in most_recent_message.tool_calls   
                if tool_call["name"] == "ConductResearch"  
            ]  
  
            # 处理think_tool调用（同步）  
            for tool_call in think_tool_calls:  
                observation = think_tool.invoke(tool_call["args"])  
                tool_messages.append(  
                    ToolMessage(  
                        content=observation,  
                        name=tool_call["name"],  
                        tool_call_id=tool_call["id"]  
                    )  
                )  
  
            # 处理ConductResearch调用（异步）  
            if conduct_research_calls:  
                # 启动并行研究智能体  
                coros = [  
                    researcher_agent.ainvoke({  
                        "researcher_messages": [  
                            HumanMessage(content=tool_call["args"]["research_topic"])  
                        ],  
                        "research_topic": tool_call["args"]["research_topic"]  
                    })   
                    for tool_call in conduct_research_calls  
                ]  
  
                # 等待所有研究完成  
                tool_results = await asyncio.gather(*coros)  
  
                # 将研究结果格式化为工具消息  
                # 每个子智能体在result["compressed_research"]中返回压缩的研究发现  
                # 我们将此压缩研究作为ToolMessage的内容写入，这允许  
                # 监督者稍后通过get_notes_from_tool_calls()检索这些发现  
                research_tool_messages = [  
                    ToolMessage(  
                        content=result.get("compressed_research", "合成研究报告时出错"),  
                        name=tool_call["name"],  
                        tool_call_id=tool_call["id"]  
                    ) for result, tool_call in zip(tool_results, conduct_research_calls)  
                ]  
                  
                tool_messages.extend(research_tool_messages)  
  
                # 聚合所有研究的原始笔记  
                all_raw_notes = [  
                    "\n".join(result.get("raw_notes", []))   
                    for result in tool_results  
                ]  
                  
        except Exception as e:  
            print(f"监督者工具中的错误: {e}")  
            should_end = True  
            next_step = END  
      
    # 带有适当状态更新的单一返回点  
    if should_end:  
        return Command(  
            goto=next_step,  
            update={  
                "notes": get_notes_from_tool_calls(supervisor_messages),  
                "research_brief": state.get("research_brief", "")  
            }  
        )  
    else:  
        return Command(  
            goto=next_step,  
            update={  
                "supervisor_messages": tool_messages,  
                "raw_notes": all_raw_notes  
            }  
        )  
  
# ===== 图构建 =====  
  
# 构建监督者图  
supervisor_builder = StateGraph(SupervisorState)  
supervisor_builder.add_node("supervisor", supervisor)  
supervisor_builder.add_node("supervisor_tools", supervisor_tools)  
supervisor_builder.add_edge(START, "supervisor")  
supervisor_agent = supervisor_builder.compile()
```

```
png_data = supervisor_agent.get_graph(xray=True).draw_mermaid_png()  
with open("supervisor_agent.png", "wb") as f:  
    f.write(png_data)  
# 运行多智能体监督器智能体  
  
research_brief = """我需要研究北京最好的10家咖啡店，基于咖啡质量的四个核心标准：价格、环境、服务、位置。具体来说：  
  
1. 价格维度：分析每家咖啡店的咖啡价格区间，包括不同饮品（如美式、拿铁、手冲等）的价格水平，但用户未指定具体的预算约束，因此考虑所有价格范围  
  
2. 环境维度：评估咖啡店的装修风格、座位舒适度、空间布局、噪音水平、整体氛围等环境因素  
  
3. 服务维度：考察员工专业程度、服务态度、出餐速度、个性化服务体验等服务质量指标  
  
4. 位置维度：分析咖啡店的地理位置便利性，包括交通可达性、周边环境、是否靠近商业区或景点等  
  
我需要收集这10家咖啡店的详细信息，包括但不限于：  
- 每家店的具体地址和联系方式  
- 营业时间  
- 价格菜单和饮品选择  
- 环境照片或描述  
- 顾客评价和评分  
- 特色咖啡和服务  
  
优先考虑来自官方咖啡店网站、大众点评、美团等本地生活平台，以及咖啡爱好者社区的真实评价和信息。研究应该基于2025年的最新数据，确保信息的时效性和准确性。"""  
  
  
asyncdef main():  
  
    result = await supervisor_agent.ainvoke({"supervisor_messages": [HumanMessage(content=f"{research_brief}.")]})  
    print(result)  
    format_messages(result['supervisor_messages'])  
  
  
# 运行异步主函数  
if __name__ == "__main__":  
    asyncio.run(main())
```

* Research的流程图
* Supervisor的流程图第一次是搜索北京一些咖啡店，然后第二次搜索重点关注包含具体排名顺序、评分、和推荐理由的完整列表，这个时候信息可起来比较不错：

不过只有8家信息，这个时候模型给出了反馈和说明，给出了一个8家咖啡店排名的结果，但是还是缺少两家

通过调试来看，用于研究迭代超过了最大次数6，然后智能体执行到下面就停止了，

## **关键特性**

### **1. 上下文隔离**

* 每个子智能体都有独立的上下文窗口
* 避免了单一智能体处理多主题时的上下文冲突
* 提高了处理复杂多面向查询的质量

### **2. 并行处理**

* 监督者可以同时启动多个研究智能体
* 显著提高了比较类和多主题研究的效率
* 通过异步处理优化了执行时间

### **3. 智能任务分配**

* 监督者评估查询复杂性来决定是否需要多个智能体
* 为简单任务偏向单智能体解决方案
* 为比较任务自动分配专门的智能体

### **4. 质量控制**

* 硬限制防止无限循环和过度工具调用
* 在每个阶段进行明确的进度评估
* 在有足够信息时自动停止研究

这个多智能体系统为复杂研究任务提供了一个强大且可扩展的解决方案，同时保持了代码的清晰性和可维护性。

# **DeepResearch的完整智能体搭建**

经过上一篇内容以及本篇内容，我们构建了几个比较关键的节点

* clarify\_with\_user：确定用户研究范围
* write\_research\_brief：写出一个研究计划的简报
* supervisor\_agent：多智能体的监督模式构建，用来指导工具调用、总结反思，下一步动作规划

这节内容我们将之前所有组件结合成一个完整的DeepResearch系统。

这是我们之前的整体研究流程：

我们已经在之前的内容中构建了研究范围界定和多智能体研究功能。

现在，我们将添加最终的报告生成步骤。

## **DeepResearch智能体**

我们可以简单地重用已经构建的组件。

```
"""  
完整的多智能体研究系统  
  
此模块集成了研究系统的所有组件：  
- 用户澄清和范围界定  
- 研究简报生成    
- 多智能体研究协调  
- 最终报告生成  
  
系统编排从初始用户输入到最终报告交付的完整研究工作流程。  
"""  
  
from langchain_core.messages import HumanMessage  
from langgraph.graph import StateGraph, START, END  
  
from deep_research_from_scratch.utils import get_today_str  
from deep_research_from_scratch.prompts import final_report_generation_prompt  
from deep_research_from_scratch.state_scope import AgentState, AgentInputState  
from deep_research_from_scratch.research_agent_scope import clarify_with_user, write_research_brief  
from deep_research_from_scratch.multi_agent_supervisor import supervisor_agent  
  
# ===== 配置 =====  
  
from langchain.chat_models import init_chat_model  
writer_model = init_chat_model(model="openai:gpt-4.1", max_tokens=32000) # model="anthropic:claude-sonnet-4-20250514", max_tokens=64000  
  
# ===== 最终报告生成 =====  
  
from deep_research_from_scratch.state_scope import AgentState  
  
asyncdef final_report_generation(state: AgentState):  
    """  
    最终报告生成节点。  
      
    将所有研究发现综合成一份全面的最终报告  
    """  
      
    notes = state.get("notes", [])  
      
    findings = "\n".join(notes)  
  
    final_report_prompt = final_report_generation_prompt.format(  
        research_brief=state.get("research_brief", ""),  
        findings=findings,  
        date=get_today_str()  
    )  
      
    final_report = await writer_model.ainvoke([HumanMessage(content=final_report_prompt)])  
      
    return {  
        "final_report": final_report.content,   
        "messages": ["这是最终报告：" + final_report.content],  
    }  
  
# ===== 图构建 =====  
# 构建整体工作流程  
deep_researcher_builder = StateGraph(AgentState, input_schema=AgentInputState)  
  
# 添加工作流节点  
deep_researcher_builder.add_node("clarify_with_user", clarify_with_user)  
deep_researcher_builder.add_node("write_research_brief", write_research_brief)  
deep_researcher_builder.add_node("supervisor_subgraph", supervisor_agent)  
deep_researcher_builder.add_node("final_report_generation", final_report_generation)  
  
# 添加工作流边  
deep_researcher_builder.add_edge(START, "clarify_with_user")  
deep_researcher_builder.add_edge("write_research_brief", "supervisor_subgraph")  
deep_researcher_builder.add_edge("supervisor_subgraph", "final_report_generation")  
deep_researcher_builder.add_edge("final_report_generation", END)  
  
# 编译完整工作流程  
agent = deep_researcher_builder.compile()
```

## **递归限制说明**

LangGraph有25步的默认递归限制来防止无限循环。对于需要迭代研究轮次的复杂研究工作流程，需要增加此限制。正如LangGraph的故障排除指南中解释的，递归限制会计算图中的每次节点执行。在我们的多智能体研究系统中：

* **单个研究智能体**：工具调用和压缩可能需要8-12步
* **多智能体监督者**：每个生成的子智能体都会增加额外的步骤
* **迭代研究**：监督者可能进行多轮研究来填补空白
* **完整工作流程**：包括范围界定、研究简报生成、监督和报告生成

我们将递归限制设置为50以适应：

* 需要多轮研究的复杂研究主题
* 并行子智能体执行
* 具有许多工具调用的深度研究
* 从范围界定到最终报告的完整工作流程执行

这允许监督者在初始发现存在空白时进行迭代轮次的研究，确保复杂研究主题的全面覆盖。

```
png_data = supervisor_agent.get_graph(xray=True).draw_mermaid_png()  
with open("full_deepresearch_multi_agent.png", "wb") as f:  
    f.write(png_data)  
  
# 添加导入asyncio模块（如果文件顶部没有的话）  
import asyncio  
  
# 创建异步主函数  
asyncdef main():  
    thread = {"configurable": {"thread_id": "1", "recursion_limit": 50}}  
    result = await full_deepresearch_multi_agent.ainvoke({"messages": [HumanMessage(content="比较Gemini和OpenAI深度研究智能体。")]}, config=thread)  
    print(result)  
    format_messages(result['messages'])  
    from rich.markdown import Markdown  
    # 显示Markdown内容  
    print(Markdown(result["final_report"]))  
      
    # 保存为Markdown文件  
    import os  
    from datetime import datetime  
      
    # 创建文件名（使用当前时间戳）  
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")  
    filename = f"research_report_{timestamp}.md"  
      
    # 保存Markdown内容到文件  
    with open(filename, "w", encoding="utf-8") as f:  
        f.write(result["final_report"])  
      
    print(f"研究报告已保存为: {filename}")  
    return result  
  
# 运行异步主函数  
if __name__ == "__main__":  
    asyncio.run(main())
```

## **系统架构特点**

### **1. 模块化设计**

* **用户澄清和范围界定**：确保研究方向明确
* **研究简报生成**：为后续研究提供清晰指导
* **多智能体协调**：并行处理复杂研究任务
* **最终报告生成**：综合所有发现形成完整报告

### **2. 工作流程编排**

* 线性流程确保每个阶段都建立在前一阶段的基础上
* 状态在各个节点间持续传递和更新
* 检查点机制支持长时间运行的研究任务

### **3. 灵活性和可扩展性**

* 可以根据需要调整递归限制
* 支持不同复杂度的研究任务
* 模块化结构便于维护和扩展

### **4. 质量保证**

* 多阶段验证确保研究质量
* 迭代改进机制处理复杂主题
* 全面的错误处理和资源管理

这个完整的多智能体研究系统提供了从用户输入到最终报告的端到端解决方案，能够处理各种复杂度的研究任务，同时保持高质量的输出和良好的用户体验。

---

## **附录：DeepResearch报告结果**

# **Google Gemini 与 OpenAI 深度研究智能体在生物医学研究中的应用比较**

## **1 概述**

基于对Google Gemini和OpenAI深度研究智能体的全面分析，本报告提供了两者在学术研究应用方面的详细比较，特别关注生物医学领域的专业需求。评估涵盖了功能特性、性能表现、应用场景、成本可用性以及具体需求维度，包括文献分析能力、实验设计建议、数据解释能力、专业可靠性和多模态能力。研究综合了官方技术文档、学术性能基准测试、用户案例研究和成本分析报告，以及生物医学研究者的实际使用反馈。

## **2 Google Gemini 深度研究能力评估**

### **2.1 功能特性**

Google Gemini，特别是其Med-Gemini变体，专为生物医学研究设计，具备强大的多模态数据处理能力。Med-Gemini能够处理文本、图像、视频和电子健康记录等多模态数据，在MedQA美国医学执照考试式基准测试中达到91.1%的准确率，创下新纪录[1]。Gemini 2.5 Pro提供Deep Research深度研究助手功能，支持100万token上下文窗口，并计划扩展到200万token[3]。

在数据分析与假设生成方面，Google Research开发了基于Gemini 2.0的"AI co-scientist"多智能体系统，专门帮助生物医学研究人员生成研究假设和提案。该系统允许科学家输入研究目标、想法和参考文献，然后使用互联网资源进行"self-improving loop"自我改进循环来优化输出。人类生物医学研究人员评估该系统并评价其高于其他非专业AI系统，指出其具有更大的影响潜力和新颖性[1]。

### **2.2 性能表现**

在准确性表现方面，Gemini在医学诊断准确性上前10位鉴别诊断列表中包含正确诊断的比例为76.5%（300/392）[1]。在专业基准测试中，Gemini 2.5 Pro在GPQA（研究生级物理问题评估）上取得84%成绩，显著优于GPT-4o的46%[3]。在MMMU基准测试中达到81.7%，在SWE-Bench基准测试中达到63.8%[3]。

然而，Gemini也存在一些可靠性问题。所有系统都出现了不适当的疾病命名错误，包括过时或拼写错误的术语[1]。Gemini Advanced更不愿意回答医学问题，需要更多提示才能响应，响应率为73.8%，而ChatGPT 4.0直接响应率为89.2%[3]。

### **2.3 多模态能力**

Gemini具有原生多模态功能，能够理解和处理文本、图像、音频、PDF和视频等不同数据类型[5]。专业医疗多模态版本包括：

* Med-Gemini-2D：处理2D医学图像
* Med-Gemini-3D：处理3D扫描如CT成像，生成的放射学报告中有超过一半被确定与放射科医师的建议一致
* Med-Gemini-Polygenic：首个从基因组数据进行疾病预测的语言模型[1]

### **2.4 成本与可用性**

Google Gemini提供多种访问方式：

* 免费版本：Gemini Pro，基本多模态功能，支持文本、语音、图像输入和基本文档分析[4]
* 付费版本：Gemini Advanced通过Google One AI Premium套餐提供，价格为每月19.99美元，包含完整Gemini 1.5 Pro访问权限、Google应用（Gmail、Docs、Sheets、Slides）的AI功能集成、2TB Google Drive存储空间、NotebookLM Plus研究笔记工具[4]

## **3 OpenAI 深度研究智能体评估**

### **3.1 功能特性**

OpenAI深度研究智能体能够自动化多步骤的互联网研究，发现、推理并整合来自网络的见解[2][3]。它生成全面、经过深思熟虑且精心引用的文献综述，输出完全记录，带有清晰的引用和思维摘要[2][3]。该智能体由专门的o3模型驱动，针对网页浏览和数据分析优化，能够处理文本、图像和PDF文件，并使用Python工具绘制图表和分析数据[2][3]。

### **3.2 性能表现**

在性能基准测试中，OpenAI深度研究智能体在"Humanity's Last Exam"测试中获得26.6%的准确率（评估专家级问题的广泛学科）[2][3]。在GAIA基准测试中达到新的最先进水平（SOTA），评估真实世界问题[2][3]。幻觉事实率显著低于现有ChatGPT模型，但有时仍会产生幻觉事实或错误推断[2][3]。

实际案例显示，在日本智能手机市场份额分析中出现重大错误：声称69% iOS/31% Android vs 实际监管数据47% iOS/53% Android[4]。根本限制在于LLM基于训练数据预测下一个token而非真正理解内容，准确率即使从85%提高到90%仍不足用于专业研究，因任何错误需验证整个输出[4]。

### **3.3 多模态能力**

OpenAI深度研究智能体能够处理文本、图像和PDF文件，在互联网上搜索、解释和分析大量多模态内容，支持用户上传文件[2][3]。

### **3.4 成本与可用性**

OpenAI于2025年2月2日推出深度研究功能，由o3模型驱动，针对网页浏览和数据分析优化[2][3]。使用限制为：

* Pro用户：每月250次查询（约$20/月）
* Plus/Team/Enterprise/Edu用户：每月25次查询
* 免费用户：每月5次查询[2][3]

## **4 综合比较分析**

### **4.1 文献分析能力对比**

Gemini在引文精确度方面显著优于GPT-4，其引文正确率达到77.2%，准确率为68.0%，而GPT-4的正确率为54.0%，准确率为49.2%（p < 0.001）[1]。GPT-4生成了更长的引言（平均332.4词 vs Gemini的256.4词，p < 0.001），但包含了更多未引用的事实和假设（1.6次 vs 1.2次，p = 0.001）[1]。

OpenAI Deep Research采用高度迭代的实时研究过程，需要5-30分钟完成研究，提供实时图表、注释和透明的研究流程[2]。Google Gemini Deep Research采用系统化研究方法，需要5-15分钟，生成结构化的文档报告，适合需要静态但全面文档的战略规划[2]。

### **4.2 实验设计建议能力**

Gemini集成强化学习与先进神经架构，旨在通过结合语言理解和推理能力来推动AI边界[4]。OpenAI以其GPT系列闻名，专注于设计用于生成类人文本、代码甚至图像的大型语言模型（LLMs）[4]。Gemini的集成强化学习和多模态能力通常在需要推理和上下文理解的任务上产生卓越性能[4]。

### **4.3 数据解释能力**

ChatGPT-4和Gemini在英语中的表现均优于阿拉伯语，ChatGPT-4在正确率和CLEAR评分方面始终优于Gemini[3]。英语表现：ChatGPT-4正确率80% vs Gemini 62.5%[3]。阿拉伯语表现：ChatGPT-4正确率65% vs Gemini 55%[3]。两种AI在低阶认知领域表现更好[3]。

### **4.4 专业可靠性**

两种模型都产生了伪造证据，限制了它们在参考文献搜索中的可靠性[1]。幻觉率数据显示：GPT-3.5为39.6%，GPT-4为28.6%，Bard为91.4%[6]。GPT-5将幻觉率大幅降低至1.4%，优于GPT-4（1.8%）和GPT-4o（1.49%）[5]。研究强调了验证AI生成学术内容的必要性[1]。

### **4.5 多模态能力**

Gemini AI由Google DeepMind开发，强调多模态AI（处理文本、图像、音频和视频）、AI对齐（符合人类价值观）和复杂推理能力[2]。Gemini采用多模态方法，处理各种数据格式包括文本、图像和音频，并专注于解决医疗保健、机器人和自主系统等领域的复杂现实世界挑战[2]。OpenAI主要关注文本生成和快速部署的商业应用，而Gemini专注于多模态数据处理和长期研究[2]。

### **4.6 成本效益分析**

Google Gemini Deep Research每月仅需20美元，而OpenAI Deep Research对Pro用户显著更贵，为每月200美元[2]。对于每个Gemini文本提示，AI推理现在消耗0.24 Wh，与一年前相比，能源消耗减少了33倍，碳足迹降低了44倍[5]。OpenAI Deep Research是深度结构化分析的最佳选择，但成本高且处理时间长[2]。Google Gemini Deep Research仍然是偏好直接研究输出且价格更低的用户的强大竞争者[2]。

### **4.7 学术机构采用情况**

90%的教职员工现在使用AI工具进行教学和研究活动[5]。尽管采用广泛，只有30%的教师接受过正式的AI培训[5]。自2022年以来，AI研究出版物增长了400%[5]。85%的大学制定了AI政策，但50%面临预算限制[5]。ChatGPT主要用于一般研究（36-37%）、学术研究（18-19%）和编程（14-15%）[5]。65%的营销人员和63%的开发人员定期使用[5]。

### **4.8 生物医学领域专业性能**

Gemini在生成可信医学研究引文方面表现更好[1]。ChatGPT-4和Gemini在病毒学多选题（MCQ）中英语和阿拉伯语回答的表现对比显示，ChatGPT-4在正确率和CLEAR评分方面始终优于Gemini[3]。两种AI模型在英语中的表现均优于阿拉伯语[3]。研究结果显示，Gemini和ChatGPT-4都在需要高级批判性思维和问题解决技能的高阶认知MCQ方面遇到困难[3]。

## **5 结论与建议**

基于全面分析，Google Gemini和OpenAI深度研究智能体在生物医学研究应用中各有优势。Gemini在医学专业内容处理、引文准确性和多模态能力方面表现更佳，特别适合生物医学文献分析和诊断支持任务。其成本效益更高，每月20美元的定价使其更适合长期学术使用。OpenAI在一般研究任务、英语内容生成和实时研究过程方面表现更好，幻觉率更低，但成本显著更高。

对于生物医学研究机构，推荐根据具体需求选择：如果需要处理专业医学文献、多模态数据和需要高引文准确性，Gemini是更好的选择；如果注重一般研究能力、实时分析过程和较低幻觉率，OpenAI可能更合适。无论选择哪种工具，都需要研究者进行人工验证和结果审核，特别是在专业医学领域。

### **来源**

[1] Advancing medical AI with Med-Gemini - Google Research: https://research.google/blog/advancing-medical-ai-with-med-gemini/

[2] Introducing deep research - OpenAI: https://openai.com/index/introducing-deep-research/

[3] Google Gemini Advanced 2025 - Baytech Consulting: https://www.baytechconsulting.com/blog/google-gemini-advanced-2025

[4] Google Gemini Pricing Explained: How Much Does it Cost? - AI Tools: https://www.godofprompt.ai/blog/google-gemini-pricing?srsltid=AfmBOoq3gftc\_vUHH7VFsUhwK1lIT0lG7ELIrBlcaC2q-raw1m0X7AOC

[5] Gemini's Performance in Academic Writing: User Feedback and Analysis: https://www.arsturn.com/blog/delving-into-user-feedback-on-geminis-performance-for-academic-writing-research-tasks

[6] Making AI Generative for Higher Education: https://sr.ithaka.org/publications/making-ai-generative-for-higher-education/

[7] Performance of Advanced Large Language Models (GPT-4o, GPT-4, Gemini 1.5 Pro, Claude 3 Opus) on the Japanese National Medical Examination: A Comparative Study: https://www.medrxiv.org/content/10.1101/2024.07.09.24310129v1.full-text

[8] A comparative study of openAI's GPT-4 and Google's gemini - PubMed: https://pubmed.ncbi.nlm.nih.gov/39667055/

[9] OpenAI vs Google: Who Does Deep Research Better?: https://www.analyticsvidhya.com/blog/2025/02/openai-vs-google-who-does-deep-research-better/

[10] The performance of OpenAI ChatGPT-4 and Google Gemini in ...: https://bmcresnotes.biomedcentral.com/articles/10.1186/s13104-024-06920-7

添加微信，备注”**LLM**“进入大模型技术交流群

> 如果你觉得这篇文章对你有帮助，别忘了点个赞、送个喜欢

>/ 作者：致Great

>/ 作者：欢迎转载，标注来源即可

---

> 更多学习资源

[LangGraph结构化输出详解：让智能体返回格式化数据](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247495182&idx=1&sn=ab776caab4483b060caa2aec36ce3349&scene=21#wechat_redirect)

[Agent实战教程：深度解析async异步编程在Langgraph中的性能优化](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247495169&idx=1&sn=b625be26a48722d64c58a6f4955ad9cb&scene=21#wechat_redirect)

[Agent实战教程：Langgraph的StateGraph以及State怎么用](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247495153&idx=1&sn=fddcd7bdaa2885774732520a3385753a&scene=21#wechat_redirect)

[Agent实战教程：LangGraph核心概念节点、边以及状态详解](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247495129&idx=1&sn=44409c4ca1e7d4db1a0ef8a83db21949&scene=21#wechat_redirect)

[Agent实战教程：LangGraph关于智能体的架构模式与核心概念](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247495134&idx=1&sn=3797452e6e5ed51e5de51f585402d0c5&scene=21#wechat_redirect)

[Agent实战教程：LangGraph中工作流与智能体区别与实现](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247495116&idx=1&sn=41efb6be3403424c3f4fc2da7f4410c0&scene=21#wechat_redirect)

[Agent实战教程：LangGraph相关概念介绍以及快速入门](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247495068&idx=1&sn=a1b85990ea29bb0c0f71e3786e6d58a3&scene=21#wechat_redirect)

[Deep Agents：用于复杂任务自动化的 AI 代理框架](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247495010&idx=1&sn=9315b7a5c66e6472e3386c0b077bbda5&scene=21#wechat_redirect)

[一图概览2024年到2025年AI Agent的发展趋势](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494998&idx=1&sn=5941cc9906ad8d4f083d72a754c5b6f0&scene=21#wechat_redirect)

[警惕AI智能体构建误区：生产级系统的实战经验分享](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494902&idx=1&sn=bbe7952784edb8087cc79cb3ac77fd76&scene=21#wechat_redirect)

[AI代理的上下文工程：构建Manus的经验教训](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494870&idx=1&sn=063772116f0ea17ba1dd3a999643a242&scene=21#wechat_redirect)

[基于Gemini API进行大模型函数调用的指南与经验总结](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494741&idx=1&sn=61441837814ab027422db17f9466226a&scene=21#wechat_redirect)

[Kimi K2智能体能力的技术突破：大规模数据合成 + 通用强化学习](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494722&idx=1&sn=b8868862267a8a08d2b72d1d9c72c382&scene=21#wechat_redirect)

[构建AI Agent的完整实战指南：从邮件助手案例看6步落地方法](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494713&idx=1&sn=cf29043ec6e2cd0a39dc11a34f8df330&scene=21#wechat_redirect)

[企业级AI智能体系统的5种核心工作流模式](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494708&idx=1&sn=adfddac739b76a7307b9495dbf63fec9&scene=21#wechat_redirect)

[Context Engineering：从Prompt Engineering到上下文工程的演进](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494673&idx=1&sn=655cb32b062d32c2df04b75b3fb03c18&scene=21#wechat_redirect)

[如何用LangGraph打造Web Research多智能体系统](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494600&idx=1&sn=13d9186e0feb80d4d8d09df14ff3474e&scene=21#wechat_redirect)

[智能体框架：11 个顶级 AI Agent 框架！](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494594&idx=1&sn=4bdd0b0bf64fa16b32b6b4b605c54256&scene=21#wechat_redirect)

[Anthropic关于智能体的经验分享：如何构建高效的Agent？](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494464&idx=1&sn=90cf04ad1824b9320fb394f97080205d&scene=21#wechat_redirect)

[AI 智能体框架对比表](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494325&idx=2&sn=57ee28c4a4e9373ac699ae0c82dff3c9&scene=21#wechat_redirect)

[Gemini开源项目DeepResearch：基于LangGraph的智能研究Agent技术原理与实现](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494255&idx=1&sn=f2a8db81e74d4d999f1f73b842951ddd&scene=21#wechat_redirect)

[智能体卷疯了，又一款Agent框架开源了Lemon AI](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494229&idx=1&sn=2a6d2052af214a043f006c7c14062cb8&scene=21#wechat_redirect)

[大模型 Agent 就是文字艺术吗？](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494174&idx=1&sn=5e74db42c52096e1f6996e55d4c63d47&scene=21#wechat_redirect)

[xAI 把 Grok 的系统提示词全部公开了，我们看看DeepResearch的系统提示词怎么设计的?](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247494081&idx=1&sn=4e9eff8e847188bcaa15e165ca7a3d4f&scene=21#wechat_redirect)

[OpenAI API JSON格式指南与json\_repair错误修复](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247493958&idx=1&sn=7aa82040053d123a1776e18d0376b7fb&scene=21#wechat_redirect)

[txtai：全能AI框架](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247493760&idx=1&sn=3c7d07fecadc2ac1cff73c19300023ba&scene=21#wechat_redirect)

[Suna -开源智能体助手](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247493697&idx=1&sn=625aac93b103d914d80b941900618266&scene=21#wechat_redirect)

[谷歌的A2A到底是什么东西？](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247493463&idx=2&sn=a5944a02d20c1eeeae8f6a6281c149f0&scene=21#wechat_redirect)

[如何在Agent中设置Memory](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247493408&idx=1&sn=3813dbabe9082c4272f109487a800815&scene=21#wechat_redirect)

[体验智能体构建过程：从零开始构建Agent](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247493390&idx=1&sn=3567bba8802a26f40648943fd4aa4c9e&scene=21#wechat_redirect)

[AI代理是大模型实现可扩展智能自动化的关键](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247493371&idx=1&sn=83d9d413a1ec7d6d028cae73db9acd99&scene=21#wechat_redirect)

[Agent系列教程01-什么是Agent？当今为什么这么重要？](https://mp.weixin.qq.com/s?__biz=MzAxOTU5NTU4MQ==&mid=2247493364&idx=1&sn=07e7f5da1f862fdd6e778d462cb2f4f6&scene=21#wechat_redirect)