> 已吸收至：[[02_Agent与AI工程/0210_sandbox/021001_AgentSandbox/021001_核心知识点/AgentSandbox隔离与审计边界|AgentSandbox隔离与审计边界]]、[[02_Agent与AI工程/0210_sandbox/021001_AgentSandbox/021001_核心知识点/Sandbox运行时实现与选型|Sandbox运行时实现与选型]]
---
title: 腾讯开源 CubeSandbox：给 AI Agent 一颗 60ms 冷启动的独立内核
author: Git Trend
date: wsleepybearwsleepybear
url: https://mp.weixin.qq.com/s?__biz=MzkwMzY1Mjg5MQ==&mid=2247486692&idx=1&sn=a0d568dd5249837e1c4c7def0f6c0623&chksm=c19b8f084b344a65953b6bb6b0acb8c9dc170e1bdcec78c8ebb18a238be8b3764ff86d03fe02&mpshare=1&scene=24&srcid=04280we0IwSU5x5xdd0EMWxR&sharer_shareinfo=b742ffe3f1d00226e2f2f7b05a8046f2&sharer_shareinfo_first=b742ffe3f1d00226e2f2f7b05a8046f2#rd
---

> **项目卡片**
>
> * **项目**：CubeSandbox[1]
> * **状态**：v28.0.0 / 3800+ Stars / 2026-04-10 发布，两周增长迅猛
> * **一句话判断**：如果你在做 AI Agent 的代码执行基础设施，这是目前开源方案里兼顾安全隔离与启动速度的最优解。

AI Agent 要执行代码，第一个问题是：放在哪里跑？

Docker 容器启动快，但共享宿主机内核——容器逃逸的阴影一直没散。传统虚拟机隔离彻底，但秒级启动扛不住 Agent 高并发创建沙箱的需求。E2B 这类商业服务解决了问题，但闭源、按量计费、数据不出你的控制。

腾讯云开源的 CubeSandbox 选了第三条路：**基于 KVM 的 MicroVM 沙箱**，每个 Agent 拿到独立 Guest OS 内核，同时做到 60ms 冷启动、单实例内存 <5MB。这个项目 4 月 10 日才开源，半个月已经 3800+ Stars——增速说明这个痛点确实普遍存在。

技术选型：为什么是 KVM 而不是容器

CubeSandbox 的核心判断：AI Agent 执行的代码本质上不可信，Namespace + cgroup 这套容器隔离对 LLM 生成的任意代码来说不够安全。

但它没有因此接受传统 VM 的代价。两个工程取舍把这个矛盾解开了：

**快照克隆取代完整引导。** 预先创建好资源池，沙箱通过内存快照克隆创建，跳过内核引导、init 系统启动等全部耗时步骤。仓库在裸金属上的实测数据：单并发 60ms，50 并发场景下平均 67ms（P95 90ms，P99 137ms）。这个数据跟我预期的一致——瓶颈不在虚拟化开销，而在资源池的预分配策略。

**CoW + 极致裁剪压低内存。** 基于 Copy-on-Write 实现内存页共享，CubeSandbox 用 Rust 重写了 Guest 内部的 agent 守护进程（前身是 Kata Containers 的 agent），对运行时做了激进裁剪。≤32GB 规格下自身额外内存开销 <5MB，单机部署密度可以到数千实例。换句话讲，成本结构跟容器方案差不多，但隔离级别高了一个量级。

架构拆解

CubeSandbox 的架构分三层：API 层、调度层、虚拟化层。

**API 层**是 CubeAPI，Rust + Axum 实现，只做一件事：兼容 E2B SDK 协议。从 E2B 迁移过来，把 `E2B_API_URL` 指向 CubeSandbox 就行，业务代码一行不动。这是我看过最实际的迁移路径——不是"提供兼容层"，而是直接实现 E2B 协议。

**调度层**包含 CubeMaster（Go，集群编排）和 Cubelet（Go，节点调度）。CubeMaster 接收 API 请求，按资源策略分发到对应 Cubelet；Cubelet 管理本机所有沙箱的完整生命周期。CubeProxy 做反向代理，把 `<port>-<sandbox_id>.<domain>` 格式的请求路由到目标实例。

