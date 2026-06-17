---
title: DolphinScheduler Agent 开源上线｜从告警到自愈一键闭环，运维终于可以“躺着把活干了”！
author: 白鲸开源
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkyODg0MzA1MQ==&mid=2247491361&idx=1&sn=8b656a215ea44dda3984440eeb28a8be&chksm=c301d60bca4e84fb123c81a78cd49ad72bb1e48e0dcb83baafdad8148b1c78b67f00d79382b6&mpshare=1&scene=24&srcid=0430gO3oBlksxB2gDvp1HZ14&sharer_shareinfo=b03e403dace5cd1eac82440f54026a59&sharer_shareinfo_first=b03e403dace5cd1eac82440f54026a59#rd
---

点亮⭐️

https://github.com/apache/dolphinscheduler

**点击蓝字 关注我们**

在 2026 Apache DolphinScheduler Meetup 技术分享中，由刘小东带来的 **DolphinScheduler Agent** 解决方案一经亮相，便成为社区焦点。这套打通「群聊告警→智能诊断→自动恢复→报告闭环」的全链路系统，可以很好地解决传统运维碎片化、高耗时、反复跨系统跳转的痛点，让大数据任务故障处理从“人肉奔波”迈入“智能自治”时代。

目前，项目核心支撑工具 **dolphinscheduler-cli（dsctl）**已正式在 GitHub 开源，面向所有开发者免费使用！

**故障恢复慢，**

**不是命令难，是跳转太碎**

日常使用 DolphinScheduler 时，任务失败处理一直给运维带来不小的压力。

你一定熟悉这样的流程：飞书群弹出告警 → 打开 DS UI 查实例状态 → 翻日志定位失败原因 → 对照 Runbook 判断风险 → 人工决策操作 → 再回到群里回复结果……

真正拖慢效率的，从来不是执行某条命令，而是**上下文在多个系统间反复断裂**。事实、证据、风险分散在不同工具里，运维人员把大量时间浪费在“找信息、拼逻辑、补上下文”上，协作断点多、排查成本高、故障恢复周期被无限拉长。

而这一切，在 DolphinScheduler Agent 面前，将成为历史。

**从碎片化人肉协作，**

**到全链路智能闭环**

针对上述断点，DolphinScheduler Agent 方案的目标非常清晰：把一次失败告警，变成一条连续、可追踪、可沉淀的处理链路。旧方式是告警、UI、日志、群聊、复盘各自独立，全程依赖人肉协作；新方式则以飞书告警为起点，经由 **Channel 会话、智能编排、执行控制、验证回复，最终沉淀为报告**，让故障从触发到闭环一气呵成，不再需要人工在多系统间反复跳转。

简单来说，就是告警发出来，Agent 自动接手；处理完成，自动在群内回复结果，并输出完整事故报告。运维人员只需要看结论，不再需要“跑断腿”。

**五层硬核架构：不是简单脚本，**

**是安全可控的智能控制链**

很多人会误以为，自动化运维就是“机器人+脚本”。

但 DolphinScheduler Agent 走了一条更稳健、更工程化的路——**五层解耦控制链**，每一层各司其职，层层递进，既保证执行能力，又牢牢守住安全边界。

1. **L1 事件与协作**

   告警直接进入飞书线程，支持人工随时介入与追问，以 workflowInstanceId 作为唯一事故锚点，信息不丢失、不分散。
2. **L2 会话接入**

   飞书事件同步至本地会话，全程保持上下文连贯，彻底消除跨系统切换带来的断点问题。
3. **L3 智能编排**

   由 Claude Code 负责信息组织与调用顺序编排，Skill 承载 DS 领域专业规则，让决策更精准。
4. **L4 执行控制**

   由 dsctl 统一承担**读证据、修故障、验结果**的核心动作，标准化命令，稳定可复用。
5. **L5 沉淀治理**

   自动完成飞书群快捷回帖、事故报告生成、审计日志留存，兼顾实时协作与后续复盘。

这样的设计切中运维人员的需求，架构解耦，能力才能稳定扩展；边界清晰，自动化才敢上线生产。

**四大核心模块：端到端支撑，**

**故障自愈真正落地**

在五层架构之上，四大模块紧密配合，让整套系统“能用、好用、敢用”。

### 📌 Channel：飞书原生入口，一站式协同

飞书群既是**告警入口、协作界面，也是结果回执页**。Agent、人工、值班流程在同一线程内协同，群内只展示精简结论，详细证据沉淀至报告，沟通高效、证据可查。

### 📌 Runtime：智能编排引擎，规则与执行分离

Claude Code 负责会话逻辑编排，Skill 承载故障响应、工作流设计、数据质量等专业规则。  
编排、规则、执行三层解耦，系统可稳定扩展，持续迭代升级。

