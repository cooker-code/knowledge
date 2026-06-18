> 已吸收至：[[07_工程与架构/0704_K8s&Docker容器化/0704_核心知识点/容器化核心知识点总览|容器化核心知识点总览]]
---
title: Docker正式发公告了，撤回之前的收费方案
author: AI产品阿颖
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg5Mjc3MjIyMA==&mid=2247561756&idx=1&sn=64df2bfa9228a5c8676a60b405bf406f&chksm=c03ab80ff74d3119249588fb1fd80065c14193ee565180777663df370ab15966d6fe8a63d26b&mpshare=1&scene=24&srcid=0325fpnCwCSGtw2mVcXfLz3u&sharer_sharetime=1679722299397&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

3 月 15 日，Docker 向所有创建了“组织”的 Docker Hub 用户发出电子邮件提醒，[称如果不升级至付费团队订阅，他们的账户和所有镜像都将被删除](http://mp.weixin.qq.com/s?__biz=Mzg5Mjc3MjIyMA==&mid=2247561656&idx=1&sn=9c10ada74e9f59e1cee462549f5072e7&chksm=c03abbabf74d32bdb06d18a4873bd9a518686b0e2d613cf578e72bf84fb50163a5d5a3a30e6e&scene=21#wechat_redirect)。这引发了各种批评，并且批评声音越来越激烈。最终，Docker 顶不住社区的压力，于 3 月 18 日[为其草率行为进行了道歉，并对之前的邮件进行了说明](http://mp.weixin.qq.com/s?__biz=Mzg5Mjc3MjIyMA==&mid=2247561671&idx=1&sn=f56157b7bfc40005706a96705ddfb9e9&chksm=c03abbd4f74d32c2e7f0a12461b9e22ad0af7bec71e3a8d8e7a55f33a37b7b43fb220ac2d41d&scene=21#wechat_redirect)。

当时 Docker 认为，之所以会出现批评，是因为他们与社区的沟通存在问题，他们的策略是完全正确的，他们还会按既定计划取消 Free Team 订阅。现在他们终于意识到，之前不管是沟通方式还是策略都是错误的。

今天 Docker 在其官网正式发公告，**撤回了之前取消取消 Free Team 订阅的方案**，下文是公告和 Free Team 订阅相关问题的说明。

在听取用户反馈并咨询社区后，我们清楚地意识到在取消 Free Team 订阅方面做出了错误的决定。上周，我们认为我们的沟通很糟糕，但我们的政策是正确的。**现在，我们清楚地认识到，无论是沟通还是政策都是错误的**，因此我们正式承诺，不再取消 Free Team 订阅：

* 如果你目前使用的是 Free Team 订阅，则无需再在 4 月 14 日之前迁移到另一个订阅。
* 在 3 月 14 日的停用公告和今天的公告之间从免费团队订阅升级到付费订阅的用户，将在接下来的 30 天内自动收到交易的全额退款，这将允许他们在购买期限内免费使用他们的新付费订阅。
* 请求迁移到 Personal 或 Pro 订阅的客户将保留其当前的 Free Team 订阅。（或者他们可以选择通过我们的网站开设一个新的个人或专业账户。）
* 在过去的 10 天里，我们收到并接受了比去年更多的 Docker-Sponsored Open Source program（DSOS）申请。我们鼓励符合条件的开源项目继续申请，我们将在几个工作日内处理这些申请。

相关问题如下。

**Docker 正在取消 Free Team 订阅吗？**

没有。Docker 于 2023 年 3 月 14 日发布了取消 Docker Free Team 订阅的决定，但该决定于 2023 年 3 月 24 日被撤销。

**其他 Docker 产品是否受到影响？**

不会。对 Docker Personal、Docker Pro、Docker Team、Docker Business、Docker-Sponsored Open Source、Docker Verified Publishers 或 Docker Official Images 都没有影响。

**Docker 还有开源项目吗？**

有。我们长期存在的 Docker-Sponsored Open Source（DSOS）订阅从未受到 Free Team 订阅（现已撤销）的影响，并且继续可用。我们建议符合条件的项目申请 DSOS 订阅，因为它提供的便利超过 Free Team 订阅。

**我怎么知道我是否在使用 Docker Free Team 订阅？**

请查阅你的 Docker 帐户的 组织页面（https://hub.docker.com/orgs），使用 Free Team 订阅的组织在 “订阅” 栏中标记为 “Docker Free Team”。不到 2% 的 Docker 用户拥有免费团队组织。

**为什么 Docker Free Team 现在不在定价页面上？**

我们在 2021 年将此订阅从注册选项中删除了。

**付费 Docker Team 订阅和 Docker Free Team 订阅有什么区别？**

付费 Docker Team 与 Docker Free Team 相比，主要优势有：

* 团队数量无限制（Free Team 只能有一个团队）
* 最多可以有 100 个席位（Free Team 最多 3 个席位）
* 无限的私有仓库（Free Team 中没有私有仓库）
* 每个用户每天可以进行 5,000 次拉取操作（Free Team 每个用户每 6 小时只能进行 200 次拉取操作）
* Free Team 订阅不包括在拥有超过 250 名员工或年收入超过 1000 万美元的公司中用于商业使用 Docker Desktop 的许可证。

**如果我超出了 Docker Free Team 订阅的限制怎么办？**

如果你需要的不仅仅是 Docker Free Team 提供的功能，你应该升级到 Docker Team 或 Docker Business。

**如果我请求从 Docker Free Team 迁移到 Docker Personal 或 Docker Pro 怎么办？**

我们不会迁移你的帐户，因为你现在可以继续使用当前的 Free Team 组织。我们的支持团队将回复所有未关闭的询问以确认此事。

**我在 3 月 14 日至 3 月 24 日期间从 Docker Free Team 升级到付费订阅，我什么时候可以收到退款？**

我们正在努力确定并退还我们在 3 月 14 日到 3 月 24 日的公告之间发生的所有此类交易。最多 30 天就能收到退款。

**我从 Docker Free Team 升级到付费订阅，但我在恢复到 Free Team 时遇到问题。**

请联系：https://hub.docker.com/support/contact/

**我可以从我的帐户中导出数据吗？**

可以。你可以从 Docker 注册表上的私有仓库中提取图像，并将这些图像推送到你选择的另一个注册表。

**Docker 是否删除了任何公共镜像或层？**

没有。Docker 从来没有打算删除或以任何方式限制对公共仓库的访问。开源项目维护者无需采取任何操作。仅当仓库的维护者决定从 Docker Hub 中删除它们时，公共镜像或层才会消失。

**其他人可以 “抢占” 命名空间吗？**

不能。Docker 没有发布命名空间，因此用户永远不会面临命名空间 “被占用” 的风险。

**这是否影响了 Docker Hub 上内容的安全性？**

不会。Docker 从来没有限制对公共仓库的访问。维护人员已经能够不间断地更新他们的仓库，并且继续完全拥有其仓库的所有权，以用于任何目的，例如修复 CVE。

推荐阅读：

* 《[比尔·盖茨：GPT是1980年以来最革命性的技术进步](http://mp.weixin.qq.com/s?__biz=Mzg5Mjc3MjIyMA==&mid=2247561718&idx=1&sn=d072b1e8cc0312b9a9fd8fcf7f6aff8b&chksm=c03abbe5f74d32f3ca8036f98bdd382520527c798604e61de25b31de4871c33e5aeb914fead8&scene=21#wechat_redirect)》
* 《[Docker发布其WebAssembly工具预览版第二版](http://mp.weixin.qq.com/s?__biz=Mzg5Mjc3MjIyMA==&mid=2247561750&idx=1&sn=ccfdba3bad79914f59a5358c03192dff&chksm=c03ab805f74d311317bcc8a3dd3d544a0379ac90d2aac38fdd6c02f7dd2ba794082e99a1c0a1&scene=21#wechat_redirect)》

---

分布式实验室策划的《[Kubernetes实战集训营](http://mp.weixin.qq.com/s?__biz=Mzg5Mjc3MjIyMA==&mid=2247561576&idx=1&sn=1813404e608ed2fcff2e56d0102a6c1a&chksm=c03abb7bf74d326dc7b890cde1f32754bddfd75ff0cd31585dd3b88d4470c4b00ef7e444c501&scene=21#wechat_redirect)》正式上线了。这门课程通过通过**5天线上培训，40个小时直播，15个随堂练习，50天课后辅导，把Kubernetes的70多个重要知识点讲给你，并通过实战帮你掌握Kubernetes**。培训重实战、重项目、更贴近工作，边学边练，5月13日正式开课。