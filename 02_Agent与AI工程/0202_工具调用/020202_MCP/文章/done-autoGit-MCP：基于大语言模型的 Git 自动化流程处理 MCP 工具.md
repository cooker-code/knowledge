> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: autoGit-MCP：基于大语言模型的 Git 自动化流程处理 MCP 工具
author: 自由意志与人工智能
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg2ODE2MzI1Mg==&mid=2247484793&idx=1&sn=3ea14c385963ed3ef15b67f066aadce6&chksm=cfaad43926c69d9c59e49e10eb06b08d586ebc8c0953df27ec40593cd587d358386f3640ff9f&mpshare=1&scene=24&srcid=1108PNul7AyxPPliCHnvl3Ul&sharer_shareinfo=9f85de01821ba108b0f0c33a6bcde18d&sharer_shareinfo_first=9f85de01821ba108b0f0c33a6bcde18d#rd
---

autoGit-MCP：基于大语言模型的 Git 自动化流程处理 MCP 工具

日期：2025-11-05　标签：工具介绍 / Git / MCP / 自动化 / 开源工具 / GitHub / Gitee / GitLab

autoGit-MCP 是一款基于 Model Context Protocol (MCP) 的 Git 自动化流程处理工具。它通过大语言模型完成 Git 操作的自动化处理，支持 GitHub、Gitee、GitLab 三大平台，提供仓库查询、提交分析、工作日志生成、Git 组合命令等丰富功能，让开发者能够通过自然语言与 Git 仓库进行交互，大大提升开发效率。

---

核心特性

多平台支持

autoGit-MCP 支持三大主流代码托管平台：

* **GitHub‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  全球最大的代码托管平台，支持完整的 API 功能
* **Gitee（码云）‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  中国领先的代码托管平台，支持国内开发者快速访问
* **GitLab‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  企业级代码托管平台，支持私有部署和完整的 CI/CD 功能


所有工具通过统一的provider参数（github/gitee/gitlab）选择平台，保持接口一致性。

git\_catalog 工具

git\_catalog是 autoGit-MCP 的核心工具之一，提供 7 个强大的子命令用于查询仓库和提交活动：
1. search\_repos - 关键词检索仓库
根据关键词、语言、Star 数等条件搜索仓库：
**功能特性**：

* ✅ 支持关键词匹配（name/description/readme）
* ✅ 支持语言限定（如 "Python", "C++", "TypeScript"）
* ✅ 支持最小 Star 数过滤
* ✅ 支持最近活跃时间过滤
* ✅ 支持 topic 和 owner 限定
* ✅ 支持多种排序方式（updated/stars/forks）


**示例查询**：

1 # 搜索 Python 相关的高 Star 仓库2 {3     "keyword": "machine learning",4     "language": "Python",5     "min\_stars": 1000,6     "sort": "stars",7     "order": "desc",8     "limit": 509 }


2. org\_repos - 组织仓库列表
列出指定组织的所有仓库：
**功能特性**：

* ✅ 支持仓库类型过滤（all/public/private/forks/sources/member）
* ✅ 支持是否包含 archived 仓库
* ✅ 支持多种排序方式
* ✅ 最多返回 5000 条记录


3. cross\_repos - 跨仓库提交查询
查询指定作者在多个仓库中的提交记录：
**功能特性**：

* ✅ 支持按作者登录名或邮箱查询
* ✅ 支持时间范围过滤（since/until）
* ✅ 支持每仓库最多抓取条数限制
* ✅ 自动去重和聚合


**应用场景**：

* 分析开发者在多个项目中的贡献
* 追踪跨项目的代码提交模式
* 生成多项目工作日志


4. repo\_authors - 仓库作者分析
分析同一仓库中不同作者的提交活动：
**功能特性**：

* ✅ 支持多作者同时查询
* ✅ 支持按登录名或邮箱查询
* ✅ 支持时间范围过滤
* ✅ 支持每作者最多抓取条数限制


**应用场景**：

* 分析团队协作情况
* 统计项目贡献者
* 生成团队工作报告


5. repos*by*author - 作者仓库列表
查询指定作者参与的所有仓库：
**功能特性**：

