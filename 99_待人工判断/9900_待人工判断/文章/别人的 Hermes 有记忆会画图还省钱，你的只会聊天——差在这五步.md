---
title: 别人的 Hermes 有记忆会画图还省钱，你的只会聊天——差在这五步
author: AI赋能说
date: AI赋能说AI赋能说
url: https://mp.weixin.qq.com/s?__biz=MzI3NjE4OTAyMg==&mid=2247488599&idx=1&sn=753bf7222dda35bf010fd0cf2e61fc3e&chksm=ea9d0a2c62fabd1759dae57270c847d8dd4d719d2166de82448e2447d87ef4a91f69ae485c7e&mpshare=1&scene=24&srcid=0427AhTZADpfeXJt9sIbDmQn&sharer_shareinfo=1b6ecdd6b3fa73beca350b520bf3a604&sharer_shareinfo_first=1b6ecdd6b3fa73beca350b520bf3a604#rd
---

装完 Hermes 只是买了辆车。没装配件的车，能开，但跑不快。

很多人装完 Hermes[1] 就开始用了。能聊天，能写代码，看起来挺好。但用几天就会发现几个问题：它不记得你昨天说了什么，它读不了网页，它画不了图，它每次对话都在烧 Token。

裸装 Hermes 是一个聪明但失忆的助手。满配 Hermes 是一个有记忆、能上网、能画图、还会省钱的 AI Agent。

差距在哪？在五套系统。

Hermes 是 Nous Research[2] 做的开源 AI Agent。GitHub 上 6 万多星。它和 ChatGPT 最大的区别是：它跑在你自己的电脑上，支持任意模型（OpenAI、Anthropic、Google、Ollama 都行），而且能自我进化——把你反复做的事变成可复用的 Skill。

但这些能力不是装完就有的。需要你一个一个配上去。

想了想，这就像买了一台裸机电脑。CPU 和主板有了，但没装内存、没接硬盘、没插显卡。能开机，但干不了正事。

下面是五套系统。按顺序装，每套解决一个问题。

### 第一套：给它一个身份

Hermes 默认是一个通用助手。什么都能聊，但什么都不精。

你需要告诉它：你是谁，你做什么，你希望它怎么配合你。

这件事通过一个叫 `SOUL.md` 的文件完成。它是 Hermes 的「人格文件」。你在里面写清楚角色定位、工作方式、沟通风格，Hermes 每次对话都会先读这个文件。

不会写？有现成的。agency-agents-zh[3] 这个仓库里有 211 个中文角色模板，覆盖工程、设计、营销、产品、金融等 18 个方向。还有 46 个针对中国市场的智能体模板，包括小红书运营、抖音投放、跨境电商。

挑一个最接近你需求的，复制过来改改就能用。

**操作：** 把选好的角色文件内容复制到 Hermes 配置目录下的 `SOUL.md`。下次启动 Hermes 时它就会按这个身份工作。

### 第二套：给它一个记忆

这是最值得装的一套。

Hermes 自带的记忆系统是一个纯文本文件，上限大约 2200 字符。它只在「觉得重要」的时候才写入。用几天就满了，旧的被挤掉。

Hindsight[4] 是一个专门给 AI Agent 做的记忆后端。它把每轮对话里的实体、事实、关系、时间戳自动提取出来，存成知识图谱。没有容量上限。下次对话时，Hermes 会自动从知识图谱里召回相关记忆。

装完之后的体感变化很明显。你不用每次都重复「我是做什么的」「我的项目叫什么」「上次我们聊到哪了」。它记得。

**操作：**

```
hermes memory setup
```

选 `hindsight`，向导会自动装依赖。然后去 Hindsight 控制台[5] 注册，拿一个免费的 API Key，填进去。

验证：

```
hermes memory status
```

看到 Hindsight 已激活就行。

### 第三套：给它一双眼睛

裸装 Hermes 读不了网页。你让它查个资料，它只能靠训练数据里的旧知识回答。

装上抓取和搜索工具，它就能看到整个互联网。

四个工具各管一块：

* Tavily[6]：AI 专用搜索引擎，每月 1000 次免费。搜索结果直接是结构化的，不用 Hermes 自己去解析网页
* DuckDuckGo：零成本兜底。Tavily 额度用完了，它顶上
* Jina Reader[7]：单页抓取。给一个 URL，返回干净的 Markdown
* Crawl4AI[8]：批量深度抓取。需要爬整个站点的时候用

**操作：**

Tavily 需要注册拿 API Key，填到 Hermes 的环境变量里。DuckDuckGo 不需要配置。Jina Reader 和 Crawl4AI 可以通过 Hermes 的 Skill 系统集成，写一个简单的 Skill 文件调用它们的 API 就行。

### 第四套：给它一张嘴和一双手

裸装 Hermes 只能输出文字。装上表达工具，它能说话，能画图。

