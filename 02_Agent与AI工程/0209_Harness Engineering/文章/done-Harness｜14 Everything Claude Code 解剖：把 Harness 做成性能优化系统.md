> 已吸收至：[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/知识沉淀与私域资产|知识沉淀与私域资产]]、[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/评测观测与质量治理|评测观测与质量治理]]、[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/长任务状态与恢复闭环|长任务状态与恢复闭环]]
---
title: Harness｜14 Everything Claude Code 解剖：把 Harness 做成性能优化系统
author: 与AI同行之路
date: 别别人别别人
url: https://mp.weixin.qq.com/s?__biz=MzkzODk3MTIwMQ==&mid=2247489355&idx=1&sn=748fb6061f1b7849fca5fe0e041c2574&chksm=c3281519935a690c3b47e90680180c19caa74e84a188bdc83d7d38b1a96c431cd7630fe6f3af&mpshare=1&scene=24&srcid=0519337KTK6O5jj9qPv1NKja&sharer_shareinfo=bee81f4f543fd1c5d1edd942d12e79c3&sharer_shareinfo_first=2945c8d556b267d666446551d3cc9efe#rd
---

第八个项目，案例解剖板块的压轴——affaan-m做的Everything Claude Code，简称ECC。

先放一组数字让你感受这个项目的体量——**166K star、21K fork、170+ contributor、1465 commits**。2025年9月Anthropic x Forum Ventures Hackathon冠军作品。这量级在GitHub的agent类项目里基本是现役第一档。

ECC在README首屏立的命题跟前面所有项目都不同——**"the agent harness performance optimization system"**——把harness本身当成一个性能优化系统来工程化。

这个视角在harness这一年的演化里挺特别。Claude Code追求"产品级稳定"、DeepAgents追求"opinionated batteries-included"、Superpowers追求"工程方法论强制执行"、gstack追求"组织角色化"——ECC追求的是**让harness跑得又快又便宜又准**。

**这是把harness从"能跑"推到"跑好"的尝试**。

—— 性能优化层面的可量化数据

ECC给了几条非常实用的优化建议，每条都有量化数据支撑——

**model从opus换sonnet，省约60%**。这条最直接。Opus确实更强，但Sonnet对绝大多数日常任务足够，成本天差地别。除了核心架构设计、复杂重构这种关键任务，日常用Sonnet就行。

**MAX\_THINKING\_TOKENS从31999降到10000，省约70%**。Claude的extended thinking很强，但默认配置过于慷慨。大多数任务thinking在几千token之内就够，30K的预算大量浪费。降到10K是一个甜蜜点。

**CLAUDE\_AUTOCOMPACT\_PCT\_OVERRIDE从95调到50**。这条我特别同意——上一篇文章讲过，等到95%才auto-compact常常已经晚了。50%就开始有意识地compact，整体context健康度好得多。

**CLAUDE\_CODE\_SUBAGENT\_MODEL换haiku**。subagent干的多是"扫文件、跑测试、整理结果"这种活，不需要Sonnet。换Haiku速度快、便宜、效果也够。

ECC 四条性能优化建议

这四条加起来，一个团队按ECC的建议调一遍配置，月度API账单省一半起。**对中国团队特别值得算这笔账——人民币付美元API费用，汇率本身就是1:7、加上某些中转服务费，能省一半就是真金白银**。

ECC还有两条经验法则——**MCP超过10个就开始吃context窗口、tool超过80个开始迷糊判断**。这两个数字记下来，不要超。

—— Continuous Learning v2加Instinct系统

ECC最有想象力的一块设计是Instinct系统——**带confidence scoring的渐进式skill沉淀**。

机制大概这样——

agent在干活的过程中产生大量小pattern——某种文件结构对应某种处理方式、某种error对应某种solution、某种环境对应某种配置。这些都不够沉淀成完整skill，但又是有价值的"小经验"。ECC把它们叫做instinct——直觉。

