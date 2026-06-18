> 已吸收至：[[05_数据分析与BI/0506_实验分析/050601_AB实验/050601_核心知识点/AB实验方差缩减与ROI评估|AB实验方差缩减与ROI评估]]
---
title: 「经验」AB实验效率翻倍！方差缩减神器CUPED
author: 小火龙说数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247518347&idx=1&sn=da08e2d1340a765bbfe6c0375469fed4&chksm=c07aca18f83e7d2c0710ef013962e9f60c4e6d1162645fa61bd6165c7a6f494bb723cec5288f&mpshare=1&scene=24&srcid=0604N3yNdPua45SvTpNd4rMy&sharer_shareinfo=902fef96556001e45fefcc9b1d7d3a7b&sharer_shareinfo_first=902fef96556001e45fefcc9b1d7d3a7b#rd
---

**00 CUPED是什么**

AB实验场景下，经常头大的问题「样本量不够用」。那试想，是否有一种方式，即能够减少样本量应用，又能够在灵敏度较高的情况下，得出置信结论。方差缩减神器CUPED，可以帮助你做到这一点。

CUPED（Controlled-experiment Using Pre-Experiment Data），名称挺长，不过原理很好理解，就是利用实验前的无偏数据，对实验「核心指标进行修正」，使得新指标的方差更低，得到更敏感的新指标，放大treatment的影响；同时，减少达到置信程度的样本量，缩减实验成本。

该方法由微软提出，通过协变量调整可将方差降低20%~50%，成为当前互联网大厂提升实验效率的“核武器”。

听起来是不是还挺有价值的，下面，详细为大家讲解下其核心原理，以及通过一个实际案例来看看如何应用。

**01 CUPED核心原理**

过多的推导公式，会让大家头疼，这里直接上核心公式。

直观理解一下：利用实验前的用户行为数据（X）预测实验指标（Y），剔除个体差异带来的噪声，使实验组与对照组的对比更聚焦于「因果效应」。同时，X与Y的相关性越强，ρ越大，方差缩减效果越显著。

**02 举一个小小的例子**

理论听起来如果不好理解，这里举一个例子，相信会更直白一些。

案例背景

某电商平台计划对首页进行改版（增加短视频推荐模块），希望通过AB实验验证新版首页是否能提升用户GMV（成交总额）。

核心指标：用户7日GMV。

AB实验难点

* GMV方差大：长尾分布，少量用户贡献大部分GMV。
* 所需用户量大：检测2%的GMV提升需要10万用户，耗时14天，流量成本高。

Step 1：选择协变量（Covariate）

目标：找到与目标指标（GMV）强相关，且不受实验干预影响的变量。

根据业务历史经验，选择候选变量如下。

通过Pearson相关系数，计算实验前各变量与GMV的相关性。

* 购买次数 vs GMV：ρ=0.68
* 活跃天数 vs GMV：ρ=0.52
* 平均点击量 vs GMV：ρ=0.45

选择「相关性最大的变量作为协变量」，这里选择「实验前30天购买次数」。

Step 2：计算调整系数θ

原理：通过线性回归Y=θX+ϵ，估计X对Y的预测系数θ，附上Python代码。

```
import numpy as np  from sklearn.linear_model import LinearRegression  # 从实验组和对照组全量数据中抽取样本（N=10000）  X = np.array([8, 2, 15, ..., 5, 12])  # 实验前购买次数  Y = np.array([450, 80, 1200, ..., 200, 600])  # 实验7日GMV  # 拟合线性模型  model = LinearRegression().fit(X.reshape(-1,1), Y)  theta = model.coef_  # 输出：theta=52.3  
```

通过计算得出，用户每增加1次历史购买，实验期GMV平均增加52.3元。

Step 3：计算调整后的GMV

通过以下公式，调整GMV指标。

这里，假设全量用户的历史购买次数均值 E[X] = 6.2，单用户指标调整如下。

这样调整，大家应该可以发现，高历史购买用户（如1003）的原始GMV被向下修正，避免个体差异掩盖实验效应；低历史购买用户（如1002）的GMV被向上修正，提升组间可比性。

Step 4：评估方差缩减效果

这里，对比下「原始GMV」与「调整后GMV」的统计数据差异。

可以发现，方差减少42%，检测灵敏度提升，如下样本量测算公式，所需样本量减少58%。

Step 5：假设检验与业务结论

通过采用T检验，验证实验效果。

通过以上数据可以看出，使用CUPED后，GMV提升的统计显著性从P=0.06（不显著）提升至P=0.01（显著），改版后首页GMV提升约13%（调整后均值差40元），置信区间排除零效应，项目可上线。

**03 一点点注意事项**

有同学可能会问，那在应用的过程中，有哪些地方有坑点，需要注意避开的吗？以下几点，大家要多多注意。

