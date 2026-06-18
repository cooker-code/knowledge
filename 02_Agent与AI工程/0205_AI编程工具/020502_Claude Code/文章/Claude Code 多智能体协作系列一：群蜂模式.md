---
title: Claude Code 多智能体协作系列一：群蜂模式
author: Asher同学
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk3NTcyMzA0Nw==&mid=2247484032&idx=1&sn=0c8e0c29e1f52f506b6047829bc73800&chksm=c5d07d1e22491f793d7bd5d6ab374116839d5a1f52e4ce97d23902b4bc937e0de1977f854fe7&mpshare=1&scene=24&srcid=1014KzA1AvYP6Uu1gcDVgyHT&sharer_shareinfo=d2f4ab23c261bd776e1a70f0c5c3bf4b&sharer_shareinfo_first=d2f4ab23c261bd776e1a70f0c5c3bf4b#rd
---

## Claude Code 多智能体协作系列一：群蜂模式

前面几篇文章基本把智能体说清楚了，

1. 1. 《[通过理解 Agent 架构，看透 Claude Code 命令行选项](https://mp.weixin.qq.com/s?__biz=Mzk3NTcyMzA0Nw==&mid=2247483843&idx=1&sn=bcef3b4cf81ac8fbf8268bc0f3704ce4&scene=21#wechat_redirect)》介绍了智能体 Agent 的结构
2. 2. 《[Claude Code 核心组件 Agent 配置，让你的 cc 拥有三头六臂（纯肝货）](https://mp.weixin.qq.com/s?__biz=Mzk3NTcyMzA0Nw==&mid=2247484014&idx=1&sn=53a0125935c5446392bbddeace7324a1&scene=21#wechat_redirect)》详细讲了Claude Code 如何定义和使用智能体

顺着这个思路，自然就引出了今天要讨论的多智能体协作的话题。

使用过 Claude Code（CC）的都知道，CC 本身是一个相当聪明的智能体，但是由于大模型上下文长度的限制，所有的事情都交给一个 Agent 去做是不现实的（过多的琐事会影响高层的决策）。

把你自己想象成一个老板，如果你成立一家公司，哪些东西是核心的业务，哪些东西要外包出去？不是所有的事情都同等重要，所以，AI 时代一个很重要的思维就是**所有 AI 能做的事情都交给 AI**，然后你就有更多的时间去做一些更有意思和创造性的东西，你自己雇佣一堆免费的 AI 给你干活，这么好的资源为什么不用?

另外实在想吐槽的是，自从 DeepSeek 出来之后，很多吸血鬼在某鱼某宝卖各种 DeepSeek 付费资料的，还有某某视频号打着 **AI 一人公司**卖3天课程的。

我只想问，这些吸血鬼到底可以给社会带来什么价值？  
人家 DeepSeek 本来就免费的，你为啥提供一个官网链接就收费？  
为了卖个课程，借助 IP 光环和粉丝信任，用尽了各种营销手段（什么一顿麦当劳的饭钱，什么一杯奶茶的钱等等），贩卖各种焦虑。

应了那句话，只要你有病，就会有人给你治。

因为信息差的存在，本来免费的东西，很多人还是会被割韭菜，

也因为缺少独立思考，本来不切实际的东西，不符合事物发展规律的东西，因为 IP 光环，很多人却愿意买单。

一个社会的良性发展，需要的是真正给社会创造价值的人，而不是天天贩卖焦虑然后从人民身上吸血的人。

## 现状痛点

如果你跟着文章 《[Claude Code 核心组件 Agent 配置，让你的 cc 拥有三头六臂（纯肝货）](https://mp.weixin.qq.com/s?__biz=Mzk3NTcyMzA0Nw==&mid=2247484014&idx=1&sn=53a0125935c5446392bbddeace7324a1&scene=21#wechat_redirect)》过了一遍，安装了大量的 Agent 之后，紧接着需要思考的问题是：为什么要安装这么多Agent，所有Agent都有用吗？

就像人生要做减法一样的道理，我们使用工具是为了解决问题，黑猫白猫，能抓住耗子的猫就是好猫。不能为了工具而工具。

那有没有一种更好的管理方式，能根据问题，自动调度合适的 Agent 去完成任务呢？

答案是：有

## 改进方向

所以我们的改进方向就是：寻找一个可以帮我们调度各种 Agent 的助理，让他去操心调度的事情。

## 改进方案和路线

社区已有的一个方案，老规矩，先上一手资料链接：

https://github.com/ruvnet/claude-flow

## Claude Flow 概要介绍

官方的文档介绍：一句话总结就是一个智能体编排平台

核心开发理论方法：

SPARC (Specification, Pseudocode, Architecture, Refinement, Completion)  
参考：https://github.com/ruvnet/claude-flow/wiki/SPARC-Methodology

多智能体协作方式有两种模式，我们今天重点讨论第一种 swarm（蜂群）模式

| 特性 | swarm（群蜂模式） 命令 | hive-mind（蜂巢模式） 命令 |
| --- | --- | --- |
| **适用情况** | 快速任务，单一目标 | 复杂项目，持久会话 |
| **设置** | 即时 - 无需配置 | 交互式向导设置 |
| **会话** | 临时协作 | 持久化，支持恢复 |
| **记忆** | 任务范围 | 项目范围，使用 SQLite 存储 |
| **智能体** | 为任务自动生成 | 手动控制，具备专业分工 |
| **使用场景** | "构建 X"，"修复 Y"，"分析 Z" | 多功能项目，团队协作 |

## Claude Flow 快速入门和实际体验

### 安装

还是那句话，AI 能做的就交给 AI

先安装

### 初始化

然后初始化 （claude-flow 我简写成 cf）

```
cf init
```

```
➜  demo git:(main) ✗ cf init  
🚀 Initializing Claude Flow v2.0.0 with enhanced features...  
✅ ✓ Created CLAUDE.md (Claude Flow v2.0.0 - Optimized)  
✅ ✓ Created .claude directory structure  
✅ ✓ Created .claude/settings.json with hooks and MCP configuration  
  ⚠️  Could not create statusline script, skipping...  
✅ ✓ Created .claude/settings.local.json with default MCP permissions  
✅ ✓ Created .mcp.json at project root for MCP server configuration  
  ✓ Created 3 analysis command docs  
  ✓ Created 3 automation command docs  
  ✓ Created 3 coordination command docs  
  ✓ Created 5 github command docs  
  ✓ Created 5 hooks command docs  
  ✓ Created 3 memory command docs  
  ✓ Created 3 monitoring command docs  
  ✓ Created 3 optimization command docs  
  ✓ Created 3 training command docs  
  ✓ Created 3 workflows command docs  
  ✓ Created 9 swarm command docs  
  ✓ Created 11 hive-mind command docs  
  ✓ Created 4 agents command docs  
  ✓ Created local claude-flow executable wrapper  
    You can now use: ./claude-flow instead of npx claude-flow  
✅ ✓ Created 6 helper scripts  
✅ ✓ Created standard directory structure  
✅ ✓ Initialized memory system  
[2025-10-05T13:01:45.383Z] INFO [memory-store] Initialized SQLite at: /Users/asher/Documents/0_输出-公众号持续写作/Oct-05/.swarm/memory.db  
[2025-10-05T13:01:45.383Z] INFO [fallback-store] Successfully initialized SQLite store  
✅ ✓ Initialized memory database (.swarm/memory.db)  
  
🧠 Initializing Hive Mind System...  
🧠 Initializing Hive Mind System...  
  📁 Creating hive-mind directory structure...  
  ✅ Hive-mind directories created  
  🧠 Initializing collective memory database...  
    📋 Creating database schema...  
    🔍 Creating performance indexes...  
  ✅ Collective memory database initialized with full schema  
  👑 Creating default queen and worker configurations...  
  ✅ Default configurations created  
  ⚙️ Creating hive-mind configuration file...  
  ✅ Hive-mind configuration created  
  📚 Creating hive-mind documentation...  
  ✅ Hive-mind documentation created  
✅ 🧠 Hive Mind System initialized successfully  
✅ ✓ Hive Mind System initialized with 6 features  
    • Collective memory database  
    • Queen and worker configurations  
    • Consensus mechanisms  
    • Performance monitoring  
    • Session management  
    • Knowledge base  
✅ ✓ Created .gitignore with Claude Flow entries  
  
🔍 Claude Code CLI detected!  
  
🔌 Setting up MCP servers for Claude Code...  
  🔄 Adding claude-flow...  
MCP server claude-flow already exists in local config  
  ⚠️  Failed to add claude-flow: Command failed: claude mcp add claude-flow npx claude-flow@alpha mcp start  
     You can add it manually with: claude mcp add claude-flow npx claude-flow@alpha mcp start  
  🔄 Adding ruv-swarm...  
MCP server ruv-swarm already exists in local config  
  ⚠️  Failed to add ruv-swarm: Command failed: claude mcp add ruv-swarm npx ruv-swarm mcp start  
     You can add it manually with: claude mcp add ruv-swarm npx ruv-swarm mcp start  
  🔄 Adding flow-nexus...  
MCP server flow-nexus already exists in local config  
  ⚠️  Failed to add flow-nexus: Command failed: claude mcp add flow-nexus npx flow-nexus@latest mcp start  
     You can add it manually with: claude mcp add flow-nexus npx flow-nexus@latest mcp start  
  🔄 Adding agentic-payments...  
MCP server agentic-payments already exists in local config  
  ⚠️  Failed to add agentic-payments: Command failed: claude mcp add agentic-payments npx agentic-payments@latest mcp  
     You can add it manually with: claude mcp add agentic-payments npx agentic-payments@latest mcp  
  
  📋 Verifying MCP servers...  
Checking MCP server health...  
  
ruv-swarm: npx ruv-swarm mcp start - ✓ Connected  
claude-flow: npx claude-flow@alpha mcp start - ✓ Connected  
flow-nexus: npx flow-nexus@latest mcp start - ✓ Connected  
agentic-payments: npx agentic-payments@latest mcp - ✓ Connected  
  
🤖 Setting up agent system...  
  ✅ Created 30 agent directories  
  📁 Using packaged agent files  
📁 Copying agent system files...  
  📂 Source: /Users/asher/.nvm/versions/node/v22.14.0/lib/node_modules/claude-flow/.claude/agents  
  📂 Target: /Users/asher/Documents/0_输出-公众号持续写作/Oct-05/demo/.claude/agents  
  ✅ Copied 74 agent files  
  📋 Agent system initialized with 64 specialized agents  
  🎯 Available categories: Core, Swarm, Consensus, Performance, GitHub, SPARC, Testing  
  🔍 Agent system validation:  
    • Categories: 19  
    • Total agents: 64  
    • Categories: analysis, architecture, consensus, core, data, development, devops, documentation, flow-nexus, github, goal, hive-mind, neural, optimization, sparc, specialized, swarm, templates, testing  
  
📚 Setting up command system...  
  📁 Using packaged command files  
📁 Copying command system files...  
  📂 Source: /Users/asher/.nvm/versions/node/v22.14.0/lib/node_modules/claude-flow/.claude/commands  
  📂 Target: /Users/asher/Documents/0_输出-公众号持续写作/Oct-05/demo/.claude/commands  
  ✅ Copied 79 command files  
  📋 Command system initialized with comprehensive documentation  
  🎯 Available categories: Analysis, Automation, GitHub, Hooks, Memory, Flow Nexus  
✅ ✓ Command system setup complete with Flow Nexus integration  
✅ ✓ Agent system setup complete with 64 specialized agents  
  
🎉 Claude Flow v2.0.0 initialization complete!  
  
🧠 Hive Mind System Status:  
  Configuration: ✅ Ready  
  Database: ✅ SQLite  
  Directory Structure: ✅ Created  
  
📚 Quick Start:  
1. View available commands: ls .claude/commands/  
2. Start a swarm: npx claude-flow@alpha swarm "your objective" --claude  
3. Use hive-mind: npx claude-flow@alpha hive-mind spawn "command" --claude  
4. Use MCP tools in Claude Code for enhanced coordination  
5. Initialize first swarm: npx claude-flow@alpha hive-mind init  
  
💡 Tips:  
• Check .claude/commands/ for detailed documentation  
• Use --help with any command for options  
• Run commands with --claude flag for best Claude Code integration  
• Enable GitHub integration with .claude/helpers/github-setup.sh  
• Git checkpoints are automatically enabled in settings.json  
• Use .claude/helpers/checkpoint-manager.sh for easy rollback
```

初始化总结一下就做了如下几件事情

1. 1. 创建 Claude Code 提示词文件：CLAUDE.md

1. 1. 定义了 **SPARC** 方法

2. 2. 引用 MCP 工具：在.mcp.json  引用了 claude-flow这个核心 MCP 服务
3. 3. 安装 Claude Code 配套措施

1. 1. 150 个斜杠命令
2. 2. 64 个 Agent

4. 4. 配置 claude-flow AI 编排平台相关内容

1. 1. 初始化记忆系统  memory.db 文件 （SQLite记忆数据库）
2. 2. 配置群蜂系统   Hive Mind System

初始化创建的文件夹和文件

详细文件

### 使用群蜂模式完成一个任务

然后按照  SPARC (Specification, Pseudocode, Architecture, Refinement, Completion)  方法论生成代码

一条命令即可

```
claude-flow swarm "Build API" --claude
```

最后生成的效果：

> Claude-flow 生成的项目文件组织规则
>
> * • `/docs` - 文档和 markdown 文件
> * • `/src` 源代码文件
> * • `/tests` 测试文件
> * • `/config` 配置文件
> * • `/scripts` - 实用程序脚本
> * • `/examples` - 示例代码

燃烧了很多token

```
> /cost   
  ⎿ Total cost:            $7.95 (costs may be inaccurate due to usage of unknown models)  
     Total duration (API):  49m 59s  
     Total duration (wall): 51m 28s  
     Total code changes:    15839 lines added, 241 lines removed  
     Usage by model:  
              glm-4.6:  831.3k input, 144.3k output, 10.7m cache read, 0 cache write ($7.86)  
              glm-4.5-air:  12.1k input, 2.0k output, 59.4k cache read, 0 cache write ($0.0848)
```

大致的工作原理如下：

## 内容小结

通过 Claude Flow 的“群蜂模式（Swarm）”，让多个智能体像蜂群一样自动协作分工，实现任务的智能调度与高效完成，从而真正做到“让 AI 管理 AI”，解放人类的创造力。

---

作者介绍

我是 Asher 同学，一个喜欢独立深入思考，热爱技术和编程，对代码质量有追求，喜欢分享的编程创作者。

目前在深耕人工智能领域，会长期更新人工智能和软件工程相关的内容。

写作不易，如果文章对你有帮助，记得点赞，转发，推荐一键三连哦！🌹🌹

如需交流学习，请移步公众号后台，添加小编好友，邀请进群