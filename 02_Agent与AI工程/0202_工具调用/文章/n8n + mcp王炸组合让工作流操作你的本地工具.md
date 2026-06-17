---
title: n8n + mcp王炸组合让工作流操作你的本地工具
author: 皮皮特的AI工厂
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkyNzIwOTgwNA==&mid=2247488716&idx=1&sn=362d8ecf82560f8fbe3b3fbda32994cb&chksm=c3014de0906c884d393a20a13d26c0f991a4d2481d3aae9e9bb552faf117efd5780c01becf52&mpshare=1&scene=24&srcid=09257eHkTt5ImU3G0D4ivJzg&sharer_shareinfo=fa428ca9992b79741be55618b8822c04&sharer_shareinfo_first=fa428ca9992b79741be55618b8822c04#rd
---

我们有了n8n之后，整个工作流就可以执行，但是如果我们想操作别的工具在大模型里，就需要我们的mcp服务了。

MCP的目标在于成为“ AI集成领域的USB-C”，支持AI应用程序与多种数据存储库、工具或API之间实现一对多的高效连接。通过标准化AI助手查询及与外部资源交互的方式，MCP显著降低了多个定制集成所带来的复杂性。

将 MCP 想象成餐馆中的服务员，你（ AI助手）安坐于桌前下单点餐。服务员（MCP）接收你的订单后，将其传递至厨房（各类数据库、API 或工具）。厨房（MCP 服务器）承接订单（获取数据或运行函数），再把做好的菜肴（所需的数据或操作结果）交还给服务员。服务员随后将菜品端回给你。如此一来，你无需亲自走进厨房掌勺烹饪（直接的 API 集成），只需告知服务员需求，MCP 便会代劳处理后续事务。

有了mcp的认知之后我们看看如果在n8n中使用mcp，我们拿使用n8n引入大模型之后调用baidu地图获取路线为例。

### **安装MCP社区节点**

n8n也包含MCP支持但是他支持的协议比较单一只支持sse协议，但是社区节点支持的更好，所以需要手动安装社区节点：

* • 登录n8n后进入个人账号设置页面
* • 点击"社区节点"

• 选择"安装一个社区节点"，输入`n8n-nodes-mcp`

### **创建MCP AI工作流**

安装MCP节点后，可以创建AI工作流：

**创建On Chat Message触发器**：作为工作流的起点

**添加AI Agent节点**：配置AI模型（如chatgpt）

如果使用官方提供的mcp节点如果调用的第三方支持sse协议则直接输入sse协议地址就行，例如

**添加MCP Tool节点（社区mcp节点）**：点击AI Agent下的Tool加号添加

* • 这是n8n连接MCP服务的核心通道
* • 配置MCP服务器地址和认证信息

接入百度地图，baidu的mcp可以在这找到：https://lbs.baidu.com/faq/api?title=mcpserver/quickstart

我们需要一个baidu的apikey，可以在这生成一个

https://lbsyun.baidu.com/apiconsole/key

之后我们配置MCP client

其实核心就是配置这个json

```
{  
    "mcpServers": {  
        "baidu-map": {  
            "command": "npx",  
            "args": [  
                "-y",  
                "@baidumap/mcp-server-baidu-map"  
            ],  
            "env": {  
                "BAIDU_MAP_API_KEY": "{您的AK}"  
            }  
        }  
    }  
}
```

注意操作部分，第一个mcp是“List Tool”用于列出MCP服务器上的所有工具。该节点的主要功能是返回Breefus Search工具列表，供上层大模型节点使用。因此，上层参数才会指示大模型选择工具。

新增一个MCP工具节点，选择“Execute Tool”操作。

**如果想获取更多AI玩法，可以加入社群后我拉你进群直接学习。**

**说一下共学信息群**

> 我建了一个AI共学信息群，助力大家能快速上手AI工具，群内有各种最新的AI玩法教程和前沿信息，Coze工作流空间，自研的各种助力插件免费使用。加入共学群的朋友还可同步被拉入Coze团队空间，获取我过去分享过的各类工作流文件，从初级到高级一应俱全，帮助你更快掌握使用技巧，想了解共学群的友友可以扫码添加上面微信进行详细了解。