> 已吸收至：[[07_工程与架构/0702_前端工程/070202_React/070202_核心知识点/React架构与质量边界准则|React架构与质量边界准则]]、[[07_工程与架构/0702_前端工程/070202_React/070202_知识地图|070202_React知识地图]]

---
title: 告别 Next.js？我在 2026 年的 React 新栈：TanStack Start + shadcn v2 + Better Auth
author: DevOps教练
date:
url: https://mp.weixin.qq.com/s?__biz=MzIyMjQ2Mjc1NQ==&mid=2247489569&idx=2&sn=1eebb0bad7a12094dbf40aa5966575ef&chksm=e97d03130c6b40d09f146ca67b5f03c8d4bcf881ae2e7c007a0021afd62cb7d54334860cfc7f&mpshare=1&scene=24&srcid=0122bG6cVScfF1dGay3z89ga&sharer_shareinfo=42ebab6d94f358073e995cb8014ff391&sharer_shareinfo_first=42ebab6d94f358073e995cb8014ff391#rd
---

多年来，我一直是一名 Next.js 开发者。它曾是我的首选框架、我的舒适区，也是我“随手启动一个新 Next 应用”的解决方案。但最近，我总感觉有些不对劲。App Router 的复杂性、持续的破坏性变更、RSC 问题，以及那个以 Vercel 为中心的生态系统，都让我感到困扰。

上图是 Shadcn Lyra 主题的登录组件

于是，我决定尝试一些新东西。坦白说，我一点也不后悔。

改变一切的技术栈

以下是我现在使用的技术栈：

* TanStack Start (React 19 + Vite 7)

* shadcn/ui v2 (Base UI + Tailwind 4)

* Better Auth (TypeScript 优先的身份验证)

让我来告诉你为什么它们中的每一个都堪称颠覆性的存在。

TanStack Start：一个无人提及的 Next.js 替代方案

你一定知道 Tanner Linsley 吧？就是那个开发了 React Query、React Table 和 React Router 的大神？没错，他构建了一个全栈 React 框架。而且，这个框架*非常棒*。

TanStack Start 为你提供了：

* React 19 及其所有新特性 (==服务器组件 (Server Components)==、操作 (Actions) 等等)

* Vite 7 作为打包工具 (告别 webpack 的噩梦)

* 基于文件的路由，设计得非常合理

* 完整的 SSR/SSG 支持，无需费脑筋

开发者体验 (DX) 令人难以置信。热重载是即时的，构建速度飞快。最重要的是——它不会试图将你锁定在任何特定的部署平台上。

```
// It's just... React. Clean, simple React.import { createFileRoute } from "@tanstack/react-router"
```

```
export const Route = createFileRoute("/")({  component: HomePage,})function HomePage() {  return <div>Hello, freedom!</div>}
```

再也不用到处写 `"use client"` 指令了，也不用再纠结你的组件是服务器端还是客户端。它就是纯粹的 React。

Shadcn V2：颠覆性的改变

好了，让我们来谈谈今天真正的主角。

如果你用过 shadcn/ui，你就会知道它有多棒。你可以直接复制粘贴组件，对自己的代码拥有完全所有权，而且底层使用了 Rad

v2 版本的主题定制是这样的：

```
:root {  --primary: oklch(0.55 0.22 25);  --primary-foreground: oklch(0.98 0.01 25);}
```

```
.dark {  --primary: oklch(0.65 0.20 25);}
```

无需 JavaScript，无需主题提供器 (theme providers)。只需随处可用的 CSS 变量。

组件质量

每个组件都经过了重新设计，对细节的关注令人惊叹：

* 恰当的焦点管理 (focus management)

* 真正可用的键盘导航

* 感觉如原生般的动画效果

* 不再像是事后才添加的暗黑模式 (Dark mode)

我花了一个小时把玩下拉菜单。嵌套子菜单、复选框项目、单选按钮组——一切都太*惊艳*了。

Better Auth：不再糟心的身份验证

我试过所有方案：NextAuth (现在叫 Auth.js)、Clerk、Supabase Auth、Firebase。它们都有各自的取舍。

然后我发现了 Better Auth。

我想为这个备受开发者热议的新兴身份验证库点个赞。

这是一个 TypeScript 优先的身份验证库，它：

* 适用于任何框架（不仅仅是 Next.js！）

* 开箱即用地支持 OAuth 提供商

* 拥有简洁明了的 API

* 将 session 存储在你自己的数据库中

配置 Google OAuth 的过程简单得令人耳目一新：

```
import { betterAuth } from "better-auth"import { prismaAdapter } from "better-auth/adapters/prisma"
```

