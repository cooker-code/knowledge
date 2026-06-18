---
title: Oh-My-OpenCode-v3.2.0-Hephaestus智能体实战指南与深度剖析
author: 万智创界
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA4MTc1MjMxOA==&mid=2457539952&idx=1&sn=783ec6be34efce8890d7e50d6f7d1e6e&chksm=89ea8a547015e1629ffa785d978de5155b7a13933320cad721e5a192e277844ebd53ef7038e8&mpshare=1&scene=24&srcid=02065UdEbH9CkF37HnhM676m&sharer_shareinfo=c7a122c78af35e162e7f0b2ea0aa4ffc&sharer_shareinfo_first=c7a122c78af35e162e7f0b2ea0aa4ffc#rd
---

# 🔨 Hephaestus：奥林匹斯的工匠之

**目标导向 · 深度探索 · 精准实现 · 验证完成**

> 在动手前充分理解材料、环境和需求，然后像大师工匠一样精确地完成工作

---

**基本信息**

| 属性 | 说明 |
| --- | --- |
| **首次亮相** | Oh My OpenCode v3.2.0 (2026-02-01) |
| **推荐模型** | GPT 5.2 Codex Medium |
| **Category** | `deep` (深度工作) |
| **核心能力** | 自主研究 + 模式匹配 + 端到端实现 |
| **适用场景** | 复杂功能、跨模块重构、第三方集成 |
| **工作理念** | 探索先行（Explore Before Acting） |

---

## 📖 开场白

近期，**Oh My OpenCode** 在 v3.2.0 版本中迎来了一位强大的新成员——**Hephaestus（赫淮斯托斯）**智能体。

作为一个自主深度工作者（Autonomous Deep Worker），Hephaestus 的加入标志着 Oh My OpenCode 智能体编排系统的一次重大升级。与以往的任务执行型智能体不同，Hephaestus 采用"**探索先行**"的工作方式：在编写第一行代码之前，他会并行启动多个探索智能体，深入研究你的代码库、查询最佳实践、查找开源实现，只有在形成完整的上下文理解后，才开始精准地实现目标。

这种"工匠式"的工作哲学，使得 Hephaestus 特别擅长处理复杂功能的完整实现、跨模块重构、第三方服务集成等需要深度理解的场景。他生成的代码不是 AI 垃圾（AI slop），而是与你的项目风格完美融合的、经过验证的高质量代码。

本文档将全面介绍 Hephaestus 智能体的特色、使用场景、工作原理以及最佳实践，帮助你充分发挥这位"奥林匹斯工匠"的全部潜力。

---

## 🔨 身份由来

**Hephaestus（赫淮斯托斯）**是希腊神话中的锻造之神、火神、工匠之神，为奥林匹斯众神打造武器和工具的神圣铁匠。

**为什么叫"Legitimate"（合法的）？**

当 Anthropic 以违反服务条款为由阻止第三方 OAuth 访问后，社区开始开玩笑地谈论"legitimate usage"（合法使用）。Hephaestus 拥抱了这个讽刺——他是**以正确方式构建事物的工匠**。

---

## 核心特色

### 1️⃣ **目标导向，而非配方导向**

```
传统智能体：  
用户 → 给步骤 1,2,3 → 智能体执行 → 可能完成 ❌  
  
Hephaestus：  
用户 → 给目标 "实现用户认证" → Hephaestus 自己决定步骤 ✅
```

**核心区别：**

* ❌ 不要：给我一个详细的步骤清单
* ✅ 要：告诉我你想要达成什么目标

### 2️⃣ **行动前深度探索（Explore Before Acting）**

Hephaestus 在写**第一行代码之前**，会并行启动 2-5 个探索智能体：

```
任务：添加 JWT 认证  
    ↓  
并行探索（同时进行）：  
├── Explore Agent 1：搜索现有代码库的认证模式  
├── Librarian Agent 1：查询 JWT 最佳实践  
├── Librarian Agent 2：查找开源实现示例  
└── Explore Agent 2：检查项目的安全配置  
    ↓  
整合所有信息 → 形成完整上下文 → 开始编码
```

**为什么这样做？**

* 避免 AI 生成不符合项目风格的代码
* 学习现有模式，保持一致性
* 发现潜在问题（如安全配置）
* 减少返工

### 3️⃣ **端到端完成，有证据才停**

