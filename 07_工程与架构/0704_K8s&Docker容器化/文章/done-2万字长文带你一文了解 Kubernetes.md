> 已吸收至：[[07_工程与架构/0704_K8s&Docker容器化/0704_核心知识点/容器化核心知识点总览|容器化核心知识点总览]]
---
title: 2万字长文带你一文了解 Kubernetes
author: 令飞编程
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0MTY1NDczMA==&mid=2247485909&idx=1&sn=65bd4e6ab123c61b0e841c7375e72773&chksm=c339aaf9c62871e7349434f54acc385b31a1f179ef3a661e7c005f6a55e74859e66f4257d2e2&mpshare=1&scene=24&srcid=0715JSY2psoZbuShxu5T7nWF&sharer_shareinfo=fe8a9d7b8339f7295e9a90fc01fe0933&sharer_shareinfo_first=fe8a9d7b8339f7295e9a90fc01fe0933#rd
---

**本文内容源自：「云原生AI实战营」中的「Kubernetes调度器开发实战课」。内容太多，难免出现错误，还望大家多多见谅。**

在开始学习调度器相关技术之前，我们有必要系统的了解下 Kuberentes 及其架构和组件功能。本节课，我就来详细介绍下 Kubernetes 架构及架构中涉及到的各个组件的功能。希望能够通过一篇文章，让你快速、完整的了解整个Kubernetes。

> 提示：本节课中的部分内容在 云原生 AI 实战营中的【Kubernetes 源码剖析课】中已经介绍过了。考虑到本课程的完备性，这里还是在介绍下 Kubernetes 的基础知识。如果里面的知识之前学过或者了解过，大家就权当温故知新了。

## Kubernetes 简介

Kubernetes 是 Google 在 2014 年开源的生产级别的容器编排技术（编排也可以简单理解为调度、管理），用于容器化应用的自动化部署、扩展和管理。它的前身是 Google 内部的 Borg 项目，Borg 是 Google 内部的大规模集群管理系统，它在数千个不同的应用程序中运行数十万个作业，跨越许多集群，每个集群拥有数万台计算机。

> 提示：Kubernetes 的简称是 K8S，其中 `8` 说明 首字母`k`和尾字母`s`之间有 8 个字符。这种命名方式也常见于其他项目的命名中。

Kubernetes 有很多特性，以下是 Kubernetes 的一些核心特性：

| 特性 | 特性说明 |
| --- | --- |
| 服务发现和负载均衡 | Kubernetes 可以使用 DNS 名称或自己的 IP 地址来暴露容器。 如果进入容器的流量很大， Kubernetes 可以负载均衡并分配网络流量，从而使部署稳定。 |
| 存储编排 | Kubernetes 允许你自动挂载你选择的存储系统，例如本地存储、公共云提供商等。 |
| 自动部署和回滚 | 你可以使用 Kubernetes 描述已部署容器的所需状态， 它可以以受控的速率将实际状态更改为期望状态。 例如，你可以自动化 Kubernetes 来为你的部署创建新容器， 删除现有容器并将它们的所有资源用于新容器。 |
| 自动完成装箱计算 | 你为 Kubernetes 提供许多节点组成的集群，在这个集群上运行容器化的任务。 你告诉 Kubernetes 每个容器需要多少 CPU 和内存 (RAM)。 Kubernetes 可以将这些容器按实际情况调度到你的节点上，以最佳方式利用你的资源。 |
| 自我修复 | Kubernetes 将重新启动失败的容器、替换容器、杀死不响应用户定义的运行状况检查的容器， 并且在准备好服务之前不将其通告给客户端。 |
| 密钥与配置管理 | Kubernetes 允许你存储和管理敏感信息，例如密码、OAuth 令牌和 SSH 密钥。 你可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。 |
| 批处理执行 | 除了服务外，Kubernetes 还可以管理你的批处理和 CI（持续集成）工作负载，如有需要，可以替换失败的容器。 |
| 水平扩容缩容 | 使用简单的命令、用户界面或根据 CPU 使用率自动对你的应用进行扩缩。 |
| IPv4/IPv6 双栈 | 为 Pod（容器组）和 Service（服务）分配 IPv4 和 IPv6 地址。 |
| 为可扩展性设计 | 在不改变上游源代码的情况下为你的 Kubernetes 集群添加功能。 |

## Kubernets 架构介绍

Kubernetes 是典型的分布式架构，其架构图如下：

### Kubernetes 中 2 大类节点

Kubernetes 是分布式架构的王者，采用了 Master-Worker 的架构模式。Master 节点也即上图中的 Control Plane Node，Worker 节点也即上图中的 Worker Node。Master 和 Worker 节点详细介绍如下：

* Master 节点：Master 节点部署了 Kubernetes 控制面的核心组件。企业 Kubernetes 集群中，Master 节点会部署 kube-apiserver、kube-controller-manager、kube-scheduler 组件，其中 kube-controller-mananger、kube-scheduler 会通过本地回环接口同 kube-apiserver 通信。kube-controller-manager 和 kube-scheduler 之间没有通信。这些核心的控制面组件用来完成 Kubernetes 资源的 CURD、并根据资源的定义执行相应的业务逻辑，例如：创建 Pod、将 Pod 调度到 Worker 节点等。Kubernetes 中的内置资源，通过 kube-apiserver 进行 CURD 操作，并将数据持久化到 Etcd 中。Etcd 采用集群化部署，在有些企业中，因为没有专门提供 Etcd 集群的中台，也会自己部署 Etcd 集群。每个控制面节点部署一个 Etcd，不同控制面节点的 Etcd 实例，组成一个 Etcd 集群。
* Worker 节点：Worker 节点主要用来运行 Pod。Worker 节点部署了 Kuberentes 的 kubelet、kube-proxy 组件。kubelet 负责跟底层的容器运行时交互，用来管理容器的生命周期。kube-proxy，作为 Kubernetes 集群内的服务注册中心，负责服务发现和负载均衡。Kubernetes 支持不同的容器运行时，例如：containerd、cri-o 等。当前用的最多的是 containerd。

因为 Master 节点部署了 Kubernetes 的核心控制面组件，为了确保控制面组件的稳定性，Master 节点不会运行 workload。

上面我介绍了 Master 节点和 Worker 节点的工作职责及上面部署的组件。接下来，我就来详细介绍上述架构图中的组件功能及交互方式。

### Kubernetes 中组件交互

本小节，我会简单介绍每个组件的功能，主要关注点在不同组件之间的功能交互。后面会详细介绍每个组件的功能。

在 Kubernetes 架构图中，我们可以看到，开发者会使用 client-go 或 kubectl 来访问 kube-apiserver。kube-apiserver 是非常核心的组件，承载着所有的资源增删改查、认证、鉴权等逻辑，其稳定性非常重要。为了确保 kube-apiserver 的高可用，企业会将 kube-apiserver 实例注册到负载均衡器中，通过负载均衡器来访问。通常建议的 kube-apiserver 副本数至少是 3 个。

client-go 其实是一个 Go 语言实现的 SDK，封装了访问 kube-apiserver API 接口的方法，可以使开发者方便的在程序中调用，以访问 kube-apiserver 提供的各种 API 接口。

kubectl 是命令行工具，运维人员可以通过 kubectl 来访问 kube-apiserver。kubectl 除了可以很方便的在命令行中直接使用之外，还可以集成在脚本中，使得开发或者运维可以很便捷的编写脚本来自动运维 kubernetes 集群。kubectl 工具，其实也是使用 client-go 来访问 kube-apiserver 的。所以，client-go、kubectl、kube-apiserver 的关系如下：

在请求到达 kube-apiserver 后，kube-apiserver 会对请求进行身份认证和资源鉴权，在认证和鉴权通过后，请求还会经过准入控制、设置默认值、校验、版本转换等逻辑，并最终将数据保存在 Etcd 中。kube-apiserver 是一个标准的 REST 服务器，内置了很多 REST 资源。kube-apiserver 也支持自定义资源，也即 CRD，通过 CRD 可以极大的提高 kube-apiserver 的扩展能力。访问 kube-apiserver 其实，就是对内置资源和自定义资源进行增删改查、Patch、Watch 等操作，并将数据保存或更新在 Etcd 中，或者从 Etcd 中删除。

kube-controller-manager、kube-scheduler、kubelet、kube-proxy 这些组件，通过 List-Watch 机制，感知 kube-apiserver 中资源的变化。当资源被执行着增删改操作之后，会产生对应的变更事件。上述 4 个组件，会及时的 Watch 到资源的变更，并根据资源的变更类型和变更内容执行相应的逻辑处理：

* kube-controller-mananger：kube-controller-manager 中内置了很多资源的 controller，这些 controller watch 到关注的资源发生变更后，会进行状态调和，根据资源的定义，确保资源的状态始终维持在所声明的状态中。维持状态的逻辑，保存在各个 controller 代码中；
* kube-scheduler：kube-scheduler 为 Kubernetes 集群的调度器，用来调度 Pod 到具体的 Node 节点中。单个 Kubernetes 支持多大几千个节点。一个 Pod 被创建出来之后，需要调度到一个具体的节点上运行，那么该如何决定调度到哪个节点上呢？这就是 kube-scheduler 的工作。kube-scheduler 会根据 Pod 的定义、节点的状态等因素，根据调度策略来将 Pod 调度到具体的节点上。
* kubelet：在 kube-scheduler 将 Pod 调度到具体的节点之后，会产生一个 Pod UPDATE 事件，kubelet watch Pod 的变更事件后，如果发现该 Pod 的 spec.nodeName 值跟 kubelet 所在的节点名相同，就会根据 Pod 的定义，调用底层的容器运行时，在节点上创建需要的容器，从而将 Pod 运行起来；
* kube-proxy：kube-proxy 是 Kubernetes 集群的服务发现组件和负载均衡器。kube-proxy 会 Watch 集群的 Service 和 Pod 资源，并动态维护一个 Service IP（VIP）和 Pod IP（RS） 列表的映射，在 Service 和 Pod 有更新时，会动态的更新这个映射表。在我们通过 Service IP 访问时，kube-proxy 会更具负载均衡策略，选择一个 Pod IP，将请求转发到这个 Pod 中，起到一个负载均衡器的功能。

从上面的介绍，我们可以知道 kube-apiserver 是 Kubernete s 集群中各组件数据交互和通信的枢纽、kube-controller-manager 是资源的控制中心、kube-scheduler 是资源的调度器、kubelet 是节点上 Kubernetes 的代理器、kube-proxy 负责 Pod 的网络访问。

上面，我详细介绍了每个组件的功能，这里我想用简短的语句，再给你总结下每个组件的核心功能：

* kube-apiserver：Web 服务器，提供 RESTful API 接口，完成对内置资源和 CRD 资源的增删改查、Watch、Patch 等操作，并将数据保存/更新到 Etcd 中，或者从 Etcd 中删除；
* kube-controller-manager：Kubernetes 资源的调和器，会 Watch kube-apiserver，在 Watch 到资源变更事件后，会根据事件的类型和内容，对资源进行调和，使资源处于预期的状态；
* kube-scheduler：Kubernetes 集群调度器，用来将 Pod 调度到合适的 Worker Node 上；
* kubelet：负责所在节点上，Pod 的生命周期管理；
* kube-proxy：Kubernetes 集群的服务发现中心和负载均衡器，可以根据服务 IP 和负载均衡策略，访问 RS 列表中具体的某一个 RS（也即 Pod）。

