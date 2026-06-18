> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor Rules与Plan工作流边界|Cursor Rules与Plan工作流边界]]、[[02_Agent与AI工程/0205_AI编程工具/020503_Cursor/020503_核心知识点/Cursor工程使用与上下文边界|Cursor工程使用与上下文边界]]
---
title: AI 编程工具 Cursor Plan 模式保姆级教程
author: 前端AI行走
date:
url: https://mp.weixin.qq.com/s?__biz=MzU5MjYwMDgzNQ==&mid=2247487384&idx=1&sn=4d53d1865392bc41a233ff506568030f&chksm=ff6a3eb039bcb49da2c432559212ea58474d9475b26720d98ef50972c2313c67d758269b8ff2&mpshare=1&scene=24&srcid=12094tftLvRYp6CPcOBZCL8p&sharer_shareinfo=61c459fb7621e997005c7d5d47028593&sharer_shareinfo_first=61c459fb7621e997005c7d5d47028593#rd
---

# 前言

#

# 在日常使用 Cursor 编程工具的使用中，一般我很少使用到Plan模式，大多数都是Agent , Ask 的模式。我一直想试一下这个工具的能力，或者想测试一下这个功能是否靠谱，是否能够按照我的要求，给我我想要的结果。后来需求来了，这块需求的项目代码我没有了解过，页面改动的细节蛮低的，我不想去看代码，不想找代码。突然想到了可以使用Plan 模式让AI 给找出来，于是我就根据需求写了一个Cursor Plan来进行需求开发。

#

#

## 一、什么是 Cursor Plan 模式？

##

Cursor Plan 模式是 Cursor AI 提供的一种**结构化任务管理方式**，它遵循“先规划、再执行”的工作流程。

### 核心特点

###

1. **分离规划与执行：**

   在 Plan 模式下，AI 会先制定详细计划，等待你确认后再开始执行代码修改
2. **任务追踪:**

   通过`todos`列表，可以清晰地看到每个任务的进度（pending → in\_progress → completed）
3. **降低风险:**

   复杂需求可以先看方案，确认无误后再执行，避免大范围改动出错
4. **可追溯性:**

   所有改动都有明确的计划和步骤，便于后续维护和优化

### 适用场景

###

* ✅ 需求复杂，涉及多个文件修改
* ✅ 需要隐藏/替换大量功能模块
* ✅ 用户明确要求“先出方案再动手”
* ✅ 存在多种实现方案，需要先选择
* ✅ 需求信息不完整，需要先澄清

---

##

##

## 二、实际案例：板块信息需求改造

##

### 案例背景：

###

以 `cursor-plan.md` 中的需求为例，这是一个典型的 Plan 模式使用场景。

### 需求概述：

###

**目标**：在板块信息模块中，通过注释方式隐藏部分功能与数据展示，不删除代码。

**涉及范围**：

* 搜索页：隐藏团险、费率查询选项，隐藏账号类型、关联账号、昵称列
* 用户详情页：隐藏昵称/身份/账号余额字段，隐藏协作业务、工具&活动 tab，隐藏服务记录板块
* 订单详情页：隐藏客户服务人、奖励券、券类型列，隐藏关联人员、订单管理 tab
* 订单信息页：隐藏产品信息入口，隐藏基本信息、连续投保信息、代扣信息板块

**关键约束**：只做代码注释操作，不做代码删除操作。

---

##

##

## 三、Plan 模式工具详解

##

### 重要说明：工具的本质

###

`ask_question`、 `create_plan`、 `todo_write` 等是我自己写教程可以方便描述的名称，实际映射的就是 **Cursor AI 的内部工具**，不是用户直接调用的命令。代码 code JOSN 信息的展示与相关引用，只是类比流程与信息，只是方便理解工具可以做什么而已。

**关键理解**：

* ❌错误理解：用户在输入框输入`ask_question("问题")` 或 `@ask_question问题`
* ✅ **正确理解**：用户用自然语言描述需求，AI 会自动判断并调用相应工具

### 用户如何使用？

###

**用户只需要做一件事**：用自然语言与 AI 对话，描述你的需求。

**示例对话**：

