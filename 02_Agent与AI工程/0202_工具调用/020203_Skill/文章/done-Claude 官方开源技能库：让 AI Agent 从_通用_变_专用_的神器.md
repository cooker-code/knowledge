> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: Claude 官方开源技能库：让 AI Agent 从"通用"变"专用"的神器
author: 红鱼AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzIxNTUxNDg4MA==&mid=2247484756&idx=6&sn=c0a62d97396dcae0640d11cc7e4e206c&chksm=96c8a9ce58f5a5ca15144c2e7715f74e078d4bbcd00f8130cb37beddcebb3f9c22579b2613d9&mpshare=1&scene=24&srcid=01156aq0lLJyhV4ODyMy56mQ&sharer_shareinfo=fe9fd89e7a21a660ebf0e066d9d10191&sharer_shareinfo_first=fe9fd89e7a21a660ebf0e066d9d10191#rd
---

项目链接：https://github.com/anthropics/skills

---

最近 AI Agent 这个词特别火，但说实话，大多数 Agent 还是"万金油"——什么都能干一点，但啥都不精。就像个只会写"Hello World"的程序员，遇到真实项目就歇菜了。

Anthropic 官方最近开源了一个叫"Skills"的项目，直接把这个问题给解决了。简单说，就是给 Claude 这类 AI 装上"技能包"，让它变成某个领域的专家。你想让它变成 Word 文档处理专家？装个技能。想让它变成 PPT 设计师？装个技能。想让它生成算法艺术？装个技能。

这玩意儿本质上就是一个文件夹，里面装着说明文档、脚本和资源，Claude 会动态加载这些技能来提升特定任务的表现。

## 一、项目核心：技能系统是什么？

在深入之前，先搞明白这项目到底是干嘛的。

这个仓库包含了各种技能示例，展示了 Claude 技能系统的可能性。范围从创意应用（艺术、音乐、设计）到技术任务（测试 web 应用、MCP 服务器生成），再到企业工作流（沟通、品牌管理）。

每个技能都是自包含的，放在自己的文件夹里，核心是一个 `SKILL.md` 文件，里面包含了指令和元数据，Claude 就靠这个文件来"学习"技能。

### 技能文件的结构

每个技能文件夹里最关键的就是 `SKILL.md`，这个文件有两个必需的字段：

1. `name`

   ：技能的唯一标识符（小写，空格用连字符）
2. `description`

   ：完整描述这个技能做什么、什么时候用

文件前面是 YAML 前置元数据，后面就是具体的指令、示例和指南。

### 技能的分类

看了一下项目结构，技能主要分为几大类：

* 文档处理技能：DOCX、PDF、PPTX、XLSX
* MCP 开发技能：MCP Builder
* 创意与设计技能：算法艺术、Slack GIF Creator、主题工厂、画布设计、前端设计、品牌指南
* 企业与沟通技能：内部沟通
* 测试技能：Web 应用测试

## 二、如何在 Claude 中使用这些技能

这部分是最实用的，好好看。

### 1. Claude Code 中使用

这个项目可以作为 Claude Code 的插件市场注册。在 Claude Code 里运行：

```
/plugin marketplace add anthropics/skills
```

```

```

然后安装特定的技能集：

1. 选择 `Browse and install plugins`
2. 选择 `anthropic-agent-skills`
3. 选择 `document-skills` 或 `example-skills`
4. 选择 `Install now`

或者直接安装：

```
/plugin install document-skills@anthropic-agent-skills/plugin install example-skills@anthropic-agent-skills
```

```

```

安装后，直接在对话中提技能名就行。比如你安装了 `document-skills` 插件，可以问 Claude："用 PDF 技能提取 `path/to/some-file.pdf` 的表单字段"

### 2. Claude.ai 中使用

这些示例技能在 Claude.ai 的付费计划里已经可用了。想用这个仓库里的技能或上传自定义技能，按照官方文档的说明操作就行。

### 3. Claude API 中使用

可以通过 Claude API 使用 Anthropic 预构建的技能，以及上传自定义技能。官方有 API 指南可以参考。

