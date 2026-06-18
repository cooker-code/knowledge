> 已吸收至：[[02_Agent与AI工程/0210_sandbox/021001_AgentSandbox/021001_核心知识点/AgentSandbox隔离与审计边界|AgentSandbox隔离与审计边界]]、[[02_Agent与AI工程/0210_sandbox/021001_AgentSandbox/021001_核心知识点/Sandbox运行时实现与选型|Sandbox运行时实现与选型]]
---
title: OpenShell：安全沙箱隔离的沙箱隔离技术
author: 极客阁楼
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0MjU5MDc2OQ==&mid=2247485037&idx=1&sn=bdcd672747fa36637dd3ac1ecd253a10&chksm=c3854440dc52448ce99d456f5cb55fbd0a77aa63b6035abbbdcbea7dcf91791e0ce7fcce6ea8&mpshare=1&scene=24&srcid=0327ZIj1QDmTXrqr8AwUXEdV&sharer_shareinfo=b3c141926b044c83aed450eeb2b4ea86&sharer_shareinfo_first=b3c141926b044c83aed450eeb2b4ea86#rd
---

NVIDIA 刚刚为 AI 代理生态系统投下了一颗重磅炸弹！运行本地机器人是危险的。一个恶意提示可以彻底摧毁你的机器。OpenShell 来了！它是一个安全的运行时环境，将你的 AI 困在一个隔离的沙箱中。如果代理试图窃取你的私钥或运行未授权的命令，它会立即被阻止！包括一个终端 UI，可以实时监控你的代理的沙箱。 它能让你的代码自己进化。 这是一个自动化的代码优化工具。 感兴趣的话就试试吧。 它会让你的开发效率大幅提升。 这就是自动化的力量。 它能让你的代码自己进化。 这是一个自动化的代码优化工具。 感兴趣的话就试试吧。 它会让你的开发效率大幅提升。 这就是自动化的力量。 它能让你的代码自己进化。

为了帮你更全面地了解这个项目，我整理了这份PDF，从核心功能到实现细节，详细展示了它的设计思路和实用价值。

看完这些架构图，我们再从工程角度聊聊这个工具的实际价值。

说实话，这个 OpenShell 真的解决了大问题。现在运行本地AI代理太危险了，一个恶意提示就能搞垮你的机器，OpenShell 用沙箱把AI关起来，想偷私钥？想跑未授权命令？门都没有！。

尤其适合那种需要运行不受信任AI代码的场景，比如测试第三方agent、运行社区贡献的技能，或者只是想安心玩AI而不用担心安全问题。

这个工具特别适合 对该领域感兴趣的开发者和技术爱好者，能让你安心地探索AI的边界，而不用担心中毒或者数据泄露。

总的来说就是给AI套了个安全笼子，既能让它干活，又能防止它搞破坏，感兴趣的可以去看看，安全第一嘛。

用起来也简单，Docker 一跑，沙箱就建好了，默认只给最小权限，需要什么权限就用 YAML  pol

https://github.com/NVIDIA/OpenShell