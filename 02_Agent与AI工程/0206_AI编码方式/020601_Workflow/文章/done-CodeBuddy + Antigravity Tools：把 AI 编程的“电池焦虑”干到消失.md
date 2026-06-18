> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: CodeBuddy + Antigravity Tools：把 AI 编程的“电池焦虑”干到消失
author: 小虎AI生活
date:
url: https://mp.weixin.qq.com/s?__biz=MjM5OTY5MzAyOQ==&mid=2650872306&idx=1&sn=d3157dfa556e1820845876e01db745a3&chksm=bd3a2c5e5ae4d19abdf5f1bd5124ac25b522db0a4d52ceafe2c635fdb5084ce8d1c1135ac1e2&mpshare=1&scene=24&srcid=0115tfCvr4kDdL0EZ7X4UM7O&sharer_shareinfo=03c72290e5712b6e0f03ccb64d754494&sharer_shareinfo_first=03c72290e5712b6e0f03ccb64d754494#rd
---

点击上方蓝色字「小虎AI生活」>右上角...>设为星标

> 我是小虎，浙江大学计算机本硕，专注用AI编程 + AI教育赋能超级个体。

承认吧，你缺的从来不是“更努力写代码”。你缺的是一块永远不断电的大电池。

我见过太多人把 AI 编程当成“省时间”的工具。 真相是：它是一套新的生产关系。 谁先把“算力入口”变成自己的基础设施，谁就先拿到工业化的一人公司。

这篇文章我讲一个反常识的组合：CodeBuddy + Antigravity Tools。 它能让你同时享受国内最顺手的 IDE 体验，又把国外顶级模型的额度调度变成“可续杯的电池”。

教程在文章的最后，请耐心往下看。

## 第一层：现象——你以为你在用模型，其实模型在“用”你的积分

这几个月我用 AI 编程的体感非常割裂。

一边是“真的好用”：一句话写完脚手架，几分钟跑通业务链路。

另一边是“非常反人类”：

* 你刚进入状态，积分没了。
* 你还没搞清它在干嘛，操作已经做完了，账单也结完了。
* 你甚至不知道：到底是你在写代码，还是它在“烧”你的余额。

这就是当下 AI 编程的核心矛盾： 能力在飞涨，续航在崩塌。

如果续航解决不了，AI 编程就只是一个“昂贵的玩具”，而不是生产力。

## 第二层：认知翻转——真正的高手，不是会写 Prompt，而是会“配电”

我在浙大玉泉读书那会儿，写过汇编，也被 C++ 的指针折磨过。 那时你想跑得快，就得理解 CPU、内存、缓存。

今天 AI 编程也一样。 你想跑得稳，就得理解：

* 你的 IDE 是“工作台”；
* 模型是“发动机”；
* 额度、账号、路由，是“供电系统”。

大多数人只盯着发动机有多强。 我更在意供电系统是不是可控。

一旦你能把供电系统抓到自己手里，你就从“用户”变成了“运营商”。

## 第三层：底层逻辑——两位黑马：CodeBuddy 负责“顺手”，Antigravity 负责“强悍”

### 1）CodeBuddy：国内首屈一指的 AI 编程工具，已经进入 4.x 的成熟期

我对 CodeBuddy 的评价很简单：它是我见过最懂国内开发者生态的 AI 编程工具之一。

它的优势不是“聊天更会说”。 它的优势是：把开发者真正用得到的东西，做成了顺滑的产品。

* 腾讯生态内置工具很强：Supabase、CloudBase、EdgeOne Pages、Lighthouse 这类集成，对国内用户是实打实的效率提升。
* MCP 生态丰富：很多实际工作流能直接接上，少折腾。
* 版本成熟：走到 4.x，稳定性、体验、工程化能力，都已经不是早期产品能比的。

你把它当成“一个 AI 对话框”，就浪费了。 更准确地说，它是一套“可插拔的 AI 开发操作系统”。

### 2）Antigravity（ag）：AI 编程工具里的黑马，强在“模型资源”和“能力上限”

如果说 CodeBuddy 像一台国产高配工程车，顺手、适配、能跑复杂路况。 那 Antigravity 更像一辆性能车：

* 模型选择多、更新快：Claude、Gemini 这类最顶级的模型家族，常常能更早用到更强版本。
* 能力上限高：复杂推理、长上下文、跨文件改造、工具链协作，确实更有“猛劲”。
* 更像一个“模型入口”而不是“单一 IDE”：你可以把它当成资源池。

但问题也在这里： 资源池很强，不代表它用起来顺。

## 第四层：真实吐槽——两边我都爱，但都不完美

我现在常用的就是 CodeBuddy 和 ag。 但它们各有各的“难受”。

### CodeBuddy 的难受：你被迫在模型里做选择题

