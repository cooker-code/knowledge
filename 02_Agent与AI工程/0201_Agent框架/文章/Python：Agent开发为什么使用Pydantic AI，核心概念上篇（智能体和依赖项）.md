---
title: Python：Agent开发为什么使用Pydantic AI，核心概念上篇（智能体和依赖项）
author: 猫咪不吃愚
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNDY1NzE0NQ==&mid=2247486948&idx=1&sn=5230ddc3b34e6da96a28fe439d4bfea5&chksm=c3fadca25404f7468f5fdd6b2676d4a16f7db7c5506ec6d3a3eacfa05e1443ae2cf01516a1d0&mpshare=1&scene=24&srcid=1105epLAoBGI5XE7NAxX7FdT&sharer_shareinfo=2bacb60ff5ff8c17946f206ed96ff3a7&sharer_shareinfo_first=2bacb60ff5ff8c17946f206ed96ff3a7#rd
---

> 字数 2631，阅读大约需 14 分钟

# Python：Agent开发为什么使用Pydantic AI，核心概念上篇（智能体和依赖项）

## 什么是pydantic ai

超级好用的AI Agent开发框架：`Pydantic AI`  

Pydantic AI 是由 Pydantic 团队开发的 Python Agent 框架,旨在帮助开发者快速、自信且轻松地构建生产级的生成式 AI 应用程序和工作流 PydanticAI（`GenAI智能体框架，Pydantic风格`）。这个框架的设计初衷是将 FastAPI 的开发体验带到生成式 AI 应用和 Agent 开发中。Pydantic AI 的设计理念是将生产级工程最佳实践带入 AI Agent 开发领域,让**构建可靠的 AI 应用变得更加简单和安全**。  
Pydantic AI 框架的文档地址[1]

## 为什么使用pydantic ai

1. 1. 业界验证的标准库：**由 Pydantic 团队原厂打造**，其验证引擎已是 OpenAI、Google、Anthropic、LangChain 等众多顶级 AI 框架和 SDK 的基石。Pydantic AI：下一代类型安全的AI Agent框架 - 知乎[2]
2. 2. 无所不包，模型无关：**几乎支持所有主流模型和云平台**（OpenAI, Anthropic, Gemini, Azure, Bedrock 等近20家），并支持轻松自定义扩展，彻底摆脱供应商锁定。
3. 3. **开箱即用的可观测性**：深度集成 Pydantic Logfire，提供实时调试、性能监控、链路追踪和成本跟踪。也兼容任何支持 OpenTelemetry 的现有平台。
4. 4. **“编译即正确”的类型安全：提供无与伦比的类型提示支持**，将大量错误从“运行时”提前到“编码时”，让开发像 Rust 语言一样可靠，极大提升 IDE 和 AI 编程助手的效率。
5. 5. 强大的系统评估能力：内置强大评估功能，可系统化测试和长期监控构建的智能体系统的性能与准确性。
6. 6. **先进的智能体生态互联：原生集成 MCP、A2A 等协议**，让智能体能安全使用外部工具、与其他智能体协作，并构建流式交互应用。
7. 7. **关键操作，人为介入**：可灵活配置，让高风险的工具调用在获得人工批准后才执行，确保安全与可控。
8. 8. 生产级可靠性：持久执行：**智能体具备“韧性”，能在 API 故障、程序重启后继续工作**，可靠处理长时、异步及人机协同工作流。
9. 9. **实时流式结构化输出**：无需等待生成完毕，即可实时获取并**验证结构化的数据流**，响应更迅速。

## 极简示例代码

```
from pydantic_ai import Agent  
  
agent = Agent(    
    'anthropic:claude-sonnet-4-0',  
    instructions='Be concise, reply with one sentence.',    
)  
  
result = agent.run_sync('Where does "hello world" come from?')    
print(result.output)  
"""  
The first known use of "hello, world" was in a 1974 textbook about the C programming language.  
"""
```

## 安装和配置

要求：Python>=3.10

```
uv add pydantic-ai
```

Installation - Pydantic AI[3]

## 核心概念（agent 和 deps）

### agent

智能体是Pydantic AI与大型语言模型交互的主要接口，单个智能体将控制整个应用程序或组件，但多个智能体也可以通过交互来实现更复杂的工作流。

1. 1. 指令
2. 2. 工具
3. 3. 结构化输出
4. 4. 类型约束
5. 5. 大语言模型
6. 6. 模型设置

