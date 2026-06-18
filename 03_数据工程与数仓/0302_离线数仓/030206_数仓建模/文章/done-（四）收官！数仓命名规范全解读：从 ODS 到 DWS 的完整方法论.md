> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030206_数仓建模/030206_核心知识点/命名规范与物理模型设计|命名规范与物理模型设计]]
---
title: （四）收官！数仓命名规范全解读：从 ODS 到 DWS 的完整方法论
author: 白鲸开源
date:
url: https://mp.weixin.qq.com/s/A3BJ5mtGORTmQ4T759qvzA
---

点击蓝字，关注我们

**《新兴数据湖仓设计与实践手册·数据湖仓建模及模型命名规范（2025年）》** 由四篇递进式指南组成，以“模型架构—公共规范—分层规范—命名规范”为主线，系统构建可演进、可治理、可共享的现代数据湖仓。

本文为系列文章收尾篇，依次详细剖析了**数仓各层的命名规范**，帮助企业用一套方法论完成从数据入湖到价值变现的全链路建设。

至此，本系列文章完结，前三篇内容请下拉至文末查看。🔽

**词根设计规范**

词根属于数仓建设中的规范，属于元数据管理的范畴，现在把这个划到数据治理的一部分。完整的数仓建设是包含数据治理的，只是现在谈到数仓偏向于数据建模， 而谈到数据治理，更多的是关于数据规范、数据管理。

表命名，其实在很大程度上是对元数据描述的一种体现，表命名规范越完善，我 们能从表名获取到的信息就越多。比如：一部分业务是关于货架的，英文名是：rack， rack 就是一个词根，那我们就在所有的表、字段等用到的地方都叫 rack，不要叫成 别的什么。这就是词根的作用，用来统一命名，表达同一个含义。

指标体系中有很多“率”的指标，都可以拆解成 XXX+率，率可以叫 rate，那我 们所有的指标都叫做 XXX\_rate。

**词根：可以用来统一表名、字段名、主题域名等等。**

举例：以流程图的方式来展示，更加直观和易懂，本图侧重 dwm 层表的命名 规范，其余命名是类似的道理：

第一个判断条件是该表的用途，是中间表、原始日志还是业务展示用的表 如果该表被判断为中间表，就会走入下一个判断条件：表是否有 group 操作 通过是否有 group 操作来判断该表该划分在 dwd 层还是 dwm 和 dws 层 如果不是 dwd 层，则需要判断该表是否是多个行为的汇总表（即宽表） 最后再分别填上事业群、部门、业务线、自定义名称和更新频率等信息即可。

* **分层**

  表的使用范围
* **事业群和部门**

  生产该表或者该数据的团队
* **业务线**

  表明该数据是哪个产品或者业务线相关
* **主题域**

  分析问题的角度，对象实体
* **自定义**

  一般会尽可能多描述该表的信息，比如活跃表、留存表等
* 更新周期：比如说天级还是月级更新

**数仓表的命名规范如下：**

1. **数仓层次**

   公用维度：dim
   DM层：dm
   ODS层：ods
   DWD层：dwd
   DWS层：dws
2. **周期/数据范围**

   日快照：d
   增量：i
   全量：f
   周：w
   拉链表：l
   非分区全量表：a

**表命名规范**

1. **常规表**

   常规表是我们需要固化的表，是正式使用的表，是目前一段时间内需要去维护去 完善的表。

**规范：分层前缀[dwd|dws|ads]\_部门\_业务域\_主题域\_XXX\_更新周期|数据范围**

业务域、主题域我们都可以用词根的方式枚举清楚，不断完善。
更新周期主要的是时间粒度、日、月、年、周等。

2. **中间表**

   中间表一般出现在 Job 中，是 Job 中临时存储的中间数据的表，中间表的作 用域只限于当前 Job 执行过程中，Job 一旦执行完成，该中间表的使命就完 成了，是可以删除的（按照自己公司的场景自由选择，以前公司会保留几天 的中间表数据，用来排查问题）。

**规范：mid\_table\_name\_[0~9|dim]**

table\_name 是我们任务中目标表的名字，通常来说一个任务只有一个目标表。这里加上表名，是为了防止自由发挥的时候表名冲突，而末尾大家可以选择自由发挥，起一些有意义的名字，或者简单粗暴，使用数字代替，各有优劣吧，谨慎选择。
通常会遇到需要补全维度的表，这里使用 dim 结尾。
如果要保留历史的中间表，可以加上日期或者时间戳。

3. **临时表**

   临时表是临时测试的表，是临时使用一次的表，就是暂时保存下数据看看，后续一般不再使用的表，是可以随时删除的表。

