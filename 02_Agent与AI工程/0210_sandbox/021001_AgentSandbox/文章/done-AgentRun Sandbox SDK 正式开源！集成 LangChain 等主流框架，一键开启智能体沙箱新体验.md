> 已吸收至：[[02_Agent与AI工程/0210_sandbox/021001_AgentSandbox/021001_核心知识点/AgentSandbox隔离与审计边界|AgentSandbox隔离与审计边界]]、[[02_Agent与AI工程/0210_sandbox/021001_AgentSandbox/021001_核心知识点/Sandbox运行时实现与选型|Sandbox运行时实现与选型]]
---
title: AgentRun Sandbox SDK 正式开源！集成 LangChain 等主流框架，一键开启智能体沙箱新体验
author: 阿里云云原生
date:
url: https://mp.weixin.qq.com/s?__biz=MzUzNzYxNjAzMg==&mid=2247580842&idx=1&sn=7e76a3a3ada98f173700fd56e9e98a43&chksm=fb9b92fca8813799b2f831273f27cc4cb6270d91592a660b12960db0b9a339cefc37bb218fa2&mpshare=1&scene=24&srcid=1230FnidAVJzcZbdCzHMNOln&sharer_shareinfo=266ff5f94cad3ac0de23467627d9de2e&sharer_shareinfo_first=266ff5f94cad3ac0de23467627d9de2e#rd
---

> 让智能体开发更轻盈，让云端运行更安全——AgentRun Sandbox SDK 开源发布，赋能 Agentic AI 快速落地。

***引言：构建面向未来的***

***Agentic AI 基础设施***

*Cloud Native*

在大模型与智能体（Agent）技术迅猛发展的今天，开发者不仅需要强大的模型能力，更亟需一个安全、弹性、易用且可扩展的运行环境来承载复杂的 Agent 逻辑。为此，我们正式推出 AgentRun Sandbox SDK 并全面开源！

