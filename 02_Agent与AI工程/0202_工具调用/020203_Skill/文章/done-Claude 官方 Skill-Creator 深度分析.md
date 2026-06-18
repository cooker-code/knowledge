> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: Claude 官方 Skill-Creator 深度分析
author: AI Inception
date:
url: https://mp.weixin.qq.com/s?chksm=c2cfd838f5b8512eb56b44bbf39b06f2196a38fcdd885705f1bdbba57b5340f1a5edc31f999f&exptype=unsubscribed_card_recommend_article_u2i_mainprocess_coarse_sort_tlfeeds&ranksessionid=1776818955_4&req_id=1776819223105340&mid=2247483999&sn=2ea02175806f770922e23da783b61d7b&idx=1&__biz=Mzk0MTY5MDYzMg%3D%3D&scene=169&subscene=200&sessionid=1776818954&flutter_pos=33&clicktime=1776819266638&enterid=1776819266638&finder_biz_enter_id=5&key=daf9bdc5abc4e8d07ca30510581994c5d9891bd1d87203e67442fd816eb8a5a5d475d03919d40a7c0807c63ba1c8be975cab2730d693b28c76dc248df768f02dd1bd80ee3ce3b73dbddcc5f05cef985f1e11b1c15f6baa4c40c50064d1dd92ba92c25ae592616b6c7d14cd1ee4b6378892f811578e6eb3dd1465aad80d16de78&ascene=0&uin=MjAwNTI2NjEyMw%3D%3D&devicetype=OHOS-22&version=f3801037&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&countrycode=CN&exportkey=n_ChQIAhIQFrzstxerC64%2FLJOmQt1K7RLfAQIE97dBBAEAAAAAADcRK0XMSyIAAAAOpnltbLcz9gKNyK89dVj0n1rEhpAN95m8oy6QOL0jVsM82jahVXzhPp%2FztoAb%2BTpJFRt2IRrdj4QThRaPbT%2FENmc2gHpC%2F5ul66ZnUGqf%2BU4gr5%2FN6bn3sAKz2BARPYsfWG46I2JkxigXgkCqkJcR7WeSSeKJMT49zZ3QxzAC1aFsT%2FGGi6rWHds8adx9ZTZCjv96DZecgiU3WniEKOzw7me57kF5iAmAOUfGOygc4eUmhy7dEtvahCtXBWUNX%2BlVm%2Fde%2BszDgZU%3D&pass_ticket=NEtXmJljTxbs8pSZtxu%2BH%2B2CLzYYcfcbCjtTHZFyR30xzxHV107QGsFlSx8HwT59&wx_header=3
---

> 基于 Claude 官方 skill-creator 源码和 Anthropic 官方博客文章的分析 官方源码：github.com/anthropics/skills`skills/skill-creator/`（485 行 SKILL.md，2026-03-07 版本）

## 1. 概述与定位

### 1.1 什么是 Skill-Creator

Skill-Creator 是 Anthropic 官方提供的一个 **"元技能"(Meta-Skill)** —— 一个用来创建、测试、评估和改进其他 Skill 的工具。它本身就是一个 Skill，展示了 Skill 系统的自举能力。

### 1.2 一句话结论（给决策层）

`skill-creator` 本质是一个"**Skill 工程化系统**"，不是单纯写 `SKILL.md` 的模板器。 它把"写技能"升级为"**假设 -> 实验 -> 度量 -> 人审 -> 迭代**"的产品研发流程，重点解决三个问题：

* Skill 是否真的提升了结果质量（而非心理安慰）
* Skill 是否在正确场景被触发（触发精度/召回）
* Skill 是否在模型升级后仍有价值（防止过时）

### 1.3 解决的核心问题

根据 Anthropic 官方博客：

> "most authors are subject matter experts, not engineers" — 大多数 Skill 作者是领域专家而非工程师。

Skill-Creator 将**软件开发的严谨性（测试、基准测试、迭代改进）带给 Skill 创作过程，**而无需作者编写代码。

### 1.4 两类 Skill 的区分

| 类型 | 英文 | 说明 | 示例 |
| --- | --- | --- | --- |
| **能力提升型** | Capability Uplift | 让 Claude 能做到基础模型无法稳定完成的事 | 使用特定技术创建文档 |
| **偏好编码型** | Encoded Preference | 按照组织工作流编排 Claude 已有的能力 | NDA 审查流程 |

> **这个区分很重要：**能力提升型 Skill 可能随模型进化而过时，偏好编码型 Skill 则需要持续验证是否匹配实际团队流程。两类 Skill 的评估逻辑不同：提升型重点看 "with-skill vs without-skill" 是否拉开；流程型重点看是否稳定遵守团队规范。

### 1.5 分析依据

* 官方源码：Claude 官方 skill-creator（`SKILL.md` 485 行，2026-03-07 版本）
* Anthropic 官方公开资料：

+ Improving skill-creator: Test, measure, and refine Agent Skills
+ Extend Claude with skills (Claude Code docs)
+ Agent Skills overview
+ Skill authoring best practices
+ Equipping agents for the real world with Agent Skills

## 2. 核心设计理念

### 2.1 "草稿 -> 测试 -> 评估 -> 改进 -> 重复" 循环

```
+--------------------------------------------------+
|                                                  |
|   捕获意图 -> 面试调研 -> 撰写 SKILL.md            |
|       |                                          |
|   创建测试用例 -> 运行(with/without skill)         |
|       |                                          |
|   评估与评分 -> 生成基准报告                       |
|       |                                          |
|   人工审核(Viewer) -> 反馈                         |
|       |                                          |
|   改进 Skill -> [回到运行测试]                     |
|       |                                          |
|   描述优化 -> 打包发布                             |
|                                                  |
+--------------------------------------------------+
```

来源：`SKILL.md` 开头的工作流概述章节。

### 2.2 设计哲学

1. **渐进式加载 (Progressive Disclosure)：**

   三层加载系统，按需消耗上下文（详见 4.1）
2. **无意外原则 (Principle of Lack of Surprise)：**

   Skill 不得包含恶意软件、漏洞利用代码或任何危害系统安全的内容；Skill 的实际行为不应超出描述范围（SKILL.md "Principle of Lack of Surprise" 章节）
3. **解释 Why 而非强制 Must：**

   用理论思维和推理代替死板的大写 MUST 指令（SKILL.md "Skill Writing Guide" 章节）
4. **泛化而非过拟合：**

   从反馈中泛化改进，而非针对特定测试用例做窄优化（SKILL.md "Improving the skill" 章节）
