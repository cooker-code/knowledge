---
title: Claude 新发布的 Agent Skills 到底是啥？居然比 MCP 还厉害？
author: 和平本记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU4MzkyNTE0OQ==&mid=2247484627&idx=1&sn=a2777bb4288c180f456e4652b5d374f9&chksm=fc02ff14b9085dd140dfbe009ff46a168ce667e62df5dbee3a684153e810061534b6cb372e6c&mpshare=1&scene=24&srcid=1018twYsNL3gQQjKWZBEEDzG&sharer_shareinfo=414f67d2e0a8dcf0729c16e0b0a9c237&sharer_shareinfo_first=414f67d2e0a8dcf0729c16e0b0a9c237#rd
---

## 1、Anthropic为什么推出 Agent Skills

**在没有 Agent Skills 之前：**

假如说你现在有一个实习生，你给这个实习生安排了一个具体的任务。比如说帮我做一份 PPT，你需要从头到尾把指令写得非常详细。

等到下一次，你再想让这个实习生帮你做一份 PPT，你又需要重新把这些指令和要求再说一遍，非常重复和低效。

**有了 Agent Skills 之后：**

Agent Skills 相当于给你的实习生准备了技能包和工具箱，想让实习生帮你执行具体任务的时候，不是去重复指令，而是把这个技能教给他，让他自己掌握。

**这样做的好处就是：**

**1）让你的 AI Agent 变成特定领域的专家**

你可以给你的 agent（Claude）安装各种技能包，比如：「PPT制作技能」「Excel数据分析技能」。这样一来你的 agent就会变成特定领域的专家。

当你再让他帮你做 PPT 时，它就会自动调用「PPT制作技能」。

**2）告别重复**

你的技能包只需要创建一次，claude 就可以在所有的对话中自动使用。你可以把平时经常需要重复性的工作都做成技能包，这样可以帮你极大的减少重复，提高效率。

**3）高效利用资源**

Claude 并不会把他所有学会的技能时时刻刻都记在脑子里，也就是我们说的加载到上下文里面。只有在需要时才会调用，用完就会放下。

当 Claude 启动时，他只会记下自己拥有哪些技能以及每个技能的简介（比如我会做 PPT、我会处理 PDF），这占用的上下文非常少。

当有任务触发时，比如说你这个时候下个指令说帮我去做个 PPT，他这时候才会去翻阅「PPT制作技能」工具箱里面的具体操作手册。

所以这种按需加载非常高效，Claude 可以掌握大量复杂的技能，也不会因为信息过载而影响当前任务。

## 2、Agent Skills 和 rules 以及 subagent 有什么区别

**1）Rules**

给 Agent 指定Rules，就像是给你的实习生一本员工手册。

它一般用来约束和规范行为，告诉 Agent 什么该做，什么不该做。

你给实习生一本员工手册，最多只能告诉他做 PPT 的时候要用公司的 LOGO，但是这本手册没办法教会他怎么去做 PPT。

**2）subagent**

这个意思是，当下达一个任务给 agent 的时候，这个 agent 会对任务做判断，比如说这是一个数据分析任务，那么它就会把这个数据分析任务专门指派给一个擅长做数据分析的 subagent。

Subagent模式是委托-执行关系。Agent本身没有学会新技能，而是学会了如何找对的人办事。

**3） Agent Skills**

给Agent添加Skills，就像是送你的实习生去参加一个专业的培训课程，并给他一套 SOP 和工具。

目的是让 Agent 自己掌握完成特定任务的一套标准方法。

**举例：**

你创建一个`create_quarterly_report`的Skill。这个Skill内部定义了清晰的步骤：

1️⃣ 从数据库API获取销售数据

2️⃣ 使用数据分析工具生成图表

3️⃣ 按照公司模板填充PPT

4️⃣ 生成总结摘要

当你说下一个指令，「制作季度报告」时，实习生（Agent）会激活他学过的这个技能，然后自己一步步按照手册上的流程去执行，并且这个技能是可以被不断复用的。

Skills是学习-执行的关系。Agent 亲自学习并掌握了完成任务的方法，而不是把它交给别人。

## 3、SKILL.md 文件介绍

```
---  
name: 技能名称  
description: 简单描述一下这个技能是干嘛的？以及什么时候使用它  
---  
  
# Your Skill Name  
  
## Instructions  
让 Claude 具体执行的指令  
  
## Examples  
[Concrete examples of using this Skill]
```

其中 `name` 和 `description` 是必填字段，`name` ：最多 64 个字符，`description` ：最多 1024 个字符

