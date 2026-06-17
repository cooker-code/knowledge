---
title: 别再死盯Grafana面板了！n8n+MCP工具组合，智能分析告警一步到位
author: 老甄说运维
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkxNDczMzQ1Mw==&mid=2247483967&idx=1&sn=8561089257f637095b204e170b229c24&chksm=c04c805bc088f4aa771873da5814a5278dd11ae64efe9575cf8edb1ada33f68c93ac207d0d3a&mpshare=1&scene=24&srcid=1115FLgX1ALaVyUFedlAUxLX&sharer_shareinfo=aae44408865465b953fbc14e0ade6e38&sharer_shareinfo_first=aae44408865465b953fbc14e0ade6e38#rd
---

**AUTUMN**

**背景说明**

很抱歉这篇文章等了两周才出来，希望让大家等的有价值，我也是最近陆续调试了3周才有比较满意的成果，来给大家分享下。

这篇文章主要讲解n8n的一个mcp工具-grafana-mcp-analyzer,用途主要是分析grafana面板数据，生成监控巡检报告，先看下效果：

当你面对一个面板很多的dashbord来查看指标是否异常的时候，是不是会很繁琐，就像下面这样的：

而有了mcp工具后，巡检报告是这样的(由于篇幅问题，只列出两项)：

这样就很直观看到每个集群，每个监控指标的健康状态，通过AI分析汇总，一目了然，不用再人工每天盯着屏幕分析数据变化趋势了，是不是很赞👍

### 

**AUTUMN**

**mcp工具介绍**

Grafana MCP Analyzer 基于 MCP (Model Context Protocol) 协议，赋能Claude、ChatGPT等AI助手具备以下超能力：

①自然语言查询 - 轻松访问监控数据，AI 一键输出专业分析

②多轮对话支持 - 支持复杂的多轮对话分析，能够基于上下文进行深入分析

③curl支持 - 直接使用浏览器 copy 的 curl 合成查询

④全数据源支持 - Prometheus、MySQL、ES 等通通支持

⑤专业 DevOps 建议 - 不只是展示数据，更提供可执行的优化方案，提升DevOps效率

💡 架构新模式：会话缓存 → 逐步获取数据 → 渐进式深入分析 → 缓存复用，让AI分析更准确、更高效。

安装很简单，只需要一条命令就搞定：

```
npm install -g grafana-mcp-analyzer#环境要求：Node.js 18+
```

涉及的方法名称及作用如下：

**AUTUMN**

**使用场景**

有两种使用方式，一是结合cursor等AI工具集成mcp配置，这样就是对话式应用场景，需要分析哪个面板，就在chat窗口输入自然语言描述即可，AI会自动识别并调用mcp工具进行分析和输出。二是结合AI工作流工具如N8N做自动化智能数据分析，这是重头戏，目前网上没有这么结合的案例，都是自己慢慢摸索调试出来的，感兴趣的伙伴可以尝试并提供宝贵的优化建议。

**AUTUMN**

**如何配置**

㈠准备条件:

①n8n 服务

②下载grafana-mcp-analyzer工具，结合n8n使用

③grafana并配置好的dashborad

㈡配置工作流

①n8n 启用mcp工具功能

②n8n 集成grafana-mcp-analyzer mcp工具

```
以docker部署n8n为例#进入dockerdocker exec -it  --user=root n8n  /bin/sh#使用npm安装grafana mcp工具npm install -g grafana-mcp-analyzer
```

配置mcp认证

选择MCP Client (STDIO) accountMCP Client (STDIO) API

Command一栏，填写grafana-mcp-analyzer

Env一栏填写CONFIG\_PATH=/home/node/grafana-config.js

这样就能正确调用该工具了。

③配置grafana-mcp-analyzer配置文件--核心要点，grafana所有面板的查询名称和查询语句都在配置文件定义，配置参考：

```
const config = {  // 健康检查配置  healthCheck: {    url: 'api/health'  },  // 查询定义  queries: {    '集群coredns请求量': {      url: 'http://x.x.x.x/api/ds/query',      method: 'POST',      headers: {        'accept': 'application/json, text/plain, */*',        'content-type': 'application/json',        'x-datasource-uid': 'xxxxx',        'x-grafana-org-id': '1',        'x-panel-id': 'xx',        'x-plugin-id': 'xxxxx',        "Authorization": "Bearer xxxxx      },      data: {        "queries": [{"datasource": { "type": "prometheus","uid": "xxxx"},"editorMode": "code","intervalFactor": 20,"expr": "sum(rate(coredns_dns_requests_total{job=\"coredns\"}[1m])) by (instance,cluster)","refId": "A","requestId": "5A","datasourceId": 4,}],"range": {"raw": {"from": "now-1d/d","to": "now-1d/d"}        },"from": "1761321600000","to": "1761407999999"},      systemPrompt: `您是资深运维专家，专注于 Kubernetes 云原生环境的稳定性保障与性能优化。请分析 Kubernetes 集群ncoredns请求量情况，重点关注：1.coredns请求量的变化趋势，给出合理的预防建议`    },    .........  }};
```