5. **人在环中 (Human-in-the-loop)：**

   关键决策由人类审核，自动化处理重复工作

## 3. 架构总览

### 3.1 三大功能模块

> **注意：**下面描述的是 skill-creator 的功能模块划分，与第 4.1 节的"渐进式加载三层"是不同维度的概念，不要混淆。

```
                        +----------------------+
                        |    用户 (User)        |
                        |  - 领域专家           |
                        |  - 非工程师背景       |
                        +----------+-----------+
                                   |
                    +--------------+--------------+
                    |      Skill-Creator          |
                    |     (Meta-Skill)            |
                    +--------------+--------------+
                                   |
           +-----------------------+-----------------------+
           |                       |                       |
    +------+------+        +------+------+        +------+------+
    |  创建模块    |        |  评测模块    |        |  优化模块    |
    |             |        |             |        |             |
    | - 意图捕获   |        | - 并行测试   |        | - 描述优化   |
    | - 面试调研   |        | - 评分系统   |        | - 盲比较     |
    | - SKILL.md  |        | - 基准聚合   |        | - 循环改进   |
    |   生成      |        | - 可视化审核  |        | - 打包发布   |
    +-------------+        +------+------+        +-------------+
                                  |
                    +-------------+-------------+
                    |             |             |
             +------+---+  +-----+----+  +----+------+
             | Grader   |  |Comparator|  | Analyzer  |
             | Agent    |  | Agent    |  | Agent     |
             +----------+  +----------+  +-----------+
```

### 3.2 核心组件关系

| 组件 | 职责 | 输入 | 输出 |
| --- | --- | --- | --- |
| SKILL.md | 技能定义与指令 | 用户意图 | 结构化技能文档 |
| Grader Agent | 评分断言 + 批判评估质量 | 执行记录 + 输出文件 | grading.json |
| Comparator Agent | 盲比较 | 两份输出(A/B) | comparison.json |
| Analyzer Agent | 模式分析（双模式，详见 6.3） | benchmark.json / comparison.json | analysis.json / notes |
| Eval Viewer | 可视化审核 | 运行结果 + 基准数据 | feedback.json |
| 优化循环 | 描述优化 | 触发评估集 | best\_description |

## 4. 目录结构详解

```
skill-creator/
+-- SKILL.md                    # [485行] 核心技能定义文档
+-- LICENSE.txt                 # Apache 2.0 许可证
|
+-- agents/                     # 子智能体指令文档
|   +-- grader.md               # [223行] 评分智能体
|   +-- comparator.md           # [202行] 盲比较智能体
|   +-- analyzer.md             # [274行] 分析智能体（含两种模式）
|
+-- scripts/                    # 可执行 Python 工具
|   +-- __init__.py             # 包初始化
|   +-- utils.py                # [47行] 共享工具函数(SKILL.md 解析器)
|   +-- run_eval.py             # [310行] 触发评估脚本
|   +-- run_loop.py             # [328行] 完整优化循环(训练/测试分割)
|   +-- improve_description.py  # [247行] Claude 驱动的描述改进
|   +-- aggregate_benchmark.py  # [401行] 基准结果聚合
|   +-- generate_report.py      # [326行] HTML 报告生成(含自动刷新)
|   +-- package_skill.py        # [136行] 打包为 .skill 文件
|   +-- quick_validate.py       # [102行] 基础验证
|
+-- references/                 # 参考文档
|   +-- schemas.md              # [430行] 所有 JSON 格式规范
|
+-- eval-viewer/                # 交互式审核界面
|   +-- generate_review.py      # [471行] 生成并服务审核页面
|   +-- viewer.html             # 交互式 HTML 查看器模板
|
+-- assets/                     # 模板文件
    +-- eval_review.html        # 评估集审核 HTML 模板
```

### 4.1 渐进式加载三层（Progressive Disclosure）

这是所有 Skill 共享的核心加载机制（SKILL.md "Progressive Disclosure" 章节）：

| 层级 | 内容 | 加载时机 | 大小约束 |
| --- | --- | --- | --- |
| **Level 1: 元数据** | name + description | 始终在上下文中 | ~100 words |
| **Level 2: SKILL.md 正文** | 详细指令 | 技能触发时加载 | ≤500 行（推荐值） |
| **Level 3: 捆绑资源** | scripts/, references/, assets/ | 按需加载 | 无限制 |

> **关于目录结构的两种场景：**SKILL.md 描述的基本迭代流程（Step 1-5）将 outputs 和 grading 直接放在 `with_skill/` 下；而 `aggregate_benchmark.py` 则期望 `with_skill/run-1/、``run-2/` 这样的多次运行子目录（`config_dir.glob("run-*")`）。前者是"逐轮迭代 + 人审"的流程，后者是"多次重复跑以计算方差"的基准流程。如果你要用 `aggregate_benchmark.py，`务必按 `run-*` 子目录组织。

> **关键设计点：**脚本可以直接执行而无需加载到上下文中（SKILL.md Progressive Disclosure §L3），这大幅节省了 token 消耗。当 SKILL.md 接近 500 行时，应拆分内容到 `references/` 并在 SKILL.md 中给出清晰指针。

## 5. 核心文件 SKILL.md 解析

### 5.1 YAML Frontmatter（必填）

```
---
name:skill-creator
description:Createnewskills,modifyandimproveexistingskills,andmeasure
skillperformance.Usewhenuserswanttocreateaskillfromscratch,
edit,oroptimizeanexistingskill,runevalstotestaskill,
benchmarkskillperformancewithvarianceanalysis,oroptimizea
skill'sdescriptionforbettertriggeringaccuracy.
---
```

**字段约束：**

* `name:`

  kebab-case，最长 64 字符，不能以连字符开头/结尾
* `description:`

  最长 1024 字符，不能包含尖括号，100-200 词最佳
* 可选字段（本仓库 `quick_validate.py` 校验范围）: `license`, `allowed-tools`, `metadata`, `compatibility`
* **注意：**

  Claude Code 产品面实际支持更多 frontmatter 字段（如 `user-invocable`, `argument-hint`, `disable-model-invocation`, `context`, `agent`, `hooks` 等），以上仅为本仓库校验器的限制，不等同官方完整规范。

### 5.2 Description 的设计策略

Description 是 **Skill 触发的主要机制。**Anthropic 特别指出 Claude 倾向于"欠触发"（不使用可用的 Skill），因此建议描述要有一定"推动性"（参见 Anthropic "Improving skill-creator" 博文）：

