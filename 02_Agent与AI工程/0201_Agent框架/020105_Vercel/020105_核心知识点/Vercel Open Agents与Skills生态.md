# Vercel Open Agents 与 Skills 生态

## 原文锚点

- 本地文件：[Vercel 把云端 Agent 底盘开源了：为什么 Open Agents 值得所有做 AI 开发工具的人看一眼](<../文章/done-Vercel 把云端 Agent 底盘开源了：为什么 Open Agents 值得所有做 AI 开发工具的人看一眼.md>)；[AI 界的 npm 来了！Vercel 开源 add-skill，一行命令搞定所有 Agent 技能管理！](<../文章/done-AI 界的 npm 来了！Vercel 开源 add-skill，一行命令搞定所有 Agent 技能管理！.md>)；[Agent Skill 太多？Vercel 用排行榜帮你选](<../文章/done-Agent Skill 太多？Vercel 用排行榜帮你选.md>)；[推荐两个来自Vercel官方无敌好用的开发工具：agent-skills和agent-browser](<../文章/done-推荐两个来自Vercel官方无敌好用的开发工具：agent-skills和agent-browser.md>)
- 原文链接：本地文章包含 `vercel-labs/open-agents`、`vercel-labs/add-skill`、`vercel-labs/agent-skills`、`vercel-labs/agent-browser`、`skills.sh`，正式补证阶段再核对。
- 关键段落：Open Agents 的 Web/workflow/sandbox/GitHub 分层、agent is not the sandbox、add-skill 跨工具安装、skills.sh 排行、agent-browser。
- 关键图：Open Agents 文章提到截图和结构图，但本地 Markdown 未保留可用图片。

## 图片处理

| 图片 | 类型 | 是否保留 | 理由 | 处理方式 |
|---|---|---|---|---|
| Open Agents 仓库截图 | 配图/证据图 | 不进入核心知识点 | 只证明仓库存在，不解释机制 | 后续补官方链接 |
| Web/workflow/sandbox/GitHub 结构图 | 架构图 | 原图缺失 | 有助于理解控制面和执行面分离 | 在 Vercel AGENTS.md 用 Mermaid 重建 |

## 一句话结论

Vercel 在 Agent 工程里的新增量不是又做了一个助手，而是把云端 coding agent 的控制面、执行面、Skills 分发和浏览器执行工具拆成可参考的基础设施组件。

## 用户相关性判断

| 项 | 内容 |
|---|---|
| 用户当前认知层级 | Agent 工程 / Skills / AI 编程工具：L2-L3 |
| 认知成熟度 | draft |
| 阅读投入建议 | 精读 |
| 阅读投入理由 | 能补云端 Agent 平台和 Skills 分发的工程边界；但安全、版本、部署复杂度需要官方和本地实验补证 |
| 对用户的新信息 | Open Agents 的关键是 agent 不等于 sandbox；add-skill/skills.sh 则把技能从散落目录推向可安装、可排行、可复用 |
| 问题指纹 | Vercel + Open Agents/add-skill/agent-skills/agent-browser + workflow/sandbox/Skills 分发 + 云端 coding agent 与能力包管理 + 控制面执行面分离 |
| 排重判断 | 新建 |
| 置信度 | 中 |

## 认知校准点

| 校准点 | 文章观点/信息 | 与用户认知或价值观的关系 | 处理建议 |
|---|---|---|---|
| Agent 不应等同于沙箱 | Open Agents 把 workflow 控制面和 sandbox 执行环境分开 | 补长任务运行时和安全边界 | 以后看 coding agent 平台先问控制面是否可持久运行 |
| Skills 分发不是收藏列表 | add-skill 处理跨工具安装，skills.sh 做来源和热度发现 | 补 Skill 工程化边界 | 关注版本、权限、目录写入和回滚 |
| 浏览器工具是执行面组件 | agent-browser 给 Agent 提供网页操作能力 | 补工具调用与浏览器执行边界 | 需要会话隔离、截图/日志证据和权限控制 |
| 官方/品牌背书不等于质量保证 | skills.sh 来源明确，但仍需质量和安全验证 | 符合反标题党和证据优先 | 排行只能作为发现入口，不能作为采用依据 |
| Reference app 不是现成平台 | Open Agents 需要 OAuth、GitHub App、数据库、sandbox 配置 | 补落地成本 | 不把它写成即插即用方案 |

## 冲突点

| 冲突类型 | 具体表现 | 影响 | 处理 |
|---|---|---|---|
| 标题降权 | “AI 界的 npm”“无敌好用”等表达偏宣传 | 可能过度采信 | 保留机制，不保留口号 |
| 证据不足 | 官方状态、兼容工具数量、安装路径未本轮验证 | 可能过时 | 后续补证 |
| 安全边界缺失 | 安装 skills 会写入本机多工具配置目录 | 可能影响工具行为 | 后续实验必须记录写入路径和回滚 |
| 实践门槛不足 | 有命令但本轮未运行 | 不能判实践 | 降为精读 |

## 待吸收点

