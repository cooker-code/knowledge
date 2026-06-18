> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: Airflow 把 AI Agent 正式接进 DAG。
author: 大数据技能圈
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247493833&idx=1&sn=a08b20682cc0dfbb7a85d92939cf706e&chksm=c13357cc63991ff2af037a53e5aec5c2ca25d83dce150d690b6903e84cb4bc736829697171db&mpshare=1&scene=24&srcid=0416Oh7S3OQRqwKiIWZVjdy2&sharer_shareinfo=f8768b5fd90aeac4623150f249282b0b&sharer_shareinfo_first=f8768b5fd90aeac4623150f249282b0b#rd
---

前两天，跟一个做数据平台的朋友聊天。

他提了一句特别实在的话，我到现在都记得。

他说，现在大家都在讲 AI 落地，听起来好像什么都能做了。可一旦真要把东西接进生产，问题马上就变了。不是模型够不够强，也不是 Prompt 写得漂不漂亮，而是它出了结果以后，谁来审，错了怎么办，失败了怎么续，花出去的钱怎么算，最后又怎么接回原本那条老老实实跑了很多年的数据链路。

说白了，很多 AI 系统最难的，从来都不是“生成”，而是“管理”。

所以我看到 Apache Airflow 在 2026 年 4 月 14 日正式发布 `common-ai provider` 的时候，第一反应不是“又一个项目接了大模型”，而是：**终于有人开始在编排层认真处理 AI 的工程问题了。**

这件事为什么值得写？

因为它切中的是系统层。

过去这一年，我们看了太多 AI 基础设施的叙事。模型一层比一层强，工具调用一层比一层花，Agent 框架一个比一个像未来。但真到了企业里，最后能不能上线，往往不是拼谁最会讲概念，而是拼谁能把审批、重试、日志、责任链、成本控制这些东西讲明白、做扎实。

而 Airflow 这次做的，恰恰就是这件事。

从官方文档来看，这个新 provider 叫 `apache-airflow-providers-common-ai`，当前版本是 `0.1.0`，最低要求 Airflow `3.0.0`。官方给它的定义也很克制：`AI/LLM hooks and operators for Airflow pipelines using pydantic-ai`。

这句话看着不炸裂，但其实挺有味道的。

因为它已经把边界说清楚了。

Airflow 并没有想再造一个通用 AI 平台，也没有试图重新发明一套 agent runtime。它选择站在 `pydantic-ai` 之上，把自己的精力放回最擅长的地方：hooks、operators、任务上下文、重试语义、DAG 编排、日志和治理。

这很像一个成熟基础设施项目会做的选择。

不抢戏。

但抓关键。

官方博客里展示的能力，核心有三类：`@task.llm`、`@task.agent` 和 `@task.llm_file_analysis`。

先说 `@task.llm`。

它表面上看是最普通的模型调用能力，但关键点不在“能返回一段文本”，而在“可以返回结构化对象，再通过 `XCom` 往下游传”。这个细节很工程。因为真正让 AI 流程变得难维护的，很多时候不是第一步把模型调起来，而是第二步开始，后面的任务到底怎么稳定消费前面的结果。

如果输出永远是一坨需要你自己再 parse 的文本，那系统大概率还是停留在 demo 阶段。

但如果输出开始是有类型、有结构、可传递、可被下游任务直接消费的对象，味道就完全不一样了。它开始像一个正式的数据流程节点，而不是一个临时拼上去的调用片段。

再往下看，真正更有意思的是 `@task.agent`。

因为这一层，才是真正会把工程系统搞得很拧巴的地方。

官方博客给出的描述是，多步 agent 可以在任务里调工具、查数据库、访问 API、读文件，甚至还展示了 SQL toolset 和任务日志。乍一看，这像是在说“Airflow 也能跑 agent 了”。

但我觉得这还不是重点。

重点是，**Airflow 开始把 agent 当成一个需要被调度、被记录、被治理、被恢复的任务实体，而不是一个单独漂在系统外面的聪明黑盒。**

这句话很重要。

因为过去很多团队做 agent，做着做着就会变味。最开始只是写个脚本，后来脚本包成服务，再后来服务变成一个没人敢动的黑盒。你问它运行链路是什么，说不清。你问它失败后怎么续，说不好。你问它的输出怎么审、谁负责、怎么回滚，基本更难回答。

