---
title: AI生成UI的工程化实践：json-render概念、与A2UI对比及基于Qwen的实现
author: 暮北林
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI0MzY0NTQ4Nw==&mid=2247484610&idx=1&sn=ddbca5631c727542eb7ff8a574980b4d&chksm=e80273b0795272be3f7d430c6d885079d1e5ba7a193ac69b72b24ac8b40ff76b428f29440396&mpshare=1&scene=24&srcid=0416zHhIF0KeRvxZ6MlN4kbz&sharer_shareinfo=ae21c70deb17257bc7c50dc6e2cfadac&sharer_shareinfo_first=ae21c70deb17257bc7c50dc6e2cfadac#rd
---

## 引言

让大模型直接生成HTML、JSX或CSS一直是AI生成UI领域最直观的思路。然而，这种“裸写代码”的方式在实际工程中几乎寸步难行：模型输出的代码结构不稳定，组件边界无法保证，生成的代码经常无法编译或违背设计系统约束，更不用说权限控制和审计追踪了。

Vercel开源的**json-render**正是为解决这一痛点而设计的。它不要求AI写前端代码，而是引入一个中间层——**JSON UI AST**，将AI的输出严格约束在开发者预定义的组件词汇表中，从而实现UI生成的可预测性与安全性。自2026年1月开源以来，json-render已在GitHub上获得超过13000颗星标，并提供了React、Vue、Svelte、Solid、React Native等9个渲染目标。

本文将从三个方面深入剖析json-render：首先介绍其核心概念与使用方法；然后对比Google同期推出的A2UI协议，分析两者的设计差异；最后结合一个**基于Express和通义千问（Qwen）的真实Node.js示例**，展示如何在实际项目中轻松落地——前端只需几行代码就能完成流式UI渲染。

---

## 一、json-render的核心概念与使用方法

json-render并非一个传统的UI框架，而是一个**DSL（领域特定语言）执行系统**。它由三个核心层次构成：

* **Catalog**：定义了AI可以使用的全部组件、属性以及可触发的action，是AI的能力边界。
* **JSON UI Tree**：AI输出的唯一形式，是一个扁平化的、符合Catalog约束的JSON对象。
* **Renderer**：由开发者实现，负责将JSON UI Tree解释并渲染为真实的前端组件。

整个工作流程可以概括为：**AI → JSON → UI**。

### 1.1 Catalog：为AI划清边界

Catalog是整个系统的基石。开发者使用Zod（或TypeScript类型）定义每个组件的props schema，并可以附加描述信息帮助模型理解组件的用途。

typescript

```
import { defineCatalog } from '@json-render/core';  
import { z } from 'zod';  
  
export const catalog = defineCatalog(schema, {  
  components: {  
    Metric: {  
      props: z.object({  
        label: z.string(),  
        valuePath: z.string(),  
        format: z.enum(['currency', 'percent', 'number']),  
      }),  
      description: "用于展示关键业务指标，如营收、转化率等",  
    },  
    Card: {  
      props: z.object({ title: z.string(), padding: z.enum(['sm','md','lg']) }),  
      hasChildren: true,  
    },  
  },  
  actions: {  
    export_report: { description: "将当前仪表盘导出为PDF" },  
  },  
});
```

Catalog相当于一门语言的语法定义文件——AI后续生成的所有JSON都必须严格符合这套grammar。

### 1.2 JSON UI Tree：扁平化的UI描述

与传统的嵌套JSON不同，json-render采用**扁平化结构**：所有组件节点平铺在`elements`对象中，通过`children`数组（存key）表示父子关系。这种设计让AI生成时无需维护复杂的嵌套层级，出错率更低；前端渲染时也能通过key直接定位节点，渲染和更新效率更高。

json

```
{  
  "root": "dashboard",  
  "elements": {  
    "dashboard": {  
      "type": "Card",  
      "props": { "title": "Revenue Dashboard", "padding": "md" },  
      "children": ["revenue"]  
    },  
    "revenue": {  
      "type": "Metric",  
      "props": {  
        "label": "Total Revenue",  
        "valuePath": "/metrics/revenue",  
        "format": "currency"  
      }  
    }  
  }  
}
```

### 1.3 流式渲染：边生成边显示

json-render内置了`SpecStreamCompiler`，可以将模型输出的文本流实时编译为JSON补丁，实现UI的增量更新。这极大减少了用户的等待时间。

typescript

```
import { createSpecStreamCompiler } from '@json-render/core';  
  
const compiler = createSpecStreamCompiler<MySpec>();  
let buffer = '';  
  
// 每次接收到LLM的chunk时调用  
const { result, newPatches } = compiler.push(chunk);  
if (result) {  
  setSpec(result);  // 触发UI渲染  
}
```