**规范：tmp\_xxx**

只要加上 tmp 开头即可，其他名字随意，注意 tmp 开头的表不要用来实际使用，只是测试验证而已。

4. **维度表**

   维度表是基于底层数据，抽象出来的描述类的表。维度表可以自动从底层表抽象出来，也可以手工来维护。

**规范：dim\_xxx**

维度表，统一以 dim 开头，后面加上，对该指标的描述。

5. **手工表**

   手工表是手工维护的表，手工初始化一次之后，一般不会自动改变，后面变更，也是手工来维护。
   一般来说，手工的数据粒度是偏细的，所以暂时统一放在 dwd 层，后面如果有目标值或者其他类型手工数据，再根据实际情况分层。

**规范：dwd\_业务域\_manual\_xxx**

手工表，增加特殊的主题域，manual，表示手工维护表。

例如：**dwd\_t01\_client\_info\_l 就是dwd层，t01主题域（业务域），名字是client\_info的拉链表**。

**字段命名规范**

1. **公共规则**

* 所有单词小写
* 单词之间下划线分割（反例：appName 或 AppName）
* 可读性优于长度 (词根，避免出现同一个指标，命名一致性)
* 禁止使用 sql 关键字，如字段名与关键字冲突时 +col
* 数量字段后缀 \_cnt 等标识...
* 金额字段后缀 \_price 标识
* 天分区使用字段 dt，格式统一（yyyymmdd 或 yyyy-mm-dd）
* 小时分区使用字段 hh，范围（00-23）
* 分钟分区使用字段 mi，范围（00-59）
* 布尔类型标识：is\_{业务}，不允许出现空值

2. **指标命名规范**

   结合指标的特性以及词根管理规范，将指标进行结构化处理。

1. 基础指标词根，即所有指标必须包含以下基础词根：
2. 业务修饰词，用于描述业务场景的词汇，例如trade-交易。
3. 日期修饰词，用于修饰业务发生的时间区间。
4. 聚合修饰词，对结果进行聚集操作。
5. 基础指标，单一的业务修饰词+基础指标词根构建基础指标 ，例如：交易金额-trade\_amt。
6. 派生指标，多修饰词+基础指标词根构建派生指标。派生指标继承基础指标的特性，例如：安装门店数量-install\_poi\_cnt。
7. 普通指标命名规范，与字段命名规范一致，由词汇转换即可以。

## **小结**

完成上述模型设计工作之后，就可以进入下一个环节——利用白鲸开源的数据集成调度一体化平台 WhaleStudio 进行数据采集同步 ETL 任务以及不同数据仓库表之间 SQL、Python 等任务的设计，请参考下一个系列文章——《数据湖仓与 DataOps 开发规范》。

白鲸开源 WhaleStudio 是由大数据调度平台 Apache DolphinScheduler 和新一代多模态、高性能的海量数据同步工具 Apache SeaTunnel 原班人马打造的商业版数据同步与调度工具，提供功能更多、稳定性更强的商业版本，以解决用户调度、数据开发、数据同步和 ETL 的问题，目前支持 200+ 种数据库（信创、云、开源、ERP 等）的 ETL 与数据开发，目前已在中信证券、中信建投等多个行业头部企业实现成功应用。

对 WhaleStudio 感兴趣的用户可直接发送邮件至service@whaleops.com 进行详细咨询，或扫码二维码了解详情：

**·END·**

**白鲸开源**

