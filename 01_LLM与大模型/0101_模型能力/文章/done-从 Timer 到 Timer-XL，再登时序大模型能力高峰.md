---
title: 从 Timer 到 Timer-XL，再登时序大模型能力高峰
author: Apache IoTDB
date:
url: http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247501373&idx=1&sn=39841cbf891df42a268a936920e5af2a&chksm=fc0df172989e2de1c15013638feb7c3e764a41a76f5fb608cad497d1724908c009998db517c4&mpshare=1&scene=24&srcid=0326lXZmk54X4MT7AsGkdQ7W&sharer_shareinfo=eaec407aa7bf9624fca035bfe428aa32&sharer_shareinfo_first=eaec407aa7bf9624fca035bfe428aa32#rd
---
> 已吸收至：[[01_LLM与大模型/0101_模型能力/0101_核心知识点/模型能力来源校准与跨域路由准则|模型能力来源校准与跨域路由准则]]

IoTDB 团队自研时序大模型成果，预测效果卓越，可应用于工业时序分析多场景！

时序数据库在支持时序特性写入、存储、查询等功能后，正朝着深度分析方向发展。自动化异常监测与智能化趋势预测，成为时序数据管理的核心需求。

时序数据库 IoTDB 团队不断探索，自研推出了面向时间序列的大模型 Timer 和扩展版 Timer-XL，性能取得多项突破并在国际机器学习顶级会议发表成果，为时序分析预测提供了更高效的解决方案。

|  |  |
| --- | --- |
| **01** | Timer 模型 |
| **02** | Timer-XL 模型 |
| **03** | 模型使用场景 |
| **04** | 模型效果验证 |

01

**Timer 模型**

深度时序分析通用基础模型 Timer 和时序数据库 IoTDB 一样，发源于清华大学软件学院，具备可观的分析能力和对真实世界数据的理解能力。通过显著的少样本能力和多任务适配能力，Timer 模型能够处理多样化的下游任务，为多类实际应用场景提供通用解决方案。

Timer 模型拥有以下特点：

* 泛化性：模型能够通过使用少量样本进行微调，达到行业内领先的深度模型预测效果。

* 通用性：模型设计灵活，能够适配多种不同的任务需求，并且支持变化的输入和输出长度，使其在各种应用场景中都能发挥作用。

* 可扩展性：随着模型参数数量的增加或预训练数据规模的扩大，模型的性能会持续提升，确保模型能够随着时间和数据量的增长而不断优化其预测效果。

02

**Timer-XL 模型**

作为 Timer 模型的扩展版本，Timer-XL 模型在继承其优秀特性的基础上，实现了三大核心突破：

* 扩展上下文输入：该模型突破了传统时序预测模型的限制，支持处理数千个Token（相当于数万个时间点）的输入，有效解决了上下文长度的瓶颈问题。

* 多变量预测场景覆盖：支持多种预测场景，包括非平稳时间序列的预测、涉及多个变量的预测任务以及包含协变量的预测，满足多样化的业务需求。

* 大规模工业时序数据集训练：采用万亿大规模工业物联网领域的时序数据集进行预训练，数据集兼有庞大的体量、卓越的质量和丰富的领域等重要特质，覆盖能源、航空航天、钢铁、交通等多领域。

03

**模型使用场景**

目前，Timer 模型已经内置在时序数据库 IoTDB 的智能分析节点 AINode 中，用户能够非常方便地进行调用。得益于 Timer 模型的优异性能，时序数据库 IoTDB 可以有效地为异常检测、数据填补、时序预测等工业场景提供解决方案。

**(1) 异常检测**

利用时序大模型精准识别与正常趋势偏离过大的异常值，可支持静态阈值告警和动态阈值告警，实现告警自动化。

下图蓝色曲线代表预测告警数据，红色曲线为实际数据趋势，可见预测告警结果准确性较高。

**(2) 数据填补**

调用时序大模型进行预测，在缺失值可能发生之前，使用预测值对缺失数据进行补全，实现补全算法智能化，不局限于线性插值补全。

下图蓝色曲线代表预测填补数据，红色曲线为实际数据趋势，两曲线反映趋势高度吻合。

**(3)****时序预测**

利用时序大模型的预测能力，可原生支持多变量预测，能够准确预测时间序列的未来变化趋势，相比单变量预测实现更优的预测准确性。

下图蓝色曲线代表预测趋势，红色曲线为实际趋势，两曲线高度吻合。

04

**模型效果验证**

多类大模型的预测结果对比中，Timer-XL 模型在零样本预测、单变量预测、多变量预测、协变量预测场景，结果准确性都优于其他大模型，并具有更强的泛化性能，可以在训练集之外的数据集保持高可靠性。

