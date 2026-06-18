> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 给AI装"技能包"：Agent Skills从入门到精通（下）
author: X Agentic
date:
url: https://mp.weixin.qq.com/s?__biz=MzYzODA4ODUxOQ==&mid=2247483833&idx=1&sn=f1e091cc56ccc6074d179feaa14d2e61&chksm=f154d73d2f619823dcf48ac482a4360729f1ea9dc4f537469c05772962a1e99d5806434e97f9&mpshare=1&scene=24&srcid=0108I2QVC9VNN1lDzXEdIoxs&sharer_shareinfo=5aba5ea12508c977c6431beb3f2135bc&sharer_shareinfo_first=5aba5ea12508c977c6431beb3f2135bc#rd
---

> 💡 上篇我们聊了Skills的基础概念和如何创建第一个Skill。今天我们来深入探讨高级技巧：如何设计工作流、编写脚本、评估Skill质量，以及如何与Claude协作开发高质量的Skills。

## 🔄 第一部分：工作流和反馈循环

### ❓ 为什么需要工作流？

简单任务可能只需要一个指令，但复杂任务需要：

* 📋 多个步骤按顺序执行
* ✅ 每个步骤的验证
* ⚠️ 错误处理和恢复
* 📊 进度跟踪

### ✅ 工作流模式：检查清单

对于复杂任务，提供可复制的检查清单，让Agent可以跟踪进度。

**示例：PDF表单填充工作流**

```
## PDF form filling workflow

Copy this checklist and check off items as you complete them:


  Task Progress:
  - [ ] Step 1: Analyze the form (run analyze_form.py)
  - [ ] Step 2: Create field mapping (edit fields.json)
  - [ ] Step 3: Validate mapping (run validate_fields.py)
  - [ ] Step 4: Fill the form (run fill_form.py)
  - [ ] Step 5: Verify output (run verify_output.py)


**Step 1: Analyze the form**

Run: `python scripts/analyze_form.py input.pdf`

This extracts form fields and their locations, saving to `fields.json`.

**Step 2: Create field mapping**

Edit `fields.json` to add values for each field.

**Step 3: Validate mapping**

Run: `python scripts/validate_fields.py fields.json`

Fix any validation errors before continuing.

**Step 4: Fill the form**

Run: `python scripts/fill_form.py input.pdf fields.json output.pdf`

**Step 5: Verify output**

Run: `python scripts/verify_output.py output.pdf`

If verification fails, return to Step 2.
```

**为什么有效？**

* 🎯 清晰的步骤防止Agent跳过关键验证
* 📋 检查清单帮助Agent和用户跟踪进度
* 🔄 明确的错误处理路径

### 🔁 反馈循环模式：验证-修复-重复

对于质量关键的任务，实现反馈循环。

**示例：文档编辑流程**

```
## Document editing process

1. Make your edits to `word/document.xml`
2. **Validate immediately**: `python ooxml/scripts/validate.py unpacked_dir/`
3. If validation fails:
   - Review the error message carefully
   - Fix the issues in the XML
   - Run validation again
4. **Only proceed when validation passes**
5. Rebuild: `python ooxml/scripts/pack.py unpacked_dir/ output.docx`
6. Test the output document
```

**为什么有效？**

* ⚡ 早期发现错误，避免后续问题
* 🤖 机器可验证，提供客观检查
* 📝 清晰的错误消息指导修复

### 🔀 条件工作流：根据情况选择路径

```
## Document modification workflow

1. Determine the modification type:

   **Creating new content?** → Follow "Creation workflow" below
   **Editing existing content?** → Follow "Editing workflow" below

2. Creation workflow:
   - Use docx-js library
   - Build document from scratch
   - Export to .docx format

3. Editing workflow:
   - Unpack existing document
   - Modify XML directly
   - Validate after each change
   - Repack when complete
```

## 💻 第二部分：代码和脚本的最佳实践

### ❓ 何时提供脚本？

**提供脚本的情况**：

* ⚠️ 操作脆弱且容易出错
* 🔄 需要一致性（每次执行相同）
* 💰 节省token（脚本执行不加载到上下文）

**不提供脚本的情况**：

* 🎯 任务灵活，需要根据上下文调整
* 🤖 Agent可以轻松生成代码
* 📝 代码很简单，不值得维护

### 📋 脚本设计原则

#### 1️⃣ 原则1：解决问题，不要推给Claude

**❌ 错误：推给Claude**

```
def process_file(path):
    # Just fail and let Claude figure it out
    return open(path).read()
```

