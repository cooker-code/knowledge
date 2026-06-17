# RAGFlow Markdown 语义切块机制

## 原文锚点

- 本地文件：[【解密源码】 RAGFlow 切分最佳实践 - General 模式语义切块（markdown 篇）](../文章/【解密源码】 RAGFlow 切分最佳实践 - General 模式语义切块（markdown 篇）.md)
- 原文链接：https://mp.weixin.qq.com/s?__biz=Mzk4ODg5NTU4Mg==&mid=2247483714&idx=1&sn=b5fb49f32204550dbeeab7ef4c84f879
- 关键段落：MarkdownParser、MarkdownElementExtractor、表格解析、图片提取、VisionFigureParser、naive_merge_with_images。
- 关键图：无架构图；原文主要是代码片段。

## 图片处理

| 图片 | 类型 | 是否保留 | 理由 | 处理方式 |
|---|---|---|---|---|
| 代码片段 | 说明图/代码 | 删除 | 本地 Markdown 已保留关键代码文本，图片不是必要锚点 | 用机制摘要替代 |

## 一句话结论

这篇文章对当前知识库非常相关：它说明 Markdown 文档切块不应简单按长度切，而要先保留标题、列表、代码、引用、表格和图片语义。

## 用户相关性判断

| 项 | 内容 |
|---|---|
| 用户当前认知层级 | RAG / 知识库 L2 draft |
| 认知成熟度 | draft |
| 阅读投入建议 | 精读 |
| 阅读投入理由 | 直接关联本地 Markdown 知识库的解析质量，但缺 RAGFlow 版本、测试集和召回指标 |
| 对用户的新信息 | RAGFlow Markdown 切块采用结构解析、表格处理、图片语义增强、文本合并和分词的流水线 |
| 问题指纹 | RAGFlow + MarkdownParser/MarkdownElementExtractor/VisionFigureParser + Markdown 结构化切块 + 本地知识库文档解析 |
| 排重判断 | 新建 |
| 置信度 | 高 |

## 认知校准点

| 校准点 | 文章观点/信息 | 与用户认知或价值观的关系 | 处理建议 |
|---|---|---|---|
| 切块不是 token 切割 | RAGFlow 先抽取标题、代码块、列表、引用和文本块 | 纠偏：RAG 质量不只在向量模型 | 写入 RAGFlow index |
| Markdown 是结构化输入 | Markdown 自带标题、列表、表格、代码块 | 补充：本地 knowledge 是 Markdown，应该利用结构 | 后续做 knowledge 切分实验 |
| 表格有两种处理模式 | 分离模式保留结构化输入，渲染模式保留视觉外观 | 补充：表格解析要按使用目标选 | 后续对复杂表格单独验证 |
| 图片语义会进入 chunk | 图片链接提取后用视觉模型生成描述并合并到 section | 补充：技术图可成为检索证据 | 与 AGENTS 图片处理规则对齐 |

## 冲突点

| 冲突类型 | 具体表现 | 影响 | 处理 |
|---|---|---|---|
| 实践判定偏宽 | 有源码片段，但没有版本、仓库 commit、测试数据和结果 | 不能直接判实践 | 降为精读 |
| 证据不足 | 没有对比固定切块、标题切块、RAGFlow 切块的召回指标 | 不能直接采信“最佳实践” | 标为待实验 |
| 标题降权 | 标题含“解密源码”“最佳实践” | 容易高估结论稳定性 | 只保留机制和待验证点 |

## 待吸收点

| 分级 | 内容 | 为什么值得吸收 | 后续动作 |
|---|---|---|---|
| 理解 | `MarkdownElementExtractor` 按标题、代码块、列表、引用、文本块的优先级抽取语义单元 | 解释 Markdown 切块如何保留结构 | 用于设计本地 knowledge 切分 |
| 理解 | 表格可走分离模式或渲染模式 | 表格检索和视觉复原需求不同 | 后续对技术对比表测试 |
| 记住 | Markdown 的标题层级是 chunk 上下文，不应该被丢掉 | 会影响检索命中和引用质量 | 写入 RAGFlow index |
| 记住 | 图片描述需要和所属 section 绑定，而不是孤立入库 | 与技术图保留策略一致 | 后续处理架构图 |
| 实践 | 用同一篇 knowledge Markdown 对比三种切分策略 | 这是判断是否采用 RAGFlow 的关键证据 | 待实验 |

## 已知可跳过

| 内容 | 跳过理由 |
|---|---|
| Markdown 有标题、列表、代码块 | 用户大概率知道 |
| RAGFlow 是 RAG 平台 | 已在 RAG/RAGFlow index 中覆盖 |
| 源码片段逐行解释 | 后续实践时再看源码即可 |

## 实践门槛

| 门槛 | 判断 | 证据 |
|---|---|---|
| 可运行 | 部分 | 有代码片段和函数名 |
| 可验证 | 否 | 无测试集、召回率、引用准确率 |
| 可排障 | 部分 | 能定位解析模块，但缺日志和错误分类 |
| 可迁移 | 是 | 直接适用于本地 Markdown 知识库 |
| 结论 | 降为精读 | 机制清楚，验证不足 |

## 归类判断

| 项 | 内容 |
|---|---|
| 技术本体 | RAGFlow 是 RAG 平台 |
| 文章主问题 | RAGFlow General 模式如何对 Markdown 做语义切块 |
| 使用场景 | Markdown 文档进入 RAG 知识库 |
| 关键词干扰 | 源码、图片、分词器 |
| 最终归类 | Agent 与 AI 工程 / RAG 与知识库 / RAGFlow |
| 归类理由 | 主问题是 RAG 文档解析和切块，不是通用 Markdown 工具 |

## 纵向理解

| 维度 | 判断 |
|---|---|
| 全局架构 | RAGFlow 文档解析 -> 语义切块 -> 图片增强 -> 分词 -> 向量化 |
| 本文位置 | 只讲 Markdown 类型的 General 模式切块 |
| 核心机制 | MarkdownElementExtractor、表格解析、图片提取、VisionFigureParser、merge/tokenize |
| 使用链路 | 读取 Markdown -> 提取语义单元 -> 处理表格和图片 -> 合并为 chunk -> 分词入库 |
| 前置条件 | 文档语法相对规范，图片可访问，视觉模型成本可接受 |
| 边界 | 不解决后续召回、重排和答案评估问题 |

## 横向对标

| 对标技术 | 实现方式 | 优势 | 劣势 | 适合场景 |
|---|---|---|---|---|
| 固定 token 切分 | 按长度截断 | 简单稳定 | 容易切断标题、表格、代码语义 | 长文本粗检索 |
| 按标题切分 | 用 Markdown 标题层级切块 | 保留结构 | 过长章节仍需二次切分 | 技术文档 |
| RAGFlow Markdown 切分 | 结构解析 + 表格 + 图片语义增强 | 更完整保留文档语义 | 成本和复杂度更高 | 含表格、图片、代码的知识库 |
| 手工 knowledge 摘要 | 人工提炼判断准则 | 认知校准强 | 覆盖慢 | 长期核心知识沉淀 |

## 后续追查

- 关键词：RAGFlow MarkdownParser、MarkdownElementExtractor、VisionFigureParser、semantic chunking、table parsing。
- 相关技术：RAGFlow General 模式、RAGFlow 召回策略、RAG 评估、LLM Wiki。
- 需要补读的文章：RAGFlow pdf/docx/book/paper 切分、RAGFlow 召回策略、Chunking 质量评估指标。
