> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: LangChain DeepAgents + MCP 构建深度研究型智能体应用
author: 狂热JAVA小毕超
date:
url: https://mp.weixin.qq.com/s?__biz=MzUxODg5Mjg3Ng==&mid=2247491753&idx=1&sn=8fd6e10ff42b4e7f12850d9e821e5842&chksm=f8db4a18f8f38acbc975a0aceace065dd7c9896800509b795ead725671050a2d2eaf9ab0dec0&mpshare=1&scene=24&srcid=1225KJqc4pmAdN5myCedFX4R&sharer_shareinfo=c068ce72e8c6cd7357a31da01c2a24b9&sharer_shareinfo_first=c068ce72e8c6cd7357a31da01c2a24b9#rd
---

## 一、DeepAgents 深度研究智能体应用

`DeepAgents` 是 `LangChain` 团队开源的一款高性能智能体框架，专为长周期、高复杂度任务设计。基于 `LangChain` 和 `LangGraph` 构建，通过内置的任务规划、文件系统、子智能体等能力，让智能体能够更高效地完成复杂、多步骤的任务，而无需开发者从零搭建底层逻辑。

本文基于 `LangChain DeepAgents` + `Tavily Search API` ， 快速构建一个可深度研究的智能体应用。其中 `Tavily Search API` 采用封装为`MCP Server`的方式提供给智能体调用。

实验使用模型这里采用 `claude-sonnet-4` ，模型来源使用 `OpenRouter`，整体过程如下所示：

关于 `LangChain``DeepAgents` 的详细介绍和使用说明可以参考官方文档和`Github`：

官方介绍文档地址：

> https://docs.langchain.com/oss/python/deepagents/quickstart

Github 地址：

> https://github.com/langchain-ai/deepagents

实验所使用的主要依赖版本如下：

```
tavily-python==0.7.12
mcp==1.9.2
langchain==1.1.3
langchain-mcp-adapters==0.1.4
langgraph==1.0.5
fastapi==0.115.14
```

最终实验后执行效果如下所示，测试问题：埃菲尔铁塔与最高建筑相比有多高？

执行过程部分截图：

研究报告部分截图：

## 二、Tavily Search API  封装 MCP Server

Tavily Search API 介绍文档地址：

> https://docs.tavily.com/documentation/quickstart

封装为 `MCP Server`，这里主要封装两个能力：`Web`网络搜索 和 网页内容查看，整体逻辑如下所示：

```
from mcp.server.fastmcp import FastMCP
from typing import Literal
from tavily import TavilyClient

mcp = FastMCP("Web-Search-Server")

tavily_client = TavilyClient(api_key="tvly-xxxx") ## 修改为你的key


@mcp.tool()
def web_search(query: str, max_results: int = 5,
               topic: Literal["general", "news", "finance"] = "general",
               include_raw_content: bool = False):
    """Run a web search"""
    return tavily_client.search(
        query,
        max_results=max_results,
        include_raw_content=include_raw_content,
        topic=topic,
    )
    
@mcp.tool()
def extract(url: str):
    """Extract web page content from URL."""
    return tavily_client.extract(url)

if __name__ == "__main__":
    mcp.settings.port = 6030
    mcp.run("sse")
```

启动 `MCP``Server`

## 三、深度智能体搭建

关于深度智能体的创建和 `LangChain`  之前 `Agent` 的创建几乎相同，只需使用 `deepagents` 下的 `create_deep_agent` ，参数和使用方式和之前也几乎相同，关于`MCP`工具的使用，也可以直接使用 `MultiServerMCPClient` 抽取的 `MCP Tools` ，实现过程如下所示：

