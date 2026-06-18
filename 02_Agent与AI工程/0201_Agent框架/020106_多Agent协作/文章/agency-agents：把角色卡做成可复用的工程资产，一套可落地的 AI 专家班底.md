---
title: agency-agents：把角色卡做成可复用的工程资产，一套可落地的 AI 专家班底
author: Endorphins.cn
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI5NDU2MTI3NQ==&mid=2247485102&idx=1&sn=685f830c53d37bd442a9b0cffd23b84e&chksm=ed7e354da5c49a35cef7a8183b07fb354a76d23c630ed1511c2322a02abf41bb20e84f49ad0c&mpshare=1&scene=24&srcid=031093r1JELjatyOKb470qeM&sharer_shareinfo=3ac747660d343c4895b95cb7e82ac981&sharer_shareinfo_first=3ac747660d343c4895b95cb7e82ac981#rd
---

> msitarzewski/agency-agents 提供 55+ 领域化 AI Agent Markdown：人格、流程、交付物、指标齐全。本文从后端/DevOps视角拆解控制面/数据面、给出最小Demo与验收点、常见坑与上线前检查清单。

## 01｜它解决的不是写提示词，而是把交付流程固化成资产

msitarzewski/agency-agents（The Agency）本质上是一套可 fork 的 AI 专家库：每个 Agent 不是一句“扮演后端工程师”，而是一份结构化 Markdown，包含人格设定、工作流、交付物模板、示例代码与成功指标。

对后端/DevOps 团队来说，它的价值不在“让模型更聪明”，而在于把工程实践中的隐性知识（例如：蓝绿发布要带回滚、监控要有告警阈值、交付要有验收点）变成显式、版本化、可审计的文本资产。

---

## 02｜Repo 结构与 Agent 文件长什么样

仓库核心是分类目录 + Agent Markdown 文件。README 列了 9 个 Division、55+ Agent，例如工程侧的：

* Backend Architect
* DevOps Automator
* Reality Checker / Evidence Collector

单个 Agent 文件通常具备：

* YAML Frontmatter（name/description/color）
* Identity & Memory（角色、经验）
* Core Mission（默认职责）
* Critical Rules（必须遵守的工程规则）
* Technical Deliverables（交付物模板与代码片段）
* Workflow Process（步骤化流程）
* Success Metrics（量化指标）

以 `engineering/engineering-devops-automator.md` 为例，它不仅给出 CI 流水线骨架、IaC 片段、监控告警规则示例，还把“默认要求：监控、告警、自动回滚”写进规则层，属于典型的把最佳实践前置到角色卡。

---

## 03｜架构拆解：控制面 vs 数据面（从怎么用到怎么管）

控制面（Control Plane）：决定“用谁、怎么用、如何收敛输出”

* Agent 选择与组合：你决定用 DevOps Automator 还是 Backend Architect，或多 Agent 串联（设计→实现→验证）。
* 角色激活入口：README 推荐与 Claude Code 集成，把 Agent 文件放到 `~/.claude/agents/`，通过指令激活。
* 输出约束与质量门禁：Critical Rules / Deliverables / Success Metrics 就是控制面策略。
* 版本化与治理：把 Agent Markdown 当作代码（Git 管理、PR 审核、变更记录），实现团队级一致性。

数据面（Data Plane）：决定“具体干活怎么跑、上下文怎么流动”

* 输入数据：需求、现有代码/配置、日志与监控指标、环境约束。
* 执行载体：LLM +（可选）工具能力（文件读写、命令执行、代码修改）。
* 产出数据：PR 改动、CI 配置、IaC 模板、监控告警规则、Runbook、验收清单。
* 反馈闭环：Reality Checker/Evidence Collector 强调证据化，把结论落到可验证输出。

一句话：agency-agents 把控制面做成可配置资产，让数据面（真实工程上下文）更容易被正确加工。

---

## 04｜最小 Demo：用 DevOps Automator 把一个服务接入 CI/CD + 观测

