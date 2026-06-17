---
title: Coze+ VisActor Skill：智能图表，触手可及
author: VisActor
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA4NDk5NTYwNw==&mid=2651431427&idx=1&sn=9778251acf0fe76f1c5541a0add92c25&chksm=85f7d8a61d96e69a7a432d00fb60047eac90051deb81c4bae436ec3fd47b3ac8341082fb5113&mpshare=1&scene=24&srcid=030625CtM5CBuPLNM6nmnHCx&sharer_shareinfo=a52e12cb3306b32a47d5fadecc721d1e&sharer_shareinfo_first=a52e12cb3306b32a47d5fadecc721d1e#rd
---

# Coze 技能商店简介

## Coze 简介

Coze（扣子）是字节跳动推出的新一代 AI Agent 平台。

**使用入口** ：

* 公开版（国内）：https://coze.cn（网页端）/ 扣子 APP（移动端）
* 公开版（海外）：https://coze.com

**核心能力**

扣子提供了强大的功能，全面提升生产力，支持多种应用场景，满足不同用户的需求。

* **生产力全面提升**：从回答问题，到解决问题，让 AI Agent 帮你完成更多的工作。
* **技能**：AI 从“理解指令”向“掌握方法”跨越的核心。
* **能力边界拓展延伸**：MCP 扩展集成，无限拓展 AI Agent 能力边界。

## Coze 技能商店

熟悉AI 应用的同学，对 skill 的概念和应用一定不陌生，Coze 也推出了 技能商店，各种技能如雨后春笋般涌现。

# VisActor图表技能

VisActor 图表技能以其图表种类丰富、图表主题美观、智能配置灵活、代码生成准确 等诸多优点，在众多图表（可视化）技能 中脱颖而出。

VisActor 图表技能背后依托 字节跳动开源可视化解决方案 VisActor，基础组件和智能化生成能力千锤百炼。目前的图表Skill 背后主要依托 VChart 和 VBI 组件库。

**官网**：https://visactor.com/ ；https://visactor.com/vchart ；https://www.visactor.com/vtable ； https://visactor.github.io/VBI/

**github**：https://github.com/VisActor ； https://github.com/VisActor/VChart ；https://github.com/VisActor/VTable；https://github.com/VisActor/VBI

## VisActor 图表技能快速体验

通过链接 https://www.coze.cn/?skill\_share\_pid=7598468851759759366 或者 在Coze 技能商店搜索“VisActor” 都可以找到 VisActor图表技能。

打开卡片弹窗，点击“使用”。

接下来，我们可以在Coze中 @ VisActor 图表技能来实现可视化了。

输入需求之后，静静等待Coze执行任务。

Coze 会自动搜集数据，然后调用 VisActor 图表skill 生成图表。

VisActor 图表skill 会生成一个可以交互的图表，而且你可以通过修改VSeed spec 的方式进一步对图表进行调整，满意之后可以下载图片使用，同时可以分享页面链接。

VisActor 图表skill 支持丰富的图表类型，具体可参考 https://www.visactor.com/vchart/example。

## 数据透视

我们也可以传入自定义数据，供 VisActor 图表skill 进行分析。例如：

VisActor 图表skill 具备其他同类skill 不具备的数据透视能力。例如：

运行结果如下：

## 多轮编辑

VisActor 图表skill 提供了强大的编辑能力，用户可以持续在Coze中通过对话修改图表。例如这里我可以继续将一组柱图变成红色。

## 数据标注

数据标注也是 VisActor 图表skill 独有的强大能力。比如，我想标记一条折线的最高点。

更多标注能力可以参考：https://www.visactor.com/vchart/demo/marker/mark-line-basic

## 动态图表

动态图表多用于数据作品或者数据视频中，可以动态展示数据变化过程， VisActor 图表skill 目前提供了动态条形图，动态折线图，动态饼图，未来会追加更多的动态图表。

## 表格可视化

表格作为使用最为广泛的可视化形式，当然不能缺席。 VisActor Skill 底层封装了 VTable （https://github.com/VisActor/VTable）。比如：

在此基础上，进一步调整数据，“添加区域信息（东北，华北...)"

进一步调整，“按区域进行透视”，

我们进一步修改表格的样式，“为环比超过100 的单元格 设置红色背景”：

## 更详细（高级）指令

VisActor Skill 支持用户更加细致、多样化的需求，如果你想定制高级可视化能力，可以参考 VBI 文档关于 VSeed 的部分（https://visactor.github.io/VBI/vseed/guide/quickStart.html）。

## 常见问题及解决

因为用户的需求千差万别，偶尔会有无法生成或者出错的情况，用户可以将报错信息同步给Coze，让它修复重新生成。

当然我们会搜集用户的反馈，不断改进。

### 问题1：多轮编辑的时候更新html产物失败？

**解决办法**：VisActor 图表技能是能够支持多轮编辑的，这种情况一般是由于大模型产出偶尔不稳定导致的，可以告诉大模型：“编辑失败，请重试”；

### 问题2: 产出的html页面报错怎么办？

**解决办法：** 将页面上的报错信息复制，告诉大模型，让大模型更正代码

### 问题3:在某轮编辑中，大模型解决报错问题失败？

**解决方案**：上面的问题明显是因为大模型没有调用技能直接去修复问题，导致修复问题失败，我们在技能中以及明确的写明解决任何问题需要调用图表技能，如果存在这种大模型偷懒的情况，可以强调一下，让大模型调用图表技能进行解决

对于上面提到的问题，如果按照提供的解决办法，尝试多次，都不能解决，可以联系我们进行优化～

# 信息图、数据视频——值得期待

单一的图表生成并不能满足可视化的应用场景，接下来我们会推出信息图和数据视频的生成功能，敬请期待。

# 欢迎交流

最后，欢迎大家以各种方式参与 VisActor 的开源建设，提 issue，写代码，加评论，点 Star，都是对开源项目的支持。

VisActor：https://visactor.com/

VBI：https://github.com/VisActor/VBI

VChart：GitHub - VisActor/VChart: VChart, more than just a cross-platform charting library, but also an expr

VTable：https://github.com/VisActor/VTable

VisActor 飞书交流群：

VisActor 微信公众号：