| 分级 | 内容 | 为什么值得吸收 | 后续动作 |
|---|---|---|---|
| 记住 | 云端 coding agent 的控制面不应塞进 sandbox | 决定持久运行、恢复和多沙箱演进能力 | 和长任务 Agent 运行时对齐 |
| 理解 | Open Agents = Web UI + durable workflow + sandbox + GitHub 集成 | 形成平台拆分视角 | 后续补部署链路 |
| 理解 | add-skill 把 Skills 安装从人工复制变成跨工具分发 | 解决技能碎片化 | 记录支持工具、目录写入和冲突处理 |
| 了解 | skills.sh 是技能发现和排行入口 | 可用于发现高质量技能 | 不把下载量当质量结论 |
| 理解 | agent-browser 是浏览器执行工具，不等于完整 Agent 框架 | 防止误分类 | 与 Playwright、Browser 插件横向对比 |

## 已知可跳过

| 内容 | 跳过理由 |
|---|---|
| Vercel 品牌叙事 | 不直接形成工程准则 |
| 下载量、排行榜热度 | 时效强，不能替代质量验证 |
| “一行命令搞定所有” | 忽略权限、冲突和回滚成本 |

## 实践门槛

| 门槛 | 判断 | 证据 |
|---|---|---|
| 可运行 | 部分 | 原文给出 add-skill / skills add 命令和 GitHub 项目 |
| 可验证 | 部分 | 可检查写入目录、安装结果、技能是否被工具识别 |
| 可排障 | 否 | 缺失败日志、冲突处理、回滚说明 |
| 可迁移 | 是 | 可迁移到用户本机 Codex/Claude/Cursor skill 管理 |
| 结论 | 降为精读 | 需要本地安装实验和安全审计后再判实践 |

## 归类判断

| 项 | 内容 |
|---|---|
| 技术本体 | Vercel Open Agents、add-skill、agent-skills、agent-browser |
| 文章主问题 | Vercel 如何组织云端 Agent 平台、技能分发和浏览器执行能力 |
| 使用场景 | 云端 coding agent、内部开发平台、AI 编程工具能力包管理、浏览器自动化 |
| 关键词干扰 | Vercel 部署、React、Next.js、GitHub 是生态和依赖，不是主类目 |
| 最终归类 | Agent 与 AI 工程 / Agent 框架 / Vercel |
| 归类理由 | 主问题是 Agent 运行底盘和能力包管理，不是普通前端部署或文档工具 |

## 技术定位

| 项 | 内容 |
|---|---|
| 技术类型 | Agent 基础设施 / Skills 分发生态 / 浏览器执行工具 |
| 所属领域 | Agent 与 AI 工程 |
| 二级类目 | Agent 框架 |
| 全局架构位置 | Agent 控制面、执行面和能力分发之间的基础设施层 |
| 涉及模块 | Web UI、durable workflow、sandbox VM、GitHub App、OAuth、add-skill、agent-skills、agent-browser、skills.sh |
| 解决问题 | 让 Agent 能在云端持续运行、接入代码仓库、隔离执行环境，并复用可安装技能 |
| 原文局限 | 未补官方版本、部署成本、安全策略和本地兼容实验 |
| 我的结论 | 以后关注，尤其适合作为 coding agent 平台和 skill 分发机制的参考 |

## 纵向理解

| 维度 | 判断 |
|---|---|
| 全局架构 | Web 负责交互，workflow 负责持续运行，sandbox 负责执行代码，GitHub 负责仓库闭环，skills 提供能力包 |
| 本文位置 | 云端 Agent 底盘和 Skills 分发，不是模型能力或普通部署 |
| 核心机制 | 控制面/执行面分离、跨工具安装、来源排行、浏览器执行 |
| 使用链路 | 安装/发现 skill -> 工具识别能力 -> Agent 通过 workflow 调度任务 -> sandbox/browser 执行 -> GitHub/预览/结果回传 |
| 前置条件 | OAuth、GitHub App、数据库、云沙箱、工具配置目录和回滚策略 |
| 边界 | 轻量本地任务不需要 Open Agents；技能安装需警惕权限和目录污染；浏览器执行需要证据链 |

## 横向对标

| 对标技术 | 实现方式 | 优势 | 劣势 | 适合场景 |
|---|---|---|---|---|
| 本地 Claude/Codex Skills | 本地目录和项目规则 | 控制直接、易审计 | 跨工具分发弱 | 单机或项目级规则 |
| Vercel add-skill | 跨工具安装 skill | 分发方便、适合多工具 | 需要安全和回滚机制 | 多 AI 编程工具共用技能 |
| 手工维护技能清单 | 人工复制和整理 | 简单透明 | 易漂移、难升级 | 少量稳定技能 |
| Open Agents | 云端 workflow + sandbox + GitHub | 长任务和组织接入更完整 | 部署复杂、云依赖强 | 内部 coding agent 平台 |
| 本地 IDE Agent | 本地会话和人工控制 | 低部署成本 | 持久运行和组织权限弱 | 个人开发辅助 |

## 后续追查

- 核验 `open-agents` 是否仍是 reference platform，部署依赖和 license 如何。
- 本地试跑 `add-skill` 前先记录会写入哪些目录，并准备回滚清单。
- 对比 `agent-browser` 与 Playwright/Browser 插件：会话、权限、截图证据、失败重试和可观测性。