### Kubernetes 组件功能介绍

通过上面的学习，相信你对 Kubernetes 的功能、架构及核心概念已经有所了解。本小节，我再来详细介绍下 Kubernetes 中组件的功能。

#### 控制面组件

Kubernetes 控制面是 Kubernetes 集群的核心部分，负责管理和维护整个集群的状态。它由多个组件组成，每个组件都有特定的功能。以下是 Kubernetes 控制面的核心组件：

* kube-apiserver；
* kube-controller-manager；
* kube-scheduler；
* cloud-controller-manager；
* Etcd。

##### kube-apiserver

kube-apiserver 提供了 Kubernetes 资源对象的唯一操作入口，其他所有的组件都必须通过它提供的 API 接口来操作资源数据。只有 kube-apiserver 会与 Etcd 通信，其他模块都必须通过 kube-apiserver 访问 Etcd。kube-apiserver 作为 Kuberentes 系统的入口，封装了核心对象的增删改查等操作。kube-apiserver 提供 RESTful API 给外部客户端和内部组件调用。它还提供了认证、授权和准入控制等安全机制。 kube-apiserver 设计上考虑了水平伸缩，也就是说，它可通过部署多个实例进行伸缩。 你可以运行 kube-apiserver 的多个实例，并在这些实例之间平衡流量。

kube-apiserver 主要负责以下工作:

* API 服务：提供 Kubernetes API 接口，供用户和其他组件（如 Kubelet、Kube-controller-manager 等）进行交互；
* 资源管理：负责创建、更新、删除和列出集群中的资源（如 Pods、Services、Deployments 等）；
* 认证和授权：

+ 处理用户身份验证，支持多种认证方法（如基本认证、Bearer Tokens、Webhook 等）；
+ 检查用户是否有权限执行特定操作（即授权）。支持 ABAC、RBAC、Webhook 等授权策略。

* 审计日志：记录所有 API 请求和操作的审计日志，可以用于安全审计和问题排查；
* API 版本管理：

+ 支持不同版本的 API，确保向后兼容性和逐步迁移；
+ 提供不同的版本（如 v1、v1beta 等），允许客户端选择适当的 API 版本。

* 数据存储：

+ 与 Etcd 交互，负责持久化存储集群状态和配置数据；
+ 处理所有对 Etcd 的读写请求，确保数据一致性和高可用性。

* 对象的 Watch 机制：

+ 支持客户端订阅对象变化，实现实时更新通知（Watch）；
+ 允许系统组件基于资源变化进行响应。

* API 速率限制：实现请求速率限制，以确保 API Server 的稳定性和性能，防止过载；

##### kube-controller-manager

kube-controller-manager 是一组控制器的集合，用于监控和管理集群中的各种资源。它包括节点控制器、副本控制器、服务控制器、端点控制器等。这些控制器负责监视资源的状态变化，并根据需要采取相应的操作，确保集群中的资源处于期望的状态。

控制器是运行无限控制循环的程序。这意味着它会持续运行并观察对象的实际和期望状态。如果实际状态和期望状态存在差异，控制器会进行操作确保 kubernetes 资源/对象处于期望状态。

假设您想要创建一个 Deployment，您可以在 YAML 中指定它期望的状态，比如，2 个副本、1 个卷挂载、 configmap 等。内置的 controller 将确保 Deployment 始终处于期望的状态。如果用户将副本数更改为 5，Controller 将感知这个变化，并保证 Deployment 的副本的个数变成 5 个。

kube-controller-mananger 内置了很多控制器，这些控制器彼此独立工作，在启动时，也可以通过 `--controllers` 命令行选项，来控制开启/禁用哪些 controller。v1.30.3 版本的 kube-controller-manager 内置了 41 个控制器，你可以在 controller\_names.go[1] 文件中找到内置的控制器。例如有（controller 太多了，功能描述就用 GPT 来生成了，望理解）：

| Kubernetes 控制器 | 功能描述 |
| --- | --- |
| serviceaccount-token-controller | 管理 ServiceAccount 的令牌，自动创建和更新与 ServiceAccount 关联的令牌 |
| endpoints-controller | 构建和更新 Endpoints 对象以反映 Pod 的状态，确保服务的访问列表正确 |
| endpointslice-controller | 管理 EndpointSlice 对象，为服务提供更有效的端点管理 |
| endpointslice-mirroring-controller | 将 EndpointSlice 与传统的 Endpoints 资源之间进行同步，以支持逐步迁移 |
| replicationcontroller-controller | 监控 ReplicationController 对象，确保拥有指定数量的 Pod 副本 |
| pod-garbage-collector-controller | 处理已终止的 Pods，确保它们及其相关资源被正确清理 |
| resourcequota-controller | 监控和强制执行资源配额，确保命名空间内的资源使用不超过预设的限制 |
| namespace-controller | 处理命名空间的生命周期，包括创建、删除和资源清理 |
| serviceaccount-controller | 在创建新命名空间时自动生成 ServiceAccount，并在需要时更新它们 |
| garbage-collector-controller | 管理 Kubernetes 中的垃圾收集，确保不再使用的资源能够被删除 |
| daemonset-controller | 管理 DaemonSet 对象，确保在集群中的每个节点上运行指定的 Pod |
| job-controller | 监控 Job 对象，管理并确保任务能够成功完成 |
| deployment-controller | 管理 Deployment 对象，实现声明式的应用程序部署和滚动更新 |
| replicaset-controller | 监控 ReplicaSet，确保指定数量的 Pod 实例在运行 |
| horizontal-pod-autoscaler-controller | 根据负载自动调整 Pod 副本数，以实现水平扩缩 |
| disruption-controller | 管理 Pod 的中断，如节点维护或升级，确保系统的高可用性 |
| statefulset-controller | 处理 StatefulSet 对象，确保有序部署和管理有状态服务的 Pod |
| cronjob-controller | 管理 CronJob 对象，定时运行 Job |
| certificatesigningrequest-signing-controller | 处理证书签名请求并生成相应的证书 |
| certificatesigningrequest-approving-controller | 自动审批或拒绝证书签名请求 |
| certificatesigningrequest-cleaner-controller | 清理已完成或过期的证书签名请求 |
| ttl-controller | 管理带有 TTL（存活时间限制）的资源，如自动删除过期的对象 |
| bootstrap-signer-controller | 管理 Pod 启动时的证书签名和安全证明 |
| token-cleaner-controller | 管理并删除不再使用的 ServiceAccount 令牌 |
| node-ipam-controller | 管理节点的 IP 地址分配，确保 IP 地址的正确使用 |
| node-lifecycle-controller | 监控节点的生命周期，如上报节点状态和处理节点故障 |
| taint-eviction-controller | 根据污点（taint）策略决定是否将 Pod 驱逐出节点 |
| persistentvolume-binder-controller | 管理 PVC（PersistentVolumeClaim）和 PV（PersistentVolume）之间的绑定关系 |
| persistentvolume-attach-detach-controller | 处理持久卷的附加和分离操作 |
| persistentvolume-expander-controller | 处理持久卷的扩展操作 |
| clusterrole-aggregation-controller | 处理 ClusterRole 的聚合，以自动便捷地管理 RBAC 规则 |
| persistentvolumeclaim-protection-controller | 保护 PersistentVolumeClaim，防止不必要的删除 |
| persistentvolume-protection-controller | 保护 PersistentVolume 对象，确保不被意外删除 |
| ttl-after-finished-controller | 管理在 Job 完成后删除这些 Job 的 TTL |
| root-ca-certificate-publisher-controller | 处理根 CA 证书的发布 |
| ephemeral-volume-controller | 管理临时卷的生命周期 |
| storageversion-garbage-collector-controller | 处理不同存储版本的垃圾收集 |
| resourceclaim-controller | 管理资源声明，确保资源的请求与分配 |
| legacy-serviceaccount-token-cleaner-controller | 清理旧版 ServiceAccount 令牌，确保系统内没有多余的令牌 |
| validatingadmissionpolicy-status-controller | 管理 ValidatingAdmissionPolicy 的状态 |
| service-cidr-controller | 管理集群的服务 CIDR 范围 |
| storage-version-migrator-controller | 处理存储版本迁移，确保数据兼容性 |

##### cloud-controller-mananger

当在云环境中部署 kubernetes 时， cloud-controller-mananger 充当云平台 API 接口和 Kubernetes 集群之间的桥梁。Kubernetes 核心组件可以独立工作，也允许通过插件方式与云提供商集成。(例如，Kubernetes 集群和 AWS 云 API 之间的接口)

cloud-controller-mananger 包含一组云平台控制器，用于保证 cloud 组件（如：nodes，Loadbalancers，storage）的状态符合预期。cloud-controller-mananger 的三个主要控制器为：

* Node controller：该控制器通过与云提供商 API 通信，更新节点相关信息。例如，节点的 Label、Annotation、主机名、CPU 和内存、节点运行状况等；
* Route controller：负责在云平台上配置网络路由。这样不同节点上的 Pod 就可以互相通信了；
* Service controller：负责为 kubernetes 服务部署负载均衡器，分配 IP 地址等。

cloud-controller-mananger 的经典使用场景如下：

* 部署负载均衡器类型的 Kubernetes 服务。在这里，Kubernetes 提供了一个特定于云的负载均衡器，并与 Kubernetes Service 集成；
* 通过云存储解决方案为 Pod 提供存储卷（PV）。

##### kube-sheduler

kube-scheduler 负责将新创建的 Pod 调度到集群中的节点上。它负责监视新创建的、未指定运行节点（node）的 Pods，选择节点让 Pod 在上面运行。调度决策考虑的因素包括单个 Pod 和 Pod 集合的资源需求、硬件/软件/策略约束、亲和性和反亲和性规范、数据位置、工作负载间的干扰和最后时限。

下图显示了调度器工作原理的主要流程：

##### 非 Kubernetes 组件：Etcd

Etcd 是 Kubernetes 的分布式键值存储系统，用于存储集群的配置数据和状态信息。它提供高可用性和一致性，并用于存储集群中的各种信息，如节点信息、Pod 信息、服务信息、配置信息等。Etcd 是控制平面中唯一的有状态组件。

Etcd 是一个开源的强一致性分布式键值存储：

* 强一致性：如果对一个节点进行了更新，强一致性将确保它立即更新到集群中的所有其他节点。在 CAP 定理的限制下，在强一致性和分区容忍的情况下，实现 100% 可用性是不可能的；
* 分布式：Etcd 被设计成在保留强一致性的前提下，作为一个集群在多个节点上运行；
* 键值存储：将数据存储为键和值的非关系数据库。它还公开了一个键值 API。数据存储构建在 BoltDB 之上（BoltDB 是 BoltDB 的一个分支）。