#### 运行智能体

**智能体实例化一次，作为全局模块变量，在程序里重复使用**，`关注流式输出、最终输出、智能体图谱`

```
from pydantic_ai import Agent, RunContext  
  
roulette_agent = Agent(    
    'openai:gpt-5',  
    deps_type=int,  
    output_type=bool,  
    system_prompt=(  
        'Use the `roulette_wheel` function to see if the '  
        'customer has won based on the number they provide.'  
    ),  
)  
  
  
@roulette_agent.tool  
async def roulette_wheel(ctx: RunContext[int], square: int) -> str:    
    """check if the square is a winner"""  
    return 'winner' if square == ctx.deps else 'loser'  
  
  
# Run the agent  
success_number = 18    
result = roulette_agent.run_sync('Put my money on square eighteen', deps=success_number)  
print(result.output)    
#> True  
  
result = roulette_agent.run_sync('I bet five is the winner', deps=success_number)  
print(result.output)  
#> False
```

智能体包含5种运行方式：

1. 1. **agent.run()** — 一个异步函数，返回包含完整响应的RunResult。
2. 2. **agent.run\_sync()** — 一个简单的同步函数，返回包含完整响应的RunResult（在内部，它只是调用loop.run\_until\_complete(self.run())）。
3. 3. **agent.run\_stream()** — 一个异步上下文管理器，它会返回一个StreamedRunResult，该结果包含将文本和结构化输出作为异步可迭代对象进行流式传输的方法。
4. 4. **agent.run\_stream\_events()** — 一个返回异步可迭代的AgentStreamEvents和包含最终运行结果的AgentRunResultEvent的函数。
5. 5. **agent.iter()** — 一个上下文管理器，返回一个AgentRun，即智能体底层Graph节点上的异步可迭代对象。

```
from pydantic_ai import Agent, AgentRunResultEvent, AgentStreamEvent  
  
agent = Agent('openai:gpt-5')  
  
result_sync = agent.run_sync('What is the capital of Italy?')  
print(result_sync.output)  
#> The capital of Italy is Rome.  
  
  
async def main():  
    result = await agent.run('What is the capital of France?')  
    print(result.output)  
    #> The capital of France is Paris.  
  
    async with agent.run_stream('What is the capital of the UK?') as response:  
        async for text in response.stream_text():  
            print(text)  
            #> The capital of  
            #> The capital of the UK is  
            #> The capital of the UK is London.  
  
    events: list[AgentStreamEvent | AgentRunResultEvent] = []  
    async for event in agent.run_stream_events('What is the capital of Mexico?'):  
        events.append(event)  
    print(events)  
    """  
    [  
        PartStartEvent(index=0, part=TextPart(content='The capital of ')),  
        FinalResultEvent(tool_name=None, tool_call_id=None),  
        PartDeltaEvent(index=0, delta=TextPartDelta(content_delta='Mexico is Mexico ')),  
        PartDeltaEvent(index=0, delta=TextPartDelta(content_delta='City.')),  
        PartEndEvent(  
            index=0, part=TextPart(content='The capital of Mexico is Mexico City.')  
        ),  
        AgentRunResultEvent(  
            result=AgentRunResult(output='The capital of Mexico is Mexico City.')  
        ),  
    ]  
    """
```

更多代码示例，需要阅读：Agents - Pydantic AI[4]

#### 智能图谱

在内部，Pydantic AI 中的每个**Agent都使用pydantic-graph来管理其执行流程**。pydantic-graph是一个通用的、以类型为中心的库，用于在 Python 中构建和运行有限状态机。Pydantic AI 利用它来协调智能体运行中模型请求和模型响应的处理。**在开发过程中：完全不需要担心pydantic-graph**；调用agent.run(...)只需从头到尾遍历底层图即可。在特定阶段注入自己的逻辑——Pydantic AI会通过Agent.iter暴露更低层级的迭代过程。

```
graph TD  
    A[UserPromptNode] --> B[ModelRequestNode]  
    B --> C[CallToolsNode]  
    C --> D{需要更多工具调用?}  
    D -->|是| B  
    D -->|否| E[End]
```

确定性执行路径：每个节点的职责清晰，执行顺序可预测  
状态持久化：完整的消息历史和状态追踪  
异常处理：内置重试机制和错误恢复策略

关于图的更多信息，以后根据实践讲解图的使用：PydanticAI图解：AI Agent工作流的革命性变革[5]

#### 额外配置

