---
title: AI Coding 进阶之路：Claude Code 两周实战总结
author: MaxCreator
date: 
url: https://mp.weixin.qq.com/s?__biz=MzY5NDIyNDAzMQ==&mid=2247483658&idx=1&sn=c032f5ae6fb9925a1cd4ed2c7a2d4d7d&chksm=f59bfb1c1f4e90ea064b205311de7afe8aff54f514f88248dc0febd5269459ee1e61b8d4672d&mpshare=1&scene=24&srcid=0424ELcdbaqwabNR3uNIOje7&sharer_shareinfo=a5eec7127319d1a986be62cb6f98adc5&sharer_shareinfo_first=a5eec7127319d1a986be62cb6f98adc5#rd
---

## 前言

过去两周，我通过 Claude Code 系统学习了 AI Agent 的核心知识体系。从 Agent Loop 到 Memory 管理，从 Hooks 机制到 MCP 协议，每天都在进步。这篇文章整理这两周的学习历程和核心收获。

---

## 📅 学习时间线

### 第一周（04-06 ~ 04-12）：打基础

| 日期 | 学习内容 |
| --- | --- |
| 04-07 | AI Agent 课程学习 |
| 04-09 | AgentScope 调用 MCP |
| 04-11 | learn-claude-code 仓库下载 + 学习计划制定 |
| 04-12 | s01\_agent\_loop.py 源码解读 |

### 第二周（04-13 ~ 04-15）：实战提升

| 日期 | 学习内容 |
| --- | --- |
| 04-13 | AgentScope SKILL 调用 |
| 04-14 | Claude Code day7\_hooks + day8\_memory + 全知识点复习 |
| 04-15 | MCP 基础知识 + 旅游规划开源项目试用 |

---

## 📚 核心知识点

### 1. Agent Loop（Agent 循环）

**核心概念：** LLM 思考 → 判断是否需要工具 → 执行工具 → 返回结果 → 继续思考

```
用户输入 → LLM 思考 → 判断是否需要工具                          ↓ 是                    返回 tool_call（工具名+参数）                          ↓                    Agent 执行工具                          ↓                    格式化结果返回给 LLM                          ↓                    LLM 继续思考 → 最终回复
```

**本质：** LLM 是大脑，Agent 是手，工具是工具箱，循环是血液循环。

---

### 2. Context Management（上下文管理）

**核心问题：** Token 限制下如何管理上下文

**两种策略：**

| 策略 | 做法 |
| --- | --- |
| 滑动窗口 | 固定大小缓冲区，保留最近 N 条 |
| 摘要 | 压缩成摘要，保留关键信息 |

---

### 3. Planning（规划）

**两种模式：**

| 模式 | 做法 |
| --- | --- |
| 一次性规划 | 先拆解所有步骤，再执行 |
| 增量规划 | 执行一步，根据结果决定下一步 |

**类比：** 增量规划 = 迭代开发，计划修正 = 敏捷变更

---

### 4. Hooks（钩子）

**核心思想：** 在不修改核心代码的情况下，插入自定义逻辑

**三种类型：**

| 类型 | 时机 | 用途 |
| --- | --- | --- |
| Pre-Hook | 执行前 | 参数验证、权限检查 |
| Post-Hook | 执行后 | 日志记录、结果处理 |
| Error-Hook | 出错时 | 错误处理、重试机制 |

**Hook vs 插件：**

|  | Hook | 插件 |
| --- | --- | --- |
| 重量 | 轻量（单点） | 重型（完整模块） |
| 典型用途 | 日志、校验 | 微信、飞书 |

---

### 5. Memory（记忆）

**核心问题：** Agent 如何跨会话记住信息？

**两种 Memory 类型：**

| 类型 | 作用 | 持久化 |
| --- | --- | --- |
| 短期记忆 | 会话内消息历史 | 内存，会话结束消失 |
| 长期记忆 | 跨会话持久化 | 文件/数据库 |

**Memory 的读写时机：**

* **读取：**

  用户输入时，加载到上下文
* **写入：**

  执行过程中，重要信息持久化

---

### 6. MCP 协议

**核心问题：** 如何让 Agent 调用外部服务？

**MCP 架构：**