### 📌 Control Plane：dsctl 统一控制面，自动化的“执行底座”

dsctl 是整个 Agent 的能力核心，提供标准化、可被自动化调用的 CLI 能力：

* 读证据：doctor / digest / log 快速定位故障现场
* 修实例：recover-failed / edit --dry-run 安全修复，支持预演
* 验结果：watch 实时监控状态，digest 输出总结
* 统一输出：所有动作标准化返回，可观测、可追溯、可审计

正是有了 dsctl，手动命令才能变成稳定的自动化能力。

### 7 步标准闭环：双路径保障，生产环境更放心

从告警触发到复盘沉淀，Agent 严格遵循 7 步标准状态机：告警解析 → 诊断 → 决策 → 执行 → 验证 → 回复 → 沉淀

* 面对低风险、证据充足的场景，自动走**顺利路径**：读证据→生成执行计划→恢复失败任务→验证→群内短回帖 + 输出报告
* 遇到证据不足、高风险或验证失败，则走**升级路径**：转交人工，保留完整上下文，不虚假上报成功

全程可追溯、可审计、可复盘，真正做到放心上线、稳定运行。

### 📌 Safety：四级风险管控，安全是第一前提

生产环境的自动化，**安全永远比速度更重要**。  
系统按风险等级设置边界，将操作分为四类：

* 自动（默认允许）：只读查询、查看日志等无风险操作
* 自动+防护：recover-failed 等低风险恢复操作
* 人工审批：实例修改等高风险动作，必须人工确认
* 禁止：数据强制成功等高危操作直接拦截

这样就明确了系统的核心安全主张：**Agent 的强大，不在于“敢跑”，而在于知道“什么时候不跑”。**

****逐步放权，走向自治运维****

为了保证在生产环境安全落地，Agent 采用分步放权、小步迭代的路线，以确保生产稳定：

* MVP 阶段：先实现只读诊断，跑通短回帖闭环；
* V1 阶段：开放 recover-failed 低风险自动恢复；
* V2 阶段：接入审批机制，扩展更多可控操作；
* V3 阶段：沉淀 Runbook / Skill，面向社区共建。

这套方案最有价值的，不是某个提示词，而是 **Channel + Skill + CLI + Report + Safety** 一整套可复制、可迁移的工程化边界。

**Demo演示**

为了大家能对 DolphinScheduler Agent 的能力有直观的理解，刘小东还在现场进行了 Demo 演示，详见文首视频 57:10 及以后内容。⬆️

**正式开源，dsctl已上线**

好消息是，支撑 DolphinScheduler Agent 实现全能力的核心项目 **dolphinscheduler-cli（dsctl）已正式开源！**

🔗 GitHub 地址：https://github.com/sketchmind/dolphinscheduler-cli

项目提供完整命令行工具，支持：

* DolphinScheduler 配置与环境管理
* 工作流编写、Lint 检查、DryRun 预演
* 运行时监控、实例查看、日志拉取
* 故障恢复、失败重跑、批量运维
* 标准化输出，完美适配自动化与 Agent 调用

项目采用 Apache-2.0 开源协议，支持 pip 一键安装，兼容 DolphinScheduler 3.3.2 / 3.4.0 / 3.4.1 等主流版本，开箱即用。

**写在**

最后

DolphinScheduler Agent 的出现，重新定义了大数据任务运维范式：**把人从重复、琐碎、跨系统跳转中解放出来，让系统负责处理故障，让人专注决策与治理。**

从告警弹出，到自动恢复、自动回帖、自动沉淀报告，一键闭环，全程无忧。如果运行顺利，运维真的可以说是 **“躺着把活干了”**。

欢迎所有 DolphinScheduler 用户、运维开发者、大数据工程师前往 GitHub 体验 dsctl，一起参与社区共建，让运维更简单、更智能、更高效！

**·END·**

**白鲸开源**

