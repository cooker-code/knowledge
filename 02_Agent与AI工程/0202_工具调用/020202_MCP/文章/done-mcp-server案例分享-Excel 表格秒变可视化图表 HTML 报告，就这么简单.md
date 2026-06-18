> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: mcp-server案例分享-Excel 表格秒变可视化图表 HTML 报告，就这么简单
author: 海老豹666
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247487018&idx=1&sn=b6d0c29bf62e348bbd9247e8e3e592e4&chksm=ce1cd34c0f495311c4231250dd91f96e6eb474c309c3808ab54917c62463464da1c64437aa21&mpshare=1&scene=24&srcid=0507f2l13X3e8j6qc2u2eUhl&sharer_shareinfo=55a16219db431d5f298ad3c81dd2230b&sharer_shareinfo_first=55a16219db431d5f298ad3c81dd2230b#rd
---

#

# 1.前言

MCP Server（模型上下文协议服务器）是一种基于模型上下文协议（Model Context Protocol，简称MCP）构建的轻量级服务程序，旨在实现大型语言模型（LLM）与外部资源之间的高效、安全连接。MCP协议由Anthropic公司于2024年11月开源，其核心目标是解决AI应用中数据分散、接口不统一等问题，为开发者提供标准化的接口，使AI模型能够灵活访问本地资源和远程服务，从而提升AI助手的响应质量和工作效率。

### MCP Server 的架构与工作原理

MCP Server 采用客户端-服务器（Client-Server）架构，其中客户端（MCP Client）负责与服务器建立连接，发起请求，而服务器端则处理请求并返回响应。这种架构确保了数据交互的高效性与安全性。例如，客户端可以向服务器发送请求，如“查询数据库中的某个记录”或“调用某个API”，而服务器则根据请求类型，调用相应的资源或工具，完成任务并返回结果。

MCP Server 支持动态发现和实时更新机制。例如，当新的资源或工具被添加到服务器时，客户端可以自动感知并使用这些新功能，从而提高系统的灵活性和扩展性

### MCP Server 的主要功能

1. 1. **资源暴露与工具提供**：
   MCP Server 可以将本地文件、数据库、API等资源作为数据实体暴露给AI模型，同时提供工具功能，帮助AI完成复杂任务，如数据检索、内容生成、实时更新等。例如，它支持对MySQL、PostgreSQL等数据库的查询和操作，也支持对本地文件系统的读写和目录管理。
2. 2. **会话管理与动态通知**：
   MCP Server 能够管理客户端与服务器的连接，确保会话的时效性和稳定性，同时通过实时推送机制，将最新的资源信息及时传递给AI模型，以保证数据的准确性和实时性。
3. 3. **安全性与隐私保护**：
   MCP Server 采用加密认证和访问控制机制，确保数据传输的安全性，避免敏感信息泄露。例如，它支持本地运行，避免将敏感数据上传至第三方平台，从而保护用户隐私。
4. 4. **标准化与模块化**：
   MCP Server 提供了标准化的通信协议，支持两种传输协议（STDIO和SSE），并允许开发者通过插件扩展功能，使其具备灵活性和扩展性。例如，它支持通过HTTP标准POST请求与客户端进行交互，同时支持WebSocket实现实时数据推送。
5. 5. **多场景应用**：
   MCP Server 可以应用于多种场景，包括但不限于：

* • **本地资源集成**：如文件操作、数据库管理、API调用等。
* • **云服务交互**：如与GitHub、Slack、Google Drive等云服务的集成。
* • **AI助手扩展**：如为ChatGPT等AI助手提供上下文支持和工具调用能力

目前mcp-server发展速度非常快。截止2025年4月26日目前mcp-server在mcp.so市场上已经发展超过10000多个mcp-server

image-20250426134845647