* ✅ 支持最小提交数阈值过滤
* ✅ 支持时间范围过滤
* ✅ 支持仓库类型过滤
* ✅ 自动统计每个仓库的提交数


6. authors*by*repo - 仓库活跃作者
查询指定仓库的所有活跃作者：
**功能特性**：

* ✅ 支持最小提交数阈值过滤
* ✅ 支持时间范围过滤
* ✅ 支持作者主键偏好（login/email/name）
* ✅ 自动统计每个作者的提交数


7. user\_repos - 用户仓库列表
查询用户拥有或 Star 的项目列表：
**功能特性**：

* ✅ 支持 owned/starred/both 三种模式
* ✅ 支持私有仓库查询（需要 token 权限）
* ✅ 支持 archived 和 forks 过滤
* ✅ 支持多种排序方式
* ✅ 最多返回 5000 条记录

git\_work 工具

git\_work工具支持从本地或远程仓库收集提交记录并生成工作日志：
**功能特性**：

* ✅**多数据源支持**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  支持本地 Git 仓库、GitHub、Gitee、GitLab
* ✅**工作会话分析**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  自动计算工作会话，检测并行工作时间
* ✅**AI 总结生成**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  集成 DeepSeek 和 OpenAI，生成中文工作总结
* ✅**多项目支持**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  支持同时分析多个本地或远程仓库
* ✅**时间范围过滤**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  支持指定时间范围进行提交分析
* ✅**智能去重**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  自动去重相同提交，避免重复统计


**工作日志格式**：

 1 ## 工作日志 - 2025-11-05 2  3 ### 项目 1: example-project 4 - \*\*会话 1\*\*: 09:00-12:00 (3小时) 5   - 提交 1: 修复代码块空格问题 6   - 提交 2: 优化滚动兼容性 7 - \*\*会话 2\*\*: 14:00-17:30 (3.5小时) 8   - 提交 3: 添加新功能模块 9 10 ### AI 总结11 今天主要完成了代码格式优化和新功能开发，共工作 6.5 小时...

git\_comb 工具

git\_comb工具提供 Git 组合命令功能，支持复杂的 Git 操作流程：
**功能特性**：

* ✅**危险命令防护**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  默认禁止执行危险命令（如reset --hard、push --force）
* ✅**Dry Run 模式**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  支持预览执行计划，不实际执行
* ✅**超时控制**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  可配置命令执行超时时间
* ✅**错误处理**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  完善的错误处理和提示机制


**支持的组合命令**：

* commit‌‍‌‍‌‍：‌‍

  提交更改
* merge‌‍‌‍‌‍：‌‍

  合并分支
* reset‌‍‌‍‌‍：‌‍

  重置操作
* revert‌‍‌‍‌‍：‌‍

  撤销提交
* clean‌‍‌‍‌‍：‌‍

  清理未跟踪文件
* stash‌‍‌‍‌‍：‌‍

  暂存更改
* 其他标准 Git 命令

health 工具

health工具提供健康检查功能，可以检查配置状态和工具可用性：
**功能特性**：

* ✅**配置检查**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  检查所有必需和可选的环境变量
* ✅**密钥掩码显示**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  安全地显示配置状态（密钥以掩码形式显示）
* ✅**工具可用性检查**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  检查各个工具的可用性
* ✅**详细状态报告**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  提供详细的配置和工具状态报告

reload\_config 工具

reload\_config工具支持配置热重载，无需重启服务器即可重新加载配置：
**功能特性**：

* ✅**热重载**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  支持在不重启服务器的情况下重新加载环境变量
* ✅**配置验证**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  自动验证新配置的有效性
* ✅**错误提示**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  配置错误时提供清晰的错误提示

---

安全特性

危险命令防护

默认情况下，以下危险命令需要显式设置allow\_destructive: true才能执行：

* reset --hard

  - 硬重置，会丢失未提交的更改
* clean -fd

  - 强制清理未跟踪文件
* push --force

  - 强制推送，可能覆盖远程分支
* stash drop

  /stash clear- 删除 stash，可能丢失暂存更改
* 其他可能导致数据丢失的操作

Dry Run 模式

对于以下命令支持dry\_run: true预览执行计划：

* commit‌‍‌‍‌‍：‌‍

  预览提交内容
* merge‌‍‌‍‌‍：‌‍

  预览合并计划
