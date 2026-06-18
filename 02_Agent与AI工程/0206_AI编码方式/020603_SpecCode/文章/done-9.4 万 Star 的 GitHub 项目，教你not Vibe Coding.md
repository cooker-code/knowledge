> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020603_SpecCode/020603_核心知识点/SpecCode规格驱动与验收边界|SpecCode规格驱动与验收边界]]
---
title: 9.4 万 Star 的 GitHub 项目，教你not  Vibe Coding
author: 顶尖架构技术栈
date:
url: https://mp.weixin.qq.com/s?__biz=MzkwNzUzODAzNQ==&mid=2247489077&idx=1&sn=017fb5a182a8cd796ba1bdce50377558&chksm=c1afc68361c55fdb49d669c8b371ecb56d1fc4ed4e6eefff3f2edf6b2dbc4dfaab21c1a5712e&mpshare=1&scene=24&srcid=0521ZHbN0tbiyoTKKFSREpMx&sharer_shareinfo=76b40328e5d18a6237a33ba95e4e0744&sharer_shareinfo_first=76b40328e5d18a6237a33ba95e4e0744#rd
---

3 个月前我让 AI 帮我写了一个批量处理模块，当时测了一下，跑得挺好。上周需求变了，要改一个核心逻辑，打开那段代码看了 10 分钟，决定整个删掉重写——没有测试，接口东一块西一块，逻辑散在五个文件里。

这不是 AI 的问题。AI 只是把你脑子里"差不多能用"的期望实现出来了，不多也不少。

