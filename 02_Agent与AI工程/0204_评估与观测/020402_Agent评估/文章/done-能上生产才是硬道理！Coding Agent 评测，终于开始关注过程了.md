> 已吸收至：[[02_Agent与AI工程/0204_评估与观测/020402_Agent评估/020402_核心知识点/Agent评估体系与观测边界|Agent评估体系与观测边界]]
---
title: 能上生产才是硬道理！Coding Agent 评测，终于开始关注过程了
author: DataFunTalk
date:
url: https://mp.weixin.qq.com/s?__biz=MzU1NTMyOTI4Mw==&mid=2247774332&idx=1&sn=a6d83da3ee289c9edaf90ef762137a9a&chksm=fa32f4ff76b631b1bc57dd1b3134164d02b0e63af42d89283401f552ec495825ddfe5cdd663a&mpshare=1&scene=24&srcid=0121LZsgo28gC9VRtMZntAMu&sharer_shareinfo=dccd8dfa4f9e19e17a258af417c42dff&sharer_shareinfo_first=dccd8dfa4f9e19e17a258af417c42dff#rd
---

今天是一期硬核的话题讨论：

**Coding Agent 评测。**

AI 编程能力进步飞速，在国外御三家和国产中厂四杰的努力下，AI 编程基准 SWE-bench 的分数从年初的 30% 硬生生拉到了年底的 70%+。

2025 年用 AI 写代码成了日常，我在 X 上看到有开发者说：“我发布的有些代码自己从未读过”。

这恐怕就是现在 Vibe Coding 的常态。AI 写代码，AI 跑测试，人类就负责点确认。

但是，AI 写的代码测试都通过了，就真的没有问题了吗？

如果你关注模型榜单，就会知道 AI 编程的主流评测基准大致有：

* SWE-bench 系列（最流行）
* HumanEval / MBPP / LiveCodeBench 系列
* 其他 Benchmark（AgentBench / Aider / Terminal-Bench）

这些主流的 Coding Agent Benchmark 的核心指标是：Pass@k（k 次尝试中通过测试的比例）。

只要最终 patch 通过测试，无论过程如何都算成功。

但是，**Coding Agent 的过程对了吗？**

比如：

* 有没有改不该改的文件？
* 有没有违反开发规范？
* 是不是用低效的方式完成？

这些过程中的“违规操作”，SWE-bench 都看不见。

更关键的是，真实的 Coding agent 需要同时处理：

* System Prompt 全局指令
* 用户查询 的具体需求
* Memory 中的历史上下文
* Tool Schemas 的工具规范
* 配置文件（如 .cursorrules、claude.md、skill.md）的额外约束

这是一个优先级排序和冲突解决的复杂博弈，但传统 benchmark 对此完全失明， 只关注「能不能解决问题」，不关注「在多重约束下能不能正确解决问题」。

而这，恰恰是 Agentic AI 时代的核心需求。

我们能造出完成任务的 Agent，但不知道它们是怎么完成的，不知道它们在什么情况下会失败。

Sourcegraph 研究员 Stephanie Jarmak 说出了一个真相—

> 我们构建 Agent 的能力，已经远远甩开了我们评估 Agent 的能力。

昨天，我看到前不久上市的 MiniMax 开源了一个新基准—**OctoCodingBench。**

> 传送门：
>
> huggingface.co/datasets/MiniMaxAI/OctoCodingBench

我觉得这个事儿还挺有意义的。

一是做评测基准这种又苦又累不讨好的事儿，企业很少会做；二是瞄准了行业盲区，可以把 SWE-bench 看不见的“编程过程违规”，揪出来并且量化成指标。

它首次引入了—**过程评估。**不只关注任务的解决率，还会关注 Agent 在解决过程中是否遵循指令和规则。

它的核心思路是不再把 Coding Agent 当成「会做题的模型」，而是当成「要上生产的队友」来考核。

它不像传统题库那样做“填空题”，而是直接拉起 Docker 容器，进行全链路的仿真测试：

**1. 环境仿真，注入真实的项目约束：**

