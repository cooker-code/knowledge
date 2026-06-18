---
title: AI 写的 React 全是坑？Million.js 作者祭出「代码体检」神器，GitHub 狂揽近万 Star！
author: 林叔说事
date: 图灵硅硅图灵硅硅
url: https://mp.weixin.qq.com/s?__biz=MzkwNDY0MTI1OQ==&mid=2247494642&idx=1&sn=d4f745e001dca18fdcdad43a8db1b493&chksm=c1b9ac7b9c989b965b46565d5542dbc7032071d5d12d170dd34eaa78f2f917ca67a6e5648aa9&mpshare=1&scene=24&srcid=0516CdgoSxgu31uFVWx2vury&sharer_shareinfo=ca47254b669f90149da92dfc825b2bf0&sharer_shareinfo_first=ca47254b669f90149da92dfc825b2bf0#rd
---

导读  
AI Agent 帮你写 React，写完你敢直接上线吗？Million.js 作者 Aiden Bai 推出 React Doctor，一条命令扫描整个代码库，给出 0-100 的健康评分，覆盖安全、性能、正确性、无障碍、包体积、架构六大维度。GitHub 上线三个月，star 数逼近一万，还能直接教你的 AI Agent 学会 React 最佳实践。

## 一个扎心的问题：你的 Agent 写的 React，到底有多烂？

2026 年，AI 写代码已经不新鲜了。Cursor、Claude Code、Codex……各家 Agent 都能帮你从零搭一个 React 项目，速度快到离谱。

但一个越来越明显的问题浮上来了：**Agent 写的 React 代码，质量堪忧。**

`useEffect` 里塞 fetch 请求、state 派生逻辑乱套、没有做 cleanup 的副作用到处飘、`dangerouslySetInnerHTML` 随手就用、barrel import 把 bundle size 撑到爆炸——这些问题在 AI 生成的代码里高频出现，但人工 review 又极其耗时。

尤其是当你让 Agent 连续迭代、自动修 bug 的时候，它可能修了一个问题，又悄悄引入三个新的反模式。

**代码写得快，谁来兜底质量？**

Million.js 的作者 Aiden Bai 给出了他的答案：**React Doctor**。

## 一条命令，给你的 React 代码做全身体检

React Doctor 的核心理念可以用 GitHub 上的那句 slogan 概括：

> "Your agent writes bad React. This catches it."

「你的 Agent 写了一堆烂 React 代码，这个工具帮你兜住。」

用法极其简单，在项目根目录跑一条命令：

```bash npx -y react-doctor@latest . ```

它会扫描你的整个代码库，输出一个**0 到 100 的健康评分**，并附带可执行的诊断建议。

▲ React Doctor 的 GitHub 仓库，已获得 9.4k star，296 fork

评分分三档：

* **75 分以上**

  ：Great，代码质量过关
* **50 到 74 分**

  ：Needs work，有明显需要改进的地方
* **50 分以下**

  ：Critical，赶紧修

评分公式也很透明：`100 - (触发的 error 级规则数 × 1.5) - (触发的 warning 级规则数 × 0.75)`。注意，这里计算的是**触发了多少种规则**，50 个地方犯同一种错只扣一次分。

## 六个维度，把 AI 写的坑逐个揪出来

React Doctor 的扫描覆盖六大维度，每一个都在针对 AI Agent 的常见失误：

**1. State & Effects（状态与副作用）**

AI 最爱犯的错。`useEffect` 里做数据获取、state 之间互相派生触发连锁更新、没有 cleanup 的定时器和订阅。React Doctor 内置了 `no-fetch-in-effect`、`no-derived-state-effect`、`no-cascading-set-state` 等规则，直接对准这些高频坑。

**2. Security（安全）**

`dangerouslySetInnerHTML` 带来的 XSS 风险、GET handler 里的服务端状态变更——AI 在处理安全相关逻辑时经常缺乏边界意识，React Doctor 会把这类问题标记为 error 级别。

**3. Performance（性能）**

barrel import 导致的 tree-shaking 失败、渲染函数内部声明组件、array index 做 key——这些性能陷阱 Agent 踩得非常勤快。

**4. Accessibility（无障碍）**

缺少 alt 属性、autofocus 滥用、交互元素缺少键盘支持。AI 对无障碍的重视程度远低于人类开发者。

**5. Bundle Size（包体积）**

dead code 检测、未使用的导出和文件扫描。React Doctor 集成了 knip 做全项目级的死代码检测，帮你找到那些 Agent 生成后又弃用的文件。

**6. Architecture（架构）**

