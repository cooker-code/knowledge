> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: Skills系列: requirements-analyst 深度解析
author: TRAE一下
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0MjI5MDc1NQ==&mid=2247483743&idx=1&sn=0f7d83bc1dae7cbe53b5bafa000e9cdd&chksm=c32d8b1fd5a132a49307be16553a9f195c7051aa2b1bad073956b36d3e4774e2609a0e49d50a&mpshare=1&scene=24&srcid=0423TYHT3BB6fSVilMHb7Utw&sharer_shareinfo=ab2e0f49defb26d2a37638537c43f97e&sharer_shareinfo_first=ab2e0f49defb26d2a37638537c43f97e#rd
---

Skills 系列第 2 篇

# requirements-analyst 深度解析

装上它，SOLO 就多了一个产品经理

在上一篇[SOLO 自带的 6 个神Skills 装上就能起飞](https://mp.weixin.qq.com/s?__biz=Mzk0MjI5MDc1NQ==&mid=2247483735&idx=1&sn=15edec8f0723b2f243afa33064be6d0f&scene=21#wechat_redirect)，我们把 SOLO 自带的 6 个核心 Skill 介绍了一遍。从这篇开始，我们**逐个深入**，今天是系列第 2 篇——排在**第一位**的 requirements-analyst。

## 为什么它排第一？

社区 71 赞精华帖把它列为**Top 6 Skill 之首**，不是没有原因的。

想想你平时怎么跟 AI 说话的：

"我想做一个记账 App"

"帮我写个博客"

"给网站加个好玩的功能"

这些需求听起来很正常，但问题是——**AI 根本不知道你具体想要什么**。没有这个 Skill，AI 会直接猜一个方案然后开始写代码，结果大概率不是你想要的。

装上 requirements-analyst 后，SOLO 会**先停下来问你**，像产品经理一样帮你把需求想清楚，再动手。

## 四大核心能力

### 1. 需求采访

面对模糊需求时，AI 会进入"采访模式"，主动追问 3-5 个核心问题。比如你说"给博客加个好玩的功能"，它会问：

* 这个功能的具体目标是什么？
* 目标用户是谁？
* 期望的展示形式是什么？
* 有没有参考的竞品或案例？

### 2. 意图抽离

从你的非技术描述中识别出真实的业务目标。你说"加个好玩的功能"，它不会直接加个弹窗游戏，而是帮你搞清楚你真正想要的是**提升用户停留时间**还是**增加互动性**。

### 3. 规划对齐

自动检查你的新需求是否与项目路线图（roadmap.md）和待办事项（todo.md）冲突，确保开发方向不跑偏。

### 4. 影响评估

评估需求变更对用户体验和业务逻辑的影响，帮你提前规避"改了一个地方，崩了三个功能"的尴尬。

## 装了 vs 没装，区别有多大？

没装 Skill

你说"做个记账App"，AI 直接开始写代码，做完发现：

* 功能不是你想要的
* 没有分类功能
* 数据结构设计不合理
* 返工重做...

装了 Skill

AI 先问你 5 个问题，搞清楚后：

* 需求拆解成 8 个子任务
* 每个任务有验收标准
* 数据结构提前设计好
* 一次做对，不用返工

核心区别：先想清楚再动手 vs 猜着做再返工

## SKILL.md 里到底写了什么？

这个 Skill 的结构非常精简，只有一个 SKILL.md 文件，但指令设计很巧妙：

```
# Requirement Analyst Skill

## 四条核心指令：

1. 强制阅读 roadmap.md 和 todo.md
   → 先了解项目现状再分析需求

2. 启动采访 模糊需求必须追问
   → "为了确保精准实现，我需要就以下几点与您对齐"

3. 遵循规范 使用 Momei 评分矩阵
   → 新阶段任务按标准规划

4. 输出产物 生成 todo.md Task 项
   → 标记优先级 P0/P1/P2
```

注意第 2 条指令的设计——它不是"建议"AI 去追问，而是**强制**的。只要检测到模糊需求，AI 必须进入采访模式。这就是 Skill 的威力：**用规则约束 AI 的行为**。

## 5 个实用技巧

1

项目初期就激活

不要等项目写了一半才想起来。在 SOLO 对话的**最开始**就让它帮你分析需求。

2

配合项目文档结构

建立 `docs/plan/` 目录，放好 roadmap.md 和 todo.md，Skill 的规划对齐功能才能生效。

3

认真回答采访问题

AI 追问时别嫌烦，这个"苏格拉底式提问"的过程正是价值所在——把脑子里模糊的想法变成结构化需求。

4

按顺序组合其他 Skill

需求分析完 → frontend-design 设计 → fullstack-developer 开发 → webapp-testing 测试。这套流程社区验证最有效。

5

大胆自定义

SKILL.md 是纯文本，完全可以改。调整采访问题模板、添加行业特定评估维度，让它更懂你的项目。

## 三种安装方式

方式一：对话安装（推荐）

直接跟 SOLO 说：`"帮我安装 requirements-analyst"`

方式二：斜杠命令

输入 `/` 选择 Skill，一键安装

方式三：手动安装

将 SKILL.md 放到 `.trae/skills/requirement-analyst/` 目录下

## 需要注意的坑

* **applyTo 限制**：该 Skill 设置为 `docs/plan/*.md`，主要在处理规划文档时自动激活。如果你的项目没有这个目录，可能需要手动触发。
* **Momei 评分矩阵**：Skill 引用了 `docs/standards/planning.md` 中的评分工具，项目中没有这个文件的话该功能无法使用。
* **版本较旧**：当前版本 1.0.0，功能相对基础。对于更复杂的需求分析场景，可以考虑社区增强版如 `business-analyst`（173行，功能更丰富）。

## 一句话总结

requirements-analyst 的核心价值就是：**让 AI 先想清楚再动手**。

它不帮你写一行代码，但它能让你少写 100 行废代码。

## 系列预告

第1篇 [SOLO 自带的 6 个神Skills 装上就能起飞](https://mp.weixin.qq.com/s?__biz=Mzk0MjI5MDc1NQ==&mid=2247483735&idx=1&sn=15edec8f0723b2f243afa33064be6d0f&scene=21#wechat_redirect)第2篇 requirements-analyst第3篇 前端设计双雄
第4篇 全栈开发
第5篇 自动化测试第6篇 skill-creator

下一篇：前端设计双雄 —— frontend-design + ui-ux-pro-max 联合解析

TRAE一下 | 让 AI 编程更简单

如果觉得有用，欢迎点赞、在看、转发