> 用户:我想在板块信息模块中隐藏一些字段，先帮我制定一个计划
>
> AI: [自动调用 create\_plan 工具，创建计划并展示给你]
>
> 用户: 可以开始执行;
>
> AI: [自动调用 todo\_write 更新任务状态，开始执行代码修改]

```

```

**如果需求不明确**：

```

```

> 用户: 我需要替换订单&售后功能，但不确定接口是否准备好了
>
> AI: [自动调用 ask\_question 工具，向你提问]
>
>     请问订单&售后和线下收益是否有接口，还是先使用 mock 数据？
>
> 用户: 先使用 mock 数据
>
> AI: [根据你的回答，调用 create\_plan 创建包含 mock 数据的计划]

###

### 工具详细说明：

###

#### 1. `ask_question` - 澄清需求工具

####

**作用**：AI 在创建 Plan 之前，如果发现需求不明确，会自动使用此工具向你提问。

**使用场景**：

* 需求信息不完整
* 存在多种实现方案需要选择
* 需要确认技术细节（如接口、数据源等）

**用户操作**：

* 不需要任何特殊操作
* AI 会自动提问，你只需正常回答

**示例**：

```
> AI: [自动调用 ask_question]
>
>
>
>     订单&售后和线下收益是否有接口，还是先使用 mock 数据？
>
>
>
> 用户: 先使用 mock 数据
>
>
>
> AI: [继续创建 Plan，包含 mock 数据的设计]
```

####

#### 2. `create_plan` - 创建计划工具

####

**作用**：AI 根据你的需求，自动创建结构化的执行计划。

**参数结构**类比如下：

```

```

```
{  "name": "plan-名称",           // 计划的唯一标识  "overview": "一句话总结",       // 计划概述  "steps": [                     // 详细步骤列表    "Step 1 – 具体操作描述",    "Step 2 – 具体操作描述"  ],  "todos": [                     // 任务列表    {       "id": "todo-1",       "content": "任务描述",      "status": "pending"        // pending/in_progress/completed    }  ]}
```

**用户操作**：

* 描述需求后，AI 会自动创建 Plan
* 查看 Plan 的 `overview` 和 `steps`，确认是否符合预期
* 如果符合，回复"可以开始执行"或"批准这个 Plan"
* 如果不符合，告诉 AI 需要调整的地方

**示例**：

```

```

> 用户: 我想隐藏信息模块中的某些字段;
>
> AI: [自动调用 create\_plan，创建计划]
>
>     我已经创建了 Plan，包含以下步骤：
>
>     - Step 1: 搜索页调整
>
>     - Step 2: 用户详情裁剪
>
>     - Step 3: 订单详情调整
>
>     ...
>
>     
>
>     请确认是否可以开始执行？
>
> 用户: 可以开始执行
>
> AI: [开始执行 Plan 中的任务]

####

####

#### 3. `todo_write` - 任务管理工具

####

**作用**：AI 在执行过程中，使用此工具管理任务状态和添加新任务。

**主要用途**：

1. **更新任务状态:**

   `pending` → `in_progress` → `completed`
2. **添加新任务:**

   执行过程中发现遗漏，动态添加任务

**参数结构类比如下：**

**更新任务状态**：

```
{  "merge": true,                    // 合并到现有 todos  "todos": [    {      "id": "todo-1",      "status": "in_progress"       // 或 "completed"    }  ]}
```

```

```

**添加新任务**：

```
{  "merge": true,                    // 必须为 true，否则会替换所有 todos  "todos": [    {      "id": "todo-new",      "content": "新任务描述",      "status": "pending"    }  ]}
```

```

```

**用户操作**：

* 不需要直接调用，AI 会自动使用
* 可以查看 todos 列表了解当前进度
* 如果需要中途添加任务，直接告诉 AI："还需要修复表格滚动条问题"

**示例**：

```
AI: [自动调用 todo_write]    todo_write({      "merge": true,      "todos": [        { "id": "todo-search", "status": "in_progress" }      ]    })
    开始执行 todo-search...[执行代码修改...]AI: [自动调用 todo_write]    todo_write({      "merge": true,      "todos": [        { "id": "todo-search", "status": "completed" }      ]    })
    todo-search 已完成！
```

