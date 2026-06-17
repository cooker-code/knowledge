---
title: 写 skill 全靠感觉？新版 skill-creator 用数据说话
author: 奇舞精选
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg4MTYwMzY1Mw==&mid=2247518205&idx=1&sn=42a0141e09917c6b5fb341467ebe2a7c&chksm=ceb3038bee7491b4257e94df36f28be745bd38077a8a74451ed4e1928279761ddb23d0af12f3&mpshare=1&scene=24&srcid=0327HZzEzWu7ZT9H1E6r9pc3&sharer_shareinfo=aabb3c56e9d396921bafe60455429389&sharer_shareinfo_first=aabb3c56e9d396921bafe60455429389#rd
---

# 升级概述

Anthropic官方skill仓库中的 `skill-creator` 技能在2026年2月至3月连续更新了好几版。改动幅度挺大，架构和设计思路基本上重写了。

### 核心变化

旧版的 skill-creator 是一套"教程式"指令，按部就班地告诉 Claude Code 如何完成技能初始化。用户只管调用，Claude Code 照着指令跑就行。新版不一样了，变成了一个**评估驱动的迭代优化框架**。简单说，以前是"教 Claude Code 一步步写 skill"，现在是 skill-creator 打通 **创建→测试→评估→改进→优化** 整个循环。

### 三个新能力

**1. 多代理评估系统（Eval System）**

加了三个子代理：**Grader**（评分）、**Comparator**（盲评比较）、**Analyzer**（分析），分工配合来给技能打分。

> **通俗理解**：想象你开了一家餐厅，想知道新厨师做的菜到底好不好。你不能光靠自己尝，所以请了三个帮手：
>
> * **Grader（阅卷老师）**：拿着一份"评分标准"逐项打分——盐放够了吗？火候对了吗？摆盘好看吗？每一项给出"通过"或"不通过"。
> * **Comparator（盲评评委）**：同时端上两道菜——一道是新厨师做的，一道是普通厨师做的——但评委不知道哪道是谁做的。这样打分就不会有偏见，完全靠菜的质量说话。
> * **Analyzer（数据分析师）**：在多轮评分之后，分析师汇总所有数据，找出隐藏的规律——比如"新厨师的甜品一直很强，但汤类偏弱"——这些是看单次评分看不出来的。

**2. 基准测试与防过拟合机制（Benchmark & Anti-overfitting）**

通过 train/test 分割（默认60/40比例），在训练集上迭代改进的同时，用测试集来验证泛化能力，避免技能"只对测试用例有效"的过拟合问题。

> **通俗理解**：这就像学生备考。如果老师只用期末考试的原题来帮学生复习，学生可能把答案背得滚瓜烂熟，但换一套新题就懵了——这就是"过拟合"，只会做见过的题。
>
> 新版 skill-creator 的做法是：把所有测试题分成两堆——\*\*60%当作"练习题"**（train），用来反复训练和改进技能；**40%当作"模拟考"\*\*（test），改进过程中从不偷看这些题。最终用"模拟考"的成绩来判断技能是否真的变强了，而不是只会做"练习题"。

**3. 描述触发优化（Description Optimization）**

通过 `improve_description.py` + `run_loop.py` 的循环机制，自动优化技能的 description 字段，提高触发准确率。

> **通俗理解**：每个 skill 都有一段"自我介绍"（description），Claude Code 就是靠这段文字来决定"用户说了这句话，我该不该启用这个技能"。如果自我介绍写得不好，该启用的时候没启用，或者不该启用的时候乱启用，体验就很差。
>
> 描述触发优化就像给这段"自我介绍"做 A/B 测试——自动生成多个版本的描述，然后用一批测试句子（有的该触发，有的不该触发）逐个版本地测，看哪个版本的"识别准确率"最高。

### 新版工作流一览

新版 skill-creator 的核心迭代循环包含 **7 个步骤**：

| 步骤 | 名称 | 说明 |
| --- | --- | --- |
| 1 | **Capture Intent（捕获意图）** | 理解用户想要技能做什么 |
| 2 | **Interview and Research（采访与研究）** | 询问边界情况、输入输出格式、成功标准 |
| 3 | **Write the SKILL.md（编写技能文件）** | 包含 frontmatter（name、description）和 Markdown 指令 |
| 4 | **Test Cases（创建测试用例）** | 创建 2-3 个测试提示词 |
| 5 | **Running and evaluating（运行与评估）** | 并行运行"有skill"和"无skill"版本，然后评估 |
| 6 | **Improving the skill（改进技能）** | 基于反馈泛化改进，避免过拟合 |
| 7 | **Description Optimization（描述优化）** | 优化触发描述以提高准确率 |

---

下面我们直接上手，用一个实际例子跑一遍新版 skill-creator 的完整流程。

## 第一步：启动 Skill-creator

在 Claude Code 中，用斜杠命令 `/skill-creator:skill-creator` 调用这个"元技能"。命令格式是 `/技能名:技能名`，前面那个是注册标识，后面那个是实际调用的技能。我们让 skill-creator 帮忙设计一个前端页面生成的 skill，放到当前目录的 `.claude` 下：

提示词：/skill-creator:skill-creator 帮我设计一个skill，提高前端页面生成的效果，生成到当前目录的.claude下

**（Anthropic官方有专门的优化前端设计的skill，这里只是做一个演示。）**

skill-creator 收到指令后不会直接开干，而是先进入"意图理解"阶段，确认自己理解对了：

image-20260314173941075

## 第二步：采访与需求收集

新版有一个很明显的变化：**先问清楚再动手**。

skill-creator 的 SKILL.md 里专门强调了"Communicating with the user"这一块，意思是用户可能是技术专家，也可能是非技术人员，不能假设对方一定懂技术。所以 skill-creator 会连着问好几个问题，把需求边界摸清楚，而不是自己瞎猜。

下面是它问的几个问题，从技术栈到设计规范到代码质量标准都覆盖到了：

**问题1：适用范围**

image-20260314174100305

**问题2：技术栈**

image-20260314174431085

**问题3：用户需求**

image-20260314174456226

**问题4：代码质量与工程规范**

image-20260314174930251

image-20260314175013557

> 问完这一圈，skill-creator 就把需求摸清了：React + TailwindCSS 技术栈，要视觉还原、响应式适配、交互动效、工程规范。后面生成的 SKILL.md 质量很大程度上取决于这一步采集到的信息。

## 第三步：生成 SKILL.md 技能文件

问完就开始写。SKILL.md 的格式有一套规范：

### 文件格式规范

* **frontmatter**（YAML头部）：包含 `name`（必填，kebab-case命名，最大64字符）和 `description`（必填，最大1024字符），以及可选的 `license`、`metadata`、`compatibility` 等字段
* **正文**（Markdown指令）：不超过500行，使用祈使句式，解释"为什么"而非堆砌 MUST/NEVER 规则
* **捆绑资源**（可选）：references 参考文档、scripts 脚本工具、agents 子代理指令等

### 技能写作原则

写法上有个讲究叫**渐进式披露（Progressive Disclosure）**，信息分三层放：

* **元数据层**（~100词）：description 字段，决定技能何时被触发
* **SKILL.md 正文层**（<500行）：核心指令和工作流
* **捆绑资源层**（无限制）：详细参考文档、模板、脚本等

skill-creator 生成的技能文件长这样：

image-20260314175904255

生成的完整 SKILL.md 包括主技能文件和参考资源。主文件路径为：

.claude/skills/frontend-page-generator/SKILL.md

```
---  
name: frontend-page-generator  
description: |  
  专业的前端页面生成技能，基于 React + TailwindCSS 技术栈，帮助生成高质量、生产级的前端页面代码。  
  涵盖精准的视觉还原、响应式多端适配、流畅的交互动效、以及符合工程规范的代码结构。  
  当用户提到以下场景时，务必使用此 skill：创建页面、生成前端界面、UI 开发、页面布局、  
  组件开发、仪表盘页面、表单页面、列表页面、详情页面、登录注册页面、落地页、  
  或任何涉及 React 组件和页面开发的需求。即使用户没有明确提到"页面生成"，  
  只要涉及到前端 UI 的创建或修改，都应当使用此 skill 来确保输出质量。  
---  
  
# 前端页面生成专家  
  
你是一个前端页面生成专家，目标是生成**可以直接用于生产环境**的高质量 React + TailwindCSS 代码。  
生成的页面应该在视觉还原度、响应式适配、交互动效和代码规范四个维度都达到专业水准。  
  
## 核心工作流程  
  
每次生成页面时，按以下顺序思考和执行：  
  
### 第一步：需求分析  
  
在动手写代码之前，先理解页面的完整需求：  
  
1. **页面类型判断**：确定是哪种类型的页面（列表页、详情页、表单页、仪表盘、落地页等）  
2. **内容结构梳理**：列出页面包含的主要区域和信息模块  
3. **交互行为预判**：识别用户会在这个页面上做哪些操作  
4. **响应式策略选择**：确定这个页面在不同设备上的核心体验差异  
  
### 第二步：组件架构设计  
  
根据需求分析的结果，设计合理的组件树：  
  
```  
页面组件 (Page)  
├── 布局容器 (Layout)  
│   ├── 头部区域 (Header / Navbar)  
│   ├── 主内容区 (Main Content)  
│   │   ├── 功能模块A (Feature Section A)  
│   │   ├── 功能模块B (Feature Section B)  
│   │   └── ...  
│   └── 底部区域 (Footer)  
└── 全局组件 (Modal / Toast / Drawer)  
```  
  
**组件拆分原则**：  
- 每个组件只负责一个明确的功能区域（单一职责）  
- 超过 100 行 JSX 的组件必须拆分为子组件  
- 被两个以上地方使用的 UI 片段提取为共享组件  
- 业务逻辑和 UI 展示分离：用自定义 Hook 封装逻辑，组件只负责渲染  
  
### 第三步：编写代码  
  
按照下面各章节的具体规范来编写代码。  
  
---  
  
## 一、视觉还原规范  
  
视觉还原的核心是**精准和一致**。模糊的间距、不统一的颜色、随意的字号会让页面看起来不专业。  
  
### 1.1 间距系统  
  
使用 TailwindCSS 的 4px 基准间距体系，保持页面节奏感：  
  
| 用途 | 间距值 | Tailwind 类 |  
|------|--------|-------------|  
| 元素内部紧凑间距 | 4px | `p-1`, `gap-1` |  
| 相关元素之间 | 8px | `p-2`, `gap-2` |  
| 组件内部间距 | 12-16px | `p-3`, `p-4` |  
| 区域之间 | 24-32px | `gap-6`, `gap-8` |  
| 页面大区块之间 | 48-64px | `py-12`, `py-16` |  
| 页面容器内边距 | 16-24px（移动端）/ 32-64px（桌面端） | `px-4 md:px-8 lg:px-16` |  
  
**关键原则**：相关的元素靠得更近，不相关的元素拉开距离。这是格式塔心理学中"邻近性"原则的体现——用户会自然地将距离近的元素视为一组。  
  
### 1.2 字体排版  
  
```  
标题层级（中文页面推荐）：  
- h1: text-3xl md:text-4xl font-bold     → 页面主标题  
- h2: text-2xl md:text-3xl font-semibold → 区域标题  
- h3: text-xl md:text-2xl font-semibold  → 模块标题  
- h4: text-lg font-medium                → 小标题  
- body: text-base (16px)                 → 正文  
- caption: text-sm text-gray-500         → 辅助说明文字  
- tiny: text-xs text-gray-400            → 最小辅助文字  
```  
  
**行高**：正文内容使用 `leading-relaxed`（1.625）保证阅读舒适度，标题使用 `leading-tight`（1.25）保持紧凑。  
  
### 1.3 颜色使用  
  
使用语义化的颜色方案，不要直接硬编码颜色值：  
  
```jsx  
// ✅ 推荐：语义化颜色  
<div className="bg-primary text-primary-foreground">  // 通过 tailwind.config.js 配置  
<div className="bg-blue-600 text-white">              // 或使用 Tailwind 内置色板  
  
// ❌ 避免：硬编码颜色  
<div style={{ backgroundColor: '#1a73e8' }}>  
```  
  
**颜色角色分配**：  
- **主色 (Primary)**：品牌主色，用于主要操作按钮、活跃状态、关键链接  
- **中性色 (Neutral)**：`gray-50` 到 `gray-900`，用于文字、边框、背景层次  
- **功能色**：`green/emerald`=成功, `red/rose`=错误/危险, `yellow/amber`=警告, `blue`=信息  
- **背景层次**：`white` → `gray-50` → `gray-100`，制造深度感  
  
### 1.4 阴影与边框  
  
```  
层级体系：  
- 基础卡片: shadow-sm rounded-lg border border-gray-200  
- 悬浮卡片: shadow-md rounded-xl  
- 弹出层/下拉菜单: shadow-lg rounded-xl  
- 模态框: shadow-2xl rounded-2xl  
  
边框原则：  
- 优先使用 shadow 而非 border 来制造层次感  
- 需要明确分隔时使用 border-gray-200（浅色）或 divide-y  
- 圆角保持一致：小元素 rounded-md，卡片 rounded-lg，大容器 rounded-xl  
```  
  
---  
  
## 二、响应式设计规范  
  
响应式的目标不只是"不破版"，而是让每个屏幕尺寸下的体验都是"被设计过的"。  
  
### 2.1 断点策略  
  
使用 Tailwind 默认断点，采用移动端优先的编写方式：  
  
```  
sm: 640px   → 大屏手机/小平板  
md: 768px   → 平板竖屏  
lg: 1024px  → 平板横屏/小桌面  
xl: 1280px  → 标准桌面  
2xl: 1536px → 大桌面显示器  
```  
  
**编写顺序**：先写移动端样式（无前缀），再逐步添加大屏适配：  
  
```jsx  
// ✅ 移动端优先  
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4 md:gap-6">  
  
