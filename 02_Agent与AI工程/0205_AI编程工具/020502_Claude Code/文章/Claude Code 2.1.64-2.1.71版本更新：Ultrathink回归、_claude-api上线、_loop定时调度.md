---
title: Claude Code 2.1.64-2.1.71版本更新：Ultrathink回归、/claude-api上线、/loop定时调度
author: 与AI同行之路
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzODk3MTIwMQ==&mid=2247488973&idx=1&sn=906c6d820d01c10501f6faebf343e62e&chksm=c30f24872e887d2b066681d291e9224b5be0cbd93f0365f36829990abacd788e59c0f89ecd13&mpshare=1&scene=24&srcid=0309SvLCMB9v3QQEqsT8TWmB&sharer_shareinfo=30aae3b431f91cc9d54959f027e18ba6&sharer_shareinfo_first=30aae3b431f91cc9d54959f027e18ba6#rd
---

阅读前记得关注+星标，及时获取更新推送

上篇《[Claude Code 2.1.50-2.1.63版本更新：Remote Control来了、/simplify+/batch双剑合璧、HTTP Hooks开门](https://mp.weixin.qq.com/s?__biz=MzkzODk3MTIwMQ==&mid=2247488968&idx=1&sn=8e1babb5dfe4e88761c7063de8f33828&scene=21#wechat_redirect)》收尾时，HTTP Hooks刚刚打通了Claude Code和外部系统的通道，/batch和/simplify把多Agent编排包成了一行命令就能用的工具。没想到这口气还没喘匀，接下来几天又出了一批东西——而且这几张牌踩的位置都很准。

2.1.68，Ultrathink回来了。这个被移除之后社区一直在喊"Claude变笨了"的功能，重新出现在工具里——但背后的机制跟以前不一样了，不再是硬写token预算，而是对接了Opus 4.6的自适应思维体系。2.1.69，`/claude-api`内置技能上线，专门解决"在Claude Code里写Claude应用却要不停开浏览器查文档"这个反复打断节奏的痛点。然后到2.1.71，`/loop`来了——定时调度，让Claude Code会话里能跑cron任务，轮询部署状态、盯着PR合并、给自己设一个小时后的提醒，现在都能在session里搞定。

这一波更新量没有上几波那么密集，但信息密度很高。Ultrathink的回归牵出了整个努力控制体系的重新设计，`/claude-api`和`/loop`这类内置技能的方向越来越清晰——原本要查文档、要自己写脚本、要切换工具才能解决的事，Anthropic在一件一件地往工具里收。

---

## 版本概览

| 版本 | 发布日期 | 主要更新 | 重要程度 |
| --- | --- | --- | --- |
| **2.1.64-2.1.67** | 3月初 | 稳定性修复、性能优化 | ⭐⭐⭐ |
| **2.1.68** | 3月初 | **Ultrathink回归** 、Opus 4.6默认中等投入、Opus 4/4.1正式下线 | ⭐⭐⭐⭐⭐ |
| **2.1.69** | 3月初 | **/claude-api内置技能** 上线 | ⭐⭐⭐⭐ |
| **2.1.70** | 3月初 | Bug修复、稳定性改进 | ⭐⭐⭐ |
| **2.1.71** | 3月初 | **/loop定时调度** 内置技能上线 | ⭐⭐⭐⭐⭐ |

重点版本是**2.1.68**（Ultrathink+努力体系重构）、**2.1.69**（/claude-api）和**2.1.71**（/loop）。

---

## Ultrathink回来了，但不是原来那个Ultrathink

先说清楚背景。Ultrathink在更早的版本里被移除过，当时的实现原理很粗暴：触发时直接把推理token上限拉满——31,999个扩展推理token，强迫Claude做最深度的思考。效果是有，但资源开销也是真实的，Anthropic砍掉它应该是性能和成本双重考量的结果。

然后GitHub上出现了issue #19098，有人报告移除Ultrathink之后模型回答质量明显下滑，帖子2月份被标记"已完成"并关闭——然后Ultrathink就在2.1.68里回来了。

但这次回归，底层逻辑变了。

**Opus 4.6用的是自适应思维（adaptive thinking）**，模型会根据任务的复杂程度自行判断推理深度。之前Opus 4.6在Max和Team订阅用户上是默认高投入——意味着不管是重构核心模块还是帮你改个变量名，Claude都会认认真真地思考一遍。2.1.68开始，**默认改成了中等投入（medium effort）**。日常任务该快的快，复杂任务自然会多想一点，不再所有事一视同仁地高强度推理。

另外顺带一提：**Opus 4和4.1已经从Anthropic第一方API里移除**，原来固定使用这两个版本的用户会被自动迁移到Opus 4.6。这算是一次悄悄清理旧版本的动作，没有大张旗鼓，但影响面不小。

### 三个档位怎么理解

现在努力控制有三个拨盘：

* • 中等投入：默认状态，绝大多数日常任务够用
* • 永久高投入：通过`/model`或者`--effort high`设置，每次提示都走高强度思考
* • Ultrathink：在当前提示里加上`ultrathink`关键词，这一轮触发高投入，下一轮恢复默认

Ultrathink本质上是**单次超控**。它等于高投入，不是什么神秘的更高档位，也不会比你手动设`/effort high`更强。区别只有一个：范围。高投入是全局永久生效，Ultrathink是这一轮专用。

有一个细节值得注意：**只有当你当前处于中等或低投入时，Ultrathink才会生效**。如果你已经设了全局高投入，再输入ultrathink什么都不会发生——你本来就已经在高投入了。

另外还有一个`/effort`命令可以做精细控制，两个旋钮总比一个好。

### 中等投入和Ultrathink的实际差距

同一个调试任务：一个Node.js API，身份验证中间件有竞态条件，只有在并发请求下才会出现。

中等投入给的回答：找到了可能原因，指出异步处理器的问题，建议用互斥锁，然后继续往下走。好回答，够用。

Ultrathink给的回答：把整个请求生命周期梳理了一遍，找出了两个问题——我问的竞态条件，以及一个我没想到的token过期极端情况。还顺带给出了一套在负载下验证修复效果的测试策略。

中等投入给你答案，Ultrathink给你答案加上你没想到的问题。哪种场景该用哪种，自己掂量。

---

## /claude-api -- 把API文档装进会话里

这是2.1.69加的内置技能，解决的是一个很具体的问题。

在Claude Code里写使用Claude API的应用，之前的体验是这样的：写几行代码，忘了流式传输的事件格式，开浏览器标签，翻文档，回来发现context跑偏了，再想想刚才写到哪儿……这个来回切换的节奏，开发者应该都熟悉。

`/claude-api`干的事情很简单：**把完整的Anthropic SDK参考注入到当前会话上下文里**。调用之后，Claude就有了你花30分钟查文档才能得到的那份知识——不用离开终端，不用打开浏览器。

具体加载的内容包括：你项目所用语言的完整Claude API参考、工具使用模式和流式实现细节、消息批处理和结构化输出、还有常见坑的说明。对Python和TypeScript项目，还会额外加载Agent SDK参考。

支持的语言覆盖了Anthropic目前官方支持的所有语言：Python、TypeScript、Java、Go、Ruby、C#、PHP，以及cURL。

### 自动激活

更聪明的地方是：**如果你的项目里已经有Anthropic的import，不用手动调用`/claude-api`**，Claude检测到import之后会自动激活这个技能。

触发的import分别是：

```
import anthropic                          # Python
```

```
import Anthropic from '@anthropic-ai/sdk'  // TypeScript
```

```
from claude_agent_sdk import query        # Agent SDK
```

检测到这三种任意一个，技能就自动加载进来。你写代码，Claude自己知道要参考哪份文档。

### 实际效果

这是一个"内置技能做上下文注入"的典型案例。以前你要写流式输出的处理逻辑，Claude经常把事件类型搞错，或者用了一个已经deprecated的API。加载`/claude-api`之后，因为会话里已经有完整的SDK参考，Claude第一次就能给出正确的方法名、正确的事件处理格式、正确的参数结构——不需要你去纠偏。

这类内置技能的价值在于**精准消除一类上下文缺失**。它不是通用的"让Claude变聪明"，而是专门把某个领域的知识确定性地装进去。开发者使用体验的改善是具体的，不是玄学。

---

## /loop -- 会话里的定时调度器

2.1.71加的`/loop`，是让我觉得"这东西早该有"的那种功能。

```
/loop 5m check if the deployment finished and tell me what happened
```

就这一行。Claude解析时间间隔，转换成cron表达式，在后台设一个定时任务，每5分钟执行一次你给的那个提示，直到你关掉session或者手动取消。

这解决的是一个非常真实的痛点：你启动了一个部署，想知道它什么时候完成；你提了一个PR，想在CI跑完之后马上知道结果；你有一个长时间跑的构建，不想一直盯着终端刷新。以前这些场景要么靠你自己不停去问，要么在另一个终端里开一个watch命令，要么什么都不做，等着感觉差不多了再去看。

现在：

```
/loop 20m /review-pr 1234
```

每20分钟让Claude帮你跑一次PR review，自动更新状态，你去干别的事情。

### 间隔语法

灵活程度比我预期的高。几种写法都认：

```
/loop 30m check the build              # 前置时间  
/loop check the build every 2 hours   # 后置every子句  
/loop check the build                  # 不写时间，默认10分钟
```

单位支持s（秒，向上取整到分钟）、m、h、d。不整除的间隔（比如7m、90m）会自动取整，Claude会告诉你实际用的是什么间隔。

### 一次性提醒

`/loop`也能处理一次性的事：

```
remind me at 3pm to push the release branch  
in 45 minutes, check whether the integration tests passed
```

用自然语言描述，Claude转换成只触发一次的cron任务，到时候执行完就自动删掉自己。

### 管理任务

问Claude就行：

```
what scheduled tasks do I have?  
cancel the deploy check job
```

底层有三个工具：CronCreate、CronList、CronDelete。你不需要直接调这三个，自然语言就能管。每个任务有8位ID，单个session最多50个任务。

### 几个要知道的限制

**任务是session级别的**。关掉终端，或者退出Claude Code进程，所有定时任务都消失。它不是守护进程，不是系统cron，活在当前session里。

**循环任务有3天自动过期**。跑满72小时后，任务触发最后一次，然后自动删除。这个设计是为了防止一个忘记取消的任务无限跑下去，合理。如果你需要更久，在过期前取消重建，或者用Desktop scheduled tasks。

**任务之间不会并发**。调度器检查到任务到期，会等Claude当前回合结束再执行，不会在Claude给你回答到一半的时候突然插进来一个定时任务。这个行为设计得很正确。

**没有补发机制**。如果到点时Claude正忙，任务等Claude空闲后触发一次——不是补发漏掉的所有次。

如果你想要能在没有活跃终端时跑的定时任务，得用Desktop scheduled tasks或者GitHub Actions的schedule trigger，那是另一套体系。`/loop`是会话级的轻量工具，定位很清楚。

---

## 社区的一些声音

Ultrathink回归之后，最直接的反馈是：以前那个"Claude感觉变笨了"的印象开始被纠正。GitHub上那个追踪质量下滑的issue被标记完成关闭，不是偶然。

关于中等投入作为新默认值，有人觉得正好，有人觉得某些场景下还是太保守——这个分歧其实挺正常的，不同任务密度的用户感受会不一样。好在现在有足够多的旋钮让你自己调，不需要接受一刀切的默认值。

`/claude-api`上线之后，反馈比较一致：有了它，第一次就能得到正确的实现，来回纠偏的次数明显少了。有人在Reddit上说"以前每次写streaming都要看一遍文档，现在直接写"。

`/loop`目前是最新的，早期反馈里有人碰到v2.1.71里找不到这个命令，Boris Cherny本人的回复是"重启一下刷新配置"——刚发布的内置技能偶尔有这种缓存问题，重启基本能解决。

---

## 写在最后

这一波让我感触比较深的，是内置技能这个方向越来越清晰了。

从`/simplify`（代码审查）、`/batch`（大规模迁移），到`/claude-api`（API上下文注入），再到`/loop`（定时调度）——Anthropic在把原本需要你切换工具、写脚本、查文档才能搞定的事，一件一件地往Claude Code里收。每一个内置技能解决的都是一个具体的、反复出现的痛点，不是为了功能数量堆砌出来的。

Ultrathink的回归意义不只是"那个功能又能用了"。它代表着努力控制体系整体上成熟了一层：默认中等投入保住了日常任务的速度，单次超控保住了需要深度推理时的质量上限，中间档位也有了，用户自己能根据场景做精准控制，不用一刀切。

如果说上几波更新是在扩展Claude Code的"触手"（Remote Control让它走出终端，HTTP Hooks让它主动联系外部系统），这一波更多是在精细化"脑子"的工作方式，和把常用工具封装成对普通开发者来说更友好的形态。

两件事都要做，缺哪样都跛脚。

你有没有用过Ultrathink感受到明显的质量差别？或者`/loop`用在了什么场景？欢迎留言说说。

---

**相关链接：**

* • 官方Changelog
* • GitHub Releases
* • 定时调度官方文档
* • Hooks官方文档

如果这篇文章对你有帮助，欢迎点赞转发，关注不迷路

五天，几个版本。Ultrathink回来了，努力控制体系长大了；/claude-api把API文档装进了会话；/loop让终端里多了一个定时调度器。每一件单独看都不算大，合在一起看，你会发现Claude Code离一个"完整的AI研发工作台"越来越近了。