**中途添加任务**：

```

```

```
用户: "表格有滚动条，希望全量展示"AI: [自动调用 todo_write 添加新任务]    todo_write({      "merge": true,      "todos": [        {           "id": "todo-fix-table-width",           "content": "调整表格宽度，确保全量展示无滚动条",           "status": "pending"         }      ]    })
    已添加新任务：todo-fix-table-width
```

###

### 工具使用流程图

```

```

###

```
用户描述需求    ↓AI 判断需求是否明确    ↓[不明确] → ask_question → 用户回答 → 继续    ↓[明确]    ↓AI 调用 create_plan 创建计划    ↓用户确认 Plan    ↓AI 开始执行，调用 todo_write 更新状态    ↓执行过程中发现遗漏    ↓AI 调用 todo_write 添加新任务    ↓所有任务完成
```

###

###

### 常见问题

###

**Q1: 我需要在输入框输入工具调用代码吗？**

* 不需要。直接用自然语言描述需求即可。

**Q2: 如何让 AI 使用某个工具？**

* 不需要主动调用。AI 会根据上下文自动判断是否需要使用工具。
* 例如：说"先帮我制定计划"，AI 会自动调用 `create_plan`

**Q3: 我可以直接看到工具调用吗？**

* 在 Cursor 的对话界面中，AI 的工具调用通常是透明的
* 你主要看到的是 AI 的回复和 Plan 内容，而不是工具调用的代码

**Q4: 如何知道当前任务进度？**

* AI 会在对话中展示 todos 列表和当前状态
* 你可以随时查看 Plan 中的 todos 了解进度

---

##

##

## 四、如何编写 Plan

##

### Step 1: 澄清需求（可选但推荐）

###

在创建 Plan 之前，如果需求有疑问，AI 会自动使用 `ask_question` 工具向你提问。

**示例问题**：

* "订单&售后和线下收益是否有接口，还是先使用 mock 数据？"
* "隐藏功能是否需要保留代码以便后续恢复？"

### Step 2: 创建 Plan 的结构

###

使用 `create_plan` 工具，需要包含以下核心部分：、

```
{  "name": "plan-名称",  "overview": "一句话总结目标和关键动作",  "steps": [    "Step 1 – 具体步骤描述（包含文件路径和操作）",    "Step 2 – 具体步骤描述",    "..."  ],  "todos": [    { "id": "todo-1", "content": "任务描述" },    { "id": "todo-2", "content": "任务描述" }  ]}
```

### Step 3: 实际案例 - cursor-plan.md 的 Plan 示例

###

基于 `cursor-plan.md` 的需求，一个完整的 Plan 应该是这样的：

```

```

```
{  "name": "legal-affairs-hide-fields",  "overview": "通过注释方式隐藏信息模块中指定的功能与字段，确保界面仅展示允许的内容，不删除任何代码。",  "steps": [    "Step 1 – 搜索页调整：在 search-home.vue 中注释团险、费率查询单选项；个险条件下注释订单来源、代理人手机号、支付单号输入框；列表中注释账号类型、关联账号、昵称列。",    "Step 2 – 用户详情裁剪：在 user-detail.vue 中注释 userInfo 中昵称/身份/账号余额字段；tab 区域注释协作业务、工具&活动；右侧服务记录模块整体注释；账号信息→快捷导航仅保留投诉记录；资产收益→资产总览模块，快捷导航仅保留佣金&奖励、提现&税，并在佣金&奖励表中注释预计可提现时间与打款信息。",    "Step 3 – 订单详情调整：在 components/order/order-detail-header.vue 表格中注释客户服务人、奖励券、券类型列；在 components/order-detail.vue 中注释关联人员、订单管理 tabs；右侧订单记录模块整体注释。",    "Step 4 – 订单信息模块精简：在 components/order/order-info.vue 中快捷导航隐藏产品信息入口；下方产品信息、订单信息板块中的基本信息、连续投保信息、代扣信息用注释隐藏；险种信息表格中注释退款方式、投保份数、每月领取列。",    "Step 5 – 校验与测试：使用 read_lints 检查所有修改文件，确保注释语法正确，无语法错误。"  ],  "todos": [    { "id": "todo-search", "content": "注释搜索页多余选项与列（search-home.vue）" },    { "id": "todo-user-detail", "content": "裁剪 user-detail.vue 指定内容（userInfo字段、tabs、服务记录、快捷导航等）" },    { "id": "todo-order-detail", "content": "调整 order-detail 相关组件（order-detail-header.vue、order-detail.vue）" },    { "id": "todo-order-info", "content": "精简 order-info.vue 信息块（快捷导航、产品信息、险种信息表格）" },    { "id": "todo-verify", "content": "联调检查与 lint 验证" }  ]}
```