Etcd 采用 Raft 共识算法，具有较强的一致性和可用性。它以领导者-成员方式工作，以实现高可用性并承受节点故障。

Etcd 在 Kubernetes 集群中的 作用包括：

* Etcd 存储 Kubernetes 对象的所有配置、状态和元数据， 包括：Pods、Secrets、Daemonsets、Deployment、Configmaps、Statfulsets 等对象；
* Etcd 允许客户端使用 Watch() API 订阅事件。kube-apiserver 使用 Etcd 的 Watch 功能来跟踪对象状态的变化；
* Etcd 使用 gRPC 公开键值 API。此外，gRPC 网关作为 RESTful 代理，将所有 HTTP API 调用转换为 gRPC 消息。这使得它成为 Kubernetes 的理想数据库；
* Etcd 以键值方式，将所有对象存储在 `/registry` 目录下。例如，一个在 `default` 命名空间中，名字为`nginx` 的 Pod，可以在 `/registry/pods/default/nginx` 下找到。

#### 数据面组件

Kubernetes 数据面是指与集群内部运行的 Pods 及其网络、存储和其他服务直接相关的部分。与控制面负责状态管理、调度和 API 请求不同，数据面的主要关注点是处理应用程序的实际运行和数据流。以下是数据面的核心组件：

* kubelet；
* kube-proxy；
* container runtime。

##### kubelet

kubelet 是运行在每个节点上的代理程序，负责管理和监控节点上的容器。它与 kube-apiserver 通信，接收来自控制平面的指令，并根据指令创建、启动、停止和销毁容器。它还负责监控容器的状态和健康状况，并上报给控制平面。

kubelet 是负责容器真正运行的核心组件，主要职责如下所示：

* 负责 Node 节点上 Pod 的创建、修改、监控、删除等全生命周期的管理；
* 定时上报本地 Node 的状态信息给 kube-apiserver；
* kubelet 是 Master 和 Node 之间的桥梁，接收 kube-apiserver 分配给它的任务并执行；
* kubelet 通过 kube-apiserver 间接与 Etcd 集群交互来读取集群配置信,息；
* kubelet 在 Node 上做的主要工作具体如下:

+ 设置容器的环境变量、给容器绑定 Volume、给容器绑定 Port；
+ 为 Pod 创建，更新，删除容器；
+ 负责处理存活（Liveness）、就绪（Readiness）和启动（Startup）探针；
+ 通过读取 Pod 配置，在主机上为卷挂载创建相应的目录来挂载卷；

Kubelet 除了能够接收来自 kube-apiserver 的 Pod 资源定义外，还可以接受来自文件、HTTP 端口和 HTTP 服务器的 Pod 定义。Kubernetes 的静态 Pod，就是是“从文件中获取 Pod 定义”的一个很好的例子。（静态 Pod 由 kubelet 控制，而不是 kube-apiserver）

##### kube-proxy

kube-proxy 负责为 Pod 提供网络代理和负载均衡功能。它维护集群中的网络规则和转发表，并将请求转发到合适的目标 Pod 上。它支持多种代理模式，如用户空间代理、iptables 代理和 IPVS 代理，以满足不同网络环境的需求。

kube-proxy 是集群中每个节点（Node）上所运行的网络代理， 实现 Kubernetes 服务（Service） 概念的一部分。

kube-proxy 是一个 Kubernetes Daemonset，在每个节点上运行。它是实现 Kubernetes Services 概念的代理组件。(为每组 Pod 提供具有负载均衡的 DNS)。它主要代理 UDP、TCP 和 SCTP。（不能直接代理 HTTP）

当你使用 Service (ClusterIP)公开 Pod 时，kube-proxy 将创建网络规则，将流量发送到分组在 Service 对象下的后端 Pod。也就是说，所有的负载平衡和服务发现都由 kube-rproxy 处理。

##### 非 Kuberentes 组件：Container Runtime

容器运行时是负责在节点上创建和管理容器的软件。Kubernetes 支持多种容器运行时，如 Docker、containerd、CRI-O 等。容器运行时负责拉取镜像、启动容器、管理容器的生命周期，并提供容器的隔离和资源管理。

kubernetes 要求容器运行时必须实现了 CRI（Container Runtime Interface 容器运行时接口）。CRI 是一个插件接口，它使 kubelet 能够使用各种容器运行时，无需重新编译集群组件，是 kubelet 和容器运行时之间通信的主要协议。

常见的容器运行时有：

* containerd：官网推荐的容器运行时，是从 docker 分离出来的；
* CRI-O：是 containerd 的一个替代品，专门为 kubernetes 创建的一个容器运行时；
* Docker Engine：需要先调用 dockershim，然后调用 docker，再调用 containerd，最后调用底层的 runc。

CRI-O, Containerd, Docker 三者区别见下图：

以 CRI-O 为例，容器运行时与 kubernetes 的交互见下图：

1. 当 kube-apiserver 对 Pod 发出新的请求时，kubelet 与 CRI-O 守护进程交互，通过 Kubernetes 容器运行时接口启动所需的容器；
2. CRI-O 使用 containers/image 库，根据配置的容器信息，检查并 Pull 镜像；
3. CRI-O 为容器生成 OCI 运行时规范(OCI specification ，JSON 格式);
4. CRI-O 启动一个 OCI 兼容的运行时(runc)，按照运行时规范启动容器进程。

#### 客户端组件：kubectl

kubectl 是 Kubernetes 的命令行工具，它是 Kubernetes 的官方命令行客户端。它允许用户通过命令行与 Kubernetes 集群进行交互，并执行各种操作，包括管理集群中的资源对象、配置集群、故障排查和日志查看等。

kubectl 提供了与 Kubernetes API 进行通信的功能，它可以向 Kubernetes 集群发送命令和请求，然后接收和解析 API 响应，并将结果返回给用户。通过 kubectl，用户可以通过简单的命令操作来管理和控制 Kubernetes 集群，而无需直接与底层的 API 进行交互。

kubectl 支持丰富的命令和选项，可以用于创建、查看、更新和删除 Kubernetes 集群中的各种资源对象，如 Pod、Service、Deployment 等。它还提供了许多功能，如集群管理、配置管理、故障排查和日志查看等，使得用户可以轻松地管理和操作 Kubernetes 集群。

### 通过 Pod 创建流程了解 Kubernetes

为了使你更好的理解 Kubernetes 中各个组件的功能和交互，本小节，我通过 Kubernetes 创建 Pod 的流程，来给你展示下 Kubernetes 种各个组件的功能和交互。

在 Kubernetes 中，可以通过多种方式来创建一个 Pod，例如：创建一个裸 Pod、创建 Deployment、创建 ReplicaSet、创建 CronJob、创建 Job 等，为了能够给你简洁、全面的介绍 Kubernetes 的资源处理流程，这里我通过创建一个 ReplicaSet 来看下 Kuberentes 具体是如何创建 Pod 的。

创建流程如下：

#### 步骤 0：kube-controller-manager Watch kube-apiserver

首先，在启动 kube-controller-manager 时，kube-controller-manager 会跟 kube-apiserver 建立连接，并通过 List-Watch 机制，实时监听 kube-apiserver 中的资源变更事件，根据变更事件，调用相应的 Handler 来调和资源。此时，`kube-apiserver` 会保持这个连接，并在有相关资源变化时（如创建、更新或删除操作）通过流式响应的方式将变更信息推送给 `kube-controller-manager`。

其他组件，例如：kube-scheduler、kubelet 等组件也是用同样的方式跟 kube-apiserver 建立连接。

#### 步骤 1：创建 ReplicaSet 资源

用户通过 kubectl 工具调用 kube-apiserver 接口创建一个 ReplicaSet 资源。创建 ReplicaSet 资源的 YAML 定义如下：

```
apiVersion: apps/v1
kind:ReplicaSet
metadata:
namespace:default
name:my-replicaset
spec:
replicas:2# 指定要运行的 Pod 数量
selector:
    matchLabels:
      app:my-app# 与 Pod 的标签匹配
template:
    metadata:
      labels:
        app:my-app# Pod 的标签
    spec:
      containers:
      -name:my-container# 容器名称
        image:nginx# 容器镜像（示例使用 nginx）
        ports:
        -containerPort:80# 容器暴露的端口
```

将上述内容保存在 replicaset.yaml 文件中，并执行以下命令来创建 ReplicaSet 资源：

```
$ kubectl create -f replicaset.yaml -v 10
```

上述命令中，`-v 10`会指定 kubectl 非常详细的操作日志。为了让你更清晰的看到 `kubectl create -f replicaset.yaml -v 10`后，kubectl 的核心操作，我简化了 kubectl 的日志输出内容，并在日志中添加了注释，具体如下：

