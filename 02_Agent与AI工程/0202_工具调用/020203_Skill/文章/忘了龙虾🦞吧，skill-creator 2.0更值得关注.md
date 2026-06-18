---
title: 忘了龙虾🦞吧，skill-creator 2.0更值得关注
author: AI咖啡馆
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247487152&idx=1&sn=76cae86ecbf7f4c3213492b8200983dc&chksm=9ad49b5ff98c815f353d5bbf7538adbecb26a42b510134170f76dcbad36bddc15b1984275987&mpshare=1&scene=24&srcid=03269GrQNjsgjMI0P5CfVxqM&sharer_shareinfo=b9b8d0e3e5733bbbc4104ee1c50d2063&sharer_shareinfo_first=b9b8d0e3e5733bbbc4104ee1c50d2063#rd
---

# Skill —>工程：Anthropic推出skill-creator 2.0

Anthropic 在 Agent Skills 的演进路线上又往前推了一步。表面看，这似乎只是一次对 skill-creator 的功能增强，但把它放进完整的 Agent 演化脉络中，你会发现这是一次方向明确的升级：skill-creator 正在从**辅助写提示词的工具**，正式走向**管理 Skill 全生命周期的工程平台**。

**我更愿意把它称为 skill-creator 2.0。**

因为它解决的已经不是“怎么写出一个 Skill”，而是另一个更现实也更棘手的工程问题：写出来之后，你怎么证明它真的有用？怎么知道它没有退化？怎么确认它会在该触发的时候被准确调用？怎么判断一次修改到底是真实的性能提升，还是开发者的一厢情愿？

### Skill 体系最大的痛点

很多 Skill 作者是业务专家或领域专家，他们清楚工作流，也知道什么样的输出才算合格。但他们往往缺乏一套工程化的方法去验证：这个 Skill 在新模型上是否依然有效，修改描述后触发率是升了还是降了，基础模型变强后它还有没有存在的价值。

过去这恰恰是 Agent 体系最大的短板。

Anthropic 这次补上的正是这一整套缺口：评测、基准测试、并行测试、盲测对比以及触发优化。

说得更直白一点，他们正在把 Skill 从依靠经验的提示词资产，往可测试、可比较、可维护的软件工程资产方向推进。这件事非常关键。因为一旦 Skill 进入团队协作流程并参与真实业务，它就不再只是一个简单的文本文件，而是系统的一部分。

既然是系统的一部分，它就必须接受软件工程最基本的拷问：**是否稳定、能否验证、可否复现、升级会不会带来回退、成本是否处于可控范围**。

### 两类 Skill 与衰减方式：过时 vs 失真

Anthropic 官方将 Skill 分为两类，这个划分非常有实操价值。

**能力增强型 Skill**： 它的作用是帮助 Agent 完成基础模型暂时做不好或表现不稳定的任务。比如复杂的文档生成或精确的 PDF 处理。这类 Skill 的本质是把特定的技巧和执行模式固化下来，让结果优于单纯的对话提示。

**偏好编码型 Skill**： 它不一定提升模型的基础能力，而是把团队的特定流程、审查顺序、输出规范和协作习惯编码进去。模型本来就能完成任务，但未必会按你们团队的组织方式去完成。这类 Skill 的价值在于把能做变成按我们的规矩做。

理解这个分类很重要，因为它揭示了两种完全不同的失效风险。**能力增强型 Skill 的风险在于过时**。如果基础模型升级后，不加载 Skill 也能完美处理同样的问题，这个 Skill 就不再有存在的必要。而**偏好编码型 Skill 的风险在于失真**。随着团队流程、约束条件或输入源的变化，Skill 虽然还在运行，但可能已经无法忠实反映真实的工作流了。

因此，引入评测的价值不只是看看效果，而是把一个“感觉上能用”的 Skill，变成一个有数据、有依据证明它能用的组件。

### 核心转变：从“生成”走向“验证闭环”

这次升级最值得重视的核心，不是 skill-creator 更会写文本了，而是它开始帮开发者建立完整的验证闭环。

其中最基础的能力就是评测。现在你可以为一个 Skill 设计测试用例：给定特定的提示词和文件输入，定义什么样的输出才算合格。这个过程完全就是软件测试的逻辑。你不再依赖直觉去判断一个 Skill 的好坏，而是用一组可重复执行的测试来衡量。