Pydantic AI 提供了一个UsageLimits结构，帮助限制模型运行时的使用量（令牌、请求和工具调用）。

```
from pydantic_ai import Agent, UsageLimitExceeded, UsageLimits  
  
agent = Agent('anthropic:claude-sonnet-4-5')  
  
result_sync = agent.run_sync(  
    'What is the capital of Italy? Answer with just the city.',  
    usage_limits=UsageLimits(response_tokens_limit=10),  
)  
print(result_sync.output)  
#> Rome  
print(result_sync.usage())  
#> RunUsage(input_tokens=62, output_tokens=1, requests=1)  
  
try:  
    result_sync = agent.run_sync(  
        'What is the capital of Italy? Answer with a paragraph.',  
        usage_limits=UsageLimits(response_tokens_limit=10),  
    )  
except UsageLimitExceeded as e:  
    print(e)  
    #> Exceeded the output_tokens_limit of 10 (output_tokens=32)
```

Pydantic AI 提供了一个settings.ModelSettings结构来帮助微调请求。该结构允许配置影响模型行为的常见参数，例如temperature、max\_tokens、timeout等.可以在在模型级、智能体、运行时级别单独设置并生效。

```
from pydantic_ai import Agent, ModelSettings  
from pydantic_ai.models.openai import OpenAIChatModel  
  
# 1. Model-level defaults  
model = OpenAIChatModel(  
    'gpt-5',  
    settings=ModelSettings(temperature=0.8, max_tokens=500)  # Base defaults  
)  
  
# 2. Agent-level defaults (overrides model defaults by merging)  
agent = Agent(model, model_settings=ModelSettings(temperature=0.5))  
  
# 3. Run-time overrides (highest priority)  
result_sync = agent.run_sync(  
    'What is the capital of Italy?',  
    model_settings=ModelSettings(temperature=0.0)  # Final temperature: 0.0  
)  
print(result_sync.output)  
#> The capital of Italy is Rome.
```

#### 运行和对话

一个智能体的一次运行可能代表一次完整的对话——单次运行中可以交换的消息数量没有限制。不过，一次对话也可能由多次运行组成，尤其是当你需要在不同的交互或API调用之间维持状态时。

```
from pydantic_ai import Agent  
  
agent = Agent('openai:gpt-5')  
  
# First run  
result1 = agent.run_sync('Who was Albert Einstein?')  
print(result1.output)  
#> Albert Einstein was a German-born theoretical physicist.  
  
# Second run, passing previous messages  
result2 = agent.run_sync(  
    'What was his most famous equation?',  
    message_history=result1.new_messages(),    
)  
print(result2.output)  
#> Albert Einstein's most famous equation is (E = mc^2).
```

### Deps: 依赖项

Pydantic AI 采用依赖注入系统，为智能体的系统提示词、工具和输出验证器提供数据和服务。

#### 什么是依赖项

依赖项可以是任何Python类型。在简单情况下，你或许可以将单个对象作为依赖项传递（例如HTTP连接），但当你的依赖项包含多个对象时，数据类通常是一种便捷的容器

```
from dataclasses import dataclass  
  
import httpx  
  
from pydantic_ai import Agent, RunContext  
  
  
@dataclass  
class MyDeps:  
    api_key: str  
    http_client: httpx.AsyncClient  
  
  
agent = Agent(  
    'openai:gpt-5',  
    deps_type=MyDeps,  
)  
  
  
@agent.system_prompt    
async def get_system_prompt(ctx: RunContext[MyDeps]) -> str:    
    response = await ctx.deps.http_client.get(    
        'https://example.com',  
        headers={'Authorization': f'Bearer {ctx.deps.api_key}'},    
    )  
    response.raise_for_status()  
    return f'Prompt: {response.text}'  
  
  
async def main():  
    async with httpx.AsyncClient() as client:  
        deps = MyDeps('foobar', client)  
        result = await agent.run('Tell me a joke.', deps=deps)  
        print(result.output)  
        #> Did you hear about the toothpaste scandal? They called it Colgate.
```

#### 异步和同步依赖

系统提示函数、功能工具和输出验证器均在智能体运行的异步上下文中运行。如果这些函数不是协程（例如async def），它们会在线程池中通过run\_in\_executor被调用，因此，在依赖项执行IO操作时，使用async方法略胜一筹，不过同步依赖项也能正常工作。`智能体始终在异步环境里执行，于依赖是否异步无关`