### 1.4 前端渲染器与Hook

json-render为React、Vue等主流框架提供了开箱即用的渲染器和React Hook。特别是`useUIStream`这个Hook，封装了流式请求、JSON解析、UI树增量更新的全部逻辑，开发者只需提供API端点，即可获得实时渲染的UI树。

tsx

```
import { Renderer, useUIStream, ActionProvider, DataProvider, VisibilityProvider } from "@json-render/react";  
  
function Dashboard() {  
  const {  
    tree: apiTree,    // 自动更新的UI树  
    isStreaming,      // 是否正在生成中  
    send,             // 发送提示词触发生成  
    clear,            // 清空当前UI树  
  } = useUIStream({  
    api: "/api/ai/generate-ui",  
    onError: (err) => console.error("Generation error:", err),  
  });  
  
  return (  
    <div>  
      <button onClick={() => send("创建一个销售仪表盘，包含总营收、订单量和转化率三个指标")}>  
        生成仪表盘  
      </button>  
      {apiTree && <Renderer spec={apiTree} />}  
    </div>  
  );  
}
```

配合`ActionProvider`、`DataProvider`和`VisibilityProvider`，可以轻松实现权限控制、数据绑定和条件渲染。

---

## 二、json-render与Google A2UI的区别

2025年底，Google发布了**A2UI（Agent-to-User Interface）** 协议，目标同样是让AI Agent生成安全、可交互的UI。两者在高层设计上相似，但在定位、适用场景和生态绑定上有显著差异。

### 2.1 相同之处

* **核心管道一致**：AI → 受限的JSON → 客户端原生渲染。
* **安全性优先**：通过预定义组件目录杜绝代码注入风险。
* **组件化思想**：都要求AI在固定的组件词汇表内工作，而非自由生成HTML/JSX。

### 2.2 核心差异

| 维度 | json-render | Google A2UI |
| --- | --- | --- |
| **定位** | 与特定应用程序组件集紧密耦合的**工具** | 跨代理互操作的**协议** |
| **适用场景** | 基于React/Vue的SaaS仪表盘、内部工具 | 跨Web、Android、Flutter的多端统一渲染 |
| **组件标准** | 由开发者自定义，灵活但无跨应用互通性 | 定义了一套通用基础组件集（Card、Form、Chart等） |
| **渲染目标** | 支持React、Vue、Svelte、Solid、React Native、PDF、邮件等9+种 | 专注于Web和移动端原生渲染 |
| **生态成熟度** | 2026年初开源，已有36个预置shadcn/ui组件 | 2025年底发布，协议仍在演进中 |

### 2.3 如何选择？

* **选择json-render**：如果你正在构建一个特定的Web应用（如管理后台、数据分析平台），希望快速让AI生成UI，并且你的组件库已经成型，那么json-render是最直接的解决方案。
* **选择A2UI**：如果你的系统需要接收来自多个不同AI Agent的UI描述，并且需要在一套多端（Web、iOS、Android）应用中统一渲染，那么A2UI的标准化协议更有优势。

值得一提的是，json-render社区正在开发A2UI适配器，未来两者可以互操作。

---

## 三、使用Qwen实现：基于Express的实战示例

很多开发者在实际项目中已经尝试让大模型生成UI。下面是一个**基于Express和通义千问（Qwen）的真实Node.js示例**，它完美配合json-render的`useUIStream`——后端只需流式输出符合规范的JSON，前端用几行Hook代码即可完成全部渲染逻辑。

### 3.1 完整的Express服务代码

javascript

```
import express from 'express';  
import cors from 'cors';  
import { createDeepSeek } from '@ai-sdk/deepseek';  
import { streamText } from 'ai';  
  
const app = express();  
  
app.use(cors());  
app.use(express.json());  
  
app.post('/api/ai/generate-ui', async (req, res) => {  
    let { prompt, context, currentTree } = req.body;  
      
    // 构建系统提示词：要求模型输出标准UITree  
    const systemPrompt = `You are a dashboard UI generator.   
