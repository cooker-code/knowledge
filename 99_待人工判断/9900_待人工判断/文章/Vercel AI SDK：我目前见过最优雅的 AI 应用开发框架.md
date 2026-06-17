---
title: Vercel AI SDK：我目前见过最优雅的 AI 应用开发框架
author: Koala電波
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkyMDQyMzkwMA==&mid=2247483688&idx=1&sn=473bd8518b817633571c548b2989753a&chksm=c03aabb3b47b993809f890e08c2643bc5b69543789cff1b81daeb4ded7126022c8eb257d2eaa&mpshare=1&scene=24&srcid=0413b8I8TVKQKq7TobxggUO5&sharer_shareinfo=34e946d1f105e7c4039d82a79c67e336&sharer_shareinfo_first=34e946d1f105e7c4039d82a79c67e336#rd
---

故事是这样的。

我最近在开发一个 AI Agent，说白了就是想让 AI 能够真正替我干活的智能体。结果一上来就给我干懵了。

你想让 AI 替你干活，你得先接入各种模型吧。OpenAI 一套、Anthropic 一套、Google 一套、国内还有一堆。models.dev 上面光我能用的模型可能就有上百个，每一个都有自己的一套接口。我一开始是自己写适配层，写了一个星期，心态差点崩了。

换模型等于重写，这种日子什么时候是个头？

然后我就开始琢磨，能不能有什么抽象层，让我写一次，随便换模型。

然后我就发现了 Vercel AI SDK。

说实话，第一反应是，这玩意不是给 Next.js 用的吗？但是仔细一看，发现它其实是纯 TypeScript 的 SDK，跟什么框架都没关系。而且背后站着 Vercel 这个爸爸，Claude、GPT、Gemini 这些主流模型都给你接好了。

## 它解决的是什么问题

我们先把这个问题讲清楚。

现在做 AI 应用，你基本上得跟各种模型提供商打交道。OpenAI 一套接口，Anthropic 一套接口，Google 又一套。代码写多了之后，你会发现一个问题，就是你的业务逻辑会跟这些模型的 API 搅在一起。哪天你想把 GPT-4 换成 Claude，光改代码可能就得改一整天。

AI SDK 就是来解决这个的。

它抽象了一层 provider，不管你用哪家模型，GPT 也好 Claude 也好 Gemini 也好，调用方式一模一样。你就记住四个核心 API：`generateText`、`streamText`、`generateObject`、`streamObject`，剩下的事情框架帮你搞定。

## 三种接入方式：官方、兼容、自定义

这里需要好好讲一下，因为这是 AI SDK 最核心的东西，但文档里讲得比较散。

AI SDK 的 provider 接入有三种姿势：

**第一种，官方 provider。** 就是 Vercel 官方维护的包，`@ai-sdk/openai`、`@ai-sdk/anthropic`、`@ai-sdk/google` 这些。大厂官方出，品质有保证，更新也及时。

**第二种，OpenAI 兼容接口。** 现在大多数模型提供商，包括国内的那些，基本都是 OpenAI 兼容的。换句话说，他们的 API 格式跟 OpenAI 几乎一模一样，区别只是地址和 key 不一样。AI SDK 专门有一个包 `@ai-sdk/openai-compatible` 来处理这个，你只需要配置 baseURL 和 apiKey，剩下的代码一模一样：

```
import { createOpenAICompatible } from &#x27;@ai-sdk/openai-compatible&#x27;  
  
const myProvider = createOpenAICompatible({  
  name: &#x27;my-provider&#x27;,  
  baseURL: &#x27;https://api.models.dev/v1/chat/completions&#x27;,  
  apiKey: process.env.MY_PROVIDER_API_KEY,  
})
```

**第三种，社区 provider。** 就是社区自发做的适配包。models.dev 上面那些模型，很多都有社区贡献的 provider。比如 MiniMax，有 `vercel-minimax-ai-provider` 这个包，直接 `npm install` 就能用，而且 MiniMax 比较特殊，它同时支持 Anthropic 格式和 OpenAI 格式两种接口。

## 核心 API 怎么用

先说最常用的 `generateText`。

```
import { generateText } from &#x27;ai&#x27;  
import { openai } from &#x27;@ai-sdk/openai&#x27;  
  
const result = await generateText({  
  model: openai(&#x27;gpt-4o&#x27;),  
  prompt: &#x27;用一句话解释量子计算&#x27;,  
})  
  
console.log(result.text)
```