其一：协变量选择有讲究。协变量最好选择「强相关性、无干预影响、高覆盖率」的指标，避免出现短期波动，例如：不要选择「过去1天购买次数」这种指标。

其二：剔除极端值，防止过拟合。对于核心指标较大的用户，例如：GMV>10w的用户，单独分析，不进入主要回归流程。

其三：新指标数据验证。调整后的指标需要做方差验证，确保方差出现下降，同时，还要保证均值一致，防止数据出现偏差。

**1、实战进阶书籍《数据分析实践：专业知识和职场技巧》，侧重案例实操。**

目标群体：需要系统学习数据分析全流程，通过更多案例实现落地的同学。

详细介绍：**[《数据分析实践：专业知识和职场技巧》](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247515268&idx=1&sn=78dbb59d94f0b6ee328831ce6b772dd5&scene=21&token=188095369&lang=zh_CN#wechat_redirect)**

**2、将12年工作经验沉淀成「数据分析方法论图谱」，侧重场景与方法。**

目标群体：需要快速进阶，近期在准备面试的同学。

详细介绍：**[数据分析方法论图谱v2.0更新版](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247518247&idx=1&sn=b3d852323f11dab0e58e874eff91063c&scene=21&token=188095369&lang=zh_CN#wechat_redirect)**

**3、「简历修改、面试辅导、职业咨询」，助同学们成功上岸。**

目标群体：准备找工作、正在找工作的同学。

详细介绍：**[简历修改及面试辅导](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247515652&idx=1&sn=79d0da1e1eb34d3998092410804f6ea7&scene=21&token=188095369&lang=zh_CN#wechat_redirect)**

**往期推荐**

# [「经验」指标体系全景图『搜索场景』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247514940&idx=1&sn=5eb658ca4725cfd43a522e562ce1d289&chksm=c1405942f637d05428cedbefdef949748fbed6353ca680e779b4340a34f988b9cc230f0d8e3b&scene=21#wechat_redirect)

# [「经验」指标体系全景图『短视频场景』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247515456&idx=1&sn=d8b6bf01064c2f5da5a1aa9fa369a1df&chksm=c1405f3ef637d628a0a4135618225a657320eef9d9eb4fcaad52c9e0ed98de285544c99c142b&scene=21#wechat_redirect)

# [「经验」数据埋点很重要，这些内容你需要掌握『上篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247496930&idx=1&sn=e00eae4127e44f57e884a3be1a01f8b5&chksm=c140169cf6379f8aa36068bf5b90f1fa669d88a1558de69f1dce5d00ca5d63e3e52c0d57b87a&scene=21#wechat_redirect)

# [「经验」数据埋点很重要，这些内容你需要掌握『下篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247497187&idx=1&sn=bb179720bb62b29d5ecf274d3c0f32d1&chksm=c140179df6379e8b56925650c60cbb313f4c81ba95295b01f25461f4c2cabc1e94f1ee07a4bc&scene=21#wechat_redirect)

# [「经验」站在数据分析师角度，浅谈数据仓库需要掌握到的程度！](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247505375&idx=1&sn=61d8bce3a06011e8b736c56dde0d2751&chksm=c14037a1f637beb7b342ae37af0cef4e34966a2f25ce34532b3ac270ec3446326aef61e4ba8c&scene=21#wechat_redirect)

# [「经验」如何搭建“业务化”的指标体系？](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247484372&idx=1&sn=1b85e6c663b27f74ca12770ff305bb80&chksm=c143e1aaf63468bc7a605c82a1a2531f476ddd665a64d53e43086269e8221e49eecfc445688c&scene=21#wechat_redirect)

# [「经验」如何30min内排查出指标异动的原因『归因上篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247484504&idx=1&sn=ef89775220793ca2f0969b8aa3b06c4e&chksm=c143e626f6346f302b4ddbe9d290501523541b8339dc54a9bffc28adc8da0061600b91af505e&scene=21#wechat_redirect)

# [「经验」指标异动排查中，3种快速定位异常维度的方法『归因中篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247485346&idx=1&sn=e0e2bd33fbd6827d0a9fc3305a8e824b&chksm=c143e5dcf6346cca8d32b2f48496cee69e73fe60a56e015fdc4a01c7d984ff124020e65437a5&scene=21#wechat_redirect)

# [「经验」指标异动排查中，如何量化对大盘的贡献程度『归因下篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247486884&idx=1&sn=0a581f94e501ed41ce9fcab9fcb083fb&chksm=c143efdaf63466cc0e04deb46d5a13a081a59010ad09594d637ee95bd6ebc3016aeca33ff319&scene=21#wechat_redirect)

# [「经验」汇总指标异动的十大原因，涵盖日常90%问题](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247499823&idx=1&sn=eea95308f04e0943ad3780cec4984b09&chksm=c1402251f637ab474a84a1046dbc8939cde29c1204263d62e0fba984785b0971be27bc868d64&scene=21#wechat_redirect)

# [「经验」时间序列预测神器-Prophet『理论篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247494141&idx=1&sn=76761ecc7739508db2f9538f4eb4af41&chksm=c1400b83f637829519ae0b58e633dc503781737c7e48f6dd02062a72985a1bc786c533ee8840&scene=21#wechat_redirect)

# [「经验」时间序列预测神器-Prophet『实现篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247494192&idx=1&sn=53a9b77af75be1fa72ac4fb160ecce13&chksm=c140084ef6378158b641a960fc81381ae36cccf3d15bcdcfe41b59c9e0e3f519d01cd65fd57b&scene=21#wechat_redirect)

# [「经验」带你掌握AB实验最佳流程](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247488606&idx=1&sn=f2bbe3a7d9e51edcf6ad4e349339e994&chksm=c143f620f6347f360bdfff32cda9e1780620d1d16e57f9e5bb13713d666a30b5bbe8d93c3ec2&scene=21#wechat_redirect)

# [「经验」如何创建实验假设？这5步你需要掌握！『AB详解系列1』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247500564&idx=1&sn=91305bb3c3a2a4c3bc19f5ec3340b91a&chksm=c140216af637a87c43fa968359bc20e6efb9a01cb4dcdfe0c3e7495989015b8de21b6cba5aeb&scene=21#wechat_redirect)

# [「经验」不适合做AB实验的场景下，通过这4种方式来衡量策略效果](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247514265&idx=1&sn=a44021f3816bc0d3814b5772f003bc7b&chksm=c1405ae7f637d3f15fdda757c677a688c065a07258c46cca493d61d14d7df441abab9047e9f2&scene=21#wechat_redirect)

# [「经验」因果推断Matching方式实现代码](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247516013&idx=1&sn=9a5b2db7b9cdc1a9f29cf51392433764&chksm=c1405d13f637d405e3f4580313ade9856c0ea3928db120f072f35f149ec67e047f1fd23dd689&scene=21#wechat_redirect)

# [「经验」我对用户增长的理解『获客篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247493422&idx=1&sn=65ef4126450d293ff0aab84a1cc2d3a5&chksm=c1400550f6378c4617b2319e40ee48851aa05ff3920686b12cd752ed82a0b293f787d17c9c07&scene=21#wechat_redirect)

# [「经验」我对用户增长的理解『新用户篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247495430&idx=1&sn=5c44cffcbe6ec21ae964f4d6c8690867&chksm=c1400d78f637846ee24330a6f4909cdb0e03f10a1f90e6c746dc7ff67c7d80c5025fb9592464&scene=21#wechat_redirect)

# [「经验」我对用户增长的理解『流失预警篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247496329&idx=1&sn=2b3b23c3f11a7effcf1f13332c478c04&chksm=c14010f7f63799e13c4926afe6171e657392b3afc9b9e489e465953da7ee44984e6b7782cc94&scene=21#wechat_redirect)

# [「经验」用户画像对于业务如此重要？这几点你需要掌握！](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247500226&idx=1&sn=ee4f96fcf1c1514f4c15a8f290b6809b&chksm=c14023bcf637aaaac9e672e9b4a85cab6df3922ec0106c01e9b42783d54a41242606e702a015&scene=21#wechat_redirect)

# [「经验」用户增长渠道归因的五种常见方式](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247505640&idx=1&sn=a1291f3e62dd877253cd7dc1df93d481&chksm=c1403496f637bd8075b60eb3759c07e1b606a7d207a87a359c1cc1b12b53edf2d25e00b47f01&scene=21#wechat_redirect)

# [「经验」如何精准找到业务目标用户？提供几个小Tips！](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247517739&idx=1&sn=ef88ca2a9c1acb060ae9e524ccf6487d&scene=21#wechat_redirect)

# [「经验」如何做好探索性分析？这5步需要掌握！](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247496657&idx=1&sn=90dc8f0d589e8b62c64582c5974fde81&chksm=c14011aff63798b9f5a24a462beda38c90951221b130be075800e6d9673855db44e5794cf55a&scene=21#wechat_redirect)

# [「经验」相关性分析竟能带来如此大的业务价值？](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247501304&idx=1&sn=a2e9629f3ee30ec1eeea831b35c930c8&chksm=c1402786f637ae90bd840f1c1f367806f74ab466196a5b92a74047a015ff947f675ca24b1ffb&scene=21#wechat_redirect)

# [「经验」链路分析竟能带来如此大的业务价值？](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247500320&idx=1&sn=5007b02164f2d9a4199b66d60fc64a7c&chksm=c140205ef637a948cdf17d311b130f763794db8540292cbc2adf4965b2d563e2fe941689f8de&scene=21#wechat_redirect)

# [「经验」浅谈分类模型在工作中的应用，附上实战场景！](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247502351&idx=1&sn=0414e6fa585068d0e0e5881b75df523c&chksm=c1402871f637a167c72a415512fd2afb6407e5d5999a6ec2d7ee32dd797122ee04fdd6a740cc&scene=21#wechat_redirect)

# [「经验」浅谈聚类分析在工作中的应用](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247495197&idx=1&sn=03320cc139616c8aed1347d1fbc9815a&chksm=c1400c63f637857509ea1b05379d27d97ca539a39c736014ee4916e1fc45901cfe85fd0b1d69&scene=21#wechat_redirect)

# [「经验」数据分析这7个场景下，可以利用算法解决问题](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247495962&idx=1&sn=11bf90116ddb78621395b912ad6b289e&chksm=c1401364f6379a7229ec003a4fce50b0f7156393e7026f19a19afe93904ae84ae41d214f3985&scene=21#wechat_redirect)

# [「经验」爬虫在工作中的实战应用『理论篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247499015&idx=1&sn=2f3b9e9a6643a583adacdeb61789192b&chksm=c1401f79f637966ffc6ee73cab265dd87960c0893663085cc63de645b1c5922e7285f7705ff6&scene=21#wechat_redirect)

# [「经验」爬虫在工作中的实战应用『实现篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247499271&idx=1&sn=d45c7c37eb782f29645c668cb1a7237e&chksm=c1401c79f637956fab0af9b667b77358a3de9eb1393d42a74c3787ea5bf316c9b42737014dcf&scene=21#wechat_redirect)

# [「经验」互联网广告基础知识汇总『广告系列1』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247508821&idx=1&sn=7fb8f9d0a55135272d2bcfc35cf6358e&chksm=c140412bf637c83d8b9ab3e6c3c168cb455e747a8a1f2772fdd377fcd64155b4a2d06f57e70b&scene=21#wechat_redirect)

# [「经验」互联网广告出价及计费方式汇总『广告系列2』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247508877&idx=1&sn=d8f838e6c4d372843efbbe52bdade666&chksm=c14041f3f637c8e546571ce3bd2d5786c3911744a295ca991cad75b085ed3c864b56676c49ac&scene=21#wechat_redirect)

# [「经验」竞品分析需要掌握的思路及诀窍](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247510354&idx=1&sn=d624e38fc1754ff36ff43f793e461dff&chksm=c1404b2cf637c23af868570b0ef035cc63ea2aa5a1fd65f7b3942c49352e942035328cb8a317&scene=21#wechat_redirect)

# [「经验」用户成长体系对于业务的价值『概念篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247509555&idx=1&sn=b11dafbfd8b390102d1b954ea81242ba&chksm=c140444df637cd5be97a591478ce4324e50aecd255083065ebdfa344f839bd7c3c7d11b2ab52&scene=21#wechat_redirect)

# [「经验」用户成长体系对于业务的价值『玩法篇』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247512061&idx=1&sn=3ae51cee4aaa76d13b7bb4521f501206&chksm=c1404d83f637c495ae02ac7175b4a2e1ca96ac399448dfd6362acba5b143aca661f4094a71b6&scene=21#wechat_redirect)

# [「经验」从0到1撰写行业研报的核心思路](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247512463&idx=1&sn=a6dd28836e17085eb1352dd21d587162&chksm=c14053f1f637dae73f4c3e09b63809188dfc424cb12302493e2e54f3b88aa936b6a47ec38df8&scene=21#wechat_redirect)

# [「经验」短视频0vv专项分析『实战案例1』](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247512965&idx=1&sn=007d867e63949d6ce8d80ccd7e12d1eb&chksm=c14051fbf637d8eda3edad1128f23b34604a92d998bd6e7ffa292a5307dc9bb94da477cc15e7&scene=21#wechat_redirect)

# [「经验」浅谈视频质量评估方式](http://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247514497&idx=1&sn=a1c2bf361750f0f867e4014174e6e1bc&chksm=c1405bfff637d2e9df67b61882778be96a778e73e9c0fd77216639d7837096b0d21a7b6af02f&scene=21#wechat_redirect)

# [「经验」活动效果评估！数据分析师必备能力](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247518187&idx=1&sn=26cd2e4f21803902c8ddf24376c7f45e&scene=21#wechat_redirect)

# [「经验」点击率暴涨19%！数据人必看的AB实验爆款秘籍](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247518229&idx=1&sn=bf33e76804ec48db34924c3296264dc3&scene=21#wechat_redirect)

# [「经验」学会『边际ROI分析法』，从此告别广告预算浪费！](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247518269&idx=1&sn=659707fcf5b5c131705afefc85e71ee3&scene=21#wechat_redirect)

#

**持续追更哦**