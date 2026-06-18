---
title: Claude Code 最被低估的 skill：文档技能
author: 左手用AI
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYzNjA1NzMwNQ==&mid=2247485032&idx=1&sn=dcd83ad79a8ea3f2fa960ab65be8f6ec&chksm=f1748ce3c4ba25363036096c6a9b582d503097e5bd9f22b82505c2ab8288598d15939a9f2d4e&mpshare=1&scene=24&srcid=0105vRB1AbwrTmkwZecQ648i&sharer_shareinfo=bfba84cd1adf5f07a9d36acdfc3529ae&sharer_shareinfo_first=bfba84cd1adf5f07a9d36acdfc3529ae#rd
---

## **1  Claude Code 不只是写代码的**

昨天有人问我：Skills 那么多，到底该装哪个？

我想了想，答案是：先装文档技能。

我之前写过几篇 Skills 的文章，讲怎么安装、怎么创建。但一直没专门聊过这个官方技能。

[Claude Skills实测：让AI生成PPT终于靠谱了](https://mp.weixin.qq.com/s?__biz=MzYzNjA1NzMwNQ==&mid=2247484842&idx=1&sn=58c503330bd4e995b88b0d6e44b57bd8&scene=21#wechat_redirect)

[从需求到交付：手把手创建你的第一个 Claude Skill](https://mp.weixin.qq.com/s?__biz=MzYzNjA1NzMwNQ==&mid=2247484896&idx=1&sn=be0b9bbad1c8ca388b1c1d7a91777573&scene=21#wechat_redirect)

[Claude Skill进阶：从能用到好用](https://mp.weixin.qq.com/s?__biz=MzYzNjA1NzMwNQ==&mid=2247484938&idx=1&sn=fe87c2e3fa917283f434fc9b986cf75f&scene=21#wechat_redirect)

[用 Claude Code 能干什么？看看这些场景是你需要的么？](https://mp.weixin.qq.com/s?__biz=MzYzNjA1NzMwNQ==&mid=2247484882&idx=1&sn=3ebc9e215c0257a704bd269bd9d6dace&scene=21#wechat_redirect)

它能处理 Word、Excel、PPT、PDF 四种格式。听起来普通，但用过之后你会发现——这可能是最实用的一个。

## **2  文档技能是什么**

简单说：让 Claude Code 能读写常见文档格式。

官方提供了四个文档相关的 Skill：

| Skill | 功能 | 典型场景 |
| --- | --- | --- |
| docx | Word 文档 | 会议纪要、合同模板、开题报告、论文格式 |
| xlsx | Excel 表格 | 数据分析、报表、带公式的表格 |
| pptx | PPT 演示 | 汇报材料、培训课件 |
| pdf | PDF 处理 | 读取 PDF、提取信息、填表单 |

这四个基本覆盖了日常办公要用的格式。

## **3  为什么说它被低估**

三个原因。

**第一，AI 天生擅长 Markdown，但我们不用 Markdown。**

AI 处理 Markdown 格式是最顺手的。但现实生活中，谁用 Markdown 写周报？谁用 Markdown 做汇报？

我们用的是 Word、Excel、PPT、PDF。这些才是真正的办公格式。

文档技能正好解决了这个衔接问题——让 AI 的能力能直接输出成我们日常用的格式。

**第二，很多人不知道有这个功能。**

Claude Code 的定位是"编程助手"，大家默认它只能写代码。但 Skills 系统让它能做更多事。文档技能就是其中之一。

**第三，以前处理文档很麻烦。**

没有文档技能之前，想让 Claude Code 生成一个 Excel，得写 Python 脚本，用 openpyxl 库，调格式、加公式、画图表。一个简单的表格，代码写半天。

现在不用了。直接说"帮我做个 Excel"，它就能生成。

## **4  实测：三个场景**

我测了三个场景，看看实际效果。

### **4.1  Excel 生成**

Prompt：

```
bash

帮我创建一个Excel文件，内容是：  
- 2024年Q1-Q4的销售数据（随机生成）  
- 包含：月份、销售额、环比增长率  
- 自动计算总计和平均值  
- 加一个简单的柱状图
```

▲ 提示词界面

两三秒就生成了。打开看，公式、图表都有，格式也整齐。

▲ 生成的excel

以前做这种表格，我得打开 Excel，手动输数据、写公式、插图表。现在一句话搞定。

### **4.2  PDF 读取**

这个我之前做测试点分解 Skill 的时候用过。

给它一个 PDF 文档（比如芯片规格书），让它提取关键信息，整理成结构化的数据。

▲ cc运行界面

分解出来的结果，保存成 excel 了：

▲ 生成的excel

对于经常要处理 PDF 的人来说，这个功能很实用。比如读合同、读规格书、读报告，不用自己一页页翻了。

### **4.3  Word 文档生成**

Prompt：

```
bash

帮我写一份会议纪要模板，Word格式，包含：  
- 会议基本信息（时间、地点、参会人）  
- 议题列表  
- 决议事项  
- 待办跟进
```

▲ 提示词界面

生成的模板比我自己写的还全。它加了待办事项跟进表，还有责任人和截止日期。说实话，我自己写的话可能会忘记这些。

▲ 生成的word

## **5  什么时候用**

说几个我觉得好用的场景：

* **快速出模板**：会议纪要、周报、数据表格，一句话生成
* **读 PDF**：合同、规格书、报告，不用自己一页页翻
* **数据整理**：把零散数据整理成带公式的 Excel

不太适合的场景：

* 复杂排版的 PPT（还是用专业工具）
* 设计要求高的文档（字体、颜色、布局讲究的）
* 超大文件（几百页的 PDF 会慢）

简单说：日常办公够用，专业设计还是用专业工具。

## **6  怎么用**

两步。

**第一步：安装**

在 Claude Code 里输入 `/plugins`，选择 marketplaces，然后选 第一个，没有的话添加下对应 git。

安装完重启一下就能用了。

▲ 插件界面

**第二步：直接说**

安装好之后，用自然语言描述你要做什么，它会自动调用对应的 Skill。

比如"帮我生成一个 Excel"或者"读取这个 PDF"，不用指定用哪个 Skill，它自己会判断。

不复杂，试一次就会了。

## **7  总结**

文档技能解决了一个实际问题：让编程工具也能处理日常文档。

实测下来：

* Excel：两三秒出结果，公式图表都有
* PDF：能提取结构化信息，太大的话让它按页来提取
* Word：模板比我自己写的还全

如果你用 Claude Code，但还没试过这个，建议试试。

一句话生成文档，不用写代码。这可能是最被低估的功能了。

有问题评论区聊。

✨ 喜欢这篇内容？欢迎点赞、分享、推荐，关注我获取更多干货！