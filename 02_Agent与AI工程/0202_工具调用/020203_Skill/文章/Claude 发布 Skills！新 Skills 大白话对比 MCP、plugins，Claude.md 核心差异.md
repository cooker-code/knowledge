---
title: Claude 发布 Skills！新 Skills 大白话对比 MCP、plugins，Claude.md 核心差异
author: 泽安的AI编程
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYyMzU2MDk4MA==&mid=2247484061&idx=1&sn=6e0f240c8ab2a1b5611cd707871c89ef&chksm=fe0784af2f0eb6d5cc5138c63c6963d9ac634e48c6b8d3b4f77a5fb8887c082c9eecda97587b&mpshare=1&scene=24&srcid=1028An2wNZd49ubKKMuwD1N9&sharer_shareinfo=acab94959d53e044babba2e0bbf6c82e&sharer_shareinfo_first=acab94959d53e044babba2e0bbf6c82e#rd
---

大家好，我是泽安！见字如面～

这是泽安的第 16 篇文章，欢迎帮我点点关注！泽安今天给大家讲清楚 Claude Code 最新发布的 Skills 功能！

我看到 Skills 的时候，发现很多人都是复制一堆官网的内容，泽安看的也是云里雾里的，不清楚与 Claude.md 和 MCP 的本质区别，

今天泽安根据自己的理解，通过大白话给您讲清楚

# Skills 到底是什么？

咱就截取官方的这句话：Skills 是"folders of instructions， scripts， and resources that Claude loads dynamically"---》翻译过来就是： Claude Code 动态加载的指令，脚本，资源文件夹

❤️

核心指令：动态加载

核心资源：脚本，资源文件夹

泽安把 Cluade Code Skills 比作为 Cluade Code 的工作流，

举个例子：我想做一个 Excel 数据分析与可视化 Skill

你让 Claude 处理一个 Excel 文件：进行复杂的数据清洗、计算聚合指标、并生成可视化图表。Claude 能读懂你粘贴的少量表格数据，但直接读取二进制 Excel 文件、执行 Pandas 操作、生成图表文件这种事，原生的 Cluade Code 做不到或者每次做不好。

装上 Excel 数据分析与可视化 Skill 后，Claude 每次就可以通过一句话：

* 调用 Pandas 和 NumPy 库处理数据：读取 .xlsx 或 .csv 文件，执行数据清洗、筛选、排序、分组、计算新字段等复杂操作。
* 进行统计分析：计算描述性统计（均值、中位数、标准差等）、进行相关性分析、或运行简单的回归模型。
* 生成动态图表：调用 Matplotlib 或 Seaborn 库，创建柱状图、折线图、散点图、热力图等，并保存为图片文件。
* 输出结构化报告：将分析结果和图表整合，生成一份包含关键发现和数据摘要的 Markdown 或 HTML 格式报告。
* 处理大型文件：高效处理远超聊天窗口字数限制的大型数据集。

看下核心结构：

```
skill-name/  
├── SKILL.md          # 核心指令文件  
├── scripts/             # 执行脚本  
├── templates/         # 模板文件  
└── LICENSE.txt        # 许可证
```

# 和 CLAUDE.md 有什么区别？

通过上面的介绍，有很多人误以为和 Claude.md 定义的规则非常的相似，但是，本质是完全不同的；

Claude.md 在项目启动的时候会加载项目里，他会一直占用上下文，并且会随着上下文的迭代，会更新与迭代，本质上 Cluade.md 是告诉 claude code 怎么干活，一般在这里面房架构说明，需求文档，编码规范，环境要求等之类的要求

而 Skills 首先是动态加载的，不占用上下文的，更像一个自定义的脚本工作流，告诉 Claude code 完成相对标准化的任务，可以按照规则扩展代码，一般执行数据分析，数据库初始化，git 操作规范流程，运维 devops 流程，如打包代码等之类的标化流程的提效利器，

本质差异：Skills 是可以做动作的，而 Claude.md 只可以定义规范的上下文

# 和 MCP 有什么区别？

MCP 是一个基于 JSON-RPC 的通用“链接协议”，Claude Code 来发起请求，MCP 返回结果，更多的 Claude Code 与外部系统的集成，他可以被 Skills 调用，

Skills 是轻量级的“能力包”，将规则+脚本+资源给串联起来，构建起来的一个超能力，Cluade Code 可以自主识别并选取调用，本质是一个单点的标准化的微服务。

本质差异：skills 解决单点怎么做的问题，mcp 解决 Claude Code 与外部链接如何做的问题。

# 和 Plugins 有什么区别？

Plugins 可以理解为一个应用商店，里面有各种各样的第三方小工具，功能多样化，稀奇古怪的需求都可以去寻找，社群目前比较活跃，更新快，有免费和收费的；但是质量良莠不齐，需要自己安装，同时也有可能引发 Plugins 之间的兼容问题，另外安全性也很难保障

Skills 就是自己在本地定义的应用商店，可以自己来用，和团队项目来用

本质差异：Skills 自己定义自己的规则和应用，Plugins 就像在 App Store 中下载第三方应用

# 安装

给大家介绍一种安装方式：通过 Skills 插件市场来安装

在 claude code 中，只需要执行这一条命令，即可使用官方提供的 skills

安装后，在。claude code 下面会有一个这样的目录

选择 skills，这里安装的时候我选择的是“document-skills”

安装完成后，需要重启 Claude Code 客户端

安装完成后，在 claude 中提问，你有哪些 skills？ ，即可看到可以使用的工具了！

如果你使用SKills 有什么见解和问题，欢迎进群聊聊

>/ 作者：泽安

能看到这里的都是真爱！

如果觉得不错，随手点个赞、小心心、转发，评论四连吧~

如果想第一时间收到推送，加上关注，给我个星标⭐

谢谢你耐心看完我的文章~