**弱描述：**

```
How to build a simple fast dashboard to display internal data.
```

**强描述：**

```
How to build a simple fast dashboard to display internal data.
Make sure to use this skill whenever the user mentions dashboards,
data visualization, internal metrics, or wants to display any kind
of company data, even if they don't explicitly ask for a 'dashboard.'
```

**描述应包含：**

1. Skill 做什么（功能）
2. 什么时候应该触发（场景枚举）
3. 使用祈使语气（"Use this skill when..."）
4. 解释 WHY Claude 应该触发，而不仅仅是 WHAT

### 5.3 SKILL.md 正文结构模式

基于 skill-creator 自身的 SKILL.md 结构提取：

```
# Skill Name

## 概述
[简要说明技能做什么和核心工作流]

## 与用户沟通
[如何适配不同技术背景的用户]

## 创建/执行流程
[详细的分步指令]

## 评估与改进
[如何测试和迭代]

## 高级功能
[可选的进阶用法]

## 平台适配
[Claude Code / Claude.ai / Cowork 差异]

## 参考文件
[指向 agents/, references/, scripts/ 的链接和使用时机]
```

### 5.4 写作风格指南

来源：SKILL.md "Skill Writing Guide" 章节 + Anthropic 官方最佳实践

| 原则 | 说明 | 来源 |
| --- | --- | --- |
| 祈使语气 | 直接告诉模型做什么 | "Prefer using the imperative form" |
| 解释 Why | 给出原因而非硬性规则 | "explain to the model why things are important in lieu of heavy-handed musty MUSTs" |
| 示例驱动 | 用 Input/Output 对展示 | SKILL.md 示例章节 |
| 避免过度约束 | 如果发现自己在写大写的 MUST/NEVER，改用推理解释 | "if you find yourself writing ALWAYS or NEVER in all caps...that's a yellow flag" |
| 理论思维 | 让模型理解任务而非死记步骤 | "Use theory of mind" |

## 6. 多智能体系统设计

### 6.1 Grader Agent（评分智能体）

**文件:**`agents/grader.md`（223 行）

**双重使命**（grader.md: "You have two jobs: grade the outputs, and critique the evals themselves"）：

1. **评分：**

   基于执行记录和输出文件，判断每个期望是否通过
2. **批判评估：**

   识别弱断言、遗漏断言、不可验证的断言

**8 步流程：**

```
Step 1: 读取执行记录（完整阅读 transcript）
Step 2: 检查输出文件（用工具检查，不只看 transcript 的声称）
Step 3: 逐条评估断言（搜索证据 + 判定 PASS/FAIL）
Step 4: 提取隐含声明（事实性、过程性、质量性声明并验证）
Step 5: 读取用户笔记（user_notes.md，即使断言通过也可能有问题）
Step 6: 批判评估质量（只在确有差距时指出）
Step 7: 写入评分结果（grading.json）
Step 8: 读取执行指标和计时（metrics.json + timing.json）
```

**评分标准**（严格证据导向）：

| 判定 | 条件 |
| --- | --- |
| PASS | 有明确证据证明期望为真，且反映**真实任务完成**而非表面合规 |
| FAIL | 无证据、证据矛盾、无法验证、或仅是表面满足（如文件名正确但内容为空） |
| 不确定时 | 举证责任在期望一方——偏向 FAIL（grader.md: "The burden of proof to pass is on the expectation"） |

**关键设计：**Grader 不仅判断 pass/fail，还会：

* **提取隐含声明**

  (claims)：事实性、过程性、质量性声明（Step 4）
* **反馈评估质量**

  (eval\_feedback)：指出哪些断言太弱或有遗漏（Step 6）
* eval\_feedback 有质量门槛："flag things the eval author would say 'good catch' about, not to nitpick every assertion"（grader.md）

### 6.2 Comparator Agent（盲比较智能体）

**文件:**`agents/comparator.md`（202 行）

**职责：**在不知道哪个 Skill 产生了哪个输出的情况下，判断哪个输出更好

**关键设计 —— 消除偏见：**

* 两个输出标记为 A 和 B
* 比较者不知道也**不能尝试推断**哪个 Skill 产生了哪个输出（comparator.md: "DO NOT try to infer which skill produced which output"）
* 仅基于输出质量和任务完成度判断

**评分体系：**

```
内容维度 (Content Rubric)
+-- 正确性 (Correctness): 1-5
+-- 完整性 (Completeness): 1-5
+-- 准确性 (Accuracy): 1-5

结构维度 (Structure Rubric)
+-- 组织性 (Organization): 1-5
+-- 格式化 (Formatting): 1-5
+-- 可用性 (Usability): 1-5

决策优先级：
1. 主要：总体评分 (内容 + 结构)
2. 次要：断言通过率
3. 平手：真正相等才宣布 TIE（comparator.md: "ties should be rare"）
```

### 6.3 Analyzer Agent（分析智能体）—— 双模式

**文件:**`agents/analyzer.md`（274 行）

> **重要**：Analyzer 有**两个完全独立的模式，**由文件中的 `---` 分隔，职责和输出格式完全不同。混用会导致错误。

#### 模式 1：事后分析（Post-hoc Analysis，analyzer.md 前半部分）

**触发场景：**盲比较完成后运行，"解盲"分析

**流程：**

1. 读取盲比较结果 -> 读取双方 Skill 和执行记录
2. 分析指令遵循度（评分 1-10）
3. 识别赢家优势和输家劣势
4. 生成带优先级的改进建议（high/medium/low）
5. 输出 `analysis.json`（包含 `improvement_suggestions`）

**改进建议分类：**

| 类别 | 说明 |
| --- | --- |
| `instructions` | Skill 文本指令的修改 |
| `tools` | 脚本、模板、工具的增删改 |
| `examples` | 示例输入/输出的补充 |
| `error_handling` | 错误处理指导 |
| `structure` | Skill 内容重组 |
| `references` | 外部文档或资源引用 |

#### 模式 2：基准分析（Benchmark Analysis，analyzer.md 后半部分）

**触发场景：**基准测试聚合后运行

**职责**：发现模式和异常，**明确禁止提改进建议**

```
analyzer.md: "DO NOT: Suggest improvements to the skill
(that's for the improvement step, not benchmarking)"
```

**分析内容：**

* 总是通过/失败的断言（无区分力）
* 高方差评估（可能是 flaky）
* 时间/token 权衡
* 输出 notes 数组（JSON array of strings）

## 7. 评估与测试体系