```
Hephaestus 工作流程：  
┌─────────────────────────────────────┐  
│ 1. 理解目标                          │  
│ 2. 并行探索                          │  
│ 3. 制定计划                          │  
│ 4. 执行实现                          │  
│ 5. 运行测试 ← 必须通过                │  
│ 6. 代码审查 ← 必须通过                │  
│ 7. 验证功能 ← 必须通过                │  
│ 8. 完成 ✅                           │  
└─────────────────────────────────────┘
```

**不是：** "我写完了代码，你自己测吧"**而是：** "功能已实现，测试通过，代码已审查，这是证据"

### 4️⃣ **模式匹配（Pattern Matching）**

Hephaestus 会搜索现有代码库，学习项目的编码风格：

```
// 如果项目使用这种风格：  
export const getUserById = async (id: string) => {  
  const result = await db.query('SELECT * FROM users WHERE id = $1', [id])  
  return result.rows[0]  
}  
  
// Hephaestus 会生成类似的代码，而不是：  
export function getUserById(id: string) {  // ❌ 不匹配  
  return await this.db.findOne({ id })      // ❌ 不匹配  
}
```

**效果：** 生成的代码看起来像人类写的，而不是 AI 垃圾代码（AI slop）

### 5️⃣ **合法精度（Legitimate Precision）**

像大师铁匠一样：

* **外科手术式**：只改需要改的地方
* **极简主义**：不添加不必要的代码
* **精确所需**：不多不少，恰好满足需求

---

## 典型使用场景

### ✅ 最佳场景

| 场景 | 为什么用 Hephaestus |
| --- | --- |
| **复杂功能实现** | 需要深度理解代码库，不能简单拼接 |
| **跨模块重构** | 需要理解多个模块的交互和模式 |
| **集成第三方服务** | 需要研究文档 + 查看类似实现 + 匹配项目风格 |
| **性能优化** | 需要深度分析现有代码，找出瓶颈 |
| **安全漏洞修复** | 需要理解安全上下文，引入最佳实践 |
| **架构级改动** | 需要验证多个模块，确保一致性 |

### ❌ 不适合场景

| 场景 | 为什么不用 Hephaestus | 替代方案 |
| --- | --- | --- |
| **简单 typo 修复** | 杀鸡用牛刀，浪费 token | `quick` category |
| **单一文件修改** | 不需要深度探索 | 直接编辑或 `unspecified-low` |
| **快速原型验证** | 太重了，探索时间长 | `quick` 或 `artistry` |
| **重复性任务** | 不需要重新探索 | 使用 `quick` + skill |

---

## 实战案例

### 案例 1：添加用户认证系统

**用户输入（错误示范）：**

```
"添加用户认证，使用 JWT，先做这个，再做那个，然后..."
```

❌ 这不是 Hephaestus 的正确用法

**用户输入（正确示范）：**

```
"为我们的 API 添加基于 JWT 的用户认证系统。  
当前项目使用 Express + TypeScript，  
需要保护 /api/* 路由，并在登录时颁发 token。"
```

✅ Hephaestus 会：

#### 第一步：并行探索（30-60 秒）

```
启动 4 个并行智能体：  
  
├── Explore：搜索现有认证相关代码  
│   └── 发现有 auth.ts，但只有基础中间件  
│  
├── Librarian：查询 JWT 最佳实践  
│   └── 找到 Auth0 官方指南和 RFC 7519  
│  
├── Librarian：查找 Express JWT 中间件示例  
│   └── 找到 express-jwt 和自定义中间件实现  
│  
└── Explore：检查项目的错误处理模式  
    └── 发现项目使用 { success, error } 响应格式
```

#### 第二步：制定计划

```
1. 安装必要的依赖（jsonwebtoken 等）  
2. 创建 auth 模块（匹配项目结构 src/auth/）  
3. 实现 JWT 签发函数（使用项目的 logger）  
4. 实现 JWT 验证中间件（使用项目的错误处理）  
5. 保护 /api/* 路由（使用项目的路由模式）  
6. 添加登录端点（使用项目的验证模式）  
7. 添加单元测试（使用 Vitest，与项目一致）
```

#### 第三步：执行实现

每个步骤都会参考探索阶段收集的信息：