结合 Anthropic 给出的 PDF 处理案例来看，过去 Claude 在处理不可填写表单时表现不稳定，因为需要将文字精确映射到固定坐标，而表单本身缺乏定义好的字段。在没有评测机制时，这类问题只能靠零散的用户反馈来修补。有了评测之后，失败的特定场景可以被单独隔离出来，修复前后的效果可以直接进行量化比较。

这释放出了一个明确的信号：Skill 正在脱离“提示词工程”的范畴，全面进入“执行框架工程”的范畴。在 Agent 阶段，真正重要的问题不再只是“怎么说话模型才会听”，而是：

* • 这个 Skill 什么时候触发？
* • 调用后的成功率是多少？
* • 执行耗时多久？
* • 消耗了多少 Token？
* • 新版本到底有没有实质性改进？
* • 模型升级后是否出现了能力回退？
* • 这些本质上都是严谨的系统级问题。
* • 基准测试与盲测：量化评估的分水岭

如果说评测让 Skill 可以被检查，那么基准测试就是让 Skill 可以被量化比较。

Anthropic 新增的基准测试模式会跟踪几个关键指标：**通过率、耗时和 Token 消耗**。这三个指标的结合极其关键，因为 Skill 的价值从来不是单一维度的。通过率虽高但耗时极长、成本巨大的并非好 Skill；运行极快、极省 Token 但输出质量大幅下降的同样不合格。只有将效果、速度和成本放在同一个基准线上衡量，优化才有实际意义。

更进一步，skill-creator 2.0 引入了多 Agent 并行评测和盲测对比。

**前者解决了测试速度和上下文污染的问题**。传统的顺序测试不仅效率低，前一个测试的记忆还可能干扰后一个测试。并行启动独立的 Agent 在干净的上下文中分别执行，结果显然更加可靠。

**盲测对比解决的则是一个更微妙的人性弱点**：开发者很容易高估自己刚刚修改过的版本。很多所谓的优化，常常只是把提示词写得更长、更复杂、显得更专业，但实际效果未必提升。盲测不看作者的主观偏好，只看最终的输出质量。这样才能确认一次代码提交到底有没有带来真实的业务价值。

### 长期被低估的命题：触发机制即路由接口

很多人在开发 Skill 时，全部精力都放在了逻辑内容本身，却忽略了更底层的命题：它是否会在正确的时机被准确调用。

这其实是多 Agent 体系中最容易失控的一环。如果描述写得太宽泛，Skill 会被频繁误触发；写得太窄，又可能在需要时完全不触发。一旦系统内的 Skill 数量增加，这个问题会呈指数级放大，最终导致系统在运行时出现乱触发、抢夺触发权，或者集体沉默。

现在 skill-creator 会主动分析描述文本与样例提示词之间的匹配程度，帮助开发者减少误触发和漏触发。这说明行业正在直面一个现实：Skill 的描述文本根本不是写给人看的文案，而是写给系统的路由规则。

未来，SKILL 配置文件不能再被当成普通的说明书，它是一种半结构化的规范文件。它虽然使用的是自然语言，但承担的却是接近 API 接口定义的职责。一旦自然语言承担了接口职责，任何模糊不清的表达都会直接变成系统 Bug。

### 2.0 版本暴露出的四大意图

透过这些功能更新，我们可以清晰地看到 Anthropic 的战略方向。

**将 Skill 推进为生产级工作流资产**。 过去 Skills 更像是一个生态概念，强调个人的复用和组合。现在强调持续集成、基准测试和盲测，说明目标是让企业团队敢于将其投入生产环境、敢于重度依赖并长期维护。

**开发生命周期的平台化**。 谁掌握了评测框架、基准测试体系和触发优化机制，谁就不只是在提供一个文件格式，而是在定义整个 Agent 领域的质量标准。未来一个 Skill 到底优不优秀，将由平台的验证体系用数据说话。

**基础模型将吞噬技巧型 Skill**。 这意味着 Skill 生态不会无意义地无限膨胀。未来能够长期沉淀下来的核心资产更可能是两类：深度连接外部工具与业务环境的执行型 Skill，以及深度编码组织内部流程与约束的规范型 Skill。

**把开发重心从“怎么做”推向“做什么”**。 底层执行细节会被不断变强的模型压缩。开发者需要投入更多精力的，是目标定义、成功标准、触发边界和失败条件的设定。Skill 的未来不是走向消亡，而是层级上移，从操作说明书升级为可验证的行为规范。

