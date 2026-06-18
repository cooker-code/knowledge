> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020117_DataAgent/020117_核心知识点/Jupyter数据分析Agent工作流边界|Jupyter数据分析Agent工作流边界]]
---
title: Multi-Agent实战：自动化数据分析
author: AI出海Zack
date:
url: https://mp.weixin.qq.com/s?__biz=MzY0MDIxNTE4Nw==&mid=2247483831&idx=1&sn=5f780d1f8fe95e62a8672a9f103b09b4&chksm=f1113b635ae430b72f112fa6b2fc28f8e34ddf9b7ea983aad7b9499adc6f54f9e15e18d26c7d&mpshare=1&scene=24&srcid=1210xrj1wn2sxiNnRmHiHoBW&sharer_shareinfo=7301c18d88caabdecccd94524b42f714&sharer_shareinfo_first=7301c18d88caabdecccd94524b42f714#rd
---

业务方提出一个数据需求，你需要反复沟通理解需求、编写复杂的SQL查询、清洗分析数据、制作可视化图表，最后还要撰写分析报告。整个流程下来，一个简单的需求可能就要花费几个小时。能不能让AI自动完成这一切？

## 流水线式架构

要实现自动化数据分析，我们需要构建一个**流水线式协作系统**，将复杂任务拆解为多个专业环节：

| 传统方式 | Multi-Agent方式 |
| --- | --- |
| 一个人完成所有步骤 | 5个专家Agent各司其职 |
| 串行执行，出错需返工 | 流水线传递，状态可追溯 |
| 依赖人工判断 | 自动规划和执行 |
| 难以复用和优化 | 模块化，易扩展 |

**Agent职责划分**

**- Planner（规划器）**：理解用户需求，制定分析计划

**- SQL（查询执行器）**：生成并执行SQL查询语句

**- Analysis（分析师）**：对数据进行统计分析和解读

**- Viz（可视化专家）**：创建图表和可视化

**- Report（报告生成器）**：整合信息生成最终报告

## 系统设计

### 流程

```
用户需求 → Planner → SQL → Analysis → Viz → Report → 最终报告 

           (规划)   (查询)  (分析)   (图表) (生成)
```

每个Agent接收上一个Agent的输出作为输入，并将自己的输出传递给下一个Agent。通过LangGraph管理整个状态流转。

### 状态定义

```
#... import ... 

 

class AnalysisState(TypedDict): 

    """分析流程的状态""" 

    user_query: str              # 用户原始需求 

    analysis_plan: str           # 分析计划 

    sql_query: str               # 生成的SQL 

    query_result: List[dict]     # 查询结果 

    analysis_text: str           # 分析结论 

    visualization_path: str      # 图表路径 

    final_report: str            # 最终报告 

    error: Optional[str]         # 错误信息
```

## 示例代码

### Planner Agent

```
#import ...

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0) 

 

def planner_agent(state: AnalysisState) -> AnalysisState: 

    """规划器：理解需求并制定分析计划""" 

    system_prompt = """你是数据分析规划专家。根据用户需求，制定清晰的分析计划。
    
    输出格式：
    1. 分析目标：[明确要回答什么问题]
    #...
    """ 

     

    messages = [ 

        SystemMessage(content=system_prompt), 

        HumanMessage(content=f"用户需求：{state['user_query']}") 

    ] 

     

    response = llm.invoke(messages) 

    state['analysis_plan'] = response.content 

    

    return state
```

### SQL Agent

```
#import ...

 

def sql_agent(state: AnalysisState) -> AnalysisState: 

    """SQL执行器：生成并执行SQL查询""" 

    system_prompt = """你是SQL专家。根据分析计划生成SQL查询语句。
    
    数据库schema：
    - sales表：id, product, category, amount, date, region
    
    要求：
    1. 只输出SQL语句，不要其他文字
    #...
    """ 

     

    messages = [ 

        SystemMessage(content=system_prompt), 

        HumanMessage(content=f"分析计划：{state['analysis_plan']}") 

    ] 

     

    response = llm.invoke(messages) 

    sql = response.content.strip() 

     

    # 清理SQL

    #....

     

    state['sql_query'] = sql 

    

     

    # 执行SQL查询 

    #...

    #    state['query_result'] = df.to_dict('records') 

    #...

     

    return state
```