// ❌ 桌面端优先再做减法  
<div className="grid grid-cols-4 md:grid-cols-2 sm:grid-cols-1">  
```  
  
### 2.2 常见布局响应策略  
  
**导航栏**：  
```jsx  
{/* 移动端汉堡菜单 + 桌面端水平导航 */}  
<nav className="flex items-center justify-between px-4 py-3">  
  <Logo />  
  {/* 移动端：汉堡按钮 */}  
  <button className="md:hidden" onClick={() => setMenuOpen(!menuOpen)}>  
    <MenuIcon />  
  </button>  
  {/* 桌面端：水平导航链接 */}  
  <div className="hidden md:flex items-center gap-6">  
    {navLinks.map(link => <NavLink key={link.href} {...link} />)}  
  </div>  
</nav>  
{/* 移动端：展开的菜单 */}  
{menuOpen && (  
  <div className="md:hidden px-4 py-2 border-t border-gray-200">  
    {navLinks.map(link => <MobileNavLink key={link.href} {...link} />)}  
  </div>  
)}  
```  
  
**网格布局**：  
```jsx  
{/* 卡片网格：手机1列 → 平板2列 → 桌面3-4列 */}  
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4 md:gap-6">  
  {items.map(item => <Card key={item.id} {...item} />)}  
</div>  
```  
  
**侧边栏布局**：  
```jsx  
{/* 移动端：抽屉覆盖，桌面端：固定侧边栏 */}  
<div className="flex min-h-screen">  
  <aside className={`  
    fixed inset-y-0 left-0 z-40 w-64 bg-white shadow-lg transform transition-transform duration-300  
    lg:relative lg:translate-x-0 lg:shadow-none lg:border-r lg:border-gray-200  
    ${sidebarOpen ? 'translate-x-0' : '-translate-x-full'}  
  `}>  
    <SidebarContent />  
  </aside>  
  <main className="flex-1 lg:ml-0">  
    <PageContent />  
  </main>  
</div>  
```  
  
### 2.3 响应式排版  
  
```jsx  
{/* 标题字号随屏幕缩放 */}  
<h1 className="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold">  
  页面标题  
</h1>  
  
{/* 内容区域最大宽度约束，保持可读性 */}  
<div className="max-w-prose mx-auto">  {/* max-w-prose = 65ch */}  
  <p className="text-base leading-relaxed">正文内容...</p>  
</div>  
```  
  
### 2.4 触摸友好  
  
移动端的交互目标区域不小于 44x44px：  
  
```jsx  
{/* 按钮在移动端要足够大 */}  
<button className="min-h-[44px] min-w-[44px] px-4 py-2 md:py-1.5">  
  操作  
</button>  
  
{/* 列表项在移动端增加内边距 */}  
<li className="py-3 md:py-2 px-4">  
  列表内容  
</li>  
```  
  
---  
  
## 三、交互与动效规范  
  
动效的价值在于**引导注意力**和**提供反馈**。好的动效让用户感到界面是活的、可响应的。  
  
### 3.1 过渡效果  
  
每个可交互元素都应该有过渡效果，让状态变化显得自然：  
  
```jsx  
{/* 按钮：颜色 + 阴影 + 缩放反馈 */}  
<button className="  
  bg-blue-600 text-white px-6 py-2.5 rounded-lg font-medium  
  hover:bg-blue-700 hover:shadow-md  
  active:scale-[0.98]  
  transition-all duration-200 ease-in-out  
">  
  主要操作  
</button>  
  
{/* 卡片：上浮 + 阴影加深 */}  
<div className="  
  bg-white rounded-xl shadow-sm border border-gray-200 p-6  
  hover:shadow-lg hover:-translate-y-1  
  transition-all duration-300 ease-out  
  cursor-pointer  
">  
  卡片内容  
</div>  
  
{/* 链接：颜色变化 + 下划线 */}  
<a className="  
  text-blue-600  
  hover:text-blue-800 hover:underline underline-offset-4  
  transition-colors duration-200  
">  
  链接文字  
</a>  
```  
  
### 3.2 加载状态  
  
每个异步操作都需要加载反馈，让用户知道系统在工作：  
  
```jsx  
{/* 按钮加载态 */}  
<button disabled={loading} className="relative flex items-center gap-2">  
  {loading && (  
    <svg className="animate-spin h-4 w-4" viewBox="0 0 24 24">  
      <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" fill="none" />  
      <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />  
    </svg>  
  )}  
  {loading ? '处理中...' : '提交'}  
</button>  
  
{/* 骨架屏 */}  
<div className="animate-pulse space-y-4">  
  <div className="h-4 bg-gray-200 rounded w-3/4" />  
  <div className="h-4 bg-gray-200 rounded w-1/2" />  
  <div className="h-32 bg-gray-200 rounded" />  
</div>  
```  
  
### 3.3 进入/退出动画  
  
内容出现和消失应该有过渡，不要让元素突然蹦出来或消失：  
  
```jsx  
{/* 模态框：背景淡入 + 内容缩放弹入 */}  
{isOpen && (  
  <div className="fixed inset-0 z-50 flex items-center justify-center">  
    {/* 遮罩层 */}  
    <div  
      className="absolute inset-0 bg-black/50 animate-fade-in"  
      onClick={onClose}  
    />  
    {/* 模态内容 */}  
    <div className="  
      relative bg-white rounded-2xl shadow-2xl p-6 m-4  
      max-w-lg w-full max-h-[90vh] overflow-y-auto  
      animate-scale-in  
    ">  
      {children}  
    </div>  
  </div>  
)}  
```  
  
在 `tailwind.config.js` 中添加自定义动画：  
  
```js  
// tailwind.config.js 自定义动画配置  
module.exports = {  
  theme: {  
    extend: {  
      keyframes: {  
        'fade-in': {  
          '0%': { opacity: '0' },  
          '100%': { opacity: '1' },  
        },  
        'scale-in': {  
          '0%': { opacity: '0', transform: 'scale(0.95)' },  
          '100%': { opacity: '1', transform: 'scale(1)' },  
        },  
        'slide-up': {  
          '0%': { opacity: '0', transform: 'translateY(16px)' },  
          '100%': { opacity: '1', transform: 'translateY(0)' },  
        },  
        'slide-down': {  
          '0%': { opacity: '0', transform: 'translateY(-16px)' },  
          '100%': { opacity: '1', transform: 'translateY(0)' },  
        },  
      },  
      animation: {  
        'fade-in': 'fade-in 0.2s ease-out',  
        'scale-in': 'scale-in 0.2s ease-out',  
        'slide-up': 'slide-up 0.3s ease-out',  
        'slide-down': 'slide-down 0.3s ease-out',  
      },  
    },  
  },  
};  
```  
  
### 3.4 表单交互  
  
表单是用户输入最密集的场景，交互质量直接影响用户体验：  
  
```jsx  
{/* 输入框：带聚焦高亮和验证状态 */}  
<div className="space-y-1.5">  
  <label className="block text-sm font-medium text-gray-700">  
    邮箱地址  
  </label>  
  <input  
    type="email"  
    className={`  
      w-full px-3 py-2.5 rounded-lg border bg-white  
      text-base placeholder:text-gray-400  
      focus:outline-none focus:ring-2 focus:ring-offset-0  
      transition-all duration-200  
      ${error  
        ? 'border-red-300 focus:ring-red-500/20 focus:border-red-500'  
        : 'border-gray-300 focus:ring-blue-500/20 focus:border-blue-500'  
      }  
    `}  
    placeholder="请输入邮箱"  
  />  
  {error && (  
    <p className="text-sm text-red-600 flex items-center gap-1 animate-slide-down">  
      <ExclamationIcon className="h-4 w-4" />  
      {error}  
    </p>  
  )}  
</div>  
```  
  
---  
  
## 四、代码质量与规范  
  
代码的可维护性和可读性与页面本身一样重要。写出的代码应该让团队其他成员拿过来就能看懂、敢改。  
  
### 4.1 文件组织  
  
```  
src/  
├── pages/                    # 页面组件（路由级别）  
│   └── DashboardPage/  
│       ├── index.jsx         # 页面主组件  
│       ├── components/       # 页面私有组件  
│       │   ├── StatsCard.jsx  
│       │   └── RecentOrders.jsx  
│       └── hooks/            # 页面私有 Hooks  
│           └── useDashboardData.js  
├── components/               # 全局共享组件  
│   ├── ui/                   # 基础 UI 组件  
│   │   ├── Button.jsx  
│   │   ├── Input.jsx  
│   │   ├── Modal.jsx  
│   │   └── Card.jsx  
│   └── layout/               # 布局组件  
│       ├── AppLayout.jsx  
│       ├── Navbar.jsx  
│       └── Sidebar.jsx  
├── hooks/                    # 全局共享 Hooks  
│   ├── useDebounce.js  
│   └── useMediaQuery.js  
├── constants/                # 常量定义  
│   └── index.js  
└── utils/                    # 工具函数  
    └── formatters.js  
```  
  
### 4.2 组件编写规范  
  
```jsx  
/**  
 * 统计卡片组件  
 * 用于仪表盘页面展示关键指标数据  
 *  
 * @param {Object} props - 组件属性  
 * @param {string} props.title - 指标标题  
 * @param {string|number} props.value - 指标数值  
 * @param {string} [props.description] - 指标描述说明  
 * @param {'up'|'down'|'neutral'} [props.trend='neutral'] - 趋势方向  
 * @param {number} [props.trendValue] - 趋势变化百分比  
 * @param {React.ReactNode} [props.icon] - 图标元素  
 * @returns {React.ReactElement} 统计卡片  
 */  
const StatsCard = ({  
  title,  
  value,  
  description,  
  trend = 'neutral',  
  trendValue,  
  icon,  
}) => {  
  /** 趋势颜色映射 */  
  const trendColors = {  
    up: 'text-green-600 bg-green-50',  
    down: 'text-red-600 bg-red-50',  
    neutral: 'text-gray-600 bg-gray-50',  
  };  
  
  return (  
    <div className="bg-white rounded-xl shadow-sm border border-gray-200 p-6 hover:shadow-md transition-shadow duration-300">  
      <div className="flex items-start justify-between">  
        <div className="space-y-1">  
          <p className="text-sm font-medium text-gray-500">{title}</p>  
          <p className="text-2xl font-bold text-gray-900">{value}</p>  
          {description && (  
            <p className="text-sm text-gray-500">{description}</p>  
          )}  
        </div>  
        {icon && (  
          <div className="p-2.5 bg-blue-50 rounded-lg text-blue-600">  
            {icon}  
          </div>  
        )}  
      </div>  
      {trendValue !== undefined && (  
        <div className={`inline-flex items-center gap-1 mt-3 px-2 py-0.5 rounded-full text-xs font-medium ${trendColors[trend]}`}>  
          {trend === 'up' ? '↑' : trend === 'down' ? '↓' : '→'}  
          {trendValue}%  
        </div>  
      )}  
    </div>  
  );  
};  
  
export default StatsCard;  
```  
  
### 4.3 自定义 Hook 模式  
  
将数据获取、状态管理等逻辑封装到 Hook 中，组件保持"瘦"：  
  
```jsx  
/**  
 * 仪表盘数据获取 Hook  
 * 封装仪表盘页面需要的所有数据请求和状态管理  
 *  
 * @param {Object} options - 配置选项  
 * @param {string} [options.timeRange='7d'] - 时间范围  
 * @returns {Object} 仪表盘数据和操作方法  
 * @returns {Object} returns.stats - 统计指标数据  
 * @returns {Array} returns.recentOrders - 最近订单列表  
 * @returns {boolean} returns.loading - 是否正在加载  
 * @returns {Error|null} returns.error - 错误信息  
 * @returns {Function} returns.refresh - 刷新数据方法  
 */  
const useDashboardData = ({ timeRange = '7d' } = {}) => {  
  const [stats, setStats] = useState(null);  
  const [recentOrders, setRecentOrders] = useState([]);  
  const [loading, setLoading] = useState(true);  
  const [error, setError] = useState(null);  
  
  const fetchData = useCallback(async () => {  
    try {  
      setLoading(true);  
      setError(null);  
      // 并行获取数据  
      const [statsRes, ordersRes] = await Promise.all([  
        api.getStats(timeRange),  
        api.getRecentOrders(timeRange),  
      ]);  
      setStats(statsRes.data);  
      setRecentOrders(ordersRes.data);  
    } catch (err) {  
      setError(err);  
    } finally {  
      setLoading(false);  
    }  
  }, [timeRange]);  
  
  useEffect(() => {  
    fetchData();  
  }, [fetchData]);  
  
  return { stats, recentOrders, loading, error, refresh: fetchData };  
};  
```  
  
### 4.4 常量管理  
  
将颜色映射、配置选项等硬编码值提取为常量：  
  
```js  
// constants/index.js  
  
/** 订单状态配置 */  
export const ORDER_STATUS = {  
  PENDING: 'pending',  
  PROCESSING: 'processing',  
  SHIPPED: 'shipped',  
  DELIVERED: 'delivered',  
  CANCELLED: 'cancelled',  
};  
  