### 7.1 测试执行的 5 步流程

来源：SKILL.md "Running and evaluating test cases" 章节

```
Step 1: 并行启动所有运行（with-skill + baseline）
     |   - 新技能: baseline = 无技能
     |   - 改进技能: baseline = 旧版本快照
     |   关键：同一 turn 启动所有 subagent（"Launch everything at once"）
     v
Step 2: 利用等待时间起草断言
     |   - 客观可验证
     |   - 描述性命名（一眼看懂，而非 "test-1"）
     |   - 避免主观判断
     v
Step 3: 捕获计时数据
     |   - total_tokens, duration_ms
     |   - 来自任务通知，一次性机会（"only opportunity to capture this data"）
     v
Step 4: 评分 + 聚合 + 启动查看器
     |   a) Grader 评分
     |   b) aggregate_benchmark.py 聚合
     |   c) Analyzer 基准分析模式
     |   d) generate_review.py 查看器
     v
Step 5: 读取反馈 (feedback.json)
     |   - 空反馈 = 认可（"Empty feedback means the user thought it was fine"）
     |   - 聚焦有具体意见的用例
```

### 7.2 工作空间目录结构

```
skill-name-workspace/
+-- iteration-1/
|   +-- eval-descriptive-name/
|   |   +-- eval_metadata.json     # 评估元数据（含 prompt, assertions）
|   |   +-- with_skill/            # 始终存在：用新版 skill 执行的运行
|   |   |   +-- outputs/           # 技能产出
|   |   |   +-- grading.json       # 评分结果
|   |   |   +-- timing.json        # 计时数据
|   |   +-- without_skill/         # 基线（新建 skill 时：无 skill）
|   |       +-- outputs/           # 或改为 old_skill/（改进已有 skill 时：旧版本）
|   |       +-- grading.json
|   |       +-- timing.json
|   +-- benchmark.json             # 聚合基准
|   +-- benchmark.md               # 可读基准报告
|   +-- feedback.json              # 用户反馈
+-- iteration-2/
|   +-- ...
+-- skill-snapshot/                 # 旧版本快照(改进模式)
```

> **注意：**上面是 SKILL.md 中描述的基本迭代结构（每个 eval 一次运行）。若要使用 `aggregate_benchmark.py` 做多次重复基准测试，需要在 config 目录下增加 `run-*` 子目录层级：

```
+-- eval-descriptive-name/
|   +-- with_skill/
|   |   +-- run-1/
|   |   |   +-- outputs/
|   |   |   +-- grading.json
|   |   +-- run-2/
|   |       +-- outputs/
|   |       +-- grading.json
|   +-- without_skill/
|       +-- run-1/
|       |   +-- ...
|       +-- run-2/
|           +-- ...
```

### 7.3 断言质量要求

| 好的断言 | 坏的断言 |
| --- | --- |
| 客观可验证 | 主观判断 |
| 有区分力（skill vs no-skill 结果不同） | 总是通过（如"输出是文件"） |
| 检查内容正确性 | 仅检查文件名存在（Grader 也会批判这类弱断言） |
| 描述性命名（一眼看懂） | 泛化命名如"test-1" |
| 可程序化验证的用脚本（"write and run a script rather than eyeballing it"） | 所有都人工目视检查 |

### 7.4 Eval Viewer 交互式审核

**Outputs 标签页：**

* 一次显示一个测试用例
* 显示 Prompt、输出文件（内联渲染：文本/图片/PDF/XLSX）
* 评分详情（折叠展示）
* 反馈文本框（自动保存）
* 迭代 2+ 显示上次输出和反馈（折叠）
* 支持键盘导航（左右箭头）

**Benchmark 标签页：**

* 统计摘要：通过率、计时、token 使用
* 每个配置的对比（配置名动态读取，常见命名为 with\_skill vs without\_skill 或 old\_skill，viewer 不硬编码）
* 分析师观察笔记

**实时刷新**（源码验证）：Server 模式下每次 GET 请求都会重新扫描 workspace 目录并重新生成 HTML（`generate_review.py` 的 HTTP handler），这意味着评测运行中刷新浏览器就能看到新结果，不需要重启 server。

## 8. 描述优化系统

### 8.1 触发机制原理

来源：SKILL.md "Trigger optimization" 章节

Skill 出现在 Claude 的 `available_skills` 列表中，Claude 基于 description 决定是否使用某个 Skill。

**重要认知：**Claude 只会为自己无法轻易独立处理的任务咨询 Skill。简单的一步操作（如"读取这个 PDF"）即使描述完美匹配，也可能不会触发 Skill——因为 Claude 可以直接用基础工具处理。

### 8.2 四步优化流程

#### Step 1: 生成触发评估查询集（20 条）

来源：SKILL.md "Trigger optimization" 章节 Step 1

**应触发 (8-10 条)：**

* 不同措辞的相同意图
* 不显式提及 Skill 名称但明显需要的场景
* 罕见用例和与其他 Skill 竞争但应胜出的场景

**不应触发 (8-10 条)：**

* "近似失误" —— 共享关键词但实际需要不同的查询
* 模糊措辞（朴素关键词匹配会触发但不应触发）
* 涉及 Skill 功能但在其他工具更合适的上下文

**查询质量要求：**

差：`"Format this data"`, `"Extract text from PDF"`

好：`"ok so my boss just sent me this xlsx file (its in my downloads, called something like 'Q4 sales final FINAL v2.xlsx') and she wants me to add a column that shows the profit margin as a percentage. The revenue is in column C and costs are in column D i think"`

#### Step 2: 用户审核（HTML 界面）

* 使用 `assets/eval_review.html` 模板
* 可编辑查询、切换 should-trigger/should-not-trigger、增删条目
* 导出为 `eval_set.json`

#### Step 3: 运行优化循环

```
python -m scripts.run_loop \
  --eval-set <path> \
  --skill-path <path> \
  --model <model-id> \
  --max-iterations 5 \
  --verbose
```

**内部工作流：**

1. **分层抽样分割：**

   默认 60% 训练集 / 40% 测试集（`--holdout` 参数可调，默认 0.4；设为 0 则不拆分）
2. **评估当前描述：**

   每条查询运行 3 次（默认），计算触发率
3. **Claude 自动改进描述：**

   通过 `claude -p` 子进程调用 Claude，基于失败案例提出改进（只传训练集结果，详见 12.3）
4. **重新评估：**

   在训练集 + 测试集上评估新描述
5. **选择最佳**

   ：按**测试集**分数选择（避免过拟合）
