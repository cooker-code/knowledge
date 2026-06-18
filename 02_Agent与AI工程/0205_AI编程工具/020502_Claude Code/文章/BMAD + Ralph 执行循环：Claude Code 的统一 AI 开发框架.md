---
title: BMAD + Ralph 执行循环：Claude Code 的统一 AI 开发框架
author: AI灵感闪现
date: 
url: https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489763&idx=1&sn=3230665edbfcc6e4855c6f9f8e2687c7&chksm=a7106f9ab2906361dafe5c7d1a24065a1e643e133549e4688ad17306ded49251b0aadf998fee&mpshare=1&scene=24&srcid=0215U4Gyga8Y1C0of1o30yb6&sharer_shareinfo=837a8c277e76080262a624a1ff382abb&sharer_shareinfo_first=837a8c277e76080262a624a1ff382abb#rd
---

# 

用 Claude Code 写代码时，你可能遇到过这两种极端：

**"随缘编程"模式**：描述想要什么，AI 生成代码，复制粘贴。快是快了，但对话一长，AI 就开始"忘记"之前的决定，代码质量参差不齐。

**"过度仪式"模式**：安装一堆框架，配置无数模板，写大量文档。还没开始写代码，一下午就过去了。

BMAD 方法和 Ralph Wiggum 循环的结合提供了一条中间路线：结构化的开发阶段 + 自主的执行循环。你得到的是可追踪的决策记录和无需持续监督的执行能力。

> **我是 AI灵感闪现，使用 Claude Code + BMAD AI 驱动敏捷开发框架，让 AI 自主开发和交付软件来表达想法和灵感。****是 MoneyMind 省钱思维 App 和 HeartPetBond 心宠纽带 App 开发者。正在实践和分享让 AI 自主解决健康、生活、投资和等方面的问题。****我尽可能让 AI 自己完成从目标到交付以及演进的闭环，以最少的人为交互与监督，让 AI 自己跑流程。我只给 AI 想法或目标，全程不陪跑，让 AI 自主运行类似 Tesla FSD 自动驾驶。**

## 为什么需要统一框架

单独使用 BMAD 或 Ralph 都有局限：

| 框架 | 优势 | 短板 |
| --- | --- | --- |
| **BMAD** | 结构化流程、角色分工、可追溯工件 | 执行阶段需要人工监督 |
| **Ralph Wiggum** | 自主执行、迭代优化、无需干预 | 缺乏结构、容易偏离目标 |

把两者结合起来，BMAD 提供"做什么"和"为什么"，Ralph 提供"怎么做"的自主执行。

## 统一框架的四个阶段

### 阶段一：发现（Discovery）

**目标**：理解问题空间，收集上下文

BMAD 的 Analyst 代理负责这个阶段。它的工作是：

* 梳理业务需求和技术约束
* 识别利益相关者和成功标准
* 产出：Brief 文档

**Ralph 模式**：这个阶段不用 Ralph。发现需要人的参与和判断。

### 阶段二：规划（Planning）

**目标**：制定实现方案，拆分任务

BMAD 的 PM 和 Architect 代理介入：

* PM：产出 PRD（产品需求文档）
* Architect：产出技术架构设计
* 产出：PRD + Architecture Doc + Story List

**Ralph 模式**：仍然不用。规划需要架构决策，风险太高。

### 阶段三：执行（Execution）

**目标**：按 Story 实现代码

这是 Ralph Wiggum 大显身手的地方。

**配置 Ralph 执行循环**：

```
# 安装 Ralph Wiggum 插件  
/plugin install ralph-wiggum@claude-plugins-official
```

**执行模式**：

1. Ralph 读取 Story 和验收标准
2. 生成代码实现
3. 运行测试验证
4. 如果失败，自动调整并重试
5. 循环直到所有验收标准通过

### 阶段四：验证（Verification）

**目标**：端到端质量检查

BMAD 的 QA 代理 + Ralph 协同：

* QA：定义验证场景和边界用例
* Ralph：自动执行验证流程

## 实战案例：构建 API 网关

一个团队用这个统一框架重构了他们的 API 网关：

**阶段一（发现）**：2 小时

* Analyst 代理梳理了 15 个 API 端点的依赖关系
* 产出：Brief（12 页）

**阶段二（规划）**：4 小时

* PM 产出 PRD（8 页）
* Architect 产出架构设计（含 3 个备选方案的对比）
* Story List：12 个 Story

**阶段三（执行）**：3 天（无人值守）

