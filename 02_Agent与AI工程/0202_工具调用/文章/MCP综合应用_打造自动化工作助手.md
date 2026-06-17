---
title: MCP综合应用:打造自动化工作助手
author: AI出海Zack
date: 
url: https://mp.weixin.qq.com/s?__biz=MzY0MDIxNTE4Nw==&mid=2247483785&idx=1&sn=dd73314697c67a602b7a012b7ffdc8b8&chksm=f1983575615830a7aa26e04a9485301fbc1917ba4b9b772d818c2ef6fc89b856de36a44cb2ef&mpshare=1&scene=24&srcid=1204ci5VADLJEDqKV4d5sded&sharer_shareinfo=f89ae26c870fa5bf8dc1b5f2d04a5094&sharer_shareinfo_first=f89ae26c870fa5bf8dc1b5f2d04a5094#rd
---

## 单一工具为何不够用?

一个能查数据库的Agent很好，但真实业务场景往往需要同时访问文件系统、查询数据库、检索知识库。如果让Agent只用一种工具,就像让厨师只用一把刀做满汉全席——理论上可行，实际上低效。本文将整合3个MCP服务器,构建一个能自动生成周报的全能工作助手。

## 概念速览

| 概念 | 说明 | 应用场景 |
| --- | --- | --- |
| **多MCP服务器集成** | 同时连接多个MCP服务器,提供不同能力 | 一个Agent同时访问文件、数据库、知识库 |
| **智能工具路由** | Agent根据任务自动选择合适的工具 | 数据分析用数据库,资料查找用文件系统 |
| **工具协同** | 多个工具配合完成复杂任务 | 从数据库获取数据→写入文件→知识库总结 |
| **业务编排** | 将多个工具调用组合成业务流程 | 周报生成=数据查询+文件整理+内容总结 |

**3个MCP服务器能力对比**:

* 🗂️ **文件系统MCP**:读写本地文件、搜索文档
* 🗄️ **数据库MCP**:查询业务数据、生成统计报表
* 📚 **知识库MCP**:检索文档、总结知识点

## 实战

### （受限于篇幅，仅展示流程，完整代码见文末 GitHub 地址。）

### 步骤1:整合MCP服务器

创建统一的MCP客户端管理器。

```
# mcp_manager.py 

import asyncio 

from mcp import ClientSession, StdioServerParameters 

from mcp.client.stdio import stdio_client 

from typing import Dict, List 

 

class MCPManager: 

    """管理多个MCP服务器连接""" 

     

    def __init__(self): 

        self.sessions: Dict[str, ClientSession] = {} 

        self.tools: Dict[str, List] = {} 

     

    async def connect_server(self, name: str, command: str, args: List[str]): 

        """连接单个MCP服务器""" 

        server_params = StdioServerParameters( 

            command=command, 

            args=args 

        ) 

         

        # 启动服务器连接 

        read, write = await stdio_client(server_params).__aenter__() 

        session = ClientSession(read, write) 

        await session.initialize() 

         

        self.sessions[name] = session 

         

        # 获取该服务器提供的工具列表 

        tools_response = await session.list_tools() 

        self.tools[name] = tools_response.tools 

         

        print(f"✅ 已连接 {name} MCP服务器,提供 {len(self.tools[name])} 个工具") 

     

    async def connect_all(self): 

        """连接所有MCP服务器""" 

        # 1. 文件系统服务器 

        await self.connect_server( 

            name="filesystem", 

            command="python", 

            args=["filesystem_mcp_server.py"] 

        ) 

         

        # 2. 数据库服务器 

        # ...

         

        # 3. 知识库服务器 

        # ...

     

    async def call_tool(self, server_name: str, tool_name: str, args: dict): 

        """调用指定服务器的工具""" 

        session = self.sessions.get(server_name) 

        if not session: 

            raise ValueError(f"服务器 {server_name} 未连接") 

         

        result = await session.call_tool(tool_name, args) 

        return result 

     

    def get_all_tools(self) -> List[dict]: 

        """获取所有可用工具列表""" 

        all_tools = [] 

        for server_name, tools in self.tools.items(): 

            for tool in tools: 

                all_tools.append({ 

                    'server': server_name, 

                    'name': tool.name, 

                    'description': tool.description 

                }) 

        return all_tools
```

### 步骤2:实现智能工具路由

让Agent根据任务自动选择合适的工具。

