---
title: Verify, then build：先验证再构建
author: 思维编译
date: V1kiV1ki
url: https://mp.weixin.qq.com/s?__biz=MzYyNTAyNzk1OQ==&mid=2247484901&idx=1&sn=7a5fd5fbcfc61490adbf3961445d4133&chksm=f18f2e172174ca1c95c39e3767be0c8f8f61f5012a0d2bda2300d4ba94bcbba67bc807ee89d8&mpshare=1&scene=24&srcid=06029PqMhZt4G4nLZlkyoCYk&sharer_shareinfo=94e33f7d3489cb48da9252bf0a7c5686&sharer_shareinfo_first=94e33f7d3489cb48da9252bf0a7c5686#rd
---

Verify, then build：agent 写代码之前，先让 agent 写验证系统

Warp 把 Mermaid 渲染器移植到 Rust 的过程，真正值得看的不是 agent 数量，而是验证循环。

   
   
   先验证，再构建
   先把成功标准变成可执行环境，再让 agent 写代码。
   
     
     
     
     
   
   
     
   
   
     
     用户问题
     输入与约束
     
     验证环境
     测试 / 视觉差异
     
     编码实现
     修复一个差异
     
     验证反馈
     通过 / 差异 / 回归
   
   
   工程重点：让“对不对”变成 agent 能自己检查的东西
 

Zach Lloyd 这篇 X Article 的标题很短：**Verify, then build**。

它表面上是在讲 Warp 怎么把 Mermaid diagram 支持做进自己的 Markdown editor。更具体一点，是怎么把一个原本依赖 JavaScript 和浏览器环境的 Mermaid 渲染链路，改造成一个 pure Rust 的 Mermaid → SVG library。

但这篇文章真正有意思的地方，不在 Mermaid，也不在 Rust，而在一个更通用的 agent 工程判断：

**当写代码的成本接近 0，真正的瓶颈就不再是“能不能写出来”，而是“怎么知道它写对了”。**

一、为什么 Warp 不能直接用 mermaid.js

Mermaid 是一种用文本描述图的 DSL。你写一段类似 Markdown 的语法，它可以渲染 flowchart、class diagram、sequence diagram、Sankey chart 等 20 多种图。

主流实现是 **mermaid.js**。问题在于，mermaid.js 的常规使用方式需要 JavaScript / TypeScript binding，也通常依赖浏览器环境完成渲染。对 Electron app 或 VSCode fork 来说，这不是什么大问题，因为它们本来就带着 Web runtime。

但 Warp 不一样。Warp 是一个 pure Rust app，有自己的 rendering engine，直接走 GPU。如果为了 Mermaid 支持引入 JS runtime 或外部浏览器环境，就会变得慢、重，而且和产品架构不匹配。

   
   
   同一个 Mermaid，两个产品架构
   浏览器路线很方便；Warp 需要 Rust 原生路线。
   
     
     浏览器路线
     mermaid.js
     浏览器渲染
     Electron / VSCode
   
   
     
     Rust 原生路线
     解析 → 语法树
     布局计算
     生成 SVG → GPU 绘制
     
   
   
   
   
   Warp 的选择：文本 → 解析 → 布局 → SVG
 

所以 Zach 的目标变成：做一个 pure Rust 的 Mermaid → SVG library。

我把他们开源的 `warpdotdev/mermaid-to-svg` 拉到本地看了一遍。README 里写得很直接：架构是 **Mermaid text → Parser → AST → Layout(dagre\_rust) → SVG Renderer → SVG string**。

代码上，`lib.rs` 会按 diagram type 分发到不同 renderer：flowchart、sequenceDiagram、classDiagram、stateDiagram、pie、gantt、timeline、journey、kanban、quadrantChart、C4 等。README 同时也明确说：当前 flowchart 是 primary，其它 diagram type 还处于 experimental / in progress。

二、真正的难点：图像正确性怎么验证

普通功能可以写 unit test。输入是什么，输出是什么，断言一下就行。

但 Mermaid 渲染器的麻烦在于：它的输出不是一个简单字符串，而是一张图。图里有节点、边、箭头、label、subgraph、layout、spacing、颜色、弯曲路径。你不能只看 SVG 是否生成成功，因为“能生成”离“生成得对”差很远。

Zach 的做法是把问题转成一个 black-box verification problem：已有 mermaid.js 这个 reference implementation，那就先让 agent 生成大量 Mermaid diagrams，再用 reference implementation 生成 canonical renderings。

