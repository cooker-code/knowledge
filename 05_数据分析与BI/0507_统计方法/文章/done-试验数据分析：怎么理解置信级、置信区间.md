---
title: 试验数据分析：怎么理解置信级、置信区间
author: 邱工笔记
date:
url: http://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485965&idx=1&sn=e5d3be89c8ee7afd8979d208751c0d70&chksm=fa5c0cc4cd2b85d2b78b3b48e316d6d681ac6aee242ba7b4423098ee26abb0cfd4d105e90854&mpshare=1&scene=24&srcid=0709U7QYEZnnvvdGygOYvqRL&sharer_shareinfo=01ea7f3c9c890bbee2faed166e89b536&sharer_shareinfo_first=01ea7f3c9c890bbee2faed166e89b536#rd
---


> 已吸收至：[[05_数据分析与BI/0507_统计方法/0507_核心知识点/统计检验方法选择与多重比较边界|统计检验方法选择与多重比较边界]]

**关键词：标准值   置信区间   置信级   保证率**

复杂结构常常需要力学试验辅以验证理论计算模型是否可靠、安全。

对试验得到的结果进行分析的时候，会用到概率和统计学相关知识，其中有3个常见、基础且重要的概念：置信级、置信区间和保证率。

比如根据置信级和保证率才能确定临界值，然后联合临界值和试验数据提取标准值，将标准值和理论计算结果进行比对，然后研判计算模型的准确性。

比如某市地标里面是这么写的，如图1所示。

图1   某地标对试验的要求

这个表达式里面的3.4对应的就是5组试验数据下的临界值。

除了这个地标，至少建筑工程领域，对于材料性能不稳定的结构试验一般都是5组，再少也不能低于3组。

主要是因为性能越不稳定的材料试验数据的离散性也越大，所以试验数据过少会导致得到的标准值过低，会严重低估你设计的结构承载力。相反，试验数据越多越能客观地、真实地表达该结构的实际承载力，而且得到的承载力也越高。

但是仅仅是出于验证性的试验，数量多少就无关紧要了，也不在本帖探讨范围内。

标准要求5组试验，可是即便如此，如果是混凝土破坏，鉴于混凝土稳定性相比钢材较差，也有可能导致分析结果过低、过于保守，造成结构过度设计、材料浪费。

为了（基于真实的试验数据）提高结构承载力，有时候会加做试验，比如10组或者20组甚至更多，数据越丰富分析得到的结果越准确，对修正计算模型亦是大有裨益。

可是问题在于，图1中计算公式中的临界值3.4仅适用于试验数量为5组的情况，更多试验组数的临界值该怎么取值？

下面小编一步一步解释图1公式的来历，顺带聊聊置信级、置信区间和保证率。

**所谓“置信级”、“置信区间”**

比方说你想要了解全国男性平均身高。

全国男性人口可太多了，而且随时有人来到这个世界、随时有人离开这个世界，人数本身就是动态变化的，所以你不可能挨个去调查。

但是你能做的是随机抽样然后根据样本数量和数据“预测”男同胞们的平均身高范围。既然是预测，那必然涉及准确性的问题。

上一段提到的“平均身高范围”，它有一个专有名词叫做**“confidence interval”——置信区间**。它指的是，在一定程度的可靠度下，你要预测的数值所在的范围。

这里小编说一下，为什么你要预测的不是某一单个数值而是一个范围？因为你不可能准确预测出全国男性身高具体是多少，无论你的预测值多少，概率都必然是1/全国男性人口数，几乎为0，这样的预测是毫无意义的。

但是如果你预测一个范围，那么这个范围就会囊括很多人，相应的概率（或准确性）就越大，比如你预测全国男性平均身高在0~3m之间，那么小编恭喜你，你的预测绝对正确，准确性高达100%，因为所有男性身高都在这个范围内，这个范围内身高的男性除以全国男性数量当然就是100%！

但是如果你预测在1.7~2m之间，那么相应的落在这个范围的男性数量就变少了，相应的分子变少，准确率（或概率）就低了。小编这段话里的“准确率（或概率）”也有一个专有名词叫做**“Confidence level”——即所谓的“置信级”**。

我们把上面啰嗦的一堆用图2来做进一步说明。

图2   样本正态分布

图2是典型的正态分布曲线（又称“高斯分布曲线”），基本上所有样品数据的分布都符合正态分布曲线的形式——两边小、中间大。

正态分布曲线类似回归曲线，可以把它看做是所有数据走势图，并不一定代表真正的样本数据。

比如在高斯曲线中，最大值（最高点）是曲线中间部位的所有数据的平均值，但是实际样品最大值不一定比这个平均值小，这一点可以从图2看出来。

我们把图2做进一步处理得到图3。图3就涉及到上面提到的置信区间和置信级了。

图3   高斯分布曲线