```
import os, config

from langchain_core.messages import  AIMessageChunk

os.environ["OPENAI_BASE_URL"] = "https://openrouter.ai/api/v1"  ## 支持openai协议的 API BASE
os.environ["OPENAI_API_KEY"] = "sk-or-v1-xxxx"  ## 你的API KEY
from deepagents import create_deep_agent
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain.chat_models import init_chat_model
import asyncio

system_prompt = """
你是一位研究专家。你的工作是根据用户的要求进行彻底的研究，然后写一份润色的报告。
## 工作流程
1. 理解核心需求，识别关键要素。
2. 制定任务规划，明确待办事项。
3. 逐步研究，记录发现，验证结论。
4. 整体结果和审查, 必要时重新制定解决链路。
5. 输出详细研究报告。
"""


async def main():
    ## 初始化模型
    llm = init_chat_model(
        f"openai:anthropic/claude-sonnet-4",
        temperature=0
    )
    ## 定义MCP Server, 执行前面创建的
    client = MultiServerMCPClient(
        {
            "web-search": {
                "url": "http://localhost:6030/sse",
                "transport": "sse"
            }
        }
    )
    ## 获取 MCP Tools
    tools = await client.get_tools()
    ## 创建深度智能体
    agent = create_deep_agent(
        model=llm,
        tools=tools,
        system_prompt=system_prompt
    )
    ## 执行深度研究，以流式的方式输出
    async for stream_type, chunk in agent.astream(
            input={
                "messages": [
                    {"role": "user", "content": "埃菲尔铁塔与最高建筑相比有多高？"}
                ]
            },
            stream_mode=["updates", "messages"]
    ):
        if stream_type == "messages" and type(chunk[0]) is AIMessageChunk:
            content = chunk[0].content
            if not content:
                continue
            print(content, end="", flush=True)
        elif stream_type == "updates":
            if "model" in chunk:
                model = chunk["model"]
                if "messages" in model:
                    messages = model["messages"]
                    for message in messages:
                        tool_calls = message.tool_calls
                        for tool_call in tool_calls:
                            name = tool_call["name"]
                            args = tool_call["args"]
                            if name == "write_todos":
                                todos = args["todos"]
                                print_todos = []
                                for todo in todos:
                                    todo_content = todo["content"]
                                    status = todo["status"]
                                    print_todos.append(f"> {todo_content}  -->  {status}")
                                if print_todos:
                                    print_todos = "\n".join(print_todos)
                                    print(f"\n> TODO: \n{print_todos}\n\n")
                            else:
                                print(f"\n> Call MCP: {name}, args: {args}\n")

            if "tools" in chunk:
                tools = chunk["tools"]
                if "messages" in tools:
                    messages = tools["messages"]
                    for message in messages:
                        content = message.content
                        if "Updated todo list" in content:
                            continue
                        print("\nMCP Result: \n", content, "\n")

if __name__ == '__main__':
    asyncio.run(main())

    
```

运行后可以看到智能体先是指定初步的待办过程，然后逐步执行每个待办：

最后可以看到研究报告：


下面将上述逻辑封装为 OpenAI 协议接口，使用 Cherry Studio 客户端访问，可以更好的进行交互和展示效果。

## 四、封装为 OpenAI 协议接口

上述已经通过控制台实现了 `DeepAgents` 的执行，下面使用 `FastAPI` 封装为 `OpenAI` 协议接口，便于在客户端使用。

注意：这里主要为实现功能，`api_key` 写死在程序中为： `sk-da4b6cb4a41e4cascascasc9508deb556942` （随机生成的）， 后续使用客户端连接时，需要填写该 `api_key` 。

这里由于工具输出内容较多，工具的输出暂时没有吐给客户端，如果需要查看直接将注释放开即可。

整体逻辑如下：

