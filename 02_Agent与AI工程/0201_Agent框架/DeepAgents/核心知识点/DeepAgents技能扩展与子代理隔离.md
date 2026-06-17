# DeepAgents 技能扩展与子代理隔离

## 原文锚点

- 本地文件：[LangChain Deep Agents技能扩展全指南：从Anthropic Skills到子代理封装](../文章/LangChain Deep Agents技能扩展全指南：从Anthropic Skills到子代理封装.md)
- 原文链接：`https://mp.weixin.qq.com/s?__biz=MzIzNTcxOTU3OA==&mid=2247487835&idx=1&sn=3c50b7ee238686c11806bc3b1943955c`
- 关键段落：SkillsMiddleware、渐进式披露、SkillRegistry、`CompiledSubAgent`、工具封装、PythonREPLTool 安全提示、主代理路由。
- 关键图：原文无可用技术图，主要是代码和流程描述。

## 图片处理

| 图片 | 类型 | 是否保留 | 理由 | 处理方式 |
|---|---|---|---|---|
| 无 | 无 | 不适用 | 本地 Markdown 未保留技术图 | Mermaid 重建见技术 index |

## 一句话结论

这篇文章值得精读但要降权 PDF 业务细节：DeepAgents 的核心增量是把技能做成“菜单 + 按需读取 + 子代理隔离”，而不是把所有脚本和说明一次性塞进主 Agent 上下文。

## 用户相关性判断

| 项 | 内容 |
|---|---|
| 用户当前认知层级 | Agent 工程 / Skill / 子代理：L2-L3 |
| 认知成熟度 | draft |
| 阅读投入建议 | 精读 |
| 阅读投入理由 | 能补 DeepAgents、Skills、SubAgent 的边界，但原文实现偏示例工程，核心包能力和版本需后续补证 |
| 对用户的新信息 | SkillRegistry 只注入技能摘要和路径，真正执行时再读 SKILL.md；复杂能力用 CompiledSubAgent 隔离 |
| 问题指纹 | DeepAgents + Skills/CompiledSubAgent + 渐进式披露/工具封装 + 能力扩展和上下文控制 + 防止主 Agent 工具膨胀 |
| 排重判断 | 新建 |
| 置信度 | 中 |

## 认知校准点

| 校准点 | 文章观点/信息 | 与用户认知或价值观的关系 | 处理建议 |
|---|---|---|---|
| 技能不是直接注入全文 | CLI 只把技能列表和路径放入系统提示词，执行时再读取 | 与当前 Codex skill 使用方式一致 | 作为 Skills 机制排重准则 |
| 技能可以封装为子代理 | 一个 Skill 对应一个 CompiledSubAgent，隔离工具和上下文 | 补充多 Agent 协作边界 | 复杂能力优先子代理隔离，不让主 Agent 直接持有全部工具 |
| 核心包能力未必等同 CLI | 原文称核心 Python 包当时未直接支持 Skills | 需要版本补证 | 不写死为当前事实，后续查官方 |
| PythonREPLTool 是风险点 | 原文提示生产应替换为沙箱 | 符合安全偏好 | 任何动态执行能力都要标注最小权限和沙箱 |
| PDF 示例不应抢主类目 | PDF 是演示能力，不是 Agent 框架本体 | 防止误归到文档工具 | 只保留架构机制 |

## 冲突点

| 冲突类型 | 具体表现 | 影响 | 处理 |
|---|---|---|---|
| 关键词误导 | 大量 PDF 处理代码、Anthropic Skills、LangChain 工具 | 可能误归到文档工具或工具调用 | 主问题按 DeepAgents 技能扩展归类 |
| 证据不足 | “核心包截至某日期不支持 Skills”未官方补证 | 可能过时 | 标为后续补证 |
| 实践门槛不足 | 代码多但依赖复杂，未本地运行 | 不能直接判实践 | 降为精读 |
| 安全风险 | PythonREPLTool 可执行本地任意代码 | 生产误用风险 | 写入安全边界，不推荐直接生产使用 |

## 待吸收点

| 分级 | 内容 | 为什么值得吸收 | 后续动作 |
|---|---|---|---|
| 理解 | SkillRegistry 扫描 SKILL.md frontmatter 生成技能菜单 | 解决技能发现和上下文控制 | 对比 Codex skill 机制 |
| 记住 | 渐进式披露 = 先暴露摘要，命中后再读完整指令 | 防止上下文膨胀 | 写入技能/工具扩展准则 |
| 理解 | CompiledSubAgent 封装专业能力 | 隔离工具范围和专业上下文 | 后续和多 Agent 协作对标 |
| 记住 | 动态代码执行必须进入沙箱 | 防止 PythonREPLTool 变成安全洞 | 关联 Agent 安全与权限 |
| 了解 | 主代理需要显式路由提示 | 降低委派错误 | 后续实验验证路由准确性 |