### 上述在的step-1到5的具体信息，就是你的具有需求的描述

###

### 编写 Plan 的注意事项

1. **overview**

   要简洁明确，一句话说清楚要做什么
2. **steps**

   要具体到文件路径和操作，不要写抽象描述
3. **todos**

   要原子化，每个 todo 对应一个可独立完成的任务
4. **命名规范**

   todo id 使用 `todo-` 前缀，便于识别

---

##

## 五、Plan 模式下常见问题及解决方案

##

### 问题 1：Plan 创建后，如何确认？

**现象**：创建 Plan 后，AI 会等待你的确认，此时不能修改代码。

**解决**：

* 仔细阅读 Plan 的 `overview` 和 `steps`，确认是否符合需求
* 如果不符合，告诉 AI 需要调整的地方，AI 会重新创建 Plan
* 确认无误后，明确告诉 AI "可以开始执行" 或 "批准这个 Plan"

### 问题 2：执行过程中发现 Plan 有遗漏怎么办？

**现象**：执行到某写个 todo 时，发现需要额外的工作。

**解决**：

* **小调整**

  可以在当前 Plan 中继续执行，通过 `todo_write` 添加新的 todo
* **大调整**

  如果方向性改变，需要重新创建 Plan

**示例 - 中途新增任务**：

```

```

```
// 原有 todos[  { "id": "todo-mock", "content": "设计 mock 数据模块" },  { "id": "todo-order-tab", "content": "替换订单&售后标签" }]// 执行中发现需要新增todo_write({  "merge": true,  "todos": [    { "id": "todo-fix-scroll", "content": "修复表格滚动条问题", "status": "pending" }  ]})
```

###

###

### 问题 3：执行时遇到技术问题（如样式、接口）怎么办？

**现象**：实现某个功能时，发现样式不对、接口缺失等。

**解决**：

* **样式问题**

  先按 Plan 目标继续实现，完成后统一调整样式
* **接口问题**

  如果 Plan 中已考虑（如使用 mock），按 Plan 执行；如果未考虑，先询问用户，再决定是否调整 Plan
* **不影响 Plan 目标的小问题**

  记录问题，继续执行，最后统一处理

### 问题 4：用户中途反馈需要调整怎么办？

**现象**：执行过程中，用户提出新的要求（如调整列顺序、修改样式）。

**解决**：

* **符合 Plan 目标的调整**

  直接在当前 Plan 中迭代，不需要重建 Plan
* 例如：Plan 目标是"替换订单&售后"，用户反馈"列顺序不对"，这是实现细节，直接调整即可
* **偏离 Plan 目标的调整**

  需要评估是否重建 Plan
* 例如：Plan 目标是"隐藏字段"，用户要求"删除字段"，这是方向性改变，需要重新 Plan

###

### 问题 5：如何知道当前执行到哪个任务？

**现象**：Plan 有多个 todos，想知道进度。

**解决**：

* AI 会在执行每个 todo 时，使用 `todo_write` 更新状态：
* `pending`

  → `in_progress`：开始执行
* `in_progress`

  → `completed`：完成
* 你可以随时查看 todos 列表，了解当前进度

---

##

##

## 六、Plan 执行过程展示

##

### 执行流程示例

以 `cursor-plan.md` 的需求为例，执行过程如下：

#### 1. Plan 创建阶段

```

```

####

