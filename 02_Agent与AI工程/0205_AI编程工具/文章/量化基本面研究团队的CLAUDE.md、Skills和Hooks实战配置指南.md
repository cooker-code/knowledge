---
title: 量化基本面研究团队的CLAUDE.md、Skills和Hooks实战配置指南
author: Zen Traders
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkxMTIxNzgyMw==&mid=2247489627&idx=1&sn=e9fdcfeea61876c2bdb10d55000d152b&chksm=c03a97ab399431f6d18e403a04880ea2e07b995082faf5d8780c0e70ebe67240cb544e0dcccd&mpshare=1&scene=24&srcid=0416aJY4gGafkFCA6zAO2vz6&sharer_shareinfo=9ed07e5753da063052db2bc8592bd65a&sharer_shareinfo_first=9ed07e5753da063052db2bc8592bd65a#rd
---

> “
>
> 五人基本面研究团队的CLAUDE.md、Skills和Hooks实战配置指南

## 为什么要做这件事

我们是一个五人的量化基本面研究团队，日常围绕美股portfolio策略展开工作——从因子研究、SEC Filing解析到信号回测，大量时间花在重复性的数据处理和代码编写上。

拿到Claude Team版本后，我做的第一件事不是直接开干，而是**先搭好基础设施**。就像量化系统里你不会裸写策略一样，Claude Code也需要一套"策略框架"来确保团队五个人的使用体验一致、效率最大化。

这篇文章完整分享我们的配置方案，包括三大核心组件：**CLAUDE.md记忆系统**、**Skills技能扩展**和**Hooks自动化钩子**。

## 先回答最重要的问题：数据安全

做量化的人把策略代码和持仓数据喂给AI，第一反应一定是：**这些数据会不会被拿去训练模型？**

答案取决于你用的是哪个版本。

Claude的产品线分两大类：消费级（Free / Pro / Max）和商业级（Team / Enterprise / API）。在2025年9月的隐私政策更新后，消费级版本引入了训练数据的opt-in机制——如果用户选择同意，对话数据可能被保留最长5年用于模型训练。

**但Team版本属于商业级产品（Claude for Work），适用的是Commercial Terms而非Consumer Terms。** 核心区别在于：Anthropic在商业版中充当数据处理者（data processor），你的组织是数据控制者（data controller）。用户对话数据不会进入模型训练管道，无论你是否做了任何设置。

具体来说，Team版本的数据保护要点：

* **不用于训练：对话和代码数据默认不用于模型改进**
* **组织管控：Primary Owner可管理所有成员数据访问权限，可做数据导出审计**
* **与消费级隔离：完全不受Consumer Terms和opt-in机制影响**

所以对于做美股量化的团队，**Team版本在数据保护层面已经足够**。当然好的操作习惯仍然重要：不要在对话中直接粘贴完整持仓明细或API密钥——这是使用任何SaaS产品的基本原则。

CLAUDE.md里写的是代码规范和工作流，不是实际持仓数据，这一点分清楚就行。

## 一、CLAUDE.md：给Claude植入团队DNA

CLAUDE.md是Claude Code的持久记忆文件。每次会话启动时自动读取作为上下文。可以理解为：**你写给AI的团队手册**。

### 文件放在哪？

Claude Code支持多层级CLAUDE.md，作用域从大到小：

* **个人全局（`~/.claude/CLAUDE.md`）：你所有项目通用**
* **项目共享（`./CLAUDE.md`）：团队通过Git共享**
* **个人项目（`./CLAUDE.local.md`）：仅自己可见，加入.gitignore**

核心配置放在项目级CLAUDE.md里，通过Git共享给所有人。

### 我们的CLAUDE.md长什么样

以下是项目根目录CLAUDE.md的精简版：

```
# Quant Fundamental Research — US Equities  
  
## 技术栈  
- Python 3.11+ / Pandas / NumPy / Polars  
- 数据源：Bloomberg API, Compustat via WRDS, SEC EDGAR  
- 回测框架：自研engine（src/backtest/）  
- 因子库：src/factors/ | 策略库：src/strategies/  
  
## 代码规范  
- type hints + docstring  
- DataFrame列名snake_case  
- 因子命名：{category}_{name}_{window} 如 value_ep_ttm  
- 日期格式YYYY-MM-DD  
- 回测指标必含：Ann Return, Max DD, Sharpe, Turnover  
  
## 数据约定  
- Ticker格式：AAPL（纯ticker）或 AAPL US Equity（BBG）  
- GICS行业分类为默认行业体系  
- Portfolio权重之和 = 1.0  
- 缺失值：行业中位数填充优先  
- 收益率：对数收益率  
- 市值单位：百万美元
```

