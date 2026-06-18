---
title: 别再堆 Prompt 了：用 LangChain 1.0 搭建“深度思考 Agent”
author: AGI启程号
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk2NDQ2ODU0OQ==&mid=2247483956&idx=1&sn=c48b594912b96981560aaa3441f518c8&chksm=c5fce47214628e66c454c0c309d67a52708ef17b58c78c1be918de725ec347d2bc16ff9272ad&mpshare=1&scene=24&srcid=1120Ub4ddZG7MqVtfnRPZfYV&sharer_shareinfo=dd04768ca629a017fdde7d682e387e4a&sharer_shareinfo_first=dd04768ca629a017fdde7d682e387e4a#rd
---

**“** 从“会聊天”到“会思考”：我用 LangChain 1.0 + Qwen 做了一个旅游 Agent，一个真正能“思考”的旅游规划 Agent。

在这个过程中，我顺手把“深度思考（reasoning）”这一套，从 模型 → API → LangChain 1.0 → 中间件，走了一遍闭环，也踩了不少坑。

这篇文章就借这个项目，聊聊 Reasoning 到底怎么开、怎么控、怎么落地。

**”**

关注微信公众号及时获取更多技术资讯

往期内容链接: 

[LoRA 微调实战：手把手教你打造专属领域 AI（附代码)](https://mp.weixin.qq.com/s?__biz=Mzk2NDQ2ODU0OQ==&mid=2247483931&idx=1&sn=27c305782f066adf7cace9698776250d&scene=21#wechat_redirect)

[MCP应用开发实战：把大模型接到现实世界](https://mp.weixin.qq.com/s?__biz=Mzk2NDQ2ODU0OQ==&mid=2247483946&idx=1&sn=e1c0a93bd34e6fcbe16a9449b43102e6&scene=21#wechat_redirect)

......

本期代码运行实例详见github：https://github.com/SWUSTcyt/langchain-travel-agent.git

01 为什么要关心 LangChain 1.0？

在 0.x 时代，我们做 Agent 差不多是这么个流程：

* 选一个大模型；
* 绑几个工具（搜索、数据库、地图 API……）；
* 用 Agent 框架帮你决定“什么时候该用工具”。

能跑，但有几个老问题：

* **Prompt、裁剪、Guardrails 分散**：逻辑散落在各种链和回调里，难维护；
* 想要“先自评再重写”“先检索再回答”，得自己手搓一堆链；
* 工程化之后，逻辑越来越多，全是 if-else 拼命长。

LangChain 1.0 做了一件很关键的小事：

> 把这些“前后处理、反思、限流、审计”的能力，**系统化收拢到了 Middleware（中间件）** 里。

简单一句话：

> **Agent = 一条生产线，Middleware = 插在生产线各个路口的“小工位”。**

有了它，我们可以更优雅地回答这三个问题：

1. 这个模型**什么时候开始深度思考**？
2. 它的**思考预算**谁来控制？
3. 出来的结果**谁来质检和纠偏**？

02 模型层：先把“深度思考”开起来

在这个 Demo 里我用的是 **Qwen-Turbo**，它本身支持“深度思考 + 流式输出”。模型初始化放在 `agent.py` 中，核心配置大概长这样：

```
from langchain_community.chat_models.tongyi import ChatTongyifrom tools import tools  # MCP（高德）工具在 tools.py 中加载  
qwen = ChatTongyi(    dashscope_api_key=DASHSCOPE_API_KEY,    model="qwen-turbo-latest",    streaming=True,         # 流式输出    model_kwargs={        "enable_thinking": True,    # 开启深度思考        "incremental_output": True, # 思考 + 流式 必须一起开        "temperature": 0.7,        "top_p": 0.8,        "max_output_tokens": 1500,  # 模型级成本护栏    },)  
# 交给 LangChain 处理工具调用qwen = qwen.bind_tools(tools)
```

这一层你可以理解为：  
**“告诉模型：你可以多想一会儿，但最长只能说这么多。”**

剩下的就交给 LangChain 1.0 的 Middleware 去“管住它”。

03 把 Agent 想成一条生产线：Middleware 就是闸口

官方文档会画一张循环图：

> 模型调用 →（可能）工具调用 → 再次模型调用 → …… → 结束

LangChain 1.0 提供了几个关键钩子（hook）：

* `before_model`：模型调用**之前**，适合改 Prompt、抽取结构化信息、打系统提示；
* `wrap_model_call`：像给 `model.invoke()` 加一层代理，适合控参数、做“自评→重写”等多段推理；
* `wrap_tool_call`：在工具真正被调用前拦一下，适合修参数、加权限、做预算限制；
* `after_model`：模型调用**之后**，适合做结果质检、串城检查、命中关键词再触发纠偏；
* `after_agent`：整轮 Agent 结束后的收尾工作。

在我的旅游 Agent 里，我只重点做了三件事：

1. 在 **before\_model** 里：自动解析城市 + 注入强约束；
2. 在 **wrap\_model\_call** 里：控制 Reasoning 的“成本”；
3. 在 **wrap\_tool\_call + after\_model** 里：强制工具对齐城市，并在最后做一次“串城质检”。

下面就用**几个关键中间件**快速过一遍。

04 三段关键中间件：一张“深思链路骨架”

### 4.1 城市解析 & 系统提示：before\_model

### 

用户说的每一句话里，其实都隐藏着一些重要变量，比如“城市”。  
在旅游场景下，城市就是一切的“坐标系”。

所以我在 `middleware.py` 里写了一个 `pin_cities_and_adcodes`：

* 从最后一条用户消息里自动**抽取城市**；
* 把城市写进 `runtime.state["target_cities"]`，供后续统一使用；
* 重写 System Prompt：

+ 指定这个 Agent 是“旅游助手”；
+ 明确要求优先用 MCP（高德）工具；
+ 加上“禁止跨城串场”的强约束。

核心逻辑大致是这样的（只保留关键部分）：

```
from langchain.agents.middleware import before_modelfrom langchain_core.messages import SystemMessage, HumanMessagefrom .utils import extract_cities, msg_text  
@before_modeldef pin_cities_and_adcodes(state, runtime):    messages = state.get("messages", [])    human_msgs = [m for m in messages if isinstance(m, HumanMessage)]    query = msg_text(human_msgs[-1]) if human_msgs else ""  
    # 1) 抽城市    cities = extract_cities(query)    runtime.state["target_cities"] = set(cities)    runtime.state["target_cities_source_text"] = query  
    # 2) 拼城市相关的系统提示    if cities:        city_tips = (            f"\n【强制约束】本轮用户指定城市：{', '.join(cities)}。"            "仅在这些城市范围内规划；所有 MCP 工具调用必须携带匹配的 city/cityd。"        )    else:        city_tips = "\n【强制约束】请从用户问题中判断目标城市；禁止跨城或引入未出现的城市。"  
    sys_msg = SystemMessage(        content=(            "你是旅行规划助手。需要本地信息时优先使用 MCP(amap-maps) 的工具："            "searchPOI / getRoute / getWeather。风格：\"轻松不赶\"。"            + city_tips        )    )  
    # 3) 把 SystemMessage 放到最前面，替换旧的 system    new_messages = [sys_msg] + [        m for m in messages if not isinstance(m, SystemMessage)    ]    return {"messages": new_messages}
```

这一段做的是**语义层的预处理**：

> “先帮模型把城市这个‘关键条件’想清楚，再让它进入 reasoning 模式。”

---

### 

### 4.2 成本护栏：wrap\_model\_call

### 

第二个关键点，是控制“模型思考的成本”。

在项目里，我统一用一个 `cost_guard` 来兜底：

* 确保每次模型调用都有：

+ `max_output_tokens` 限制；
+ `enable_thinking=True`；
+ `incremental_output=True`；

* 顺手再做一次 SystemMessage 置顶，防止出现在非首位时被模型拒绝。

核心代码简化后大概是这样：

```
from langchain.agents.middleware import wrap_model_call  
@wrap_model_calldef cost_guard(req, handler):    # 确保 system 在第一位（略）    ...  
    # 强制统一模型参数（Reasoning 预算）    guarded = req.override(        model_settings={            "max_output_tokens": 1200,            "incremental_output": True,            "enable_thinking": True,        }    )    return handler(guarded)
```

这就相当于在 API 层把：

* reasoning 开关、
* 输出 token 上限，

都集中放到 **一个地方统一管理**。以后要调参，只改这个中间件就行。

---

### 4.3 工具护栏：预算 + 城市对齐 + 串城质检

### 

第三块，是整个项目比较“工程化”的部分：  
**我们希望：工具调用既要节制，又要绝对不“串城市”。**

我这里拆成三个小环节：

#### 4.3.1 工具预算（轮次）计数：`track_tool_budget`

* 使用 `wrap_tool_call`；
* 并行工具调用只算一轮；
* 把轮次写到 `runtime.state["tool_rounds"]` 里，超出上限就直接返回一个错误 ToolMessage，不再执行工具。

```
from langchain.agents.middleware import wrap_tool_callfrom langchain_core.messages import ToolMessage  
@wrap_tool_calldef track_tool_budget(req, handler):    MAX_TOOL_ROUNDS = 2    # 读取当前轮次，按工具调用批次递增（略细节）    ...    if current_round > MAX_TOOL_ROUNDS:        return ToolMessage(            tool_call_id=call_id or "call_budget_exceeded",            content='{"error":"预算已耗尽","message":"工具调用已达到上限，请直接给出最终答案。"}',            name="budget_guard",        )    return handler(req)
```

#### 4.3.2 城市对齐：`enforce_city_on_tools`

* 同样是 `wrap_tool_call`；
* 对所有地图/路线/天气类工具，把 `city` / `cityd` 强制改成我们先前解析出来的城市；
* 若参数中带有 `adcode`，也会清空或重置，避免“杭州 adcode 跑去成都”。

```
@wrap_tool_calldef enforce_city_on_tools(req, handler):    # 只拦截与地图/天气相关工具    if not tool_name_contains_map_or_weather(req):        return handler(req)  
    cities = set(req.runtime.state.get("target_cities") or [])    if not cities:        return handler(req)    pinned_city = next(iter(cities))  
    params = dict(getattr(req, "tool_input", {}) or {})    params["city"] = pinned_city    params["cityd"] = pinned_city    if "adcode" in params:        params["adcode"] = ""  
    return handler(req.override(tool_input=params))
```

#### 4.3.3 串城质检：`strip_unasked_cities`（after\_model）

最后一道小保险放在 `after_model`：

* 生成完最终回答后，再跑一遍城市抽取；
* 如果发现有不在目标城市集合里的地名，就在答案末尾加一段“自动修正说明”；
* 不再触发二次模型调用，避免递归。

这样，整个“城市约束链路”就闭合了：

> before\_model 抽城市 →  
> wrap\_tool\_call 强制对齐工具参数 →  
> after\_model 最终文本再做一次检查。

05 实际体验：一个“会思考”的旅游助手

从用户视角看，这个小项目有两层体验：

**1. 正常用户模式**

* 只看到一个“旅游助手”对话框；
* 输入「规划成都 2 天美食 + 人文轻松路线」这类需求；
* Agent 会先并行调用高德工具（POI + 路线 + 天气），然后给出一份比较完整的、带可验证信息的行程建议。

**2. 开发者模式**

* `🛰 解析城市：成都`
* `🔧 工具 amap-searchPOI | city=成都 …`
* `🤖 模型生成中…`  
  的形式，展示整条 Middleware 生产线上的关键事件；

* 勾选“开发者模式”，会在对话下面展开一块“过程与日志”；
* 其中的摘要日志会以类似：
* 如果需要，还可以看到完整的 JSON 事件，方便 Debug。

06 Reasoning 的三个层次：不只是“让模型多想一会儿”

用这个项目小结一下，我现在会把 Reasoning 分成三个层次来看：

1. **模型层 Reasoning**

* 选支持深度思考的模型（例如 Qwen 的 `enable_thinking`）；
* 在模型参数里打开“思考模式”，给出一个大致的 token 预算。

2. **API / Middleware 层 Reasoning**

* 用 `cost_guard` 统一控制输出长度、thinking 开关；
* 用 `before_model` 把城市、约束条件、行为准则提前抽出来；
* 用 `wrap_tool_call` 管住工具预算 & 参数合理性；
* 用 `after_model` 做结果质检、自动修正与提示。

3. **结构层 Reasoning（工程级）**

* 用中间件修复不规范的工具调用结构；
* 用日志 + 开发者模式，把“深思过程”暴露给人类可观察、可调优。

如果以后需要走到更重的业务，比如电价预测报告、合规文书生成、金融风控决策，可以再叠加 **LangGraph + interrupt/checkpoint** 做“长事务 + 人审”，但底层这条 Middleware 生产线不需要推翻重来。

07 结语：你可以直接复用哪些东西？

这篇文章本质上只是用一个旅游 Agent，把这几个思路串了一遍：

* **用中间件统一管理“深度思考”的入口与出口**；
* 把抽取变量（城市）、预算控制、工具参数修正这些“工程活”，从 Prompt 里解放出来；
* 用日志 + 开发者模式，把 Reasoning 的路径“翻译成人话”。

如果你也在做 Agent / RAG / 报告生成类应用，可以直接照抄下面这几个套路：

* 一个 `before_model`，负责：抽取关键变量 + 重写 System Prompt；
* 一个 `wrap_model_call`，负责：统一模型参数 & 成本护栏；
* 一到两个 `wrap_tool_call`，负责：工具预算、参数对齐、权限检查；
* 一个 `after_model`，负责：结果质检 + 轻量纠偏。

剩下的，就抽时间跑一跑仓库里的 Demo，把旅游场景换成你自己的业务即可。

**希望这篇文章能帮你从“会用模型”进一步走向“会设计一条能思考、可控的生产线”。**