## 三、核心技能详解：文档处理三剑客

### DOCX 技能：Word 文档的专业处理

这个技能简直是办公利器，支持文档创建、编辑和分析，包括修订痕迹、评论、格式保留和文本提取。

**工作流程决策树：**

* 读取/分析内容 → 用文本提取或原始 XML 访问
* 创建新文档 → 用 "创建新 Word 文档" 工作流
* 编辑现有文档：

+ 你自己的文档 + 简单修改 → 用基础 OOXML 编辑工作流
+ 别人的文档 → 用修订痕迹工作流（推荐默认选择）
+ 法律、学术、商业或政府文档 → 必须用修订痕迹工作流

**创建新 Word 文档：**

从零创建 Word 文档时，使用 docx-js，允许用 JavaScript/TypeScript 创建 Word 文档。

工作流程：

1. **强制要求**

   ：完整阅读 `docx-js.md`（约 500 行），不要设置任何行数限制
2. 创建 JavaScript/TypeScript 文件，使用 Document、Paragraph、TextRun 组件
3. 用 Packer.toBuffer() 导出为 .docx

**编辑现有 Word 文档：**

编辑现有 Word 文档时，使用 Document 库（一个用于 OOXML 操作的 Python 库）。这个库自动处理基础设施设置，提供文档操作方法。

工作流程：

1. **强制要求**

   ：完整阅读 `ooxml.md`（约 600 行）
2. 解包文档：`python ooxml/scripts/unpack.py <office_file> <output_directory>`
3. 创建并运行 Python 脚本
4. 打包最终文档：`python ooxml/scripts/pack.py <input_directory> <office_file>`

**修订痕迹工作流：**

这个工作流允许在用 markdown 实施修订痕迹之前，规划全面的修订痕迹。**关键**：要获得完整的修订痕迹，必须系统地实施所有更改。

分批策略：将相关更改组织成 3-10 个更改的批次。这使调试可管理，同时保持效率。每批测试后再进行下一批。

原则：最少、精确的编辑

实施修订痕迹时，只标记实际更改的文本。重复未更改的文本会使编辑更难审查，看起来不专业。将替换分解为：[未更改文本] + [删除] + [插入] + [未更改文本]。通过保留原始运行的 RSID 来保留未更改文本的格式。

示例 - 将句子中的 "30 天" 改为 "60 天"：

```
坏的 - 替换整句话'<w:del><w:r><w:delText>期限为30天。</w:delText></w:r></w:del><w:ins><w:r><w:t>期限为60天。</w:t></w:r></w:ins>'好的 - 只标记更改的内容，保留原始 <w:r> 用于未更改文本'<w:r w:rsidR="00AB12CD"><w:t>期限为 </w:t></w:r><w:del><w:r><w:delText>30</w:delText></w:r></w:del><w:ins><w:r><w:t>60</w:t></w:r></w:ins><w:r w:rsidR="00AB12CD"><w:t> 天。</w:t></w:r>'
```

```

```

修订痕迹工作流步骤：

1. 获取 markdown 表示：将文档转换为 markdown 并保留修订痕迹：

   ```
   pandoc --track-changes=all path-to-file.docx -o current.md
   ```
2. 识别和分组更改：审查文档并识别所有需要的更改，组织成逻辑批次：

   定位方法（在 XML 中查找更改）：

   批次组织（每批 3-10 个相关更改）：

* 按章节："批次 1：第 2 节修正"、"批次 2：第 5 节更新"
* 按类型："批次 1：日期更正"、"批次 2：当事人名称更改"
* 按复杂性：从简单的文本替换开始，然后处理复杂的结构更改
* 顺序："批次 1：第 1-3 页"、"批次 2：第 4-6 页"

* 章节/标题编号（如 "3.2 节"、"第四条"）
* 段落标识符（如果编号）
* 用唯一周围文本的 grep 模式
* 文档结构（如 "第一段"、"签名块"）
* 不要使用 markdown 行号——它们不映射到 XML 结构

3. 阅读文档并解包：

