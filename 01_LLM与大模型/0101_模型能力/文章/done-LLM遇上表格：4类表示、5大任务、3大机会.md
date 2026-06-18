---
title: LLM遇上表格：4类表示、5大任务、3大机会
author: PaperAgent
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247496688&idx=1&sn=ba140d500bfdb93a00bafd7cdadb4a5b&chksm=c3a3935de576acf1edef8ad9e36873e99b08b3bd406d71da4cf731f0bd5f3b48457afa82232c&mpshare=1&scene=24&srcid=091264YKZAedVcc0Jk7SHNbx&sharer_shareinfo=537612e2c69804d76459533d53fc4903&sharer_shareinfo_first=537612e2c69804d76459533d53fc4903#rd
---
> 已吸收至：[[01_LLM与大模型/0101_模型能力/0101_核心知识点/LLM表格理解能力边界|LLM 表格理解能力边界]]

## 1. 表格让大模型头疼？

文本是线性的，而表格是二维、结构多变、目的多样的——从严谨的数据库到多层嵌套的 Excel，再到 Wikipedia 的 Infobox。
把 LLM 处理表格的“痛苦”总结为三点：

| 痛点 | 概况 |
| --- | --- |
| 任务单一 | 90% 的 Benchmark 都在考「检索+简单数学」，真正需要推理的很少 |
| 输入复杂就崩 | 长表、多表、层级表、跨文档表，人类 80+ 分，SOTA 模型 50 分不到 |
| 表示不统一 | 同一张表换个 JSON / HTML / Markdown，性能就能掉 5 个点 |

左侧用Text-To-Sql可解决，相比之下，右侧展示的是需要高级推理或涉及复杂输入的任务。

大模型表格处理任务的工作流

## 2. 先把“表”说清楚：四种输入表示法

把 LLM 能“吃进”的表格表示分成 4 大类（对应 Figure 4）：

| 表示方式 | 优点 | 缺点 | 典型 Benchmark |
| --- | --- | --- | --- |
| **Serialization** 序列化 | 直接用文本，最简单 | 结构信息易丢失 | WTQ, TabFact |
| **Schema** 只给表头+列类型 | 省 token | 细节全丢 | Spider, SEDE |
| **Image** 表格截图 | 保留完整视觉结构 | 受分辨率限制 | VISTABNet |
| **Table Encoder** 专用编码器 | 结构感知最强 | 需要额外预训练 | TableGPT2, TAPAS |

实验发现：同样一道题，把 Markdown 换成 LaTeX，EM 分数最多差 ***20%***
给了三种序列化示例：

## 3. 5大人任务全景：不止Text-to-SQL

整理了 **3 大经典任务 + 2 个新兴方向**，并给出所有 Benchmark 一览（Table 1~4）：

| 任务 | 输入 | 输出 | 热门数据集 |
| --- | --- | --- | --- |
| **Table QA** 表问答 | 表(+文本)+问题 | 答案单元格 / 数字 / 自由文本 | WTQ, HiTab, MULTIHIERTT |
| **Table-to-Text** 表到文本 | 表(+高亮区域) | 一段描述或摘要 | ToTTo, LogicNLG, QTSUMM |
| **Fact Verification** 表事实核查 | 表+声明 | Supported / Refuted / NEI | TabFact, FEVEROUS |
| **Text-to-SQL** 自然语言转 SQL | 问题+数据库 | SQL 查询 | Spider, BIRD, Spider2 |
| **Leaderboard Construction** 排行榜自动构建 | 论文表格 | (任务, 数据集, 指标, 分数) 四元组 | AxCell, TeLin |

## 4. 三大发现：新研究机会？

### 4.1 任务复杂度

* 现有 Benchmark 大多是“把 SQL 翻译成自然语言”再让模型反推；
* 真正的诊断、预测、洞察类问题（图 3）几乎空白；
* **Spider2** 首次引入意图级问题：用户说“给我一份每日关键销售报告”，模型得自己猜要查哪些字段。

### 4.2 输入复杂度：长表、多表、层级表 = 模型噩梦

* **MULTIHIERTT**：人 83% vs 模型 <50%；
* **HiTab**：层级多维表，模型同样翻车；
* 科学论文中的消融表 + 长文本，是未来绝佳试验田。

### 4.3 表示统一：换个格式就掉点

* 同一任务里，JSON ↔ Markdown ↔ LaTeX 之间没有统一规范；
* 未来可以搞“格式互译”任务，让模型见多识广。

```
https://arxiv.org/pdf/2508.00217Tabular Data Understanding with LLMs: A Survey of Recent Advances and Challenges
```

推荐阅读

* •  [科研痛点无了，试试我用智谱开源的GLM-4.5做的一个Agent助理](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247496384&idx=1&sn=320cc5d148928e69475f9ccac5fe0345&scene=21#wechat_redirect)

+ •  [挑战Transformer，谷歌全新架构Mixture-of-Recursions推理速度飙升2倍](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247495548&idx=1&sn=87dd3f518c89602c9259c34fcc08e447&scene=21#wechat_redirect)
+ •  [动手设计AI Agents：（编排、记忆、插件、workflow、协作）](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247492838&idx=2&sn=1e25832e7300ef312721325d0def30b4&scene=21#wechat_redirect)

  • [一篇200+文献的视觉强化学习（RL）技术最新综述](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247496438&idx=1&sn=3f1653d9e9a6df3fd81a8d7d51ff2c3d&scene=21#wechat_redirect)

---

欢迎关注我的公众号“**PaperAgent**”，每天一篇大模型（LLM）文章来锻炼我们的思维，简单的例子，不简单的方法，提升自己。