```
// 例如，生成的代码会匹配项目风格：  
  
// src/auth/jwt.ts  
import { sign, verify } from 'jsonwebtoken'  
import { logger } from '@/utils/logger' // 使用项目的 logger  
import type { TokenPayload } from '@/types/auth'  
  
export const generateToken = (payload: TokenPayload): string => {  
  return sign(payload, process.env.JWT_SECRET!, {  
    expiresIn: '7d'  
  })  
}  
  
// src/middleware/auth.ts  
import type { Request, Response, NextFunction } from 'express'  
import { verifyToken } from '@/auth/jwt'  
import { errorResponse } from '@/utils/response' // 项目的错误处理  
  
export const requireAuth = async (  
  req: Request,  
  res: Response,  
  next: NextFunction  
) => {  
  try {  
    const token = req.headers.authorization?.split(' ')[1]  
  
    if (!token) {  
      return errorResponse(res, 'No token provided', 401)  
    }  
  
    const decoded = verifyToken(token)  
    req.user = decoded  
    next()  
  } catch (error) {  
    return errorResponse(res, 'Invalid token', 401)  
  }  
}
```

#### 第四步：验证完成

```
✅ 运行测试套件：vitest run —— 通过  
✅ 手动测试登录：POST /api/login —— 成功获取 token  
✅ 验证受保护路由：GET /api/users with token —— 成功  
✅ 验证未授权访问：GET /api/users without token —— 401 错误  
✅ 检查代码风格：prettier check —— 通过  
✅ 类型检查：tsc --noEmit —— 通过  
  
验证完成。功能已实现，所有测试通过。
```

### 案例 2：性能优化 - 数据库查询慢

**用户输入：**

```
"我们的 /api/products 接口响应时间超过 2 秒。  
请分析并优化性能。  
项目使用 PostgreSQL + Prisma ORM。"
```

#### Hephaestus 的执行过程：

**深度分析阶段：**

1. **搜索 /api/products 实现**

   ```
   // 发现问题：N+1 查询  
   app.get('/api/products', async (req, res) => {  
     const products = await prisma.product.findMany()  
     
     // ❌ 对每个产品单独查询评论  
     for (const product of products) {  
       product.reviews = await prisma.review.findMany({  
         where: { productId: product.id }  
       })  
     }  
     
     res.json(products)  
   })
   ```
2. **检查数据库索引**

   ```
   -- 发现缺少索引  
   SELECT * FROM products;  // 全表扫描  
   SELECT * FROM reviews WHERE product_id = ?;  // 没有 product_id 索引
   ```
3. **制定优化方案**

* 使用 Prisma 的 include 进行关联查询
* 添加数据库索引
* 添加查询缓存（如果项目已有 Redis）

**优化实现：**

```
// 优化后的代码  
app.get('/api/products', async (req, res) => {  
  const products = await prisma.product.findMany({  
    include: {  
      reviews: {  // ✅ 一次查询获取所有数据  
        select: {  
          id: true,  
          rating: true,  
          comment: true  
        }  
      }  
    }  
  })  
  
  res.json(products)  
})
```

**添加索引：**

```
-- 迁移文件  
CREATE INDEX idx_reviews_product_id ON reviews(product_id);
```

**验证优化效果：**

```
优化前：/api/products 响应时间 2100ms  
优化后：/api/products 响应时间 180ms  
性能提升：91.4% ✅  
  
✅ 功能测试：数据完整性保持  
✅ 压力测试：并发 100 请求，平均响应 200ms
```

### 案例 3：集成 Stripe 支付

**用户输入：**

```
"集成 Stripe 支付功能。  
项目使用 Next.js 14 App Router，  
需要支持订阅和一次性支付。"
```

#### Hephaestus 的执行：

**并行研究（60 秒）：**

```
├── Librarian：Stripe 官方文档  
│   └── Payment Intents、Subscriptions API、Webhooks  
│  
├── Librarian：Next.js Stripe 集成示例  
│   └── 找到 15 个生产级示例，分析最佳实践  
│  
├── Explore：项目的 API 结构  
│   └── 发现使用 /app/api/[route]/route.ts 模式  
│  
└── Explore：项目的环境变量管理  
    └── 发现使用 .env.local + @t3-oss/env-nextjs
```

**实现集成：**