* **强制要求**

  ：完整阅读 `ooxml.md`（约 600 行），特别注意 "Document Library" 和 "Tracked Change Patterns" 部分
* 解包文档：`python ooxml/scripts/unpack.py <file.docx> <dir>`
* 注意建议的 RSID：解包脚本会建议一个用于修订痕迹的 RSID。复制这个 RSID 在步骤 4b 中使用

4. 分批实施更改：将更改按逻辑分组（按章节、类型或邻近度），在单个脚本中一起实施。这种方法：

* 使调试更容易（批次小 = 更容易隔离错误）
* 允许增量进度
* 保持效率（3-10 个更改的批次大小效果很好）

5. 打包文档：所有批次完成后，将解包的目录转换回 .docx：

   ```
   python ooxml/scripts/pack.py unpacked reviewed-document.docx
   ```
6. 最终验证：对完整文档进行全面检查：

* 将最终文档转换为 markdown：

  ```
  pandoc --track-changes=all reviewed-document.docx -o verification.md
  ```
* 验证所有更改都正确应用：

  ```
  grep "原短语" verification.md  # 不应该找到grep "替换短语" verification.md  # 应该找到
  ```
* 检查是否引入了意外更改

### PDF 技能：PDF 处理的瑞士军刀

这个技能是综合 PDF 操作工具包，用于提取文本和表格、创建新 PDF、合并/拆分文档以及处理表单。

**快速开始：**

```
from pypdf import PdfReader, PdfWriter# 读取 PDFreader = PdfReader("document.pdf")print(f"页数: {len(reader.pages)}")# 提取文本text = ""for page in reader.pages:    text += page.extract_text()
```

**合并 PDF：**

```
from pypdf import PdfWriter, PdfReaderwriter = PdfWriter()for pdf_file in ["doc1.pdf", "doc2.pdf", "doc3.pdf"]:    reader = PdfReader(pdf_file)    for page in reader.pages:        writer.add_page(page)with open("merged.pdf", "wb") as output:    writer.write(output)
```

**拆分 PDF：**

```
reader = PdfReader("input.pdf")for i, page in enumerate(reader.pages):    writer = PdfWriter()    writer.add_page(page)    with open(f"page_{i+1}.pdf", "wb") as output:        writer.write(output)
```

**提取表格：**

```
import pandas as pdwith pdfplumber.open("document.pdf") as pdf:    all_tables = []    for page in pdf.pages:        tables = page.extract_tables()        for table in tables:            if table:  # 检查表格是否为空                df = pd.DataFrame(table[1:], columns=table[0])                all_tables.append(df)# 合并所有表格if all_tables:    combined_df = pd.concat(all_tables, ignore_index=True)    combined_df.to_excel("extracted_tables.xlsx", index=False)
```

### PPTX 技能：演示文稿设计专家

这个技能支持演示文稿创建、编辑和分析，包括布局、评论和演讲者笔记。

**无模板创建新演示文稿：**

从零创建新演示文稿时，使用 html2pptx 工作流将 HTML 幻灯片转换为 PowerPoint 并准确定位。

设计原则：在创建任何演示文稿之前，先分析内容并选择合适的设计元素：

1. 考虑主题：演示文稿是关于什么的？什么基调、行业或情绪？
2. 检查品牌：如果用户提到公司/组织，考虑他们的品牌颜色和身份
3. 将调色板与内容匹配：选择反映主题的颜色
4. 说明你的方法：在编写代码之前解释你的设计选择

工作流程：

1. **强制要求**

   ：完整阅读 `html2pptx.md`
2. 为每张幻灯片创建一个 HTML 文件，使用正确的尺寸（如 16:9 的 720pt × 405pt）

* 使用 `<p>`、`<h1>`-`<h6>`、`<ul>`、`<ol>` 处理所有文本内容
* 使用 `class="placeholder"` 表示图表/表格将添加的区域
* 用 Sharp 先将渐变和图标栅格化为 PNG 图像，然后在 HTML 中引用
* 对于有图表/表格/图像的幻灯片，使用全幻灯片布局或双列布局以获得更好的可读性