### 三条核心写作原则

1. **控制在200行以内。过长消耗context窗口，降低指令遵循度**
2. **指令具体可验证。"代码写好"是废话，"type hints + docstring"才有效**
3. **避免互相矛盾。五个人各自加规则，定期review删冲突内容**

### 进阶：rules目录拆分

CLAUDE.md过长时用`.claude/rules/`拆分，并支持路径匹配——只有编辑因子代码时才加载因子规范：

```
---  
paths:  
  - \"src/factors/**/*.py\"  
---  
# 因子开发规范  
- 因子函数接受DataFrame输入返回Series  
- 必须处理NaN  
- 必须包含IC测试
```

## 二、Skills：量化工具箱

Skills是技能扩展系统。写一个SKILL.md，Claude就学会一项新能力。可通过`/skill-name`手动调用，也可让Claude自动加载。

### 核心Skills一览

**因子IC分析** — `/factor-ic value_ep_ttm` 一键跑完Rank IC时序、ICIR、分组回测、多头组合绩效，输出结论。

**SEC Filing解析** — `/parse-filing AAPL-10K-2025` 自动提取Revenue、Net Income、Gross Margin、ROE、FCF，计算YoY增速，对比GICS行业中位数，标注异常值。

**策略回测报告** — `/backtest-report momentum_v2` 加载配置YAML，跑回测引擎，生成月度热力图、vs SPY超额曲线、分年度绩效表，输出PDF。设置了`disable-model-invocation: true`防止Claude自行触发。

**数据质量检查** — `/data-check sp500_prices` 统计缺失值、检测异常值、验证corporate action影响、检查前复权价格。

### Skill设计要点

每个Skill可以带辅助文件——模板、示例、脚本。保持SKILL.md精简，详细内容按需加载：

```
.claude/skills/factor-ic/  
├── SKILL.md           # 主指令  
├── template.md        # 报告模板  
├── examples/sample.md # 示例输出  
└── scripts/ic.py      # 分析脚本
```

两种内容类型要分清：**Reference**（行业知识、数据字典）让Claude自动加载；**Task**（回测、部署）加`disable-model-invocation`手动触发。

## 三、Hooks：确定性自动化

Hooks是生命周期钩子——在编辑文件、完成任务等节点自动执行Shell命令。和Skills不同，Hooks提供**确定性控制**，不是"尽量做"而是**一定执行**。

### 团队配置的四个关键Hook

**Python文件自动格式化** — Claude每次编辑.py文件后自动跑Black + isort。团队代码风格零分歧。用PostToolUse事件匹配Edit和Write工具。

**保护关键文件** — 生产策略配置（configs/prod\_\*）、密钥文件（.env）、回测引擎核心代码（engine.py），Claude碰不了。用PreToolUse + exit 2拦截。

**任务完成通知** — 跑长任务时可以切去做别的事，Claude干完活弹桌面通知。用Notification事件触发osascript。

**Compact后注入上下文** — 长对话被压缩后自动重新注入关键信息：当前sprint目标、benchmark设置、数据截止日期。用SessionStart匹配compact。

## 四、落地清单

**第一步** — 项目级CLAUDE.md：技术栈、代码规范、目录结构。200行以内，提交Git

**第二步** — .claude/rules/路径规则：因子开发、回测、数据处理各一个文件

**第三步** — 团队Skills：IC分析、Filing解析、回测报告、数据质检

**第四步** — Hooks自动化：格式化、文件保护、通知、Compact注入

**第五步** — 个人CLAUDE.local.md：常用universe、测试路径、Bloomberg端口。加入.gitignore

**关于安全** — 确认Team版本（Claude for Work）。CLAUDE.md写规范不写持仓。API key和alpha信号永远不粘贴到对话中

## 最后

这套CLAUDE.md + Skills + Hooks体系，本质在做一件事：**把隐性知识显性化，把重复操作自动化**。对量化团队来说，这和构建因子库、策略模板的思路完全一致——标准化才能规模化。配置一次，五个人受益。每人每天省30分钟，一个月就是50小时的研究时间。

在Alpha越来越难找的今天，效率本身就是Alpha。