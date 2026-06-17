---
title: 字节面试官：什么是 Multi-Agent？
author: 小林面试笔记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247484018&idx=1&sn=3ab9e4bae930b64947edd1db404f68e3&chksm=f20cf15c79b24a1bd4d72d085c3731b4b86e4422b6afcc25776f01d2c438b3b5705026e03c6c&mpshare=1&scene=24&srcid=0416Vu9nsuXWzkb0nvpLL62S&sharer_shareinfo=12e9439b24a7dd19f91a6237bc77443e&sharer_shareinfo_first=12e9439b24a7dd19f91a6237bc77443e#rd
---

👔面试官：（指尖敲了敲桌面，笑着抛题）同学，咱们聊点落地的，说说什么是Multi-Agent？别光背定义，讲点人话。 

🧑‍💻我：（松了口气，顺势玩梗）收到！这题我熟～说白了，Multi-Agent就是AI界的“团队作战”，再也不让一个AI打工人累死累活包揽全活了！

👔面试官：（挑眉打趣）哦？单个AI还能累死？那多智能体到底强在哪？

🧑‍💻我：那可太有说法了！单个Agent就是个全能打杂的，又搜资料又写内容，脑子（context窗口）分分钟炸掉，干啥都不精通；Multi-Agent直接搞分工协作，专人专事，效率和专业度直接拉满，我这就给您细细拆解～

> 这道经典字节面试题，核心就是拆解多智能体的协作逻辑，下面结合场景讲透Multi-Agent的本质、优势和设计思路。

## 💡 简要回答

多智能体系统（Multi-Agent）就是多个 Agent 协作完成任务，每个 Agent 各有分工，有的负责搜索、有的负责写代码、有的负责做评审。

我理解单个 Agent 主要受两个限制：一是 context 窗口大小，复杂任务信息量一多就撑爆了；二是单点能力，什么都让一个 Agent 做，每件事都是泛才。

Multi-Agent 通过专业分工和并行执行，能处理更复杂、更长流程的任务，这是我在实际项目里选择多智能体方案的核心原因。

## 📝 详细解析

想象这样一个场景：你让 Agent 帮你完成「写一份完整的 AI 行业竞品分析报告」。

它需要搜索十几家竞品、读懂每家的产品功能、梳理核心差异、整理对比数据、最后写结论……光是搜索下来，每家竞品几百字，十家就是几千字的搜索结果，再加上来回确认的对话历史和中间推理，还没开始写结论，整个工作台就已经快撑满了。

这里说的「工作台」，就是 LLM 的 context window。LLM 处理任务的方式，是把它当前能看到的所有内容，包括你的指令、它自己的推理过程、工具返回的搜索结果、历史对话记录，全部摆在这张工作台上，一起处理。

这个工作台是有大小上限的，常见的模型限制是 10 万到 20 万个 token，塞满了之后，早期的内容就会开始「掉落」，就像一张桌子放满了东西，新的东西要放进来，旧的就得推到地上。于是，你三十分钟前确认的方案、搜集的第一批资料，就这么悄悄消失了，Agent 开始「遗忘」。

context 有上限，这是第一个硬限制。但更深的问题其实是「专业度」的问题。让一个 Agent 既搜信息、又写代码、又做测试、又写文档，它在每一件事上都得兼顾，精力是分散的，就像一个人同时担任产品经理、程序员、测试工程师和文档工程师，每个角色都做得不够专注，互相干扰。而且一旦某个环节出问题，整条链路就卡住了，没有隔离性，排查起来也很痛苦。

### Multi-Agent 核心思路

Multi-Agent 的核心思路，就是「团队作战代替单打独斗」。

与其让一个 Agent 包揽所有事，不如把任务按职能拆开，每个 Agent 只负责一件事，专心做好自己那块，做完把结果传给下一个。

就像公司里的部门协作：产品经理负责需求梳理、开发负责写代码、测试负责验收，每个人专注自己的职责，信息传递清晰，哪个环节出了问题也好定位责任。Multi-Agent 系统就是把这套分工思想搬到 AI 里。

还是以「开发一个爬虫工具」为例，来感受一下两种做法的差距。

不用 Multi-Agent 的情况：一个 Agent 接到任务，同时在想需求文档、代码结构、测试策略，context 里塞满了各种信息，思路乱成一锅粥，写出来的东西哪块都不够好，而且任何一步失误都得从头来。