* Ralph 循环执行 12 个 Story
* 平均每个 Story 3-5 轮迭代
* 自动生成测试覆盖率 87%

**阶段四（验证）**：4 小时

* QA 定义 45 个验证场景
* Ralph 自动执行，发现 3 个边界问题
* 自动修复 2 个，人工修复 1 个

**结果**：

* 代码行数：12,000+
* 人工介入时间：约 8 小时
* Bug 密度：0.3 个/千行（对比行业平均 15-50）

## 如何开始

### Step 1：安装 BMAD

```
# 项目目录下安装  
cd your-project  
npx bmad-method@alpha install
```

选择模块时，至少安装：

* Core（核心）
* BMM（BMAD Method Module）

### Step 2：安装 Ralph Wiggum

```
# Claude Code 中安装插件  
/plugin install ralph-wiggum@claude-plugins-official
```

### Step 3：配置执行循环

创建 `.bmad/ralph-config.yaml`：

```
# Ralph 执行配置  
max_iterations:20  
exit_conditions:  
-all_tests_pass  
-no_type_errors  
-build_success  
-coverage_above_80  
  
notification:  
on_complete:true  
on_iteration:false
```

### Step 4：运行第一个工作流

```
# 启动 BMAD Quick Dev  
/bmad:bmm:workflows:quick-dev  
  
# 在执行阶段，切换到 Ralph  
/ralph-loop start --story=Story-001
```

## 注意事项

### 何时不用 Ralph

不是所有任务都适合自动循环：

* **架构决策**：需要人的判断
* **安全相关代码**：需要人工审核
* **用户体验改动**：需要人工验证
* **外部 API 集成**：可能产生真实费用

### 成本控制

Ralph 循环会消耗 API 额度。设置防护：

```
# 成本限制  
cost_limit:  
  max_usd_per_task: 5.00  
  alert_at: 3.00
```

### 上下文管理

长循环会导致上下文膨胀。解决方法：

1. 每个 Story 完成后重启 Ralph
2. 使用 git 作为记忆层
3. 定期执行 `/compact` 压缩上下文

## 与其他框架的对比

| 框架 | 结构化程度 | 自主执行 | 学习曲线 |
| --- | --- | --- | --- |
| **BMAD + Ralph** | 高 | 高 | 中 |
| **纯 BMAD** | 高 | 低 | 中 |
| **纯 Ralph** | 低 | 高 | 低 |
| **GSD** | 中 | 中 | 低 |
| **Spec Kit** | 高 | 低 | 高 |

## 总结

BMAD 阶段 + Ralph 执行循环的组合，本质上是把软件开发分成两层：

1. **决策层**（BMAD）：人 + AI 协作，做判断、定方向
2. **执行层**（Ralph）：AI 自主，按既定规则迭代直到完成

这种分层让你在保持控制力的同时，获得自动化的效率提升。不是完全放手，也不是事必躬亲。

从小任务开始，验证流程，再逐步扩展到更大的项目。

---

**参考资源**：

* BMAD 方法 GitHub[1]
* Ralph Wiggum 插件[2]
* 规范驱动开发概述[3]

### 引用链接

[1]BMAD 方法 GitHub: *https://github.com/bmad-code-org/BMAD-METHOD*

[2]Ralph Wiggum 插件: *https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum*

[3]规范驱动开发概述: *https://www.thoughtworks.com/en-us/insights/blog/agile-engineering-practices/spec-driven-development-unpacking-2025-new-engineering-practices*

Claude Agent SDK 系列

