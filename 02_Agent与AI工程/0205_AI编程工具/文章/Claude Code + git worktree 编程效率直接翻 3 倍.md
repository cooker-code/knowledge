---
title: Claude Code + git worktree 编程效率直接翻 3 倍
author: 碳基 Agent
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIyMTUyMTU5Mg==&mid=2247484137&idx=1&sn=fb122ef222928a7c46ec26131c3e9d7e&chksm=e9796154ed29cb29b70c0135af4b264689c54cf2e51de961cae08f46a091e75b3c8f26f19453&mpshare=1&scene=24&srcid=1206pA4DnNydQdzIxxDYarcq&sharer_shareinfo=8dc51b3fcba74b819670b8ca735bb13e&sharer_shareinfo_first=8dc51b3fcba74b819670b8ca735bb13e#rd
---

最近需要大规模重构现有的代码，由于历史原因，要从 monorepo 多packages 模式切换到单一 repository ，同时每天还要处理一些零散的线上需求。

为了确保业务稳定，避免引入不可控的 bug，线上需求是基于 staging 或者 master 分支开发，而重构任务由于未经充分测试，所以是基于 develop 分支开发。

两个分支由于代码差异巨大，每次切换分支，都需要重新 install & build，而且还需要经常使用 git stash 保存临时代码，后来我发现 claude desktop 是通过 git worktree 来解决这个问题的，于是我就在我的工作流中引入了 git worktree。

原本的 workspace 只处理与线上需求相关的任务，Cursor + iTerm + Claude Code + Figma MCP，一套工具下来，几乎可以胜任绝大部分的任务。

而基于 develop 分支的重构任务，通过 

```
git worktree add ../refactor-dir -b refactor/monorepo
```

然后就在 refactor-dir 目录下实现并提交重构任务代码，为了区分原本的工作流，我就用 VS Code + Mac 自带 Terminal + Claude Code。

两套工具对应两个任务环境，不会在焦头烂额的时候输错 Claude 提示词，同一个环境如果需要多开 sessions，直接在 Terminal 中开多个 tab 即可，完全不会干扰。

与此同时，对于不熟悉的方法、库、概念，我也会同步地和 Claude 不停地交流，这部分可以通过 Claude Desktop 来实现，它本身自带 worktree，所以也不会干扰到之前的两套工具。

最近看到 Gabirel 的一个 Youtube 视频，一个高中辍学生通过和 ChatGPT 不停对话，通过递归学习的方式，一路走到 OpenAI 人工智能研究科学家的故事，深受启发。

拿我熟悉的前端开发举例，我现在经常会直接问 ChatGPT 或者 Claude 一连串的问题，比如 React 为什么需要状态管理？状态管理本质是在解决什么问题？那些流行的状态管理库为什么要那么设计？需要平衡哪些因素？......

这个问题清单会很长很长，直到问到我真正懂的地步，我参考的正是 Gabirel 的递归学习方法，他在刚学习大模型时，啥也不懂，线性代数、微积分都没学过，没关系，他上来就问学习大模型需要哪些知识，建议先做一个什么 demo 来入门。

ChatGPT 会给他一个很简单的 demo，于是他开始和 ChatGPT 一起 debug 直到把代码跑通，然后再针对每一部分代码穷追猛打式地问  ChatGPT，如果 ChatGPT 的回答他看不懂，就让 ChatGPT 再换一种方式讲解，比如画流程图、给示例代码、或者用 12 岁小孩子能听懂的语言讲。

他就是通过上述方法，自顶向下地学完了大模型所需要的知识，而我们传统的教学方式是先学习一堆的数学和编程知识，然后再往上慢慢学习如何开发应用程序，Gabirel 的递归学习方法刚好翻过来。

于是我现在养成了一个习惯，想到什么问题直接拿起手机或者电脑问 ChatGPT 或者 Claude，网络条件不好的时候问 Qwen，这种随时随地问 AI 的习惯是 Gabirel 极力推荐的。

AI 时代，我们的学习方式其实已经在悄悄发生变化，可能绝大部分人还不自知。

最近因为工作的需要，面试了不少前端开发者，很多人都还只是用 AI 来熟悉代码，写技术文档，然后手动画 UI ，手动写大量模板代码，殊不知这一切已经完全可以用 AI 来实现了。

未来已来，只是尚未流行。以前听到这句话，只觉得鸡汤味很浓，而如今我只有敬畏。