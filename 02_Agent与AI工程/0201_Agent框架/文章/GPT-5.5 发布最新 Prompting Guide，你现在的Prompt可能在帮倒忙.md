---
title: GPT-5.5 发布最新 Prompting Guide，你现在的Prompt可能在帮倒忙
author: 阿丸AI笔记
date: 阿丸笔记阿丸笔记
url: https://mp.weixin.qq.com/s?__biz=MzA4NjQ3ODQ2Mw==&mid=2247486763&idx=1&sn=f7e3cfc2f663223712af979792baa1f6&chksm=9eec4ba9dc7a7988e4a580eff964dc85723722efbaf716da99f3236c07aa3f4bb99e6c131f61&xtrack=1&req_id=1778290004528584&scene=90&subscene=93&sessionid=1778290008&flutter_pos=0&clicktime=1778290012996&enterid=1778290012996&finder_biz_enter_id=4&ranksessionid=1778290004&key=daf9bdc5abc4e8d0633000b7e7fd72f220396f43778609613af69dfe801e2bd4a0226fc50716419239161b77a68df4c3e08f0a3aac1e63ad558b2b84736544f6f1b6a8eee2dd82408222ad18a4757d4687232375f7ccc922422c203d080545479ffe6c0216be4dd4bad65b921821c92d05717898cae59fd2464f6037ccfaaaa3&ascene=0&uin=MjAwNTI2NjEyMw%3D%3D&devicetype=OHOS-22&version=f3801037&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&countrycode=CN&exportkey=n_ChQIAhIQt1PJs0R3WDTiC1sdqVVv8hLfAQIE97dBBAEAAAAAANy6LenONREAAAAOpnltbLcz9gKNyK89dVj0w5KG93om%2Fp5sjFlatplsZQgdpE73qfLG10grKIxoK%2Ftw6QVF%2BnLhr5uhqKqwf%2BVNSYPSFlEUq1S0BpFgFoRfJObq5jKhcP%2F6lxVnObCP3F%2BK8tyU736zFVVeYflZpXXsL32CoaU1L%2FWkyAlQAXKhMcszosCTU0UnzDe078K2IOVsEtS0xnDhYpHhiiEJUU5piQkOvDhxAekmLvkqQhfJEgYvOUOGwx2aKRfp3hYnbP1jMwC7RR7kc4o%3D&pass_ticket=hsDLPgFFkeHwX7wEl%2FpsH9X0MCafWJgS3LdkS%2BY375pxH10YBScwqhBC06eTUGbK&wx_header=3
---

你有没有发现，花在Prompt上的时间越来越长，但用新模型的效果有时候反而不如预期？

我最近读了OpenAI随GPT-5.5一起发布的Prompting Guide，读完觉得有点意思。

这份Guide的核心逻辑，其实是在告诉你：你以前积累的那些Prompt经验，有一部分正在帮倒忙。

这件事值得认真说一说。

Guide说了什么

先把核心内容摆出来。

GPT-5.5的官方指南开宗明义：不要把旧模型的Prompt直接迁移过来，要从最小提示词重新开始。

原文里有这么一句话："旧提示词通常会过度规定流程，因为早期模型需要更多帮助才能保持正轨。对于GPT-5.5来说，这样做会增加噪音，缩小模型的搜索空间，或者导致过于机械的回答。"

这是第一个核心点：你给它的限制越多，它发挥得越差。

Guide推崇的是"outcome-first"提示方式，即告诉模型你想要什么结果、成功的判断标准是什么、有哪些约束，然后让它自己选择路径。而不是把每一步骤都写清楚：先做A，再做B，然后对比C，最后输出D。

GPT-5.5 works best when prompts define the outcome and leave room for the model to choose an efficient solution path.

其他几个值得注意的点：

一是**明确停止条件**。指南里建议加上这样的规则："每次工具调用后，问自己：我现在是否已经能用足够的证据回答用户的核心问题？如果是，立即给出答案。" 这是为了防止模型过度思考、无效循环。

二是**preamble机制**，即在多步骤任务的任何工具调用之前，先发送一条简短的用户可见更新，说明你在做什么。这个设计是为了解决一个具体问题：用户看到模型在思考，但屏幕上没有任何输出，以为它卡死了。

三是新增的text.verbosity和reasoning\_effort两个参数，用来精细控制输出详细程度和推理深度。

为什么现在发这份Guide

这是我觉得最值得想的问题。