```
import os

from langchain_core.messages import AIMessageChunk

os.environ["OPENAI_BASE_URL"] = "https://openrouter.ai/api/v1"  ## 支持openai协议的 API BASE
os.environ["OPENAI_API_KEY"] = "sk-or-v1-xxxx"  ## 你的API KEY

from fastapi import FastAPI, HTTPException, Header, Depends
from fastapi.responses import StreamingResponse
from pydantic import BaseModel
from typing import List, Dict, Any, Optional, Generator
from langchain.chat_models import init_chat_model
from langchain_mcp_adapters.client import MultiServerMCPClient
from deepagents import create_deep_agent
from datetime import datetime
import time, uuid

app = FastAPI(title="OpenAI Compatible Chat API")
api_key = "sk-da4b6cb4a41e4cascascasc9508deb556942"
agent = None
system_prompt = """
你是一位研究专家。你的工作是根据用户的要求进行彻底的研究，然后写一份润色的报告。
## 工作流程
1. 理解核心需求，识别关键要素。
2. 制定任务规划，明确待办事项。
3. 逐步研究，记录发现，验证结论。
4. 整体结果和审查, 必要时重新制定解决链路。
5. 输出详细研究报告。
"""
llm = init_chat_model(
    f"openai:anthropic/claude-sonnet-4",
    temperature=0
)


class ChatCompletionRequest(BaseModel):
    model: str
    messages: List[Dict[str, Any]]
    temperature: Optional[float] = 1.0
    max_tokens: Optional[int] = None
    stream: Optional[bool] = False


class ChatCompletionChoice(BaseModel):
    index: int
    message: List[Dict[str, Any]]
    finish_reason: str


class ChatCompletionResponse(BaseModel):
    id: str
    object: str = "chat.completion"
    created: int
    model: str
    choices: List[ChatCompletionChoice]
    usage: Dict[str, int]


class ChatCompletionChunk(BaseModel):
    id: str
    object: str = "chat.completion.chunk"
    created: int
    model: str
    choices: List[Dict[str, Any]]


async def create_agent():
    '''创建MCP Agent'''
    client = MultiServerMCPClient(
        {
            "web-search": {
                "url": "http://localhost:6030/sse",
                "transport": "sse"
            }
        }
    )
    tools = await client.get_tools()
    return create_deep_agent(
        model=llm,
        tools=tools,
        system_prompt=system_prompt
    )

async def invoke_agent(messages: []):
    global agent
    if not agent:
        agent = await create_agent()
    async for stream_type, chunk in agent.astream({"messages": messages}, stream_mode=["updates", "messages"]):
        if stream_type == "messages" and type(chunk[0]) is AIMessageChunk:
            content = chunk[0].content
            if not content:
                continue
            yield content
        elif stream_type == "updates":
            if "model" in chunk:
                model = chunk["model"]
                if "messages" in model:
                    messages = model["messages"]
                    for message in messages:
                        tool_calls = message.tool_calls
                        for tool_call in tool_calls:
                            name = tool_call["name"]
                            args = tool_call["args"]
                            if name == "write_todos":
                                todos = args["todos"]
                                print_todos = []
                                for todo in todos:
                                    todo_content = todo["content"]
                                    status = todo["status"]
                                    print_todos.append(f"> {todo_content}  -->  {status}")
                                if print_todos:
                                    print_todos = "\n".join(print_todos)
                                    yield f"\n> TODO: \n{print_todos}\n\n"
                            else:
                                yield f"\n> Call MCP Server: {name}, args: {args}\n\n"
            ## 客户端暂时不输出工具的输出，如果需要，放开注释即可
            # if "tools" in chunk:
            #     tools = chunk["tools"]
            #     if "messages" in tools:
            #         messages = tools["messages"]
            #         for message in messages:
            #             content = message.content
            #             if "Updated todo list" in content:
            #                 continue
            #             yield f"> MCP Response: \n\n```json\n{content}\n```\n"

async def handle_stream_response(messages: [],
                                 model: str, request_id: str) -> Generator[str, None, None]:
    # 执行 agent
    async for msg in invoke_agent(messages):
        chunk_data = ChatCompletionChunk(
            id=request_id,
            created=int(time.time()),
            model=model,
            choices=[{
                "index": 0,
                "delta": {
                    "content": msg
                },
                "finish_reason": None
            }]
        )
        yield f"data: {chunk_data.model_dump_json()}\n\n"

    final_chunk = ChatCompletionChunk(
        id=request_id,
        created=int(time.time()),
        model=model,
        choices=[{
            "index": 0,
            "delta": {},
            "finish_reason": "stop"
        }]
    )
    yield f"data: {final_chunk.model_dump_json()}\n\n"
    yield "data: [DONE]\n\n"


async def verify_auth(authorization: Optional[str] = Header(None)) -> bool:
    '''验证token'''
    if not authorization:
        return False
    if authorization.startswith("Bearer "):
        token = authorization[7:]
    else:
        token = authorization
    return token == api_key


@app.post("/v1/chat/completions")
async def chat_completions(request: ChatCompletionRequest, auth_result: bool = Depends(verify_auth)):
    # 检查身份验证结果
    if not auth_result:
        raise HTTPException(
            status_code=401,
            detail={
                "error": {
                    "message": "Invalid authentication credentials",
                    "type": "invalid_request_error",
                    "param": None,
                    "code": "invalid_api_key"
                }
            },
            headers={"WWW-Authenticate": "Bearer"}
        )
    ## 暂不支持非流式返回
    if not request.stream:
        raise HTTPException(
            status_code=400,
            detail={
                "error": {
                    "message": "Streaming responses are not implemented in this mock API",
                    "type": "invalid_request_error",
                    "param": "stream",
                    "code": "invalid_parameter"
                }
            }
        )
    try:
        # 触发 agent 并流式返回
        request_id = f"chatcmpl-{uuid.uuid4().hex[:8]}"
        return StreamingResponse(
            handle_stream_response(request.messages, request.model, request_id),
            media_type="text/plain",
            headers={
                "Cache-Control": "no-cache",
                "Connection": "keep-alive",
                "Content-Type": "text/event-stream"
            }
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@app.get("/v1/models")
async def list_models():
    """列出可用模型"""
    return {
        "object": "list",
        "data": [
            {
                "id": "agent_model",
                "object": "model",
                "created": int(time.time()),
                "owned_by": "agent"
            }
        ]
    }


@app.get("/health")
async def health_check():
    """健康检查"""
    return {"status": "healthy", "timestamp": datetime.now().isoformat()}


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

启动服务：

## 五、使用 Cherry Studio 连接测试

注意这里不限于 `Cherry Studio` ，可以使用任何支持 `OpenAI` 协议的客户端连接测试，比如 `OpenWebUI`。

这里已 `Cherry Studio` 为例，添加一个 `OpenAI` 连接：

注意：这里`API`秘钥，使用前面提到的 `api key` ：

### 交互测试

> 提问：**埃菲尔铁塔与最高建筑相比有多高？**

执行过程部分截图：

研究报告部分截图：


完整输出内容如下(注意：上面封装`OpenAI`接口时，由于工具输出内容较多，工具的输出暂时没有吐给客户端，如果需要查看直接将注释放开即可)：

```
我来帮您研究埃菲尔铁塔与世界最高建筑的高度对比。这需要查找准确的数据来进行比较。
> TODO: 
> 搜索埃菲尔铁塔的准确高度信息  -->  in_progress
> 搜索世界最高建筑的信息和高度  -->  pending
> 对比分析两者的高度差异  -->  pending
> 整理并撰写详细的对比报告  -->  pending