* reset‌‍‌‍‌‍：‌‍

  预览重置操作
* revert‌‍‌‍‌‍：‌‍

  预览撤销操作
* clean‌‍‌‍‌‍：‌‍

  预览清理计划

错误处理

所有工具都包含完善的错误处理机制：

* **参数验证错误‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  提供清晰的错误消息，指出哪个参数无效
* **命令执行错误‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  返回 Git 命令的 stdout 和 stderr
* **超时错误‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  可配置超时时间，超时时返回明确提示
* **网络错误‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  区分 HTTP 错误和连接错误
* **API 密钥错误‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  提示需要设置的环境变量

---

安装与配置

快速安装

1 # 1. 克隆仓库2 git clone https://github.com/Mapoet/autoGit-MCP.git3 cd autoGit-MCP4 5 # 2. 安装 Python 依赖6 pip install -r requirements.txt7 8 # 3. 配置环境变量（见下方说明）

环境变量配置

autoGit-MCP 使用集中配置管理，支持从环境变量或 MCP 配置文件的env字段读取配置。
GitHub 配置

1 # GitHub Personal Access Token2 export GITHUB\_TOKEN=ghp\_xxxxxxxxxxxxxxxxxxxx


Gitee 配置

1 # Gitee Personal Access Token2 export GITEE\_TOKEN=gitee\_xxxxxxxxxxxxxxxxxxxx


GitLab 配置

1 # GitLab Personal Access Token2 export GITLAB\_TOKEN=glpat-xxxxxxxxxxxxxxxxxxxx3 # GitLab 实例地址（可选，默认为 gitlab.com）4 export GITLAB\_URL=https://gitlab.com


AI 总结配置（可选）

1 # DeepSeek API Key（用于生成工作总结）2 export DEEPSEEK\_API\_KEY=sk-xxxxxxxxxxxxxxxxxxxx3 # 或使用 OpenAI API Key4 export OPENAI\_API\_KEY=sk-xxxxxxxxxxxxxxxxxxxx


其他配置

1 # Git 命令超时时间（秒，默认 30）2 export GIT\_TIMEOUT=30

MCP 配置文件示例

在 MCP 配置文件中添加 autoGit-MCP 服务器配置：

 1 { 2   "mcpServers": { 3     "autogit-mcp": { 4       "command": "python", 5       "args": ["http://localhost:9010/mcp/"], 6       "env": { 7         "GITHUB\_TOKEN": "ghp\_xxxxxxxxxxxxxxxxxxxx", 8         "GITEE\_TOKEN": "gitee\_xxxxxxxxxxxxxxxxxxxx", 9         "GITLAB\_TOKEN": "glpat-xxxxxxxxxxxxxxxxxxxx",10         "DEEPSEEK\_API\_KEY": "sk-xxxxxxxxxxxxxxxxxxxx"11       }12     }13   }14 }

---

项目架构

autoGit-MCP 采用模块化设计，代码结构清晰：

 1 autoGit-MCP/ 2 ├── README.md                    # 项目说明 3 ├── requirements.txt             # Python 依赖 4 ├── LICENSE                      # MIT 许可证 5 ├── docs/                        # 文档目录 6 │   ├── code-structure.md       # 代码结构详细说明 7 │   ├── environment-variables.md # 环境变量详细说明 8 │   ├── git-cheatsheet.md       # Git 命令速查表 9 │   ├── git\_comb.md             # Git 组合命令说明10 │   ├── mcp-git-tool.md         # MCP Git 工具设计文档11 │   └── troubleshooting.md      # 故障排查指南12 ├── src/                         # 源代码目录13 │   └── git\_tool/               # Git 工具模块14 │       ├── \_\_init\_\_.py15 │       ├── config.py           # 集中配置管理（Pydantic Settings）16 │       ├── mcp\_server.py       # MCP 服务器入口17 │       ├── tools/               # 工具实现18 │       │   ├── git\_catalog.py   # 仓库查询工具19 │       │   ├── git\_work.py     # 工作日志工具20 │       │   ├── git\_comb.py     # Git 组合命令工具21 │       │   ├── health.py       # 健康检查工具22 │       │   └── reload\_config.py # 配置重载工具23 │       └── providers/          # 平台提供者24 │           ├── github.py       # GitHub API 实现25 │           ├── gitee.py        # Gitee API 实现26 │           └── gitlab.py       # GitLab API 实现27 └── tests/                       # 测试目录