```
AI: 我已经创建了 Plan，包含以下步骤和 todos：- Step 1: 搜索页调整- Step 2: 用户详情裁剪- Step 3: 订单详情调整- Step 4: 订单信息模块精简- Step 5: 校验与测试Todos:- todo-search (pending)- todo-user-detail (pending)- todo-order-detail (pending)- todo-order-info (pending)- todo-verify (pending)请确认是否可以开始执行？
```

####

####

#### 2. 执行阶段

**Todo 1: todo-search**

```

```

```
AI: 开始执行 todo-searchtodo_write({ "id": "todo-search", "status": "in_progress" })// AI 执行操作：1. read_file("search-home.vue")2. search_replace(...) // 注释团险、费率查询3. search_replace(...) // 注释个险条件下的字段4. search_replace(...) // 注释列表中的列todo_write({ "id": "todo-search", "status": "completed" })
```

**Todo 2: todo-user-detail**

```
AI: 开始执行 todo-user-detailtodo_write({ "id": "todo-user-detail", "status": "in_progress" })// AI 执行操作：1. read_file("user-detail.vue")2. search_replace(...) // 注释 userInfo 字段3. search_replace(...) // 注释 tabs4. search_replace(...) // 注释服务记录模块// ... 更多操作todo_write({ "id": "todo-user-detail", "status": "completed" })
```

**Todo 3-5: 类似流程...**

#### 3. 完成阶段

```

```

---

##

```
AI: 所有 todos 已完成！已完成的任务：✅ todo-search: 搜索页调整完成✅ todo-user-detail: 用户详情裁剪完成✅ todo-order-detail: 订单详情调整完成✅ todo-order-info: 订单信息模块精简完成✅ todo-verify: 校验通过，无 lint 错误修改的文件：- src/views/components/search-home.vue- src/views/components/user-detail.vue- src/views/components/order/order-detail-header.vue- src/views/components/order-detail.vue- src/views/components/order/order-info.vue
```

##

##

## 七、中途是否可以新增任务？

##

**答案：可以！**

### 场景 1：执行中发现遗漏

在执行过程中，如果发现 Plan 中遗漏了某个任务，可以动态添加：

```

```

```
// 使用 todo_write 添加新任务todo_write({  "merge": true,  "todos": [    {       "id": "todo-fix-style",       "content": "修复表格样式问题",       "status": "pending"     }  ]})
```

**注意事项**：

* 使用 `merge:true` 来合并到现有 todos，而不是替换
* 新任务通常放在最后，或者根据依赖关系插入到合适位置

### 场景 2：用户中途反馈

如果用户在执行过程中提出新的要求，可以：

1. **小调整**：直接添加 todo，继续执行
2. **大调整**：评估是否需要重建 Plan

* 如果新需求与 Plan 目标一致，添加 todo 即可
* 如果新需求偏离 Plan 目标，建议重新创建 Plan

### 实际案例

###

在 `cursor-plan-2.md` 的执行过程中，用户反馈"表格有滚动条，希望全量展示"，这是一个实现细节问题，可以直接添加 todo：

```
todo_write({  "merge": true,  "todos": [    { "id": "todo-fix-table-width", "content": "调整表格宽度，确保全量展示无滚动条", "status": "pending" }  ]})
```

然后继续执行，不需要重建 Plan。

---

##

##

## 八、执行完成后如何优化 Plan？

##

### 优化场景

Plan 执行完成后，可能会遇到以下情况需要优化：

1. **功能已实现，但需要优化**

   性能、样式、代码质量
2. **发现更好的实现方案**

   重构、提取公共组件
3. **用户反馈需要改进**

   交互体验、数据展示

### 优化方式

###

#### 方式 1：基于现有 Plan 迭代

如果优化方向与 Plan 目标一致，可以继续使用现有 Plan，添加新的 todos：

```

```

```
todo_write({  "merge": true,  "todos": [    { "id": "todo-optimize-performance", "content": "优化表格渲染性能", "status": "pending" },    { "id": "todo-refactor-components", "content": "提取公共组件，减少代码重复", "status": "pending" }  ]})
```

####

####

#### 方式 2：创建新的 Plan