### 结语

如果仅仅把这次更新看作是“skill-creator 变强了”，那未免太浮于表面。Anthropic 实际上正在推动整个 Agent 技能体系走出早期的提示词工程阶段，大步迈入软件工程阶段。

skill-creator 2.0 的真正意义，不在于帮你更快地写出一个文件，而在于它提供了一套工具，帮你回答那个最严肃的问题：这个 Skill 到底凭什么值得被系统和团队信任？

能不能把它们变成真正可信、可测、可维护的资产，这才是 Agent 体系走向成熟的真正标志。

### 参考链接：

* • Improving skill-creator: Test, measure, and refine Agent Skills
* • The Complete Guide to Building Skills for Claude | Anthropic
* • Claude Skills: Build repeatable workflows in Claude | Zapier

 

更多优质信息，请关注！

🔥推荐阅读

#经验&技巧

[Claude Code负责人的干货分享：极简与并行](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247487020&idx=1&sn=c4f1a8075eea6dee6d54c8d1f9bd22d6&scene=21#wechat_redirect)

[如何用好AI大模型：苏格拉底式提问法](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247484418&idx=1&sn=c9a63801c07bc8c8cac23ce84fa608d3&scene=21#wechat_redirect)

[选择大模型的三个最关键因素](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486020&idx=1&sn=9438de82589c6ef09b1b33aaa3d8dede&scene=21#wechat_redirect)

[使用AI的术与道](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485892&idx=1&sn=8d6fefe21c82c3ed4a3d7cbba56f9986&scene=21#wechat_redirect)

#vibecoding

[2026年，如何终结代码评审？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247487143&idx=1&sn=a1938585074cce3d8cb051afd539c7da&scene=21#wechat_redirect)

[Harness Engineering：OpenAI团队的“Agent-first”实践](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247487125&idx=1&sn=7451e3aca8241d83f72a9ba7a4e3dce3&scene=21#wechat_redirect)

[Andrej Karpathy对于AI编程的最新感悟](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247487094&idx=1&sn=a484423d27dbda68ff680e07cfbea669&scene=21#wechat_redirect)

[突然爆火的Ralph是什么？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247487043&idx=1&sn=f85173363970230673c1b776d27fbdcc&scene=21#wechat_redirect)

[SPEC驱动开发真的那么好吗？未必](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247487031&idx=1&sn=9b558a7fc59641b0266f26c4e93faa48&scene=21#wechat_redirect)

[AK的焦虑是Vibe Coding的年度总结](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486995&idx=1&sn=7e71582d71a8cf9c266ab5ff7a790441&scene=21#wechat_redirect)

[一位资深工程师的Vibe之年](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486922&idx=1&sn=591503d1b61a2c941dd3233d443c8e7e&scene=21#wechat_redirect)

[OpenAI如何用Codex在28天内构建Sora Android版？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486906&idx=1&sn=1882a13503116ba9ac13caceaa92139e&scene=21#wechat_redirect)

[什么编程语言更适合Vibe Coding？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485517&idx=1&sn=e1ae86693d56834ab41b505c5ccb4fc7&scene=21#wechat_redirect)

[不要刻舟求剑，关于Vibe Coding的几点感想](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485006&idx=1&sn=19ff3c6bd200d6f94a82f695ae93864b&scene=21#wechat_redirect)

[我所了解的优秀系统设计（译）](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485640&idx=1&sn=566b0fbb36676f0811956c1fd5e1aeae&scene=21#wechat_redirect)

[没有银弹：软件工程的本质与偶然](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486030&idx=1&sn=4a0be1382cd23b7e61e244363c3bc41e&scene=21#wechat_redirect)

[“心智模型”是人类工程师无法被替代的壁垒](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485738&idx=1&sn=286517469b0b2272f7b3bf69747916f6&scene=21#wechat_redirect)

[「可验证性」是AI编程的极限所在](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486119&idx=1&sn=83e6941d88dfe9fb2088e9bfbac7162d&scene=21#wechat_redirect)

[OpenAI Codex 团队关于AI编程的洞见分享](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486093&idx=1&sn=b4c981019388bf787f034bba2e433ab7&scene=21#wechat_redirect)

#Claude

[Claude 4.5 Opus神秘的“灵魂文档”](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486875&idx=1&sn=4d212e3503f9adb25bd9208d5576b796&scene=21#wechat_redirect)

[Claude针对长时运行Agent的高效策略](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486746&idx=1&sn=170841633f714fa17b621781f2aedc8b&scene=21#wechat_redirect)

[用CLAUDE.md文件为代码库定制 Claude Code](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486740&idx=1&sn=e654835126994ca550553edc599611d0&scene=21#wechat_redirect)

[Claude Skills构建指南：步骤、局限性与实战案例](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486736&idx=1&sn=9afb9380cd4cc184e2aadaa01b7e3d47&scene=21#wechat_redirect)

[Claude推出高级工具调用，迈向“智能编排”](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486723&idx=1&sn=e6c9725579e218d393deae1c2ebc27a5&scene=21#wechat_redirect)

[如何使用Claude Code快速提升前端设计？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486705&idx=1&sn=ff01f037f9bfa70d6ec06ccdec6003f4&scene=21#wechat_redirect)

[一文讲透何时使用Claude Skills：与Projects、MCP、subagents的对比](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486656&idx=1&sn=e176ee5e2dc38a0e1238b3f617cfdf23&scene=21#wechat_redirect)

[Claude推出Agent Skills: 上下文工程的优雅实践](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486292&idx=1&sn=71eb78883abf1e2f6da679bc85520094&scene=21#wechat_redirect)

#AI洞见

[a16z关于2026年AI应用的思考](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247487038&idx=1&sn=6cb36265114692f5935bdec546447b1c&scene=21#wechat_redirect)

[Jim Fan关于机器人领域的三个观点](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247487005&idx=1&sn=e2d7017fec4c05e7cebbab2fc1e6c9f1&scene=21#wechat_redirect)

[AI如何影响科技工作者的生产力](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486955&idx=1&sn=fe698036f28058530e1fdcda14e1ba14&scene=21#wechat_redirect)

[谷歌是否后悔发表了Transformer论文？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486864&idx=1&sn=3bbc519d4971313f26d1b0f2cd0d4477&scene=21#wechat_redirect)

[OpenAI与谷歌、英伟达的星球大战，广告模式是必然选项？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486836&idx=1&sn=ab82e4336aca026154490a52b987dcad&scene=21#wechat_redirect)

[OpenAI 将以新模型反击 Google](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486825&idx=1&sn=e5b6200c30db6df12f1657106cccee7d&scene=21#wechat_redirect)

[Ilya最新访谈：从Scaling Law重回“研究时代”](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486731&idx=1&sn=63f9179a59e612b60d97db18166fe0ae&scene=21#wechat_redirect)

[OpenAI内部拉响“红色警报”](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486809&idx=1&sn=fb6cb20bb3594309a3df9ac98c1384a6&scene=21#wechat_redirect)

[AK最新演讲：我们处于软件3.0时代](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485221&idx=1&sn=c3943017566bc03984351d8578a8bd7f&scene=21#wechat_redirect)

[a16z深度好文：AI正开启万亿美元级的新技术栈](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486400&idx=1&sn=0d3b356e9071d2d1ff9c7da39f3e4136&scene=21#wechat_redirect)

[对AI的技术乐观与恰当恐惧](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486249&idx=1&sn=b5caaff802e90227f902daec633a050f&scene=21#wechat_redirect)

[AI：我们时代的信息塑料](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486177&idx=1&sn=daeada73bfdf3be7f41539c29c3a9692&scene=21#wechat_redirect)

[“AI-Ready”是AI提效和转型的前提](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485178&idx=1&sn=53e69c655d4c587211e69a774ac2c4fd&scene=21#wechat_redirect)

[AI原生时代，GUI转向“文本优先”的技术必然性](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485057&idx=1&sn=0ab49a9bd4f3ffbd7843acd4953a8636&scene=21#wechat_redirect)

[核心员工对OpenAI的思考](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485733&idx=1&sn=7bfe1cd2497c63b7c33866abf3564672&scene=21#wechat_redirect)

[OpenAI关于人机关系的思考与实践](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485050&idx=1&sn=39268c00e593081927fe3ddbc7f39be8&scene=21#wechat_redirect)

[AI的下半场：从解决问题到定义问题](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247484717&idx=1&sn=cadcd05afb486c704963dc4ee95001c6&scene=21#wechat_redirect)

[谷歌：欢迎来到AI的经验时代](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247484692&idx=1&sn=3a05eec5ea6c9af31c3f4b2db0ea7a4a&scene=21#wechat_redirect)

#上下文工程

[Cursor的动态上下文发现机制](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247487049&idx=1&sn=ce42018608f0f8ce47262bf2e40c9a22&scene=21#wechat_redirect)

[化解MCP上下文瓶颈：工具调用 -&gt; 即时代码执行](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486544&idx=1&sn=bcdb0e8142b27cc0741bec31324e5b91&scene=21#wechat_redirect)

[Cline是如何思考上下文工程的？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485777&idx=1&sn=89b303e2f527c01c3e3a3c879c903a1a&scene=21#wechat_redirect)

[Manus的上下文工程：从构建AI Agent中学到的教训](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485469&idx=1&sn=7086d6be440f9274a5dd804d78f637af&scene=21#wechat_redirect)

[没有prompt，一切都是上下文？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247484523&idx=1&sn=7497623fb31fc384ba76ec36f3c1b314&scene=21#wechat_redirect)

#模型&Agent

[OpenAI：坦白机制如何使AI保持诚实](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486865&idx=1&sn=38ccd731bafface63c588412e25fdb76&scene=21#wechat_redirect)

[KiloCode：深度评测GPT-5.1、Gemini 3.0、Opus 4.5](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486791&idx=1&sn=d2923daef31f8e9f1b63ce780791233b&scene=21#wechat_redirect)

[近距离观察：打入Cursor内部的60天](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486564&idx=1&sn=5600505bc773cb8f4688538f280b7330&scene=21#wechat_redirect)

[StackOverflow 2025调查报告关键词：务实](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486717&idx=1&sn=44d0a3203c912836963e8f84e6928692&scene=21#wechat_redirect)

[30多位Founder眼中的Agentic AI落地现状](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486533&idx=1&sn=42b7753501f79d4fbabfa73a42b4e413&scene=21#wechat_redirect)

[创企Every的六位工程师的 AI 工作流程](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486481&idx=1&sn=85ca67791563343c01e9b2bd9b270015&scene=21#wechat_redirect)

[规范驱动开发(SDD)的哲学：代码服务于规范](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486449&idx=1&sn=01c63d079c75771a0552ee6888e7ed99&scene=21#wechat_redirect)

[AI编码悖论：既令人震撼，也同样愚蠢](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486427&idx=1&sn=b60f547fb8de8998b67ce7da0bf86e1e&scene=21#wechat_redirect)

[只有5%的AI Agent生产可用？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486244&idx=1&sn=7823d2c3ed1f0eb8816766b26a36b3cc&scene=21#wechat_redirect)

[Agents 2.0：从浅循环到深度 Agent](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486232&idx=1&sn=c045306d0d17735bf5f3c1bf0a6b4ee1&scene=21#wechat_redirect)

[Workflow vs Agent 构建器，LangChain创始人的思考](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486213&idx=1&sn=e3be1a4eb2e251f9bcfc93f4b302a556&scene=21#wechat_redirect)

#产品&思维

[数据模型：产品的基因与格局](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486340&idx=1&sn=aff6602919cef44d29e9ca2e87431025&scene=21#wechat_redirect)

[RL之父萨顿：苦涩的教训](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486406&idx=1&sn=4e1996d28f5d1caaa8cea390ee7ff6ff&scene=21#wechat_redirect)

[YC创始人：做那些无法规模化的事](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485632&idx=1&sn=d42e44f00603841d543a06deceb5f8fc&scene=21#wechat_redirect)

[写作即思考，YC创始人对“好文笔”的思考](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247484970&idx=1&sn=aa7e6f1238632ee4b1b6d74ce0091232&scene=21#wechat_redirect)

[以「可读性」框架理解大型软件公司的怪现象](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485953&idx=1&sn=8162d34b772da077d25904e9b8e18c7d&scene=21#wechat_redirect)

[AI入侵文化，人类的想象力将归于何处？](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247485793&idx=1&sn=5ce171a02353775ee24d76681305e782&scene=21#wechat_redirect)

[验证的不对称性与验证器法则](https://mp.weixin.qq.com/s?__biz=MzAxNzk3NzM2Ng==&mid=2247486125&idx=1&sn=268e856820a1e2292a20c493171e39b1&scene=21#wechat_redirect)

#skills #agent #agentskills #skill  #harness #skill-creator