核心组件

1. **config.py‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

   集中配置管理模块，使用 Pydantic Settings 管理环境变量
2. **git\_catalog‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

   仓库和提交查询工具，支持 7 个子命令，覆盖 GitHub/Gitee/GitLab
3. **git\_work‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

   工作日志生成工具，支持多数据源和 AI 总结
4. **git\_comb‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

   Git 组合命令工具，支持危险命令防护和 Dry Run
5. **health‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

   健康检查工具，检查配置状态和工具可用性
6. **reload\_config‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

   配置热重载工具，支持动态更新配置
7. **providers‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

   平台提供者模块，实现统一的 API 接口

---

使用场景

1. 代码仓库搜索与分析

**场景**：寻找特定技术栈的开源项目

 1 # 使用 git\_catalog 搜索 Python 机器学习项目 2 { 3     "tool": "git\_catalog", 4     "subcommand": "search\_repos", 5     "keyword": "machine learning", 6     "language": "Python", 7     "min\_stars": 100, 8     "sort": "stars", 9     "order": "desc",10     "limit": 2011 }

2. 跨项目贡献分析

**场景**：分析开发者在多个项目中的贡献情况

1 # 使用 git\_catalog 查询跨仓库提交2 {3     "tool": "git\_catalog",4     "subcommand": "cross\_repos",5     "author\_email": "developer@example.com",6     "owner": "my-org",7     "since": "2025-01-01",8     "until": "2025-11-05"9 }

3. 工作日志生成

**场景**：自动生成每日/每周工作总结

 1 # 使用 git\_work 生成工作日志 2 { 3     "tool": "git\_work", 4     "sources": [ 5         {"type": "local", "path": "/path/to/project1"}, 6         {"type": "github", "repo": "owner/repo2"} 7     ], 8     "since": "2025-11-01", 9     "until": "2025-11-05",10     "generate\_summary": true11 }

4. 团队协作分析

**场景**：分析团队在项目中的协作情况

1 # 使用 git\_catalog 查询仓库作者2 {3     "tool": "git\_catalog",4     "subcommand": "authors\_by\_repo",5     "repo\_full": "my-org/my-project",6     "min\_commits": 10,7     "since": "2025-01-01"8 }

5. Git 操作自动化

**场景**：通过大语言模型执行复杂的 Git 操作

 1 # 使用 git\_comb 执行 Git 操作 2 { 3     "tool": "git\_comb", 4     "command": "commit", 5     "args": { 6         "message": "修复代码块空格问题", 7         "allow\_empty": false 8     }, 9     "dry\_run": false10 }

---

技术亮点

1. 统一的平台接口

autoGit-MCP 通过统一的provider参数支持 GitHub/Gitee/GitLab 三大平台，所有工具都使用相同的接口设计，只需切换provider参数即可在不同平台间切换。

2. 集中配置管理

使用 Pydantic Settings 实现集中配置管理，支持：

* 从环境变量读取配置
* 从 MCP 配置文件的env字段读取配置
* 配置验证和类型检查
* 配置热重载

3. 完善的错误处理

所有工具都包含完善的错误处理机制：

* 参数验证错误‌‍‌‍‌‍：‌‍

  清晰的错误消息
* 命令执行错误‌‍‌‍‌‍：‌‍

  返回详细的 stdout 和 stderr
* 网络错误‌‍‌‍‌‍：‌‍

  区分 HTTP 错误和连接错误
* API 速率限制‌‍‌‍‌‍：‌‍

  自动处理速率限制

4. 安全防护机制

* **危险命令防护‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  默认禁止执行危险命令
* **Dry Run 模式‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  支持预览执行计划
* **密钥掩码显示‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  健康检查时安全显示配置状态
* **超时控制‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  防止命令执行超时

5. 智能工作日志

* **工作会话分析‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  自动计算工作会话，检测并行工作时间
* **AI 总结生成‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  集成 DeepSeek 和 OpenAI，生成中文工作总结
* **多项目支持‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  支持同时分析多个项目
* **智能去重‌‍**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  自动去重相同提交

