> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Subagent - Claude Code的王炸
author: 黄博聊AI量化
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk3NTk0NzU3OA==&mid=2247483675&idx=1&sn=93159a8aa017a5a7f86a9f19c9862ead&chksm=c5f7f4a0fd85e322c480f05da3dcaceb907dd40488d8d5fa6b242567388771f13654c32a2e07&mpshare=1&scene=24&srcid=1114hyhuUveWrc3yBGZdwshR&sharer_shareinfo=fe9879059d98574df5fcba6a5a43faa6&sharer_shareinfo_first=fe9879059d98574df5fcba6a5a43faa6#rd
---

大家好，我是老黄。

大家都知道 Claude Code 有个 subagent 功能，但你可能会说：“Claude Code 本身不就是一个 coding agent 吗？为什么它还要带一堆 subagent 呢？”

这个问题是一个很好的问题。

今天就跟大家讲一下，这个 subagent 它到底在解决一个什么样子的问题。

为了把这件事情讲清楚呢，我还专门画了一堆图，咱们就直接看图说话。

#### 没 Subagent 的痛点：被“塞爆”的上下文

Claude Code 的 subagent 功能，主要解决的是大模型的上下文管理问题。

大家都知道，大模型的交互本身是有上下文长度限制的，像 Claude Code 它是有 200k，也就是20万个 Token。

那通常当我们没有 subagent 的时候，我们交给大模型的任务，就会遇到这么一个问题。

比如说，你看，你给大模型一个任务，它通常可能会：

1. 先去网上搜索一些相关的资料；
2. 然后他会抓取网页、读取网页，从这里面获得信息；
3. 他可能会重复这个过程很多次；
4. 在这个过程中，他可能还会调用 MCP 这类的工具；
5. 最后，他开始更新代码，然后给你一个结果。

比如下面这张图展示的这样：

但这个过程中，由于他读了大量的文档，这些文档其实是“塞爆”他上下文的罪魁祸首。你会发现，如果你在做完这个任务之后，去看他的那个 context，这 context 基本上就被 MCP 还有他自己的一些 memory 给塞满了。就是因为他读了一堆文档。

那这个时候，如果你紧接着再问他一个问题的话，你会发现，他可能会出现一些答非所问的情况。为啥？因为上下文已经爆了。

比如这样：

#### Subagent 如何解决难题？解放“主线程”！

Subagent，就是为了解决这个上下文管理的难题，而出现的一个方案。

有了 subagent 之后呢，情况是这样的。

比如说，你同样地让 Claude Code 去给你做一个任务，但是在这个过程中，你跟他说：“使用 **python-expert**”。（就假设我们已经定义好了一个叫做 `python expert` 的 agent）

对，那这个时候会发生什么事情呢？

就是 Claude Code 他还是会去做这个任务，但他会说：“`python expert`，请你去帮我做这个任务。”

接着，这个任务会给到一个叫 `python expert` 的 subagent。

比如这样：

然后，还是刚才那一套流程（搜索、读取、分析...），跑下来之后呢，`python expert` 无论是完成任务还是没完成任务，他都会给 Claude Code 的一个 **summary**，也就是关于这个任务他完成得怎么样的的一个报告。

那么在这个过程中，所有消耗的上下文，都是 `python expert` 的上下文，而不是 Claude Code 这个 **“主线程”** 的上下文。

他主线程消耗的上下文，只是在跟 `python expert` 进行沟通而已。对，那么你会发现，他的上下文其实根本没有太多消耗。

这样一来，当你在给他一个新的任务的时候呢，他还是可以很好地执行。并且，如果你让他去分发给其他 subagent，那基本上不会对主线程进行任何的堵塞，在整个跟 coding agent 的交互过程中，包括让他做任务的时候，体验会更丝滑。

#### 接下来聊什么？

那其实关于 subagent 呢，还有很多可以聊的。

比如说，你可以用一个叫 SuperClaude**的工具，它里面已经定义好了各种 subagent 的原设，这样你就可以即加即用。**

下一期，我会跟大家展示一下：

* 怎么样调用 subagent？
* 以及怎么去看自己的上下文消耗了多少？
* 然后在这个上下文的管理过程中呢，有什么比较好的方法？

（别忘了**点赞、在看、分享**三连！）