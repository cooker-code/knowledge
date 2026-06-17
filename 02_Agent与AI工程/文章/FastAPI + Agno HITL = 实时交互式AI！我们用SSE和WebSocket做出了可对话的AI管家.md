---
title: FastAPI + Agno HITL = 实时交互式AI！我们用SSE和WebSocket做出了可对话的AI管家
author: AI 博物院
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIzNTcxOTU3OA==&mid=2247487480&idx=1&sn=1e9f08395ef01d8191f09394210a76d8&chksm=e94f07bbe0f7788cc59ddd42285166a2e2112e7104b38f5b4ed3d4cf3759c85a953774e30f8b&mpshare=1&scene=24&srcid=1117Wzy7HuB54JIl4MJ6sZYE&sharer_shareinfo=5b9b2dcffc26c5f1fa5f2ec2d325a5d7&sharer_shareinfo_first=5b9b2dcffc26c5f1fa5f2ec2d325a5d7#rd
---

在AI Agent的实际应用中，完全自主的决策往往存在风险。特别是在涉及敏感操作、重要决策或关键业务流程时，人类的监督和干预显得尤为重要。Agno框架作为一个高性能的多智能体开发框架，在1.5.4版本中引入了强大的Human-in-Loop（HITL）功能，让开发者能够优雅地实现人机协作的智能体系统。

本文将深入探讨Agno框架的HITL实现机制、流式输出架构，并提供生产级的代码示例，帮助开发者快速构建可控、高效的智能体应用。

## 一、Agno框架概述

### 1.1 核心特性

Agno是一个轻量级、高性能的智能体框架，具有以下突出特点：

* **极致性能**：智能体实例化仅需~2μs，内存占用仅~3.75KiB，比LangGraph快10,000倍
* **原生多模态**：支持文本、图像、音频、视频的输入和输出
* **推理优先**：内置三种推理方法（推理模型、推理工具、自定义思维链）
* **团队协作**：支持多智能体团队架构，实现专业化分工
* **生产就绪**：提供预构建的FastAPI路由，快速部署上线

### 1.2 架构设计理念

Agno采用模块化、可组合的设计理念，每个组件都是即插即用的模块：

```
# 简洁的声明式接口  
from agno.agent import Agent  
from agno.models.openai import OpenAIChat  
  
agent = Agent(  
    model=OpenAIChat(id="gpt-4o"),  
    tools=[...],  
    memory=True,  
    reasoning=True  
)
```

## 二、Human-in-Loop机制详解

### 2.1 HITL的设计模式

Agno提供了四种主要的HITL控制流模式：

#### 2.1.1 用户确认流（User Confirmation Flow）

对于敏感操作，在执行前要求用户确认：

```
from agno.tools import tool  
from agno.agent import Agent  
from agno.models.openai import OpenAIChat  
  
@tool(requires_confirmation=True)  
def delete_database(db_name: str) -> str:  
    """删除数据库的敏感操作"""  
    # 实际的删除逻辑  
    returnf"已成功删除数据库: {db_name}"  
  
@tool(requires_confirmation=True)  
def transfer_funds(amount: float, to_account: str) -> str:  
    """转账操作，需要用户确认"""  
    # 转账逻辑  
    returnf"已转账 ${amount} 到账户 {to_account}"  
  
# 创建需要确认的智能体  
agent = Agent(  
    model=OpenAIChat(id="gpt-4o"),  
    tools=[delete_database, transfer_funds],  
    description="财务管理助手",  
    markdown=True  
)  
  
# 执行需要确认的操作  
response = agent.run("删除test_db数据库")  
  
# 处理确认流程  
if agent.is_paused:  
    for tool in agent.run_response.tools_requiring_confirmation:  
        print(f"工具 {tool.tool_name} 需要确认")  
        print(f"参数: {tool.tool_args}")  
          
        # 获取用户确认  
        confirmed = input("确认执行？(y/n): ").lower() == "y"  
        tool.confirmed = confirmed  
      
    # 继续执行  
    response = agent.continue_run()  
    print(response.content)
```

#### 2.1.2 用户输入流（User Input Flow）

当需要收集额外信息时，暂停执行并等待用户输入：