```
# 1. 加载 kubeconfig 文件，该文件指定了连接 kube-apiserver 的证书、访问地址等信息
I0814 22:40:10.326070 2634693 loader.go:395] Config loaded from file:  /home/colin/.kube/config
# 2. kubeclt -v =10 会打印请求 kube-apiserve r的 curl 命令，方便你阅读、排障
I0814 22:40:10.327486 2634693 round_trippers.go:466] curl -v -XGET  -H "Accept: application/json, */*" -H "User-Agent: kubectl/v1.30.1 (linux/amd64) kubernetes/6911225"'https://10.37.43.62:6443/openapi/v3?timeout=32s'
I0814 22:40:10.328510 2634693 round_trippers.go:510] HTTP Trace: Dial to tcp:10.37.43.62:6443 succeed
# 3. kubectl 访问 kube-apiserver 的 GET /openapi/v3 接口。kubectl 访问 GET /openapi/v3 接口主要是为了获取 Kubernetes API 的 OpenAPI 规范。这一规范提供了关于 API 端点、请求和响应格式的结构化信息，帮助 kubectl 了解集群的 API以及如何与之交互。
I0814 22:40:10.335157 2634693 round_trippers.go:553] GET https://10.37.43.62:6443/openapi/v3?timeout=32s 200 OK in 7 milliseconds
I0814 22:40:10.335266 2634693 round_trippers.go:570] HTTP Statistics: DNSLookup 0 ms Dial 0 ms TLSHandshake 4 ms ServerProcessing 1 ms Duration 7 ms
# GET /openapi/v3 接口返回头
I0814 22:40:10.335282 2634693 round_trippers.go:577] Response Headers:
I0814 22:40:10.335294 2634693 round_trippers.go:580]     Accept-Ranges: bytes
I0814 22:40:10.335355 2634693 round_trippers.go:580]     Content-Type: text/plain; charset=utf-8
I0814 22:40:10.335366 2634693 round_trippers.go:580]     ...
# GET /openapi/v3 返回体
I0814 22:40:10.336639 2634693 request.go:1212] Response Body: {"paths":{"apis/apps":{"serverRelativeURL":"/openapi/v3/apis/apps?hash=xxxx"},"apis/apps/v1":{"serverRelativeURL":"/openapi/v3/apis/apps/v1?hash=xxx"}}}
I0814 22:40:10.337349 2634693 round_trippers.go:466] curl -v -XGET  -H "Accept: application/json" -H "User-Agent: kubectl/v1.30.1 (linux/amd64) kubernetes/6911225"'https://10.37.43.62:6443/openapi/v3/apis/apps/v1?hash=xxx&timeout=32s'
# 请求 kube-apiserver 的 GET /openapi/v3/apis/apps/v1 接口，获取 Kubernetes 中特定 API 组（在这个例子中是 apps 组）的 OpenAPI 规范。这一规范详细描述了该 API 组中的资源（例如ReplicaSet、Deployment 等）的结构、操作和约束。
I0814 22:40:10.341199 2634693 round_trippers.go:553] GET https://10.37.43.62:6443/openapi/v3/apis/apps/v1?hash=xxx&timeout=32s 200 OK in 3 milliseconds
I0814 22:40:10.341233 2634693 round_trippers.go:570] HTTP Statistics: DNSLookup 0 ms Dial 0 ms TLSHandshake 0 ms Duration 3 ms
# GET /openapi/v3/apis/apps/v1 接口返回头
I0814 22:40:10.341241 2634693 round_trippers.go:577] Response Headers:
I0814 22:40:10.341250 2634693 round_trippers.go:580]     Last-Modified: Sat, 25 May 2024 09:27:53 GMT
I0814 22:40:10.341309 2634693 round_trippers.go:580]     ...
# GET /openapi/v3/apis/apps/v1 返回体，指定了 apiVersion = apps/v1， kind = ReplicaSet 的 OpenAPI 3.0 接口定义
I0814 22:40:10.346282 2634693 request.go:1212] Response Body: {"openapi":"3.0.0", ...}
# 请求 POST /apis/apps/v1/namespaces/default/replicasets 接口，创建 ReplicaSet 资源
I0814 22:40:10.375058 2634693 request.go:1212] Request Body: {"apiVersion":"apps/v1","kind":"ReplicaSet","metadata":{"name":"my-replicaset","namespace":"default"},"spec":{"replicas":2,"selector":{"matchLabels":{"app":"my-app"}},"template":{"metadata":{"labels":{"app":"my-app"}},"spec":{"containers":[{"image":"nginx","name":"my-container","ports":[{"containerPort":80}]}]}}}}
# 打印请求 POST /apis/apps/v1/namespaces/default/replicasets 接口的 curl 命令
I0814 22:40:10.375203 2634693 round_trippers.go:466] curl -v -XPOST  -H "Accept: application/json" -H "Content-Type: application/json" -H "User-Agent: kubectl/v1.30.1 (linux/amd64) kubernetes/6911225"'https://10.37.43.62:6443/apis/apps/v1/namespaces/default/replicasets?fieldManager=kubectl-create&fieldValidation=Strict'
# 请求 POST /apis/apps/v1/namespaces/default/replicasets 接口
I0814 22:40:10.382675 2634693 round_trippers.go:553] POST https://10.37.43.62:6443/apis/apps/v1/namespaces/default/replicasets?fieldManager=kubectl-create&fieldValidation=Strict 201 Created in 7 milliseconds
# 返回头
I0814 22:40:10.382737 2634693 round_trippers.go:577] Response Headers:
I0814 22:40:10.382751 2634693 round_trippers.go:580]     Audit-Id: d7974927-8e25-476a-a9b1-4b39e6d0a0ad
I0814 22:40:10.382758 2634693 round_trippers.go:580]     ...
# 返回体
I0814 22:40:10.382894 2634693 request.go:1212] Response Body: {"kind":"ReplicaSet","apiVersion":"apps/v1","metadata":{"name":"my-replicaset","namespace":"default","uid":"b22383e9-e414-423c-a03e-0633a2b1484a","resourceVersion":"42746512","generation":1,"creationTimestamp":"2024-08-14T14:40:10Z"},"spec":{"replicas":2,"selector":{"matchLabels":{"app":"my-app"}},"template":{"metadata":{"creationTimestamp":null,"labels":{"app":"my-app"}},"spec":{"containers":[{"name":"my-container","image":"nginx","ports":[{"containerPort":80,"protocol":"TCP"}],"resources":{},"terminationMessagePath":"/dev/termination-log","terminationMessagePolicy":"File","imagePullPolicy":"Always"}],"restartPolicy":"Always","terminationGracePeriodSeconds":30,"dnsPolicy":"ClusterFirst","securityContext":{},"schedulerName":"default-scheduler"}}},"status":{"replicas":0}}
```

通过上面的日志，我们可以看到在 `kubectl create -f replicaset.yaml -v 10`具体执行了以下操作：

1. 加载 kubeconfig 文件，该文件指定了连接 kube-apiserver 的证书、访问地址等信息；
2. kubeclt -v =10 会打印请求 kube-apiserve r 的 curl 命令，方便你阅读、排障；
3. kubectl 访问 kube-apiserver 的 `GET /openapi/v3` 接口，主要是为了获取 Kubernetes API 的 OpenAPI 规范。这一规范提供了关于 API 端点、请求和响应格式的结构化信息，帮助 kubectl 了解集群的 API 以及如何与之交互；
4. 请求 kube-apiserver 的 `GET /openapi/v3/apis/apps/v1` 接口，获取 Kubernetes 中特定 API 组（在这个例子中是 `apps` 组）的 OpenAPI 规范。这一规范详细描述了该 API 组中的资源（例如 ReplicaSet、Deployment 等）的结构、操作和约束。了解这些可以对 API 资源定义进行验证、生成请求对象等；
5. 请求 `POST /apis/apps/v1/namespaces/default/replicasets` 接口，创建 ReplicaSet 资源；
6. 打印请求 `POST /apis/apps/v1/namespaces/default/replicasets` 接口的 curl 命令；
7. 请求 `POST /apis/apps/v1/namespaces/default/replicasets` 接口创建 ReplicaSet 资源。

#### 步骤 2：kube-apiserver 将 ReplicaSet 资源创建数据写入 Etcd

在调用 `kubectl create -f replicaset.yaml`请求 kube-apiserver 创建 ReplicaSet 资源后，kube-apiserver 会对请求进行认证、鉴权，对资源设置默认值、准入控制、参数校验等，并将资源数据保存在 Etcd 中。

我们这时候，可以通过 `etcdctl`工具查看 Etcd 中保存的资源数据：

```
$ etcdctl --endpoints=https://10.37.43.62:2379,https://10.37.91.93:2379 --cacert=/etc/kubernetes/cert/ca.pem --cert=/etc/kubernetes/cert/kubernetes.pem --key=/etc/kubernetes/cert/kubernetes-key.pem get /registry/replicasets/default/my-replicaset --write-out=json | jq .
{
"header": {
    "cluster_id": 8279541938341675000,
    "member_id": 6830259116106892000,
    "revision": 43161970,
    "raft_term": 6
  },
"kvs": [
    {
      "key": "L3JlZ2lzdHJ5L3JlcGxpY2FzZXRzL2RlZmF1bHQvbXktcmVwbGljYXNldA==",
      "create_revision": 42746512,
      "mod_revision": 42746579,
      "version": 5,
      "value": "azhzAAoVCgdhcHBzL3YxEgpSZXBsaWNhU2V0EuEICvgGCg1teS1yZXBsaWNhc2V0EgAaB2RlZmF1bHQiACokYjIyMzgzZTktZTQxNC00MjNjLWEwM2UtMDYzM2EyYjE0ODRhMgA4AUIICMqD87UGEACKAdIECg5rdWJlY3RsLWNyZWF0ZRIGVXBkYXRlGgdhcHBzL3YxIggIyoPztQYQADIIRmllbGRzVjE6mAQKlQR7ImY6c3BlYyI6eyJmOnJlcGxpY2FzIjp7fSwiZjpzZWxlY3RvciI6e30sImY6dGVtcGxhdGUiOnsiZjptZXRhZGF0YSI6eyJmOmxhYmVscyI6eyIuIjp7fSwiZjphcHAiOnt9fX0sImY6c3BlYyI6eyJmOmNvbnRhaW5lcnMiOnsiazp7XCJuYW1lXCI6XCJteS1jb250YWluZXJcIn0iOnsiLiI6e30sImY6aW1hZ2UiOnt9LCJmOmltYWdlUHVsbFBvbGljeSI6e30sImY6bmFtZSI6e30sImY6cG9ydHMiOnsiLiI6e30sIms6e1wiY29udGFpbmVyUG9ydFwiOjgwLFwicHJvdG9jb2xcIjpcIlRDUFwifSI6eyIuIjp7fSwiZjpjb250YWluZXJQb3J0Ijp7fSwiZjpwcm90b2NvbCI6e319fSwiZjpyZXNvdXJjZXMiOnt9LCJmOnRlcm1pbmF0aW9uTWVzc2FnZVBhdGgiOnt9LCJmOnRlcm1pbmF0aW9uTWVzc2FnZVBvbGljeSI6e319fSwiZjpkbnNQb2xpY3kiOnt9LCJmOnJlc3RhcnRQb2xpY3kiOnt9LCJmOnNjaGVkdWxlck5hbWUiOnt9LCJmOnNlY3VyaXR5Q29udGV4dCI6e30sImY6dGVybWluYXRpb25HcmFjZVBlcmlvZFNlY29uZHMiOnt9fX19fUIAigHOAQoXa3ViZS1jb250cm9sbGVyLW1hbmFnZXISBlVwZGF0ZRoHYXBwcy92MSIICMyD87UGEAAyCEZpZWxkc1YxOoUBCoIBeyJmOnN0YXR1cyI6eyJmOmF2YWlsYWJsZVJlcGxpY2FzIjp7fSwiZjpmdWxseUxhYmVsZWRSZXBsaWNhcyI6e30sImY6b2JzZXJ2ZWRHZW5lcmF0aW9uIjp7fSwiZjpyZWFkeVJlcGxpY2FzIjp7fSwiZjpyZXBsaWNhcyI6e319fUIGc3RhdHVzEtcBCAISDwoNCgNhcHASBm15LWFwcBq/AQofCgASABoAIgAqADIAOABCAFoNCgNhcHASBm15LWFwcBKbARJWCgxteS1jb250YWluZXISBW5naW54KgAyDQoAEAAYUCIDVENQKgBCAGoUL2Rldi90ZXJtaW5hdGlvbi1sb2dyBkFsd2F5c4ABAIgBAJABAKIBBEZpbGUaBkFsd2F5cyAeMgxDbHVzdGVyRmlyc3RCAEoAUgBYAGAAaAByAIIBAIoBAJoBEWRlZmF1bHQtc2NoZWR1bGVywgEAIAAaCggCEAIYASACKAIaACIA"
    }
  ],
"count": 1
}
```

etcdctl 命令中，`-w`指定了输出的格式，key 的值是经过 base64 编码，需要解码后才能看到实际值，如：

```
$ echo L3JlZ2lzdHJ5L3JlcGxpY2FzZXRzL2RlZmF1bHQvbXktcmVwbGljYXNldA==|base64 -d
/registry/replicasets/default/my-replicaset
```