每个instinct有自己的confidence score——用得越多、效果越好、confidence越高。低confidence的instinct谨慎使用，高confidence的可以放心信任。

**Instinct的命令体系——`/instinct-status`查看、`/instinct-import`和`/instinct-export`在团队间共享、`/evolve`把相关的instincts聚合成正式skill、`/prune`清掉过期的（默认30天TTL）**。

这套设计比Hermes的"自动生成skill"高了一个层次——Hermes是直接从tool call抽skill，没有confidence的概念，质量参差。ECC的instinct机制把"未充分验证的小经验"和"已确认有价值的skill"分开两层管理——**新经验先做instinct慢慢沉淀，confidence到了再升级成skill**。

Continuous Learning + Instinct 三层渐进沉淀

这本质上是软件工程里"实验性feature和稳定feature分开"的思路在agent skill上的应用。**质量门槛清楚，渐进沉淀，整个pipeline比"agent乱写skill"靠谱得多**。

中国团队做企业内部知识沉淀特别可以参考这套——很多企业知识沉淀失败，是因为一开始就要求"标准化、文档化、可审计"，门槛太高没人做。Instinct这种"先粗略沉淀、慢慢演化"的渐进式方案，跟人类组织里"先有口口相传、再有内部维基、再有正式标准"的演化路径是一样的。

—— DRY Adapter Pattern跨harness工程化

ECC支持九个harness——Claude Code、Codex、Cursor、OpenCode、Gemini CLI、Antigravity、Kiro、Trae、CodeBuddy。仓库里有九个独立配置目录。

但这九个目录的配置不是各写各的——**ECC用了一个DRY adapter pattern**。

举个具体例子。Cursor有20+个hook event，Claude Code只有8个。如果你为每个harness单独写一套hook脚本，维护噩梦。ECC的做法是——

底层脚本只写一份，放在`scripts/hooks/*.js`。每个harness写一层薄adapter——比如`.cursor/hooks/adapter.js`，把Cursor的stdin JSON格式转成统一的"内部格式"，然后调底层脚本。底层脚本不需要知道是哪个harness在调它。

**这套架构的工程价值——一份核心逻辑跨九个harness运行**。改一次底层脚本，九个harness同时受益。新加一个harness支持，只需要写一个adapter，工作量从"重写一套"降到"翻译一层"。

DRY Adapter Pattern — 一份核心逻辑跨九个 harness

这个pattern我自己在多端集成项目里反复用过——Web、iOS、Android、小程序四端共享后端逻辑，每端只写thin adapter。**工程世界里"DRY"是大词，但真做到的项目不多**。ECC在harness这件事上做到了，是个人开发者派最值得抄的工程范本。

—— Hooks——把工程纪律物化进生命周期

ECC的hooks.json值得单独拎一段。第七篇讲Claude Code的hooks时我说过"hooks是把'团队规矩'物化成'系统行为'的最干净方式"，ECC把这话演到了极致——一份hooks配置里挂了二十多个hook，覆盖SessionStart、PreToolUse、PostToolUse、PreCompact、Stop、SessionEnd全生命周期。

ECC hooks 全生命周期挂点

挑几个我觉得最有讲点的——

**SessionStart的session:start——自动加载上次的上下文**。每次新session启动，hook跑一遍`session-start-bootstrap.js`，做两件事——把上一次session留下的状态文件读回来塞进context、自动检测当前项目用的是npm还是yarn还是pnpm。这意味着你换台机器、重启agent、几天后回来接着干，第一句话agent就知道"上次干到哪、当前项目用什么包管理器"，不用你手动交代。这是"长任务可恢复性"的最低门槛实现，也是06篇讲的"把状态外化成artifact"思路在SessionStart这一头的落地。

