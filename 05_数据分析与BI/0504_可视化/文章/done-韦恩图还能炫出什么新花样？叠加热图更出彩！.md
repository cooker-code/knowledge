> 已吸收至：[[05_数据分析与BI/0504_可视化/0504_核心知识点/图表选择与视觉编码准则|图表选择与视觉编码准则]]
---
title: 韦恩图还能炫出什么新花样？叠加热图更出彩！
author: SCIPainter
date:
url: http://mp.weixin.qq.com/s?__biz=MzIyOTY3MDA3MA==&mid=2247544083&idx=1&sn=40a87d268d26125aa5fdc46a5e942341&chksm=e8bd40cadfcac9dc9f535d6170354a739656a4bc4bebc20d261d3a67974b00da4eaf70838950&mpshare=1&scene=24&srcid=072193grHyitKrGMjb8rdTpq&sharer_shareinfo=e18c016c8ff31fff03d707048b9998c9&sharer_shareinfo_first=e18c016c8ff31fff03d707048b9998c9#rd
---

韦恩图还能玩出什么新花样？分享文献中几类组合可视化形式，如韦恩图+分类热图组合，可同步展示每个分区中独有或共有的目标/候选元素：

Fig1（*Nat. Plants*，2023）

**往期R语言教程：**[《如何绘制韦恩图+热图的组合图表？》](http://mp.weixin.qq.com/s?__biz=MzA5NzQzOTgzMw==&mid=2650989793&idx=1&sn=f286f20de2280dbbbd359c2fba17222d&chksm=8b569dd7bc2114c18bbbee84981e4b9fd7eec8a2e94ad52d42be0a0289392dbf51277de9b73c&scene=21#wechat_redirect)

韦恩图+样本/分组示意图，让表达更加生动：

Fig2（*Nat Commun*，2024）

韦恩图+实验流程/大纲示意图，让说明更加具体：

Fig3（*Nat Commun*，2024）

今天我们以Fig1为例，学习如何使用在线工具完成这类韦恩图+分类热图的绘制，无代码新手友好，小白也能绘制进阶组合图！

**内容一览：**

**1）韦恩图绘制**

**2）各分区共有/特有元素提取**

**3）分类型热图绘制**

**4）AI组图**

**1**

韦恩图绘制

首先使用OmicShare云工具平台的**动态韦恩图**工具绘制图表，使用无门槛，参数全都可以动态交互！不管是元素类数据还是丰度类数据都支持，具体格式需求可参考工具右侧详细说明。

**工具链接：**

https://www.omicshare.com/tools/home/report/reportvenn.html

这里我们以元素数据文件为例，每一列表示一个组，每一个单元格中的文本表示当前分组中的一个元素（如差异基因），列名为组名，参考格式如下：

上传数据并提交后，直接在页面下方的结果展示中按需调整图表即可。首先在参数调整-基本设置中适当调大文本字号，然后将分组名称加粗；

在参数调整-图形设置中，选择自己喜欢的配色方案（也可自定义），再将描边颜色修改为白色，描边粗细改为2：

满意后导出svg格式矢量图表，效果如下：

**2**

分类热图绘制

绘制韦恩图后，如何轻松提取图中各个区域所对应的共有或特有元素呢？在OS动态韦恩图工具中超级简单！

我们回到基本设置中，只要点击韦恩图中各个区域，在分组信息参数栏中会实时显示子集的名称、数量、元素名称等信息。

点击下载子集元素，可获得子集元素列表的txt，也可直接复制。

使用各子集的目标元素，制作绘制分类热图的数据。将元素名作为表达矩阵的行名，列名为分组，将交集标注为1，差集标注为0即可，格式如下：

使用OmicShare云平台的动态热图工具绘制热图，使用方法相同，上传数据并提交即可，简直so easy！

**工具链接：**

https://www.omicshare.com/tools/home/report/reportheatmap.html

在下方结果展示中按需调整热图样式，这里先在基本设置中关掉了图例，然后在图形设置中关掉了归一化和聚类，并更改配色为粉灰style。

如果想要绘制“扁扁”的热图，可在图片尺寸修改中通过增加宽度达成目的，想绘制更加“修长”热图同理。

导出svg格式矢量图，效果如下：

其它子集元素的提取、热图绘制的操作完全相同，这里不再赘述。

**3**

AI组图

最后，我们使用AI（Adobe Illustrator）进行组图和最终的细节调整，如文本大小统一等，具体操作在[《如何绘制韦恩图+热图的组合图表？》](http://mp.weixin.qq.com/s?__biz=MzA5NzQzOTgzMw==&mid=2650989793&idx=1&sn=f286f20de2280dbbbd359c2fba17222d&chksm=8b569dd7bc2114c18bbbee84981e4b9fd7eec8a2e94ad52d42be0a0289392dbf51277de9b73c&scene=21#wechat_redirect)中已为大家详细介绍，可直戳无缝衔接学习，这里不再赘述。

最终效果如下：

好啦，今日分享毕！更多科研干货、绘图技能分享关注SCIPainter不迷路！

OmicShare是基迪奥生物旗下，以交互式生信工具、原创组学书籍、生信论坛以及视频教程于一体的生信平台，现140000+科研人注册使用，超4500+篇SCI引用。即刻注册，轻松开启NCS绘图之旅！

**\*海量工具使用无门槛：**161+工具覆盖99%生命科学期刊发表所需，无需任何编程基础，提交数据即可完成绘图；

**\*发表级美图直出：**顶刊审美，参数/配色可实时交互；

**\*支持免费使用：**每个工具均可免费使用2次；还可通过【邀请好友】或【论坛任务】获取奥币，解锁更多免费次数。详情戳：[《OS新手使用说明》](http://mp.weixin.qq.com/s?__biz=MzIyOTY3MDA3MA==&mid=2247542726&idx=1&sn=cc64351390ff79fcc3e138419cb53430&chksm=e8bd4a1fdfcac3096d381d0677734f058a0674fdfb8f72ace7579e38db00da883b23554318f6&scene=21#wechat_redirect)

**\*完整权益体验：**升级【会员/超级会员】，实现绘图自由，还能尊享更多权益：

**OS注册：**

https://www.omicshare.com/user/register.php?lang=zh

**OS工具：**https://www.omicshare.com/tools/

**OS会员：**https://www.omicshare.com/vip/

参考文献

Libourel, C., Keller, J., Brichet, L. et al. Comparative phylotranscriptomics reveals ancestral and derived root nodule symbiosis programmes. Nat. Plants 9, 1067–1080 (2023).

Kumar, P., Goettemoeller, A.M., Espinosa-Garcia, C. et al. Native-state proteomics of Parvalbumin interneurons identifies unique molecular signatures and vulnerabilities to early Alzheimer’s pathology. Nat Commun 15, 2823 (2024).

READ MORE

延伸阅读

**\*未经许可，不得以任何方式复制或抄袭本篇文章之部分或全部内容。版权所有，侵权必究。**

**# SCIPainter**

基迪奥旗下绘图公众号

分享科研绘图技能与工具

欢迎关注与转发~

**你的好友拍了拍你**

**并请你帮她点一下****“分享”****~**