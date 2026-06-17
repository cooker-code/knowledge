---
title: Agent Harness 架构真相：Prompt Cache 如何决定 Skill、MCP 与 SubAgent 设计
author: AI 小老六
date: AI 小老六AI 小老六
url: https://mp.weixin.qq.com/s?__biz=Mzg2OTcxMDIzNA==&mid=2247483872&idx=1&sn=a44f5c7cc75abe0f4cae01c1dbabd68f&chksm=cf260244e32ee63feef3c18dcf74fc089287dbde6c644f67b6992f40d698c38f52dd2cb741cb&mpshare=1&scene=24&srcid=06022LvAkxK3r9Z0agz6dsUE&sharer_shareinfo=19fc236686abee2da9d1e27c2ec9f81a&sharer_shareinfo_first=19fc236686abee2da9d1e27c2ec9f81a#rd
---

> 拆解 Agent Harness 的上下文分层、Skill 延迟加载与 SubAgent 隔离设计。

很多人第一次看 Claude Code、Codex CLI 或类似 Coding Agent 的实现，注意力会落在 Skill、MCP、SubAgent 这些新名词上。它们看起来像一组彼此独立的能力：Skill 像插件，MCP 像外部工具协议，SubAgent 像多智能体协作。

真正把代码和请求链路翻开之后，结论会朴素很多：这些机制大多不是模型侧的新能力，而是 **Agent Harness 在 `system`、`tools`、`messages` 三个输入面上做的工程编排**。难点不在于“能不能塞进去”，而在于**什么时候塞、塞到哪里、怎样不把 Prompt Cache 打碎**。

Prompt Cache 这件事很容易被低估。Agent 每轮请求都可能带上系统提示词、工具定义、历史消息、文件片段、运行时提醒和工具结果。如果**前缀稳定**，缓存命中能明显降低延迟和成本；如果动态内容插得太靠前，从变化位置之后的内容就会失去缓存价值。Claude Code 很多看似绕的设计，本质上是在守这条边界。

## Harness 的输入面：三块上下文决定了所有扩展点

*图：Agent Harness 把稳定规则、工具能力与动态消息分层管理*

一次 LLM 调用通常不神秘。无论是 Anthropic Messages API，还是 OpenAI Chat Completions 风格的接口，核心输入都绕不开三类内容：

```
{  
  "model": "claude-opus-4.7",  
  "system": "You are a helpful coding assistant.",  
  "messages": [  
    { "role": "user", "content": "..." }  
  ],  
  "tools": [  
    {  
      "name": "read_file",  
      "description": "...",  
      "input_schema": {}  
    }  
  ]  
}
```

这三个字段对应 **Harness 的三条主线**。

| 输入面 | 适合放什么 | 不适合放什么 |
| --- | --- | --- |
| `system` | 稳定身份、全局规则、安全边界 | 高频变化的项目状态、动态 skill 列表 |
| `tools` | 相对稳定的工具名、参数 schema、权限边界 | 大段文档、可变 agent 列表、临时状态 |
| `messages` | 用户输入、历史对话、工具结果、运行时注入 | 需要长期稳定缓存的大块静态规则 |

Agent Harness 的大部分能力都可以落回这张表。Skill 是把一段 prompt 延后放进上下文；SubAgent 是通过一个工具调用开出一套新的上下文；MCP 是把外部能力包装成工具 schema；attachment 是把运行时生成的信息变成额外 message。概念变多了，底层入口没有变。

## Agent Loop：模型只决定下一步，Harness 负责把世界接回来

Agent 不是一次请求给出完整答案，而是在“模型决策”和“工具执行”之间来回切换。模型输出 `tool_use`，Harness 调用真实工具，再把 `tool_result` 写回 messages，下一轮继续请求模型。

*图：Agent Loop 在模型决策与工具执行之间循环*

真实系统会在这条循环外再加很多护栏，比如最大轮次、token 上限、工具超时、权限过滤、错误重试、上下文压缩。但这些护栏不改变主结构：**LLM 负责判断下一步，Harness 负责执行下一步**。

Skill 和 SubAgent 的差别，也可以放进这条循环里看。Skill 不会自己执行代码，它只是改变下一轮模型能看到的指令。SubAgent 则更重：一次外层工具调用会启动另一个独立的 Agent Loop，外层只拿最终结果。

## Prompt Cache 是架构约束，不是性能小优化

如果把所有内容一股脑塞进 `system` 或工具描述，功能也许能跑，但缓存会很快变差。原因是 **Prompt Cache 依赖前缀稳定**。**越靠前的 token 越应该稳定，越动态的内容越应该靠后**。

可以把一次请求粗略拆成这样：

*图：稳定上下文靠前，动态信息靠后*