[Claude Agent SDK 构建 AI Agent 实践：如何实现与上传文件的对话](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489575&idx=1&sn=31bd2ed68ba99d12c6ae16e3eea08f81&scene=21#wechat_redirect)

[Claude Agent SDK 构建 AI Agent 实践：服务端向 Claude Agent SDK 注入环境变量的实践](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489567&idx=1&sn=1b589f444911dcfb40e21b28b9ea2a14&scene=21#wechat_redirect)

[Claude Agent SDK + 微信小程序：AI Agent 项目实践复盘 2026-01-21](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489493&idx=1&sn=59f8dd794de6447e713298ae1f305330&scene=21#wechat_redirect)

[Claude Agent SDK + 微信小程序为个人打造 AI 分身：vs-ai-agents 项目技术实践复盘](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489492&idx=1&sn=dda6a2df2490d4c2ab6eff9fd4c9aeba&scene=21#wechat_redirect)

BMAD AI 驱动敏捷开发系列

[BMAD 突破性 AI 驱动敏捷开发框架：v6.0.0-alpha.23 升级体验：全新安装之旅](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489133&idx=1&sn=4973d5907ef08bb9ebd67a6eeda8f203&scene=21#wechat_redirect)

[BMAD Method 入门指南：用 Quick Dev 工作流更快、更稳地交付](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489101&idx=1&sn=f78aaea74ba3db553fbca71e83abe868&scene=21#wechat_redirect)

[实战测评：用 Claude Code + BMAD + GLM-4.7 打造 HeartPetBond App (心宠纽带)](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487840&idx=1&sn=53acb782e5b9eb1df9744f31745e60ab&scene=21#wechat_redirect)

[BMAD V6 安装配置完全指南：项目目录安装最佳实践](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486789&idx=1&sn=7bbfe9ef92964ef9be4c722b8d357991&scene=21#wechat_redirect)

[BMAD v6 安装更新：模块化 + AgentVibes “会说话”的开发体验](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486758&idx=1&sn=5d88d95112b626015377effd19227384&scene=21#wechat_redirect)

[用 Claude Code + BMAD AI 驱动敏捷，把一个想法变成 省钱思维 (MoneyMind) App](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486723&idx=1&sn=45dddd81a5d856ced875b4769cc54d09&scene=21#wechat_redirect)

[AI 时代的"文档屎山"？BMAD、Spec-Kit、OpenSpec 等面向文档AI编程的利弊](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486451&idx=1&sn=eeba9d266868b49759d855277e323562&scene=21#wechat_redirect)

[在 Codex 里像 Claude Code 一样用 BMAD：把多角色 AI 团队装进你的仓库](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486257&idx=1&sn=436d00899f6adcd45c962b47bdfad04e&scene=21#wechat_redirect)

[BMAD 突破性 AI 驱动敏捷开发框架：深度解析 26 个代理、68 个工作流和 655 个文件](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489141&idx=1&sn=1aa0e66547dcf30940b30b7132308271&scene=21#wechat_redirect)

每周总结

[【AI灵感闪现2026第2周】：从 App Store 上架到批量内容生产，思考软件开发和人生发展的底层逻辑](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489014&idx=1&sn=67bdce213a152c779c5f2a97c237f6aa&scene=21#wechat_redirect)

AI 自主开发 MoneyMind 省钱思维 App

[AI 自主开发 App 成功上架：历时 14 天审核，MoneyMind 省钱思维 App 今天发布了](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247488720&idx=1&sn=e5b4a2165194be07f642e5cadf10dc28&scene=21#wechat_redirect)

[MoneyMind 省钱思维 App 审核又被拒：粗心提交错误版本的惨痛教训](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487799&idx=1&sn=0e00268ba1aea481407848d1ea0e494b&scene=21#wechat_redirect)

[被苹果审核拒绝不要怕：这次用 Google Antigravity AI 快速修复 App Store 审核问题](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487193&idx=1&sn=18e5dee00ad05d815f5106d8ac7ae0c4&scene=21#wechat_redirect)

[Claude Code 自主开发 MoneyMind（省钱思维）iOS 应用送审 App Store](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487054&idx=1&sn=3d609b27f0c79c67b752b3645f6addc0&scene=21#wechat_redirect)

[用 Claude Code + BMAD AI 驱动敏捷，把一个想法变成 省钱思维 (MoneyMind) App](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486723&idx=1&sn=45dddd81a5d856ced875b4769cc54d09&scene=21#wechat_redirect)

AI 自主开发 HeartPetBond 心宠纽带 App

[全网首发？第一款 GLM 4.7 + Claude Code AI 自主开发的心宠纽带 App 首次通过 App Store 审核并上架发布](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489028&idx=1&sn=ade948ba611d1df9ae3c15cd9f692797&scene=21#wechat_redirect)

[智谱 GLM 4.7 模型 AI 自主开发 HeartBetBond 心宠纽带 App，从想法到提交 App Store 仅用 12 天](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247488932&idx=1&sn=8ace2b6e8d016d0628ca54c208e0e889&scene=21#wechat_redirect)

[实战测评：用 Claude Code + BMAD + GLM-4.7 打造 HeartPetBond App (心宠纽带)](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487840&idx=1&sn=53acb782e5b9eb1df9744f31745e60ab&scene=21#wechat_redirect)

## 加入 AI灵感闪现 微信群

长按下图二维码进入 AI灵感闪现 微信群

长按下图二维码添加微信好友 VibeSparking 加群

## 关注 AI灵感闪现 微信公众号