3. 创建并运行 JavaScript 文件，使用 `html2pptx.js` 库
4. 视觉验证：生成缩略图并检查布局问题

## 四、创意技能：算法艺术生成

这个技能使用 p5.js 和种子随机性创建算法艺术，支持交互式参数探索。

**算法哲学创建：**

首先创建一个算法哲学（不是静态图像或模板），通过以下方式解释：

* 计算过程、涌现行为、数学之美
* 种子随机性、噪声场、有机系统
* 粒子、流、场、力
* 参数变化和受控混沌

**实现步骤：**

1. 命名运动（1-2 个词）："有机湍流"/"量子和谐"/"涌现静止"
2. 阐述哲学（4-6 段——简洁但完整）
3. 通过代码表达：创建 p5.js 草图，90% 算法生成，10% 基本参数

**关键原则：**

* 算法哲学：创建一个通过代码表达的计算世界观
* 过程胜于产品：始终强调美从算法执行中涌现——每次运行都是独特的
* 参数化表达：思想通过数学关系、力、行为交流——不是静态组合
* 艺术自由：下一个 Claude 从算法角度解释哲学——提供创意实施空间
* 纯生成艺术：这是关于制作活的算法，不是带随机性的静态图像

## 五、如何创建自己的技能

这个项目的精髓不只是用现成的技能，而是能创建自己的技能。

**创建基本技能：**

技能创建很简单——就是一个带有 `SKILL.md` 文件的文件夹，文件包含 YAML 前置元数据和指令。

使用仓库中的 template-skill 作为起点：

```
---name: my-skill-namedescription: 技能功能的清晰描述以及何时使用它---# 我的技能名称[在这里添加 Claude 在此技能激活时将遵循的指令]## 示例- 示例用法 1- 示例用法 2## 指南- 指南 1- 指南 2
```

前置元数据只需要两个字段：

* `name`

  ：技能的唯一标识符（小写，空格用连字符）
* `description`

  ：技能功能的完整描述以及何时使用它

markdown 内容包含指令、示例和指南。

## 六、实际应用场景

### 场景一：法律文档批量处理

律师事务所需要定期审查合同模板，批量替换条款并保留修订痕迹。使用 DOCX 技能的修订痕迹工作流，可以：

1. 将数百份合同解包
2. 按条款类型分组更改（日期、当事人名称、法律条款）
3. 分批实施修订，每批 3-10 个更改
4. 自动验证所有更改是否正确应用
5. 重新打包并输出

### 场景二：财务报告自动生成

财务团队需要每月将 Excel 数据转换为专业的 PDF 报告，包括：

* 用 PPTX 技能创建演示文稿，使用专业的配色方案
* 用 PDF 技能提取数据源中的表格
* 用 DOCX 技能生成详细的文字报告
* 所有步骤都保持一致的品牌风格

### 场景三：学术论文协作

学术团队在研究过程中需要：

1. 用 PDF 技能从大量 PDF 文献中提取表格和数据
2. 用 DOCX 技能管理草稿，通过修订痕迹追踪所有更改
3. 用 PPTX 技能创建会议演示，使用学术风格的配色方案
4. 用 XLSX 技能进行数据分析和重新计算

### 场景四：营销材料批量生产

营销部门需要为客户批量生成定制化材料：

1. 用算法艺术技能为不同客户生成独特的视觉元素
2. 用 PPTX 技能创建模板演示文稿，然后批量替换内容
3. 用品牌指南技能确保所有输出符合客户的视觉识别
4. 用 PDF 技能合并和输出最终文档

### 场景五：内部知识库建设

企业需要将各种文档转换为统一的知识库：

* 用 PDF 技能提取大量 PDF 文档的文本和表格
* 用 DOCX 技能将内容整理成标准格式
* 用 PPTX 技能创建培训材料
* 用主题工厂技能应用统一的品牌主题

这个项目把 AI Agent 从"通用工具"变成了"专业工程师"，每个技能都像给 Claude 加了个专业认证。最妙的是，它还是开源的，你可以直接学官方怎么写技能，然后自己造。

技术这东西，最后拼的不是谁会玩，而是谁会造。