[函数计算 AgentRun](https://mp.weixin.qq.com/s?__biz=MzUzNzYxNjAzMg==&mid=2247580646&idx=1&sn=840d2fcc0e99bd012b89efdf86ae8c59&scene=21#wechat_redirect)**[****1]**是以高代码为核心、生态开放、灵活组装的一站式 Agentic AI 基础设施平台，致力于为企业级 Agentic 应用提供从开发、部署到运维的全生命周期支持。平台深度构建于阿里云函数计算（Function Compute, FC）之上，天然继承了 Serverless 架构的三大核心优势：极致弹性、按量付费、零运维负担。

更重要的是，函数计算 AgentRun 通过深度集成 AgentScope、LangChain、Dify、RAGFlow、Mem0 等主流开源生态，打造了一个高性能、高安全、高可观测的智能体运行底座。平台提供五大核心能力：高性能 Sandbox 执行环境、统一模型代理与高可用保障、全链路可观测性、工具与 MCP（Model Context Protocol）统一管理，以及完善的数据安全与隔离治理机制。这一切，只为让你专注于智能体的业务逻辑本身，而无需被底层基础设施的复杂性所困扰。

**多模态沙箱能力，满足多样智能体需求**

函数计算 AgentRun 的核心亮点之一，是其内置的多类型 Sandbox 运行环境，基于阿里云 FC 安全隔离架构构建，确保每一次执行都安全可控：

* **Code Interpreter Sandbox（代码解释器沙箱）****提供隔离的 Python/JavaScript 执行环境，支持文件系统读写、命令执行、数值计算与数据分析。适用于需要动态生成/执行代码的场景，如数据可视化、公式求解、自动化脚本等。**

* **Browser Sandbox（浏览器沙箱）****内置无头浏览器、VNC 可视化客户端及操作录制功能，支持模拟真实用户行为，实现网页抓取、表单填写、信息提取等操作，为智能体赋予“上网”能力。**

* **All-in-One Sandbox（二合一沙箱）****融合代码执行与浏览器能力于一体，一站式支持复杂任务流——例如：先爬取网页数据，再用 Python 分析并生成图表，最后返回结构化结果。真正实现“端到端智能体工作流”。**

为降低接入门槛，函数计算 **AgentRun 特别开源推出****基于 Python 语言的 Sandbox SDK**，开发者仅需几行配置即可将任意智能体接入沙箱服务。无需修改原有框架逻辑，即可享受 Serverless 架构下的安全、弹性与高性能。

***LangChain × Codelnterpreter：***

***为智能体注入“代码大脑”***

*Cloud Native*

LangChain 是当前最流行的 Agent 编排框架之一。现在，通过 AgentRun Sandbox SDK，你可以零改造地为 LangChain Agent 添加安全可靠的代码执行能力。

**本地快速实践（5 分钟上手）**

安装 AgentRun Sandbox SDK 后，在 LangChain 工具中注册 Code Interpreter 工具，即可让 Agent 自主编写并运行 Python 代码。整个过程无需改动原有项目结构，轻松完成本地调试。

#### 1. 安装 Serverless Devs

运行脚手架，您需要使用 Serverless Devs 工具，请参考对应安装教程**[****2]**。

如果您拥有 NodeJS 开发环境，可以使用 `npm i -g @serverless-devs/s` 快速安装 Serverless Devs。您也可以直接下载 Serverless Devs 二进制程序**[****3]**使用 Serverless Devs。

#### 2. 初始化脚手架应用

使用快速创建脚手架创建您的 Agent。

**注意！**您需要确保您的 python 环境在 3.10 以上。

```
# 初始化模板s init agentrun-quick-start-langchain# 按照实际情况进入代码目录cd agentrun-quick-start-langchain/code# 初始化虚拟环境并安装依赖uv venv && uv pip install -r requirements.txt
```

#### 3. 配置认证信息

首次使用前，需要登录函数计算 AgentRun 控制台**[****4]**，创建服务关联角色（SLR）。

设置环境变量（建议通过 .env 配置您的环境变量）。

```
export AGENTRUN_ACCESS_KEY_ID="your-access-key-id"export AGENTRUN_ACCESS_KEY_SECRET="your-access-key-secret"export AGENTRUN_ACCOUNT_ID="your-account-id"export AGENTRUN_REGION="cn-hangzhou"
```

#### 4. 了解 Agent 如何与 LangChain 集成

使用 from agentrun.integration.langchain import model, sandbox\_toolset 导入 langchain 的集成能力，这里默认提供了 model、sandbox\_toolset、toolset，可以快速创建 langchain 可识别的大模型、工具。

同时，通过 AgentRunServer 可以快速开放 HTTP Server 供其他业务集成。

```
from agentrun.integration.langchain import model, sandbox_toolsetfrom agentrun.sandbox import TemplateTypefrom agentrun.server import AgentRequest, AgentRunServerfrom agentrun.utils.log import logger# 请替换为您已经创建的 模型 和 沙箱 名称MODEL_NAME = "<your-model-name>"SANDBOX_NAME = "<your-sandbox-name>"if MODEL_NAME.startswith("<"):    raise ValueError("请将 MODEL_NAME 替换为您已经创建的模型名称")code_interpreter_tools = []if SANDBOX_NAME and not SANDBOX_NAME.startswith("<"):    code_interpreter_tools = sandbox_toolset(        template_name=SANDBOX_NAME, # 创建好的沙箱模型的名称        template_type=TemplateType.CODE_INTERPRETER, # 沙箱的类型        sandbox_idle_timeout_seconds=300, # 沙箱空闲超时时间（秒）    )else:    logger.warning("SANDBOX_NAME 未设置或未替换，跳过加载沙箱工具。")# ...# 自动启动 http server，提供 OpenAI 协议AgentRunServer(invoke_agent=invoke_agent).start()
```

#### 5. 调用 Agent

```
curl 127.0.0.1:9000/openai/v1/chat/completions \  -XPOST \  -H "content-type: application/json" \  -d '{"messages": [{"role": "user", "content": "Calculate how many r's are in the word 'strawberry'"}], "stream":true}'
```

**云端一键部署：“函数求值计算专家”示例**

平台已为你准备好开箱即用的模板应用——“函数求值计算专家”。该智能体能理解用户输入的数学求值问题（如计算sin(x) + x^2的极值），自动生成数值计算代码并在沙箱中执行，最终返回数值计算分析结果。

#### 1. 登录“函数计算 AgentRun”控制台，创建好模型、沙箱等资源

登录阿里云函数计算 AgentRun 控制台**[****5]**，首先创建好模型和代码解释器沙箱资源：

* 模型管理 >> 大语言模型 >> 添加模型

选择模型供应商、选择模型、配置凭证并点击创建模型，创建好您的模型

* 运行时与沙箱 >> Sandbox 沙箱 >> 创建沙箱模板 >> 代码解释器

选择沙箱配置信息、日志链路追踪信息，点击创建解释器，创建好您的代码解释器资源

#### 2. 进入探索页面，点击“函数求值计算专家”应用，一键部署

进入函数计算 AgentRun 探索页面**[****6]**，点击“函数求值计算专家”应用，一键部署。

进入探索页面，点击函数求值专家，快速部署您的 Agent 应用

选择刚刚创建好的大语言模型和沙箱资源，点击确认创建，创建此 Agent 应用

#### 3. 点击自动生成域名，进入网页体验智能体应用

等待前端后端服务部署完成，点击生成的域名，跳转至智能体应用页面快速体验相关能力。

等待部署成功，点击此按钮，跳转至应用体验页面

您也可以在运行时与沙箱 >> Agent 运行时页面，查看您刚刚部署 Agent 应用的详细信息，基于 WebIDE 也可以进行在线调试与二次开发。

跳转到应用 Web 链接，通过对话体验基于代码解释器沙箱的数值计算能力

***AgentScope × Browser：***

***让智能体“看得见”互联网***

*Cloud Native*

AgentScope 是由阿里通义实验室推出的开源智能体框架，强调模块化与可组合性。结合函数计算 AgentRun 的 Browser Sandbox，你的 Agent 将具备实时联网、信息检索与交互操作的能力。

**本地快速实践**

通过 AgentRun SDK，只需简单配置即可启用浏览器工具。随后，Agent 即可执行如“访问新浪财经，获取今日腾讯控股股价”等指令，并返回结构化数据。

#### 1. 安装 Serverless Devs

运行脚手架，您需要使用 Serverless Devs 工具，请参考对应安装教程。

如果您拥有 NodeJS 开发环境，可以使用 `npm i -g @serverless-devs/s` 快速安装 Serverless Devs。您也可以直接下载 Serverless Devs 二进制程序使用 Serverless Devs。

#### 2. 初始化脚手架应用

使用快速创建脚手架创建您的 Agent。

**注意！**您需要确保您的 python 环境在 3.10 以上。

```
# 初始化模板s init agentrun-finance-demo# 按照实际情况进入代码目录cd agentrun-finance-demo/code/agentrun-backend# 初始化虚拟环境并安装依赖uv venv && uv pip install -r requirements.txt
```

#### 3. 配置认证信息

设置环境变量。（建议通过 .env 配置您的环境变量）

```
export AGENTRUN_ACCESS_KEY_ID="your-access-key-id"export AGENTRUN_ACCESS_KEY_SECRET="your-access-key-secret"export AGENTRUN_ACCOUNT_ID="your-account-id"export AGENTRUN_REGION="cn-hangzhou"
```

#### 4. 了解 Agent 如何与 AgentScope 集成

使用 from agentrun.integration.agentscope import model, sandbox\_toolset 导入 AgentScope 的集成能力，这里默认提供了 model、sandbox\_toolset、toolset，可以快速创建 AgentScope 可识别的大模型、工具。

同时，通过 AgentRunServer 可以快速开放 HTTP Server 供其他业务集成。

```
from agentrun.integration.agentscope import model, sandbox_toolsetfrom agentrun.sandbox import TemplateTypefrom agentrun.server import AgentRequest, AgentRunServerfrom agentrun.utils.log import logger# 请替换为您已经创建的 模型 和 沙箱 名称MODEL_NAME = os.getenv("MODEL", "<your-model-name>")SANDBOX_NAME = os.getenv("BROWSER_TEMPLATE", "<your-sandbox-name>")if MODEL_NAME.startswith("<"):    raise ValueError("请将 MODEL_NAME 替换为您已经创建的模型名称")# ...agent = ReActAgent(    name="agentscope-finance-assistant-agent",    sys_prompt=PROMPT,    model=model(MODEL_NAME),    formatter=OpenAIChatFormatter(),    toolkit=toolkit,    memory=InMemoryMemory(),)# ...# 自动启动 http server，提供 OpenAI 协议AgentRunServer(invoke_agent=invoke_agent).start()
```

#### 5. 基于 AgentRun Sandbox SDK 进行二次开发

在此版代码示例中，通过开源的 AgentRun Sandbox SDK 对浏览器（Browser）沙箱进行灵活二次开发为 AgentScope 原生工具，针对工具调用的开始前准备阶段，可以通过 SDK 创建沙箱示例或者选择一个正在运行的沙箱；在调用结束后，可以通过 SDK 及时删除沙箱，节省资源消耗。通过 AgentRun Sandbox SDK，可以和代码无缝集成，灵活对沙箱的生命周期进行全流程管理操作。

```
_browser_sandbox = Nonedef get_browser_sandbox():    """获取或创建 browser sandbox"""    global _browser_sandbox    if _browser_sandbox is None:        try:            print(f"正在创建 browser sandbox: {SANDBOX_NAME}")            _browser_sandbox = Sandbox.create(                template_type=TemplateType.BROWSER,                template_name=SANDBOX_NAME,                sandbox_idle_timeout_seconds=1800,            )            # 等待 browser 准备就绪（最多尝试15次，每次等待1秒）            max_retries = 15            for i in range(max_retries):                health_status = _browser_sandbox.check_health()                if health_status["status"] == "ok":                    print(f"browser sandbox 准备就绪 id: {_browser_sandbox.sandbox_id}")                    break                import time                time.sleep(1)            else:                # 超过最大重试次数仍未就绪                raise Exception(f"browser sandbox 在 {max_retries} 秒内未能准备就绪")        except Exception as e:            print(f"创建 browser sandbox 失败: {str(e)}")            import traceback            traceback.print_exc()            return None    return _browser_sandboxasync def browser_search(keyword: str, page: int = 1, wait_seconds: float = 1.5) -> ToolResponse:    """使用Bing搜索指定关键词，并查看指定页码的结果，返回搜索结果标题列表    Args:        keyword: 搜索关键词        page: 页码，从1开始，默认为第1页        wait_seconds: 等待页面加载的秒数，默认为1.5秒    Returns:        搜索结果标题列表，每个标题包含文本内容和链接    """    try:        import asyncio        from urllib.parse import quote        # 构造Bing搜索URL        # first参数：第1页=0, 第2页=10, 第3页=20，以此类推        first = (page - 1) * 10 + 1        encoded_keyword = quote(keyword)        url = f"https://www.bing.com/search?q={encoded_keyword}&first={first}"        print(f"[DEBUG] url: {url}")        browser = get_browser_sandbox()        if browser is None:            return make_tool_response("浏览器工具不可用，请透出让用户检查browser是否存在的信息")        async with browser.async_playwright(record=True) as playwright:            try:                # 设置导航超时为5秒，并等待页面加载                await playwright.goto(url, timeout=5000)                # 等待页面加载完成                if wait_seconds > 0:                    print(f"等待 {wait_seconds} 秒以确保页面加载完成")                    await asyncio.sleep(wait_seconds)                # 获取页面HTML内容                html = await playwright.html_content()                print(f"[DEBUG] html length: {len(html)}")            except Exception as page_error:                # 捕获页面操作相关的错误                error_msg = str(page_error)                if "TargetClosedError" in error_msg or "closed" in error_msg.lower():                    return make_tool_response("搜索失败: 页面在加载过程中被关闭，该网站可能不允许自动化访问")                elif "timeout" in error_msg.lower():                    return make_tool_response(f"搜索失败: 页面加载超时（{url}），请稍后重试")                else:                    raise  # 其他错误继续抛出，由外层捕获            # 使用工具函数提取搜索结果            search_results = extract_bing_search_results(html)            if not search_results:                return make_tool_response(f"搜索 '{keyword}'（第{page}页）完成，但未找到搜索结果")            # 格式化输出            formatted_results = []            for i, result in enumerate(search_results, 1):                formatted_results.append(f"{i}. {result['title']}\n   链接: {result['link']}")                print(f"{i}. {result['title']}\n   链接: {result['link']}")            result_text = "\n\n".join(formatted_results)            return make_tool_response(f"成功搜索 '{keyword}'（第{page}页），找到 {len(search_results)} 个结果：\n\n{result_text}")    except Exception as e:        import traceback        traceback.print_exc()        return make_tool_response(f"搜索失败: {str(e)}")async def browser_text_content(url: str, wait_seconds: float = 1.5) -> ToolResponse:    """访问指定网页并获取其纯文本内容（已过滤HTML标签和脚本）    先导航到指定URL，等待页面加载，然后提取页面的可读文本内容。    Args:        url: 要访问的网页URL地址        wait_seconds: 等待页面加载的秒数，默认为1.5秒（大多数网页足够）    返回页面的可读文本内容，已自动过滤：    - 所有HTML标签    - JavaScript代码    - CSS样式    - HTML注释    Returns:        页面的纯文本内容    注意：    - 默认等待1.5秒通常足够大多数网页加载    - 如果内容没有完全加载，可以增加wait_seconds参数重试（如wait_seconds=3.0）    - 某些网页可能会拦截自动化工具访问，导致无法获取内容    - 如果遇到访问失败，可以尝试其他来源的链接    """    # 检查URL是否在黑名单中    import asyncio    try:        browser = get_browser_sandbox()        if browser is None:            return make_tool_response("浏览器工具不可用，请透出让用户检查browser是否存在的信息")        async with browser.async_playwright(record=True) as playwright:            try:                # 先导航到指定URL，设置超时为5秒                await playwright.goto(url, timeout=5000)                # 等待页面加载完成                if wait_seconds > 0:                    await asyncio.sleep(wait_seconds)                # 获取HTML内容                html = await playwright.html_content()                # 使用过滤函数提取纯文本                text = filter_html_to_text(html)                print(f"{url}:\n{text}\n{'=' * 100}")                # 如果文本内容太长，只返回前50000个字符和总长度信息                if len(text) > 50000:                    return make_tool_response(f"成功访问 {url}\n\n文本内容（前10000字符）:\n{text[:50000]}\n\n... (总长度: {len(text)} 字符)")                return make_tool_response(f"成功访问 {url}\n\n文本内容:\n{text}")            except Exception as page_error:                # 捕获页面操作相关的错误                error_msg = str(page_error)                if "TargetClosedError" in error_msg or "closed" in error_msg.lower():                    return make_tool_response(f"获取网页内容失败: {url}\n错误: 页面在加载过程中被关闭\n注意: 该网页可能对自动化工具进行了拦截，建议尝试其他来源")                elif "timeout" in error_msg.lower():                    return make_tool_response(f"获取网页内容失败: {url}\n错误: 页面加载超时（5秒）\n建议: 可以尝试增加 wait_seconds 参数或尝试其他来源")                else:                    raise  # 其他错误继续抛出，由外层捕获    except Exception as e:        return make_tool_response(f"获取网页内容失败: {url}\n错误: {str(e)}\n注意: 该网页可能对自动化工具进行了拦截，建议尝试其他来源")
```

AgentRun Sandbox SDK 中，集成了 PlayWright，用户可以根据上述代码进行二次开发为相应工具，browser\_search()通过必应搜索相关内容。同时，browser\_text\_content()通过网页抓取，为 Agent 提供更多信息。

此外，get\_browser\_sandbox()通过 SDK 二次开发，可以灵活管理 Sandbox 的生命周期，包含创建、删除等一站式生命周期管理。

#### 6. 调用 Agent

```
curl 127.0.0.1:9000/openai/v1/chat/completions \  -XPOST \  -H "content-type: application/json" \  -d '{"messages": [{"role": "user", "content": "查询下当前的上证指数"}], "stream":true}'
```

**云端部署实战：“金融股票专家”智能体**

我们基于此能力打造了“金融股票专家”应用：用户输入股票名称或代码，Agent 自动打开财经网站，抓取最新行情、财报摘要与新闻舆情，综合分析后生成投资建议。

#### 1. 登录“函数计算 AgentRun”控制台，创建好模型、沙箱等资源

登录阿里云函数计算 AgentRun 控制台，首先创建好模型和代码解释器沙箱资源：

* 模型管理 >> 大语言模型 >> 添加模型

选择模型供应商、选择模型、配置凭证并点击创建模型，创建好您的模型

* 运行时与沙箱 >> Sandbox 沙箱 >> 创建沙箱模板 >> 浏览器

#### 2. 进入探索页面，点击“股票金融专家”应用，一键部署

进入函数计算 AgentRun 探索页面，点击“股票金融专家”应用，一键部署。

进入探索页面，点击股票金融专家快速部署您的 Agent 应用

选择刚刚创建好的大语言模型和沙箱资源，点击确认创建，创建此 Agent 应用

#### 3. 点击自动生成域名，进入网页体验智能体应用

等待前端后端服务部署完成，点击生成的域名，跳转至智能体应用页面快速体验相关能力。

等待部署成功，点击此按钮，跳转至应用体验页面

您也可以在运行时与沙箱 >> Agent 运行时页面，查看您刚刚部署 Agent 应用的详细信息，基于 WebIDE 也可以进行在线调试与二次开发。

跳转到应用 Web 链接，通过对话体验基于浏览器沙箱的网页内容检索能力

***结语：智能随心，开发成趣***

*Cloud Native*

Agentic AI 的未来，不应被基础设施的复杂性所束缚。AgentRun Sandbox SDK 的开源，正是为了打破这一壁垒。

无论你是 LangChain 的忠实用户，还是 AgentScope 的探索者；无论你在本地调试原型，还是在云端部署生产级应用——函数计算 AgentRun 都能为你提供安全、弹性、免运维的沙箱运行时，让每一个智能体都能轻盈地奔跑在云端。

**现在就加入我们！**

* GitHub 开源地址：https://github.com/Serverless-Devs/agentrun-sdk-python
* 文档与示例：https://docs.agent.run/
* 加入“函数计算 AgentRun 客户群”群的钉钉群号：134570017218

欢迎 Star、Fork、提 Issue，一起共建开放的 Agentic 生态！

**智能随心，开发成趣 —— 函数计算 AgentRun，让智能体开发回归创造力本身。**

# 相关链接：

# [1] AgentRun

# https://functionai.console.aliyun.com/agent/explore [2] 安装教程

https://serverless-devs.com/docs/user-guide/install

# [3] Serverless Devs 二进制程序

https://github.com/Serverless-Devs/Serverless-Devs/releases

# [4] AgentRun 控制台

https://functionai.console.aliyun.com/agent/explore

# [5] AgentRun 控制台

https://functionai.console.aliyun.com/welcome

[6] AgentRun 探索页面

https://functionai.console.aliyun.com/agent/explore