* 目前我体感最好用的是 default（Claude 4.5）。 但大模型再强也有缺陷：会误改、会漏改、会自信胡说。
* default 的积分系数是 2.2。 我订阅号每天赠送 100 积分，没用几次就耗没了。 很多时候“啥也没干成”，积分先没了。
* GPT 5.1 / 5.2 的问题： 它们经常埋头干活，不告诉你在干嘛。 一个操作结束，积分耗光，你还莫名其妙。
* Gemini 2.5 / 3 的问题： 积分消耗不多，但油嘴滑舌。 把你的代码搞坏，还振振有词。 你让它别动代码，它偏要动给你看。 像一个“死都不认错、坚决不改”的痞子。

### ag 的难受：国外工具，强但不够顺

* 经常会出错，链路稳定性不如国内工具。
* MCP 和集成工具的体验，不如 CodeBuddy 方便。
* 账号、额度、切换、封禁，管理成本也高。

所以问题来了： 有没有可能鱼和熊掌兼得？

有。

## 解决方案：ag tools 出现之后，“供电系统”终于可控了

ag tools（准确说，是开源项目 Antigravity-Manager）干了一件非常关键的事：

它把“多账号管理 + 协议转换 + 智能请求调度”做成了一个本地网关。

你可以把它理解成： 把不同厂商的 API 全部翻译成一种统一语言，并且还能自动换电池。

我去看了它的仓库介绍，核心亮点非常硬：

* 统一协议：兼容 OpenAI、Anthropic（Claude）、Gemini 原生接口，做协议转换与中继。
* 账号仪表盘：实时监控配额、健康状态，并推荐最佳账号。
* 自动重试与轮换：处理 429/401 等错误，静默轮换账号。
* 模型路由中心：支持正则映射、分组、降级路由，把“用哪个模型”变成可控策略。
* 多模态与大请求体：支持大体积（如 100MB）请求，图像生成/识别也能打。

一句话： 它不是“又一个聊天工具”，它是你的 AI 供电系统。

我之前也写过一篇[白嫖 Claude Opus？Antigravity“无限续杯”秘籍大公开！](https://mp.weixin.qq.com/s?__biz=MjM5OTY5MzAyOQ==&mid=2650872251&idx=1&sn=49db239633e4fdf34d0b97620ac55379&scene=21#wechat_redirect)，讲过如何把 ag 的资源搬到本地编程环境里。 今天这篇，是把这件事进一步做成“日常可用”的工作流。

## 缘起：学习群里一句话，把我脑子里的开关拨开了

事情来自我 AI 编程学习群里一个学员。 他随口提了一句： “CodeBuddy 能够修改内置大模型了。”

我当时第一反应是： 还有这么好的事？

第二反应更狠： 那如果我能配上 ag tools，让 ag tools 再连上 ag，再配合多几个 Google 账号——这不就是一个无限续航的大电池吗？

思路一旦出现，行动就只剩 SOP。

## 实操：把 ag tools 接进 CodeBuddy（核心 SOP）

下面这段非常朴素，但非常关键。

1）先把 Antigravity-Manager（ag tools）跑起来

* 下载并安装（Windows/macOS/Linux 都有）。

* 在工具里导入你的账号（支持批量导入、迁移、封禁检测等）。

* 打开它的 API 代理 / 本地反代 服务，拿到两样东西：

+ Base URL（通常是本机 127.0.0.1 的某个端口）
+ API Key（工具生成或你设置的密钥）

2）在 CodeBuddy 的配置 JSON 里新增一个自定义模型 Provider

学员给了一个 URL 指引，我照做：

* 打开 CodeBuddy 对应的 JSON 配置文件，通常在 C 盘用户目录的。CodeBuddy 的 models.json（C:\Users\xxx\。codebuddy\models.json），如果没有就自己新建一个

* 增加 baseUrl、apiKey、model 名称（按你的 ag tools 路由来填）

* 重启 CodeBuddy

3）验证：切换到新模型，问一句“你是什么大模型？”

我当时怀着期待打开模型切换。 果然，出现了一个新的自定义大模型。

我切过去，第一句就问： “你是什么大模型？”

它老老实实地告诉我。 我当场就笑了：成功了。

4）把常用模型一次性配齐

随后我把 ag 里这几个常用大模型都配置上了：

* Claude 4.5 Opus
* Claude 4.5 Sonnet
* Gemini 3 Pro High
* Gemini 3 Flash

从此我的工作流变成： CodeBuddy 负责体验与工程化，ag tools 负责续航与调度，ag 负责模型能力上限。

## 这套组合为什么值钱：你终于拥有“可复制的 AI 产能”

很多人还没意识到： AI 编程真正的门槛已经不是“会不会写代码”。 而是：你能不能把产能变成 SOP，变成流水线，变成可复制的交付。

当你把供电系统稳定下来，你就能做三类最现实的商业实验。

### 实验 1：内容工厂——把公众号从“灵感创作”升级为“工业产线”

用 CodeBuddy 产出：

* 选题拆解
* 结构大纲
* 资料整理
* 标题 A/B

用 ag tools 保障：

* 长文推理不断电
* 多模型互证减少胡说

结果是： 你不是更勤奋，你是更工业化。

### 实验 2：小工具交付——把“写脚本”变成可卖的产品

不是每个人都要做 SaaS。 但每个人都能做“一个解决具体问题的小工具”。

例如：

* 文章批量排版器
* 视频字幕清洗器
* 知识库整理器

当你有稳定续航，你就能把交付从“做一次”变成“复刻十次”。

### 实验 3：训练营/社群 SOP——把你的工作流卖给更多人

你会发现： 大家缺的不是工具。 大家缺的是“能跑通的工作流”。

把这套组合沉淀成 SOP：

* 安装
* 配置
* 模型选择策略
* 复盘与纠错

你就能把自己的产能，变成一套可传递的杠杆。

## 最后一句冷峻的提醒

这套方法本质是在做“本地中转与调度”。 请尊重开源协议与服务条款，只用于学习与提效，别把聪明用错地方。

旧时代拼的是加班。 新时代拼的是基础设施。

你要做的不是找一个更强的模型。 你要做的是：把模型变成你自己的电池。

 （如果你也想把这套 SOP 跑通，评论区告诉我你卡在哪一步。)