这里，我们再多探索一步，来看下，Kubernetes 中 Etcd 中数据的保存内容和格式。执行以下命令查看 Etcd 中保存的 Key：

```
$ etcdctl --endpoints=https://10.37.43.62:2379,https://10.37.91.93:2379 --cacert=/etc/kubernetes/cert/ca.pem --cert=/etc/kubernetes/cert/kubernetes.pem --key=/etc/kubernetes/cert/kubernetes-key.pem get / --prefix --keys-only
/registry/ThirdPartyResourceData/istio.io/istioconfigs/default/route-rule-details-default
/registry/ThirdPartyResourceData/istio.io/istioconfigs/default/route-rule-productpage-default
/registry/ThirdPartyResourceData/istio.io/istioconfigs/default/route-rule-ratings-default
...
/registry/configmaps/default/namerctl-script
/registry/configmaps/default/namerd-config
/registry/configmaps/default/nginx-config
...
/registry/deployments/default/sdmk-page-sdmk
/registry/deployments/default/sdmk-payment-web
/registry/deployments/default/sdmk-report
...
```

我们可以看到所有的 Kuberentes 的所有元数据都保存在`/registry`目录下，下一层就是 API 对象类型（复数形式），再下一层是`namespace`，最后一层是对象的名字。Kubernetes 使用了 Etcd V3 API。V3 版本的数据存储没有目录层级关系，而是采用了平展（flat）模式，换句话说 `/a`和 `/a/b`并没有嵌套关系，而只是 key 的名字有差别而已，这个和 AWS S3 及 OpenStack Swift 对象存储一样，没有目录的概念，但是 key 的名称支持 `/`字符，从而实现起来像目录的伪目录，但是存储结构上不存在层级关系。

另外，为了提高存储性能，Kubernetes 在存储在 Etcd 中的 `value`是 protobuf 格式，我们不能直接查看其中的内容，需要借助 etcdhelper[2] 工具来查看。查看命令如下：

```
$ etcdhelper -key master.etcd-client.key -cert master.etcd-client.crt -cacert ca.crt get /openshift.io/imagestreams/openshift/python
```

#### 步骤 3：Etcd 发送 ReplicaSet 创建事件给 kube-apiserver

在 Etcd 保存了 ReplicaSet 的资源数据后，会发送一个 ReplicaSet 的 CREATE 事件给 kube-apiserver。

#### 步骤 4：kube-controller-manager Watch 到 kube-apiserver ReplicaSet 创建事件

在 kube-apiserver 收到 Etcd 返回的 ReplicaSet 创建事件之后，会将该事件推送给所有 Watch 了 kube-apiserver 的组件，例如：kube-controller-manager。

#### 步骤 5：创建 Pod

在 kube-controller-manager Watch 到 ReplicaSet 的创建事件之后，会解析事件的对象数据，其实就是 ReplicaSet 的资源定义：

```
apiVersion: apps/v1
kind:ReplicaSet
metadata:
namespace:default
name:my-replicaset
spec:
replicas:2# 指定要运行的 Pod 数量
selector:
    matchLabels:
      app:my-app# 与 Pod 的标签匹配
template:
    metadata:
      labels:
        app:my-app# Pod 的标签
    spec:
      containers:
      -name:my-container# 容器名称
        image:nginx# 容器镜像（示例使用 nginx）
        ports:
        -containerPort:80# 容器暴露的端口
```

解析 ReplicaSet 的资源定义之后，并可以根据其 Spec 定义进行资源调和，具体逻辑为：解析资源定义数据，例如：需要创建 Pod 的副本数为 2、需要创建的容器名和镜像地址、容器端口等。之后会调用 kube-apiserver 的 Pod 创建接口，来创建指定副本数的 Pod。创建的 Pod 名字格式为 `my-replicaset-xxx`，其中 `xxx`为 kube-apiserver 自定生成的字符串。

可以通过 kubectl 命令来查看 Pod 的创建情况：

```
$ kubectl get pod|grep my-replicaset
my-replicaset-qswlg                                    1/1     Running            0                  10h
my-replicaset-ts2mx                                    1/1     Running            0                  10h
```

通过以下命令，还可以查看到该 Pod 具体是由哪个资源（`ownerReferences`）创建而来的：

```
kubectl getpodmy-replicaset-qswlg-oyaml
apiVersion:v1
kind:Pod
metadata:
creationTimestamp:"2024-08-14T14:40:10Z"
generateName:my-replicaset-
labels:
    app:my-app
name:my-replicaset-qswlg
namespace:default
ownerReferences:
-apiVersion:apps/v1
    blockOwnerDeletion:true
    controller:true
    kind:ReplicaSet
    name:my-replicaset
    uid:b22383e9-e414-423c-a03e-0633a2b1484a
resourceVersion:"42746576"
uid:a795d4c8-1f75-4dca-83c2-588d90cdf51d
spec:
containers:
-image:nginx
    imagePullPolicy:Always
    name:my-container
    ...
```

#### 步骤 6：kube-apiserver 将 Pod 资源创建数据写入 Etcd

在 kube-apiserver 收到 Pod 的创建请求后，会对请求进行认证、鉴权，对资源设置默认值、准入控制、参数校验等，并将资源数据保存在 Etcd 中。

#### 步骤 7：Etcd 发送 Pod 创建事件给 kube-apiserver

在 Etcd 保存了 Pod 的资源数据后，会发送一个 Pod 的 CREATE 事件给 kube-apiserver。

#### 步骤 8：kube-scheduler Watch 到 kube-apiserver Pod 创建事件

在 kube-apiserver 收到 Etcd 返回的 Pod 创建事件之后，会将该事件推送给所有 Watch 了 kube-apiserver 的组件，例如：kube-scheduler。

#### 步骤 9：kube-scheduler 调度 Pod 后，更新 Pod 资源

在 kube-scheduler Watch 到 Pod 的创建事件之后，会解析事件的对象数据，其实就是 Pod 的资源定义。并根据集群中的节点个数及状态，Pod 的资源定义，通过一系列的调度插件，将 Pod 调度到合适的节点上。在 kube-scheduler 找到一个适合 Pod 运行的 Node 之后，会将 NodeName 写入 Pod 的 `spec.nodeName`字段，例如：

```
apiVersion: v1
kind:Pod
metadata:
...
labels:
    app:my-app
name:my-replicaset-qswlg
namespace:default
...
uid:a795d4c8-1f75-4dca-83c2-588d90cdf51d
spec:
containers:
-image:nginx
    ...
nodeName:k8s-01
```

kube-scheduler 通过将 节点名写入 Pod 的 `spec.nodeName`字段，来将 Pod 调度到该节点上。kube-sheduler 更新完 Pod 的资源定义后，会请求 kube-apiserver，将更新后的资源定义写入 Etcd。

#### 步骤 10：kube-apiserver 将 Pod 资源更新数据写入 Etcd

kube-apiserver 在收到 kube-scheduler 更新 Pod 的请求后，会对请求进行认证、鉴权，对资源设置默认值、准入控制、参数校验等，并将资源数据保存在 Etcd 中。

#### 步骤 11：Etcd 发送 Pod 变更事件给 kube-apiserver

在 Etcd 保存了 Pod 的资源更新数据后，会发送一个 Pod 的 UPDATE 事件给 kube-apiserver。

#### 步骤 12：kubelet Watch 到 kube-apiserver Pod 变更事件，并创建 Pod

在 kube-apiserver 收到 Etcd 返回的 Pod 更新事件之后，会将该事件推送给所有 Watch 了 kube-apiserver 的组件，例如：kubelet。

kubelet 获取到 Pod 的资源变更之后，会过滤掉所有 `spec.nodeName`值不是 kubelet 所在节点的节点名的 Pod 资源的所有事件。这样，kubelet 就只会处理调度在其所在节点的 Pod，例如上述的 Pod。

kubelet 之后会解析 Pod 的资源定义，调用底层的容器运行时，例如 containerd， 下载镜像、绑定存储卷、创建网络等，最终创建出 Pod，使 Pod 处在 `Running` 状态。kubelet 还会定期调用 kube-apiserver 接口，更新 Pod 的状态。例如：

```
$ kubectl get pods|grep my-replicaset
my-replicaset-qswlg                                    1/1     Running            0                 12h
my-replicaset-ts2mx                                    1/1     Running            0                 12h
```

## Kubernetes 基本概念和术语

Kubernetes 中有众多的概念和术语。本小节，我来介绍其中的核心概念及术语，以让你对 Kubernetes 有更全面的了解。Kubernetes 中涉及到的核心概念及术语如下：

### Resource

Kubernetes 中的大部分概念如 Node、Pod、Replication Controller、Service 等都可以被看做一种资源对象，几乎所有资源对象都可以通过 Kubernetes 提供的 kubectl 工具（或 API 编程调用）执行增、删、改、查等操作并将其保存在 etcd 中持久化存储。

Kubernetes 其实是一个高度自动化的资源控制系统，通过跟踪对比 etcd 库里保存的“资源期望状态”与当前环境的“实际资源状态”的差异来实现自动控制和自动纠错的高级功能。

### Kubernetes API version

查看可用的 API Version 命令：

```
$ kubectl api-versions
acme.cert-manager.io/v1
admissionregistration.k8s.io/v1
admissionregistration.k8s.io/v1alpha1
admissionregistration.k8s.io/v1beta1
...
```

k8s 官方将 api version 分成了三个大类型，alpha、beta、stable：

* Alpha：未经充分测试，可能存在 bug，功能可能随时调整或删除；
* Beta：经过充分测试，功能细节可能会在未来进行修改；
* Stable：稳定版本，将会得到持续支持。

### Pod（核心概念）

Pod 是 Kubernetes 最重要的基本概念，是 Kubernetes 的最小调度单位。每个 Pod 都有一个特殊的被称为“根容器”的 Pause 容器。除了 Pause 跟根容器外，每个 Pod 还包含了一个或多个紧密相关的用户业务容器。

为什么 Kubernetes 会这么设计呢？

1. 在实际情况下，存在多个紧密相关的业务容器需要部署为一组容器，那么 pod 支持将多个容器部署在一个 pod，pod 里的多个业务容器共享 pause 的 IP 和 Volume，解决了密切关联的容器之间的通信问题，数据共享问题。
2. 一个 pod 里面包含多个业务容器，在这种情况下，Kubernetes 无法对 pod 这个整体的状态做出正确的判断。所以引入了 Pause 根容器，以 pause 根容器的状态来代表 pod 的状态。例如：pod 是否死亡的问题。

Kubernetes 为每个 Pod 都分配了唯一的 IP 地址，称之为 Pod IP。一个 Pod 里的多个容器共享 Pod IP 地址。

Kubernetes 要求底层网络支持集群内任意两个 Pod 之间的 TCP/IP 直接通信。例如：flannel、Open vSwitch 等。因此，我们需要记住一个 pod 里的容器与另外主机上的 Pod 容器能够直接通信。

Pod 有两种类型：

