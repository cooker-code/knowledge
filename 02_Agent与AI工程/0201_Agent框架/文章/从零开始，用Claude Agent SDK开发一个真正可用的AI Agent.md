---
title: 从零开始，用Claude Agent SDK开发一个真正可用的AI Agent
author: 1Mi智能
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk5MDA0Nzg2OA==&mid=2247483874&idx=1&sn=28740db987f45774553abfbb2f03c732&chksm=c49150a90a76b9d75ff98c854d3fded41dfe856c533c4c65aed8e8db42660e6b5006d7b68174&mpshare=1&scene=24&srcid=1101Pd2CgpXymP1BCBlssmPY&sharer_shareinfo=cca80ceca8478524ab0acb64a7a0ba21&sharer_shareinfo_first=cca80ceca8478524ab0acb64a7a0ba21#rd
---

最近在研究Claude Agent SDK的时候，发现了一个普遍存在的问题：几乎所有的教程和文档都在教你怎么调用API，却没有人告诉你怎么做出一个真正能用的Agent。

打开官方文档，你看到的是这样的代码：

```
async for message in query("Hello, how are you?"):    print(message)
```

这段代码确实能运行，也确实能得到Claude的回复。但问题是，这和直接调用ChatGPT的API有什么区别？你想要的是一个能够理解你业务逻辑、能调用内部系统、能持续对话的智能助手，而不是一个简单的问答接口。

这篇文章会从实际需求出发，一步步告诉你如何开发一个真正可用的、具有专属能力的AI Agent。

## 理解AI Agent的本质

在动手写代码之前，我们需要先理解一个问题：什么样的AI Agent才算是"好用"的？

想象一下，你的公司需要一个代码审查助手。这个助手不仅要能看懂代码，还要了解公司的编码规范、安全策略、性能要求。它需要能够读取你的代码仓库，运行静态分析工具，甚至可能需要查询历史的Bug记录来判断某段代码是否存在风险。

这样一个Agent需要三个核心要素：

第一个是系统提示词。这不是简单的"你是一个代码审查助手"，而是需要详细定义它的专业能力、工作流程、判断标准、交互方式。一个好的系统提示词能让Agent具备专家级的判断力和执行力。

第二个是工具。通用的AI聊天工具只能回答你的问题，但一个专业的Agent需要能够主动获取信息、执行操作。它要能读取文件、运行命令、调用API、查询数据库。这些能力是通过工具来赋予的。

第三个是流程控制。复杂任务往往需要分解成多个步骤，有些步骤可以并行，有些必须串行。Agent需要能够规划任务、管理上下文、协调多个子任务，甚至在必要时调用其他专门的子Agent。

Claude Agent SDK在第三个问题上做得很好，它内置了强大的任务规划和执行能力。但前两个问题，系统提示词和工具，必须由你来解决。因为只有你最了解自己的业务场景和需求。

## 从单一任务开始

让我们从最简单的场景开始：开发一个只执行单一任务的Agent。

假设你需要一个Agent来分析Python代码文件，找出其中可能存在的性能问题。使用Claude Agent SDK，最直接的写法是：

```
import asynciofrom claude_agent_sdk import query, ClaudeAgentOptions  
async def analyze_code():    options = ClaudeAgentOptions(        system_prompt="你是Python性能分析专家，擅长发现代码中的性能瓶颈",        allowed_tools=["Read", "Grep"]    )  
    async for message in query(        prompt="分析./src目录下的所有Python文件，找出性能问题",        options=options    ):        print(message)  
asyncio.run(analyze_code())
```

这段代码能工作，但它是一次性的。任务执行完就结束了，没有后续交互的可能。如果分析结果中有你不理解的地方，你需要重新运行程序，修改prompt，再次执行。这种方式在实际工作中很不方便。

## 引入多轮对话

实际场景中，我们需要的是一个可以持续对话的Agent。你问它"分析代码性能"，它给出结果；你接着问"第三个问题怎么解决"，它能记住上下文，给出针对性的建议；你再问"帮我修改这个文件"，它能理解你说的是哪个文件。

要实现这种效果，需要使用`ClaudeSDKClient`而不是简单的`query`函数：

```
import asynciofrom claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, AssistantMessage, TextBlock  
async def main():    options = ClaudeAgentOptions(        system_prompt="你是Python性能分析专家",        allowed_tools=["Read", "Write", "Grep", "Glob"]    )  
    async with ClaudeSDKClient(options=options) as client:        # 第一轮对话        await client.query("分析./src目录的Python文件性能")        async for message in client.receive_response():            if isinstance(message, AssistantMessage):                for block in message.content:                    if isinstance(block, TextBlock):                        print(block.text)  
        # 第二轮对话 - Claude会记住第一轮的内容        await client.query("第一个性能问题应该怎么优化？")        async for message in client.receive_response():            if isinstance(message, AssistantMessage):                for block in message.content:                    if isinstance(block, TextBlock):                        print(block.text)  
asyncio.run(main())
```