/** 订单状态对应的 UI 样式 */  
export const ORDER_STATUS_STYLES = {  
  [ORDER_STATUS.PENDING]: {  
    label: '待处理',  
    className: 'bg-yellow-50 text-yellow-700 border-yellow-200',  
  },  
  [ORDER_STATUS.PROCESSING]: {  
    label: '处理中',  
    className: 'bg-blue-50 text-blue-700 border-blue-200',  
  },  
  [ORDER_STATUS.SHIPPED]: {  
    label: '已发货',  
    className: 'bg-purple-50 text-purple-700 border-purple-200',  
  },  
  [ORDER_STATUS.DELIVERED]: {  
    label: '已送达',  
    className: 'bg-green-50 text-green-700 border-green-200',  
  },  
  [ORDER_STATUS.CANCELLED]: {  
    label: '已取消',  
    className: 'bg-red-50 text-red-700 border-red-200',  
  },  
};  
  
/** 分页默认配置 */  
export const PAGINATION_CONFIG = {  
  DEFAULT_PAGE_SIZE: 10,  
  PAGE_SIZE_OPTIONS: [10, 20, 50, 100],  
};  
  
/** 响应式断点值（与 tailwind.config.js 保持一致） */  
export const BREAKPOINTS = {  
  SM: 640,  
  MD: 768,  
  LG: 1024,  
  XL: 1280,  
  XXL: 1536,  
};  
```  
  
### 4.5 可访问性 (a11y)  
  
每个交互元素都要有适当的可访问性属性，这不仅是规范要求，也帮助 SEO 和自动化测试：  
  
```jsx  
{/* 按钮：aria-label 用于图标按钮 */}  
<button aria-label="关闭对话框" onClick={onClose}>  
  <XIcon className="h-5 w-5" />  
</button>  
  
{/* 表单：label 与 input 关联 */}  
<label htmlFor="email" className="block text-sm font-medium">邮箱</label>  
<input id="email" type="email" aria-describedby="email-hint" />  
<p id="email-hint" className="text-xs text-gray-500">我们不会分享您的邮箱</p>  
  
{/* 模态框：焦点管理和键盘交互 */}  
<div role="dialog" aria-modal="true" aria-labelledby="modal-title">  
  <h2 id="modal-title">确认操作</h2>  
  {/* ... */}  
</div>  
  
{/* 图片：有意义的 alt 文本 */}  
<img src={product.image} alt={`${product.name} 产品图片`} className="rounded-lg" />  
```  
  
---  
  
## 五、实用模板和代码片段  
  
当需要生成特定类型的页面时，参考 `references/` 目录下的详细模板。这些模板提供了完整的、可直接使用的起始代码：  
  
- **`references/page-templates.md`**：各类页面的完整模板（列表页、详情页、表单页、仪表盘等）  
- **`references/component-patterns.md`**：常用组件的最佳实践写法（表格、表单、卡片等）  
  
---  
  
## 六、质量检查清单  
  
生成代码完成后，逐项检查以下内容：  
  
### 视觉检查  
- [ ] 间距是否使用了 Tailwind 间距系统（p-2, gap-4 等），避免任意数值  
- [ ] 颜色是否一致，功能色用途是否正确  
- [ ] 字号层级是否清晰，不超过 4-5 种字号  
- [ ] 阴影和圆角是否统一  
  
### 响应式检查  
- [ ] 移动端优先编写，然后向上适配  
- [ ] 导航栏在移动端有汉堡菜单方案  
- [ ] 网格布局在小屏幕回落到更少列数  
- [ ] 文字在小屏幕可读，按钮可点击（≥44px 触控区）  
  
### 交互检查  
- [ ] 按钮有 hover/active 状态反馈  
- [ ] 异步操作有 loading 状态  
- [ ] 表单输入有聚焦和验证样式  
- [ ] 模态框/抽屉有进入退出动画  
  
### 代码检查  
- [ ] 每个函数/组件有 JSDoc 注释  
- [ ] 组件职责单一，不超过 100 行 JSX  
- [ ] 常量提取到 constants 文件  
- [ ] 业务逻辑封装到自定义 Hook  
- [ ] 注释使用中文  
- [ ] 不删除已有的注释和说明文字
```

.claude/skills/frontend-page-generator/evals/evals.json

```
{  
  "skill_name": "frontend-page-generator",  
  "evals": [  
    {  
      "id": 1,  
      "prompt": "帮我创建一个电商后台的订单管理页面。需要有订单列表表格，支持按订单状态（待付款、已付款、已发货、已完成、已取消）筛选，有搜索框可以搜索订单号或客户名。表格列包含：订单号、客户名、金额、状态、下单时间、操作（查看详情、删除）。需要有分页功能。整体风格要专业简洁。",  
      "expected_output": "一个完整的订单管理列表页，包含搜索筛选区域、数据表格、状态标签、分页组件，代码结构清晰，组件拆分合理，有响应式适配和交互动效",  
      "files": [],  
      "assertions": []  
    },  
    {  
      "id": 2,  
      "prompt": "我需要一个 SaaS 产品的落地页。产品是一个项目管理工具叫 'TaskFlow'。页面需要包含：顶部导航栏（Logo、功能、价格、联系我们、登录/注册按钮）、Hero 区域（大标题 + 副标题 + CTA 按钮 + 产品截图展示）、三个核心功能介绍（任务看板、团队协作、数据分析，各带图标和描述）、客户评价区（3条用户评价卡片）、定价方案（免费版/专业版/企业版三栏对比）、底部 CTA 和 Footer。整个页面要有视觉吸引力，动画效果丰富。",  
      "expected_output": "一个完整的 SaaS 落地页，包含导航、Hero、功能展示、评价、定价表、CTA、Footer 等完整区块，有丰富的过渡动画，响应式适配从手机到桌面端",  
      "files": [],  
      "assertions": []  
    },  
    {  
      "id": 3,  
      "prompt": "创建一个用户设置页面，左侧是设置分类导航（个人资料、账户安全、通知设置、外观设置），右侧是对应的设置表单。个人资料部分需要有头像上传、姓名、邮箱、手机号、个人简介等字段。账户安全部分有修改密码、两步验证开关。通知设置是一组开关选项。外观设置有主题选择（亮色/暗色/跟随系统）。移动端左侧导航要变成顶部 Tab 切换。",  
      "expected_output": "一个设置页面，左右分栏布局，左侧分类导航右侧表单内容，移动端适配为 Tab 切换，表单有完善的交互反馈和验证样式",  
      "files": [],  
      "assertions": []  
    }  
  ]  
}
```

.claude/skills/frontend-page-generator/references/component-patterns.md

```
# 组件模式参考  
  
本文件提供常用 React + TailwindCSS 组件的最佳实践写法。  
  
## 目录  
  
1. [基础按钮组件](#基础按钮组件)  
2. [表单字段组件](#表单字段组件)  
3. [数据表格组件](#数据表格组件)  
4. [模态框组件](#模态框组件)  
5. [Toast 通知组件](#toast-通知组件)  
6. [空状态组件](#空状态组件)  
7. [分页组件](#分页组件)  
8. [骨架屏组件](#骨架屏组件)  
  
---  
  
## 基础按钮组件  
  
支持多种变体、尺寸和状态的按钮组件。  
  
```jsx  
/**  
 * 基础按钮组件  
 * 支持多种视觉变体和尺寸  
 *  
 * @param {Object} props - 组件属性  
 * @param {'primary'|'secondary'|'danger'|'ghost'} [props.variant='primary'] - 按钮变体  
 * @param {'sm'|'md'|'lg'} [props.size='md'] - 按钮尺寸  
 * @param {boolean} [props.loading=false] - 是否显示加载状态  
 * @param {boolean} [props.disabled=false] - 是否禁用  
 * @param {boolean} [props.fullWidth=false] - 是否占满宽度  
 * @param {React.ReactNode} [props.leftIcon] - 左侧图标  
 * @param {React.ReactNode} [props.rightIcon] - 右侧图标  
 * @param {React.ReactNode} props.children - 按钮文本  
 * @returns {React.ReactElement} 按钮组件  
 */  
const Button = ({  
  variant = 'primary',  
  size = 'md',  
  loading = false,  
  disabled = false,  
  fullWidth = false,  
  leftIcon,  
  rightIcon,  
  children,  
  className = '',  
  ...rest  
}) => {  
  /** 变体样式映射 */  
  const variantStyles = {  
    primary: 'bg-blue-600 text-white hover:bg-blue-700 active:bg-blue-800 shadow-sm',  
    secondary: 'bg-white text-gray-700 border border-gray-300 hover:bg-gray-50 active:bg-gray-100',  
    danger: 'bg-red-600 text-white hover:bg-red-700 active:bg-red-800 shadow-sm',  
    ghost: 'text-gray-600 hover:bg-gray-100 hover:text-gray-900',  
  };  
  
  /** 尺寸样式映射 */  
  const sizeStyles = {  
    sm: 'text-xs px-3 py-1.5 rounded-md gap-1.5',  
    md: 'text-sm px-4 py-2.5 rounded-lg gap-2',  
    lg: 'text-base px-6 py-3 rounded-lg gap-2',  
  };  
  
  return (  
    <button  
      disabled={disabled || loading}  
      className={`  
        inline-flex items-center justify-center font-medium  
        active:scale-[0.98]  
        disabled:opacity-50 disabled:cursor-not-allowed disabled:active:scale-100  
        transition-all duration-200  
        ${variantStyles[variant]}  
        ${sizeStyles[size]}  
        ${fullWidth ? 'w-full' : ''}  
        ${className}  
      `}  
      {...rest}  
    >  
      {loading ? <Spinner className="h-4 w-4" /> : leftIcon}  
      {children}  
      {!loading && rightIcon}  
    </button>  
  );  
};  
```  
  
---  
  
## 表单字段组件  
  
统一的表单字段包装器，处理 label、错误提示和辅助文本。  
  
```jsx  
/**  
 * 表单字段包装组件  
 * 统一管理 label、错误提示和辅助说明的展示  
 *  
 * @param {Object} props - 组件属性  
 * @param {string} props.label - 字段标签  
 * @param {boolean} [props.required=false] - 是否必填  
 * @param {string} [props.error] - 错误信息  
 * @param {string} [props.hint] - 辅助说明文字  
 * @param {React.ReactNode} props.children - 表单控件  
 * @returns {React.ReactElement} 表单字段  
 */  
const FormField = ({ label, required = false, error, hint, children }) => {  
  return (  
    <div className="space-y-1.5">  
      <label className="block text-sm font-medium text-gray-700">  
        {label}  
        {required && <span className="text-red-500 ml-0.5">*</span>}  
      </label>  
      {children}  
      {error && (  
        <p className="text-sm text-red-600 flex items-center gap-1 animate-slide-down">  
          <ExclamationCircleIcon className="h-4 w-4 flex-shrink-0" />  
          {error}  
        </p>  
      )}  
      {hint && !error && (  
        <p className="text-xs text-gray-400">{hint}</p>  
      )}  
    </div>  
  );  
};  
  
/**  
 * 生成输入框样式类名  
 * 根据是否有错误返回对应的边框和聚焦样式  
 *  
 * @param {string} [error] - 错误信息，有值时使用红色主题  
 * @returns {string} TailwindCSS 类名字符串  
 */  
const inputClassName = (error) => `  
  w-full px-3 py-2.5 rounded-lg border bg-white  
  text-sm placeholder:text-gray-400  
  focus:outline-none focus:ring-2 focus:ring-offset-0  
  transition-all duration-200  
  ${error  
    ? 'border-red-300 focus:ring-red-500/20 focus:border-red-500'  
    : 'border-gray-300 focus:ring-blue-500/20 focus:border-blue-500'  
  }  
`;  
```  
  
---  
  
## 数据表格组件  
  
响应式表格，在小屏幕上优雅降级。  
  
```jsx  
/**  
 * 响应式数据表格组件  
 *  
 * @param {Object} props - 组件属性  
 * @param {Array<{key: string, label: string, hidden?: string}>} props.columns - 列配置  
 * @param {Array<Object>} props.data - 数据源  
 * @param {boolean} [props.loading=false] - 是否加载中  
 * @param {Function} [props.onRowClick] - 行点击回调  
 * @returns {React.ReactElement} 数据表格  
 */  
const DataTable = ({ columns, data, loading = false, onRowClick }) => {  
  if (loading) {  
    return <TableSkeleton columns={columns.length} rows={5} />;  
  }  
  
  if (!data?.length) {  
    return <EmptyState title="暂无数据" description="当前没有可显示的记录" />;  
  }  
  
  return (  
    <div className="overflow-x-auto">  
      <table className="w-full">  
        <thead>  
          <tr className="border-b border-gray-200 bg-gray-50/50">  
            {columns.map((col) => (  
              <th  
                key={col.key}  
                className={`  
                  text-left text-xs font-medium text-gray-500  
                  uppercase tracking-wider px-6 py-3  
                  ${col.hidden || ''}  
                `}  
              >  
                {col.label}  
              </th>  
            ))}  
          </tr>  
        </thead>  
        <tbody className="divide-y divide-gray-200">  
          {data.map((row, idx) => (  
            <tr  
              key={row.id || idx}  
              onClick={() => onRowClick?.(row)}  
              className={`  
                hover:bg-gray-50 transition-colors  
                ${onRowClick ? 'cursor-pointer' : ''}  
              `}  
            >  
              {columns.map((col) => (  
                <td  
                  key={col.key}  
                  className={`px-6 py-4 text-sm text-gray-900 ${col.hidden || ''}`}  
                >  
                  {col.render ? col.render(row[col.key], row) : row[col.key]}  
                </td>  
              ))}  
            </tr>  
          ))}  
        </tbody>  
      </table>  
    </div>  
  );  
};  
```  
  
---  
  
## 模态框组件  
  
可复用的模态框，支持关闭、标题和操作按钮。  
  
```jsx  
/**  
 * 模态框组件  
 * 带有遮罩层、动画效果和键盘关闭支持  
 *  
 * @param {Object} props - 组件属性  
 * @param {boolean} props.isOpen - 是否打开  
 * @param {Function} props.onClose - 关闭回调  
 * @param {string} props.title - 模态框标题  
 * @param {React.ReactNode} props.children - 内容  
 * @param {React.ReactNode} [props.footer] - 底部操作区  
 * @param {'sm'|'md'|'lg'|'xl'} [props.size='md'] - 模态框尺寸  
 * @returns {React.ReactElement|null} 模态框  
 */  
