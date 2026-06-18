---
title: Claude 开发者平台 2026.05.11：AWS 账单 + Anthropic 全功能，绕开 Bedrock
author: 克劳德猎手
date: 克劳德猎手克劳德猎手
url: https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247484406&idx=1&sn=0d8fff9a66e4e24c734e8040619101cb&chksm=c3aa7962c94d129a2d2680f161cac7c6594f723aa6f93c932f6f00182446b244cc8262dcca0b&mpshare=1&scene=24&srcid=0513ya5ttFP838oprfAqHwS3&sharer_shareinfo=6ae3cbe6c657497e80bd4f9710584f02&sharer_shareinfo_first=6ae3cbe6c657497e80bd4f9710584f02#rd
---

|  |  |
| --- | --- |
| |  | | --- | | Anthropic 自己的 inference 走 AWS 账单——跟 Bedrock 互补，feature 跟第一方 API 同日上新。 | |

|  |
| --- |
| **2026 年 5 月 11 日**，Anthropic 开发者平台 release notes 只放出一条：**Claude Platform on AWS**公开上线。条目短得不像大版本，但点开文档一看是个大动作——这条路解决了"**AWS 计费 + Anthropic 全功能**"这个企业市场的老空白。下面拆开看：跟 Bedrock 怎么区分、怎么接入、能用什么、不能用什么。 |

这版要先打消一个误解——它**不是**Amazon Bedrock 的扩容版，也不是 Bedrock 的替代。两条路在 AWS 上并存，目标客户和能力边界各自不同。

|  |
| --- |
| 1这不是 Bedrock——三条路怎么分 |

AWS 上现在通过 Anthropic 调 Claude 一共三条路。先把关键差别列出来：

|  |  |  |
| --- | --- | --- |
| 维度 | Claude Platform on AWS（新） | Amazon Bedrock |
| 谁运维 inference | Anthropic（自己的 capacity pool） | AWS（AWS-controlled infra） |
| API surface | /v1/messages（Anthropic 原版） | /anthropic/v1/messages |
| Base URL | aws-external-anthropic.{region}.api.aws | bedrock-mantle.{region}.api.aws |
| Feature 上新 | 跟第一方 API 同日 | 按 AWS release schedule（常滞后 1–2 周） |
| `anthropic-beta` header | ✓ 支持透传 | ✗ 不支持 |
| Agent Skills / 沙盒 / Managed Agents | ✓ 可用 | ✗ 不可用 |
| 计费 | AWS Marketplace | AWS 原生服务 |
| 合规（HIPAA / FedRAMP / IL4–5） | ✗ 不可用 | ✓ 可用 |

