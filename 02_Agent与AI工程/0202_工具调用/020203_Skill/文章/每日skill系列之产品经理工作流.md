---
title: 每日skill系列之产品经理工作流
author: AI工程化实战派
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA4MDUzNjMzOQ==&mid=2447658640&idx=1&sn=c9770bb9ad5021734fdee622e900a137&chksm=8a8e839e5708f71475b129ee8348818bf31a5dbee7acc9806e08cc796dadb9d57f6624ee19fc&mpshare=1&scene=24&srcid=0409COpeuKb6leHFcSaHeDct&sharer_shareinfo=53dbd0e04a0f521872f3b1abc6e31c0e&sharer_shareinfo_first=53dbd0e04a0f521872f3b1abc6e31c0e#rd
---

大家好，我是祥子。

最近一年在研究 AI 工程化，记录了一些企业落地的实践经验。

---

Product Manager Skills、AI PM Workflows 还是 PM Skill Templates？产品经理该用哪个工作流框架？

刚接触 AI 辅助产品管理的 PM 都会遇到这个问题。

---

## 三大产品经理工作流框架

### Product Manager Skills：46 个 PM 技能库

GitHub 上有一个产品经理技能库，包含 46 个可重用的 PM 技能和命令工作流。不同于简单的提示词，这些技能教会你和 AI 代理如何专业地完成产品管理工作。

**三类技能：**

1. Component Skills（34 个） - 单一任务

* PRD 编写、用户故事拆分、优先级排序
* 竞品分析、市场调研、数据解读
* 用户画像、价值主张设计

2. Interactive Skills（6 个） - 上下文感知推荐

* AI 会询问 3-5 个关于你上下文的问题
* 提供定制化推荐（3-5 个选项）
* 使用正确的组件技能执行
* 例如：“我应该用哪个优先级框架？” → 询问产品阶段、团队规模、数据可用性 → 推荐 RICE/ICE/Kano

3. Workflow Skills（6 个） - 端到端流程

* 策略会议、发现周期、路线图规划、PRD 创建
* 编排多个组件和交互技能
* 完整的 PM 工作流（30-90 分钟）

**核心特点：**

* 结构化决策 - 不再是“写一个 PRD”然后期待最好的结果
* 可重用工作流 - 从问题定义到执行输出
* 知识传递 - PM 理解“为什么”，代理执行“如何”
* 持续改进 - 技能库随时间演进

安装：

```
1

2

git clone https://github.com/deanpeters/Product-Manager-Skills.git  
cp -r Product-Manager-Skills ~/.claude/skills/
```

有 Medium 文章评价：“AI 改变了我们做发现、定义成功、管理风险、与工程和数据团队协作的方式。它迫使 PM 适应不确定性、权衡取舍和持续学习循环。”

适用于需要全面 PM 技能支持、从策略到执行的完整流程。

---

### AI Product Management Workflows：结构化发现到验证

2026 年的产品管理工作流强调结构化、可重复的流程。不再是临时聊天、提示词和期待魔法，而是一个可靠、可测试的系统。

**核心工作流：**

1. Research（研究） - 用户反馈分析、市场调研

* 自动分类用户反馈
* 识别痛点和机会
* 竞品功能对比

2. Validation（验证） - 假设验证、原型测试

* 设计验证实验
* 用户访谈脚本生成
* A/B 测试方案

3. Prototyping（原型） - Vibe Coding 快速生成

* 使用 Gemini + Claude Code
* 几分钟内生成可交互的 HTML 原型
* 不是静态模型，而是功能完整的单页应用

4. Documentation（文档） - 自动化文档生成

* PRD 自动生成
* 用户故事拆分
* 验收标准定义

**Vibe Coding 工作流详解：**

* Gemini 分析需求和上下文
* Claude Code 生成功能性原型
* 可以点击、导航、批评、迭代
* 反馈循环从几周缩短到几小时

ProductSide 博客总结：“AI 不会取代 PM。它暴露薄弱的 PM。AI 不会让 PM 更好。它让差距更明显。”

适用于快速验证想法、生成原型测试、缩短反馈循环。

---

### PM Skill Templates：标准化模板库

PM Skill Templates 提供一套标准化的产品经理工作模板，覆盖产品开发生命周期的各个阶段。

**核心模板：**

1. Strategy Templates

* 产品愿景画布
* 商业模式画布
* 市场定位模板

2. Discovery Templates

* 用户画像模板
* 用户旅程地图
* 问题验证框架

3. Planning Templates

* PRD 模板
* 路线图模板
* 优先级矩阵

4. Metrics Templates

* OKR 模板
* 北极星指标框架
* KPI 看板

**核心特点：**

* 标准化流程 - 每个模板都有明确的填写指引
* 最佳实践 - 基于行业标准和成功案例
* 即用即走 - 无需复杂设置，直接使用
* 团队协作 - 易于共享和协作

安装：

```
1

2

git clone https://github.com/your-org/pm-skill-templates.git  
cp -r pm-skill-templates ~/.claude/skills/
```

适用于需要标准化文档、团队统一模板、快速上手的场景。

---

## 对比维度

| 维度 | Product Manager Skills | AI PM Workflows | PM Skill Templates |
| --- | --- | --- | --- |
| 技能数量 | 46 个（34 组件 + 6 交互 + 6 工作流） | 4 个核心工作流 | 15+ 标准模板 |
| 核心理念 | 技能库 + 知识传递 | 结构化流程 + 快速验证 | 标准化模板 |
| 上下文感知 | 有（Interactive Skills） | 有（AI 驱动） | 无 |
| 原型生成 | 无 | 有（Vibe Coding） | 无 |
| 学习曲线 | 中 | 低 | 低 |
| 定制化 | 高（可修改技能） | 中 | 低（固定模板） |
| 团队协作 | 高 | 中 | 高（共享模板） |
| 文档覆盖 | 全面 | 重点领域 | 全面 |

---

## 我的建议

需要全面的 PM 技能支持、从策略到执行的完整流程、AI 教会你和代理如何专业工作，选 Product Manager Skills。46 个技能覆盖产品管理的各个方面。

需要快速验证想法、生成原型测试、缩短反馈循环、重视实践验证，选 AI PM Workflows。Vibe Coding 让你在几分钟内得到可交互的原型。

需要标准化文档、团队统一模板、快速上手、重视一致性和协作，选 PM Skill Templates。固定模板确保团队使用相同的标准。

---

**组合使用：**

* 完整 PM 流程： Product Manager Skills（技能库）+ AI PM Workflows（快速验证）+ PM Skill Templates（标准化输出）
* 快速迭代： AI PM Workflows（原型）+ PM Skill Templates（文档）
* 团队协作： PM Skill Templates（统一模板）+ Product Manager Skills（深度技能）

---

**快速安装：**

Product Manager Skills：

```
1

2

git clone https://github.com/deanpeters/Product-Manager-Skills.git  
cp -r Product-Manager-Skills ~/.claude/skills/
```

AI PM Workflows：

```
1

2

# 使用 Claude Code 内置技能  
# 在 Claude 中："使用 AI PM Workflows 快速验证这个想法：[需求描述]"
```

PM Skill Templates：

```
1

2

git clone https://github.com/your-org/pm-skill-templates.git  
cp -r pm-skill-templates ~/.claude/skills/
```

---

如果这篇文章对你有帮助，欢迎点赞支持、分享给朋友、在评论区分享你的想法。期待交流。