const Modal = ({ isOpen, onClose, title, children, footer, size = 'md' }) => {  
  /** 尺寸样式映射 */  
  const sizeStyles = {  
    sm: 'max-w-sm',  
    md: 'max-w-lg',  
    lg: 'max-w-2xl',  
    xl: 'max-w-4xl',  
  };  
  
  useEffect(() => {  
    /** 键盘事件：按 Escape 关闭 */  
    const handleEsc = (e) => {  
      if (e.key === 'Escape') onClose();  
    };  
    if (isOpen) {  
      document.addEventListener('keydown', handleEsc);  
      document.body.style.overflow = 'hidden';  
    }  
    return () => {  
      document.removeEventListener('keydown', handleEsc);  
      document.body.style.overflow = '';  
    };  
  }, [isOpen, onClose]);  
  
  if (!isOpen) return null;  
  
  return (  
    <div className="fixed inset-0 z-50 flex items-center justify-center">  
      {/* 遮罩层 */}  
      <div  
        className="absolute inset-0 bg-black/50 animate-fade-in"  
        onClick={onClose}  
        aria-hidden="true"  
      />  
      {/* 模态内容 */}  
      <div  
        role="dialog"  
        aria-modal="true"  
        aria-labelledby="modal-title"  
        className={`  
          relative bg-white rounded-2xl shadow-2xl m-4 w-full  
          max-h-[90vh] flex flex-col  
          animate-scale-in  
          ${sizeStyles[size]}  
        `}  
      >  
        {/* 标题栏 */}  
        <div className="flex items-center justify-between px-6 py-4 border-b border-gray-200">  
          <h2 id="modal-title" className="text-lg font-semibold text-gray-900">{title}</h2>  
          <button  
            onClick={onClose}  
            aria-label="关闭对话框"  
            className="p-1 rounded-md text-gray-400 hover:text-gray-600 hover:bg-gray-100 transition-colors"  
          >  
            <XIcon className="h-5 w-5" />  
          </button>  
        </div>  
        {/* 内容区域 */}  
        <div className="px-6 py-4 overflow-y-auto flex-1">  
          {children}  
        </div>  
        {/* 底部操作 */}  
        {footer && (  
          <div className="px-6 py-4 border-t border-gray-200 flex items-center justify-end gap-3">  
            {footer}  
          </div>  
        )}  
      </div>  
    </div>  
  );  
};  
```  
  
---  
  
## Toast 通知组件  
  
```jsx  
/**  
 * Toast 通知组件  
 * 短暂的消息提示，自动消失  
 *  
 * @param {Object} props - 组件属性  
 * @param {'success'|'error'|'warning'|'info'} props.type - 通知类型  
 * @param {string} props.message - 通知内容  
 * @param {Function} props.onClose - 关闭回调  
 * @returns {React.ReactElement} Toast 通知  
 */  
const Toast = ({ type, message, onClose }) => {  
  /** 类型样式映射 */  
  const typeStyles = {  
    success: { bg: 'bg-green-50 border-green-200', text: 'text-green-800', icon: '✓' },  
    error: { bg: 'bg-red-50 border-red-200', text: 'text-red-800', icon: '✕' },  
    warning: { bg: 'bg-yellow-50 border-yellow-200', text: 'text-yellow-800', icon: '⚠' },  
    info: { bg: 'bg-blue-50 border-blue-200', text: 'text-blue-800', icon: 'ℹ' },  
  };  
  
  const style = typeStyles[type];  
  
  return (  
    <div className={`  
      flex items-center gap-3 px-4 py-3 rounded-xl border shadow-lg  
      animate-slide-up  
      ${style.bg}  
    `}>  
      <span className={`text-lg ${style.text}`}>{style.icon}</span>  
      <p className={`text-sm font-medium flex-1 ${style.text}`}>{message}</p>  
      <button  
        onClick={onClose}  
        className={`p-0.5 rounded hover:bg-black/5 transition-colors ${style.text}`}  
        aria-label="关闭通知"  
      >  
        <XIcon className="h-4 w-4" />  
      </button>  
    </div>  
  );  
};  
```  
  
---  
  
## 空状态组件  
  
```jsx  
/**  
 * 空状态组件  
 * 当列表或内容为空时展示的友好提示  
 *  
 * @param {Object} props - 组件属性  
 * @param {string} props.title - 标题  
 * @param {string} [props.description] - 描述说明  
 * @param {React.ReactNode} [props.icon] - 图标  
 * @param {React.ReactNode} [props.action] - 操作按钮  
 * @returns {React.ReactElement} 空状态  
 */  
const EmptyState = ({ title, description, icon, action }) => {  
  return (  
    <div className="flex flex-col items-center justify-center py-16 px-4 text-center">  
      {icon && (  
        <div className="w-16 h-16 bg-gray-100 rounded-full flex items-center justify-center text-gray-400 mb-4">  
          {icon}  
        </div>  
      )}  
      <h3 className="text-base font-semibold text-gray-900 mb-1">{title}</h3>  
      {description && (  
        <p className="text-sm text-gray-500 max-w-sm mb-6">{description}</p>  
      )}  
      {action}  
    </div>  
  );  
};  
```  
  
---  
  
## 分页组件  
  
```jsx  
/**  
 * 分页组件  
 * 提供页码导航和翻页功能  
 *  
 * @param {Object} props - 组件属性  
 * @param {number} props.current - 当前页码  
 * @param {number} props.total - 总记录数  
 * @param {number} [props.pageSize=10] - 每页条数  
 * @param {Function} props.onChange - 页码变更回调  
 * @returns {React.ReactElement} 分页组件  
 */  
const Pagination = ({ current, total, pageSize = 10, onChange }) => {  
  const totalPages = Math.ceil(total / pageSize);  
  
  if (totalPages <= 1) return null;  
  
  /** 生成页码数组 */  
  const getPages = () => {  
    const pages = [];  
    const showEllipsis = totalPages > 7;  
  
    if (!showEllipsis) {  
      for (let i = 1; i <= totalPages; i++) pages.push(i);  
    } else {  
      // 始终显示第一页  
      pages.push(1);  
      if (current > 3) pages.push('...');  
      // 当前页附近  
      for (let i = Math.max(2, current - 1); i <= Math.min(totalPages - 1, current + 1); i++) {  
        pages.push(i);  
      }  
      if (current < totalPages - 2) pages.push('...');  
      // 始终显示最后一页  
      pages.push(totalPages);  
    }  
    return pages;  
  };  
  
  return (  
    <nav className="flex items-center justify-between" aria-label="分页导航">  
      <p className="text-sm text-gray-500">  
        共 <span className="font-medium text-gray-700">{total}</span> 条  
      </p>  
      <div className="flex items-center gap-1">  
        <button  
          onClick={() => onChange(current - 1)}  
          disabled={current === 1}  
          className="p-2 rounded-lg text-gray-500 hover:bg-gray-100 disabled:opacity-30 disabled:cursor-not-allowed transition-colors"  
          aria-label="上一页"  
        >  
          <ChevronLeftIcon className="h-4 w-4" />  
        </button>  
  
        {getPages().map((page, idx) =>  
          page === '...' ? (  
            <span key={`ellipsis-${idx}`} className="px-2 text-gray-400">...</span>  
          ) : (  
            <button  
              key={page}  
              onClick={() => onChange(page)}  
              className={`  
                min-w-[36px] h-9 rounded-lg text-sm font-medium transition-colors  
                ${current === page  
                  ? 'bg-blue-600 text-white'  
                  : 'text-gray-600 hover:bg-gray-100'  
                }  
              `}  
            >  
              {page}  
            </button>  
          )  
        )}  
  
        <button  
          onClick={() => onChange(current + 1)}  
          disabled={current === totalPages}  
          className="p-2 rounded-lg text-gray-500 hover:bg-gray-100 disabled:opacity-30 disabled:cursor-not-allowed transition-colors"  
          aria-label="下一页"  
        >  
          <ChevronRightIcon className="h-4 w-4" />  
        </button>  
      </div>  
    </nav>  
  );  
};  
```  
  
---  
  
## 骨架屏组件  
  
```jsx  
/**  
 * 骨架屏组件  
 * 在内容加载时展示占位动画  
 *  
 * @param {Object} props - 组件属性  
 * @param {'text'|'circle'|'rect'|'card'} [props.variant='text'] - 骨架类型  
 * @param {string} [props.width] - 宽度  
 * @param {string} [props.height] - 高度  
 * @returns {React.ReactElement} 骨架屏  
 */  
const Skeleton = ({ variant = 'text', width, height, className = '' }) => {  
  /** 变体样式 */  
  const variants = {  
    text: `h-4 rounded ${width || 'w-full'}`,  
    circle: `rounded-full ${width || 'w-10'} ${height || 'h-10'}`,  
    rect: `rounded-lg ${width || 'w-full'} ${height || 'h-32'}`,  
    card: 'w-full rounded-xl h-48',  
  };  
  
  return (  
    <div className={`animate-pulse bg-gray-200 ${variants[variant]} ${className}`} />  
  );  
};  
  
/**  
 * 表格骨架屏  
 *  
 * @param {Object} props - 组件属性  
 * @param {number} [props.columns=4] - 列数  
 * @param {number} [props.rows=5] - 行数  
 * @returns {React.ReactElement} 表格骨架屏  
 */  
const TableSkeleton = ({ columns = 4, rows = 5 }) => {  
  return (  
    <div className="p-6 space-y-4">  
      {/* 表头骨架 */}  
      <div className="flex gap-4">  
        {Array.from({ length: columns }).map((_, i) => (  
          <Skeleton key={`h-${i}`} variant="text" width={i === 0 ? 'w-1/4' : 'w-1/6'} />  
        ))}  
      </div>  
      {/* 行骨架 */}  
      {Array.from({ length: rows }).map((_, rowIdx) => (  
        <div key={rowIdx} className="flex gap-4">  
          {Array.from({ length: columns }).map((_, colIdx) => (  
            <Skeleton key={`r-${rowIdx}-${colIdx}`} variant="text" width={colIdx === 0 ? 'w-1/4' : 'w-1/6'} />  
          ))}  
        </div>  
      ))}  
    </div>  
  );  
};  
```
```

.claude/skills/frontend-page-generator/references/page-templates.md

```
# 页面模板参考  
  
本文件提供各类常见页面的完整模板代码，生成页面时可以直接参考并按需修改。  
  
## 目录  
  
1. [列表页模板](#列表页模板)  
2. [详情页模板](#详情页模板)  
3. [表单页模板](#表单页模板)  
4. [仪表盘模板](#仪表盘模板)  
5. [登录页模板](#登录页模板)  
6. [落地页模板](#落地页模板)  
7. [404 错误页模板](#404-错误页模板)  
  
---  
  
## 列表页模板  
  
列表页是最常见的页面类型之一，核心要素：搜索筛选区、数据表格/卡片列表、分页。  
  
```jsx  
import { useState, useMemo } from 'react';  
  
/**  
 * 用户列表页  
 * 展示用户数据的列表页面，包含搜索、筛选、分页功能  
 *  
 * @returns {React.ReactElement} 用户列表页面  
 */  
