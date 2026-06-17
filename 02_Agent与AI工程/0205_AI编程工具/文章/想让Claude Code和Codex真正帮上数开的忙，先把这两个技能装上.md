---
title: 想让Claude Code和Codex真正帮上数开的忙，先把这两个技能装上
author: 语兴数据
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwODYwNjExMA==&mid=2247488694&idx=1&sn=3699076c6398846d22e76b0103fa7c7d&chksm=c1a8c00c686baa7a1b4270c21c995d72e22d13434bbd89748f52079b602509ea2be464c0349b&mpshare=1&scene=24&srcid=0328O7bxktNDFaqPy0bjVRUg&sharer_shareinfo=a134da513195e0fa4d485f0cdf016b7d&sharer_shareinfo_first=a134da513195e0fa4d485f0cdf016b7d#rd
---

大家好，我是老颜，见字如面。

我最近发现，很多人用 Claude Code / Codex 用不顺，不是不会问，而是问完之后还是要自己从头干一遍。

你想要的其实不是“答案”，而是把你常用的动作交出去：先把问题定位清楚，再给出一套能落地的改法。

这篇我只写两个技能，适合数开：SQL 和 Spark。

## SQL

慢 SQL 的麻烦不在于“慢”，而在于它会反复出现：这个月是这条，下个月是那条。你如果每次都靠临场发挥，就等于把时间交给了随机性。

我这边已经验证安装通过的 SQL skills 有两个。

第一个叫 **sql-optimization**（github/awesome-copilot@sql-optimization），页面： https://skills.sh/github/awesome-copilot/sql-optimization

第二个叫 **sql-optimization-patterns**（wshobson/agents@sql-optimization-patterns），页面： https://skills.sh/wshobson/agents/sql-optimization-patterns

它们解决的不是“帮你写一条更漂亮的 SQL”，而是把优化的常用路径写成默认动作。

你先看过滤条件是不是把字段包进了函数（比如 YEAR(create\_time) 这种），再看 join 条件是不是把过滤放错了位置，再看索引能不能贴合查询，再看分页是不是 offset 把自己拖死。

你把这一套套路固化下来，最大的收益是：同事甩给你一条 SQL，你不用先靠经验猜；你可以先让工具把最典型的坑列出来，你再决定要不要动结构、动索引，还是先动查询。

## Spark

Spark 优化最容易走偏：刚慢一点就开始加 executor、调内存、改并发。最后跑完一轮，自己也说不清到底为什么快了（或者为什么没快）。

我这边验证安装通过的 Spark skill 叫 **spark-optimization**（wshobson/agents@spark-optimization），页面在这里：
https://skills.sh/wshobson/agents/spark-optimization

它更像一张排查路线图：先把 stage 里到底有没有 shuffle 搞清楚，再看是不是数据倾斜把少数 task 拖成了尾巴，再看 partition 是不是不合理导致调度开销或 spill，再看 join 是否应该 broadcast。

对数据开发来说，这类技能的意义在于：它让你少做无用功。你先用结构化的方式定位瓶颈，再谈怎么改。

## 最小闭环

安装就三条命令（一次性）：

```
npx skills add github/awesome-copilot@sql-optimization -g -y
npx skills add wshobson/agents@sql-optimization-patterns -g -y
npx skills add wshobson/agents@spark-optimization -g -y
```

真正使用时，别下“帮我优化”这种泛命令。你可以把任务写成三步，让它按流程走：

第一步，把慢点定位出来（SQL 看执行计划 / Spark 看 stage、shuffle、skew）。

第二步，给 2-3 个改法，并说清楚代价（改索引、改写 SQL、改分区、加盐、广播 join）。

第三步，给验证方法（改完怎么对比、看哪些指标）。

你会发现，一旦你把话说具体，Claude Code / Codex 的价值就立刻就能展现出来：它不是替你背锅，但能替你把“排查套路”跑一遍。

## 写在最后

skills 装得再多，如果不进入工作流，也只是收藏。

我更建议你先把 SQL 和 Spark 这两类装起来，用一周时间把“定位→改法→验证”的闭环跑顺。

等你把这套节奏跑稳，再去补调度、dbt、质量、血缘，完善你的数据开发智能体。

更多精彩内容，尽在y data知识星球。