> Call MCP Server: web_search, args: {'query': '埃菲尔铁塔高度 准确数据 米'}

> TODO: 
> 搜索埃菲尔铁塔的准确高度信息  -->  completed
> 搜索世界最高建筑的信息和高度  -->  in_progress
> 对比分析两者的高度差异  -->  pending
> 整理并撰写详细的对比报告  -->  pending


> Call MCP Server: web_search, args: {'query': '世界最高建筑 哈利法塔 迪拜塔 高度 米'}

> Call MCP Server: web_search, args: {'query': '2024年世界最高建筑排名 最新'}

> TODO: 
> 搜索埃菲尔铁塔的准确高度信息  -->  completed
> 搜索世界最高建筑的信息和高度  -->  completed
> 对比分析两者的高度差异  -->  in_progress
> 整理并撰写详细的对比报告  -->  pending


> TODO: 
> 搜索埃菲尔铁塔的准确高度信息  -->  completed
> 搜索世界最高建筑的信息和高度  -->  completed
> 对比分析两者的高度差异  -->  completed
> 整理并撰写详细的对比报告  -->  in_progress

# 埃菲尔铁塔与世界最高建筑高度对比研究报告

## 研究概述

本报告通过详细研究，对比分析了法国巴黎埃菲尔铁塔与当前世界最高建筑的高度差异，为您提供准确的数据对比。

## 核心发现

### 埃菲尔铁塔高度
- **当前高度**：330米（2022年3月更新）
- **历史变化**：
  - 1889年建成时：312米（包含旗杆）
  - 原始结构：300米（被称为"300米塔"）
  - 2000年增高：324米（安装数字电视天线）
  - 2022年最新增高：330米（安装6米数字音频广播天线）

### 世界最高建筑 - 哈利法塔（迪拜）
- **高度**：828米
- **楼层**：162层（地上）/ 169层（总计）
- **建成时间**：2010年
- **保持记录**：连续14年世界最高建筑（2010-2024年）

## 高度对比分析

### 数据对比
| 建筑名称 | 高度 | 建成年份 | 位置 |
|---------|------|----------|------|
| 哈利法塔 | 828米 | 2010年 | 阿联酋迪拜 |
| 埃菲尔铁塔 | 330米 | 1889年 | 法国巴黎 |
| **高度差** | **498米** | **121年** | - |