---

版本更新

v1.5（最新版本）

* ✅**集中配置管理**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  新增config.py模块，使用 Pydantic Settings 集中管理环境变量
* ✅**健康检查工具**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  新增health工具，检查配置状态和工具可用性
* ✅**配置热重载**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  新增reload\_config工具，支持动态更新配置

v1.4

* ✅**多平台支持**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  git*catalog**和git*work工具支持 Gitee 和 GitLab
* ✅**统一接口设计**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  通过provider参数选择平台
* ✅**完整的 API 实现**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  实现了所有 7 个子命令的 Gitee 和 GitLab 版本

v1.3

* ✅**新增 git\_catalog 工具**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  支持 7 个子命令查询 GitHub 仓库和提交活动
* ✅**跨仓库提交查询**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  支持查询指定作者在多个仓库中的提交记录
* ✅**仓库作者统计**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  支持查询仓库中多个作者的提交活动
* ✅**速率限制优化**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  优化 API 调用频率

v1.2

* ✅**新增 git\_work 工具**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  支持从本地/GitHub/Gitee/GitLab 收集提交并生成工作日志
* ✅**工作会话分析**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  自动计算工作会话，检测并行工作时间
* ✅**AI 总结生成**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  集成 DeepSeek 和 OpenAI

v1.1

* ✅**代码重构**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  分离接口定义与实现逻辑
* ✅**Pydantic V2 迁移**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  所有验证器已迁移到 Pydantic V2
* ✅**完善的错误处理**‌‍‌‍‌‍‌‍‌‍‌‍：‌‍‌‍

  为所有工具添加了分类异常处理

---

使用示例

示例 1：搜索高 Star 的 Python 项目

1 { 2     "tool": "git\_catalog", 3     "subcommand": "search\_repos", 4     "keyword": "data science", 5     "language": "Python", 6     "min\_stars": 1000, 7     "sort": "stars", 8     "order": "desc", 9     "limit": 10,10     "provider": "github"11 }

示例 2：生成工作日志

1 { 2     "tool": "git\_work", 3     "sources": [ 4         {"type": "local", "path": "/path/to/project"}, 5         {"type": "github", "repo": "owner/repo"} 6     ], 7     "since": "2025-11-01", 8     "until": "2025-11-05", 9     "generate\_summary": true,10     "ai\_provider": "deepseek"11 }

示例 3：执行 Git 操作（Dry Run）

1 {2     "tool": "git\_comb",3     "command": "merge",4     "args": {5         "branch": "feature-branch"6     },7     "dry\_run": true8 }

示例 4：检查配置状态

1 {2     "tool": "health"3 }

---

项目优势

1. 功能强大

* 支持三大主流代码托管平台（GitHub/Gitee/GitLab）
* 提供 7 个强大的仓库查询子命令
* 支持工作日志生成和 AI 总结
* 支持 Git 组合命令和自动化操作

2. 易于使用

* 统一的 MCP 接口，易于集成
* 清晰的文档和示例
* 完善的错误处理和提示
* 支持 Dry Run 模式，安全可靠

3. 安全可靠

* 危险命令防护机制
* 密钥掩码显示
* 超时控制
* 完善的错误处理

4. 可扩展性强

* 模块化设计
* 统一的平台接口
* 易于添加新功能
* 支持配置热重载

---

总结

autoGit-MCP 是一款专为大语言模型设计的 Git 自动化工具，它通过 Model Context Protocol (MCP) 提供强大的 Git 操作能力。无论是仓库搜索、提交分析、工作日志生成，还是 Git 操作自动化，autoGit-MCP 都能帮助开发者更高效地管理代码仓库。

无论您是个人开发者、团队管理者，还是 DevOps 工程师，autoGit-MCP 都能帮助您通过大语言模型更智能、更高效地管理 Git 仓库，让代码管理变得更加简单。

---


**作者**：Mapoet
**项目地址**：https://github.com/Mapoet/autoGit-MCP
**许可证**：MIT License
**贡献**：欢迎提交 Issue 和 Pull Request

来源：gnss.ac.cn《autoGit-MCP：基于大语言模型的 Git 自动化流程处理 MCP 工具》