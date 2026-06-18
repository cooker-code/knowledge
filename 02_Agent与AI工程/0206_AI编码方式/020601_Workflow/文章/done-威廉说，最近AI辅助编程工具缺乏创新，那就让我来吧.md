> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: 威廉说，最近AI辅助编程工具缺乏创新，那就让我来吧
author: 祝威廉
date:
url: https://mp.weixin.qq.com/s?__biz=MzIyNzQyNzgxNQ==&mid=2247484964&idx=1&sn=cea4c3d4aad37b290ca8a80ea34316e5&chksm=e94f3ad5abf116dda6bc2db5eeef1e6810e3b7c4b301e68f8746c1db8de59372214409eb05e4&mpshare=1&scene=24&srcid=1030L7gUsXRCsdY2W5KenA4I&sharer_shareinfo=6eb57ab77eb411887d142e7f90cc7e71&sharer_shareinfo_first=6eb57ab77eb411887d142e7f90cc7e71#rd
---

前几天，我们给大家提供了第一个开胃菜：[RAG 终于到第四代了，Agentic RAG SDK 来了](https://mp.weixin.qq.com/s?__biz=MzIyNzQyNzgxNQ==&mid=2247484953&idx=1&sn=426e965b2525c5ed3493abe498f275f1&scene=21#wechat_redirect)

今天来我们的第二个开胃菜。

前言

Claude Code 从刚开始出现到现在，我喜欢创新其实就两个： subagent 和 内置 todo，再之后就没啥创新了，一些功能增强，比如Agent SDK 越来越完善。现在虽然每天更新都很勤快，但基本就是交互的一些小改进。 而其他工具，诸如 codex 以及后续一堆的跟进产品，几乎都毫无创新在追赶而已。

既然这些厂商都不给力，那么就让我来做一些创新吧。今天是第一篇，主要涉及 Code Agent 下一代交互方式。

下一代新交互方式：异步 SubAgent

在Vibe coding中，大部分用户普遍被Code Agent 的交付质量问题所困扰，其次就是无法真正的并行处理任务（CC 的 subagent 一般只能用于并行处理不相干的任务），并且即使用后通过worktree + 多terminal(tab) 也面沉重的 review 和 merge 负担。

auto-coder.chat 通过创新的异步 subagent 完美解决了这些问题。 auto-coder.chat 的 async subagent 具有三个特点：

1. 首先我们把 CLI （主agent）退化成了一个任务管理系统。

2. 人类第一次可以精确的配置 subagent 的运行时间，确保交付质量。

3. 人类review 完后， 可以让 主agent 同时自动合并多个异步subagent的结果。

让我们实际跑个例子来看看。

```
coding@auto-coder.chat:~$ /auto /async /time 10m /name async_drop_task  "在 @./src/autocoder/inner/async_command_handler.py 中添加一个新的子命令 drop, 然后删除相应的task"
```

其中:

1. /async 表示这是异步 subagent任务

2. /time 10m 告诉 subagent, 你要少跑十分钟，仔仔细细检查了再给我review,而不是写完就给我review.

3. /name async\_drop\_task 则表示当前的任务名称，需要保持唯一

回车提交后：

可以看到，系统已经提交异步任务，底部 status bar Async Tasks 那部分显示有一个任务在运行。你可以继续做其他的操作了。

你也可以查看任务列表：

也可以通过具体查看某个任务当前的状态：

还可以通过 cursor 来查看某个任务已经完成的变更，以及查看实时日志：

```
coding@auto-coder.chat:~$ !cursor ~/.auto-coder/async_agent/tasks/async_drop_task
```

打开后，可以关注着两个主要区域：

使用 cursor 主要方便你十分钟后 reivew结果，判断是否要做合并。

你可以同时提交多个任务，给每一个任务指定运行时间，并且不用担心任务之间是否会冲突。

一旦任务完成，并且觉得合适，就可以进行合并操作。具体做法为：

```
coding@auto-coder.chat:~$ /auto /merge async_drop_task
```

如果是多个任务，则按逗号分割即可：

```
coding@auto-coder.chat:~$ /auto /merge task1,task2,task3
```

如果多个任务之间有冲突，系统会自动解决合并冲突。

除了可以指定运行时间来提升AI的交付质量以外，我们还可以通过让多个模型同时运行相同的任务，比如：

```
/auto /async /model v3 /time 10m /name task1 "需求"/auto /async /model glm4-6 /time 10m /name task2 "需求"
```

上面的例子同时提交任务给到 v3, glm4-6 两个模型，然后他们会独立运行。最后运行结束后，你可以选择其中一个合并，也可以让主agent 自动选一个合并：

```
coding@auto-coder.chat:~$ /auto /merge task1,task2 -query "两个任务选择一个你认为最好的做合并"
```

这样就可以实现系统自动选择最好的实现来合并到主分支上。

总结

异步 subagent 的能力，完全重构了vibe coding的使用方式，对很多人来说，可能还会有使用习惯上改变，

但是我相信会是人机交互的一种落地形态之一，甚至可能是半年到一年内最好的交互形态。

我们充分利用了应用层scaling, 通过指定运行时间和多个模型，并且解决了合并代码的问题，极大的提升了自动化率和效果。相比传统 vibe coding,给效率带来一个质的变化。

附录：安装

我把一些创新功能放在 auto-coder.chat (cli 命令行版) 的社区版里了。大家可以安装后体验我后续提到的功能。执行下面的命令来安装 auto-coder.chat:

```
mkdir auto-coderuv venv --python 3.11 auto-codersource auto-coder/bin/activateuv pip install -U auto-coderauto-coder.chat
```

或者

```
# python 3.10 / 3.11 / 3.12 三个版本pip install -U auto-coderauto-coder.chat
```

如果执行顺利,应该就能进入界面：

我们推荐火山的 v3-1-terminus（当然效果最好还是 Sonnet 4.5 和 GPT 5）,可以先看模型列表：

```
/models /list
```

然后给指定名字的模型添加 api key:

```
/models volcengine/deepseek-v3-1-terminus <YOUR_API_KEY>
```

设置使用该模型：

```
/conf model:volcengine/deepseek-v3-1-terminus
```

现在可以开始跑前面的例子了，比如

```
/auto /async /time 10m /name try_try """我想实现....."""
```

特别注意

社区版目前并不开源代码，并且后续会引入和多新的探索和激进的功能。个人用户可以免费使用，且需要遵循安装包中的licence 要求。

对于稳定性或者商业有需求的同学，可以尝试基于auto-coder.chat 的商业版： https://aitocoder.com