### 关键对比指标

1. **绝对高度差异**：498米
   - 哈利法塔比埃菲尔铁塔高出498米
   - 相当于埃菲尔铁塔高度的1.51倍

2. **相对比例**：
   - 哈利法塔是埃菲尔铁塔高度的2.51倍
   - 埃菲尔铁塔仅为哈利法塔高度的39.9%

3. **时代差异**：
   - 两座建筑相隔121年建成
   - 体现了建筑技术的巨大进步

## 历史意义对比

### 埃菲尔铁塔（1889年）
- 建成时是世界最高建筑
- 保持世界最高记录40年（1889-1929年）
- 标志着钢铁结构建筑的里程碑
- 法国文化象征和巴黎地标

### 哈利法塔（2010年）
- 现代超高层建筑技术的巅峰
- 多项世界纪录保持者
- 代表21世纪建筑工程成就
- 迪拜城市发展的象征

## 结论

埃菲尔铁塔虽然在其建成的1889年是世界最高建筑，但与当前世界最高建筑哈利法塔相比，存在显著的高度差异。哈利法塔以828米的高度，比埃菲尔铁塔的330米高出498米，是埃菲尔铁塔高度的2.51倍。

这一对比不仅展现了建筑技术在过去一个多世纪中的巨大进步，也反映了人类对于建造更高建筑的不断追求。埃菲尔铁塔作为19世纪工业革命的杰作，与哈利法塔这一21世纪建筑奇迹，共同见证了人类建筑史的辉煌发展。
> TODO: 
> 搜索埃菲尔铁塔的准确高度信息  -->  completed
> 搜索世界最高建筑的信息和高度  -->  completed
> 对比分析两者的高度差异  -->  completed
> 整理并撰写详细的对比报告  -->  completed

研究已完成！根据最新数据，埃菲尔铁塔目前高度为330米，而世界最高建筑哈利法塔高度为828米。**哈利法塔比埃菲尔铁塔高出498米，是埃菲尔铁塔高度的2.51倍**。

这个巨大的高度差异不仅体现了建筑技术在121年间的飞跃发展，也展现了人类建筑工程能力的显著提升。
```

### 交互测试

> 提问：**深度研究下CSDN小毕超博主**

执行过程部分截图：

研究报告部分截图：

完整输出内容如下：

```
我将深度研究CSDN小毕超博主的相关信息。这是一个复杂的研究任务，需要多个步骤来收集和分析信息。
> TODO: 
> 搜索CSDN小毕超博主的基本信息和简介  -->  in_progress
> 分析小毕超的技术专长和主要内容领域  -->  pending
> 研究其代表性文章和技术贡献  -->  pending
> 分析其在CSDN平台的影响力和数据表现  -->  pending
> 收集其他平台的相关信息（如果有）  -->  pending
> 整理和撰写详细研究报告  -->  pending


> Call MCP Server: web_search, args: {'query': 'CSDN 小毕超 博主 简介 技术专长', 'max_results': 10}


> Call MCP Server: extract, args: {'url': 'https://blog.csdn.net/qq_43692950'}


> TODO: 
> 搜索CSDN小毕超博主的基本信息和简介  -->  completed
> 分析小毕超的技术专长和主要内容领域  -->  in_progress
> 研究其代表性文章和技术贡献  -->  pending
> 分析其在CSDN平台的影响力和数据表现  -->  pending
> 收集其他平台的相关信息（如果有）  -->  pending
> 整理和撰写详细研究报告  -->  pending


> Call MCP Server: web_search, args: {'query': '小毕超 CSDN 博客专家 人工智能 微服务 技术文章', 'max_results': 8}


> Call MCP Server: web_search, args: {'query': '"小毕超" CSDN 博客专家 粉丝数 文章数 影响力', 'max_results': 8}


> TODO: 
> 搜索CSDN小毕超博主的基本信息和简介  -->  completed
> 分析小毕超的技术专长和主要内容领域  -->  completed
> 研究其代表性文章和技术贡献  -->  in_progress
> 分析其在CSDN平台的影响力和数据表现  -->  pending
> 收集其他平台的相关信息（如果有）  -->  pending
> 整理和撰写详细研究报告  -->  pending


