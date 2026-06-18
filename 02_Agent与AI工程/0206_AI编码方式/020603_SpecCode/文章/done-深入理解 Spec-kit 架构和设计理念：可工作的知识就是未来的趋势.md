> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020603_SpecCode/020603_核心知识点/SpecCode规格驱动与验收边界|SpecCode规格驱动与验收边界]]
---
title: 深入理解 Spec-kit 架构和设计理念：可工作的知识就是未来的趋势
author: Asher同学
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk3NTcyMzA0Nw==&mid=2247484287&idx=1&sn=15422b8efe3b45c86039d7084e482b7f&chksm=c5599515e7e1fe4566ec549bff205081f18856bb59bc0c6ea54aaf78e8cc64029d0ccfeb6740&mpshare=1&scene=24&srcid=1015AHQz6joJQRy6QaUxdyDc&sharer_shareinfo=d5a5acb8b94f562bdf7ba969aff48ec9&sharer_shareinfo_first=d5a5acb8b94f562bdf7ba969aff48ec9#rd
---

做学问和研究要耐得住寂寞，在评论和反对一个东西之前我们不妨先把官方的所有英文文档都通读一遍，努力理解背后的思想。本着没有调查就没有发言权的原则，在评论Spec-kit 好不好用之前，我先闭上我的嘴，等一段时间深入实践研究之后，再和评论区的同学一起切磋。

