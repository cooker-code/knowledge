---
title: SQL性能优化的新视界 - PawSQL Plan Visualizer
author: PawSQL
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkyODM0NzE1Ng==&mid=2247484500&idx=1&sn=c3e4df52bdae0ab210763f9c4c3b002e&chksm=c21b612ff56ce8390ba17bb66d8fe32a64bb22b4d3bcae338ed0e46a1a4be6da66990fcd9e1e&mpshare=1&scene=24&srcid=0703hIEjWS9dagzF98GBT4O1&sharer_shareinfo=65b2f2d3899685570a9c28b07e1d5d45&sharer_shareinfo_first=65b2f2d3899685570a9c28b07e1d5d45#rd
---

在数据库的世界中，执行计划是SQL查询的蓝图，是数据库性能优化的基石。随着数据库应用的日益复杂化，SQL查询的执行计划也变得愈发错综复杂。传统的文本执行计划阅读起来既耗时又难以把握，给数据库管理员(DBA)带来了不小的挑战。

🌟 PawSQL Plan Visualizer：可视化的力量

PawSQL Plan Visualizer（PPV）基于开源项目pev/pev2开发。它将复杂的执行计划转化为直观的流程图，让DBA能够一目了然地理解SQL的执行路径和关键操作。PPV不仅提高了分析效率，还能快速定位性能瓶颈，实现精准优化。

立即体验: https://pawsql.com/ppv

🛠️ 主要功能亮点

**1. 多数据库支持**：兼容MySQL、PostgreSQL、openGauss、Oracle等多种数据库。

**2. 多格式输入**：支持MySQL的json/tree/analyze格式，PostgreSQL的多种format，Oracle的表单格式等多种输入方式。

**3. 交互式分析**：通过执行时间、代价、行数的高亮展示，实现交互式分析。

**4. Metrics统计**：基于数据库对象和算子的详细统计，为优化提供数据支撑。

🌐 创建可视化执行计划

将执行计划的文本输出提交至PPV，即可获得一个清晰的树形结构展示。算子上方的图标提示了可能的问题，如耗时过长或估算代价过高，帮助DBA快速定位并排查问题。

🔍 交互式分析

* **快速定位**：点击头部概要信息，迅速找到最耗时或代价最大的节点。
* **详细信息展示**：每个算子点击后展开，展示具体耗时、条数等详细信息。
* **进度条展示**：点击时间、代价、行数，以进度条形式直观展示各节点占比。

‍📈 基于对象和算子的统计

页面右侧提供了基于对象（表/索引）和算子的统计分析，包括执行时间和估算代价等关键指标。

🔄 与PawSQL优化功能的集成

PPV还能获取优化前后的执行计划，通过PEV进行可视化对比，帮助DBA验证性能优化的效果。

‍

## PawSQL往期文章精选

[SQL审核 | PawSQL的审核规则集体系](http://mp.weixin.qq.com/s?__biz=MzkyODM0NzE1Ng==&mid=2247484404&idx=1&sn=51a8859a30dd6c5587bf8cc3cf2d731b&chksm=c21b668ff56cef99ffd5ef7e79922dc4766465e589a2a7b1c00a2eb0d06154fa79222a92fcba&scene=21#wechat_redirect)

[SQL质量的终极解决方案，PawSQL审核平台重磅上线!](http://mp.weixin.qq.com/s?__biz=MzkyODM0NzE1Ng==&mid=2247484485&idx=1&sn=959588a0ced47b9701e575dc9fda56b2&chksm=c21b613ef56ce8287cb479cd4462510bf798651b8611b4592bbb98ee73638cbeceb1f4378176&scene=21#wechat_redirect)

[SQLE、SQM和PawSQL：企业级SQL审核平台的深度评测](http://mp.weixin.qq.com/s?__biz=MzkyODM0NzE1Ng==&mid=2247484469&idx=1&sn=955893606aabc3ee7e802b98ef96dd5a&chksm=c21b614ef56ce85821357898fc05db19d448d0a73e073363fd317d4b19da6200c870471e44cf&scene=21#wechat_redirect)

[PawSQL优化 | 分页查询太慢？别忘了投影下推！](http://mp.weixin.qq.com/s?__biz=MzkyODM0NzE1Ng==&mid=2247484473&idx=1&sn=39b69a3aa8edb7cfa0b89ed6bcb9cf1f&chksm=c21b6142f56ce854b19b097232fd75272ac25328536a6c55d0f694e93e94029c51fa4ed03934&scene=21#wechat_redirect)

**🌐关于PawSQL**

PawSQL专注数据库性能优化的自动化和智能化，提供的解决方案覆盖SQL开发、测试、运维的整个流程，支持MySQL，PostgreSQL，openGauss，Oracle等各种数据库。

* PawSQL优化平台：https://pawsql.com/app
* PawSQL审核平台：https://pawsql.com/audit
* PawSQL巡检平台：https://pawsql.com/ppt
* PawSQL Plan Visualizer: https://pawsql.com/app/explain

欢迎点击关注PawSQL公众号