> Call MCP Server: web_search, args: {'query': '小毕超 CSDN 热门文章 ChatGPT 知识图谱 微服务 大模型', 'max_results': 10}


> Call MCP Server: web_search, args: {'query': '小毕超 CSDN 博客专家 华为云专家 软考高级 TOGAF 获奖经历', 'max_results': 8}


> TODO: 
> 搜索CSDN小毕超博主的基本信息和简介  -->  completed
> 分析小毕超的技术专长和主要内容领域  -->  completed
> 研究其代表性文章和技术贡献  -->  completed
> 分析其在CSDN平台的影响力和数据表现  -->  in_progress
> 收集其他平台的相关信息（如果有）  -->  pending
> 整理和撰写详细研究报告  -->  pending


> Call MCP Server: web_search, args: {'query': '"小毕超" 阿里云 华为云 技术专家 开发者社区', 'max_results': 8}


> TODO: 
> 搜索CSDN小毕超博主的基本信息和简介  -->  completed
> 分析小毕超的技术专长和主要内容领域  -->  completed
> 研究其代表性文章和技术贡献  -->  completed
> 分析其在CSDN平台的影响力和数据表现  -->  completed
> 收集其他平台的相关信息（如果有）  -->  completed
> 整理和撰写详细研究报告  -->  in_progress

# CSDN小毕超博主深度研究报告

## 一、博主基本信息概览

### 个人简介
**小毕超**（CSDN用户名：qq_43692950）是CSDN平台上的知名技术博主，具有深厚的技术背景和丰富的实践经验。他是一位专注于人工智能、微服务架构领域的专业技术人员，同时也是多个技术认证的持有者。

### 专业资质与认证
- **国内软考高级系统架构设计师职称**
- **国际TOGAF鉴定级企业架构设计师**
- **PMP项目管理师**
- **华为云·云享专家**
- **CSDN博客专家**

### 获奖经历
- "联想杯"移动互联比赛 江苏省一等奖和国家二等奖
- 第四届全国应用型人才比赛"兄弟连杯"国家一等奖

## 二、技术专长与内容领域分析

### 核心技术领域
根据研究发现，小毕超的技术专长主要集中在以下几个方面：

#### 1. 人工智能与机器学习
- **大语言模型应用**：深度研究ChatGPT、Qwen系列、DeepSeek等主流大模型
- **模型微调与训练**：专注于医疗问答、命名实体识别、文本分类等任务的模型微调
- **计算机视觉**：目标检测、YOLO系列算法应用与优化
- **自然语言处理**：知识图谱构建、语义相似性判断、文本解析等

#### 2. 微服务架构
- **Spring Cloud生态**：Spring Cloud Kubernetes、微服务治理
- **容器化技术**：Docker、Kubernetes集群部署与管理
- **服务网格**：微服务间通信与治理
- **API网关**：微服务架构中的流量管理

#### 3. 大数据处理
- **数据仓库技术**：Hive数据处理与分析
- **分布式计算**：Hadoop生态系统应用
- **数据可视化**：结合机器学习的数据分析平台

#### 4. 云原生技术
- **Kubernetes**：集群搭建、多租户管理、KubeSphere应用
- **DevOps**：Jenkins自动化部署、CI/CD流程优化
- **云平台服务**：华为云、阿里云等云服务应用

## 三、代表性技术贡献与文章分析

### 最新技术文章（2024-2025年）
1. **《使用 DeepSeek R1 心理医疗健康数据集 + Qwen2.5-0.5B-Instruct，蒸馏专有领域思考模型》**
   - 展示了在垂直领域应用大模型的创新实践

2. **《基于 Qwen2.5-0.5B 微调训练 Ner 命名实体识别任务》**
   - 深入探讨了小参数模型在特定NLP任务中的应用

3. **《文档解析利器 PaddleOCR-VL 视觉文档解析模型本地部署和测试》**
   - 结合了计算机视觉与文档处理的实用技术

4. **《基于 PyTorch 完全从零手搓 GPT 混合专家 (MOE) 对话模型》**
   - 展现了深度学习模型架构的深度理解

5. **《YOLO 家族全新一代 YOLO v13 上手使用及微调实验》**
   - 紧跟计算机视觉领域最新发展

### 技术创新特点
- **前沿性**：紧跟AI技术发展趋势，及时分享最新模型和技术
- **实用性**：注重理论与实践结合，提供完整的部署和测试方案
- **深度性**：从底层原理到实际应用的全栈技术覆盖
- **系统性**：构建完整的技术体系，形成专栏化内容

