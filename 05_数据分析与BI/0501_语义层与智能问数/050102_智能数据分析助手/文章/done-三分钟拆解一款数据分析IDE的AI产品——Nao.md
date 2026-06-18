---
title: 三分钟拆解一款数据分析IDE的AI产品——Nao
author: AI Startups
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5ODAwODU4Mw==&mid=2247485941&idx=1&sn=c0ed2a3b04d1f1adbdaa82910e7b965a&chksm=97c308a8871cc36d5ac4edb6aafb5bbdbb3578f2437216790ef0904c08b8cc74b0c1e330e778&mpshare=1&scene=24&srcid=1128pyeuU4SgEzjnSqDObTAc&sharer_shareinfo=f87ba6aa9f073cd21b23234210224713&sharer_shareinfo_first=f87ba6aa9f073cd21b23234210224713#rd
---


> 已吸收至：[[05_数据分析与BI/0501_语义层与智能问数/050102_智能数据分析助手/050102_核心知识点/AI数据分析工作区与质量控制|AI数据分析工作区与质量控制]]

Nao是一款专为数据团队打造的AI数据集成开发环境（IDE），针对传统数据工作流程中的多工具切换、缺乏数据上下文支持、代码变更易引发数据质量问题等痛点开发。

Nao今年3月发布，通过订阅收费，MRR为1.8万美元，MAU未公布，预计在数百人左右，用户群体为各类数据从业者。

创始人兼CEO Claire Gouze毕业于巴黎高商，曾任BCG数据科学家、独角兽Sunday数据负责人。2025年，获入选YC孵化50万美元投资，同时获知名数据顾问、畅销书作者Joe Reis等多名天使投资人投资。

### 最终成果展示

### 跑一遍核心流程

打开首页，点击下载并安装nao。

通过谷歌登录。

快捷向导中会提示设置快捷键、配置数据库等。

进入IDE，整体布局和Cursor类似，点击“创建一个沙箱环境”。

进入沙箱环境，打开准备好的Kaggle泰坦尼克数据。

右侧交互窗口中，可以选择使用的AI模型，包括GPT-5、Gemini 3 Pro、Sonnet 4.5。交互模式可以选择plan、chat、edit模式。

输入提示词：“这是 Titanic 数据集，请帮我自动完成： 1. 数据结构解析（字段含义、类型） 2. 统计缺失情况 3. 计算各 Pclass、Sex 的生还率 4. 生成两张图”并执行。

将“Always Ask”设置为自动执行，运行约5分钟，生成完整的数据分析结果。

输入提示词，生成一份完整的报告文件。