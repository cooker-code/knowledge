---
title: DeerFlow 2.0：字节跳动开源的 Super Agent Harness 到底强在哪？
author: 智物社
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI4MTc1Njg0MA==&mid=2247485579&idx=1&sn=9c9f870c900a2e5feb3d3b6929a22479&chksm=ea3a82489a60b67ce3b5fe5fb273cda2d1704bef6c91e57bf4930717a48c186d409a36b10d2e&mpshare=1&scene=24&srcid=0419zH2wlbZNdxzgT1t7p4zx&sharer_shareinfo=3448ed80394865f516e9f712ae9f0760&sharer_shareinfo_first=3448ed80394865f516e9f712ae9f0760#rd
---

2026年2月28日，字节跳动在 GitHub 上发布了 DeerFlow 2.0。

这货一发布就冲上了 Trending 第一名。

我一开始没当回事——开源 Agent 项目太多了，卷来卷去的。但仔细一看，发现有点意思：**它不是另一个 Agent 框架，而是一个" Harness"，也就是"马具"，负责把各种组件组装起来。**

今天就来拆解一下这个项目，看看它葫芦里卖的什么药。

---

## 01 DeerFlow 到底是什么？

官方定义是：**DeerFlow = Deep Exploration and Efficient Research Flow**

翻译过来就是一个"深度探索和研究工作流"的框架。

但我觉得更准确的描述是：**它是一个 Super Agent 的"组装厂"。**

什么意思？

市面上大多数 Agent 项目，都是自己从头实现一切。但 DeerFlow 不一样——它更像一个**调度中心**，负责把 Sub-agents、Memory、Skills、Sandbox 这些现成的组件拼起来，让它们协同工作。

```
  DeerFlow  
├── Skills & Tools（技能系统）  
├── Sub-Agents（子代理）  
├── Sandbox（沙箱）  
├── Memory（记忆系统）  
└── Claude Code Integration（Claude Code 集成）
```

这种思路有点像"乐高"——不自己造积木，而是提供一个标准接口，让各种积木能插在一起。

---

## 02 核心架构：它是怎么组织的？

DeerFlow 2.0 是一个**完全重写的版本**，跟 v1 没有任何共享代码。

从架构上看，它的核心是这样的：

```
  # 这是一个简化的架构示意  
class DeerFlow:  
    def __init__(self):  
        self.skills = SkillRegistry()      # 技能注册表  
        self.subagents = SubAgentPool()    # 子代理池  
        self.memory = MemorySystem()       # 记忆系统  
        self.sandbox = SandboxManager()    # 沙箱管理  
      
    async def run(self, task):  
        # 1. 理解任务  
        plan = await self.planner.plan(task)  
          
        # 2. 调度子代理  
        results = await self.subagents.execute(plan)  
          
        # 3. 在沙箱中执行  
        output = await self.sandbox.run(results)  
          
        # 4. 记住上下文  
        await self.memory.save(task, output)  
          
        return output
```

重点来了——**它用 LangGraph 来做 agent 的编排**。

```
  # config.yaml  
models:  
  - name: claude-sonnet  
    use: deerflow.models.claude_provider:ClaudeChatModel  
    model: claude-sonnet-4-6  
    supports_thinking: true
```

这就很聪明了——不重复造轮子，直接站在 LangGraph 的肩膀上。

---

## 03 它和其他 Agent 有什么不同？

我对比了一下市面上的主流开源 Agent：

| 特性 | DeerFlow | OpenHands | AutoGPT | Claude Code |
| --- | --- | --- | --- | --- |
| Sub-agents | ✅ | ❌ | ✅ | ✅ |
| Memory | ✅ | 有限 | ✅ | 有限 |
| Sandbox | ✅ | ✅ | ❌ | ❌ |
| Skills | ✅ | ✅ | ❌ | ✅ |
| Claude Code 集成 | ✅ | ❌ | ❌ | N/A |
| LangGraph | ✅ | ❌ | ❌ | ❌ |

几个关键点：

### 1. Sandbox 是标配

DeerFlow **强制使用沙箱执行**，不安全的行为（文件操作、网络请求）都在隔离环境里跑。

这很重要——之前有 Agent 删库跑路的例子。

### 2. Claude Code 可以直接接入

```
  # 配置 Claude Code 作为推理引擎  
models:  
  - name: claude-sonnet-4.6  
    use: deerflow.models.claude_provider:ClaudeChatModel  
    model: claude-sonnet-4-6  
    supports_thinking: true
```

注意那个 `supports_thinking: true`——这是 Claude Code 的推理模型支持。

### 3. InfoQuest 集成

DeerFlow 还接入了字节自己的**InfoQuest**——一个智能搜索和爬取工具。

这意味着：DeerFlow 不仅能执行代码，还能自己**去网上搜资料**。

---

## 04 怎么跑起来？

最简单的方式是 Docker：

```
  git clone https://github.com/bytedance/deer-flow.git  
cd deer-flow  
make config  # 生成配置文件  
# 然后编辑 config.yaml 填入你的 API Key  
make docker-start  
# 访问 http://localhost:2026
```

本地开发模式：

```
  make install  
make dev  
# 也是访问 http://localhost:2026
```

支持的模型（官方推荐）：

* • **Doubao-Seed-2.0-Code**（字节亲儿子）
* • **DeepSeek v3.2**
* • **Kimi 2.5**
* • **GPT-4**
* • **Claude Sonnet 4.6**

---

## 05 它适合谁？

DeerFlow 不是给普通用户玩的。它的目标用户是：

1. 1. **想自己组装 Agent 的人**——不想要一个黑盒，想要可控、可定制的系统
2. 2. **研究 Agent 架构的人**——代码开源，可以直接看它怎么调度 sub-agents
3. 3. **企业级应用**——Sandbox + Memory + Skills 的组合，适合构建自动化工作流

如果你只是想找个 Agent 帮你写代码，可能 Claude Code 或 Cursor 更合适。

但如果你想**理解 Agent 的底层逻辑**，或者**构建自己的 Agent 系统**，DeerFlow 是个很好的参考。

---

## 06 总结

DeerFlow 2.0 最大的价值不是它自己有多强，而是它提供了一个**可组合的架构**。

* • 用 LangGraph 做编排
* • 用 Claude Code 做推理
* • 用 Sandbox 做安全隔离
* • 用 Memory 做长期记忆
* • 用 Skills 做能力扩展

这种"组装思维"可能是未来 Agent 发展的方向——**与其自己造轮子，不如做一个轮子商店。**

---

参考来源：

* • GitHub: https://github.com/bytedance/deer-flow
* • 官网: https://deerflow.tech

---

觉得有用，点个赞、在看，转发给需要的朋友。

我是智物社，下次见。