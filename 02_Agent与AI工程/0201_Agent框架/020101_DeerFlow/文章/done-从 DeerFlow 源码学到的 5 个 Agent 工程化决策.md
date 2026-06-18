> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020101_DeerFlow/020101_核心知识点/DeerFlow源码中的中间件与Harness边界|DeerFlow源码中的中间件与Harness边界]]
---
title: 从 DeerFlow 源码学到的 5 个 Agent 工程化决策
author: 启琛
date:
url: https://mp.weixin.qq.com/s?__biz=MzIzNjQ4NDU0MA==&mid=2247486997&idx=1&sn=8fa17babf27b8ebef5efd2605a2d58fc&chksm=e97714f3d431008f986714fd45265774c546e9abf2b4729c0d6c9180ebda906877924d38607f&mpshare=1&scene=24&srcid=03317mntqGdM3dOjc3E8nH4p&sharer_shareinfo=b5b8d41d54e7cf94bb3f66b065fd636d&sharer_shareinfo_first=b5b8d41d54e7cf94bb3f66b065fd636d#rd
---

#

昨天我的小伙伴在群里发了一条消息：

> "新来的小伙子推荐的字节的 agent 架构，这个 repo 最近比较火"

又附加一句："我经常看到，但是没时间细究。"

然后今天又在另一个群里看到有人提到deer-flow

我看到这条消息的时候，正好也是"经常看到但没细究"的状态。DeerFlow 这个项目，我也刚听说，趁这个机会，让 Claude 帮我把后端 15K 行 Python 梳理了一遍——它负责检索和整理代码结构，我来判断哪些值得讲。

说实话，这种方式读开源项目效率高很多。以前读一个陌生项目，光搞清楚目录结构和模块关系就要半天。现在丢进去让模型先跑一遍，把调用链、关键类、设计决策都列出来，我再重点看那几个真正有意思的地方。

架构设计上有几个判断让我觉得值得认真讲——不是因为它完美，而是因为它把**从 Demo 走到生产这段路上会踩的坑，基本都提前踩了一遍**。

---

## DeerFlow 是什么

先说清楚这东西解决什么问题。

DeerFlow（Deep Exploration and Efficient Research Flow）定位是"超级智能体框架"，基于 LangGraph 构建，核心能力是：

* 编排子智能体并行执行任务
* 持久化长期记忆系统
* 隔离沙箱环境（本地 / Docker 均支持）
* MCP 协议集成，支持外部工具动态接入
* 多模型支持（思维链、视觉理解）

它不是一个"用 AI 做某件事"的应用，而是一个**让你能快速搭建生产级 Agent 系统的框架**。

架构上分三层：

```
Frontend (Next.js)  →  Gateway API (FastAPI)  →  LangGraph Server
                                                        ↓
                                          Sandbox / Memory / MCP
```

后端核心又分两个包：

* **deerflow-harness**：框架层，纯 Python，无 Web 依赖，可独立发布
* **app**：应用层，FastAPI + IM 渠道集成（飞书、钉钉等）

这个分层是有意为之的设计决策，后面会说为什么重要。

---

## 设计哲学：它在用什么思路解决问题

读完源码，我觉得 DeerFlow 的设计哲学可以用三句话概括：

**横切关注点不能污染主流程。****基础设施细节不能泄漏给 Agent。****能异步的事情绝不同步阻塞。**

这三个判断贯穿整个系统。展开来说。

---

### 判断一：横切关注点用中间件隔离

大多数 LangGraph 教程给你看的是这样的：

```
agent = create_react_agent(model, tools)
result = agent.invoke({"messages": [HumanMessage(content="帮我查一下天气")]})
```

十行代码，跑起来，很爽。

然后生产需求来了：怎么加持久化记忆？怎么支持文件上传？怎么让 Agent 调用另一个 Agent？怎么防止无限循环？怎么做上下文压缩？怎么控制并发？

每加一个需求，代码开始膨胀，主流程里开始出现各种 if 判断和前置后置处理。最后你会发现，**真正难的不是"怎么调 LLM"，而是怎么把十几个横切关注点整合进来，同时不让主流程变成一锅粥**。

