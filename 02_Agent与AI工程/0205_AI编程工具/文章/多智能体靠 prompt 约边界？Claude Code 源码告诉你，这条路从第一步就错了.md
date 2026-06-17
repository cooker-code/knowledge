---
title: 多智能体靠 prompt 约边界？Claude Code 源码告诉你，这条路从第一步就错了
author: 数镜智心
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485614&idx=1&sn=4fe70dd2efeb9e085e6eace81f85006e&chksm=ceae495e827f44be19612239b64f6163e1536ef141536d2be71b4c1299be482eff22e8821a3c&mpshare=1&scene=24&srcid=04110jSDo2ZX8wdQvUIKuFOZ&sharer_shareinfo=6dadb1e8f86a44991b6e2cf1b0efc194&sharer_shareinfo_first=6dadb1e8f86a44991b6e2cf1b0efc194#rd
---

> 上一篇我拆了 Claude Code 的提示词系统，得到一个结论，它不是「一条 prompt」，而是一整条分层装配管线。继续往下看多智能体相关代码后，这个判断又往前走了一步。
>
> Claude Code 的 agent 体系，至少从这份源码快照来看，重点并不在「让几个角色互相聊天」，而在于怎么把 agent 做成一个可配置、可隔离、可恢复的运行时单元。

---

## 大多数人先入为主的地方，恰好是偏的

很多人脑子里对「多智能体」的第一反应，都是角色分工。

一个 Planner 负责规划，一个 Coder 负责实现，一个 Reviewer 负责审查，一个 QA 负责验证。它们彼此对话、互相传话，像一个小团队在开会。

这种想象当然不算错。今天不少框架和教程，默认入口就是这条路，先定义角色，再给每个角色一段不同的 prompt，最后把它们编排起来。

但 Claude Code 这份源码快照给我的感觉不太一样。它当然也有 prompt，也有不同 agent 类型，但底层真正在驱动的东西，不在「角色扮演」这一层。

**agent 在这里首先是运行时配置对象，人设只是其中一个字段。**

一旦换到这个视角，源码里很多看起来分散的设计就串起来了。

* 为什么工具边界不是只靠 prompt 约束
* 为什么通信里会出现结构化消息
* 为什么会有 background、worktree、remote 这些不同运行形态
* 为什么恢复逻辑依赖 transcript 和 metadata，而不是只靠上下文窗口

所以更准确的说法不是「Claude Code 有很多角色」。它更像是在用一套统一执行底座，调度多种不同的 agent 运行形态。

两种理解看着像文字游戏，往下走会发现它们指向完全不同的工程路径。

---

## 底层真正统一的，不是角色，而是执行链

先看最底下那条链。

主会话这边，`QueryEngine` 的注释写得非常直接。

> One QueryEngine per conversation.

它负责一个 conversation 级别的生命周期和状态推进。每次 `submitMessage()`，不是重新造一套系统，而是在同一条会话链上往前走一格。

再往 worker 侧看，无论是 subagent，还是 in-process teammate，最后都会落到 `runAgent() -> query() -> tools` 这条路径上。

很多人看到「多智能体」，容易先看最表层，有几个 agent，有几个名字，有几个 prompt。但 Claude Code 这套实现更值得看的，是它底下其实在复用同一套 query/tool/runtime 基础设施。

严格一点说，不是所有形态都共享同一个 `QueryEngine` 实例。从源码实现看，**它们共享同一套执行机制，而不是共享同一个对象。**

如果把结构画粗一点，大概是这样。

```
主会话:  
  QueryEngine.submitMessage()  
    -> query()  
    -> tool loop  
  
worker / teammate:  
  runAgent()  
    -> query()  
    -> tool loop
```

这背后的味道已经不是「几个 prompt 摆在一起」了。上层是不同入口，中层是不同运行形态，底层是同一条执行主链。一个 runtime 该有的分层，它基本都具备了。

---

---

## Agent 在这里首先是配置，不只是 prompt

再往上一层看 `AgentDefinition / BaseAgentDefinition`。

如果你之前把 agent 理解成「一段 system prompt」，那这里会有点反直觉。因为源码里 agent 明显不止 prompt 这一维。至少从当前泄露的源码快照里，agent 这一层已经带着一整组运行时属性。

