> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020303_RAG/020303_核心知识点/RAG生命周期与评估边界|RAG生命周期与评估边界]]
---
title: UI Design Brain：给 Cursor 增加 60+ 组件设计知识库，让 AI 生成更像“资深产品设计”
author: Endorphins.cn
date:
url: https://mp.weixin.qq.com/s?__biz=MzI5NDU2MTI3NQ==&mid=2247484815&idx=3&sn=28cd73b1f7e7e89362331751a58c8dcb&chksm=eda6c4a3e2aaf11bc1424bb762292c5262c97afa895ec56f9c5d3883343a66e274b498879d49&mpshare=1&scene=24&srcid=0309oPWUuhiPyC5D6wbNeHBt&sharer_shareinfo=b48cf1b2a62356796e6da66089ac15cd&sharer_shareinfo_first=b48cf1b2a62356796e6da66089ac15cd#rd
---

很多人用 AI 写前端时会遇到同一个问题：代码能跑，但 UI “像拼出来的”，缺少成熟产品的布局逻辑、组件细节与可用性约束。原因很简单：模型会猜，但它不知道你团队的设计系统与组件最佳实践。

**UI Design Brain** 是一个 Cursor skill：把 60+ 常见界面组件的“最佳实践、布局模式、别名、反模式、无障碍要点”整理成可被 AI 检索与遵循的知识库，让 AI 生成 UI 时更像在用真实设计系统。

项目地址：https://github.com/carmahhawwari/ui-design-brain[1]

## 1）它做了什么？

* 覆盖 60 个组件：Button、Table、Modal、Form、Tabs、Toast、Tooltip、Pagination…
* 每个组件提供：
  + best practices（交互/尺寸/状态/可访问性）
  + common layouts（成熟布局组合）
  + aliases（同义词，方便识别需求）
  + anti-patterns（明确禁止）
* 提供 5 个风格预设：Modern SaaS、Apple Minimal、Enterprise、Creative、Data Dashboard

## 2）安装方式

* 个人全局：clone 到 `~/.cursor/skills/ui-design-brain`
* 项目共享：clone 到 `.cursor/skills/ui-design-brain`

安装后无需显式引用，按 README 描述会在“请求构建 UI”时自动触发。

## 3）注意事项

* 这是“知识库增强”，不是 UI 组件库本身：最终仍要结合团队的具体组件实现（shadcn/ui、MUI、AntD 等）
* 组件数据来源于 component.gallery，建议团队二次校对与本地化
* 无障碍建议应结合实际 QA（键盘可达、ARIA、对比度）验证

---

当 AI 写前端进入“能上线”的阶段，差距往往不在语法，而在组件知识、布局习惯与反模式规避。UI Design Brain 把这部分经验从人的脑子里搬进了 skill。

### 引用链接

[1]*https://github.com/carmahhawwari/ui-design-brain*