组件拆分是否合理、渲染逻辑是否过度嵌套。这一层把审查从单个规则提升到了项目结构级别。

## 最狠的一招：直接教你的 Agent 学规矩

React Doctor 有个特别值得关注的功能：**Agent 安装模式**。

```bash npx -y react-doctor@latest install ```

跑完这条命令，它会检测你项目里用了哪些 AI Agent（Claude Code、Cursor、Codex、OpenCode 等 50 多个），然后自动往项目里写入对应的规则文件——SKILL.md、AGENTS.md、.cursorrules。

这意味着什么？你的 Agent 在写代码之前，就已经知道了 React Doctor 的所有规则。**它从源头上减少了 Agent 写出坏代码的概率。**

这个设计思路很有意思：与其写完再修，不如让 Agent 写之前就知道什么能写、什么别碰。

## 不只是 CLI：GitHub Action + 在线排行榜

React Doctor 的野心不止于本地命令行。

**GitHub Action 原生支持**

你可以直接在 CI 流水线里接入 React Doctor。它作为 GitHub Marketplace 上的 composite action，支持在每次 PR 和 push 时自动扫描，并把结果以评论的形式贴回 PR 页面。

```yaml

* uses: millionco/react-doctor@main

with: diff: main github-token: ${{ secrets.GITHUB\_TOKEN }} ```

还支持 `--diff` 模式（只扫描改动文件）和 `--staged` 模式（只扫描暂存区，适合 pre-commit hook），在大型代码库里省时间。

**React Review 在线平台**

React Doctor 还有一个配套网站 react.review，可以直接粘贴 GitHub 仓库 URL 进行在线审计，并且有一个**公开排行榜**，展示扫描过的 React 项目评分排名。

▲ React Review 的在线排行榜，展示了各项目的健康评分

排行榜上，排名靠前的项目评分在 89-100 之间。这给了开源社区一个有趣的横向对比维度：你的 React 项目在行业里处于什么水平？

## 生态位：ESLint 和 oxlint 之上的专项层

有人可能会问：ESLint 不是已经有 react 插件了吗？

React Doctor 做的事和 ESLint 有交集，但定位不同。ESLint 的 react 插件是通用型 linter 的一个扩展，规则粒度偏细、需要你自己组合配置。React Doctor 把**React 特有的反模式**打包成了一个「体检套餐」，开箱即用，一条命令出报告。

而且它同时支持作为**oxlint 插件**和**ESLint 插件**独立运行，可以直接嵌入你现有的 lint 工具链。项目里已有的 `.eslintrc.json` 或 `.oxlintrc.json` 配置会被自动合并。

▲ npm 上的 react-doctor，当前最新版本 0.1.6，周下载量持续增长

从 npm 数据来看，react-doctor 已经发布了 50 多个版本，迭代相当频繁。作为一个今年 2 月才上线的项目，三个月做到 9400+ star，说明开发者社区对这类工具有真实需求。

## 值得关注的设计细节

**评分的「规则去重」机制**

前面提到，评分按「触发了多少种规则」计算，而非「触发了多少次」。这意味着你修了 49 处同类问题但还剩 1 处没修，分数不会变。把最后那 1 处也修了，整条规则的扣分才会移除。这个设计鼓励你**彻底消灭每一类问题**，而非零散修补。

**版本升级可能降分**

React Doctor 明确告知用户：新版本可能加入新规则，导致评分下降。分数降了不代表代码变差了，只是工具覆盖面更广了。如果你在 CI 里需要稳定分数，可以锁定版本号。

**配置灵活度**

支持全局关闭规则、按文件忽略规则、按文件 + 规则组合忽略（overrides），三层粒度。还可以用 `ignore.tags` 按标签批量关闭某一类规则，比如 `"design"` 标签关闭所有设计相关的 opinionated 规则。

## AI 写代码的下半场：从生成到守门

React Doctor 背后折射出的趋势值得重视：**当 AI Agent 写代码变成常态，代码审查工具的需求会跟着爆发。**

现在的 Agent 擅长快速生成功能代码，但对框架的最佳实践、安全边界、性能陷阱的理解仍然有限。人类 reviewer 不可能每一行都 review 到，自动化的专项审查工具就成了刚需。

React Doctor 瞄准的就是这个缺口：**Agent 写完，工具兜底。**再进一步，通过 install 模式把规则前置到 Agent 的上下文里，从源头降低出错率。

如果你的团队正在大量使用 AI Agent 写 React，建议跑一次 React Doctor 看看分数。结果可能会让你重新评估对 Agent 生成代码的信任度。

---

— END —

— END —