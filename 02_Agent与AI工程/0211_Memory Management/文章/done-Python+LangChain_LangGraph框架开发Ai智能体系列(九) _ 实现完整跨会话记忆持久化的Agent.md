> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020102_LangGraph/020102_核心知识点/LangGraph记忆与反馈循环|LangGraph记忆与反馈循环]]
---
title: Python+LangChain/LangGraph框架开发Ai智能体系列(九) | 实现完整跨会话记忆持久化的Agent
author: 极简工具盒
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0MzQyNDA0Mg==&mid=2247484251&idx=1&sn=9b099569cea5dd1591efe3147f72c684&chksm=c2dd440e104f5f29139a0613d5db94ab6a83d21da2e2b8e7f2e738ff7f5f03143b029f855bb6&mpshare=1&scene=24&srcid=0410wrfCw21bl8lPYLD2mbvK&sharer_shareinfo=aaf6a2a5ec439871e57ee81b57fd1b7f&sharer_shareinfo_first=aaf6a2a5ec439871e57ee81b57fd1b7f#rd
---

1. 学习目标

- 为 Agent 添加短期记忆 + 长期记忆，熟悉了解实现跨会话的记忆持久化的流程。

2.核心概念速览

-架构图

### 核心特点

-关键知识点总结

短期记忆：

  Buffer    → 全量保留，简单粗暴

  Window    → 滑动窗口，省 Token

  Summary   → 摘要压缩，更省 Token

长期记忆：

  关键词检索  → 精确但不够智能

  向量检索   → 语义匹配，效果好

  混合检索   → 两者结合（Day9 详细学）

⚡ 核心原则：

  短期记忆 = 给 LLM 看的上下文

  长期记忆 = 存在外部数据库里的持久知识

3.代码实现（三大核心组件）

3.1 🤖组件 1：ShortTermMemory（滑动窗口）

3.2 🤖组件 2：LongTermMemory（SQLite 持久化）

3.2.1  数据库初始化

3.2.2  Jaccard 相似度检索

3.2.3  检索函数

3.2 🤖 组件 3：MemoryAgent（融合编排）

3.3 🤖 五类信息提取规则

3.代码运行

完整知识地图：

执行结果：

---

**热点文章推荐：**

[小白10分钟搭建腾讯版龙虾WorkBuddy环境并绑定微信控制电脑干活生成一个agent应用](https://mp.weixin.qq.com/s?__biz=Mzk0MzQyNDA0Mg==&mid=2247484095&idx=1&sn=cc4f7bda9abc6313d43d3659e64a98dd&scene=21#wechat_redirect)

[分享一个Coze扣子智能体自动生成ai视频上传抖音等视频平台的方案](https://mp.weixin.qq.com/s?__biz=Mzk0MzQyNDA0Mg==&mid=2247484062&idx=1&sn=e0fa84626f05fe7655b17b43551f71e6&scene=21#wechat_redirect)

后续分享更系列教程关注“极简工具盒”公众号探索更多精彩内容，谢谢！