DeerFlow 的答案是中间件管道。主 Agent（`make_lead_agent`）创建时按固定顺序装载 12 个中间件：

```
1. ThreadDataMiddleware      → 创建线程隔离目录
2. UploadsMiddleware         → 注入 <uploaded_files> 到上下文
3. SandboxMiddleware         → 启动/回收沙箱
4. DanglingToolCallMiddleware → 修补 LLM 漏掉的 ToolMessage
5. GuardrailMiddleware       → 工具调用授权检查
6. SummarizationMiddleware   → 上下文超长时自动压缩
7. TodoMiddleware            → Plan Mode 下的任务追踪
8. TitleMiddleware           → 首次完整对话后自动生成标题
9. MemoryMiddleware          → 异步队列更新持久化记忆
10. ViewImageMiddleware      → 注入 base64 图片（视觉支持）
11. SubagentLimitMiddleware  → 截断超出并发限制的子任务
12. ClarificationMiddleware  → 拦截澄清请求（必须最后）
```

每个中间件职责单一。顺序不是随意的——`ThreadData` 必须在 `Sandbox` 前面，因为沙箱需要 `thread_id` 来决定工作目录；`Clarification` 必须在最后，因为它要在所有处理完成后拦截最终响应。

**顺序依赖就是真实的业务依赖，这本身就是设计的一部分。**

中间件可以通过配置在运行时启用或禁用：

```
config.configurable = {
    "is_plan_mode": True,        # 启用 TodoMiddleware
    "subagent_enabled": True,    # 启用 task 工具 + 并发限制
    "thinking_enabled": False,   # 关闭扩展思维模式
}
```

遇到新问题，加一个中间件，不动主流程。

---

### 判断二：基础设施细节不该泄漏给 Agent

###

Agent 看到的文件系统路径：

```
/mnt/user-data/workspace/
/mnt/user-data/uploads/
/mnt/user-data/outputs/
```

实际物理路径：

```
backend/.deer-flow/threads/{thread_id}/user-data/workspace/
```

`LocalSandbox` 维护双向映射，输入时把 Agent 的虚拟路径翻译成物理路径，执行后再把命令输出里的物理路径反向替换回虚拟路径：

```
def _resolve_path(self, path: str) -> str:
    # 最长前缀优先匹配，避免路径前缀冲突
    for container_path, local_path in sorted(self.path_mappings.items(),
                                              key=lambda x: len(x[0]), reverse=True):
        if path.startswith(container_path + "/"):
            relative = path[len(container_path):].lstrip("/")
            return str(Path(local_path) / relative)
    return path
```

Agent 不知道自己在操作本地文件系统还是 Docker 容器，也不关心 `thread_id` 是什么。底层换实现，Agent 的代码和提示词不用改。

**这事说白了是一个字符串替换，但它把基础设施细节封在了正确的地方。**

同样的思路体现在 MCP 工具上。MCP 服务器可能暴露几十上百个工具，如果全部发给 LLM，上下文直接爆。DeerFlow 的做法是延迟注册：MCP 工具对 LLM 不可见，LLM 只有一个 `tool_search` 工具：

```
if config.tool_search.enabled:
    registry = DeferredToolRegistry()
    for t in mcp_tools:
        registry.register(t)      # 不发给 LLM
    builtin_tools.append(tool_search_tool)  # LLM 只看到这一个
```

需要某个工具时，先 `tool_search` 查，查到了再调用。工具数量对 LLM 上下文透明。

---

### 判断三：能异步的事绝不同步阻塞

###

记忆更新是个典型场景。每次对话结束后需要调用 LLM 提取事实、更新记忆文件——这个操作耗时不定，不能让用户等着。

DeerFlow 的处理链：

1. `MemoryMiddleware` 过滤消息（只保留用户输入 + 最终 AI 输出），入队
2. `MemoryQueue` 做 **30 秒去抖动**，合并频繁触发的更新请求
3. `MemoryUpdater` 异步调用 LLM 提取事实，写入文件
4. 文件写入用 `temp → rename` 原子操作

记忆结构分三层：

```
user.workContext      → 工作环境总结
user.personalContext  → 个人信息
user.topOfMind        → 当前关注点

history.recentMonths      → 近期历史
history.earlierContext    → 较早历史
history.longTermBackground → 长期背景

facts[]               → 具体事实，带 confidence 分值
```