```
from typing import List  
from agno.tools import tool  
from agno.tools.function import UserInputField  
  
@tool(requires_user_input=True, user_input_fields=["to_address", "cc_addresses"])  
asyncdef send_email(subject: str, body: str, to_address: str, cc_addresses: str = None) -> str:  
    """  
    发送邮件，需要用户提供收件人地址  
      
    Args:  
        subject: 邮件主题  
        body: 邮件正文  
        to_address: 收件人地址（需要用户输入）  
        cc_addresses: 抄送地址（可选）  
    """  
    cc_text = f"，抄送给 {cc_addresses}"if cc_addresses else""  
    returnf"已发送邮件到 {to_address}{cc_text}，主题：{subject}"  
  
# 异步执行智能体  
import asyncio  
  
agent = Agent(  
    model=OpenAIChat(id="gpt-4o-mini"),  
    tools=[send_email],  
    markdown=True,  
    debug_mode=True  
)  
  
asyncdef handle_user_input():  
    await agent.arun("发送会议通知邮件，主题是'季度总结会议'")  
      
    if agent.is_paused:  
        for tool in agent.run_response.tools_requiring_user_input:  
            input_schema: List[UserInputField] = tool.user_input_schema  
              
            for field in input_schema:  
                print(f"\n需要输入: {field.name}")  
                print(f"类型: {field.field_type.__name__}")  
                print(f"描述: {field.description}")  
                  
                if field.value isNone:  
                    user_value = input(f"请输入 {field.name}: ")  
                    field.value = user_value  
          
        # 提供用户输入后继续执行  
        response = await agent.acontinue_run()  
        print(response.content)  
  
# 运行异步函数  
asyncio.run(handle_user_input())
```

#### 2.1.3 外部工具执行流（External Tool Execution）

标记某些工具在智能体上下文之外执行：

```
@tool(external_execution=True)  
def execute_shell_command(command: str) -> str:  
    """  
    执行shell命令（外部执行）  
    注意：这个函数不会在智能体内部执行  
    """  
    pass# 实际执行逻辑在外部处理  
  
agent = Agent(  
    model=OpenAIChat(id="gpt-4o"),  
    tools=[execute_shell_command],  
    description="系统管理助手"  
)  
  
response = agent.run("列出当前目录的所有文件")  
  
if agent.is_paused:  
    for tool in agent.run_response.tools_to_execute_externally:  
        print(f"需要外部执行: {tool.tool_name}")  
        print(f"参数: {tool.tool_args}")  
          
        # 在外部环境执行  
        import subprocess  
        if tool.tool_name == "execute_shell_command":  
            result = subprocess.run(  
                tool.tool_args['command'],   
                shell=True,   
                capture_output=True,   
                text=True  
            )  
            tool.result = result.stdout  
      
    # 将结果返回给智能体  
    response = agent.continue_run()
```

#### 2.1.4 动态用户输入（Dynamic User Input）

使用UserControlFlowTools让智能体动态决定何时需要用户输入：

```
from agno.tools.user_control_flow import UserControlFlowTools  
  
agent = Agent(  
    model=OpenAIChat(id="gpt-4o"),  
    tools=[UserControlFlowTools()],  
    description="交互式助手",  
    markdown=True  
)  
  
response = agent.run("帮我创建一个项目计划")  
  
while response.is_paused:  
    if response.tools_requiring_user_input:  
        for tool in response.tools_requiring_user_input:  
            # 智能体决定需要什么信息  
            print(f"智能体请求: {tool.description}")  
            user_input = input("您的输入: ")  
            tool.value = user_input  
      
    response = agent.continue_run()
```

### 2.2 工具级别的控制

Agno允许在工具包级别精细控制哪些工具需要确认：

```
from agno.tools.yfinance import YFinanceTools  
from agno.tools.duckduckgo import DuckDuckGoTools  
  
agent = Agent(  
    model=OpenAIChat(id="gpt-4o"),  
    tools=[  
        # 只有股票价格查询需要确认  
        YFinanceTools(  
            requires_confirmation_tools=["get_current_stock_price"],  
            stock_price=True,  
            analyst_recommendations=True  
        ),  
        # 网络搜索不需要确认  
        DuckDuckGoTools()  
    ],  
    description="金融分析助手"  
)
```

## 三、流式输出架构实现

### 3.1 基础流式响应

Agno支持流式输出，提供更好的用户体验：