就这几行代码，你就完成了一次 AI 调用。`result` 里面还有 `finishReason` 告诉你为什么结束，还有 `usage` 告诉你消耗了多少 token。

但是说实话，简单的文本生成其实用场不多。真正有意思的是工具调用。

## 工具调用：让 AI 替你执行动作

我跟你说，AI SDK 的工具调用是我见过最优雅的实现。

```
import { generateText, tool } from &#x27;ai&#x27;  
import { z } from &#x27;zod&#x27;  
  
const result = await generateText({  
  model: openai(&#x27;gpt-4o&#x27;),  
  tools: {  
    getWeather: tool({  
      description: &#x27;获取某个城市的天气&#x27;,  
      parameters: z.object({  
        location: z.string().describe(&#x27;城市名称&#x27;),  
      }),  
      execute: async ({ location }) => {  
        // 这里调用真实的天气 API  
        return { temperature: 22, condition: &#x27;晴天&#x27; }  
      },  
    }),  
  },  
  prompt: &#x27;北京今天天气怎么样？&#x27;,  
})
```

你定义一个工具，告诉 AI 这个工具是干什么用的，AI 自己决定什么时候调用它。你不需要写什么复杂的意图识别、函数分发，框架全给你安排好了。

而且工具可以串联。AI 调用完一个工具，返回结果可以继续传给下一个工具，形成一个多步骤的 agent。

## 流式响应：做聊天机器人的关键

静态生成其实应用场景有限。现在大家都在做聊天机器人，核心就是流式响应。`streamText` 就是干这个的。

```
const result = await streamText({  
  model: openai(&#x27;gpt-4o&#x27;),  
  prompt: &#x27;给我讲一个关于机器人的故事&#x27;,  
})  
  
for await (const chunk of result.textStream) {  
  process.stdout.write(chunk)  
}
```

这个 `for await` 会一块一块地拿到 AI 返回的内容，你可以实时展示给用户，体感完全不一样。

配合 Next.js 的 API route：

```
export async function POST(req: Request) {  
  const { messages } = await req.json()  
    
  const result = await streamText({  
    model: openai(&#x27;gpt-4o&#x27;),  
    messages,  
  })  
  
  return result.toDataStreamResponse()  
}
```

前端用一个 `useChat` hook 就能拿到完整聊天体验：

```
const { messages, input, handleInputChange, handleSubmit } = useChat({  
  api: &#x27;/api/chat&#x27;,  
})
```

说实话，这个 hook 封装得挺骚的。你基本上三五行代码就能撸出来一个能跑的聊天界面。

## 结构化输出：解决 AI 幻觉的利器

这是我觉得最有技术含量的部分。

有时候你不只是想让 AI 返回一段文字，你想要一个结构化的对象，比如 JSON。你可以让 AI 返回一个电影信息、用户资料、或者任何有固定格式的数据。

```
import { generateObject } from &#x27;ai&#x27;  
import { z } from &#x27;zod&#x27;  
  
const result = await generateObject({  
  model: openai(&#x27;gpt-4o&#x27;),  
  schema: z.object({  
    name: z.string().describe(&#x27;电影名称&#x27;),  
    year: z.number().describe(&#x27;上映年份&#x27;),  
    rating: z.number().describe(&#x27;评分&#x27;),  
    genres: z.array(z.string()).describe(&#x27;类型&#x27;),  
  }),  
  prompt: &#x27;给我推荐一部好看的科幻电影&#x27;,  
})  
  
const movie = result.object  
console.log(movie.name, movie.year)
```

配合 Zod 做 schema 定义，类型安全，而且 AI 会严格按照你定义的格式输出。

## Provider 切换：真的就是换个名字的事

这是最让我惊讶的地方。

假设你原来用的是 OpenAI，现在想换成 Claude。正常情况下你得改一堆代码。但是在 AI SDK 里面：

```
import { openai } from &#x27;@ai-sdk/openai&#x27;  
import { anthropic } from &#x27;@ai-sdk/anthropic&#x27;  
  
// 原来  
const result = await generateText({  
  model: openai(&#x27;gpt-4o&#x27;),  
  prompt: &#x27;你好&#x27;,  
})  
  
// 换成 Claude  
const result = await generateText({  
  model: anthropic(&#x27;claude-3-5-sonnet-20241022&#x27;),  
  prompt: &#x27;你好&#x27;,  
})
```

