> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 一个神级 Claude Code 画图插件，开源了！
author: GitCake
date: 小 G小 G
url: https://mp.weixin.qq.com/s?__biz=MzkyNDE2Njk1NQ==&mid=2247497243&idx=1&sn=e2cabf9b0d53f92246134045222dfeaa&chksm=c024ef7c3a9d85fc8617193e667883dd9fa3753d65678758eb45aa3e218c92c71f331f0b93c2&mpshare=1&scene=24&srcid=0513KDv1tXIOrlKPBwap6kcR&sharer_shareinfo=a4b7f845bd24d0aa03cdcc5d03bc86a0&sharer_shareinfo_first=a4b7f845bd24d0aa03cdcc5d03bc86a0#rd
---

写过技术文档的同学应该都懂，画架构图、流程图这件事，最折磨人的不是构思，而是动手。

脑子里的逻辑明明很清楚，到了 draw.io 或者 Visio 面前，就开始一个框一个框地拖，一根线一根线地连，光是对齐就要花掉半小时。

之前曾介绍过一款名为 next-ai-draw-io 的开源项目，能把 AI 接进 draw.io，相当受欢迎。

而这次更狠。draw.io 官方亲自下场，开源了一款名为 **drawio-mcp** 的项目，让 Claude 这类 AI 工具，可以直接生成原生的 draw.io 图表。

该项目目前已收获 3500+ GitHub Star，基于 Apache-2.0 协议开源，由 draw.io 母公司 JGraph 团队亲自维护。

说点题外话，draw.io 这个产品的来头其实不小。

在 Atlassian 插件市场上，它常年位居绘图工具安装量第一。据官方介绍，99% 的财富 500 强企业都在使用其产品。

换句话说，这次是行业头部主动把自家工具与 MCP 协议进行打通，分量与社区作品完全不在一个层级。

接下来，进入正题，聊聊这个项目最有意思的几个地方。

**第一个亮点**，是图表能直接渲染在聊天窗口里。

drawio-mcp 提供了一种叫 MCP App Server 的模式，基于 MCP Apps 协议实现。

意思是，当我们在 Claude.ai 里用自然语言描述一个架构图，生成的图表会以交互式窗口的形式，直接嵌在对话里。

不用跳转，不用下载，可以缩放、平移、切换图层。

还能一键跳转到 draw.io 网页版进行二次编辑，体验相当丝滑。

**第二个亮点**，是内置了一万多个形状的搜索能力。

这点对画云架构图的同学来说，是个刚需。

drawio-mcp 提供了一个叫 search\_shapes 的工具，可调用 draw.io 的全部官方图标库。

覆盖 AWS、Azure、GCP、Kubernetes，以及 UML、BPMN、电气图、思科网络图等。

调用之后，返回的是可直接用于 XML 的样式字符串。

也就是说，AI 画出来的 AWS Lambda 图标，是真的 Lambda 图标，而不是一个写着「Lambda」的方块。

**第三个亮点**，是提供了四种接入方式，按需选择。

不同开发者用 AI 的姿势不一样，drawio-mcp 把这件事考虑得很周到。

习惯在 Claude.ai 网页端办公的，直接添加一个 MCP 服务器地址，图表嵌在聊天里。

用 Claude Desktop 的，装个 npm 包，图表在浏览器中打开。

用 Claude Code 写代码的，复制一个 Skill 文件到本地，命令行里直接 `/drawio` 一句话出图。

如果不想装任何东西，把官方提供的项目指令贴到 Claude.ai 的 Project 里也能用。

四种方式背后共用同一份 XML 生成规范，保证了出图风格的一致性。

值得一提的是，Claude Code Skill 模式还支持导出 PNG、SVG、PDF 三种格式。

并且导出的文件里，都内嵌了原始的 XML 数据。

也就是说，即便导出成了图片，把文件拖回 draw.io 还能继续编辑，数据不会丢，这点设计相当细致。

### 写在最后

drawio-mcp 这个项目，单看功能不算惊艳，但放到整个行业里看，信号意义远比功能本身要大。

过去半年，MCP 生态最热闹的还是 Notion、Linear、GitHub 这类「云原生」工具。

而真正决定 AI 能否进入企业核心工作流的，是那些用了十几年、沉淀着海量企业资产的传统生产力工具。

draw.io 算是其中一个。

它服务着 99% 的财富 500 强，绑定着无数企业的 Confluence、Jira 文档体系。

当这种量级的工具开始主动接入 MCP，说明 AI 应用层正在从「试用尝鲜」走向「嵌入存量市场」这个更难、也更值钱的阶段。

紧接着可能跟进的，会是 Figma、Notion 之外的更多老牌玩家，比如 Miro、Lucid、甚至 Visio。

谁先把 MCP 这层接口做好，谁就能在下一轮 AI 原生工作流里，拿到入场券。

至于普通开发者，能拿到的实惠很直接，画图这件事，从此可以少花点时间在工具上，多花点时间在思考上。

GitHub 项目地址：https://github.com/jgraph/drawio-mcp

今天的分享到此结束，感谢大家抽空阅读，我们下期再见，Respect！