每个测试用例都会创建一个模拟的代码仓库，里面包含:

* **Repo Specific Rules：**类似 CLAUDE.md 的项目级规范文件，定义了哪些文件不能动、哪些操作是危险的、团队的命名规范是什么。
* **Skills & Tools：**预定义的工具和技能清单，要求 Agent 必须按规范调用指定的工具，而不是随意发挥。
* **系统约束：**System Prompt 中的全局指令，模拟真实 AI 产品的使用环境。

**2.压力测试：多重指令冲突与记忆干扰**

这是 OctoCodingBench 最创新的部分——它会主动给 Agent 挖坑。

* **Memory 模块**注入跨会话的记忆干扰：植入过时的项目规范、插入矛盾的历史指令。
* **Conflict 模块**制造指令优先级冲突：System Prompt 说“永远不要删除配置文件”，User Query 说“清理所有 .bak 文件”，CLAUDE.md 说“删除前必须先备份”。

OctoCodingBench 测试智能体对 7 种不同指令来源的合规性——

这些指令同时出现，Agent 能正确处理吗？

**3. 轨迹收集与 LLM-as-Judge**

传统 benchmark 只看最终 diff，OctoCodingBench 收集完整的交互轨迹：

* Agent 调用了哪些工具
* 读取了哪些文件
* 修改的顺序是什么
* 是否遵循了每一条约束

然后用 LLM 作为裁判，逐条检查是否有违规操作。

目前包含 72 个精心设计的测试用例，覆盖 Python、Java、C++ 等多语言。每个用例都针对一个特定的「过程遵循」场景，类别分布如下：

OctoCodingBench 设计了两个互补的指标:

1. **Check-level Success Rate（CSR）— 过程规范性**

检查 Agent 在执行过程中是否遵循了所有约束条件：

* 文件修改权限：是否改了不该改的文件？
* 工具使用规范：是否用了指定的工具？
* 执行顺序要求：是否按规定先备份再删除？
* 编码规范遵循：是否符合团队的命名规范？

2. **Instance-level Success Rate（ISR）— 综合成功率**

任务是否最终正确完成**且无违规**。这里有个关键设计：「单违规即失败」机制。

即使最终结果正确，只要过程中违反任何一个约束，整个任务就判定为失败。听起来很严格，但这才是企业级开发的真实要求啊！

**01**

**测试结果**

MiniMax 公布的测试结果，给当下的 Coding Agent 泼了一盆冷水，也给了一点惊喜。

发现 1：过程合格率不足 1/3

几乎所有模型的 CSR 指数都能达到 80% 以上，说明模型大概懂规则。但 ISR 出现了断崖式下跌。

即便是地表最强的 Claude 4.5 Opus，任务通过率也只有 36.2%。

这意味着，在接近 2/3 的任务中，即使是目前的绝对强者，也会在某个细微的规范上“违规。

###

发现 2: 国产模型的追赶速度惊人

看榜单细节，开源和国产模型正在快速逼近闭源巨头。MiniMax M2.1 和 DeepSeek V3.2 的 ISR 分别达到了 26.1% 和 26%。

这个成绩不仅咬得很紧，甚至在部分指标上超过了公认强大的 Claude 4.5 Sonnet (22.8%) 和 Gemini 3 Pro (22.9%)。

这也印证了一个趋势：在 Coding 这种强逻辑场景下，国产模型已经具备了极强的竞争力。

发现 3：多轮交互后就会智商掉线

下图展示了是 ISR 随对话轮数的变化趋势，横轴是对话轮次，纵轴是解决率。

可以看到，随着对话轮次的增加，绝大多数模型开始震荡或下滑，指令遵循能力逐渐下降。“过程合规”在长流程任务中非常脆弱，模型聊着聊着就忘了规则。

所以，长时程的复杂任务的**过程监督**非常有必要，AI 写代码也需要 code review 。

而且，这些中间过程的反馈信号，对模型训练至关重要，可以在 RLHF 阶段给予更精准的奖励信号。

**02**

**结语**

很多家人们可能会问：我们真的要用起来这样的基准测试吗？