注入时取前 15 条 facts + 上下文摘要，插入系统提示词的 `<memory>` 块。

30 秒的去抖动窗口意味着连续对话时多次更新会合并成一次 LLM 调用。**不是最聪明的方案，但够用，不复杂，对主流程零影响。**

子智能体也遵循同样的思路。`task()` 工具让 Agent 把子任务委托给子 Agent 并行执行，用双线程池分离调度和执行：

```
_scheduler_pool = ThreadPoolExecutor(max_workers=3)  # 接收提交
_execution_pool  = ThreadPoolExecutor(max_workers=3)  # 实际运行

MAX_CONCURRENT_SUBAGENTS = 3
SUBAGENT_TIMEOUT = 15 * 60  # 15 分钟超时
```

子任务状态机：`PENDING → RUNNING → COMPLETED / FAILED / TIMED_OUT`

如果 LLM 一次提交 5 个任务，`SubagentLimitMiddleware` 截断多余的，让 Agent 知道哪些没被接受，下一轮再提。**限流不拒绝意图，只控制并发度。**

---

## 一条必须从第一天画好的线

DeerFlow 把框架层（harness）和应用层（app）严格分开，规则是：**harness 不能导入 app 的任何东西**。

这条规则用 CI 测试强制检查：

```
# tests/test_harness_boundary.py
# 扫描 harness 所有 Python 文件
# 发现任何 "from app." 或 "import app." → 测试失败
```

好处显而易见：harness 可以在没有 FastAPI 的环境独立运行，可以被其他应用复用，测试时不需要启动整个 Web 服务。

代价是执行成本——一旦有人"顺手"从 app 导了个工具类，边界就破了。所以用 CI 来强制，不靠人工自觉。

**不是所有项目都需要这种分层，但一旦你想让框架层可独立发布，这条线必须从第一天画好，因为后期重构的代价极高。**

---

## 你能从中借鉴什么

拆完这个项目，我觉得以下几个思路是可以直接用到自己项目里的，跟你用不用 LangGraph、用不用 DeerFlow 都没关系：

**1. 用中间件隔离横切关注点**

不只是 Agent 系统，任何有"主流程 + 多个旁路逻辑"的系统都适用。关键是把横切关注点从主流程里提出来，让它们可以独立测试、独立开关。

判断标准：如果你的核心函数里有超过 3 个"顺便处理 X"的逻辑，考虑中间件。

**2. 基础设施细节不要泄漏给上层**

Agent 不该知道文件存在哪里，LLM 不该知道有多少个 MCP 工具。每个层次只接触它需要的抽象。这不是过度设计，这是在保护你的上层逻辑不被下层变更拖累。

**3. 异步队列 + 去抖动处理非关键更新**

记忆更新、日志写入、统计上报这类操作，不要放在主请求链路上。30 秒去抖动这个模式比"立即异步"更省资源，因为它合并了高频触发的重复请求。

**4. 架构边界用测试来守护**

不要靠 code review 和团队自觉。把关键的架构约束写成测试，CI 上跑。边界一旦破了立刻红，不等到 review 环节。

**5. 状态机 + 超时是并发控制的标配**

子任务的 `PENDING → RUNNING → COMPLETED / FAILED / TIMED_OUT` 状态机不复杂，但它让你在任何时刻都知道每个任务处于什么状态，出了问题有迹可查。15 分钟超时是个合理的默认值——不是精确调优，是防止任务永远挂着。

---

## 最后

DeerFlow 现在还在早期阶段，不是说你拿来直接用就万无一失。但它展示了一种**把"让 Agent 能用"和"让系统能跑"这两件事同时做好**的工程方式。

这两件事在 Demo 阶段是同一件事，在生产阶段是两件完全不同的事。

中间件模式的代价是调试成本——遇到问题你需要理解整个执行链才能定位。这个代价在小项目上可能不值得，但在需要持续演进的系统里，它换来的是每次新增需求时不动主流程的能力。

建议把 `backend/packages/harness/deerflow/agents/middlewares/` 目录翻一遍。不是为了抄，而是为了知道你的系统在哪些问题上还没有答案。

---