Output a JSON object with the following shape:  
{  
  "root": "componentKey",  
  "elements": {  
    "componentKey": { "type": "ComponentName", "props": {...}, "children": ["childKey1", ...] }  
  }  
}  
Available components: ${context.catalog.join(", ")}  
Example for a revenue dashboard:  
{  
  "root": "main-card",  
  "elements": {  
    "main-card": { "type": "Card", "props": {"title": "Revenue Dashboard"}, "children": ["revenue-metric"] },  
    "revenue-metric": { "type": "Metric", "props": {"label": "Total Revenue", "valuePath": "/analytics/revenue", "format": "currency"} }  
  }  
}  
Only output valid JSON, no extra text.`;  
  
    const promptInput = `${prompt}\n\nAVAILABLE DATA:\n${JSON.stringify(context.data)}`;  
      
    // 这里以DeepSeek为例，实际可替换为Qwen（见下文）  
    const deepseek = createDeepSeek({  
        apiKey: process.env.DEEPSEEK_API_KEY,  
        baseURL: 'https://api.deepseek.com',  
    });  
      
    const result = streamText({  
        model: deepseek('deepseek-reasoner'),  
        system: systemPrompt,  
        prompt: promptInput,  
    });  
      
    result.pipeTextStreamToResponse(res);  
});  
  
app.listen(8089, () => console.log('Server running on port 8089'));
```

### 3.2 替换为通义千问（Qwen）

若想使用Qwen，只需替换模型调用部分（其他代码不变）：

javascript

```
import { createQwen } from '@ai-sdk/qwen';  
  
const qwen = createQwen({ apiKey: process.env.QWEN_API_KEY });  
const result = streamText({  
    model: qwen('qwen-plus'),  
    system: systemPrompt,  
    prompt: promptInput,  
    experimental_responseFormat: { type: 'json_object' }, // 强制JSON输出  
});  
result.pipeTextStreamToResponse(res);
```

### 3.3 前端：使用`useUIStream`完成全部集成

前端不再需要手动处理`fetch`、流读取、JSON解析和patch应用。json-render的`useUIStream` Hook将所有复杂性封装在内：

tsx

```
import { Renderer, useUIStream, ActionProvider, DataProvider, VisibilityProvider } from "@json-render/react";  
import { catalog } from "./catalog";  // 与后端共享的组件定义  
  
function DashboardApp() {  
  const {  
    tree: apiTree,  
    isStreaming,  
    send,  
    clear,  
  } = useUIStream({  
    api: "/api/ai/generate-ui",  
    onError: (err) => console.error("Generation error:", err),  
  });  
  
  return (  
    <ActionProvider>      {/* 可选：处理组件触发的action */}  
      <DataProvider>      {/* 可选：提供数据绑定上下文 */}  
        <VisibilityProvider> {/* 可选：控制组件显隐 */}  
          <div>  
            <button   
              onClick={() => send("创建一个销售仪表盘，包含总营收、订单量和转化率三个指标")}  
              disabled={isStreaming}  
            >  
              {isStreaming ? "生成中..." : "生成仪表盘"}  
            </button>  
            <button onClick={clear}>清空</button>  
            {apiTree && <Renderer spec={apiTree} />}  
          </div>  
        </VisibilityProvider>  
      </DataProvider>  
    </ActionProvider>  
  );  
}
```

**这段代码已经完整实现了**：

* 发送用户提示词到后端
* 自动处理流式响应，实时更新UI树
* 渲染最终UI（使用你之前定义的组件映射）

你无需编写任何手动流读取、JSON解析或patch应用的逻辑。整个前端集成代码不超过30行。

### 3.4 后端输出格式说明

上述示例中，后端输出的是**完整的UITree JSON对象**（而非多行JSONL patch）。`useUIStream`内部使用`SpecStreamCompiler`自动处理流式JSON的增量解析，所以后端只需持续输出JSON片段即可，无需关心分块边界。

例如，后端可以分多次发送：

text

```
{"root": "main
```

text

```
-card", "elements": {"main-card": {"type": "Card"...
```

`useUIStream`会将这些碎片自动拼接、校验，并在每得到一个完整的UITree时触发UI更新。

---

## 四、总结与展望

json-render为AI生成UI提供了一条工程化的路径：它不试图让AI成为前端专家，而是让AI在开发者划定的“安全区”内高效工作。通过Catalog、UITree和流式编译器的组合，json-render在保证安全性和可控性的同时，极大降低了将大模型集成到UI生成场景的门槛。

与Google A2UI相比，json-render更贴近现有前端生态，特别适合那些已经拥有成熟组件库的团队快速落地。而A2UI的跨Agent、跨平台愿景，则代表了更长远的标准方向。

通过本文的实战部分，我们展示了一个**基于Express和通义千问（Qwen）的完整后端实现**，以及**前端仅需几行Hook代码**的极致简洁集成方式。`useUIStream`封装了所有复杂的流式处理，让开发者可以专注于业务逻辑和UI设计。

如果你正在考虑为你的产品增加“一句话生成界面”的能力，不妨从json-render开始——它不会让你陷入手动处理流、解析JSON的细节，而是让你用最熟悉的React组件，优雅地迎接AI时代