```
from dataclasses import dataclass  
  
import httpx  
  
from pydantic_ai import Agent, RunContext  
  
  
@dataclass  
class MyDeps:  
    api_key: str  
    http_client: httpx.Client    
  
  
agent = Agent(  
    'openai:gpt-5',  
    deps_type=MyDeps,  
)  
  
  
@agent.system_prompt  
def get_system_prompt(ctx: RunContext[MyDeps]) -> str:    
    response = ctx.deps.http_client.get(  
        'https://example.com', headers={'Authorization': f'Bearer {ctx.deps.api_key}'}  
    )  
    response.raise_for_status()  
    return f'Prompt: {response.text}'  
  
  
async def main():  
    deps = MyDeps('foobar', httpx.Client())  
    result = await agent.run(  
        'Tell me a joke.',  
        deps=deps,  
    )  
    print(result.output)  
    #> Did you hear about the toothpaste scandal? They called it Colgate.
```

#### 用于工具和输出验证器

```
from dataclasses import dataclass  
  
import httpx  
  
from pydantic_ai import Agent, ModelRetry, RunContext  
  
  
@dataclass  
class MyDeps:  
    api_key: str  
    http_client: httpx.AsyncClient  
  
  
agent = Agent(  
    'openai:gpt-5',  
    deps_type=MyDeps,  
)  
  
  
@agent.system_prompt  
async def get_system_prompt(ctx: RunContext[MyDeps]) -> str:  
    response = await ctx.deps.http_client.get('https://example.com')  
    response.raise_for_status()  
    return f'Prompt: {response.text}'  
  
  
@agent.tool    
async def get_joke_material(ctx: RunContext[MyDeps], subject: str) -> str:  
    response = await ctx.deps.http_client.get(  
        'https://example.com#jokes',  
        params={'subject': subject},  
        headers={'Authorization': f'Bearer {ctx.deps.api_key}'},  
    )  
    response.raise_for_status()  
    return response.text  
  
  
@agent.output_validator    
async def validate_output(ctx: RunContext[MyDeps], output: str) -> str:  
    response = await ctx.deps.http_client.post(  
        'https://example.com#validate',  
        headers={'Authorization': f'Bearer {ctx.deps.api_key}'},  
        params={'query': output},  
    )  
    if response.status_code == 400:  
        raise ModelRetry(f'invalid response: {response.text}')  
    response.raise_for_status()  
    return output  
  
  
async def main():  
    async with httpx.AsyncClient() as client:  
        deps = MyDeps('foobar', client)  
        result = await agent.run('Tell me a joke.', deps=deps)  
        print(result.output)  
        #> Did you hear about the toothpaste scandal? They called it Colgate.
```

---

## 彩蛋结尾

嘿，别滑了！手指停一停，听我说句悄悄话👇

🌟 关注我：下次更新，系统会自动弹窗提醒你，就像外卖到了那样准时！再也不怕错过我的脑洞和干货啦~

📌 收藏本文：这篇宝藏文章，现在不码住，以后想找只能捶胸顿足！点个收藏，让它成为你的私人知识库，随时回来挖宝~

❤️ 点赞在看：如果逗笑你了或者对你有用，麻烦高抬贵手点个赞！你的每个赞都是我熬夜写文的“鸡血”，让我更有动力产出更多有趣内容~

#### 引用链接

`[1]` Pydantic AI 框架的文档地址: *https://ai.pydantic.org.cn/*  
`[2]` Pydantic AI：下一代类型安全的AI Agent框架 - 知乎: *https://zhuanlan.zhihu.com/p/1945407545297572981*  
`[3]` Installation - Pydantic AI: *https://ai.pydantic.dev/install/#slim-install*  
`[4]` Agents - Pydantic AI: *https://ai.pydantic.dev/agents/#streaming-events-and-final-output*  
`[5]` PydanticAI图解：AI Agent工作流的革命性变革: *https://www.toolify.ai/zh/ai-news-cn/pydanticai%E5%9B%BE%E8%A7%A3ai-agent%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%9A%84%E9%9D%A9%E5%91%BD%E6%80%A7%E5%8F%98%E9%9D%A9-3405676?\_\_cf\_chl\_tk=h6luKRfJ\_UCUTMEL7OfvwJtdS8X5.maL7Y.WcpVgjxc-1762158161-1.0.1.1-6FFrSkQ.xAnvHDb28I1bLtqhiZFCidLjfjp7gzsiTj0*