目标：验证这套 Agent 库在后端/DevOps 场景的可用性。

前置条件：

* 本地可用 Claude Code
* 有一个最小服务仓库，至少包含 Dockerfile、基本测试命令、HTTP health endpoint

步骤 A｜安装 Agent（来自 README）

```
cp -r agency-agents/* ~/.claude/agents/
```

步骤 B｜激活 DevOps Automator

* 目标：为当前仓库补齐 CI/CD、镜像构建推送、部署策略与监控告警骨架，并给出回滚方案与验收标准。

步骤 C｜要求它按交付物模板输出
至少生成：

* `.github/workflows/ci.yml`
* `infra/terraform/`（或等价 IaC 目录）
* `monitoring/`
* `RUNBOOK.md`

三条验收点（必须可验证）

1. CI 可复现：新环境拉仓库后 CI 能跑通（测试命令与退出码标准明确）。
2. 可回滚：部署阶段存在回滚触发条件 + 回滚动作。
3. 可观测：至少 2 条告警规则（5xx 错误率、P95 延迟），阈值与处理建议明确。

---

## 05｜工程化落地建议：把角色卡纳入交付流水线

* 常用 Agent 固定到团队模板仓库：二次裁剪，加入你们的云/集群约束。
* 把 Success Metrics 改成真实 SLO/SLI：MTTR、发布频率、变更失败率、告警噪音比。
* 在 PR 模板中写“由哪位 Agent 产出/复核”，便于复盘。
* 引入证据化交付：测试/发布必须附日志片段、监控截图（或指标导出）。

---

## 06｜常见坑点：症状 / 原因 / 处理

坑 1｜输出完整但不适配你们环境

* 原因：角色卡是通用最佳实践，不知道你们的云厂商、网络拓扑、权限边界、镜像仓库、命名规范。
* 处理：先给环境事实表；要求输出可替换变量清单（REGISTRY\_URL/CLUSTER\_CONTEXT 等）。

坑 2｜CI/CD YAML 很长但关键门禁缺失

* 原因：你没把质量门禁写成硬约束。
* 处理：明确安全扫描/测试/发布三类门禁；写清楚必须失败即阻断的条件。

坑 3｜监控方案像文档落不了地

* 原因：缺少指标来源、采集方式、标签维度与阈值上下文。
* 处理：输出最小可用告警集（2~5 条）；附 PromQL 与 Runbook。

坑 4｜多 Agent 输出互相打架

* 原因：缺少统一系统约束，上下文漂移。
* 处理：先产出系统约束单页；最后用 Reality Checker 做一致性审计。

---

## 07｜上线前检查清单

交付物一致性

* CI 配置与本地脚本一致
* IaC 与运行时环境一致
* 变量与密钥通过 Secrets 管理，无明文落盘

发布与回滚

* 部署策略明确且有回滚路径
* 健康检查定义清晰并有超时策略
* 回滚触发条件写入流水线或 Runbook

可观测性

* 最小指标集：请求量、错误率、延迟（P95）
* 最小告警集：5xx/延迟/资源异常
* 日志具备结构化字段（service/env/version/request\_id）

可靠性与容量

* 关键依赖有降级策略
* 有容量与限流策略（并发、连接池、超时、重试上限）
* 有备份/恢复演练路径

变更治理

* PR 模板包含风险点、验证证据、回滚说明
* 关键配置变更有审阅人
* 发布后观察窗口与指标对比口径明确

---

## 08｜结语：把提示工程升级为流程工程

agency-agents 的亮点不在写得华丽，而在于把专业岗位的默认动作、交付物与指标体系写成可复制的 Markdown。

对后端/DevOps 而言，它更像一套工程流程脚手架：环境事实给准、验收门禁写硬，就能持续产出可落地的 CI/CD、IaC、监控告警与运行手册，并且能被团队复用、审阅与迭代。

**项目地址**
https://github.com/msitarzewski/agency-agents[1]

### 引用链接

[1]*https://github.com/msitarzewski/agency-agents*