## 4、Skills 的三种内容类型和三个加载层级

**第一级：元数据（始终加载）**

包含技能名称和技能描述。

Claude 在启动时会加载这个元数据，因为只有名称和简短描述，所以就算有几百个技能，也不会给 Claude 的上下文造成负担。

元数据是 Claude用来判断我应该使用哪个技能的依据。因此，`description`写得好不好，直接决定了技能能否被准确触发。

**案例：**

```
---  
name: PDF 处理  
description: 从PDF文件中提取文本和表格，填写表格，合并文档。当处理PDF文件或用户提及PDF、表格或文档提取时使用。  
---
```

**第二级：指令（触发时加载）**

告诉 Claude 具体要做些什么，你可以认为是文字版的 SOP。

这里存放了完成任务所需的所有思考链、逻辑步骤和工作流程。

只有在第一级匹配成功后，这部分内容才会被加载到Claude的上下文窗口中去执行。

这极大地节省了成本和上下文空间，让Claude可以处理非常复杂的任务，而不必担心指令过长。

**存放位置：SKILL.md（你在 Skill 目录中编写的主说明文件）**

```
# PDF 处理  
  
## Quick start  
  
使用 pdfplumber 从 PDF 中提取文本:  
  
```python  
import pdfplumber  
  
with pdfplumber.open("document.pdf") as pdf:  
    text = pdf.pages[0].extract_text()  
```  
  
如涉及到表单填写，请参照[FORMS.md](FORMS.md).
```

**如何触发？**

每个 Skill 的 YAML 里有 **description**（例如：当用户提到 PDF、表单或文档抽取时使用）。

当你的请求语义与这个描述匹配（例如，帮我抽取这个 PDF 的文本），Claude 判定相关性高，于是触发该 Skill。

触发后，Claude 会执行类似命令：读取 SKILL.md，把其中流程与注意事项放入上下文。

**触发后会干什么？**

1）读取正文

把 SKILL.md 的内容读取到上下文中

2）执行任务

Claude 会按正文里的步骤开展任务，而不是即兴发挥。如果正文提到其他文件（如 FORMS.md），且任务确实需要，Claude再读取那些文件，否则不读，节省令牌。

**案例：**

1️⃣ 你说：请帮我总结一下这份 PDF 的内容

2️⃣ 匹配描述：这与 PDF Processing 这个 Skill 的 description 相符 → 触发。

3️⃣ Claude 读取：SKILL.md，看到先用 pdfplumber 从 PDF 中提取文本

4️⃣ Claude执行：按指南提取文本，若不涉及表单，就不会读取 FORMS.md，最后基于抽取结果进行总结。

**第三级：资源和代码（按需加载）**

第三级指的是，Skills 目录里除 SKILL.md 之外的扩展文件：更多说明文档、可执行脚本、和参考资料。它们不会自动进入上下文，只有在 SKILL.md 或任务需要时才被读取或执行。

```
pdf-skill/  
├── SKILL.md (main instructions)  
├── FORMS.md (form-filling guide)  
├── REFERENCE.md (detailed API reference)  
└── scripts/  
    └── fill_form.py (utility script)
```

**一般会有下面三种内容：**

**1）说明文档**

如 FORMS.md、REFERENCE.md。提供更细的流程与指南，灵活性高，适合复杂或专项场景（比如表单填充的步骤、API 的详细用法）。

当 SKILL.md 提到并且任务确实需要时，Claude 用 bash 读取这些文件，把文字内容加入上下文。

**2）代码**

如 scripts/fill\_form.py。Claude 在虚拟机里通过 bash 运行脚本，得到确定性的输出（例如，Validation passed或具体错误）。

脚本的源代码本身不进入上下文，只把运行结果带入，从而更省令牌、更可靠。

**3）资源**

如数据库 schema、API 文档、模板、示例。用于事实查阅或生成标准化产物。只有被引用时才读取相关片段；未用到的资源完全不占上下文。

**举例，什么时候会用到第三级？**

假如你要填写 PDF 表单并校验字段

1️⃣ 触发 PDF Skill → 读取 SKILL.md（Level 2）

2️⃣ SKILL.md 指出复杂表单需看 FORMS.md → 读取 FORMS.md（说明文档）。

3️⃣ FORMS.md 提到要运行 scripts/fill\_form.py → Claude 执行脚本。脚本输出进入上下文，脚本源码不进入。

4️⃣ 若需要查看字段规则或 API 细节 → 引用 REFERENCE.md 或相关 schema（资源）。