最近翻到一个仓库：mattpocock/skills (https://github.com/mattpocock/skills)。作者是 Matt Pocock，做 TypeScript 教育的，出过 Total TypeScript 系列，在前端圈挺有名气。这个仓库跟 TypeScript 没什么关系，讲的是怎么让 AI 辅助编程不把工程质量带垮。9.4 万 Star，副标语三个字：Not Vibe Coding。

装起来很简单：

```
npx skills@latest add mattpocock/skills
```

装进去是 13 个斜线命令，分工程、生产力、杂项三个目录。拆开看，是一套完整的四阶段工作流：**开始前对齐 → 编码阶段 → 调试阶段 → 效率工具**。

---

## 第一阶段：开始写代码之前

这是当前博客版本里完全缺失的部分，也是整个方法论的起点。

### /grill-with-docs：用项目自己的词汇对齐需求

很多 AI 编程的失败不发生在写代码阶段，发生在「AI 理解的需求」和「你真正要的」之间的偏差上。`/grill-with-docs` 就是为这个设计的——在动手之前，先让 AI 把你的计划拿着问题一个个砸一遍。

不只是问问题，它在问的过程中会做三件事：

**用项目词汇挑战你的措辞。** 如果 CONTEXT.md 里把退款申请定义成 `RefundClaim`，而你在描述需求时说了 `RefundRequest`，它会当场叫停："你的词汇表把这个叫 RefundClaim，你现在说的是同一件事吗？"

**用代码验证你的陈述。** 你说"部分取消是支持的"，它翻开代码发现实现里只有整单取消——然后问你，代码和你刚才说的哪个是对的。

**边问边更新 CONTEXT.md 和 ADR。** 每解决一个术语歧义，立刻写进词汇表；每做一个实质性的架构决定，判断要不要记 ADR。不是等到最后统一整理，是当场捕获。

ADR 记录有严格条件：必须同时满足「改变成本高」「没有上下文会让人困惑」「有真实的方案取舍」，才值得记一条。防止 ADR 变成流水账。

### /prototype：先用抛弃型代码回答一个问题

在真正写代码之前，如果有设计上的不确定性，先做一个 prototype。它会根据你的问题走两条路：

**问题是"这个逻辑/状态模型对不对"** → 做一个最小化的终端交互程序，把状态机推过那些在纸上难以推演的边界情况。没有持久化，没有测试，没有 error handling，只需要能跑起来，能打印出每步操作后的完整状态。

**问题是"这个界面应该长什么样"** → 在一个路由上生成几个风格截然不同的 UI 变体，通过 URL 参数切换，加一个浮动 bar 来快速对比。

两条路共同的规则：**从第一天就是抛弃型代码，明确标注出来，放在靠近被原型化的模块旁边，而不是单独开目录。** prototype 回答完那个问题就删掉，或者把验证过的决策提炼进正式代码。不要让它腐烂在仓库里。

---

## 第二阶段：写可以改的代码

### 测试要测行为，不测实现

判断一个测试写没写好，有个很简单的标准：**改了内部实现，测试挂了吗？** 挂了，说明你在测实现。

好测试只关心外部可观察的行为：

```
test("user can checkout with valid cart", async () => {
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

坏测试测的是内部调用了哪个函数，一旦你重构内部结构，测试就挂了——但行为没变。

还有一条更关键的：不要先把所有测试批量写完再统一实现。skill 里叫这个 "Horizontal Slicing"，批评说批量写出来的测试，测的是你**想象中的**接口，而不是跑通一个之后才知道的**真实接口**。正确做法是 "Vertical Tracer Bullet"：一个测试 → 立刻实现 → 下一个测试。

TDD 两种做法的对比

### Mock 只放在边界，接口设计要可测

Mock 只用在系统边界：外部 API、数据库、时间和随机数。你自己的模块不要 Mock。

让 Mock 变容易的关键是接口设计上的三条规则：接受依赖不要在内部创建（依赖注入）；返回结果不产生副作用；接口面积尽量小。

```
// 可测试：依赖从外部传入
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// 不可测试：内部自己 new，无法在测试里替换
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

### 小接口，大实现

来自 Ousterhout《软件设计哲学》的 Deep Module 概念：接口越小，实现越深，模块越有价值。

深模块 vs 浅模块

判断方法是"删除测试"：想象把这个模块删掉，复杂度消失了说明它是透传层；复杂度重新分散到 N 个调用方里，说明它在发挥真正作用。

AI 生成代码有一个自然倾向：把每一步都提取成函数，造成大量 3-5 行的浅函数，接口负担很重。

还有一条 Seam 纪律：**一个 Adapter = 假设的 Seam，两个 Adapter = 真实的 Seam**。没有两种实现（production + test），就不要引入这个抽象。为了"将来可能替换"加接口，大多数情况下是在提前还没欠下的债。

---

## 第三阶段：调试

### /diagnose：先建反馈闭环，再猜原因

`/diagnose` 是 6 个阶段的结构化调试流程，最核心的是 Phase 1。

> 拥有快速、确定性、可自动运行的 pass/fail 信号，就等于解决了 90% 的 bug。

在开始分析原因之前，先想清楚怎么验证。skill 里列了 10 种构建反馈循环的方式，从写一个失败的单元测试，到无头浏览器脚本，到 `git bisect` 自动化，按优先级排列。2 秒内出结果的确定性信号，比"加 log 盯着看"效率高一个量级。

Phase 3 要求：先列出 3-5 个排好序的假设，给用户看，再开始验证。每个假设必须是可证伪的——

> "如果 X 是原因，那么改 Y 会让 bug 消失 / 改 Z 会让它变严重。"

说不出预测的假设是直觉，不是假设。

一个实用细节：debug 日志用固定前缀标记（如 `[DEBUG-a4f2]`），调试完 `grep` 一次删干净。否则临时日志散在代码里，下次 AI 看到了还以为是规范写法。

---

## 第四阶段：从对话到可执行的 Issue

这部分是整个项目里最有意思的一块——把 AI 对话直接接到项目管理工作流上。

### /to-prd：把对话变成结构化 PRD

`/to-prd` 不是再采访你，是直接综合当前对话上下文和代码库现状，生成一份结构化的 PRD，包含问题陈述、用户故事、实现决策、测试决策、Out of Scope，然后推送到 issue tracker。

有一个有意思的细节：如果 prototype 阶段产出了某段代码片段，可以直接内联进 PRD——不是整个 prototype，是那段"比文字描述更精确"的决策片段，比如状态机结构、schema 形状、类型定义。其他代码都不要放，会过时。

### /to-issues：按垂直切片拆 Issue

`/to-issues` 把 PRD 或计划拆成可以独立执行的 issue，每个 issue 是一个"垂直切片"——穿过所有层（schema、API、UI、测试），完成后可以独立演示或验证。

它还会给每个 issue 打标：**AFK**（away from keyboard，可以完全交给 AI agent 执行）或 **HITL**（human in the loop，需要人工决策）。拆完让你确认依赖关系和粒度，再按依赖顺序发布到 issue tracker。

### /triage：Issue 的状态机

这是整个仓库里工程化程度最高的一个 skill，把 issue 管理做成了状态机。

两个分类 role：`bug` / `enhancement`

五个状态 role：

* `needs-triage` → 还没评估
* `needs-info` → 等报告者补充信息
* `ready-for-agent` → 已完整描述，可以交给 AI agent
* `ready-for-human` → 需要人工实现（判断类、外部权限类）
* `wontfix` → 不会处理

bug 类型的 issue 在 grilling 之前，会先尝试复现——读报告里的步骤，追代码路径，跑测试。能复现的 bug 变成 `ready-for-agent` 的信心要高得多。

---

## 效率工具

### /zoom-out：找不到方向时要一张地图

`/zoom-out` 是最简单的 skill，一句话：

> 我对这片代码不熟悉。帮我上升一个抽象层级，给我一张相关模块和调用方的地图。

进入一个陌生代码区域的第一步，不是直接开始改，是先要地图。

### /caveman：省掉 75% 的废话

触发方式：在对话里说 "caveman mode"。触发后永久生效，直到说 "stop caveman"。

**正常模式：**

> Sure! I'd be happy to help you with that. The issue you're experiencing is likely caused by a problem in the authentication middleware...

**Caveman 模式：**

> Bug: auth middleware. Token expiry check `<` → `<=`. Fix:

技术信息一个字没少，废话全没了。长对话里每轮省几百 token，积累可观。遇到破坏性操作或安全警告时自动恢复完整表达，操作确认完再切回 caveman。

### /handoff：会话交接文档

跨 session 工作时，`/handoff` 把当前对话压缩成一份交接文档，保存到系统临时目录。包含：当前进展、待完成的工作、建议的下一步调用哪些 skill。不重复 PRD、ADR、issue 里已有的内容，直接引用路径。

---

## 给 AI 一本项目词典

`CONTEXT.md` 放在项目根目录，记录项目的领域词汇——不是给人看的文档，是给 AI 和人都用的共同语言。

配套的 ADR（架构决策记录）。每次讨论出了一个实质性决定，特别是"决定不做某件事"，记一条 ADR 写清楚原因。下次让 AI 分析架构，它不会重新建议已经否决的方向。

两个文件合起来，相当于给 AI 建了一个跨对话的项目记忆。

---

## 全貌

把 13 个 skill 放在一起，是一条完整的流水线：

```
/grill-with-docs 或 /grill-me   →  对齐需求，校准词汇
/prototype                      →  抛弃型代码回答设计问题
/to-prd                         →  对话 → 结构化 PRD
/to-issues                      →  PRD → 可执行垂直切片 Issue
/triage                         →  Issue 状态机管理
/tdd + /diagnose                →  写代码 + 调试
/improve-codebase-architecture  →  找深化机会
/zoom-out                       →  不熟悉区域要地图
/caveman + /handoff             →  降低对话成本，跨 session 交接
CONTEXT.md + ADR                →  项目记忆
```

这不是一堆独立的最佳实践，是一套把软件工程原则装进 AI 工作流每个节点的完整系统。

我打算在当前几个项目里跑一段时间，到时候再分享哪几个 skill 真正改变了工作方式。