6. **生成实时 HTML 报告：**

   带自动刷新

#### Step 4: 应用结果

取 `best_description` 更新 SKILL.md frontmatter

## 9. 数据流架构

### 9.1 完整数据流

```
用户输入 (意图、反馈)
    |
    v
+---------------------+
|    SKILL.md          | <-- 包含 name, description, 指令
|    (技能定义)        |
+----------+----------+
           |
           v
+---------------------+
|   evals.json         | <-- 测试用例 (prompt, expectations)
|   (测试定义)         |
+----------+----------+
           |
     +-----+------+
     |             |
     v             v
+---------+  +----------+
|with_skill|  |without_  |     <-- 并行子智能体执行（同一 turn）
|  运行    |  |skill 运行|
+----+----+  +----+-----+
     |             |
     v             v
+---------------------+
|   Grader Agent       | <-- 读取 agents/grader.md
|   -> grading.json     |
+----------+----------+
           |
           v
+---------------------+
|  aggregate_benchmark | <-- scripts/aggregate_benchmark.py
|  -> benchmark.json    |
|  -> benchmark.md      |
+----------+----------+
           |
           v
+---------------------+
|  Analyzer Agent      | <-- agents/analyzer.md (基准分析模式)
|  -> notes[]           |     注意：此模式不输出改进建议
+----------+----------+
           |
           v
+---------------------+
|  Eval Viewer         | <-- eval-viewer/generate_review.py
|  (浏览器交互)        |     Server 模式实时刷新，刷新即看新结果
+----------+----------+
           |
           v
+---------------------+
|  feedback.json       | <-- 用户反馈
+----------+----------+
           |
           v
+---------------------+
|  改进 SKILL.md       | <-- 基于反馈修改
+----------+----------+
           |
           +-->  [回到执行步骤，新的 iteration-N+1]
```

### 9.2 盲比较数据流（高级模式）

```
两个 Skill 版本各自产生输出
         |
         v
  Comparator Agent          <-- 盲比较，标记为 A/B
  -> comparison.json
         |
         v
  Analyzer Agent (事后分析模式)  <-- 解盲，分析原因
  -> analysis.json                   此模式才输出 improvement_suggestions
```

## 10. JSON Schema 体系

### 10.1 Schema 一览

| Schema | 文件位置 | 用途 | 关键字段 |
| --- | --- | --- | --- |
| **evals.json** | `evals/evals.json` | 测试用例定义 | skill\_name, evals[]{id, prompt, expected\_output, files, expectations} |
| **history.json** | workspace 根目录 | 版本进化追踪（Improve 模式）。注意：当前 `scripts/*` 主流程不直接读写此文件，它是 `references/schemas.md` 中定义的规范格式 | started\_at, skill\_name, current\_best, iterations[] |
| **eval\_metadata.json** | `<eval-dir>/` | 单个评估元数据 | eval\_id, eval\_name, prompt, assertions |
| **grading.json** | `<run-dir>/grading.json` | 评分输出 | expectations[]{text, passed, evidence}, claims[], eval\_feedback |
| **metrics.json** | `<run-dir>/outputs/metrics.json` | 执行指标 | tool\_calls, files\_created, errors |
| **timing.json** | `<run-dir>/timing.json` | 计时数据 | total\_tokens, duration\_ms, total\_duration\_seconds |
| **benchmark.json** | iteration 目录 | 基准聚合 | runs[], run\_summary{mean, stddev}, notes[] |
| **comparison.json** | `<grading-dir>/` | 盲比较结果 | winner, rubric, output\_quality |
| **analysis.json** | `<grading-dir>/` | 事后分析 | improvement\_suggestions[], instruction\_following |
| **feedback.json** | workspace/iteration 目录 | 用户反馈 | reviews[]{run\_id, feedback}, status |

### 10.2 字段名精确性要求

> **重要警告：**Viewer 读取精确的字段名。使用 `config` 而非 `configuration，`或将 `pass_rate` 放在 run 顶层而非嵌套在 `result` 下，都会导致 Viewer 显示空值。

关键精确要求（已验证 `aggregate_benchmark.py`）：

* benchmark.json 中必须用 `configuration`（非 `config`）
* 结果必须嵌套在 `result` 对象下
* grading.json 的 expectations 必须使用 `text`, `passed`, `evidence`（非 `name`/`met`/`details`）

> **源码术语不一致警告：**官方源码中 `assertions` 和 `expectations` 两个术语并存于不同 JSON 契约中：
>
> * `eval_metadata.json`
>
>   （SKILL.md 定义）使用 `assertions`
> * `evals.json`
>
>   和 `grading.json`（schemas.md 定义）使用 `expectations`
>
> 两者指代相同概念但字段名不同，使用时以各自 JSON 文件对应的 schema 为准，不要混用。

## 11. 多平台适配策略

### 11.1 三平台对比

来源：SKILL.md "Platform Adaptation" 章节

| 特性 | Claude Code | Claude.ai | Cowork |
| --- | --- | --- | --- |
| 子智能体 | 完整支持 | 无 | 支持（可能超时则串行） |
| 并行执行 | 完整 | 串行 | 完整（超时问题时降级串行） |
| 浏览器 | 直接打开 | 可能不可用（条件判断） | 无显示 |
| Viewer 模式 | HTTP 服务（实时刷新） | 跳过/内联展示 | `--static` 静态 HTML |
| 基线运行 | with/without 或 old\_skill | 仅 with-skill | with/without 或 old\_skill |
| 定量基准 | 完整 | 跳过 | 完整 |
| 盲比较 | 完整 | 需要子智能体，跳过 | 完整 |
| 描述优化 | 完整（需要 `claude -p`） | 不可用 | 完整 |
| 打包 | 完整 | 完整 | 完整 |

### 11.2 Claude.ai 适配要点

来源：SKILL.md "Platform Adaptation" 章节

* 无子智能体：自己读取 SKILL.md 然后执行，一次一个（"less rigorous than independent subagents"）
* 浏览器可能不可用：原文为条件判断 "If you can't open a browser (e.g., Claude.ai's VM has no display, or you're on a remote server), skip the browser reviewer entirely"。如果不可用则直接在对话中展示结果。
* 跳过定量基准：依赖用户定性反馈
* 迭代循环不变：改进 -> 重新执行 -> 请求反馈

### 11.3 Cowork 适配要点

来源：SKILL.md "Platform Adaptation" 章节