```
AgentDefinition {  
  tools  
  disallowedTools  
  mcpServers  
  permissionMode  
  maxTurns  
  memory  
  background  
  isolation  
  skills  
  hooks  
  model  
  effort  
  initialPrompt  
...  
}
```

在 Claude Code 里，agent 的「定义」实际上是多维度的配置。

* prompt 定义行为倾向
* tools 和 disallowedTools 定义能力边界
* permissionMode 定义权限模式
* memory 定义记忆范围
* background 和 isolation 定义运行方式
* maxTurns 定义执行上限

这和「我给它写了一段人设 prompt」完全不是一回事。

说真的，这里最值得学的，不是字段有多少，而是工程取向变了。

如果你的系统只是靠 prompt 告诉模型「不要做这个」「尽量别做那个」，那说到底还是软约束。

模型听不听，最终还是概率问题。

而 Claude Code 这套代码更像是在做硬边界，能不能用这个工具，运行在哪个隔离层，有没有后台执行权限，这些不是靠文案提醒，而是靠配置截断。**prompt 是行为偏好，配置才是能力边界。** 

两者差一个工程时代。

> 💡 顺便提一嘴我自己的体感。之前做 AtomStorm🔗的 Skills 架构时，走过一模一样的弯路——一开始拼命往 system prompt 里加约束条款，Agent 照样越界。后来干脆把能力边界切成运行时配置：主 Agent 手里只捏着元技能描述，具体 MCP 工具只在对应 Skill 触发时才加载，执行完立刻卸载上下文。这和 Claude Code 的 `disallowedTools` + 场景隔离，本质上是同一件事——**边界靠配置管，不靠 prompt 守。** 关于多 Skill 场景下 Context 怎么分配、注意力怎么保活，之前专门写过一篇，这里不展开。

---

## 真正的协作信号，不靠聊天，靠协议

Claude Code 的 teammate 通信，是另一个很能说明问题的地方。

如果是纯「角色式多智能体」，最自然的做法通常是一个 agent 说一段话，另一个 agent 回一段话。自然语言既是内容，也是控制信号。但 Claude Code 这里已经明显不是这么玩的。

在 `teammateMailbox.ts` 里，能直接看到一批结构化消息类型。

* `permission_request` / `permission_response`
* `sandbox_permission_request` / `sandbox_permission_response`
* `plan_approval_request` / `plan_approval_response`
* `shutdown_request` / `shutdown_approved` / `shutdown_rejected`
* `task_assignment`
* `team_permission_update`

权限请求怎么发，计划审批怎么回，关闭流程怎么走，任务怎么分配，这些全是协议层的事，不是「你说一句、我理解一句」能兜住的。

这个区别在 demo 阶段不一定明显，但一进生产环境就很大。因为自然语言适合表达任务内容，不适合承担所有控制信号。

前者需要弹性，后者需要确定性。

Claude Code 这套设计的取向很清楚，**任务内容可以自然语言，控制信号尽量结构化。** 所以它看起来更像一个 runtime 在做消息路由，而不是一个剧场在安排对白。

---

## 它更像多种运行形态，不是多种人格

按这份源码快照归纳，Claude Code 至少已经长出了几类不同的 agent 运行形态。

| 形态 | 典型机制 | 隔离特征 |
| --- | --- | --- |
| 同步 subagent | `runAgent()` 直接执行 | 独立上下文，通常不脱离当前会话 |
| 异步 background agent | `run_in_background` / `background: true` | 后台任务，和主线程解耦 |
| in-process teammate | `spawnInProcessTeammate()` | 同进程、独立上下文 |
| worktree-isolated agent | `isolation: "worktree"` | 独立工作目录 |
| remote agent | `isolation: "remote"` | 委托到 CCR 远端环境 |

这里我还是要压一手，免得写飘了。`worktree` 是明确可见的，`background` 也很清楚，`in-process teammate` 更是直接落在代码里。`remote` 这条分支也能看到，但它带着很明显的内部分支和条件编译痕迹，所以更稳的说法不是「这已经是公开稳定能力」，而是，**从这份快照能看出来，Claude Code 的 agent 运行形态已经在朝分层、分隔离级别的方向设计。**