1. **设置 Stripe Webhook**

   ```
   // app/api/stripe/webhook/route.ts  
   import { headers } from 'next/headers'  
   import { Stripe } from 'stripe'  
   import { buffer } from 'micro'  
     
   const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!)  
   const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!  
     
   export async function POST(req: Request) {  
     const body = await buffer(req)  
     const signature = headers().get('stripe-signature')!  
     
     const event = stripe.webhooks.constructEvent(  
       body,  
       signature,  
       webhookSecret  
     )  
     
     switch (event.type) {  
       case 'checkout.session.completed':  
         await handleCheckoutCompleted(event.data.object)  
         break  
       // ... 其他事件  
     }  
     
     return Response.json({ received: true })  
   }
   ```
2. **创建支付 API 路由**

   ```
   // app/api/payments/checkout/route.ts  
   import { stripe } from '@/lib/stripe'  
   import { requireAuth } from '@/middleware/auth'  
     
   export async function POST(req: Request) {  
     const user = await requireAuth(req)  
     
     const session = await stripe.checkout.sessions.create({  
       payment_method_types: ['card'],  
       line_items: [{ price: 'price_xxx', quantity: 1 }],  
       mode: 'payment',  
       success_url: `${origin}/success?session_id={CHECKOUT_SESSION_ID}`,  
       cancel_url: `${origin}/checkout`,  
       customer_email: user.email,  
     })  
     
     return Response.json({ url: session.url })  
   }
   ```
3. **实现前端支付组件（匹配项目 UI 风格）**

   ```
   // components/CheckoutButton.tsx  
   'use client'  
     
   import { Button } from '@/components/ui/button' // 项目的 Button 组件  
   import { useToast } from '@/hooks/use-toast' // 项目的 toast  
     
   export function CheckoutButton() {  
     const { toast } = useToast()  
     
     const handleCheckout = async () => {  
       const res = await fetch('/api/payments/checkout', {  
         method: 'POST',  
       })  
     
       const { url } = await res.json()  
       window.location.href = url  
     }  
     
     return <Button onClick={handleCheckout}>立即购买</Button>  
   }
   ```

**完整测试：**

```
✅ Stripe 测试模式支付流程  
✅ Webhook 端点验证（stripe webhook signing secret）  
✅ 一次性支付：$19.99 —— 成功  
✅ 订阅支付：$9.99/月 —— 成功  
✅ 支付失败场景：卡片拒绝 —— 正确处理  
✅ Webhook 事件：payment_intent.succeeded —— 数据库记录正确  
✅ 退款流程：测试退款 —— 成功
```

---

## 如何调用 Hephaestus

### 方式 1：通过 Category 系统自动调用

Oh My OpenCode 3.2+ 中，Hephaestus 是 `deep` category 的默认模型：

```
// ~/.config/opencode/oh-my-opencode.json  
{  
  "categories": {  
    "deep": {  
      "description": "目标导向的自主问题解决，在行动前进行彻底研究",  
      "agent": "hephaestus",  
      "model": "gpt-5-2-codex-medium",  
      "requiresModel": true  
    }  
  }  
}
```

当任务需要深度研究时，系统会自动使用 Hephaestus。

### 方式 2：手动指定

```
# 在 OpenCode 中使用斜杠命令  
/deep 实现用户认证系统  
  
# 或者直接指定智能体  
/hephaestus 为我们的 API 添加 JWT 认证  
  
# 使用 category 关键字  
/deep 集成 Stripe 支付功能
```

### 方式 3：通过 delegate\_task 显式调用

在配置中或直接使用时指定：

```
{  
  "task": "实现复杂的认证系统，包含 JWT、刷新令牌和中间件",  
  "category": "deep",  
  "agent": "hephaestus",  
  "load_skills": ["git-master", "systematic-debugging"]  
}
```

### 方式 4：在代码注释中触发

```
// TODO: [deep] 实现 OAuth 2.0 登录流程（Google、GitHub）
```

Hephaestus 会识别 `[deep]` 标签并接管任务。

---

## 与其他智能体的对比

### Oh My OpenCode 智能体矩阵