目前各大互联网厂商也陆续实现的MCP-Servers广场了。前端时间给大家介绍过关于[《dify案例分享-魔搭+Dify王炸组合!10分钟搭建你的专属 生活小助理》](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486875&idx=1&sn=3cf9516eefea1f3a46df0f9e4ebb2dd2&scene=21#wechat_redirect) 和[dify案例分享-私有化 MCP 广场搭建与网页小游戏智能体工作流实战](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486983&idx=1&sn=220c4008b3433f0908fd2119b9ec2f98&scene=21#wechat_redirect)

今天带大家一起使用cherry-studio和trae实现excel表格一键生成可视化图表html报告的MCP案例，下面先给大家看一下效果

测试的excel表格数据

image-20250505145037365

## cherry-studio

image-20250505144647193

image-20250505144727660

F:\tmp\mcpfiles 有我们的分享报告，我们看一下

image-20250505144839941

image-20250505144907641

看起来生成的html报告还不错，通过简单的excel表格通过几个MCP 实现了一个分析报告。

## trae

image-20250505145256692

image-20250505145500897

同样它在我电脑的F:\tmp\mcpfiles目录下生成一个zz\_report.html 报告

image-20250505145553956

image-20250505150022601

image-20250505150034881

我们通过2个mcp-client实现了excel表格一键生成可视化图表html报告。那么这个mcp-server用到了哪些工具，如何实现的呢？话不多说下面带大家一起来实现。

# 2 mcp-server配置

上面的mcp-server其实用到了4个mcp-server.分别是

sequential-thinking

server-filesystem

excel-mcp-server

quickchart-server

## trae配置mcp-server

他们的配置在trae非常简单，下面是他们的配置文件

### quickchart-server

```
{
  "mcpServers":{
    "quickchart-server":{
      "command":"npx",
      "args":[
        "-y",
        "@gongrzhe/quickchart-mcp-server"
      ]
    }
}
}
```

### excel-mcp-server

```
{
  "mcpServers":{
    "excel-mcp-server":{
      "command":"npx",
      "args":[
        "--yes",
        "@zhiweixu/excel-mcp-server"
      ],
      "env":{
        "LOG_PATH":"F:\\tmp\\mcpfiles",
        "CACHE_MAX_AGE":"1",
        "CACHE_CLEANUP_INTERVAL":"4",
        "LOG_RETENTION_DAYS":"7",
        "LOG_CLEANUP_INTERVAL":"24"
      }
    }
}
}
```

### server-filesystem

```
{
  "mcpServers":{
    "server-filesystem":{
      "command":"npx",
      "args":[
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "F:\\tmp\\mcpfiles"
      ]
    }
}
}
```

### sequential-thinking

```
{
  "mcpServers":{
    "sequential-thinking":{
      "command":"npx",
      "args":[
        "-y",
        "@modelcontextprotocol/server-sequential-thinking"
      ]
    }
}
}
```

首选我们需要确保电脑上安装了Node.js、uvx 等工具，详细安装可以参考trae帮助文档

https://docs.trae.ai/ide/model-context-protocol?\_lang=zh

image-20250505150757834

安装好后，我们在trae配置添加，下面以一个mcp-server介绍一下配置

打开trae,选择mcp (trae建议升级到最新版本)

image-20250505150936693

image-20250505150903896

点击手工配置

image-20250505151012849

在弹出mcp-server配置对话框里面，把我上面的4个mcp-server添加配置

image-20250505151102479

点击确定按钮完成添加配置，稍等片刻，后我们看到变成可用状态，说明trae已经完成了mcp-server安装。

image-20250505151159799

分别将上面4个mcp-server 添加完成即可。

## cherry-studio配置mcp-server

cherry-studio的配置和上面的配置类似。

我们打开cherry-studio,找到mcp-server配置

image-20250505151507054

我们可以在右上角使用编辑mcp配置的方式，也是可以手工添加的方式。

image-20250505151637326

添加MCP配置的方式和上面的trae比较类似，都是json文件格式复制一下即可。它多了一个"type": "stdio", 类型。

另外考虑我电脑上是windows，所以我们command 改成 cmd命令， args 添加 -c 和 npx。这种方式是解决windows 安装npx 不可用的问题。关于这块知识就不做详细展开。

image-20250505151916286

区别的部分我们用红框标注了，其他都和 trae配置一样。

手工添加方式也非常简单，类型选择studio ，命令行 cmd ,参数填写即可

image-20250505152057339

设置完成后，点击右上角保存按钮完成设置。

当cherry-studio MCP-Server 配置都变成绿色就是可以使用状态。

image-20250505152217963

以上步骤就完成了cherry-studio MCP-Server的配置

# 3.验证及测试

下面我们以cherry-studio 为案例测试。

在使用它之前我们先设置一下提示词。

提示词内容如下

```
## 角色定位

你是一位顶尖的数据可视化与分析专家，具备卓越的Excel数据处理能力和敏锐的商业洞察力。你精通使用先进的MCP工具（`sequential-thinking`, `server-filesystem`, `excel`, `quickchart-server`）来解读、处理、分析数据，并能基于数据特征和用户需求，智能推荐并生成高质量的可视化图表。最终，你将以专业、美观、符合Apple设计风格的响应式HTML报告，清晰呈现富有洞察力的分析结果。

## 核心能力

### 1. 数据理解与处理

- **工作流程管理 (`sequential-thinking`)：** 利用`sequential-thinking` MCP服务规划并执行复杂的分析任务，确保分析过程逻辑清晰、步骤严谨。
- **Excel数据访问 (`excel`)：** 使用`excel` MCP服务精确、高效地读取用户提供的Excel文件内容（支持.xlsx, .xlsm, .xltx, .xltm格式），包括工作表名称、单元格数据、公式等。
- **数据预处理：** 识别并专业处理数据结构问题，包括缺失值、异常值、数据类型不一致等，应用适当的清洗和转换技术，确保数据质量满足严谨的分析要求。
- **文件系统交互 (`server-filesystem`)：** 通过`server-filesystem` MCP服务安全地访问和管理本地文件系统中的分析所需文件（根目录限定于 `F:\tmp\mcpfiles`）。

### 2. 智能可视化推荐与生成

- **需求深度分析：** 深入理解用户的显式和隐式目标，结合数据本身的特性，提炼出核心的分析维度和需要关注的关键指标。
- **图表智能推荐：** 基于数据类型（如时间序列、分类、比例、分布、关系等）和分析目的，智能推荐最能有效传达信息的可视化方案。
- **专业图表生成 (`quickchart-server`)：** 利用`quickchart-server` MCP服务，根据选定的数据和图表类型，生成清晰、准确、美观且具有信息传递效率的可视化图表（如柱状图、折线图、饼图、散点图、热力图等）。

### 3. 深度数据分析与洞察提炼

- **探索性数据分析 (EDA)：** 对数据进行全面的探索性分析，运用统计方法和可视化手段识别关键模式、趋势、周期性、相关性以及潜在的异常点。
- **洞察总结与提炼：** 超越表面数据，挖掘其背后的业务含义和深层原因，提炼出具有价值的核心洞见，并以简洁、精准、易于理解的语言进行阐述。
- **报告内容撰写：** 基于分析结果，撰写结构清晰、逻辑严谨的数据分析文字报告，包含关键发现、数据解读、趋势预测（如果适用）和切实可行的建议。

### 4. 精美HTML报告构建与输出

- **内容有机整合：** 将生成的可视化图表与数据分析文字报告无缝集成，确保图文互补，共同服务于分析目标的呈现。
- **Apple风格设计：** 报告视觉设计遵循**Apple设计风格**原则：注重**简洁、清晰、优雅**。使用**卡片式布局**组织内容，确保**充足的留白**；采用**清晰的无衬线字体**（优先使用系统UI字体）、**圆角元素**、**细微阴影**效果和**专业、和谐的色彩搭配**；适当运用**高质量图标**增强信息传达，提升整体专业感和现代感。
- **响应式HTML输出：** 生成单一、完整的HTML文件。确保报告**内容丰富、结构合理、导航清晰、易于阅读**，并在不同设备（桌面、平板、手机）上均具备良好的**响应式**布局和阅读体验。

## 工作流程

1. **需求理解与数据接入：** 接收用户指令和Excel文件路径，使用`excel`工具初步读取数据结构（如工作表名称、列名）。主动向用户确认对分析目标的理解。
2. **数据清洗与准备：** 检查数据质量，进行必要的数据清洗、转换和整理。若发现严重问题，及时向用户反馈。
3. **探索性分析与洞察发掘：** 执行深入的数据分析，识别关键模式、趋势和异常。
4. **可视化方案设计与生成：** 根据分析发现和用户目标，推荐并使用`quickchart-server`生成合适的可视化图表。
5. **分析报告撰写：** 撰写包含核心洞察、图表解读和建议的文字报告。
6. **HTML报告构建与整合：** 使用`filesystem`工具（如果需要创建或写入文件），将文字报告和图表整合成符合Apple设计风格的HTML报告。
7. **结果呈现与迭代：** 向用户展示最终的HTML报告。如有必要，根据用户反馈进行调整和优化，或提出进一步分析的建议。

## 交互原则

- **主动确认：** 在关键步骤（如确定分析目标、执行复杂操作）前，主动总结你的理解并寻求用户确认，避免方向性错误。
- **透明沟通：** 在分析过程中，若遇到数据歧义、需要做出假设或存在多种分析路径，应向用户清晰说明情况，解释你的判断依据或寻求用户指导。
- **用户为中心：** 始终以帮助用户解决问题、达成目标为核心，提供清晰、准确、有价值的分析结果。

## 健壮性与错误处理

- **异常情况应对：** 若遇到无法访问文件、文件格式不支持、数据质量问题阻碍分析、MCP工具执行失败等情况，必须立即停止当前无效尝试，清晰地向用户报告具体问题，并尽可能提供错误信息和建议的解决方案（例如，请求用户检查文件路径、格式或提供更清晰的数据）。
- **工具失败处理：** 若特定MCP工具调用失败，尝试理解失败原因。如果可能，尝试替代方案或告知用户该功能暂时无法完成。

## 数据隐私

- **安全规范：** 在处理用户提供的任何数据时，严格遵守数据隐私和安全规范，仅在完成用户请求的分析任务范围内使用数据，任务完成后不保留用户数据。
```

我们在cherry-studio添加一个智能体名字叫做“数据分析专家”

```
image-20250505152515868

image-20250505152525641

完成智能体添加后，我们在聊天助手中添加一个叫做“数据分析专家”智能体

image-20250505152602600

模型这里我们选择2025年4月29日阿里发布的qwen3-235B-A22B模型，关于这个模型大家可以在摩搭社区里面找，这里就不做详细介绍。

接下来我们用到4个mcp-server 所以在聊天对话框把4个mcp-server都勾选上

image-20250505152924134

我们的提示词

```
请根据“F:\tmp\mcpfiles\zz.xlsx”进行全面分析，并生成一份包含关键洞察和可视化图表的HTML报告
```
```

后面它就开始调用模型提供的函数。

image-20250505153020720

image-20250505153030985

生成总结报告，并把生成的html输出到F:\tmp\mcpfiles\zz\_analysis\_report.html 中

image-20250505153121513

image-20250505153150856

后面的trae测试方法和这个比较类似，这里就不做详细展开了。

相关资料和文档可以看我开源的项目 https://github.com/wwwzhouhui/dify-for-dsl

# 4.总结

今天主要带大家了解并实现了使用 cherry - studio 和 trae 实现 excel 表格一键生成可视化图表 html 报告的 MCP 案例。我们详细讲解了 mcp - server 的配置过程，分别介绍了在 trae 和 cherry - studio 中配置 sequential - thinking、server - filesystem、excel - mcp - server 和 quickchart - server 这 4 个 mcp - server 的方法。同时，还展示了如何设置提示词、添加智能体、选择模型以及勾选所需的 mcp - server 来完成 excel 数据的分析和可视化报告的生成。这个方案属于比较实用且具有一定创新性的方案，能够帮助用户快速、便捷地将 excel 表格数据转化为可视化的 HTML 报告，提升数据分析和展示的效率。感兴趣的小伙伴可以按照本文步骤去尝试。今天的分享就到这里结束了，我们下一篇文章见。

 

[dify案例分享-私有化 MCP 广场搭建与网页小游戏智能体工作流实战](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486983&idx=1&sn=220c4008b3433f0908fd2119b9ec2f98&scene=21#wechat_redirect)

[小白必看！启智平台轻松搞定 Qwen3 模型推理与训练](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486948&idx=1&sn=bfb259e1d1805f0ed7102aef0d51fd37&scene=21#wechat_redirect)

[dify案例分享-Qwen3 vs 传统合同审查，这场对决谁能胜出？](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486914&idx=1&sn=303396d3a1eb2725b5e33417b44052af&scene=21#wechat_redirect)

[dify案例分享-魔搭+Dify王炸组合!10分钟搭建你的专属 生活小助理](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486875&idx=1&sn=3cf9516eefea1f3a46df0f9e4ebb2dd2&scene=21#wechat_redirect)

[dify案例分享-0 代码搭建 Text2SQL 智能查询！用 Dify + 知识库 + Agent 实现自然语言秒变 SQL](https://mp.weixin.qq.com/s?__biz=Mzg3OTYzMjc1NQ==&mid=2247486834&idx=1&sn=8e256d0d0937d9cc1b88522f5fb48273&scene=21#wechat_redirect)