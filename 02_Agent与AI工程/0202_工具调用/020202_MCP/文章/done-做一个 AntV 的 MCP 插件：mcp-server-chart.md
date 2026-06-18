> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: 做一个 AntV 的 MCP 插件：mcp-server-chart
author: 数据可视化 AntV
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg3NTU4OTc3OA==&mid=2247497501&idx=1&sn=19437ce15cc126517d66e29ab33767d1&chksm=ce6703a0711ef26a7a7f748fc1cbc2e208854fa204576343649746fe0837f281af0aa34eda99&mpshare=1&scene=24&srcid=0430NzwgLq1VKTYlQtMwHeen&sharer_shareinfo=727a4a0596adaf5ba62ebe62b941df83&sharer_shareinfo_first=727a4a0596adaf5ba62ebe62b941df83#rd
---

## 背景

当前 AI 模型因数据孤岛限制而无法充分发挥潜力，MCP 方案的出现，使得 AI 应用能够安全地访问和操作本地及远程数据，为 AI 应用提供了连接万物的接口，解决了数据获取的问题。

简单来讲，MCP 有以下作用：

* **拓展 AI 能力**：让 LLM 不止能回答问题，还能与外部系统交互，拓展应用场景，这里的外部系统除了公开的三方服务（例如高德）外，还包括用户本地的软件（例如 Chrome 浏览器）。
* **标准化接口**：提供统一的协议，降低 AI 工具集成的复杂度，可以简单理解外系统公开的 OpenAPI 或 Function Call
* **安全可控**：LLM 与 MCP service 之间的交互是透明的，这里包括了 LLM 什么时候调用什么工具，以及调用结果。
* **灵活拓展**：可以根据需要，开发自定义 MCP 服务，降低了 LLM 与业务结合的成本。

在 AI 应用平台中，很重要的概念就是 Tool / 服务，其中数据可视化在数据可视化、展现上就是很重要的一个 Tool， 所以，我们计划 **基于 AntV 实现一套 AI 可视化插件，以及 MCP 服务能力，在百宝箱、Dify 上应用。**

## 整体流程

目标：基于面向 AI 消费的 GPT-Vis 可视化组件库，通过 MCP 服务能力实现一套高效出图的 AI 可视化插件， 在各大 AI 应用开发平台（dify、百宝箱）落地。

画板

相关涉及的内容：

* 可视化图表 GPT-Vis[1]，提供 20+ 可供AI消费的图表；
* G2、G6 支持 SSR 渲染，基于此，实现 GPT-Vis SSR [2]支持图表的 SSR 渲染和图表的描述；
* GPT-Vis SSR 资产[2] 提供静态出图的服务，在内部使用 NodeJS 搭建，然后对外可访问；
* 依据协议，实现 MCP Server[3] 插件并开源（mcp-server-chart），欢迎使用和 star。
* 上线 mcp 市场和 Agent 平台。

## MCP Server 开发

### 如何开发一个MCP Tools

MCP Tools  是 Model Context Protocol 的核心组件，允许服务端向客户端暴露可执行功能模块。通过 Tools，LLMs 可实现：执行系统级操作（文件管理/命令执行）；集成第三方 API（GitHub/邮件服务/天气）；处理结构化数据（CSV分析/数据库查询）等。

具体实现方式请参考官网： https://modelcontextprotocol.io/docs/concepts/tools[4]。

### 如何调试

#### 开发过程中 API 调用调试

```
npx @modelcontextprotocol/inspector node build/index.js
```

启动之后，就可以看到调试界面了。

#### 客户端调试

1.安装 Cline 插件

2.配置 mcpServer

3.配置模型 （📢： 默认 API Provider 收费）百炼控制台[5]（阿里云百炼模型配置）

4.调试 mcpServer

### 上架

Glama[6] 需提 PR 至 https://github.com/punkpeye/awesome-mcp-servers[7]，其他各类 mcp 市场基本都有提交流程。

## MCP 插件使用

在 Cline 中增加如下配置接入即可。

```
{  "mcpServers": {    "mcp-server-chart": {      "command": "npx",      "args": [        "-y",        "@antv/mcp-server-chart"      ]    }  }}
```

## 未来规划

1. 开源代码增加单测，保证后续的可持续迭代；
2. 不断优化图表 json schema 配置，针对双轴、散点、鱼骨图等可以准确推荐；
3. 尝试上架至 dify、coze 等 Agent 平台；
4. 提供更稳定的服务。

### Reference

[1] 

GPT-Vis: *https://gpt-vis.antv.vision/*

[2] 

GPT-Vis SSR : *https://github.com/antvis/GPT-Vis/tree/main/bindings/gpt-vis-ssr*

[3] 

MCP Server: *https://github.com/antvis/mcp-server-chart*

[4] 

https://modelcontextprotocol.io/docs/concepts/tools: *https://modelcontextprotocol.io/docs/concepts/tools*

[5] 

百炼控制台: *https://bailian.console.aliyun.com/?tab=doc#/doc/?type=model&url=https%3A%2F%2Fhelp.aliyun.com%2Fdocument\_detail%2F2880898.html*

[6] 

Glama: *https://glama.ai/mcp/servers?query=mcp-server-chart&sort=search-relevance%3Adesc*

[7] 

https://github.com/punkpeye/awesome-mcp-servers: *https://github.com/punkpeye/awesome-mcp-servers/pull/797#event-17430016847*