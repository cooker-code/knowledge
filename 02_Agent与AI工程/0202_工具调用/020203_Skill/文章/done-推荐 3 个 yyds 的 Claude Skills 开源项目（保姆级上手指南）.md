> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 推荐 3 个 yyds 的 Claude Skills 开源项目（保姆级上手指南）
author: 吴哥AI实操笔记
date:
url: https://mp.weixin.qq.com/s?__biz=MzU4MDAwMzI4MA==&mid=2247488088&idx=1&sn=6f8e3346e4c82d752aab3fbecc2442af&chksm=fc800f1e5a9ee51881cc30394d8144ea7fbcfa8b79402a50d627c464a7c2a2eb20dbafdd07f7&mpshare=1&scene=24&srcid=11245JzrMekIcYk27o9Adprg&sharer_shareinfo=80481a8dc5e6fe87fddcd207c242e6c5&sharer_shareinfo_first=80481a8dc5e6fe87fddcd207c242e6c5#rd
---

大家好，我是吴哥，专注AI编程、AI智能体。立志持续输出、帮助小白轻松上手AI编程和AI智能体。

今天这篇，给你带来 **3 个“yyds 级”Claude Skills 开源项目**，不卖关子、直接能用、上手即爽。

## 文章核心观点

* **要找“现成技能库”直接抄作业？** → 选 **Composio 的《awesome-claude-skills》**：门类全、结构规范、可拿来即用/即改。
* **要在 Claude Code 里“像生产环境一样”自动触发技能？** → 用 **claude-code-infrastructure-showcase**：钩子机制 + 渐进披露，**真工程化**。
* **要让 AI 写代码摆脱“随缘”，强制走专家级流程（头脑风暴→计划→TDD→实现→质检）？** → 装 **superpowers**：**思维方式升级**，质量上限明显提高。

---

## 如何选型

```
如果你是「第一次玩 Skills」→ awesome-claude-skills
如果你是「想在 CC 里自动触发、按场景路由」→ infrastructure-showcase
如果你是「工程师、要严肃交付质量」→ superpowers（流程与标准）
```

## 01｜Claude Skills 大合集：awesome-claude-skills

> 开源地址：
>
> https://github.com/ComposioHQ/awesome-claude-skills
>
> 参考扩展清单：
>
> https://github.com/BehiSecc/awesome-claude-skills

### 它是什么？

一个**精心整理的 Skills 选集**。按场景分门别类（文档处理、开发、数据分析、营销、创作…），每个技能目录里都有 `SKILL.md`（**用途/边界/步骤/指令**），复杂的还配**脚本、模板和资源**。比如 `playwright-skill` 能让 Claude 直接跑 Playwright 做**网页自动化测试与验证**。

### 优势

* **覆盖面广**：从办公自动化到工程工具，**一站式挑**。
* **规范统一**：`SKILL.md` 可读性强，**复制—改造—复用**成本低。
* **入门友好**：不会写 Skill？照着它的目录骨架抄，**最快路径**。

### 3 分钟上手（建议姿势）

```
# 1) 克隆仓库
git clone https://github.com/ComposioHQ/awesome-claude-skills
cd awesome-claude-skills

# 2) 选择一个子技能（示例：playwright-skill）
#   - 阅读 SKILL.md（输入/输出/步骤/常见错误）
#   - 将 /scripts /templates /references 按需拷进你的项目 .claude/skills 下
#
# 3) 在 Claude Code 中加载（示意）
/skills list
/skills load playwright
/playwright run --url https://example.com --check login-flow
```

### 适合谁？

* **刚上手 Skills**、想一步到位拥有一堆可复用能力的人/团队；
* 需要**快速拼装一个“能力市场”**的人（企业内部也能这么用的）。

## 02｜Claude Code 基础设施：infrastructure showcase

> 开源地址：
>
> https://github.com/diet103/claude-code-infrastructure-showcase

### 它是什么？

一套开发者实战 6 个月打磨出来的 **Claude Code “技能基础设施”**。**最大亮点：自动触发**——不再“手动想起用哪个 Skill”，而是通过**钩子机制**让 CC **根据上下文智能建议/激活最相关技能**。所有主要 Skill 控制在 ~500 行以内，**配资源文件渐进披露**，既不炸上下文，又保留深度。

### 优势

* **“会自己来活儿”**：输入提示/改动文件→**自动路由**到合适技能（后端/前端/路由测试/错误追踪…）。
* **真工程化**：**限长主文件 + 资源分拆**，适配 CC 实际上下文预算。
* **落地稳**：来源于生产环境，**不是“演示仓库”**。

### 5 个核心 Skills（示例）

* **后端开发指南**：约束风格、测试、异常策略；
* **前端开发指南**：组件/状态/交互规范；
* **技能开发元技能**：教你**写 Skill 的 Skill**；
* **路由测试**：自动生成/校验路由；
* **错误追踪**：统一收敛、出报告。

### 3 分钟上手（建议姿势）