我之前也写过一篇文章介绍多智能体， [Claude Code 多智能体协作系列一：群蜂模式](https://mp.weixin.qq.com/s?__biz=Mzk3NTcyMzA0Nw==&mid=2247484032&idx=1&sn=0c8e0c29e1f52f506b6047829bc73800&scene=21#wechat_redirect)， 后台有的人说把我 token 烧完了！这玩意没用。有用没用因人而异，因为每个人的标准不一样，开源项目是用脚投票的，别人提出一个理念并且实现了本身就是一个很了不起的事情，一个人如果一直保持一个学习者的心态，空怀若谷，静水流深，总是会吸收到一些营养的。

回到 spec-kit 这个开源项目，首先从本质上来看，这个项目本身就不是一个传统的软件，其本质就是提示词（Prompt）。SDD 想法的落地强烈依赖于各种各样的强大的 Agent 智能体，比如 Claude Code ， Codex 等等。所以，这个项目的价值在于提示词，而仔细研究里面的模板文件和流程设计的话，你会发现它基本把敏捷文化的精髓都展示出来了。原滋原味一手资料请阅读：https://github.com/github/spec-kit/blob/main/spec-driven.md

去年在学习徐昊老师的课程时，注意到一段话，让我热血澎湃：

Asher同学在这里乐观的断言一下：随着大模型的能力的飞跃， 可工作的知识毫无疑问就是趋势，AI 时代就是最好的时代，一个知识繁荣的时代。（如果 GLM4.6 不行，你换 Claude Opus 等世界一流的模型再试试）

## 技术系列问题思考

> 1. 1. 这个技术出现的背景、初衷和要达到什么样的目标或是要解决什么样的问题。
> 2. 2. 这个技术的优势和劣势分别是什么，或者说，这个技术的 trade-off 是什么。
> 3. 3. 这个技术适用的场景。
> 4. 4. 技术的组成部分和关键点。
> 5. 5. 技术的底层原理和关键实现。

### 这个技术出现的背景、初衷和要达到什么样的目标或是要解决什么样的问题。

背景：Agent 智能水平已经足够高，编程能力平替中低级程序员。

初衷和目标：

Spec-Driven Development（规格驱动开发，简称 SDD）尝试颠覆传统的软件开发结构：不是规格为代码服务，而是代码为规格服务。产品需求文档（PRD）不再只是指导实现的说明书，而是**生成实现的源头**。技术方案也不再只是用于指导开发的文档，而是**直接定义并产出代码的精确定义**。

### 这个技术用来解决什么问题？

1. 1. 通过自动化代码翻译（规格到代码）来增强人类的能力
2. 2. 构建一个规格、研究和代码共同发展的反馈循环，每次迭代都能带来更深入的理解，并在意图和实现之间实现更好的协调

### 这个技术不能解决什么问题？

1. 1. 不是取代程序员
2. 2. 不是自动化创造力

### 这个技术适用的场景

Agent 智能水平已经足够高,可以完成一些复杂的调研和编码任务。

### 技术的组成部分和关键点

1. 1. 宪法文件（.specify/memory/constitution.md）

   定义项目的架构约束
2. 2. 知识模板（.specify/templates/）

* • 防止过早涉及实现细节
* • 分层级细节管理
* • 强制使用明确的不确定性标记
* • 通过清单实现结构化思考
* • 通过审查机制确保架构合规 （门控机制，防止架构腐败）
* • 测试优先思维保证代码的可测试性
* • 防止臆想性功能泛滥

3. 3. 可工作的知识（.claude/commands/）

* • 规范就是可执行的资产（specifications as executable artifacts）

### 技术的底层原理

SDD（Spec-Driven Development Methodology） 方法论：https://github.com/github/spec-kit/blob/main/spec-driven.md

## 项目初始文件骨架

## 项目核心流程概览

核心是 5个流程：

1. 1. constitution 定义宪法文件（架构约束和项目规约）
2. 2. specify （根据需求生成用户故事）
3. 3. plan （根据用户故事生成技术方案）
4. 4. tasks （用户故事拆分为可执行的任务）
5. 5. implement（可执行的任务变成可工作的代码）

辅助的3个流程：

1. 1. clariy （澄清需求描述中模糊的地方）
2. 2. analyze （检查生成的用户故事，技术方案以任务是否满足架构约束和项目规范）
3. 3. checklist （检查用户故事是否有遗漏的功能点）

## 项目核心模板文件

### 1，Spec 模板

核心概念：

* • User Story（用户故事） + 验收条件（Acceptance Criteria）
* • **支撑用户故事的**系统功能  Functional Requirements + 系统验收标准 （Success Criteria）

关注点：

1. 1. 清楚地表达业务需求
2. 2. 需求有可衡量的验收标准

### 2， Plan 模板

核心概念：

* • 技术上下文（Technical Context）
* • 宪法检查（Constitution Check）
* • 项目结构 （Project Structure）
* • 复杂度追踪 （Complexity Tracking）

关注点：

1. 1. 技术实现背景（技术上下文 + 项目结构）
2. 2. 技术要服务业务（宪法检查）
3. 3. 技术实现要简单（复杂度追踪）

### 3， task 模板

核心概念：

* • 任务分阶段结构（项目初始化 ， 核心技术底座， 实现第 1 个用户故事，实现第 2 个用户故事, .... , 文档更新，验证）
* • 每个用户故事固定的流程

+ • 0 先写测试（验收标准，对应TDD的红）
+ • 1 建模 （model）
+ • 2 服务（service）
+ • 3 API
+ • 4 验收测试

* • 独立的用户故事并行开发（Parallel Team Strategy）

关注点：

1. 1. 任务要和用户故事关联
2. 2. 每个用户故事的任务尽可能独立
3. 3. 独立的用户故事的任务并行执行

### 4， CheckList 模板

核心概念：

* • 检查清单

关注点：

* • 生成的任务清单 tasks.md 一方面要服务用户故事，另外一方面要符合架构约束和项目规范

## 可工作的知识(斜杠命令)

### 1， 定规约 `/speckit.constitution`

**输入**:

* • `$ARGUMENTS`（原则或修订内容）
* • 读取文件：`.specify/memory/constitution.md` 模板及相关 `.specify/templates/*` 文件

**执行过程**:

1. 1. 解析现有宪法模板并定位 `[PLACEHOLDER]`
2. 2. 根据输入与上下文填充值，决定版本号增量
3. 3. 草拟更新内容并确保无未解释的占位符
4. 4. 逐一核对计划、规范、任务模板及命令/运行文档的宪法依赖
5. 5. 生成同步影响报告（HTML 注释）
6. 6. 校验格式、日期、措辞严谨度
7. 7. 写回 `.specify/memory/constitution.md` 并报告变更

**输出**:

* • 更新后的宪法文件
* • 影响报告与后续行动/提交建议

### 2， 需求细化和分解 `/speckit.specify`

**输入**:

* • `$ARGUMENTS`（功能描述）
* • 读取文件：`.specify/templates/spec-template.md`

**执行过程**:

1. 1. 运行 `.specify/scripts/bash/create-new-feature.sh --json "$ARGUMENTS"` 并获取 `BRANCH_NAME`, `SPEC_FILE`
2. 2. 解析描述，提炼角色、动作、数据、约束
3. 3. 在合理假设下完成规范，最多留下3个 `[NEEDS CLARIFICATION]`
4. 4. 填充用户场景、功能/非功能需求、成功标准、关键实体
5. 5. 写入 `spec.md` 并生成 `checklists/requirements.md`
6. 6. 进行最多3轮质量校验和自动修订；若仍未通过，记录原因并提醒用户
7. 7. 输出分支、文件路径与下一步建议

**输出**:

* • `SPEC_FILE`
* • `checklists/requirements.md`
* • 质量校验结果与推荐后续动作

### 3，需求澄清，`/speckit.clarify`

**输入**:

* • `$ARGUMENTS`
* • 环境变量：`FEATURE_DIR`, `FEATURE_SPEC`
* • 读取文件：当前 `spec.md`

**执行过程**:

1. 1. 运行 `.specify/scripts/bash/check-prerequisites.sh --json --paths-only`
2. 2. 扫描10大分类，评估 Clear/Partial/Missing
3. 3. 组建按影响排序的候选问题队列（≤5）
4. 4. 逐题交互，提供推荐答案或建议值，等待用户确认/调整
5. 5. 将确认答案写入 `## Clarifications` 与相关章节，确保结构与术语一致
6. 6. 每次写入后检查格式与残留模糊项
7. 7. 保存 `spec.md` 并在会话末输出覆盖表与建议下一步

**输出**:

* • 更新后的 `spec.md`
* • 澄清会话摘要与分类状态表

### 4， 技术方案规划 `/speckit.plan`

**输入**:

* • `$ARGUMENTS`
* • 环境变量：`FEATURE_SPEC`, `IMPL_PLAN`, `SPECS_DIR`, `BRANCH`
* • 读取文件：`FEATURE_SPEC`, `.specify/memory/constitution.md`, 计划模板

**执行过程**:

1. 1. 运行 `.specify/scripts/bash/setup-plan.sh --json`
2. 2. 加载规范、宪法与模板，初始化技术上下文
3. 3. 填写模板：技术栈、宪法检查、门控
4. 4. Phase 0：派发研究任务，产出 `research.md`
5. 5. Phase 1：生成 `data-model.md`、`contracts/`、`quickstart.md`，并更新代理上下文脚本
6. 6. 重新执行宪法门控校验并总结输出

**输出**:

* • `research.md`, `data-model.md`, `contracts/*`, `quickstart.md`
* • 更新的代理上下文文件与计划执行摘要

### 5， 用户故事任务分解 `/speckit.tasks`

 

**输入**:

* • `$ARGUMENTS`
* • 环境变量：`FEATURE_DIR`, `AVAILABLE_DOCS`
* • 读取文件：`plan.md`, `spec.md`, `data-model.md`, `contracts/`, `research.md`, `quickstart.md`

**执行过程**:

1. 1. 运行 `.specify/scripts/bash/check-prerequisites.sh --json`
2. 2. 汇总设计工件，抓取用户故事优先级与技术约束
3. 3. 依据模板生成阶段化任务：

* • Phase 1：共享 Setup
* • Phase 2：Foundational 阻塞项
* • Phase 3+：按优先级的用户故事（含 `[P]` 标记、依赖和独立验收标准）
* • 尾声：Polish & Cross-Cutting

4. 4. 根据需求决定是否生成测试任务，保持编号 `T001` 起递增
5. 5. 校验每个故事的自洽性与独立可测试性
6. 6. 输出 `tasks.md` 并汇总任务数、并行机会、建议 MVP 范围

**输出**:

* • `tasks.md`
* • 任务统计、并行建议、MVP 推荐

 

### 6， 用户需求检查`/speckit.checklist`

**输入**:

* • `$ARGUMENTS`（用户提供的关注点）
* • 环境变量：`FEATURE_DIR`, `AVAILABLE_DOCS`
* • 读取文件：`spec.md`, `plan.md`, `tasks.md`（若存在）

**执行过程**:

1. 1. 运行 `.specify/scripts/bash/check-prerequisites.sh --json`
2. 2. 基于上下文生成最多3个初始澄清问题，按需再追加≤2个高影响问题
3. 3. 解析用户意图，确定检查单主题、范围与必选项
4. 4. 定向加载规范/计划/任务中的相关片段
5. 5. 依据质量维度生成检查项，确保 ≥80% 项带追踪标记（如 `[Spec §FR-2]`、`[Gap]`）
6. 6. 遵循 `.specify/templates/checklist-template.md` 结构输出
7. 7. 报告文件路径、项目统计、深度等级与焦点

**输出**:

* • `FEATURE_DIR/checklists/<type>.md`
* • 运行摘要与后续建议

### 7， `/speckit.analyze`

**输入**:

* • `$ARGUMENTS`（可选）
* • 环境变量：`FEATURE_DIR`, `AVAILABLE_DOCS`
* • 读取文件：`spec.md`, `plan.md`, `tasks.md`, `.specify/memory/constitution.md`

**执行过程**:

1. 1. 运行 `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks`
2. 2. 渐进式加载关键章节
3. 3. 构建需求、用户故事、任务覆盖及宪法规则模型
4. 4. 执行重复、模糊、欠缺、宪法对齐、覆盖、矛盾六大检测
5. 5. 按严重性分级并限制发现总数≤50
6. 6. 输出表格化报告与覆盖统计
7. 7. 给出下一步建议并询问是否需要补救方案

**输出**:

* • Markdown 分析报告（只读）
* • 覆盖率与宪法对齐摘要
* • 建议的后续命令或人工修复步骤

### 8. 任务实现`/speckit.implement`

**输入**:

* • `$ARGUMENTS`
* • 环境变量：`FEATURE_DIR`, `AVAILABLE_DOCS`
* • 读取文件：`tasks.md`, `plan.md`, `data-model.md`, `contracts/`, `research.md`, `quickstart.md`

**执行过程**:

1. 1. 运行 `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks`
2. 2. 汇总 `checklists/` 状态，必要时等待用户确认继续
3. 3. 加载实现上下文与依赖文档
4. 4. 自动创建/补充忽略文件（Git/Docker/ESLint/Prettier/Terraform等）
5. 5. 解析任务阶段、依赖、文件路径与 `[P]` 标记
6. 6. 按阶段与依赖顺序执行任务，必要时先写测试
7. 7. 每完成任务即在 `tasks.md` 标记 `[X]` 并报告进度
8. 8. 收尾校验实现是否覆盖规范和计划要求

**输出**:

* • 修改后的项目代码与文档
* • 进度与完成状态报告

## 内容小结

Spec-Kit 所倡导的规格驱动开发，其核心思想在于一场深刻的范式转移：将需求规格本身置于开发流程的中心，使其从静态的指导文档转变为可执行的、能直接驱动智能体生成代码的“源头活水”。这套方法论并非旨在取代开发者，而是通过一套精密的流程设计——从定义架构约束的“宪法”，到层层递进的需求细化、技术规划和任务分解——将人类的架构智慧与智能体的执行能力深度融合。它本质上是一套封装了敏捷精髓与工程最佳实践的“超级提示词”，其巨大潜力在于构建了一个从意图到实现的强化反馈循环。尽管其成功高度依赖于大模型的能力且面临工程落地的挑战，但它无疑为我们勾勒出了一幅AI时代下，知识如何被精确封装并直接转化为生产力的激动人心的未来图景。