读懂这张表的关键是**"谁是 inference 的数据处理方"**。Bedrock 是 AWS——所以能挂上 HIPAA、FedRAMP High、IL4/5 这些 AWS 合规招牌，但 feature 跟不上。Claude Platform on AWS 是 Anthropic——所以 inference 数据要按 Anthropic 的数据政策来，但能拿到跟[5/6 公开 Beta 的 Multiagent + Outcomes](https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247484375&idx=2&sn=15aff7c02e33dabf9260f8bee60745ca&scene=21#wechat_redirect)这类新功能完全同步的能力。

|  |
| --- |
| 🚫 法务和合规要先看这一条  官方文档原话："data may not reside in AWS; inference may route to Anthropic's primary cloud; and subservices may move under the hood without notice." Anthropic 是 inference 数据的处理方，AWS 只负责账单和身份元数据。要 inference 严格留在 AWS infra 里的——回 Bedrock；要数据严格留在 US 的——用`inference_geo: "us"`（1.1x 价钱）。 |

另一个关键点：这条路是**独立 capacity pool**。它跟第一方 Claude API、跟 Bedrock 都是分开的容量池——意味着可以三条路并存做 failover。三方都挂的时候，三个池子同时挂的概率比一个池子低。这个组合策略对生产环境跑大流量的团队值得规划进容灾。

|  |
| --- |
| 2接入流程：4 步 + 1 个必踩的坑 |

|  |  |
| --- | --- |
| 1 | AWS Console 订阅登 AWS Console，找**Claude Platform on AWS**service page，点 Sign up，过 EULA。AWS Marketplace 自动给你拉订阅，等几分钟。 |
| 2 | 初始化 Anthropic 组织跳转`platform.claude.com/partner-signup`，填组织信息——这里会创一个**新的 Anthropic 组织**，跟你已有的第一方 Anthropic 账号完全独立，API key 和 workspace 不互通。 |
| 3 | 建 workspace，记下 IDWorkspace 绑定单一 AWS region。ID 格式`wrkspc_*`，每个 API 请求都要带`anthropic-workspace-id`header 指向它。 |
| 4 | 开 outbound web identity federation（必做）这是**最常见的踩坑**——这项 STS 能力默认关闭，必须 IAM 显式启用一次。下面有命令。 |

|  |
| --- |
| AWS CLI · 开启 outbound web identity federation |
| ``` # 一次性启用（账号级） aws iam enable-outbound-web-identity-federation  # 验证 aws iam get-outbound-web-identity-federation-info  # 没启用就发请求，会一直返回： # "Outbound web identity federation is disabled for your account" ``` |

这一步的原理：网关在你后端跑`sts:GetWebIdentityToken`生成 JWT 转发到 Anthropic 那边。这个 STS 能力 AWS 默认关，需要 IAM 显式开。**第一次调 API 全部失败、错误信息又看着不像权限问题——多半就是这条没开**。

|  |
| --- |
| 3SDK + 认证：SigV4 是主路，API key 兜底 |

SDK 新加了`AnthropicAWS`client class（beta），Python / TypeScript / C# / Go / Java / PHP / Ruby 全语言覆盖。安装：

|  |
| --- |
| SHELL · 安装 SDK |
| ``` # Python（多了 [aws] extra） pip install -U "anthropic[aws]"  # TypeScript npm install @anthropic-ai/aws-sdk  # Go go get github.com/anthropics/anthropic-sdk-go ``` |

最小可跑的 Python 示例——客户端从环境变量读 region 和 workspace ID，认证走默认 AWS credential provider chain：

|  |
| --- |
| PYTHON · 第一次调用 |
| ``` export AWS_REGION='us-west-2' export ANTHROPIC_AWS_WORKSPACE_ID='wrkspc_01AbCdEf...'  # Python from anthropic import AnthropicAWS  client = AnthropicAWS()  # 从 env 读 region + workspace message = client.messages.create(     model="claude-sonnet-4-6",     max_tokens=1024,     messages=[{"role": "user", "content": "Hello!"}], ) ``` |

认证有两条路：

|  |
| --- |
| 💡 SigV4（主路）  走 AWS 默认 credential provider chain——env 变量、`~/.aws/credentials`、SSO、IRSA、EC2 IMDS 都通。SigV4 service 名是`aws-external-anthropic`，签名 region 必须和 endpoint URL 一致。企业部署 99% 走这条。 |

|  |
| --- |
| 💡 API key（兜底 + 短期 token）  脚本和本地开发可以用长期 API key，在**AWS Console**（不是 Claude Console）里生成，IAM 上挂`aws-external-anthropic:CallWithBearerToken`权限。LLM gateway / Lambda / 第三方工具不支持 SigV4 但支持 bearer token 的场景，可以用**token-generator**库从 AWS credential 生成 12 小时短期 token。 |

单次请求的 cURL 形态——核心是`--aws-sigv4`和`anthropic-workspace-id`header：

|  |
| --- |
| CURL · 不走 SDK 直接调 |
| ``` curl "https://aws-external-anthropic.us-west-2.api.aws/v1/messages" \   --aws-sigv4 "aws:amz:us-west-2:aws-external-anthropic" \   --user "$AWS_ACCESS_KEY_ID:$AWS_SECRET_ACCESS_KEY" \   -H "x-amz-security-token: $AWS_SESSION_TOKEN" \   -H "anthropic-version: 2023-06-01" \   -H "anthropic-workspace-id: $ANTHROPIC_AWS_WORKSPACE_ID" \   -H "content-type: application/json" \   -d '{"model": "claude-sonnet-4-6", "max_tokens": 1024,       "messages": [{"role": "user", "content": "Hello!"}]}' ``` |

注意`x-amz-security-token`只在用临时凭证（IAM role / SSO / STS）时需要——长期 IAM user 凭证不用带，否则会触发"通用签名拒绝"这种没法定位的报错。

|  |
| --- |
| 4能用什么 / 不能用什么 |

这条路本质是"Anthropic 全平台能力 + AWS 计费/认证层"，所以 feature 跟第一方 API**绝大部分对齐**。但有几样是 explicitly 不可用，企业部署时要先看清。

|  |
| --- |
| ▸ ✓ 全部可用 |
| Messages API · Files API · Message Batches API · Claude Managed Agents · Agent Skills · code execution · tool use（含 computer use）· extended thinking · streaming · prompt caching（5 分钟 + 1 小时 + 自动缓存）· beta headers（透传，跟第一方一致）· AWS PrivateLink（VPC 内联）。 |

|  |
| --- |
| ▸ ✗ 不可用（需要换路径） |
| **HIPAA-ready**（要 → Bedrock） **OAuth 认证**（要 → 第一方 Claude API） **Admin API 大部分端点**——只有 workspace 那一组端点可用；成员管理、API key 管理、usage / cost report 都不可用，靠 AWS IAM 顶上 **Spend limits**（用 AWS billing controls 顶） **Fast mode**、**OpenAI-compatible endpoint**都没有 **Claude Code 专属 workspace 和 Analytics API**没有——Claude Code 用量混在 general usage 里 |

还有一条容易踩——**Managed Agents 的自治 session 上限 6 小时**。第一方 API 上 Managed Agents session 可以无限制无人值守跑；这条路上跑了 6 小时没有 user 事件，session 就会要求重新认证（发任意 user-role 事件即可续命）。这是 AWS 侧的安全约束，长跑 agent 工作流要把这条规划进去。

· · ·

|  |
| --- |
| 写在最后 Anthropic 在企业市场又开了一条新路 |

放在 Anthropic 这两个月的脉络里看——[5/7 Claude 接进 Office 全家桶](https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247484375&idx=1&sn=f2c9fb18bcca01e9e61320553742e0ac&scene=21#wechat_redirect)、[5/6 Managed Agents 上 Multiagent + Outcomes](https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247484375&idx=2&sn=15aff7c02e33dabf9260f8bee60745ca&scene=21#wechat_redirect)、4/16 Bedrock 全面开放——这次 Claude Platform on AWS 是同一条主线的延伸：**把企业市场的不同采购入口都接上**。Office 走微软 AppSource 路径，AWS 走 Marketplace，Anthropic 自己的合同路径继续在。客户在哪个生态，就在哪个生态采购。

对采购方来说，这版的关键判断点不复杂：**合规优先选 Bedrock，feature 优先选这条新路**。两条路在 AWS 上并存，按业务需求选；要 failover 就同时挂。HIPAA、FedRAMP 那批合规招牌还会继续往 Bedrock 上挂，但中间一大批"想要 AWS 计费但又不想等 feature"的企业，从这版开始有了第三个选项。

技术上唯一要慎重的是**数据流向**——inference 不在 AWS 跑，data 可能 route 到 Anthropic 的 primary cloud。法务/合规给签字之前，`inference_geo`这个 per-request 参数和 ZDR（Zero Data Retention，需要联系 Anthropic 开）要先弄明白。

|  |
| --- |
| 开发者平台每一刀都给你拆开  下次更新接着盯  关注公众号「克劳德猎手」获取更多内容 👇 |

|  |
| --- |
| 📌 相关阅读 |
| [Claude 开发者平台 2026.05.06：Managed Agents 上新四件套](https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247484375&idx=2&sn=15aff7c02e33dabf9260f8bee60745ca&scene=21#wechat_redirect) |
| [Claude 接入 Office 全家桶：Excel、Word、PPT 正式可用，Outlook 进 Beta](https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247484375&idx=1&sn=f2c9fb18bcca01e9e61320553742e0ac&scene=21#wechat_redirect) |
| [Anthropic 推出 Managed Agents：Agent 开发不用自建沙箱，从原理到实操一篇通](https://mp.weixin.qq.com/s?__biz=MzkzNTY0NDA4Mg==&mid=2247483848&idx=1&sn=5b51e0da888e9a0ac2d72f1b01989287&scene=21#wechat_redirect) |