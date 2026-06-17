---
title: 不训练，解锁Agentic智能！微软：只需给LLM一台电脑
author: PaperAgent
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247503231&idx=1&sn=3bcd603db1b44139a56b47a52b4342b5&chksm=c330c682ef55f1627f28ac4839d829fb95645f44a3a2ecabf6a4c8c4b91645daeff0265f0bee&mpshare=1&scene=24&srcid=0205WG06rK1AyvwnOG0IJx5S&sharer_shareinfo=0adeb7505fb9ef953d5e3d474d10b01c&sharer_shareinfo_first=0adeb7505fb9ef953d5e3d474d10b01c#rd
---

大家好，我是PaperAgent，不是Agent！

当大模型拥有“电脑”后会发生什么？ 人大、MSR、清华的**LLM-in-Sandbox**开启「通用代理智能」新范式

把大模型放进一台“虚拟机”里，让它像人一样装软件、写代码、查资料、管文件——结果模型在数学、物理、化学、医学、长文本、指令遵循 6 大任务上**全线暴涨**，最高 +24.2%，而且**零额外训练**即可解锁“通用代理智能”。

Figure 1 直观展示了同一台“虚拟机”如何让 7 个主流模型**正收益**：

## 范式演化

| 范式演进 | 代表工作 | 能力天花板 |
| --- | --- | --- |
| 纯文本生成 | GPT-3 | 受限于上下文长度与幻觉 |
| 链式思维 | CoT | 推理步骤长但无法验证 |
| 工具调用 | ChatGPT Plugins | 工具固定、无法动态扩展 |
| **LLM-in-Sandbox** | 本文 | **任意工具、可验证、可扩展、跨模态** |

## 核心思想：把“电脑”当作 LLM 的新上下文

把代码沙箱抽象成 3 个元能力：

1. **外联**：curl / pip / apt 装任何包
2. **存储**：文件系统 = 无限长“外部记忆”
3. **执行**：Python/bash 即时验证结果

Table 1 对比了传统软件工程 Agent 与 LLM-in-Sandbox 的设计差异：

## 训练-free 实验：强模型自发“玩电脑”

### 4.1 实验设置

* 评测 7 个模型：Claude-4.5、GPT-5、DeepSeek-V3.2、MiniMax-M2、Kimi-K2、Qwen-Coder-30B、Qwen-4B
* 6 大领域：数学(AIME)、物理(UGPhysics)、化学(ChemBench)、医学(MedXpert)、长文本(AA-LCR)、指令遵循(IFBench)

### 4.2 结果一览

Table 2 给出**绝对准确率**与\*\*Δ(沙箱-纯文本)\*\*：

> 结论：**越强模型，收益越大**；小模型（Qwen-4B）因“不会用电脑”反而掉分。

## 强化学习版：让弱模型也学会“用电脑”

### 5.1 LLM-in-Sandbox-RL

只用**通用上下文任务**（百科、小说、论坛帖子等）做奖励训练，**零代码领域数据**。  
核心技巧：把上下文放进 `/testbed/documents/` 目录，模型必须**主动搜索**才能答题。

Figure 3 示意了多文档+干扰文件的沙箱布局：

### 5.2 训练后效果

Table 6 显示 Qwen-4B **全线反超**：

## 行为解析：强模型到底怎么“玩电脑”？

Figure 2 统计了不同任务下三类行为占比：

* **数学**：43.4% 回合在做数值计算
* **化学**：18.4% 回合安装/调用外部库（RDKit、OPSIN）
* **长文本**：高频文件 IO，但零网络请求

Table 4 进一步对比“强-弱”模型：

## 效率与部署：沙箱并不贵

### 7.1  token 成本

Table 10 显示长文本任务**最高省 90 % token**（100K→13K）；综合所有任务仍打 5-8 折。

### 7.2 吞吐与资源

Table 11 & Table 12：

* 环境 token 用 fast prefill，占比 50 % 但耗时 <4 %
* 单容器 idle 仅 50 MB，512 并发只占 5 % 内存
* **存储从 TB 级降到 1.1 GB**（通用镜像）

## 超越文本：4 个案例看见“数字工匠”

Figure 5 演示了**纯文本进，多媒体出**：

1. 旅行规划 → 可交互 Tokyo 地图 HTML
2. 会议 JSON → 专业海报 PNG/SVG
3. 生日主题 → 11 s 倒计时视频 MP4
4. 风格描述 → 原创钢琴曲 MIDI+WAV+谱面

```
https://arxiv.org/pdf/2601.16206  
LLM-in-Sandbox Elicits General Agentic Intelligence  
https://github.com/llm-in-sandbox/llm-in-sandbox
```

推荐阅读

[动手设计AI Agents：（编排、记忆、插件、workflow、协作）](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247492838&idx=2&sn=1e25832e7300ef312721325d0def30b4&scene=21#wechat_redirect)

[分享两篇Claude Skills最新论文，有3个核心结论](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247502780&idx=1&sn=2671e0e0e6e15dd5a2020b1fc1281cf7&scene=21#wechat_redirect)

[2026，新风向： 世界模型  × 具身智能 最新综述](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247502059&idx=1&sn=7e3a7a4a0ccf390b1904165aff3728d8&scene=21#wechat_redirect)    
[2026，做Agentic AI，绕不开这两篇开年综述](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247502666&idx=1&sn=d6a467896c6753c8d8634c7400d8dbb4&scene=21#wechat_redirect)

---

每天一篇大模型Paper来锻炼我们的思维~已经读到这了，不妨点个👍、❤️、↗️三连，加个星标⭐，不迷路哦~