* 使用 `--static <output_path>` 生成静态 HTML
* 反馈通过下载 `feedback.json` 文件
* **必须**

  在自己评估之前先生成 Eval Viewer 给人看（大写强调 "GENERATE THE EVAL VIEWER *BEFORE* evaluating inputs yourself"）
* 描述优化正常工作（使用 `claude -p` 子进程）

## 12. 关键工程机制深度解析

> 本节基于源码逐行验证，是企业复用价值最高的部分。

### 12.1 触发评测引擎：`run_eval.py`

**核心创新：**不是简单 regex，而是通过 `claude -p` + 流式事件检测 Skill 是否被调用。

**技术实现：**

1. **注入被测描述：**创建临时 `.claude/commands/<name>.md` 文件，使描述出现在 Claude 的 `available_skills` 列表中（`run_eval.py``run_single_query` 函数）
2. **流式早期检测：**使用 `--output-format stream-json --include-partial-messages` 参数，通过 `content_block_start` 和 `content_block_delta` 事件做早期检测，而非等待完整输出（`run_single_query` 函数内 stream event 处理）
3. **嵌套执行处理：**移除 `CLAUDECODE` 环境变量以允许在 Claude Code 会话内嵌套运行 `claude -p`（`run_single_query` 函数）：

   ```
   # Remove CLAUDECODE env var to allow nesting claude -p inside a
   # Claude Code session. The guard is for interactive terminal conflicts;
   # programmatic subprocess usage is safe.
   env = {k: v for k, v in os.environ.items() if k != "CLAUDECODE"}
   ```

   > 如果企业要复用这套脚本，不了解这个机制可能会遇到嵌套执行失败的问题。
4. **并行多次运行：**使用 `ProcessPoolExecutor` 并行执行（`run_eval` 函数），每条查询默认跑 3 次（`--runs-per-query` 参数），计算触发率后与阈值（默认 0.5）比较判定通过/失败
5. **触发判定的早返回逻辑**（`run_single_query` 函数内）：

   ```
   if tool_name in ("Skill", "Read"):
       pending_tool_name = tool_name
       accumulated_json = ""
   else:
   returnFalse# 第一个工具调用不是 Skill/Read 就判定未触发
   ```

   > 这是**有意的设计决策：**如果 Claude 的第一个工具调用就不是读取 Skill，说明它选择了直接处理，不需要 Skill。这在绝大多数场景下是正确的，但理论上存在极小概率的假阴性——Claude 先做了一个预处理工具调用再读 Skill。

### 12.2 自动优化循环：`run_loop.py`

**核心价值：**把 description 优化建模为机器学习问题，引入 train/test split 防过拟合。

**关键参数**（均可命令行覆盖）：

* `--holdout 0.4：`

  默认 40% 测试集（设为 0 则不拆分）
* `--max-iterations 5：`

  最大迭代次数
* `--runs-per-query 3：`

  每条查询重复次数
* `--trigger-threshold 0.5：`

  触发率阈值

**分层抽样：**按 `should_trigger` 分层（`split_eval_set` 函数），确保训练集和测试集都包含正例和负例。

**选型逻辑**（`run_loop` 函数）：

```
# Find the best iteration by TEST score (or train if no test set)
if test_set:
    best = max(history, key=lambda h: h["test_passed"] or0)
else:
    best = max(history, key=lambda h: h["train_passed"])
```

### 12.3 Blinded History —— 最精妙的防过拟合设计

> **这是整个系统中最值得注意的工程细节，容易被忽略。**

`run_loop.py` 在调用 `improve_description` 时，会**剥除所有 test 相关数据：**

```
# run_loop.py — blinded history 机制
# Strip test scores from history so improvement model can't see them
blinded_history = [
    {k: v for k, v in h.items() ifnot k.startswith("test_")}
for h in history
]
new_description = improve_description(
    ...
    eval_results=train_results,  # 只传训练集结果
    history=blinded_history,     # 历史中也没有测试分数
    ...
)
```

**含义**：改进模型（通过 `claude -p` 调用的 Claude）**完全不知道测试集的存在，**也看不到测试分数。它只能基于训练集的失败进行泛化改进。最终选型时才用测试集评判。

这是一个标准的 ML 工程最佳实践，完整实现了"训练集指导优化、测试集评判泛化"的分离。

### 12.4 Description 改写机制：`improve_description.py`

**实现方式：**通过 `claude -p --output-format text` 子进程调用 Claude（与 run\_eval.py 相同的认证模式，使用会话级 Claude Code auth，无需单独的 ANTHROPIC\_API\_KEY）。prompt 通过 stdin 传入（因为包含完整 SKILL.md 内容，可能超出 argv 长度限制）。

**关键设计：**

1. **失败驱动：**分析 false negative（应触发没触发）和 false positive（不应触发却触发了）
2. **历史记忆：**注入所有历史尝试，要求"try something structurally different"，避免重复无效策略
3. **泛化要求：**prompt 中明确要求不要过拟合：
   > "what I DON'T want you to do is produce an ever-expanding list of specific queries that this skill should or shouldn't trigger for. Instead, try to generalize..."
4. **字符限制：**超过 1024 字符自动触发二次压缩（内联上一轮输出到新 prompt 中请求缩短）

### 12.5 Benchmark 聚合：`aggregate_benchmark.py`

**关键设计：**

1. **动态配置发现：**不硬编码 `with_skill/without_skill，`而是遍历目录动态发现所有配置（`load_run_results` 函数），支持 A/B 多版本扩展
2. **双目录布局支持：**兼容 workspace 布局和 legacy 布局
3. **Delta 计算**：取前两个 config 的均值差（`aggregate_results` 函数），**顺序敏感**——SKILL.md 强调 "Put each with\_skill version before its baseline counterpart"
4. **统计精度：**使用样本标准差（n-1 除法，`calculate_stats` 函数），但不做显著性检验

### 12.6 人审界面：`eval-viewer/generate_review.py`

**关键设计：**

1. **Python 侧零依赖：**仅使用 Python 标准库（`http.server`, `base64`, `json`）。但前端 HTML 模板（`viewer.html`）依赖外部 CDN（Google Fonts、SheetJS 等）。离线环境中 XLSX 预览和字体加载会失败，文本类输出不受影响。
2. **双模式服务：**

* Server 模式（默认）：HTTP 服务 + 自动打开浏览器
* Static 模式（`--static`）：生成独立 HTML 文件

