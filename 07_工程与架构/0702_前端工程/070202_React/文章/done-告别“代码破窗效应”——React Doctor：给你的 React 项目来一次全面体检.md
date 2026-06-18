> 已吸收至：[[07_工程与架构/0702_前端工程/070202_React/070202_核心知识点/React架构与质量边界准则|React架构与质量边界准则]]、[[07_工程与架构/0702_前端工程/070202_React/070202_知识地图|070202_React知识地图]]

---
title: 告别“代码破窗效应”——React Doctor：给你的 React 项目来一次全面体检
author: 渡一前端每日精选
date:
url: https://mp.weixin.qq.com/s?__biz=MzI2NTQ5NTE4OA==&mid=2247565475&idx=1&sn=9e53c99e98537888881746759ea95f0b&chksm=eb634a71dc8c4ad496793339dd5ff1f51386a6a9163bd6ab7c522cc03fc3fabc1d4d6074c4b7&mpshare=1&scene=24&srcid=0604Ww6HujxrR8rts0A67HXv&sharer_shareinfo=c3efa71d466253ea1bd1580baf20bdcf&sharer_shareinfo_first=c3efa71d466253ea1bd1580baf20bdcf#rd
---

## 写在前面

在日常的 React 业务迭代中，你是否也经历过这样的困境：项目跑了很久，需求越堆越多，代码库的健康状况却变成了一笔糊涂账。谁也不知道代码深处隐藏着多少陈旧的 Effect 逻辑、性能隐患或安全漏洞。

传统的 Code Review 极度依赖人工经验，标准 ESLint 往往触及不到深层架构问题；而如今 AI 编程助手虽然提升了产出速度，但若缺乏专业规则约束，生成的代码也常常埋下隐患。

有没有一种工具，能像体检报告一样，直观地告诉你项目的健康状况？

答案就是 **React Doctor**。

## 一、React Doctor 是什么？

**React Doctor** 是由 Million.js 团队打造的一款开源 CLI 工具，专注于 React 代码质量的全面诊断。

你可以把它理解成一个“全自动的资深 React 技术专家”。只需一个命令，它就能扫描你的代码库，针对**安全性、性能、正确性、架构设计**等多个维度给出深度诊断，并输出一个直观的 0~100 分的健康评分。它不仅能自动识别你的框架环境（Next.js、Vite、Remix 等），还能通过并行运行的 60 余条专业 Lint 规则与死代码扫描，一次性给出可操作的修复建议。

## 二、什么情况下你需要它？

React Doctor 并非锦上添花的玩具，而是直击痛点的生产力工具。以下几种场景，你会尤其需要它：

### 2.1 接手遗留项目 / 代码质量“黑盒化”

当你接手一个老项目时，最头疼的就是不知道里面藏了多少“历史债务”。React Doctor 可以一键扫描出所有潜在问题，让你对项目质量一目了然。

### 2.2 AI 编程助手“放飞自我”

现在的 AI 编程助手虽然高效，但生成的代码经常违反 React 最佳实践——比如在渲染中调用 `setState`、`useEffect` 依赖数组遗漏、将数组索引用作 key 等。React Doctor 就是为此而生的“AI 代码质检员”。

### 2.3 CI/CD 自动化质量把关

在 PR 阶段自动扫描变更代码，发现新引入的问题就立即拦截，避免劣质代码流入主干。React Doctor 原生支持 GitHub Actions 集成，一条配置就能搞定。

### 2.4 团队代码规范落地

当团队内部对“什么是好代码”缺乏统一标准时，React Doctor 提供了量化的评价体系，让代码质量“看得见、比得了”。

## 三、核心检查维度详解

React Doctor 内置了 **60+ 条检测规则**，按问题类型分为以下几大维度：

### 3.1 状态与副作用（State & Effects）

这是 React Doctor 最核心的检查维度之一，专门捕获 Hooks 使用中的典型反模式：

* **`no-derived-useState`**：检测从 props 或其他状态派生新 state 的错误模式。比如 `const [localSearch, setLocalSearch] = useState(searchQuery)` 这种写法，直接用 `useMemo` 或 render 期间计算即可。
* **`useEffect` 依赖问题**：当 useEffect 依赖数组中遗漏了必要的依赖项，规则会立即触发警告。
* **`useEffect` 缺少清理函数**：在 useEffect 中发起数据请求却没有清理逻辑时，会埋下竞态条件的隐患。
* **`useEffect` 中计算派生状态**：本应在 render 期间直接计算的值，却错误地放在了 useEffect 中，这会导致多余的一次渲染。