```
# tool_router.py 

from langchain.tools import StructuredTool 

from typing import List 

import asyncio 

 

class ToolRouter: 

    """智能工具路由器""" 

     

    def __init__(self, mcp_manager: MCPManager): 

        self.mcp_manager = mcp_manager 

     

    def create_langchain_tools(self) -> List[StructuredTool]: 

        """将MCP工具转换为LangChain工具""" 

        tools = [] 

         

        # 文件系统工具 

        tools.append(StructuredTool.from_function( 

            func=lambda path: asyncio.run( 

                self.mcp_manager.call_tool("filesystem", "read_file", {"path": path}) 

            ), 

            name="read_file", 

            description="读取指定路径的文件内容" 

        )) 

         

        #...

         

        # 数据库工具 

        #...

         

        # 知识库工具 

        #...

         

        return tools
```

### 步骤3:构建周报生成业务流程

完整实现从数据收集到报告生成的自动化流程。

```
# weekly_report_agent.py 

from langchain_openai import ChatOpenAI 

from langchain.agents import AgentExecutor, create_tool_calling_agent 

from langchain.prompts import ChatPromptTemplate 

from datetime import datetime, timedelta 

 

class WeeklyReportAgent: 

    """周报自动生成Agent""" 

     

    def __init__(self, mcp_manager: MCPManager): 

        self.mcp_manager = mcp_manager 

        self.router = ToolRouter(mcp_manager) 

        self.llm = ChatOpenAI(model="gpt-4o", temperature=0) 

        self.agent = self._create_agent() 

     

    def _create_agent(self): 

        """创建Agent""" 

        tools = self.router.create_langchain_tools() 

         

        prompt = ChatPromptTemplate.from_messages([ 

            ("system", """你是一个高效的工作助手,专门生成周报。  
  
任务流程:  
1. 使用 query_database 查询本周的关键数据(销售额、订单量等)  
2. ....  
  
周报格式要求:  
- 本周工作总结(3-5条)  
- ....

"""), 

            ("human", "{input}"), 

            ("placeholder", "{agent_scratchpad}") 

        ]) 

         

        agent = create_tool_calling_agent(self.llm, tools, prompt) 

        return AgentExecutor(agent=agent, tools=tools, verbose=True) 

     

    def generate_report(self, output_path: str = "weekly_report.md") -> str: 

        """生成周报""" 

        # 计算本周时间范围 

        today = datetime.now() 

        week_start = today - timedelta(days=today.weekday()) 

        week_end = week_start + timedelta(days=6) 

         

        task = f"""  
请生成 {week_start.strftime('%Y-%m-%d')} 至 {week_end.strftime('%Y-%m-%d')} 的周报。  
  
具体要求:  
1. 查询数据库获取本周销售数据  
2. ....  
""" 

         

        result = self.agent.invoke({"input": task}) 

        return result['output']
```

### 步骤4:完整运行演示

```
# demo_weekly_report.py 

import asyncio 

 

async def main(): 

    """周报生成完整演示""" 

     

    # 1. 初始化MCP管理器 

    print("🚀 启动MCP服务器...") 

    mcp_manager = MCPManager() 

    await mcp_manager.connect_all() 

     

    # 2. 显示可用工具 

    print("📋 可用工具列表:") 

    for tool in mcp_manager.get_all_tools(): 

        print(f"  - [{tool['server']}] {tool['name']}: {tool['description']}") 

     

    # 3. 创建周报Agent 

    print("🤖 创建周报生成Agent...") 

    agent = WeeklyReportAgent(mcp_manager) 

     

    # 4. 生成周报 

    print("📝 开始生成周报...") 

    result = agent.generate_report("reports/weekly_2024W48.md") 

     

    print("✅ 周报生成完成!") 

    print(f"{result}") 

 

if __name__ == "__main__": 

    asyncio.run(main())
```

**自动化收益**:

* ⏱️ **时间节省**:人工生成周报需1小时,现在1-3分钟完成
* 📊 **数据准确**:自动从数据库获取,避免人工统计错误
* 📝 **格式统一**:使用固定模板,保证周报格式一致性
* 🔄 **可复用**:月报、季报只需调整时间范围即可复用

**生成的周报质量**:包含结构化总结、准确数据、问题分析和下周计划,符合企业周报标准。

## 总结

**多MCP服务器集成实践**:

1. **统一管理**:用MCPManager集中管理所有服务器连接,避免重复代码
2. **工具转换**:将MCP工具封装为LangChain工具,复用Agent生态
3. **智能路由**:让LLM自动选择工具,无需硬编码业务逻辑
4. **业务编排**:用自然语言描述流程,Agent自动完成多步骤任务
5. **错误隔离**:单个MCP服务器失败不影响其他服务器,保证系统健壮性

本文整合多个MCP服务器,构建了一个"全能助手",它能跨系统协同工作,自动完成复杂的业务流程。(当然真实企业应用会更庞大，调用更多工具、涉及更多的流程，比如 vibe coding 工具这种复杂系统。)

完整代码已上传 https://github.com/afewnotes/mcp-demos