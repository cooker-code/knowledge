> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: Chatbox支持接入LangGraph智能体？一切都靠Trae Solo!
author: 大模型真好玩
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485307&idx=1&sn=b6fef5472f1f3cffd83d1e04ea26b4e0&chksm=c516fa0b1bd31ae051a1a4ef65409b9dfa28ea51e2a8383da879b4386291e19a61df24f213c6&mpshare=1&scene=24&srcid=1230b48IxC4klIOi01hZ3e7O&sharer_shareinfo=cff457e908ebaf0b760e9249eaf33488&sharer_shareinfo_first=cff457e908ebaf0b760e9249eaf33488#rd
---

# 前言

作为大模型智能体应用开发者，笔者长期使用 LangChain 与 LangGraph 框架开发各类智能体，并通过专栏[LangChain/LangGraph系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4073973355617976325#wechat_redirect)教程与大家分享相关经验。然而，开发完成的智能体往往面临一个实际挑战：直接部署并使用它们并不总是便捷。例如，LangChain官方推荐的 agent-chat-ui 通常需要额外启动服务，过程稍显繁琐。而笔者日常习惯使用 Chatbox 客户端与大模型交互，这让笔者不禁思考：能否通过 FastAPI 将智能体封装为服务，并通过 Chatbox 直接连接，从而实现更轻量、快速的调用？

这个想法虽然直接，代码量也不多。但在如今大模型辅助编程的日常中，动手实现的需求似乎总被“交给 AI 来完成”（归根结底程序员变懒了）。正巧，最近看到字节跳动的 Trae 推出了“Solo 模式”，它被称为“国产 Cursor”，恰好为笔者提供了验证的机会——是否能够借助 Trae Solo，自动、高效地完成这个从想法到可运行服务的过程？

PS:鉴于后台私信越来越多，我建了一些大模型交流群，大家在日常学习生活工作中遇到的大模型知识和问题都可以在群中分享出来大家一起解决！如果大家想交流大模型知识，可以关注我并回复加群。

# 一、Chatbox 与 Trae 简介

## 1.1 Chatbox：跨平台 AI 客户端

Chatbox 是一款功能全面的跨平台 AI 客户端，支持 Windows、macOS等主流操作系统。它采用 OpenAI 兼容的 API 调用方式，能够便捷地连接多种大模型，包括 OpenAI、Claude、DeepSeek 等云端 API，也支持本地vllm、ollama部署的模型。凭借其简洁的界面与稳定的体验，Chatbox 已成为众多开发者和AI爱好者常用的客户端工具之一。

## 1.2 Trae：AI 驱动的编程助手

Trae 是由字节跳动推出的 AI 编程工具，被许多开发者称为“国产 Cursor”。它已经从最初的代码生成助手，演进为一个支持端到端开发的智能平台。其最新推出的 **Solo 模式**，尤其适合快速实现想法：用户只需用自然语言描述需求（例如“开发一个在线背单词应用”），Trae 便能自主完成从需求分析、代码编写、测试到部署的完整流程。这正好为我们“懒于动手”但又想快速验证想法的场景，提供了理想的实验环境。

# 二、Chatbox 接入 LangChain 智能体的基本思路

Trae Solo能力“吹的响”，总不至于这么个小需求它搞不了吧，在测试它的能力前笔者先自我思考一下如何实现这个需求？根据 Chatbox 官方(https://chatboxai.app/zh)介绍，它主要通过发起 **OpenAI API 格式** 的请求来与各类后端服务通信，无论是 vLLM、Ollama 等本地部署的模型，还是远程托管的服务。这为笔者提供了一个清晰的对接标准。

因此，将 LangChain/LangGraph 构建的智能体接入 Chatbox 的核心思路便非常明确：

1. **构建适配层：使用 FastAPI 编写一个标准化接口，接收来自 Chatbox 的 OpenAI 格式请求。**
2. **格式转换与调用：将该请求解析并转化为 LangChain 智能体能处理的对话消息格式，然后调用智能体执行。**
3. **结果返回：将智能体返回的结果，重新封装成符合 OpenAI 响应格式的数据，通过 API 返回给 Chatbox。**

这样一来，对 Chatbox 而言，智能体服务就像一个兼容的“模型后端”，实现了无缝接入。

# 三、使用Trae Solo 完成 天气助手智能体的接入

**1. 回顾基础智能体代码：首先回顾一下之前用 LangChain 1.0 的**`create_react_agent` 编写的天气助手智能体。这个智能体使用了心知天气的免费 API，不熟悉这个服务的读者可以参考笔者的文章[《从0到1开发DeepSeek天气助手智能体——你以为大模型只会聊天？Function Calling让它"上天入地"》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247483986&idx=1&sn=70de5c6bdb8bb6a2ed24dab92963b024&scene=21#wechat_redirect)进行注册。关于 LangChain 1.0 智能体开发的基础知识，可参阅[《LangChain1.0速通指南（二）——LangChain1.0 create\_agent api 基础知识》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485146&idx=1&sn=1397839d622039d1665868f97a329f43&scene=21#wechat_redirect)

```
from langchain.chat_models import init_chat_modelfrom langchain.tools import toolfrom langchain.agents import create_agentfrom langgraph.config import get_stream_writer
@tool(args_schema=WeatherQuery)def get_weather(loc):    """        查询即时天气函数        :param loc: 必要参数，字符串类型，用于表示查询天气的具体城市名称，\        :return：心知天气 API查询即时天气的结果，具体URL请求地址为："https://api.seniverse.com/v3/weather/now.json"        返回结果对象类型为解析之后的JSON格式对象，并用字符串形式进行表示，其中包含了全部重要的天气信息    """    url = "https://api.seniverse.com/v3/weather/now.json"    params = {        "key": "你注册的心知天气api key",        "location": loc,        "language": "zh-Hans",        "unit": "c",    }    response = requests.get(url, params=params)    temperature = response.json()    return temperature['results'][0]['now']
SYSTEM_PROMPT = "你是一名可爱的天气预报播报员，可以模仿林志玲的语气播报天气，具备调用get_weather天气函数获取指定地点天气的能力"
model = init_chat_model(    model="deepseek-chat",    base_url="https://api.deepseek.com",    api_key="你注册的deepseek api key")
agent = create_agent(    model=model,    tools=[get_weather],    system_prompt=SYSTEM_PROMPT)
```

**2. 下载并安装 Trae：前往 Trae 官网 下载安装包。目前 Trae 国内版已全面上线 SOLO 模式，并且可以免费使用。Windows 平台的安装非常简单，只需一路点击"下一步"即可完成。安装完成后打开 Trae，你会看到类似 VS Code 的界面风格。默认进入的是****Build 模式**，这类似于笔者在文章[《零门槛！手把手教你用VS Code + DeepSeek 免费玩转AI编程！（5分钟编写部署个人网站）》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247483779&idx=1&sn=798d9079947b59c17e965c3a71803607&scene=21#wechat_redirect)中使用的 VS Code Cline 插件，可以通过与 AI 助手对话辅助编写代码。

**3. 切换到 SOLO 模式：** 今天笔者要体验的是更强大的 **SOLO 模式**。点击左上角的 IDE 标签即可切换到该模式，它能实现更高程度的开发自动化。SOLO 模式类似于笔者在[《Gemini3.0深度解析，它在重新定义智能](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485241&idx=1&sn=70aaf6c182354e6f801f1249c7333a92&scene=21#wechat_redirect)[》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485241&idx=1&sn=70aaf6c182354e6f801f1249c7333a92&scene=21#wechat_redirect)一文中提到的 Vibe 编程概念：用户只需用自然语言描述需求（例如"开发一个在线背单词应用"），AI 就能自主完成从需求分析、编码、测试到部署的全流程

**4. 用自然语言描述需求：在 SOLO Coder 对话框中输入以下提示词**

```
你是一名python编程专家，熟练掌握langchain1.0开发智能体, fastapi编写接口的能力，可以满足用户提出的任何需求。现在项目中的langchain_weather.py文件中包含一个天气助手的智能体，我希望使用fastapi编写openai 风格的接口，注意需要使用流式接口。使该智能体可以被chatbox等客户端以api格式接入，请实现我的需求并本地启动接口，请完成我的需求
```

**5. 观察 Trae Solo 的自动化工作流程：** 按下回车键后，Trae Solo 开始自动执行任务：

* **智能分析**

  首先分析 `langchain_weather.py` 源文件，并识别出了代码中的一个 Bug——未定义 `WeatherQuery` 类型。
* **代码生成**

  新建 `main.py` 文件，编写符合 OpenAI 风格的 API 接口。
* **自我修复**

  进行测试时发现导入问题，自动修复相关 Bug。
* **环境管理**

  启动应用时使用 uv 创建虚拟环境，确保环境稳定性。
* **端口处理**

  检测到端口占用时，自动进行循环优化处理。
* **全面测试**

  通过编写 `test-api.py`、使用 curl 命令等多种方式测试服务稳定性。

**6. 生成的 FastAPI 代码分析：查看 Trae Solo 生成的**`main.py` 代码，其结构清晰、可维护性强

```
from fastapi import FastAPI, HTTPExceptionfrom fastapi.middleware.cors import CORSMiddlewarefrom fastapi.responses import StreamingResponsefrom pydantic import BaseModel, Fieldfrom typing import List, Optional, Union, Generatorimport uvicornimport timeimport jsonfrom langchain_weather import agent
app = FastAPI(    title="Weather Assistant API",    description="OpenAI风格的天气助手API，支持chatbox等客户端接入",    version="1.0.0")
# 添加CORS中间件，允许跨域请求app.add_middleware(    CORSMiddleware,    allow_origins=["*"],  # 允许所有来源，生产环境应限制具体域名    allow_credentials=True,    allow_methods=["*"],    allow_headers=["*"],)
# 定义OpenAI风格的请求模型class Message(BaseModel):    role: str    content: str
class ChatCompletionRequest(BaseModel):    model: str = "deepseek-chat"    messages: List[Message]    temperature: Optional[float] = 0.7    max_tokens: Optional[int] = None    stream: Optional[bool] = False
# 定义OpenAI风格的响应模型class Choice(BaseModel):    index: int    message: Message    finish_reason: str
class ChatCompletionResponse(BaseModel):    id: str    object: str = "chat.completion"    created: int    model: str    choices: List[Choice]    usage: Optional[dict] = None
# 定义流式响应的Chunk模型class ChunkChoice(BaseModel):    index: int    delta: dict    finish_reason: Optional[str] = None
class ChatCompletionChunk(BaseModel):    id: str    object: str = "chat.completion.chunk"    created: int    model: str    choices: List[ChunkChoice]
@app.post("/chat/completions")async def chat_completions(request: ChatCompletionRequest):    """处理聊天请求，调用天气助手智能体"""    try:        # 提取用户最后一条消息        user_message = request.messages[-1].content
        # 生成响应ID        response_id = f"chatcmpl-{hash(str(request))}"        created_time = int(time.time())
        if request.stream:            # 流式响应处理            def generate_stream() -> Generator[str, None, None]:                # 调用智能体处理请求                result = agent.invoke({"messages": [{"role": "user", "content": user_message}]})
                # 获取智能体返回的消息内容                assistant_content = ""                if isinstance(result, dict) and "messages" in result:                    # 如果返回的是字典格式                    last_message = result["messages"][-1]                    if isinstance(last_message, dict):                        assistant_content = last_message.get("content", "")                    else:                        # 如果是Message对象                        assistant_content = last_message.content                else:                    # 直接处理Message对象                    assistant_content = result.content
                # 模拟流式输出，逐字发送                chunk_id = 0                for char in assistant_content:                    chunk = ChatCompletionChunk(                        id=response_id,                        created=created_time,                        model=request.model,                        choices=[                            ChunkChoice(                                index=0,                                delta={"content": char},                                finish_reason=None                            )                        ]                    )                    yield f"data: {json.dumps(chunk.dict(), ensure_ascii=False)}\n\n"                    time.sleep(0.05)  # 控制流式输出速度
                # 发送结束标志                end_chunk = ChatCompletionChunk(                    id=response_id,                    created=created_time,                    model=request.model,                    choices=[                        ChunkChoice(                            index=0,                            delta={},                            finish_reason="stop"                        )                    ]                )                yield f"data: {json.dumps(end_chunk.dict(), ensure_ascii=False)}\n\n"                yield "data: [DONE]\n\n"
            return StreamingResponse(generate_stream(), media_type="text/event-stream")        else:            # 非流式响应处理            # 调用智能体处理请求            result = agent.invoke({"messages": [{"role": "user", "content": user_message}]})
            # 获取智能体返回的消息内容            assistant_content = ""            if isinstance(result, dict) and "messages" in result:                # 如果返回的是字典格式                last_message = result["messages"][-1]                if isinstance(last_message, dict):                    assistant_content = last_message.get("content", "")                else:                    # 如果是Message对象                    assistant_content = last_message.content            else:                # 直接处理Message对象                assistant_content = result.content
            # 构建响应            response = ChatCompletionResponse(                id=response_id,                created=created_time,                model=request.model,                choices=[                    Choice(                        index=0,                        message=Message(                            role="assistant",                            content=assistant_content                        ),                        finish_reason="stop"                    )                ]            )
            return response    except Exception as e:        raise HTTPException(status_code=500, detail=str(e))
if __name__ == "__main__":    uvicorn.run(app, host="0.0.0.0", port=8001)
```

**7. 配置 Chatbox 连接：** 接下来，笔者将这个服务接入 Chatbox。下载并安装 Chatbox 后，按以下步骤配置：

* 点击"设置" → "模型"
* 添加新的模型提供方
* 配置参数：API 主机：`http://localhost:8001`，API 路径：`/chat/completions`

**8. 测试效果：**

输入北京的天气：

`输入上海的天气：`

可以看到，智能体成功返回了天气信息，并以林志玲的甜美语气进行播报。整个过程从智能体代码到完整的 API 服务，几乎都由 Trae Solo 自动完成！

# 四、展望

通过这个简单的智能体接入实验，我们足以感受到 Trae SOLO 模式的强大潜力。它不仅能自动生成代码，更能完成从分析、测试到服务部署的全流程，真正实现了“描述即开发”的愿景。

也许有人会说，这只是一个小型演示项目，Trae SOLO 能否应对更复杂的生产场景仍有待验证。但我相信，凭借字节跳动深厚的技术积累和众多优秀工程师的努力，这款工具必将持续进化，越来越好。

更重要的是，这次尝试为我们打开了一扇新的窗口：如果 Trae 能够理解并编写智能体接入代码，那么它是否可以直接开发基于 LangChain 的智能体？或许，我们只需要用自然语言描述需求，就能通过 Trae 自动生成智能体，再借助本文的接入方案快速集成到 Chatbox 等客户端中。

想象一下：一个人、一个想法、一句描述，就能快速构建起可用的智能体服务——这正是 Vibe Coding 的魅力所在。它不只是一种编程方式，更是一种让想象力迅速照进现实的开发范式。未来，或许每个人都能轻松打造属于自己的“智能体帝国”。

# 五、总结

本期笔者借助 Trae Solo，快速实现了将 LangChain 智能体接入 Chatbox 客户端的完整流程。这不仅为智能体提供了一个轻量、便捷的调用入口，也让笔者亲身感受到了“自然语言驱动开发”的高效与直观。通过本次实践，笔者见证了 Trae Solo 在代码生成、接口适配、测试验证乃至环境部署上的自动化能力。如果你也有类似的需求——无论是简单的工具对接，还是更复杂的应用开发——都不妨尝试用 Trae Solo 来描述你的想法，让它帮你快速搭建原型、验证逻辑。

笔者打算继续利用 Trae Solo为专栏《[深入浅出LangChain&LangGraph AI Agent 智能体开发》](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4073973355617976325#wechat_redirect)中的多模态 RAG 系统系列文章，开发一个美观且实用的前端界面。欢迎大家和我一起体验更多“描述即实现”的创作乐趣！

对于有经验喜欢写代码的开发者可以阅读笔者的LangChain/LangGraph系列教程专栏，目前已经更完29节。该专栏融合了笔者在实战中积累的深度经验，系统讲解如何基于LangChain与LangGraph框架高效开发智能体，助你快速构建专业级应用。大家可关注笔者微信公众号: **大模型真好玩**, 每期分享涉及的代码均可在公众号私信: **LangChain智能体开发**获得。