## 四、CSDN平台影响力分析

### 平台地位
- **CSDN博客专家**：获得官方认证的技术专家称号
- **Java技术领域优质创作者**：在后端开发技术领域具有专家地位
- **博客之星候选人**：在2022年博客之星排行榜中表现优异

### 内容数据表现
- **专栏数量**：创建了73篇微服务相关文章的专栏，关注人数8W+人
- **技术覆盖面**：涉及机器学习、大模型、微服务、大数据处理等多个技术领域
- **更新频率**：保持高频率的技术文章更新，紧跟技术发展趋势

### 社区影响力
- **技术引领**：在AI大模型应用、微服务架构等领域具有一定的技术引领作用
- **知识分享**：通过详细的技术教程和实践案例，为开发者社区贡献价值
- **跨平台影响**：内容被华为云开发者社区等其他平台转载和引用

## 五、其他平台表现

### 华为云开发者社区
小毕超的技术文章在华为云开发者社区也有广泛传播，包括：
- 《使用kubeadm快速部署一个K8s集群》
- 《Nexus3 搭建私服maven 仓库》
- 《MyCat搭建mysql读写分离介绍，并使用SpringBoot测试》

这些文章的转载表明其技术内容具有较高的实用价值和影响力。

### 技术社区认可
- 被多个技术平台引用和转载
- 在开发者社区中具有一定的知名度
- 技术文章质量得到同行认可

## 六、技术发展趋势与特色

### 技术发展轨迹
1. **早期阶段**：专注于传统后端技术，如Spring、微服务架构
2. **发展阶段**：扩展到大数据处理、容器化技术
3. **当前阶段**：深度投入AI大模型、机器学习应用领域

### 技术特色
- **全栈能力**：从基础架构到AI应用的全技术栈覆盖
- **实践导向**：注重技术的实际应用和落地
- **持续学习**：紧跟技术发展趋势，不断更新知识体系
- **知识传播**：通过高质量的技术文章推动技术普及

## 七、总结与评价

### 综合评价
小毕超是CSDN平台上一位具有较高技术水平和影响力的博主。他在人工智能、微服务架构、大数据处理等多个技术领域都有深入的研究和实践经验。其技术文章质量高、实用性强，为开发者社区提供了宝贵的技术资源。

### 技术价值
1. **技术深度**：在AI大模型应用方面具有较深的技术积累
2. **实践经验**：丰富的项目实践经验，能够提供完整的解决方案
3. **知识分享**：积极分享技术知识，推动技术社区发展
4. **持续创新**：紧跟技术发展趋势，不断探索新技术应用

### 发展前景
基于当前的技术发展趋势和小毕超的技术积累，预计他将在AI大模型应用、智能化系统构建等领域继续发挥重要作用，为技术社区贡献更多有价值的内容。
> TODO: 
> 搜索CSDN小毕超博主的基本信息和简介  -->  completed
> 分析小毕超的技术专长和主要内容领域  -->  completed
> 研究其代表性文章和技术贡献  -->  completed
> 分析其在CSDN平台的影响力和数据表现  -->  completed
> 收集其他平台的相关信息（如果有）  -->  completed
> 整理和撰写详细研究报告  -->  completed

我已经完成了对CSDN小毕超博主的深度研究。通过系统性的信息收集和分析，我为您提供了一份详细的研究报告，涵盖了以下几个方面：

## 研究成果总结：

1. **个人背景**：小毕超是一位具有多项专业认证的技术专家，包括软考高级、TOGAF、PMP等资质，并获得过多项技术竞赛奖项。

2. **技术专长**：主要专注于人工智能与机器学习、微服务架构、大数据处理和云原生技术四大领域。

3. **技术贡献**：在AI大模型应用、模型微调、计算机视觉、微服务架构等方面有深入研究，发表了大量高质量的技术文章。

4. **平台影响力**：在CSDN平台具有博客专家称号，是Java技术领域的优质创作者，内容被多个技术平台转载。

5. **发展趋势**：从传统后端技术逐步发展到AI大模型应用，展现了持续学习和技术创新的能力。

这份研究报告全面展现了小毕超博主在技术社区中的地位、贡献和影响力，为了解这位技术专家提供了详实的参考资料。
```