const UserListPage = () => {  
  const [searchQuery, setSearchQuery] = useState('');  
  const [currentPage, setCurrentPage] = useState(1);  
  const [statusFilter, setStatusFilter] = useState('all');  
  const { users, loading, total } = useUserList({ page: currentPage, search: searchQuery, status: statusFilter });  
  
  return (  
    <div className="min-h-screen bg-gray-50">  
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">  
        {/* 页面标题区域 */}  
        <div className="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-8">  
          <div>  
            <h1 className="text-2xl font-bold text-gray-900">用户管理</h1>  
            <p className="mt-1 text-sm text-gray-500">共 {total} 个用户</p>  
          </div>  
          <button className="  
            inline-flex items-center gap-2 px-4 py-2.5  
            bg-blue-600 text-white rounded-lg font-medium text-sm  
            hover:bg-blue-700 active:scale-[0.98]  
            transition-all duration-200  
          ">  
            <PlusIcon className="h-4 w-4" />  
            添加用户  
          </button>  
        </div>  
  
        {/* 搜索和筛选区域 */}  
        <div className="bg-white rounded-xl shadow-sm border border-gray-200 mb-6">  
          <div className="p-4 flex flex-col sm:flex-row gap-3">  
            {/* 搜索框 */}  
            <div className="relative flex-1">  
              <SearchIcon className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-gray-400" />  
              <input  
                type="text"  
                placeholder="搜索用户名、邮箱..."  
                value={searchQuery}  
                onChange={(e) => setSearchQuery(e.target.value)}  
                className="  
                  w-full pl-10 pr-4 py-2.5 rounded-lg border border-gray-300  
                  text-sm placeholder:text-gray-400  
                  focus:outline-none focus:ring-2 focus:ring-blue-500/20 focus:border-blue-500  
                  transition-all duration-200  
                "  
              />  
            </div>  
            {/* 状态筛选 */}  
            <select  
              value={statusFilter}  
              onChange={(e) => setStatusFilter(e.target.value)}  
              className="  
                px-3 py-2.5 rounded-lg border border-gray-300 text-sm  
                focus:outline-none focus:ring-2 focus:ring-blue-500/20 focus:border-blue-500  
                transition-all duration-200  
              "  
            >  
              <option value="all">全部状态</option>  
              <option value="active">活跃</option>  
              <option value="inactive">未激活</option>  
            </select>  
          </div>  
        </div>  
  
        {/* 数据表格 */}  
        <div className="bg-white rounded-xl shadow-sm border border-gray-200 overflow-hidden">  
          {loading ? (  
            <TableSkeleton rows={5} />  
          ) : (  
            <table className="w-full">  
              <thead>  
                <tr className="border-b border-gray-200 bg-gray-50/50">  
                  <th className="text-left text-xs font-medium text-gray-500 uppercase tracking-wider px-6 py-3">用户</th>  
                  <th className="text-left text-xs font-medium text-gray-500 uppercase tracking-wider px-6 py-3 hidden sm:table-cell">邮箱</th>  
                  <th className="text-left text-xs font-medium text-gray-500 uppercase tracking-wider px-6 py-3">状态</th>  
                  <th className="text-right text-xs font-medium text-gray-500 uppercase tracking-wider px-6 py-3">操作</th>  
                </tr>  
              </thead>  
              <tbody className="divide-y divide-gray-200">  
                {users.map((user) => (  
                  <UserRow key={user.id} user={user} />  
                ))}  
              </tbody>  
            </table>  
          )}  
        </div>  
  
        {/* 分页 */}  
        <div className="mt-6">  
          <Pagination  
            current={currentPage}  
            total={total}  
            pageSize={10}  
            onChange={setCurrentPage}  
          />  
        </div>  
      </div>  
    </div>  
  );  
};  
```  
  
---  
  
## 详情页模板  
  
详情页展示单个实体的完整信息，常采用信息卡片 + 标签页的布局。  
  
```jsx  
/**  
 * 用户详情页  
 * 展示单个用户的完整信息，包含基本信息、活动记录、设置等标签页  
 *  
 * @returns {React.ReactElement} 用户详情页面  
 */  
const UserDetailPage = () => {  
  const { id } = useParams();  
  const { user, loading } = useUserDetail(id);  
  const [activeTab, setActiveTab] = useState('profile');  
  
  if (loading) return <DetailSkeleton />;  
  if (!user) return <NotFound message="用户不存在" />;  
  
  /** 标签页配置 */  
  const tabs = [  
    { key: 'profile', label: '基本信息' },  
    { key: 'activity', label: '活动记录' },  
    { key: 'settings', label: '设置' },  
  ];  
  
  return (  
    <div className="min-h-screen bg-gray-50">  
      <div className="max-w-5xl mx-auto px-4 sm:px-6 lg:px-8 py-8">  
        {/* 返回导航 */}  
        <button  
          onClick={() => navigate(-1)}  
          className="inline-flex items-center gap-1.5 text-sm text-gray-500 hover:text-gray-700 mb-6 transition-colors"  
        >  
          <ArrowLeftIcon className="h-4 w-4" />  
          返回列表  
        </button>  
  
        {/* 用户信息头部 */}  
        <div className="bg-white rounded-xl shadow-sm border border-gray-200 p-6 mb-6">  
          <div className="flex flex-col sm:flex-row items-start gap-4">  
            <img  
              src={user.avatar}  
              alt={`${user.name} 头像`}  
              className="h-20 w-20 rounded-full object-cover ring-4 ring-gray-50"  
            />  
            <div className="flex-1 min-w-0">  
              <h1 className="text-2xl font-bold text-gray-900">{user.name}</h1>  
              <p className="text-gray-500 mt-1">{user.email}</p>  
              <div className="flex flex-wrap gap-2 mt-3">  
                <StatusBadge status={user.status} />  
                <span className="text-sm text-gray-400">注册于 {formatDate(user.createdAt)}</span>  
              </div>  
            </div>  
            <div className="flex gap-2 self-start">  
              <button className="px-4 py-2 text-sm font-medium rounded-lg border border-gray-300 text-gray-700 hover:bg-gray-50 transition-colors">  
                编辑  
              </button>  
              <button className="px-4 py-2 text-sm font-medium rounded-lg bg-red-50 text-red-600 hover:bg-red-100 transition-colors">  
                禁用  
              </button>  
            </div>  
          </div>  
        </div>  
  
        {/* 标签页导航 */}  
        <div className="border-b border-gray-200 mb-6">  
          <nav className="flex gap-6 -mb-px overflow-x-auto">  
            {tabs.map((tab) => (  
              <button  
                key={tab.key}  
                onClick={() => setActiveTab(tab.key)}  
                className={`  
                  pb-3 text-sm font-medium whitespace-nowrap border-b-2 transition-colors  
                  ${activeTab === tab.key  
                    ? 'border-blue-600 text-blue-600'  
                    : 'border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300'  
                  }  
                `}  
              >  
                {tab.label}  
              </button>  
            ))}  
          </nav>  
        </div>  
  
        {/* 标签页内容 */}  
        <div>  
          {activeTab === 'profile' && <UserProfile user={user} />}  
          {activeTab === 'activity' && <UserActivity userId={user.id} />}  
          {activeTab === 'settings' && <UserSettings user={user} />}  
        </div>  
      </div>  
    </div>  
  );  
};  
```  
  
---  
  
## 表单页模板  
  
表单页注重交互体验和验证反馈。  
  
```jsx  
/**  
 * 创建产品表单页  
 * 用于新增产品的表单页面，包含验证、提交和反馈  
 *  
 * @returns {React.ReactElement} 创建产品页面  
 */  
const CreateProductPage = () => {  
  const { formData, errors, handleChange, handleSubmit, submitting } = useProductForm();  
  
  return (  
    <div className="min-h-screen bg-gray-50">  
      <div className="max-w-2xl mx-auto px-4 sm:px-6 py-8">  
        <h1 className="text-2xl font-bold text-gray-900 mb-2">创建产品</h1>  
        <p className="text-gray-500 mb-8">填写以下信息来创建新产品</p>  
  
        <form onSubmit={handleSubmit} className="space-y-6">  
          {/* 产品名称 */}  
          <FormField  
            label="产品名称"  
            required  
            error={errors.name}  
          >  
            <input  
              type="text"  
              name="name"  
              value={formData.name}  
              onChange={handleChange}  
              placeholder="输入产品名称"  
              className={inputClassName(errors.name)}  
            />  
          </FormField>  
  
          {/* 产品描述 */}  
          <FormField  
            label="产品描述"  
            error={errors.description}  
          >  
            <textarea  
              name="description"  
              value={formData.description}  
              onChange={handleChange}  
              rows={4}  
              placeholder="详细描述产品特点和用途..."  
              className={`${inputClassName(errors.description)} resize-none`}  
            />  
            <p className="text-xs text-gray-400 mt-1">  
              {formData.description.length}/500 字  
            </p>  
          </FormField>  
  
          {/* 价格和库存（两列布局） */}  
          <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">  
            <FormField label="价格" required error={errors.price}>  
              <div className="relative">  
                <span className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400">¥</span>  
                <input  
                  type="number"  
                  name="price"  
                  value={formData.price}  
                  onChange={handleChange}  
                  className={`${inputClassName(errors.price)} pl-8`}  
                  placeholder="0.00"  
                />  
              </div>  
            </FormField>  
  
            <FormField label="库存数量" required error={errors.stock}>  
              <input  
                type="number"  
                name="stock"  
                value={formData.stock}  
                onChange={handleChange}  
                className={inputClassName(errors.stock)}  
                placeholder="0"  
              />  
            </FormField>  
          </div>  
  
          {/* 操作按钮 */}  
          <div className="flex items-center justify-end gap-3 pt-6 border-t border-gray-200">  
            <button  
              type="button"  
              onClick={() => navigate(-1)}  
              className="px-4 py-2.5 text-sm font-medium text-gray-700 border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors"  
            >  
              取消  
            </button>  
            <button  
              type="submit"  
              disabled={submitting}  
              className="  
                inline-flex items-center gap-2 px-6 py-2.5 text-sm font-medium  
                bg-blue-600 text-white rounded-lg  
                hover:bg-blue-700 active:scale-[0.98]  
                disabled:opacity-50 disabled:cursor-not-allowed  
                transition-all duration-200  
              "  
            >  
              {submitting && <Spinner className="h-4 w-4" />}  
              {submitting ? '提交中...' : '创建产品'}  
            </button>  
          </div>  
        </form>  
      </div>  
    </div>  
  );  
};  
```  
  
---  
  
## 仪表盘模板  
  
仪表盘以数据展示为核心，需要合理利用卡片和图表组织信息层次。  
  
```jsx  
/**  
 * 概览仪表盘页面  
 * 展示业务关键指标、趋势图表和最近活动  
 *  
 * @returns {React.ReactElement} 仪表盘页面  
 */  
const DashboardPage = () => {  
  const { stats, recentOrders, loading, error } = useDashboardData();  
  
  if (error) return <ErrorState message="数据加载失败" onRetry={refresh} />;  
  
  return (  
    <div className="min-h-screen bg-gray-50">  
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">  
        {/* 标题区域 */}  
        <div className="mb-8">  
          <h1 className="text-2xl font-bold text-gray-900">概览</h1>  
          <p className="text-gray-500 mt-1">欢迎回来，这是今天的业务数据概览</p>  
        </div>  
  
        {/* 统计卡片网格 */}  
        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 md:gap-6 mb-8">  
          {loading ? (  
            Array.from({ length: 4 }).map((_, i) => <StatsCardSkeleton key={i} />)  
          ) : (  
            stats.map((stat) => (  
              <StatsCard  
                key={stat.label}  
                title={stat.label}  
                value={stat.value}  
                trend={stat.trend}  
                trendValue={stat.trendPercent}  
                icon={stat.icon}  
              />  
            ))  
          )}  
        </div>  
  
        {/* 图表 + 列表区域 */}  
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">  
          {/* 趋势图表（占2/3） */}  
          <div className="lg:col-span-2 bg-white rounded-xl shadow-sm border border-gray-200 p-6">  
            <h2 className="text-lg font-semibold text-gray-900 mb-4">收入趋势</h2>  
            <RevenueChart />  
          </div>  
  
          {/* 最近订单（占1/3） */}  
          <div className="bg-white rounded-xl shadow-sm border border-gray-200 p-6">  
            <div className="flex items-center justify-between mb-4">  
              <h2 className="text-lg font-semibold text-gray-900">最近订单</h2>  
              <a href="/orders" className="text-sm text-blue-600 hover:text-blue-800 transition-colors">  
                查看全部  
              </a>  
            </div>  
            <div className="space-y-3">  
              {recentOrders?.map((order) => (  
                <OrderItem key={order.id} order={order} />  
              ))}  
            </div>  
          </div>  
        </div>  
      </div>  
    </div>  
  );  
};  
```  
  
---  
  
## 登录页模板  
  
```jsx  
/**  
 * 登录页面  
 * 包含登录表单、第三方登录、注册入口  
 *  
 * @returns {React.ReactElement} 登录页面  
 */  
const LoginPage = () => {  
  const { formData, errors, handleChange, handleSubmit, submitting } = useLoginForm();  
  
  return (  
    <div className="min-h-screen bg-gray-50 flex items-center justify-center px-4 py-12">  
      <div className="w-full max-w-md">  
        {/* Logo 和标题 */}  
        <div className="text-center mb-8">  
          <Logo className="h-10 w-auto mx-auto" />  
          <h1 className="mt-6 text-2xl font-bold text-gray-900">登录你的账户</h1>  
          <p className="mt-2 text-sm text-gray-500">  
            还没有账户？{' '}  
            <a href="/register" className="text-blue-600 hover:text-blue-800 font-medium transition-colors">  
              立即注册  
            </a>  
          </p>  
        </div>  
  
        {/* 登录表单 */}  
        <div className="bg-white rounded-2xl shadow-sm border border-gray-200 p-8">  
          <form onSubmit={handleSubmit} className="space-y-5">  
            <FormField label="邮箱" error={errors.email}>  
              <input  
                type="email"  
                name="email"  
                value={formData.email}  
                onChange={handleChange}  
                placeholder="name@example.com"  
                className={inputClassName(errors.email)}  
                autoComplete="email"  
              />  
            </FormField>  
  
            <FormField label="密码" error={errors.password}>  
              <PasswordInput  
                name="password"  
                value={formData.password}  
                onChange={handleChange}  
                className={inputClassName(errors.password)}  
                autoComplete="current-password"  
              />  
            </FormField>  
  
            <div className="flex items-center justify-between text-sm">  
              <label className="flex items-center gap-2 cursor-pointer">  
                <input type="checkbox" className="rounded border-gray-300 text-blue-600 focus:ring-blue-500" />  
                <span className="text-gray-600">记住我</span>  
              </label>  
              <a href="/forgot-password" className="text-blue-600 hover:text-blue-800 font-medium transition-colors">  
                忘记密码？  
              </a>  
            </div>  
  
            <button  
              type="submit"  
              disabled={submitting}  
              className="  
                w-full py-2.5 text-sm font-medium  
                bg-blue-600 text-white rounded-lg  
                hover:bg-blue-700 active:scale-[0.99]  
                disabled:opacity-50 disabled:cursor-not-allowed  
                transition-all duration-200  
              "  
            >  
              {submitting ? '登录中...' : '登录'}  
            </button>  
          </form>  
  
          {/* 分隔线 */}  
          <div className="relative mt-6 mb-6">  
            <div className="absolute inset-0 flex items-center">  
              <div className="w-full border-t border-gray-200" />  
            </div>  
            <div className="relative flex justify-center text-xs">  
              <span className="bg-white px-3 text-gray-400">或者使用</span>  
            </div>  
          </div>  
  
          {/* 第三方登录 */}  
          <div className="grid grid-cols-2 gap-3">  
            <SocialButton provider="google" label="Google" />  
            <SocialButton provider="github" label="GitHub" />  
          </div>  
        </div>  
      </div>  
    </div>  
  );  
};  
```  
  
---  
  
## 落地页模板  
  
落地页的重点是视觉吸引力和信息传达效率。  
  
```jsx  
/**  
 * 产品落地页  
 * 包含 Hero 区域、特性展示、定价和 CTA  
 *  
 * @returns {React.ReactElement} 落地页  
 */  