**✅ 正确：处理错误**

```
def process_file(path):
    """Process a file, creating it if it doesn't exist."""
    try:
        withopen(path) as f:
            return f.read()
    except FileNotFoundError:
        # Create file with default content instead of failing
        print(f"File {path} not found, creating default")
        withopen(path, 'w') as f:
            f.write('')
        return''
    except PermissionError:
        # Provide alternative instead of failing
        print(f"Cannot access {path}, using default")
        return''
```

#### 2️⃣ 原则2：避免"魔法数字"

**❌ 错误：未解释的常量**

```
TIMEOUT = 47  # Why 47?
RETRIES = 5   # Why 5?
```

**✅ 正确：自文档化**

```
# HTTP requests typically complete within 30 seconds
# Longer timeout accounts for slow connections
REQUEST_TIMEOUT = 30

# Three retries balances reliability vs speed
# Most intermittent failures resolve by the second retry
MAX_RETRIES = 3
```

#### 3️⃣ 原则3：提供清晰的错误消息

**❌ 错误：模糊的错误**

```
if not field:
    raise ValueError("Invalid field")
```

**✅ 正确：具体的错误消息**

```
if not field:
    raise ValueError(
        f"Field 'signature_date' not found. "
        f"Available fields: {', '.join(available_fields)}"
    )
```

### 📋 计划-验证-执行模式

对于复杂、开放式的任务，使用这个模式来早期捕获错误。

**场景**：更新PDF中的50个表单字段

**问题**：没有验证，Agent可能：

* ❌ 引用不存在的字段
* ⚠️ 创建冲突的值
* 📋 遗漏必需字段
* 🔧 错误应用更新

**解决方案**：添加中间验证步骤

```
## Batch form update workflow

1. Analyze the form: `python scripts/analyze_form.py input.pdf`
2. **Create plan file**: Create `changes.json` with all updates
3. **Validate plan**: `python scripts/validate_changes.py changes.json`
4. If validation fails:
   - Review error messages
   - Fix issues in changes.json
   - Validate again
5. **Only proceed when validation passes**
6. Execute: `python scripts/apply_changes.py input.pdf changes.json output.pdf`
7. Verify: `python scripts/verify_output.py output.pdf`
```

**为什么有效？**

* ⚡ 早期捕获错误，在应用更改之前
* 🤖 机器可验证，提供客观检查
* 🔄 可逆规划，可以迭代计划而不影响原始文件
* 🐛 清晰的调试，错误消息指向具体问题

### ⚙️ 脚本执行 vs 参考

在Skill中明确说明脚本的用途：

**执行脚本**（最常见）：

```
## Utility scripts

**analyze_form.py**: Extract all form fields from PDF

python scripts/analyze_form.py input.pdf > fields.json
```

**作为参考**（复杂逻辑）：

```
## Field extraction algorithm

See `scripts/analyze_form.py` for the complete field extraction algorithm.
Use this as reference when you need to understand the extraction logic.
```

## 📊 第三部分：评估驱动开发

### ❓ 为什么先评估？

**常见错误**：先写大量文档，再测试

**正确方法**：先创建评估，再写最小化指令

**为什么？**

* 🎯 确保解决真实问题，而不是想象的问题
* 📈 建立基线，衡量改进效果
* 🚫 避免过度工程

### 评估结构

```
{
  "skills":["pdf-processing"],
"query":"Extract all text from this PDF file and save it to output.txt",
"files":["test-files/document.pdf"],
"expected_behavior":[
    "Successfully reads the PDF file using an appropriate PDF processing library",
    "Extracts text content from all pages without missing any pages",
    "Saves the extracted text to a file named output.txt in a clear, readable format"
]
}
```

### 🔄 评估驱动开发流程

1. **🔍 识别差距**

   ：在没有Skill的情况下运行Claude，记录失败
2. **📝 创建评估**

   ：构建3个测试这些差距的场景
3. **📊 建立基线**

   ：测量没有Skill时的性能
4. **✏️ 编写最小指令**

   ：只写足够通过评估的内容
5. **🔄 迭代**

   ：执行评估，与基线比较，改进

**示例**：

**1️⃣ 步骤1：识别差距**

* 📋 任务：提取PDF文本
* ❌ 问题：Claude使用了错误的库，提取不完整

**2️⃣ 步骤2：创建评估**

```
{
  "query": "Extract text from document.pdf",
  "expected": [
    "Uses pdfplumber library",
    "Extracts all pages",
    "Handles encoding correctly"
  ]
}
```