如果优化涉及大范围改动或方向性调整，建议创建新的 Plan：

```

```

```
{  "name": "legal-affairs-optimization",  "overview": "优化信息模块的性能和代码质量，提取公共组件，优化用户体验。",  "steps": [    "Step 1 – 性能优化：优化表格渲染，使用虚拟滚动",    "Step 2 – 代码重构：提取公共组件，减少重复代码",    "Step 3 – 用户体验：优化交互反馈，添加加载状态"  ],  "todos": [    { "id": "todo-performance", "content": "优化表格性能" },    { "id": "todo-refactor", "content": "重构公共组件" },    { "id": "todo-ux", "content": "优化用户体验" }  ]}
```

###

###

### 优化 Checklist

执行完成后，可以按以下清单检查是否需要优化：

* **代码质量**：是否有重复代码？是否可以提取公共组件？

  **性能**：是否有性能瓶颈？表格渲染是否流畅？

  **用户体验**：交互是否顺畅？是否有加载状态？错误提示是否友好？

  **可维护性**：代码结构是否清晰？注释是否充分？

  **测试**：是否需要添加单元测试？是否需要 E2E 测试？

### 实际案例

在 `cursor-plan-2.md` 执行完成后，可能的优化方向：

1. **性能优化：**

   如果订单&售后表格数据量大，可以考虑虚拟滚动
2. **代码重构：**

   如果线下收益的两个 panel 有相似逻辑，可以提取公共组件
3. **用户体验：**

   添加加载状态、错误提示、空数据提示

这些优化可以创建新的 Plan，或者如果改动不大，可以直接在当前 Plan 中添加 todos。

---

##

##

## 九、总结

##

### Plan 模式的核心价值

###

1. **降低风险：**

   复杂需求先看方案，确认后再执行
2. **提高效率：**

   清晰的步骤和任务列表，避免遗漏
3. **便于协作：**

   用户和 AI 对目标一致，减少返工
4. **可追溯性：**

   所有改动都有明确计划，便于维护

### 最佳实践

###

1. **Plan 要具体：**

   步骤要包含文件路径和操作，不要写抽象描述
2. **Todos 要原子化：**

   每个 todo 应该是可独立完成的任务
3. **执行要顺序：**

   按 Plan 中的顺序执行，完成一个再下一个
4. **及时更新状态：**

   每个 todo 完成后立即更新状态
5. **灵活调整：**

   遇到问题及时沟通，小调整直接迭代，大调整重建 Plan

### 常见误区

###

❌ **误区 1**：Plan 创建后就不能修改

✅ 正确：小调整可以直接添加 todos，大调整可以重建 Plan

❌ **误区 2**：Plan 必须一次性写完整

✅ 正确：可以在执行过程中根据实际情况添加任务

❌ **误区 3**：Plan 执行完成后就不能优化

✅ 正确：可以基于现有 Plan 迭代，或创建新的优化 Plan

###

### 快速参考

###

| 操作 | 工具/方法 | 用户操作方式 | 说明 |
| --- | --- | --- | --- |
| 创建 Plan | `create_plan` | 描述需求，AI 自动调用 | AI 根据需求自动创建计划 |
| 更新 Todo 状态 | `todo_write` | AI 自动调用 | AI 执行时自动更新任务状态 |
| 添加新任务 | `todo_write` + `merge:true` | 告诉 AI 需要添加任务 | AI 自动添加新任务到列表 |
| 检查代码质量 | `read_lints` | AI 自动调用 | 执行完成后自动检查 |
| 澄清需求 | `ask_question` | AI 自动调用 | AI 发现需求不明确时自动提问 |

**重要提示**：所有工具都是 AI 自动调用的，用户只需要用自然语言与 AI 对话即可。

---

##

---

通过以上教程，你应该能够：

1. ✅ 理解 Cursor Plan 模式的核心概念
2. ✅ 根据需求编写清晰的 Plan
3. ✅ 处理执行过程中的常见问题
4. ✅ 灵活调整和优化 Plan
5. ✅ 高效完成复杂需求的开发

祝你在使用 Plan 模式时更加得心应手！💪