注意这里的关键差异：使用`async with ClaudeSDKClient`创建一个持续的会话，在这个会话中可以多次调用`client.query()`，每次调用都能获取之前对话的上下文。

但这样的代码仍然是预设好对话流程的，不够灵活。我们需要让用户能够自由地输入问题。

## 构建交互式Agent

一个真正可用的Agent应该像这样工作：程序启动后，用户可以随时输入问题，Agent实时响应，对话可以无限持续，直到用户主动退出。

这需要一个交互循环：

```
import asynciofrom claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, AssistantMessage, TextBlock  
async def interactive_agent():    options = ClaudeAgentOptions(        system_prompt="你是Python开发专家，能够帮助用户分析代码、修复Bug、优化性能。",        allowed_tools=["Read", "Write", "Edit", "Bash", "Grep", "Glob"],        permission_mode="acceptEdits"    )  
    async with ClaudeSDKClient(options=options) as client:        print("Agent已启动，输入exit退出")  
        while True:            user_input = input("\n你: ").strip()  
            if user_input.lower() == "exit":                break  
            if not user_input:                continue  
            await client.query(user_input)  
            print("Claude: ", end="", flush=True)            async for message in client.receive_response():                if isinstance(message, AssistantMessage):                    for block in message.content:                        if isinstance(block, TextBlock):                            print(block.text, end="", flush=True)            print()  
asyncio.run(interactive_agent())
```

现在这个Agent可以真正与用户对话了。用户可以问任何问题，Agent会基于完整的上下文历史来回答。这才是一个可用的AI Agent的基础形态。

## 封装成可复用的框架

在实际项目中，你可能需要开发多个不同的Agent：代码审查Agent、数据分析Agent、文档生成Agent等等。每次都写这样的交互循环显然不合适，我们需要把通用的部分封装起来。

```
class InteractiveClaudeAgent:    def __init__(self, system_prompt, allowed_tools, permission_mode="acceptEdits", working_directory=None):        self.options = ClaudeAgentOptions(            system_prompt=system_prompt,            allowed_tools=allowed_tools,            permission_mode=permission_mode,            cwd=working_directory,            max_turns=50,            include_partial_messages=True        )        self.client = None        self.conversation_history = []  
    async def start(self):        self.client = ClaudeSDKClient(options=self.options)        await self.client.__aenter__()        print("Agent已启动，输入exit退出，interrupt中断当前任务")  
    async def stop(self):        if self.client:            await self.client.__aexit__(None, None, None)  
    async def send_message(self, user_input):        self.conversation_history.append({"role": "user", "content": user_input})  
        await self.client.query(user_input)  
        assistant_response = []        print("Claude: ", end="", flush=True)  
        async for message in self.client.receive_response():            if isinstance(message, AssistantMessage):                for block in message.content:                    if isinstance(block, TextBlock):                        print(block.text, end="", flush=True)                        assistant_response.append(block.text)  
        print()  
        if assistant_response:            self.conversation_history.append({                "role": "assistant",                "content": "".join(assistant_response)            })  
    async def run_interactive(self):        await self.start()  
        try:            while True:                user_input = input("\n你: ").strip()  
                if not user_input:                    continue  
                if user_input.lower() == "exit":                    break  
                if user_input.lower() == "interrupt":                    await self.client.interrupt()                    print("任务已中断")                    continue  
                await self.send_message(user_input)  
        finally:            await self.stop()
```

有了这个基础框架，创建新的Agent就变得非常简单：

```
async def create_code_review_agent():    agent = InteractiveClaudeAgent(        system_prompt="你是代码审查专家，关注代码质量、安全性和性能",        allowed_tools=["Read", "Grep", "Glob"]    )    await agent.run_interactive()  
asyncio.run(create_code_review_agent())
```

## 设计有效的系统提示词

框架有了，接下来的关键就是如何设计系统提示词。一个好的系统提示词能让Agent的表现有质的提升。

很多人写系统提示词只写一句话，比如"你是Python专家"。这样的提示词太过宽泛，Agent不知道应该怎么做。

一个有效的系统提示词应该包含这几个部分：

首先是角色定义，明确告诉Agent它是谁、负责什么。其次是能力描述，详细列出它擅长的领域和具备的技能。然后是工作流程，告诉它遇到问题应该按什么步骤处理。最后是行为准则，定义它应该做什么、不应该做什么。

