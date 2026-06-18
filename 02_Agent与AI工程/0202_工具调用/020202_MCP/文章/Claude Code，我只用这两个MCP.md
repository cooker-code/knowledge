---
title: Claude Code，我只用这两个MCP
author: AI编程实验室
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5ODY5MDU4Mw==&mid=2247483665&idx=1&sn=e9dc848c277df19fdeff016b6faa487b&chksm=977f9feed19f3bbff78b91e276f7a13155672823bb8379a3ec3abdd72359ba5e3cd3377fff29&mpshare=1&scene=24&srcid=1023CdmQ30b82DrZpd5GTIRa&sharer_shareinfo=245fa0db42b258044e8bc2efd7639d2b&sharer_shareinfo_first=245fa0db42b258044e8bc2efd7639d2b#rd
---

大家好，我是鲁工。

节前开了这个新号之后，还没正式开始更新内容。

先定个小目标，保持连续日更30天。

经常玩Agent的朋友都知道，通过调用外部工具能够有效缓解大模型的幻觉问题并拓展大模型的能力边界。最开始的时候叫函数调用（Function Calling），或者工具调用（Tool Use）。

但后来工具太多，大模型接入没有一个统一的规范和协议，所以又有了模型上下文协议（MCP）。本质上是将外部工具以上下文的形式提供给大模型。

那么，我们在使用Claude Code等AI编程工具时，是不是MCP安装的越多越好？当然不是。一般大模型都存在有限的上下文窗口，比如Claude Sonnet 4.5的200K上下文窗口、Kimi K2的256K上下文窗口等。安装太多的MCP通常会挤占项目上下文空间，降低上下文的使用效率。还有就是MCP工具多了，AI在决策时选择哪个工具也会有成本。

所以，理想的MCP配置应遵循按需精选策略：针对具体项目类型和开发阶段，只保留核心必需的工具，确保每个MCP都有明确的使用场景。另外也可以定期对MCP进行盘点，清理不再使用的MCP。本质上也是一个有限资源约束下（大模型有限的上下文窗口）的优化问题（最优的AI响应与执行效率）。

为了保证Claude Code的任务执行效率，我在使用Claude Code时，目前只配置了两个通用的MCP工具。咱们分开说。

第一个是Chrome DevTools MCP，也就是Chrome开发者工具MCP，Chrome DevTools MCP能让Claude Code控制和检查实时运行的Chrome浏览器。这个MCP之前我在主号机器学习实验室也推荐过：

[推荐一款前端项目Vibe Coding必备的MCP！让AI随时能看到网页开发效果](https://mp.weixin.qq.com/s?__biz=MzI4ODY2NjYzMQ==&mid=2247501730&idx=1&sn=2af95940bd06e99ab265a276984b0e8d&scene=21#wechat_redirect)

简单来说，有了这款MCP之后，你就再也不用去在前端页面中截图拖拽到命令行，也不用去复制粘贴控制台的报错，Chrome DevTools提供的一切信息随时作为上下文喂给大模型，相当于给Claude Code加了一双随时监控网页效果的眼睛。这简直就是前端开发类项目的神兵利器。

Chrome DevTools MCP项目地址：

https://github.com/ChromeDevTools/chrome-devtools-mcp

第二个是context7。经常写代码的朋友都知道，对于编程语言、软件和开源项目等来说，文档和教程非常重要。或许你曾有如下经历：AI写的代码，经常报某个函数、某个类的某个属性不存在等问题，其实一般就是版本问题，是你当前用的库版本与大模型给你写的代码的库版本不一致造成的。

而Context7就是一个专门为大模型和AI编程工具提供最新代码文档的MCP服务器工具。其核心价值在于解决AI编程助手的三大痛点：训练数据过时导致的代码示例陈旧、API幻觉问题，以及缺乏版本特定的库文档支持。

Claude Code中Context7安装方法如下：

```
claude mcp add --transport http context7 https://mcp.context7.com/mcp --header "CONTEXT7_API_KEY: YOUR_API_KEY"
```

需要去Context7官网申请一下api key：

https://context7.com/dashboard

在使用Claude Code进行开发时，我们可以在提示词中添加"use context7"指令，系统会自动从源头拉取实时文档并注入到LLM上下文中。也可以直接将使用context7作为规则添加到CLAUDE.md文档中，比如：

> Always use context7 when I need code generation, setup or configuration steps, or library/API documentation. This means you should automatically use the Context7 MCP tools to resolve library id and get library docs without me having to explicitly ask.

Context7项目地址：

https://github.com/upstash/context7

所以，如果你也在用Claude Code深度Vibe Coding，欢迎体验上述两个MCP。相信会给你带来不一样的开发体验。

感谢您阅读我的文章。我是鲁工，八年AI算法老兵，AI全栈开发者。欢迎关注我，深耕AI编程赛道。感兴趣的朋友也可以加我微信（louwill\_）交个朋友。

>/ 作者：鲁工