1. 普通 Pod：Pod 被创建后，会被存放到 etcd 中，随后被调度到具体的 node 节点上运行。在默认情况下，当 pod 里的某个容器停止运行时，Kubernetes 会自动检车到这个问题并重新启动这个 pod，如果 pod 所在 node 机器出现宕机，就会将这个 node 上的 pod 重新调度到其他 node 上；
2. 静态 Pod：静态 Pod 被存放到某个具体的 Nod 上的具体的文件当中，并且只在该 node 上启动、运行，不能被调度到其他节点。例如：master 节点组件均是以静态 pod 的方式运行，文件存放目录为：`/etc/kubernetes/manifests/`。

另外，对于绝大多数容器来说，一个 CPU 的资源配额相当大，所以在 Kubernetes 里通常以千分之一的 CPU 配额为最小单位，用 m 来表示。通常一个容器的 CPU 配个被定义为 100 ～ 300m，即占用 0.1 ～ 0.3 个 CPU。

```
spec:
  containers:
-name:db
    image:mysql
    resources:
      requests:
        memory:"64Mi"
        cpu:"250m"
      limits:
        memory:"128Mi"
        cpu:"500m"
```

通常，Requests 被设置为一个较小的值，表示容器平时的工作负载情况下的资源需求，而 limits 设置为峰值负载情况下资源占用的最大值。

### Label（核心概念）

Label（标签）是 Kubernetes 系统中另外一个核心概念。一个 Label 是一个 key=value 的键值对，其中 key 与 value 由用户自己指定。Label 可以被附加到各种资源对象上，例如：Node、Pod、Service、RC 等。

一个资源对象可以定义任意数量的 Label，同一个 Label 也可以被添加到任意数量的资源对象上。通过给指定资源对象绑定 Label 从而实现资源对象的分组管理，方便 Kubernetes 进行资源分配、调度和部署等工作。

通过给指定的资源对象绑定一个或多个不同的 Label 来实现多维度的资源分组管理功能，以便灵活、方便地进行资源分配、调度、部署等管理工作。例如：

* 版本标签：`release:stable`、`release:beta`；
* 环境标签：`environment:dev`、`environment:qa`、`environment:production`。

通过 Label 为资源对象贴上相对应的标签，随后通过 Label Selector（标签选择器）查询和筛选拥有某些 Label 的资源对象。当前有两种 Label Selector 表达式：

1. 基于等式：`name=myweb`，`env != production`；
2. 基于集合：`name in (myweb，myweb1)`，`name not in (myweb，myweb1)`。

可通过多个 Label Selector 表达式的组合实现复杂的条件选择，多个表达式之间用“，”分隔，条件之间是“AND”关系，即同时满足多个条件。

```
# 该片段表示：选择标签 key值为 app，value值为 myweb的资源
selector:
app:myweb

# 该片段表示：选择标签 key值为 app且value值为 myweb
# 且 key 值为tier 且 value值 in （frontend，backend）
selector：
matchLabels:
app:myweb
matchExpressions:
-{key:tier,operator:In,values:[frontend,backend]}
# 或使用以下写法
-key:tier
    operator:In# In, NotIn, Exists and DoesNotExist
        values:["frontend","backend"]
```

### Replication Controller（不推荐）

Repliacation Controller （简称 RC），RC 是 Kubernetes 核心概念之一。简单的说，RC 定义了一个期望的场景，即声明某种 Pod 的副本数量在任何时刻都符合某一个预期值。所以 RC 的定义包括：

1. Pod 期待的副本数；
2. 用于筛选目标 Pod 的 Label Selector；
3. 当 Pod 的副本数量少于期望值时，用于创建 Pod 的模板；
4. RC 定义如下所示：

```
apiVersion: v1
kind:ReplicationController# 副本控制器 RC
metadata:
name:myhello-rc# RC名称，全局唯一
labels:
        name:myhello-rc
spec:
replicas:5# Pod副本期待数量
selector:
        name:myhello-pod
template:
        metadata:
          labels:
                name:myhello-pod
        spec:
          containers:# Pod 内容的定义部分
          -name:myhello#容器的名称
                image:long-xu/hello:1.0.0#容器对应的 Docker Image
                imagePullPolicy:IfNotPresent
                ports:
                -containerPort:80
                env:# 注入到容器的环境变量
                -name:env1
                  value:"k8s-env1"
                -name:env2
                  value:"k8s-env2"
```

我们定义一个 RC 后，Kubernetes 的 controller manager 组件会定时巡检目标 pod 的副本数量是否与预期数量一致。当实际副本数量大于预期数量则关闭一些目标 pod，当实际副本数量小于预期数量，则通过 RC 中 pod 的定义模板创建一些新的目标 Pod。

### Replica Set

Replica Set 是 Kubernetes1.2 版本中引入的一个新概念，Replica Set 为 RC 的升级版本。二者的区别在于：RC 的 Label Selector 只支持基于等式的表达式，而 RS 的 Label Selector 支持基于集合的表达式；在线编辑 RS 后，RS 会自动更新 Pod，而 RC 的修改不会自动更新现有 Pod。

RC（Replica Set）作用：

1. 通过定义 RC/RS 来自动创建 pod 以及 pod 副本数量的自动控制；
2. 通过 Label Selector 机制筛选目标 pod；
3. 通过改变 RC/RS 所定义的副本数量来实现 Pod 的所对应服务的伸缩；
4. 通过改变 RC/RS 模板中的镜像，可以实现 Pod 的升级操作。

### Deployment（重要概念）

Deployment 是 Kubernetes 在 1.2 版本中引入的概念，用于更好的解决 pod 的部署、升级、回滚等问题。

Deployment 内部会自动创建 RS 用于 Pod 的副本控制。Deployment 相较于 RC/RS 有以下优势：

1. Deployment 资源对象会自动创建 RS 资源对象来完成部署，对 Deployment 的修改并发布会产生新的 RS 资源对象，为新的发布版本服务；
2. Deployment 支持查看部署进度，以确定部署操作是否完成；
3. 更新 Deployment，会触发部署从而更新 pod，而 RC 的更新不会自动触发 pod 的更新。RS 可以触发 pod 更新；
4. Deployment 支持 Pause 操作，暂停之后对 Deployment 的修改不会触发发布动作。当完成修改之后可以通过 Resume 操作来发布新的 Deployment；
5. Deployment 支持回滚操作，可以回滚到上一个版本或者回滚到指定的发布版本；
6. Deployment 支持重新启动操作，重新启动会触发 Pod 的更新；
7. Deployment 会自动清理不再需要的 RS。

### Horizontal Pod Autoscaler

Horizontal Pod Autoscaler（简称 HPA），其主要作用是用于 Pod 横向自动伸缩。根据追踪和分析指定 RC/RS 控制的所有目标 Pod 的负载变化情况，来确定是否需要针对性地调整目标 Pod 的副本数量。

### StatefulSet

在 Kubernetes 系统中，Pod 的管理对象 RC、Deployment、DaemonSet 和 Job 等都是面向无状态的服务。但现实中有很多服务是有状态的，StatefulSet 就是用来管理有状态应用的 Pod 的。和 Deployment 类似，StatefulSet 也是通过 Label Selector 来管理一组相同定义的 Pod。但和 Deployment 不同的是，StatefulSet 为它的每一个 Pod 都维护了一个唯一的 ID。虽然每一个 pod 都是基于相同的定义文件而创建的，但是它们之间不能相互替换：无论怎么调度，每个 pod 都有一个永久不变的 ID。

哪些情况下应该使用 StatefulSet:

1. 需要稳定的、唯一的网络标识的应用程序；
2. 需要稳定的持久化存储的应用程序；
3. 需要有序的部署、更新和缩放的应用程序。

有哪些限制：

1. Pod 的存储必须由 PersistentVolume 驱动；
2. 删除或收缩 StatefulSet 不会删除关联的存储卷；
3. 需要使用 Headless Service 来负责 Pod 的网络标识，因此需要创建 Headless Service；
4. 删除 StatefulSet 时，不能保证删除它管理的 Pod。可以通过调整副本数量为 0 来实现。

常见的有状态应用程序有：MySQL 集群、MongoDB 集群、kafka 集群等。

### DaemonSet

DaemonSet 确保全部（或者某些）节点上运行一个 Pod 的副本。 当有节点加入集群时， 也会为他们新增一个 Pod 。 当有节点从集群移除时，这些 Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。

DaemonSet 的一些典型用法：

* 在每个节点上运行集群守护进程；
* 在每个节点上运行日志收集守护进程；
* 在每个节点上运行监控守护进程。

一种简单的用法是为每种类型的守护进程在所有的节点上都启动一个 DaemonSet。 一个稍微复杂的用法是为同一种守护进程部署多个 DaemonSet；每个具有不同的标志， 并且对不同硬件类型具有不同的内存、CPU 要求。

### Service（重要概念）

Service 服务也是 Kubernetes 里的核心资源对象之一，其主要作用是将运行在一组 Pods 上的应用程序公开为网络服务，这其实就是我们经常提起的微服务。通过 service 资源对象定义一个 service 的访问入口地址。前端应用通过访问这个入口，从而访问其背后的一组有 Pod 副本组成的集群实例。service 所针对的目标 Pods 集合通常通过 Label Selector 来确定，如下图：

Service 一旦被定义，就被分配了一个不可变更的 Cluster IP，在整个 Service 的生命周期内，该 IP 地址都不会发生改变。

### Job

Job 即工作任务，Job 会创建一个或者多个 Pods，来执行工作任务。Job 会跟踪记录成功完成的 Pod 数量，当成功完成的数量达到了指定的成功个数时，Job 结束。当执行过程中 Pod 出现失败的情况，Job 会创建新的 Pod 来替代该 Pod。 删除 Job 的操作会清除所创建的全部 Pods。 挂起 Job 的操作会删除 Job 的所有活跃 Pod，直到 Job 被再次恢复执行。Job 通常为单次任务，如果需要运行定时 Job 则应该使用 CronJob。

### Volume

Volume 是 k8s 抽象出来的对象，用于解决 Pod 中的容器运行时，文件存放的问题以及多容器数据共享的问题。Pod 可以同时使用任意数量的 Volume 类型。 临时卷类型的生命周期与 Pod 相同，但持久卷的生命周期与 Pod 无关。 当 Pod 不再存在时，Kubernetes 会销毁临时卷；不过 Kubernetes 不会销毁持久卷。 对于给定 Pod 中任何类型的卷，在容器重启期间数据都不会丢失，也就是说数据卷的生命周期与容器无关。Volume 的核心是一个目录，其中可能存有数据，Pod 中的容器可以访问该目录中的数据。 所采用的特定的 Volume 类型将决定该目录如何形成的、使用何种介质保存数据以及目录中存放的内容。

Volume 支持多种 Volume 类型，例如：cephfs、configMap、csi、downwardAPI、emptyDir、fc（fibre channel）、gcePersistentDisk、glusterfs、hostPath、iscsi、local、nfs、persistentVolumeClaim、projected、secret、rbd 等。

Volume 的使用比较简单，通常情况下，我们在 Pod 上声明一个 Volume，然后再容器里引用该 Volume 并挂载到容器的某个目录上即可，例如：