```
from typing import Iterator  
from agno.agent import Agent, RunResponse  
from agno.models.openai import OpenAIChat  
  
agent = Agent(  
    model=OpenAIChat(id="gpt-4o"),  
    markdown=True,  
    show_tool_calls=True  
)  
  
# 方式1：使用print_response直接流式输出  
agent.print_response(  
    "分析NVDA股票的投资价值",  
    stream=True,  
    show_full_reasoning=True,  
    stream_intermediate_steps=True  
)  
  
# 方式2：获取流式响应迭代器  
run_response: Iterator[RunResponse] = agent.run(  
    "生成季度财务报告",  
    stream=True  
)  
  
for chunk in run_response:  
    if chunk.content:  
        print(chunk.content, end="", flush=True)
```

### 3.2 与Web框架集成的流式输出

#### 3.2.1 FastAPI SSE（Server-Sent Events）实现

```
from fastapi import FastAPI  
from fastapi.responses import StreamingResponse  
from agno.agent import Agent  
from agno.models.openai import OpenAIChat  
import json  
import asyncio  
  
app = FastAPI()  
  
# 创建全局智能体实例  
agent = Agent(  
    model=OpenAIChat(id="gpt-4o"),  
    tools=[...],  
    markdown=True  
)  
  
asyncdef generate_sse_response(query: str):  
    """生成SSE格式的流式响应"""  
    asyncfor chunk in agent.arun(query, stream=True):  
        if chunk.content:  
            # SSE格式：data: {json}\n\n  
            yieldf"data: {json.dumps({'content': chunk.content})}\n\n"  
          
        # 如果需要确认  
        if chunk.is_paused:  
            yieldf"data: {json.dumps({'type': 'confirmation_required', 'tools': [t.dict() for t in chunk.tools_requiring_confirmation]})}\n\n"  
              
@app.post("/chat/stream")  
asyncdef stream_chat(request: dict):  
    """流式聊天接口"""  
    query = request.get("query")  
      
    return StreamingResponse(  
        generate_sse_response(query),  
        media_type="text/event-stream",  
        headers={  
            "Cache-Control": "no-cache",  
            "Connection": "keep-alive",  
            "X-Accel-Buffering": "no"# 禁用Nginx缓冲  
        }  
    )  
  
@app.post("/chat/confirm")  
asyncdef confirm_tool(request: dict):  
    """确认工具执行"""  
    tool_id = request.get("tool_id")  
    confirmed = request.get("confirmed", False)  
      
    # 处理确认逻辑  
    if agent.is_paused:  
        for tool in agent.run_response.tools_requiring_confirmation:  
            if tool.id == tool_id:  
                tool.confirmed = confirmed  
          
        # 继续执行并返回流式响应  
        return StreamingResponse(  
            generate_sse_response(None),  # 继续之前的执行  
            media_type="text/event-stream"  
        )
```

#### 3.2.2 WebSocket实现双向通信

```
from fastapi import WebSocket, WebSocketDisconnect  
import json  
  
@app.websocket("/ws/agent")  
asyncdef websocket_endpoint(websocket: WebSocket):  
    await websocket.accept()  
      
    try:  
        whileTrue:  
            # 接收客户端消息  
            data = await websocket.receive_text()  
            message = json.loads(data)  
              
            if message["type"] == "query":  
                # 处理查询请求  
                asyncfor chunk in agent.arun(message["content"], stream=True):  
                    if chunk.content:  
                        await websocket.send_json({  
                            "type": "content",  
                            "data": chunk.content  
                        })  
                      
                    # 处理HITL  
                    if chunk.is_paused:  
                        await websocket.send_json({  
                            "type": "confirmation_required",  
                            "tools": [  
                                {  
                                    "id": tool.id,  
                                    "name": tool.tool_name,  
                                    "args": tool.tool_args  
                                }  
                                for tool in chunk.tools_requiring_confirmation  
                            ]  
                        })  
                          
            elif message["type"] == "confirm":  
                # 处理确认消息  
                tool_id = message["tool_id"]  
                confirmed = message["confirmed"]  
                  
                for tool in agent.run_response.tools_requiring_confirmation:  
                    if tool.id == tool_id:  
                        tool.confirmed = confirmed  
                  
                # 继续执行  
                asyncfor chunk in agent.acontinue_run(stream=True):  
                    if chunk.content:  
                        await websocket.send_json({  
                            "type": "content",  
                            "data": chunk.content  
                        })  
                          
    except WebSocketDisconnect:  
        print("Client disconnected")
```