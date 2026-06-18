> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020101_DeerFlow/020101_核心知识点/DeerFlow源码中的中间件与Harness边界|DeerFlow源码中的中间件与Harness边界]]
---
title: DeerFlow2.0源码分析
author: AI2ML人工智能to机器学习
date:
url: https://mp.weixin.qq.com/s?__biz=MzIzMjU1NTg3Ng==&mid=2247492158&idx=1&sn=f3ed063b1323fef03a7e99f52f7a74b9&chksm=e947f7a65a626426198b25471628d59391007048e00861762dd24d967605a7e7ff344588d18f&mpshare=1&scene=24&srcid=0410gSDndQTG9bgUXrRt3bjN&sharer_shareinfo=6354397e6b6990097487a4e12ea8152a&sharer_shareinfo_first=6354397e6b6990097487a4e12ea8152a#rd
---

我们在“[小龙虾OpenClaw四十问](https://mp.weixin.qq.com/s?__biz=MzIzMjU1NTg3Ng==&mid=2247492142&idx=1&sn=041933929f96b88a59c20f526577de66&token=1360191343&lang=zh_CN&scene=21#wechat_redirect)”详细分析了OpenClaw的调度方式， 目前大部分开源项目例如Nanobot和CoPaw在能力上还是距离OpenClaw非常的远。

最近字节发布了DeerFlow2.0，我们来看看， 它是如何实现的？

1. 首先DF2.0不是一个纯Python或者TypeScript(TS)项目。

DF2.0的基本架构

1）前端是typescripts开发的，基于React框架来实现的。

2）后端是Python开发的，基于LangGraph框架的。

3）前后端之间是通过FastAPI REST 来进行接口封装的。

2. 核心能力在deerflow-harness包的实现。

其中Agents 和 SubAgents是分开实现的两个包， 还有一个重要的包是Skills。

3. Skills调度，通过load\_skills函数来完成的。

从Skill的类别就可以看到， 目前DF2.0对Skills的支持还非常初级。 这方面与Nanobot还非常类似。

主要还是通过installer和 loader来进行调度。

主要支持三种Agent协同模式，

1. LangGraph 工作流的方式 。

2. 父子层次化SubAgent方式。

3. ACP协议Agent方式。

4. DF 2.0 基于 **LangGraph** 构建，利用其“有向无环图 (DAG)”与“循环图”的能力来管理 Agent 的状态流转。同时支持，**生成 (Spawning)，**Lead Agent 动态生成具有特定 System Prompt 和工具集的 Sub-Agents（子智能体）。

Agent的核心是AgentMiddleware来支持多种钩子下的插件加载方式。

通过LeadAgent来启动流程：

加载上下文：

启动loop循环。

5. SubAgent动态委托 而非预定义的固定子 Agent，调度完全由 Lead Agent 通过内置工具触发，采用异步线程池 + 限流 + 轮询机制。

SubAgent核心就是任务执行， 要比LeadAgent简单非常多。

小结：

总体来看， Deer-Flow2.0是最像OpenClaw的Python能力实现了，但是依然有差距。 譬如Agent协作方式， Skills能力， 多通道排队等方面都有待进一步加强。 再次感叹OpenClaw的思考维度的厉害（参考[小龙虾OpenClaw四十问](https://mp.weixin.qq.com/s?__biz=MzIzMjU1NTg3Ng==&mid=2247492142&idx=1&sn=041933929f96b88a59c20f526577de66&scene=21#wechat_redirect)）！！！

但是选择Python的LangGraph的工作流框架作为Agent的骨干，非常适合快速开发上手。