```
template：
  metadata：
        labels:
          app:myapp
spec:
        volumes:
          -name:datavol
                emptyDir:{}
        containers:
        -name:nginx
          image:nginx
          volumeMounts:
                -mountPath:/mydata
                  name:datavol
```

### Persistent Volume

持久卷（PersistentVolume，PV）是集群中的一块存储，可以由管理员事先供应，或者 使用存储类（Storage Class）来动态供应。 持久卷是集群资源，就像节点也是集群资源一样。PV 持久卷和普通的 Volume 一样，也是使用 Volume 插件来实现的，只是它们拥有独立的生命周期。

Pod 通过 PersistentVolumeClaim（PVC）来申领 PV 作为存储卷使用。集群会通过 PVC 找到其绑定的 PV，并将该 PV 挂载到 Pod。

下面是一个 NFS 类型的 PV 的一个 YAML 定义文件，声明了 8Gi 的存储空间：

```
apiVersion: v1
kind:PersistentVolume
metadata:
name:pv0001
spec:
capacity:
        storage:8Gi
accessModes:
        -ReadWriteOnce
nfs:
        path:/somepath
        server:192.168.0.106
```

PV 的 accessModes 属性：

1. ReadWriteOnce：读写权限，只允许被单个 Node 挂载；
2. ReadOnlyMany：只读权限，允许被多个 Node 挂载；
3. ReadWriteMany：读写权限，允许被多个 Node 挂载。

如果某个 Pod 需要申请某种类型的 PV，则需要先定义一个 PersistentVolumeClaim 对象：

```
apiVersion: v1
kind:PersistentVolumeClaim
metadata:
name:myclaim
spec:
accessModes:
        -ReadWriteOnce
resources:
        requests:
          storage:8Gi
```

然后，在 Pod 的 Volume 定义中引用 PVC 即可：

* volumes: name: mypd persistentVolumeClaim: claimName: myclaim

最后说说 PV 的状态；PV 的状态包括以下几种：

1. Available：空闲状态；
2. Bound：已经绑定到某个 PVC 上；
3. Released：对应的 PVC 已经被删除，但资源还没有被集群回收；
4. Failed：PV 自动回收失败。

### Namespace（重要的概念）

Namespace（命名空间）是 Kubernetes 系统中的另一个非常重要的概念，主要提供资源隔离。通过命名空间可以将同一个集群中的资源划分为相互隔离的组。统一命名空间下的资源名称必须唯一，但是不同命名空间下的资源名称可以一样。命名空间的作用仅针对那些带有命名空间的资源对象，例如：

Deployment，service，pod，rc 等。对集群对象不适用，例如：Node，namespace，PV 等。

### Annotation

Annotation（注解）与 Label 类似，也是使用 key/value 键值对的形式进行定义。不同的是 Label 义的是 Kubernetes 对象的元数据（metadata），并且用于 Label Selector。Annotation 则是用户任意定义的附加信息，以便于外部工具查找。

### ConfigMap

ConfigMap 用于将其他资源对象所需要使用的非机密配置项的数据保存在键值对中。例如：应用程序的配置文件。如此一来，就可以集中管理集群所使用的所有配置项。

### Secret

Secret 与 ConfigMap 类似，但是 Secret 是专门用来存储机密数据的。

## Kubernetes 集群搭建

### Kubernetes 部署方式介绍

社区当前有很多部署 Kubernetes 的方案和工具，每种方案和工具都有其自身的特点、优缺点和适用场景。我们可以根据需要选择合适的方式来部署 Kubernetes 集群。当前社区常用的部署工具有：minikube、kind、kubeadam、k3d、microk8s、二进制部署、Kops、Kubespray、Terraform 等。其中，用的最多的是 minikube、kind、kubeadam 和二进制部署。当然，如果你舍得花钱，也可以直接在云服务提供商一键创建一个生产级的 Kubernetes 集群。

上面列举的各种部署方式中，kind 部署方式最简单、便捷。本套课程的目的是介绍 Kubernetes 调度器相关的技术。为了减轻你的学习压力，本小节，我使用 kind 来教你如何快速部署一个 kind 集群。

### Kind 集群部署

#### kind 命令安装

Kind 官网提供了多种安装方式，详见：Quick Start-Installation[3]。这里我选择平台无关的二进制文件安装方式。安装命令如下：

```
$$ [ $$(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.19.0/kind-linux-amd64
$ chmod +x ./kind
$$ mv kind $$HOME/bin
$$ kind completion bash > $${HOME}/.kind-completion.bash # 配置 kind bash 自动补全
$$ $${ONEX_ROOT}/scripts/add-completion.sh kind bash # 需要 clone onex 项目，并配置 onex 项目根路径变量 ONEX_ROOT
$ kind version # 如果 kind version 能够输出 kind 版本号，说明创建成功。
kind v0.19.0 go1.20.4 linux/amd64
```

当前 kind 最新版本为 v0.22.0，这个版本我在测试过程中，会遇到创建集群失败的问题。所以，这里我下载安装了经过我测试过的 v0.19.0 版本。如果你感兴趣，可以升级版本到 v0.22.0 自行创建集群测试。

如果你克隆了 onex 项目仓库，还可以选择执行以下 Makefile 规则来快速安装：

```
$ make tools.install.kind
```

#### kind 集群配置

Kind 提供了丰富的配置选项，有集群级别的，也有节点级别的。Kind 缺省使用 kubeadm 创建和配置 Kubernetes 集群，通过 Kubeadm Config Patches 机制提供了针对 Kubeadm 的各种配置。Kind 配置详情可参考其官方文档：Configuration[4]。

一般的开源项目或工具，都会在项目仓库中，存放一个示例配置文件，但是 kind[5] 仓库没有。所以这里我阅读了Configuration[6]文档后，将所有的配置项，整理到一个配置文件中，并给你解释每个配置项的作用，所有的配置项内容及释义如下：

```
# 提示：本 Kind 集群配置，可在 kind v0.19.0 版本下正常工作
kind:Cluster
apiVersion:kind.x-k8s.io/v1alpha4
# Kind 集群名字
name:onex
featureGates:
# 开启 CSIMigration 特性。这个特性在 Kubernetes 开发课程中，用不到
# Kubernetes 支持的所有 FeatureGate 请参考 https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/
# "CSIMigration": true
# 用来配置 kube-apiserver 启动参数。所有的启动参数可参考 https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/
runtimeConfig:
"api/alpha":"false"
networking:
# 绑定到宿主机上的地址，如果需要外部访问请设置为宿主机 IP
# 注意：这里需要设置为你的宿主机 IP 地址
apiServerAddress:10.37.83.200
# 绑定到宿主机上的端口，如果建多个集群或者宿主机已经占用需要修改为不同的端口
apiServerPort:16443
# 设置 Pod 子网
# 默认情况下，kind 对 IPv4 使用 10.244.0.0/16 pod 子网，对 IPv6 使用 fd00:10:244::/56 pod 子网。
podSubnet:"10.244.0.0/16"
# 设置 Kubernetes Service 子网
# 默认情况下，kind 对 IPv4 使用 10.96.0.0/16 服务子网，对 IPv6 使用 fd00:10:96::/112 服务子网。
serviceSubnet:"10.96.0.0/12"
# 设置集群网络模式为双栈，ipFamily 可用取值为 dual, ipv6, ipv4
ipFamily:dual
# 是否使用默认的 CNI 插件 kindnet
# 你可以禁用默认的 CNI 插件，安装自己的 CNI 插件，这里我们使用默认的 CNI 插件
disableDefaultCNI:false
# kube-proxy 使用的网络模式，none 表示不需要 kube-proxy 组件
kubeProxyMode:"ipvs"
# kubeadm 配置设置，以 Patch 方式来设置
# 可以设置 InitConfiguration, ClusterConfiguration, KubeProxyConfiguration, KubeletConfiguration
# 详细的 kubeadm 配置见：https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/control-plane-flags/
kubeadmConfigPatches:
-|
  kind: ClusterConfiguration
  networking:
    dnsDomain: "onex.io"
-|
  apiVersion: kubelet.config.k8s.io/v1beta1
  kind: KubeletConfiguration
  # 开启 imageGC，防止磁盘空间被 image 占满
  # imageGCHighThresholdPercent: 80 # NOTICE: 该选项慎开启，可能导致创建多 worker 节点的集群失败
  #evictionHard: # 不要打开，否则 nodes 可能 NotReady
  #nodefs.available: "0%"
  #nodefs.inodesFree: "0%"
  #imagefs.available: "90%"
nodes:
# master 节点列表。一个列表元素，代表一个 Kubernetes 节点
-role:control-plane
# 自定义节点使用的镜像及版本
image:kindest/node:v1.28.0@sha256:dad5a6238c5e41d7cac405fae3b5eda2ad1de6f1190fa8bfc64ff5bb86173213
kubeadmConfigPatches:
-|
    kind: ClusterConfiguration
    apiServer:
        extraArgs:
          # 自动创建命名空间
          enable-admission-plugins: NamespaceAutoProvision,NamespaceExists,NamespaceLifecycle
# 宿主机和节点文件共享挂载
extraMounts:
    # 宿主机目录
-hostPath:/kind/onex
    # 节点数据目录
    containerPath:/data
    # 是否以只读方式挂载
    readOnly:false
    # 是否重新生成 SELinux 标签
    selinuxRelabel:false
    propagation:HostToContainer
    # 节点端口到宿主机端口映射
extraPortMappings:
    # 节点端口 nodeport
-containerPort:32080# 对应到 traefik web.nodePort
    # 宿主机端口
    hostPort:18080
    # 宿主机端口监听地址，需要外部访问设置为"0.0.0.0"
    listenAddress:"0.0.0.0"
    # 通信协议
    protocol:TCP
-containerPort:32443# 对应到 traefik websecure.nodePort
    hostPort:18443
    listenAddress:"0.0.0.0"
    protocol:TCP
# worker 节点，配置同 master 节点
-role:worker
labels:
    # 设置节点标签
    nodePool:onex
image:kindest/node:v1.28.0@sha256:dad5a6238c5e41d7cac405fae3b5eda2ad1de6f1190fa8bfc64ff5bb86173213
-role:worker
image:kindest/node:v1.28.0@sha256:dad5a6238c5e41d7cac405fae3b5eda2ad1de6f1190fa8bfc64ff5bb86173213
-role:worker
image:kindest/node:v1.28.0@sha256:dad5a6238c5e41d7cac405fae3b5eda2ad1de6f1190fa8bfc64ff5bb86173213
```

上面是 Kind 支持的几乎全部的配置，其中有一些配置项可以满足一些重要的、常见的功能需求，接下来，我再给你讲解下。

##### 端口映射

在测试、开发的过程中，我们在 Kinud 集群中部署了一个 Deployment，并为该 Deployment 创建了一个 NodePort 类型的 Service，但是我们发现，在宿主机上无法访问 Service 中配置的 nodePort 端口，进而访问 Pod 端口。这是因为：Service 中配置的 nodePort 是监听在 Kubernetes 节点容器上的，而非宿主机。