白鲸开源是一家开源原生的DataOps商业公司，是国家高新技术企业，由多个Apache Foundation Member成立，80%员工都是 Apache Committer，运营2个全球Apache开源项目(DolphinScheduler, SeaTunnel）。白鲸开源已根据全球最佳实践发布商业版产品WhaleStudio(含白鲸数据调度平台WhaleScheduler和白鲸数据集成平台WhaleTunnel）。我们致力于打造下一代开源原生的DataOps 平台，助力企业在大数据和云时代，智能化地完成多数据源、多云及信创环境的数据集成、调度开发和治理，以提高企业解决数据问题的效率，提升企业分析洞察能力和决策能力。

**了解更多**

公司网站:www.whaleops.com

联系邮箱: xiyan@whaleops.com

如果您希望深入了解文中提到的数据质量功能，或者讨论如何将 WhaleStudio 与你的业务流程相结合，我们非常愿意为你提供帮助。**欢迎扫码获取****WhaleStudio产品白皮书**。

---

**下滑探索**更多WhaleStudio的优势，让我们帮助你构建一个高效、安全的大数据解决方案。🚀

## 金融行业的应用实例

****↓↓↓**点击下面链接阅读↓↓↓**

[国内某头部理财服务提供商基于白鲸调度系统建立统一调度和监控运维](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485832&idx=1&sn=28d8b40b2a752a431359a8cede446542&chksm=c171c1daf60648cc1724b11fc42a52c16ac0b8e29d26d6208cf897ba9c613ae0a1be152638a3&scene=21#wechat_redirect)

[白鲸调度系统助力国内头部券商打造国产信创化 DataOps 平台](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485636&idx=1&sn=a32f18255fdbf8b3510181ecf5d24a0d&chksm=c171c096f60649807f7c65a1a9b8de96b990539177752d3d31b21d9e15720d09f7647d1f8b98&scene=21#wechat_redirect)

[白鲸开源 DataOps 平台助力证券行业实现信创数字化转型](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247484746&idx=1&sn=545477e0c76725117276d0aa0ab157b7&chksm=c171cd18f606440e0e683dfe8f346f98965f0dcd85def2ebb51371a871d202d40792567371a1&scene=21#wechat_redirect)

[最佳实践 | 从Airflow迁移到Apache DolphinScheduler](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485916&idx=1&sn=b9aa20d813ea8c729c601167a2e31183&chksm=c171c18ef6064898a0b996fd9f83c9ad0a6d507d0a5429ff0fd2b407112fd67ec3fd7382c79b&scene=21#wechat_redirect)

[Apache DolphinScheduler VS WhaleScheduler](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485887&idx=1&sn=59b4ca05a425734b02b5e530eeb082a6&chksm=c171c1edf60648fb49ed9ad7a9b6315a470daff4f22534e52a7a017fd10f3084f2fd006f5fcd&scene=21#wechat_redirect)

[代立冬：基于Apache Doris+WhaleTunnel 实现多源实时数据仓库解决方案探索实践](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247486052&idx=1&sn=b869e1243e2b1e65f3435c73efb6b1f4&chksm=c171c236f6064b2059ff35e5e104392d2b82801158d531eca1e5def35b474eacef78edb29ea6&scene=21#wechat_redirect)

[白鲸开源在中信建投 DataOps 应用实践](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247486014&idx=1&sn=b157c765dee67f7a53dcb73efffaf871&chksm=c171c26cf6064b7a472bf00207ba93a88b23a164b9c14a84b4a35212e8060ee8955b69a5ad21&scene=21#wechat_redirect)

## 商业版技术解析实例

**点击下面链接阅读↓↓↓**

[被热议的“DataOps”是炒作？](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247486151&idx=1&sn=81cd58a5d2d4248570ec1413b4a9111a&chksm=c171c295f6064b83d52e3f782ea99377355f06811ecb22ad32661c1d2e233d2add7cfcb84760&scene=21#wechat_redirect)

[WhaleScheduler：高并发下的稳定性与性能实践](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485953&idx=1&sn=7a9077f6e6b4067df585b16f6894cc09&chksm=c171c253f6064b454931eac815afc084110ab6c89098e2b6bb9255fa9e90b3834b873284f859&scene=21#wechat_redirect)

[驾驭数据的未来：WhaleStudio与DataOps的完美结合](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485494&idx=1&sn=2ea61e0878601f6518ac542b7af4e01b&chksm=c171c064f6064972f48ac5dd6bd24e6b1ec040d913a8503e17579f6c06426a724b5d3ce355f3&scene=21#wechat_redirect)

[WhaleStudio：创新性解决大数据挑战的工具](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485713&idx=1&sn=f970b361de1b5dece21d323787077271&chksm=c171c143f6064855ec5de00d6b54b4f5e6bb46e61a80100a59e0e31bb772855ddb7e03946c84&scene=21#wechat_redirect)

[支持全生态调度：构建企业数字化转型的桥梁](http://mp.weixin.qq.com/s?__biz=MzkxNDMwMTI0NQ==&mid=2247485769&idx=1&sn=b9f39414db17bdd5b2efd308f78a55f6&chksm=c171c11bf606480d01406f37501b63fb28c181bcd15208acb6cb5a94bf9a4726a6ba59656269&scene=21#wechat_redirect)

**运营开源项目**

目前，北京白鲸开源科技有限公司运营着已经从 Apache 基金会毕业的大数据工作流调度平台 Apache DolphinScheduler，以及数据集成平台 Apache SeaTunnel，诚邀全球伙伴加入开源共建！

**Apache DolphinScheduler：**

仓库地址：https://github.com/apache/dolphinscheduler

官网：https://dolphinscheduler.apache.org/

**Apache SeaTunnel：**

仓库：https://github.com/apache/seatunnel

官网：https://seatunnel.apache.org/

点个在看你最好看