这解释了 Claude Code 里一些不直觉的选择：动态列表不一定塞进 `system`，而是作为额外 message 注入；工具 schema 尽量保持稳定；某些工具定义可以 defer-loading，先暴露搜索入口，命中后再加载完整定义。

这不是洁癖。一个可扩展 Agent 会不断遇到新文件、新 skill、新 MCP server、新 agent 配置和用户临时指令。只要其中一类内容频繁变化，放错位置就会拖累后面的缓存命中。

## Skill：不要把它想成插件，先把它看成延迟加载的提示词

**Skill 的核心价值不是“多一个工具”，而是把某类任务的做法沉淀成一个可按需加载的 prompt**。它通常有 `SKILL.md`，也可能带参考资料、示例和脚本。

```
my-skill/  
├── SKILL.md  
├── reference.md  
├── examples/  
└── scripts/
```

`SKILL.md` 本身不具备执行能力。它会告诉 Agent：遇到这类任务时应该怎么读文件、怎么调用 bash、怎么编辑代码、怎么验收。真正干活的仍然是 Harness 已经提供的工具。

更合理的 Skill 系统一般分两步：

*图：Skill 从短描述到完整指令的按需加载*

这样做有两个好处。第一，初始上下文只放很短的 **`name/description`**，不会把所有 skill 内容都塞进去。第二，skill 可以渐进加载，真正命中后再读 body，缓存压力小很多。

Slash Command 是另一个入口，但不是 Skill 的本质。Command 更像用户主动点名：“把这段 prompt 展开后送进 Agent Loop”。Skill 更重要的部分，是 Harness 能让模型先感知有哪些能力，再决定是否加载完整说明。

Command 和 Skill 的关系可以这么看：

| 机制 | 入口 | 放进上下文的内容 | 典型用途 |
| --- | --- | --- | --- |
| Slash Command | 用户显式输入 `/xxx` | 展开后的 command prompt | 快速触发固定流程 |
| Skill Meta | Harness 自动注入 | `name` 、`description` | 让模型知道可选能力 |
| Skill Tool | 模型按需调用 | 完整 `SKILL.md` 与资源 | 复杂任务的过程约束 |

Claude Code 将 Skill 注册进 CommandRegistry，是一个产品和工程上的复用选择。用户可以像调用命令一样主动触发 Skill；模型也可以在看到 skill 列表后，按需把完整指令加载进来。

## Attachment：把动态清单放到消息层，而不是污染稳定前缀

**Skill 列表、SubAgent 列表、运行时提醒**，这些内容有一个共同点：它们有用，但变化频率高。项目里新增一个 Skill、插件更新一个 Agent、会话中临时挂载一个能力，都会改变这部分内容。

把它们写死在 `system` 里会很省事，但会破坏稳定前缀。更稳的做法是把这类内容变成 attachment，作为额外 user message 注入，常见形式是包一层类似 `<system-reminder>` 的标签，让模型知道它是系统级提醒，但又不把它放进最前面的静态 system prompt。

```
<system-reminder>  
Available skills:  
- code-review: inspect diffs and report risks  
- explain-code: explain implementation and call chain  
</system-reminder>
```

这层设计解决的是“动态信息如何被模型感知”的问题。它不负责执行，也不保证模型一定调用，只是把运行时状态以较低缓存代价送进上下文。

可以把它理解成 Harness 的上下文公告栏：不稳定的信息贴在公告栏上，稳定规则留在前缀里。

## SubAgent：一次工具调用，换来一条隔离的执行链

*图：父 Agent 保持主线干净，子 Agent 消化高噪声探索过程*

SubAgent 很容易被说成“多个智能体协作”。这个说法没错，但会遮住实现重点。工程上更准确的说法是：父 Agent 调用一个 `Agent` 工具，Harness 为子任务创建新的 system、tools 和 messages，然后在这个新上下文里再跑一条 Agent Loop。

```
const AgentTool = {  
  name: "Agent",  
  description: "Run a subagent in an isolated context.",  
  
  async call(input: { subagent_type: string; prompt: string }) {  
    const agent = resolveAgent(input.subagent_type)  
  
    const childContext = createContext({  
      systemPrompt: agent.prompt,  
      tools: filterTools(agent.allowedTools),  
    })  
  
    const childMessages = [  
      { role: "user", content: input.prompt },  
    ]  
  
    return AgentLoop({  
      context: childContext,  
      messages: childMessages,  
    })  
  },  
}
```

这条链路和普通工具不同。普通工具返回的是文件内容、命令结果或 API 数据；**SubAgent 返回的是另一个 Agent Loop 压缩后的结论**。父 Agent 不需要消费子 Agent 的每一步探索，也不需要把子 Agent 查过的所有文件、失败尝试、临时推理都塞回主上下文。