| 智能体 | 定位 | 工作方式 | 探索深度 | 执行代码 | 适用场景 |
| --- | --- | --- | --- | --- | --- |
| **Sisyphus** | 主编排器 | 委派 + 监督 | 中等（委托） | 是（委托） | 日常开发 |
| **Hephaestus** | 深度工作者 | 独立深度完成 | 极深（亲自） | 是（亲自） | 复杂/未知 |
| **Oracle** | 高智商顾问 | 只读分析 | 深度分析 | 否 | 架构、调试 |
| **Prometheus** | 战略规划师 | 访谈 + 计划 | 需求澄清 | 否 | 需求不明确 |
| **Atlas** | 总编排器 | 管理 + 验证 | 管理任务 | 否 | 执行工作计划 |
| **Metis** | 预规划顾问 | 隐藏需求分析 | 深度分析 | 否 | 发现隐藏意图 |
| **Momus** | 质量审查员 | 计划审查 | 审查计划 | 否 | 评估工作计划 |

### 选择决策树

```
是否有明确需求？  
├─ 否 → Prometheus（访谈澄清）  
└─ 是 →  
    是否是复杂/未知的领域？  
    ├─ 是 → Hephaestus（深度研究）  
    └─ 否 →  
        是否有详细工作计划？  
        ├─ 是 → Atlas（执行计划）  
        └─ 否 → Sisyphus（日常开发）
```

### 具体对比

#### Hephaestus vs Sisyphus

| 方面 | Sisyphus | Hephaestus |
| --- | --- | --- |
| **探索方式** | 委托给 Explore/Librarian | 亲自并行探索 |
| **代码执行** | 主要委托给子智能体 | 亲自编写 |
| **适用场景** | 日常开发、快速迭代 | 复杂功能、未知领域 |
| **速度** | 快（并行委派） | 慢（深度探索） |
| **质量** | 高（团队协作） | 极高（工匠精神） |
| **Token 消耗** | 中等 | 较高 |

**选择建议：**

* 🎯 日常 bug 修复、功能添加 → **Sisyphus**
* 🔨 复杂系统集成、性能优化、架构重构 → **Hephaestus**

#### Hephaestus vs Oracle

| 方面 | Oracle | Hephaestus |
| --- | --- | --- |
| **定位** | 顾问（只读） | 工匠（执行） |
| **工作模式** | 分析、建议、指导 | 研究、实现、验证 |
| **输出** | 分析报告、建议 | 完整的、经过验证的代码 |
| **何时使用** | 架构设计、代码审查、疑难调试 | 功能实现、系统集成 |

**协作示例：**

```
1. Oracle 分析架构问题 → 给出建议  
2. Hephaestus 根据 Oracle 的建议实现解决方案  
3. Oracle 审查 Hephaestus 的实现 → 验证质量
```

---

## Hephaestus 工作原理

### 技术架构

```
┌────────────────────────────────────────────────────────┐  
│                  Hephaestus Agent                      │  
│               (GPT 5.2 Codex Medium)                   │  
└────────────────────────────────────────────────────────┘  
                         │  
         ┌───────────────┼───────────────┐  
         │               │               │  
    ┌────▼────┐    ┌────▼────┐    ┌────▼────┐  
    │接收目标  │    │并行探索  │    │计划执行  │  
    └─────────┘    └─────────┘    └─────────┘  
                       │  
         ┌─────────────┼─────────────┐  
         │             │             │  
    ┌────▼────┐  ┌────▼────┐  ┌────▼────┐  
    │Explore  │  │Librarian│  │Explore  │  
    │Agent 1  │  │Agent 1  │  │Agent 2  │  
    │(代码库) │  │(文档)   │  │(模式)   │  
    └─────────┘  └─────────┘  └─────────┘  
         │             │             │  
         └─────────────┼─────────────┘  
                       │  
              整合信息 → 形成上下文  
                       │  
              制定计划 → 执行实现  
                       │  
              运行测试 → 验证完成
```

### Prompt 策略

Hephaestus 的提示经过专门优化：

