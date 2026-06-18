> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 【Skills 进化论 05 系列】快速上手：安装、配置并使用你的第一个 Claude Skill
author: 前端AI行走
date:
url: https://mp.weixin.qq.com/s?__biz=MzU5MjYwMDgzNQ==&mid=2247487623&idx=1&sn=a53547855eddc27842b8c7ebd901500b&chksm=ff6e868978addbbd37373a795246427bd599ba93be31defb97e65b2983465e0a35b63d0ae3e5&mpshare=1&scene=24&srcid=0106gb7iziTrMwPL7IQqyx72&sharer_shareinfo=b1f81ee9084740bdd2eeb32945372e31&sharer_shareinfo_first=b1f81ee9084740bdd2eeb32945372e31#rd
---

关于Skills的实现，大家可以看下我之前写的

> AI “软实力”
>
> 前端AI行走，公众号：前端AI行走[借力打力：如何从0到1通过 Skills 生成专属你的 Skills，提升自身AI“软实力”](https://mp.weixin.qq.com/s/5xG_jTVM5oR77gJajVVlQQ)

> AI “硬实力”
>
> 前端AI行走，公众号：前端AI行走[开发 AI Skills 的“硬实力”：从 Prompt 走向工程化执行](https://mp.weixin.qq.com/s/FbvXudzKfkHqIQhSjqnC6w)

里面也具体写了相关一些Skills的基本功能与如何去开发一个专属的Skill。大家可以互相学习一下。

那么AI开发工程师们？准备好开启 AI 代理的“进化”了吗？本篇将指导你如何在本地环境安装并使用第一个 Skill，以及如何进行极速实验。

## 1. 环境准备

### 1.1 系统要求

| 组件 | 要求 | 说明 |
| --- | --- | --- |
| **Node.js** | ≥ 18.0.0 | 运行 OpenSkills 工具 |
| **npm** | ≥ 9.0.0 | 包管理器 |
| **Git** | 任意版本 | 版本控制（推荐） |
| **编辑器** | Claude Code / Cursor / Antigravity | 支持 Skills 的 AI 助手 |

### 1.2 安装 OpenSkills

目前，使用 Skills 最便捷方式是通过 **Claude Code** 或 **OpenSkills**。

```
# 检查 Node.js 版本
node --version  # 应该 >= 18.0.0

# 安装 OpenSkills
npm i -g openskills

# 验证安装
openskills --version
```

### 1.3 选择技能位置：个人 vs 项目

在开始之前，你需要决定技能的位置：

```

```

| 位置 | 路径 | 适用场景 | Git 管理 |
| --- | --- | --- | --- |
| **个人技能** | `~/.claude/skills/` | 个人实验、偏好设置 | ❌ 不提交 |
| **项目技能** | `.claude/skills/` | 团队共享、项目规范 | ✅ 提交到 Git |

**推荐**：初次使用建议从**项目技能**开始，便于团队协作和版本管理。

## 2. 安装与使用现有技能

### 2.1 安装官方技能库

从 Anthropic 官方仓库一键安装：

```
# 进入项目目录
cd /path/to/your/project

# 安装官方技能（推荐）
openskills install anthropics/skills --universal
```

**安装内容**：包括 PDF 处理、Excel 分析、代码操作等 20+ 个官方技能。

### 2.2 安装特定技能

```
# 从 GitHub 仓库安装
openskills install <github-username>/<repo-name>

# 示例：安装社区技能
openskills install someuser/vue-expert-skills
```

### 2.3 高级参数说明

| 参数 | 说明 | 使用场景 |
| --- | --- | --- |
| `--universal` | 跨工具同步 | 在 Cursor、Windsurf、Aider 等多个 AI 助手中使用 |
| `--force` | 强制重新安装 | 技能文件损坏或需要覆盖本地修改 |
| `--symlink` | 建立符号链接 | 本地开发技能，实时感知代码变动 |

### 2.4 验证安装

```
# 查看已安装的技能
openskills list

# 管理技能（启用/禁用）
openskills manage

# 同步技能到 AI 助手
openskills sync
```

**预期输出**：

```
✅ 已安装技能：
  - pdf (Anthropic)
  - excel (Anthropic)
  - book-analyzer (Local)
  - wechat-publisher (Local)
```

### 2.5 使用现有技能示例

**示例 1：使用 `book-analyzer` 拆解书籍**

```
# 如果项目中有 book-analyzer 技能
# 在 Claude 中直接说：
"帮我拆解这本技术书，生成小红书笔记"
```

**示例 2：使用 `wechat-publisher` 发布文章**

```
# 配置微信公众号凭证
cat > .claude/skills/wechat-publisher/.env << 'EOF'
WECHAT_APP_ID=your_app_id
WECHAT_APP_SECRET=your_app_secret
EOF

# 在 Claude 中直接说：
"帮我把 read-zj.md 发布到微信公众号"
```

## 3. 极速实验：3 分钟创建你的第一个技能

### 3.1 创建 "Hello World" 技能

如果你想亲手感触技能的力量，可以执行以下步骤：

**步骤 1：创建目录结构**

```
# 在项目根目录执行
mkdir -p .claude/skills/hello-skill
cd .claude/skills/hello-skill
```

**步骤 2：编写 SKILL.md**

```
cat > SKILL.md << 'EOF'
---
name: hello-skill
description: 当用户说"你好"时，以"超级特工"的口吻回复。用于测试技能是否正常工作。
---

# Hello Skill

## 快速开始
直接对 Claude 说："你好"，它会以超级特工的身份回复你。

## 指令
每当用户说"你好"、"hello"或"hi"时，请以"超级特工"的口吻回复，语气要酷炫、专业且略带神秘感。
EOF
```

**步骤 3：同步技能**

```
# 返回项目根目录
cd ../../..

# 同步技能到 AI 助手
openskills sync
```

**步骤 4：测试技能**在 Claude 对话框中输入："你好"，你应该看到类似这样的回复：

> "你好，我是你的专属超级特工。有什么任务需要我协助吗？"

### 3.2 创建更实用的技能：代码审查助手

让我们创建一个更实用的技能示例：

```
mkdir -p .claude/skills/code-review-helper
cd .claude/skills/code-review-helper

cat > SKILL.md << 'EOF'
---
name: code-review-helper
description: 代码审查助手。当用户要求审查代码、检查代码质量或寻找 Bug 时激活此技能。自动检查代码规范、性能问题和安全隐患。
---

# 代码审查助手

## 使用场景
- 审查 Pull Request
- 检查代码质量
- 寻找潜在 Bug
- 性能优化建议

## 审查清单
1. **代码规范**：检查命名、格式、注释
2. **性能问题**：识别慢查询、内存泄漏
3. **安全隐患**：SQL 注入、XSS 漏洞
4. **最佳实践**：是否符合团队规范

## 输出格式
- 问题严重程度（高/中/低）
- 具体位置（文件名 + 行号）
- 问题描述
- 修复建议
EOF

cd ../../..
openskills sync
```

**测试**：在 Claude 中粘贴一段代码，说："帮我审查这段代码"，技能会自动激活。

### 3.3 技能目录结构详解

一个完整的技能可能包含以下结构：

```
my-skill/
├── SKILL.md              # 必需：核心指令文件
├── README.md             # 可选：使用说明
├── CHANGELOG.md          # 可选：版本变更记录
├── scripts/              # 可选：可执行脚本
│   └── helper.py
├── references/           # 可选：参考资料
│   └── api-docs.md
├── assets/               # 可选：静态资源
│   └── template.html
└── .env.example          # 可选：配置示例
```

**最小结构**：只需要 `SKILL.md` 即可！

## 4. 验证技能是否生效

### 4.1 检查技能列表

```
# 查看所有已安装的技能
openskills list

# 查看特定技能的详细信息
openskills show hello-skill
```

### 4.2 测试技能激活

**方法 1：直接测试**在 Claude 对话框中输入技能描述中的关键词，观察是否激活。

**方法 2：查看技能描述**

```
# 查看技能的 description 字段
cat .claude/skills/hello-skill/SKILL.md | head -5
```

确保 `description` 字段包含你期望的触发词。

### 4.3 调试技巧

```

```

## 5. 常见报错处理

### 5.1 `sync` 失败

**错误信息**：

```
Error: Failed to sync skills
Permission denied: .claude/skills
```

**解决方案**：

```
# 1. 确保项目已初始化 Git
git init

# 2. 检查目录权限
ls -la .claude/skills

# 3. 手动创建目录
mkdir -p .claude/skills
chmod 755 .claude/skills

# 4. 重新同步
openskills sync
```

### 5.2 技能不生效

**可能原因及解决方案**：

| 问题 | 检查项 | 解决方案 |
| --- | --- | --- |
| **技能未激活** | `description` 字段是否包含触发词？ | 更新 description，添加更多关键词 |
| **YAML 语法错误** | frontmatter 格式是否正确？ | 检查 `---` 是否成对出现 |
| **文件位置错误** | 是否在 `.claude/skills/` 下？ | 确保目录结构正确 |
| **未同步** | 是否运行了 `openskills sync`？ | 运行同步命令 |
| **AI 助手未重启** | 是否重启了 Claude Code？ | 重启 AI 助手 |

### 5.3 YAML 语法错误

**常见错误**：

```
# ❌ 错误：缺少结束标记
---
name:my-skill
description:Myskill

# ✅ 正确：完整的 frontmatter
---
name:my-skill
description:Myskill
---

# 正文内容从这里开始
```

**验证 YAML**：

```
# 使用 Python 验证（如果已安装）
python3 -c "import yaml; yaml.safe_load(open('.claude/skills/my-skill/SKILL.md'))"
```

### 5.4 技能冲突

**问题**：多个技能有相似的 `description`，导致激活混乱。

**解决方案**：

* 1.使 `description` 更具体、更独特
* 2.使用 `openskills manage` 禁用不需要的技能
* 3.检查技能优先级（项目技能优先于个人技能）

## 6. 下一步：进阶学习

### 6.1 推荐学习路径

```

```

### 6.2 实践建议

1. **从简单开始**：先创建只有 `SKILL.md` 的技能
2. **逐步扩展**：需要时再添加 `scripts/`、`references/`
3. **使用真实场景**：基于你的实际工作流程创建技能
4. **迭代优化**：根据使用反馈不断改进

### 6.3 使用 `skill-writer` 辅助创建

如果你觉得手动编写技能有难度，可以使用项目中的 `skill-writer` 技能：

```
# 在 Claude 中直接说：
"使用 skill-writer 技能，帮我创建一个代码审查助手技能"
```

`skill-writer` 会自动生成符合规范的 `SKILL.md` 文件。

相关流程如下：

code-review-helper 代码附录如下（依次是SKILL.md，reference.md）：

```
---name: code-review-helperdescription: 代码审查助手。当用户要求审查代码、检查代码质量或寻找 Bug 时激活此技能。自动根据团队规范、性能要求和安全标准提供深度反馈。---# 代码审查助手 (Code Review Helper)这个技能旨在通过模拟资深架构师的视角，为开发者提供高质量、专业且细致的代码审查反馈。## 快速开始1. **激活**：直接粘贴代码并输入“帮我审查这段代码”或“检查代码质量”。2. **反馈**：AI 将基于性能、安全性、可读性和架构设计四个维度输出结构化报告。## 核心指令 (Instructions)当你受命进行代码审查时，请严格遵循以下流程：1. **深度扫描**：    - 检查代码是否符合现代开发标准（如 TypeScript 的类型完备性）。    - 识别潜在的逻辑错误、竞态条件或边界处理缺失。    - 搜索性能瓶颈（如昂贵的循环、多余的列表重绘）。2. **多维度评价**：    - **安全性**：检查 SQL 注入、XSS 风险、硬编码秘钥等。    - **性能**：关注算法复杂度、缓存使用、资源释放。    - **可读性**：评估命名描述性、函数长度和模块解耦。    - **一致性**：确认是否遵循团队既定的样板文件和架构模式。3. **建设性反馈**：    - 不要只指出错误，必须提供**具体的修复代码片段**。    - 解释背后的原因（Why），而不仅仅是做了什么（What）。    - 鼓励遵循 SOLID 原则和 DRY原则。## 输出格式规范每次审查必须包含：- **Summary**：一句话总结整体代码质量。- **High Priority**：必须立即修复的严重问题（红色标记）。- **Medium Priority**：建议优化的项目（黄色标记）。- **Code Refactor**：提供重构前后的对比代码块。- **Security Check**：明确声明是否存在安全隐患。## 参考资料详尽的核对清单请参阅 [reference.md](reference.md)。---*Powered by Claude Digital Expert Series*
```

```
# 代码审查核对清单 (Code Review Checklist)本文件提供了在执行代码审查时应关注的详细技术细节，作为资深专家的思维导图。## 1. 基础质量 (Basic Quality)- [ ] **命名**：变量名是否具有描述性？是否避免了缩写冲突？- [ ] **注释**：是否解释了“为什么这样做”而不是“正在做什么”？- [ ] **异常处理**：是否有适当的 `try-catch` 或错误边界？- [ ] **复杂性**：单个函数是否超过 50 行？逻辑嵌套是否过深？## 2. 性能优化 (Performance)- [ ] **数据库**：是否有 N+1 查询问题？查询是否命中了索引？- [ ] **前端**：在 React/Vue 中是否有不必要的重渲染 (Re-renders)？- [ ] **计算**：是否存在在高频执行函数内的 O(n^2) 算法？- [ ] **内存**：计时器或订阅是否在组件卸载时被正确清除？## 3. 安全防御 (Security)- [ ] **输入**：用户输入是否经过了严格的验证和清洗？- [ ] **秘钥**：是否存在硬编码在代码中的 API Key 或环境变量？- [ ] **授权**：敏感操作是否在后端验证了用户权限？- [ ] **协议**：是否使用了非加密的传输方式？## 4. 架构设计 (Architecture)- [ ] **解耦**：组件是否过于臃肿？逻辑是否可以抽象为通用的 Hook 或工具类？- [ ] **SOLID**：类或函数是否只负责一件事情（SRP）？- [ ] **可测试性**：逻辑是否易于编写单元测试（Unit Tests）？## 5. 最佳实践 (Best Practices)- [ ] **DRY**：是否有重复的代码片段可以合并？- [ ] **Modern syntax**：是否使用了最优雅的 ES6+ 或特定框架语法？- [ ] **Type safety**：如果是 TS 项目，`any` 的使用是否合理且最小化？---*此清单将随着团队技术的演进而持续更新。*
```

AI 已经帮你完成了较为完善的代码检查SKills 功能，那么作为开发人员，你需要好好审视AI帮你写的Skill是否满足你的要求，然后你可以进行多次多轮对话让AI帮你完善。

我刚刚只是简单提示词，没有写任何的要求。也可以自己去看AI 给出的结论是什么，你在人为的加加减减，或者你可以直接导入团队技术中涉及的代码规范文档，让AI帮你根据团队文档，进一步的完善Skills。

那么随着迭代层级的一步步完善，一套完善可复用的skill就这样的诞生了。而且可以持续迭代，可维护，从而提升了团队的整体代码质量。

## 7. 总结：开启你的“数字化专家”进化之旅

恭喜你！到这里，你已经完成了从环境搭建到亲手创建第一个 Skill 的全过程。这不仅是技术上的一个步，更是你作为“AI 数字开发管家”在思维模式上的重要转型。

### 7.1 快速回顾：你的技能升级路线

* **工具链就绪**：通过安装 `openskills` CLI，你已经拥有了一套标准化的技能管理引擎。
* **即插即用**：借助于官方和社区技能库，你实现了 Claude 能力的瞬间增幅。
* **资产化起步**：通过 "Hello World" 和 "代码审查助手" 的实验，你学会了如何将零散的 Prompt 沉淀为可复用的**数字资产**。

### 7.2 实战心法：从“会用”到“精通”

* **坚持“原子化”**：不要试图在第一天就写出一个全能技能。从解决一个具体的小痛点（如：自动写 Git Commit）开始。
* **善用“脚手架”**：当你感到无从下手时，记得回来看 `skill-writer` 或本项目提供的样板，它们是你最好的设计导师。
* **关注“反馈环”**：Skill 是活的代码。在与 Claude 的交互中观察其激活表现，根据 AI 的反馈不断打磨你的 `description` 和 `Instructions`。

**最终建议**：不要把 Skills 仅仅看作一个配置文件，它是你个人和团队**知识智慧的容器**。每一个被精心编写的 Skill，都是在为你的未来“自动化人生”节省宝贵的时间。

---

*下一篇：*【Skills 进化论 06 系列】*Skills 常见问题解答与进阶 FAQ。*