Kind 支持 extraPortMappings 配置项，用来将宿主机的端口映射到 Kubernetes 节点容器上的某个端口，可以通过 extraPortMappings 配置，来实现宿主机访问 Kubernetes 节点容器端口，进而访问 Kubernetes Pod 端口的目的。具体配置如下：

* extraPortMappings: # 节点端口 nodeport containerPort: 32080 # 对应到 traefik web.nodePort containerPort: 32443 # 对应到 traefik websecure.nodePort hostPort: 18443 listenAddress: "0.0.0.0" protocol: TCP

##### 暴露 kube-apiserver

在测试、开发的过程中， 有时候我们可能想在 B 机器上访问 A 机器上的 Kind 集群，这时候，你会发现访问不通。这是因为，默认情况下，Kind 集群的 kube-apiserver 是监听在 A 机器上的 lo 网络设备上的，并且监听端口也是随机的。如果你想从 B 机器上访问，就需要使 Kind 集群的 kube-apiserver 监听在可访问网络设备的（例如：eth0）的某个端口上。

Kind 提供了 apiServerAddress 和 apiServerPort 来满足此种场景的需求。具体配置如下：

```
networking:
  # 绑定到宿主机上的地址，如果需要外部访问请设置为宿主机 IP
  # 注意：这里需要设置为你的宿主机 IP 地址
  apiServerAddress: 10.37.83.200
  # 绑定到宿主机上的端口，如果建多个集群或者宿主机已经占用需要修改为不同的端口
  apiServerPort: 16443
```

##### 启用 Feature Gates

Kubernetes 支持 Feature Gates，我们可以开启或关闭 Feature Gates 来开启或关闭某些功能。Kind 也提供了开启或关闭 Feature Gates 功能的配置，具体配置如下：

```
featureGates:
  # 开启 CSIMigration 特性。这个特性在 Kubernetes 开发课程中，用不到
  # Kubernetes 支持的所有 FeatureGate 请参考 https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/
  # "CSIMigration": true
```

#### kind 常用操作

安装完 kind 命令后，可以执行 kind -h 来查看 kind 工具支持的命令：

```
$ kind -h
kind 使用 Docker 容器 'nodes' 创建和管理本地 Kubernetes 集群
Usage:
  kind [command]
Available Commands:
  build       构建 Node 镜像
  completion  为指定的 shell（bash、zsh 或 fish）输出 shell 补全代码
  create      创建 Kind 集群
  delete      删除 Kind 集群
export      导出集群 kubeconfig 配置和集群日志
  get         获取 Kind 集群列表、指定集群的 Node 列表、指定集群的 kubeconfig
help        打印帮助信息
  load        将宿主机 Docker 镜像加载到 Kind 集群的 Node 节点
  version     打印 kind 命令行工具版本
Flags:
  -h, --help              打印帮助信息
      --loglevel string   DEPRECATED: see -v instead
  -q, --quiet             不输出错误到标准错误
  -v, --verbosity int32   设置 log 日志级别
      --version           打印 kind 命令行工具版本
使用 "kind [command] --help" 了解更多关于命令的信息
```

接下来，我会给你详细介绍下常用的 kind 命令操作。

##### 创建 Kind 集群

可以执行以下命令快速创建一个带默认配置的 Kind 集群：

```
$ kind create cluster -n test-k8s
```

kind create cluster 提供了一些参数，参数如下：

```
$ kind create cluster -h
创建一个本地测试用的 Kubernetes 集群

用法：
  kind create cluster [flags]

标志：
      --config string       指定创建 Kind 集群时的配置文件
  -h, --help                打印 kind create cluster 的帮助信息
      --image string        指定创建 Kind 集群节点的 docker 镜像
      --kubeconfig string   设置创建出的 Kind 集群的 kubeconfig 文件保存路径，替代 $KUBECONFIG 或 $HOME/.kube/config 所指定的默认路径
  -n, --name string         设置集群名称，可以覆盖 KIND_CLUSTER_NAME、配置文件中指定的集群名字
      --retain              在集群创建失败时，保留节点以进行调试
      --wait duration       等待，直到控制面节点准备就绪 (默认 0s)
```

上面的配置文件，在使用 kind 命令时，都会被经常用到。

##### 查询 Kind 集群

可以使用 kind get 命令查询集群信息，具体可以查询以下集群信息：

* 获取 Kind 集群列表；
* 获取指定集群的 Node 列表；
* 获取指定集群的 kubeconfig。

具体命令如下：

```
# 获取所有的 Kind 集群
$ kind get clusters
onex
test-k8s
# 查询名为 test-k8s Kind 集群的节点列表
$ kind get nodes -n test-k8s
test-k8s-control-plane
# 查询所有 Kind 集群的节点列表
$ kind get nodes -A
onex-control-plane
onex-worker
onex-worker3
onex-worker2
test-k8s-control-plane
# 获取名为 test-k8s Kind 集群的 kubeconfig 文件内容
$ kind get kubeconfig -n test-k8s
apiVersion: v1
clusters:
- cluster:
...
```

kind get clusters 命令，在实际开发中最经常被用到。其他查询命令使用的较少。

##### 导出 Kind 集群的 kubeconfig 文件

使用 kind export 命令，可以导出集群的 kubeconfig 文件和日志。具体命令如下：

```
# 导出名为 test-k8s Kind 集群的 kubeconfig，并保存到 /tmp/test-k8s 文件中
$ kind export kubeconfig -n test-k8s --kubeconfig /tmp/test-k8s
Set kubectl context to "kind-test-k8s"
# 导出名为 test-k8s Kind 集群的日志到 test-k8s-log 目录中
$ kind export logs -n test-k8s test-k8s-log
Exporting logs for cluster "test-k8s" to:
test-k8s-log
```

kind export logs 命令，会将我们需要排障用的各类日志，例如：docker 和 kind 的版本、kubelet 日志、containerd 日志、标准输出、镜像列表、journald 日志、容器日志等导出到指定的目录中。

kind export kubeconfig、kind export logs 2 个命令，在测试开发中，经常被用到。

##### 导入镜像到 Kind 节点容器

在 Kubernetes 开发中，我们通常会在开发机上使用 docker build 命令构建需要测试的组件镜像，例如：ccr.ccs.tencentyun.com/superproj/onex-usercenter-amd64:v0.1.0。构建的镜像保存在宿主机上，并没有保存在 Kubernetes 节点容器中。我们在节点容器中创建 Pod 加载的镜像是节点容器中的镜像。所以，我们还需要将宿主机上的 Docker 镜像加载到节点容器中。

加载命令如下：

```
# 将 ccr.ccs.tencentyun.com/superproj/onex-usercenter-amd64:v0.1.0 镜像加载到名为 onex Kind 集群的所有节点上
$ kind load docker-image --name onex ccr.ccs.tencentyun.com/superproj/onex-usercenter-amd64:v0.1.0
```

上述命令会将指定的镜像加载到所有的 Kind 集群节点中，kind load docker-image 还提供了 --nodes 参数，该参数可以将镜像加载到指定的节点容器中（逗号分裂，例如：--nodes onex-worker,onex-worker2），例如：

```
$ kind load docker-image --name onex --nodes onex-worker,onex-worker2 ccr.ccs.tencentyun.com/superproj/onex-usercenter-amd64:v0.1.0 --nodes onex-worker
```

##### 删除集群

可以执行以下命令来删除一个或多个 Kind 集群：

```
# 删除名为 onex 的 Kind 集群
$ kind delete cluster -n onex
# 删除所有的 Kind 集群
$ kind delete clusters -A
```

## 往期文章回顾

* [从 Kubernetes 学习 Go 接口封装](https://mp.weixin.qq.com/s?__biz=Mzk0MTY1NDczMA==&mid=2247485850&idx=1&sn=c519b7ea70cf84493397dd697c06db06&token=103780397&lang=zh_CN&scene=21#wechat_redirect)
* [Kubernetes 编程实战课开始更新了，来看看有什么？](https://mp.weixin.qq.com/s?__biz=Mzk0MTY1NDczMA==&mid=2247485835&idx=1&sn=29cab938d9c5efb17752fee789ff2da3&token=103780397&lang=zh_CN&scene=21#wechat_redirect)
* [容器/Kubernetes开发者工作内容有哪些？](https://mp.weixin.qq.com/s?__biz=Mzk0MTY1NDczMA==&mid=2247485577&idx=1&sn=5b61d8894cf2259984141a01d6e365d3&token=103780397&lang=zh_CN&scene=21#wechat_redirect)
* [从 Kubernetes 学习大规模 Go 项目架构](https://mp.weixin.qq.com/s?__biz=Mzk0MTY1NDczMA==&mid=2247485442&idx=1&sn=affb99ea31cb7b0b938eed971c0603ae&token=103780397&lang=zh_CN&scene=21#wechat_redirect)
* [Kubernetes 节点自动伸缩（Cluster Autoscaler）原理与实践](https://mp.weixin.qq.com/s?__biz=Mzk0MTY1NDczMA==&mid=2247485435&idx=1&sn=d68a77d6958801a5697e698ede1f41b6&token=103780397&lang=zh_CN&scene=21#wechat_redirect)
* [你真的了解学习 Kubernetes 带来的职业价值吗？](https://mp.weixin.qq.com/s?__biz=Mzk0MTY1NDczMA==&mid=2247485163&idx=1&sn=930025ed75d6fe42edc0cc15e7edba7c&token=103780397&lang=zh_CN&scene=21#wechat_redirect)
* [带你从0到1部署一个功能完备、生产可用的Kubernetes集群](https://mp.weixin.qq.com/s?__biz=Mzk0MTY1NDczMA==&mid=2247484939&idx=1&sn=b2bb3e0c429ec8fb79e858c38ea68446&token=103780397&lang=zh_CN&scene=21#wechat_redirect)
* [昨天更新完一套K8S源码剖析课，来看看学习K8S源码能带给我们什么？](https://mp.weixin.qq.com/s?__biz=Mzk0MTY1NDczMA==&mid=2247485829&idx=1&sn=13007f4bc76cf2b53780d62a62ed94c2&token=103780397&lang=zh_CN&scene=21#wechat_redirect)
* [Go 为何天生适合云原生？](https://mp.weixin.qq.com/s?__biz=Mzk0MTY1NDczMA==&mid=2247485367&idx=1&sn=5633e318b1bda067e645ab115ef87cd0&token=103780397&lang=zh_CN&scene=21#wechat_redirect)

参考资料

[1] 

controller\_names.go: *https://github.com/kubernetes/kubernetes/blob/v1.30.4/cmd/kube-controller-manager/names/controller\_names.go#L44*

[2] 

etcdhelper: *https://github.com/openshift/origin/tree/main/tools/etcdhelper*

[3] 

Quick Start-Installation: *https://kind.sigs.k8s.io/docs/user/quick-start/#installation*

[4] 

Configuration: *https://kind.sigs.k8s.io/docs/user/configuration/*

[5] 

kind: *https://github.com/kubernetes-sigs/kind*

[6] 

Configuration: *https://kind.sigs.k8s.io/docs/user/configuration/*