* Whisper[9]：语音识别。支持 99 种语言。你对着麦克风说话，它转成文字喂给 Hermes
* Edge TTS[10]：语音合成。免费。Hermes 的回复可以读出来
* Fal.ai[11]：图片生成。接入后 Hermes 可以直接画图

装完之后，Hermes 从一个「只能打字的助手」变成了「能听能说能画的助手」。

### 第五套：给它一个省钱脑

用 Hermes 最大的隐性成本是 Token。每次对话、每次调用工具、每次读文件，都在消耗 Token。不监控的话，月底账单会吓你一跳。

三个工具解决这个问题：

Tokscale[12]：Token 用量监控。实时看全局消耗，按模型、按会话拆解。

```
npx tokscale@latest
```

启动后在终端里就能看到一个可视化面板。

RTK[13]：Rust 写的 Token 压缩器。它拦截终端命令的输出，过滤掉噪音，只把有用的信息传给 Hermes。一个 `ls` 命令的输出可能有几百行，RTK 压完只剩关键信息。实测能减少 60-90% 的 Token 消耗[14]。

```
brew install rtk  
rtk init -g
```

装完后 Hermes 的所有终端命令会自动走 RTK 压缩。你不需要改任何习惯。

Hermes Agent Self-Evolution[15]：用遗传算法自动优化 Hermes 的提示词和行为。它会不断试不同的写法，留下效果最好的。时间越长，Hermes 越省 Token，回答越准。

### 装完之后

五套系统装完，Hermes 从一个「能聊天的 AI」变成了：

* 有身份：知道自己是谁，怎么配合你
* 有记忆：记得你说过的每一件事
* 有眼睛：能搜索、能抓取、能读网页
* 有嘴和手：能说话、能画图
* 会省钱：Token 消耗降六到九成

这才是满配。

最后推荐两个生态入口，装完五套系统之后可以继续探索：

* awesome-hermes-agent[16]：一站式资源汇总
* hermes-ecosystem[17]：80+ 工具可视化地图

裸装和满配的差距，用过就知道。

参考资料：

* Hermes Agent 完整指南[18]
* Hindsight 记忆系统集成[19]
* RTK：Token 消耗降 80%[20]
* Hermes Agent 记忆系统对比[21]
* Hermes Agent 开发者指南[22]

Reference

[1] 

Hermes: *https://github.com/NousResearch/hermes-agent*

[2] 

Nous Research: *https://nousresearch.com/*

[3] 

agency-agents-zh: *https://github.com/jnMetaCode/agency-agents-zh*

[4] 

Hindsight: *https://hindsight.vectorize.io/sdks/integrations/hermes*

[5] 

Hindsight 控制台: *https://ui.hindsight.vectorize.io/connect*

[6] 

Tavily: *https://tavily.com/*

[7] 

Jina Reader: *https://r.jina.ai/*

[8] 

Crawl4AI: *https://github.com/unclecode/crawl4ai*

[9] 

Whisper: *https://github.com/openai/whisper*

[10] 

Edge TTS: *https://github.com/rany2/edge-tts*

[11] 

Fal.ai: *https://fal.ai/*

[12] 

Tokscale: *https://github.com/palaklive/tokscale*

[13] 

RTK: *https://github.com/rtk-ai/rtk*

[14] 

实测能减少 60-90% 的 Token 消耗: *https://madplay.github.io/en/post/rtk-reduce-ai-coding-agent-token-usage*

[15] 

Hermes Agent Self-Evolution: *https://github.com/NousResearch/hermes-agent-self-evolution*

[16] 

awesome-hermes-agent: *https://github.com/NousResearch/awesome-hermes-agent*

[17] 

hermes-ecosystem: *https://github.com/NousResearch/hermes-ecosystem*

[18] 

Hermes Agent 完整指南: *https://www.nxcode.io/resources/news/hermes-agent-complete-guide-self-improving-ai-2026*

[19] 

Hindsight 记忆系统集成: *https://hindsight.vectorize.io/sdks/integrations/hermes*

[20] 

RTK：Token 消耗降 80%: *https://madplay.github.io/en/post/rtk-reduce-ai-coding-agent-token-usage*

[21] 

Hermes Agent 记忆系统对比: *https://vectorize.io/articles/hermes-agent-memory-providers-compared*

[22] 

Hermes Agent 开发者指南: *https://lushbinary.com/blog/hermes-agent-developer-guide-setup-skills-self-improving-ai/*

**下方是赋能君的AI学习交流永久免费星球，想学习更多内容，欢迎扫码加入。**

🙌 如果你阅读到这里，说明我们对信息的认可区域是有一定交集的，可以说我们是同道中人，所以如果你有自认为不错的信息获取渠道，欢迎留言或者私聊我，谢谢。

都看到这里了，就给个关注吧👀：

喜欢我的文章，可以请你右下角顺手来一波点赞&在看&分享三连么👉