3. **实时内容刷新：**每次 GET 请求重新扫描目录，不缓存
4. **丰富的文件嵌入：**文本内联显示、图片 base64 嵌入、PDF 嵌入、XLSX 嵌入、二进制文件下载链接
5. **跨迭代对比：**`--previous-workspace` 参数加载上轮输出和反馈

## 13. 关键设计模式与最佳实践

### 13.1 Skill 编写 12 条军规

来源：综合 SKILL.md 全文 + Anthropic 官方博客

1. **Description 要"推"一点：**

   主动列举触发场景，对抗"欠触发"倾向
2. **SKILL.md < 500 行：**

   超长内容移到 references/ 并加目录
3. **解释 Why 而非强制 Must：**

   用推理代替死板规则
4. **用示例驱动：**

   Input/Output 对比 1000 字描述更有效
5. **按领域组织参考文件：**

   如 aws.md / gcp.md 分开，只加载需要的
6. **重复工作脚本化：**

   如果多个测试都独立写了类似脚本，就打包到 scripts/
7. **断言要有区分力：**

   不能 with/without skill 都通过（Grader + Analyzer 都会识别）
8. **泛化改进：**

   从反馈泛化，不要针对特定测试过拟合
9. **保持精简：**

   删除不发挥作用的内容
10. **阅读执行记录：**

    不仅看输出，还要看模型做了什么无用功
11. **计时数据立即捕获：**

    来自任务通知，错过就没了
12. **先人工审核再改进：**

    不要跳过 Viewer 环节

### 13.2 改进策略的思维模型

来源：SKILL.md "Improving the skill" 章节

```
用户反馈 -> 理解真实需求（不仅是字面意思）
              |
          泛化为规则（对百万次使用有效）
              |
          尝试新方法（不同隐喻、不同工作模式）
              |
          保持精简（删除无用部分）
              |
          解释推理（让模型理解 Why）
```

> SKILL.md: "This task is pretty important (we are trying to create billions a year in economic value here!) and your thinking time is not the blocker; take your time and really mull things over."

### 13.3 通信风格适配

来源：SKILL.md "Communication" 章节

Skill-Creator 特别关注与不同技术背景用户的沟通：

| 术语 | 使用条件 |
| --- | --- |
| "evaluation", "benchmark" | 可以直接使用（borderline, but OK） |
| "JSON", "assertion" | 需要看到用户有技术背景的线索 |
| 技术术语 | 可以简短解释后使用 |

## 14. 已识别的实现边界与风险

> 以下结论均经源码验证，标注了具体代码位置。

### 14.1 触发检测的保守策略

`run_eval.py``run_single_query` 函数中的 `return False` 意味着：如果 Claude 的**第一个**工具调用不是 Skill 或 Read，立即判定未触发。这是有意的设计决策（如果 Claude 直接用其他工具处理说明不需要 Skill），但在极端场景下可能存在假阴性。

### 14.2 查询唯一性假设

`run_eval.py` 在聚合结果时用 query 字符串做 key（`run_eval` 函数），默认假设 eval query 唯一。如果评估集中有重复 query（不同 should\_trigger），会导致结果覆盖。

### 14.3 Benchmark Delta 顺序依赖

`aggregate_benchmark.py``aggregate_results` 函数：delta 基于前两个发现的配置目录的字典序。SKILL.md 通过约定 "Put each with\_skill version before its baseline counterpart" 来解决，但没有代码强制保障——如果目录命名不规范，delta 正负号可能反转。

### 14.4 统计精度有限

`aggregate_benchmark.py` 使用均值和样本标准差，没有做显著性检验（如 t-test、bootstrap CI）。对于严格的 A/B 决策，建议在此基础上补充统计检验。

### 14.5 安全模型

官方安全模型基于 "Principle of Lack of Surprise" 原则：Skill 不得包含恶意软件、漏洞利用代码或危害系统安全的内容，实际行为不应超出描述范围。在 Claude Code 本地场景下，脚本在用户本机环境中执行，无额外沙箱隔离。

## 15. 创建自定义 Skill 的实操指南

### 15.1 从零开始的流程

```
Phase 1: 意图与调研
+-- 1.1 明确 Skill 要做什么
+-- 1.2 确定触发场景
+-- 1.3 定义输出格式
+-- 1.4 调研依赖和 MCP
+-- 1.5 询问边界情况

Phase 2: 编写 Skill
+-- 2.1 创建目录结构
+-- 2.2 编写 SKILL.md frontmatter (name + description)
+-- 2.3 编写核心指令
+-- 2.4 添加示例
+-- 2.5 创建 scripts/ (如有确定性任务)
+-- 2.6 创建 references/ (如有大量参考)

Phase 3: 测试与评估
+-- 3.1 创建 2-3 个测试用例
+-- 3.2 并行运行 with/without skill（同一 turn）
+-- 3.3 起草断言（利用等待时间）
+-- 3.4 评分 + 聚合 + 查看器
+-- 3.5 收集用户反馈

Phase 4: 迭代改进
+-- 4.1 基于反馈泛化改进（不要过拟合）
+-- 4.2 重新运行测试
+-- 4.3 审核新结果
+-- 4.4 重复直到满意

Phase 5: 优化与发布
+-- 5.1 描述优化（20 条查询集）
+-- 5.2 运行 run_loop.py
+-- 5.3 应用最佳描述
+-- 5.4 验证打包
+-- 5.5 发布 .skill 文件
```

### 15.2 更新已有 Skill 的注意事项

当用户要求更新而非创建 Skill 时：

1. **保留原始 name：**

   使用目录名和 frontmatter `name` 字段原值不变（如原名 `research-helper，`输出 `research-helper.skill，`不要加 `-v2` 后缀）
2. **复制到可写目录再编辑：**

   已安装的 Skill 路径可能是只读的，先 `cp -r <skill-path> /tmp/skill-name/，`在副本上编辑
3. **打包时先暂存到 `/tmp/：`**

   直接写入输出目录可能因权限失败，先打包到 `/tmp/` 再复制到目标位置
4. **改进模式的基线处理：**

   编辑前快照旧版本（`cp -r <skill-path> <workspace>/skill-snapshot/`），将基线 subagent 指向快照，结果保存到 `old_skill/outputs/`

### 15.3 SKILL.md 模板

