---
title: 3步！教会你用 Doris MCP + LangChain 搭建AI问数系统（保姆级教程）
author: 一臻数据
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI3NjA1MTcyMQ==&mid=2650488377&idx=1&sn=5725e019140fd435bfdc097eee7d76ff&chksm=f2a25d876b173b46f851e01419e3dbd3f7ee2028378503ee45d61fa143ef32409ad04ebbfc1c&mpshare=1&scene=24&srcid=0830aRAjbQlTDKuud7DzUO7z&sharer_shareinfo=0f47fa6a9c551bae06917b7d26930a8c&sharer_shareinfo_first=0f47fa6a9c551bae06917b7d26930a8c#rd
---

见字如面，我是一臻 90后新手奶爸，探索Doris x AI

点击关注 👇 免费获取数字AI知识库

> ❝
>
> 白天刚哄完娃，正巧看到 `Doris x AI` 群有个哥们在群里问："**有没有大佬落地过AI问数这一类的项目**"。 
>
> 这在以往可能还得单独在AI应用中维护Doris的元数据、Comment信息和Connector等，使得系统能够较为准确地结合LLM与Doris交互。 
>
> 但现在，实现起来变得非常简单了。只需要通过**Doris MCP + LangChain**组合，则可以**直接用自然语言与Doris数据库交互**。

## 前言

传统查询模式存在几个核心问题：**SQL语法学习成本高 -> 复杂查询编写难度大 -> 查询结果缺乏业务语境的解释**

MCP协议的引入恰巧能简化这些问题。该协议定义了AI应用与外部数据源之间的标准化通信接口，使得不同系统能够通过统一的方式进行数据交换。

而要真正解决传统查询模式的痛点，仅有协议标准还不够，还需要强大的数据处理引擎和易用智能化的AI框架。这正是Apache Doris和LangChain发挥价值的地方。

Apache Doris作为现代化的实时分析数据库，在性能、扩展性、易用性和生态方面都有着显著优势。其基于MPP架构的向量化执行引擎，能够将查询性能相比传统数据库提升数倍，轻松支持PB级数据的秒级响应。同时，Doris支持在线扩缩容，既能应对业务增长带来的数据量激增，也能满足实时决策的业务需求，大幅降低了运维复杂度，让企业能够专注于业务价值而非技术细节。

在AI应用开发层面，LangChain框架则提供了完整的解决方案。不仅能够构建具备复杂推理能力的智能代理，还拥有丰富的工具生态系统，可以无缝集成各类外部服务。LangChain对主流LLM提供商的兼容支持，确保了最佳的性能和成本平衡，而其内置的优化机制，更是帮助开发者快速构建高质量的AI交互体验。

当Apache Doris遇上LangChain，产生的不仅仅是技术叠加，更是能力的指数级提升。再结合MCP协议，LangChain能够直接理解Doris的表结构和数据特征，自动生成最优查询策略，实现真正的智能化数据访问。

例如当用户输入"`哪个客户下单最多`"这样的自然语言查询时，Doris MCP + LangChain能够自动进行意图识别、调用对应的 Doris MCP Tool 生成最优的 Doris SQL、执行查询并返回包含深度商业洞察的分析结果。这种无缝且易用的技术融合，实现了"`让数据会说话，让洞察触手可及`"的愿景。

## 第一步｜准备环境

fei话到此，接下来直接实测体验下效果如何。

开局我们需要简单准备下基础环境，主要分为Doris、Python和Doris MCP Server环境即可：

##### 1. Doris

如果已经有现成的Doris集群，直接用即可。

若当前还没Doris环境，可以参考Doris官方文档，基于Docker或本地化快速部署搭建一套Doris集群 🔗 ：`https://doris.apache.org/zh-CN/docs/dev/gettingStarted/quick-start`

实测数据集主要使用 TPC-H Benchmark 🔗：`https://doris.apache.org/zh-CN/docs/benchmark/tpch`

##### 2. Python环境

Python版本需要>=**3.12**

主流程coding所需python包（requirements.txt）：

```
# LangChain MCP 相关依赖  
langchain-mcp-adapters  
langchain  
langchain-community  
langchain-openai  
  
# 环境变量管理  
python-dotenv  
  
# 异步支持  
aiohttp  
aiofiles
```

##### 3. Doris MCP Server

Doris MCP Server 本文荐直接用git clone到本地的方式：

```
# 1. MCP Server 克隆到本地：  
git clone https://github.com/apache/doris-mcp-server.git  
cd /本地仓库oris-mcp-server的路径  
  
# 2. 安装依赖：  
pip install -r requirements.txt  
  
# 3. 配置数据库信息  
cp .env.example .env  
vim .env  
## Doris FE connection settings  
DORIS_HOST=xxx  
DORIS_PORT=9030  
DORIS_USER=root  
DORIS_PASSWORD=xxx  
DORIS_DATABASE=xxx  
  
# 4. 启动doris mcp server服务  
./start_server.sh &
```