```
# Hephaestus: The Legitimate Craftsman  
  
你是 Hephaestus，奥林匹斯的工匠之神。  
  
## 核心原则  
  
1. **目标导向**：用户给你目标，你自己决定步骤  
2. **探索先行**：编码前并行探索，形成完整上下文  
3. **模式匹配**：学习现有代码风格，保持一致性  
4. **极简主义**：只做必要的事，不过度设计  
5. **证据导向**：完成必须有验证证据  
  
## 工作流程  
  
### 阶段 1：并行探索（必须）  
启动 2-5 个探索智能体，同时：  
- 搜索现有代码库的相关实现  
- 查询官方文档和最佳实践  
- 查找开源实现示例  
- 分析项目的编码模式和约定  
  
### 阶段 2：制定计划  
基于探索结果，制定实现计划：  
- 匹配项目结构  
- 使用项目的工具链  
- 遵循项目的编码规范  
  
### 阶段 3：执行实现  
- 参考探索阶段的所有发现  
- 确保代码风格一致  
- 使用项目的错误处理模式  
  
### 阶段 4：验证完成  
- 运行测试（必须通过）  
- 手动验证功能  
- 检查代码质量  
- 提供验证证据  
  
## 禁止事项  
  
- ❌ 不要跳过探索阶段  
- ❌ 不要忽略现有代码模式  
- ❌ 不要添加不必要的代码  
- ❌ 不要在未验证的情况下声称完成  
- ❌ 不要使用项目未使用的库  
  
## 代码风格  
  
匹配现有代码：  
- 导入顺序和方式  
- 命名约定  
- 错误处理模式  
- 日志记录方式  
- 测试框架和风格  
  
记住：你的代码应该与人类编写的无法区分。
```

---

## 注意事项

### ⚠️ 使用前必读

#### 1. Token 消耗较高

Hephaestus 在探索阶段会并行启动多个子智能体：

```
典型场景：添加 JWT 认证  
├── Hephaestus 主智能体：~5,000 tokens  
├── Explore Agent：~2,000 tokens  
├── Librarian Agent 1：~3,000 tokens  
├── Librarian Agent 2：~3,000 tokens  
└── Explore Agent 2：~2,000 tokens  
总计：~15,000 tokens（仅探索阶段）
```

**成本估算（GPT-5.2 Codex Medium）：**

* 输入：0.015
* 输出：0.02
* **单次任务成本：约 0.05**

**建议：** Hephaestus 适合重要任务，不适合简单修改。

#### 2. 需要明确的目标

**好的目标：**

```
"为我们的 Express API 添加基于 JWT 的认证系统。  
项目使用 TypeScript，当前有 /api/users 路由。  
需要保护所有 /api/* 路由，登录端点是 /api/login。"
```

**不好的目标：**

```
"修改 auth.ts 文件"（太模糊）  
"添加 JWT 功能"（缺少上下文）  
"实现认证，第一步导入 jsonwebtoken..."（不要给步骤）
```

#### 3. 首次运行较慢

**时间分解：**

* 并行探索：30-60 秒
* 制定计划：20-30 秒
* 执行实现：2-5 分钟（取决于复杂度）
* 验证测试：1-2 分钟
* **总计：约 3-8 分钟**

**vs Sisyphus：**

* Sisyphus：1-3 分钟（快速委派）
* Hephaestus：3-8 分钟（深度探索）

**但：** Hephaestus 的一次成功率更高，减少返工时间。

#### 4. 需要模型支持

**推荐模型：**

* ✅ GPT-5.2 Codex Medium（最佳）
* ✅ GPT-5.2 Codex High（更智能，更贵）
* ⚠️ Claude Opus 4.5（可以，但不如 Codex）
* ❌ Gemini 3 Pro（不推荐，缺少代码优化能力）

**模型效果对比：**

| 模型 | 代码质量 | 探索深度 | 速度 | 成本 |
| --- | --- | --- | --- | --- |
| GPT-5.2 Codex Medium | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 中等 | 中等 |
| GPT-5.2 Codex High | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 快 | 高 |
| Claude Opus 4.5 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 中等 | 高 |
| Gemini 3 Pro | ⭐⭐⭐ | ⭐⭐⭐ | 快 | 低 |

### 💡 最佳实践

#### 1. 提供足够的上下文

```
✅ 好的提示模板：  
  
"项目背景：  
- 技术栈：[列出主要技术]  
- 当前问题：[描述要解决的问题]  
- 目标：[明确的目标]  
  
项目约定：  
- 代码风格：[描述编码风格]  
- 测试框架：[使用的测试框架]  
- 其他约束：[数据库、API 版本等]  
  
期望输出：  
- [描述预期的交付物]  
"
```

#### 2. 信任 Hephaestus 的判断

```
❌ 不要这样做：  
"实现 JWT 认证，具体步骤：  
1. 导入 jsonwebtoken  
2. 创建 sign 函数  
3. 创建 verify 函数  
..."  
  
✅ 应该这样做：  
"实现 JWT 认证系统，包含登录、令牌验证和路由保护。  
项目使用 Express + TypeScript。"
```