白鲸开源是一家开源原生的DataOps商业公司，是国家高新技术企业，由多个Apache Foundation Member成立，80%员工都是 Apache Committer，运营2个全球Apache开源项目(DolphinScheduler, SeaTunnel）。白鲸开源已根据全球最佳实践发布商业版产品WhaleStudio(含白鲸数据调度平台WhaleScheduler和白鲸数据集成平台WhaleTunnel）。我们致力于打造下一代开源原生的DataOps 平台，助力企业在大数据和云时代，智能化地完成多数据源、多云及信创环境的数据集成、调度开发和治理，以提高企业解决数据问题的效率，提升企业分析洞察能力和决策能力。

**了解更多**

公司网站:www.whaleops.com

联系邮箱: xiyan@whaleops.com

如果您希望深入了解文中提到的数据质量功能，或者讨论如何将 WhaleStudio 与你的业务流程相结合，我们非常愿意为你提供帮助。**欢迎扫码获取****WhaleStudio产品白皮书**。

---

**下滑探索**更多WhaleStudio的优势，让我们帮助你构建一个高效、安全的大数据解决方案。🚀

## 金融行业的应用实例

****↓↓↓**点击下面链接阅读↓↓↓**

[国内某头部理财服务提供商基于白鲸调度系统建立统一调度和监控运维](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485832&idx=1&sn=28d8b40b2a752a431359a8cede446542&chksm=c171c1daf60648cc1724b11fc42a52c16ac0b8e29d26d6208cf897ba9c613ae0a1be152638a3&scene=21#wechat_redirect)

[白鲸调度系统助力国内头部券商打造国产信创化 DataOps 平台](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485636&idx=1&sn=a32f18255fdbf8b3510181ecf5d24a0d&chksm=c171c096f60649807f7c65a1a9b8de96b990539177752d3d31b21d9e15720d09f7647d1f8b98&scene=21#wechat_redirect)

[白鲸开源 DataOps 平台助力证券行业实现信创数字化转型](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247484746&idx=1&sn=545477e0c76725117276d0aa0ab157b7&chksm=c171cd18f606440e0e683dfe8f346f98965f0dcd85def2ebb51371a871d202d40792567371a1&scene=21#wechat_redirect)

[最佳实践 | 从Airflow迁移到Apache DolphinScheduler](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485916&idx=1&sn=b9aa20d813ea8c729c601167a2e31183&chksm=c171c18ef6064898a0b996fd9f83c9ad0a6d507d0a5429ff0fd2b407112fd67ec3fd7382c79b&scene=21#wechat_redirect)

[Apache DolphinScheduler VS WhaleScheduler](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485887&idx=1&sn=59b4ca05a425734b02b5e530eeb082a6&chksm=c171c1edf60648fb49ed9ad7a9b6315a470daff4f22534e52a7a017fd10f3084f2fd006f5fcd&scene=21#wechat_redirect)

[代立冬：基于Apache Doris+WhaleTunnel 实现多源实时数据仓库解决方案探索实践](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247486052&idx=1&sn=b869e1243e2b1e65f3435c73efb6b1f4&chksm=c171c236f6064b2059ff35e5e104392d2b82801158d531eca1e5def35b474eacef78edb29ea6&scene=21#wechat_redirect)

[白鲸开源在中信建投 DataOps 应用实践](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247486014&idx=1&sn=b157c765dee67f7a53dcb73efffaf871&chksm=c171c26cf6064b7a472bf00207ba93a88b23a164b9c14a84b4a35212e8060ee8955b69a5ad21&scene=21#wechat_redirect)

## 商业版技术解析实例

**点击下面链接阅读↓↓↓**

[被热议的“DataOps”是炒作？](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247486151&idx=1&sn=81cd58a5d2d4248570ec1413b4a9111a&chksm=c171c295f6064b83d52e3f782ea99377355f06811ecb22ad32661c1d2e233d2add7cfcb84760&scene=21#wechat_redirect)

[WhaleScheduler：高并发下的稳定性与性能实践](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485953&idx=1&sn=7a9077f6e6b4067df585b16f6894cc09&chksm=c171c253f6064b454931eac815afc084110ab6c89098e2b6bb9255fa9e90b3834b873284f859&scene=21#wechat_redirect)

[驾驭数据的未来：WhaleStudio与DataOps的完美结合](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485494&idx=1&sn=2ea61e0878601f6518ac542b7af4e01b&chksm=c171c064f6064972f48ac5dd6bd24e6b1ec040d913a8503e17579f6c06426a724b5d3ce355f3&scene=21#wechat_redirect)

[WhaleStudio：创新性解决大数据挑战的工具](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485713&idx=1&sn=f970b361de1b5dece21d323787077271&chksm=c171c143f6064855ec5de00d6b54b4f5e6bb46e61a80100a59e0e31bb772855ddb7e03946c84&scene=21#wechat_redirect)

[支持全生态调度：构建企业数字化转型的桥梁](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485769&idx=1&sn=b9f39414db17bdd5b2efd308f78a55f6&chksm=c171c11bf606480d01406f37501b63fb28c181bcd15208acb6cb5a94bf9a4726a6ba59656269&scene=21#wechat_redirect)

**运营开源项目**

目前，北京白鲸开源科技有限公司运营着已经从 Apache 基金会毕业的大数据工作流调度平台 Apache DolphinScheduler，以及数据集成平台 Apache SeaTunnel，诚邀全球伙伴加入开源共建！

**Apache DolphinScheduler：**

仓库地址：https://github.com/apache/dolphinscheduler

官网：https://dolphinscheduler.apache.org/

**Apache SeaTunnel：**

仓库：https://github.com/apache/seatunnel

官网：https://seatunnel.apache.org/

点个在看你最好看