多变量预测方面，Timer-XL 模型预测效果达到了 SOTA 水平（State-of-the-Art，在该领域当前达到的最佳性能水平）。协变量预测方面，Timer-XL 的表现优于 SOTA 水平的专门化模型。

上述成果代表 Timer-XL 模型能够熟练地在大量数据中自适应地选择信息，从而实现卓越的预测性能，充分证明了 Timer-XL 模型在时序预测领域的领先地位与强大实力。

来源：ICLR 2025 论文《Timer-XL: Long-Context Transformers For Unified Time Series Forecasting》

未来，IoTDB 团队将持续优化时序大模型架构及功能，拓展其在工业多领域、多场景应用，推动时序数据分析进入智能决策新阶段。

**规上企业应用实例**

**能源电力：[中核武汉](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247497813&idx=1&sn=1f75ff66a325a1e13b786607dd8f9a92&chksm=fdfbbc90ca8c35862e82f4de58328336d90f177a423cdb2e8065c672c8fee781c127ae8f56ac&scene=21#wechat_redirect)｜[国网信通产业集团](http://mp.weixin.qq.com/s?__biz=Mzg4OTcyNzA3NQ==&mid=2247485447&idx=1&sn=c093643695ae91e4aee9a4b9ecbaf2a5&chksm=cfe6390bf891b01d6f1b6a09e222b2e10ff389871037bfb1942f4e933efc9c10cabcffb2a1eb&scene=21#wechat_redirect)｜[华润电力](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247497404&idx=1&sn=c27fbcd24adb783ad4d2420179cacf62&chksm=fdfbb279ca8c3b6f20431ed86449dbff060deb4573ea8025650c3604602136e08315151f97bb&scene=21#wechat_redirect)｜[大唐先一](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247497802&idx=1&sn=d5020ba8b143a07d3cc4a66e8fe4d48c&chksm=fdfbbc8fca8c3599bd6ec71e96174c29922f4205a28c22cf24a14df6c59ac50cdc276070c09a&scene=21#wechat_redirect)｜[上海电气国轩](http://mp.weixin.qq.com/s?__biz=Mzg4OTcyNzA3NQ==&mid=2247486346&idx=1&sn=bad65b546c5b1ef08870269fd2d0c719&chksm=cfe63a86f891b39015494ba0ad1b37a12e143dc0ce154b5c635fcde5a5356866d0b7d5b2ae60&scene=21#wechat_redirect)｜[清安储能](http://mp.weixin.qq.com/s?__biz=Mzg4OTcyNzA3NQ==&mid=2247485619&idx=1&sn=e3259a2af3b46d06c7ca1bda42a11897&chksm=cfe639bff891b0a912a850df4423be989b99b4e467a9b73db719cdf5e3f9c823e9c9b06d5732&scene=21#wechat_redirect)｜[某储能厂商](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247499237&idx=1&sn=94b14735fd11e705d8145c462c4a361d&chksm=fdfbb920ca8c303657d612ef7814b59fec79ad1abbc0b5cf814b290ce904beb473208650d397&scene=21#wechat_redirect)**｜[太极股份](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247487315&idx=1&sn=7e7c1eb7ea94075ed24a36dd326388ac&chksm=fdf84b96ca8fc280e223f147623ffa337b0418a4c508adfee35161403165e3dfb96509d7b67c&scene=21#wechat_redirect)

**航天航空：**[中航机载共性](http://mp.weixin.qq.com/s?__biz=Mzg4OTcyNzA3NQ==&mid=2247485520&idx=1&sn=1b4fe1708563d9df5c48999830d2b8e9&chksm=cfe6395cf891b04a7b24a61de9fd6aa70e469a4fc23e343e5e135224ed7d11a353b758ab7cdd&scene=21#wechat_redirect)｜[北邮一号卫星](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247493702&idx=1&sn=eaf85c97af4added0b2281ee968f2c9a&chksm=fdfbac83ca8c259578ebfa698b291d487f669fa922e93ed1e05939b504f45d1dd74971b68ef6&scene=21#wechat_redirect)

**钢铁、金属冶炼：**[宝武钢铁](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247499884&idx=1&sn=b99ca9605e8e0aed3c09ce7c67fb7a79&chksm=fdfb84a9ca8c0dbf66073ded9e2f4d06a35d7f2a2dfbe5d47ba4c712eeb1a1264ee57ce74af5&scene=21#wechat_redirect)｜[中冶赛迪](http://mp.weixin.qq.com/s?__biz=Mzg4OTcyNzA3NQ==&mid=2247484436&idx=1&sn=302667e7e4edb0332b7534749d587828&chksm=cfe63518f891bc0e917dcfc5d5aac7b570e598837d3ba41ce280262387635c65bbefc590392c&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=Mzg4OTcyNzA3NQ==&mid=2247484436&idx=1&sn=302667e7e4edb0332b7534749d587828&chksm=cfe63518f891bc0e917dcfc5d5aac7b570e598837d3ba41ce280262387635c65bbefc590392c&scene=21#wechat_redirect)[中国恩菲](https://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247500103&idx=1&sn=f421031a0e0404a8313ee23a834e5d69&scene=21#wechat_redirect)

**交通运输：**[中车四方](http://mp.weixin.qq.com/s?__biz=Mzg4OTcyNzA3NQ==&mid=2247483791&idx=1&sn=ef6ec63516b29b79b5e77d1d8adfc551&chksm=cfe63083f891b9954829ce6363eb2065b7407262a23c4a4eb591b8c84d0bf45fee934aba658d&scene=21#wechat_redirect)｜[长安汽车](http://mp.weixin.qq.com/s?__biz=Mzg4OTcyNzA3NQ==&mid=2247486451&idx=1&sn=19de14ad459b737f9dfe07a58cd020a2&chksm=cfe63afff891b3e9e0a9bf17e1f3a42e824c889c2fe149629bc5fc71bc7698b34392315c2178&scene=21#wechat_redirect)｜[城建智控](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247499610&idx=1&sn=25ed45ab71ed483d407b0d26734d7846&chksm=fdfbbb9fca8c32891219a8d963f8f5a92a5c89716f1838ce86d2247d12461e9909bf8542fb0f&scene=21#wechat_redirect)｜[德国铁路](https://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247501300&idx=1&sn=8710856bf8ee59eb9015d11f8c657c09&scene=21#wechat_redirect)

**智慧工厂与物联：**[PCB 龙头企业](http://mp.weixin.qq.com/s?__biz=Mzg4OTcyNzA3NQ==&mid=2247484322&idx=1&sn=cc4f9757959001d82cafa050ab585626&chksm=cfe632aef891bbb883f5fc2d3e5abd01ccfd8e4a9ca8a890ec4d7f6160418701cf11fa2429e1&scene=21#wechat_redirect)｜[博世力士乐](http://mp.weixin.qq.com/s?__biz=Mzg4OTcyNzA3NQ==&mid=2247484444&idx=1&sn=aacfb33aae3da196de863424c871995b&chksm=cfe63510f891bc0604bfb97ad8e0999cb90e88fec2b9b13a319bb03870483329eb328361b703&scene=21#wechat_redirect)｜[德国宝马](https://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247501300&idx=1&sn=8710856bf8ee59eb9015d11f8c657c09&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247497926&idx=1&sn=2e0dddb269072a8ad401d6e3cde9f702&chksm=fdfbbc03ca8c3515d33e905f860654fdf9e9b286790b26f9716aac02c2ced4c791a6a1be43ef&scene=21#wechat_redirect)[北斗智慧物联](https://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247500027&idx=1&sn=0aa7325936c113b807ca639193ea7bd9&scene=21#wechat_redirect)｜[某物联大厂](https://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247501256&idx=1&sn=97e66022e2adffcb72ddaed170940393&scene=21#wechat_redirect)｜[昆仑数据](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247497921&idx=1&sn=0e3057bda45c6d411d5911104ec7faae&chksm=fdfbbc04ca8c3512ea0758c1c3ab78177da7e544ce147a6545ad2d8d34115cfb8e116633021a&scene=21#wechat_redirect)｜[怡养科技](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247486814&idx=1&sn=f3f1b0f1dbd42b4d4eb2eefc0a36a1ff&chksm=fdf8499bca8fc08d7bf306cf3852e853edc29ae6f825d8d70b0947ff787dcb64c29a90eca28b&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247486814&idx=1&sn=f3f1b0f1dbd42b4d4eb2eefc0a36a1ff&chksm=fdf8499bca8fc08d7bf306cf3852e853edc29ae6f825d8d70b0947ff787dcb64c29a90eca28b&scene=21#wechat_redirect)[绍兴安瑞思](http://mp.weixin.qq.com/s?__biz=MzU4NjU4NTUxNA==&mid=2247486543&idx=1&sn=ad1dad1b6f5c456d4bd209ef6ea22933&chksm=fdf8488aca8fc19c79ebdf0a3374a7d4687caf45fec9e5d042a18b33321c69b1da164fec24fe&scene=21#wechat_redirect)