以代码审查Agent为例，一个完整的系统提示词应该是这样的：

```
system_prompt = """你是一位资深的代码审查专家，负责审查Python项目的代码质量。  
你的核心能力包括：识别代码中的逻辑错误和潜在Bug发现安全漏洞，特别是SQL注入、XSS等常见问题评估代码性能，发现不必要的循环、重复计算等性能问题检查是否符合Python最佳实践和PEP 8规范判断代码可读性和可维护性  
你的工作流程是：第一步，使用Glob工具找到需要审查的Python文件第二步，使用Read工具逐个阅读文件内容第三步，分析代码，按严重程度分类：严重问题、重要问题、改进建议第四步，对每个问题给出具体的代码位置、问题说明、修复建议第五步，给出审查总结和整体评分  
你必须遵守的原则：只进行只读操作，不修改任何文件发现严重安全问题时必须明确标注给出的建议要具体可执行，不要泛泛而谈保持客观公正，不因代码风格偏好影响判断  
现在开始你的工作，等待用户的审查请求。"""
```

这样的系统提示词给了Agent明确的指引。它知道自己能做什么、应该怎么做、什么情况下该怎么处理。

再举一个数据分析Agent的例子：

```
system_prompt = """你是数据分析专家，擅长使用Python进行数据处理和分析。  
你掌握的技能：使用pandas进行数据清洗、转换、聚合使用numpy进行数值计算使用matplotlib和seaborn创建可视化图表进行统计分析和假设检验构建简单的机器学习模型  
你的分析流程：首先了解数据源和分析目标，必要时询问用户使用pandas读取数据，查看基本信息（形状、列名、数据类型）进行数据质量检查，识别缺失值、异常值、重复数据根据分析目标进行数据清洗和特征工程执行统计分析或建模创建可视化图表展示发现生成分析报告，包含关键发现和建议  
工作原则：在进行数据操作前先备份原始数据遇到大数据集时要注意内存使用可视化图表要清晰易懂，包含标题和标签结论要基于数据，不做过度推断如果数据质量很差，要先告知用户风险  
准备好帮助用户进行数据分析。"""
```

注意这些系统提示词都很具体，没有模糊的表述。Agent读到这样的提示词后，会清楚地知道自己的定位和工作方式。

## 赋予Agent专属工具

系统提示词定义了Agent的"大脑"，工具则是Agent的"手脚"。Claude Agent SDK内置了一些基础工具，比如Read读取文件、Write写入文件、Bash执行命令、Grep搜索内容等。但实际业务中，你往往需要给Agent提供专属的工具。

假设你的公司有一套工单系统，你希望Agent能够查询和创建工单。这就需要自定义工具：

```
from claude_agent_sdk import tool, create_sdk_mcp_server  
@tool("query_tickets", "查询工单系统", {    "status": "string",    "assignee": "string"})async def query_tickets(args):    # 调用你的工单系统API    status = args.get("status", "open")    assignee = args.get("assignee", "")  
    # 这里是你的实际API调用逻辑    tickets = fetch_tickets_from_api(status, assignee)  
    result = f"找到{len(tickets)}个工单：\n"    for ticket in tickets:        result += f"- {ticket['id']}: {ticket['title']} ({ticket['status']})\n"  
    return {        "content": [{            "type": "text",            "text": result        }]    }  
@tool("create_ticket", "创建工单", {    "title": "string",    "description": "string",    "priority": "string"})async def create_ticket(args):    title = args["title"]    description = args["description"]    priority = args.get("priority", "medium")  
    # 调用API创建工单    ticket_id = create_ticket_in_api(title, description, priority)  
    return {        "content": [{            "type": "text",            "text": f"工单创建成功，ID: {ticket_id}"        }]    }  
# 创建MCP服务器ticket_server = create_sdk_mcp_server(    name="ticket_system",    tools=[query_tickets, create_ticket])  
# 在Agent中使用options = ClaudeAgentOptions(    system_prompt="你是IT支持助手，能够帮助用户管理工单",    mcp_servers={"tickets": ticket_server},    allowed_tools=["mcp__tickets__query_tickets", "mcp__tickets__create_ticket"])
```

这样，Agent就能够根据用户的请求自动查询工单或创建新工单了。用户只需要说"查看我的待处理工单"或"创建一个关于服务器故障的高优先级工单"，Agent会自动调用相应的工具。

再举一个例子，如果你需要Agent能够访问公司内部的知识库：