const LandingPage = () => {  
  return (  
    <div className="min-h-screen bg-white">  
      {/* Hero 区域 */}  
      <section className="relative overflow-hidden pt-16 pb-20 md:pt-24 md:pb-32">  
        {/* 背景装饰 */}  
        <div className="absolute inset-0 bg-gradient-to-b from-blue-50/50 to-white pointer-events-none" />  
        <div className="relative max-w-5xl mx-auto px-4 sm:px-6 text-center">  
          <span className="inline-block px-3 py-1 text-xs font-medium bg-blue-100 text-blue-700 rounded-full mb-6 animate-fade-in">  
            🎉 全新发布 v2.0  
          </span>  
          <h1 className="text-4xl sm:text-5xl md:text-6xl font-bold text-gray-900 leading-tight animate-slide-up">  
            让你的工作流  
            <span className="text-blue-600">快 10 倍</span>  
          </h1>  
          <p className="mt-6 text-lg sm:text-xl text-gray-600 max-w-2xl mx-auto leading-relaxed">  
            简单而强大的工具，帮助团队更高效地协作。无需复杂配置，开箱即用。  
          </p>  
          <div className="mt-10 flex flex-col sm:flex-row items-center justify-center gap-4">  
            <a href="/signup" className="  
              w-full sm:w-auto px-8 py-3.5 text-base font-medium  
              bg-blue-600 text-white rounded-xl  
              hover:bg-blue-700 hover:shadow-lg hover:-translate-y-0.5  
              active:scale-[0.98]  
              transition-all duration-200  
            ">  
              免费开始使用  
            </a>  
            <a href="/demo" className="  
              w-full sm:w-auto inline-flex items-center justify-center gap-2 px-8 py-3.5 text-base font-medium  
              text-gray-700 border border-gray-300 rounded-xl  
              hover:bg-gray-50 hover:border-gray-400  
              transition-all duration-200  
            ">  
              <PlayIcon className="h-5 w-5" />  
              观看演示  
            </a>  
          </div>  
        </div>  
      </section>  
  
      {/* 特性展示 */}  
      <section className="py-16 md:py-24 bg-gray-50">  
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">  
          <div className="text-center mb-12 md:mb-16">  
            <h2 className="text-3xl md:text-4xl font-bold text-gray-900">为什么选择我们</h2>  
            <p className="mt-4 text-lg text-gray-500 max-w-2xl mx-auto">  
              我们专注于解决团队协作中最痛的问题  
            </p>  
          </div>  
          <div className="grid grid-cols-1 md:grid-cols-3 gap-8">  
            {features.map((feature) => (  
              <div  
                key={feature.title}  
                className="bg-white rounded-2xl p-8 shadow-sm border border-gray-200 hover:shadow-lg hover:-translate-y-1 transition-all duration-300"  
              >  
                <div className="w-12 h-12 bg-blue-100 rounded-xl flex items-center justify-center text-blue-600 mb-5">  
                  {feature.icon}  
                </div>  
                <h3 className="text-xl font-semibold text-gray-900 mb-3">{feature.title}</h3>  
                <p className="text-gray-500 leading-relaxed">{feature.description}</p>  
              </div>  
            ))}  
          </div>  
        </div>  
      </section>  
  
      {/* CTA 区域 */}  
      <section className="py-16 md:py-24">  
        <div className="max-w-4xl mx-auto px-4 sm:px-6 text-center">  
          <h2 className="text-3xl md:text-4xl font-bold text-gray-900 mb-4">  
            准备好开始了吗？  
          </h2>  
          <p className="text-lg text-gray-500 mb-10">  
            加入超过 10,000 个团队，一起提升工作效率  
          </p>  
          <a href="/signup" className="  
            inline-block px-10 py-4 text-lg font-medium  
            bg-blue-600 text-white rounded-xl  
            hover:bg-blue-700 hover:shadow-lg hover:-translate-y-0.5  
            active:scale-[0.98]  
            transition-all duration-200  
          ">  
            免费注册  
          </a>  
        </div>  
      </section>  
    </div>  
  );  
};  
```  
  
---  
  
## 404 错误页模板  
  
```jsx  
/**  
 * 404 页面  
 * 友好的错误提示页面，引导用户回到正常路径  
 *  
 * @returns {React.ReactElement} 404 页面  
 */  
const NotFoundPage = () => {  
  return (  
    <div className="min-h-screen bg-gray-50 flex items-center justify-center px-4">  
      <div className="text-center max-w-md">  
        <p className="text-6xl font-bold text-blue-600 mb-4">404</p>  
        <h1 className="text-2xl font-bold text-gray-900 mb-2">页面未找到</h1>  
        <p className="text-gray-500 mb-8">  
          抱歉，你访问的页面不存在。可能已被移除或地址有误。  
        </p>  
        <div className="flex flex-col sm:flex-row items-center justify-center gap-3">  
          <a href="/" className="  
            px-6 py-2.5 text-sm font-medium  
            bg-blue-600 text-white rounded-lg  
            hover:bg-blue-700 active:scale-[0.98]  
            transition-all duration-200  
          ">  
            回到首页  
          </a>  
          <button  
            onClick={() => window.history.back()}  
            className="  
              px-6 py-2.5 text-sm font-medium  
              text-gray-700 border border-gray-300 rounded-lg  
              hover:bg-gray-50 transition-colors  
            "  
          >  
            返回上一页  
          </button>  
        </div>  
      </div>  
    </div>  
  );  
};  
```
```

> 生成的内容很长，SKILL.md 主文件加上 page-templates.md 参考模板，覆盖了视觉还原、响应式、交互动效、代码质量四个方面的规范，还附带了各类页面的完整模板代码。

## 第四步：创建测试用例与运行评估

技能文件写完了，接下来是新版最有意思的部分：**拿数据说话**。

### 评估怎么跑的

思路很直接，做对比实验：

* **with-skill 运行**：使用新创建的技能来完成任务
* **without-skill 运行（基线）**：不使用技能，让 Claude 直接完成同样的任务
* 两组运行通过**并行 subagent**同时启动，确保公平对比

三个子代理分别负责不同的评估角色：

* **Grader 代理**：逐条检查预期结果，给 PASS 或 FAIL
* **Comparator 代理**：盲评。它不知道哪个输出是哪个版本的，只看内容和结构打分
* **Analyzer 代理**：汇总多轮数据，找出统计数字背后的规律

image-20260314175938890

image-20260314175945541

同一个提示词在两种条件下各跑一遍，然后对比输出：

image-20260314180018169

跑的过程中，每个用例的输出、评分、执行指标都会记下来

image-20260314180224393

image-20260314180258775

image-20260314181131439

### 评估审查界面（Eval Review）

评估跑完后，`generate_review.py` 会生成一个 HTML 评审页面。

* **Outputs**：逐个看测试用例，显示 prompt、输出文件、评分，还能直接写反馈
* **Benchmark**：定量统计（通过率、耗时、token数），支持 XLSX 在线渲染

以下是生成的review.html的路径：

frontend-page-generator-workspace/iteration-1/review.html

```
<!DOCTYPE html>  
<html lang="en">  
<head>  
<meta charset="UTF-8">  
<meta name="viewport" content="width=device-width, initial-scale=1.0">  
<title>Eval Review</title>  
<link rel="preconnect" href="https://fonts.googleapis.com">  
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>  
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@500;600&family=Lora:wght@400;500&display=swap" rel="stylesheet">  
<script src="https://cdn.sheetjs.com/xlsx-0.20.3/package/dist/xlsx.full.min.js" integrity="sha384-EnyY0/GSHQGSxSgMwaIPzSESbqoOLSexfnSMN2AP+39Ckmn92stwABZynq1JyzdT" crossorigin="anonymous"></script>  
</script  
    
  ……省略……  
</body>  
</html>
```

frontend-page-generator-workspace/iteration-1/order-management-page/eval\_metadata.json示例

```
{  
  "eval_id": 1,  
"eval_name": "order-management-page",  
"prompt": "帮我创建一个电商后台的订单管理页面。需要有订单列表表格，支持按订单状态（待付款、已付款、已发货、已完成、已取消）筛选，有搜索框可以搜索订单号或客户名。表格列包含：订单号、客户名、金额、状态、下单时间、操作（查看详情、删除）。需要有分页功能。整体风格要专业简洁。",  
"assertions": [  
    {  
      "id": "a1-jsdoc",  
      "text": "每个组件和函数都有 JSDoc 文档注释",  
      "type": "code_quality"  
    },  
    {  
      "id": "a1-constants",  
      "text": "订单状态等常量提取到独立的 constants 文件中",  
      "type": "code_quality"  
    },  
    {  
      "id": "a1-component-split",  
      "text": "组件拆分合理，页面至少拆分为 3 个以上子组件（如搜索区、表格、分页等）",  
      "type": "code_quality"  
    },  
    {  
      "id": "a1-responsive",  
      "text": "使用了响应式类名（如 md:, lg: 等断点前缀），适配移动端和桌面端",  
      "type": "responsive"  
    },  
    {  
      "id": "a1-hover-effects",  
      "text": "按钮和可交互元素有 hover/active 状态样式和 transition 过渡效果",  
      "type": "interaction"  
    },  
    {  
      "id": "a1-loading-state",  
      "text": "包含加载状态处理（loading 骨架屏或 spinner）",  
      "type": "interaction"  
    },  
    {  
      "id": "a1-status-badge",  
      "text": "订单状态使用了不同颜色的标签/徽章来区分",  
      "type": "visual"  
    },  
    {  
      "id": "a1-search-filter",  
      "text": "包含可用的搜索输入框和状态筛选下拉/选项",  
      "type": "functionality"  
    },  
    {  
      "id": "a1-pagination",  
      "text": "包含分页组件，支持翻页操作",  
      "type": "functionality"  
    },  
    {  
      "id": "a1-chinese-comments",  
      "text": "代码注释使用中文",  
      "type": "code_quality"  
    }  
  ]  
}
```

评分流程grading

image-20260314180839290

image-20260314180935545

frontend-page-generator-workspace/grader.py

