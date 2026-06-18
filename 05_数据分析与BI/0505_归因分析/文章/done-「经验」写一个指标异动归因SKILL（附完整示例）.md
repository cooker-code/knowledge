> 已吸收至：[[05_数据分析与BI/0505_归因分析/0505_核心知识点/指标异动贡献拆解与业务归因|指标异动贡献拆解与业务归因]]
---
title: 「经验」写一个指标异动归因SKILL（附完整示例）
author: 小火龙说数据
date: 小火龙说数据小火龙说数据
url: https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247519119&idx=1&sn=585c88976e57a0924299a4f43c4bdb7d&chksm=c07703b883fafabb125cc7eb1e45045dff1b1fac6184ca45ccff438d5b3ad4ac4f422e176eb2&mpshare=1&scene=24&srcid=0602lizhWhNU4d1IDeRAInBO&sharer_shareinfo=be37bfbb52e1fee68bad378c503c4052&sharer_shareinfo_first=be37bfbb52e1fee68bad378c503c4052#rd
---

00 前言

异动归因是每一个数据分析、数据科学同学必备技能，过往也分享过很多异动归因的方法和技巧，需要的同学可以自行戳链接。

[「经验」如何30min内排查出指标异动的原因](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247484504&idx=1&sn=ef89775220793ca2f0969b8aa3b06c4e&scene=21#wechat_redirect)

[「经验」指标异动排查中，3种快速定位异常维度的方法](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247485346&idx=1&sn=e0e2bd33fbd6827d0a9fc3305a8e824b&scene=21#wechat_redirect)

[「经验」指标异动排查中，如何量化对大盘的贡献程度](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247486884&idx=1&sn=0a581f94e501ed41ce9fcab9fcb083fb&scene=21#wechat_redirect)

[「经验」汇总指标异动的十大原因，涵盖日常90%问题](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247499823&idx=1&sn=eea95308f04e0943ad3780cec4984b09&scene=21#wechat_redirect)

但随着AI的发展，我们可以让其自助产出异动分析报告，这里分享给大家一个让大模型（小火龙用的claude）自行写归因OpenClaw SKILL的示例，拿来即用。

01 input

要让大模型给咱写一个SKILL，就需要告诉它如何做，输入内容如下。

1、归因方法：通过哪些维度归因

* 按用户新增/回流/留存拆解
* 按渠道/来源拆解
* 按功能模块拆解

2、数据来源：数据从哪里取

* 直接查Hive表？对应的表名
* 调用BI平台API
* 还是已有现成SQL模板？

3、输出形式：期望产出什么

* 生成一段 SQL 给用户自己跑
* 直接返回数据结果
* 生成分析报告文字

4、触发语：用户一般怎么问这类问题

* 触达条件是什么

02 output

1、产出文件树

```
dau-attribution/├── SKILL.md                            # 核心入口，触发条件 + 工作流程└── references/  ├── sql-templates.md                  # 4 个 SQL 模板  └── interpretation-guide.md           # 解读指南 + 结论模板
```

2、文件内容摘要

SKILL.md（触发+导航）

触发词：DAU为什么涨跌、DAU归因分析、DAU拆解等。

归因定义：新增/回流/留存的判断逻辑。

工作流程：4 步走（理解需求→生成SQL→解读结果→输出结论）。

references/sql-templates.md（4个SQL模板）

模板一：单日整体归因。

模板二：近N天趋势。

模板三：按维度拆解（垂类/渠道/地区等）。

模板四：今日 vs 昨日环比对比。

references/interpretation-guide.md（解读指南）

常见波动模式及原因分析。

不同产品阶段的健康结构参考。

结论输出的标准模板。

3、文件详细内容

SKILL.md

```
---name: dau-attributiondescription: "DAU 归因分析：将日活用户拆解为新增用户、回流用户、留存用户三类，定位 DAU 波动的驱动因子。当用户询问 DAU 为什么涨跌、DAU 归因分析、今天的 DAU 是怎么构成的、新增回流留存用户分别是多少、帮我做一下 DAU 拆解等问题时使用。"---
# DAU 归因 Skill
## 归因逻辑
DAU = 新增用户 + 回流用户 + 留存用户
| 用户类型 | 定义 ||---------|------|| 新增用户 | 当天首次活跃，历史上从未活跃过 || 回流用户 | 当天活跃，但昨天未活跃，且历史上活跃过 || 留存用户 | 当天活跃，且昨天也活跃 |
注：三者互斥且完整，之和等于当日 DAU。
## 工作流程
1. 理解需求：确认分析日期、分析维度（整体 or 按某维度拆分）2. 生成 SQL：读取 references/sql-templates.md 中的模板，按需填充3. 解读结果：对数据做业务解读，参考 references/interpretation-guide.md4. 输出结论：结论先行，给出波动原因判断和建议
## 关键约束
- 表名、字段名必须来自用户提供或工具返回的实际结构，禁止凭空构造- 涉及分区表时，先确认可用分区再生成 SQL- 如果用户未提供表名，先用 table-discovery Skill 搜索合适的底表- 默认分析昨天的数据（T-1），除非用户指定日期
```

interpretation-guide.md

```
# DAU 归因结果解读指南
## 一、基本解读框架
拿到归因数据后，按以下顺序解读：
1. **总量判断**：DAU 是涨是跌，幅度多大（环比、同比）2. **分量贡献**：三类用户各自的变化方向和绝对量3. **主因定位**：哪个分量贡献了最大的变化量4. **结构判断**：三类用户的比例是否健康
---
## 二、各分量的业务含义
| 分量 | 上升意味着 | 下降意味着 ||------|-----------|-----------|| **新增用户** | 拉新效果好 / 有新渠道带量 / 产品曝光增加 | 拉新疲软 / 渠道衰减 / 活动结束 || **回流用户** | 唤回活动有效 / 用户有新需求 / 节假日效应 | 流失用户难以唤回 / 竞品分流 || **留存用户** | 用户习惯好 / 核心功能粘性强 | 核心用户流失 / 产品体验问题 / 竞品冲击 |
---
## 三、常见波动模式与解释
### 模式 1：DAU 上涨，主要靠新增- **信号**：新增占比明显高于正常水位- **可能原因**：营销活动投放、热点事件带流量、新功能上线吸引尝鲜- **风险**：如果留存没有同步提升，新增带来的 DAU 会很快回落- **建议**：关注这批新增用户的次日、7日留存
### 模式 2：DAU 上涨，主要靠回流- **信号**：回流用户数显著增加，新增和留存变化不大- **可能原因**：唤回推送/短信效果好、节假日（春节、长假前后）、竞品出现问题- **风险**：回流用户留存率通常低于留存用户，需观察是否二次流失- **建议**：分析回流用户的行为路径，判断是否真正找到留住他们的点
### 模式 3：DAU 下跌，留存用户下降为主- **信号**：留存用户数下降，而新增/回流变化不大- **可能原因**：产品 bug / 体验劣化、核心功能被竞品替代、内容质量下降- **严重程度**：⚠️ 最需要警惕，留存下降说明原有用户在流失，是最危险的信号- **建议**：立即排查近期上线的版本或策略变更
### 模式 4：DAU 下跌，新增用户下降为主- **信号**：新增用户明显减少- **可能原因**：活动结束、投放收缩、渠道效率下降- **严重程度**：相对可控，留存盘子还在- **建议**：排查拉新渠道，评估是否需要新的增长策略
### 模式 5：三类分量同步下降- **信号**：新增、回流、留存全面下滑- **可能原因**：外部环境因素（竞品大促、行业整体疲软）、平台整体问题- **建议**：结合外部数据（竞品动态、行业指数）综合判断
---
## 四、健康结构参考
不同产品阶段的健康比例不同，以下是通用参考：
| 产品阶段 | 新增占比 | 回流占比 | 留存占比 ||---------|---------|---------|---------|| 早期成长期 | 30~50% | 10~20% | 30~50% || 成熟稳定期 | 10~20% | 10~20% | 60~80% || 衰退期 | <10% | 20~30% | 60~70% |
> 成熟产品的 DAU 主要由留存用户构成，这是健康的标志。如果成熟产品过度依赖新增/回流，说明留存体系有问题。
---
## 五、输出结论模板
```【DAU 归因结论】  
📊 今日 DAU：XX 万（环比 ±XX%，同比 ±XX%）  
构成拆解：  - 新增用户：XX 万（占比 XX%，环比 ±XX%）  - 回流用户：XX 万（占比 XX%，环比 ±XX%）  - 留存用户：XX 万（占比 XX%，环比 ±XX%）  
🔍 主因：DAU [上涨/下跌] 主要由【留存/新增/回流】用户驱动，  [留存/新增/回流]用户环比变化了 XX 万（贡献了 XX% 的总变化量）。  
💡 初步判断：[结合业务背景写 1\~2 句原因判断]  
⚠️ 建议关注：[需要进一步排查的方向]  ```
```

03 SKILL注意事项

最后，说说我踩过的坑，写SKILL时的注意事项。

注意1：SKILL涵盖内容。主要涵盖head和body，head包含「name、description」，body包含「执行流程概览、执行流程细节、注意事项」，这五个模块缺一不可，当然，很多SKILL还会增加一些示例。

注意2：写给Agent看，不是写给人看。Agent不会像人一样通读全文后再理解，它更像是在做模式匹配。所以，description要包含关键触发词，也就是用户可能说的话；参数说明简洁，不要有歧义；如果有示例，要可直接复制执行，不需要额外改动。

注意3：简洁简洁再简洁。SKILL.md一般小于500行，不要过长，浪费token；过多的内容可以放在references里面；同时，采取渐进式披露，说白了，只在需要时读取references，不要一股脑儿把信息都扔进来。

**1、实战进阶书籍《数据分析实践：专业知识和职场技巧》，侧重案例实操。**

目标群体：需要系统学习数据分析全流程，通过更多案例实现落地的同学。

详细介绍：**[《数据分析实践：专业知识和职场技巧》](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247515268&idx=1&sn=78dbb59d94f0b6ee328831ce6b772dd5&scene=21#wechat_redirect)**

**2、将13年工作经验沉淀成「数据分析方法论图谱」，侧重场景与方法。**

目标群体：需要快速进阶，近期在准备面试的同学。

详细介绍：**[数据分析方法论图谱v3.0更新版](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247518979&idx=1&sn=322b66b1b31e78fe97170b873e034365&scene=21#wechat_redirect)**

**3、「简历修改、面试辅导、职业咨询」，助同学们成功上岸。**

目标群体：准备找工作、正在找工作的同学。

详细介绍：**[简历修改及面试辅导](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247515652&idx=1&sn=79d0da1e1eb34d3998092410804f6ea7&scene=21&token=1992161495&lang=zh_CN#wechat_redirect)**

往期推荐

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

# [「经验」点击率暴涨19%！数据人必看的AB实验爆款秘籍](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247518229&idx=1&sn=bf33e76804ec48db34924c3296264dc3&scene=21#wechat_redirect)

# [「经验」学会『边际ROI分析法』，从此告别广告预算浪费！](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247518269&idx=1&sn=659707fcf5b5c131705afefc85e71ee3&scene=21#wechat_redirect)

# [「经验」AB实验效率翻倍！方差缩减神器CUPED](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247518347&idx=1&sn=da08e2d1340a765bbfe6c0375469fed4&scene=21#wechat_redirect)

# [「经验」因果推断三剑客 PSM、DID、CUPED](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247518356&idx=1&sn=dcdbee96f62c11b133e60f79e94b4e53&scene=21#wechat_redirect)

# [「经验」头部企业RFM实战全拆解](https://mp.weixin.qq.com/s?__biz=MzkxNzMwNTEwNQ==&mid=2247518368&idx=1&sn=07447dd39d73fbac3329b91fc9736fe0&scene=21#wechat_redirect)

#

持续追更哦