用了 Multi-Agent 的情况：

* 第一个 Agent 是「需求分析师」，它只做一件事，把用户需求转化成清晰的功能列表，输出之后就完成使命，退出了，它的工作台是干净的；
* 第二个 Agent 是「程序员」，拿到功能列表，专注写代码，不需要知道需求是怎么来的，context 里只有代码相关的信息；
* 第三个 Agent 是「测试工程师」，拿到代码，专注写测试用例……每个 Agent 的工作台都很干净，只有自己这块任务相关的内容，专业度也更高。

更关键的是，需求分析这步结束之后，程序员 Agent 和测试 Agent 其实可以并行工作，测试框架的搭建不需要等代码写完，两件事同时进行，整体速度也快了。

Multi-Agent 系统的组织方式主要有两种：一种是中心化，由一个统一的调度者来分配任务、收集结果；另一种是去中心化，Agent 之间自行协商、直接通信。这两种方案各有取舍，下一题会展开讲。

推荐阅读：

[淘天面试官：你知道 Agent 的记忆机制吗？](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247484016&idx=1&sn=f7259c5c56ab87cdea2e1fd43f132240&scene=21#wechat_redirect)

[蚂蚁一面：“复杂任务怎么做拆解？”，我：“大模型上下文都 200 万了，直接全塞进去硬算啊”，面试官：“回去等通知吧。”](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483931&idx=1&sn=32eaad0357bd0de5482cee788f940a64&scene=21#wechat_redirect)

[字节面试官：“Plan-and-Execute 比 ReAct 强在哪？”，我：“名字比较长，显得更高级？”，面试官：“门在那边，走好。”](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483916&idx=1&sn=bf4830ee91d928a47a3e7124bb5bf174&scene=21#wechat_redirect)

[快手面试官连环炮：“除了 ReAct，你还知道哪些 Agent 推理范式？”，还在死磕 Prompt 的我愣住了...](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483902&idx=1&sn=0cf5ac123f30b3da3108a988811dff5a&scene=21#wechat_redirect)

[阿里一面怒怼：“连 Tools、Workflow 和 Agent 三者区别都搞不清，也敢来面试？”，我当场尬住...](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483901&idx=1&sn=5825bd451088770c732845c6342d3cc1&scene=21#wechat_redirect)

[腾讯三面面试官刚想拿“Agent和Workflow 的区别”难倒我，我反手甩出一张架构对比图，他当场让我等 HR 面。](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483898&idx=1&sn=90101573e94272a1f4ceb6f3b5596758&scene=21#wechat_redirect)

[小红书二面秒挂！问“Agent 核心组件有哪些”，我答“大模型+Prompt”，面试官冷笑...](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483788&idx=1&sn=45c1ec9d5a578a689d27179f06833b14&scene=21#wechat_redirect)

[美团面试官：“既然大模型已经这么强了，为什么还要做 Agent？”，我给出了满分回答，他眼睛亮了。](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483777&idx=1&sn=0fee73150fd3c92d666391a2e0797260&scene=21#wechat_redirect)

[字节二面：被问“大模型知识过时了怎么解？”，我答“微调”，面试官当场黑脸：“听说过 RAG 吗？”](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483658&idx=1&sn=02673313e69b2a6a304ad7462e4e4a29&scene=21#wechat_redirect)

[腾讯面试官怒怼：“连 MCP 协议都没玩过，还敢在简历上写精通 Agent 开发？”，只会调包的我懵了...](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483673&idx=1&sn=0010e78aece8b05388b94d0396444a03&scene=21#wechat_redirect)

[阿里技术官拷问：“简历上写着精通 Agent 开发，那你说明白 A2A 和 MCP 到底啥区别？”，我：“呃... 不都是协议吗？”](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483728&idx=1&sn=8feb775f055f697517d0a3ad424a8c7b&scene=21#wechat_redirect)

[腾讯面试官一句追问："做了这么多Agent项目，大模型怎么学会调用外部工具都说不清？" 我当场卡壳，二面凉了...](https://mp.weixin.qq.com/s?__biz=MzY4NTE2NjU5MQ==&mid=2247483763&idx=1&sn=b3f2abee33ad0cf37282f8814b0960a5&scene=21#wechat_redirect)