```
---
name:my-skill-name
description:>
  [简述做什么]. Use when [具体场景1], [场景2], [场景3].
  Also use when [不明显的场景]. Trigger even if user doesn't
  explicitly mention [关键词].
---

# [Skill 名称]

[1-2句概述核心价值和工作流]

## 核心工作流

### Step 1: [步骤名]
[具体指令，解释why]

### Step 2: [步骤名]
[具体指令，解释why]

## 输出格式

[使用模板展示期望输出]

## 示例

**Example1:**
Input: [输入]
Output: [输出]

## 边界情况

[常见问题和处理方式]

## 参考文件

-`scripts/xxx.py`-- [何时使用]
-`references/xxx.md`-- [何时读取]
```

### 15.4 测试用例模板

> **流程提醒：**推荐先只写 prompt 启动运行，利用等待时间再补充 expectations（SKILL.md: "Don't write assertions yet -- just the prompts"）。下面模板展示的是最终完整格式。
>
> **术语注意：**官方源码中 `assertions` 和 `expectations` 并存于不同 JSON 契约中——`eval_metadata.json` 使用 `assertions，``evals.json` 和 `grading.json` 使用 `expectations。`详见 §10.2 的完整说明。

```
{
"skill_name":"my-skill",
"evals":[
{
"id":1,
"prompt":"真实用户会说的话，有具体细节和上下文",
"expected_output":"期望结果的人类可读描述",
"files":[],
"expectations":[
"输出包含 X 格式的 Y 数据",
"使用了 scripts/helper.py 处理数据",
"最终文件大小在合理范围内"
]
}
]
}
```

## 16. 企业落地建议

> **注意：**本节为基于 skill-creator 设计思想的推导建议，非源码直接分析。建议按实际业务情况取舍。

### 16.1 组织分工（建议最小角色）

* **Skill Owner（业务）：**

  定义任务边界与验收标准
* **Skill Engineer（平台）：**

  维护模板、脚本、评测管线
* **Evaluator（业务+质控）：**

  做人审与反馈打标
* **Governance（安全）：**

  做脚本与外部依赖审计

### 16.2 目录规范（建议）

* 每个业务 Skill 强制包含：

+ `SKILL.md`
+ `evals/evals.json`
+ `references/`

  （业务规则、字段口径）
+ `scripts/`

  （可验证、可复现操作）

* 每次迭代保存：

+ `iteration-N/*`

  （outputs、grading、timing、benchmark、feedback）

### 16.3 指标体系（建议上线门槛）

* 质量指标：`pass_rate(with_skill) - pass_rate(baseline) >= +X`（X 由业务定义）
* 触发指标：should-trigger 召回 + should-not-trigger 精度
* 成本指标：time/token 增幅在可接受范围
* 人审指标：关键 eval 无负反馈或问题闭环完成

### 16.4 落地路线图（建议 3 阶段）

**Phase 1（1-2 周）：试点**

* 选 1-2 个高频、可验证业务流程
* 跑通 `draft -> eval -> viewer -> improve -> package` 完整闭环
* 建立对整套工具链的团队理解

**Phase 2（3-6 周）：工程化**

* 统一 Skill 模板、评测模板、评分模板
* 接入版本管理与发布门禁
* 加入描述触发回归测试（run\_eval/run\_loop 思路）

**Phase 3（持续）：规模化**

* 扩展到多业务线
* 定期回归（模型升级后自动跑 benchmark）
* 建立"过时 Skill 下线"机制（尤其 capability uplift 类）

### 16.5 企业复用 run\_eval.py 时的注意事项

1. 确保移除 `CLAUDECODE` 环境变量（已内置于脚本中，但需理解原因）
2. 评估集中避免重复 query
3. 统一 with\_skill 目录名排在 baseline 前
4. 考虑补充统计显著性检验

## 17. 附录

### 17.1 关键脚本命令速查

```
# 验证 Skill
python -m scripts.quick_validate <skill-path>

# 运行触发评估
python -m scripts.run_eval \
  --eval-set <eval-set.json> \
  --skill-path <skill-path>

# 完整优化循环
python -m scripts.run_loop \
  --eval-set <eval-set.json> \
  --skill-path <skill-path> \
  --model <model-id> \
  --max-iterations 5

# 聚合基准
python -m scripts.aggregate_benchmark \
  <workspace>/iteration-N \
  --skill-name <name>

# 启动审核查看器
python eval-viewer/generate_review.py \
  <workspace>/iteration-N \
  --skill-name "my-skill" \
  --benchmark <workspace>/iteration-N/benchmark.json

# 启动审核查看器（静态模式，用于 Cowork/无 GUI 环境）
python eval-viewer/generate_review.py \
  <workspace>/iteration-N \
  --skill-name "my-skill" \
  --static <output.html>

# 打包 Skill
python -m scripts.package_skill <skill-path>
```

### 17.2 核心概念术语表

| 术语 | 说明 |
| --- | --- |
| **Skill** | 一组 Markdown 指令 + 可选资源，指导 Claude 完成特定任务 |
| **SKILL.md** | Skill 的核心定义文件，包含 YAML frontmatter 和 Markdown 指令 |
| **Frontmatter** | SKILL.md 开头的 YAML 元数据（name, description 等） |
| **Progressive Disclosure** | 三层加载系统：元数据(永驻) -> 正文(触发时) -> 资源(按需) |
| **Eval** | 一个测试用例，包含 prompt 和 expectations |
| **Assertion/Expectation** | 对输出的可验证声明 |
| **Benchmark** | 多次运行的聚合统计数据 |
| **Blind Comparison** | 不知道来源的 A/B 比较 |
| **Trigger Rate** | Skill 在给定查询上被触发的概率 |
| **Train/Test Split** | 将评估集分为训练和测试，防止过拟合（默认 60/40） |
| **Blinded History** | 改进模型看不到测试集分数的机制，防止对测试集的隐式优化 |
| **Capability Uplift** | 让模型做到原本不稳定/做不好的事的 Skill |
| **Encoded Preference** | 按组织流程编排模型已有能力的 Skill |

### 17.3 共享工具函数

`scripts/utils.py` 提供 `parse_skill_md(skill_path)` 函数，是整个脚本系统的共享 SKILL.md 解析器，返回 `(name, description, content)` 三元组。被 `run_eval.py、``run_loop.py、``improve_description.py` 共同依赖。

> **注意**：`skill_path` 参数是 **Skill 目录路径**（如 `./my-skill/`），不是 SKILL.md 文件路径。函数内部会自动拼接 `skill_path / "SKILL.md"` 读取。

> **文档维护说明：**本文档基于 Claude 官方 skill-creator 源码（485 行 SKILL.md，2026-03-07 版本）和 Anthropic 官方博客分析。如官方 skill-creator 有重大更新，请相应更新本文档。关键验证点已标注函数名/章节名。