我们以平均值为界，把高斯曲线一分为二，左、右半边各占50%。

从中间往左、右对称取一定的百分比，比如34.1%，对应的横轴长度为S，这个S叫做**Standard Error——标准误差**。那么两个34.1%就是68.2%，它的含义就是“置信级”为68.2%，对应的“置信区间”则是（平均值-S，平均值+S）。

**这一区域的含义是，任何一个样本数据（包括你选取的和未选取的）落在这个置信区间（平均值-S，平均值+S）的概率是68.2%，换句话说你预测的范围准确率是68.2%！**

所以，我们可以归纳出计算置信区间的公式为：

**置信区间=X±z×S   公式1**

X指的是**“mean value”——样本平均值**；

z指的是**“critical value”——临界值**，不同置信级对应不同的临界值，例如90%置信级对应1.645，95%对应1.96，99%则对应2.58（z有一定的计算方式，以后有机会小编会单独开贴介绍）；

S指的是**标准误差（Standard Error）**，它等于样品**标准差s（sample standard deviation）**与**样本数量n**开根号之比S=s/n0.5。

结合z值，我们继续看图3，你会发现置信级越高，即准确率越高，那么临界值越大，反映到图3里，就意味着置信区间的区域越大。

这个逻辑是对的，你把范围放的越宽，能落到你预测范围的样品就越多，自然准确率就越高。再以90%的置信级为例，置信区间=（平均值-1.645S，平均值+1.645S）。

说到这里，我们就把置信级、置信区间说明白了。

**保证率**

小伙伴可能会有些困惑，怎么这个公式1和图1的不一样？错了，实际是一样的。

图1公式中的3.4就是公式1里面的临界值z，图1公式里面的V指的是变异系数，而变异系数等于标准误差与平均值之比，即：

**V=S/X   公式2**

图1公式里的Rv就是平均值X，所以图1公式跟公式1形式其实是一致的。

不过，为什么图1里面公式要把临界值取为3.4？这里就涉及本帖最后一个概念——保证率。

小编在开头就说了，根据置信级和保证率才能确定临界值。

可以简单的这么认为，一个置信级下面还设有多个保证率，每个保证率和不同样本数量结合之后各自对应一个临界值。

例如置信级90%下面还有保证率50%、75%、90%、95%、99%、99.9%，当样本数量为5个时，每个保证率对应的临界值分别为0.686、1.698、2.743、3.4、4.666、6.112，样本数量变化时，这些临界值也会变。

**置信区间和标准值的关系**

最后，小编再说一个比较关键的知识点，那就是为什么我们说了那么多置信区间，可是到了做试验的时候怎么只有一个确定的标准值而不是范围了？

这是因为我们在进行结构设计时，**构件的所谓承载力不可能是一个范围，它必须是一个确定的、单独的一个数值**。

比如说，你可以通过试验数据得到梁的承载力在5~7kN（即置信区间），但是在方案设计时，出于安全考虑，你必须选用下限5kN用于指导设计，即外荷载不能超过5kN，**这个置信区间的下限就是我们所说的“标准值”**。

如果你选7kN，就意味着在你看来外荷载6kN也是可接受的，可是你的试验实测数据里面有低于6kN的5kN，这样的设计就有超载的可能，是不安全的。

除非你想牢饭管饱，否则不建议这样做。

END

往期精选