你想想看，真正可用的多智能体系统，最后拼的往往不是「角色数量」，而是这些判断，哪些任务该同步，哪些该后台，哪些该共享文件系统，哪些必须 worktree 隔离，哪些甚至需要远端环境。一旦你从这个角度看问题，系统味道就彻底不一样了。

---

## 可恢复执行存在，但别把它简化成一句「稳定 agentId」

坦率的讲，原来最容易被写错的，就是这块。

先说清楚，teammate 的 ID 确实是确定性的，格式是 `agentName@teamName`。但普通 subagent 的 ID，不是这种稳定命名，而是 `createAgentId()` 生成的运行时 ID。

所以不能粗暴写成「每个 agent 都有一个稳定的 agentId」。

更准确的说法是，**Claude Code 已经为 agent 恢复准备了独立的持久化材料，恢复依赖的是 transcript、metadata 和专门的 resume 逻辑。**

源码里能直接看到几件事。

* subagent transcript 会单独写 sidechain
* agentType、worktreePath、description 会写 sidecar metadata
* `resumeAgentBackground()` 会把 transcript、metadata、replacement state🔗 重新拼回去

它已经不是那种「一旦上下文窗口清掉，之前的一切都要靠模型重新想起来」的设计了。。。

模型上下文负责当下计算，关键状态放到外部持久层，恢复时用代码重建执行现场。思路非常接近传统 runtime 的 checkpoint/restart。

---

## Team Memory 这层是实的，但别神化

还有一个容易被忽略，但很能体现系统成熟度的点，Team Memory。

这不是一句抽象概念。源码里能看见一整套实体，team memory 目录、prompt 注入、sync 服务、路径越界校验，甚至还有 secret guard 防止把敏感信息写进共享记忆。

连 UI 文案都直接把它写成。

> shared team memory, synced across the organization

Claude Code 已经不满足于「大家看同一段共享 prompt」这种最原始的共享方式了。它开始把「团队级共享信息」抽成一个独立层。

当然，这里也不能写飞。我自己也在反复确认，从当前可见代码里，我能确定它确实有共享记忆层，确实有同步机制和安全边界，确实在防越界、防泄密。但我不会把它直接夸成「完整的细粒度 ACL 记忆系统」，因为这一步从当前证据还推不到。

相对准确一点的说法是，**Claude Code 已经把共享信息从 prompt 拼接，推进到了独立记忆层。** 这一步本身值得认真对待。

---

## 这套设计真正值得学的，是运行时观，不是角色观

把这些点连起来看，Claude Code 这套实现里最值钱的，不是它造了多少 agent，也不是它给角色写了多少人设。真正有价值的是，它在慢慢把 agent 系统从「prompt 编排」推向「运行时工程」。

* agent 是配置对象，不是只有人设
* 通信里关键控制面是协议，不是全靠自然语言
* 运行形态有分层，不是一把锤子打天下
* 状态能恢复，不是全压在上下文窗口里
* 共享信息有独立层，不是无限往 prompt 里塞

**很多人还在把多智能体当「角色设计问题」，但 Claude Code 明显已经在把它当「运行时设计问题」处理。** 这两种视角的差别，不在文案层，而在系统最终能不能做成生产级。

我是真的觉得，真到了复杂任务里，决定系统上限的通常不是「角色设定够不够丰富」，而是边界能不能控制、状态能不能恢复、协作能不能路由、隔离能不能分级、生命周期能不能管理。而这些，恰好都是 runtime 的语言，不是角色扮演的语言。

---

## 最后

如果只从外面看，你完全可以把 Claude Code 描述成「它支持很多 agent」。但只要往代码里再钻一层，会发现这句话其实没说到重点。

Claude Code 的多智能体实现，更接近一套配置驱动、可隔离、可恢复的 agent runtime，而不是多个角色 prompt 的编排。**它不是让多个角色彼此对话，而是让一个统一的 runtime 去调度不同执行形态的 agent。**

这不是抬杠，也不是换个更高级的词。一种是在设计角色，另一种是在设计运行时。前者适合做演示，后者才更像是在为生产环境做准备。