```
export const auth = betterAuth({  database: prismaAdapter(prisma, { provider: "postgresql" }),  socialProviders: {    google: {      clientId: process.env.GOOGLE_CLIENT_ID!,      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,    },  },})
```

在客户端：

```
import { signIn, useSession } from "@/lib/auth-client"
```

```
// Sign in with Googleawait signIn.social({ provider: "google" })// Get current sessionconst { data: session } = useSession()
```

就这样。无需用 provider 包装你的应用，没有复杂的配置。它就是能用。

最棒的是什么？首次使用的用户会自动注册，无需为 OAuth 单独设置注册流程。

完整蓝图

下面是我的项目结构：

```
src/├── components/│   ├── ui/           # shadcn components│   ├── mode-toggle.tsx│   └── user-menu.tsx├── lib/│   ├── auth.ts       # Better Auth server│   ├── auth-client.ts│   └── utils.ts├── routes/│   ├── api/auth/$.ts # Auth API handler│   ├── __root.tsx│   ├── index.tsx│   └── login.tsx└── styles.css        # Tailwind + theme
```

所有东西都位于同一处 (colocated)。所有东西都有类型定义。所有东西都很快。

你应该切换吗？

听着，我不是说 Next.js 不好。它仍然是一个拥有庞大生态系统的优秀框架。

但如果你正为以下问题而感到痛苦：

* 持续不断的破坏性变更 (breaking changes)

* 令人困惑的服务器组件 (Server Component) 边界

* 对 Vercel 平台锁定的担忧

* 缓慢的开发构建速度

……那么，也许是时候探索其他选择了。

TanStack Start 已经为生产环境准备就绪。shadcn v2 是一次巨大的升级。Better Auth 提供了我们应得的身份验证开发体验 (DX)。

React 生态系统远比 Next.js 广阔。在即将到来的 2026 年，我们拥有了更多选择。

往期推荐

PREVIOUS RECOMMENDATIONS

01

[弃用了 Jira 和 Slack后，三分之二的工程师表示：公司的文化变了，感觉之前领导层都没辙！](https://mp.weixin.qq.com/s?__biz=MzIyMjQ2Mjc1NQ==&mid=2247489228&idx=1&sn=2f003192bf191a0cb1046139517ed204&scene=21#wechat_redirect)

02

[我认识的最聪明的人，都痴迷于一种许多人曾以为“没用”的能力](https://mp.weixin.qq.com/s?__biz=MzIyMjQ2Mjc1NQ==&mid=2247488899&idx=1&sn=f726c5185d96f1bcfbce4be06712a21f&scene=21#wechat_redirect)

03

微[我为我的初创公司选择了 Golang——我一生中最大的错误](https://mp.weixin.qq.com/s?__biz=MzIyMjQ2Mjc1NQ==&mid=2247488845&idx=1&sn=8f1fab142bc0d8375381c042d7a8a0d9&scene=21#wechat_redirect)

04

[面试官的“阴招”：为什么 Java 里 1==1 为 true，但 1000==1000 却为 false？](https://mp.weixin.qq.com/s?__biz=MzIyMjQ2Mjc1NQ==&mid=2247489413&idx=2&sn=1508f6b99a8e10b151dbfe2353fb19b0&scene=21#wechat_redirect)

05

[Go 1.22 偷走了 Rust 的“秘密武器”：Arena 内存管理来了，性能直追却不用借用检查器](https://mp.weixin.qq.com/s?__biz=MzIyMjQ2Mjc1NQ==&mid=2247489378&idx=1&sn=565958806ab6ff02ffbdc24f237c71f8&scene=21#wechat_redirect)

06

[敏捷的终结：为何科技巨头正悄然放弃 Scrum](https://mp.weixin.qq.com/s?__biz=MzIyMjQ2Mjc1NQ==&mid=2247488991&idx=1&sn=6c87457adf8eaf3583cec5eca5b75475&scene=21#wechat_redirect)

07

[2025 正在悄然取代旧势力的 5 门语言：Zig、Elixir、Julia、Bun、Mojo 全曝光](https://mp.weixin.qq.com/s?__biz=MzIyMjQ2Mjc1NQ==&mid=2247489370&idx=2&sn=224788b3f323ddcfd716aab02a690b4f&scene=21#wechat_redirect)

08

[10 种常见软件架构模式一览](https://mp.weixin.qq.com/s?__biz=MzIyMjQ2Mjc1NQ==&mid=2247488945&idx=2&sn=1254ef271f2f583f9ffc4226f469877c&scene=21#wechat_redirect)