> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 凌晨告警出问题，Skill 3 秒找到出错的 CASE WHEN
author: 小友Data+AI
date: 小友Data+AI小友Data+AI
url: https://mp.weixin.qq.com/s?__biz=MzYyMzMzNzk0MQ==&mid=2247484031&idx=1&sn=5c54641ad728a4b05e5ca9f963295515&chksm=fef0c03deb69bb5f67996638a3e3f41694f6c5bc3e4eb1fc17ba4c9804097fc862f6c856778f&mpshare=1&scene=24&srcid=0514QzqE6bwuIqE8CGeUi0Xe&sharer_shareinfo=7d52dedb4b6c2b6c978c6a0bd2c72655&sharer_shareinfo_first=7d52dedb4b6c2b6c978c6a0bd2c72655#rd
---

skill 系列：

[数开的未来在哪？我用一套 Skills 干掉了数开 70% 的重复开发](https://mp.weixin.qq.com/s?__biz=MzYyMzMzNzk0MQ==&mid=2247483893&idx=1&sn=429ca8c39e4d836d95838d2d76031fa9&scene=21#wechat_redirect)

[数仓部署还在点点点？完整 Skill 实现，看完直接抄](https://mp.weixin.qq.com/s?__biz=MzYyMzMzNzk0MQ==&mid=2247483905&idx=1&sn=abd0572a9aad20562dc56095bb936114&scene=21#wechat_redirect)

[数仓半夜报警，再也不用爬起来看了 —— 让 Skill 自动化处理](https://mp.weixin.qq.com/s?__biz=MzYyMzMzNzk0MQ==&mid=2247483951&idx=1&sn=4d750cdf1688b7848b455d5ebfee8333&scene=21#wechat_redirect)

今天是Skill 系列第四篇，来拆解哨兵命中异常时调用的那把杀手锏：字段级溯源。

# 一个真实的对账故事

凌晨 3 点，对账系统跑出 BX10001 的 settlement\_date 是 2026-04-25，财务给的基线是 2026-04-20。

按以前的路径，我得爬起来：DataWorks 拉 SQL → 对照 DWD / DWS / ADS 三层 → 翻 CASE WHEN → 把 BX10001 的 biz\_type、order\_date 挨个查出来代入分支 → 找到值"还没变形"前的那一站。

3-6 小时是常态。

现在的实际操作：

```
dw trace-back --bill-no BX10001 \  --field settlement_date \  --baseline 2026-04-20 \  --start-table dws_settle_summary
```

3 秒后：

```
首发层: DWD / dwd_settle_detail分支命中:  [ ] biz_type = 'SALE'  → order_date  [✓] biz_type = 'LEASE' → DATE(order_date, '+5 days')  [ ] ELSE               → order_date建议: 核对 LEASE 分支的 +5 days 规则；     上游 biz_type 是否近期发生编码变更
```

哪一层、哪段 SQL、哪条分支——全给到具体位置。

这事为什么以前没人这么做？

不是没人想过，是工程难度比想象大。三个障碍：

字段级血缘。普通血缘是表与表的关系，但 CASE WHEN 嵌套时同一字段的来源依赖于"走的哪条分支"。需要把 SQL AST 拆到字段 + 条件级别。

分支条件评估。SQL 里写 biz\_type = 'LEASE'，要判断 BX10001 走了哪条，必须把这条单号的实际 biz\_type 反查出来代进去算。

跨层组合爆炸。三层 ETL、每层 20 个 CASE WHEN，全展开是几千种路径——只有把单号的真实值带进来"挑出走过的那条"才能裁剪。

LLM 解决不了这事——它读得懂 SQL，但不擅长一致性。同一条 SQL 喂两次能给出两个不同的分支判定。凌晨 3 点你不敢用它的回答。

# trace-back 三件事

① 用 sqlglot 抽字段级血缘。给定目标字段（如 settlement\_date），AST 反向遍历，找出该字段在每一层 SQL 里的来源列、关联条件、参与运算。结果是一张"字段层级图"，确定每一站要查什么。

② 把单号实际值带入 CASE WHEN 评估。每一层抽出 CASE 分支条件后,反查该单号在上游表的字段值，代入条件计算"哪条命中"。精确到字段，不靠模糊匹配。

③ 自顶向下走链路找首发层。从 ADS 一路往 ODS 追，第一个出现"值开始偏离基线"的层就是首发层——也是要看 SQL 的那一层。

整个过程确定性：相同输入永远给出相同输出。LLM 只在最后出场，把结构化报告译成人话。

**关键工程决策：****SQL****解析、分支评估交给 sqlglot——精确、可重现、便宜。****LLM****只负责把结构化结果讲成人话。**

这是 Skill 工程里**最重要的一条分工原则**：LLM 不擅长强一致性，擅长解释和总结。把它放在它擅长的位置。

# 闭环到哨兵

第三篇说哨兵命中异常时自动调 trace-back 给结论——就是这把工具。

哨兵 + trace-back 一起，告警从"出事了"变成"出了什么事 + 谁干的 + 怎么办"。前者负责发现，后者负责定位——完整闭环。

---

skill 获取：公众号「小友 Data+AI」后台私信 "skill"。