### 3.2 性能（Performance）

React 的性能问题往往源于不必要的重渲染和错误的记忆化策略：

* **缺少 `useMemo` / `useCallback`**：检测高开销计算或频繁变更的引用值是否缺少记忆化包装。
* **不必要的记忆化**：反之，对简单值使用 `useMemo` / `React.memo` 反而会增加内存开销，React Doctor 同样能识别出来。
* **`no-render-in-render`**：检测在 JSX 中直接调用返回新组件的函数这种反模式，每次渲染都会创建新的组件引用，严重破坏 React 的 diffing 效率。
* **大组件拆分建议**：组件过大时，任何状态变更都会触发整个组件树的重渲染。

### 3.3 架构（Architecture）

关注组件设计和项目结构层面的问题：

* **组件嵌套过深**
* **循环依赖检测**
* **不恰当的组件抽象**
* **状态提升反模式**

### 3.4 正确性（Correctness）

捕获可能导致运行时错误的代码模式：

* **`no-array-index-as-key`**：将数组索引作为 key，在列表项重新排序时会导致渲染错误。
* **对象作为 React 子元素**：直接渲染对象而非字符串或 JSX 会触发 React 运行时错误。
* **条件渲染中的 Hook 调用**：在条件分支中调用 Hook 违反了 React 的 Hook 规则。

### 3.5 安全性（Security）

聚焦应用安全漏洞：

* **`react/no-danger`**：检测 `dangerouslySetInnerHTML` 的使用，避免 XSS 攻击。
* **服务端操作缺少鉴权**：如 Server Action 中没有身份验证检查。
* **敏感信息泄露**：API key、token 等硬编码在源码中的检测。

### 3.6 可访问性（Accessibility）

确保应用对残障用户友好：

* **`jsx-a11y/no-autofocus`**：检测不当的自动聚焦行为。
* **缺少 `prefers-reduced-motion` 检查**：动画未考虑用户的减少动效偏好设置。
* **缺少 aria 属性**：交互元素缺少必要的无障碍标注。

### 3.7 打包体积（Bundle Size）

* **未使用的依赖检测**
* **重复的类型定义**
* **Tree-shaking 优化建议**

### 3.8 死代码检测

React Doctor 会在独立通道中检测以下几类冗余代码：

* **未使用的文件**：源文件目录中未被任何模块引用的“孤儿文件”。
* **未使用的导出**：已导出但无消费者引用的函数、变量、类型。
* **重复文件**：内容高度相似甚至完全一致的重复定义。

## 四、实现原理：AST 级别分析 + 双路并行扫描

### 4.1 AST 分析：不止于字符串匹配

React Doctor 的核心扫描引擎基于 **AST（抽象语法树）** 分析构建。这意味着它能深入理解代码的结构语义，而非仅仅进行字符串模式匹配。

具体流程是：首先将源代码解析为语法树，然后根据预定义的 60 多条规则对树中的节点进行遍历和匹配。每条规则对应一种已知的反模式或不良实践——例如在渲染方法中调用 `setState`，或在 useEffect 依赖数组中遗漏必要的依赖项。

为什么要用 AST 而不是简单的正则匹配？正则表达式只能检测表面的代码模式，容易产生误报或漏报。而 AST 分析能够准确理解代码的语义上下文。比如，一条检测“在渲染中调用 setState”的规则，无论代码经过怎样的格式化、注释包裹或条件包装，都能精准识别，不会漏网。

### 4.2 双路并行扫描，效率拉满

当你运行 React Doctor 时，它会同时启动两个分析通道：

1. **Lint 通道**：执行 60+ 条专业规则，逐条对照 AST 节点进行匹配，覆盖状态、性能、架构、安全等多个维度。
2. **死代码检测通道**：独立扫描未使用的文件、导出、类型和重复内容。

两条通道并行运行，这意味着即使是大型项目，也能在数秒内完成全面扫描。

### 4.3 框架自动适配

React Doctor 在执行扫描前会先检测你的项目环境——识别框架类型（Next.js / Vite / Remix）、React 版本以及编译器配置，然后自动激活与之匹配的规则子集。这意味着同一个工具可以在不同技术栈的项目中通用，零配置即可上手。

### 4.4 加权评分体系

扫描结束后，React Doctor 会根据问题严重程度进行加权计算：

* **75 分及以上** → **Great**（优秀）
* **50 ~ 74 分** → **Needs work**（需要改进）
* **低于 50 分** → **Critical**（危急）

