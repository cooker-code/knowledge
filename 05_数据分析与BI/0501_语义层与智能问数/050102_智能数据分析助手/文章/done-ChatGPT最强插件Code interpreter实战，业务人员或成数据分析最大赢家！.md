> 已吸收至：[[05_数据分析与BI/0501_语义层与智能问数/050102_智能数据分析助手/050102_核心知识点/AI数据分析工作区与质量控制|AI数据分析工作区与质量控制]]
---
title: ChatGPT最强插件Code interpreter实战，业务人员或成数据分析最大赢家！
author: 与数据同行
date:
url: http://mp.weixin.qq.com/s?__biz=MzIwNDI0ODY1OA==&mid=2655983857&idx=2&sn=e9662528f95cf6f547478f39cb5e3dae&chksm=8d78bbccba0f32da3a90f2c45f76a83b81d6542d2b161c3d712ca8c7c807e7742984ee05467b&mpshare=1&scene=24&srcid=01020TWc1UZlNRNhRqP2ofD0&sharer_shareinfo=c8e40711b66bd09d655a9e614f37d3d2&sharer_shareinfo_first=c8e40711b66bd09d655a9e614f37d3d2#rd
---

7月8日凌晨，OpenAI向所有ChatGPT Plus用户开放了代码解析器功能。这是自OpenAI发布GPT-4以来，最强大的功能！在官方的博客中，是这样介绍 Code interpreter 的：

官方也给出了三个基于此能力的推荐用法：

* Solving mathematical problems, both quantitative and qualitative 解决数学问题，包括定量和定性问题。
* Doing data analysis and visualization 进行数据分析和可视化
* Converting files between formats 在不同格式之间转换文件

这将允许ChatGPT 运行代码，并且可以访问用户上传的文件，可实现分析数据、创建图表、编辑文件、执行数学运算等复杂操作。

其中，**数据分析功能非常非常强大，使得很多不会专业代码的业务人员，通过自然语言文本、数据文件等，就能快速创建可视化数据分析图表**，适用于销售、人力资源、医疗、制造、媒体、金融等业务场景。

想使用该功能非常简单，我们只需要在自己的ChatGPT plush账户上启用 Code Interpreter即可。

启动方式如下：Setting > Beta Features > Code interpreter

接下来我们试试这个插件，点击GPT4.0，再点击以下即可：

下面是我自己测试的过程，的确是比较强大的：

**1、导入数据**

可以上传任何文件到ChatGPT进行分析，包括PDF，word，excel，CSV，gif，MP4等等。

**2、数据分析**

***Prompt***

*（1）显示文件内容*

*（2）显示10条数据样例，每列如果超过10个字符就截断*

*（3）给出数据集的概貌*

*（4）给出这份数据的概况*

*（5）请分析评分与其他相关变量的关系*

*（6）请做一张报表，维度是type，指标是Recommended为1的比例值，Positive\_Feedback\_Count的合计*

*（7）假如你是一名数据分析师，请以图表形式给出对输入数据的分析，并给出一份数据分析报告*

**3、机器学习**

***Prompt***

*（1）以推荐为标签，采用至少5个算法给出预测模型，然后比较优劣，最后给出推荐的模型*

*（3）给出随机森林模型中最重要的变量排名*

*（4）请基于梯度提升模型做一个模拟推理，输入自行决定*

**4、可视化**

***Prompt***

*（1）请自行选择合适的数据，至少画出20种以上类型的图表，每张图表一个说明，一张一张的按顺序展示*

*（3）请分别画出堆积柱状图、饼图、直方图、散点图、等高线图、热图、气泡图、雷达图、3D 图、面积图、阶梯图、蜡烛图、词云图*

**5、流程图**

**6、图像处理**

*****Prompt*****

*（1）显示原图尺寸，居中裁剪出最大方形，并显示图片*

*（2）以 576x576 作为原图的滑动窗口，每次移动图片宽度的 1/50，将其制作为循环播放的 gif。*

*（3）转换为黑白色，并显示*

*（4）将原图转换为 JPG 格式提供下载链接，并显示原图*

**7、格式转换**

Code interpreter让我想到了一个当前AI投资圈的热词JARVIS。

JARVIS是一个协作系统，该系统由LLM作为控制器和众多专家模型作为协作执行者（来自HuggingFace Hub）组成，系统的工作流程包括四个阶段：

（1）任务规划：使用ChatGPT分析用户的请求以了解他们的意图，并将其分解成可能解决的任务。

（2）模型选择：为了解决计划的任务，ChatGPT 根据他们的描述选择托管在拥抱脸上的专家模型。

（3）任务执行：调用并执行每个选定的模型，并将结果返回给 ChatGPT。

（4）响应生成：最后，使用 ChatGPT 集成所有模型的预测，并生成响应。

Code interpreter就是类似的协作系统，它本身不具备执行能力，但它通过ChatGPT能理解用户的意图，从而能选择到合适的工具或模型（比如python）来执行，并且能把结果返回给ChatGPT，ChatGPT再向人做出响应。

我突然发现，自然语言已经能逐步替代码农让机器直接执行指令，这导致了码农的快速贬值，进而影响到那些调参侠式的数据分析师。从这个角度来讲，Code interpreter极大降低了数据分析的技术门槛。

此消彼长，有了Code interpreter后，业务人员在这个过程中成了最大的受益者，其次是数据治理者，因为原始生产资料的质量还是需要得到保障，而中间的那些加工者，则被极大的压缩了舞台。

[中国最容易和最难被GPT所代替的TOP25职业！](https://mp.weixin.qq.com/s?__biz=MzU4NjgzNzk4MQ==&mid=2247503168&idx=1&sn=1caf9a75b5700057a595445fbeccb580&scene=21#wechat_redirect)

[一文搞懂ChatGPT相关概念和区别：GPT、大模型、AIGC、LLM、Transformer、羊驼、LangChain…..](https://mp.weixin.qq.com/s?__biz=MzU4NjgzNzk4MQ==&mid=2247503156&idx=1&sn=eedabc483b56e047f6779577ce7c5842&scene=21#wechat_redirect)

[五级数据挖掘工程师，你处在哪一级？](https://mp.weixin.qq.com/s?__biz=MzU4NjgzNzk4MQ==&mid=2247502972&idx=1&sn=bfe85470c4169580deb446f3b30fa24e&scene=21#wechat_redirect)

[从芝麻信用分透露的详细数据设计，我们能从中得到什么启示？](https://mp.weixin.qq.com/s?__biz=MzU4NjgzNzk4MQ==&mid=2247485538&idx=1&sn=4304df62347cc3444b98f25471c5227d&scene=21#wechat_redirect)

[全部文章](https://mp.weixin.qq.com/s?__biz=MzU4NjgzNzk4MQ==&mid=2247492609&idx=1&sn=f9ff1da0ac2d859b0c6e4ff3d7473c62&scene=21#wechat_redirect)

点击左下角“**阅读原文**”查看更多精彩文章，后台回复【**加群**】申请加入万人**数据学习**社群

🧐**分享、点赞、在看**，给个**3连击**呗！**👇**