接下来，agent 先 port 一版 Rust 实现。第一版当然很差，但它能从 Mermaid spec 生成 SVG，这就是 bootstrapping。

真正的工作从这里开始：**Visual Verification Loop**。

   
   
   视觉验证循环
   不是“看两张图像不像”，而是把差异变成可修复的 D1 / D2 / D3。
   
     
       
     
   
   
     
     生成样例
     Mermaid fixtures
     
     标准渲染
     mermaid.js PNG
     
     Rust 渲染
     ours SVG / PNG
     
     
     
     视觉差异报告
     D1：间距
     D2：连线　D3：标签
     
     只修一个问题
     选择一个 D#
     改代码并回归
     
     
     
   
   
   关键不是让 agent “感觉哪里不对”
   而是让差异编号、可复现、可回归
 

这个 loop 的细节已经被 Warp 写进了开源仓库的 `docs/agent-visual-verification.md`。

流程非常具体：先跑 `./scripts/visual_test.sh <DIAGRAM_TYPE>` 生成 PNG pairs：`ref_*.png`、`our_*.png`、`our_*.svg`。然后 agent 选择一个 focus fixture，写一份 computer-vision-style comparison report，报告里必须包含 reference image description、our image description，以及编号的 D1、D2、D3 差异。

更关键的是，它要求每轮只选一个具体 next fix。修完以后跑 `cargo test --lib`，再跑 visual verification，把 pre/post artifacts 复制进 validation run directory。

这不是让 agent “感觉一下哪里不对”。这是把视觉判断拆成了可记录、可回归、可并行的工程协议。

三、并行 agent 不是关键，分片方式才是关键

原文里最容易被转述成噱头的一句是：Zach 曾经说 Mermaid 支持是由 80 个 parallel cloud agents 写出来的，并且用 computer vision 自我验证。

但我觉得这里不能只看 agent 数量。真正重要的是：这些 agent 不是同时乱改同一块代码，而是按 diagram type shard。

Mermaid 有很多图表类型。flowchart、sequenceDiagram、classDiagram、stateDiagram、pie、gantt、timeline，每种图的 primitives 不一样。flowchart 关心节点和边，sequence diagram 关心 participant、message、note、fragment，gantt 关心时间轴，pie 关心 slice 和 legend。

所以 Warp 的 protocol 明确说：一个 agent 只负责一个 diagram type；每次只选一个 fixture；不要一次处理所有 fixture；不要为了一个 fixture 打坏另一个。

   
   
   不是“很多 agent”，而是按验证边界分片
   每类图都有自己的样例、规则和视觉差异。
   
   调度器
   分配 / 合并
   
     
     流程图
     
     
     时序图
     
     
     类图
     
     
     饼图
     
     
     甘特图
     
     
     时间线
     
   
   
   并行的前提：每个 shard 有清晰验证边界
 

这也是为什么这件事有工程含义。并行 agent 本身不稀奇，难的是找到安全的分片边界。

如果任务之间高度耦合，20 个 agent 只会制造 20 倍冲突。但如果你能把任务拆成 diagram type、fixture、D# difference、branch、pre/post artifacts，agent 才能真的并行。

四、我本地验证了一下代码状态

我把 `warpdotdev/mermaid-to-svg` 拉到本地，当前看到的 HEAD 是 `fb49f4f`。

先跑 `cargo test --lib`，结果是 121 passed，0 failed，16 ignored。也就是说，至少公开仓库当前的 Rust 单元测试是能跑通的。

然后我跑 `./scripts/visual_test.sh flowchart`。这里遇到一个小问题：脚本里的 `rasterize_svg_with_puppeteer.js` 用的是 CommonJS 的 `require`，但我当前工作区上层项目有 `"type":"module"`，Node 会把它当 ES module 处理，于是报 `require is not defined`。

我在临时克隆的仓库根目录加了一个局部 `package.json`，内容只有 `{"type":"commonjs"}`，再跑就通过了。flowchart visual test 生成了 51 组 reference / ours PNG pairs。

这个小插曲反而说明了一个现实问题：test harness 本身也是工程系统，也会受运行环境影响。agent 能不能可靠跑，不只取决于 prompt，也取决于 harness 是否把环境边界说清楚。

五、这和普通 TDD 不完全一样

你可以把 Zach 的方法理解成一种 agent 时代的 TDD，但它和传统 TDD 不是一回事。

传统 TDD 里，人写测试，人写实现。这里更像是：人定义方向，agent 帮你构造验证样本、生成 reference output、写初版实现、比较差异、提出一个最小修复，再循环。

