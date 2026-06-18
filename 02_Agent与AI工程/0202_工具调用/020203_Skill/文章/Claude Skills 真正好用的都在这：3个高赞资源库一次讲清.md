---
title: Claude Skills 真正好用的都在这：3个高赞资源库一次讲清
author: 噪点noisepoint
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI2NTAzNDM0Ng==&mid=2247484645&idx=1&sn=50befd9f9340cc3f33a7bca68053fe93&chksm=ebbe41d78dac9dc97619bede52b46589fe1fc75173a40271f7c60da886b31af3e8984c94002a&mpshare=1&scene=24&srcid=0106SvozBaoCEkzcHb3G356P&sharer_shareinfo=34279643a7b75839f71b870ab5e59551&sharer_shareinfo_first=34279643a7b75839f71b870ab5e59551#rd
---

大家好，我是噪点

要说当下Agent圈最火的，无疑就是claude skills。这个可复制、可迭代、可迁移的标准技能包，受到越来越多人的欢迎和使用。只需让AI调用skills，就能把沉淀好的解决对应问题的工作流带出来，自动根据sop执行任务。

但很多朋友自己创建的skills效果没那么好，简单用了几次就不用了；再就是因为不知道有哪些skills，所以不清楚在什么时候和场景可以用skills来提效，以及到底能解决什么实际问题？

今天就给大家分享下市面上一些常见、高赞的skills项目资源库，能很好解决上面提到的问题，极大提效；本文主要会从skills资源库使用、skills资源库推荐 2部分展开；看完这篇基本也对市面上90%以上的skills都了解了。

**skills资源库使用**

现在skills在claude code、codex里都已支持使用，而且都是通用的。

安装使用方法非常简单，直接在skills资源库里找到对应skills，将skills文件夹下载下来：

如果想Claude code里使用，就将文件夹移到 ~/.claude/skills/

如果想codex里使用，就将文件夹移到~/.codex/skills/

这样就安装好了，在cc或codex里直接说skills名字显示调用就行，自动调用触发不怎么准；如果想要查看安装了多少skills可以直接通过/skills或自然语言询问即可；

下载skills文件夹方法：如果是skills聚合网站一般都有直接下载按钮，点击即可；如果是在github里，直接点击项目主页代码按钮，下载zip压缩包，里面找到skills文件夹即可；

**skills资源库：anthropics/skills**

anthropics/skills：

地址：

https://github.com/anthropics/skills?tab=readme-ov-file

介绍：这是claude的官方skills仓库，里面内置了16个skills，涵盖设计、办公、开发等；

推荐skills：

文档办公4件套（docs skills、pdf skills、pptx skills、xlsx skills）：

可以直接在cc或codex里进行相关文档操作，比如pdf 内容提取修改、编辑分析word内容、ppt创建修改、excel表格数据分析及可视化；算是打工人接触最多的场景了，如果能把这些skills发挥好，日常效率绝对飞起；

frontend-design：前端设计skill；可以生成各种风格、高颜值、交互丰富的UI设计，而不是统一的AI味设计；非常适合vibe coding产品时使用；

Skill- creator：帮你创建skill的skill，类似元skill；可以简单理解为我们在创建skills的脚手架，在自己创建skills时调用它，ai会通过问答交互的形式获取相关信息，直接帮我们生成对应skill；

**skills资源库：SkillsMP**

SkillsMP：

地址：https://skillsmp.com/

介绍：专业的skills 市场，应该是我目前发现的收录skills最多的市场，目前已经收录4.4w多个skills；可以按分类、热度等进行skills搜索筛选，支持直接下载skills文件夹；

推荐skills：

code-reviewer：可以对代码自动化分析审查；包括审查拉取请求、提供代码反馈、识别问题等，确保代码质量符合标准；在让AI写完代码提交之前，调用这个skill来进行一遍审查，bug率能降低不少；

test-driven-development：TDD测试驱动开发；先写测试用例，观察它是否失败，然后用最少的代码使其通过测试；和ai沟通完需求后，可以调用这个skill采用这个开发范式来帮助提升需求执行准确率；

Github-release-management：github发布管理skill；GitHub 发布编排，包括自动化版本控制、测试、部署和回滚管理等；如果对github不熟悉的，可以直接用这个skill，一站式操作；

这几个星标都在1w+，尤其是对于非技术同学，能提效不少；当然除了这些，这个市场里还有非常非常多好用的skills，可以根据自己需求进行搜索。

**skills资源库：awesome-claude-skills**

ComposioHQ awesome-claude-skills

地址：

https://github.com/ComposioHQ/awesome-claude-skills

介绍：这是在github上非常火的一个skills资源库，14.4k的星标，里面包括了办公文档、代码开发、数据分析、市场营销、内容创作等非常多领域的优质skills；

推荐skills：

Claude Code Terminal Title：非常简单实用的一个功能skill；可以让Claude Code 终端窗口生成动态标题，描述该窗口正在执行的任务，这样我们每个窗口就一目了然，不会弄混了；

Skill Seekers ：可以将文档内容、GitHub 代码库和 PDF 文件转换skills，这样就不用再去人工阅读文档后再提示ai操作；比如在安装开源工具时，可以直接通过skill seekers把仓库相关文档和readme生成skill，让ai 调用这个生成的skill去操作就行了，极大解放生成力;

NotebookLM Integration：宝藏知识库skill；直接打通了notebooklm 和claude code，可以让claude code与notebooklm 里的文档内容进行问答、协同、执行任务；

以上就是本篇skills资源的分享，分享了skills资源库安装使用教程，只需下载skills文件夹到对应工具的skill目录，显示调用即可；也分享了3个高赞skills 资源库，包括官方库、skillsMP聚合网站、composioHQ的awesome claude skills库，基本涵盖了市面上90%以上的主流skills，看完基本也就知道可以在什么场景能使用skills了；同时也精选推荐了十几个我觉得非常有用的skills。

现在来看skills已经成为保证质量和效率的标准能力趋势，如果还没实际使用的朋友可以体验下看看，真的非常提效。

如果觉得本篇分享对你有帮助，欢迎一键三连～

对AI产品工具、vibe coding、AIGC内容感兴趣的内容也欢迎关注，期待交流，感谢！