里面带x的地方都需要替换的，其中如下四个值获取方式，打开浏览器调试工具如下图所示，复制粘贴即可

```
'x-datasource-uid': 'xxxxx','x-grafana-org-id': '1',        'x-panel-id': 'xx',        'x-plugin-id': 'xxxxx',
```

④配置n8n工作流

整体思路：

获取时间--通过command节点修改配置文件时间（非必须，有些数据源不认{"from": "now-1d/d","to": "now-1d/d"}，只能写具体的时间戳）--配置面板名称--首次调用grafana-mcp工具的分析analyze\_query方法获取时间点数与数值的的列表（这里设置每20分钟1个点位，间隔不要太短，否则数据量太多，超过AI模型的上下文长度）---代码处理获取requestid---配置AI Agent（配置大模型和mcp工具）第二次调用mcp工具，但这里方法名是chunk\_workflow，它会根据前面传过来的requestid获取缓存数据，并自动分片处理，直到获取到最后一个分片数据才会结束，最终AI根据mcp返回的数据做分析，根据提示词内容输出最终maridown表格格式--数据存储到临时表中，这里就把每个面板分析数据都存到临时表中，最后再统一查询数据表，将数据结果合并到一个字段交给AI Agent，根据提示词内容输出最终的报告。

配置文件搞定后，接下来配置工作流(非正式，调试环境)

主要讲解下关键节点的配置：

首先是查询的面板名称节点

根据面板名称第一次调用grafana-mcp工具，来根据配置文件的内容获取查询语句、数据源以及时间范围

处理查询响应数据，获取requestid（主要给grafana-mcp工具的方法是使用）

AIAgent节点关于mcp工具配置

每次使用mcp的工具方法名称要选对，否则数据处理会失败。

**AUTUMN**

**结尾彩蛋**

经过多轮调试，最终输出结果的提示词如下

```
- Role: 资深运维专家和系统性能分析师- Background: 用户作为运维人员，需要通过Grafana Dashboard的数据来判断当前系统的指标是否异常。这涉及到对系统性能数据的解读、历史数据的对比以及潜在问题的识别。- Profile: 你是一位在IT运维领域拥有多年经验的资深专家，对系统性能监控和数据分析有着深入的理解。你熟悉各种监控工具，尤其是Grafana，能够快速识别数据中的异常模式，并提供准确的诊断。- Skills: 你具备强大的数据分析能力、系统性能评估能力、故障排查能力和问题解决能力。能够熟练使用Grafana Dashboard进行数据可视化分析，并结合历史数据和阈值判断指标是否异常。当mcp工具返回“工作流完成”字样时停止调用mcp工具，将mcp返回的数据进行分析- Goals: 通过分析Grafana Dashboard上的数据，快速判断当前系统指标是否异常，并提供详细的分析报告。- Constrains: 分析应基于实际数据，避免主观臆断。分析结果需清晰明了，便于运维人员理解和采取行动。- OutputFormat: 分析报告应包括关键指标的当前值、历史对比、阈值判断以及异常原因的初步分析。- Workflow:  1. 收集Grafana Dashboard上的关键指标数据，包括CPU使用率、内存使用率、磁盘I/O、网络流量等。  2. 对比历史数据，分析当前指标是否超出正常范围或阈值。  3. 如果发现异常，结合系统日志和相关配置，初步分析异常原因。  4. 只输出markdown格式内容，其他内容不要输出，不要多余的描述- Examples:  # k8s集群grafana监控巡检报告  # 巡检时间段-2025年x月x日(00:00:00-07:00:00)  | 巡检项 | `A-dc1-prod`集群 | `B-hec-prod`集群 | `C-qc-prod`集群 || --- | --- | --- | --- || 集群node节点cpu使用率 | 76.009% (xxx-k8n120) | **异常**：多个节点持续 >95%，如 `10.2.26.246`, `10.2.26.65` 等接近或达到100%。 | 68.03% - 92.02% (`10.2.32.76` 节点从91.77%下降至68.03%)，波动较大。 |
```

**晴天☀️**

复盘总结 

技术共进

AI运维实践

[<上一篇·丝滑解决硬件监控，categraf完美替换zabbix方案大揭秘](https://mp.weixin.qq.com/s?__biz=MzkxNDczMzQ1Mw==&mid=2247483961&idx=1&sn=f821ca3422cd23ec9373bc4493b6bbb3&scene=21#wechat_redirect)