Agent 当然可以很聪明。

但如果它不能被系统接住，它就永远只是一个很会表演的模块。

而 Airflow 这一步，至少是在认真尝试把这部分重新接回任务系统。

Airflow agent SQL analysis logs

`@task.llm_file_analysis` 也是我很喜欢的一块。

因为它不花哨，但特别接地气。

官方明确写到，它可以分析对象存储里的 CSV、Parquet、Avro、JSON，甚至图像这类多模态文件。你仔细想一想，这其实比很多“会聊天的 AI”更接近数据团队的真实工作流。

数据团队日常高频遇到的场景，往往不是和模型闲聊，而是分析文件、理解样本、解释报表、看异常、做快速归纳。真正有用的 AI，很多时候就是嵌在这些很具体、很琐碎、但高频重复的动作里。

所以我会觉得，这个能力看起来没那么耀眼，反而更像能活下来的那种能力。

不是拿来秀的。

是拿来反复用的。

Airflow file analysis for CSV with LLM

如果说前面这些能力回答的是“AI 任务能怎么跑”，那这篇博客里真正让我觉得有工程分量的，其实是另外两块：`Human-in-the-Loop` 和 `Durable Execution`。

先说 `Human-in-the-Loop`。

官方博客写得很直接，不是每一个 LLM 输出都应该直接进入生产，所以 provider 支持人工监督。尤其是 approval gate 这类机制，任务生成结果之后可以先暂停，等人来确认，再决定是不是继续放行到下游。

这听起来好像很朴素。

但它其实非常关键。

因为很多 AI 系统迟迟上不了正式流程，不是因为生成效果不行，而是因为责任说不清。谁批准的？谁兜底的？谁来决定这次结果能不能往后走？如果这些问题没有系统入口，那所谓“AI 落地”通常也只能停在一层展示价值。

而一旦审批这件事开始被做成编排系统里的原生动作，事情就不一样了。

它意味着这个系统不再只是“会调模型”，而是开始具备被纳入业务流程的资格。

Airflow HITL approval interface

再说 `Durable Execution`。

这一块，我觉得反而是整篇博客里最值钱的部分之一。

为什么？

因为它谈的是钱，也是恢复。

官方给了一个很典型的例子：一个 10 步 agent，如果在第 8 步失败，重试不应该把前面所有步骤和模型调用再来一遍。于是 provider 提供 `durable=True`，把模型响应和工具结果缓存到 `ObjectStorage`，失败重试的时候直接回放已经完成的步骤。

这看起来像一个参数。

但工程上，这不是小事。

今天很多人讲 agent，最爱讲它会规划、会调用、会迭代、会自治。可真到了线上，最让人头疼的往往不是它会不会想，而是它一旦跑长链路以后，失败成本到底谁来承担。

你让一个 agent 在数据库、对象存储、外部 API 和模型之间来回跑十来步，本来就已经够贵了。如果失败一次就全量重来，那它越聪明，反而越危险。

所以我会觉得，Airflow 这次最务实的地方，是它没有只讲“AI 能把事情做成什么样”，而是认真回答了另一个更现实的问题：**如果它没做成，系统怎么接住它。**

这才是生产级问题。

Airflow durable execution summary

Airflow durable retry resumes from cached steps

还有一个很重要的点，是这次发布并不是临时起意。

从官方 commits 页面可以看到，`common-ai` provider 在 2026 年 4 月 2 日被推进到 ready state，4 月 8 日进入 provider release prepare，4 月 11 日又修复了 HITL review plugin crash，接着 4 月 14 日官方博客正式发出来。

这条时间线很说明问题。

它不是那种“先写一篇博客占个坑，再慢慢补实现”的打法，而是先把东西准备、打包、修补，再正式往外发。

这会让人对它的可信度高很多。

那这件事对 AI 工程和整个数据栈意味着什么？

我觉得最实际的影响，有三层。

第一层，是 AI 工作流开始重新回到企业已经熟悉的编排平面。

很多团队现在的系统都挺割裂的。传统 ETL、数仓刷新、资产更新，规规矩矩跑在调度器里；可一到了 AI 任务，就开始飘，飘进脚本、飘进服务、飘进某个谁也说不清边界的内部平台里。

