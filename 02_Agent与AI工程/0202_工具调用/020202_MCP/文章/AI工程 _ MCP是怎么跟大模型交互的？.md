---
title: AI工程 | MCP是怎么跟大模型交互的？
author: 非写不可
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI5MjE2MDQ2Mg==&mid=2651366460&idx=1&sn=241f9724370c302250bfb033fd5e1e2e&chksm=f64f9c9e1fd15970f1bf34734b04a471bbc554ab56fd222a72da26f0beb4468a74f55b77d4a9&mpshare=1&scene=24&srcid=0910iWVLwH66fIm5np6wclvu&sharer_shareinfo=42abeed1aa7d3364f9cae11eb67e2533&sharer_shareinfo_first=42abeed1aa7d3364f9cae11eb67e2533#rd
---

## 导语

书接上文，[AI工程 | MCP不是一天冒出来的](https://mp.weixin.qq.com/s?__biz=MzI5MjE2MDQ2Mg==&mid=2651366455&idx=1&sn=3b76f3e16cf266969495e1ac14dbf74b&scene=21#wechat_redirect)提到了MCP是Function Call的递进。这种关系好比普通话和方言，在小地方你可以继续说“方言”，也就是说函数个数少的简单场景你用FC就足够了。但在大城市人多流动大的地方用“普通话”交流就更高效，好比需要连接多种MCP Server的场景，用MCP协议才能让MCP Host程序开发和管理更高效。

接下来几篇文章将围绕MCP架构落地时的4个关键问题展开聊聊，本篇着重讲第1点。

1. MCP是怎么跟大模型交互的？
2. Client和Server如何通信？
3. 如何开发MCP Server？
4. 如何开发MCP Client？

影响大模型的4个方式：预训练、后训练、精调、Prompt

不知道大家第一次听到MCP技术时有没有这样的疑问：MCP在各种协议调用组装各种工具的一顿操作后，它究竟是怎么和大模型本身交互的？要回答这个问题，我们先回到原点来看，能够影响大模型的方式无外乎4种：预训练、后训练、精调、Prompt（提示词）。

预训练和后训练一般是大模型厂商在发布一个模型前要做的事。像DeepSeek最近开源的V3.1就同时包含了Base模型和后训练模型两个版本。预训练是为了训练模型的基础而通用的能力比如理解和生成文字。后训练针对专业领域能力进行额外训练，比如DS V3.1就特别在工具使用与智能体等任务中额外进行了训练。

精调和提示词一般给大模型的上层用户使用。精调相比后训练更多在于针对的专业领域大小，从而有训练数据量和成本的区别。然而对于大多数应用场景来说使用提示词才是我们的日常。众所周知提示词能很大程度激发和规范大模型的生成能力，提示词水平跟大模型的生成答案质量成正相关，因此业界也专门把提示词当做一个工程领域，俗称PE(Prompt Engineering)。当然技术永远在追求进步，目前这个概念进一步扩大为上下文工程，即CE(Context Engineering)，这是最近非常火的一个领域，前者侧重“单轮提示优化”，后者更关注“多工具调用时的上下文连贯性管理”，后续单独开篇来讲讲。

MCP是怎么跟大模型交互的？

回到MCP怎么和大模型交互这个问题上来，答案是提示词。可以进一步细分为用户提示词和系统提示词两种方式。从调用大模型的API的入参字段来看，messages字段就是我们常说的提示词，也就是用户提示词。tools字段背后就是系统会组装调用工具的提示词，也就是系统提示词。

By 系统提示词：toos字段

前一篇说的为什么MCP和Function Call没有本质区别的原因就是两者都是通过tools字段把工具或函数信息传给大模型，但要需要注意的是，需要看下所调用的大模型是否具备function call能力、是否支持了openai接口规范，所幸的是目前主流的大模型厂商都支持了，潜含的意思是大模型在训练阶段就针对函数调用做过专门训练。

MCP Host程序运行时MCP Client会把MCP Server提供的tools信息塞到这个tools字段里（注：每个大模型能支持的tools个数一般在百级别，最新的DeepSeek V3.1 API里提示最多支持128个）。当你问问题的时候，问题会被塞到messages字段，和tools字段一起传给大模型，大模型厂商内部的程序会系统提示词。相应地大模型返回的消息里面有两个字段，一个是回答内容content，一个是大模型认为该选用哪些工具的tool\_calls数组信息。

下面是组装MCP tools并调用大模型的代码示例：（FC同样适用）

```
// 准备message字段，包含你提的问题msg := []*model.ChatCompletionMessage{    {        Role: model.ChatMessageRoleUser,        Content: &model.ChatCompletionMessageContent{            StringValue: &qry,        },    },}// 准备tools信息tools := make([]*model.Tool, 0)for _, v := range functions {    tools = append(tools, &model.Tool{        Type: model.ToolTypeFunction,        Function: &model.FunctionDefinition{            Name:        v.Name,            Description: v.Description,        },    })}// 准备调用API的字段req := model.CreateChatCompletionRequest{    Model:    MODELNAME,    Messages: msg,    Tools:    tools,    Thinking: &model.Thinking{        Type: model.ThinkingTypeAuto,    },}// 调用大模型APIresp, err := client.CreateChatCompletion(    ctx,    req,)
```

By 用户提示词：messages字段

对于不支持tools的大模型，那就只能靠用户提示词了。虽然提示词可以表达任何你想要的，但大模型能否正确地理解又是另一回事。因此通过这种方法的最大缺点就是理解质量可能不好，稳定性也欠缺些。

通过提示词把mcp的工具信息（也就是名称、参数、示例等）传给大模型可以有多种格式，比如xm、json、markdown，甚至就单纯的文本形式。有一点要注意的是，你需要考虑大模型的输出格式是什么，毕竟后续步骤你要根据输出内容提取调用工具的参数。如何写好这种提示词有一个技巧，可以参考一些开源工具类使用的提示词模板，比如Claude Desktop、Cursor。这就属于传统的如何优化提示词范畴了，模板大同小异，需要结合实际情况打磨优化。

举个例子：

```
# 任务背景你需要根据我的问题，判断是否需要调用以下MCP工具来辅助回答。如果需要，请严格按照指定格式生成工具调用指令；如果不需要，直接回答问题即可。# MCP可用工具列表（请仔细阅读工具信息）以下是当前可调用的MCP工具详情，包含工具名称、功能说明、必填/可选参数及参数格式要求：1. 工具名称：【工具1名称，如：get_weather_mcp】   - 功能：【工具1具体作用，如：查询指定城市的实时天气及未来24小时预报】   - 必填参数：     - city：城市名称（格式：中文全称，如“北京市”“上海市”）     - data_type：数据类型（格式：可选“realtime”（实时）/“forecast”（预报），二选一）   - 可选参数：     - unit：温度单位（格式：“celsius”（摄氏度，默认）/“fahrenheit”（华氏度））   - 调用示例：【工具1名称】{"city":"广州市","data_type":"forecast","unit":"celsius"}2. 工具名称：【工具2名称，如：generate_image_mcp】   - 功能：【工具2具体作用，如：根据文本描述生成高清图片】   - 必填参数：     - prompt：图片描述（格式：中文/英文，至少10字，包含风格、主体、场景，如“赛博朋克风格的未来城市夜景，有飞行汽车和全息广告牌”）     - size：图片尺寸（格式：可选“512x512”/“1024x1024”，二选一）   - 可选参数：     - style：细分风格（格式：如“oil_painting”（油画）/“anime”（动漫），不填则默认写实）   - 调用示例：【工具2名称】{"prompt":"古风山水图，江面有一叶扁舟，远处是雪山","size":"1024x1024","style":"ink_wash"}【注：根据实际MCP工具数量增减，每个工具必须包含“名称+功能+参数要求+调用示例”，参数需明确“必填/可选”和“格式约束”】# 工具调用规则1. 判断逻辑：先分析我的问题是否需要工具辅助（如“查天气”“生成图片”需调用，“解释概念”无需调用）。2. 格式要求：若调用工具，**仅返回工具调用指令**，无需额外文字；指令必须严格遵循“【工具名称】{“参数名1”:“参数值1”,“参数名2”:“参数值2”}”格式，参数值需符合工具的格式要求。3. 错误处理：若问题中缺少工具必填参数（如“查天气但没说城市”），请直接反问我获取关键信息（如“请告诉我你要查询哪个城市的天气？”），不生成无效调用指令。# 我的问题：【此处填写用户的实际问题，如：“帮我查一下深圳市明天的天气预报，用摄氏度显示”】
```

通过提示词方法要达到可落地的效果，往往需结合对大模型进行精调。特别是如果涉及特殊的提示词格式以及特殊的输出格式。顺便说一句智能体的规划步骤用到的提示词，你会怎么设计？其往往比组装工具列表有更复杂的诉求，一般都要精调才能达到可接受的理解效果。

简单对比下两种方式，优先使用tools方式，组装方式更简单更规范，理解效果更有保障。messages方式作为不支持tools情况下的补充。messages方式不受限于大模型是否支持tools，上层组装灵活，但同时会带来更长的上下文，降低大模型的理解效果和调用成本，解析大模型的输出内容时也会带来不确定性。

结语

从大模型角度来看，MCP并非新东西，是Function Call既有技术的改造和升华。它们的区别更多在上述MCP架构图的另外三个方面，详情请听下回分解。

**我是『****飞邪****』，感谢阅读，欢迎关注『****非写不可****』获取最新文章：**👇****👇****👇************

**往期节选：**

[AI工程 | MCP不是一天冒出来的](https://mp.weixin.qq.com/s?__biz=MzI5MjE2MDQ2Mg==&mid=2651366455&idx=1&sn=3b76f3e16cf266969495e1ac14dbf74b&scene=21#wechat_redirect)

[API系列文章第3篇：不得不提的OpenAPI规范](https://mp.weixin.qq.com/s?__biz=MzI5MjE2MDQ2Mg==&mid=2651366257&idx=1&sn=fd0878d5ce0c07a9494b4fe8e34b0bbd&scene=21#wechat_redirect)

[API系列第5篇：SDK设计心得](https://mp.weixin.qq.com/s?__biz=MzI5MjE2MDQ2Mg==&mid=2651366304&idx=1&sn=e974e78e5c0a6ae68cbf5b249b4a2f94&scene=21#wechat_redirect)