Hephaestus 会自己决定如何实现。

#### 3. 准备接受"慢即是快"

**场景对比：**

```
使用 Sisyphus（快）：  
├── 第 1 次尝试：3 分钟，但测试失败  
├── 第 2 次修复：2 分钟，还有 bug  
├── 第 3 次修复：2 分钟，通过  
└── 总计：7 分钟 + 3 次迭代  
  
使用 Hephaestus（慢）：  
├── 并行探索：1 分钟  
├── 深度分析：1 分钟  
├── 实现：3 分钟  
├── 测试：1 分钟  
└── 总计：6 分钟 + 1 次成功 ✅
```

**慢即是快。**

#### 4. 利用验证证据

Hephaestus 完成后会提供详细的验证报告：

```
## 验证报告  
  
### 功能测试  
✅ 登录成功并返回 token  
✅ 受保护路由验证 token  
✅ 无效 token 返回 401  
✅ Token 过期正确处理  
  
### 代码质量  
✅ Prettier 检查通过  
✅ ESLint 检查通过  
✅ TypeScript 类型检查通过  
✅ 所有测试通过（15/15）  
  
### 性能  
✅ 登录端点响应时间 < 200ms  
✅ 验证中间件响应时间 < 50ms  
  
### 安全  
✅ 使用环境变量存储密钥  
✅ Token 过期时间合理（7 天）  
✅ 实现了 CSRF 保护
```

**仔细阅读验证报告**，确保符合预期。

---

## 配置示例

### 完整配置

```
// ~/.config/opencode/oh-my-opencode.json  
{  
  "agents": {  
    "hephaestus": {  
      "model": "gpt-5-2-codex-medium",  
      "temperature": 0.3,  
      "description": "自主深度工作者 - 在行动前进行彻底研究",  
      "systemPromptAppend": "\n\n特别注意：我们的项目使用严格的类型检查，不要使用 any。"  
    }  
  },  
  "categories": {  
    "deep": {  
      "agent": "hephaestus",  
      "model": "gpt-5-2-codex-medium",  
      "description": "目标导向的自主问题解决，在行动前进行彻底研究",  
      "requiresModel": true  
    }  
  },  
  "backgroundTask": {  
    "maxConcurrentTasks": 5,  
    "providers": {  
      "openai": 3,  
      "anthropic": 2  
    }  
  },  
  "disabledHooks": [  
    "comment-checker"  
  ]  
}
```

### 项目级配置覆盖

```
// .opencode/oh-my-opencode.json  
{  
  "agents": {  
    "hephaestus": {  
      "model": "gpt-5-2-codex-high", // 这个项目用更智能的模型  
      "systemPromptAppend": "\n\n这个项目对性能要求极高，优化所有查询。"  
    }  
  }  
}
```

---

## 常见问题（FAQ）

### Q1: Hephaestus 和 Sisyphus 的区别是什么？

**A:**

| 特性 | Sisyphus | Hephaestus |
| --- | --- | --- |
| **角色** | 主编排器 | 深度工作者 |
| **探索** | 委托给子智能体 | 亲自并行探索 |
| **执行** | 主要委托 | 亲自编写 |
| **适用** | 日常开发 | 复杂/未知任务 |
| **类比** | 项目经理 | 大师工匠 |

**简单记忆：**

* 日常任务 → Sisyphus
* 复杂任务 → Hephaestus

### Q2: 什么时候应该用 Hephaestus？

**A:** 满足以下任一条件时：

* ✅ 你之前没做过类似的事情
* ✅ 需要深度理解现有代码库
* ✅ 任务涉及多个模块/系统
* ✅ 需要集成不熟悉的第三方服务
* ✅ 性能优化或安全相关
* ✅ "如果做错了代价很大"

### Q3: Hephaestus 太慢了怎么办？

**A:** 首先确认是否真的需要 Hephaestus：

```
任务是否复杂？  
├─ 否 → 使用 Sisyphus 或 quick category  
└─ 是 →  
    是否可以接受 5-10 分钟的探索时间？  
    ├─ 是 → 使用 Hephaestus（一次成功）  
    └─ 否 → 使用 Sisyphus（快速迭代）
```

**记住：** Hephaestus 的"慢"是投资，不是浪费。

### Q4: Hephaestus 可以用更便宜的模型吗？