都到这里啦，记得点赞、点爱心、转发。

小手一赞，年入百万+

更多阅读：

* [被 110 人点赞的"上瘾帖"背后：Google Antigravity 正在悄悄颠覆编程的"勤奋价值观"](https://mp.weixin.qq.com/s?__biz=MjM5OTY5MzAyOQ==&mid=2650872293&idx=1&sn=c925b27e406baf5d8598867d0e34a41d&scene=21#wechat_redirect)
* [职场效率王炸！有这个AI神器在手，下班比同事遛的都快](https://mp.weixin.qq.com/s?__biz=MjM5OTY5MzAyOQ==&mid=2650872280&idx=1&sn=dddffb44da35de009067f14d8b8c80e1&scene=21#wechat_redirect)
* [2025 年终复盘：五十知天命？不，五十始新生！一位 70 后程序员的 AI 觉醒实录](https://mp.weixin.qq.com/s?__biz=MjM5OTY5MzAyOQ==&mid=2650872286&idx=1&sn=9ce70cd287f937035e437d498e27d34f&scene=21#wechat_redirect)
* [白嫖 Claude Opus？Antigravity“无限续杯”秘籍大公开！](https://mp.weixin.qq.com/s?__biz=MjM5OTY5MzAyOQ==&mid=2650872251&idx=1&sn=49db239633e4fdf34d0b97620ac55379&scene=21#wechat_redirect)
* [独立开发者的财富密码：用 AI 编程扫描竞品差评，找到那些年入百万的“烂软件”](https://mp.weixin.qq.com/s?__biz=MjM5OTY5MzAyOQ==&mid=2650872232&idx=1&sn=ae1e104db4f2c9e457bb660e3950bef7&scene=21#wechat_redirect)
* [从claude 到AG+CodeBuddy：我的AI编程工具链进化论（附独家避坑指南）](https://mp.weixin.qq.com/s?__biz=MjM5OTY5MzAyOQ==&mid=2650872222&idx=1&sn=1b021f824ec2ce304a83e4354886e74c&scene=21#wechat_redirect)
* [别再学编程了！AI时代，你那些“不值钱的瞎想”才是真金白银](https://mp.weixin.qq.com/s?__biz=MjM5OTY5MzAyOQ==&mid=2650872217&idx=1&sn=5df25c3ffaf380e10f326efbb82ce518&scene=21#wechat_redirect)
* [深度复盘 Google Antigravity：当初嫌它太超前，现在才懂它的香](https://mp.weixin.qq.com/s?__biz=MjM5OTY5MzAyOQ==&mid=2650872213&idx=1&sn=4698dbc260a7275a85dff2ec0127a24e&scene=21#wechat_redirect)
* [21 天，70+个产品落地：这场 AI 编程实验，让我看到了普通人的无限可能](https://mp.weixin.qq.com/s?__biz=MjM5OTY5MzAyOQ==&mid=2650872203&idx=1&sn=52c47951535eb531052aac8e6c135bbb&scene=21#wechat_redirect)
* [别光羡慕 NotebookLM 了，我用 AI 陪我写代码，三天“复刻”了它的播客功能](https://mp.weixin.qq.com/s?__biz=MjM5OTY5MzAyOQ==&mid=2650872192&idx=1&sn=3388d1bffb764530ea92fbc50ea7fb95&scene=21#wechat_redirect)

Hey，大家好！我是小虎，浙江大学计算机本硕，专注用AI编程 + AI教育赋能超级个体。

每天2条朋友圈，分享AI干货。（很多不方便公开讲的都在朋友圈里）

关注我，后台回复 AI编程 领取 AI编程知识库大礼包（持续更新中）

👇👇👇扫码+V，链接我👇👇👇

点个「爱心」吧？👇