最后一行出现如下日志 `Uvicorn running on` 则表示启动成功：

## 第二步｜手撕代码

经过七七四十九秒的编写调试，完整代码如下：

```
"""  
Doris MCP + LangChain AI Query System  
Intelligent database query system with business context analysis.  
"""  
  
import asyncio  
import logging  
from typing import Dict, Any, Optional  
from langchain.agents import AgentExecutor, create_openai_tools_agent  
from langchain.chat_models import init_chat_model  
from langchain.prompts import ChatPromptTemplate  
from langchain_mcp_adapters.client import MultiServerMCPClient  
  
# Configuration Constants  
DEEPSEEK_API_KEY = "your_deepseek_api_key_here"  
MODEL_NAME = "deepseek-chat"  
MODEL_PROVIDER = "deepseek"  
MCP_CONFIG = {  
    "doris_mcp_server": {  
        "transport": "streamable_http",  
        "url": "http://localhost:3000/mcp"  
    }  
}  
class PromptManager:  
    """Business context prompt management for Doris AI assistant."""  
      
    SYSTEM_PROMPT = """  
🤖 你是基于Apache Doris数据库的专业AI问数系统，具备强大的数据分析和业务洞察能力。  
  
核心职责：  
1. 分析用户问题语境，理解业务需求  
2. 精准调用Doris MCP Server工具执行查询  
3. 提供深度业务语境分析和专业解释  
4. 将技术数据转化为可执行的业务洞察  
  
回答格式要求（必须包含以下结构）：  
📈 查询结果：准确的数据查询结果，使用表格或图表符号展示  
🔍 业务解读：数据背后的业务含义和趋势分析  
💎 关键洞察：重要发现、异常点和数据亮点  
🚀 行动建议：基于数据的具体可执行建议  
  
展示风格：  
- 使用📊📈📉等图表符号增强数据可视化效果  
- 适当使用✅❌⚠️等状态符号标识重要信息  
- 用🔥💪⭐等表情符号突出关键发现  
- 保持专业、准确、有洞察力的分析风格  
- 让数据"说话"，用生动的方式传达专业见解  
    """.strip()  
      
    @classmethod  
    def enhance_query(cls, query: str) -> str:  
        """Enhance user query with business context."""  
        returnf"{cls.SYSTEM_PROMPT}\n\n用户查询：{query}\n请执行查询并提供业务分析，包括数据洞察、业务影响和行动建议。"  
  
class Config:  
    """Application configuration management."""  
      
    def __init__(self,   
                 api_key: Optional[str] = None,  
                 model: Optional[str] = None,  
                 provider: Optional[str] = None,  
                 mcp_config: Optional[Dict[str, Any]] = None) -> None:  
          
        self.api_key = self._validate_api_key(api_key or DEEPSEEK_API_KEY)  
        self.model = model or MODEL_NAME  
        self.provider = provider or MODEL_PROVIDER  
        self.mcp_config = mcp_config or MCP_CONFIG  
      
    @staticmethod  
    def _validate_api_key(key: str) -> str:  
        """Validate API key configuration."""  
        ifnot key or key == "your_deepseek_api_key_here":  
            raise ValueError("Valid DeepSeek API key required")  
        return key  
  
class DorisMCPAgent:  
    """Doris MCP Agent with intelligent business context analysis."""  
      
    def __init__(self, config: Config) -> None:  
        self.config = config  
        self._mcp_client: Optional[MultiServerMCPClient] = None  
        self._agent_executor: Optional[AgentExecutor] = None  
      
    asyncdef initialize(self) -> None:  
        """Initialize MCP client and LangChain agent."""  
        self._mcp_client = MultiServerMCPClient(self.config.mcp_config)  
          
        tools = await self._mcp_client.get_tools()  
        ifnot tools:  
            raise RuntimeError("Failed to load tools from MCP server")  
          
        llm = init_chat_model(  
            model=self.config.model,  
            model_provider=self.config.provider,  
            api_key=self.config.api_key  
        )  
          
        prompt = ChatPromptTemplate.from_messages([  
            ("system", PromptManager.SYSTEM_PROMPT),  
            ("human", "{input}"),  
            ("placeholder", "{agent_scratchpad}")  
        ])  
          
        agent = create_openai_tools_agent(llm, tools, prompt)  
        self._agent_executor = AgentExecutor(  
            agent=agent,  
            tools=tools,  
            verbose=True,  
            max_iterations=3  
        )  
      
    asyncdef run_interactive(self) -> None:  
        """Start interactive chat session."""  
        ifnot self._agent_executor:  
            raise RuntimeError("Agent not initialized")  
          
        print("\n" + "="*60)  
        print("🤖 欢迎使用 Doris AI 问数系统 🤖")  
        print("="*60)  
          
        print("🔥 您可以这样问我：")  
        print("   1️⃣  当前Doris有哪些库表？")  
        print("   2️⃣  请帮我切换到tpch库并分析哪个客户下单最多")  
        print("   3️⃣  等等等...")  
          
        print("\n💬 输入您的问题开始分析，输入 'quit' 退出系统")  
        print("="*60 + "\n")  
          
        whileTrue:  
            try:  
                query = input("You: ").strip()  
                  
                if query.lower() in {'quit', 'exit', 'q'}:  
                    print("Goodbye!")  
                    break  
                  
                ifnot query:  
                    continue  
                  
                enhanced_query = PromptManager.enhance_query(query)  
                result = await self._agent_executor.ainvoke({"input": enhanced_query})  
                print(f"\nAI: {result['output']}\n")  
                  
            except KeyboardInterrupt:  
                print("\nGoodbye!")  
                break  
            except Exception as e:  
                print(f"Error: {e}")  
  
asyncdef main() -> None:  
    """Application entry point."""  
    logging.basicConfig(level=logging.WARNING)  
      
    try:  
        config = Config()  
        agent = DorisMCPAgent(config)  
        await agent.initialize()  
        await agent.run_interactive()  
    except Exception as e:  
        print(f"Error: {e}")  
        print("Please check the configuration section")  
  
  
if __name__ == "__main__":  
    try:  
        asyncio.run(main())  
    except KeyboardInterrupt:  
        print("\nInterrupted")  
    except Exception as e:  
        print(f"Startup failed: {e}")
```