**A:** 技术上可以，但不推荐。

```
推荐配置：  
GPT-5.2 Codex Medium → 最佳平衡 ⭐⭐⭐⭐⭐  
  
可用配置：  
GPT-5.2 Codex High → 更智能，更贵  
Claude Opus 4.5 → 可以，但代码优化不如 Codex  
  
不推荐：  
Gemini 3 Pro → 缺少代码优化能力  
任何免费模型 → 无法支持并行探索
```

### Q5: Hephaestus 失败了怎么办？

**A:** Hephaestus 有自我修复机制：

```
Hephaestus 执行  
    ↓  
测试失败？  
    ├─ 否 → 完成 ✅  
    └─ 是 →  
        分析失败原因  
            ↓  
        修复问题  
            ↓  
        重新测试  
            ↓  
        重复直到通过
```

如果 3 次尝试后仍失败：

* 会自动调用 Oracle 进行诊断
* 提供详细失败报告
* 建议替代方案

### Q6: 可以同时运行多个 Hephaestus 吗？

**A:** 技术上可以，但不建议：

```
配置：maxConcurrentTasks: 5  
  
问题：  
- 每个 Hephaestus 会启动 2-5 个子智能体  
- 并行 2 个 Hephaestus = 4-10 个子智能体  
- Token 消耗爆炸  
- 可能达到提供商速率限制
```

**建议：** 一次只运行一个 Hephaestus。

### Q7: Hephaestus 的探索阶段可以跳过吗？

**A:** 不可以，这是核心特性。

```
Hephaestus 的价值链：  
  
探索阶段（30-60 秒）  
    ↓  
学习项目模式  
    ↓  
生成一致的高质量代码  
    ↓  
一次测试通过  
    ↓  
节省返工时间  
  
跳过探索 = 跳过价值
```

如果你不需要探索，应该用 Sisyphus。

---

## 进阶技巧

### 1. 结合 Skills 使用

Hephaestus 可以结合特定技能：

```
{  
  "task": "实现完整的 OAuth 2.0 登录系统",  
  "category": "deep",  
  "agent": "hephaestus",  
  "load_skills": [  
    "systematic-debugging",  // 系统化调试  
    "test-driven-development", // TDD  
    "git-master"              // Git 原子提交  
  ]  
}
```

### 2. 事后 Oracle 审查

Hephaestus 完成后，可以让 Oracle 审查：

```
# Hephaestus 完成  
/deep 实现 JWT 认证系统  
  
# 让 Oracle 审查实现  
/oracle 审查刚才实现的 JWT 认证系统，检查：  
1. 安全性  
2. 性能  
3. 代码质量
```

### 3. 结合 Prometheus 计划

对于超大型任务：

```
1. Prometheus 制定详细计划  
    ↓  
2. /start-work 激活 Atlas  
    ↓  
3. Atlas 为子任务分配智能体  
    ↓  
4. 复杂子任务 → Hephaestus  
5. 简单子任务 → Sisyphus-Junior
```

### 4. 自定义 System Prompt

可以在配置中追加自定义提示：

```
{  
  "agents": {  
    "hephaestus": {  
      "systemPromptAppend": "\n\n# 项目特殊要求\n\n1. 所有数据库查询必须使用 prepared statements\n2. 所有 API 端点必须有速率限制\n3. 所有用户输入必须验证和清理\n4. 使用项目的 logger，不要用 console.log\n5. 所有错误必须使用项目的 errorResponse 函数"  
    }  
  }  
}
```

---

## 总结

### Hephaestus 的核心价值

```
目标 → 深度探索 → 理解上下文 → 精确实现 → 验证完成
```

**不是工具的集合，而是工匠精神的体现。**

### 何时使用 Hephaestus

✅ **用 Hephaestus：**

* 复杂功能的完整实现
* 需要深度理解代码库
* 跨模块重构和优化
* 集成复杂第三方服务
* 任何"之前没人做过"的事

❌ **不用 Hephaestus：**

* 简单 bug 修复
* 单一文件修改
* 快速原型验证
* 重复性任务

### 核心理念

> **"像大师铁匠一样，在动手前充分理解材料、环境和需求，然后精确地完成工作。"**

记住：**给 Hephaestus 一个目标，他会给你一个完整的、经过验证的解决方案。** 🔨

---

*万智创界更新于：2026年2月4日*