```
@tool("search_knowledge", "搜索内部知识库", {    "query": "string",    "category": "string"})async def search_knowledge(args):    query = args["query"]    category = args.get("category", "all")  
    # 调用知识库搜索API    results = search_kb_api(query, category)  
    if not results:        return {            "content": [{                "type": "text",                "text": f"没有找到关于'{query}'的知识库文档"            }]        }  
    response = f"找到{len(results)}篇相关文档：\n\n"    for doc in results:        response += f"标题：{doc['title']}\n"        response += f"摘要：{doc['summary']}\n"        response += f"链接：{doc['url']}\n\n"  
    return {        "content": [{            "type": "text",            "text": response        }]    }
```

工具的设计要注意几个原则：工具的功能要单一明确，不要让一个工具做太多事情；参数定义要清晰，让Agent知道需要传什么参数；返回结果要结构化，方便Agent理解和使用；错误处理要完善，遇到问题要返回明确的错误信息。

## 实现权限控制

给Agent提供工具的同时，也要考虑安全性。不是所有操作都应该被允许，特别是涉及到写入、删除、执行命令这类敏感操作。

Claude Agent SDK提供了灵活的权限控制机制。最简单的方式是通过`permission_mode`参数：

```
options = ClaudeAgentOptions(    permission_mode="default"  # 每次操作都需要用户确认)
```

但这样可能会很繁琐。更好的方式是实现自定义的权限处理器：

```
async def permission_handler(tool_name, input_data, context):    # 阻止写入系统目录    if tool_name == "Write":        file_path = input_data.get("file_path", "")        if any(forbidden in file_path for forbidden in ["/etc/", "/sys/", "/usr/"]):            return {                "behavior": "deny",                "message": f"安全策略禁止写入系统目录：{file_path}"            }  
    # 危险命令需要确认    if tool_name == "Bash":        command = input_data.get("command", "")        dangerous = ["rm -rf", "format", "dd if=", "mkfs"]        if any(keyword in command for keyword in dangerous):            print(f"警告：检测到危险命令：{command}")            confirm = input("是否允许执行？(yes/no): ")            if confirm.lower() != "yes":                return {                    "behavior": "deny",                    "message": "用户拒绝执行危险命令"                }  
    # 默认允许    return {"behavior": "allow"}  
options = ClaudeAgentOptions(    can_use_tool=permission_handler)
```

这样的权限控制既保证了安全性，又不会过度干扰正常操作。

## 处理复杂场景

在实际应用中，你可能会遇到更复杂的需求。比如需要多个专门的子Agent协同工作，或者需要根据任务类型动态切换Agent的行为模式。

对于协同工作的场景，可以创建多个专门的Agent，每个负责一个领域：

```
class CodeReviewAgent(InteractiveClaudeAgent):    def __init__(self):        super().__init__(            system_prompt="代码审查专家的详细提示词...",            allowed_tools=["Read", "Grep", "Glob"],            permission_mode="default"        )  
class CodeFixAgent(InteractiveClaudeAgent):    def __init__(self):        super().__init__(            system_prompt="代码修复专家的详细提示词...",            allowed_tools=["Read", "Write", "Edit"],            permission_mode="default"        )  
class CodeTestAgent(InteractiveClaudeAgent):    def __init__(self):        super().__init__(            system_prompt="测试编写专家的详细提示词...",            allowed_tools=["Read", "Write", "Bash"],            permission_mode="acceptEdits"        )
```

然后在主流程中协调这些Agent：

```
async def code_quality_workflow(file_path):    # 第一步：代码审查    review_agent = CodeReviewAgent()    await review_agent.start()    await review_agent.send_message(f"审查文件：{file_path}")    await review_agent.stop()  
    # 第二步：修复问题    fix_agent = CodeFixAgent()    await fix_agent.start()    await fix_agent.send_message(f"根据审查结果修复{file_path}中的问题")    await fix_agent.stop()  
    # 第三步：生成测试    test_agent = CodeTestAgent()    await test_agent.start()    await test_agent.send_message(f"为{file_path}生成单元测试")    await test_agent.stop()
```

这种方式让每个Agent都专注于自己擅长的领域，整体流程更加清晰可控。

## 总结经验

开发一个真正可用的AI Agent，关键不在于掌握多少SDK的API，而在于理解你的业务需求，设计合理的系统提示词，提供必要的工具，控制好权限和流程。

从简单的单次调用开始，逐步引入多轮对话能力，封装成可复用的框架，根据具体场景定制系统提示词和工具，最终你会得到一个真正能够解决实际问题的AI Agent。

Claude Agent SDK已经为你解决了任务规划、上下文管理、工具调用等底层问题，你只需要专注于业务逻辑的实现。这正是这个SDK最大的价值所在。

代码不复杂，难的是想清楚你的Agent应该做什么、怎么做。把这些想清楚了，剩下的就是水到渠成的事情。