[**建筑物理专栏**

绿色建筑 | 点热桥](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247484968&idx=1&sn=859a48d4391de7478b71d76965124366&chksm=fa5c00e1cd2b89f70453b21c547a61a4502d9c5b20cab275e941127ff43d10d11d899423f79f&token=802001461&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247484968&idx=1&sn=859a48d4391de7478b71d76965124366&chksm=fa5c00e1cd2b89f70453b21c547a61a4502d9c5b20cab275e941127ff43d10d11d899423f79f&token=802001461&lang=zh_CN&scene=21#wechat_redirect")[**混凝土锚固专栏**

结构漫谈 | 附加钢筋如何影响预埋件的锚固性能](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485005&idx=1&sn=e0bb26f54ad7286ac4ec9b4adbc97960&chksm=fa5c0084cd2b899258db2b0809ef3e4bdb94ef5599221050b4dd3756cf723204a0a8fbd8b7de&token=802001461&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485005&idx=1&sn=e0bb26f54ad7286ac4ec9b4adbc97960&chksm=fa5c0084cd2b899258db2b0809ef3e4bdb94ef5599221050b4dd3756cf723204a0a8fbd8b7de&token=802001461&lang=zh_CN&scene=21#wechat_redirect")[结构漫谈丨混凝土锚固 · 安全系数法](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485034&idx=1&sn=d9c7dc5c5a925d77d2ef005f9b77e9ff&chksm=fa5c00a3cd2b89b5fa076fddb1c573e7be683a4ebf9a8550e6d7a54d644d48c2f5bb9009c268&token=802001461&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485034&idx=1&sn=d9c7dc5c5a925d77d2ef005f9b77e9ff&chksm=fa5c00a3cd2b89b5fa076fddb1c573e7be683a4ebf9a8550e6d7a54d644d48c2f5bb9009c268&token=802001461&lang=zh_CN&scene=21#wechat_redirect")[混凝土锚固丨混凝土锚固破坏机制（理论研究）](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485170&idx=1&sn=809540901b004a022a82a9480c71f1bf&chksm=fa5c003bcd2b892d3a5e9b81a3df4b689bf7fcdd8d47d06e3e23e124631e986297043d06063a&payreadticket=HILvvrEvRVX5xliKWzqvMKMQG8N5h_5Taj5rJie24h7-5WjUyZeVZIDM28wxj0hFtAlmH6U&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485170&idx=1&sn=809540901b004a022a82a9480c71f1bf&chksm=fa5c003bcd2b892d3a5e9b81a3df4b689bf7fcdd8d47d06e3e23e124631e986297043d06063a&payreadticket=HILvvrEvRVX5xliKWzqvMKMQG8N5h_5Taj5rJie24h7-5WjUyZeVZIDM28wxj0hFtAlmH6U&scene=21#wechat_redirect")[结构漫谈丨欧美混凝土规范里的C25/30究竟是何方神圣？](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485132&idx=1&sn=c2a5b75964ddd9198b85e8d48a2e3b5c&chksm=fa5c0005cd2b8913a7081ca61fc3546e563339d7eff00eda7611eb2351e1079c7e7a32d6127c&token=462306672&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485132&idx=1&sn=c2a5b75964ddd9198b85e8d48a2e3b5c&chksm=fa5c0005cd2b8913a7081ca61fc3546e563339d7eff00eda7611eb2351e1079c7e7a32d6127c&token=462306672&lang=zh_CN&scene=21#wechat_redirect")[结构漫谈丨不要让你的结构，像你一样疲劳](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485141&idx=1&sn=47b67294eb9a030b0c69acef30f2b375&chksm=fa5c001ccd2b890a76b4be7a43bd0d87280f0880fc1116914e533828a055e1c8e3d677286132&token=317886162&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485141&idx=1&sn=47b67294eb9a030b0c69acef30f2b375&chksm=fa5c001ccd2b890a76b4be7a43bd0d87280f0880fc1116914e533828a055e1c8e3d677286132&token=317886162&lang=zh_CN&scene=21#wechat_redirect")[结构漫谈丨混凝土锚固·材料分项安全系数](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485188&idx=1&sn=f727eaaff664948403633b410c0215d8&chksm=fa5c01cdcd2b88dba3c1607429e6bb509462e4927e2f4506564f87df78cc5106e0e0dc76fcc8&token=979277831&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485188&idx=1&sn=f727eaaff664948403633b410c0215d8&chksm=fa5c01cdcd2b88dba3c1607429e6bb509462e4927e2f4506564f87df78cc5106e0e0dc76fcc8&token=979277831&lang=zh_CN&scene=21#wechat_redirect")

结构漫谈丨如何验算混凝土锚固的疲劳？

[欧洲预制混凝土吊件设计——最新技术动态](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485228&idx=1&sn=295c6171710e47d46d448f29ea268338&chksm=fa5c01e5cd2b88f353705a87752bf1ecd4ac6107d0aa1d5b774848fcc35cdd43ad01403ecc5f&token=394373383&lang=zh_CN&scene=21#wechat_redirect)

[附加钢筋——混凝土锚固的最佳辅助](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485314&idx=1&sn=bcc123648596cd50ab6eb360c509ed0e&chksm=fa5c014bcd2b885d860e0b29f6691f8ed0972042e8e8fcb3790664067cccbbb374c549f72b1e&token=2022847177&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485314&idx=1&sn=bcc123648596cd50ab6eb360c509ed0e&chksm=fa5c014bcd2b885d860e0b29f6691f8ed0972042e8e8fcb3790664067cccbbb374c549f72b1e&token=2022847177&lang=zh_CN&scene=21#wechat_redirect")[简单聊聊：混凝土锚固体系（预埋类）](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485682&idx=1&sn=8a895e71a90634231bf6b965d5eec659&chksm=fa5c0e3bcd2b872d6a62147f87f3601d6e315f2941d424db3002269072915aeef869864844de&token=1903907397&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485682&idx=1&sn=8a895e71a90634231bf6b965d5eec659&chksm=fa5c0e3bcd2b872d6a62147f87f3601d6e315f2941d424db3002269072915aeef869864844de&token=1903907397&lang=zh_CN&scene=21#wechat_redirect")

（续）附加钢筋——混凝土锚固的最佳辅助

混凝土自攻锚栓专栏 | 初识自攻栓

混凝土自攻锚栓专栏 | 安装自攻栓的正确姿势

混凝土自攻锚栓专栏 | 有关承载力（干货）

[叠合楼板中的锚固会影响结构安全吗？](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485396&idx=1&sn=90f272f7ef27498356f3f85d27d9b7c4&chksm=fa5c011dcd2b880bd7e1ce8440c90cd9b3dd1808be9ba6498a5fefd114945ba2b9ff05c8b111&token=893621076&lang=zh_CN&scene=21#wechat_redirect)

[混凝土锚固件剪翘凭什么就比抗拉强？](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485443&idx=1&sn=f2f9ea48f06987dac908d3f61f5f423d&chksm=fa5c0ecacd2b87dc6cd3ed7b145d0251bf12c72f8a39c4326fc51ff48186f1807d1f7d9280eb&token=394373383&lang=zh_CN&scene=21#wechat_redirect)

[混凝土锚固必须考虑混凝土开裂？](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485494&idx=1&sn=60288663396e24db8b6b8fdb07ebfffd&chksm=fa5c0effcd2b87e9f0f0e048bfca3cd3a10fee06f8cbb80147bd494f0d9ea3ba2da97e9425d0&token=1369742386&lang=zh_CN&scene=21#wechat_redirect)

[混凝土开裂专栏丨试验数据](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485521&idx=1&sn=d68d7efe43711370358ac582fee9f83c&chksm=fa5c0e98cd2b878eb5af4bef3a2b0a97a6b74361c9226b3e5523b4e888b1f132203813edd9a5&token=1369742386&lang=zh_CN&scene=21#wechat_redirect)

[混凝土开裂专栏丨裂缝宽度与锚固深度的影响](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485543&idx=1&sn=3b9ae4882dc50148c235b5f7e440ac33&chksm=fa5c0eaecd2b87b88beaa6fe76aed5cfa2f77b099bb62dd82ee37ccb1dccc4f24900f29e1e19&token=1678975942&lang=zh_CN&scene=21#wechat_redirect)

[镀锌，会削弱钢筋锚固力吗？](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485790&idx=1&sn=c31bce5f94c512f87bec251d5b0f4483&chksm=fa5c0f97cd2b8681f55eebf0f5d541b704f8fc4746cc0cc166190786952023f975b9badd1dbe&token=1401333904&lang=zh_CN&poc_token=HHsrQmajSYu7Ql2Ul7oA30T4UKpzBuyPhXZZPCQk&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485790&idx=1&sn=c31bce5f94c512f87bec251d5b0f4483&chksm=fa5c0f97cd2b8681f55eebf0f5d541b704f8fc4746cc0cc166190786952023f975b9badd1dbe&token=1401333904&lang=zh_CN&poc_token=HHsrQmajSYu7Ql2Ul7oA30T4UKpzBuyPhXZZPCQk&scene=21#wechat_redirect")[混凝土后锚固体系：膨胀锚栓（扭矩控制型）](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485911&idx=1&sn=ebde5c1fa9fd2cb8bffcc0cf80f276e0&chksm=fa5c0f1ecd2b86085bf22664695fb9570eb1690c84940f411e220e4155d4955941783f521403&token=1880289720&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485911&idx=1&sn=ebde5c1fa9fd2cb8bffcc0cf80f276e0&chksm=fa5c0f1ecd2b86085bf22664695fb9570eb1690c84940f411e220e4155d4955941783f521403&token=1880289720&lang=zh_CN&scene=21#wechat_redirect")

**应力集中专栏**

[应力集中如何影响产品设计？附实例计算](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485407&idx=1&sn=0620b9b292326ed3a2f6854218a92b17&chksm=fa5c0116cd2b8800d569b2be5be9af58e5c2d034300db58260b82ce50bca87010af40c26d7ca&token=1547157793&lang=zh_CN&poc_token=HMT-LGajP-92lRrisnb2g4WbsjXRAKAwmpWwr_7c&scene=21#wechat_redirect)

[应力集中问题的拓展——塑性设计](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485465&idx=1&sn=cda78d0eb2a00725e45f38405bbf5200&chksm=fa5c0ed0cd2b87c60c2ea4d96bf51ea5881b9cf6aea66cb0dc724d8fc0a15df6b4796ac00af5&token=1937520649&lang=zh_CN&scene=21#wechat_redirect)

[受弯构件：应力集中问题](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485734&idx=1&sn=81d2fad7a945be1da7911994c79123b0&chksm=fa5c0fefcd2b86f92b364cf9c9d90925e5c1e38ae5a4934e4e806f041ad518444e5e0958987a&token=1547157793&lang=zh_CN&scene=21#wechat_redirect)

**四大强度理论专栏：**

[简单聊聊——著名的“四大强度理论”解决的是什么问题？](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485574&idx=1&sn=31696ca957305b5a7ebf8f5257c9a480&chksm=fa5c0e4fcd2b87591496679fb0f401b9fb8eb985901cb587331d41bcb67ad08ed8b97cd6d698&token=66014374&lang=zh_CN&scene=21#wechat_redirect)

[著名“四大强度理论”之一：最大剪应力理论](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485585&idx=1&sn=3e62acce9dbae24b4298d7bdc5f7390d&chksm=fa5c0e58cd2b874eca8fc5386e2996dab297043943baded1b2f081f35c4f29b820dfe06faba7&token=267475671&lang=zh_CN&scene=21#wechat_redirect)

[著名“四大强度理论”之二：最大畸变能理论](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485593&idx=1&sn=a89865a9cf3fdf4944375b8b54398d08&chksm=fa5c0e50cd2b8746800d26bf4f241fc52c651397445e7541b4be2d99d93ed263d67d2ffbac11&token=472905394&lang=zh_CN&scene=21#wechat_redirect)

[著名“四大强度理论”之三：最大正应力理论](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485605&idx=1&sn=a8925907ec569bb0612a569a835d920b&chksm=fa5c0e6ccd2b877a0eb8f52956fb4f4fb95089c76b9a219ddb57c23d1cb2f5de7fc1db2ccb65&token=129573649&lang=zh_CN&scene=21#wechat_redirect)

[著名“四大强度理论”之四：莫尔失效理论](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485619&idx=1&sn=8a6396f41669664ea4d94557ebcfa3a6&chksm=fa5c0e7acd2b876c528fc20210c29cebbf7a03bf50ef36ff8d587f150eda13b20226079560f2&token=1174561834&lang=zh_CN&scene=21#wechat_redirect)

[聊聊两大强度理论对比——Tresca准则和Mises准则：哪个更安全？哪个更准确？](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485630&idx=1&sn=849b149ac386a969b4ade90f009872e2&chksm=fa5c0e77cd2b876116e6adcff0a059ef0fa4cb091d5af6347a7173d5d2f6efd9f92a6d09aef3&token=1174561834&lang=zh_CN&scene=21#wechat_redirect)

**莫尔圆专栏：**

[也来说说——神奇的“莫尔圆”](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485662&idx=1&sn=8515f431041bdabfad3623365e821d18&chksm=fa5c0e17cd2b87014512b906d1a2b89fac1b093c274533386ff693ff5a29afe609e04c68cc82&token=1426380965&lang=zh_CN&scene=21#wechat_redirect)

[莫尔圆应用之一：计算主应力、最大剪应力](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485671&idx=1&sn=50d03d916ec1e8011461af1a38410759&chksm=fa5c0e2ecd2b8738d6df1476ab11fadaee6b7d546969352c20d6cbf8cbbacf85b6390230821e&token=639662023&lang=zh_CN&scene=21#wechat_redirect)

[从莫尔圆角度，理解所谓的“应力转换”](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485694&idx=1&sn=de8b610a1ceeb8ccdf5edbe5ba3408d0&chksm=fa5c0e37cd2b87219dfe55b16f162b0c74c0b16a56365c3886de1f08867ae7f20ce055b63851&token=1225084215&lang=zh_CN&scene=21#wechat_redirect)

[实例计算：如何用莫尔圆确定主应力面](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485702&idx=1&sn=722698b1d59477389810be8304993461&chksm=fa5c0fcfcd2b86d981bbcb05d5107f83851952a93b767a5f85657840c45d27a22cd9726a96eb&payreadticket=HKWaGhdvhHoK7YLKIaKPNAIaUGMPMKTG2pngAkI8h9EvuwEVjC2aW4EfiEnTYLxvxxoIu2E&scene=21#wechat_redirect)

[莫尔圆应用之二：三维应力分析](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485750&idx=1&sn=d50b635305335e1e00eadb20523ddcad&chksm=fa5c0fffcd2b86e97f43fca0ffef96fb01395f0820d2d1597fd380fad4d78be0f2e0f670a6c1&token=1903907397&lang=zh_CN&scene=21#wechat_redirect)

[**受弯构件分析专栏：**](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485781&idx=1&sn=99891bbd376154f22b59946416ee26ac&chksm=fa5c0f9ccd2b868ae8e205785179685d69e64160856be27b7944ec5fb771f254192538c1c7df&token=376428570&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485781&idx=1&sn=99891bbd376154f22b59946416ee26ac&chksm=fa5c0f9ccd2b868ae8e205785179685d69e64160856be27b7944ec5fb771f254192538c1c7df&token=376428570&lang=zh_CN&scene=21#wechat_redirect")

[说说“截面惯性矩”](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485765&idx=1&sn=1cebdeb1ae55905d516692a72e6116d9&chksm=fa5c0f8ccd2b869a5325773857df67750244c3e10899dd8ca694ceeabef57f295d4d15c8628a&token=1903907397&lang=zh_CN&scene=21#wechat_redirect)

[“平行轴定理”的应用：T型梁截面惯性矩](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485781&idx=1&sn=99891bbd376154f22b59946416ee26ac&chksm=fa5c0f9ccd2b868ae8e205785179685d69e64160856be27b7944ec5fb771f254192538c1c7df&token=376428570&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485781&idx=1&sn=99891bbd376154f22b59946416ee26ac&chksm=fa5c0f9ccd2b868ae8e205785179685d69e64160856be27b7944ec5fb771f254192538c1c7df&token=376428570&lang=zh_CN&scene=21#wechat_redirect")[为什么构件受弯只分析单轴应力？](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485801&idx=1&sn=68898686620b014ab1b6f0ec7ac55bf4&chksm=fa5c0fa0cd2b86b655d6458c45234bb2aaa88c636b38b0d3d7c46e5568401a60fadf3fa7ec12&token=1401333904&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485801&idx=1&sn=68898686620b014ab1b6f0ec7ac55bf4&chksm=fa5c0fa0cd2b86b655d6458c45234bb2aaa88c636b38b0d3d7c46e5568401a60fadf3fa7ec12&token=1401333904&lang=zh_CN&scene=21#wechat_redirect")[谈一谈构件在弯矩作用下的受力分析](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485811&idx=1&sn=6329d664e6545775666d314d2efa776b&chksm=fa5c0fbacd2b86acd96553147c37bb406d5f3d36ec4d337e798a1cf379a4be384431b175e591&token=325430693&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485811&idx=1&sn=6329d664e6545775666d314d2efa776b&chksm=fa5c0fbacd2b86acd96553147c37bb406d5f3d36ec4d337e798a1cf379a4be384431b175e591&token=325430693&lang=zh_CN&scene=21#wechat_redirect")[从应变角度分析：构件受弯变形的本质](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485820&idx=1&sn=8823e158870a306ef3fb2c8fa2b18c09&chksm=fa5c0fb5cd2b86a36980b93788d95736814fa7c953b3af7b8557785c35d438da83006c5c6f87&token=1012173361&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485820&idx=1&sn=8823e158870a306ef3fb2c8fa2b18c09&chksm=fa5c0fb5cd2b86a36980b93788d95736814fa7c953b3af7b8557785c35d438da83006c5c6f87&token=1012173361&lang=zh_CN&scene=21#wechat_redirect")[浅谈复合材料构件受弯](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485831&idx=1&sn=a879f7aa2e7af1dc7ee537a5dde6d622&chksm=fa5c0f4ecd2b8658d34e13fb0a3bff0313db6722131abd3e653e2cbd80e14bf105f038da61cc&token=1012173361&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485831&idx=1&sn=a879f7aa2e7af1dc7ee537a5dde6d622&chksm=fa5c0f4ecd2b8658d34e13fb0a3bff0313db6722131abd3e653e2cbd80e14bf105f038da61cc&token=1012173361&lang=zh_CN&scene=21#wechat_redirect")[受弯构件：塑性阶段受力分析——塑性弯矩](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485854&idx=1&sn=a3a8f90a13476972aa639e3918b56981&chksm=fa5c0f57cd2b8641978045e2c3d24567d512e87576a276b1e4cb81eb0976e433e4375ce9f00b&token=1308756878&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485854&idx=1&sn=a3a8f90a13476972aa639e3918b56981&chksm=fa5c0f57cd2b8641978045e2c3d24567d512e87576a276b1e4cb81eb0976e433e4375ce9f00b&token=1308756878&lang=zh_CN&scene=21#wechat_redirect")[怎么理解构件的截面“形状系数”？](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485865&idx=1&sn=e2d889e5d01e0fc364e5b5d7635ee790&chksm=fa5c0f60cd2b8676f2897c84a4260ea32265b169eadb37ab878997b32d17ea4edecba7c11ba1&token=201541533&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485865&idx=1&sn=e2d889e5d01e0fc364e5b5d7635ee790&chksm=fa5c0f60cd2b8676f2897c84a4260ea32265b169eadb37ab878997b32d17ea4edecba7c11ba1&token=201541533&lang=zh_CN&scene=21#wechat_redirect")[怎么分析受弯构件的残余应力？](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485879&idx=1&sn=a59ffd964f31396b82ee3ae2179573c9&chksm=fa5c0f7ecd2b8668d331f93ae7b1f5b34aa0364c6fa875440a88ffc4024d067a959251b51a9d&token=1534817635&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485879&idx=1&sn=a59ffd964f31396b82ee3ae2179573c9&chksm=fa5c0f7ecd2b8668d331f93ae7b1f5b34aa0364c6fa875440a88ffc4024d067a959251b51a9d&token=1534817635&lang=zh_CN&scene=21#wechat_redirect")[型钢受弯实例计算：塑性弯矩、曲率、残余应力](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485893&idx=1&sn=08477a1ab578b9075758e8c72412daf3&chksm=fa5c0f0ccd2b861a4bf183146d1c39275ef3e4b7b003bc801101237e0cc4eaf211537c1c6d53&payreadticket=HPqal71Cj8dTEWyBVlQSS-DONB04S6c9_RLK5PRxTTZjAiJCFOkoh9Tzs2eqUKO-qb7NJ-c&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485893&idx=1&sn=08477a1ab578b9075758e8c72412daf3&chksm=fa5c0f0ccd2b861a4bf183146d1c39275ef3e4b7b003bc801101237e0cc4eaf211537c1c6d53&payreadticket=HPqal71Cj8dTEWyBVlQSS-DONB04S6c9_RLK5PRxTTZjAiJCFOkoh9Tzs2eqUKO-qb7NJ-c&scene=21#wechat_redirect")[聊聊“弯曲应力”](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485956&idx=1&sn=43d0c79619c6fbb1192eaa9cf901733f&chksm=fa5c0ccdcd2b85db8503ec1460bf6e6e38fcd1c6a6ec8fbd0dd7162ca46b011a1988d37fc774&token=491883982&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485956&idx=1&sn=43d0c79619c6fbb1192eaa9cf901733f&chksm=fa5c0ccdcd2b85db8503ec1460bf6e6e38fcd1c6a6ec8fbd0dd7162ca46b011a1988d37fc774&token=491883982&lang=zh_CN&scene=21#wechat_redirect")

**不锈钢专栏**

[欧洲人花了100年发明不锈钢](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485428&idx=1&sn=70c5bf50a2696e284d0c1f3bc728c450&chksm=fa5c013dcd2b882ba145642a2f84688fef7e9a9b33d44871e69cefe88e23e15584b54fa3f08a&token=1903907397&lang=zh_CN&scene=21#wechat_redirect)

[简单聊聊不锈钢5兄弟](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485842&idx=1&sn=d5941a8e4142cdaed7ff78047e1bc8de&chksm=fa5c0f5bcd2b864d5e2757d86cf3c641b48d51e411482fbb9588e533be1408bfeaa1bdf1aca4&token=413510252&lang=zh_CN&scene=21#wechat_redirect)

[“不锈钢”之所以不锈，那是因为······](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485932&idx=1&sn=c4039920e5c64883386ebfe20e5d49eb&chksm=fa5c0f25cd2b863343c95b94a6a4c7b2123b18c79c00bf87fc3c16f7d62f1bf9f0a26175ed92&token=1235853188&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485932&idx=1&sn=c4039920e5c64883386ebfe20e5d49eb&chksm=fa5c0f25cd2b863343c95b94a6a4c7b2123b18c79c00bf87fc3c16f7d62f1bf9f0a26175ed92&token=1235853188&lang=zh_CN&scene=21#wechat_redirect")

**想到哪写到哪**

[结构漫谈丨柔度，从欧拉公式的推导说开去](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485126&idx=1&sn=86e6a75183eb43d7cab3c5a344bb1ac5&chksm=fa5c000fcd2b8919eabd461a2f636c4787e5d48ea12755696999e1f956ade538b28270a5a64f&token=53513831&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485126&idx=1&sn=86e6a75183eb43d7cab3c5a344bb1ac5&chksm=fa5c000fcd2b8919eabd461a2f636c4787e5d48ea12755696999e1f956ade538b28270a5a64f&token=53513831&lang=zh_CN&scene=21#wechat_redirect")

[抗剪强度=0.58×抗拉强度的原因找到了！](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485289&idx=1&sn=8316c09082e4ecc71a24036890e0d765&chksm=fa5c01a0cd2b88b6ed61a0111ca16b06cad8391dd725f346cb585922723b71cbe7283cf3498c&token=394373383&lang=zh_CN&scene=21#wechat_redirect)

[这么理解“泊松比”就容易多了](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485243&idx=1&sn=7686fbe6eb133b2b9d4e8360094e7d79&chksm=fa5c01f2cd2b88e43fc7186f9fe30688e597e6f5ea717d890456c5a672c4d7adf701ad08e3c0&token=394373383&lang=zh_CN&scene=21#wechat_redirect)

[900字迅速了解2种钢材腐蚀【点腐蚀&缝隙腐蚀】](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485436&idx=1&sn=ea646bcbfdc81a9995253e24bc2b0e15&chksm=fa5c0135cd2b882310637312639aab15bc0ecb947c8169e9e810aa815ff96450b4fd2b468493&token=394373383&lang=zh_CN&scene=21#wechat_redirect)

[深度聊聊：“应变计”工作原理（附视频）](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485723&idx=1&sn=dff05422bce73e282523dc194fffadc3&chksm=fa5c0fd2cd2b86c48b312032aef2aa35c84291dffb68d4369b078cb6998ffcd3364bbafac769&token=1068662393&lang=zh_CN&scene=21#wechat_redirect)

[所谓“残余应力”](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485476&idx=1&sn=b4005e631998f6bdfc0078eee0daf987&chksm=fa5c0eedcd2b87fb4ce75112fc7ea67f8bfe9a236ccd0ed6cbc7333c31ff2702b0d6c7cf7fc2&token=1903907397&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485476&idx=1&sn=b4005e631998f6bdfc0078eee0daf987&chksm=fa5c0eedcd2b87fb4ce75112fc7ea67f8bfe9a236ccd0ed6cbc7333c31ff2702b0d6c7cf7fc2&token=1903907397&lang=zh_CN&scene=21#wechat_redirect")[不难理解——“温度应力”](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485486&idx=1&sn=8607309d6c70df6f6bb14cdc461d090e&chksm=fa5c0ee7cd2b87f1404b261883f5a404386393213c8239be45e4e86d8362860f156eb3691ed4&token=1903907397&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485486&idx=1&sn=8607309d6c70df6f6bb14cdc461d090e&chksm=fa5c0ee7cd2b87f1404b261883f5a404386393213c8239be45e4e86d8362860f156eb3691ed4&token=1903907397&lang=zh_CN&scene=21#wechat_redirect")[简单聊聊——圣维南原理](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485453&idx=1&sn=1d914d2e8e580dce61d4c3c597c845fa&chksm=fa5c0ec4cd2b87d2e0c6d55b4b8a7812cf52365d23b6b5f2e5857227b9a8b4b226c387946641&token=1903907397&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485453&idx=1&sn=1d914d2e8e580dce61d4c3c597c845fa&chksm=fa5c0ec4cd2b87d2e0c6d55b4b8a7812cf52365d23b6b5f2e5857227b9a8b4b226c387946641&token=1903907397&lang=zh_CN&scene=21#wechat_redirect")

[聊聊“胡克定律”本质——弹性模量](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485533&idx=1&sn=a4f2e8eb1afa58b9b4988daedf3a75cb&chksm=fa5c0e94cd2b87821f69c002abc30eac78b48a4b08878cbdf82c5a78703581e1086f837892be&token=1678975942&lang=zh_CN&scene=21#wechat_redirect)

[追本溯源——如何计算集中荷载作用下悬挑构件任意部位挠度？](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485553&idx=1&sn=54f055d949d3e3f08f6047b2c102c685&chksm=fa5c0eb8cd2b87aeb66b3dfe2b741e9ca2680935d36190c102643950bec175be6b420a3ab843&token=2131008240&lang=zh_CN&scene=21#wechat_redirect)

[深入聊聊为什么工字钢抗剪只考虑腹板？](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485562&idx=1&sn=3e9717650280ac28e2be6f5ac992aedd&chksm=fa5c0eb3cd2b87a5e8c82b1054374d0f54bae786350cd0bd147efb803c56aa59a544cc43268e&token=1323372124&lang=zh_CN&scene=21#wechat_redirect)

[焊缝的残余应力成因分析](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485924&idx=1&sn=fef69516dd53c75dc7fdb83e1253eafd&chksm=fa5c0f2dcd2b863bec31dc8136aab8bb87360b565d04c46ea23386755233ab2c428257fa8617&token=1235853188&lang=zh_CN&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485924&idx=1&sn=fef69516dd53c75dc7fdb83e1253eafd&chksm=fa5c0f2dcd2b863bec31dc8136aab8bb87360b565d04c46ea23386755233ab2c428257fa8617&token=1235853188&lang=zh_CN&scene=21#wechat_redirect")[怎么用微积分预测构件温度变形（翘曲/反拱）](https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485945&idx=1&sn=45e8ea13a4a4692620f177f26475722b&chksm=fa5c0f30cd2b862637681d0026e5ea7163e1897c8ed2a875982fd9e3bb7df959c72fb06fbe85&payreadticket=HC0xYhanJf35J2k6JPOgD2aBhaSBOm4vO2RavtUCKc4Wg5NAPxAC0qSaDEsFIdNXI2meKSg&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzUyOTY4MzM2Ng==&mid=2247485945&idx=1&sn=45e8ea13a4a4692620f177f26475722b&chksm=fa5c0f30cd2b862637681d0026e5ea7163e1897c8ed2a875982fd9e3bb7df959c72fb06fbe85&payreadticket=HC0xYhanJf35J2k6JPOgD2aBhaSBOm4vO2RavtUCKc4Wg5NAPxAC0qSaDEsFIdNXI2meKSg&scene=21#wechat_redirect")