就改一个 model 定义，其他代码一模一样。换 provider 成本几乎为零。

## 具体怎么选：什么时候用什么方式

这个问题我一开始也挺懵的，后来想明白了。

**官方 provider**适合那些有官方维护包的大厂，OpenAI、Anthropic、Google 这些。你想用最新功能就选这个，更新最及时。

**OpenAI 兼容接口**适合那些提供 OpenAI 兼容 endpoint 的服务，比如 models.dev、硅基流动、国内各种模型服务平台。就是改个 baseURL 和 key，其他不动。

**社区 provider**适合那些有社区贡献包的小众供应商，比如 MiniMax。社区包的好处是针对那个供应商做了专门的适配，可能支持一些兼容接口没有的功能。比如 MiniMax 的社区包就同时支持 Anthropic 格式和 OpenAI 格式，而且帮你封装好了，你直接用就行：

```
import { minimax } from &#x27;vercel-minimax-ai-provider&#x27;  
import { generateText } from &#x27;ai&#x27;  
  
const result = await generateText({  
  model: minimax(&#x27;MiniMax-M2.7&#x27;),  
  prompt: &#x27;你好&#x27;,  
})
```

你说 MiniMax 这种，它其实同时支持 Anthropic 和 OpenAI 两种接口。你可以用社区包，也可以用 OpenAI 兼容接口，看你心情。但是社区包通常适配得更好一些，能用就用。

## React Server Components：真正的屠龙刀

最后说一个我最喜欢的功能，RSC 集成。

现在 Next.js 主推的玩法是 React Server Components，你的 AI 逻辑可以直接跑在服务端，然后流式返回 UI 组件。

```
import { createAI } from &#x27;@ai-sdk/rsc&#x27;  
import { streamUI } from &#x27;@ai-sdk/rsc&#x27;  
  
export const AI = createAI({  
  actions: {  
    continueStory: streamUI({  
      model: openai(&#x27;gpt-4o&#x27;),  
      initial: <div>生成中...</div>,  
      prompt: &#x27;继续这个故事：从前有一只机器人...&#x27;,  
      text: ({ content }) => <p>{content}</p>,  
    }),  
  },  
})
```

这是什么意思呢，就是你的 AI 不仅是返回文字，还能直接返回 React 组件。AI 说「给你显示一个天气卡片」，前端直接就渲染出来一个天气卡片。

不用你手动写什么「如果 AI 返回的是 weather，就渲染 WeatherCard 组件」，框架自动给你搞定。

## 怎么接入

安装其实很简单：

```
npm install ai @ai-sdk/openai
```

用哪个 provider 就装对应的包，`@ai-sdk/anthropic`、`@ai-sdk/google` 之类的，装完直接 import 就能用。

社区 provider 就 `npm install vercel-minimax-ai-provider` 这种包名。OpenAI 兼容的话就是 `@ai-sdk/openai-compatible`。

文档其实写得挺清楚的，ai-sdk.dev 上面有完整的 guide，我之前踩的一些坑文档里其实都提到了，只能说我自己没仔细看。

## 我的一点感受

用了一周下来，我最大的感受是，这玩意的抽象层次拿捏得是真准。不像有些框架，抽象过度反而限制了你发挥空间；也不像有些框架，抽象了个寂寞，该写的还是得写。

AI SDK 的边界感很好，你用简单的功能就是几行代码搞定，你想深入定制也有足够的空间。

而且 TypeScript 支持是真的完整。我之前用其他一些库，类型提示经常对不上，AI SDK 这边基本没遇到这个问题。配合 Zod 做 schema 验证，开发体验很顺。

接入 models.dev 上面那些模型的时候，我只需要写一个 provider 配置，剩下的调用逻辑完全不用动。之前我自己写的那个适配层，跟这一比简直是原始社会。

当然它也有不完美的地方。比如流式响应的 lifecycle callbacks 现在还是 experimental 状态，生产环境用可能得掂量一下。不过核心功能已经非常稳了，这些周边功能慢慢补上问题不大。

如果你正在做 AI 应用开发，或者准备做，我真心建议把这个库玩一下。不一定是最终的解决方案，但是用来快速原型、验证想法，效率提升是很明显的。

好了，今天就聊这么多。如果你觉得这篇文章对你有启发，随手点个赞、在看、转发三连吧。如果想第一时间收到推送，也可以给我个星标。

谢谢你看我的文章，我们，下次再见。