短期看，好像灵活。

长期看，特别难维护。

Airflow 这一步最大的价值，就是把“数据准备 -> 模型分析 -> 人工审批 -> 下游消费”重新串成一条可追踪、可审计、可恢复的 DAG。

第二层，是编排层在 AI 时代重新变重要了。

过去一年，太多注意力都放在模型、向量数据库、推理框架和 Agent 框架上。那当然也重要，但真要决定一个系统能不能进生产，经常不是看模型排行榜，而是看审批、观测、恢复、预算和责任链这些东西能不能闭环。

Airflow 这次没有去争“最强 AI 平台”的位置，反而让我觉得它更聪明。

因为它挑的是一个不热闹、但极接近真实生产现场的问题。

第三层，是它给数据工程师和平台工程师开了一个很现实的入口。

并不是每个团队都需要一整套宏大的 AI 平台叙事。

但很多团队都需要，在现有流程里塞进一些真正有用的能力：结构化抽取、SQL 代理分析、文件理解、人工复核、异常解释。

`common-ai provider` 提供的，恰恰就是这一层的入口。

当然，边界也要说清楚。

第一，当前版本仍然是 `0.1.0`。这说明它是正式起跑，不是成熟收官。真到了大规模生产环境，稳定性、接口演进、兼容性，都还要继续观察。

第二，博客里提到的资产集成、lineage tracking、`AIBudget` 和多 agent 编排，更像是明确方向，而不是今天就已经完整交付的现成功能。这部分不能写成既成事实。

第三，它依旧是编排层，不是低延迟在线 agent 平台。它更适合那些带审批、带恢复、带上下文、带责任链条的工作流，而不是所有实时交互型场景。

所以，如果把今天这条进展压缩成一句话，我的判断是：

**Airflow 发布 common-ai provider，真正值得关注的，不是“又接了一个 LLM”，而是数据编排层终于开始系统性接管 AI 任务的工程问题。**

这件事不算最热闹。

甚至有点“土”。

但很多真正能进生产的东西，往往就是这样。

不是先赢在故事里。

而是先活在系统里。

 

以上，既然看到这里了，如果觉得不错，随手点个赞、在看、转发三连吧，如果想第一时间收到推送，也可以给我个星标⭐～谢谢你看我的文章，我们，下次再见。

## 📚 往期精彩推荐

* [应届生10分钟解决问题，5年老员工却束手无策。](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247493801&idx=1&sn=e794eb266d5c58abd62c570ecc15ec49&scene=21#wechat_redirect)
* [5年老员工不会的Spark调优，都在这50道面试题里。](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247493731&idx=1&sn=f1de4d0db19f8e8f9da75eba2cea9441&scene=21#wechat_redirect)
* [50道Hive面试题 + 50张架构图，帮你拿下大厂offer。](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247493664&idx=1&sn=97663f5c22e5584793d0f57ae3834dd9&scene=21#wechat_redirect)
* [这50张高清架构图，把HBase讲明白了。](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247493604&idx=1&sn=8a9c15b214888230a163b2f7c8a6ec4a&scene=21#wechat_redirect)
* [我花三个月死磕Fluss，终于画出了这50张架构图。](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247493536&idx=1&sn=0f73c674ea6fe49630dc31bdc5973efc&scene=21#wechat_redirect)
* [花了3天画的50张Kafka架构图,让你30分钟看懂全部原理。。](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247493481&idx=1&sn=3afcca68e789e78ed119053623b6300c&scene=21#wechat_redirect)
* [Iceberg 最狠的50道面试题，90%的人栽在第7题！！](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247493412&idx=1&sn=a729ba08541bb93f57b4be8958d6bcef&scene=21#wechat_redirect)
* [大厂必考！Doris技术全景图：从架构到实战的50个关键问题！！](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247493319&idx=1&sn=7ff58e6c3570013b8aa51c5dd9b44729&scene=21#wechat_redirect)
* [100张Paimon架构图,让你从面试炮灰到Offer收割机！！](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247493233&idx=1&sn=c3e18ad4ee3f4cb10884b9578ecfc6cd&scene=21#wechat_redirect)