### Analysis Agent

```
def analysis_agent(state: AnalysisState) -> AnalysisState: 

    """分析师：对数据进行统计分析和解读""" 

    if state.get('error'): 

        return state 

     

    # 将查询结果转换为可读格式 

    df = pd.DataFrame(state['query_result']) 

    data_summary = df.describe().to_string() if len(df) > 0 else "无数据" 

     

    system_prompt = """你是数据分析专家。分析查询结果，提供深入洞察。
    
    分析要点：
    1. 关键指标总结
    #....
    """ 

     

    messages = [ 

        SystemMessage(content=system_prompt), 

        HumanMessage(content=f"""
分析计划：{state['analysis_plan']}

#...
        """) 

    ] 

     

    response = llm.invoke(messages) 

    state['analysis_text'] = response.content 

    

    return state
```

### Visualization Agent

```
import matplotlib.pyplot as plt 

import matplotlib 

matplotlib.use('Agg')  # 非GUI后端 

 

def viz_agent(state: AnalysisState) -> AnalysisState: 

    """可视化专家：创建数据图表""" 

    if state.get('error'): 

        return state 

     

    df = pd.DataFrame(state['query_result']) 

    if len(df) == 0: 

        state['visualization_path'] = "" 

        return state 

     

    # 自动选择图表类型 

    numeric_cols = df.select_dtypes(include=['number']).columns.tolist() 

     

    try: 

        plt.figure(figsize=(10, 6)) 

         

        if len(numeric_cols) >= 1 and len(df) <= 20: 

            # 条形图：适合分类对比 

            df.plot(kind='bar', x=df.columns[0], y=numeric_cols[0], ax=plt.gca()) 

            plt.title('数据对比分析') 

            plt.xticks(rotation=45, ha='right') 

        elif len(numeric_cols) >= 2: 

            # 折线图：适合趋势分析 

            df.plot(x=df.columns[0], y=numeric_cols[:2], ax=plt.gca()) 

            plt.title('趋势分析') 

        else: 

            # 饼图：适合占比分析 

            df[df.columns[0]].value_counts().plot(kind='pie', autopct='%1.1f%%') 

            plt.title('分布占比') 

         

        plt.tight_layout() 

        viz_path = 'analysis_chart.png' 

        plt.savefig(viz_path, dpi=100, bbox_inches='tight') 

        plt.close() 

         

        state['visualization_path'] = viz_path 

    except Exception as e: 

        state['visualization_path'] = "" 

     

    return state
```

### Report Agent

```
def report_agent(state: AnalysisState) -> AnalysisState: 

    """报告生成器：整合所有信息生成最终报告""" 

    if state.get('error'): 

        state['final_report'] = f"分析失败：{state['error']}" 

        return state 

     

    report = f"""
# 报告格式定义

#....

"""

    state['final_report'] = report.strip()

    return state
```

## LangGraph工作流

```
def should_continue(state: AnalysisState) -> str: 

    """决定是否继续流程""" 

    if state.get('error'): 

        return "report"  # 有错误直接跳到报告生成 

    return "continue" 

 

# 创建状态图 

workflow = StateGraph(AnalysisState) 

 

# 添加节点 

workflow.add_node("planner", planner_agent) 

workflow.add_node("sql", sql_agent) 

workflow.add_node("analysis", analysis_agent) 

workflow.add_node("viz", viz_agent) 

workflow.add_node("report", report_agent) 

 

# 定义边 

workflow.set_entry_point("planner") 

workflow.add_edge("planner", "sql") 

workflow.add_conditional_edges( 

    "sql", 

    should_continue, 

    { 

        "continue": "analysis", 

        "report": "report" 

    } 

) 

workflow.add_edge("analysis", "viz") 

workflow.add_edge("viz", "report") 

workflow.add_edge("report", END) 

 

# 编译工作流 

app = workflow.compile()
```

生成的报告包含：需求说明、分析计划、SQL查询、数据洞察、可视化图表，以及业务建议，完全自动化完成。

## 总结

通过这个Multi-Agent数据分析系统，我们实现了从需求到报告的全自动化流程。

这套系统可以轻松扩展：添加数据清洗Agent、预测分析Agent，或者接入更多数据源。Multi-Agent的模块化设计让复杂任务变得可管理、可优化。