```
Agent → MCP Client → MCP Server → 外部服务                          ↓              工具定义注册 + 请求转发
```

**MCP vs 普通工具：**

|  | 普通工具 | MCP |
| --- | --- | --- |
| 定义位置 | 代码 | 配置文件 |
| 调用方式 | 直接函数调用 | HTTP/WebSocket |
| 扩展性 | 低 | 高 |

---

## 💻 实战练习

### day3\_todo - Todo CLI 工具

* 实现了 Todo CLI 工具 + 单元测试

### day4\_agent\_loop - 简单 Agent Loop

* 理解了 Agent Loop 的核心模式

### day4\_real\_agent - 真正调用 MiniMax API

* 学会了调用实际的大模型 API

### day7\_hooks - Hook 机制

```
class Hook:    def before(self, context):        pass    def after(self, context, result):        passclass LoggingHook(Hook):    def before(self, context):        print(f"开始: {context}")    def after(self, context, result):        print(f"完成: {result}")
```

### day8\_memory - Memory 类实现

```
class Memory:    def save(self, key, value):        self.data[key] = value        json.dump(self.data, open(self.path, "w"))    def build_context(self):        return "【记忆】" + " | ".join([f"{k}: {v}" for k, v in self.data.items()])
```

---

## 🔍 知识弱点分析

通过这两周的学习，发现还有 4 个薄弱点需要加强：

| 序号 | 主题 | 弱点 |
| --- | --- | --- |
| 1 | Planning | 计划生成的提示词方法 |
| 2 | Context | 按 token 裁剪 vs 按条数裁剪 |
| 3 | Tool | 参数校验失败处理 |
| 4 | Memory | 向量检索 / RAG |

---

## 💡 核心感悟

### 1. 学习方法论

* **费曼学习法：**

  能用自己的话说清楚，才算真正理解
* **追问本质：**

  每个概念问自己"它的本质是什么"
* **实践驱动：**

  看十遍不如做一遍

### 2. AI 辅助学习

用 AI 辅助学习 AI，是最高效的学习模式：

* 个性化引导
* 即时反馈
* 进度追踪
* 知识串联

### 3. 组件化思维

> skill 不是"代码"，是"可执行的文档"

把学习成果组件化，可以复用、组合、优化。

---

## 📊 学习进度

### Week 1（04-06 ~ 04-12）

| 任务 | 状态 | 说明 |
| --- | --- | --- |
| AI Agent 课程 | ✅ | 通勤时间学习 |
| AgentScope MCP | ✅ | 调用方法学习 |
| learn-claude-code 仓库 | ✅ | 下载完成 |
| Claude Code 学习计划 | ✅ | 2-3周计划制定 |
| s01 源码解读 | ✅ | Agent Loop 核心模式 |

### Week 2（04-13 ~ 04-15）

| 任务 | 状态 | 说明 |
| --- | --- | --- |
| AgentScope SKILL | ✅ | 环境管理 SKILL 调通 |
| day7\_hooks | ✅ | Hook 机制练习 |
| day8\_memory | ✅ | Memory 类实现 |
| 知识点复习 | ✅ | 全知识点复习完成 |
| MCP 基础 | ✅ | 协议原理和架构 |

---

## 🔗 工作中的实践

### MCP Server 架构决策

**场景：** 微服务框架下，多服务场景

**决策：** 使用单独一个服务构建 MCP Server，简化外部调用关系

**架构图：**

```
外部系统 → MCP Server（统一入口）→ 各个微服务                    ↓            工具注册 + 调度
```

### AI Agent 全流程 Demo

**目标：** 月底前将原始微服务架构升级为 AI Agent 架构

**SKILL 规划：**

* 测试设计-硬件 Skill
* 测试设计-性能 Skill
* 测试执行 Skill
* 测试分析 Skill
* 环境管理 Skill

---

## 结语

AI Agent 的学习是一个循序渐进的过程。从 Agent Loop 到 Context Management，从 Hooks 到 Memory，从 Planning 到 MCP，每一个概念都是构建复杂 AI 系统的基石。

**学习路上，陪伴很重要。** 有 AI 助手的持续记录和反馈，学习变得更高效、更有方向。

---

*如果你也在学习 AI Agent，欢迎交流讨论，我的blog：sunrong.site*