**3️⃣ 步骤3：建立基线**

* 📊 成功率：40%（2/5测试通过）

**4️⃣ 步骤4：编写最小指令**

```
## Extract text

Use pdfplumber:

import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    for page in pdf.pages:
        text = page.extract_text()
```

**5️⃣ 步骤5：迭代**

* 🧪 测试：成功率提升到80%
* 🔧 改进：添加错误处理
* ✅ 再测试：成功率100%

## 🤝 第四部分：与Claude协作开发Skills

### ❓ 为什么与Claude协作？

Claude理解：

* 📝 如何编写有效的Agent指令
* 📚 Agent需要什么信息
* 🗂️ 如何组织内容以便Agent使用

### 🔄 开发流程

#### 1️⃣ 阶段1：完成任务（无Skill）

与Claude A一起完成任务，使用正常提示。注意：

* 🔁 你反复提供什么信息？
* 💡 你解释了什么偏好？
* 📚 你分享了什么流程知识？

**示例**：BigQuery分析任务

* 📊 提供了表名、字段定义
* 🔍 解释了过滤规则（"总是排除测试账户"）
* 📝 分享了常见查询模式

#### 2️⃣ 阶段2：识别可复用模式

完成任务后，识别哪些上下文对类似任务有用：

* 📊 表模式
* 🏷️ 命名约定
* 🔍 过滤规则
* 📝 查询模式

#### 3️⃣ 阶段3：让Claude创建Skill

直接要求Claude创建Skill：

```
"Create a Skill that captures this BigQuery analysis pattern we just used. 
Include the table schemas, naming conventions, and the rule about filtering 
test accounts."
```

Claude理解Skill格式，会生成正确结构的内容。

#### 4️⃣ 阶段4：审查简洁性

检查Claude是否添加了不必要的解释：

```
"Remove the explanation about what win rate means - Claude already knows that."
```

#### 5️⃣ 阶段5：改进信息架构

组织内容更有效：

```
"Organize this so the table schema is in a separate reference file. 
We might add more tables later."
```

#### 6️⃣ 阶段6：测试

使用Claude B（加载了Skill的新实例）测试相关用例：

* 🎯 Skill是否在预期时激活？
* 📝 指令是否清晰？
* ❓ 是否缺少什么？

#### 7️⃣ 阶段7：迭代

如果Claude B遇到问题，返回Claude A改进：

```
"When Claude used this Skill, it forgot to filter by date for Q4. 
Should we add a section about date filtering patterns?"
```

### 🔄 迭代现有Skills

同样的模式适用于改进现有Skills：

1. **🧪 使用Skill**

   ：给Claude B（加载了Skill）实际任务
2. **👀 观察行为**

   ：注意它在哪里挣扎、成功、做出意外选择
3. **💬 返回改进**

   ：与Claude A分享观察，请求改进
4. **✅ 应用和测试**

   ：更新Skill，再次测试

**示例观察**：

```
"When I asked Claude B for a regional sales report, it wrote the query 
but forgot to filter out test accounts, even though the Skill mentions this rule."
```

**改进请求**：

```
"I noticed Claude B forgot to filter test accounts when I asked for a regional 
report. The Skill mentions filtering, but maybe it's not prominent enough?"
```

**Claude A的建议**：

* 重新组织，使规则更突出
* 使用更强的语言（"MUST filter"而不是"always filter"）
* 重构工作流部分

### 👀 观察Claude如何使用Skills

注意：

* **🔍 意外的探索路径**

  ：Claude是否以你未预期的顺序读取文件？
* **🔗 错过的连接**

  ：Claude是否未能遵循重要文件的引用？
* **📖 过度依赖某些部分**

  ：如果Claude反复读取同一文件，考虑是否应该放在主SKILL.md中
* **🚫 忽略的内容**

  ：如果Claude从未访问某个文件，可能是不必要的或信号不足

基于这些观察迭代，而不是假设。

## 💡 第五部分：真实案例

### 📄 案例1：PDF处理Skill（完整示例）

**结构**：

```
pdf-processing/
├── SKILL.md
├── references/
│   ├── FORMS.md
│   ├── REFERENCE.md
│   └── EXAMPLES.md
└── scripts/
    ├── analyze_form.py
    ├── validate_fields.py
    ├── fill_form.py
    └── verify_output.py
```

**SKILL.md**：

