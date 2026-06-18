---
title: GStack：一个人就是一支AI团队
author: 柯一下
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxMzU1NDE4Mw==&mid=2458602281&idx=1&sn=ddcc7a3c53bc67d98a5c8112e4e150c0&chksm=8d6e9a24a3c3531273cb969cdb8c206ce5b0aaa40be42b80185f28346c90569c449b1ca2bde6&mpshare=1&scene=24&srcid=0409MrUiPjCYxOJiYbt5F9wX&sharer_shareinfo=a93fc3fdf608774c8ec35b24c2388686&sharer_shareinfo_first=a93fc3fdf608774c8ec35b24c2388686#rd
---

你对AI说"帮我写个功能"，它确实写了。你说"帮我看看有什么bug"，它确实看了。你说"帮我发布上线"，它也确实执行了。

但最后的结果总是哪里不对。

功能写出来了，但好像没解决真正的问题。Bug找出来了，但都是些不痛不痒的。代码发布出去了，但某天深夜你突然被生产事故叫醒。

你忍不住想：AI都这么强了，怎么还是感觉像在跟一个刚入职场的实习生干活？他确实听话，但总是少了点什么。

少了**专业度**，少了**判断力**，少了**不同场景下的不同思维方式**。

而YC总裁Garry Tan开源的GStack，就是来解决这个问题的。

---

01

AI也会"上下文污染"

也许你会问：AI也会"上下文污染"？

是的。Garry Tan在GStack的README里直接写了传统AI编程的痛点：

> "The agent takes your request literally - it never asks if you're building the right thing"
>
> "Review my PR gives inconsistent depth every time"
>
> "Ship this turns into a long back-and-forth about what to do"

翻译过来就是：AI把你的需求理解得太字面了，永远不会反过来问你是不是在做对的事。每次让AI review代码，深度都不一致。让你发布个东西，结果变成没完没了的来回拉扯。

为什么会这样？

因为**AI跟人一样，也会"上下文污染"**。当你让它同时做战略思考和代码实现时，它的注意力被分散了。

这就好像让一个产品经理同时写代码、同时做测试、同时发布上线。结果是什么？每件事都做得马马虎虎。

Garry Tan说了一句特别到位的话：

> "Planning is not review. Review is not shipping. Founder taste is not engineering rigor. If you blur all of that together, you usually get a mediocre blend of all four."

这就是问题所在。**我们一直把AI当作一个"全能助手"，但全能往往意味着全不能。**

---

02

GStack：给AI装上"档位"

Garry Tan没有试图做一个更强的"全能AI"。他做了一件更聪明的事：**让AI可以切换不同的"脑"。**

这就是GStack的核心思路。它是一套**角色切换系统**，本质上是专业化的提示词工程。

来看看最核心的5个技能：

* `/plan-ceo-review：创始人脑。用品味、野心、用户同理心思考。先问"产品到底是做什么的"，而不是字面理解需求。`
* `/plan-eng-review：工程经理脑。关注架构、数据流、边界情况、测试方案。"让想法可构建"，而不是"让想法变小"。`
* `/review：偏执狂工程师脑。专门找那些"能通过CI但在生产环境爆炸"的bug。`
* `/qa：QA工程师脑。给AI装上眼睛，登录、点击、截图、检查断裂，60秒完成一次完整QA。`
* `/browse：浏览工具脑。给AI装上眼睛，让它能真看页面、点流程、跑测试。`

每一个技能对应一个**专业的脑**。

Garry Tan说：

> "I want explicit gears."

就像汽车有档位一样，AI编程也需要档位。你不能用一个档位跑完所有路。爬坡用一档，高速用五档，倒车用倒档。

GStack就是给AI装上了档位。

---

03

效果：48小时，9700 Stars

GStack有多火？

上线48小时，GitHub上就拿到了**9700多个Stars**。这对于一个工具类产品来说，是惊人的速度。

但更惊人的是它带来的改变。

Garry Tan举了个例子：

如果你说"让卖家上传商品照片"

* **弱AI：加一个文件选择器，保存图片到服务器。完成。**
* **CEO模式：先问：照片上传是真正的功能吗？也许真正的功能是"帮助卖家创建能卖出去的列表"。能否从照片识别产品？能否自动抓取规格和定价？能否自动生成标题和描述？能否建议最佳主图？**

这就是区别。弱的AI你说什么它做什么，强的AI会反过来问你什么更重要。

再看QA测试。

以前你要手动打开浏览器，点来点去，截图对比，眯着眼睛看布局对不对。

GStack的`/qa`技能，18个工具调用，大约60秒。它注册了一个测试用户，导航到每个修改过的页面，截图，阅读它们，检查控制台错误，验证API。这是一次完整的QA通过。

Garry Tan说：**"我没有打开浏览器。"**

**AI终于有眼睛了。**

---

04

这不是升级，是进化

Garry Tan在项目里特别强调了一句话：

> "This is not incremental improvement. That is a different way of building software."

我特别认同这句话。

过去我们经历了几个阶段：

* **第一阶段：AI作为代码补全。GitHub Copilot。**
* **第二阶段：AI作为对话助手。Claude Code、Cursor。**
* **第三阶段：AI作为Agent自主执行。**
* **第四阶段：AI作为"AI软件公司"。**

GStack代表的是第三到第四阶段的过渡。它是一组可以并行工作的专家，而不只是单个能干的助手。

想象一下：

用GStack官方的Conductor工具同时运行10个Claude Code会话。每个会话在不同分支上做不同的事。一个做QA，一个review PR，一个实现功能，其他做别的分支。

**一个人，就是一支团队。**

---

GStack的核心理念，本质上是软件工程中**关注点分离**原则在AI时代的延伸。

以前我们说"单一职责"，说的是一个类一个功能。现在我们说"AI也要分工"，说的是不同阶段用不同的脑。

关注点分离是软件工程最核心的原则之一。好的架构会把不同的职责分开，让每个模块只做一件事。GStack做的也是类似的事——只不过是把AI的"职责"分开。

当你让AI切换"CEO脑"、"工程师脑"、"偏执审查脑"、"发布机器脑"时，它不再是你的助手。

它是你的**执行团队**。

而你，终于可以从"具体怎么做"的细节里抽身出来，去做真正重要的事：**决定做什么，以及为什么做。**

这就是从手工编码到AI Coding的新阶段。

这不是更快的马车的故事。这是汽车取代马车的故事。

---

> 有人可能会问："AI真的懂商业判断吗？""10个Claude同时跑，谁处理merge conflict？"
>
> 说实话，这些问题没有标准答案。GStack提供的是**思维框架**，不是魔法。它的价值不在于AI变成了CEO，而在于提醒我们：什么时候用什么脑。
>
> 剩下的，是我们自己的判断了。

---

**你试过GStack了吗？感觉如何？**

**如果你身边还有朋友把AI当"打字机"用，转给他，他会感激你。**