按理说，模型越来越聪明，Prompt应该越来越省事才对。但现实是，每次大版本跳跃，厂商都要出一份新的Prompting Guide。GPT-4有，Claude 3有，GPT-5.5也有。这到底是为什么？

我觉得有三个原因：

**第一，能力跳跃产生了兼容性断裂。**GPT-5.5的推理能力相比早期模型有本质提升。它能自主规划路径、主动选择工具、识别何时停下来。这意味着它和用户的协作关系变了。以前你需要手把手，现在你如果还手把手，它反而觉得被框死了，回答变得机械。Guide实际上是在帮你重新校准"你和AI的协作模式"。

**第二，用户积累了大量"prompt技术债"。**过去两三年，很多团队都在Prompt上投入了不小的精力：精心设计的system prompt、经验总结出来的few-shot示例、层层叠加的规则和限制。这些东西在旧模型上有效，在新模型上可能反而是负担。OpenAI需要一个官方渠道告诉大家：是时候清账了。

**第三，Agent应用场景对提示词提出了全新要求。**GPT-5.5明显是面向Agent工作负载设计的。多步骤任务、工具调用、长时间运行、中间状态管理，这些场景下的提示词设计和聊天对话的逻辑完全不同。Guide里关于stopping conditions、preamble、phase参数的内容，全都是在解决Agent场景下的实际痛点。

这份Guide有多大普适性

这是我想多说几句的地方，因为这里有一些值得存疑的地方。

**有些东西是真正普适的。**定义成功标准、给出约束而不是步骤、为模型留出自主空间——这些原则其实在任何足够强的模型上都成立。从某种角度说，这是"如何与聪明协作者沟通"的通用方法，只是以前的模型还不够聪明，所以这套方法用不上。

**但也有些东西是高度模型特定的。**比如reasoning\_effort这个参数，比如phase值的处理，比如Codex的$openai-docs migrate命令——这些都是OpenAI生态的专属能力，换成Claude或者其他模型就完全对不上。

还有一个我觉得容易被忽视的问题：这份Guide的前提是你在用一个能力足够强的模型。它告诉你"用outcome-first，别规定流程"，但如果你用的是一个能力一般的模型，这套方法可能并不适用。能力弱的模型确实需要更明确的步骤引导，否则就会跑偏。

所以这里有个容易踩的坑：把这份针对顶级模型的Guide当成通用真理，然后发现在其他场景下不管用，就怀疑自己哪里做错了。

另外，"从最小提示词重新开始"这条建议，听起来很合理，但实际执行成本不低。

如果你有一个跑在生产环境里的、被精心调优过的system prompt，真的要扔掉重来吗？大部分团队的现实答案是：不会，也没法这么做。

Guide给出的是理想状态建议，但工程实践里你还是要做渐进式调整。

我真正觉得重要的那条线索

读完这份Guide，我觉得最值得关注的不是某一条具体技巧，而是它背后折射出的一个更大趋势。

Prompt engineering正在从"用魔法词汇控制模型"变成"定义目标、给出约束、建立协作框架"。

前者像编程，后者更像管理，或者说像和一个聪明但需要你给方向的同事沟通。

这对技术人员意味着什么？Prompt Engineering的门槛其实在降低，但质量上限在提高。

任何人都可以写一个outcome-first的简短提示词；但真正能定义清楚成功标准、设计合理的stopping conditions、处理好边界情况的人，需要对自己想要什么有非常清晰的认知。这是软实力，不是靠记住几条规则就能做到的。

有意思的是，Guide里那个"Personality block"的设计。定义助手的语气、温度、何时提问、何时主动，这不是在写提示词，更像是在写岗位JD。我们正在越来越像"AI产品经理"，而不是"AI驯兽师"。

OpenAI发这份Guide，表面上是帮你用好GPT-5.5，但它也在悄悄告诉你：下一个时代的AI使用，是关于如何定义工作边界，而不是如何塞满指令。

如果你也在用GPT-5.5或者做Agent开发，欢迎来聊，加我微信一起研究。觉得这篇有用，点个**在看**，让更多人看到。

如果你对Agent记忆系统感兴趣，持续更新合集：[Agent记忆系统](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA4NjQ3ODQ2Mw==&action=getalbum&album_id=4434376184776310785#wechat_redirect)

**点击上方卡片关注阿丸公众号**

**交流AI商业化、产品、技术、培训的朋友，可以加我个人微信沟通**