**Stop阶段的session-end加cost-tracker加evaluate-session——回话结束做三件事**。每次Claude回完话，Stop hook链上跑——`session-end`把transcript\_path指向的完整对话历史持久化下来、`cost-tracker`把这一轮的token和cost累加到per-session指标里、`evaluate-session`扫一遍session看里头有没有可以抽出来当skill的pattern。这三个加起来，你的每一次对话都同时是"工作记录"、"成本账本"、"skill原料"——开源repo里既能查回某次对话干了什么，又能跑出本月API账单分布，还能把高频pattern喂给instinct系统。**生产级agent需要的可观测性，靠这三个hook就攒齐了**。

**post:edit:accumulator加stransform: translateY(format-typecheck——延迟批量执行的性能优化**。这个组合特别能体现ECC"性能优化系统"的定位——别的harness改一个文件就跑一次prettier和tsc，每次都启动node进程、读全配置、跑完再退；ECC的做法是PostToolUse的`accumulator`只把被改的JS/TS文件路径攒下来不动手，攒到Stop时`format-typecheck`一次启动跑完所有文件。一次响应里你可能改了8个文件，传统做法启动8遍工具链、ECC启动1遍。在长session里这个差距能省掉几十秒到几分钟的wall-clock time。**hook层做batch processing——这思路在企业级后端工程里再常见不过，ECC把它搬到了agent harness里**。

延迟批量执行 vs 逐次执行

**pre:edit-write:suggest-compact——在逻辑边界提醒手动compact**。这条直接呼应06篇讲的strategic compact——hook在每次Edit/Write前看一眼当前context用了多少、上一次compact是什么时候，到了"该考虑主动压一下"的时刻就给agent一个提醒。它不强制压、只建议，把"语义边界判断"的主动权留给agent和用户。**自动机制和人工判断之间留一个柔性提醒位，是个挺成熟的设计**。

**pre:config-protection——拦"为了让lint过去改lint规则"反模式**。这是我看到拍案的一条。前面第十二篇讲Superpowers把"为了让测试通过去改测试"做成显式拒绝，这条hook做的事情是同一思路在另一个方向——agent要改`.eslintrc`、`.prettierrc`、`tsconfig.json`这种linter/formatter配置时直接拦下，告诉它"该改的是代码不是规则"。**配置文件是工程规范的物化，让agent乱改等于在自己腿上锯**。

**pre:edit-write:gateguard-fact-force——首次改文件前强制做事实调查**。每个文件的第一次Edit/Write被hook拦下，要求agent先查清楚——这文件被谁import了、它对应的数据schema长什么样、用户的原始指令到底是什么。把"先改了再说"的反模式按死在第一次动手前。**这条尤其适合大型代码库——一个仓库里改一个文件可能牵动几十个上游依赖，没事实调查就动手是事故根源**。

这套hooks设计的精髓我总结一句话——**hook不是"额外的check"，是harness工程纪律的物化层**。ECC把工程团队靠CI、code review、人工把关守住的那些底线，全部下沉到了hook这一层，由系统在每次工具调用前后强制执行。你想跳过都跳不过去。

—— AgentShield——独立的agent harness安全扫描器

ECC生态里有个独立项目叫AgentShield——一个专门做agent安全扫描的工具。

数据先放上来——**1282 tests、98% coverage、102 rules**。

它的工作方式是扫描你的agent配置——CLAUDE.md、settings.json、MCP config、hooks、agent definitions、skills——这五个对象。每个对象按102条规则查，找出可能的安全风险——prompt injection向量、过度权限、敏感信息泄露、不安全的工具配置。

最炫的是`--opus`模式——跑一个三个Opus 4.6 agent的pipeline——**红队agent试图找漏洞、蓝队agent试图防御、审计员agent给出最终评估**。这种对抗性分析比单个agent的判断更可靠——单个agent可能因为bias错过漏洞，三个agent的对抗能更全面。

AgentShield 红蓝紫三 agent 对抗 pipeline

**官方slogan特别戳——"Adversarial reasoning, not just pattern matching"**——对抗性推理，不是简单的pattern match。

这套思路对国内做企业内部agent安全的人启发很大——**传统安全扫描器是基于已知规则的pattern match，遇到新型攻击就抓瞎**。AgentShield的多agent对抗模式，让安全检查具备了"自己想"的能力——能针对你的具体配置思考"攻击者会怎么钻空子"，找出pattern matcher找不到的问题。

中国企业做合规要求高的agent项目，这种工具应该早早进CI——每次配置变更跑一遍AgentShield，失败的提交不允许合并。

—— ECC Tools加生态化运营

ECC有一点我特别要强调——**它把开源repo产品化、商业化**。

这件事在开源世界其实少有人做好。大多数开源项目就是github上的代码加README，没有商业模式、没有持续运营、没有产品化思路。

ECC不一样——

**GitHub App（ecc-tools）**——通过GitHub的应用机制提供集成体验。
**独立npm包（ecc-universal）**——一行npm install就装上。
**独立网站ecc.tools**——专门的产品官网。
**AgentShield单独发布**——做为独立产品而不是ECC的子模块。
**Skill Creator GitHub App**——从git history生成SKILL.md，独立功能独立发布。

**这是"开源 + 商业 + 社区"的三轮驱动**。开源吸引贡献和验证、商业产品提供长期可持续性、社区做生态扩散。

跟前面对比——Superpowers走"开源 + sponsor"路径，靠GitHub Sponsor持续；gstack走"YC CEO自用秀肌肉"路径，靠个人品牌带流量；ECC走"开源 + 产品化"路径，是这三条路里最重也最有想象空间的。

**这种路径在中国开源圈子里其实不多见**。国内开源项目要么是大厂做的内部工具开源、要么是个人项目持续靠爱发电。ECC这种"个人开发者把开源做成可持续商业"的范式，是值得国内开发者学习的。

—— 三家个人开发者派的总结

讲完ECC，可以收一下三家个人开发者派的对比——

**Superpowers**——切入点是工程方法论。
**gstack+gbrain**——切入点是组织角色化。
**ECC**——切入点是性能优化系统化。

**三家都不只做技术，都做了产品化和生态运营**。Superpowers进Anthropic Marketplace、gstack和gbrain有YC CEO的个人IP背书、ECC有独立网站和多个独立子产品。

**三家都跨多个harness兼容**。Superpowers六个、gstack支持Claude Code和OpenClaw、ECC九个。**这是harness这个生态的共识——绑定单一platform是早期产品的过渡形态，长期看必然走向跨平台**。

**三家都用真实生产数据背书**。不是benchmark数据、不是demo视频，是真实使用产生的效率数据、token数据、cost数据。**这是这一代harness工具区别于前几代AI工具的关键特征——可以用工程指标量化讨论**。

—— Harness Generation 2026的真实状态

整个案例解剖板块到这一篇就完整了。八个项目看下来，你应该对2026年harness生态有了完整图景——

**Anthropic的Claude Code**——产品级标杆。
**LangChain的DeepAgents**——理论权威参照。
**badlogic的pi-mono**——白盒教学样板。
**Steinberger的OpenClaw**——messaging-first扩展。
**Nous Research的Hermes**——self-improving前沿。
**obra的Superpowers**——方法论强制范本。
**Garry Tan的gstack+gbrain**——个人极致定制。
**affaan-m的ECC**——性能优化和跨harness工程化。

这八家加起来构成一个完整的实践谱系——从大厂产品到个人项目、从opinionated batteries-included到极致定制、从黑盒到白盒、从CLI到messaging、从手工写skill到自我演化、从单harness到九harness兼容。

**这是harness这个抽象层在2026年的真实状态——已经成熟，已经多元，已经在生产级别使用**。

下一篇做整个案例板块的横切面综合——Skills生态。我们看完八个项目里skills的不同形态，正好可以从更高视角讨论"skill这件事本身往哪走"——它的本质是什么、有几种主要形态、自动化生成有几种路径、生态会怎么演化。

case看完了，规律要抽出来。