```
#!/usr/bin/env python3  
"""  
前端页面生成 skill 的自动评分脚本  
检查生成的代码是否满足各项质量断言  
"""  
import json  
import os  
import re  
import sys  
  
def read_all_files(directory):  
    """读取目录下所有文件的内容"""  
    contents = {}  
    ifnot os.path.exists(directory):  
        return contents  
    for root, dirs, files in os.walk(directory):  
        for f in files:  
            filepath = os.path.join(root, f)  
            try:  
                with open(filepath, 'r', encoding='utf-8') as fh:  
                    contents[filepath] = fh.read()  
            except:  
                pass  
    return contents  
  
def check_jsdoc(contents):  
    """检查是否有 JSDoc 注释"""  
    jsdoc_pattern = r'/\*\*[\s\S]*?\*/'  
    jsx_files = {k: v for k, v in contents.items() if k.endswith(('.jsx', '.js', '.tsx', '.ts'))}  
    ifnot jsx_files:  
        returnFalse, "没有找到 JS/JSX 文件"  
  
    files_with_jsdoc = 0  
    for filepath, content in jsx_files.items():  
        if re.search(jsdoc_pattern, content):  
            files_with_jsdoc += 1  
  
    ratio = files_with_jsdoc / len(jsx_files) if jsx_files else0  
    if ratio >= 0.5:  
        returnTrue, f"{files_with_jsdoc}/{len(jsx_files)} 个文件有 JSDoc 注释 ({ratio:.0%})"  
    returnFalse, f"仅 {files_with_jsdoc}/{len(jsx_files)} 个文件有 JSDoc 注释 ({ratio:.0%})"  
  
def check_constants_file(contents):  
    """检查是否有独立的常量文件"""  
    for filepath in contents:  
        if'constant'in filepath.lower() or'config'in filepath.lower():  
            if contents[filepath].strip():  
                returnTrue, f"找到常量文件: {os.path.basename(filepath)}"  
    returnFalse, "没有找到独立的常量文件"  
  
def check_component_split(contents, min_components=3):  
    """检查组件拆分数量"""  
    jsx_files = [k for k in contents if k.endswith(('.jsx', '.tsx'))]  
    component_count = len(jsx_files)  
    if component_count >= min_components:  
        returnTrue, f"拆分了 {component_count} 个组件文件 (≥{min_components})"  
    returnFalse, f"仅 {component_count} 个组件文件，不足 {min_components} 个"  
  
def check_responsive(contents):  
    """检查是否使用了响应式断点"""  
    breakpoint_pattern = r'\b(sm:|md:|lg:|xl:|2xl:)\w+'  
    files_with_responsive = 0  
    total_matches = 0  
    for filepath, content in contents.items():  
        if filepath.endswith(('.jsx', '.tsx', '.js')):  
            matches = re.findall(breakpoint_pattern, content)  
            if matches:  
                files_with_responsive += 1  
                total_matches += len(matches)  
    if files_with_responsive >= 1and total_matches >= 3:  
        returnTrue, f"{files_with_responsive} 个文件使用了响应式类名，共 {total_matches} 处"  
    returnFalse, f"响应式类名使用不足：{files_with_responsive} 个文件，{total_matches} 处"  
  
def check_hover_effects(contents):  
    """检查是否有 hover/active 交互效果和 transition"""  
    hover_count = 0  
    transition_count = 0  
    for content in contents.values():  
        hover_count += len(re.findall(r'hover:', content))  
        transition_count += len(re.findall(r'transition', content))  
    if hover_count >= 3and transition_count >= 2:  
        returnTrue, f"hover 效果 {hover_count} 处，transition 过渡 {transition_count} 处"  
    returnFalse, f"hover 效果 {hover_count} 处，transition {transition_count} 处（不足）"  
  
def check_loading_state(contents):  
    """检查是否有加载状态处理"""  
    all_content = '\n'.join(contents.values())  
    has_loading = bool(re.search(r'loading|isLoading|skeleton|Skeleton|spinner|Spinner|animate-pulse|animate-spin', all_content))  
    if has_loading:  
        returnTrue, "包含加载状态处理"  
    returnFalse, "未找到加载状态处理（loading/skeleton/spinner）"  
  
def check_status_badge(contents):  
    """检查订单状态是否用不同颜色标签区分"""  
    all_content = '\n'.join(contents.values())  
    color_patterns = ['bg-green', 'bg-red', 'bg-yellow', 'bg-blue', 'bg-purple', 'bg-orange', 'bg-emerald', 'bg-amber', 'bg-rose']  
    found_colors = [c for c in color_patterns if c in all_content]  
    if len(found_colors) >= 3:  
        returnTrue, f"状态标签使用了 {len(found_colors)} 种颜色: {', '.join(found_colors[:5])}"  
    returnFalse, f"仅找到 {len(found_colors)} 种颜色标签，不足 3 种"  
  
def check_search_filter(contents):  
    """检查是否有搜索和筛选功能"""  
    all_content = '\n'.join(contents.values())  
    has_search = bool(re.search(r'search|Search|搜索|placeholder.*搜索|type="search"|type="text"', all_content))  
    has_filter = bool(re.search(r'filter|Filter|筛选|select|Select|dropdown', all_content, re.IGNORECASE))  
    if has_search and has_filter:  
        returnTrue, "包含搜索输入框和筛选功能"  
    parts = []  
    ifnot has_search: parts.append("缺少搜索")  
    ifnot has_filter: parts.append("缺少筛选")  
    returnFalse, "、".join(parts)  
  
def check_pagination(contents):  
    """检查是否有分页组件"""  
    all_content = '\n'.join(contents.values())  
    has_pagination = bool(re.search(r'[Pp]agination|分页|pageSize|currentPage|setCurrentPage|上一页|下一页|prev|next.*page', all_content))  
    if has_pagination:  
        returnTrue, "包含分页功能"  
    returnFalse, "未找到分页组件或分页逻辑"  
  
def check_chinese_comments(contents):  
    """检查注释是否使用中文"""  
    chinese_comment_pattern = r'(//|/\*|\*).*[\u4e00-\u9fff]'  
    files_with_chinese = 0  
    jsx_files = {k: v for k, v in contents.items() if k.endswith(('.jsx', '.js', '.tsx', '.ts'))}  
    for content in jsx_files.values():  
        if re.search(chinese_comment_pattern, content):  
            files_with_chinese += 1  
    ratio = files_with_chinese / len(jsx_files) if jsx_files else0  
    if ratio >= 0.3:  
        returnTrue, f"{files_with_chinese}/{len(jsx_files)} 个文件有中文注释 ({ratio:.0%})"  
    returnFalse, f"仅 {files_with_chinese}/{len(jsx_files)} 个文件有中文注释 ({ratio:.0%})"  
  
def check_all_sections(contents):  
    """检查是否包含所有页面区块"""  
    all_content = '\n'.join(contents.values())  
    sections = {  
        '导航栏': r'[Nn]av|[Hh]eader|导航',  
        'Hero': r'[Hh]ero',  
        '功能': r'[Ff]eature|功能|核心',  
        '评价': r'[Tt]estimon|评价|review',  
        '定价': r'[Pp]ricing|定价|价格方案',  
        'CTA': r'[Cc][Tt][Aa]|立即|开始使用|免费',  
        'Footer': r'[Ff]ooter|页脚|底部',  
    }  
    found = {}  
    for name, pattern in sections.items():  
        found[name] = bool(re.search(pattern, all_content))  
    found_count = sum(found.values())  
    if found_count >= 6:  
        returnTrue, f"包含 {found_count}/7 个页面区块"  
    missing = [k for k, v in found.items() ifnot v]  
    returnFalse, f"缺少区块: {', '.join(missing)}"  
  
def check_animations(contents):  
    """检查是否有动画效果"""  
    all_content = '\n'.join(contents.values())  
    anim_patterns = ['animate-', 'animation', 'keyframes', 'transition', 'transform', 'hover:', 'scale', 'translate']  
    found = [p for p in anim_patterns if p in all_content]  
    if len(found) >= 4:  
        returnTrue, f"使用了 {len(found)} 种动画/过渡效果: {', '.join(found[:5])}"  
    returnFalse, f"动画效果不足，仅有: {', '.join(found)}"  
  
def check_responsive_nav(contents):  
    """检查导航栏是否有移动端适配"""  
    all_content = '\n'.join(contents.values())  
    has_hamburger = bool(re.search(r'md:hidden|lg:hidden|hamburger|menu.*mobile|mobile.*menu|MenuIcon|setMenuOpen|setIsOpen|menuOpen|isMenuOpen|mobileMenu', all_content, re.IGNORECASE))  
    has_desktop_nav = bool(re.search(r'hidden\s+md:flex|hidden\s+lg:flex|md:block|lg:block', all_content))  
    if has_hamburger or has_desktop_nav:  
        returnTrue, "导航栏有移动端/桌面端切换"  
    returnFalse, "未找到导航栏的移动端适配"  
  
def check_pricing_grid(contents):  
    """检查定价方案的网格布局"""  
    all_content = '\n'.join(contents.values())  
    has_grid = bool(re.search(r'grid.*cols|grid-cols', all_content))  
    has_plans = len(re.findall(r'免费|[Ff]ree|专业|[Pp]ro|企业|[Ee]nterprise', all_content)) >= 3  
    if has_grid and has_plans:  
        returnTrue, "定价使用网格布局，包含三个方案"  
    returnFalse, f"定价布局不完整: grid={has_grid}, 三方案={has_plans}"  
  
def check_cta_buttons(contents):  
    """检查CTA按钮样式"""  
    all_content = '\n'.join(contents.values())  
    has_cta = bool(re.search(r'bg-blue|bg-indigo|bg-purple|bg-gradient|primary', all_content))  
    has_hover = bool(re.search(r'hover:bg-|hover:shadow|hover:-translate', all_content))  
    if has_cta and has_hover:  
        returnTrue, "CTA 按钮有突出样式和 hover 效果"  
    returnFalse, f"CTA 按钮样式不足: 颜色={has_cta}, hover={has_hover}"  
  
def check_mobile_first(contents):  
    """检查是否移动端优先"""  
    all_content = '\n'.join(contents.values())  
    # 检查是否有基础类 + 断点向上适配的模式  
    mobile_first = len(re.findall(r'grid-cols-1\s+(?:sm:|md:|lg:)grid-cols', all_content))  
    responsive_up = len(re.findall(r'\b(?:sm:|md:|lg:|xl:)\w+', all_content))  
    if responsive_up >= 10:  
        returnTrue, f"使用了 {responsive_up} 处向上适配断点类名"  
    returnFalse, f"向上适配不足，仅 {responsive_up} 处"  
  
def check_sidebar_layout(contents):  
    """检查桌面端左右分栏布局"""  
    all_content = '\n'.join(contents.values())  
    has_flex_or_grid = bool(re.search(r'flex|grid', all_content))  
    has_sidebar = bool(re.search(r'sidebar|aside|nav.*settings|settings.*nav|w-64|w-60|w-56|w-48|分类导航', all_content, re.IGNORECASE))  
    if has_flex_or_grid and has_sidebar:  
        returnTrue, "使用了左右分栏布局"  
    returnFalse, f"分栏布局不完整: flex/grid={has_flex_or_grid}, sidebar={has_sidebar}"  
  
def check_mobile_tabs(contents):  
    """检查移动端Tab切换"""  
    all_content = '\n'.join(contents.values())  
    has_tab = bool(re.search(r'tab|Tab|标签|activeTab|currentTab', all_content, re.IGNORECASE))  
    has_responsive_switch = bool(re.search(r'(md:hidden|lg:hidden|md:block|lg:block|md:flex|lg:flex)', all_content))  
    if has_tab and has_responsive_switch:  
        returnTrue, "移动端有 Tab 切换，配合断点控制"  
    returnFalse, f"移动端 Tab: tab={has_tab}, 断点切换={has_responsive_switch}"  
  
def check_avatar_upload(contents):  
    """检查头像上传"""  
    all_content = '\n'.join(contents.values())  
    has_avatar = bool(re.search(r'avatar|头像|[Uu]pload|上传|file.*input|input.*file|type="file"', all_content))  
    if has_avatar:  
        returnTrue, "包含头像上传功能"  
    returnFalse, "未找到头像上传功能"  
  
def check_form_validation(contents):  
    """检查表单验证样式"""  
    all_content = '\n'.join(contents.values())  
    has_focus = bool(re.search(r'focus:', all_content))  
    has_validation = bool(re.search(r'error|Error|invalid|required|验证|校验|border-red|text-red', all_content))  
    if has_focus and has_validation:  
        returnTrue, "表单有聚焦样式和验证反馈"  
    returnFalse, f"聚焦={has_focus}, 验证={has_validation}"  
  
def check_toggle_switch(contents):  
    """检查开关组件"""  
    all_content = '\n'.join(contents.values())  
    has_toggle = bool(re.search(r'[Tt]oggle|[Ss]witch|开关|checkbox.*switch|role="switch"|translate-x', all_content))  
    if has_toggle:  
        returnTrue, "包含开关/切换组件"  
    returnFalse, "未找到开关/切换组件"  
  
def check_theme_selector(contents):  
    """检查主题选择"""  
    all_content = '\n'.join(contents.values())  
    has_theme = bool(re.search(r'亮色|暗色|[Ll]ight|[Dd]ark|跟随系统|[Ss]ystem|theme|Theme|主题', all_content))  
    if has_theme:  
        returnTrue, "包含主题选择功能"  
    returnFalse, "未找到主题选择功能"  
  
# 断言检查函数映射  
ASSERTION_CHECKS = {  
    # 测试用例1: 订单管理页  
    'a1-jsdoc': check_jsdoc,  
    'a1-constants': check_constants_file,  
    'a1-component-split': check_component_split,  
    'a1-responsive': check_responsive,  
    'a1-hover-effects': check_hover_effects,  
    'a1-loading-state': check_loading_state,  
    'a1-status-badge': check_status_badge,  
    'a1-search-filter': check_search_filter,  
    'a1-pagination': check_pagination,  
    'a1-chinese-comments': check_chinese_comments,  
    # 测试用例2: SaaS落地页  
    'a2-jsdoc': check_jsdoc,  
    'a2-constants': check_constants_file,  
    'a2-all-sections': check_all_sections,  
    'a2-animations': check_animations,  
    'a2-responsive-nav': check_responsive_nav,  
    'a2-pricing-grid': check_pricing_grid,  
    'a2-cta-buttons': check_cta_buttons,  
    'a2-component-split': check_component_split,  
    'a2-mobile-first': check_mobile_first,  
    'a2-chinese-comments': check_chinese_comments,  
    # 测试用例3: 设置页面  
    'a3-jsdoc': check_jsdoc,  
    'a3-constants': check_constants_file,  
    'a3-sidebar-layout': check_sidebar_layout,  
    'a3-mobile-tabs': check_mobile_tabs,  
    'a3-avatar-upload': check_avatar_upload,  
    'a3-form-validation': check_form_validation,  
    'a3-toggle-switch': check_toggle_switch,  
    'a3-theme-selector': check_theme_selector,  
    'a3-component-split': check_component_split,  
    'a3-chinese-comments': check_chinese_comments,  
}  
  
def grade_run(output_dir, eval_metadata):  
    """对单次运行进行评分"""  
    contents = read_all_files(output_dir)  
    ifnot contents:  
        return {"error": f"目录为空或不存在: {output_dir}", "expectations": []}  
  
    expectations = []  
    for assertion in eval_metadata.get('assertions', []):  
        aid = assertion['id']  
        check_fn = ASSERTION_CHECKS.get(aid)  
        if check_fn:  
            passed, evidence = check_fn(contents)  
            expectations.append({  
                "text": assertion['text'],  
                "passed": passed,  
                "evidence": evidence  
            })  
        else:  
            expectations.append({  
                "text": assertion['text'],  
                "passed": False,  
                "evidence": f"未找到断言 {aid} 的检查函数"  
            })  
  
    return {"expectations": expectations}  
  
def main():  
    workspace = sys.argv[1] if len(sys.argv) > 1else"/Users/qihoo/Documents/A_Code/CeShi/test-skill/frontend-page-generator-workspace/iteration-1"  
  
    eval_dirs = ['order-management-page', 'saas-landing-page', 'user-settings-page']  
  
    summary = {}  
  
    for eval_dir in eval_dirs:  
        eval_path = os.path.join(workspace, eval_dir)  
        metadata_path = os.path.join(eval_path, 'eval_metadata.json')  
  
        ifnot os.path.exists(metadata_path):  
            print(f"跳过 {eval_dir}: eval_metadata.json 不存在")  
            continue  
  
        with open(metadata_path, 'r', encoding='utf-8') as f:  
            metadata = json.load(f)  
  
        for run_type in ['with_skill', 'without_skill']:  
            output_dir = os.path.join(eval_path, run_type, 'outputs')  
            grading = grade_run(output_dir, metadata)  
  
            # 保存 grading.json  
            grading_path = os.path.join(eval_path, run_type, 'grading.json')  
            with open(grading_path, 'w', encoding='utf-8') as f:  
                json.dump(grading, f, ensure_ascii=False, indent=2)  
  
            # 汇总  
            passed = sum(1for e in grading['expectations'] if e['passed'])  
            total = len(grading['expectations'])  
            key = f"{eval_dir}/{run_type}"  
            summary[key] = {"passed": passed, "total": total, "rate": f"{passed}/{total}"}  
  
            print(f"\n{'='*60}")  
            print(f"📊 {eval_dir} / {run_type}: {passed}/{total} 通过")  
            print(f"{'='*60}")  
            for exp in grading['expectations']:  
                icon = '✅'if exp['passed'] else'❌'  
                print(f"  {icon} {exp['text']}")  
                print(f"     → {exp['evidence']}")  
  
    # 保存总结  
    summary_path = os.path.join(workspace, 'grading_summary.json')  
    with open(summary_path, 'w', encoding='utf-8') as f:  
        json.dump(summary, f, ensure_ascii=False, indent=2)  
  
    print(f"\n\n{'='*60}")  
    print("📋 总结")  
    print(f"{'='*60}")  
    for key, val in summary.items():  
        print(f"  {key}: {val['rate']}")  
  
if __name__ == '__main__':  
    main()
```