也就是说，agent 不只是 feature code generator，它还在参与建设自己的反馈环境。

这点很关键。很多 agent coding 失败，不是因为模型不会写代码，而是因为它没有一个足够短、足够明确、足够自动化的 feedback loop。没有 feedback loop，agent 只能靠一次性上下文里的猜测；有了 feedback loop，它就可以 hill climbing。

   
   
   什么任务适合 Verify, then build
   不是所有任务都能靠 harness 自动爬坡。
   
     
     
     适合
     有参考实现
     可自动比较
     可回归测试
     
     
     部分适合
     可抽样验证
     需要人工 review
     指标不完整
     
     
     不适合
     目标纯主观
     没有 oracle
     验证代价过高
   
   
   先问：这个任务能不能构造一个
   agent 可执行的“对不对”？
 

六、边界也要说清楚

这套方法不是所有 agent coding 的银弹。

它成立的前提是：你能找到某种 reference，或者至少能定义可执行的成功标准。Mermaid to SVG 很适合，因为已经有 mermaid.js 作为 canonical output，而且视觉差异可以被拆成 fixture-backed PNG diffs。

但如果是产品判断、交互体验、需求取舍，很多东西没法完全自动验证。你可以写 harness 辅助检查，但最终还是需要人判断。

另外，visual parity 也不等于全部正确。截图像素、字体渲染、浏览器版本、layout nondeterminism 都可能带来噪声。Warp 的文档里也专门提到，对 gitGraph commit hash 这类 nondeterministic output，要先定义 parity criteria，而不是盲目追 pixel diff。

七、结论：agent 工程的重点正在前移

我觉得这篇文章可以压缩成一句工程规则：

**不要急着让 agent 写产品代码。先判断这个任务能不能被验证；如果能，就让 agent 先把验证系统写出来。**

这件事和最近几篇关于 harness、artifact、memory benchmark 的文章其实连在一起。

Harness 不是模型外面一层可有可无的壳。它决定 agent 能不能感知结果、修正错误、并行推进。Artifact 也不是为了把输出做漂亮，而是为了让人更快审查、更快给反馈。Benchmark 也不应该只报一个 accuracy，因为真正影响生产可用性的还有 latency、cost、debuggability。

Verify, then build 说的是同一件事：agent 时代的工程能力，不只体现在 prompt 怎么写，也体现在你能不能把任务放进一个可验证、可观察、可迭代的环境里。

代码会越来越便宜。验证不会。

参考资料

Zach Lloyd — Verify, then build

https://x.com/zachlloydtweets/status/2054258738328981809

warpdotdev/mermaid-to-svg

https://github.com/warpdotdev/mermaid-to-svg

Agent visual verification docs

https://github.com/warpdotdev/mermaid-to-svg/blob/main/docs/agent-visual-verification.md

Mermaid

https://github.com/mermaid-js/mermaid

Dagre

https://github.com/dagrejs/dagre

Zach Lloyd 3 月 25 日 Mermaid 支持预告

https://x.com/zachlloydtweets/status/2036504234422608327

Warp markdown table rendering 示例

https://x.com/warpdotdev/status/2041621551544414346

关于本文

本文由 **Zero** 协助完成。

**生成与校验流程**：

- 原文采集：通过浏览器读取 Zach Lloyd 的 X Article 全文，并提取其中引用链接。

- 代码分析：克隆 warpdotdev/mermaid-to-svg，阅读 README、核心 Rust 入口、visual verification 文档与脚本。

- 本地验证：运行 cargo test --lib，确认公开仓库当前测试为 121 passed / 0 failed / 16 ignored。

- 视觉验证：运行 flowchart visual\_test，生成 reference / ours PNG pairs，并记录 CommonJS/ESM 环境边界问题。

- 配图制作：使用微信兼容 inline SVG 与 SMIL 动画手写动态图，不使用外部图片生成模型。

- 文章撰写：基于原文、开源代码、文档和本地验证结果，整理为适合微信公众号阅读的中文长文。

**Zero 是什么**：一个在 macOS 上自主执行任务的 AI Agent 系统，具备浏览器自动化、本地工具链、多 Agent 协作、长期记忆等能力。本文从原文采集、代码验证到互动式正文构建，均在与 Zero 的同一次对话中通过多轮交互完成——我提出方向，Zero 负责执行、生成、校验和迭代。

**Zero 项目地址**：https://github.com/V1ki/zero