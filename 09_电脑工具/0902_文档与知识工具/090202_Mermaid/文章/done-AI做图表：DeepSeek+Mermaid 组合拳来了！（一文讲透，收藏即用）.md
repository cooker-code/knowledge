---
title: AI做图表：DeepSeek+Mermaid 组合拳来了！（一文讲透，收藏即用）
author: 悦呀AI笔记
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247484292&idx=1&sn=84d977584431f26f70ff33d856f87426&chksm=978acaf78d116b334217c74610ee1dd3db8a6510d9cac9f78b400680371789a7fcc9832ed6cc&mpshare=1&scene=24&srcid=0909WrLwTl3q8K6MNjfuzyU8&sharer_shareinfo=e5fd2da5b1ac0c1476e580ec49f44841&sharer_shareinfo_first=e5fd2da5b1ac0c1476e580ec49f44841#rd
---
> 已吸收至：[[09_电脑工具/0902_文档与知识工具/090202_Mermaid/090202_核心知识点/Mermaid图表质量与渲染边界|Mermaid 图表质量与渲染边界]]


****欢迎点击👇关注我**

**您的关注是我持续分享的动力**

**追AI的80后 | 国企上市公司“跨界者”（技术→产品→市场→AI）。坚信AI是下一个风口，更是超级个体的成长杠杆，在这里日常更新AI学习笔记，陪你借势向上。****

用DeepSeek制作图表时，生成的图表通常无法直接编辑；若不符合需求或存在理解偏差，需通过自然语言（Prompt 提示词）多次与大模型沟通，才能 “指导” 其输出目标效果。

很多时候，“一加一大于二”。若联合Mermaid等绘图工具，可输出编辑性更强、灵活度更高的图表。

本文介绍DeepSeek与Mermaid联合制作图表的方法。

一、什么是Mermaid

DeepSeek大家比较熟悉了，这里重点说一下Mermaid。

Mermaid 是一种基于文本的图表生成工具（语言），用于绘制图表和图形（比如常见的流程图、序列图等等）。

常用的在线Mermaid工具网址：

https://mermaid.live/edit

二、两者结合做图表的“思路公式”

**自然语言 → DeepSeek → Mermaid 代码 → Mermaid 渲染 → 精美图表**

**1. 自然语言提问**：在 DeepSeek 对话框里描述你要的图，“帮我画一个新品上架流程图，风格简洁，用 Mermaid”。

2. AI出图代码：DeepSeek 立即生成符合 Mermaid 语法的文本。

3. 一键渲染：把代码粘到 Mermaid.live、draw.io、Obsidian 等任意支持 Mermaid 的工具里，图表即刻呈现。

4. 二次迭代：想改结构？直接改文本，图实时刷新；想换皮肤？Mermaid内置主题一键切换。

三、实践案例

比如我向DeepSeek输入提示词：“作为一个入门小白，我想系统性学习Prompt提示词，请以Mermaid格式，系统性输出学习步骤。”

DeepSeek会输出如下结果：

此时的图表，如果我们下载下来是不能够进行修改的。

但是可以点击左上角的“代码”：

```
flowchart TDA[Prompt提示词系统性学习路径] --> Bsubgraph B[第一阶段：基础认知]    direction LR    B1[核心概念<br>Input/Output<br>指令、上下文、角色等] --> B2[核心原则<br>清晰、具体、<br>赋予角色、迭代优化] --> B3[基础技巧<br>使用引号、样例、<br>避免歧义]endB --> Csubgraph C[第二阶段：结构化与模式]    direction LR    C1[角色扮演<br>（Act as a...）] --> C2[任务分解<br>（Step by Step）] --> C3[模板学习<br>（CRISPE、BROKE等）]endC --> Dsubgraph D[第三阶段：进阶技巧与应用]    direction LR    D1[零样本/少样本<br>提示（Few-Shot）] --> D2[链式思考<br>（CoT）] --> D3[外部工具<br>与知识结合]endD --> Esubgraph E[第四阶段：实战与迭代]    direction LR    E1[分析优秀案例] --> E2[专项任务实战<br>写作/编程/数据分析] --> E3[构建与迭代<br>自己的提示库]endE --> F[持续学习与交流]
```

复制代码，我们把代码粘贴到Mermaid在线网站（左侧框），右侧就会渲染出具体的图表。

如果需要对图表进行修改，可以在左侧修改代码。

此外，还可以点击左侧“Actions”，进行尺寸调整、并且支持以不同格式进行图表下载。

“Sample Diagrams”里还有大量其他样式的图表，大家可以选择使用，点击可以浏览样式。

这些图表格式DeepSeek也是能够识别的。

比如，你看准了哪个图表样式符合需求，你可以直接在DeepSeek的对话框中输入“XXX，请以Mermaid中的Mindmap格式输出”。

内容大概就是这些了，大家上手试试。

有什么问题，欢迎评论区交流。

*本文为【[DeepSeek做图表](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzE5MTI2ODAxMg==&action=getalbum&album_id=4099218700367036419#wechat_redirect)*】*系列最新内容，关注我获取更多AI学习笔记、AI办公技巧！*

*【重要提醒】微信调整了推荐机制。如果不想错过每期推文，请为悦呀AI笔记加关注及星标！随手点击下方名片→点“关注公众号”→点右上角（…）弹出菜单栏→点“设为星标”即可。*

往期文章：

[DeepSeek-V3.1做图表：自动化处理Excel并生成可视化报告（附全流程Prompt模板）](https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247484221&idx=1&sn=6f7a138033680621cd4c2044f21534e0&scene=21#wechat_redirect)

[DeepSeek-V3.1做图表：告别手工Excel！三句话让AI为你做完整的数据分析（附AI提示词）](https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247484227&idx=1&sn=98a0af9658dc865590504003b3988850&scene=21#wechat_redirect)

[DeepSeek-V3.1做图表：对Excel进行深度数据分析与挖掘（附高级Prompt提示词模板）](https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247484269&idx=1&sn=07265c74111b2d3f6891cb16268f708b&scene=21#wechat_redirect)

---

**以上，就是本次的分享，对你有启发吗？**

**如果对你有帮助，欢迎关注、点赞、转发、收藏**

**PS：如果你想听的我没有说到，请评论区告诉我。**

**个人观点、仅供参考**