```
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files.

# PDF Processing

## Quick start

Extract text:

import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## Advanced features

**Form filling**: See FORMS.md
**API reference**: See REFERENCE.md
**Examples**: See EXAMPLES.md

## Workflow

See FORMS.md for complete form filling workflow.

### 📊 案例2：BigQuery数据分析Skill

**结构**：

```
bigquery-skill/
├── SKILL.md
└── reference/
    ├── finance.md
    ├── sales.md
    └── product.md
```

**设计亮点**：

* 🗂️ 按领域组织，按需加载
* 🎯 用户询问销售指标时，只加载`sales.md`
* 💰 节省token，保持上下文聚焦

### 🧠 案例3：Digital Brain Skill（复杂系统）

这是一个完整的个人操作系统，展示了Skills的高级应用。

**三级加载模式**：

```
L1: SKILL.md（总是加载，~50 tokens）
  ↓
L2: MODULE.md（任务时加载，~80 tokens）
  ↓
L3: data files（需要时加载，~200 tokens）
```

**模块隔离**：

* 📦 6个独立模块（identity, content, knowledge, network, operations, agents）
* 🔄 每个模块独立加载
* 🛡️ 防止上下文污染

**Token优化效果**：

* 📊 优化前：~5000 tokens（加载所有内容）
* ⚡ 优化后：~650 tokens（按需加载）
* **💰 节省87%的token使用**

**关键设计决策**：

1. 📝 使用JSONL格式存储数据（append-only，易于解析）
2. 📋 Schema-first设计（第一行定义结构）
3. 📚 渐进式披露（三级加载）
4. 🔒 模块隔离（防止上下文污染）

## ⚠️ 第六部分：常见反模式和最佳实践

### 🚫 反模式1：Windows风格路径

**❌ 错误**：

```
See scripts\helper.py
```

**✅ 正确**：

```
See scripts/helper.py
```

**为什么？** Unix风格路径在所有平台工作，Windows风格路径在Unix系统会失败。

### 🚫 反模式2：提供过多选项

**❌ 错误**：

```
You can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image, or...
```

**✅ 正确**：

```
Use pdfplumber for text extraction:

import pdfplumber

For scanned PDFs requiring OCR, use pdf2image with pytesseract instead.
```

### 🚫 反模式3：时间敏感信息

**❌ 错误**：

```
If you're doing this before August 2025, use the old API.
After August 2025, use the new API.
```

**✅ 正确**：

```
## Current method

Use the v2 API endpoint: `api.example.com/v2/messages`

## Old patterns

<details>
<summary>Legacy v1 API (deprecated 2025-08)</summary>

The v1 API used: `api.example.com/v1/messages`
This endpoint is no longer supported.
</details>
```

### 🚫 反模式4：不一致的术语

**❌ 错误**：

* 🔀 混用"API endpoint"、"URL"、"API route"、"path"
* 🔀 混用"field"、"box"、"element"、"control"

**✅ 正确**：

* 📝 始终使用"API endpoint"
* 📝 始终使用"field"

### 🚫 反模式5：假设工具已安装

**❌ 错误**：

```
Use the pdf library to process the file.
```

**✅ 正确**：

```
Install required package: `pip install pypdf`

Then use it:
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```

### ✅ 最佳实践检查清单

在分享Skill之前，验证：

**核心质量**：

* 📝 描述具体且包含关键词
* 📋 描述包含"做什么"和"何时使用"
* 📏 SKILL.md主体在500行以内
* 📁 额外细节在单独文件中（如需要）
* ⏰ 没有时间敏感信息（或在"旧模式"部分）
* 🔤 整个Skill使用一致的术语
* 💡 示例具体，不抽象
* 📂 文件引用保持一级深度
* 📚 适当使用渐进式披露
* 📋 工作流有清晰的步骤

**代码和脚本**：

🔧 脚本解决问题，不推给Claude

⚠️ 错误处理明确且有用

🔢 没有"魔法数字"（所有值都有理由）

📦 所需包在指令中列出并验证可用

📖 脚本有清晰的文档

🚫 没有Windows风格路径（全部使用前向斜杠）

✅ 关键操作有验证/验证步骤

🔁 质量关键任务包含反馈循环

**测试**：

📊 至少创建了3个评估

🤖 在Haiku、Sonnet、Opus上测试

🎯 使用真实使用场景测试

👥 收集团队反馈（如适用）

记住：**最好的Skill是那些解决真实问题、经过实际测试、持续迭代改进的Skill**。不要追求完美，先让它工作，然后不断改进。

---

**📚 延伸阅读**：

* 📖 Agent Skills官方文档 : https://github.com/anthropics/skills

*我是丢钢笔的人，下期见~*