##### 代码解析

代码经过精简，很多block没有进行过多地细化深入。主要是为了让大家能够快速熟悉Doris MCP + LangChain 搭建AI问数系统的完整流程，后续可以结合自己需求，按模块进行调整应用。

代码主流程如下：

Doris AI 问数系统是一个基于异步架构的智能数据库查询系统，将MCP协议和LangChain框架相结合，为用户提供了一种全新的自然语言数据查询体验。

从系统的执行流程来看，整个过程可以分为三个关键阶段：`初始化准备、工具链构建和智能交互`。

**初始化阶段**，通过Config模块验证API密钥等关键配置，创建DorisMCPAgent作为整个系统的管理中心。

接下来进入**工具链构建阶段**，会建立与Doris数据库的MCP连接，同时初始化DeepSeek聊天模型，并将这些组件整合成一个完整的AgentExecutor执行器。

最后在**智能交互阶段**，进入持续的用户对话循环，每当用户提出查询时，PromptManager会自动为查询添加专业的业务分析语境，然后LangChain Agent会调用相应的数据库工具执行查询，并生成包含深度商业洞察的分析报告。

整体架构上，`异步编程架构`确保了系统在处理复杂查询时依然能够保持高效响应，而MCP协议的引入则实现了 AI与Doris数据库的标准化交互，并通过PromptManager的`智能增强`模块，能够自动为每个查询添加商业分析的专业视角，同时也能结合企业的业务场景进行DIY，使得返回的结果不再是冰冷的数据，而是富有洞察力的商业智能报告。

## 第三步｜验证效果

代码编写调试并熟悉后，来验证下效果如何👇

首先，直接提问`当前Doris有哪些库表？`，确认整体流程是否有误：

结果正确，和Doris的库表信息一致，并且给出了`数据库表的分类、个别库表的洞察分析和改善建议`。

再进行自然语言查询分析测试`分析哪个客户下单最多`：

结果正确。Doris AI 问数系统自动将自然语言转为了Doris SQL进行查询并返回结果，并且给出了专业的`业务解读、关键洞察和行动建议`。

怎一个**香**字了得？

## 结语

随着MCP协议的进一步完善和AI技术的持续演进，我们有理由相信，AI+数据库的交互模式将成为企业数字化转型的重要推动力。让数据会说话，让洞察触手可及的愿景，正在从理想照进现实。

技术的美妙之处在于化繁为简，而 **Doris MCP + LangChain** 恰恰做到了这一点。不妨动手试试，或许你会发现，**与数据直接对话原来可以如此简单而优雅**。

**完**

---

👇欢迎扫描下方二维码 👇 

备注 **666**免费领取资料  加入Doris官方群和PowerData数据社区❗️

#大数据 #开源 #OLAP #Doris #MCP #LangChain #人工智能 #AI