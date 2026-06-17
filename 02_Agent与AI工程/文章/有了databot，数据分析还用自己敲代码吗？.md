---
title: 有了databot，数据分析还用自己敲代码吗？
author: 基因学苑
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI2MjA1MDQxMg==&mid=2649721106&idx=1&sn=ff81b993841b7ec7c2eb2747969d8505&chksm=f314f1a779ef1e3d830b0612700fb4b47f3ce1919ac11ab389eec3ee4cdd3ea451871189b634&mpshare=1&scene=24&srcid=1026EFwVj8Bb57H1urgaCS7Z&sharer_shareinfo=ac99bad8991690ac020f18feda148076&sharer_shareinfo_first=ac99bad8991690ac020f18feda148076#rd
---

> Positron这款数据分析工具，目前又有了新升级，这次AI功能有了更大进步，首先支持github copilot进行对话。之前只能在行内使用。而且推出了一个databot。Databots属于posit公司自己的AI agent了，因为rstudio目前无法支持AI功能，所以是时候放弃rstudio了。

---

**Positron是什么？**

网址：https://positron.posit.co/

Positron是Posit（原Rstudio公司）推出的下一代数据分析工具，目前处于快速开发测试版。不过更新速度很快，目前最新版是Positron-2025.10.1-4版本。

Positron基于开源Code-OSS开发，所以与vscode非常像，不过不支持vscode官方应用扩展，但是有open-vsx项目，绝大部分vscode扩展都可以安装。最新版本已经支持AI功能了。

**为什么要选择Positron？**

Positron的主要目的是替换Rstudio和vscode，那么相比于这两个工具，positron有哪些优点呢？

1. Posit (原Rstudio)公司出品

2. 下一代数据分析集成开发环境

3. 同时支持R和python

4. 无需复杂配置,开箱即用,支持ipython和jupyter

5. 专门为数据分析优化

6. 类似rstudio布局,上手更方便

7. 支持创建项目,包含绘图窗口

8. 可扩展插件,实现更多功能

9. 远程登录服务器,使用R更方便

10. 支持AI功能

**AI助手**

**终于positron也支持AI功能了，虽然没有vscode诸多的插件方便，但也一直在更新，目前仅支持claude和copilot，openai。之前claude可以使用聊天模式，copilot只能行内代码补齐，不过目前Copilot也支持聊天了。国内模型目前用不了，不过可以尝试兼容OpenAI的API即可。**

**Databot数据分析助手**

Databot 是一款Positron推出 AI数据分析助手，能够通过自然语义完成数据分析的功能，支持 Python 或 R的探索性数据分析，目前是beta版本，不适合用于生产环境，仅支持 Positron工具。

目前Databot使用的是 Claude Sonnet 4模型，所以要使用还是有一定门槛，需要配置好Anthropic API。不过这也有解决方案，只需要配置一个兼容Anthropic API的国产大模型即可，例如DeepSeek，GLM均兼容。

https://positron.posit.co/databot.html

https://posit.co/blog/introducing-databot/

 Databot 是一种高度交互的体验。这很像与数据科学家进行结对编程，数据科学家打字速度非常快，从不感到无聊，并且不断有下一步该做什么的想法，但仍然会等待你的指示，然后再继续。

Databot 体验的核心是一个循环，当用户向 Databot 提出问题或指令时启动。这可以是“此数据集是否存在任何明显的数据质量问题”，也可以是“将 x 轴小刻度设置为 5 年增量”等具体内容。

然后，Databot 通过执行以下步骤（称为 **WEAR 循环**）进行响应：

1. **编写代码**

   – Databot 编写 Python 或 R 代码来回答问题或执行任务。此代码将显示给用户。
2. **执行**

   – 代码在当前 R 或 Python 会话中自动执行。输出（包括控制台输出、绘图和表格）对用户和 Databot 都可见。
3. **分析**

   – Databot 进行观察并从输出中得出结论。这可能包括回答用户的问题，或注意任何令人惊讶的结果，或提出进一步调查的想法。在此步骤之后，如果有非常明显的后续步骤，Databot 可能会选择循环。
4. **重新分组**

   – Databot 建议大约三到五个后续步骤供用户选择。这可能包括继续当前的调查路线，继续解释数据的一些意想不到的特征，或者提出一个全新的问题。

安装和使用**Databot**

确保您拥有最新版本的 Positron（2025.08.0 是最低版本）并启用了 Positron Assistant。在 Positron 中，从左侧的活动栏中选择扩展视图，或使用键盘快捷键并搜索“Databot”。单击按钮安装扩展程序。Ctrl-Shift-X，安装Databot插件。

然后需要在Positron 的设置中开启AI助手，并找到 `databot.researchPreviewAcknowledgedment`。在文本框中，键入“Acknowledged”。

然后在 Positron 中，打开命令面板并运行打开用户设置。Ctrl-Shift-P

在搜索栏中，键入“Databot”。

您将看到 Databot：研究预览确认。在文本框中，键入“已确认”。

启动 Databot

在R或者Python中读入一个文件，例如读入mtcars文件。接下来就可以通过自然语言来进行数据分析了。