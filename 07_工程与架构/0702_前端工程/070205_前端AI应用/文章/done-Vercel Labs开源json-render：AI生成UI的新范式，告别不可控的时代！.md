> 已吸收至：[[07_工程与架构/0702_前端工程/070205_前端AI应用/070205_核心知识点/前端AI生成式UI与工具调用边界|前端AI生成式UI与工具调用边界]]、[[07_工程与架构/0702_前端工程/070205_前端AI应用/070205_知识地图|070205_前端AI应用知识地图]]

---
title: Vercel Labs开源json-render：AI生成UI的新范式，告别不可控的时代！
author: 光影织梦
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5OTE0OTM4NQ==&mid=2247485338&idx=1&sn=87b169a89afeab65a09c016034bad69b&chksm=9717cfa7210d660cde51ffb720afca25c1eaf01d0d263e0ef5e8576074da5efaf0b06b0fdae2&mpshare=1&scene=24&srcid=0206pTGKdNdkYsqw7zjvqaAY&sharer_shareinfo=dd93ad9875fe6c35dc385d101ae3f342&sharer_shareinfo_first=dd93ad9875fe6c35dc385d101ae3f342#rd
---

最近，Vercel Labs开源了一个名为**json-render**的项目，这个项目正在AI前端开发圈引起热议。它彻底解决了AI生成UI时长期困扰我们的两个问题：**输出不统一**和**难于管控**。

今天，我们就来深入了解这个如何将"生成式UI"变为标准生产流的革命性工具。

## 什么是json-render？

json-render是一个由Vercel Labs开源的AI-JSON-UI框架，它的核心理念很简单：**让AI在预设的"格子"里填色，而不是随心所欲地创作**。

json-render概念图

## 核心特点

### ◈🛡️ Guardrailed（有护栏的）

通过catalog（目录）定义允许出现的组件、动作、props形状，AI只能使用这些预定义的元素。如果AI试图"出格"，系统会立即报错。

```
const catalog = createCatalog({
components: {
    Card: {
      props: z.object({ title: z.string() }),
      hasChildren: true,
    },
    Metric: {
      props: z.object({
        label: z.string(),
        valuePath: z.string(),
        format: z.enum(['currency', 'percent', 'number']),
      }),
    },
  },
actions: {
    export_report: { description: 'Export dashboard to PDF' },
    refresh_data: { description: 'Refresh all metrics' },
  },
});
```

### ◈🎯 Predictable（可预测的）

由于有了Schema约束，AI输出的JSON永远符合预期格式，不再出现"惊喜"或"惊吓"。

### ◈⚡ Fast（快速的）

支持流式渲染，AI每吐一段文本流，前端就按完整JSON片段增量解析并渲染，用户几乎无等待感。

## 工作原理

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│ User Prompt │────▶│  AI + Catalog│────▶│  JSON Tree  │
│ "dashboard" │     │ (guardrailed)│     │(predictable)│
└─────────────┘     └──────────────┘     └─────────────┘
                                               │
                    ┌──────────────┐            │
                    │  Your React  │◀───────────┘
                    │  Components  │ (streamed)
                    └──────────────┘
```

1. **定义护栏**：明确AI可以使用哪些组件、动作和数据绑定
2. **用户提示**：用户用自然语言描述需求
3. **AI生成JSON**：输出始终可预测，受限于你的目录
4. **快速渲染**：在模型响应时进行流式渲染

## 实际应用场景

### ◈📊 数据看板

一句话生成完整的数据分析仪表盘：

* 实时业务指标展示
* 自动数据刷新
* 交互式图表

### ◈🛒 电商营销动态表单

根据营销活动自动生成表单：

* A/B测试友好的组件结构
* 用户行为追踪
* 动态字段验证

### ◈🖥️ 展会大屏

实时展会数据展示：

* 实时数据流更新
* 多维度数据展示
* 警报和通知系统

## 高级功能

### ◈条件可见性

根据数据、认证或复杂逻辑显示/隐藏组件：

```
{
  "type":"Alert",
"props":{"message":"Error occurred"},
"visible":{
    "and":[
      {"path":"/form/hasError"},
      {"not":{"path":"/form/errorDismissed"}}
    ]
}
}
```

### ◈丰富的动作处理

支持确认对话框和回调的动作：

```
{
  "type":"Button",
"props":{
    "label":"Refund Payment",
    "action":{
      "name":"refund",
      "params":{
        "paymentId":{"path":"/selected/id"},
        "amount":{"path":"/refund/amount"}
      },
      "confirm":{
        "title":"Confirm Refund",
        "message":"Refund ${/refund/amount} to customer?",
        "variant":"danger"
      }
    }
}
}
```

### ◈内置验证

支持各种数据验证规则：

```
{
  "type":"TextField",
"props":{
    "label":"Email",
    "valuePath":"/form/email",
    "checks":[
      {"fn":"required","message":"Email is required"},
      {"fn":"email","message":"Invalid email"}
    ],
    "validateOn":"blur"
}
}
```

## 技术架构

json-render采用模块化设计，包含两个核心包：

| 包名 | 描述 |
| --- | --- |
| `@json-render/core` | 类型、模式、可见性、动作、验证 |
| `@json-render/react` | React渲染器、提供者、钩子 |

## 项目结构

```
json-render/
├── packages/
│   ├── core/        → @json-render/core
│   └── react/       → @json-render/react
├── apps/
│   └── web/         → 文档和演示站点
└── examples/
    └── dashboard/   → 示例仪表盘应用
```

## 为什么选择json-render？

### ◈🔄 可审计性

每个UI元素都有明确的JSON定义，便于代码审查和质量控制。

### ◈🚦 可灰度

可以轻松实现A/B测试和渐进式功能发布。

### ◈🔧 可替换模型

由于输出格式标准化，可以轻松切换不同的AI模型而不影响前端。

### ◈📈 生产就绪

将实验性的"生成式UI"转变为可以用于生产环境的标准流程。

## 快速开始

```
npm install @json-render/core @json-render/react
```

创建一个简单的仪表盘生成器：

```
import { DataProvider, ActionProvider, Renderer, useUIStream } from '@json-render/react';

function Dashboard() {
  const { tree, send } = useUIStream({ api: '/api/generate' });

  return (
    <DataProvider initialData={{ revenue: 125000, growth: 0.15 }}>
      <ActionProvider actions={{
        export_report: () => downloadPDF(),
        refresh_data: () => refetch(),
      }}>
        <input
          placeholder="Create a revenue dashboard..."
          onKeyDown={(e) => e.key === 'Enter' && send(e.target.value)}
        />
        <Renderer tree={tree} components={registry} />
      </ActionProvider>
    </DataProvider>
  );
}
```

## 演示地址

想要亲身体验？项目提供了完整的演示环境：

```
cd json-render
pnpm install
pnpm dev
```

* http://localhost:3000 — 文档和演示
* http://localhost:3001 — 示例仪表盘

## 总结

json-render的价值不仅在于技术实现，更在于它为AI生成UI领域带来了**标准化**和**可控性**。它让我们看到了AI辅助开发的未来不再是"黑盒"，而是可以在生产环境中安全、可控使用的工具。

对于需要在生产环境中使用AI生成UI的团队来说，json-render无疑是一个值得深入研究和采用的解决方案。

## 获取方式

需要获取源码的朋友，请关注我的公众号"**光影织梦**"，发送"**json-render**"即可获取开源地址和详细使用指南。

---

*口令：json-render*