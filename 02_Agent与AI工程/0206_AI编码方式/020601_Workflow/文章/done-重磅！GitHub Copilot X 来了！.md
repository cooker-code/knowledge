> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: 重磅！GitHub Copilot X 来了！
author: 玩转VS Code
date:
url: http://mp.weixin.qq.com/s?__biz=MzU1NjgwNTExNQ==&mid=2247495839&idx=1&sn=c1da199adecfa0ed9348ed8f3fabc067&chksm=fc3dce0bcb4a471ddb87f76527ed7b32ebfc5b6507d118388dcfc6e1ffaa1f6e4dbc4fa62543&mpshare=1&scene=24&srcid=032353rg834QQLYlZMrXXZDR&sharer_sharetime=1679530325834&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

公众号关注 “**玩转VS Code**”

设为 “星标”，每天带你逛最新技术！

上周，微软给 Office 全家桶都加上了 GPT-4 的功能，[并带来了焕然一新的产品 Microsoft 365 Copilot](http://mp.weixin.qq.com/s?__biz=MzU1NjgwNTExNQ==&mid=2247495811&idx=1&sn=1f5096ade687bdba34c6154e69810b3f&chksm=fc3dce17cb4a47017d44e04f51fdae012a00dbd47641b6b4caa7633b59c0690a3fc736350cf7&scene=21#wechat_redirect)。

而作为最早一个吃螃蟹的 GitHub Copilot，又怎能在这次浪潮中少了它的身影呢。

今天，它终于来了！

昨天晚上，GitHub 官方发推宣布，一款基于 AI 驱动的、跨时代代码编辑器 **GitHub Copilot X** 横空出世，将为开发者带来前所未有的编程体验！

已开通了 GitHub Copilot 的同学，可前往下方链接申请进入 waitlist。

申请内测：https://github.com/github-copilot/chat\_waitlist\_signup/join

上一代的 GitHub Copilot，在协助开发者完成编程任务时，便有着极其出色的体验。诞生至今两年，便自动生成了 46% 的代码，更将编码速度提高了 55%。

此次更新的 GitHub Copilot X，又给我们带来了哪些惊喜呢？请看：

1. **集成了 GPT-4（微软亲儿子，必须安排上）；**
2. **GitHub Copilot Chat（边写代码边跟 AI 对话）；**
3. **Copilot for Pull Requests（AI 协助处理 PR）；**
4. **Copilot for Docs（智能文档系统）；**
5. **Copilot for CLI（让命令行用起来更智能）。**
6. **Copilot Voice（直接语音生成代码，牛逼！）；**

下面，就让我们来逐个看看，对于一个开发者来说，这些功能到底能有多震撼。

### GitHub Copilot Chat

相信最近这些日子被 ChatGPT 和 Bing Chat 刷屏的你，对于这两款产品的真实实力已经没有存疑。

那么，如果把他们的功能，都加到 GitHub Copilot 上呢？

没错，本次 GitHub Copilot X 将在产品中内嵌一个聊天窗口，把 GPT-4 融合到实际开发场景，**并集成至 VS Code 和 Visual Studio 上**。

这个聊天窗口可不一般，除了更进行实时交互问答之外，它还可以完成诸如代码内容识别、报错信息显示、语音交流等操作。

通过这一功能，开发人员可深入分析和了解各个代码块的用途，快速生成单元测试，甚至还能一键修改 Bug，就问你猛不猛！

未来，或许我们不再需要一行行看代码、找 Bug、写测试了，而是直接改用 GitHub Copilot Chat 来迅速处理这些工作。

### Copilot for Pull Requests

维护过开源项目，或者用 Git 进行团队协作的同学应该知道，规范化提交 Pull Request（PR） 对于项目开发者来说，到底有多重要。

一个清晰简洁的 PR 描述信息，能让代码审查者一目了然，快速了解你的代码变动情况，减少项目合并出错的可能性，并提升沟通效率。

因此，Copilot 引入了 GPT-4 模型，试图让它通过动态提取与分析代码的变更信息，自动生成描述。

开发者人员只需在 PR 描述中插入标记，Copilot 便会自动识别，并进行扩展补写。

支持的标记，主要有以下几种：

* `copilot:summary`为 PR 生成一段摘要总结。
* `copilot:walkthrough`详细的更改列表，包括相关代码片段链接。
* `copilot:poem`写一首诗来描述本次改动。
* `copilot:all`自动生成以上所有内容。

具体操作如下所示：

随着后续产品的更新迭代，Copilot 还会新增名为 **Gentest** 和 **Ghost Text** 的两大功能。

Gentest：通过 AI 来识别 PR 中可能缺少的测试，并自动帮你构建与生成测试。

Ghost Text：在你编写 PR 描述或文档时，提供内容自动补全功能。

不仅如此，在你收到用户提交的 issue 时，如果没有比较好的解决方案，AI 会给你提供建议。审查代码觉得麻烦，AI 也能协助。

估计再过不久，AI 还要帮你自动调整代码、完善 PR、修复 Bug 了。

说白了，这货要开始抢你饭碗了...

### Copilot for docs

文档对于一个开发者来说，其重要性不言而喻。无论你是新接手一个项目，还是忘记某个 API 的具体使用方法。

这个时候，你都会殷切的盼望有一份优秀的技术文档能出现在你面前。

为了帮你更快定位文档内容，简单直接获取到想要的信息，Copilot for docs 应运而生。

用户能通过类似 ChatGPT 的界面，针对项目文档、常用代码等内容发起提问，即时获取答案。

你所需要做的，就是在输入框中，写下你想了解的问题，按下回车，Copilot 便会自动定位并返回包含在文档中的相关内容。

还有个比较厉害的地方，就是它能根据用户的不同编程水平，对该文档的了解程度，以及想要知晓的内容，返回不同的答案。

如果有需要的话，它也能在不同的第三方库文档之间来回穿梭，将其内容进行拼接，并给你返回结果。

有兴趣的话，不妨尝试把 Copilot for docs 变成你的知识库，相信会有意想不到的收获。

### Copilot for CLI

除了上面提到的处理 PR 请求与编写代码，命令行终端也是开发者日常工作必不可少的工具之一。

我经常喜欢在 iTerm 上安装各种各样的命令行生产力工具，这些工具在大多时间能帮我高效处理掉一些问题，但那些比较少用的，就会经常性忘记命令。

每当这个时候，我便总会输入 `help` 来获取关于该命令的更多信息。

对于功能较为复杂的命令行工具，还得时不时跑到 StackOverflow 上寻找正确用法。

如果有一个 AI 在身边，听明白我的诉求，帮我快速搞定一切，那就再好不过了。

于是乎，Copilot for CLI 带着它那 3 个 shell 命令来了。

这 3 个命令分别是：`??`，`git?`，`gh?`。

`??`可以作为任意 shell 命令的通用 goto，在该命令后面输入相关描述，Copilot 便会列出最适用该描述的具体命令。

比如，你不清楚如何列出所有 JavaScript 文件，只需要输入：

```
>  ?? list js file
```

AI 便会提供可参考的命令以及描述，如果不满意，还能进一步更改描述。

确定之后，在终端选中 `Run this command`，然后按下回车即可。

`git?`用于专门搜索以及调用`git`。

相比`??`, 它在生成 Git 命令方面要强大许多。

如果你明确要用的是 Git 命令，那可以优先选择使用这个。

`gh?` 结合了 GitHub CLI 命令的灵活性与查询界面的便捷性，强强联手，让搜索变得更加快捷、信息展示更为清晰。

内测申请：https://githubnext.com/projects/copilot-cli

虽然 GitHub Copilot CLI 大部分场景主要集中在 Git 以及 GitHub 上，但是与 AI 相结合之后，互动性与连贯性得到了进一步提升。这种方式，相信也能给其他的命令行工具开发者，提供一些参考思路。

### Copilot Voice

前几年，GitHub Copilot 刚放出来的时候，网上就有人讨论："AI 那么强，兴许后面我动动嘴皮子，它就能帮我写出想要的代码了"。

估计大家都没想到是，这一天来得竟如此之快。

Copilot Voice，一个极其具有突破性的编程工具，出现了。

用户通过与 GitHub Copilot 进行交谈，它就能立即开始编写代码，直接解放你的双手！

不满意的话，还可以接着说话，让它进行更改。

通过对话，Copilot Voice 可以完成：

* 代码跳转（跳转到 x 行、方法、函数）；
* 控制 IDE（开启 zen 模式、运行程序或其它 VSCode 指令）；
* 代码总结（可以问它：3-10 行代码，表示什么意思）；

所有的工作，张张嘴就能搞定。

内测申请：https://githubnext.com/projects/copilot-voice

Linux 曾经说："Talk is cheap, Show me the code."

但是这一次，不好意思，我全都要。

**将来的某一天，当你心血来潮，突然间想开发一个程序时，或许只需要喊一声 "嘿，GitHub！"，AI 便能帮你完成一切。**

### 写在最后

本次 GitHub Copilot X，围绕 AI 对话、Pull Request 提交处理、文档智能检索与阅读、命令行改造，到最后的语音生成代码，可以说是全方位颠覆了传统的编程方式。

当机器人能够准确理解人类自然语言，学会从零到一，完成项目的设计、开发、部署等工作时。**未来程序员这个群体，或许将跟电报员一样，成为某个曾经在历史上出现过的普通工种。**

事了拂衣去，深藏功与名。

**推荐阅读：**

* [真正的 ChatGPT 机器人，来了！](http://mp.weixin.qq.com/s?__biz=MzIwODE4Nzg2NQ==&mid=2650558940&idx=1&sn=743fd12041540cd294b22b65bf8a5b27&chksm=8f0e096ab879807cb2276cf6c2c1f833b551ea572db4f1d740de5b0a264dd6e3866b89e8167d&scene=21#wechat_redirect)
* [这些外企，还在招人！](http://mp.weixin.qq.com/s?__biz=MzIwODE4Nzg2NQ==&mid=2650558987&idx=1&sn=4b0b562a574ab689595d069d33cb1bbe&chksm=8f0e0abdb87983ab807dd1d1f14a758ddfa934c7ae68586b3c2fcf3603139c71b8c608f8c724&scene=21#wechat_redirect)