### 评估结果：Grading Summary

评估脚本跑完会输出一份 `grading_summary.json`，汇总两种条件下的通过率：

```
{  
  "order-management-page/with_skill": {  
    "passed": 10,  
    "total": 10,  
    "rate": "10/10"  
  },  
"order-management-page/without_skill": {  
    "passed": 8,  
    "total": 10,  
    "rate": "8/10"  
  },  
"saas-landing-page/with_skill": {  
    "passed": 10,  
    "total": 10,  
    "rate": "10/10"  
  },  
"saas-landing-page/without_skill": {  
    "passed": 8,  
    "total": 10,  
    "rate": "8/10"  
  },  
"user-settings-page/with_skill": {  
    "passed": 10,  
    "total": 10,  
    "rate": "10/10"  
  },  
"user-settings-page/without_skill": {  
    "passed": 10,  
    "total": 10,  
    "rate": "10/10"  
  }  
}
```

用 skill 的三组测试全是 **10/10**。不用 skill 的基线组有两个场景丢了分（订单管理页和 SaaS 落地页各少两项），只有用户设置页保住了满分。

### 基准测试报告：Benchmark

`aggregate_benchmark.py` 会在 grading\_summary 基础上做进一步聚合，生成 `benchmark.json`，里面有平均值、标准差、总通过数这些统计维度：

```
{  
  "skill_name": "frontend-page-generator",  
"iteration": 1,  
"configs": [  
    {  
      "name": "with_skill",  
      "label": "使用 Skill",  
      "results": [  
        {  
          "eval_name": "order-management-page",  
          "pass_rate": 1.0,  
          "passed": 10,  
          "total": 10,  
          "details": [  
            {"text": "每个组件和函数都有 JSDoc 文档注释", "passed": true},  
            {"text": "订单状态等常量提取到独立的 constants 文件中", "passed": true},  
            {"text": "组件拆分合理，至少 3 个子组件", "passed": true},  
            {"text": "使用了响应式类名", "passed": true},  
            {"text": "hover/active 状态和 transition", "passed": true},  
            {"text": "包含加载状态处理", "passed": true},  
            {"text": "订单状态使用不同颜色标签", "passed": true},  
            {"text": "搜索输入框和状态筛选", "passed": true},  
            {"text": "分页组件", "passed": true},  
            {"text": "中文注释", "passed": true}  
          ]  
        },  
        {  
          "eval_name": "saas-landing-page",  
          "pass_rate": 1.0,  
          "passed": 10,  
          "total": 10,  
          "details": [  
            {"text": "每个组件和函数都有 JSDoc 文档注释", "passed": true},  
            {"text": "常量提取到独立文件", "passed": true},  
            {"text": "包含所有页面区块", "passed": true},  
            {"text": "CSS 动画/transition 效果", "passed": true},  
            {"text": "导航栏移动端适配", "passed": true},  
            {"text": "定价网格布局", "passed": true},  
            {"text": "CTA 按钮样式突出", "passed": true},  
            {"text": "组件拆分合理", "passed": true},  
            {"text": "移动端优先编写", "passed": true},  
            {"text": "中文注释", "passed": true}  
          ]  
        },  
        {  
          "eval_name": "user-settings-page",  
          "pass_rate": 1.0,  
          "passed": 10,  
          "total": 10,  
          "details": [  
            {"text": "每个组件和函数都有 JSDoc 文档注释", "passed": true},  
            {"text": "常量提取到独立文件", "passed": true},  
            {"text": "左右分栏布局", "passed": true},  
            {"text": "移动端 Tab 切换", "passed": true},  
            {"text": "头像上传功能", "passed": true},  
            {"text": "表单聚焦和验证样式", "passed": true},  
            {"text": "开关/切换组件", "passed": true},  
            {"text": "主题选择 UI", "passed": true},  
            {"text": "组件拆分合理", "passed": true},  
            {"text": "中文注释", "passed": true}  
          ]  
        }  
      ],  
      "aggregate": {  
        "mean_pass_rate": 1.0,  
        "stddev_pass_rate": 0.0,  
        "total_passed": 30,  
        "total_assertions": 30  
      }  
    },  
    {  
      "name": "without_skill",  
      "label": "无 Skill (基线)",  
      "results": [  
        {  
          "eval_name": "order-management-page",  
          "pass_rate": 0.8,  
          "passed": 8,  
          "total": 10,  
          "details": [  
            {"text": "每个组件和函数都有 JSDoc 文档注释", "passed": false},  
            {"text": "订单状态等常量提取到独立的 constants 文件中", "passed": true},  
            {"text": "组件拆分合理，至少 3 个子组件", "passed": true},  
            {"text": "使用了响应式类名", "passed": true},  
            {"text": "hover/active 状态和 transition", "passed": true},  
            {"text": "包含加载状态处理", "passed": true},  
            {"text": "订单状态使用不同颜色标签", "passed": true},  
            {"text": "搜索输入框和状态筛选", "passed": true},  
            {"text": "分页组件", "passed": true},  
            {"text": "中文注释", "passed": false}  
          ]  
        },  
        {  
          "eval_name": "saas-landing-page",  
          "pass_rate": 0.8,  
          "passed": 8,  
          "total": 10,  
          "details": [  
            {"text": "每个组件和函数都有 JSDoc 文档注释", "passed": false},  
            {"text": "常量提取到独立文件", "passed": true},  
            {"text": "包含所有页面区块", "passed": true},  
            {"text": "CSS 动画/transition 效果", "passed": true},  
            {"text": "导航栏移动端适配", "passed": true},  
            {"text": "定价网格布局", "passed": true},  
            {"text": "CTA 按钮样式突出", "passed": true},  
            {"text": "组件拆分合理", "passed": true},  
            {"text": "移动端优先编写", "passed": true},  
            {"text": "中文注释", "passed": false}  
          ]  
        },  
        {  
          "eval_name": "user-settings-page",  
          "pass_rate": 1.0,  
          "passed": 10,  
          "total": 10,  
          "details": [  
            {"text": "每个组件和函数都有 JSDoc 文档注释", "passed": true},  
            {"text": "常量提取到独立文件", "passed": true},  
            {"text": "左右分栏布局", "passed": true},  
            {"text": "移动端 Tab 切换", "passed": true},  
            {"text": "头像上传功能", "passed": true},  
            {"text": "表单聚焦和验证样式", "passed": true},  
            {"text": "开关/切换组件", "passed": true},  
            {"text": "主题选择 UI", "passed": true},  
            {"text": "组件拆分合理", "passed": true},  
            {"text": "中文注释", "passed": true}  
          ]  
        }  
      ],  
      "aggregate": {  
        "mean_pass_rate": 0.867,  
        "stddev_pass_rate": 0.094,  
        "total_passed": 26,  
        "total_assertions": 30  
      }  
    }  
  ],  
"delta": {  
    "pass_rate_improvement": "+13.3%",  
    "with_skill_mean": 1.0,  
    "without_skill_mean": 0.867  
  }  
}
```

几个关键数字：

* **用 skill**：平均通过率 1.0，标准差 0.0，30 项全过
* **不用 skill**：平均通过率 0.867，标准差 0.094，26/30 通过
* **提升**：+13.3%

数据说的很清楚，skill 确实管用。

## 第五步：使用 Eval-Viewer 可视化审查

image-20260314181238023

数据跑完了，`generate_review.py` 会起一个本地 HTTP 服务（默认端口 3117），在浏览器里打开评审界面。

image-20260314181813666

image-20260314181829733

image-20260315131927650

### 启动所有项目进行对比预览

3个测试场景 × 2种条件 = **6个页面**。让 Claude Code 把这6个项目全启动起来，就可以在浏览器里一个个看了：

image-20260315134454872

其中一个用 skill 生成的页面效果：

image-20260315135719052

## 第六步：填写反馈与迭代改进

看完页面之后，可以给每个运行结果写反馈。这些反馈会被 skill-creator 拿去做下一轮改进。

写反馈有个要点：**不要头痛医头**。如果某个页面表格对齐有问题，不要写"把表格对齐修一下"，而是想想背后是什么通用问题（比如"响应式断点处理不够细"），这样改进才有泛化效果，不会过拟合到具体用例上。

eval-viewer 里可以逐个看结果、写评价：

image-20260315140030537

image-20260315140038698

### 反馈格式

反馈写完后 eval-viewer 会导出成 JSON。每条包含 `run_id`、`feedback` 和 `timestamp`：

```
{  
  "reviews": [  
    {  
      "run_id": "order-management-page-with_skill",  
      "feedback": "很好",  
      "timestamp": "2026-03-15T06:00:13.184Z"  
    },  
    {  
      "run_id": "order-management-page-without_skill",  
      "feedback": "页面生硬，不美观",  
      "timestamp": "2026-03-15T06:00:13.184Z"  
    },  
    {  
      "run_id": "saas-landing-page-with_skill",  
      "feedback": "符合标准",  
      "timestamp": "2026-03-15T06:00:13.184Z"  
    },  
    {  
      "run_id": "saas-landing-page-without_skill",  
      "feedback": "不够实际，偏幼稚",  
      "timestamp": "2026-03-15T06:00:13.184Z"  
    },  
    {  
      "run_id": "user-settings-page-with_skill",  
      "feedback": "效果好",  
      "timestamp": "2026-03-15T06:00:13.184Z"  
    },  
    {  
      "run_id": "user-settings-page-without_skill",  
      "feedback": "正常发挥",  
      "timestamp": "2026-03-15T06:00:13.184Z"  
    }  
  ],  
"status": "complete"  
}
```

看反馈内容就能感受到差距：用了 skill 的版本拿到的是"很好""符合标准""效果好"，没用 skill 的版本就是"页面生硬，不美观""不够实际，偏幼稚"。数字上 +13.3%，主观感受上也对得上。

提交反馈后，skill-creator 会分析这些反馈来改进技能：

image-20260315140209914

image-20260315140351150

## 总结

### 新版到底好在哪

跑完这一整套流程，几个感受比较明显：

1. **有了完整的迭代闭环**：从采访需求到写技能文件到跑评估到看反馈再到改进，形成了一个可以反复转的循环。以前是写完就完了，现在有了持续改进的手段。
2. **评估不再靠感觉**：三个代理各管一摊，Grader 负责对错、Comparator 做盲评、Analyzer 找规律。尤其盲评机制，结果更可信。
3. **eval-viewer 省事不少**：在浏览器里就能看完所有结果、写反馈，不用在命令行里翻来翻去。
4. **train/test 分割挡住了过拟合**：60/40 分堆，改进的时候只看训练集，最后用测试集验证。skill 是真变强了还是只会做"见过的题"，一测便知。

### 对开发者来说意味着什么

skill-creator 是一个"造技能的技能"。有了新版的这套评估和迭代工具，写 skill 不再是凭经验一把梭，而是有数据、有反馈、有对比的工程化过程。

-END -

**如果您关注前端+AI 相关领域可以扫码进群交流**

添加小编微信进群😊

## 关于奇舞团

奇舞团是 360 集团最大的大前端团队，非常重视人才培养，有工程师、讲师、翻译官、业务接口人、团队 Leader 等多种发展方向供员工选择，并辅以提供相应的技术力、专业力、通用力、领导力等培训课程。奇舞团以开放和求贤的心态欢迎各种优秀人才关注和加入奇舞团。