**虚拟化层**最重，也最有技术含量。CubeHypervisor 是 Cloud Hypervisor v28 的定制 fork（Rust），负责管理 KVM MicroVM；CubeShim 实现了 containerd Shim v2 接口，沙箱可以直接接入 Kubernetes 等容器编排平台。每个沙箱内部跑着一个 cube-agent（Rust，同样基于 Kata Containers 改造），通过 vsock 与宿主机通信。

CubeVS：内核态网络隔离

网络安全是沙箱的硬需求。CubeSandbox 没有用 iptables 或 OVS，而是自研了 CubeVS——三个 eBPF 程序分别挂在 TAP 设备 ingress、宿主机网卡 ingress 和 overlay 设备 egress，覆盖全部流量方向。

设计上有两个值得注意的选择：每个沙箱独占 TAP 设备，没有共享网桥，点对点延迟最低；网络策略用 LPM Trie 在 eBPF 里求值，不经过用户态。私网地址段（10.0.0.0/8、127.0.0.0/8 等）强制禁止访问，不可覆盖。会话跟踪实现了完整的 TCP 状态机，配合 5 秒一次的后台清理器管理 NAT 会话生命周期。

这套方案的性能收益来自一个根本设计：**所有数据面逻辑都在内核态完成**，策略求值、NAT、会话跟踪零上下文切换。

快速上手

CubeSandbox 需要 KVM 环境。WSL 2、Linux 物理机、云上裸金属都可以。没有裸金属也没关系，项目提供了 QEMU 虚机方案可以在虚拟环境里嵌套跑。

```
# 1. 克隆仓库
git clone https://github.com/TencentCloud/CubeSandbox.git
cd CubeSandbox/dev-env
./prepare_image.sh && ./run_vm.sh

# 2. 新终端进入环境，一键安装
./login.sh
curl -sL https://github.com/TencentCloud/CubeSandbox/raw/master/deploy/one-click/online-install.sh | bash

# 3. 创建代码解释器模板
cubemastercli tpl create-from-image \
  --image ccr.ccs.tencentyun.com/ags-image/sandbox-code:latest \
  --writable-layer-size 1G \
  --expose-port 49999 \
  --probe 49999

# 4. 用 E2B SDK 直接跑
pip install e2b-code-interpreter
export E2B_API_URL="http://127.0.0.1:3000"
export CUBE_TEMPLATE_ID="<你的模板ID>"
```

```
from e2b_code_interpreter import Sandbox
with Sandbox.create(template=os.environ["CUBE_TEMPLATE_ID"]) as sandbox:
    result = sandbox.run_code("print('Hello from Cube Sandbox!')")
```

国内用户可以用 `cnb.cool` 镜像加速 clone，安装时加 `MIRROR=cn` 走 CDN。

要注意的事

**硬件门槛是实打实的。** KVM 环境是硬性要求，普通云虚拟机大概率没有嵌套虚拟化权限。这是 MicroVM 方案的固有限制，所有类似方案（Firecracker、Cloud Hypervisor）都一样。

**模板镜像比较大。** 仓库提示预构建的代码解释器镜像相当大，首次下载和解压需要耐心。如果是我自己试，会先在 `cubemastercli tpl watch` 那步开个 `tmux` 放着跑。

**项目刚开源两周。** README 提到已在腾讯云生产环境大规模验证，这个说法我没法独立核实。作为外部用户，社区生态——插件、模板市场、集成案例——都还在建设中。API 稳定性和向前兼容性需要更多时间观察。

**License 要注意。** 主项目 Apache 2.0，但 hypervisor 目录基于 Cloud Hypervisor（Apache + BSD 双许可），agent 基于 Kata Containers。仓库保留了原始版权声明，商业化使用时建议逐个组件确认。

值得关注的场景

如果你正在为 E2B 付费跑 Agent 代码执行，CubeSandbox 提供了一条免费、自托管的退路，迁移成本几乎是零——换一个环境变量就行。

做 RL 训练基础设施的团队可以重点关注。仓库的 examples 里有 SWE-Bench 场景演示，单机数千实例的部署密度对批量执行是关键优势。对容器安全有顾虑、需要执行不可信代码的团队，CubeSandbox 的独立内核 + eBPF 网络隔离是目前开源方案里最完整的组合。

---

如果这篇对你有用，建议点个关注。我会持续把 GitHub 上值得用的 AI 工具拆成「最短上手闭环 + 坑点清单 + 可复用配置」，让你少走弯路。

### 引用链接

[1]CubeSandbox: *https://github.com/TencentCloud/CubeSandbox*