安全和可访问性问题扣分权重较高，而代码风格类问题的扣分权重较低。这种差异化设计确保评分能真实反映代码的综合质量，而不会被大量低优先级的小问题所主导。

## 五、快速上手

### 5.1 一键扫描

在项目根目录执行以下命令即可开始扫描：

```
npx -y react-doctor@latest .
```

你会立即获得一个健康评分以及按维度分类的问题列表。

如果需要查看问题所在的**具体文件和行号**，加上 `--verbose`：

```
npx -y react-doctor@latest . --verbose
```

如果只需要**快速拿到分数**（适合集成到脚本中）：

```
npx -y react-doctor@latest . --score
```

5.2 集成到 GitHub Actions

在 `.github/workflows/react-doctor.yml` 中添加以下配置：

```
name: React Doctor
on:  pull_request:  push:    branches: [main]
permissions:  contents: read  pull-requests: write
jobs:  react-doctor:    runs-on: ubuntu-latest    steps:      - uses: actions/checkout@v5        with:          fetch-depth: 0      - uses: millionco/react-doctor@main        with:          diff: main          github-token: ${{ secrets.GITHUB_TOKEN }}
```

`diff: main` 选项非常实用——它会让 Action **只扫描相对于主分支变更的文件**，这意味着 PR 中只会标记出本次改动引入的新问题，而不会因为仓库中已有的历史问题误伤开发者。

### 5.3 为 AI 编程助手植入“专家规则”

这是 React Doctor 最具前瞻性的功能。你可以将 47+ 条 React 专家级规则转化为 AI 智能体可理解的“技能包”，实现“编写即诊断”的闭环。目前支持 **Cursor、Claude Code、Codex、Windsurf** 等 50+ 种编程工具。

一键安装：

```
npx -y react-doctor@latest install
```

或者通过 curl 直接安装技能包：

```
curl -fsSL https://react.doctor/install-skill.sh | bash
```

5.4 自定义配置

在项目根目录创建 `react-doctor.config.json` 来自定义行为：

```
{  "ignore": {    "rules": ["react/no-danger", "jsx-a11y/no-autofocus"],    "files": ["src/generated/**"],    "overrides": [      {        "files": ["components/modules/diff/**"],        "rules": ["react-doctor/no-array-index-as-key"]      }    ]  }}
```

三层粒度的配置体系：

* **`ignore.rules`**：全局禁用特定规则
* **`ignore.files`**：全局排除特定文件（请谨慎使用）
* **`ignore.overrides`**：仅对匹配的文件禁用指定规则，其他规则不受影响

React Doctor 还会自动遵循 `.gitignore`、`.eslintignore` 等忽略文件，已有的排除逻辑无需重复配置。

### 5.5 完整命令行选项

```
react-doctor [directory] [options]
选项说明：  -v, --version        显示版本号  --no-lint            跳过 lint 检查  --no-dead-code       跳过死代码检测  --verbose            显示问题所在文件和行号  --score              仅输出评分  -y, --yes            跳过交互提示，扫描所有项目  --project <name>     指定要扫描的 workspace 子项目  --diff [base]        仅扫描相对于基准分支变更的文件  --hide-branding      输出纯净 HTML 报告  -h, --help           显示帮助
```

## 六、真实项目评分参考

React Doctor 官方仓库提供了一个知名的开源项目评分排行榜，可以作为你校准预期的参考：

| 项目 | 评分 |
| --- | --- |
| tldraw | 84 |
| excalidraw | 84 |
| twenty | 78 |
| plane | 78 |
| posthog | 72 |
| supabase | 69 |
| sentry | 64 |

即使是 tldraw 和 excalidraw 这样由资深团队维护的项目，评分也只在 **84 分**左右。所以**不要执着于追求满分**——React Doctor 的价值在于帮你发现真正重要的问题，而非制造“完美代码”的焦虑。

## 七、总结

React Doctor 的价值可以概括为三个关键词：**量化、自动化、可操作**。

它将模糊的“代码质量”变成了一个直观的数字；将依赖人工的代码审查变成了自动化的机器扫描；将让人头疼的问题排查变成了带行号和修复建议的可执行清单。

无论是接手遗留项目、规范 AI 辅助编程的输出质量，还是在 CI 流程中建立自动化质量门禁，React Doctor 都是一个值得放入工具箱的选择。

下次在项目根目录，不妨运行一次 `npx -y react-doctor@latest .`，看看你的项目能打多少分？

RECOMMEND

推荐阅读