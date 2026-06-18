> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: Anthropic Skills深度解析：从MCP到智能工作流的进化之路
author: 刘一缘
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0MDE2ODM2Ng==&mid=2247487412&idx=1&sn=016c0eea68d9f66d6bd40fe787fe13dd&chksm=c382085660550d7aa589dff347eda0bfebd19f520f96fba513fb3f7cf1a2cd5725833576ebdc&mpshare=1&scene=24&srcid=1114P9JklspHnK3CYWEP6DSF&sharer_shareinfo=b530860d3edee9652853cfcb6cb57d82&sharer_shareinfo_first=b530860d3edee9652853cfcb6cb57d82#rd
---

大家好，我是刘一缘。

在AI技术飞速发展的这两年，Anthropic推出了两项革命性技术规范：MCP 和 Skills。

这两者都旨在增强AI助手的能力，但它们的设计理念和应用场景却截然不同。

今天将深入解析MCP与Skills的区别，并详细介绍Skills的应用方式，希望能帮助大家理解这一技术演进的重要意义。

# Model Context Protocol (MCP)

定位：开放标准协议
目标：标准化AI与外部工具的通信
架构：客户端-服务器模式
通信：JSON-RPC协议
应用场景：企业级集成、多厂商互操作、侧重于远程通讯

# Skills

定位：模块化能力包
目标：可重用工作流的封装
架构：文件夹+SKILL.md结构
通信：渐进式加载机制
应用场景：标准化任务、团队协作、侧重于本地能力

---

这里我们要区分的是，MCP解决的是"如何让不同AI系统与不同工具互联互通"的问题，而Skills解决的是"如何让AI记住并重复使用特定工作方式"的问题。两者互补而非替代关系。

---

# Skills技术架构解析

Skills是Anthropic在不久前推出的模块化能力封装系统。

它将特定的工作流程、知识和操作封装成可重用的包，让Claude能够以标准化方式执行复杂任务。

```
my-skill/
├── SKILL.md          # 必需的元数据和指令文件
├── resources/        # 可选的资源文件夹
│   ├── templates/    # 模板文件
│   ├── examples/     # 示例数据
│   └── scripts/      # 辅助脚本
└── README.md         # 使用说明
```

## SKILL.md结构示例

```
---
name: "financial-report-generator"
description: "Generate standardized financial reports with charts and analysis"
---

# Financial Report Generator

This skill creates comprehensive financial reports following company standards.

## Instructions

1. Analyze input financial data
2. Generate executive summary
3. Create visualizations using specified color scheme
4. Format according to brand guidelines

## Examples
- "Generate Q3 financial report from this Excel data"
- "Create monthly revenue analysis with projections"

## Guidelines
- Use company brand colors (#1f2937, #3b82f6)
- Include risk disclaimers
- Format dates as YYYY-MM-DD
```

---

Skills 的好处，其实只是将 Agents 动作更规范化的向用户开放定义。

实际上在没有这个概念出来以前，我们单凭 Prompt + 少量的工具调用或本地 MCP 服务端&客户端也能做到类似效果。

但总归 MCP 不是为了这个用途而生，今天的 Skills 补充了这个空白，我们可以给 AI 提供更多的「技能」。

日本电商巨头乐天使用Skills将财务工作流程效率提升8倍。原本需要一整天的工作，现在只需一小时完成。

Canva计划集成Skills来定制AI代理，扩展设计能力，帮助团队捕获独特上下文并轻松创建高质量设计。

Box使用Skills让Claude学会处理Box内容，用户可以将存储的文件转换为符合组织标准的PPT、Excel和Word文档。

开发者可以使用Skills创建代码生成、测试自动化、文档编写等标准化开发工作流。

应用场景从「处理多个电子表格、识别异常、生成标准化报告」，再到「品牌一致性检查、模板生成、批量设计处理」、再到「文档格式转换、内容提取、批量处理」，可谓是多面开花。

# 使用指北

## 网页端

如果你是 Pro 用户，在Claude.ai网页端使用Skills是最直接的方式。

### 启用Skills功能

* • 点击右上角头像，选择"Settings"
* • 导航到"Capabilities"选项卡
* • 确保"Code execution and file creation"已启用
* • 在"Skills"部分，可以启用示例Skills或上传自定义Skills

### 上传自定义技能

* • 在Skills设置页面，点击"Upload custom skill"
* • 选择包含Skills的ZIP文件
* • Claude会自动验证并安装Skills
* • 安装成功后，Skills会显示在可用列表中

### 使用Skills

* • 返回主聊天界面
* • 在提示中提及Skills的名称或描述
* • Claude会自动识别并加载相关Skills
* • 可以在Claude的思考链中看到Skills的调用过程

## Claude Code集成

### 添加Skills市场

在Claude Code中添加Skills市场

```
claude
> /plugin marketplace add anthropics/skills
```

### 安装Skills

浏览并安装Skills

```
> /plugins
# 选择 "Browse and install plugins"
# 选择 "anthropic-agent-skills"
# 选择需要的Skills集合（document-skills 或 example-skills）
# 点击 "Install now"
```

或者直接安装特定Skills

```
> /plugin install document-skills@anthropic-agent-skills
实际使用示例
# 使用PDF Skills提取表单字段
> Use the PDF skill to extract the form fields from path/to/some-file.pdf
```

```
# 使用文档Skills创建Word报告
> Can you make a Word document based on the architecture of this codebase?
```

### 项目级Skills配置

为了在项目中使用自定义Skills，可以手动安装到项目目录：
在项目根目录创建skills文件夹

```
mkdir -p skills/my-custom-skill
cp -r /path/to/your/skill/* skills/my-custom-skill/
重新加载Skills
> /reload-skills
```

---

Skills代表了AI工作流标准化的重要一步。

通过将复杂的工作流程封装成可重用的模块，Skills不仅提高了AI助手的效率和一致性，更重要的是建立了人与AI协作的新范式。

随着Skills生态系统的不断发展，我们可以预见一个更加智能、更加个性化的AI助手时代。

无论是个人用户还是企业团队，都能通过Skills构建适合自己的AI工作流，真正实现"一次创建，到处使用"的理想。

那么看到这里的你，不妨实际动起来，将你日常经常需要做的某一个动作，包装成一个Skills，看看它能不能与 AI 在协作碰撞下产生出新的火花吧！

对了，如果你想了解官网如何介绍Skills，你可以通过点击下方的原文链接直达。

---

以上，既然看到这里了，如果觉得不错，随手点个赞、在看、转发三连吧，如果想第一时间收到推送，也可以给我个星标⭐～谢谢你看我的文章，我们，下次再见。

同时欢迎日常喜欢探讨AI编程或其他技巧的同学加入交流群，二维码如果失效了请私信或留言给我~