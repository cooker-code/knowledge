---
title: 一天一个SKILL——fireworks-tech-graph Claude Code 指定带主题的画图神器 生成图
author: 考拉搞AI
date: AI高手AI高手
url: https://mp.weixin.qq.com/s?__biz=MzUyODE4MjQxOQ==&mid=2247484608&idx=1&sn=2a773858a986eac173b9fc18792c20d8&chksm=fb3d3595bb99e82ad33f50965edc54a754f27513bb93beb42db22f98a9d75026860d9b51ef73&mpshare=1&scene=24&srcid=0603vKul5OSG10iTNq2MPHlt&sharer_shareinfo=37e6d149aca374d0bf273e77fed5fcc3&sharer_shareinfo_first=37e6d149aca374d0bf273e77fed5fcc3#rd
---

写技术文档的人都有过这样的经历：系统架构在脑子里已经跑通了，但落到图上却要折腾半天；

`Mermaid` 写起来快，但节点一多就排版崩塌，偏流程类，复杂类型不太符合；

`draw.io` 拖出来的图好看，针对一些技术类、设计类比较方便实用，但是缺少色彩，页面风格比较规矩。 可以参考:[Claude Code + Next AI Draw.io：让AI替你画架构图，真正的“画图自由”](https://mp.weixin.qq.com/s?__biz=MzUyODE4MjQxOQ==&mid=2247484512&idx=1&sn=e29e2b16f0ccb27bed755e8f1e6e0fc6&scene=21#wechat_redirect)

今天要聊的这个组合——`Claude Code` + `Fireworks Tech Graph`，可能直接改变你对“画图”这件事的认知。

废话不多说，直接上干货。

## 一、是什么

2026 年 4 月，一位开发者发布了一个 GitHub 仓库，三天内拿下了 1,562 颗星，目前星数已突破 6,000。这个项目就是 `fireworks-tech-graph`，一个专为 `Claude Code` 设计的技术图表生成 Skill。实测下来，画一张复杂的 Mem0 内存架构图，时间从原本的 40 分钟压缩到 2 分钟，且输出比手绘更有风格一致性。

## 二、使用篇（实战驱动）

```
npx skills add https://github.com/yizhiyanhua-ai/fireworks-tech-graph --skill fireworks-tech-graph
```

[一天一个SKILL——前端最佳自动化测试 webapp-testing](https://mp.weixin.qq.com/s?__biz=MzUyODE4MjQxOQ==&mid=2247484243&idx=1&sn=959174474c36059e6721680babd93eb3&scene=21#wechat_redirect)

直接在Claude Code 控制台输入提示词即可，会自动触发该技能：

| 输入示例 | 效果 |
| --- | --- |
| 画一张多智能体协作图，暗黑终端风格 | 输出多 Agent 协作架构图，暗色背景 + 霓虹配色 |
| 画一个Mem0记忆架构图，蓝图风格 | Mem0 内存架构图，深蓝底 + 网格线 |

### 完整实操流程

假设我们要生成一个 RAG 系统架构图，在 Claude Code 中输入：

```
画一张 RAG 架构图：用户把问题发送给 API Gateway，API Gateway 将请求路由到 LLM 服务，LLM 先去 Pinecone 向量数据库检索相关文档，再把文档拼接进 prompt 回答用户。风格用 dark terminal。
```

几秒钟后，Skill 自动完成以下工作：

```
图类型识别 → 判断为 RAG 流水线架构图
```

```
语义形状映射 → LLM 用双边框矩形、Vector Store 用带环圆柱
```

```
箭头语义编码 → “query → vector store”用青色箭头，“retrieve documents”用虚线
```

```
品牌匹配 → Pinecone 自动带官方 logo + 品牌色 #000000 accent
```

```
风格套用 → 暗黑终端风格自动应用黑底 + 霓虹绿/青/粉 + 等宽字体
```

```
输出双格式 → rag-architecture.svg（可编辑）+ rag-architecture.png（1920px 高清）
```

```
特别亮点：想换一种风格？只需改一个词——把 dark terminal 换成 blueprint，整张图的颜色、字体、网格背景全部自动重新生成。
```

完成的效果，会在当前项目根目录下生产svg和png 两张图片：

风格速查表

| 风格名称 | 背景 | 适合场景 |
| --- | --- | --- |
| Flat Icon（默认） | #ffffff | 博客、PPT、文档 |
| Dark Terminal | #0f0f1a + SF Mono | GitHub README、技术博客 |
| Blueprint | #0a1628 + 网格线 | 架构文档、工程报告 |
| Notion Clean | #ffffff + 系统字体 | Notion、Confluence、wiki |
| Glassmorphism | 深色渐变 + 磨砂玻璃 | 产品演示、Keynote |

以上风格定义参考自 `Fireworks Tech Graph` 官方文档。第 6、7 种分别是 `Claude` 官方风格（暖白背景 + Anthropic 品牌色）和 OpenAI 官方风格（纯白背景 + OpenAI 品牌配色）。

### 实用参数扩展

#### 指定输出路径：

```
画一张Mem0架构图，输出到 ~/Desktop/
```

#### 指定风格：

```
画一张微服务架构图，风格 2
```

详细参数写法参考自官方 SKILL.md。

## 四、进阶：把效率推到极致

### 1. 使用 Helper Scripts 进行批量生成

`Fireworks Tech Graph` 仓库内置四个辅助脚本（位于 scripts/ 目录），适合自动化场景：

`generate-diagram.sh`：验证 SVG 语法并导出 PNG，支持 -t 指定图表类型、-s 指定风格

`validate-svg.sh`：检查 SVG 的 XML 语法、标签平衡、路径数据

`generate-from-template.py`：从 JSON 描述生成 SVG 模板

`test-all-styles.sh`：批量测试所有风格，生成验证报告

对于生产级图表生成场景，推荐使用脚本，它能自动校验避免语法错误。

### 2. 用 CLAUDE.md 建立项目记忆

Claude Code 支持通过项目根目录的 CLAUDE.md 文件注入上下文。可以这样配置：

```
# CLAUDE.md — 项目上下文  
  
## 项目概述  
- 项目类型：AI Agent 服务平台  
- 核心模块：API Gateway / Agent Orchestrator / Vector Store / LLM Router  
  
## 图表规范  
- 架构图风格：默认使用 Dark Terminal  
- RAG 类图：必须包含 Embedding → Retrieval → Generation 三层  
- 输出路径：./docs/diagrams/
```

配置后，`Claude Code` 会优先理解你的架构规范和输出偏好，生成图表时自动遵循这些规则。

### 3. 与 Git 工作流集成

将生成的图表纳入版本管理，一个推荐的工作流：

`Claude Code` 生成 SVG + PNG 并存放在 ./docs/diagrams/

手动确认或微调 SVG 文件（用 Figma/Illustrator 打开可继续编辑）

执行

```
git add ./docs/diagrams/ && git commit -m "docs: update architecture diagrams"
```

推送到仓库，PR 预览时图表自动展示

## 总结

如果你也在写技术文档、做架构方案、或者日常需要产出大量 AI/Agent 相关的配图，不妨今晚就动手试试。先把 Claude Code 装好，再 `claude skills install fireworks-tech-graph`，让 AI 替你画出第一张图。

当然更多更细节的内容，也可以去github【https://github.com/yizhiyanhua-ai/fireworks-tech-graph】 去查看！

安装过程中遇到任何问题，欢迎在评论区留言交流!

[一天一个SKILL——把一本厚厚的PDF技术书转为 Claude 随身技能，内部文档直接变身工作能力](https://mp.weixin.qq.com/s?__biz=MzUyODE4MjQxOQ==&mid=2247484579&idx=1&sn=a7ee508f94307159113c38ff2c772178&scene=21#wechat_redirect)

[还在勤勤恳恳录制剪辑视频？HyperFrames 让Claude Code 用 HTML 三分钟  "写出" 一个视频](https://mp.weixin.qq.com/s?__biz=MzUyODE4MjQxOQ==&mid=2247484598&idx=1&sn=f61f8356915cad1ef7a05c0c605531ac&scene=21#wechat_redirect)

[不想做PPT？Claude Code+Obsidian 三分钟完成一个高质感PPT](https://mp.weixin.qq.com/s?__biz=MzUyODE4MjQxOQ==&mid=2247484570&idx=1&sn=4a0893623acc55b07388584fc6891ec2&scene=21#wechat_redirect)