这才是它真正有意思的地方。

说个题外话，Claude Code 的 runtime 观，和我自己在 AtomStorm 里做 Skills Vibe Agents 踩出来的结论是同源的——都是把 agent 当配置对象管，不是当角色写。如果你读完这篇觉得「配置驱动多智能体」这个思路有启发，那篇 Context Engineering 的文章里有一整套 Skills 按需加载、上下文隔离的工程拆解[全球首个Skills Vibe Agents，AtomStorm技术揭秘：我是怎么用Context Engineering让Agent不"变傻"的](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485151&idx=1&sn=13206f54618166c276507e91e322ccab&scene=21#wechat_redirect)，可以串起来看。

本文未经同意，禁止私自转载

---

我的产品studio.atomstorm.ai 已正式发布注册[4个月烧了2万刀Token，全球首款Skills Vibe Agent终于开启邀请内测，我也终于敢说：Sam Altman预言的超级个体，可能真的来了](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485129&idx=1&sn=efea9accb62a7502b464fa872b1cacdd&scene=21#wechat_redirect)。同时，我们也在结合龙虾的产品思路做一些其他的创新尝试， 会更加大胆激进。 

 感兴趣的朋友欢迎技术咨询、商务合作，让我们一起推动AI Agent让每个人在数字世界劳动自由这一天更快点到来。

栗子KK，一个在 AI 浪潮中游泳的 AI 产品 Founder

欢迎点赞、在看、关注，持续为你带来最前沿的AI资讯、认知、实战。

**往期精彩推荐：**

**[Claude Code 提示词系统深度拆解：这才是 Agent 专业度的真正分水岭](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485603&idx=1&sn=faa56be50f8eddc3c0e8ef9fcf44370f&scene=21#wechat_redirect)**

**[Claude Code 源码泄露深度分析：一次意外曝光，暴露出终端 Agent Runtime 的全貌](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485583&idx=1&sn=b03995c569b94435aaf2095eb9550505&scene=21#wechat_redirect)**

**[Harness驱动的Agent工程解密，后小龙虾时代的AI Agent新范式？](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485485&idx=1&sn=d0938cbbfbf7437a9867b291a88634bf&scene=21#wechat_redirect)**

[全球首个Skills Vibe Agents，AtomStorm技术揭秘：我是怎么用Context Engineering让Agent不"变傻"的](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485151&idx=1&sn=13206f54618166c276507e91e322ccab&scene=21#wechat_redirect)

**[Atlas for Mac 发文后被催着要下载链接](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485468&idx=1&sn=4a7b6ddb33161b4c8b910e9c2aec9496&scene=21#wechat_redirect)**

**[，两个小时，我直接用 Claude Code 做了个 LandingPage上线](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485468&idx=1&sn=4a7b6ddb33161b4c8b910e9c2aec9496&scene=21#wechat_redirect)**

**[做了两年AI Agent，我发现99%的AI Agent项目都死在了Message Flow设计上](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247484194&idx=1&sn=5db2099f5930874735b8e44ae12964de&scene=21#wechat_redirect)**

**[我翻遍了Claude Code的system prompt，发现它的"记忆"就是一个200行的markdown文件](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485369&idx=1&sn=5c1a7daec5af8cb1af0094acab1774c0&scene=21#wechat_redirect)**

**[别再让你的AI单打独斗了！Codex新版暗藏的多智能体并发功能，1分钟教你把它变成“包工头”](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485346&idx=1&sn=846c87873673b832e30029d4f20553ce&scene=21#wechat_redirect)**

**[我研究了OpenClaw的8个"反常识"设计，终于明白这个Agent为什么能火爆全球](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485292&idx=1&sn=3fe8b8968166f5dd182fd9d10fae5d3b&scene=21#wechat_redirect)**

[AI编程正式进入"团战时代"：Claude Code Agent Teams，我等了两年的功能终于来了](https://mp.weixin.qq.com/s?__biz=Mzg4MzAzNTA5Ng==&mid=2247485253&idx=1&sn=0722908710926fbfa1b59b0507f103fa&scene=21#wechat_redirect)