## 已知可跳过

| 内容 | 跳过理由 |
|---|---|
| PDF 工具函数完整实现 | 与 Agent 框架主问题无关 |
| 具体 watermark 演示输出 | 只是功能演示 |
| 安装依赖细节 | 本轮不运行、不安装 |
| “像人类学习新技能”表述 | 类比价值低，保留机制即可 |

## 实践门槛

| 门槛 | 判断 | 证据 |
|---|---|---|
| 可运行 | 部分 | 有工具封装、子代理和主代理代码片段 |
| 可验证 | 部分 | 有示例任务输出，但不是本地复现 |
| 可排障 | 部分 | 提到 import 失败、REPL 风险，但缺日志策略 |
| 可迁移 | 是 | 可迁移到用户技能库和文章处理子代理 |
| 结论 | 降为精读 | 后续补最小子代理实验后再判实践 |

## 归类判断

| 项 | 内容 |
|---|---|
| 技术本体 | DeepAgents |
| 文章主问题 | 如何在 DeepAgents 中用 Skills 和 SubAgent 扩展专业能力 |
| 使用场景 | PDF 专家、搜索专家、RAG 专家、主代理委派 |
| 关键词干扰 | PDF、Anthropic Skills、PythonREPLTool 是示例和依赖 |
| 最终归类 | Agent 与 AI 工程 / Agent 框架 / DeepAgents |
| 归类理由 | 主问题是 Agent 框架能力扩展和上下文隔离 |

## 技术定位

| 项 | 内容 |
|---|---|
| 技术类型 | Agent 框架 / 子代理编排 |
| 所属领域 | Agent 与 AI 工程 |
| 二级类目 | Agent 框架 |
| 全局架构位置 | 主 Agent 和专业工具之间的能力封装层 |
| 涉及模块 | SkillsMiddleware、SkillRegistry、SKILL.md、CompiledSubAgent、工具封装、PythonREPLTool |
| 解决问题 | 让 Agent 可扩展专业能力，同时避免上下文和工具权限无限膨胀 |
| 原文局限 | 版本状态未补证，示例偏 PDF，缺失败和评估设计 |
| 我的结论 | 以后关注，优先追查核心包和 CLI 的能力边界 |

## 纵向理解

| 维度 | 判断 |
|---|---|
| 全局架构 | 主 Agent 识别任务 -> 技能菜单提示可用能力 -> 子代理读取 SKILL.md -> 调用受限工具 -> 返回结果 |
| 本文位置 | 技能扩展和子代理隔离，不是 DeepAgents 全架构 |
| 核心机制 | 文档驱动的渐进式披露加静态子代理封装 |
| 使用链路 | 准备 skill 目录 -> 解析 frontmatter -> 注入技能菜单 -> 命中后读 SKILL.md -> 子代理执行工具 |
| 前置条件 | 技能描述准确、工具参数清晰、执行环境隔离、主代理路由提示明确 |
| 边界 | 技能过多仍需检索/排序；动态执行需要沙箱；子代理增加调试成本 |

## 横向对标

| 对标技术 | 实现方式 | 优势 | 劣势 | 适合场景 |
|---|---|---|---|---|
| 直接工具列表 | 所有工具暴露给主 Agent | 简单 | 上下文和权限膨胀 | 工具少、任务简单 |
| Skills 文档 | 按需读取 SKILL.md | 上下文节省、规则清晰 | 依赖模型正确触发 | 专业流程说明 |
| CompiledSubAgent | 专业子代理持有工具 | 隔离能力和上下文 | 多一层路由 | 复杂专业任务 |
| MCP 工具 | 协议化外部工具 | 统一连接 | 本文主问题不是 MCP | 外部服务接入 |

## 后续追查

- 关键词：DeepAgents CLI、SkillsMiddleware、SkillRegistry、CompiledSubAgent、Progressive Disclosure。
- 相关技术：Claude Skills、Codex Skills、LangGraph Subgraph、多 Agent 协作。
- 需要补读的文章：DeepAgents 官方文档、核心包 API、CLI 源码；本轮不联网，后续补证。