我的答案是：不仅需要，而且早该有了。

关注我的老朋友都知道，我一直对“评估”这件事有执念——AI 写的代码怎么评？

坦白讲，在 AI 写代码已经成为日常的 2026 年，建立系统化的评估意识，不是锦上添花，而是**保命刚需**。

MiniMax 这次开源的这个 Bench，说明他们在做深 coding 生产力场景。未来的模型必须具备更细腻的颗粒度：不仅要会写代码，更要懂规则。

Andrej Karpathy 说 AI 编程工具就像“**没有说明书的外星技术”**——能用，很强。

OctoCodingBench 的出现，某种意义上就是在为这个"外星技术"编写说明书。

在 AI 写代码已经成为日常的今天，下一个阶段一定是过程监督和精细对齐，才是「AI 进入生产环境」的第一门槛。

**毕竟，能上生产的 AI，才是真正有用的 AI**。

往期推荐

[Claude实现「永久记忆」！官方还没上线，大神已玩疯](https://mp.weixin.qq.com/s?__biz=MzU1NTMyOTI4Mw==&mid=2247774319&idx=1&sn=2714eb46202aed6be991a110b8a4aaa4&scene=21#wechat_redirect)

[如何让 BI 和 AI 用上同一份“好”数据？这份白皮书给你答案](https://mp.weixin.qq.com/s?__biz=MzU1NTMyOTI4Mw==&mid=2247774310&idx=1&sn=53f7bdc27ea229aeb489283159865b6c&scene=21#wechat_redirect)

[智能体架构与实践：构建下一代推荐与搜索系统](https://mp.weixin.qq.com/s?__biz=MzU1NTMyOTI4Mw==&mid=2247774306&idx=3&sn=c5217b57fdbacd4db5c56578527cf88c&scene=21#wechat_redirect)

[启略科技「AI 智能供应链优化方案」荣膺 2025 年度科技创新突破「星空奖」](https://mp.weixin.qq.com/s?__biz=MzU1NTMyOTI4Mw==&mid=2247774250&idx=1&sn=2866ab6938a6f54f75c7d578df7f7c2c&scene=21#wechat_redirect)

[为什么说Ontology才是工业智能的"操作系统"？一个从救火到预防的真实案例](https://mp.weixin.qq.com/s?__biz=MzU1NTMyOTI4Mw==&mid=2247774243&idx=3&sn=23021fba51a555f5ad11ae83b9540a9d&scene=21#wechat_redirect)

[「松下信息系统（上海）有限公司」荣膺 2025 年度技术应用最佳实践「星空奖」](https://mp.weixin.qq.com/s?__biz=MzU1NTMyOTI4Mw==&mid=2247774231&idx=1&sn=3981f744b055f1092bb7512cc067e14c&scene=21#wechat_redirect)

[独家对话吴恩达：解密下一代Agentic AI战略思维](https://mp.weixin.qq.com/s?__biz=MzU1NTMyOTI4Mw==&mid=2247774230&idx=1&sn=4179fc65961552812c31f7253dab7da8&scene=21#wechat_redirect)

[黄仁勋获得《时代》2025年度人物专访：AI将重塑工作，中美深度依赖，不应脱钩](https://mp.weixin.qq.com/s?__biz=MzU1NTMyOTI4Mw==&mid=2247774216&idx=1&sn=130d52ed2c9a944c928491573ac8fb8c&scene=21#wechat_redirect)

[为什么说Ontology才是工业智能的"操作系统"？一个从救火到预防的真实案例](https://mp.weixin.qq.com/s?__biz=MzU1NTMyOTI4Mw==&mid=2247774216&idx=3&sn=5b519c9f3570d2ddf02432c913ee5220&scene=21#wechat_redirect)

[Agent规模化落地的终极方法论，一次讲透！](https://mp.weixin.qq.com/s?__biz=MzU1NTMyOTI4Mw==&mid=2247774209&idx=1&sn=1cd5e357ff3d6c0bcea5b923c252a37a&scene=21#wechat_redirect)

点个在看你最好看

SPRING HAS ARRIVED