*图：父 Agent 通过工具调用获得隔离子任务结果*

它解决的不是“让系统显得更智能”，而是**上下文隔离**。复杂任务里，探索过程经常很脏：搜索范围大、失败路径多、噪声高。把这部分放到子 Agent 里，父 Agent 只拿整理后的结果，主线会干净很多。

SubAgent 的感知层也有类似取舍。agent list 可以内联进 `Agent` 工具描述，也可以走 attachment。前者简单，后者更利于缓存，因为工具 schema 会更稳定，动态 agent 清单留在消息层。

| 方案 | 优点 | 代价 |
| --- | --- | --- |
| agent list 写进 tool description | 实现直接，模型看到工具时也看到 agent 类型 | agent 增删会改变工具 schema，影响前缀稳定性 |
| agent list 走 attachment | 工具定义稳定，动态清单靠后注入 | Harness 需要多维护一层上下文注入逻辑 |

如果系统规模很小，第一种方案够用。Agent 类型一多，或者用户、项目、插件都能贡献 Agent，第二种更抗变化。

## MCP、defer-loading 与 tool\_search：同一类问题的另一种解法

MCP 也可以放进同一套框架理解。它把外部系统能力变成工具，供模型在 Agent Loop 中调用。问题是工具一多，`tools` 字段会膨胀，工具描述本身也会吃掉上下文。

defer-loading 和 `tool_search` 的思路与 Skill 渐进加载很接近：先给模型一个轻量入口，让它搜索或选择需要的工具；命中后再加载完整定义。它牺牲了一点直接性，换来更小的初始上下文和更稳定的缓存前缀。

*图：大量工具先搜索再加载，减少初始上下文压力*

这和 Skill 系统里的 `name/description -> 完整 SKILL.md` 是同一个工程套路：**不要在一开始把所有可能性都展开**。Agent Harness 越大，这条原则越重要。

## 写 Harness 时真正要做的取舍

如果从实现角度落地，重点不是先定义一堆漂亮概念，而是把上下文分层做清楚。

| 设计问题 | 更稳的处理方式 |
| --- | --- |
| 全局规则放哪里 | 放 `system`，尽量稳定，少混运行时状态 |
| 工具 schema 怎么管理 | 工具名和参数保持稳定，大工具考虑延迟加载 |
| Skill 怎么暴露 | 初始只暴露 meta，命中后再读完整 body |
| 动态能力清单怎么注入 | 用 attachment 放进 messages，避免污染前缀 |
| 子任务怎么隔离 | 用 Agent tool 创建子上下文，父 Agent 只接收结果 |
| 探索噪声怎么处理 | 让 SubAgent 消化过程，回传压缩后的判断 |

Claude Code 这类产品真正值得学的地方，不是某个名词的包装，而是它对上下文经济性的敏感：**稳定内容放前面，动态内容放后面；短描述先出现，长内容按需加载；高噪声探索放进隔离 loop**，主上下文只保留有用结论。

这套设计不复杂，但很容易被做坏。最常见的错误是把所有东西都塞进系统提示词，短期看功能全了，长期看缓存差、上下文脏、工具列表膨胀、模型注意力被稀释。Agent Harness 的工程质量，往往就体现在这些“不显眼”的放置位置上。

## 推荐阅读

## [Claude Code 如何压缩上下文：Microcompact、Prompt Cache 与 cache\_edits 工程拆解](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxMDIzNA==&mid=2247483859&idx=1&sn=d39896a395a806bbf16e6618a6ace685&scene=21#wechat_redirect)

## [平台智能化到了分水岭：为什么配置代码化才是 AI Coding 的下一代接口](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxMDIzNA==&mid=2247483850&idx=1&sn=fa9979a17b66eeacd28e6020846b6eee&scene=21#wechat_redirect)

## [为什么 AI Coding 难进生产环境？深入了解 Everything-Claude-Code！](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxMDIzNA==&mid=2247483837&idx=1&sn=e072990724fc1f6460ad29a1c49633eb&scene=21#wechat_redirect)

## [Agent Harness Runtime 架构深度解析：工具循环、状态外置与长程任务调度](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxMDIzNA==&mid=2247483815&idx=1&sn=d29421c7e5af80a4debb7edaa59e5e42&scene=21#wechat_redirect)

## [OpenClaw Dreaming 记忆流水线底层架构：状态分层、证据留痕与检索回流](https://mp.weixin.qq.com/s?__biz=Mzg2OTcxMDIzNA==&mid=2247483808&idx=1&sn=a62ec1c4c1601af2508345ccc591d7be&scene=21#wechat_redirect)