```
git clone https://github.com/diet103/claude-code-infrastructure-showcase
cd claude-code-infrastructure-showcase

# 阅读根目录说明，注意“hooks / 渐进披露”部分
# 将 /skills/* 安装到你的 CC 搜索路径（或做软链）

# 进入你的项目，唤起 CC
claude
# 正常对话/编辑代码时，会收到“建议激活 X Skill”的提示
# 也可手动：/skills load backend-guide
```

### 适合谁？

* 追求**“像人一样会判断”**的 CC 使用体验；
* 团队想要**统一工程规范**、让新人“自动走对流程”的组织。

## 03｜数据分析 Agent：superpowers

> 开源地址：https://github.com/obra/superpowers

### 它是什么？

一组把 Claude 训练成“**世界级高级工程师工作法**”的 Skills：**先头脑风暴（brainstorm）→写计划（write-plan）→写测试（TDD）→实现代码→质量检查**。**强制流程**让“写错就返工、乱写就露馅”这么个流程。

### 优势

* **流程即质量**：拒绝“直接写业务代码然后祈祷没 Bug”，**让 TDD 成为默认**。
* **可审可改**：计划是 Markdown 文档，**你授权了才开始动手**。
* **风险前置**：在“写一行代码之前”把**边界/依赖/异常**想清楚。

### 典型交互

```
你：实现【问卷系统-题库导入】功能，先脑暴风险点与关键用例，再输出执行计划。
Claude（superpowers）：
  - Brainstorm：数据格式/去重/权限/幂等/回滚...
  - Write-Plan：分解子任务、接口约定、异常矩阵、日志与埋点
  - TDD：生成关键单测（含边界用例）
  - Implement：仅在你确认计划后，开始提交代码
  - Review：风格/安全/性能检查与变更说明
```

### 3 分钟上手

```
git clone https://github.com/obra/superpowers
cd superpowers
# 阅读每个 Skill 的 SKILL.md，优先从 brainstorm / write-plan 开始组合
# 在你的项目中加载这些技能，按“先计划后编码”的姿势使用
```

### 适合谁？

* **后端/全栈/架构**开发者；
* 想把“**质量标准**”内化进 AI 工作流的团队。

## 三者怎么配合？

* **awesome-claude-skills** 当你的**技能市场/素材库**：快速挑选、拼装功能性能力（办公、抓取、测试、转码…）。
* **infrastructure-showcase** 给 CC **装上“自动判断与路由”**：输入/改动即触发最合适的技能，**减少手动切换**。
* **superpowers** 作为**工程方法论“地基”**：无论调用哪个技能，**流程先行**（脑暴→计划→TDD→实现→复盘），把交付质量托底。

> 一句话：**素材库（awesome） × 路由执行（infra） × 工程方法（superpowers）**＝ 团队级、可复制、可审计、可持续进化的 **AI 编程工作流**。

---

## 对比表

| 项目 | 主打价值 | 典型场景 | 学习/部署成本 | 上手速度 | 风险点 |
| --- | --- | --- | --- | --- | --- |
| awesome-claude-skills | **技能覆盖面广** 、规范清晰 | 初/中级用户拼装能力 | 低 | **最快** | 质量参差需筛选 |
| infrastructure-showcase | **自动触发 + 渐进披露** 、工程化 | CC 日常开发、团队规范 | 中 | 快 | 需理解钩子/路由 |
| superpowers | **流程与质量托底** （TDD/计划/评审） | 严肃交付、多人协作 | 中 | 中 | 需要团队“先计划”的共识 |

## 实操开箱

```
# 1）创建 Skills 工作区
mkdir -p .claude/skills && cd .claude/skills

# 2）把三者里“你要的技能目录”拷进来或做软链
#    注意每个技能都有 SKILL.md，先读清输入/输出/边界

# 3）进入项目启动 Claude Code
claude

# 4）列出/加载技能（命令以你本地 CC 为准）
/skills list
/skills load backend-guide
/skills load brainstorm
/skills load write-plan

# 5）从“流程”开始，而不是直接“写代码”
（示例）“为【XX模块】先输出计划与测试清单，我确认后再实现。”
```

---

## 风险与避坑

* **别把巨大知识一股脑塞进 SKILL.md**：遵循“**渐进披露**”，把长文档放 `references/`，脚本放 `scripts/`。
* **自动触发≠放飞自我**：基础设施类项目虽智能，但**权限与审批**务必开好——尤其是写/删操作。
* **流程要统一**：安装 superpowers 后，**让团队共识“先计划后编码”**，否则“有人绕流程、有人按流程”会更乱。

## 最后：如何迅速落地到你的团队？

1. **先用 awesome-claude-skills 起步**：挑 3–5 个“痛点技能”放进你的项目。
2. **接入 infrastructure-showcase 的路由**：把“想起用哪个技能”变成**系统自动判断**。
3. **用 superpowers 统一本地流程**：任何交付都走“**脑暴→计划→TDD→实现**”。
4. **做一个小仪表盘**：每周复盘「计划命中率 / 单测覆盖 / 回滚事件」，驱动优化。

这是今天分享内容，希望这3个开源项目对你有帮助，感谢阅读。

  如果你对AI编程感兴趣，欢迎交流，进群领取吴哥AI编程手册详细资料福利。要是觉得今天这碗饭喂得够香，随手点个赞、在看、转发三连吧！