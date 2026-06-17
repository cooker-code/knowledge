---
title: 藏师傅想解决 Claude Code 最恶心的问题
author: 歸藏的AI工具箱
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU0MDk3NTUxMA==&mid=2247493157&idx=1&sn=82eeb413d455d3cffea3738aed9e0c2f&chksm=fac863a01c438d174bdc0699383f349d9bd5a5c1bf0a5d3c85d1546192aa62876c01c97b7cf7&mpshare=1&scene=24&srcid=1106LInJijTAp20SSB1ShKlC&sharer_shareinfo=4c4bc4807de34a8bd519d89559704186&sharer_shareinfo_first=4c4bc4807de34a8bd519d89559704186#rd
---

我不知道你们有没有遇到过这个问题。

因为 Anthropic 的恶心操作，我们使用 Claude Code 必须使用代理 API，同时国内模型比如智谱、Kimi 的新模型对于 Claude Code 的支持。

我们需要维护的 Cluade Code 环境变量越来越多了，更别说还有 Qwen 或者 Codex 的代理这种非原生支持 Claude 的模型。

导致我换一个模型就得清理一下 Cluade 的环境变量，对于我这种不会命令行的菜逼来说，每次更换环境变量都非常痛苦，每次都得询问 GPT。

这次为了更换原生的 Claude 4.5 代理 API 甚至跟环境变量搏斗了半小时，于是我急了，想要写个软件彻底解决这个问题。

经过跟 GPT-5 的指导以及赛博黑奴 Sonnet 4.5 的辛苦劳作，我终于搞定了！

向大家介绍一下我最新的开源项目：ai-claude-start（https://github.com/op7418/ai-claude-start）。

这个项目可以让你快速配置多个 Cluade Code 模型的 API 数据，在通过 Cluade-Start 这个命令启动 Cluade Code 的时候会让你选择从哪个模型和服务商启动，Cluade Code 更换模型启动的时候费劲的问题。

你也可以用这个项目快速启动多个不同模型驱动的 Claude Code 进程，非常爽。

而且这个配置还是写在 Cluade Code 环境变量之外的，只会在每次启动的时候临时注入，不会影响 Claude Code 原始的设置，非常安全。

接下来我们看一下如何安装和使用这个项目，非常简单。

后面我会大概说一下我是怎么做的，基本全程都是 GPT-5 和 Claude Sonnet 4.5 写的我只做了一下测试和描述需求。

项目支持 npm 和 npx 安装，如果你已经撞了 Node.js 的话直接在终端里面输入下面任意一个命令就行。

如果还没有安装 Node.js 可能需要在这里（https://nodejs.org/zh-cn）下载安装包先把 Node.js 装上。

```
npm install -g ai-claude-start
```

```
npx ai-claude-start
```

如果你是小白，安装过程中有任何报错问题可以直接跟 GPT 讨论，大部分问题他都能解决，记得把你的报错内容给他，类似我刚开头那样。

然后我们就可以开始最基本的配置了，输入 ai-claude-start setup 这个命令就会启动初始配置，一般 Cluade Code 模型环境变量的替换主要是三部分内容，模型 API 地址、API Key 以及模型名称。

我这里内置了Anthropic、智谱和 Kimi 的三个 API 地址你如果是这三个之一的话可以直接选择，地址这里就别管了，只需要写一下模型名称和 API Key 就行，这个在开发者后台都有。

如果你需要的不是我预置的三个 API 的话，可以直接选择 Custom 这个时候配置名称、API 地址和模型名称和 API Key 就行。

最后将你所有的 API 都配置完成之后下次启动 Claude Code 的时候输入 Cluade-Start 就行，项目会先让你选择模型，非常方便，也解决了小白用户不会配置 Cluade Code 环境变量的问题。

这就是基本使用教程了，如果你想要修改或者看更多自定义的命令的话可以来这里看 Cluade Code 写的 Readme 文档，这里更加详细（https://github.com/op7418/ai-claude-start/blob/main/README\_CN.md）。

后面大概说一下这个项目大致的构建过程。

GPT-5 发挥了很大的作用，刚开始我是跟他讨论如何解决我自己的 Claude Code 环境变量污染问题的，后面虽然解决了。

但是我想到我已经跟他讨论了好几次这种事情了，我不想每次都这么麻烦，于是就想能不能写个项目存这些环境变量每次启动的时候让我选择。

它先是给了我一个临时的本地存储的命令行方案，后面我就说我不想要临时方案，我想给他做成一个项目，具体的要求是 XXX。

最后跟他说我想要给另一个 AI 让他写代码完成，应该如何跟那个 AI 描述需求，他就给了我一整套提示词，考虑的非常全面。

你是 Node.js CLI 工程师。请基于我提供的脚手架文档，建立一个 npm 包 ai-claude-start（可改包名），满足：

npx 包名 首次运行进入向导，内置 3 个预置：Anthropic、Moonshot、IMDS + 自定义；

Profile = { name, baseUrl, authVar(= ANTHROPIC\_AUTH\_TOKEN | ANTHROPIC\_API\_KEY), model? }；密钥安全存储（优先 keytar，不可用则本地明文 fallback 并告警）；

子命令：setup | list | default <name> | delete <name> | doctor；

运行时清空子进程中所有 ANTHROPIC\_\*，仅注入所选 Profile 所需项；若既含 token 又含 key，则默认保留 token；

支持 claude-start <profile> [传给 claude 的参数…]；

额外加一个测试用开关：--cmd <binary> 或环境变量 CLAUDE\_CMD，用于在没有 claude 时用 node -e 或自定义命令替身；

提供 README、LICENSE(MIT)、.gitignore；

加最小自动化测试（Jest 或 Vitest）：对 URL 清洗、环境注入冲突消解、配置读写 做单元/集成测试；

输出完整项目结构与关键文件内容，确保本地 npm link 能直接跑通。

之后就是启动 Claude Code 把第一条大指令给他，一次输出的结果已经可以跑了，后面我人工测试了一下，发现几个功能理解的问题，又修复了一下，就可以了，因为每次他自己会进行测试，所以简单的问题都自己修复了。

最后还让 Cluade Code 指导了一下我该如何把项目发布到 npm 让大家可以顺利的安装。

以上就是这个小项目的使用方式和构建过程了，如果你有其他想法和功能的话欢迎提交 Pull Request 。

由于我对这部分代码一窍不通都是 AI 写的所以不可避免的可能会有问题，如果有问题建议优先跟 AI 沟通，也可以发我，但我水平有限可能修的很慢。

感谢各位可以看到这里，如果有帮到你希望帮我点个赞👍或者🩷，也可以分享✈️给你的朋友，感谢🙏